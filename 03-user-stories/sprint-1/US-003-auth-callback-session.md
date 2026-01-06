# US-003 — En tant qu’utilisateur, je clique le lien magique et j’obtiens une session valide

## Bénéfice
Accéder immédiatement au produit après clic.

## Points
3

## Dépendances
- US-002

## Critères d’acceptation
- Le clic sur le lien magique crée une session NextAuth.
- Après login, redirection vers `/generate`.
- Cas d’erreurs gérés :
  - token expiré → page d’erreur + CTA “Renvoyer un lien”.
  - token invalide → page d’erreur + CTA.
- La session permet d’appeler les routes API protégées.

## Tests (scénarios)
- Cliquer un lien valide → session active, accès à `/generate`.
- Utiliser un lien expiré → erreur + possibilité de renvoyer.
