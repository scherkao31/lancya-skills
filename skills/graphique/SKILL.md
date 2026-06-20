---
name: graphique
description: "Cree un graphique de donnees affiche AU MILIEU de la conversation, modifiable (couleurs, titres), exportable en PNG et PDF, et annotable. Tu emets seulement les DONNEES (pas le dessin) : un moteur de rendu integre dessine un SVG propre. Bloc de code `lancya_chart`. Utiliser cette skill des que l'utilisateur veut VISUALISER des chiffres : un graphique, un diagramme en barres, des colonnes, une courbe / un graphe en ligne, un camembert, un donut, un comparatif chiffre, une evolution, une repartition, un 'graph'. IMPORTANT : pour un FICHIER tableur Excel (.xlsx) utiliser tableur-excel ; pour un TABLEAU de donnees (lignes/colonnes de texte) utiliser tableau ; pour une presentation complete utiliser presentation-html ; pour une analyse approfondie utiliser analyse-donnees. Ici on produit UN graphique a afficher et retoucher."
when-to-use: "L'utilisateur veut un graphique / un visuel de chiffres (barres, colonnes, courbe, camembert, donut, repartition, evolution) a afficher et modifier dans la conversation. Pas un fichier Excel, pas un tableau de texte."
always-apply: false
---

# Graphique de donnees (tu emets les donnees, le moteur dessine)

Tu produis **un graphique** dans un fichier HTML autonome, rendu comme widget au milieu de la conversation. Point cle : **tu ne dessines PAS le SVG a la main**. Tu fournis les donnees dans un element prevu pour, et le moteur de rendu integre a l'interface calcule la geometrie et dessine un graphe propre (echelles justes, barres alignees, arcs corrects). C'est plus fiable et toujours net.

## Comment : un conteneur de donnees
Le coeur du graphe est un element vide :

```html
<div class="lancya-chart" data-spec='{ ...donnees JSON... }'></div>
```

- L'attribut `data-spec` contient du **JSON valide** (guillemets doubles a l'interieur), et l'attribut lui-meme est entoure de guillemets simples. Laisse le `<div>` VIDE : le moteur le remplit.
- Si une valeur texte contient une apostrophe, ecris-la `&#39;` (ex. `"chiffre d&#39;affaires"`). Evite `<`, `>`, `&` dans les libelles.

### Format de `data-spec`
```json
{
  "type": "bar",
  "labels": ["2021", "2022", "2023", "2024"],
  "series": [ { "name": "CA", "values": [1.2, 1.5, 1.9, 2.4] } ],
  "unit": "M",
  "showValues": true
}
```
- `type` : `"bar"` (colonnes verticales), `"hbar"` (barres horizontales, bien pour des libelles longs), `"line"` (courbe), `"area"` (courbe remplie), `"pie"` (camembert), `"donut"`.
- `labels` : les categories (axe X) ou les parts (pour pie/donut).
- `series` : une ou plusieurs series `{ "name": ..., "values": [...] }`. Pour pie/donut, mets UNE seule serie (une valeur par part). Plusieurs series = barres groupees ou plusieurs courbes.
- `unit` (optionnel) : suffixe affiche apres les valeurs (`"M"`, `"%"`, `"CHF"`...).
- `showValues` (optionnel, defaut true) : afficher les valeurs chiffrees.

Les **couleurs viennent automatiquement** de `--accent` et `--accent-2` : tu n'as pas a colorer les barres/parts toi-meme.

## Gabarit complet a suivre
```html
<!DOCTYPE html>
<html lang="fr">
<head>
<meta charset="utf-8">
<style>
  :root { --bg:#FAF9F6; --text:#1A1A1A; --accent:#DA291C; --accent-2:#1A1A1A; --muted:#6b7280; }
  html, body { margin: 0; }
  .slide { display: flex; flex-direction: column; box-sizing: border-box; padding: 48px 56px;
           background: var(--bg); color: var(--text);
           font-family: -apple-system, "Segoe UI", "Helvetica Neue", Arial, sans-serif; }
  .chart-title { font-size: 34px; font-weight: 700; margin: 0 0 4px; }
  .chart-sub { font-size: 16px; color: var(--muted); margin: 0 0 20px; }
  .lancya-chart { flex: 1; min-height: 0; }
  .chart-source { font-size: 13px; color: var(--muted); margin: 16px 0 0; }
</style>
</head>
<body>
<section class="slide">
  <h2 class="chart-title">Chiffre d&#39;affaires annuel</h2>
  <p class="chart-sub">2021 a 2024, en millions de CHF</p>
  <div class="lancya-chart" data-spec='{"type":"bar","labels":["2021","2022","2023","2024"],"series":[{"name":"CA","values":[1.2,1.5,1.9,2.4]}],"unit":"M"}'></div>
  <p class="chart-source">Source : [a preciser]</p>
</section>
</body>
</html>
```

## Regles
- UNE seule `<section class="slide">`, concue pour un canevas 1280 x 720 (16:9). Le titre et la source sont du vrai texte (editables) ; le graphe est rendu par le moteur. N'ajoute AUCUN script ni SVG dessine a la main.
- Choisis le bon type : comparer des categories -> bar/hbar ; evolution dans le temps -> line/area ; repartition d'un tout -> pie/donut.
- Couleurs UNIQUEMENT en `:root` (`--bg`, `--text`, `--accent`, `--accent-2`, `--muted`). Rien en dur ailleurs.

## Contenu — du vrai
- Utilise les **vraies donnees** fournies. Si l'utilisateur demande un exemple sans donnees, mets des chiffres plausibles. Si des chiffres recents et publics aident et sont demandes sur un sujet reel, recherche puis cite la source dans `.chart-source`.
- Placeholders `[entre crochets]` uniquement pour une donnee que tu ne peux pas connaitre.

## Style
Pas d'emojis, pas de Majuscule A Chaque Mot. Titre court et factuel. Regles de style Lancya (pas de tiret cadratin, ton humain et direct).

## Livrable
Rends le tout dans un bloc de code dont le langage est EXACTEMENT `lancya_chart`, contenant le HTML complet (de `<!DOCTYPE html>` a `</html>`) :

```lancya_chart
<!DOCTYPE html>
<html lang="fr">
... le HTML complet (slide + div.lancya-chart avec data-spec, palette en :root) ...
</html>
```

Emets UNIQUEMENT ce bloc : RIEN avant, RIEN apres (le bloc est le message entier). L'interface affiche le bloc comme un graphique interactif (couleurs, edition du titre, export PNG/PDF, annotation) ET enregistre les modifications dans ce bloc (elle ne garde que les donnees `data-spec`, pas le dessin), pour que tu repartes de cette version si on te redemande de la modifier. Une phrase autour du bloc casserait cette sauvegarde.
