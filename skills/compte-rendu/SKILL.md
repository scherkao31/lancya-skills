---
name: compte-rendu
description: "Transforme des notes de reunion ou de rendez-vous en un DOCUMENT interactif et soigne (compte rendu, synthese, rapport), affiche dans un widget au milieu de la conversation : on peut l'editer, changer les couleurs, et l'exporter en PDF ou HTML. Le document est emis dans un bloc de code `lancya_doc` et peut contenir des schemas (frise, organigramme, points cles) en SVG. Utiliser cette skill quand l'utilisateur veut un compte rendu, une prise de notes mise au propre, une synthese de reunion / rendez-vous, un releve de decisions, un plan d'actions, ou un rapport structure a presenter et garder. NE PAS utiliser pour une simple reponse conversationnelle, ni pour un fichier Word .docx (autre chemin), ni pour des slides (presentation-html) ni un PowerPoint (presentation-pptx)."
when-to-use: "L'utilisateur veut un compte rendu, une prise de notes au propre, une synthese de reunion / rendez-vous, un releve de decisions, un plan d'actions, ou un rapport, sous forme de document soigne, editable et exportable."
always-apply: false
---

# Document interactif (compte rendu / synthese, HTML editable)

Tu produis un DOCUMENT soigne sous forme d'**un seul fichier HTML autonome**, rendu dans un widget. L'utilisateur pourra l'editer, le recolorer et l'exporter en PDF. Vise un rendu professionnel, clair et agreable a lire.

## Liberte de creation (priorite)
Fais un beau document, bien structure. Tu es encouragee a ajouter des **schemas en SVG** quand ils aident a comprendre : frise temporelle des etapes, mini organigramme des roles, schema des points cles ou du flux de decisions, encadres de synthese. Soigne la typographie et l'espacement. Les conventions ci-dessous encadrent, elles ne brident pas.

## Format et conventions (OBLIGATOIRES)
- **Un seul fichier HTML autonome**, sans aucune librairie externe ni CDN (tout doit marcher hors ligne). Polices systeme (`-apple-system, "Segoe UI", "Helvetica Neue", Arial, sans-serif`).
- **Couleurs en variables CSS** : declare la palette dans `:root` avec au minimum `--bg`, `--text`, `--accent` (ajoute `--muted`, `--accent-2` si besoin) et utilise UNIQUEMENT ces variables pour les couleurs, partout (schemas SVG inclus). Cela permet a l'utilisateur de recolorer le document.
- **Concu pour une page A4 portrait** : largeur de contenu autour de 760 px, marges confortables. Le document DEFILE (ce n'est pas des slides). Il sera exporte en PDF A4.

## Structure type d'un compte rendu (a adapter au contenu)
Un en-tete (titre, date, participants si fournis), une synthese courte en haut, puis les sections utiles : points abordes, decisions prises, et un plan d'actions (tache, responsable, echeance) sous forme de liste ou de tableau. Ajoute un schema SVG la ou il eclaire vraiment (par ex. une frise des prochaines etapes).

## Contenu — du vrai, pas des etiquettes
- Remplis avec le VRAI contenu fourni par l'utilisateur (ses notes, ce qu'il a dit). Ne mets pas de "Point 1" / texte bouche-trou.
- Placeholders `[entre crochets]` UNIQUEMENT pour une information que tu ne peux pas connaitre et que l'utilisateur devra completer (un nom, une date, un montant precis), avec un nom explicite (`[date du rendez-vous]`, `[nom du responsable]`). Jamais par reflexe : la plupart du temps, zero ou tres peu de placeholders.

## Style
Pas d'emojis, pas de titres en Majuscule A Chaque Mot, pas de remplissage. Ton professionnel, clair, francais naturel. Respecte les regles de style generales de Lancya (pas de tiret cadratin).

## Livrable
Emets UNIQUEMENT le document dans un bloc de code dont le langage est EXACTEMENT `lancya_doc`, contenant le HTML complet (de `<!DOCTYPE html>` a `</html>`). RIEN avant, RIEN apres (le bloc est le message entier) : l'interface l'affiche comme un document interactif ET enregistre les modifications de l'utilisateur dans ce bloc.

```lancya_doc
<!DOCTYPE html>
<html lang="fr">
... le document complet (palette en :root, contenu reel, schemas SVG si utiles) ...
</html>
```
