# US-007 — En tant qu’utilisateur, je lance une génération qui crée un job asynchrone (avec contrôle quota)

## Bénéfice
Démarrer une génération sans bloquer l’UI et garder un historique minimal.

## Points
5

## Dépendances
- US-003 (auth)
- US-008 (quota)

## Critères d’acceptation
- Endpoint `POST /api/cv/jobs` :
  - Auth obligatoire (sinon `401`).
  - Vérifie quota mensuel (5/mois) et rate-limit (1/min) avant création.
  - Crée un document `cv_jobs` avec `status=queued`, `userId`, `createdAt`.
  - Retourne `202` + `{ jobId }`.
- Si quota atteint : `403` (ou `429`) + code erreur `QUOTA_REACHED`.
- Si rate-limit : `429` + code erreur `RATE_LIMITED`.

## Tests (scénarios)
- Sans session → 401.
- Quota atteint → 403/429 + front redirige vers `/upgrade`.
- Appel OK → job `queued` en DB.
