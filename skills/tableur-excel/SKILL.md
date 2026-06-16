---
name: tableur-excel
description: "Cree, modifie et analyse de vrais fichiers tableur (.xlsx, .xlsm, .csv, .tsv) de niveau professionnel et VISUELLEMENT SOIGNE, en EXECUTANT DU CODE (interpreteur souverain) : pandas pour l'analyse, openpyxl pour les formules + une mise en page riche (bandeau de titre, sections, couleurs, format monetaire), puis recalcul des formules via LibreOffice et verification zero erreur. Utiliser cette skill des que le tableur est l'entree ou la sortie principale : ouvrir/lire/corriger un .xlsx existant, creer un classeur depuis zero ou d'autres donnees, batir un budget ou un modele financier, convertir entre formats tabulaires. Le livrable est un fichier tableur telechargeable. Ne pas utiliser quand le livrable est un document Word (skill document-word), une presentation (html-slides), ou une simple reponse texte."
when-to-use: "Le tableur (.xlsx/.csv...) est l'entree ou la sortie principale : creer, lire, modifier, analyser ou mettre en forme un fichier Excel professionnel et soigne."
always-apply: false
---

# Tableur Excel (niveau pro + mise en page soignee)

Tu travailles les tableurs en **executant du code** (Python + LibreOffice). Tu ne livres JAMAIS un tableau brut et plat : tu produis un fichier **mis en page comme un vrai document financier** (bandeau de titre, sections, couleurs, format monetaire), avec des **formules** qui se recalculent et **zero erreur**.

## Environnement (important)

- **Python** : langage **`py`**. pandas, numpy, openpyxl, xlsxwriter installes.
- **LibreOffice** (recalcul, conversion) : accessible **uniquement en Bash** (langage **`bash`**), pas en Python (isolation). Le recalcul se fait donc via une commande `soffice`.
- **Fichiers** : travaille dans **`/mnt/data`**. Ils persistent entre tes appels de code d'une meme conversation : enchaine (py construire+styler) -> (bash recalculer) -> (py verifier).

## Exigences de sortie

- **Mise en page professionnelle obligatoire** (voir recettes ci-dessous). Pas de tableau nu.
- **Zero erreur de formule** (`#REF!`, `#DIV/0!`, `#VALUE!`, `#N/A`, `#NAME?`).
- **Formules, jamais valeurs en dur** : `ws['E7']='=SUM(B7:D7)'`, pas le total calcule en Python. Place les hypotheses dans des cellules et reference-les.
- Modele existant : reproduis exactement son style. Ses conventions priment.

## Recettes de mise en page (openpyxl) — applique-les systematiquement

1. **Bandeau de titre** : fusionne la 1re ligne sur toute la largeur, fond fonce, texte blanc gras (taille 14-16). Une ligne de sous-titre/legende en dessous (fond accent ou clair, italique, petite taille).
2. **En-tetes de colonnes** : fond fonce, texte blanc gras, centre, bordure.
3. **En-tetes de SECTION** (ex. "Loyer et charges", "Salaires", "Marketing") : ligne avec fond clair, texte gras, sur toute la largeur. Regroupe les postes par section.
4. **Cellules d'INPUT (a personnaliser)** en **bleu** (Font color `0000FF`) ; les **formules en noir**. Ainsi l'utilisateur voit ce qu'il peut changer.
5. **Sous-totaux et TOTAL** : gras, fond accent (le TOTAL general plus marque).
6. **Format monetaire** : `number_format = '#,##0\\ "CHF"'` (ou la devise voulue) ; pourcentages `0.0%` ; negatifs entre parentheses.
7. **Largeurs de colonnes** (1re colonne large pour les libelles), **bordures fines** gris clair, **alignement** (nombres a droite), et **freeze_panes** sous les en-tetes.
8. Palette **sobre** : un fond fonce pour le bandeau/en-tetes, une couleur d'accent pour les totaux. Reste lisible et pro.

### Exemple complet a adapter (budget stylise)

