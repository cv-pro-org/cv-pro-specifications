# API Routes (MVP)

## Auth
- NextAuth routes standards

## Templates CV

### `GET /api/templates`
Liste tous les templates disponibles.
- Auth: non requise (public)
- Output: `TemplateSummary[]`

### `GET /api/templates/{templateId}`
Détail d'un template avec ses capacités.
- Auth: non requise (public)
- Output: `TemplateDetail`
- Errors: 404

## Modèles Templates

### TemplateSummary
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

### TemplateDetail
```python
class TemplateDetail(BaseModel):
    id: str
    name: str
    description: str
    preview_url: str
    style: str
    pages: int
    ats_friendly: bool
    supported_sections: List[str]
    limits: dict
```

---

## Jobs CV

### `POST /api/cv/jobs`
Crée un job de génération.
- Auth: obligatoire
- Input: `CVGenerateRequest` (template_id + CVFormData)
- Output: `CVJobResponse { job_id, status, created_at }`
- Errors: 401, 429 (rate/quota), 400, 404 (template invalide)

### CVGenerateRequest (Input)
```python
class CVGenerateRequest(BaseModel):
    template_id: str
    data: CVFormData
```

## Modèles de données (Pydantic)

### Experience
```python
class Experience(BaseModel):
    company: str
    position: str
    start_date: str
    end_date: Optional[str] = "Présent"
    achievements: List[str] = []
```

### Education
```python
class Education(BaseModel):
    school: str
    degree: str
    year: str
```

### CVFormData (Input)
```python
class CVFormData(BaseModel):
    first_name: str
    last_name: str
    title: str
    email: Optional[str] = None
    phone: Optional[str] = None
    location: Optional[str] = None
    summary: str
    experiences: List[Experience] = []
    education: List[Education] = []
    skills: List[str] = []
```

### CVJobResponse (Output)
```python
class CVJobResponse(BaseModel):
    job_id: str
    status: str  # "queued" | "running" | "done" | "failed"
    error: Optional[str] = None
    download_url: Optional[str] = None
    created_at: str
```

### `GET /api/cv/jobs/:jobId`
Statut job.
- Output: `{ status, error?, downloadUrl? }`

### `GET /api/cv/jobs/:jobId/download`
Télécharge le PDF (stream).
- Auth: obligatoire (job du user)
- Errors: 404, 409 (pas prêt)

## Routes internes (appelées par le worker)
### `GET /internal/cv-render?jobId=...&token=...`
Retourne HTML du CV à rendre.
- Sécurisé via token interne (HMAC / secret)

### `POST /internal/cv-store`
Optionnel si on veut que l’app écrive le PDF; sinon le worker écrit direct dans le volume.
