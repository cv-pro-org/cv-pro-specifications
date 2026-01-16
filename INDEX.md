# 📚 SkillForge — Index Documentation

> **Guide complet des spécifications techniques et fonctionnelles de la plateforme SkillForge (MVP 7 jours)**

---

## 🎯 Documents Essentiels (Commencer ici)

| Document | Description | Audience |
|----------|-------------|----------|
| **[README.md](./README.md)** | Vue d'ensemble projet + objectifs MVP | 👥 Tous |
| **[SYNTHESIS.md](./SYNTHESIS.md)** | Synthèse complète : vision, architecture, parcours user | 👥 Tous |
| **[CHANGELOG.md](./CHANGELOG.md)** | Historique changements v1 → v2 (prompt-driven) | 🔄 Migration |
| **[ADR.md](./ADR.md)** | Architecture Decision Records (décisions techniques) | 🏗️ Tech |

---

## 📖 Documentation par Thème

### 1️⃣ Architecture Technique

**Dossier** : [`01-architecture/`](./01-architecture/)

| Fichier | Contenu |
|---------|---------|
| `stack-technique.md` | Stack complète : Next.js, MongoDB, Vercel AI SDK, Docker |
| `schema-mongodb.md` | Collections MongoDB + indexes + exemples |
| `docker-compose.yml` | Configuration Traefik + app + volumes |
| `deployment-diagram.md` | Schémas infrastructure (Oracle ARM) |

**Liens rapides** :
- 🔧 [Stack Technique Détaillée](./01-architecture/stack-technique.md)
- 🗄️ [Schéma MongoDB](./01-architecture/schema-mongodb.md)

---

### 2️⃣ Spécifications Fonctionnelles

**Dossier** : [`02-specifications/`](./02-specifications/)

| Fichier | Contenu |
|---------|---------|
| **`cv-generation-prompt.md`** ⭐ | **Cœur innovation : génération prompt-driven** |
| **`api-routes-v2.md`** ⭐ | **Endpoints API complets (v2)** |
| `auth-flow.md` | Flux authentification Magic Link |
| `quota-management.md` | Gestion limites 5 CV/mois |
| `templates.md` | Système templates CV |
| `cv-generation.md.backup` | Archive ancienne version (worker async) |

**Liens rapides** :
- 🤖 [Génération Prompt-Driven](./02-specifications/cv-generation-prompt.md) — **À LIRE EN PRIORITÉ**
- 🔌 [API Routes v2](./02-specifications/api-routes-v2.md)
- 🔐 [Auth Flow](./02-specifications/auth-flow.md)

---

### 3️⃣ User Stories & Sprints

**Dossier** : [`03-user-stories/`](./03-user-stories/)

#### Sprint 1 (J1-J4) : Authentification + Génération

| Story | Titre | Points |
|-------|-------|--------|
| US-001 | Landing page publique | 3 |
| US-002 | Magic Link login | 5 |
| US-003 | Dashboard authentifié | 3 |
| US-004 | Génération CV par prompt | 8 |
| US-005 | Fallback démo offline | 3 |
| US-006 | Gestion quota utilisateur | 5 |

**Total Sprint 1** : ~27 points

#### Sprint 2 (J5-J7) : Éditeur + PDF

| Story | Titre | Points |
|-------|-------|--------|
| US-007 | Éditeur visuel CV | 8 |
| US-008 | Sélection templates | 3 |
| US-009 | Customisation couleurs/polices | 5 |
| US-010 | Génération PDF | 8 |
| US-011 | Download PDF sécurisé | 3 |

**Total Sprint 2** : ~27 points

**Liens rapides** :
- 📋 [Sprint 1 — Auth + Génération](./03-user-stories/sprint-1/)
- 📋 [Sprint 2 — Éditeur + PDF](./03-user-stories/sprint-2/)
- 📦 [Backlog Priorisé](./03-user-stories/backlog.md)

---

### 4️⃣ Déploiement & CI/CD

**Dossier** : [`04-deployment/`](./04-deployment/)

| Fichier | Contenu |
|---------|---------|
| `github-actions.md` | Workflow CI/CD complet |
| `deploy-guide.md` | Procédure déploiement pas-à-pas |
| `monitoring.md` | Logs, health checks, alertes |
| `rollback-procedure.md` | Procédure rollback en cas d'erreur |

**Liens rapides** :
- 🚀 [GitHub Actions Workflow](./04-deployment/github-actions.md)
- 📘 [Guide Déploiement](./04-deployment/deploy-guide.md)

---

### 5️⃣ Design & Maquettes

**Dossier** : [`05-design/`](./05-design/)

