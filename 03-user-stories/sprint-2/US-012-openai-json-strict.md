# US-012 — En tant que système, je transforme le formulaire en JSON CV strict via OpenAI

## Bénéfice
Rendre le rendu HTML/PDF prévisible et éviter les sorties non exploitables.

## Points
5

## Dépendances
- US-011

## Critères d’acceptation
- Définir un schéma JSON (ex: `CvDocument`) incluant : header, expériences, éducation, skills.
- Le prompt force une sortie JSON uniquement (pas de texte).
- La réponse est validée côté serveur (zod/jsonschema) :
  - si invalide → job `failed` avec code erreur `INVALID_LLM_OUTPUT`.
- Les champs texte sont nettoyés (trim), tailles limitées.

## Tests (scénarios)
- Forcer une réponse non-JSON → validation échoue → job `failed`.
- Réponse conforme → passe à l’étape rendu HTML.
