# Architecture Decision Records (ADR) — SkillForge

## ADR-001 : Génération Prompt-Driven vs Formulaire Structuré

**Date** : 2024-01-16  
**Status** : ✅ Accepted  
**Décideurs** : Product Team

### Contexte

Deux approches pour la saisie de données CV :
1. **Formulaire structuré** (approche classique)
2. **Prompt en langage naturel** (approche IA-native)

### Décision

**Choisir génération prompt-driven** avec fallback formulaire manuel (v2).

### Raisons

**Avantages prompt** :
- ✅ Friction utilisateur réduite (1 textarea vs 15+ champs)
- ✅ Différenciation concurrentielle forte
- ✅ Expérience moderne (alignée tendances IA 2024)
- ✅ Permet ciblage poste (job description)
- ✅ Support saisie vocale naturel

**Inconvénients** :
- ❌ Dépendance API IA (coût, latence)
- ❌ Qualité variable (nécessite fallback)
- ❌ Difficile validation stricte

**Mitigation** :
- Fallback démo offline si timeout IA
- Éditeur visuel pour corrections post-génération
- Quota 5 CV/mois pour contrôler coûts

### Conséquences

- Frontend simplifié (pas de formulaire complexe)
- Backend plus complexe (prompt engineering)
- Coût IA : ~$0.05-0.10 par CV
- Nécessite monitoring qualité génération

---

## ADR-002 : Génération Synchrone vs Worker Asynchrone

**Date** : 2024-01-16  
**Status** : ✅ Accepted (MVP) → ⏸️ Async v2  
**Décideurs** : Tech Lead

### Contexte

Deux architectures pour génération PDF :
1. **Synchrone** : API route bloquante (user attend)
2. **Asynchrone** : Worker + queue + polling

### Décision MVP

**Génération synchrone** pour MVP (7 jours).

### Raisons

**Avantages sync** :
- ✅ Simple à implémenter (pas de Redis, BullMQ)
- ✅ Moins de code (pas de polling frontend)
- ✅ Pas de gestion état jobs
- ✅ Retours utilisateur immédiats

**Inconvénients** :
- ❌ Latence perçue (~8s génération)
- ❌ Timeout risque si génération lourde
- ❌ Pas de retry automatique
- ❌ Scalabilité limitée

**Pourquoi acceptable MVP** :
- Latence <10s acceptable UX
- Traffic faible attendu (phase test)
- Fallback démo <1s si timeout IA

### Plan v2 (Post-MVP)

Migrer vers architecture async :
```
User submit → Job queued → Worker process → Email notification
```

Bénéfices v2 :
- Génération PDF haute qualité (Playwright, 30s-2min)
- Retry automatique si échec
- Scalabilité horizontale workers

---

## ADR-003 : react-pdf-renderer vs Playwright PDF

**Date** : 2024-01-16  
**Status** : ✅ react-pdf-renderer (MVP) → 🔄 Playwright (v2)  
**Décideurs** : Tech Lead

### Contexte

Deux librairies génération PDF :
1. **react-pdf-renderer** : React components → PDF
2. **Playwright** : HTML/CSS → Chromium → PDF

### Décision

**react-pdf-renderer pour MVP**, migration Playwright v2.

### Comparaison

| Critère | react-pdf-renderer | Playwright |
|---------|-------------------|------------|
| **Setup** | ✅ Simple (`npm install`) | ⚠️ Complexe (Chromium deps) |
| **Syntaxe** | ✅ React JSX familier | ⚠️ HTML/CSS séparé |
| **Qualité** | ⚠️ Limitée (pas web fonts) | ✅ Excellente (print CSS) |
| **Taille bundle** | ✅ Léger (~5 MB) | ❌ Lourd (~200 MB Chromium) |
| **Perf** | ✅ Rapide (~2s) | ⚠️ Lent (~10-30s) |
| **Flexibilité** | ⚠️ API limitée | ✅ CSS complet |
| **ARM compatibility** | ✅ Natif | ⚠️ Nécessite build spécial |

### Décision Rationale

**MVP** :
- react-pdf-renderer suffit pour templates basiques
- Setup rapide (contrainte 7 jours)
- Pas de souci deps ARM

**v2 (Post-MVP)** :
- Playwright pour templates premium
- Meilleure qualité justifie complexité
- Worker async absorbe latence

### Code Sample

**react-pdf-renderer (MVP)** :
```tsx
import { Document, Page, Text, View } from '@react-pdf/renderer'

const CVDocument = ({ data }: { data: CVData }) => (
  <Document>
    <Page size="A4">
      <View style={styles.header}>
        <Text>{data.personalInfo.fullName}</Text>
      </View>
    </Page>
  </Document>
)

const pdf = await renderToStream(<CVDocument data={cvData} />)
```

