---
name: pdf-creation
description: "Cree de vrais PDF mis en page de niveau professionnel en EXECUTANT DU CODE (interpreteur de code) avec reportlab (Platypus) : rapport, facture, attestation, courrier, fiche, brochure simple. Couvre format A4 et marges, bandeau de titre, sections, tableaux colores, images, polices et couleurs, en-tetes et pieds de page avec numerotation. Utiliser cette skill des que l'utilisateur veut CREER ou GENERER un PDF mis en page : rapport, facture, devis, attestation, certificat, fiche, courrier, brochure simple. Ne pas utiliser pour extraire du texte/des tableaux d'un PDF (skill pdf-extraction), pour remplir un formulaire PDF (skill pdf-formulaires), pour fusionner/decouper/pivoter des PDF (skill pdf-fusion-decoupe), ni pour produire un Word (skill document-word), un Excel (skill tableur-excel) ou une presentation/slides (skill html-slides)."
when-to-use: "L'utilisateur veut creer ou generer un PDF MIS EN PAGE : rapport, facture, devis, attestation, certificat, fiche, courrier, brochure. Pas pour lire/extraire, remplir un formulaire, fusionner/decouper, ni pour un Word/Excel/slides."
always-apply: false
---

# PDF mis en page de niveau pro, via reportlab (Platypus)

Tu produis de **vrais PDF mis en page** en **executant du code** dans l'interpreteur (reportlab est installe). Tu ne livres jamais un texte brut ni un PDF "machine a ecrire" : un document **structure et stylise** (bandeau de titre, sections, tableaux colores, en-tetes/pieds de page numerotes), propre et professionnel.

## Environnement

