# Auth flow (NextAuth — Google OAuth first)

## Décision MVP
**Google OAuth en priorité** pour maximiser le taux de conversion (~90%) et réduire la friction mobile.
Magic Link reporté en v2 (fallback pour users sans Google).

## Parcours utilisateur
1. L'utilisateur clique "Se connecter avec Google"
2. Popup/redirect Google OAuth
3. Consentement → callback NextAuth
4. Session créée → redirection vers `/generate`

## Providers NextAuth (MVP)
```typescript
providers: [
  Google({
    clientId: process.env.GOOGLE_CLIENT_ID,
    clientSecret: process.env.GOOGLE_CLIENT_SECRET,
  }),
]
```

## Exigences MVP
- Bouton "Se connecter avec Google" sur `/login`
- Gestion erreur OAuth (refus, annulation)
- Création user en DB au premier login (upsert sur email)
- Session cookie sécurisée (HTTPS only en prod)

## Sécurité
- `NEXTAUTH_SECRET` obligatoire
- Cookies `secure: true` en prod
- Callback URL whitelistée dans Google Console

## Données récupérées
- `email` (identifiant unique)
- `name` (optionnel, pré-remplir le form)
- `image` (optionnel, affichage header)

## Roadmap auth
- MVP : Google OAuth uniquement
- v2 : Magic Link (Resend) en option
- v3 : LinkedIn OAuth (pertinent pour CV)
