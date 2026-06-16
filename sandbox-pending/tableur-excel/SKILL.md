---
name: tableur-excel
description: "Cree, modifie et analyse de vrais fichiers tableur (.xlsx, .xlsm, .csv, .tsv) de niveau professionnel, en executant du code (pandas pour l'analyse, openpyxl pour les formules et la mise en forme), puis en recalculant les formules et en verifiant qu'il ne reste aucune erreur. Utiliser cette skill des que le tableur est l'entree ou la sortie principale : ouvrir/lire/corriger un fichier existant (ajouter des colonnes, des formules, de la mise en forme, des graphiques, nettoyer des donnees en vrac), creer un classeur depuis zero ou depuis d'autres donnees, ou convertir entre formats tabulaires. Le livrable est un fichier tableur. Ne pas utiliser quand le livrable principal est un document Word, une presentation, ou un rapport HTML (autres skills). NECESSITE l'environnement d'execution de code (Python + LibreOffice)."
when-to-use: "Le tableur (.xlsx/.csv...) est l'entree ou la sortie principale : creer, lire, modifier, corriger, analyser ou mettre en forme un fichier Excel de qualite professionnelle."
always-apply: false
---

# Tableur Excel (qualite professionnelle)

Tu crees, modifies et analyses de vrais fichiers tableur en **executant du code** dans l'environnement d'execution (Python + LibreOffice). Tu lis le fichier, tu travailles avec pandas (analyse) et openpyxl (formules + mise en forme), tu **recalcules** les formules, et tu **verifies qu'il ne reste aucune erreur** avant de livrer.

## Exigences de sortie (tous les fichiers)

- **Police professionnelle** cohérente (ex. Arial) sauf instruction contraire.
- **Zero erreur de formule** : le fichier livre ne doit contenir aucune erreur (`#REF!`, `#DIV/0!`, `#VALUE!`, `#N/A`, `#NAME?`).
- **Preserver un modele existant** : quand tu modifies un fichier, etudie et reproduis exactement son format, son style et ses conventions. Ne lui impose jamais une mise en forme standardisee : les conventions du fichier priment.

## Regle d'or : des formules, jamais des valeurs en dur

Calcule **dans le tableur via des formules**, pas en Python avec un resultat fige. Le fichier doit rester recalculable quand les donnees changent.

- A eviter : `sheet['B10'] = df['Ventes'].sum()` (fige 5000).
- A faire : `sheet['B10'] = '=SOMME(B2:B9)'` (Excel recalcule).

Cela vaut pour TOUS les calculs (totaux, pourcentages, ratios, ecarts). Place les hypotheses (taux de croissance, marges...) dans des cellules dediees et reference-les (`=B5*(1+$B$6)` plutot que `=B5*1.05`).

## Modeles financiers : conventions

Sauf indication de l'utilisateur ou d'un modele existant :

**Code couleur (standard du secteur)**
- Texte **bleu** (0,0,255) : entrees saisies en dur, chiffres que l'utilisateur changera (scenarios).
- Texte **noir** (0,0,0) : toutes les formules et calculs.
- Texte **vert** (0,128,0) : liens vers une autre feuille du meme classeur.
- Texte **rouge** (255,0,0) : liens externes vers d'autres fichiers.
- Fond **jaune** (255,255,0) : hypotheses cles a surveiller / cellules a mettre a jour.

**Formats de nombres**
- Annees : en texte ("2024", pas "2 024").
- Montants : format `$#,##0` (ou CHF), avec l'unite dans l'en-tete ("Chiffre d'affaires (CHF mio)").
- Zeros affiches en "-" (ex. `#,##0;(#,##0);-`).
- Pourcentages : `0.0%` par defaut.
- Multiples de valorisation : `0.0x`.
- Negatifs entre parentheses `(123)`, pas `-123`.

**Documentation des valeurs en dur** : note la source a cote ou en commentaire de cellule, format "Source : [systeme/document], [date], [reference]".

## Lire et analyser les donnees

Apercu rapide du contenu (toutes feuilles) puis analyse pandas :

```python
import pandas as pd
df = pd.read_excel('fichier.xlsx')                       # 1re feuille
toutes = pd.read_excel('fichier.xlsx', sheet_name=None)  # toutes les feuilles (dict)
df.head(); df.info(); df.describe()
```

Conseils pandas : forcer les types si besoin (`dtype={'id': str}`), lire des colonnes precises pour les gros fichiers (`usecols=[...]`), parser les dates (`parse_dates=[...]`), gerer les NaN (`pd.notna()`).

## Creer / modifier (openpyxl, pour formules + mise en forme)

```python
from openpyxl import Workbook, load_workbook
from openpyxl.styles import Font, PatternFill, Alignment

wb = load_workbook('existant.xlsx')   # ou Workbook() pour un nouveau
ws = wb.active                        # ou wb['NomFeuille']
ws['B2'] = '=SOMME(B3:B20)'           # formule, pas valeur figee
ws['A1'].font = Font(bold=True, color='0000FF')
ws['A1'].fill = PatternFill('solid', start_color='FFFF00')
ws.column_dimensions['A'].width = 22
wb.save('sortie.xlsx')
```

Rappels openpyxl : indices 1-based ; `data_only=True` lit les valeurs calculees mais ATTENTION, sauver dans ce mode efface les formules ; `read_only=True` / `write_only=True` pour les gros fichiers ; les formules ne sont pas evaluees tant qu'on n'a pas recalcule.

## Workflow obligatoire

1. **Choisir l'outil** : pandas (analyse), openpyxl (formules/mise en forme).
2. **Charger / creer** le classeur.
3. **Modifier** (donnees, formules, mise en forme).
4. **Sauvegarder**.
5. **Recalculer les formules (OBLIGATOIRE si formules)** : lancer le recalcul via LibreOffice headless (`soffice --headless --convert-to ...` / la macro de recalcul fournie dans l'environnement), qui recalcule toutes les feuilles et **scanne toutes les cellules a la recherche d'erreurs**.
6. **Verifier et corriger** : si des erreurs sont trouvees (`#REF!`, `#DIV/0!`, `#VALUE!`, `#NAME?`), localise-les, corrige, et recalcule a nouveau jusqu'a **zero erreur**.

## Checklist de verification des formules

- Tester 2-3 references avant de generaliser ; verifier le mapping des colonnes et le decalage de lignes (DataFrame ligne 5 = Excel ligne 6).
- Verifier les denominateurs (eviter `#DIV/0!`), les references (eviter `#REF!`), le format des references inter-feuilles (`Feuille1!A1`).
- Tester les cas limites (zero, negatifs, grandes valeurs) ; pas de references circulaires non voulues.

## Style de code

Python concis (peu de commentaires, pas de print inutiles). Dans le tableur lui-meme, en revanche : commente les formules complexes et les hypotheses, documente les sources des valeurs en dur.
