# US-016 — En tant qu’utilisateur, je télécharge mon PDF via un endpoint sécurisé

## Bénéfice
Récupérer instantanément le PDF une fois prêt.

## Points
3

## Dépendances
- US-015

## Critères d’acceptation
- Endpoint `GET /api/cv/jobs/:jobId/download` :
  - Auth obligatoire.
  - Vérifie owner.
  - Si job `done`, stream le PDF (`Content-Type: application/pdf`).
  - Si job pas prêt : `409` avec message “Pas encore prêt”.

## Tests (scénarios)
- User A download job B → 404/403.
- Job queued → 409.
- Job done → download OK.
