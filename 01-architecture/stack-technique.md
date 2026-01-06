# Stack technique (MVP)

## Frontend/Backend
- **Next.js 14** (App Router)
- API Routes (Route Handlers) pour : auth callbacks, génération, quota, téléchargement
- **Tailwind CSS** pour UI + **print CSS** pour le template CV (HTML)

## Auth
- **NextAuth.js** (Email Provider) + **Resend** (magic link)
- Session côté serveur (cookies) + protection endpoints (auth obligatoire)

## PDF (choix MVP)
### Option retenue : Playwright (Chromium) = rendu pixel-perfect
- Le CV est d’abord rendu en **HTML** (même code de style que le site)
- Un **worker** lance Playwright, charge une route interne (template HTML), puis `page.pdf()`
- Avantages : typographie, gestion sauts de page, cohérence rendu, rapidité d’itération design

### Pourquoi pas react-pdf-renderer en MVP "qualité premium"
- Limites layout/typo et rendu parfois difficile à stabiliser

## Asynchronisme (génération en job)
- L’API crée un document `cv_jobs` en Mongo
- Un process **worker** (container séparé) consomme les jobs (polling simple en MVP)
- À la fin : stockage PDF + update job + email notification (Resend)

## Base de données
- **MongoDB Atlas M0**
- Stockage minimal (MVP) : user + quota + jobs + (optionnel) waitlist

## Infra & déploiement
- Oracle Linux ARM bare metal
- **Docker Compose**
- **Traefik** (SSL auto)
- **GitHub Actions** build + deploy via SSH

## Flux de données (haut niveau)
```mermaid
sequenceDiagram
  participant U as Utilisateur
  participant W as Web (Next.js)
  participant DB as MongoDB
  participant Q as Worker (Playwright)
  participant E as Resend

  U->>W: Submit formulaire génération
  W->>DB: create cv_job (status=queued)
  W-->>U: 202 Accepted + jobId
  Q->>DB: poll job queued
  Q->>W: GET /internal/cv-render?jobId=... (auth token)
  Q->>Q: page.pdf() => bytes
  Q->>W: POST /internal/cv-store (pdf)
  W->>DB: update job (status=done, pdfPath/url)
  Q->>E: send email "PDF prêt" + link
  U->>W: download PDF
```
