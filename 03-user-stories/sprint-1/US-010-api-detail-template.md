# US-010 — En tant que développeur, j'expose un endpoint API pour récupérer le détail d'un template

## Bénéfice
Le frontend/worker peut connaître les capacités et limites d'un template spécifique.

## Points
2

## Dépendances
- US-009 (liste templates)

## Critères d'acceptation
- `GET /api/templates/{templateId}` retourne le détail complet
- Inclut les champs de `meta.json` + `supported_sections` et `limits` de `rules.json`
- Endpoint public (pas d'auth requise)
- Retourne 404 si template inexistant

## Modèle de réponse
```json
{
  "id": "modern-01",
  "name": "Modern One",
  "description": "CV moderne et épuré",
  "preview_url": "/templates/modern-01/preview.png",
  "style": "modern",
  "pages": 1,
  "ats_friendly": true,
  "supported_sections": ["summary", "experiences", "education", "skills"],
  "limits": {
    "experiences": 5,
    "skills": 10
  }
}
```

## Tests (scénarios)
- `GET /api/templates/modern-01` → status 200 + données complètes
- `GET /api/templates/unknown` → status 404
- `supported_sections` est un array de strings
- `limits` est un objet avec des valeurs numériques