**Playwright (v2)** :
```typescript
const browser = await chromium.launch()
const page = await browser.newPage()
await page.goto(`http://localhost:3000/cv-render/${cvId}`)
const pdf = await page.pdf({
  format: 'A4',
  printBackground: true,
  margin: { top: '0', bottom: '0' }
})
```

---

## ADR-004 : Claude Sonnet 4 vs GPT-4

**Date** : 2024-01-16  
**Status** : ✅ Accepted  
**Décideurs** : Tech Lead

### Contexte

Choisir modèle LLM pour génération CV structuré.

### Décision

**Anthropic Claude Sonnet 4** via Vercel AI SDK.

### Comparaison

| Critère | Claude Sonnet 4 | GPT-4 Turbo |
|---------|----------------|-------------|
| **Structured output** | ✅ Natif | ⚠️ Nécessite functions |
| **Latence** | ✅ ~3-5s | ⚠️ ~5-8s |
| **Coût** | ✅ $3/1M tokens | ❌ $10/1M tokens |
| **Context window** | ✅ 200k tokens | ✅ 128k tokens |
| **Qualité FR** | ✅ Excellent | ✅ Excellent |
| **Rate limits** | ✅ 5000 RPM | ⚠️ 500 RPM (tier 1) |

### Raisons

1. **Coût** : Claude 3x moins cher (~$0.03 vs $0.10 par CV)
2. **Perf** : Latence 40% inférieure
3. **DX** : Vercel AI SDK `generateObject()` plus simple
4. **Qualité** : Tests internes montrent résultats comparables

### Code

```typescript
import { generateObject } from 'ai'
import { anthropic } from '@ai-sdk/anthropic'

const { object: cv } = await generateObject({
  model: anthropic('claude-sonnet-4-20250514'),
  schema: cvSchema,
  prompt: systemPrompt,
  maxOutputTokens: 4000
})
```

### Fallback

Si Claude indisponible → GPT-4 Turbo (provider failover).

---

## ADR-005 : MongoDB Atlas vs PostgreSQL

**Date** : 2024-01-16  
**Status** : ✅ Accepted  
**Décideurs** : Tech Lead

### Contexte

Choisir base de données pour users + quotas.

### Décision

**MongoDB Atlas M0** (gratuit).

### Raisons

**Avantages MongoDB** :
- ✅ Tier gratuit généreux (512 MB, suffisant MVP)
- ✅ Schema flexible (CVData JSON natif)
- ✅ Setup rapide (Atlas cloud, pas de self-hosting)
- ✅ Adapté documents non-relationnels
- ✅ Expérience équipe (Next.js + Mongo classique)

**Inconvénients** :
- ❌ Pas de transactions ACID strictes (pas critique MVP)
- ❌ Requêtes complexes moins performantes (pas besoin MVP)

**Alternative PostgreSQL** :
- ✅ Relations strictes
- ✅ JSONB pour données CV
- ❌ Self-hosting nécessaire (overhead infra)
- ❌ Setup plus long

### Décision

MongoDB suffit pour MVP. Réévaluation si besoin analytics complexes (v2).

### Schema

```javascript
// Collection users
{
  _id: ObjectId("..."),
  email: "user@example.com",
  quota: {
    used: 3,
    limit: 5,
    resetAt: ISODate("2024-02-01T00:00:00Z")
  }
}

// Collection cvs (v2)
{
  _id: ObjectId("..."),
  userId: ObjectId("..."),
  data: { /* CVData nested */ },
  createdAt: ISODate("...")
}
```

---

## ADR-006 : Docker Compose vs Kubernetes

**Date** : 2024-01-16  
**Status** : ✅ Accepted (MVP) → 🔄 K3s (v2)  
**Décideurs** : DevOps

### Contexte

Orchestration containers sur Oracle Linux ARM bare metal.

### Décision

**Docker Compose** pour MVP.

### Raisons

**Avantages Docker Compose** :
- ✅ Simple setup (1 fichier YAML)
- ✅ Adapté single-server
- ✅ Traefik reverse proxy natif
- ✅ Pas d'overhead orchestration
- ✅ Monitoring simple (logs stdout)

**Inconvénients** :
- ❌ Pas de HA (high availability)
- ❌ Scale horizontal manuel
- ❌ Pas de rolling updates

**Pourquoi acceptable MVP** :
- Traffic faible attendu
- Single point failure acceptable (phase test)
- Redémarrage rapide si crash

### docker-compose.yml

```yaml
version: '3.8'
services:
  app:
    image: skillforge-app:latest
    environment:
      - DATABASE_URL=${MONGODB_URI}
      - NEXTAUTH_SECRET=${NEXTAUTH_SECRET}
    labels:
      - traefik.enable=true
      - traefik.http.routers.app.rule=Host(`skillforge.app`)
      - traefik.http.routers.app.tls.certresolver=letsencrypt
  
  traefik:
    image: traefik:v2.10
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
      - ./traefik.yml:/etc/traefik/traefik.yml
