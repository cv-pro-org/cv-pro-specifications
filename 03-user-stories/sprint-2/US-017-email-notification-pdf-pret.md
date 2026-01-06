# US-017 — En tant qu’utilisateur, je reçois un email quand mon PDF est disponible

## Bénéfice
Je peux quitter la page et revenir plus tard.

## Points
3

## Dépendances
- US-015
- US-004 (Resend)

## Critères d’acceptation
- Quand un job passe `done`, le système envoie un email via Resend à l’adresse du user.
- L’email contient :
  - un bouton/lien vers la page job (ex: `/jobs/:id`) ou direct download.
  - un rappel de sécurité (ne pas partager le lien si lien non-auth).
- En cas d’échec d’envoi email :
  - le job reste `done`.
  - l’erreur email est loggée.

## Tests (scénarios)
- Génération réussie → email reçu.
- Resend down → job done, log d’erreur.
