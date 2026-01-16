# API Routes — SkillForge MVP (Prompt-Driven)

## 📋 Vue d'ensemble

Architecture **React + Python FastAPI** : frontend séparé, backend API REST.

**Stack:**
- Frontend : React 18 (Vite build)
- Backend : Python 3.11+ FastAPI
- Auth : JWT tokens (Google OAuth)
- Base de données : MongoDB Atlas (Motor driver)

---

## 🔐 Authentification

### `GET /api/auth/google`

Initie le flow OAuth Google.

**Auth:** ❌ Non requise

**Response:**
```json
{
  "authUrl": "https://accounts.google.com/o/oauth2/v2/auth?client_id=..."
}
```

**Frontend action:**
```javascript
// Redirect user to authUrl
window.location.href = response.authUrl
```

---

### `GET /api/auth/google/callback`

Callback OAuth Google (après autorisation).

**Query Params:**
- `code` — Authorization code Google
- `state` — CSRF token

**Response:**
```json
{
  "access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "refresh_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "token_type": "Bearer",
  "expires_in": 3600,
  "user": {
    "id": "usr_123abc",
    "email": "user@gmail.com",
    "name": "Marie Dupont",
    "picture": "https://lh3.googleusercontent.com/..."
  }
}
```

**Frontend action:**
```javascript
// Store tokens
localStorage.setItem('access_token', response.access_token)
localStorage.setItem('refresh_token', response.refresh_token)

// Redirect to dashboard
navigate('/dashboard')
```

---

### `POST /api/auth/refresh`

Renouvelle le JWT token expiré.

**Auth:** ✅ Requise (refresh token)

**Request Body:**
```json
{
  "refresh_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
}
```

**Response:**
```json
{
  "access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "expires_in": 3600
}
```

---

### `POST /api/auth/logout`

Déconnecte l'utilisateur (invalidation token).

**Auth:** ✅ Requise

**Response:**
```json
{
  "success": true
}
```

**Frontend action:**
```javascript
// Clear tokens
localStorage.removeItem('access_token')
localStorage.removeItem('refresh_token')

// Redirect to landing
navigate('/')
```

---

## 🎨 Templates CV

### `GET /api/templates`

Liste tous les templates disponibles.

**Auth:** ❌ Non requise (public)

**Response:**
```python
# Python Pydantic models
from pydantic import BaseModel
from typing import List

class TemplateSummary(BaseModel):
    id: str                  # "modern", "classic", "minimal", "creative"
    name: str                # "Modern Professional"
    description: str         # "Design épuré et moderne..."
    preview_url: str         # "/templates/previews/modern.png"
    style: str               # "modern" | "classic" | "minimal" | "creative"
    pages: int               # 1 ou 2
    ats_friendly: bool       # Optimisé ATS
    is_premium: bool         # Réservé plan payant (v2)

class TemplatesResponse(BaseModel):
    templates: List[TemplateSummary]
```

**Exemple:**
```json
{
  "templates": [
    {
      "id": "modern",
      "name": "Modern Professional",
      "description": "Design épuré et moderne avec barre latérale",
      "previewUrl": "/templates/previews/modern.png",
      "style": "modern",
      "pages": 1,
      "atsFriendly": true,
      "isPremium": false
    },
    {
      "id": "classic",
      "name": "Classic Timeline",
      "description": "Layout traditionnel avec timeline",
      "previewUrl": "/templates/previews/classic.png",
      "style": "classic",
      "pages": 1,
      "atsFriendly": true,
      "isPremium": false
    }
  ]
}
```

---

### `GET /api/templates/:templateId`

Détail d'un template avec ses options.

**Auth:** ❌ Non requise

**Response:**
```python
class ColorScheme(BaseModel):
    id: str
    name: str
    primary: str    # Hex color
    accent: str

class TemplateDetail(TemplateSummary):
    color_schemes: List[ColorScheme]
    font_options: List[str]
    supported_sections: List[str]   # ["experience", "education", "skills", "languages"]
    customizable: dict              # {"colors": true, "fonts": true, "layout": false}
```
  accent: string
}
```

**Errors:**
- `404` — Template not found

---

## 🤖 Génération CV (IA)

### `POST /api/generate-cv`

Génère un CV structuré à partir d'un prompt.

**Auth:** ✅ Requise (JWT Bearer token)

**Headers:**
```
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
```

**Request Body:**
```python
class CVGenerateRequest(BaseModel):
    prompt: str                     # Description parcours (max 2000 chars)
    job_description: Optional[str]  # Optionnel : annonce ciblage (max 3000 chars)
    template_id: str = "modern"     # Default: "modern"
