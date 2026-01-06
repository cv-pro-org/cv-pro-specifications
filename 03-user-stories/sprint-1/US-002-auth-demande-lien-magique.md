# US-002 — En tant qu’utilisateur, je demande un lien magique par email pour me connecter

## Bénéfice
Accéder au générateur sans mot de passe.

## Points
3

## Dépendances
- US-004 (Resend configuré) si vous externalisez l’email provider.

## Critères d’acceptation
- Une page `/login` existe avec un champ email + bouton “Envoyer le lien”.
- Lors d’une demande valide :
  - l’utilisateur voit un écran “Vérifiez votre boîte mail”.
  - l’email est envoyé via Resend (ou provider NextAuth Email).
- Si email invalide : message d’erreur clair (validation front + back).
- Aucune information sensible n’est révélée (ex: “email inconnu” ne doit pas être distingué si vous le souhaitez).

## Tests (scénarios)
- Entrer un email valide → message “check inbox” + email reçu.
- Entrer un email invalide → erreur de validation.
