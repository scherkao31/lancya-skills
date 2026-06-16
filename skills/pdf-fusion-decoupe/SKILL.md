---
name: pdf-fusion-decoupe
description: "Manipule des fichiers PDF deja existants en EXECUTANT DU CODE (interpreteur souverain) : fusionner/assembler plusieurs PDF en un seul, decouper/separer un PDF en plusieurs, extraire des pages precises, reordonner les pages, faire pivoter des pages, ajouter un filigrane (watermark) texte, et compresser un PDF trop lourd. S'appuie sur pypdf (PdfWriter / PdfReader) pour l'assemblage et la rotation, reportlab pour generer l'overlay du filigrane, et LibreOffice (soffice) en bash pour la compression. Utiliser cette skill des que l'utilisateur veut fusionner, assembler, decouper, separer, reordonner, faire pivoter, extraire des pages, ajouter un filigrane ou compresser des PDF. Ne pas utiliser pour extraire le texte d'un PDF (skill pdf-extraction), remplir un formulaire PDF (skill pdf-formulaires), ni creer un PDF de zero a partir de contenu redige (skill pdf-creation)."
when-to-use: "Le PDF est l'entree ET la sortie : assembler plusieurs PDF, en separer un, extraire/reordonner/pivoter des pages, poser un filigrane texte, ou reduire le poids d'un fichier. Operations structurelles sur des pages, pas de lecture ni de redaction de contenu."
always-apply: false
---

# Manipulation de PDF (fusion, decoupe, rotation, filigrane, compression)

Tu agis sur des PDF **existants** en **executant du code**. Le contenu des pages reste intact : tu reorganises, assembles, separes, fais tourner ou habilles les pages, jamais tu ne les reecris. Livre toujours un (ou plusieurs) fichier dans `/mnt/data`.

## Environnement

