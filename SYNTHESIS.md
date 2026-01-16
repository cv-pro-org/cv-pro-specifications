# SkillForge MVP — Document de Synthèse

## 🎯 Vision Produit

**SkillForge** (anciennement CV Instant Pro) est une plateforme de génération de CV professionnels alimentée par l'IA, avec une **innovation majeure** : la création de CV par **description en langage naturel** (prompt-driven) au lieu de formulaires structurés.

### Proposition de Valeur
> "Décrivez votre parcours en quelques phrases, SkillForge crée un CV professionnel optimisé pour les recruteurs en 2 minutes."

---

## 🚀 Innovation : Prompt-Driven CV Generation

### Rupture UX

**Avant (approche classique)** :
```
User remplit formulaire structuré :
├─ Champ "Prénom" ___________
├─ Champ "Nom" ______________
├─ Champ "Email" ____________
├─ Expérience 1
│  ├─ Entreprise ____________
│  ├─ Poste _________________
│  ├─ Date début ____________
│  └─ Description ___________
└─ ... (10+ sections)
```

**Après (SkillForge)** :
```
User écrit librement :

┌─────────────────────────────────────────┐
│ Décrivez votre parcours...              │
│                                         │
│ "Je suis développeur full-stack avec   │
│  5 ans d'expérience. J'ai travaillé     │
│  chez Google puis dans une startup      │
│  fintech. Je maîtrise React, Node.js    │
│  et Python..."                          │
│                                         │
└─────────────────────────────────────────┘

☐ CV pour un poste spécifique
  ↳ [Coller annonce LinkedIn...]

[Générer mon CV →]
```

L'IA structure automatiquement :
- ✅ Extraction informations personnelles
- ✅ Détection expériences professionnelles
- ✅ Identification compétences techniques/soft
- ✅ Génération résumé professionnel
- ✅ Formatage dates en français
- ✅ Organisation hiérarchique

---

## 🏗️ Architecture Technique MVP

### Stack Simplifiée (7 jours)

```
Frontend + Backend
├─ Next.js 14 (App Router)
│  ├─ React Server Components
│  ├─ API Routes
│  └─ Tailwind CSS
│
├─ Authentification
│  └─ NextAuth.js (Magic Link Email)
│
├─ Base de données
│  └─ MongoDB Atlas M0 (gratuit)
│     ├─ Collection `users` (session, quota)
│     └─ Collection `cvs` (historique v2)
│
├─ IA Generation
│  ├─ Vercel AI SDK
│  ├─ Anthropic Claude Sonnet 4
│  └─ Fallback démo offline
│
├─ PDF Generation
│  └─ react-pdf-renderer (MVP)
│     └─ Playwright (v2)
│
└─ Email
   └─ Resend API
```

### Déploiement

```
GitHub Actions CI/CD
    ↓
Docker Build (ARM64)
    ↓
Oracle Linux ARM Bare Metal
    ├─ Docker Compose
    │  ├─ App Container (Next.js)
    │  └─ Traefik (Reverse Proxy + SSL)
    │
    └─ Volumes
       └─ /tmp/cvs (PDF temporaires)
```

---

## 📋 Features Core MVP (Must-Have)

### 1. Authentification
- ✅ **Magic Link Email** (NextAuth + Resend)
- ✅ Landing page publique accessible sans login
- ✅ Dashboard protégé après login

### 2. Génération Prompt-Driven
- ✅ **Textarea libre** : 2000 caractères max
- ✅ **Option "CV ciblé"** : checkbox + textarea annonce (3000 chars)
- ✅ **Upload CV** : bouton (parsing v2, placeholder v1)
- ✅ **Saisie vocale** : Web Speech API (navigateurs compatibles)
- ✅ **Génération IA** : Claude Sonnet 4 via Vercel AI SDK
- ✅ **Fallback démo** : génération offline si timeout IA

### 3. Éditeur Visuel
- ✅ **Preview temps réel** : CV affiché en direct
- ✅ **Édition inline** : tous champs modifiables
- ✅ **Templates** : 4 designs (Modern, Classic, Minimal, Creative)
- ✅ **Couleurs** : 4 palettes (Slate, Blue, Purple, Green)
- ✅ **Polices** : 4 typographies (Inter, Roboto, Open Sans, Playfair)
- ✅ **Sections CRUD** : ajout/suppression expériences, skills, etc.

### 4. Export PDF
- ✅ **Génération synchrone** : react-pdf-renderer
- ✅ **Download direct** : bouton téléchargement
- ✅ **Qualité professionnelle** : A4, haute résolution

