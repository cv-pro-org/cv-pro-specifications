# Génération CV Prompt-Driven — SkillForge MVP

## 🎯 Innovation Principale : Prompt-Driven Generation

**Rupture avec approche formulaire classique** : l'utilisateur décrit son parcours en **langage naturel**, l'IA structure automatiquement.

### Inspiration Maquette v0
- ✅ Textarea libre pour description parcours
- ✅ Option "CV ciblé pour un poste" avec description d'annonce
- ✅ Upload CV existant (OCR futur v2)
- ✅ Saisie vocale (Web Speech API)
- ✅ Génération via Vercel AI SDK (`generateObject()`)

### Objectif Production SkillForge MVP
Produire un **PDF haute qualité** via génération côté serveur.

---

## 📊 Modèle de Données (TypeScript)

```typescript
interface PersonalInfo {
  fullName: string
  title: string              // "Développeur Full-Stack Senior"
  email: string
  phone: string
  location: string           // "Paris, France"
  linkedin?: string
  website?: string
  summary: string            // 2-3 phrases générées par IA
}

interface Experience {
  id: string
  company: string
  position: string
  startDate: string          // "Jan 2022"
  endDate: string            // "Déc 2023" ou ""
  current: boolean
  description: string        // Paragraphe contexte
  highlights: string[]       // Bullet points réalisations
}

interface Education {
  id: string
  institution: string
  degree: string             // "Master"
  field: string              // "Informatique"
  startDate: string
  endDate: string
  description?: string
}

interface Skill {
  id: string
  name: string
  level: "beginner" | "intermediate" | "advanced" | "expert"
  category: string           // "Frontend", "Backend", "Tools", "Soft Skills"
}

interface Language {
  id: string
  name: string
  level: string              // "Natif", "Courant (C1)", "Intermédiaire (B1)"
}

interface CVData {
  personalInfo: PersonalInfo
  experiences: Experience[]
  education: Education[]
  skills: Skill[]
  languages: Language[]
}
```

---

## 🔌 API Endpoints

### `POST /api/generate-cv`

**Request Body:**
```typescript
interface CVGenerateRequest {
  prompt: string                    // Description libre du parcours
  jobDescription?: string           // Optionnel : annonce pour ciblage
  templateId?: string               // Template CV choisi (default: "modern")
}
```

**Response:**
```typescript
interface CVGenerateResponse {
  cv: CVData                        // Données structurées
  isDemo?: boolean                  // true si fallback démo
}
```

**Logique:**
1. Vérifier session NextAuth
2. Vérifier quota (5 CV/mois gratuit)
3. Construire prompt système selon contexte
4. Appel IA via Vercel AI SDK :
   ```typescript
   const { object: cv } = await generateObject({
     model: "anthropic/claude-sonnet-4",
     schema: cvSchema,
     prompt: systemPrompt,
     maxOutputTokens: 4000
   })
   ```
5. Incrémenter compteur quota user MongoDB
6. Retourner CVData structuré

**Gestion erreurs:**
- IA timeout (>10s) → Fallback `generateDemoCV(prompt)`
- Quota dépassé → 429 `{ error: "QUOTA_EXCEEDED" }`

---

### `POST /api/generate-pdf`

**Request Body:**
```typescript
interface PDFGenerateRequest {
  cvData: CVData
  templateId: string                // "modern", "classic", "minimal"
  colorScheme?: string              // "slate", "blue", "purple"
}
```

**Response:**
```typescript
interface PDFGenerateResponse {
  pdfUrl: string                    // URL téléchargement
}
```

**Logique:**
1. Vérifier session
2. Générer PDF via `react-pdf-renderer`
3. Sauvegarder temporairement : `/tmp/{userId}_{timestamp}.pdf`
4. Retourner URL sécurisée : `/api/download-pdf?token=...`

**Alternative v2 (Playwright):**
```typescript
// Meilleure qualité mais plus complexe
const browser = await playwright.chromium.launch()
const page = await browser.newPage()
await page.goto(`/internal/cv-render?cvId=...`)
const pdf = await page.pdf({ format: 'A4', printBackground: true })
```

---

## 🚀 Flow Utilisateur Complet

### 1. Landing Page (Non authentifié)

```
┌─────────────────────────────────────────┐
│  SkillForge — Créez votre CV en 2 min  │
│                                         │
│  ┌───────────────────────────────────┐ │
│  │ Décrivez votre parcours...        │ │
│  │                                   │ │
│  │ "Je suis développeur full-stack   │ │
│  │  avec 5 ans d'expérience..."      │ │
│  └───────────────────────────────────┘ │
│                                         │
│  ☐ CV pour un poste spécifique         │
│    ↳ [Coller annonce LinkedIn...]      │
│                                         │
│  [📎 Upload CV]  [🎤 Vocal]            │
│  [Générer mon CV →]                    │
└─────────────────────────────────────────┘
```

