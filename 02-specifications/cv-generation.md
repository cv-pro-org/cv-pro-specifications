# Génération CV (Form → OpenAI → HTML → PDF)

## Objectif
Produire un PDF **pro** et stable via **Playwright**, de façon asynchrone.

## Décision clé
- LLM produit un **JSON strict** (schéma CV)
- Le template CV est une page HTML interne (`/internal/cv-render`) stylée (Tailwind + print CSS)
- Le worker Playwright génère le PDF avec `page.pdf({ format: 'A4', printBackground: true })`

## Flow MVP
1. User submit form → `POST /api/cv/jobs`
2. API:
   - vérifie session
   - vérifie quota + rate-limit
   - crée `cv_jobs(status=queued)`
   - retourne `jobId`
3. Front: écran "Génération en cours" + polling `GET /api/cv/jobs/:id`
4. Worker:
   - prend un job `queued` → `running`
   - appelle OpenAI → JSON CV
   - lance Playwright et charge `/internal/cv-render?jobId=...&token=...`
   - génère PDF et l’écrit dans `PDF_STORAGE_DIR`
   - update job `done` + chemin
   - envoie email "Votre PDF est prêt" (Resend) avec lien

## Stockage PDF (MVP)
- Stockage disque local (volume docker) : `PDF_STORAGE_DIR` partagé entre app & worker
- Lien de download : route sécurisée (`GET /api/cv/jobs/:id/download`) qui stream le fichier

## Règles d’erreur
- Si OpenAI échoue : job `failed` + message court + possibilité de relancer (nouveau job)
- Timeout génération : job `failed` après X minutes

## Contraintes ARM/Docker
- Prévoir image worker qui installe Chromium + deps Playwright sur ARM
- Alternative fallback : Puppeteer + Chromium système si Playwright image non compatible