### 5. Quota Utilisateur
- ✅ **5 CV gratuits/mois** par utilisateur
- ✅ **Compteur visible** : "3/5 CV restants"
- ✅ **Reset automatique** : 1er du mois (cron manuel v1)
- ✅ **Page upgrade** : affichage limite atteinte (sans paiement)

### 6. Pages Essentielles
- ✅ **Landing page** : présentation + CTA "Créer mon CV"
- ✅ **Login page** : email magique
- ✅ **Dashboard** : historique récent + nouveau CV
- ✅ **Editor** : interface génération + personnalisation
- ✅ **Pricing** : page upgrade (statique v1)

---

## ❌ Features Post-MVP (v2)

### Différé à v2 (après 7 jours)
- ⏸️ Dashboard admin (requêtes Mongo manuelles v1)
- ⏸️ Historique CV complet + CRUD
- ⏸️ Worker asynchrone (BullMQ + Redis)
- ⏸️ PDF via Playwright (meilleure qualité)
- ⏸️ Upload CV + OCR (parsing automatique)
- ⏸️ Cron reset quota automatique
- ⏸️ Analytics avancées (Mixpanel/Posthog)
- ⏸️ A/B testing prompts
- ⏸️ Système de paiement (Stripe)
- ⏸️ Templates premium
- ⏸️ Export DOCX
- ⏸️ Partage CV (lien public)

---

## 🎨 Design System (Maquette v0)

### Composants Clés

**1. Landing Page**
```
Hero Section
├─ Logo "SkillForge" (icône Anvil)
├─ Tagline : "Créez votre CV en 2 minutes"
├─ Input prompt (large, responsive)
├─ Checkbox "CV ciblé" (collapsible)
└─ CTA "Générer mon CV"

Navigation
├─ Pricing
├─ Blog
└─ Se connecter
```

**2. Dashboard**
```
Sidebar (collapsible)
├─ Logo + Nouveau CV
├─ Historique (5 récents)
├─ Bibliothèque templates
└─ Profil + Settings

Main Area
└─ Prompt input + génération
```

**3. Editor**
```
3-Column Layout
├─ Left Sidebar
│  ├─ Templates gallery
│  ├─ Color schemes
│  └─ Font picker
│
├─ Center Preview
│  └─ CV render (scalable)
│
└─ Right Sidebar
   ├─ Section tabs
   │  ├─ Info perso
   │  ├─ Expérience
   │  ├─ Formation
   │  ├─ Skills
   │  └─ Langues
   └─ Edit forms
```

**4. Thème**
- ☀️ Mode clair / 🌙 Mode sombre
- 🎨 Design system shadcn/ui
- 🎭 Animations Framer Motion
- 📱 Responsive (mobile-first)

---

## 🧪 Parcours Utilisateur Type

### Scenario : Premier CV

```
1. Landing Page
   User visite skillforge.app
   ↓
   Lit tagline + exemples prompts
   ↓
   Écrit dans textarea : "Je suis développeur..."
   ↓
   Clique "Générer mon CV"
   ↓
   → Redirection /login si non connecté

2. Login
   Saisie email
   ↓
   Reçoit magic link
   ↓
   Clique lien → Authentifié
   ↓
   → Redirection /dashboard

3. Dashboard
   Prompt pré-rempli (depuis landing)
   ↓
   Clique "Générer"
   ↓
   ⏳ Loading (2-8s)
   ↓
   → Redirection /editor

4. Editor
   CV généré affiché en preview
   ↓
   User édite :
   - Change template → "Classic"
   - Modifie titre → "Senior Full-Stack"
   - Ajout compétence → "Docker"
   ↓
   Clique "Télécharger PDF"
   ↓
   ⏳ Génération PDF (2-3s)
   ↓
   📄 Download "alexandre-martin-cv.pdf"
   ↓
   ✅ Quota : 4/5 restants
```

---

## 🔍 Prompt Engineering Strategy

### Approche MVP

**Objectif** : Générer CV cohérent et professionnel à partir de prompt minimal.

**Techniques** :
1. **System Prompt structuré** : Instructions claires pour l'IA
2. **Few-shot examples** : Exemples de CV générés (v2)
3. **Schéma Zod strict** : Force structure de sortie
4. **Validation post-génération** : Vérification cohérence

**Prompts Testés** :
```typescript
// Base
"Tu es un expert en rédaction de CV professionnels français..."

// Job Targeting
"Optimise ce CV pour le poste décrit. Utilise vocabulaire ATS..."

// Fallback Demo
"Génère un CV plausible basé sur mots-clés détectés..."
```

