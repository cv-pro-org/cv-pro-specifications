# Auth flow (NextAuth email magique)

## Parcours utilisateur
1. L’utilisateur saisit son email
2. Réception d’un lien magique (Resend)
3. Clic → session créée → redirection vers /generate

## Exigences MVP
- Page "check your inbox"
- Gestion : lien expiré / token invalide
- Anti-spam : pas plus d’un envoi toutes les 60s par email

## Sécurité
- `NEXTAUTH_SECRET` obligatoire
- Cookies secure en prod (HTTPS)
