---
name: tableur-excel
description: "Cree, modifie et analyse de vrais fichiers tableur (.xlsx, .xlsm, .csv, .tsv) de niveau professionnel et VISUELLEMENT SOIGNE, en EXECUTANT DU CODE (interpreteur souverain) : pandas pour l'analyse, openpyxl pour les formules + une mise en page riche (bandeau de titre, sections, couleurs, format monetaire), puis recalcul des formules via LibreOffice et verification zero erreur. Utiliser cette skill des que le tableur est l'entree ou la sortie principale : ouvrir/lire/corriger un .xlsx existant, creer un classeur depuis zero ou d'autres donnees, batir un budget ou un modele financier, convertir entre formats tabulaires. Le livrable est un fichier tableur telechargeable. Ne pas utiliser quand le livrable est un document Word (skill document-word), une presentation (html-slides), ou une simple reponse texte."
when-to-use: "Le tableur (.xlsx/.csv...) est l'entree ou la sortie principale : creer, lire, modifier, analyser ou mettre en forme un fichier Excel professionnel et soigne."
always-apply: false
---

# Tableur Excel (niveau pro, mise en page imposee)

Tu produis le tableur **en executant du code** (Python + LibreOffice). REGLE ABSOLUE : tu ne livres **jamais** un tableau plat. Tu construis TOUT classeur **via la boite a outils de style ci-dessous** (a copier telle quelle dans ton code), puis tu recalcules les formules et tu verifies zero erreur. Interdit : `df.to_excel(...)` brut, ou ecrire des cellules sans passer par ces fonctions.

## Environnement

- **Python** (`py`) : pandas, numpy, openpyxl installes.
- **LibreOffice** (recalcul/conversion) : **uniquement en Bash** (`bash`).
- **Fichiers** dans **`/mnt/data`** (persistants entre tes appels de code).

## Etape 1 (py) — Copie cette boite a outils TELLE QUELLE, puis construis avec

```python
from openpyxl import Workbook
from openpyxl.styles import Font, PatternFill, Alignment, Border, Side
from openpyxl.utils import get_column_letter

ACCENT="1F3A5F"; ACCENT2="C0703A"; SECTION="EAEFF5"; WHITE="FFFFFF"; BLUE="0000FF"
_t=Side(style="thin", color="D9D9D9"); BORD=Border(left=_t,right=_t,top=_t,bottom=_t)
MONEY='#,##0\\ "CHF"'   # adapter la devise si besoin

def new_sheet(title, ncols):
    wb=Workbook(); ws=wb.active; ws.title=title
    ws.column_dimensions["A"].width=34
    for j in range(2,ncols+1): ws.column_dimensions[get_column_letter(j)].width=15
    ws.sheet_view.showGridLines=False
    return wb, ws

def banner(ws, ncols, title, subtitle=""):
    last=get_column_letter(ncols)
    ws.merge_cells(f"A1:{last}1"); c=ws["A1"]; c.value=title
    c.font=Font(bold=True,size=16,color=WHITE); c.fill=PatternFill("solid",fgColor=ACCENT)
    c.alignment=Alignment(vertical="center",indent=1); ws.row_dimensions[1].height=30
    if subtitle:
        ws.merge_cells(f"A2:{last}2"); s=ws["A2"]; s.value=subtitle
        s.font=Font(italic=True,size=9,color=WHITE); s.fill=PatternFill("solid",fgColor=ACCENT2)
        s.alignment=Alignment(vertical="center",indent=1)

def headers(ws, row, labels):
    for j,h in enumerate(labels,1):
        c=ws.cell(row,j,h); c.font=Font(bold=True,color=WHITE); c.fill=PatternFill("solid",fgColor=ACCENT)
        c.border=BORD; c.alignment=Alignment(horizontal="left" if j==1 else "center")

def section(ws, row, ncols, label):
    ws.merge_cells(start_row=row,start_column=1,end_row=row,end_column=ncols)
    c=ws.cell(row,1,label); c.font=Font(bold=True); c.fill=PatternFill("solid",fgColor=SECTION)

def line(ws, row, label, inputs, with_total=True):   # inputs = valeurs saisies (BLEU) ; total = formule (NOIR)
    ws.cell(row,1,label).border=BORD
    for k,v in enumerate(inputs,2):
        c=ws.cell(row,k,v); c.font=Font(color=BLUE); c.number_format=MONEY; c.border=BORD
        c.alignment=Alignment(horizontal="right")
    if with_total:
        a=get_column_letter(2); b=get_column_letter(1+len(inputs))
        c=ws.cell(row,2+len(inputs),f"=SUM({a}{row}:{b}{row})"); c.number_format=MONEY
        c.border=BORD; c.alignment=Alignment(horizontal="right")

def subtotal(ws, row, label, rows, ncols):           # somme de plusieurs lignes (gras)
    ws.cell(row,1,label).font=Font(bold=True)
    for j in range(2,ncols+1):
        col=get_column_letter(j); f="+".join(f"{col}{x}" for x in rows)
        c=ws.cell(row,j,f"={f}"); c.font=Font(bold=True); c.number_format=MONEY
        c.alignment=Alignment(horizontal="right")

def total(ws, row, label, subtotal_rows, ncols):      # TOTAL general (bandeau colore)
    for j in range(1,ncols+1):
        c=ws.cell(row,j); c.fill=PatternFill("solid",fgColor=ACCENT2); c.font=Font(bold=True,color=WHITE)
    ws.cell(row,1,label)
    for j in range(2,ncols+1):
        col=get_column_letter(j); f="+".join(f"{col}{x}" for x in subtotal_rows)
        c=ws.cell(row,j,f"={f}"); c.number_format=MONEY; c.alignment=Alignment(horizontal="right")
```