| Ressource | Description |
|-----------|-------------|
| **Maquette v0** | App déployée : [Vercel](https://v0-cv-generation-app-git-feature-soungsid-5856s-projects.vercel.app) |
| **Repository** | Code source : [GitHub](https://github.com/soungsid/v0-cv-generation-app) (branch `feature`) |
| `design-system.md` | Composants UI, couleurs, typographie |
| `wireframes/` | Wireframes Figma/Excalidraw |

**Liens rapides** :
- 🎨 [Design System](./05-design/design-system.md)
- 🖼️ [Wireframes](./05-design/wireframes/)

---

## 🔍 Navigation par Rôle

### Product Manager
1. [SYNTHESIS.md](./SYNTHESIS.md) — Vision produit
2. [cv-generation-prompt.md](./02-specifications/cv-generation-prompt.md) — Innovation UX
3. [User Stories Sprint 1](./03-user-stories/sprint-1/)
4. [Backlog](./03-user-stories/backlog.md)

### Développeur Backend
1. [api-routes-v2.md](./02-specifications/api-routes-v2.md) — Endpoints API
2. [schema-mongodb.md](./01-architecture/schema-mongodb.md) — Base de données
3. [cv-generation-prompt.md](./02-specifications/cv-generation-prompt.md) — Logique génération IA
4. [ADR.md](./ADR.md) — Décisions techniques

### Développeur Frontend
1. [cv-generation-prompt.md](./02-specifications/cv-generation-prompt.md) — Flow utilisateur
2. [Design System](./05-design/design-system.md) — Composants UI
3. [api-routes-v2.md](./02-specifications/api-routes-v2.md) — Intégration API
4. [Maquette v0](https://github.com/soungsid/v0-cv-generation-app) — Référence code

### DevOps
1. [docker-compose.yml](./01-architecture/docker-compose.yml) — Config containers
2. [deploy-guide.md](./04-deployment/deploy-guide.md) — Procédure déploiement
3. [github-actions.md](./04-deployment/github-actions.md) — CI/CD
4. [monitoring.md](./04-deployment/monitoring.md) — Observabilité

---

## 🚀 Quick Start (Nouveau sur le Projet)

### Étape 1 : Comprendre le Contexte
1. Lire [README.md](./README.md) (5 min)
2. Lire [SYNTHESIS.md](./SYNTHESIS.md) (15 min)
3. Consulter [Maquette v0](https://github.com/soungsid/v0-cv-generation-app) (10 min)

### Étape 2 : Approfondir Technique
1. [cv-generation-prompt.md](./02-specifications/cv-generation-prompt.md) — Innovation prompt-driven
2. [api-routes-v2.md](./02-specifications/api-routes-v2.md) — Contrat API
3. [ADR.md](./ADR.md) — Décisions d'architecture

### Étape 3 : Implémentation
1. [User Stories Sprint 1](./03-user-stories/sprint-1/) — Backlog priorisé
2. [stack-technique.md](./01-architecture/stack-technique.md) — Setup environnement
3. [deploy-guide.md](./04-deployment/deploy-guide.md) — Déploiement local

**Temps total** : ~1h pour maîtriser le projet

---

## 📊 Métriques Documentation

| Métrique | Valeur |
|----------|--------|
| **Documents totaux** | 40+ fichiers |
| **User Stories** | 22 stories |
| **Points Fibonacci** | 54 points (27+27) |
| **Pages specs** | ~150 pages A4 équivalent |
| **ADR** | 8 décisions majeures |
| **API Endpoints** | 12 routes |

---

## 🔄 Versions & Historique

| Version | Date | Changements Majeurs |
|---------|------|---------------------|
| **2.0** | 2024-01-16 | Refonte prompt-driven, renommage SkillForge |
| **1.1** | 2024-01-12 | Ajout templates système |
| **1.0** | 2024-01-10 | Specs initiales (worker async) |

Voir [CHANGELOG.md](./CHANGELOG.md) pour détails complets.

---

## 🛠️ Outils & Technologies

### Stack Production
- **Frontend/Backend** : Next.js 14, React 18, Tailwind CSS
- **IA** : Vercel AI SDK, Claude Sonnet 4 (Anthropic)
- **Base de données** : MongoDB Atlas M0
- **Auth** : NextAuth.js (Magic Link)
- **Email** : Resend API
- **PDF** : react-pdf-renderer (MVP), Playwright (v2)
- **Infra** : Docker Compose, Traefik, Oracle Linux ARM

### Stack Dev
- **Language** : TypeScript 5.0+
- **Validation** : Zod schemas
- **Animations** : Framer Motion
- **UI Library** : shadcn/ui
- **Icons** : Lucide React

---

## 🤝 Contribution

### Proposer Modifications

1. **Issues** : Signaler bugs/suggestions sur GitHub
2. **ADR** : Documenter décisions techniques majeures
3. **User Stories** : Suivre template existant (critères acceptation + points)

### Standards Documentation

- **Format** : Markdown + frontmatter YAML
- **Langue** : Français (specs fonctionnelles), Anglais (code)
- **Ligne max** : 120 caractères
- **Diagrammes** : Mermaid ou ASCII art

---

## 📞 Support

**Questions** : Ouvrir [Discussion GitHub](https://github.com/cv-pro-org/skillforge/discussions)  
**Bugs specs** : Créer [Issue](https://github.com/cv-pro-org/skillforge/issues)  
**Contact équipe** : `team@skillforge.app`

---

## 📜 Licence

Documentation © 2024 SkillForge Team  
Licence : [CC BY-SA 4.0](https://creativecommons.org/licenses/by-sa/4.0/)

---

**Dernière mise à jour** : 2024-01-16  
**Version** : 2.0.0  
**Contributeurs** : Product Team, Tech Team, Design Team
