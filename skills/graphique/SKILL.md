---
name: graphique
description: "Cree un graphique de donnees affiche AU MILIEU de la conversation, modifiable (couleurs, valeurs, libelles), exportable en PNG et PDF, et annotable. Le graphe est dessine en SVG/CSS autonome (aucune librairie) et emis dans un bloc de code `lancya_chart`. Utiliser cette skill des que l'utilisateur veut VISUALISER des chiffres : un graphique, un diagramme en barres, des colonnes, une courbe / un graphe en ligne, un camembert, un donut, un comparatif chiffre, une evolution, une repartition, un 'graph'. IMPORTANT : pour un FICHIER tableur Excel (.xlsx) utiliser tableur-excel ; pour un TABLEAU de donnees (lignes/colonnes de texte) utiliser tableau ; pour une presentation complete utiliser presentation-html ; pour une analyse approfondie de donnees utiliser analyse-donnees. Ici on produit UN graphique a afficher et retoucher."
when-to-use: "L'utilisateur veut un graphique / un visuel de chiffres (barres, colonnes, courbe, camembert, donut, repartition, evolution) a afficher et modifier dans la conversation. Pas un fichier Excel, pas un tableau de texte."
always-apply: false
---

# Graphique de donnees (SVG autonome, modifiable)

Tu produis **un seul graphique** dans un fichier HTML autonome, rendu comme widget au milieu de la conversation. L'objectif : un graphe clair, juste et beau, structure juste assez pour que l'utilisateur puisse ensuite changer ses couleurs et ses textes via l'interface.

## Liberte de creation
Fais un graphe remarquable et lisible : bon choix de type (barres, colonnes, courbe, aires, camembert, donut, jauge) selon la donnee, titre clair, axes et echelles corrects, libelles et valeurs lisibles, legende si utile. Sobre et moderne par defaut. Choisis le type adapte : comparer des categories -> barres/colonnes ; une evolution dans le temps -> courbe ou aires ; une repartition d'un tout -> camembert/donut.

## Convention 1 — Couleurs en variables CSS
- Declare la palette dans `:root` : au minimum `--bg`, `--text`, `--accent` ; ajoute `--accent-2`, `--muted` au besoin (utile pour plusieurs series).
- Utilise UNIQUEMENT ces variables pour toutes les couleurs, y compris dans le SVG (`fill="var(--accent)"`, `stroke="var(--text)"`). Aucun code couleur en dur hors de `:root`.

## Convention 2 — Un canevas unique
- Tout le graphe tient dans une seule `<section class="slide">`, enfant direct de `<body>`, concue pour un canevas FIXE de **1280 x 720 px** (16:9). Dimensionne titre, axes et police pour cette taille. L'interface met ce canevas a l'echelle automatiquement (apercu et plein ecran) ; tu n'as pas a gerer le responsive.
- N'ajoute AUCUNE navigation ni bouton : l'interface fournit deja edition, couleurs, export et plein ecran.

## Comment dessiner (autonome, sans librairie)
- Dessine en **SVG inline** (le plus fiable) ou en CSS. AUCUNE librairie, AUCUN CDN, AUCun script de charting.
- Donne au SVG un `viewBox` et laisse-le remplir le canevas. Calcule toi-meme les hauteurs de barres, points de courbe et arcs a partir des valeurs (echelle proportionnelle a la valeur max).
- Mets les **valeurs chiffrees** en clair (au-dessus des barres, au bout des points, dans la legende du camembert) : c'est ce que l'utilisateur veut lire.
- Camembert / donut : trace les parts avec des `<path>` d'arc ou des cercles `stroke-dasharray`. Verifie que la somme des parts est coherente (100 % pour une repartition).
- Texte des libelles et valeurs en vrai texte SVG (`<text>`), pas en image, pour qu'il reste editable et net a l'export.

## Contenu — du vrai
- Utilise les **vraies donnees** fournies par l'utilisateur. S'il n'en donne pas mais demande un exemple, utilise des chiffres plausibles et nomme-les clairement. Si des chiffres recents et publics aident (et qu'il les demande sur un sujet reel), fais une recherche web puis cite la source en petit sous le graphe.
- Placeholders `[entre crochets]` UNIQUEMENT pour une donnee que tu ne peux pas connaitre (ex. `[chiffre d'affaires 2025]`). N'en mets pas par reflexe.

## Style
Pas d'emojis, pas de Majuscule A Chaque Mot. Titre court et factuel. Respecte les regles de style Lancya (pas de tiret cadratin, ton humain et direct).

## Livrable
Rends le graphique dans un bloc de code dont le langage est EXACTEMENT `lancya_chart`, contenant le HTML complet (de `<!DOCTYPE html>` a `</html>`), avec la `<section class="slide">` et la palette en `:root` :

```lancya_chart
<!DOCTYPE html>
<html lang="fr">
... le graphique complet (SVG dans une section.slide, palette en :root) ...
</html>
```

Emets UNIQUEMENT ce bloc : RIEN avant, RIEN apres (le bloc est le message entier). L'interface affiche le bloc comme un graphique interactif (couleurs, edition, export PNG/PDF, annotation) ET enregistre les modifications de l'utilisateur dans ce bloc, pour que tu repartes de leur version si on te redemande de le modifier. Une phrase autour du bloc casserait cette sauvegarde.
