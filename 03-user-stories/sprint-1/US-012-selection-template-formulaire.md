# US-012 — En tant qu'utilisateur, je sélectionne un template avant de remplir le formulaire CV

## Bénéfice
Mon CV sera généré avec le design que j'ai choisi.

## Points
2

## Dépendances
- US-008 (catalogue templates)
- US-004 (formulaire CV)

## Critères d'acceptation
- Le formulaire `/generate` lit le paramètre `?template={id}` de l'URL
- Si template valide : affiché en haut du formulaire (nom + mini preview)
- Si template absent/invalide : redirection vers `/templates`
- Le `template_id` est inclus dans le payload `POST /api/cv/jobs`
- L'utilisateur peut changer de template via un lien "Changer de template"

## Tests (scénarios)
- `/generate?template=modern-01` : formulaire affiché avec "Modern One" visible
- `/generate` sans param : redirection vers `/templates`
- `/generate?template=invalid` : redirection vers `/templates`
- Submit formulaire : payload contient `template_id: "modern-01"`