```

### Plan v2 (Post-MVP)

Migrer vers **K3s** (Kubernetes lightweight) si :
- Traffic >1000 users/jour
- Besoin HA (99.9% uptime)
- Multi-services (workers, cache, etc.)

---

## ADR-007 : Magic Link vs Password Authentication

**Date** : 2024-01-16  
**Status** : ✅ Accepted  
**Décideurs** : Product Team

### Contexte

Méthode authentification utilisateur.

### Décision

**Magic Link Email** (passwordless).

### Raisons

**Avantages** :
- ✅ UX frictionless (pas de mot de passe)
- ✅ Sécurité (pas de leaks password DB)
- ✅ Onboarding rapide (email suffit)
- ✅ NextAuth support natif

**Inconvénients** :
- ❌ Dépend email delivery (Resend)
- ❌ Pas de login offline
- ❌ Users sans email bloqués

**Mitigation** :
- Resend 99.9% uptime SLA
- Fallback Google OAuth (v2)

### Flow

```
1. User saisie email
   ↓
2. Backend génère token JWT + envoie email
   ↓
3. User clique lien email
   ↓
4. Token vérifié → Session créée
   ↓
5. Redirection dashboard
```

### Code

```typescript
// next-auth.config.ts
import EmailProvider from 'next-auth/providers/email'

providers: [
  EmailProvider({
    server: {
      host: process.env.EMAIL_SERVER_HOST,
      port: process.env.EMAIL_SERVER_PORT,
      auth: {
        user: process.env.EMAIL_SERVER_USER,
        pass: process.env.EMAIL_SERVER_PASSWORD,
      },
    },
    from: process.env.EMAIL_FROM,
  }),
]
```

---

## ADR-008 : GitHub Actions vs GitLab CI

**Date** : 2024-01-16  
**Status** : ✅ Accepted  
**Décideurs** : DevOps

### Contexte

CI/CD pipeline pour build + deploy.

### Décision

**GitHub Actions**.

### Raisons

**Avantages** :
- ✅ Intégration native GitHub (repo déjà hébergé)
- ✅ Runners gratuits (2000 min/mois)
- ✅ Marketplace actions riches
- ✅ Secrets management intégré
- ✅ ARM64 runners disponibles

**GitLab CI** :
- ✅ Pipeline YAML puissant
- ❌ Nécessite migration repo GitHub → GitLab
- ❌ Runners self-hosted (overhead)

### Workflow

```yaml
# .github/workflows/deploy.yml
name: Deploy to Production

on:
  push:
    branches: [main]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: docker/setup-buildx-action@v3
      - run: docker build --platform linux/arm64 -t skillforge:latest .
      - run: docker save skillforge:latest | gzip > app.tar.gz
  
  deploy:
    needs: build
    runs-on: ubuntu-latest
    steps:
      - uses: actions/download-artifact@v3
      - run: |
          scp app.tar.gz ${{ secrets.SERVER_HOST }}:/tmp/
          ssh ${{ secrets.SERVER_HOST }} '
            docker load < /tmp/app.tar.gz
            docker-compose up -d
          '
```

---

## Résumé Décisions

| ADR | Décision | Status | Raison Principale |
|-----|----------|--------|-------------------|
| 001 | Prompt-driven | ✅ | UX moderne, différenciation |
| 002 | Sync generation | ✅ MVP | Simplicité (async v2) |
| 003 | react-pdf-renderer | ✅ MVP | Setup rapide (Playwright v2) |
| 004 | Claude Sonnet 4 | ✅ | Coût 3x inférieur GPT-4 |
| 005 | MongoDB Atlas | ✅ | Gratuit + flexible |
| 006 | Docker Compose | ✅ MVP | Simple (K3s v2) |
| 007 | Magic Link | ✅ | UX passwordless |
| 008 | GitHub Actions | ✅ | Native GitHub |

---

**Dernière mise à jour** : 2024-01-16  
**Version** : 2.0.0
