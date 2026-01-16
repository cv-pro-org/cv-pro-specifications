# SkillForge — Changelog des Spécifications

## Version 2.0 — Prompt-Driven Generation (2024-01-16)

### 🎯 Changements Majeurs

#### 1. Renommage Plateforme
- **Avant** : CV Instant Pro
- **Après** : **SkillForge**
- **Raison** : Branding plus moderne et international

#### 2. Innovation UX : Génération par Prompt
- **Ancien flow** : Formulaire structuré avec champs obligatoires
- **Nouveau flow** : Textarea libre + description langage naturel
- **Inspiration** : Maquette v0 déployée sur Vercel

**Exemple ancien (formulaire)** :
```
Prénom : [_______]
Nom : [_______]
Email : [_______]
Expérience 1 :
  Entreprise : [_______]
  Poste : [_______]
  Date début : [_______]
  ...
```

**Exemple nouveau (prompt)** :
```
Textarea libre :
"Je suis développeur full-stack avec 5 ans d'expérience.
J'ai travaillé chez Google puis dans une startup fintech.
Je maîtrise React, Node.js et Python..."

[Générer mon CV]
```

#### 3. Features Ajoutées

##### CV Ciblé pour Poste
- **Toggle** : "CV pour un poste spécifique"
- **Input** : Textarea annonce LinkedIn/Indeed
- **IA** : Optimisation vocabulaire ATS + mots-clés

##### Saisie Vocale
- **API** : Web Speech Recognition
- **Langues** : Français (fr-FR)
- **UX** : Bouton micro avec animation pulse

##### Upload CV (Placeholder v1)
- **Format** : PDF, DOCX, TXT
- **v1** : Affiche nom fichier seulement
- **v2** : OCR + parsing automatique

##### Fallback Démo Offline
- **Trigger** : Timeout IA (>10s) ou erreur API
- **Logique** : Détection mots-clés + CV template pré-rempli
- **Flag** : `isDemo: true` dans réponse API

#### 4. Architecture Technique Simplifiée

**Changements stack** :

| Composant | v1.0 (Worker) | v2.0 (MVP) |
|-----------|---------------|------------|
| **Génération CV** | Async worker | Synchrone API route |
| **PDF** | Playwright + Chromium | react-pdf-renderer |
| **Queue** | BullMQ + Redis | ❌ (v2) |
| **Stockage PDF** | Volume Docker | Temporaire /tmp |
| **IA** | OpenAI direct | Vercel AI SDK |
| **Modèle** | GPT-4 | Claude Sonnet 4 |

**Raison** : Simplification pour livrer MVP en 7 jours.

#### 5. Modèle de Données Enrichi

**Ajouts** :
```typescript
interface CVData {
  personalInfo: {
    summary: string        // ✨ Nouveau : résumé IA
    linkedin?: string      // ✨ Nouveau
    website?: string       // ✨ Nouveau
  }
  experiences: {
    highlights: string[]   // ✨ Nouveau : bullet points
  }
  skills: {
    level: enum            // ✨ Nouveau : beginner → expert
    category: string       // ✨ Nouveau : Frontend, Backend...
  }
}
```

#### 6. API Routes Refactorisées

**Nouveaux endpoints** :
- `POST /api/generate-cv` — Génération prompt → CVData
- `POST /api/generate-pdf` — CVData → PDF download

**Endpoints supprimés** (v1) :
- ~~`POST /api/cv/jobs`~~ — Queue async (déplacé v2)
- ~~`GET /api/cv/jobs/:id`~~ — Polling status
- ~~`GET /internal/cv-render`~~ — HTML render pour Playwright

**Raison** : Génération synchrone suffit pour MVP.

---

## Version 1.0 — Worker Asynchrone (2024-01-10)

### Features Initiales

#### Architecture
- Next.js 14 App Router
- MongoDB Atlas M0
- Worker Python + Playwright
- Docker Compose (app + worker + Traefik)

#### Flow Utilisateur
1. User remplit formulaire structuré
2. Submit → Job créé (status: queued)
3. Worker prend job → Génère PDF
4. Email notification + download link

#### Templates CV
- Système de templates avec `rules.json`
- Contraintes sections (max expériences, skills...)
- Preview statiques

#### Quota
- 5 CV/mois par utilisateur
- Reset manuel (cron manuel)

---

## Migration Guide (v1 → v2)

### Code à Migrer

#### 1. Remplacer Formulaire par Prompt Input

**Avant** :
```tsx
<form onSubmit={handleSubmit}>
  <input name="firstName" required />
  <input name="lastName" required />
  <input name="email" type="email" />
  {/* 15+ champs... */}
</form>
```

**Après** :
```tsx
<form onSubmit={handleGenerate}>
  <textarea
    value={prompt}
    onChange={(e) => setPrompt(e.target.value)}
    placeholder="Décrivez votre parcours..."
    maxLength={2000}
  />
  <button type="submit">Générer mon CV</button>
</form>
```

#### 2. API Generate-CV

**Avant** :
```typescript
// POST /api/cv/jobs
{
  templateId: "modern",
  data: {
    firstName: "Marie",
    lastName: "Dupont",
    email: "marie@example.com",
    experiences: [...]
  }
}
```