```

**Response:**
```python
class CVGenerateResponse(BaseModel):
    cv: CVData                      # Données structurées (voir types ci-dessous)
    is_demo: bool                   # true si fallback démo utilisé
    quota_remaining: int            # Nombre CV restants ce mois
```

**Errors:**
- `401 Unauthorized` — Non authentifié
- `429 Too Many Requests` — Quota mensuel dépassé
  ```json
  {
    "error": "QUOTA_EXCEEDED",
    "message": "Limite mensuelle de 5 CV atteinte",
    "upgradeUrl": "/pricing"
  }
  ```
- `400 Bad Request` — Prompt invalide (trop court/long)
- `500 Internal Server Error` — Erreur génération

**Exemple Request:**
```json
{
  "prompt": "Je suis développeur full-stack avec 5 ans d'expérience. J'ai travaillé chez Google puis dans une startup fintech. Je maîtrise React, Node.js, Python et PostgreSQL. J'ai une formation d'ingénieur en informatique.",
  "jobDescription": "Nous recherchons un Lead Developer Full-Stack pour notre équipe produit. Compétences requises : React, Node.js, AWS, management d'équipe.",
  "templateId": "modern"
}
```

**Exemple Response:**
```json
{
  "cv": {
    "personalInfo": {
      "fullName": "Alexandre Martin",
      "title": "Lead Developer Full-Stack",
      "email": "alexandre.martin@email.com",
      "phone": "06 12 34 56 78",
      "location": "Paris, France",
      "linkedin": "linkedin.com/in/alexandre-martin",
      "summary": "Lead Developer Full-Stack avec 5+ ans d'expérience..."
    },
    "experiences": [ /* ... */ ],
    "education": [ /* ... */ ],
    "skills": [ /* ... */ ],
    "languages": [ /* ... */ ]
  },
  "isDemo": false,
  "quotaRemaining": 3
}
```

---

## 📄 Génération PDF

### `POST /api/generate-pdf`

Génère le PDF final à partir des données CV.

**Auth:** ✅ Requise

**Request Body:**
```typescript
interface PDFGenerateRequest {
  cvData: CVData
  templateId: string                // "modern", "classic", etc.
  colorScheme?: string              // "slate", "blue", "purple", "green"
  fontFamily?: string               // "Inter", "Roboto", "Open Sans"
}
```

**Response:**
```typescript
interface PDFGenerateResponse {
  pdfUrl: string                    // URL temporaire de téléchargement
  expiresAt: string                 // ISO timestamp (30 min)
}
```

**Exemple Response:**
```json
{
  "pdfUrl": "/api/download-pdf?token=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "expiresAt": "2024-01-20T15:30:00Z"
}
```

**Errors:**
- `401 Unauthorized`
- `400 Bad Request` — CVData invalide
- `500 Internal Server Error` — Erreur génération PDF

---

### `GET /api/download-pdf`

Télécharge le PDF généré.

**Auth:** ✅ Token sécurisé dans query param

**Query Params:**
- `token` — JWT signé contenant `{ userId, cvId, exp }`

**Response:**
- `Content-Type: application/pdf`
- `Content-Disposition: attachment; filename="prenom-nom-cv.pdf"`
- Body: PDF binary stream

**Errors:**
- `401 Unauthorized` — Token invalide/expiré
- `404 Not Found` — PDF inexistant

---

## 👤 User Profile & Quota

### `GET /api/user/profile`

Récupère infos utilisateur + quota.

**Auth:** ✅ Requise

**Response:**
```typescript
interface UserProfile {
  id: string
  email: string
  name?: string
  createdAt: string
  quota: {
    used: number              // CV générés ce mois
    limit: number             // 5 pour plan gratuit
    resetsAt: string          // ISO timestamp (1er du mois suivant)
  }
  plan: "free" | "pro"        // v2
}
```

**Exemple:**
```json
{
  "id": "usr_123abc",
  "email": "user@example.com",
  "name": "Marie Dupont",
  "createdAt": "2024-01-15T10:00:00Z",
  "quota": {
    "used": 3,
    "limit": 5,
    "resetsAt": "2024-02-01T00:00:00Z"
  },
  "plan": "free"
}
```

---

### `PATCH /api/user/profile`

Met à jour profil utilisateur.

**Auth:** ✅ Requise

**Request Body:**
```typescript
interface UserProfileUpdate {
  name?: string
  preferences?: {
    defaultTemplateId?: string
    defaultColorScheme?: string
  }
}
```

**Response:**
```typescript
{
  success: boolean
  profile: UserProfile
}
```

---

## 📂 Historique CV (v2 — Post-MVP)

### `GET /api/cvs`

Liste les CV générés par l'utilisateur.

**Auth:** ✅ Requise

**Query Params:**
- `limit` — Number (default: 10, max: 50)
- `offset` — Number (default: 0)

**Response:**
```typescript
interface CVListResponse {
  cvs: CVSummary[]
  total: number
  hasMore: boolean
}

