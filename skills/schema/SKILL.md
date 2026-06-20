---
name: schema
description: "Cree un schema / diagramme affiche AU MILIEU de la conversation, modifiable (couleurs, libelles) et exportable en PNG et PDF, annotable. Dessine en SVG/CSS autonome (aucune librairie) et emis dans un bloc de code `lancya_diagram`. Utiliser cette skill quand l'utilisateur veut representer une STRUCTURE ou un FLUX : un schema, un diagramme, un logigramme, un organigramme, un processus / des etapes, un parcours, une arborescence, un flux, une carte conceptuelle, un cycle, un 'flowchart'. IMPORTANT : pour un graphique de CHIFFRES (barres, courbe, camembert) utiliser graphique ; pour une presentation complete utiliser presentation-html ; pour un tableau de donnees utiliser tableau. Ici on produit UN schema a afficher et retoucher."
when-to-use: "L'utilisateur veut un schema / diagramme de structure ou de flux (processus, etapes, organigramme, arborescence, cycle, logigramme) a afficher et modifier dans la conversation. Pas un graphique de chiffres."
always-apply: false
---

# Schema / diagramme (SVG autonome, modifiable)

Tu produis **un seul schema** dans un fichier HTML autonome, rendu comme widget au milieu de la conversation. L'objectif : un schema clair qui se lit d'un coup d'oeil, structure juste assez pour que l'utilisateur puisse ensuite en changer les couleurs et les libelles via l'interface.

## Liberte de creation
Fais un schema remarquable et lisible : choisis la bonne forme selon le propos (chaine d'etapes avec fleches, organigramme hierarchique, cycle en boucle, arborescence, matrice, parcours). Boites nettes, fleches orientees, hierarchie visible, libelles courts. Sobre et moderne par defaut.

## Convention 1 — Couleurs en variables CSS
- Declare la palette dans `:root` : au minimum `--bg`, `--text`, `--accent` ; ajoute `--accent-2`, `--muted` au besoin.
- Utilise UNIQUEMENT ces variables pour toutes les couleurs, y compris dans le SVG (`fill="var(--accent)"`, `stroke="var(--text)"`). Aucun code couleur en dur hors de `:root`.

## Convention 2 — Un canevas unique
- Tout le schema tient dans une seule `<section class="slide">`, enfant direct de `<body>`, concue pour un canevas FIXE de **1280 x 720 px** (16:9). Dimensionne formes, fleches et polices pour cette taille. L'interface met ce canevas a l'echelle automatiquement (apercu et plein ecran).
- N'ajoute AUCUNE navigation ni bouton : l'interface fournit deja edition, couleurs, export et plein ecran.

## Comment dessiner (autonome, sans librairie)
- Dessine en **SVG inline** (recommande pour les fleches et connexions) ou avec des boites HTML/CSS positionnees. AUCUNE librairie, AUCUN CDN (pas de mermaid, pas de d3).
- Fleches : utilise un `<marker>` de fleche reutilisable, ou des `<polygon>` aux extremites des traits. Aligne proprement les boites (grille mentale) pour que les connexions soient droites et lisibles.
- Les libelles en vrai texte (`<text>` SVG ou texte HTML), pas en image, pour rester editables et nets a l'export.
- Laisse de l'air : marges et espaces entre les boites, pas de surcharge.

## Contenu — du vrai
- Represente reellement la structure ou le processus demande, avec les vraies etapes / entites. Pas de "Etape 1 / Etape 2" generique si l'utilisateur a donne un sujet.
- Placeholders `[entre crochets]` UNIQUEMENT pour une information que tu ne peux pas connaitre (un nom de service, une personne). N'en mets pas par reflexe.

## Style
Pas d'emojis, pas de Majuscule A Chaque Mot. Libelles courts et concrets. Respecte les regles de style Lancya (pas de tiret cadratin, ton humain et direct).

## Livrable
Rends le schema dans un bloc de code dont le langage est EXACTEMENT `lancya_diagram`, contenant le HTML complet (de `<!DOCTYPE html>` a `</html>`), avec la `<section class="slide">` et la palette en `:root` :

```lancya_diagram
<!DOCTYPE html>
<html lang="fr">
... le schema complet (SVG/boites dans une section.slide, palette en :root) ...
</html>
```

Emets UNIQUEMENT ce bloc : RIEN avant, RIEN apres (le bloc est le message entier). L'interface affiche le bloc comme un schema interactif (couleurs, edition, export PNG/PDF, annotation) ET enregistre les modifications de l'utilisateur dans ce bloc, pour que tu repartes de leur version si on te redemande de le modifier. Une phrase autour du bloc casserait cette sauvegarde.
