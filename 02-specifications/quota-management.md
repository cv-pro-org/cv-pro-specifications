# Quota management (5 CV / mois)

## Règle
- 5 générations *réussies* par mois et par user (month = `YYYY-MM`)

## Anti-abus MVP
- Rate-limit : 1 job / minute / user (`lastGeneratedAt`)
- Option: blocage temporaire si trop d’échecs consécutifs

## Reset
- Pas de cron en v1 (reset auto via clé `{userId, month}`)
