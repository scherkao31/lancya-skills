---
name: presentation-pptx
description: "Genere des presentations au format FICHIER PowerPoint (.pptx) de niveau professionnel avec python-pptx dans le sandbox d'execution de code : slides titre, section, puces, deux colonnes, image, tableau et graphique natif, avec une charte graphique imposee. Utiliser cette skill quand le livrable voulu est un FICHIER PowerPoint : des que l'utilisateur dit PowerPoint, .pptx, un fichier de presentation a telecharger, ou un deck modifiable dans PowerPoint / Keynote. IMPORTANT : si l'utilisateur precise plutot HTML, web, slides interactives, ou une presentation a afficher et modifier dans la conversation, NE PAS utiliser cette skill mais presentation-html. Ne pas utiliser non plus pour un fichier Excel ou tableur (tableur-excel), ni un document Word ou rapport texte (document-word)."
when-to-use: "Quand le livrable est un FICHIER PowerPoint .pptx : l'utilisateur dit 'PowerPoint' ou '.pptx', ou veut un fichier de presentation a telecharger et modifier dans PowerPoint/Keynote. Pas pour du HTML/web (presentation-html), de l'Excel ou du Word."
always-apply: false
---

# Presentations PowerPoint (.pptx) avec python-pptx

Cette skill sert a produire des presentations PowerPoint soignees, directement exploitables par un public professionnel, depuis le sandbox d'execution de code de Lancya. Le moteur est la librairie Python `python-pptx`, qui ecrit de vrais fichiers `.pptx` (format Office Open XML) ouvrables dans PowerPoint, Keynote, Google Slides ou LibreOffice Impress.

La regle absolue : **ne jamais produire une slide nue ou mal alignee.** Tout passe par la boite a outils de style fournie plus bas. Le modele construit le deck en appelant ces fonctions, jamais en posant des zones de texte au hasard.

## Environnement du sandbox

A respecter strictement, sinon le code echoue.

- **Python**, bloc d'execution `id='py'`. Librairies disponibles : `python-pptx`, `pandas`, `matplotlib`, `pillow`. Pas besoin de `pip install`.
- **LibreOffice (`soffice`)** est disponible **uniquement en Bash**, bloc `id='bash'`. Il ne s'appelle jamais depuis Python. Toute conversion (par exemple `.pptx` vers `.pdf`) se fait par une commande `soffice` dans un bloc bash.
- **Dossier de travail : `/mnt/data`.** Les fichiers y persistent entre les executions. On y ecrit le `.pptx` final et tous les fichiers intermediaires (images, graphiques matplotlib exportes, etc.).
- Le fichier de sortie standard est `/mnt/data/out.pptx`. Quand l'utilisateur fournit un nom, l'employer ; sinon garder `out.pptx`.

Workflow type :

1. **(py)** construire le deck avec la boite a outils, puis `prs.save("/mnt/data/out.pptx")`.
2. **(bash, optionnel)** convertir en PDF avec `soffice` si l'utilisateur veut un PDF ou un apercu.

## Notions de base de python-pptx

### Presentation, slides, layouts

```python
from pptx import Presentation

prs = Presentation()                 # deck vide au format 4:3 par defaut
slide_layout = prs.slide_layouts[6]  # layout "vide" (blank) du theme par defaut
slide = prs.slides.add_slide(slide_layout)
```

Un `Presentation()` sans argument part d'un theme par defaut qui expose une serie de layouts indexes :

- `0` : diapositive de titre (Title Slide)
- `1` : titre + contenu (Title and Content)
- `5` : titre seul (Title Only)
- `6` : vide (Blank)

Dans cette skill on travaille presque toujours sur le layout **vide (`6`)** et on pose soi-meme tous les elements via la boite a outils. C'est ce qui garantit un controle total de la mise en page et un rendu coherent, plutot que de dependre des placeholders du theme.

### Le format 16:9

Le 16:9 est le standard attendu aujourd'hui. On force les dimensions juste apres la creation du deck :

```python
from pptx.util import Inches
prs.slide_width = Inches(13.333)
prs.slide_height = Inches(7.5)
```

La boite a outils ci-dessous le fait automatiquement dans `deck()`.

### Unites de mesure

python-pptx ne travaille pas en pixels mais en EMU (English Metric Units). On n'ecrit jamais d'EMU a la main : on utilise les helpers de `pptx.util`.

