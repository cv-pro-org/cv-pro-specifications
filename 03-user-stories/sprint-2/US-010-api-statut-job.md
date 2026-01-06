# US-010 — En tant qu’utilisateur, je consulte le statut de mon job de génération

## Bénéfice
Avoir un feedback et un lien de téléchargement dès que prêt.

## Points
3

## Dépendances
- US-007

## Critères d’acceptation
- Endpoint `GET /api/cv/jobs/:jobId` :
  - Auth obligatoire.
  - N’autorise l’accès qu’au propriétaire du job.
  - Retourne `{ status, error?, downloadUrl? }`.
  - Si `done`, fournit un `downloadUrl`.

## Tests (scénarios)
- User A tente de lire job user B → 404/403.
- Job `queued/running` → status correct.
- Job `done` → `downloadUrl` présent.
