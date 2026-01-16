# Schéma MongoDB (MVP)

## `users`
- `_id`
- `email` (unique index)
- `createdAt`
- `lastLoginAt`

Indexes:
- `email` unique

## `usage_counters` (quota)
- `_id`
- `userId`
- `month` (format `YYYY-MM`)
- `count` (int)
- `updatedAt`

Indexes:
- `{ userId: 1, month: 1 }` unique

## `cv_jobs`
- `_id`
- `userId`
- `templateId` : identifiant du template utilisé (ex: `modern-01`)
- `status` : `queued | running | done | failed`
- `createdAt`, `startedAt`, `finishedAt`
- `input` : payload normalisé du formulaire (MVP: stockage minimal, voire anonymisé)
- `result` :
  - `pdfPath` (si stockage local) ou `pdfUrl`
  - `expiresAt` (si lien signé/expiration)
- `error` : message court (si failed)

Indexes:
- `{ userId: 1, createdAt: -1 }`
- `{ status: 1, createdAt: 1 }` (polling worker)

## (Optionnel) `waitlist`
- `_id`
- `email`
- `createdAt`

Indexes:
- `email` unique