- `Inches(1)` = 1 pouce = 914400 EMU.
- `Pt(18)` = 18 points (pour les tailles de police et certaines positions).
- `Emu(...)` existe mais on l'evite.

Reperes utiles sur une slide 16:9 de 13.333 x 7.5 pouces :

- largeur totale 13.333 po, hauteur totale 7.5 po ;
- marge laterale confortable : 0.9 po a gauche et a droite, donc une zone de contenu de 13.333 - 1.8 = 11.533 po de large ;
- une position se definit par quatre valeurs : `left, top, width, height`.

### Placeholders vs zones de texte

Deux facons de poser du texte :

- **Placeholder** : zone predefinie par le layout (titre, sous-titre, corps). Pratique mais le rendu depend du theme, et l'alignement est moins previsible.
- **Zone de texte (`add_textbox`)** : un cadre qu'on positionne soi-meme au pixel pres. C'est l'approche retenue ici car elle donne un rendu identique partout et un controle total.

```python
from pptx.util import Inches, Pt
tb = slide.shapes.add_textbox(Inches(0.9), Inches(0.6), Inches(11.5), Inches(1.0))
tf = tb.text_frame
tf.word_wrap = True
p = tf.paragraphs[0]
p.text = "Mon titre"
p.font.size = Pt(32)
p.font.bold = True
```

### Anatomie d'un text_frame

- Un `text_frame` contient un ou plusieurs **paragraphes** (`tf.paragraphs`). Le premier existe toujours.
- On ajoute un paragraphe avec `tf.add_paragraph()`.
- Chaque paragraphe contient des **runs** (`p.runs`), un run etant un fragment de texte avec une mise en forme homogene. Affecter `p.text = "..."` cree un run unique.
- Mise en forme du paragraphe : `p.alignment`, `p.level` (niveau de puce, 0 a 4), `p.space_before`, `p.space_after`, `p.line_spacing`.
- Mise en forme de la police : `run.font.size`, `run.font.bold`, `run.font.italic`, `run.font.name`, `run.font.color.rgb`.

```python
from pptx.enum.text import PP_ALIGN
from pptx.util import Pt
p.alignment = PP_ALIGN.LEFT
p.space_after = Pt(6)
p.line_spacing = 1.1
```

### Couleurs

Les couleurs s'expriment en `RGBColor` :

```python
from pptx.dml.color import RGBColor
run.font.color.rgb = RGBColor(0x1A, 0x1A, 0x2E)   # depuis hex
run.font.color.rgb = RGBColor.from_string("1A1A2E")  # depuis une chaine hex
```

Pour remplir une forme :

```python
shape.fill.solid()
shape.fill.fore_color.rgb = RGBColor.from_string("0F6FFF")
shape.line.fill.background()   # pas de bordure
```

## Boite a outils de style OBLIGATOIRE

Le modele **doit** construire chaque deck avec ces fonctions. Elles imposent une palette, une typographie et des positions coherentes. Elles s'executent dans le bloc `id='py'`. Coller ce bloc en tete du script, puis composer le deck en appelant `deck()`, `slide_titre()`, `slide_section()`, `slide_puces()`, `slide_tableau()`, `slide_image()`, et au besoin `slide_deux_colonnes()` et `slide_graphique()`.

La palette ci-dessous est sobre et professionnelle, adaptee a un public d'entreprise. On peut l'adapter aux couleurs d'une marque en changeant uniquement le dictionnaire `PALETTE`, sans toucher au reste.

