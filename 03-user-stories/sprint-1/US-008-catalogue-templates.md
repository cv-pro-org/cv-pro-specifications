# US-008 — En tant qu'utilisateur, je consulte le catalogue de templates pour choisir le design de mon CV

## Bénéfice
Voir les templates disponibles et choisir celui qui correspond à mon profil/secteur.

## Points
3

## Dépendances
- US-001 (landing page)

## Critères d'acceptation
- La page `/templates` affiche une liste de cards templates
- Chaque card affiche :
  - Image preview (preview.png)
  - Nom du template
  - Style (badge : Modern, Classic...)
  - Badge ATS Friendly si applicable
  - Bouton "Utiliser ce template"
- Le catalogue est responsive (grille adaptative)
- Les données viennent de `GET /api/templates`
- Cliquer "Utiliser" redirige vers `/generate?template={id}`

## Tests (scénarios)
- Ouvrir `/templates` : voir au moins 2 templates
- Cliquer "Utiliser" sur un template : URL contient `?template=modern-01`
- Mobile : cards empilées verticalement sans overflow
