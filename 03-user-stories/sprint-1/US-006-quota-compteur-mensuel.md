# US-008 — En tant que système, je limite à 5 CV réussis par mois et par utilisateur

## Bénéfice
Maitriser les coûts et cadrer le MVP.

## Points
5

## Dépendances
- US-003

## Critères d’acceptation
- Un compteur mensuel est calculé via clé `{ userId, month(YYYY-MM) }`.
- Incrémentation : uniquement quand un job passe en `done` (succès).
- Blocage : si `count >= 5` alors création job refusée.
- Un rate-limit “1 job / minute / user” est appliqué (champ `lastGeneratedAt` ou équivalent).
- Un mécanisme simple d’override admin existe (ex: variable env `BYPASS_QUOTA_EMAILS`).

## Tests (scénarios)
- 5 jobs `done` dans le mois → le 6e `POST /api/cv/jobs` est refusé.
- Jobs `failed` ne consomment pas le quota.
- Rate-limit : 2 créations < 60s → 2ème = 429.
