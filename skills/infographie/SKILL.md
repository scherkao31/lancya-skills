---
name: infographie
description: "Cree une infographie : un visuel unique vertical (format affiche / portrait) affiche AU MILIEU de la conversation, modifiable (couleurs, textes) et exportable en image PNG et PDF, annotable. HTML/SVG autonome (aucune librairie) emis dans un bloc de code `lancya_visual`. Utiliser cette skill quand l'utilisateur veut une INFOGRAPHIE ou un visuel one-page qui met en avant des chiffres cles, des points cles, des etapes ou une idee de maniere graphique et impactante (affiche, poster, recap visuel, 'infographic'). IMPORTANT : pour un graphique de chiffres seul utiliser graphique ; pour un carrousel reseaux sociaux (plusieurs cartes) utiliser carrousel ; pour une presentation utiliser presentation-html ; pour un schema de flux utiliser schema. Ici on produit UN visuel vertical a afficher et retoucher."
when-to-use: "L'utilisateur veut une infographie / un visuel unique vertical (affiche, poster, recap de chiffres cles ou points cles, mise en avant graphique) a afficher et modifier dans la conversation. Pas un carrousel multi-cartes, pas une presentation."
always-apply: false
---

# Infographie (visuel vertical autonome, modifiable)

Tu produis **un seul visuel vertical** dans un fichier HTML autonome, rendu comme widget au milieu de la conversation. L'objectif : une infographie claire et marquante, structuree juste assez pour que l'utilisateur puisse ensuite en changer les couleurs et les textes via l'interface.

## Liberte de creation
Fais une infographie remarquable : titre fort en haut, puis une lecture verticale fluide (chiffres cles mis en avant, sections, pictos vectoriels simples en SVG, petits graphes si utile). Hierarchie visuelle nette, beaucoup d'air, un ou deux accents de couleur. Sobre et moderne par defaut.

## Convention 1 — Couleurs en variables CSS
- Declare la palette dans `:root` : au minimum `--bg`, `--text`, `--accent` ; ajoute `--accent-2`, `--muted` au besoin.
- Utilise UNIQUEMENT ces variables pour toutes les couleurs, SVG inclus. Aucun code couleur en dur hors de `:root`.

## Convention 2 — Canevas vertical
- Tout le visuel tient dans une seule `<section class="slide">`, enfant direct de `<body>`, concue pour un canevas FIXE **portrait de 1080 x 1350 px** (format 4:5, ideal affiche / reseaux). Declare ce format avec, dans le `<head>` :

```html
<meta name="lancya-canvas" content="1080x1350">
```

- Dimensionne titres, chiffres et espacements pour ce portrait. L'interface met le canevas a l'echelle automatiquement (apercu et plein ecran). N'ajoute AUCUNE navigation ni bouton.

## Comment dessiner (autonome, sans librairie)
- HTML/CSS pour la mise en page, **SVG inline** pour pictos, petits graphes et formes. AUCUNE librairie, AUCUN CDN, AUCUNE police a telecharger (pile systeme).
- Mets les **chiffres cles en gros**, avec leur libelle dessous. Le texte reste du vrai texte (editable et net a l'export), jamais aplati en image.

## Contenu — du vrai
- Remplis avec du vrai contenu sur le sujet demande (vrais chiffres, vrais points). Si des donnees recentes et publiques aident et sont demandees, recherche puis cite la source en petit en bas.
- Placeholders `[entre crochets]` UNIQUEMENT pour une information que tu ne peux pas connaitre. N'en mets pas par reflexe.

## Style
Pas d'emojis decoratifs a outrance, pas de Majuscule A Chaque Mot. Textes courts et percutants. Respecte les regles de style Lancya (pas de tiret cadratin, ton humain et direct).

## Livrable
Rends l'infographie dans un bloc de code dont le langage est EXACTEMENT `lancya_visual`, contenant le HTML complet (de `<!DOCTYPE html>` a `</html>`), avec le `<meta name="lancya-canvas" content="1080x1350">`, la `<section class="slide">` et la palette en `:root` :

```lancya_visual
<!DOCTYPE html>
<html lang="fr">
... l'infographie complete (section.slide 1080x1350, palette en :root) ...
</html>
```

Emets UNIQUEMENT ce bloc : RIEN avant, RIEN apres (le bloc est le message entier). L'interface affiche le bloc comme un visuel interactif (couleurs, edition, export image/PDF, annotation) ET enregistre les modifications de l'utilisateur dans ce bloc, pour que tu repartes de leur version si on te redemande de le modifier. Une phrase autour du bloc casserait cette sauvegarde.
