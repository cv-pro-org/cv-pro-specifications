# Migration Stack React + Python — SkillForge

**Date** : 2024-01-16  
**Context** : Clarification stack technique finale

---

## ✅ Stack Confirmée

### Frontend
```
React 18
├─ Build: Vite (ou Create React App)
├─ Routing: React Router v6
├─ UI: Tailwind CSS + shadcn/ui
├─ HTTP: Axios
├─ State: React Context/Zustand
└─ Déploiement: Nginx (static files)
```

### Backend
```
Python 3.11+
├─ Framework: FastAPI
├─ Validation: Pydantic
├─ Auth: JWT (PyJWT + authlib pour OAuth)
├─ DB: MongoDB (Motor async driver)
├─ IA: anthropic SDK (Claude) ou openai SDK
├─ PDF: WeasyPrint (MVP) ou Playwright Python (v2)
└─ Server: Uvicorn (ASGI)
```

---

## 🔧 Changements Techniques Majeurs

### 1. Authentification

**Avant (Next.js)** :
```typescript
// NextAuth.js handles everything
import NextAuth from "next-auth"
import GoogleProvider from "next-auth/providers/google"
```

**Après (React + FastAPI)** :
```python
# Backend: FastAPI + authlib
from authlib.integrations.starlette_client import OAuth
from jose import jwt

@app.get("/api/auth/google")
async def google_login():
    return {"auth_url": oauth.google.authorize_redirect(...)}

@app.get("/api/auth/google/callback")
async def google_callback(code: str):
    token = await oauth.google.authorize_access_token(code)
    user_info = await oauth.google.userinfo()
    
    # Create JWT
    access_token = create_access_token(user_info)
    return {"access_token": access_token, "user": user_info}
```

```javascript
// Frontend: React + Axios
const handleGoogleLogin = async () => {
  const { data } = await axios.get('/api/auth/google')
  window.location.href = data.auth_url
}

// In callback page
const { code } = useQuery()
const { data } = await axios.get(`/api/auth/google/callback?code=${code}`)
localStorage.setItem('access_token', data.access_token)
navigate('/dashboard')
```

### 2. API Calls

**Avant (Next.js)** :
```typescript
// Server-side direct
import { getServerSession } from "next-auth"

export async function POST(req: Request) {
  const session = await getServerSession()
  const body = await req.json()
  // ...
}
```

**Après (React + FastAPI)** :
```python
# Backend: FastAPI
from fastapi import Depends, HTTPException
from fastapi.security import HTTPBearer

security = HTTPBearer()

async def get_current_user(token: str = Depends(security)):
    try:
        payload = jwt.decode(token.credentials, JWT_SECRET)
        return payload
    except:
        raise HTTPException(401, "Invalid token")

@app.post("/api/generate-cv")
async def generate_cv(
    request: CVGenerateRequest,
    user = Depends(get_current_user)
):
    # Check quota
    # Generate CV
    return {"cv": cv_data}
```

```javascript
// Frontend: React + Axios
import axios from 'axios'

const api = axios.create({
  baseURL: process.env.REACT_APP_API_URL,
  headers: {
    'Authorization': `Bearer ${localStorage.getItem('access_token')}`
  }
})

const generateCV = async (prompt) => {
  const { data } = await api.post('/api/generate-cv', { prompt })
  return data
}
```

### 3. Génération IA

**Avant (Next.js + Vercel AI SDK)** :
```typescript
import { generateObject } from 'ai'
import { anthropic } from '@ai-sdk/anthropic'

const { object: cv } = await generateObject({
  model: anthropic('claude-sonnet-4'),
  schema: cvSchema,
  prompt: systemPrompt
})
```

**Après (Python + Anthropic SDK)** :
```python
from anthropic import Anthropic
import json

client = Anthropic(api_key=settings.ANTHROPIC_API_KEY)

response = client.messages.create(
    model="claude-sonnet-4-20250514",
    max_tokens=4000,
    system=system_prompt,
    messages=[{
        "role": "user",
        "content": user_prompt
    }]
)

# Parse structured output
cv_data = json.loads(response.content[0].text)
cv = CVData(**cv_data)  # Pydantic validation
```

### 4. Génération PDF

**Option A : WeasyPrint (Simple, MVP)**
```python
from weasyprint import HTML
from jinja2 import Template

# Template HTML/CSS
template = Template('''
<!DOCTYPE html>
<html>
<head>
  <style>
    @page { size: A4; margin: 2cm; }
    body { font-family: Arial, sans-serif; }
  </style>
</head>
<body>
  <h1>{{ cv.personal_info.full_name }}</h1>
  <p>{{ cv.personal_info.summary }}</p>
  <!-- ... -->
</body>
</html>
''')

html_content = template.render(cv=cv_data)
pdf_bytes = HTML(string=html_content).write_pdf()

return StreamingResponse(
    io.BytesIO(pdf_bytes),
    media_type="application/pdf",
    headers={"Content-Disposition": "attachment; filename=cv.pdf"}
)
```

**Option B : Playwright Python (Meilleure qualité)**
```python
from playwright.async_api import async_playwright

async def generate_pdf(cv_data: CVData):
    async with async_playwright() as p:
        browser = await p.chromium.launch()
        page = await browser.new_page()
        
        # Render HTML template
        html_content = render_template(cv_data)
        await page.set_content(html_content)
        
        # Generate PDF
        pdf_bytes = await page.pdf(format='A4')
        await browser.close()
        
        return pdf_bytes
```

---

## 📂 Structure Projet

