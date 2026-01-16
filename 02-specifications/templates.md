# Templates CV — Spécification

## Principe fondamental

> **Le template ne doit JAMAIS imposer le modèle de données.**
> 👉 Il s'adapte à `CVFormData`, pas l'inverse.

Le modèle de données est fixe :
- `CVFormData` (first_name, last_name, title, email, phone, location, linkedin, github, website, summary, experiences, education, skills, languages, qualities)
- `Experience` (company, position, start_date, end_date, location, achievements)
- `Education` (school, degree, year)
- `Language` (name, level)

Les templates **exposent des capacités**, pas une structure rigide.

---

## 1. Architecture des templates

### 1.1 Structure fichiers

```
templates/
└── modern-01/
    ├── template.html      # Template Handlebars/Jinja
    ├── preview.png        # Aperçu statique (card catalogue)
    ├── meta.json          # Métadonnées publiques (catalogue)
    └── rules.json         # Contrat interne (capacités/limites)
```

### 1.2 meta.json (données publiques)

Exposé via l'API pour le catalogue frontend.

```json
{
  "id": "modern-01",
  "name": "Modern One",
  "description": "CV moderne et épuré, idéal pour les profils tech",
  "style": "modern",
  "pages": 1,
  "ats_friendly": true
}
```

| Champ | Type | Description |
|-------|------|-------------|
| `id` | string | Identifiant unique (slug) |
| `name` | string | Nom affiché |
| `description` | string | Description courte |
| `style` | enum | `modern`, `classic`, `creative`, `minimal` |
| `pages` | int | Nombre de pages prévu |
| `ats_friendly` | bool | Compatible ATS (parsers RH) |

### 1.3 rules.json (contrat interne)

Utilisé par le backend/worker pour valider et adapter les données.

```json
{
  "supports": {
    "summary": true,
    "experiences": true,
    "education": true,
    "skills": true,
    "languages": true,
    "qualities": true,
    "social_links": true
  },
  "limits": {
    "experiences": 5,
    "skills": 10,
    "achievements_per_experience": 4,
    "languages": 3,
    "qualities": 5
  },
  "layout": {
    "density": "compact"
  }
}
```

| Champ | Description |
|-------|-------------|
| `supports` | Sections supportées par le template |
| `limits` | Limites d'éléments (tronqué si dépassé) |
| `layout.density` | `compact` ou `spacious` |

---

## 2. Modèles API

### 2.1 TemplateSummary (liste catalogue)

```python
class TemplateSummary(BaseModel):
    id: str
    name: str
    description: str
    preview_url: str
    style: str              # modern, classic, creative, minimal
    pages: int
    ats_friendly: bool
```

### 2.2 TemplateDetail (détail complet)

```python
class TemplateDetail(BaseModel):
    id: str
    name: str
    description: str
    preview_url: str
    style: str
    pages: int
    ats_friendly: bool
    supported_sections: List[str]   # ["summary", "experiences", ...]
    limits: dict                     # {"experiences": 5, "skills": 10}
```

---

## 3. Endpoints API

### 3.1 Liste des templates

```
GET /api/templates
```

**Auth:** Non requis (public)

**Response:**
```json
[
  {
    "id": "modern-01",
    "name": "Modern One",
    "description": "CV moderne et épuré",
    "preview_url": "/templates/modern-01/preview.png",
    "style": "modern",
    "pages": 1,
    "ats_friendly": true
  },
  {
    "id": "classic-01",
    "name": "Classic Pro",
    "description": "CV traditionnel et professionnel",
    "preview_url": "/templates/classic-01/preview.png",
    "style": "classic",
    "pages": 2,
    "ats_friendly": true
  }
]
```

### 3.2 Détail d'un template

```
GET /api/templates/{templateId}
```

**Auth:** Non requis (public)

**Response:**
```json
{
  "id": "modern-01",
  "name": "Modern One",
  "description": "CV moderne et épuré",
  "preview_url": "/templates/modern-01/preview.png",
  "style": "modern",
  "pages": 1,
  "ats_friendly": true,
  "supported_sections": ["summary", "experiences", "education", "skills"],
  "limits": {
    "experiences": 5,
    "skills": 10
  }
}
```

**Errors:**
- `404` : Template non trouvé

---

## 4. Intégration génération CV

### 4.1 Modification POST /api/cv/jobs

Le `template_id` devient **obligatoire** dans la requête :

```python
class CVGenerateRequest(BaseModel):
    template_id: str
    data: CVFormData
```

**Request:**
```json
{
  "template_id": "modern-01",
  "data": {
    "first_name": "Jean",
    "last_name": "Dupont",
    "title": "Développeur Full Stack",
    "summary": "...",
    "experiences": [...],
    "education": [...],
    "skills": ["Python", "React", "Docker"]
  }
}
```

### 4.2 Validation backend

1. Vérifier que `template_id` existe
2. Charger `rules.json` du template
3. Tronquer les données si elles dépassent les limites
4. Passer au worker pour génération

---

## 5. Mapping template → CVFormData

Le template HTML utilise les variables du modèle directement :

```html
<h1>{{first_name}} {{last_name}}</h1>
<h2>{{title}}</h2>

<section class="summary">
  <p>{{summary}}</p>
</section>

<section class="experiences">
  {{#each experiences}}
  <div class="experience">
    <strong>{{position}}</strong> – {{company}}
    <span class="dates">{{start_date}} - {{end_date}}</span>
    <ul>
      {{#each achievements}}
      <li>{{this}}</li>
      {{/each}}
    </ul>
  </div>
  {{/each}}
</section>

<section class="education">
  {{#each education}}
  <div class="education-item">
    <strong>{{degree}}</strong> – {{school}} ({{year}})
  </div>
  {{/each}}
</section>

<section class="skills">
  {{#each skills}}
  <span class="skill-tag">{{this}}</span>
  {{/each}}
</section>
```

---

## 6. Templates MVP (Sprint 1)

Pour le MVP, 2 templates suffisent :

| ID | Nom | Style | Pages | ATS |
|----|-----|-------|-------|-----|
| `modern-01` | Modern One | modern | 1 | ✅ |
| `classic-01` | Classic Pro | classic | 1 | ✅ |

---

## 7. Scope Sprint 1 vs v2

### ✅ Sprint 1 (MVP)
- Endpoint liste templates (`GET /api/templates`)
- Endpoint détail template (`GET /api/templates/{id}`)
- 2 templates statiques avec preview.png
- Intégration `template_id` dans génération
- Affichage catalogue frontend (cards simples)

### ❌ v2 (Post-MVP)
- Preview interactive (données mock injectées)
- Thèmes couleur par template
- Upload template custom
- A/B testing templates
