# US-004 — En tant que système, je limite la fréquence d’envoi des liens magiques pour éviter l’abus

## Bénéfice
Réduire le spam et les coûts email.

## Points
2

## Dépendances
- US-002

## Critères d’acceptation
- Pour un même email, il est impossible de demander un lien magique plus d’1 fois / 60 secondes.
- Le serveur renvoie `429` avec un message exploitable (“Réessayez dans X secondes”).
- Le front affiche un feedback clair.

## Tests (scénarios)
- 2 demandes consécutives < 60s → 2ème request = 429.
- Après 60s → envoi à nouveau OK.
