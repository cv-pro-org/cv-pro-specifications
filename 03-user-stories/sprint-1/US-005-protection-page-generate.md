# US-005 — En tant qu’utilisateur non connecté, je suis redirigé vers /login si j’accède à /generate

## Bénéfice
Assurer que la génération est réservée aux comptes.

## Points
2

## Dépendances
- US-003

## Critères d’acceptation
- `/generate` est protégée : sans session, redirection vers `/login`.
- Après login, l’utilisateur revient sur `/generate`.

## Tests (scénarios)
- Ouvrir `/generate` sans session → redirection `/login`.
- Se connecter → retour `/generate`.