- **Python** (langage **`py`**, `id='py'`) : `reportlab`, `pillow` (PIL), `pandas` installes.
- **Fichiers** dans **`/mnt/data`** (persistants entre tes appels de code d'une meme conversation). Ecris toujours la sortie la, ex. `/mnt/data/facture.pdf`.
- **Alternative .docx -> PDF** : pour un courrier tres "bureautique", tu peux produire un `.docx` (skill document-word) puis le convertir en **Bash** avec LibreOffice : `soffice --headless --convert-to pdf --outdir /mnt/data /mnt/data/fichier.docx`. Mais pour une **vraie mise en page maitrisee** (facture, attestation, rapport stylise), **reportlab Platypus est la voie a privilegier** : tu controles marges, couleurs, tableaux et pieds de page au point pres.

## Deux niveaux d'API reportlab

- **`canvas`** (bas niveau) : tu dessines toi-meme texte, lignes, rectangles a des coordonnees absolues. Utile pour le bandeau et le pied de page (voir la boite a outils).
- **Platypus** (haut niveau) : tu empiles des *flowables* (`Paragraph`, `Spacer`, `Table`, `Image`...) dans un `SimpleDocTemplate` qui gere la mise en page, le retour a la ligne et les sauts de page automatiquement. **C'est ton outil principal pour le corps du document.**

Les deux se combinent : Platypus pour le contenu, le `canvas` (via `onPage`) pour ce qui se repete sur chaque page.

## Regles de rendu (a respecter)

- **Format A4**, marges genereuses (2 cm), unites en `cm` ou `mm` (`from reportlab.lib.units import cm, mm`).
- **Couleurs centralisees** : une palette en haut du script, jamais de couleur "en dur" dispersee.
- **Jamais de `\n` dans un `Paragraph`** pour sauter une ligne : un paragraphe = un `Paragraph`, et on espace avec `Spacer`. Pour un retour a la ligne *dans* un paragraphe, utiliser la balise `<br/>`.
- **Tableaux toujours stylises** (`TableStyle`) : entete colore, texte clair sur entete, filets discrets, lignes alternees (zebrage) si plus de quelques lignes.
- **Encodage** : avec les polices Unicode (voir plus bas) ecris les accents normalement. Les `Paragraph` acceptent du mini-HTML : `<b>`, `<i>`, `<font color=...>`, `<br/>`.
- **Toujours un pied de page numerote** sur les documents multi-pages (rapport), et un bandeau de titre sur les documents "officiels" (facture, attestation, rapport).

## BOITE A OUTILS DE STYLE (OBLIGATOIRE)

Tout document que tu generes **doit reutiliser cette boite a outils**. Elle centralise la palette, les styles de paragraphe, le bandeau de titre, les blocs de section, les tableaux colores et le pied de page numerote. Copie-la en tete de ton script, adapte la palette et le pied si besoin, puis construis le contenu avec ces helpers. **Ne reinvente pas le style a la main : c'est ce kit qui garantit un rendu pro et coherent.**

```python
# === BOITE A OUTILS PDF LANCYA (reportlab Platypus) ===========================
from reportlab.lib.pagesizes import A4
from reportlab.lib.units import cm, mm
from reportlab.lib import colors
from reportlab.lib.enums import TA_LEFT, TA_CENTER, TA_RIGHT, TA_JUSTIFY
from reportlab.lib.styles import getSampleStyleSheet, ParagraphStyle
from reportlab.platypus import (
    SimpleDocTemplate, Paragraph, Spacer, Table, TableStyle, Image,
    HRFlowable, KeepTogether, PageBreak,
)
from reportlab.pdfbase import pdfmetrics
from reportlab.pdfbase.ttfonts import TTFont

# --- Palette (a personnaliser une seule fois) --------------------------------
PRIMARY   = colors.HexColor("#1F3A5F")   # bleu fonce (bandeau, titres)
ACCENT    = colors.HexColor("#2E86AB")   # bleu accent (filets, liens)
LIGHT     = colors.HexColor("#EEF3F8")   # fond clair (zebrage, encadres)
DARK      = colors.HexColor("#222222")   # texte courant
MUTED     = colors.HexColor("#6B7280")   # gris (legendes, pied de page)
LINE      = colors.HexColor("#D5DCE5")   # filets de tableau

# --- Polices Unicode (accents fiables). Fallback Helvetica si TTF absentes ----
FONT, FONT_B = "Helvetica", "Helvetica-Bold"
for name, bold, candidates in [
    ("Body",  False, ["/usr/share/fonts/truetype/dejavu/DejaVuSans.ttf"]),
    ("BodyB", True,  ["/usr/share/fonts/truetype/dejavu/DejaVuSans-Bold.ttf"]),
]:
    for path in candidates:
        try:
            pdfmetrics.registerFont(TTFont(name, path))
            if not bold:
                FONT = "Body"
            else:
                FONT_B = "BodyB"
            break
        except Exception:
            pass

# --- Styles de paragraphe ----------------------------------------------------
def make_styles():
    s = getSampleStyleSheet()
    s.add(ParagraphStyle("Doc",      parent=s["Normal"], fontName=FONT,  fontSize=10,
                         leading=14, textColor=DARK))
    s.add(ParagraphStyle("DocJust",  parent=s["Doc"],    alignment=TA_JUSTIFY))
    s.add(ParagraphStyle("H1",       parent=s["Normal"], fontName=FONT_B, fontSize=16,
                         leading=20, textColor=PRIMARY, spaceBefore=14, spaceAfter=6))
    s.add(ParagraphStyle("H2",       parent=s["Normal"], fontName=FONT_B, fontSize=12,
                         leading=16, textColor=PRIMARY, spaceBefore=10, spaceAfter=4))
    s.add(ParagraphStyle("Small",    parent=s["Doc"],    fontSize=8,  textColor=MUTED))
    s.add(ParagraphStyle("Right",    parent=s["Doc"],    alignment=TA_RIGHT))
    s.add(ParagraphStyle("CellH",    parent=s["Normal"], fontName=FONT_B, fontSize=9,
                         leading=12, textColor=colors.white))
    s.add(ParagraphStyle("Cell",     parent=s["Normal"], fontName=FONT,  fontSize=9,
                         leading=12, textColor=DARK))
    return s

STYLES = make_styles()

# --- Helper : bandeau de titre (a poser comme flowable en tete du contenu) ----
def title_band(title, subtitle=None, width=None):
    """Bandeau colore plein largeur avec titre (et sous-titre)."""
    para = Paragraph(
        '<font color="white"><b>%s</b></font>' % title +
        ('<br/><font color="#DCE6F2" size="9">%s</font>' % subtitle if subtitle else ''),
        ParagraphStyle("band", fontName=FONT_B, fontSize=18, leading=22,
                       textColor=colors.white, alignment=TA_LEFT),
    )
    t = Table([[para]], colWidths=[width] if width else None)
    t.setStyle(TableStyle([
        ("BACKGROUND", (0, 0), (-1, -1), PRIMARY),
        ("LEFTPADDING", (0, 0), (-1, -1), 14),
        ("RIGHTPADDING", (0, 0), (-1, -1), 14),
        ("TOPPADDING", (0, 0), (-1, -1), 12),
        ("BOTTOMPADDING", (0, 0), (-1, -1), 12),
    ]))
    return t

# --- Helper : filet horizontal accentue --------------------------------------
def rule(color=ACCENT, thickness=1.2, space_after=8):
    return HRFlowable(width="100%", thickness=thickness, color=color,
                      spaceBefore=2, spaceAfter=space_after)

# --- Helper : tableau colore (entete + zebrage + filets) ---------------------
def styled_table(data, col_widths=None, header=True, align_right_cols=()):
    """data : liste de lignes ; chaque cellule = str ou Paragraph.
    Convertit les str en Paragraph pour un rendu/retour a la ligne propre."""
    body = []
    for r, row in enumerate(data):
        cells = []
        for cell in row:
            if isinstance(cell, str):
                style = STYLES["CellH"] if (header and r == 0) else STYLES["Cell"]
                cells.append(Paragraph(cell, style))
            else:
                cells.append(cell)
        body.append(cells)

    t = Table(body, colWidths=col_widths, repeatRows=1 if header else 0)
    ts = [
        ("GRID",        (0, 0), (-1, -1), 0.5, LINE),
        ("VALIGN",      (0, 0), (-1, -1), "MIDDLE"),
        ("LEFTPADDING", (0, 0), (-1, -1), 6),
        ("RIGHTPADDING",(0, 0), (-1, -1), 6),
        ("TOPPADDING",  (0, 0), (-1, -1), 5),
        ("BOTTOMPADDING",(0, 0),(-1, -1), 5),
    ]
    if header:
        ts += [
            ("BACKGROUND",  (0, 0), (-1, 0), PRIMARY),
            ("LINEBELOW",   (0, 0), (-1, 0), 1, PRIMARY),
            ("ROWBACKGROUNDS", (0, 1), (-1, -1), [colors.white, LIGHT]),  # zebrage
        ]
    else:
        ts += [("ROWBACKGROUNDS", (0, 0), (-1, -1), [colors.white, LIGHT])]
    for c in align_right_cols:
        ts.append(("ALIGN", (c, 0), (c, -1), "RIGHT"))
    t.setStyle(TableStyle(ts))
    return t

# --- En-tete + pied de page numerote (dessines via onPage, sur CHAQUE page) ---
def header_footer(canvas, doc, left_text="", right_text="", footer_text=""):
    canvas.saveState()
    w, h = A4
    # En-tete : fin filet + libelle gauche/droite
    canvas.setFont(FONT, 8)
    canvas.setFillColor(MUTED)
    if left_text:
        canvas.drawString(doc.leftMargin, h - 1.2 * cm, left_text)
    if right_text:
        canvas.drawRightString(w - doc.rightMargin, h - 1.2 * cm, right_text)
    canvas.setStrokeColor(LINE)
    canvas.setLineWidth(0.5)
    canvas.line(doc.leftMargin, h - 1.35 * cm, w - doc.rightMargin, h - 1.35 * cm)
    # Pied : filet + texte a gauche + "Page X / Y" a droite
    canvas.line(doc.leftMargin, 1.35 * cm, w - doc.rightMargin, 1.35 * cm)
    canvas.setFillColor(MUTED)
    if footer_text:
        canvas.drawString(doc.leftMargin, 1.0 * cm, footer_text)
    page = "Page %d / %d" % (canvas.getPageNumber(), getattr(doc, "total_pages", 0))
    canvas.drawRightString(w - doc.rightMargin, 1.0 * cm, page)
    canvas.restoreState()

# --- Doc template a 2 passes (pour connaitre le nombre TOTAL de pages) --------
class NumberedDocTemplate(SimpleDocTemplate):
    """1re passe : on compte les pages. 2e passe : on dessine 'Page X / Y'.
    'total_pages' est fixe AVANT la 2e passe, donc 'Y' est correct des la page 1."""
    def __init__(self, *a, on_page=None, **k):
        super().__init__(*a, **k)
        self.total_pages = 0
        self._counting = True            # vrai pendant la passe de comptage
        self._on_page = on_page or (lambda c, d: None)

    def _draw(self, canvas, doc):
        if not self._counting:           # on ne dessine qu'a la 2e passe
            self._on_page(canvas, doc)

    def afterPage(self):
        if self._counting:
            self.total_pages = self.page  # derniere valeur = nombre total

    def build_numbered(self, flowables):
        from copy import deepcopy
        # Passe 1 : comptage (sortie jetable, flowables clones car consommes)
        self._counting = True
        super().build(deepcopy(flowables),
                      onFirstPage=self._draw, onLaterPages=self._draw)
        # Passe 2 : rendu reel avec le total connu
        self._counting = False
        super().build(flowables, onFirstPage=self._draw, onLaterPages=self._draw)

def build_pdf(path, flowables, left="", right="", footer=""):
    """Construit le PDF avec en-tete/pied numerote 'Page X / Y' sur chaque page."""
    doc = NumberedDocTemplate(
        path, pagesize=A4,
        leftMargin=2 * cm, rightMargin=2 * cm, topMargin=2 * cm, bottomMargin=2 * cm,
        title="Document Lancya",
        on_page=lambda c, d: header_footer(c, d, left, right, footer),
    )
    doc.build_numbered(flowables)
    return path
# === FIN DE LA BOITE A OUTILS =================================================
```

> Note sur la numerotation "Page X / Y" : reportlab ne connait le **total** de pages qu'apres avoir tout dispose. `NumberedDocTemplate` fait donc **deux passes** : une 1re passe muette qui compte les pages (`afterPage`), puis une 2e passe qui dessine reellement, avec `total_pages` deja connu (le "Y" est correct des la page 1). Les flowables etant consommes a la 1re passe, on les **clone** (`deepcopy`) pour la passe de comptage. Si tu n'as **pas besoin du total**, plus simple : `SimpleDocTemplate(...).build(flowables, onFirstPage=..., onLaterPages=...)` et n'afficher que `Page X` via `canvas.getPageNumber()`.

## Workflow

1. **Comprendre la commande** : quel type de document (rapport / facture / attestation / courrier / fiche), quel contenu, quelle palette/logo eventuels.
2. **Coller la boite a outils** en tete du script (`py`), ajuster `PRIMARY`/`ACCENT` a la marque si demande.
3. **Construire la liste `story`** de flowables : `title_band(...)`, `Spacer`, `Paragraph(..., STYLES["H1"])`, `styled_table(...)`, `Image(...)`, etc.
4. **Generer** avec `build_pdf("/mnt/data/<nom>.pdf", story, left=..., right=..., footer=...)`.
5. **Verifier** : reimprimer le chemin et la taille du fichier ; en cas de doute, ouvrir/recompter les pages. Annoncer le fichier produit a l'utilisateur.

## Exemple complet : une FACTURE stylisee

```python
# (la boite a outils ci-dessus doit deja avoir ete executee dans la session)

EMETTEUR = ["Lancya SA", "Rue de l'Exemple 12", "1200 Geneve", "TVA CHE-123.456.789"]
CLIENT   = ["Client Demo Sarl", "Avenue du Test 7", "1003 Lausanne"]
NUM, DATE, ECHEANCE = "F-2026-0042", "17.06.2026", "17.07.2026"

lignes = [
    # description, qte, prix unitaire HT, total HT
    ("Conseil et integration IA (hebergee en Suisse)", 12, 180.00),
    ("Hebergement Infomaniak (mensuel)",      1, 90.00),
    ("Formation equipe (demi-journee)",       2, 600.00),
]
TVA = 0.081  # 8.1 %

# --- En-tete : emetteur a gauche, bloc facture a droite ---
sous_total = sum(q * pu for _, q, pu in lignes)
tva_montant = round(sous_total * TVA, 2)
total = sous_total + tva_montant
fmt = lambda x: "CHF " + format(x, ",.2f").replace(",", "'")

story = []
story.append(title_band("FACTURE", "N° %s — %s" % (NUM, DATE)))
story.append(Spacer(1, 10))

infos = styled_table(
    [[Paragraph("<b>Emetteur</b><br/>" + "<br/>".join(EMETTEUR), STYLES["Cell"]),
      Paragraph("<b>Facture a</b><br/>" + "<br/>".join(CLIENT), STYLES["Cell"])]],
    col_widths=[8.5 * cm, 8.5 * cm], header=False,
)
story.append(infos)
story.append(Spacer(1, 6))
story.append(Paragraph("Echeance de paiement : <b>%s</b>" % ECHEANCE, STYLES["Doc"]))
story.append(Spacer(1, 12))

# --- Tableau des prestations ---
data = [["Description", "Qte", "Prix unit. HT", "Total HT"]]
for desc, q, pu in lignes:
    data.append([desc, str(q), fmt(pu), fmt(q * pu)])
story.append(styled_table(
    data, col_widths=[9 * cm, 1.8 * cm, 3.2 * cm, 3 * cm],
    align_right_cols=(1, 2, 3),
))
story.append(Spacer(1, 8))

# --- Totaux (tableau aligne a droite) ---
totaux = styled_table(
    [["Sous-total HT", fmt(sous_total)],
     ["TVA 8.1 %",     fmt(tva_montant)],
     ["Total TTC",     fmt(total)]],
    col_widths=[4 * cm, 3 * cm], header=False, align_right_cols=(1,),
)
totaux.setStyle(TableStyle([
    ("FONTNAME", (0, -1), (-1, -1), FONT_B),
    ("BACKGROUND", (0, -1), (-1, -1), LIGHT),
    ("TEXTCOLOR", (0, -1), (-1, -1), PRIMARY),
    ("LINEABOVE", (0, -1), (-1, -1), 1, PRIMARY),
]))
wrap = Table([[totaux]], colWidths=[17 * cm])
wrap.setStyle(TableStyle([("ALIGN", (0, 0), (-1, -1), "RIGHT")]))
story.append(wrap)

story.append(Spacer(1, 16))
story.append(rule(color=LINE, thickness=0.5))
story.append(Paragraph(
    "Paiement par virement a reception. Coordonnees bancaires : IBAN CH00 0000 0000 "
    "0000 0000 0. Merci de rappeler le numero de facture %s." % NUM, STYLES["Small"]))

out = build_pdf(
    "/mnt/data/facture_%s.pdf" % NUM, story,
    left="Lancya SA", right="Facture %s" % NUM,
    footer="Lancya SA — Geneve — contact@exemple.ch",
)
print("PDF genere :", out)
```

## Variantes rapides selon le type de document

- **Rapport multi-pages** : `title_band` + `Paragraph(..., STYLES["H1"]/["H2"])` + paragraphes `STYLES["DocJust"]` + `styled_table` + `PageBreak()` entre grandes parties. Le pied numerote `Page X / Y` se met tout seul via `build_pdf`. Pour garder un titre avec son tableau, enrober dans `KeepTogether([...])`.
- **Attestation / certificat** : bandeau, un grand `Paragraph` centre (`alignment=TA_CENTER`, gros `leading`), date et lieu, puis bloc signature (un `Table` sans bordure avec une ligne `_______` et le nom dessous).
- **Courrier** : pas forcement de bandeau ; en-tete expediteur, bloc destinataire aligne a droite, objet en gras, corps justifie, formule de politesse, signature. (Pour un courrier purement bureautique, l'alternative .docx -> PDF reste valable.)
- **Fiche / brochure simple** : `title_band`, sections courtes en `H2`, `styled_table` pour les caracteristiques, eventuellement une image.

## Images et logo

```python
from reportlab.platypus import Image
logo = Image("/mnt/data/logo.png", width=4 * cm, height=1.5 * cm)  # respecte le ratio toi-meme
logo.hAlign = "LEFT"
story.insert(0, logo)
```
Pour un logo **dans le bandeau d'en-tete repete**, dessine-le dans `header_footer` avec `canvas.drawImage("/mnt/data/logo.png", x, y, width=..., height=..., mask='auto')`.

## Pieges courants

- **Accents casses** : utiliser une police TTF Unicode (DejaVu, gere par la boite a outils) ; en dernier recours Helvetica gere le latin-1 mais pas tous les caracteres.
- **`Table` qui deborde** : toujours fournir `col_widths` dont la somme tient dans la largeur utile (A4 moins marges ≈ 17 cm) ; mettre les textes longs en `Paragraph` (retour a la ligne auto), pas en `str` nu.
- **Pied de page manquant sur la 1re page** : passer la meme fonction a `onFirstPage` et `onLaterPages` (c'est ce que fait `build_pdf`).
- **`Page X / Y` incomplet** : il faut `multiBuild` (2 passes), pas `build` simple.

## References (doc publique reportlab)

- Guide Platypus : https://docs.reportlab.com/reportlab/userguide/ch5_platypus/
- Manuel utilisateur (PDF) et reference API : https://docs.reportlab.com/
