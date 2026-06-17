---
name: analyse-donnees
description: "Analyse de donnees et visualisations a partir d'un CSV, Excel ou JSON, en EXECUTANT DU CODE (interpreteur de code) : pandas + numpy pour charger, nettoyer, explorer et calculer des statistiques, matplotlib + seaborn pour produire des graphiques soignes sauves en PNG. Utiliser cette skill des que l'utilisateur veut ANALYSER des donnees, calculer des statistiques, faire un tableau croise, reperer des tendances, comparer des groupes, mesurer des correlations, ou produire des GRAPHIQUES (histogramme, barres, courbe, nuage de points, boite a moustaches, heatmap) a partir d'un fichier CSV / Excel / JSON. Ne pas utiliser quand le livrable est un fichier tableur mis en forme et telechargeable (skill tableur-excel), ni quand une simple reponse texte sans calcul ni graphique suffit."
when-to-use: "L'utilisateur fournit (ou evoque) un CSV / Excel / JSON et veut une analyse chiffree, des statistiques, un tableau croise, des tendances, des correlations ou des graphiques. Le livrable est une synthese chiffree accompagnee de figures PNG, pas un classeur mis en forme."
always-apply: false
---

# Analyse de donnees et visualisations (pandas + matplotlib)

Tu analyses des donnees **en executant du code Python** dans l'interpreteur de code, puis tu restitues une **synthese chiffree accompagnee de graphiques**. Regle absolue : toute affirmation chiffree vient d'un calcul reel sur les donnees (jamais d'estimation), et **chaque figure est sauvee en PNG dans `/mnt/data`** avec un titre, des axes legendes et une mise en forme soignee.

Documentation de reference : pandas (https://pandas.pydata.org/docs/) et matplotlib (https://matplotlib.org/stable/).

## Environnement

- **Python** (`id='py'`) : `pandas`, `numpy`, `matplotlib`, `seaborn`, `scikit-learn`, `statsmodels` deja installes.
- **Fichiers** dans **`/mnt/data`** (persistants entre tes appels de code). Le fichier fourni par l'utilisateur s'y trouve ; tu y ecris aussi tes PNG.
- Les **PNG** que tu sauves dans `/mnt/data` sont ensuite affiches et telechargeables par l'application. C'est ton seul canal d'image : ne te repose pas sur un affichage interactif.

## Methode

Procede dans cet ordre, en commentant tes resultats au fur et a mesure : **charger -> nettoyer -> explorer -> calculer -> visualiser -> conclure**. Adapte chaque etape aux donnees reelles (ne plaque pas un code generique sans regarder ce que contient le fichier).

## Etape 0 (py) — Style des figures (copie cette base, puis reutilise-la)

Place ceci en tete pour que TOUTES tes figures partagent une charte propre.

```python
import pandas as pd, numpy as np
import matplotlib
matplotlib.use("Agg")                 # rendu fichier, pas d'affichage interactif
import matplotlib.pyplot as plt
import matplotlib.ticker as mtick
import seaborn as sns

OUT = "/mnt/data"
PALETTE = ["#1F3A5F", "#C0703A", "#4E8A77", "#9C6B9E", "#C7A33B", "#7A7A7A"]
sns.set_theme(style="whitegrid", palette=PALETTE)
plt.rcParams.update({
    "figure.dpi": 130, "savefig.dpi": 130, "figure.autolayout": True,
    "axes.titlesize": 14, "axes.titleweight": "bold",
    "axes.labelsize": 11, "axes.edgecolor": "#888888",
    "font.size": 10, "legend.frameon": False,
})

def finir(fig, ax, titre, x="", y="", fichier="figure.png", note=""):
    """Pose titre/axes, formate les milliers, sauve en PNG dans /mnt/data."""
    ax.set_title(titre, pad=12)
    if x: ax.set_xlabel(x)
    if y: ax.set_ylabel(y)
    try:
        ax.yaxis.set_major_formatter(mtick.FuncFormatter(lambda v, _: f"{v:,.0f}".replace(",", " ")))
    except Exception:
        pass
    if note:
        fig.text(0.01, 0.01, note, fontsize=8, color="#888888")
    chemin = f"{OUT}/{fichier}"
    fig.savefig(chemin, bbox_inches="tight", facecolor="white")
    plt.close(fig)
    return chemin
```

## Etape 1 (py) — Charger

Detecte le format et les pieges (separateur, encodage, decimales, dates). Reste robuste.

```python
# CSV : autodetection separateur + encodage de secours
def lire_csv(chemin):
    for enc in ("utf-8", "latin-1", "cp1252"):
        try:
            return pd.read_csv(chemin, sep=None, engine="python", encoding=enc)
        except (UnicodeDecodeError, pd.errors.ParserError):
            continue
    return pd.read_csv(chemin, sep=";", encoding="latin-1")  # cas FR frequent

# df = lire_csv("/mnt/data/donnees.csv")
# df = pd.read_excel("/mnt/data/donnees.xlsx", sheet_name=0)   # sheet_name=None -> toutes les feuilles
# df = pd.read_json("/mnt/data/donnees.json")                  # ou pd.json_normalize(...) si JSON imbrique

print(df.shape)
print(df.dtypes)
df.head()
```

Conseils de chargement :
- **Decimales FR** (`1 234,50`) : `decimal=","` et `thousands=" "` dans `read_csv`.
- **Dates** : convertis explicitement, `pd.to_datetime(df["date"], errors="coerce", dayfirst=True)`. Garde une vraie colonne datetime pour les series temporelles.
- **Gros fichier** : limite la memoire avec `usecols=[...]`, `dtype={...}`, `parse_dates=[...]` ; au besoin lis par morceaux `pd.read_csv(..., chunksize=100_000)` et agrege.

## Etape 2 (py) — Nettoyer

```python
df.columns = df.columns.str.strip()                 # libelles propres
av = len(df); df = df.drop_duplicates(); print("Doublons retires:", av - len(df))
print(df.isna().sum())                               # valeurs manquantes par colonne

# Types : numerique tolerant, categories pour economiser la memoire
# df["montant"] = pd.to_numeric(df["montant"], errors="coerce")
# df["categorie"] = df["categorie"].astype("category")

# Manquants : selon le sens metier (NE remplis pas a l'aveugle)
# df["montant"] = df["montant"].fillna(df["montant"].median())   # numerique
# df["categorie"] = df["categorie"].fillna("Inconnu")            # categoriel
```

Signale toujours ce que tu as decide (lignes supprimees, manquants combles ou laisses, types corriges) : c'est une partie de l'analyse.

## Etape 3 (py) — Explorer

```python
df.describe(include="all").T                          # vue d'ensemble
df["categorie"].value_counts(normalize=True).round(3) # repartition d'une categorie

# Agregation par groupe
df.groupby("categorie")["montant"].agg(["count", "sum", "mean", "median"]).round(2)

# Tableau croise (somme du montant par categorie x mois)
pd.pivot_table(df, index="categorie", columns="mois", values="montant",
               aggfunc="sum", fill_value=0, margins=True)
```

## Etape 4 (py) — Statistiques

```python
num = df.select_dtypes("number")
corr = num.corr(numeric_only=True).round(2)           # correlations
print(corr)

# Tendance temporelle (serie mensuelle)
serie = (df.set_index("date").sort_index()["montant"]
           .resample("M").sum())
croissance = serie.pct_change().mul(100).round(1)      # variation % periode/periode
```

Interprete : une correlation forte n'est pas une causalite, et une tendance se lit sur une serie ordonnee dans le temps.

## Etape 5 (py) — Visualiser (1 figure PNG soignee par graphique)

Choisis le graphique selon la question. Chaque bloc se termine par `finir(...)`.

```python
# Histogramme — distribution d'une variable numerique
fig, ax = plt.subplots(figsize=(8, 5))
sns.histplot(df["montant"].dropna(), bins=30, kde=True, color=PALETTE[0], ax=ax)
finir(fig, ax, "Distribution des montants", "Montant (CHF)", "Effectif", "hist_montant.png")

# Barres — comparaison entre categories (triees)
g = df.groupby("categorie")["montant"].sum().sort_values(ascending=False)
fig, ax = plt.subplots(figsize=(8, 5))
sns.barplot(x=g.values, y=g.index, hue=g.index, palette=PALETTE, legend=False, ax=ax)
finir(fig, ax, "Montant total par categorie", "Montant total (CHF)", "", "barres_categorie.png")

# Ligne — serie temporelle
fig, ax = plt.subplots(figsize=(9, 5))
serie.plot(ax=ax, color=PALETTE[0], marker="o", linewidth=2)
finir(fig, ax, "Evolution mensuelle du montant", "Mois", "Montant (CHF)", "serie_temporelle.png")

# Nuage de points — relation entre deux variables numeriques
fig, ax = plt.subplots(figsize=(7, 6))
sns.scatterplot(data=df, x="quantite", y="montant", hue="categorie",
                palette=PALETTE, alpha=0.7, ax=ax)
ax.legend(title="Categorie", bbox_to_anchor=(1.02, 1), loc="upper left")
finir(fig, ax, "Montant en fonction de la quantite", "Quantite", "Montant (CHF)", "nuage_qte_montant.png")

# Boite a moustaches — dispersion par groupe (reperer les valeurs extremes)
fig, ax = plt.subplots(figsize=(8, 5))
sns.boxplot(data=df, x="categorie", y="montant", hue="categorie",
            palette=PALETTE, legend=False, ax=ax)
ax.tick_params(axis="x", rotation=30)
finir(fig, ax, "Dispersion des montants par categorie", "Categorie", "Montant (CHF)", "boite_categorie.png")

# Heatmap — matrice de correlation
fig, ax = plt.subplots(figsize=(7, 6))
sns.heatmap(corr, annot=True, fmt=".2f", cmap="RdBu_r", center=0,
            vmin=-1, vmax=1, square=True, linewidths=0.5, cbar_kws={"shrink": 0.8}, ax=ax)
ax.set_title("Matrice de correlation", pad=12, fontweight="bold")
fig.savefig(f"{OUT}/heatmap_correlation.png", bbox_inches="tight", facecolor="white")
plt.close(fig)
```

Regles de style imposees pour CHAQUE figure :
- **Sauvee en PNG dans `/mnt/data`** (sinon l'utilisateur ne voit rien).
- **Titre** clair, **labels d'axes** avec l'unite, **legende** des qu'il y a plusieurs series.
- Nombres lisibles (milliers separes par une espace), barres **triees**, libelles longs pivotes, palette `PALETTE` coherente d'un graphique a l'autre.
- Une figure = une idee. Plusieurs PNG distincts plutot qu'une planche surchargee.

## Etape 6 — Restituer

Termine par une **synthese en francais** structuree :
1. Ce que contient le jeu de donnees (lignes, colonnes, periode couverte) et ce que tu as nettoye.
2. Les **chiffres cles** (totaux, moyennes/medianes, repartitions, tendances, correlations notables), en valeurs reelles issues des calculs.
3. Les **graphiques** produits (renvoie les PNG), chacun accompagne d'une phrase d'interpretation.
4. Limites et pistes d'analyse complementaire si pertinent.

## Rappels

- Tout chiffre vient d'un calcul reel sur les donnees, jamais d'une estimation.
- Chaque graphique est sauve en PNG dans `/mnt/data`, titre et axes soignes, palette coherente.
- Dis ce que tu fais des manquants, doublons et types : c'est de l'analyse, pas un detail technique.
- Si le livrable attendu est un classeur tableur mis en forme, bascule sur la skill `tableur-excel`.