```python
# ============================================================
#  BOITE A OUTILS LANCYA - presentations .pptx (python-pptx)
#  A coller en tete de tout script de generation de deck.
# ============================================================
from pptx import Presentation
from pptx.util import Inches, Pt, Emu
from pptx.dml.color import RGBColor
from pptx.enum.text import PP_ALIGN, MSO_ANCHOR
from pptx.enum.shapes import MSO_SHAPE
from pptx.chart.data import CategoryChartData
from pptx.enum.chart import XL_CHART_TYPE, XL_LEGEND_POSITION, XL_LABEL_POSITION

# ---- Charte graphique -------------------------------------
PALETTE = {
    "fond":      "FFFFFF",  # fond des slides de contenu
    "fond_sombre": "12203A", # fond des slides titre / section
    "encre":     "1A2233",  # texte principal sur fond clair
    "encre_claire": "F3F6FB",# texte sur fond sombre
    "primaire":  "0F6FFF",  # bleu d'accent
    "secondaire":"0FB5A6",  # vert/teal secondaire
    "gris":      "5B6577",  # texte secondaire, sous-titres
    "gris_clair":"E6EAF1",  # filets, fonds de cellules d'en-tete
    "rouge":     "E0574B",  # alerte / point negatif
}

POLICE_TITRE = "Calibri"      # remplacable par la police de marque
POLICE_TEXTE = "Calibri"

# ---- Geometrie standard (slide 16:9, 13.333 x 7.5 po) -----
SLIDE_W = 13.333
SLIDE_H = 7.5
MARGE   = 0.9                 # marge gauche/droite
ZONE_W  = SLIDE_W - 2 * MARGE # largeur utile du contenu
HAUT_TITRE = 0.55             # position verticale du titre
HAUT_CONTENU = 1.7            # debut de la zone de contenu

def _rgb(nom):
    return RGBColor.from_string(PALETTE[nom])

def _fond(slide, couleur):
    """Remplit toute la slide d'une couleur de fond."""
    rect = slide.shapes.add_shape(
        MSO_SHAPE.RECTANGLE, 0, 0, Inches(SLIDE_W), Inches(SLIDE_H)
    )
    rect.fill.solid()
    rect.fill.fore_color.rgb = _rgb(couleur)
    rect.line.fill.background()
    rect.shadow.inherit = False
    # envoyer le rectangle au fond (premier de l'ordre de dessin)
    sp = rect._element
    sp.getparent().remove(sp)
    slide.shapes._spTree.insert(2, sp)
    return rect

def _filet(slide, top, couleur="primaire", left=MARGE, width=2.2, h=0.07):
    """Petit filet horizontal d'accent (sous un titre)."""
    bar = slide.shapes.add_shape(
        MSO_SHAPE.RECTANGLE, Inches(left), Inches(top),
        Inches(width), Inches(h)
    )
    bar.fill.solid()
    bar.fill.fore_color.rgb = _rgb(couleur)
    bar.line.fill.background()
    bar.shadow.inherit = False
    return bar

def _texte(slide, left, top, width, height, texte, taille=18,
           gras=False, italique=False, couleur="encre",
           align=PP_ALIGN.LEFT, police=POLICE_TEXTE, interligne=1.1,
           anchor=MSO_ANCHOR.TOP):
    """Pose une zone de texte simple, mono-paragraphe."""
    box = slide.shapes.add_textbox(
        Inches(left), Inches(top), Inches(width), Inches(height)
    )
    tf = box.text_frame
    tf.word_wrap = True
    tf.vertical_anchor = anchor
    tf.margin_left = 0
    tf.margin_right = 0
    tf.margin_top = 0
    tf.margin_bottom = 0
    p = tf.paragraphs[0]
    p.alignment = align
    p.line_spacing = interligne
    run = p.add_run()
    run.text = texte
    run.font.size = Pt(taille)
    run.font.bold = gras
    run.font.italic = italique
    run.font.name = police
    run.font.color.rgb = _rgb(couleur)
    return box

def _bandeau_titre(slide, titre, sur_titre=None):
    """En-tete standard d'une slide de contenu : titre + filet d'accent."""
    if sur_titre:
        _texte(slide, MARGE, HAUT_TITRE - 0.05, ZONE_W, 0.4,
               sur_titre.upper(), taille=12, gras=True,
               couleur="primaire", police=POLICE_TITRE)
        ty = HAUT_TITRE + 0.32
    else:
        ty = HAUT_TITRE
    _texte(slide, MARGE, ty, ZONE_W, 0.9, titre, taille=30, gras=True,
           couleur="encre", police=POLICE_TITRE)
    _filet(slide, ty + 0.78)

# ---- 1. Creation du deck ----------------------------------
def deck():
    """Cree un deck 16:9 vierge et renvoie l'objet Presentation."""
    prs = Presentation()
    prs.slide_width = Inches(SLIDE_W)
    prs.slide_height = Inches(SLIDE_H)
    return prs

def _slide_vide(prs):
    return prs.slides.add_slide(prs.slide_layouts[6])

# ---- 2. Slide de titre ------------------------------------
def slide_titre(prs, titre, sous_titre=None, pied=None):
    slide = _slide_vide(prs)
    _fond(slide, "fond_sombre")
    _filet(slide, 3.05, couleur="primaire", left=MARGE, width=2.4, h=0.09)
    _texte(slide, MARGE, 3.3, SLIDE_W - 2 * MARGE, 1.6, titre,
           taille=44, gras=True, couleur="encre_claire",
           police=POLICE_TITRE, interligne=1.05)
    if sous_titre:
        _texte(slide, MARGE, 4.9, SLIDE_W - 2 * MARGE, 1.0, sous_titre,
               taille=20, couleur="gris_clair", police=POLICE_TEXTE)
    if pied:
        _texte(slide, MARGE, SLIDE_H - 0.7, SLIDE_W - 2 * MARGE, 0.4,
               pied, taille=12, couleur="gris", police=POLICE_TEXTE)
    return slide

# ---- 3. Slide de section / intercalaire -------------------
def slide_section(prs, titre, numero=None):
    slide = _slide_vide(prs)
    _fond(slide, "primaire")
    if numero:
        _texte(slide, MARGE, 2.5, ZONE_W, 1.0, str(numero),
               taille=22, gras=True, couleur="encre_claire",
               police=POLICE_TITRE)
    _texte(slide, MARGE, 3.1, ZONE_W, 1.8, titre, taille=40, gras=True,
           couleur="encre_claire", police=POLICE_TITRE,
           anchor=MSO_ANCHOR.MIDDLE, interligne=1.05)
    return slide

# ---- 4. Slide a puces -------------------------------------
def slide_puces(prs, titre, puces, sur_titre=None):
    """puces : liste de chaines, ou de tuples (texte, niveau)."""
    slide = _slide_vide(prs)
    _fond(slide, "fond")
    _bandeau_titre(slide, titre, sur_titre)
    box = slide.shapes.add_textbox(
        Inches(MARGE), Inches(HAUT_CONTENU), Inches(ZONE_W),
        Inches(SLIDE_H - HAUT_CONTENU - 0.6)
    )
    tf = box.text_frame
    tf.word_wrap = True
    for i, item in enumerate(puces):
        if isinstance(item, tuple):
            texte, niveau = item
        else:
            texte, niveau = item, 0
        p = tf.paragraphs[0] if i == 0 else tf.add_paragraph()
        p.level = niveau
        p.space_after = Pt(10)
        p.line_spacing = 1.12
        puce = "•   " if niveau == 0 else "–   "
        run = p.add_run()
        run.text = puce + texte
        run.font.size = Pt(20 if niveau == 0 else 17)
        run.font.name = POLICE_TEXTE
        run.font.color.rgb = _rgb("encre" if niveau == 0 else "gris")
        run.font.bold = (niveau == 0)
    return slide

# ---- 5. Slide deux colonnes -------------------------------
def slide_deux_colonnes(prs, titre, titre_g, puces_g, titre_d, puces_d,
                        sur_titre=None):
    slide = _slide_vide(prs)
    _fond(slide, "fond")
    _bandeau_titre(slide, titre, sur_titre)
    col_w = (ZONE_W - 0.6) / 2
    for (cx, t_col, p_col, coul) in [
        (MARGE, titre_g, puces_g, "primaire"),
        (MARGE + col_w + 0.6, titre_d, puces_d, "secondaire"),
    ]:
        _texte(slide, cx, HAUT_CONTENU, col_w, 0.5, t_col, taille=18,
               gras=True, couleur=coul, police=POLICE_TITRE)
        box = slide.shapes.add_textbox(
            Inches(cx), Inches(HAUT_CONTENU + 0.6), Inches(col_w),
            Inches(SLIDE_H - HAUT_CONTENU - 1.2)
        )
        tf = box.text_frame
        tf.word_wrap = True
        for i, item in enumerate(p_col):
            p = tf.paragraphs[0] if i == 0 else tf.add_paragraph()
            p.space_after = Pt(8)
            p.line_spacing = 1.1
            run = p.add_run()
            run.text = "•   " + item
            run.font.size = Pt(16)
            run.font.name = POLICE_TEXTE
            run.font.color.rgb = _rgb("encre")
    return slide

# ---- 6. Slide tableau -------------------------------------
def slide_tableau(prs, titre, entetes, lignes, sur_titre=None,
                  largeurs=None):
    """entetes : liste de chaines. lignes : liste de listes (cellules)."""
    slide = _slide_vide(prs)
    _fond(slide, "fond")
    _bandeau_titre(slide, titre, sur_titre)
    n_lig = len(lignes) + 1
    n_col = len(entetes)
    gf = slide.shapes.add_table(
        n_lig, n_col, Inches(MARGE), Inches(HAUT_CONTENU),
        Inches(ZONE_W), Inches(0.5 * n_lig)
    )
    table = gf.table
    table.first_row = True
    table.horz_banding = True
    if largeurs:
        total = sum(largeurs)
        for j, w in enumerate(largeurs):
            table.columns[j].width = Inches(ZONE_W * w / total)
    # en-tete
    for j, txt in enumerate(entetes):
        cell = table.cell(0, j)
        cell.fill.solid()
        cell.fill.fore_color.rgb = _rgb("primaire")
        cell.vertical_anchor = MSO_ANCHOR.MIDDLE
        p = cell.text_frame.paragraphs[0]
        p.alignment = PP_ALIGN.LEFT
        r = p.add_run(); r.text = str(txt)
        r.font.bold = True; r.font.size = Pt(14)
        r.font.color.rgb = _rgb("encre_claire"); r.font.name = POLICE_TITRE
    # corps
    for i, ligne in enumerate(lignes, start=1):
        for j, val in enumerate(ligne):
            cell = table.cell(i, j)
            cell.fill.solid()
            cell.fill.fore_color.rgb = _rgb(
                "fond" if i % 2 else "gris_clair")
            cell.vertical_anchor = MSO_ANCHOR.MIDDLE
            p = cell.text_frame.paragraphs[0]
            r = p.add_run(); r.text = str(val)
            r.font.size = Pt(13)
            r.font.color.rgb = _rgb("encre"); r.font.name = POLICE_TEXTE
    return slide, table

# ---- 7. Slide image ---------------------------------------
def slide_image(prs, titre, chemin_image, legende=None, sur_titre=None):
    """Image cadree dans la zone de contenu, ratio preserve."""
    slide = _slide_vide(prs)
    _fond(slide, "fond")
    _bandeau_titre(slide, titre, sur_titre)
    from PIL import Image
    img = Image.open(chemin_image)
    iw, ih = img.size
    ratio = iw / ih
    dispo_w = ZONE_W
    dispo_h = SLIDE_H - HAUT_CONTENU - (0.8 if legende else 0.4)
    if dispo_w / dispo_h > ratio:
        h = dispo_h; w = h * ratio
    else:
        w = dispo_w; h = w / ratio
    left = MARGE + (dispo_w - w) / 2
    top = HAUT_CONTENU + (dispo_h - h) / 2
    slide.shapes.add_picture(chemin_image, Inches(left), Inches(top),
                             Inches(w), Inches(h))
    if legende:
        _texte(slide, MARGE, SLIDE_H - 0.7, ZONE_W, 0.4, legende,
               taille=12, italique=True, couleur="gris",
               align=PP_ALIGN.CENTER)
    return slide

# ---- 8. Slide graphique natif -----------------------------
def slide_graphique(prs, titre, categories, series, type_chart="colonnes",
                    sur_titre=None, legende=True):
    """series : liste de tuples (nom, [valeurs...]).
    type_chart : 'colonnes', 'barres', 'lignes' ou 'secteurs'."""
    slide = _slide_vide(prs)
    _fond(slide, "fond")
    _bandeau_titre(slide, titre, sur_titre)
    data = CategoryChartData()
    data.categories = categories
    for nom, valeurs in series:
        data.add_series(nom, valeurs)
    types = {
        "colonnes": XL_CHART_TYPE.COLUMN_CLUSTERED,
        "barres":   XL_CHART_TYPE.BAR_CLUSTERED,
        "lignes":   XL_CHART_TYPE.LINE_MARKERS,
        "secteurs": XL_CHART_TYPE.PIE,
    }
    gf = slide.shapes.add_chart(
        types.get(type_chart, XL_CHART_TYPE.COLUMN_CLUSTERED),
        Inches(MARGE), Inches(HAUT_CONTENU), Inches(ZONE_W),
        Inches(SLIDE_H - HAUT_CONTENU - 0.5), data
    )
    chart = gf.chart
    chart.has_title = False
    if legende and type_chart != "secteurs":
        chart.has_legend = True
        chart.legend.position = XL_LEGEND_POSITION.BOTTOM
        chart.legend.include_in_layout = False
    elif type_chart == "secteurs":
        chart.has_legend = True
        chart.legend.position = XL_LEGEND_POSITION.RIGHT
        chart.legend.include_in_layout = False
        chart.plots[0].has_data_labels = True
        chart.plots[0].data_labels.show_percentage = True
        chart.plots[0].data_labels.position = XL_LABEL_POSITION.OUTSIDE_END
    else:
        chart.has_legend = False
    return slide, chart

# ---- 9. Notes d'orateur -----------------------------------
def notes(slide, texte):
    slide.notes_slide.notes_text_frame.text = texte
```

