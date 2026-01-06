# US-011 — En tant que système, un worker consomme les jobs queued de manière fiable (MVP)

## Bénéfice
Exécuter la génération sans bloquer le serveur web.

## Points
5

## Dépendances
- US-007

## Critères d’acceptation
- Un process `worker` tourne séparément (docker service).
- Le worker récupère des jobs `queued` et les passe à `running` avec un mécanisme anti double-consommation.
  - MVP acceptable : `findOneAndUpdate` atomique sur `{status:'queued'}`.
- Le worker pose `startedAt` et, en fin, `finishedAt`.
- En cas de crash worker :
  - les jobs `running` depuis trop longtemps peuvent être remis `queued` (ou marqués `failed`) via une règle simple.

## Tests (scénarios)
- Lancer 2 workers (test) → un job ne doit être pris qu’une seule fois.
- Simuler exception → job passe `failed` avec message court.
