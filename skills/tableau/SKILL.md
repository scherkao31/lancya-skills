---
name: tableau
description: "Cree un tableau de donnees affiche AU MILIEU de la conversation, avec cellules MODIFIABLES, export CSV (ouvrable dans Excel) et PDF, couleurs editables et annotation. HTML autonome emis dans un bloc de code `lancya_table`. Utiliser cette skill quand l'utilisateur veut un TABLEAU a afficher et retoucher dans la conversation : un comparatif, une matrice, un planning, un budget en lignes/colonnes, une grille de donnees, un recapitulatif structure en colonnes. IMPORTANT : si l'utilisateur veut un FICHIER tableur Excel (.xlsx) a telecharger avec formules, utiliser tableur-excel ; pour un graphique de chiffres utiliser graphique ; pour une presentation utiliser presentation-html. Ici on produit un tableau interactif a afficher, editer et exporter en CSV."
when-to-use: "L'utilisateur veut un tableau / comparatif / matrice / planning / grille de donnees a afficher, editer et exporter (CSV) dans la conversation. Pas un fichier Excel .xlsx avec formules (tableur-excel)."
always-apply: false
---

# Tableau de donnees (HTML autonome, editable, export CSV)

Tu produis **un tableau** dans un fichier HTML autonome, rendu comme widget au milieu de la conversation. L'objectif : un tableau clair et bien mis en forme, structure juste assez pour que l'utilisateur puisse ensuite editer les cellules, changer les couleurs, et l'exporter en CSV (Excel) ou PDF via l'interface.

## Liberte de creation
Fais un tableau lisible et soigne : en-tete de colonnes net, lignes aerees, alignement coherent (texte a gauche, nombres a droite), un leger zebrage des lignes si utile, mise en valeur discrete de l'en-tete. Sobre et moderne.

## Convention 1 — Couleurs en variables CSS
- Declare la palette dans `:root` : au minimum `--bg`, `--text`, `--accent` ; ajoute `--muted`, `--accent-2` au besoin (en-tete, bordures, zebrage).
- Utilise UNIQUEMENT ces variables pour les couleurs (fond d'en-tete, bordures, texte). Aucun code couleur en dur hors de `:root`.

## Convention 2 — Un vrai tableau HTML
- Le contenu central DOIT etre au moins un `<table>` avec `<thead>` (les `<th>` d'en-tete) et `<tbody>` (les lignes `<tr>` de `<td>`). C'est ce qui permet l'edition cellule par cellule et l'export CSV.
- Mets une cellule par donnee (ne fusionne pas plusieurs valeurs dans une meme cellule). Premiere ligne = les intitules de colonnes.
- Tu peux ajouter un titre (`<h2>`) au-dessus et une note de source en dessous, mais le coeur reste le `<table>`.
- N'ajoute AUCUN bouton ni script : l'interface fournit deja edition, couleurs, export CSV/PDF et plein ecran. Le tableau s'affiche a sa hauteur naturelle (pas besoin de gerer une taille fixe).

## Autonomie technique
- Un seul fichier HTML autonome. AUCUNE librairie, AUCUN CDN, AUCUNE police a telecharger (pile systeme `-apple-system, "Segoe UI", "Helvetica Neue", Arial, sans-serif`).

## Contenu — du vrai
- Remplis avec les **vraies donnees** demandees. S'il manque des valeurs que tu ne peux pas connaitre, mets un placeholder nomme `[entre crochets]` (`[montant], [date]`), avec parcimonie. Sinon, du vrai contenu, jamais "Colonne 1 / Valeur".
- Si des chiffres recents et publics aident et sont demandes sur un sujet reel, recherche puis cite la source sous le tableau.

## Style
Pas d'emojis, pas de Majuscule A Chaque Mot. Intitules de colonnes courts. Respecte les regles de style Lancya (pas de tiret cadratin, ton humain et direct).

## Livrable
Rends le tableau dans un bloc de code dont le langage est EXACTEMENT `lancya_table`, contenant le HTML complet (de `<!DOCTYPE html>` a `</html>`), avec le `<table>` et la palette en `:root` :

```lancya_table
<!DOCTYPE html>
<html lang="fr">
... le tableau complet (un <table> thead/tbody, palette en :root) ...
</html>
```

Emets UNIQUEMENT ce bloc : RIEN avant, RIEN apres (le bloc est le message entier). L'interface affiche le bloc comme un tableau interactif (edition des cellules, couleurs, export CSV/PDF, annotation) ET enregistre les modifications de l'utilisateur dans ce bloc, pour que tu repartes de leur version si on te redemande de le modifier. Une phrase autour du bloc casserait cette sauvegarde.