**Après** :
```typescript
// POST /api/generate-cv
{
  prompt: "Je suis développeur full-stack...",
  jobDescription?: "Annonce LinkedIn...",
  templateId: "modern"
}
```

#### 3. Génération Synchrone

**Avant (worker async)** :
```typescript
const { jobId } = await createCVJob(data)
// Polling status...
while (status !== "done") {
  await sleep(2000)
  status = await checkJobStatus(jobId)
}
const pdfUrl = await getDownloadUrl(jobId)
```

**Après (sync)** :
```typescript
const { cv } = await fetch('/api/generate-cv', {
  method: 'POST',
  body: JSON.stringify({ prompt })
})
// Immédiat, pas de polling
```

---

## Breaking Changes

### ⚠️ API Routes

| Route | v1 | v2 | Action |
|-------|----|----|--------|
| `POST /api/cv/jobs` | ✅ | ❌ | **Supprimé** → Utiliser `POST /api/generate-cv` |
| `GET /api/cv/jobs/:id` | ✅ | ❌ | **Supprimé** (pas de polling) |
| `POST /api/generate-cv` | ❌ | ✅ | **Nouveau** endpoint principal |
| `POST /api/generate-pdf` | ❌ | ✅ | **Nouveau** export PDF |

### ⚠️ Types TypeScript

**Supprimés** :
```typescript
interface CVJobResponse { ... }  // ❌
interface CVFormData { ... }     // ❌
```

**Ajoutés** :
```typescript
interface CVGenerateRequest { ... }   // ✅
interface CVGenerateResponse { ... }  // ✅
interface CVData { ... }              // ✅ (refactorisé)
```

### ⚠️ Base de Données

**Collection `cv_jobs`** :
- v1 : Utilisée pour queue worker
- v2 : ❌ Supprimée (pas de jobs async)

**Collection `users`** :
- v1 : `{ email, quota_used, quota_limit }`
- v2 : `{ email, quota: { used, limit, resetAt }, preferences }`

---

## Nouveaux Fichiers

### Spécifications
- ✅ `02-specifications/cv-generation-prompt.md` — Détails génération prompt-driven
- ✅ `02-specifications/api-routes-v2.md` — API complète v2
- ✅ `SYNTHESIS.md` — Document de synthèse projet
- ✅ `CHANGELOG.md` — Ce fichier

### Maquette v0
- ✅ Repository séparé : `v0-cv-generation-app` (branch `feature`)
- ✅ Déploiement Vercel : `v0-cv-generation-app-git-feature-*.vercel.app`

---

## Roadmap Post-MVP

### v2.1 — Worker Async (Semaine 2)
- [ ] BullMQ + Redis
- [ ] Queue jobs génération lourde
- [ ] Email notification quand prêt
- [ ] Timeout 5 min (vs 10s sync)

### v2.2 — PDF Premium (Semaine 3)
- [ ] Playwright + Chromium
- [ ] Templates HTML/CSS avancés
- [ ] Print CSS optimisé
- [ ] Meilleure qualité que react-pdf

### v2.3 — Upload CV (Semaine 4)
- [ ] OCR Tesseract.js
- [ ] Parsing PDF/DOCX
- [ ] Amélioration/reformatage IA
- [ ] Détection format ATS

### v3.0 — Monétisation (Mois 2)
- [ ] Stripe integration
- [ ] Plans Pro/Premium
- [ ] Templates premium
- [ ] CV illimités
- [ ] Export DOCX/TXT
- [ ] Analytics avancées

---

## Notes de Migration

### Pour Développeurs

1. **Cloner nouvelle branche** :
   ```bash
   git clone -b v2-prompt-driven git@github.com:cv-pro-org/skillforge.git
   ```

2. **Installer dépendances** :
   ```bash
   npm install
   # Nouvelles : vercel ai, zod, react-pdf-renderer
   ```

3. **Variables environnement** :
   ```env
   # Nouvelles
   ANTHROPIC_API_KEY=sk-ant-...
   VERCEL_AI_SDK_PROVIDER=anthropic
   
   # Supprimées
   REDIS_URL=...
   WORKER_QUEUE_NAME=...
   ```

4. **Migration DB** :
   ```javascript
   // Script MongoDB
   db.users.updateMany({}, {
     $set: {
       "quota": {
         "used": "$quota_used",
         "limit": 5,
         "resetAt": new Date("2024-02-01")
       }
     },
     $unset: { "quota_used": "", "quota_limit": "" }
   })
   ```

5. **Tester localement** :
   ```bash
   npm run dev
   # Tester prompt generation
   # Tester PDF export
   ```

### Pour Utilisateurs

**Aucune action requise** — Migration transparente.

**Changements visibles** :
- ✅ Nouveau design (plus moderne)
- ✅ Génération plus rapide (<10s vs 30s-2min)
- ✅ Édition plus fluide (temps réel)

---

## Feedback & Support

**Questions migration** : ouvrir issue GitHub
**Bugs** : [skillforge/issues](https://github.com/cv-pro-org/skillforge/issues)
**Feature requests** : [skillforge/discussions](https://github.com/cv-pro-org/skillforge/discussions)

---

**Dernière mise à jour** : 2024-01-16  
**Version specs** : 2.0.0
