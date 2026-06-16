---
name: ocr-document
description: "Extrait le texte d'un document SCANNE, d'une image ou d'une photo (PDF scanne, JPG, PNG, capture d'ecran) en EXECUTANT DU CODE (interpreteur souverain) : reconnaissance optique de caracteres (OCR) avec pytesseract, pretraitement de l'image avec OpenCV (niveaux de gris, binarisation, redressement), traitement d'un PDF scanne page par page (pdf2image), reconstruction du texte et, en option, generation d'un PDF cherchable. Langues francais + anglais. Utiliser cette skill des que l'utilisateur veut extraire le texte d'un document SCANNE, d'une image ou d'une photo, faire de l'OCR, lire un PDF non selectionnable, numeriser un document, ou rendre un PDF scanne cherchable. Ne pas utiliser pour un PDF qui contient deja du texte selectionnable (skill pdf-extraction) : dans ce cas il n'y a pas d'OCR a faire."
when-to-use: "L'utilisateur veut recuperer le texte d'un document image : PDF scanne ou photographie, JPG/PNG, capture d'ecran, document numerise. Faire de l'OCR, rendre un scan cherchable, numeriser un papier. Pas pour un PDF dont le texte est deja selectionnable."
always-apply: false
---

# OCR de documents scannes et d'images

Tu extrais le texte d'un document **image** (PDF scanne, photo, JPG, PNG, capture d'ecran) **en executant du code** dans l'interpreteur souverain. Tu ne devines jamais le contenu : tu le lis reellement avec l'OCR, et tu restitues le texte tel que reconnu.

