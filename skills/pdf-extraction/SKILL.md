---
name: pdf-extraction
description: "Extrait et analyse le contenu reel d'un PDF en EXECUTANT DU CODE dans l'interpreteur souverain : texte (global ou page par page) avec pdfminer.six, tableaux (heuristique de colonnes ou conversion + pandas), metadonnees et nombre de pages avec PyPDF2, detection PDF scanne vs PDF texte, extraction d'une plage de pages, sortie structuree en Markdown ou CSV. Gere les PDF proteges par mot de passe et les PDF scannes (OCR). Utiliser cette skill des que l'utilisateur veut extraire, lire, analyser, depouiller, recuperer ou exploiter le contenu d'un PDF (texte, tableaux, donnees, metadonnees, structure). Ne pas utiliser pour remplir un formulaire PDF (pdf-formulaires), pour fusionner ou decouper des PDF (pdf-fusion-decoupe), ni pour creer un PDF de zero (pdf-creation)."
when-to-use: "L'utilisateur veut extraire ou analyser le contenu d'un PDF joint : recuperer le texte, des tableaux, des metadonnees, ou une plage de pages, et obtenir une sortie exploitable."
always-apply: false
---

# Extraction et analyse de PDF (par code)

Tu extrais le contenu **reel** d'un PDF en **executant du code** dans l'interpreteur, jamais de memoire ni en supposant ce que contient le fichier. Tu ouvres le fichier, tu lis ce qu'il contient vraiment, puis tu produis une sortie propre et exploitable.

## Environnement

- **Python** (`py`) : `pdfminer.six` (texte et mise en page), `PyPDF2` (metadonnees, pages, dechiffrement), `pdf2image` + `pytesseract` (rendu image et OCR), `pandas` (tableaux et CSV).
- **LibreOffice / soffice** : **uniquement en Bash** (`bash`), utile pour convertir un PDF ou produire un autre format.
- **Fichiers** dans **`/mnt/data`** (persistants entre tes appels de code). Le PDF fourni y est generalement deja depose ; sinon, demande son chemin exact.

## Workflow

1. **Diagnostic d'abord** (etape 1) : nombre de pages, metadonnees, chiffrement, et surtout **texte ou scanne**. Ce diagnostic decide de toute la suite.
2. **Choisis la methode** selon le besoin de l'utilisateur : texte complet, texte d'une plage de pages, tableaux, ou metadonnees seules.
3. **Produis une sortie structuree** : Markdown pour du texte lisible, CSV (via pandas) pour des tableaux.
4. **Verifie** que tu as bien extrait quelque chose (longueur du texte non nulle) avant de presenter le resultat.

## Etape 1 (py) — Diagnostic du PDF

A lancer en premier sur tout PDF. Il dit combien de pages, s'il est chiffre, et s'il est **scanne** (image sans couche texte) ou **texte**.

```python
from pypdf import PdfReader            # PyPDF2 recent expose l'API sous "pypdf"
from pdfminer.high_level import extract_text

SRC = "/mnt/data/in.pdf"              # adapter au vrai nom du fichier

reader = PdfReader(SRC)
chiffre = reader.is_encrypted
if chiffre:
    # PDF protege : tenter un mot de passe vide (cas frequent), sinon demander le mot de passe
    try:
        reader.decrypt("")
        chiffre = False
    except Exception:
        pass

n_pages = len(reader.pages)
meta = reader.metadata or {}
print("Pages :", n_pages)
print("Chiffre :", chiffre)
print("Titre :", meta.get("/Title"))
print("Auteur :", meta.get("/Author"))
print("Producteur :", meta.get("/Producer"))
print("Cree le :", meta.get("/CreationDate"))

# Texte ou scanne ? On extrait le texte des premieres pages et on mesure.
echantillon = extract_text(SRC, page_numbers=list(range(min(3, n_pages)))) if not chiffre else ""
n_car = len((echantillon or "").strip())
print("Caracteres extraits (3 pages) :", n_car)
print("=> PDF SCANNE (OCR requis)" if n_car < 50 else "=> PDF TEXTE (extraction directe)")
```

Lis le verdict avant de continuer :

