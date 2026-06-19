---
name: presentation-html
description: "Cree des presentations web elegantes et autonomes en HTML (un seul fichier, sans aucune librairie externe ni CDN), affichees et navigables slide par slide dans l'apercu. Utiliser cette skill des que l'utilisateur veut une presentation, un slide deck, des diapositives, un pitch, un support visuel a montrer a l'ecran, ou transformer un sujet / un document en slides. Produire un fichier HTML autonome avec navigation visible. NE PAS utiliser pour un fichier PowerPoint .pptx (utiliser presentation-pptx), ni pour un simple document texte, un tableur ou un PDF."
when-to-use: "L'utilisateur veut une presentation, des slides, des diapositives, un pitch ou un support visuel a presenter a l'ecran (format web, navigable, et modifiable ensuite)."
always-apply: false
---

# Presentations web (HTML autonome, navigables et modifiables)

Tu produis une presentation sous forme d'**un seul fichier HTML autonome**, rendu comme artifact. L'objectif : un deck superbe, et structure juste assez pour que l'utilisateur puisse ensuite en changer le texte et les couleurs via l'interface. Les deux conventions ci-dessous sont OBLIGATOIRES ; elles n'enlevent rien a ta liberte de design.

## Liberte de creation (priorite n°1)
Fais un travail remarquable. Tu es libre du style, des mises en page, du rythme. Tu es encouragee a enrichir : graphiques en SVG, schemas, diagrammes, petites illustrations vectorielles, frises, quand ils servent le propos. Vise par defaut un rendu premium, sobre et moderne (facon studio de design), mais adapte au sujet. Les conventions techniques plus bas encadrent, elles ne brident pas.

## Convention 1 — Couleurs en variables CSS (pour que l'utilisateur puisse recolorer)
- Declare la palette principale dans `:root`, avec au minimum : `--bg` (fond), `--text` (texte), `--accent` (couleur d'accent). Ajoute si besoin `--muted`, `--accent-2`.
- Utilise UNIQUEMENT ces variables pour les couleurs, partout (titres, traits, fonds, graphiques SVG inclus). Pas de code couleur en dur ailleurs que dans `:root`.

```html
<style>
  :root {
    --bg: #FAF9F6;
    --text: #1A1A1A;
    --accent: #DA291C;
    --muted: #6b7280;
  }
</style>
```

## Convention 2 — Une slide = une section, navigation visible
- Chaque diapositive est une `<section class="slide">...</section>`, enfant direct d'un conteneur. UNE seule slide visible a la fois, en plein cadre, format 16:9.
- Ajoute une navigation VISIBLE et autonome (code-la toi-meme en JavaScript simple, sans librairie) : bouton precedent et bouton suivant bien visibles, plus un compteur (par exemple 03 / 09). Les fleches du clavier doivent aussi marcher.
- Chaque slide s'affiche d'un coup. PAS d'effet ou il faut cliquer plusieurs fois pour reveler le texte morceau par morceau.

## Autonomie technique (robustesse)
- Un seul fichier HTML, totalement autonome. AUCUNE librairie externe, AUCUN CDN, AUCUNE police a telecharger : tout doit fonctionner hors ligne (dans l'apercu comme une fois telecharge). N'utilise PAS reveal.js. Les polices : pile systeme (`-apple-system, "Segoe UI", "Helvetica Neue", Arial, sans-serif`).

## Contenu — du vrai, pas des etiquettes
- Remplis avec du **vrai contenu developpe** sur le sujet demande. Si l'utilisateur a precise un sujet, fais reellement la presentation sur ce sujet (recherche web d'abord si des chiffres recents ou des sources aident, puis cite-les sur une derniere slide). Jamais de "Titre" / "Point 1" / texte bouche-trou.
- Placeholders `[entre crochets]` : UNIQUEMENT pour une information que tu ne peux pas connaitre et que l'utilisateur devra fournir (un chiffre interne, un nom, une date precise). Donne-leur un nom explicite (`[nom du client]`, `[chiffre d'affaires 2025]`). N'en mets PAS par reflexe : si une info n'a pas de sens en placeholder, ne la mets pas. La plupart des presentations doivent avoir zero ou tres peu de placeholders.

## Style d'ecriture
Pas d'emojis, pas de titres en Majuscule A Chaque Mot, pas de remplissage. Phrases nettes, une idee par slide. Respecte les regles de style generales de Lancya (pas de tiret cadratin, ton humain et direct).

## Livrable
Rends le tout dans un seul artifact HTML, pret a etre presente en plein ecran et telecharge.