**Ce que tu fais :**
- Pretraiter l'image (niveaux de gris, binarisation, redressement) pour ameliorer la reconnaissance.
- Reconnaitre le texte avec **pytesseract** (langues **francais + anglais**).
- Traiter un PDF scanne **page par page** (pdf2image) et reconstruire le texte complet.
- En option, produire un **PDF cherchable** (texte selectionnable par-dessus l'image).

**Ce que tu ne fais pas :**
- OCR sur un PDF qui a deja du texte selectionnable : inutile, c'est la skill **pdf-extraction**. Verifie d'abord (voir Etape 0).
- Garantir une transcription parfaite : la qualite depend du scan. Tu signales les zones douteuses.

## Environnement

- **Python** (langage **`py`**) : `pytesseract`, `pdf2image`, `pillow` (import `PIL`), `opencv-python-headless` (import `cv2`) installes.
- **Fichiers** dans **`/mnt/data`** (persistants entre tes appels de code d'une meme conversation).
- **Bash** (langage **`bash`**) disponible pour les binaires systeme (LibreOffice/`soffice`).

> ATTENTION dependances binaires. Les **bibliotheques Python** sont installees, mais elles s'appuient sur des **binaires systeme** qui ne sont **pas garantis** dans le sandbox :
> - `pytesseract` a besoin du binaire **`tesseract`** (et, pour le francais, du **pack de langue `fra`**).
> - `pdf2image` a besoin de **poppler** (binaire **`pdftoppm`**).
>
> Tu **verifies leur presence au tout debut** (Etape 1) et tu adaptes : si un binaire manque, tu le dis clairement et tu proposes le repli (Etape 6). Ne fais jamais comme si l'OCR avait reussi alors qu'il a echoue faute de binaire.

## Etape 0 (py) — Le document est-il deja cherchable ?

Si c'est un PDF, verifie d'abord qu'il n'a pas deja du texte selectionnable. Si oui, **arrete-toi** : c'est la skill pdf-extraction, pas d'OCR a faire.

```python
# Test rapide : ce PDF contient-il deja du texte ?
try:
    from pypdf import PdfReader
    r = PdfReader('/mnt/data/doc.pdf')
    txt = "".join((p.extract_text() or "") for p in r.pages[:3]).strip()
    if len(txt) > 50:
        print("Ce PDF contient deja du texte selectionnable -> pas d'OCR (voir pdf-extraction).")
    else:
        print("Peu ou pas de texte extractible -> c'est bien un scan, on fait l'OCR.")
except Exception as e:
    print("pypdf indisponible ou echec, on traite comme un scan :", e)
```

## Etape 1 (py) — Verifier les dependances binaires (TOUJOURS)

```python
import shutil

def check_tesseract():
    path = shutil.which('tesseract')
    if not path:
        return False, [], "Binaire 'tesseract' INTROUVABLE."
    try:
        import pytesseract
        langs = list(pytesseract.get_languages(config=''))
    except Exception as e:
        return True, [], f"tesseract present mais langues illisibles : {e}"
    return True, langs, f"tesseract OK ({path}), langues : {langs}"

def check_poppler():
    path = shutil.which('pdftoppm')
    return (path is not None), (path or "INTROUVABLE")

t_ok, t_langs, t_msg = check_tesseract()
p_ok, p_path = check_poppler()
print(t_msg)
print("poppler/pdftoppm :", "OK" if p_ok else "INTROUVABLE", "-", p_path)

has_fra = 'fra' in t_langs
if t_ok and not has_fra:
    print("ATTENTION : pack de langue 'fra' absent -> OCR en anglais ('eng') seulement, accents/francais degrades.")
```

Choisis la suite selon le resultat :
- **`tesseract` present** -> Etapes 2 a 5 (OCR normal). Langue : `fra+eng` si `fra` dispo, sinon `eng`.
- **`tesseract` absent** -> tu ne peux pas faire d'OCR ici. Va a l'Etape 6 (repli / limite) et sois honnete avec l'utilisateur.
- **`poppler` absent** mais entree = PDF -> tu ne peux pas convertir le PDF en images via pdf2image. Repli Bash en Etape 6 pour produire les images, puis OCR si tesseract present.

## Etape 2 (py) — Pretraitement de l'image (OpenCV)

Un bon pretraitement change tout sur un scan ou une photo. Niveaux de gris, debruitage, binarisation (Otsu), et redressement automatique de l'inclinaison.

```python
import cv2
import numpy as np

def deskew(gray):
    """Redresse une image legerement inclinee (skew)."""
    inv = cv2.bitwise_not(gray)
    thr = cv2.threshold(inv, 0, 255, cv2.THRESH_BINARY | cv2.THRESH_OTSU)[1]
    coords = np.column_stack(np.where(thr > 0))
    if coords.shape[0] < 50:
        return gray
    angle = cv2.minAreaRect(coords)[-1]
    angle = -(90 + angle) if angle < -45 else -angle
    if abs(angle) < 0.3:
        return gray
    h, w = gray.shape
    M = cv2.getRotationMatrix2D((w / 2, h / 2), angle, 1.0)
    return cv2.warpAffine(gray, M, (w, h),
                          flags=cv2.INTER_CUBIC,
                          borderMode=cv2.BORDER_REPLICATE)

def preprocess(img_bgr):
    """BGR (OpenCV) -> image binaire propre, prete pour l'OCR."""
    gray = cv2.cvtColor(img_bgr, cv2.COLOR_BGR2GRAY)
    gray = deskew(gray)
    gray = cv2.fastNlMeansDenoising(gray, h=10)          # debruitage doux
    # Binarisation adaptative : robuste a l'eclairage inegal d'une photo
    bw = cv2.adaptiveThreshold(gray, 255,
                               cv2.ADAPTIVE_THRESH_GAUSSIAN_C,
                               cv2.THRESH_BINARY, 31, 15)
    return bw
```

Conseils : pour un document photographie a faible resolution, agrandir (`cv2.resize`, facteur 1.5 a 2, interpolation `INTER_CUBIC`) avant l'OCR ameliore nettement le resultat. Pour un scan deja propre, une simple binarisation Otsu (`cv2.threshold(..., THRESH_OTSU)`) suffit souvent et evite le bruit de l'adaptatif.

## Etape 3 (py) — OCR d'une image (pytesseract)

```python
import pytesseract
from PIL import Image

# Langue : 'fra+eng' si dispo, sinon 'eng' (cf. Etape 1)
LANG = 'fra+eng' if has_fra else 'eng'

# config : --oem 1 (moteur LSTM), --psm 3 (page complete, mise en page auto)
OCR_CONFIG = r'--oem 1 --psm 3'

def ocr_image_path(path):
    img = cv2.imread(path)                 # BGR
    bw = preprocess(img)
    pil = Image.fromarray(bw)
    return pytesseract.image_to_string(pil, lang=LANG, config=OCR_CONFIG)

texte = ocr_image_path('/mnt/data/photo.jpg')
print(texte[:1500])
```

Reglages utiles de `--psm` : `3` page complete (defaut), `6` bloc de texte uniforme (utile pour un paragraphe ou une colonne), `4` colonnes de tailles variables, `11`/`12` texte epars. Pour un score de confiance par mot, utiliser `pytesseract.image_to_data(..., output_type=pytesseract.Output.DICT)` et filtrer les mots dont `conf` est bas pour reperer les zones douteuses.

## Etape 4 (py) — PDF scanne, page par page (pdf2image)

```python
from pdf2image import convert_from_path

# dpi 300 = bon compromis qualite/temps pour l'OCR
pages = convert_from_path('/mnt/data/doc.pdf', dpi=300)   # liste d'images PIL
print(f"{len(pages)} page(s)")

morceaux = []
for i, page in enumerate(pages, 1):
    bgr = cv2.cvtColor(np.array(page), cv2.COLOR_RGB2BGR)  # PIL(RGB) -> OpenCV(BGR)
    bw = preprocess(bgr)
    t = pytesseract.image_to_string(Image.fromarray(bw), lang=LANG, config=OCR_CONFIG)
    morceaux.append(f"\n\n===== Page {i} =====\n{t.strip()}")
    print(f"Page {i} : {len(t.split())} mots")

texte_complet = "".join(morceaux).strip()
with open('/mnt/data/texte_ocr.txt', 'w', encoding='utf-8') as f:
    f.write(texte_complet)
print("Texte ecrit dans /mnt/data/texte_ocr.txt")
```

## Etape 5 (py, option) — PDF cherchable

Genere un PDF qui superpose une couche de texte selectionnable sur l'image d'origine (utile pour archiver un scan tout en pouvant le rechercher). `pytesseract.image_to_pdf_or_hocr` produit un PDF par page ; on les assemble.

```python
from pypdf import PdfWriter
import io

def image_to_searchable_pdf_bytes(bgr):
    bw = preprocess(bgr)
    return pytesseract.image_to_pdf_or_hocr(
        Image.fromarray(bw), lang=LANG, extension='pdf')

writer = PdfWriter()
for page in pages:                       # 'pages' vient de l'Etape 4
    bgr = cv2.cvtColor(np.array(page), cv2.COLOR_RGB2BGR)
    pdf_bytes = image_to_searchable_pdf_bytes(bgr)
    from pypdf import PdfReader
    for p in PdfReader(io.BytesIO(pdf_bytes)).pages:
        writer.add_page(p)

with open('/mnt/data/doc_cherchable.pdf', 'wb') as f:
    writer.write(f)
print("PDF cherchable : /mnt/data/doc_cherchable.pdf")
```

## Etape 6 — Repli et limites (quand un binaire manque)

Sois **honnete**. Si l'OCR n'est pas possible dans le sandbox, dis-le clairement plutot que d'inventer un texte.

**Cas A — `poppler` absent, entree = PDF (mais `tesseract` present).** Convertis le PDF en images en Bash avec LibreOffice, puis reprends l'OCR (Etape 3) sur les images produites.

```bash
# Rasterise chaque page du PDF en PNG via LibreOffice (sans poppler)
mkdir -p /mnt/data/pages
soffice --headless --convert-to pdf --outdir /mnt/data /mnt/data/doc.pdf 2>/dev/null
# A defaut de pdftoppm, on peut aussi passer par pdftocairo si poppler-utils partiel :
command -v pdftocairo >/dev/null && pdftocairo -png -r 300 /mnt/data/doc.pdf /mnt/data/pages/p
ls -la /mnt/data/pages
```

Si LibreOffice ne sait pas rasteriser directement en PNG, une autre voie est `PyMuPDF` (import `fitz`, s'il est installe) : `fitz.open(pdf).load_page(i).get_pixmap(dpi=300).save(png)` ne depend d'aucun binaire externe. Teste sa presence avant de l'utiliser.

**Cas B — `tesseract` absent.** Aucun OCR local possible. Explique-le a l'utilisateur sans detour : indique que le moteur OCR n'est pas disponible dans cet environnement, propose une alternative (fournir une image plus nette, ou un document deja en texte), et ne fabrique pas de contenu. Si seul le pack `fra` manque, precise que le francais sera mal reconnu (accents, ligatures) et propose l'OCR en anglais a titre indicatif.

## Restitution a l'utilisateur

- Donne le **texte reconnu** (ou le fichier `/mnt/data/texte_ocr.txt`), et le **PDF cherchable** si demande.
- Precise la **langue OCR utilisee** (`fra+eng` ou `eng`) et, si pertinent, le **nombre de pages** traitees.
- Signale les **limites observees** : pages illisibles, faible confiance, mots douteux, binaire manquant. Mieux vaut une reserve claire qu'une fausse certitude.
