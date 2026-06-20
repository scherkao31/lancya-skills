---
name: schema
description: "Cree un schema / diagramme affiche AU MILIEU de la conversation, modifiable (couleurs, titre), exportable en PNG et PDF, annotable. Tu emets seulement les NOEUDS et les LIENS (pas le dessin) : un moteur de rendu integre place les boites et trace les fleches proprement. Bloc de code `lancya_diagram`. Utiliser cette skill quand l'utilisateur veut representer une STRUCTURE ou un FLUX : un schema, un diagramme, un logigramme, un organigramme, un processus / des etapes, un parcours, une arborescence, un flux, un cycle, un 'flowchart'. IMPORTANT : pour un graphique de CHIFFRES (barres, courbe, camembert) utiliser graphique ; pour une presentation complete utiliser presentation-html ; pour un tableau de donnees utiliser tableau. Ici on produit UN schema a afficher et retoucher."
when-to-use: "L'utilisateur veut un schema / diagramme de structure ou de flux (processus, etapes, organigramme, arborescence, cycle, logigramme) a afficher et modifier dans la conversation. Pas un graphique de chiffres."
always-apply: false
---

# Schema / diagramme (tu emets les noeuds et les liens, le moteur place tout)

Tu produis **un schema** dans un fichier HTML autonome, rendu comme widget au milieu de la conversation. Point cle : **tu ne dessines PAS le SVG a la main**. Tu decris les **noeuds** et les **liens**, et le moteur de rendu integre calcule la mise en page (placement des boites, niveaux, routage des fleches). Resultat : toujours aligne et lisible, sans fleches qui se croisent.

## Comment : un conteneur de donnees
Le coeur du schema est un element vide :

```html
<div class="lancya-diagram" data-spec='{ ...donnees JSON... }'></div>
```

- `data-spec` contient du **JSON valide** (guillemets doubles a l'interieur), l'attribut entoure de guillemets simples. Laisse le `<div>` VIDE : le moteur le remplit.
- Apostrophe dans un libelle : ecris-la `&#39;`. Evite `<`, `>`, `&` dans les libelles.

### Format de `data-spec`
```json
{
  "layout": "flow",
  "direction": "lr",
  "nodes": [
    { "id": "a", "label": "Demande recue", "sub": "formulaire web" },
    { "id": "b", "label": "Qualification" },
    { "id": "c", "label": "Devis envoye" }
  ],
  "edges": [
    { "from": "a", "to": "b" },
    { "from": "b", "to": "c", "label": "si interesse" }
  ]
}
```
- `layout` :
  - `"flow"` : une suite d'etapes reliees (processus, parcours). `direction` `"lr"` (de gauche a droite) ou `"tb"` (de haut en bas).
  - `"tree"` : une hierarchie / organigramme (un parent vers plusieurs enfants via `edges`). `direction` `"tb"` (defaut) ou `"lr"`.
  - `"cycle"` : une boucle (les noeuds s'enchainent et reviennent au depart).
- `nodes` : chaque boite a un `id` unique, un `label` court, et un `sub` optionnel (precision en plus petit).
- `edges` : les fleches `{ "from": id, "to": id, "label": "optionnel" }`. Pour `flow` et `cycle`, si tu omets `edges`, le moteur enchaine les noeuds dans l'ordre.

Les **couleurs viennent automatiquement** de la palette (`--accent` pour les bordures et fleches) : tu ne colories rien toi-meme.

## Gabarit complet a suivre
```html
<!DOCTYPE html>
<html lang="fr">
<head>
<meta charset="utf-8">
<style>
  :root { --bg:#FAF9F6; --text:#1A1A1A; --accent:#DA291C; --accent-2:#1A1A1A; --muted:#6b7280; }
  html, body { margin: 0; }
  .slide { display: flex; flex-direction: column; box-sizing: border-box; padding: 44px 52px;
           background: var(--bg); color: var(--text);
           font-family: -apple-system, "Segoe UI", "Helvetica Neue", Arial, sans-serif; }
  .diagram-title { font-size: 32px; font-weight: 700; margin: 0 0 4px; }
  .diagram-sub { font-size: 16px; color: var(--muted); margin: 0 0 16px; }
  .lancya-diagram { flex: 1; min-height: 0; }
</style>
</head>
<body>
<section class="slide">
  <h2 class="diagram-title">Parcours d&#39;une demande client</h2>
  <p class="diagram-sub">De la reception au devis</p>
  <div class="lancya-diagram" data-spec='{"layout":"flow","direction":"lr","nodes":[{"id":"a","label":"Demande recue"},{"id":"b","label":"Qualification"},{"id":"c","label":"Devis envoye"}],"edges":[{"from":"a","to":"b"},{"from":"b","to":"c"}]}'></div>
</section>
</body>
</html>
```

## Regles
- UNE seule `<section class="slide">`, concue pour un canevas 1280 x 720 (16:9). Le titre est du vrai texte (editable) ; le schema est rendu par le moteur. N'ajoute AUCUN script ni SVG dessine a la main.
- Choisis le bon layout : etapes qui s'enchainent -> `flow` ; hierarchie / qui depend de qui -> `tree` ; boucle qui revient -> `cycle`.
- Couleurs UNIQUEMENT en `:root`. Libelles courts (le moteur gere le retour a la ligne, mais vise 1 a 4 mots par boite).

## Contenu — du vrai
- Represente reellement la structure ou le processus demande, avec les vraies etapes / entites. Pas de "Etape 1 / Etape 2" generique si un sujet est donne.
- Placeholders `[entre crochets]` uniquement pour une information que tu ne peux pas connaitre.

## Style
Pas d'emojis, pas de Majuscule A Chaque Mot. Libelles concrets. Regles de style Lancya (pas de tiret cadratin, ton humain et direct).

## Livrable
Rends le tout dans un bloc de code dont le langage est EXACTEMENT `lancya_diagram`, contenant le HTML complet (de `<!DOCTYPE html>` a `</html>`) :

```lancya_diagram
<!DOCTYPE html>
<html lang="fr">
... le HTML complet (slide + div.lancya-diagram avec data-spec, palette en :root) ...
</html>
```

Emets UNIQUEMENT ce bloc : RIEN avant, RIEN apres (le bloc est le message entier). L'interface affiche le bloc comme un schema interactif (couleurs, edition du titre, export PNG/PDF, annotation) ET enregistre les modifications dans ce bloc (elle ne garde que les donnees `data-spec`, pas le dessin), pour que tu repartes de cette version si on te redemande de la modifier. Une phrase autour du bloc casserait cette sauvegarde.