```
skillforge/
├── frontend/                    # React app
│   ├── src/
│   │   ├── components/
│   │   │   ├── LandingPage.jsx
│   │   │   ├── Dashboard.jsx
│   │   │   └── Editor.jsx
│   │   ├── api/
│   │   │   └── client.js       # Axios instance
│   │   ├── hooks/
│   │   │   └── useAuth.js
│   │   └── App.jsx
│   ├── public/
│   ├── package.json
│   └── vite.config.js
│
├── backend/                     # FastAPI app
│   ├── app/
│   │   ├── main.py             # FastAPI app entry
│   │   ├── auth.py             # OAuth + JWT
│   │   ├── models.py           # Pydantic models
│   │   ├── database.py         # MongoDB connection
│   │   ├── cv_generator.py     # AI generation
│   │   ├── pdf_generator.py    # PDF rendering
│   │   └── routers/
│   │       ├── auth.py
│   │       ├── templates.py
│   │       ├── cv.py
│   │       └── user.py
│   ├── requirements.txt
│   └── Dockerfile
│
├── docker-compose.yml
└── .env
```

---

## 🐳 Docker Configuration

### Dockerfile Frontend
```dockerfile
# Build stage
FROM node:18-alpine AS build
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

# Production stage
FROM nginx:alpine
COPY --from=build /app/dist /usr/share/nginx/html
COPY nginx.conf /etc/nginx/nginx.conf
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

### Dockerfile Backend
```dockerfile
FROM python:3.11-slim

# Install system deps for WeasyPrint (or Playwright)
RUN apt-get update && apt-get install -y \
    libpango-1.0-0 libpangocairo-1.0-0 \
    libgdk-pixbuf2.0-0 libffi-dev shared-mime-info \
    && rm -rf /var/lib/apt/lists/*

WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY ./app ./app

EXPOSE 8000
CMD ["uvicorn", "app.main:app", "--host", "0.0.0.0", "--port", "8000"]
```

### docker-compose.yml
```yaml
version: "3.9"

services:
  traefik:
    image: traefik:v3.2
    command:
      - --providers.docker=true
      - --entrypoints.web.address=:80
      - --entrypoints.websecure.address=:443
      - --certificatesresolvers.le.acme.tlschallenge=true
      - --certificatesresolvers.le.acme.email=${LETSENCRYPT_EMAIL}
      - --certificatesresolvers.le.acme.storage=/letsencrypt/acme.json
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock:ro
      - letsencrypt:/letsencrypt

  frontend:
    build: ./frontend
    labels:
      - traefik.enable=true
      - traefik.http.routers.frontend.rule=Host(`${APP_DOMAIN}`)
      - traefik.http.routers.frontend.entrypoints=websecure
      - traefik.http.routers.frontend.tls.certresolver=le

  backend:
    build: ./backend
    environment:
      - MONGODB_URI=${MONGODB_URI}
      - JWT_SECRET=${JWT_SECRET}
      - GOOGLE_CLIENT_ID=${GOOGLE_CLIENT_ID}
      - GOOGLE_CLIENT_SECRET=${GOOGLE_CLIENT_SECRET}
      - ANTHROPIC_API_KEY=${ANTHROPIC_API_KEY}
    labels:
      - traefik.enable=true
      - traefik.http.routers.backend.rule=Host(`${APP_DOMAIN}`) && PathPrefix(`/api`)
      - traefik.http.routers.backend.entrypoints=websecure
      - traefik.http.routers.backend.tls.certresolver=le

volumes:
  letsencrypt:
```

---

## 🔄 CORS Configuration

**Backend FastAPI :**
```python
from fastapi.middleware.cors import CORSMiddleware

app.add_middleware(
    CORSMiddleware,
    allow_origins=[
        "http://localhost:3000",  # Dev
        "https://skillforge.app"  # Prod
    ],
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)
```

---

## 🚀 Déploiement

### Build Images
```bash
# Frontend
cd frontend
docker build -t skillforge/frontend:latest .

# Backend
cd ../backend
docker build -t skillforge/backend:latest .
```

### GitHub Actions
```yaml
name: Deploy to Production

on:
  push:
    branches: [main]

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Build frontend
        run: |
          cd frontend
          docker build -t skillforge/frontend:latest .
          docker save skillforge/frontend:latest | gzip > frontend.tar.gz
      
      - name: Build backend
        run: |
          cd backend
          docker build --platform linux/arm64 -t skillforge/backend:latest .
          docker save skillforge/backend:latest | gzip > backend.tar.gz
      
      - name: Deploy to server
        run: |
          scp frontend.tar.gz backend.tar.gz ${{ secrets.SERVER_HOST }}:/tmp/
          ssh ${{ secrets.SERVER_HOST }} '
            cd /opt/skillforge
            docker load < /tmp/frontend.tar.gz
            docker load < /tmp/backend.tar.gz
            docker-compose up -d
          '
```

---

## ✅ Checklist Migration

- [x] Stack technique confirmée React + Python
- [x] docker-compose.yml mis à jour (2 services)
- [x] stack-technique.md mis à jour
- [x] api-routes-v2.md : changement auth NextAuth → JWT
- [ ] SYNTHESIS.md : mise à jour stack
- [ ] ADR.md : ajout décision React+Python
- [ ] cv-generation-prompt.md : exemples code Python
- [ ] User stories : ajout points setup CORS/JWT

---

**Note** : La maquette v0 (Next.js) reste référence **visuelle et UX** uniquement. Le code backend sera réécrit en Python FastAPI.
