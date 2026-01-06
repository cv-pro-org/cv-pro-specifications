# Diagramme système — CV Instant Pro

## Architecture globale

```mermaid
flowchart TB
    subgraph Internet
        U[Utilisateur]
        G[Google OAuth]
    end

    subgraph Infra["Oracle Linux ARM (Docker)"]
        T[Traefik<br/>SSL + Routing]
        
        subgraph App["Next.js App"]
            FE[Pages React<br/>Tailwind]
            API[API Routes]
            AUTH[NextAuth<br/>Google Provider]
            RENDER[/internal/cv-render]
        end
        
        W[Worker<br/>Playwright + Chromium]
        PDF[(Volume PDFs)]
    end
    
    subgraph External
        MONGO[(MongoDB Atlas)]
        RESEND[Resend API]
        OPENAI[OpenAI API]
    end

    U -->|HTTPS| T
    T --> FE
    T --> API
    
    U <-->|OAuth| G
    AUTH <-->|Verify| G
    
    API <-->|Session/Users/Jobs| MONGO
    
    W -->|Poll jobs| MONGO
    W -->|Generate text| OPENAI
    W -->|GET HTML| RENDER
    W -->|Write| PDF
    W -->|Job done email| RESEND
    
    API -->|Stream| PDF
```

---

## Flux utilisateur complet (Google OAuth)

```mermaid
sequenceDiagram
    autonumber
    actor U as Utilisateur
    participant FE as Frontend
    participant API as API Routes
    participant G as Google OAuth
    participant DB as MongoDB
    participant W as Worker
    participant AI as OpenAI
    participant PW as Playwright
    participant R as Resend

    %% Auth Google
    U->>FE: Clique "Connexion Google"
    FE->>G: Redirect OAuth
    G-->>U: Consent screen
    U->>G: Accepte
    G-->>FE: Callback + code
    FE->>API: GET /api/auth/callback/google
    API->>G: Exchange code → tokens
    G-->>API: Access token + user info
    API->>DB: Upsert user (email, name, image)
    API-->>FE: Session cookie
    FE-->>U: Redirect /generate

    %% Génération
    U->>FE: Remplit formulaire (name pré-rempli)
    U->>FE: Clique "Générer"
    FE->>API: POST /api/cv/jobs
    API->>DB: Check quota (usage_counters)
    alt Quota OK
        API->>DB: Insert cv_job (queued)
        API-->>FE: 202 { jobId }
        FE-->>U: Écran "en cours"
        
        %% Worker async
        W->>DB: findOneAndUpdate (queued→running)
        W->>AI: Prompt → JSON CV
        AI-->>W: JSON structuré
        W->>API: GET /internal/cv-render?jobId=...
        API-->>W: HTML template
        W->>PW: page.pdf()
        PW-->>W: PDF bytes
        W->>DB: Update job (done, pdfPath)
        W->>DB: Incr usage_counters
        W->>R: Email "PDF prêt"
        R-->>U: Email notification
        
        %% Download
        U->>FE: Clique lien ou poll
        FE->>API: GET /api/cv/jobs/:id
        API-->>FE: { status: done, downloadUrl }
        U->>FE: Clique télécharger
        FE->>API: GET /api/cv/jobs/:id/download
        API-->>U: Stream PDF
    else Quota atteint
        API-->>FE: 403 QUOTA_REACHED
        FE-->>U: Redirect /upgrade
    end
```

---

## États du job

```mermaid
stateDiagram-v2
    [*] --> queued: POST /api/cv/jobs
    queued --> running: Worker pick
    running --> done: PDF généré
    running --> failed: Erreur (OpenAI/Playwright)
    failed --> queued: Retry manuel (nouveau job)
    done --> [*]
```

---

## Composants Docker

```mermaid
flowchart LR
    subgraph docker-compose
        traefik["traefik:v3.2<br/>:80/:443"]
        app["cv-pro/app<br/>Next.js :3000"]
        worker["cv-pro/worker<br/>Node + Playwright"]
        vol_pdf[("pdfs<br/>volume")]
        vol_ssl[("letsencrypt<br/>volume")]
    end
    
    traefik -->|proxy| app
    worker -->|HTTP interne| app
    app --> vol_pdf
    worker --> vol_pdf
    traefik --> vol_ssl
```

---

## Sécurité résumée

```mermaid
flowchart TD
    A[Request] --> B{Session?}
    B -->|Non + route protégée| C[Redirect /login]
    B -->|Oui| D{Owner?}
    D -->|Non| E[403 / 404]
    D -->|Oui| F{Quota?}
    F -->|Dépassé| G[403 QUOTA_REACHED]
    F -->|OK| H{Rate limit?}
    H -->|Bloqué| I[429 RATE_LIMITED]
    H -->|OK| J[Process request]
```
