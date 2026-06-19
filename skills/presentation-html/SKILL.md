---
name: presentation-html
description: "Cree des presentations au format HTML interactif : un deck affiche AU MILIEU de la conversation, navigable slide par slide, avec plein ecran, telechargement, et modification en direct des couleurs et des textes. Le HTML (autonome, sans librairie externe) est emis dans un bloc de code `lancya_deck`. Utiliser cette skill quand l'utilisateur veut une presentation HTML, web ou interactive : des qu'il dit HTML, 'slides web', 'interactif', une presentation a afficher / presenter dans la conversation ou a l'ecran, ou qu'il veut pouvoir changer ensuite les couleurs et le texte. IMPORTANT : si l'utilisateur veut plutot un FICHIER PowerPoint (.pptx) a telecharger, NE PAS utiliser cette skill mais presentation-pptx. Ne pas utiliser non plus pour un document texte, un tableur ou un PDF."
when-to-use: "L'utilisateur veut une presentation HTML / web / interactive a afficher et modifier dans la conversation, ou precise 'HTML'. Pas pour un fichier PowerPoint .pptx (presentation-pptx)."
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

## Convention 2 — Une slide = une section (la navigation est automatique)
- Chaque diapositive est une `<section class="slide">...</section>`, enfant direct de `<body>`. Concois chaque slide pour remplir tout le cadre, format 16:9.
- N'ajoute PAS toi-meme de navigation (ni boutons precedent/suivant, ni script qui change de slide, ni compteur). L'interface s'en charge automatiquement : elle affiche une slide a la fois, ajoute les boutons, le compteur et les fleches du clavier. Tu te contentes d'ecrire les slides les unes apres les autres.
- Chaque slide s'affiche d'un coup. PAS d'effet ou il faut cliquer plusieurs fois pour reveler le texte morceau par morceau.

## Autonomie technique (robustesse)
- Un seul fichier HTML, totalement autonome. AUCUNE librairie externe, AUCUN CDN, AUCUNE police a telecharger : tout doit fonctionner hors ligne (dans l'apercu comme une fois telecharge). N'utilise PAS reveal.js. Les polices : pile systeme (`-apple-system, "Segoe UI", "Helvetica Neue", Arial, sans-serif`).

## Contenu — du vrai, pas des etiquettes
- Remplis avec du **vrai contenu developpe** sur le sujet demande. Si l'utilisateur a precise un sujet, fais reellement la presentation sur ce sujet (recherche web d'abord si des chiffres recents ou des sources aident, puis cite-les sur une derniere slide). Jamais de "Titre" / "Point 1" / texte bouche-trou.
- Placeholders `[entre crochets]` : UNIQUEMENT pour une information que tu ne peux pas connaitre et que l'utilisateur devra fournir (un chiffre interne, un nom, une date precise). Donne-leur un nom explicite (`[nom du client]`, `[chiffre d'affaires 2025]`). N'en mets PAS par reflexe : si une info n'a pas de sens en placeholder, ne la mets pas. La plupart des presentations doivent avoir zero ou tres peu de placeholders.

## Style d'ecriture
Pas d'emojis, pas de titres en Majuscule A Chaque Mot, pas de remplissage. Phrases nettes, une idee par slide. Respecte les regles de style generales de Lancya (pas de tiret cadratin, ton humain et direct).

## Livrable
Rends le deck dans un bloc de code dont le langage est EXACTEMENT `lancya_deck`, contenant le HTML complet (de `<!DOCTYPE html>` a `</html>`) :

```lancya_deck
<!DOCTYPE html>
<html lang="fr">
... le deck complet (slides en sections, palette en :root) ...
</html>
```

Emets UNIQUEMENT ce bloc dans ta reponse : RIEN avant, RIEN apres (le bloc est le message entier, pas de phrase d'introduction ni de conclusion). C'est important : l'interface affiche le bloc comme une presentation interactive (navigation, plein ecran, telechargement) ET enregistre les modifications de l'utilisateur (couleurs, texte) directement dans ce bloc, pour que tu repartes de leur version si on te redemande de la modifier. Une phrase autour du bloc casserait cette sauvegarde.
