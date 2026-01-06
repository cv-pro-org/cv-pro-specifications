# US-002 — En tant qu'utilisateur, je me connecte via Google OAuth pour accéder rapidement au générateur

## Bénéfice
Connexion en 2 clics, sans friction email, excellente UX mobile.

## Points
2

## Dépendances
Aucune.

## Critères d'acceptation
- Une page `/login` existe avec un bouton "Se connecter avec Google".
- Au clic :
  - Redirect vers Google OAuth consent screen.
  - Après consentement, callback NextAuth crée/update le user en DB.
  - Session cookie créée.
  - Redirection vers `/generate`.
- En cas de refus/annulation Google :
  - Retour sur `/login` avec message d'erreur clair.
- Les données Google récupérées : `email` (obligatoire), `name`, `image` (optionnels).

## Détails techniques
```typescript
// app/api/auth/[...nextauth]/route.ts
import NextAuth from "next-auth"
import Google from "next-auth/providers/google"
import { MongoDBAdapter } from "@auth/mongodb-adapter"

export const { handlers, signIn, signOut, auth } = NextAuth({
  adapter: MongoDBAdapter(clientPromise),
  providers: [
    Google({
      clientId: process.env.GOOGLE_CLIENT_ID!,
      clientSecret: process.env.GOOGLE_CLIENT_SECRET!,
    }),
  ],
  pages: {
    signIn: "/login",
    error: "/login/error",
  },
})
```

## Configuration requise (Google Console)
1. Créer projet dans Google Cloud Console
2. Activer "Google+ API" (ou "Google Identity")
3. Créer OAuth 2.0 Client ID (type: Web application)
4. Authorized redirect URIs : `https://yourdomain.com/api/auth/callback/google`
5. Récupérer `GOOGLE_CLIENT_ID` et `GOOGLE_CLIENT_SECRET`

## Tests (scénarios)
- Clic "Connexion Google" → popup/redirect Google → consentement → session active → `/generate`.
- Annuler sur Google → retour `/login` avec message.
- User existant → session créée, pas de doublon en DB.
- User nouveau → création en DB avec email/name/image.
