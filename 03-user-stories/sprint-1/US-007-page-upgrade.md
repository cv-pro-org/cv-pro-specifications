# US-009 — En tant qu’utilisateur, je vois une page “upgrade” quand mon quota est atteint

## Bénéfice
Comprendre pourquoi je ne peux plus générer et garder un chemin de conversion.

## Points
1

## Dépendances
- US-008

## Critères d’acceptation
- Une page `/upgrade` existe.
- Elle explique : quota atteint + retour date (mois en cours) + CTA (ex: “Revenir le mois prochain” ou “Rejoindre la liste d’attente premium”).
- Le front redirige vers `/upgrade` lors de l’erreur `QUOTA_REACHED`.

## Tests (scénarios)
- Simuler quota atteint → redirection vers `/upgrade`.