**Actions:**
- User tape prompt → Click "Générer"
- Si non connecté → Redirection vers login
- Si connecté → Appel `/api/generate-cv`

---

### 2. Dashboard (Authentifié)

```
┌──────────┬────────────────────────────────┐
│ Sidebar  │  Nouveau CV                    │
│          │  ┌──────────────────────────┐  │
│ + Nouveau│  │ Votre prompt ici...      │  │
│          │  └──────────────────────────┘  │
│ Historique│                                │
│ • CV Dev │  ☐ CV ciblé pour poste         │
│ • CV Mkt │                                │
│          │  [Générer →]                   │
│ Biblio   │                                │
│ • Modern │                                │
└──────────┴────────────────────────────────┘
```

**Features:**
- Historique récents (5 derniers)
- Compteur quota : "3/5 CV utilisés ce mois"
- Templates preview

---

### 3. Éditeur CV (Après génération)

```
┌─────────────┬──────────────┬─────────────┐
│ Template    │   Preview    │  Édition    │
│             │              │             │
│ • Modern ✓  │  ┌────────┐  │ ✏️ Info Perso│
│ • Classic   │  │  CV    │  │ • Nom       │
│ • Minimal   │  │  PDF   │  │ • Email     │
│             │  │ RENDER │  │             │
│ Couleurs    │  │        │  │ ✏️ Expérience│
│ 🎨 Slate ✓  │  └────────┘  │ + Ajouter   │
│ 🎨 Blue     │              │             │
│             │              │ ✏️ Skills    │
│ Polices     │              │ • React ⭐⭐⭐│
│ 📝 Inter ✓  │              │ • Node ⭐⭐   │
└─────────────┴──────────────┴─────────────┘
         [⬅️ Retour]  [💾 Sauvegarder]  [⬇️ Télécharger PDF]
```

**Features:**
- **Preview temps réel** : scaling responsive
- **Édition inline** : champs modifiables
- **Templates** : switch instant
- **Customisation** : couleurs, polices
- **Export** : PDF haute qualité

---

## 🧠 Prompt Engineering

### Prompt Base (Sans Job Description)

```
Tu es un expert en rédaction de CV professionnels français.
À partir de cette description, génère un CV complet et professionnel en français.

Description du candidat:
{userPrompt}

Instructions:
- Génère des informations réalistes et cohérentes basées sur la description
- Le résumé professionnel doit être percutant et mettre en valeur les points forts
- Les descriptions d'expérience doivent inclure des réalisations concrètes et mesurables
- Utilise un ton professionnel et des verbes d'action
- Si certaines informations manquent, invente des détails plausibles et cohérents
- Les dates doivent être au format français (Jan 2020, Mars 2022, etc.)
- Génère des IDs uniques pour chaque élément (UUID)
- Inclus au moins 5-8 compétences pertinentes
- Si aucune langue n'est mentionnée, inclus le Français (Natif) et l'Anglais (Courant)
```

### Prompt avec Job Targeting

```
Tu es un expert en rédaction de CV professionnels optimisés pour ATS et recruteurs.
À partir de la description du candidat, génère un CV adapté au poste visé.

Description du candidat:
{userPrompt}

Description du poste visé:
{jobDescription}

Instructions:
- Adapte le CV au poste en mettant en avant les compétences pertinentes
- Le résumé professionnel doit être percutant et orienté vers le poste
- Les descriptions d'expérience doivent inclure des réalisations alignées avec l'annonce
- Utilise un ton professionnel et des verbes d'action
- Utilise le vocabulaire de l'annonce pour optimiser pour les ATS
- Les dates doivent être au format français (Jan 2020, Mars 2022, etc.)
- Génère des IDs uniques pour chaque élément
- Inclus au moins 5-8 compétences pertinentes pour le poste
- Mets en avant l'expérience la plus pertinente pour le poste
```

---

## 🛡️ Gestion Erreurs & Fallback

### Stratégie Resilience

```typescript
export async function POST(req: Request) {
  try {
    const { prompt, jobDescription } = await req.json()
    
    try {
      // Tentative génération IA
      const { object: cv } = await generateObject({
        model: "anthropic/claude-sonnet-4",
        schema: cvSchema,
        prompt: buildSystemPrompt(prompt, jobDescription),
        maxOutputTokens: 4000
      })
      
      return Response.json({ cv })
      
    } catch (aiError) {
      console.error("AI generation failed, using demo fallback:", aiError)
      
      // Fallback démo offline
      const cv = generateDemoCV(prompt, jobDescription)
      return Response.json({ cv, isDemo: true })
    }
    
  } catch (error) {
    console.error("CV generation error:", error)
    return Response.json(
      { error: "Failed to generate CV" }, 
      { status: 500 }
    )
  }
}
```