**Métriques** :
- Latence : <10s (IA) / <1s (démo)
- Taux succès : >95%
- Cohérence : validation manuelle 5 cas tests

---

## 📊 Base de Données (MongoDB)

### Collection `users`
```javascript
{
  _id: ObjectId("..."),
  email: "user@example.com",
  name: "Marie Dupont",
  createdAt: ISODate("2024-01-15T10:00:00Z"),
  quota: {
    used: 3,              // CV générés ce mois
    limit: 5,             // Plan gratuit
    resetAt: ISODate("2024-02-01T00:00:00Z")
  },
  preferences: {
    defaultTemplateId: "modern",
    defaultColorScheme: "slate"
  }
}
```

### Collection `cvs` (v2)
```javascript
{
  _id: ObjectId("..."),
  userId: ObjectId("..."),
  title: "CV Développeur Full-Stack",
  data: { /* CVData JSON */ },
  templateId: "modern",
  colorScheme: "slate",
  createdAt: ISODate("..."),
  updatedAt: ISODate("..."),
  pdfUrl: "/tmp/cvs/user123_cv456.pdf"  // Temporaire
}
```

---

## 🚦 État d'Avancement

### ✅ Complété (Maquette v0)
- [x] Design system complet
- [x] Landing page responsive
- [x] Dashboard avec sidebar
- [x] Éditeur 3 colonnes
- [x] Génération CV prompt-driven (démo)
- [x] Templates multiples
- [x] Customisation couleurs/polices
- [x] Mode clair/sombre

### 🚧 En Cours (Production MVP)
- [ ] NextAuth integration
- [ ] MongoDB setup
- [ ] API routes implémentation
- [ ] Vercel AI SDK integration
- [ ] react-pdf-renderer templates
- [ ] Quota management
- [ ] Docker deployment

### 📋 À Faire (7 jours)
- [ ] Tests end-to-end
- [ ] CI/CD GitHub Actions
- [ ] Monitoring (logs, errors)
- [ ] Documentation déploiement
- [ ] User testing (5 personnes)

---

## 🎯 Succès Metrics (MVP)

### KPIs Semaine 1
- ⚡ **Uptime** : >99%
- 🚀 **Latence génération** : <10s (p95)
- 👥 **Test users** : 5 personnes
- ✅ **Taux succès génération** : >95%
- 📄 **Qualité PDF** : ⭐⭐⭐⭐ (4/5 satisfaction)

### Critères Validation MVP
- ✅ User peut générer CV en <3 min (premier usage)
- ✅ PDF téléchargeable haute qualité
- ✅ Quota système fonctionnel
- ✅ Pas de crash en production
- ✅ Déploiement automatisé opérationnel

---

## 📚 Documentation Liée

### Spécifications Techniques
- [`cv-generation-prompt.md`](./02-specifications/cv-generation-prompt.md) — Détails génération IA
- [`api-routes-v2.md`](./02-specifications/api-routes-v2.md) — Endpoints API complets
- [`auth-flow.md`](./02-specifications/auth-flow.md) — Flux authentification
- [`quota-management.md`](./02-specifications/quota-management.md) — Gestion limites utilisateur

### Architecture
- [`01-architecture/stack-technique.md`](./01-architecture/stack-technique.md) — Stack détaillée
- [`01-architecture/schema-mongodb.md`](./01-architecture/schema-mongodb.md) — Modèles données
- [`01-architecture/docker-compose.yml`](./01-architecture/docker-compose.yml) — Configuration déploiement

### User Stories
- [`03-user-stories/sprint-1-auth.md`](./03-user-stories/sprint-1-auth.md) — Stories authentification
- [`03-user-stories/sprint-2-generation.md`](./03-user-stories/sprint-2-generation.md) — Stories génération CV

---

## 🎬 Next Steps

### Priorité Immédiate (J1-J2)
1. ✅ Setup repository Next.js 14
2. ✅ Configuration NextAuth + MongoDB
3. ✅ Intégration Vercel AI SDK
4. ✅ Création templates react-pdf-renderer basiques

### Sprint 1 (J1-J4)
- Auth flow complet
- API generate-cv opérationnelle
- Editor basique fonctionnel

### Sprint 2 (J5-J7)
- Génération PDF production-ready
- Quota management
- Déploiement Docker + CI/CD
- Tests utilisateurs

---

**Version** : 2.0 (Prompt-Driven)  
**Dernière mise à jour** : 2024-01-16  
**Auteur** : SkillForge Team
