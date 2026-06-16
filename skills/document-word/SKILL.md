---
name: document-word
description: "Cree, modifie et analyse de vrais documents Word (.docx) de niveau professionnel, en EXECUTANT DU CODE (interpreteur souverain) avec python-docx : titres et styles, table des matieres, tableaux mis en forme, images, en-tetes et pieds de page avec numerotation, listes a puces/numerotees, mise en page (format, marges, orientation, colonnes), sauts de page, hyperliens, et conversion en PDF. Utiliser cette skill des que l'utilisateur veut produire ou retravailler un document Word : lettre, courrier, rapport, memo, note, contrat, attestation, modele, compte rendu mis en forme, ou extraire/reorganiser le contenu d'un .docx existant. Ne pas utiliser pour un tableur Excel (skill tableur-excel), une presentation (html-slides), ni un PDF a remplir."
when-to-use: "L'utilisateur veut creer, modifier ou analyser un document Word (.docx) professionnel : lettre, rapport, memo, contrat, modele, avec mise en forme (titres, table des matieres, tableaux, images, en-tetes/pieds)."
always-apply: false
---

# Documents Word (.docx) de niveau pro, via python-docx

Tu produis de vrais documents Word **en executant du code** dans l'interpreteur (python-docx est installe). Tu ne livres pas un texte brut : un document **mis en page** (styles, titres, tableaux, en-tetes/pieds, numerotation), propre et professionnel.

## Environnement