### Fonction Fallback Démo

```typescript
function generateDemoCV(prompt: string, jobDescription?: string): CVData {
  const lowerPrompt = prompt.toLowerCase()
  
  // Détection métier par mots-clés
  let title = "Développeur Full-Stack"
  let skills = [
    { id: "1", name: "JavaScript", level: "expert", category: "Frontend" },
    { id: "2", name: "React", level: "expert", category: "Frontend" },
    { id: "3", name: "Node.js", level: "advanced", category: "Backend" },
    // ...
  ]
  
  if (lowerPrompt.includes("design") || lowerPrompt.includes("ux")) {
    title = "UX/UI Designer"
    skills = [
      { id: "1", name: "Figma", level: "expert", category: "Design" },
      // ...
    ]
  } else if (lowerPrompt.includes("marketing")) {
    title = "Responsable Marketing Digital"
    skills = [
      { id: "1", name: "SEO/SEA", level: "expert", category: "Marketing" },
      // ...
    ]
  }
  
  // Extraction nom si possible
  const nameMatch = prompt.match(/(?:je suis|m'appelle|nom est)\s+([A-ZÀ-Ÿ][a-zà-ÿ]+(?:\s+[A-ZÀ-Ÿ][a-zà-ÿ]+)?)/i)
  const fullName = nameMatch ? nameMatch[1] : "Marie Dupont"
  
  // Personnalisation summary si job description
  let summary = "Professionnel passionné avec 5+ ans d'expérience..."
  if (jobDescription) {
    summary = "Professionnel expérimenté correspondant au profil recherché..."
  }
  
  return {
    personalInfo: { fullName, title, email: "...", summary, /* ... */ },
    experiences: [ /* ... */ ],
    education: [ /* ... */ ],
    skills,
    languages: [
      { id: "1", name: "Français", level: "Natif" },
      { id: "2", name: "Anglais", level: "Courant (C1)" }
    ]
  }
}
```

---

## 📦 Stockage & Download (MVP)

### Approche Simple (Recommandée MVP)
```typescript
// Génération à la demande (pas de cache)
POST /api/generate-pdf
→ Génère buffer PDF en mémoire
→ Retourne blob au client
→ Client télécharge directement
```

**Avantages:**
- Simple à implémenter
- Pas de gestion stockage
- Pas de TTL/cleanup

**Inconvénients:**
- Régénération si user re-télécharge
- Latence génération (~2-3s)

### Alternative v2 (Post-MVP)
```typescript
// Stockage temporaire avec TTL
POST /api/generate-pdf
→ Génère PDF
→ Upload S3/R2 avec expiration 7j
→ Retourne URL signée
→ Cleanup automatique après expiration
```

---

## 🚀 Roadmap Technique

### MVP (7 jours) ✅
- [x] Prompt-driven generation
- [x] OpenAI/Claude via Vercel AI SDK
- [x] Éditeur visuel basique
- [x] PDF via react-pdf-renderer
- [x] Fallback démo offline
- [x] Quota simple (compteur MongoDB)

### v2 (Post-MVP) 📋
- [ ] Worker async (BullMQ + Redis)
- [ ] PDF via Playwright (meilleure qualité)
- [ ] Upload CV + OCR (Tesseract)
- [ ] Historique CV complet
- [ ] Versioning CV
- [ ] Stockage S3/R2
- [ ] Analytics génération
- [ ] A/B testing prompts

---

## 🧪 Tests & Validation

### Scénarios Test Critiques

**1. Génération CV basique**
```
Prompt: "Je suis développeur full-stack avec 5 ans d'expérience en React et Node.js"
Attendu: CV structuré avec expériences plausibles, skills pertinents
```

**2. Génération CV ciblée**
```
Prompt: "Je suis data scientist..."
Job: "Offre Lead Data Scientist - Python, ML, AWS..."
Attendu: CV optimisé avec mots-clés annonce
```

**3. Fallback démo**
```
Scénario: OpenAI timeout
Attendu: CV démo généré rapidement, flag isDemo: true
```

**4. Quota dépassé**
```
Scénario: User a déjà généré 5 CV ce mois
Attendu: 429 error avec message upgrade
```

### Métriques Qualité

- ⚡ **Latence génération** : <10s (IA) ou <1s (démo)
- 📊 **Taux succès IA** : >95%
- 🎯 **Satisfaction CV** : Test utilisateur (5 personnes)
- 🔒 **Sécurité** : Pas de fuite données prompt dans logs
