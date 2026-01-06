# US-015 — En tant que système, je stocke le PDF et je marque le job comme terminé

## Bénéfice
Permettre le téléchargement fiable et traçable.

## Points
3

## Dépendances
- US-014

## Critères d’acceptation
- Le PDF est enregistré dans `PDF_STORAGE_DIR` (volume partagé app/worker) avec un nom unique (jobId).
- Le job est mis à jour : `status=done`, `result.pdfPath` (ou url), `finishedAt`.
- Si écriture échoue → job `failed`.

## Tests (scénarios)
- Après succès, `GET status` contient un `downloadUrl`.
- Fichier manquant → download renvoie 409/404.
