---
name: tableur-excel
description: "Cree, modifie et analyse de vrais fichiers tableur (.xlsx, .xlsm, .csv, .tsv) de niveau professionnel, en EXECUTANT DU CODE (interpreteur souverain) : pandas pour l'analyse, openpyxl pour les formules et la mise en forme, puis recalcul des formules via LibreOffice et verification qu'il ne reste aucune erreur. Utiliser cette skill des que le tableur est l'entree ou la sortie principale : ouvrir/lire/corriger un .xlsx existant (ajouter colonnes, formules, mise en forme, graphiques, nettoyer des donnees en vrac), creer un classeur depuis zero ou d'autres donnees, convertir entre formats tabulaires, batir un modele financier. Le livrable est un fichier tableur telechargeable. Ne pas utiliser quand le livrable principal est un document Word (skill document-word), une presentation (skill html-slides), ou une simple reponse texte."
when-to-use: "Le tableur (.xlsx/.csv...) est l'entree ou la sortie principale : creer, lire, modifier, corriger, analyser ou mettre en forme un fichier Excel de qualite professionnelle."
always-apply: false
---

# Tableur Excel (qualite professionnelle, via l'interpreteur de code)

Tu travailles les tableurs en **executant du code** dans l'interpreteur (Python + LibreOffice disponibles). Tu lis le fichier, tu analyses avec pandas, tu construis/edites avec openpyxl, tu **recalcules** les formules avec LibreOffice, et tu **verifies zero erreur** avant de livrer le fichier.

## Environnement d'execution (important)

- **Python** : lance le code avec le langage **`py`**. pandas, numpy, openpyxl, xlsxwriter sont installes.
- **LibreOffice** (recalcul, conversion xlsx->pdf) : il N'EST accessible **qu'en Bash** (langage **`bash`**), pas en Python (contrainte d'isolation du bac a sable). Donc le recalcul des formules se fait via une **commande bash** `soffice`.
- **Fichiers** : travaille dans **`/mnt/data`**. Les fichiers y persistent entre tes appels de code successifs d'une meme conversation, donc tu peux enchainer : (py) construire -> (bash) recalculer -> (py) verifier.

## Exigences de sortie

- **Police professionnelle** coherente (ex. Arial).
- **Zero erreur de formule** dans le fichier livre (`#REF!`, `#DIV/0!`, `#VALUE!`, `#N/A`, `#NAME?`).
- **Preserver un modele existant** : reproduis exactement le format, le style et les conventions du fichier que tu modifies. Ses conventions priment.

## Regle d'or : formules, jamais valeurs en dur

Calcule **dans le tableur via des formules** (`ws['B10']='=SOMME(B2:B9)'`), pas en Python avec un resultat fige. Le fichier doit rester recalculable. Place les hypotheses (taux, marges) dans des cellules dediees et reference-les (`=B5*(1+$B$6)`).

## Modeles financiers : conventions

Code couleur : **bleu** (0,0,255) entrees en dur ; **noir** formules ; **vert** (0,128,0) liens inter-feuilles ; **rouge** liens externes ; **fond jaune** hypotheses a surveiller.
Formats : montants `#,##0` avec unite dans l'en-tete ; zeros en "-" ; pourcentages `0.0%` ; negatifs entre parentheses `(123)` ; multiples `0.0x`. Documente la source des valeurs en dur (commentaire de cellule).

## Workflow obligatoire (avec recalcul)

1. **(py)** Lire / analyser : `pd.read_excel('/mnt/data/in.xlsx', sheet_name=None)` ; `df.describe()`, etc.
2. **(py)** Construire / editer avec openpyxl (formules en chaines, mise en forme) et sauvegarder dans `/mnt/data/out.xlsx`.
3. **(bash) RECALCUL OBLIGATOIRE si formules** : faire recalculer et figer les valeurs par LibreOffice, puis remettre le fichier en place :
   ```bash
   rm -rf /mnt/data/_recalc && mkdir -p /mnt/data/_recalc
   soffice --headless -env:UserInstallation=file:///tmp/lo_profile \
     --calc --convert-to xlsx --outdir /mnt/data/_recalc /mnt/data/out.xlsx
   mv -f /mnt/data/_recalc/out.xlsx /mnt/data/out.xlsx
   ```
4. **(py) Verifier zero erreur** : rouvrir en valeurs calculees et scanner les erreurs ; corriger et recalculer jusqu'a 0.
   ```python
   from openpyxl import load_workbook
   wb = load_workbook('/mnt/data/out.xlsx', data_only=True)
   errs = ['#REF!','#DIV/0!','#VALUE!','#N/A','#NAME?','#NUM!','#NULL!']
   bad = [(ws.title,c.coordinate,c.value) for ws in wb for row in ws.iter_rows() for c in row
          if isinstance(c.value,str) and c.value in errs]
   print('ERREURS:', bad if bad else 'aucune')
   ```
5. Livrer `/mnt/data/out.xlsx` (telechargeable). Pour un apercu, tu peux aussi convertir en PDF via bash (`soffice ... --convert-to pdf`).

## Bonnes pratiques

- pandas pour l'analyse/volume ; openpyxl pour formules/format. Indices openpyxl 1-based. `data_only=True` lit les valeurs calculees (ne pas re-sauver dans ce mode, ca efface les formules).
- Verifier denominateurs (`#DIV/0!`), references (`#REF!`), format inter-feuilles (`Feuille1!A1`). Tester sur 2-3 cellules avant de generaliser.
- Code Python concis. Dans le tableur : commenter les formules complexes et documenter les sources.
