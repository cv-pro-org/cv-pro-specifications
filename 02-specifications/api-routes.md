# API Routes (MVP)

## Auth
- NextAuth routes standards

## Jobs CV
### `POST /api/cv/jobs`
Crée un job de génération.
- Auth: obligatoire
- Input: form payload (profil, expérience, etc.)
- Output: `{ jobId }`
- Errors: 401, 429 (rate/quota), 400

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