- **PDF TEXTE** : passe a l'extraction directe (etapes 2 a 4).
- **PDF SCANNE** : la couche texte est absente, l'extraction directe renverra du vide. Fais de l'**OCR** (etape 5) ou oriente vers la skill **ocr-document** si elle est disponible.
- **PDF chiffre** sans mot de passe vide : demande le mot de passe a l'utilisateur, puis `reader.decrypt("mot_de_passe")` avant toute extraction.

## Etape 2 (py) — Extraire le texte (global ou page par page)

`extract_text` rend le texte complet. Pour le texte page par page (utile pour citer une page, ou traiter un gros fichier sans tout charger), on itere avec `extract_pages`.

```python
from pdfminer.high_level import extract_text, extract_pages
from pdfminer.layout import LTTextContainer

# Texte complet
texte = extract_text(SRC)
print(texte[:2000])

# Texte page par page, dans un dictionnaire {numero_de_page: texte}
pages = {}
for i, layout in enumerate(extract_pages(SRC), start=1):
    morceaux = [el.get_text() for el in layout if isinstance(el, LTTextContainer)]
    pages[i] = "".join(morceaux).strip()

print("Page 1 :", pages.get(1, "")[:500])
```

## Etape 3 (py) — Extraire une plage de pages

Pour ne traiter que les pages utiles (ex. pages 5 a 12). `page_numbers` est indexe a partir de 0, donc page 5 = index 4.

```python
from pdfminer.high_level import extract_text

debut, fin = 5, 12                              # pages voulues, numerotation humaine (1-based)
indices = list(range(debut - 1, fin))           # -> indices 0-based pour pdfminer
texte_plage = extract_text(SRC, page_numbers=indices)
print(f"Texte des pages {debut} a {fin} :")
print(texte_plage[:2000])
```

## Etape 4 (py) — Extraire des tableaux

pdfminer ne « comprend » pas les tableaux : il rend des blocs de texte positionnes. On reconstitue les lignes et colonnes a partir des coordonnees. Approche robuste : regrouper les mots par ligne (meme position verticale), puis detecter les colonnes par les sauts horizontaux.

```python
import pandas as pd
from pdfminer.high_level import extract_pages
from pdfminer.layout import LTTextLine, LTTextContainer

PAGE_TABLEAU = 1            # page (1-based) ou se trouve le tableau

def lignes_de_page(src, page_1based):
    layouts = list(extract_pages(src))
    layout = layouts[page_1based - 1]
    lignes = []
    for el in layout:
        if isinstance(el, LTTextContainer):
            for ligne in el:
                if isinstance(ligne, LTTextLine):
                    txt = ligne.get_text().strip()
                    if txt:
                        # y0 = position verticale (pour trier), x0 = horizontale (pour ordonner)
                        lignes.append((round(ligne.y0), round(ligne.x0), txt))
    return lignes

brut = lignes_de_page(SRC, PAGE_TABLEAU)

# Regrouper par ligne : meme y a ~3 points pres = meme rangee du tableau
brut.sort(key=lambda t: (-t[0], t[1]))
rangees, courante, y_ref = [], [], None
for y, x, txt in brut:
    if y_ref is None or abs(y - y_ref) <= 3:
        courante.append(txt); y_ref = y if y_ref is None else y_ref
    else:
        rangees.append(courante); courante = [txt]; y_ref = y
if courante:
    rangees.append(courante)

# Normaliser le nombre de colonnes pour construire un DataFrame propre
largeur = max((len(r) for r in rangees), default=0)
rangees = [r + [""] * (largeur - len(r)) for r in rangees]
df = pd.DataFrame(rangees[1:], columns=rangees[0]) if rangees else pd.DataFrame()
print(df)

df.to_csv("/mnt/data/tableau.csv", index=False, encoding="utf-8-sig")  # utf-8-sig = accents OK dans Excel
```

Cette heuristique couvre les tableaux simples et reguliers. Pour des tableaux complexes (cellules fusionnees, bordures fines), une alternative fiable est de **convertir le PDF en classeur via LibreOffice** puis de lire avec pandas :

