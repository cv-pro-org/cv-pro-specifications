# US-018 — En tant qu’opérateur, je dispose d’un healthcheck et de logs exploitables

## Bénéfice
Diagnostiquer rapidement les erreurs en prod.

## Points
2

## Dépendances
Aucune.

## Critères d’acceptation
- Endpoint `GET /api/health` retourne 200 + infos minimales (ex: status, timestamp).
- Logs structurés (au moins `info/error`) sur :
  - création job
  - quota refusé
  - début/fin génération
  - erreurs OpenAI/Playwright
- Aucun secret (tokens) ne fuit dans les logs.

## Tests (scénarios)
- Appeler `/api/health` → 200.
- Provoquer une erreur génération → log `error` lisible.
