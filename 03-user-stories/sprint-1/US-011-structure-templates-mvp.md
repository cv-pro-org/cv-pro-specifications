# US-011 — En tant que développeur, je crée la structure fichiers pour 2 templates MVP

## Bénéfice
Avoir les templates prêts à être utilisés par l'API et le worker de génération.

## Points
3

## Dépendances
Aucune.

## Critères d'acceptation
- Dossier `templates/` créé à la racine du projet
- 2 templates créés : `modern-01` et `classic-01`
- Chaque template contient :
  - `meta.json` (métadonnées catalogue)
  - `rules.json` (capacités et limites)
  - `preview.png` (image 400x560px environ)
  - `template.html` (template Handlebars)
- Les fichiers JSON sont valides et conformes au schéma
- Les templates HTML utilisent les variables `CVFormData`

## Structure attendue
```
templates/
├── modern-01/
│   ├── meta.json
│   ├── rules.json
│   ├── preview.png
│   └── template.html
└── classic-01/
    ├── meta.json
    ├── rules.json
    ├── preview.png
    └── template.html
```

## Tests (scénarios)
- Tous les fichiers JSON sont parsables
- Les `id` dans `meta.json` correspondent aux noms de dossiers
- Les templates HTML contiennent `{{first_name}}`, `{{experiences}}`, etc.