## Etape 2 (py) — Remplis avec les VRAIES donnees de l'utilisateur

Schema a suivre : `new_sheet` -> `banner` -> `headers` (ligne 4) -> alterner `section` / `line` / `subtotal` -> `total` -> `ws.freeze_panes` -> `save`. Exemple (a adapter au sujet reel) :

```python
wb, ws = new_sheet("Budget", ncols=5)
banner(ws, 5, "Budget previsionnel 3 mois", "Montants en CHF  .  cellules bleues = a personnaliser")
headers(ws, 4, ["Poste","Mois 1","Mois 2","Mois 3","Total"])
section(ws,5,5,"Loyer et charges"); line(ws,6,"Loyer bureau",[2800,2800,2800]); line(ws,7,"Charges",[350,350,350]); subtotal(ws,8,"Sous-total loyer",[6,7],5)
section(ws,9,5,"Salaires"); line(ws,10,"Salaire 1",[9000,9000,9000]); line(ws,11,"Salaire 2",[4000,4000,4000]); subtotal(ws,12,"Sous-total salaires",[10,11],5)
section(ws,13,5,"Marketing"); line(ws,14,"Publicite Meta",[1500,1800,2000]); line(ws,15,"Publicite LinkedIn",[800,900,1000]); subtotal(ws,16,"Sous-total marketing",[14,15],5)
total(ws,18,"TOTAL MENSUEL",[8,12,16],5)
ws.freeze_panes="A5"
wb.save("/mnt/data/out.xlsx")
```

Adapte le nombre de colonnes, les sections, les postes et les valeurs au besoin reel. Garde TOUJOURS : bandeau, en-tetes colores, sections, inputs bleus, sous-totaux, TOTAL colore, format CHF.

## Etape 3 (bash) — Recalcul OBLIGATOIRE des formules

```bash
rm -rf /mnt/data/_r && mkdir -p /mnt/data/_r
soffice --headless -env:UserInstallation=file:///tmp/lo_profile --calc --convert-to xlsx --outdir /mnt/data/_r /mnt/data/out.xlsx
mv -f /mnt/data/_r/out.xlsx /mnt/data/out.xlsx
```

## Etape 4 (py) — Verifier zero erreur

```python
from openpyxl import load_workbook
wb=load_workbook('/mnt/data/out.xlsx', data_only=True)
errs={'#REF!','#DIV/0!','#VALUE!','#N/A','#NAME?','#NUM!','#NULL!'}
bad=[(ws.title,c.coordinate,c.value) for ws in wb for row in ws.iter_rows() for c in row if isinstance(c.value,str) and c.value in errs]
print('ERREURS:', bad if bad else 'aucune')
```
Si des erreurs : corrige les formules et refais l'etape 3.

## Lire / modifier un fichier fourni

Charge avec `pd.read_excel('/mnt/data/in.xlsx', sheet_name=None)` pour analyser, puis reconstruis proprement avec la boite a outils (ou edite avec openpyxl en preservant le style existant si le fichier a deja une charte). Livrer `/mnt/data/out.xlsx`.

## Rappels

- Tu DOIS passer par la boite a outils : bandeau + en-tetes colores + sections + inputs bleus + sous-totaux + TOTAL colore + format CHF. Jamais un tableau plat.
- Formules (pas de valeurs figees), recalcul LibreOffice, verification zero erreur. Code Python concis.