```bash
# Bash : LibreOffice convertit le PDF en .xlsx, pandas le relit ensuite proprement
soffice --headless -env:UserInstallation=file:///tmp/lo_profile \
  --convert-to xlsx --outdir /mnt/data /mnt/data/in.pdf
```

```python
import pandas as pd
feuilles = pd.read_excel("/mnt/data/in.xlsx", sheet_name=None)   # dict {feuille: DataFrame}
for nom, t in feuilles.items():
    print(nom, t.shape)
```

Si la conversion LibreOffice ne donne rien d'exploitable (PDF non structure), reste sur l'heuristique de colonnes ci-dessus et signale a l'utilisateur que le tableau a ete reconstruit au mieux.

## Etape 5 (py) — OCR d'un PDF scanne

Si le diagnostic a conclu **PDF SCANNE**, on rend chaque page en image puis on lit le texte avec Tesseract. Pour le francais, le pack `fra` doit etre installe (`lang="fra"`).

```python
from pdf2image import convert_from_path
import pytesseract

# dpi 300 = bon compromis qualite / vitesse ; first_page/last_page pour une plage
images = convert_from_path(SRC, dpi=300, first_page=1, last_page=5)

texte_ocr = []
for i, img in enumerate(images, start=1):
    t = pytesseract.image_to_string(img, lang="fra")   # "fra+eng" si document bilingue
    texte_ocr.append(f"\n----- Page {i} -----\n{t.strip()}")

resultat = "".join(texte_ocr)
print(resultat[:2000])
```

L'OCR est plus lent et imparfait (qualite du scan, polices, mise en page). Previens l'utilisateur que le texte OCR peut contenir des erreurs et merite une relecture. Si une skill **ocr-document** dediee existe dans l'environnement, oriente plutot vers elle.

## Etape 6 (py) — Sortie structuree

Selon ce que veut l'utilisateur :

- **Texte lisible** : assemble un Markdown avec un titre par page et restitue le contenu fidelement.
- **Donnees tabulaires** : un **CSV par tableau** (encodage `utf-8-sig`), plus un apercu en Markdown dans la reponse.
- **Metadonnees** : une petite table (titre, auteur, pages, date de creation, producteur).

```python
# Exemple : exporter le texte complet en Markdown, page par page
md = []
for i in sorted(pages):
    md.append(f"## Page {i}\n\n{pages[i]}\n")
with open("/mnt/data/extraction.md", "w", encoding="utf-8") as f:
    f.write("\n".join(md))
print("Ecrit : /mnt/data/extraction.md")
```

## Gros fichiers

- Ne charge pas tout en memoire d'un coup : traite **par plage de pages** (etape 3) ou itere avec `extract_pages` page par page (etape 2), en ecrivant le resultat au fur et a mesure dans un fichier de `/mnt/data`.
- Pour l'OCR d'un long document, decoupe en tranches (`first_page` / `last_page`) plutot que de rendre toutes les pages en images simultanement (la conversion en images est gourmande en memoire).
- Annonce a l'utilisateur si tu ne traites qu'une partie du fichier, et laquelle.

## Cas particuliers a gerer

- **PDF protege** : tenter `decrypt("")` ; si echec, demander le mot de passe, ne jamais deviner.
- **PDF scanne** : extraction directe vide -> OCR (etape 5) ou skill ocr-document.
- **Tableau mal reconstruit** : tenter la conversion LibreOffice (etape 4), sinon livrer le meilleur resultat en le signalant.
- **Texte vide alors que le PDF semble normal** : verifier qu'il n'est pas chiffre ou scanne avant de conclure.

## Fidelite et style

- **N'invente rien.** Restitue le contenu reellement extrait. Si une page est vide ou illisible, dis-le.
- Distingue le **contenu du PDF** de ton **interpretation** ou de ton resume.
- Ecris comme un humain, clair et sobre, dans la langue de l'utilisateur.
- **Jamais de tiret cadratin (—) ni demi-cadratin (–).**

## References

- pdfminer.six : https://pdfminersix.readthedocs.io/
- pypdf (ex-PyPDF2) : https://pypdf.readthedocs.io/