## Comment composer un deck

Toujours dans le bloc `id='py'`, apres la boite a outils. Exemple complet et executable.

```python
prs = deck()

# 1) Couverture
s = slide_titre(
    prs,
    "Strategie commerciale 2026",
    sous_titre="Plan d'action et objectifs trimestriels",
    pied="Lancya  ·  Document interne  ·  Juin 2026",
)
notes(s, "Introduire le contexte : marche en croissance, objectif de doubler le pipeline.")

# 2) Intercalaire de section
slide_section(prs, "Ou en sommes-nous ?", numero="01")

# 3) Slide a puces (avec sous-niveaux)
slide_puces(
    prs,
    "Constat de depart",
    [
        "Le chiffre d'affaires progresse de 12 % sur un an",
        ("La marge se concentre sur trois comptes cles", 1),
        ("Le risque de dependance est eleve", 1),
        "Le cycle de vente reste trop long sur les PME",
    ],
    sur_titre="Diagnostic",
)

# 4) Deux colonnes (forces / faiblesses)
slide_deux_colonnes(
    prs,
    "Forces et points de vigilance",
    "Forces", ["Notoriete locale solide", "Equipe experimentee", "Produit mature"],
    "Vigilance", ["Acquisition lente", "Outillage CRM partiel", "Peu de recurrent"],
)

# 5) Tableau
slide_tableau(
    prs,
    "Objectifs par trimestre",
    ["Trimestre", "Nouveaux clients", "Revenu (CHF)", "Statut"],
    [
        ["T1", "8", "120 000", "Atteint"],
        ["T2", "12", "180 000", "En cours"],
        ["T3", "15", "240 000", "Prevu"],
        ["T4", "18", "300 000", "Prevu"],
    ],
    largeurs=[1.2, 1.4, 1.4, 1.0],
)

# 6) Graphique natif
slide_graphique(
    prs,
    "Evolution du pipeline",
    ["T1", "T2", "T3", "T4"],
    [("2025", [90, 110, 130, 150]), ("2026", [120, 180, 240, 300])],
    type_chart="colonnes",
    sur_titre="Projection",
)

prs.save("/mnt/data/out.pptx")
print("Deck enregistre : /mnt/data/out.pptx")
```