```python
from openpyxl import Workbook
from openpyxl.styles import Font, PatternFill, Alignment, Border, Side

wb = Workbook(); ws = wb.active; ws.title = "Budget"
ACCENT="1F3A5F"; ACCENT2="C0703A"; SECTION="EAEFF5"; WHITE="FFFFFF"; BLUE="0000FF"
thin=Side(style="thin", color="D9D9D9"); border=Border(thin,thin,thin,thin)
money='#,##0\\ "CHF"'
hdr_fill=PatternFill("solid", fgColor=ACCENT); sec_fill=PatternFill("solid", fgColor=SECTION)
tot_fill=PatternFill("solid", fgColor=ACCENT2)

ws.column_dimensions["A"].width=34
for col in "BCDE": ws.column_dimensions[col].width=15

# Bandeau
ws.merge_cells("A2:E2"); t=ws["A2"]; t.value="Budget previsionnel 3 mois"
t.font=Font(bold=True,size=16,color=WHITE); t.fill=hdr_fill; t.alignment=Alignment(vertical="center",indent=1)
ws.row_dimensions[2].height=28
ws.merge_cells("A3:E3"); s=ws["A3"]
s.value="Montants en CHF  .  Cellules bleues = a personnaliser"; s.fill=tot_fill
s.font=Font(italic=True,color=WHITE,size=9); s.alignment=Alignment(vertical="center",indent=1)

# En-tetes (ligne 5)
for j,h in enumerate(["Poste","Mois 1","Mois 2","Mois 3","Total"],1):
    c=ws.cell(5,j,h); c.font=Font(bold=True,color=WHITE); c.fill=hdr_fill; c.border=border
    c.alignment=Alignment(horizontal="left" if j==1 else "center")

def section(r,label):
    ws.merge_cells(start_row=r,start_column=1,end_row=r,end_column=5)
    c=ws.cell(r,1,label); c.font=Font(bold=True); c.fill=sec_fill
def poste(r,label,vals,total=True):           # vals = inputs (bleus) ; total = formule (noir)
    ws.cell(r,1,label).border=border
    for j,v in enumerate(vals,2):
        c=ws.cell(r,j,v); c.font=Font(color=BLUE); c.number_format=money; c.border=border; c.alignment=Alignment(horizontal="right")
    if total:
        c=ws.cell(r,5,f"=SUM(B{r}:D{r})"); c.number_format=money; c.border=border; c.alignment=Alignment(horizontal="right")
def soustotal(r,label,rows):
    ws.cell(r,1,label).font=Font(bold=True)
    for j in range(2,6):
        col=chr(64+j); f="+".join(f"{col}{x}" for x in rows)
        c=ws.cell(r,j,f"={f}"); c.font=Font(bold=True); c.number_format=money; c.alignment=Alignment(horizontal="right")

# Exemple de contenu (a remplacer par la demande reelle)
section(6,"Loyer et charges"); poste(7,"Loyer bureau",[2800,2800,2800]); poste(8,"Charges",[350,350,350]); soustotal(9,"Sous-total loyer",[7,8])
section(10,"Salaires (charges incluses)"); poste(11,"Salaire 1",[9000,9000,9000]); poste(12,"Salaire 2",[4000,4000,4000]); soustotal(13,"Sous-total salaires",[11,12])
section(14,"Marketing"); poste(15,"Publicite Meta",[1500,1800,2000]); poste(16,"Publicite LinkedIn",[800,900,1000]); soustotal(17,"Sous-total marketing",[15,16])

# TOTAL general
for j in range(1,6):
    c=ws.cell(19,j); c.fill=tot_fill; c.font=Font(bold=True,color=WHITE)
ws.cell(19,1,"TOTAL MENSUEL")
for j in range(2,6):
    col=chr(64+j); ws.cell(19,j,f"={col}9+{col}13+{col}17").number_format=money; ws.cell(19,j).alignment=Alignment(horizontal="right")

ws.freeze_panes="A6"
wb.save("/mnt/data/out.xlsx")
```

## Workflow (avec recalcul)

1. **(py)** Lire/analyser si fichier fourni (`pd.read_excel('/mnt/data/in.xlsx', sheet_name=None)`).
2. **(py)** Construire **et styler** avec openpyxl (recettes ci-dessus), sauver `/mnt/data/out.xlsx`.
3. **(bash) RECALCUL OBLIGATOIRE** :
   ```bash
   rm -rf /mnt/data/_r && mkdir -p /mnt/data/_r
   soffice --headless -env:UserInstallation=file:///tmp/lo_profile --calc --convert-to xlsx --outdir /mnt/data/_r /mnt/data/out.xlsx
   mv -f /mnt/data/_r/out.xlsx /mnt/data/out.xlsx
   ```
4. **(py) Verifier zero erreur** : `load_workbook('/mnt/data/out.xlsx', data_only=True)`, scanner les cellules pour `#REF!/#DIV/0!/#VALUE!/#N/A/#NAME?`, corriger et recalculer jusqu'a 0.
5. Livrer `/mnt/data/out.xlsx`.

## Rappels

- Toujours regrouper par **sections** avec sous-totaux, inputs **bleus**, totaux **gras/colores**, format **CHF**, bandeau de titre. Vise le rendu d'un vrai modele financier, pas un tableau Excel par defaut.
- pandas pour l'analyse ; openpyxl pour formules/format (indices 1-based ; `data_only=True` lit les valeurs mais ne pas re-sauver dans ce mode). Code Python concis.