- **Python** (`py`) : `pypdf` (API moderne ; `PyPDF2` est l'ancien nom, accepte en repli) pour lire/assembler/pivoter, `reportlab` pour fabriquer l'overlay du filigrane.
- **LibreOffice** (`soffice`) : **uniquement en Bash** (`bash`) — utile pour la compression.
- **Fichiers** dans **`/mnt/data`** (persistant entre tes appels de code). Les fichiers fournis par l'utilisateur y sont deja ; ecris tes sorties au meme endroit.

Import standard a placer en tete de chaque bloc Python :

```python
from PyPDF2 import PdfReader, PdfWriter   # repli : from PyPDF2 import PdfReader, PdfWriter
```

## Regle prealable — fichiers proteges par mot de passe

Avant toute operation, ouvre chaque source et **dechiffre si besoin**. Un PDF chiffre fait echouer l'assemblage silencieusement ou leve une erreur.

```python
def ouvrir(chemin, mot_de_passe=None):
    r = PdfReader(chemin)
    if r.is_encrypted:
        # mot de passe vide d'abord (beaucoup de PDF sont chiffres sans mot de passe utilisateur)
        if r.decrypt(mot_de_passe or "") == 0:
            raise ValueError(f"Mot de passe requis ou incorrect pour {chemin}")
    return r
```

Si l'utilisateur n'a pas donne le mot de passe et que `decrypt("")` echoue, demande-le avant de continuer.

## Fusionner / assembler plusieurs PDF

`PdfWriter.append(...)` copie tout le document (ou une selection de pages) a la suite. C'est la methode moderne recommandee.

```python
writer = PdfWriter()
for chemin in ["/mnt/data/a.pdf", "/mnt/data/b.pdf", "/mnt/data/c.pdf"]:
    writer.append(ouvrir(chemin))          # tout le document, dans l'ordre
# Selection de pages a l'assemblage : writer.append(ouvrir(chemin), pages=(0, 3))  # pages 1 a 3
with open("/mnt/data/fusion.pdf", "wb") as f:
    writer.write(f)
print("Fusion ->", len(PdfReader("/mnt/data/fusion.pdf").pages), "pages")
```

Note : les numeros de page de pypdf commencent a **0**. La page 1 humaine = index `0`.

## Decouper / separer par plages de pages

Decoupe un PDF en plusieurs fichiers selon des plages (numerotation humaine, 1-base, conversion en index ci-dessous).

```python
src = ouvrir("/mnt/data/source.pdf")
plages = {"partie1": (1, 5), "partie2": (6, 10), "partie3": (11, None)}  # (debut, fin) inclus ; None = jusqu'a la fin
for nom, (debut, fin) in plages.items():
    fin = fin or len(src.pages)
    w = PdfWriter()
    for i in range(debut - 1, fin):        # -1 : passage humain -> index
        w.add_page(src.pages[i])
    with open(f"/mnt/data/{nom}.pdf", "wb") as f:
        w.write(f)
    print(nom, "->", fin - debut + 1, "pages")
```

Pour eclater **une page par fichier** : boucle sur `enumerate(src.pages)`, un `PdfWriter` par page.

## Extraire des pages precises

Sortie d'un seul PDF ne contenant que les pages voulues (ici pages 1, 3 et 7, dans cet ordre).

```python
src = ouvrir("/mnt/data/source.pdf")
voulues = [1, 3, 7]                         # numerotation humaine
w = PdfWriter()
for n in voulues:
    w.add_page(src.pages[n - 1])
with open("/mnt/data/extrait.pdf", "wb") as f:
    w.write(f)
```

## Reordonner les pages

Fournis l'ordre cible (numerotation humaine). L'exemple inverse un document de 4 pages, mais `ordre` peut etre n'importe quelle permutation.

```python
src = ouvrir("/mnt/data/source.pdf")
ordre = [4, 3, 2, 1]                        # nouvel ordre voulu
w = PdfWriter()
for n in ordre:
    w.add_page(src.pages[n - 1])
with open("/mnt/data/reordonne.pdf", "wb") as f:
    w.write(f)
```

## Faire pivoter des pages

`page.rotate(angle)` tourne dans le sens horaire ; l'angle doit etre un multiple de 90 (90, 180, 270, -90).

```python
src = ouvrir("/mnt/data/source.pdf")
w = PdfWriter()
a_pivoter = {2: 90, 5: 180}                 # page humaine -> angle ; les autres restent telles quelles
for i, page in enumerate(src.pages, start=1):
    if i in a_pivoter:
        page.rotate(a_pivoter[i])
    w.add_page(page)
with open("/mnt/data/pivote.pdf", "wb") as f:
    w.write(f)
```

Pour pivoter **toutes** les pages : applique `page.rotate(90)` a chacune dans la boucle.

## Ajouter un filigrane (watermark) texte

Deux temps : reportlab fabrique une page transparente portant le texte en diagonale, puis on **fusionne** cet overlay sur chaque page du PDF source via `page.merge_page(overlay)`.

```python
import io
from reportlab.pdfgen import canvas
from reportlab.lib.colors import Color

def faire_overlay(largeur, hauteur, texte):
    buf = io.BytesIO()
    c = canvas.Canvas(buf, pagesize=(largeur, hauteur))
    c.saveState()
    c.translate(largeur / 2, hauteur / 2)
    c.rotate(45)
    c.setFont("Helvetica-Bold", 48)
    c.setFillColor(Color(0.5, 0.5, 0.5, alpha=0.25))   # gris translucide
    c.drawCentredString(0, 0, texte)
    c.restoreState()
    c.save()
    buf.seek(0)
    return PdfReader(buf).pages[0]

src = ouvrir("/mnt/data/source.pdf")
w = PdfWriter()
for page in src.pages:
    # overlay aux dimensions exactes de la page (gere les formats mixtes)
    overlay = faire_overlay(float(page.mediabox.width), float(page.mediabox.height), "CONFIDENTIEL")
    page.merge_page(overlay)                # depose le filigrane PAR-DESSUS le contenu
    w.add_page(page)
with open("/mnt/data/filigrane.pdf", "wb") as f:
    w.write(f)
```

Filigrane image (logo) plutot que texte : dessine-le dans reportlab avec `c.drawImage("/mnt/data/logo.png", x, y, width=..., height=..., mask='auto')` et garde le meme `merge_page`.

## Compresser un PDF trop lourd

Deux approches, de la plus douce a la plus efficace.

**A. pypdf seul** — recompresse les flux de contenu et deduplique les objets. Sans perte, gain modere.

```python
src = ouvrir("/mnt/data/source.pdf")
w = PdfWriter()
for page in src.pages:
    w.add_page(page)
for page in w.pages:
    page.compress_content_streams()         # recompresse le contenu (peut etre lent sur gros volumes)
# reduit les images bitmap si presentes :
for page in w.pages:
    for img in page.images:
        img.replace(img.image, quality=60)  # ne s'applique qu'aux pages avec images raster
with open("/mnt/data/compresse.pdf", "wb") as f:
    w.write(f)
```

**B. LibreOffice (bash)** — souvent le meilleur ratio quand le PDF contient des images lourdes. A lancer apres avoir exporte/normalise le fichier.

```bash
soffice --headless -env:UserInstallation=file:///tmp/lo_profile \
  --convert-to "pdf:writer_pdf_Export:{\"Quality\":{\"type\":\"long\",\"value\":\"60\"},\"ReduceImageResolution\":{\"type\":\"boolean\",\"value\":true},\"MaxImageResolution\":{\"type\":\"long\",\"value\":\"150\"}}" \
  --outdir /mnt/data /mnt/data/source.pdf
```

Compare ensuite les tailles avant/apres et garde la plus petite version qui reste lisible :

```python
import os
for n in ["source.pdf", "compresse.pdf"]:
    p = f"/mnt/data/{n}"
    if os.path.exists(p):
        print(n, round(os.path.getsize(p) / 1024, 1), "Ko")
```

## Gros volumes

- Ne charge pas tout en memoire inutilement : traite/ecris page par page comme ci-dessus, c'est deja en flux.
- `compress_content_streams()` est couteux en CPU : sur un PDF de plusieurs centaines de pages, previens que l'operation prend du temps, ou limite-la aux pages reellement lourdes.
- Pour fusionner beaucoup de fichiers, `writer.append(...)` dans une boucle suffit ; ferme/ecris une seule fois a la fin.
- Verifie toujours le resultat : `len(PdfReader(sortie).pages)` doit correspondre a ce qui etait attendu.

## Workflow type

1. Identifie l'operation demandee (fusion, decoupe, extraction, reordonnancement, rotation, filigrane, compression) et les fichiers dans `/mnt/data`.
2. Ouvre chaque source avec `ouvrir(...)` (dechiffre si chiffre ; demande le mot de passe si necessaire).
3. Construis le `PdfWriter` avec le bloc correspondant ci-dessus. Rappel : indices pypdf en 0-base, numerotation utilisateur en 1-base.
4. Ecris la sortie dans `/mnt/data`, puis verifie le nombre de pages (et la taille pour la compression).
5. Annonce clairement le ou les fichiers produits et ce qu'ils contiennent.

## Rappels

- Tu manipules des pages, tu ne reecris jamais le contenu. Pour lire le texte -> pdf-extraction ; remplir un formulaire -> pdf-formulaires ; rediger un PDF neuf -> pdf-creation.
- API pypdf moderne : `PdfReader` / `PdfWriter`, `writer.append`, `writer.add_page`, `page.rotate`, `page.merge_page`, `page.compress_content_streams`. Numerotation interne a partir de 0.
- Toujours gerer le cas chiffre via `is_encrypted` + `decrypt`.
- Documentation de reference : https://pypdf.readthedocs.io/ et https://pypdf.readthedocs.io/en/stable/user/merging-pdfs.html
