---
name: carrousel
description: "Cree un carrousel pour LinkedIn ou Instagram : une suite de cartes verticales (format 4:5) affichee dans un widget interactif au milieu de la conversation, qu'on peut editer, recolorer et EXPORTER EN PDF pour publier (un carrousel LinkedIn est un PDF). Les cartes sont emises dans un bloc de code `lancya_carousel`, HTML autonome. Utiliser cette skill quand l'utilisateur veut un carrousel LinkedIn / Instagram, un post a slides pour les reseaux sociaux, du contenu a swiper, ou transformer une idee / un article en carrousel. NE PAS utiliser pour une presentation a projeter a l'ecran (presentation-html), un PowerPoint (presentation-pptx), un document/compte rendu (compte-rendu), ni un simple texte de post (le rediger normalement)."
when-to-use: "L'utilisateur veut un carrousel LinkedIn ou Instagram, un post a swiper, du contenu reseaux sociaux en plusieurs cartes a publier (et exporter en PDF)."
always-apply: false
---

# Carrousel reseaux sociaux (LinkedIn / Instagram, cartes 4:5)

Tu produis un carrousel sous forme d'**un seul fichier HTML autonome**, rendu dans un widget et exportable en PDF (une carte = une page, pret a publier sur LinkedIn).

## Liberte de creation (priorite)
Fais des cartes belles et percutantes, pensees pour etre vues en petit sur mobile : **gros texte lisible**, une seule idee par carte, beaucoup d'espace, contraste fort. Tu peux ajouter des elements graphiques en SVG (chiffres mis en avant, pictos simples vectoriels, formes). Vise un rendu moderne et soigne, adapte au sujet et au ton de l'utilisateur (pas a Lancya).

## Format et conventions (OBLIGATOIRES)
- Chaque carte est une `<section class="slide">`, enfant direct de `<body>`. Concois chaque carte pour un canevas FIXE de **1080 x 1350 pixels (format 4:5 vertical)** : dimensionne polices, marges et espacements pour cette taille. Pas de responsive a gerer, l'interface met le canevas a l'echelle.
- N'ajoute PAS de navigation toi-meme (ni boutons, ni script de slides) : l'interface s'en charge. Tu ecris juste les cartes les unes apres les autres.
- **Couleurs en variables CSS** : palette dans `:root` (`--bg`, `--text`, `--accent`, + `--muted`/`--accent-2` au besoin), utilisee partout. Permet de recolorer.
- **Un seul fichier HTML autonome**, sans librairie externe ni CDN. Polices systeme.

## Structure d'un bon carrousel
- **Carte 1 = l'accroche** : un titre fort qui donne envie de swiper (la promesse, la question, le chiffre choc).
- **Cartes du milieu** : une idee par carte, concise, percutante. Numerote si pertinent (1/5, 2/5...).
- **Derniere carte = appel a l'action** (CTA) : ce que le lecteur doit faire (commenter, suivre, contacter, partager).
- 5 a 10 cartes en general.

## Contenu — du vrai, pas des etiquettes
Remplis avec le VRAI sujet et les vraies idees de l'utilisateur. Placeholders `[entre crochets]` UNIQUEMENT pour une info que tu ne peux pas connaitre (nom, lien, chiffre precis), avec un nom explicite. La plupart du temps, zero placeholder.

## Style
Direct, vivant, sans jargon. Pas d'emojis decoratifs en exces. Respecte les regles de style generales de Lancya (pas de tiret cadratin). Ecris dans la langue de l'utilisateur.

## Livrable
Emets UNIQUEMENT le carrousel dans un bloc de code dont le langage est EXACTEMENT `lancya_carousel`, contenant le HTML complet (de `<!DOCTYPE html>` a `</html>`). RIEN avant, RIEN apres.

```lancya_carousel
<!DOCTYPE html>
<html lang="fr">
... les cartes en <section class="slide">, palette en :root ...
</html>
```