- **Python** (langage **`py`**) : `python-docx` (import `docx`), pandas, etc. installes.
- **LibreOffice** pour convertir en PDF ou ouvrir un vieux `.doc` : **uniquement en Bash** (langage **`bash`**), via `soffice`.
- **Fichiers** dans **`/mnt/data`** (persistants entre tes appels de code d'une meme conversation).

## Principes

- **Police par defaut sobre** (Arial 11) sauf demande. Titres en noir, lisibles.
- **Un paragraphe = un `add_paragraph`** : jamais de `\n` dans un run pour sauter une ligne.
- **Jamais de puces tapees a la main** (`•`, `-`) : utiliser les **styles de liste** ("List Bullet", "List Number").
- Structure claire : titre, sections (titres de niveau), corps, et si pertinent table des matieres + en-tetes/pieds.

## Creation : base

```python
from docx import Document
from docx.shared import Pt, Cm, RGBColor
from docx.enum.text import WD_ALIGN_PARAGRAPH

doc = Document()
# Police par defaut
st = doc.styles['Normal']; st.font.name = 'Arial'; st.font.size = Pt(11)

doc.add_heading('Titre du document', level=0)        # titre principal
doc.add_heading('1. Introduction', level=1)           # H1
p = doc.add_paragraph('Texte normal. ')
run = p.add_run('Mot en gras.'); run.bold = True
p.add_run(' Et en ').italic = False
r = p.add_run('rouge'); r.font.color.rgb = RGBColor(0xC0, 0x00, 0x00)
p.alignment = WD_ALIGN_PARAGRAPH.JUSTIFY

doc.save('/mnt/data/out.docx')
```

## Titres et styles personnalises

`doc.add_heading(text, level=0..9)` utilise les styles "Heading N" (necessaires pour la table des matieres). Pour personnaliser un style :

```python
h1 = doc.styles['Heading 1']
h1.font.name='Arial'; h1.font.size=Pt(16); h1.font.bold=True
h1.font.color.rgb=RGBColor(0x1F,0x3A,0x5F)
```

## Listes (vraies puces / numerotation)

```python
doc.add_paragraph('Premier point', style='List Bullet')
doc.add_paragraph('Sous-point', style='List Bullet 2')   # niveau 2
doc.add_paragraph('Etape 1', style='List Number')
```

## Tableaux mis en forme

```python
from docx.shared import Pt
from docx.oxml.ns import qn
from docx.oxml import OxmlElement

def shade(cell, hexcolor):                       # fond de cellule
    tcPr = cell._tc.get_or_add_tcPr()
    sh = OxmlElement('w:shd'); sh.set(qn('w:fill'), hexcolor); tcPr.append(sh)

t = doc.add_table(rows=1, cols=3)
t.style = 'Light Grid Accent 1'                  # style integre avec bordures
hdr = t.rows[0].cells
for i, txt in enumerate(['Poste','Montant','Note']):
    hdr[i].text = txt
    hdr[i].paragraphs[0].runs[0].font.bold = True
    shade(hdr[i], '1F3A5F')
    hdr[i].paragraphs[0].runs[0].font.color.rgb = RGBColor(0xFF,0xFF,0xFF)
row = t.add_row().cells
row[0].text='Loyer'; row[1].text='2 500 CHF'; row[2].text=''
# Largeurs de colonnes
for r in t.rows:
    r.cells[0].width=Cm(7); r.cells[1].width=Cm(4); r.cells[2].width=Cm(5)
```

## Images

```python
doc.add_picture('/mnt/data/logo.png', width=Cm(4))
doc.paragraphs[-1].alignment = WD_ALIGN_PARAGRAPH.CENTER
```

## En-tetes, pieds de page et numerotation

```python
from docx.oxml.ns import qn
from docx.oxml import OxmlElement

sec = doc.sections[0]
sec.header.paragraphs[0].text = 'Lancya'
foot = sec.footer.paragraphs[0]; foot.alignment = WD_ALIGN_PARAGRAPH.CENTER
foot.add_run('Page ')
# champ "numero de page"
def page_field(par):
    r = par.add_run(); fld = OxmlElement('w:fldSimple'); fld.set(qn('w:instr'), 'PAGE')
    r._r.addnext(fld)
page_field(foot)
```

## Mise en page (format, marges, orientation)

```python
from docx.shared import Cm
from docx.enum.section import WD_ORIENT
s = doc.sections[0]
s.page_width = Cm(21.0); s.page_height = Cm(29.7)             # A4 portrait
s.top_margin=s.bottom_margin=Cm(2.5); s.left_margin=s.right_margin=Cm(2.5)
# Paysage : echanger largeur/hauteur + orientation
# s.orientation = WD_ORIENT.LANDSCAPE; s.page_width, s.page_height = s.page_height, s.page_width
```

## Sauts de page

```python
doc.add_page_break()
```

## Table des matieres

python-docx n'a pas de TOC native : on insere un champ Word qui se remplit a l'ouverture (ou apres conversion LibreOffice).

```python
from docx.oxml.ns import qn
from docx.oxml import OxmlElement
def add_toc(doc):
    p = doc.add_paragraph(); r = p.add_run()
    f1=OxmlElement('w:fldChar'); f1.set(qn('w:fldCharType'),'begin')
    instr=OxmlElement('w:instrText'); instr.set(qn('xml:space'),'preserve'); instr.text='TOC \\o "1-3" \\h \\z \\u'
    f2=OxmlElement('w:fldChar'); f2.set(qn('w:fldCharType'),'separate')
    f3=OxmlElement('w:t'); f3.text="Mets a jour le champ (clic droit > Mettre a jour) pour generer la table."
    f4=OxmlElement('w:fldChar'); f4.set(qn('w:fldCharType'),'end')
    for e in (f1,instr,f2,f3,f4): r._r.append(e)
```

Astuce : pour livrer une TOC deja remplie, convertir le .docx en .docx via LibreOffice (bash) met aussi a jour les champs.

## Lire / modifier un .docx existant

```python
doc = Document('/mnt/data/in.docx')
for p in doc.paragraphs:
    if 'ANCIEN' in p.text:
        for run in p.runs:
            run.text = run.text.replace('ANCIEN', 'NOUVEAU')   # remplace en gardant la mise en forme
for table in doc.tables:
    for row in table.rows:
        for cell in row.cells:
            ... # lire/editer le contenu des cellules
doc.save('/mnt/data/out.docx')
```

Pour extraire tout le texte (analyse) : parcourir `doc.paragraphs` et `doc.tables`.

## Conversion PDF (apercu / livraison)

```bash
soffice --headless -env:UserInstallation=file:///tmp/lo_profile --convert-to pdf --outdir /mnt/data /mnt/data/out.docx
```

## Workflow

1. **(py)** Construire/editer le document avec python-docx (styles, titres, tableaux, en-tetes/pieds...), sauver `/mnt/data/out.docx`.
2. **(bash)** Si table des matieres a remplir ou PDF demande : convertir via LibreOffice.
3. Livrer `/mnt/data/out.docx` (et le PDF si demande).

## Rappels

- Vise un vrai document mis en page (styles de titre, table des matieres pour un rapport, en-tete/pied avec numero de page, tableaux propres), pas un bloc de texte.
- Police coherente, paragraphes separes, vraies listes, images dimensionnees. Adapte au contexte (lettre suisse : lieu+date, objet en gras, formule de politesse).
- Reponds dans la langue de l'utilisateur.
