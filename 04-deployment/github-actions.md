# GitHub Actions (build & deploy)

## Pipeline MVP
- Build images `app` et `worker`
- Push registry (GHCR)
- SSH vers serveur → `docker compose pull && docker compose up -d`

## Secrets
- `SSH_HOST`, `SSH_USER`, `SSH_KEY`
- `MONGODB_URI`, `NEXTAUTH_SECRET`, `RESEND_API_KEY`, `APP_DOMAIN`, `LETSENCRYPT_EMAIL`
