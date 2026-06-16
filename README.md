# lancya-skills

Bibliothèque de skills pour **Lancya** (l'IA souveraine suisse, basée sur LibreChat).

Chaque skill est un dossier sous `skills/<nom-en-kebab-case>/` contenant un fichier `SKILL.md`.
LibreChat synchronise ce dépôt automatiquement (feature `skillSync.github`) : pour ajouter,
modifier ou importer une skill, il suffit de pousser ici, sans redéployer l'application.

## Convention `SKILL.md`

Frontmatter YAML en tête de fichier :

```yaml
---
name: nom-en-kebab-case          # identifiant (minuscules, chiffres, tirets). REQUIS.
description: "Quand utiliser cette skill, avec des déclencheurs explicites."
when-to-use: "Résumé court de la situation déclencheuse."
always-apply: false              # true = injectée à chaque message
---
```

Puis le corps en Markdown (les instructions de la skill).

## Style Lancya

Sortie destinée à des professionnels : claire, humaine, jamais « ton IA ».
Pas de tiret cadratin (—). Les skills restent neutres et s'adaptent au sujet de
l'utilisateur (pas de branding Lancya imposé dans leur sortie).

## Skills actuelles

- `droit-du-bail` — droit du bail suisse (CO 253 ss)
- `contract-review` — analyse de contrats (droit suisse)
- `correspondance-pro` — emails et courriers professionnels
- `synthese-documents` — résumé / analyse / comparaison de documents
- `meeting-notes` — comptes rendus de réunion
- `html-slides` — présentations HTML (reveal.js)