## Graphiques : aller plus loin

`slide_graphique` couvre les cas courants. Pour personnaliser, on recupere l'objet `chart` renvoye et on agit dessus.

```python
slide, chart = slide_graphique(prs, "Repartition du CA", ["Produit A", "Produit B", "Produit C"],
                               [("Part", [45, 35, 20])], type_chart="secteurs")
# Acceder a une serie et a son trace
plot = chart.plots[0]
plot.has_data_labels = True
plot.data_labels.number_format = '0"%"'
plot.data_labels.number_format_is_linked = False
```

Points cles de l'API graphique :

- Les donnees passent par un `CategoryChartData` : `data.categories = [...]` puis `data.add_series(nom, valeurs)` pour chaque serie.
- `slide.shapes.add_chart(type, left, top, width, height, data)` renvoie un `GraphicFrame`. Le graphique reel est `gf.chart` (un graphique n'est pas une forme, il est contenu dans le cadre).
- Types via `XL_CHART_TYPE` : `COLUMN_CLUSTERED`, `BAR_CLUSTERED`, `LINE`, `LINE_MARKERS`, `PIE`, `DOUGHNUT`, `XY_SCATTER`, `AREA`, etc.
- Legende : `chart.has_legend = True`, `chart.legend.position = XL_LEGEND_POSITION.BOTTOM`, `chart.legend.include_in_layout = False` (evite que la legende empiete sur le trace).
- Etiquettes : `chart.plots[0].has_data_labels = True`, puis `data_labels.show_value`, `show_percentage`, `position`, `number_format`.

**Astuce matplotlib :** pour un visuel qui n'existe pas en natif (carte de chaleur, courbe complexe), generer un PNG avec matplotlib, l'enregistrer dans `/mnt/data`, puis l'inserer avec `slide_image`. Les graphiques natifs `add_chart` restent prefererables quand ils suffisent, car ils sont editables dans PowerPoint.

```python
import matplotlib
matplotlib.use("Agg")
import matplotlib.pyplot as plt
fig, ax = plt.subplots(figsize=(10, 5.2), dpi=160)
ax.plot([1, 2, 3, 4], [10, 25, 18, 32])
fig.tight_layout()
fig.savefig("/mnt/data/courbe.png", transparent=True)
plt.close(fig)
slide_image(prs, "Tendance annuelle", "/mnt/data/courbe.png", legende="Source : donnees internes")
```

## Images

- `slide.shapes.add_picture(chemin, left, top, width=None, height=None)`. Si on ne donne ni largeur ni hauteur, l'image garde sa taille native (en EMU), ce qui deborde souvent. `slide_image` calcule donc le cadrage en preservant le ratio via Pillow.
- Pour ne fixer qu'une dimension, donner `width=Inches(...)` seule : la hauteur s'ajuste proportionnellement.
- Formats acceptes : PNG, JPEG, GIF. Pour de la transparence (logo, graphique matplotlib), preferer le PNG.

## Notes d'orateur

Chaque slide expose un `notes_slide`. La fonction `notes(slide, texte)` y ecrit le commentaire de l'orateur, invisible a la projection mais present en mode presentateur et a l'impression des documents.

```python
notes(slide, "Insister sur le delai. Repondre a l'objection budgetaire avec le ROI a 6 mois.")
```

## Conversion en PDF avec LibreOffice (bash)

La conversion se fait **uniquement en Bash**, jamais en Python. Apres avoir enregistre le `.pptx`, lancer dans un bloc `id='bash'` :

```bash
cd /mnt/data && soffice --headless --convert-to pdf --outdir /mnt/data out.pptx
```

Cela produit `/mnt/data/out.pdf`. Le PDF est utile pour un apercu rapide, un envoi par mail ou une verification visuelle du rendu.

Remarques :

- Le rendu LibreOffice est tres proche de PowerPoint mais pas identique au pixel pres (substitution de police possible si la police n'est pas installee). Calibri et Arial passent bien.
- Pour exporter aussi en images (apercu de chaque slide), on convertit d'abord en PDF puis on rasterise, ou on passe par `--convert-to png` qui n'exporte que la premiere slide. Pour toutes les slides en images, convertir en PDF puis utiliser un outil de rasterisation si disponible.
- Si `soffice` n'est pas dans le PATH, essayer `libreoffice` a la place du binaire `soffice`.

## Regles de qualite a appliquer systematiquement

- **Une idee par slide.** Si une slide a puces depasse 6 a 7 lignes, la scinder en deux.
- **Texte court.** Des puces de quelques mots, pas des phrases entieres. Le detail va dans les notes d'orateur.
- **Hierarchie claire.** Couverture, puis sections numerotees, puis contenu. Une slide de section avant chaque grande partie.
- **Toujours passer par la boite a outils.** Jamais de zone de texte posee a la main hors des fonctions fournies : c'est ce qui garantit l'alignement, les marges et la palette.
- **Coherence des couleurs.** Le bleu primaire pour les accents, le teal pour le secondaire, le rouge reserve aux alertes. Ne pas multiplier les couleurs.
- **Chiffres lisibles.** Dans les tableaux et graphiques, formater les nombres (separateurs de milliers, unite explicite comme CHF ou %).
- **Verifier le livrable.** Apres `prs.save`, confirmer la creation du fichier (taille non nulle) et, si demande, generer le PDF pour controle visuel.

## Resume du workflow

1. Bloc `id='py'` : coller la boite a outils, composer le deck (`deck()` puis les `slide_*`), `prs.save("/mnt/data/out.pptx")`.
2. Bloc `id='bash'` (optionnel) : `cd /mnt/data && soffice --headless --convert-to pdf --outdir /mnt/data out.pptx`.
3. Livrer `/mnt/data/out.pptx` (et `/mnt/data/out.pdf` le cas echeant).

## References

- Documentation officielle python-pptx : https://python-pptx.readthedocs.io/en/latest/
- Guide des graphiques python-pptx : https://python-pptx.readthedocs.io/en/latest/user/charts.html
