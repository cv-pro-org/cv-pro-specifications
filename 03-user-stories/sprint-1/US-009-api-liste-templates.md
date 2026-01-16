# US-009 — En tant que développeur, j'expose un endpoint API pour lister les templates disponibles

## Bénéfice
Le frontend peut récupérer dynamiquement la liste des templates sans hardcoder.

## Points
2

## Dépendances
Aucune.

## Critères d'acceptation
- `GET /api/templates` retourne un tableau JSON
- Chaque élément contient : `id`, `name`, `description`, `preview_url`, `style`, `pages`, `ats_friendly`
- Les données sont lues depuis les fichiers `meta.json` du dossier `templates/`
- Endpoint public (pas d'auth requise)
- Response status 200

## Modèle de réponse
```json
[
  {
    "id": "modern-01",
    "name": "Modern One",
    "description": "CV moderne et épuré",
    "preview_url": "/templates/modern-01/preview.png",
    "style": "modern",
    "pages": 1,
    "ats_friendly": true
  }
]
```

## Tests (scénarios)
- `GET /api/templates` → status 200
- Response est un array non vide
- Chaque objet a les champs requis
- `preview_url` pointe vers un fichier existant