interface CVSummary {
  id: string
  title: string              // "CV Développeur Full-Stack"
  templateId: string
  createdAt: string
  updatedAt: string
  pdfUrl?: string            // Si encore disponible
}
```

---

### `GET /api/cvs/:cvId`

Récupère les données complètes d'un CV.

**Auth:** ✅ Requise (owner uniquement)

**Response:**
```typescript
{
  cv: CVData
  metadata: {
    id: string
    createdAt: string
    templateId: string
    colorScheme: string
  }
}
```

---

### `DELETE /api/cvs/:cvId`

Supprime un CV de l'historique.

**Auth:** ✅ Requise (owner uniquement)

**Response:**
```typescript
{
  success: boolean
}
```

---

## 🔧 Health Check

### `GET /api/health`

Vérifie l'état de santé du service.

**Auth:** ❌ Non requise

**Response:**
```typescript
{
  status: "ok" | "degraded" | "down"
  timestamp: string
  services: {
    database: "ok" | "down"
    ai: "ok" | "down"
  }
}
```

---

## 📊 Types Partagés

### CVData (Complet)

```typescript
interface CVData {
  personalInfo: PersonalInfo
  experiences: Experience[]
  education: Education[]
  skills: Skill[]
  languages: Language[]
}

interface PersonalInfo {
  fullName: string
  title: string
  email: string
  phone: string
  location: string
  linkedin?: string
  website?: string
  summary: string
}

interface Experience {
  id: string
  company: string
  position: string
  startDate: string          // "Jan 2022"
  endDate: string            // "Déc 2023" ou ""
  current: boolean
  description: string
  highlights: string[]
}

interface Education {
  id: string
  institution: string
  degree: string
  field: string
  startDate: string
  endDate: string
  description?: string
}

interface Skill {
  id: string
  name: string
  level: "beginner" | "intermediate" | "advanced" | "expert"
  category: string
}

interface Language {
  id: string
  name: string
  level: string              // "Natif", "Courant (C1)", etc.
}
```

---

## 🚀 Rate Limiting (v2)

Toutes les routes authentifiées sont rate-limitées :

- **Génération CV** : 10 requêtes / 15 min par user
- **Génération PDF** : 20 requêtes / 15 min par user
- **Autres routes** : 100 requêtes / 15 min par user

Headers de réponse :
```
X-RateLimit-Limit: 10
X-RateLimit-Remaining: 7
X-RateLimit-Reset: 1642684800
```

Si limite dépassée :
```json
{
  "error": "RATE_LIMIT_EXCEEDED",
  "message": "Trop de requêtes. Réessayez dans 5 minutes.",
  "retryAfter": 300
}
```

---

## 🔒 Sécurité

### Headers requis
- `Content-Type: application/json`
- `Cookie: next-auth.session-token=...` (géré automatiquement par NextAuth)

### CORS
- Désactivé (same-origin uniquement)
- Pas d'API publique externe (v1)

### CSRF Protection
- NextAuth CSRF token automatique
- Toutes les mutations POST/PATCH/DELETE protégées

### Input Validation
- Zod schemas pour toutes les entrées
- Sanitization des prompts (pas d'injection)
- Limite taille requêtes : 10 MB
