# US-013 — En tant que système, je dispose d’une route interne HTML “CV template” imprimable

## Bénéfice
Avoir un rendu pixel-perfect en PDF via Chromium.

## Points
5

## Dépendances
- US-012

## Critères d’acceptation
- Route interne `GET /internal/cv-render?jobId=...&token=...` retourne une page HTML complète.
- Le template utilise Tailwind + styles print (A4, marges, sauts de page).
- Sécurité : accès uniquement avec un token interne (HMAC) et/ou réseau interne docker.
- Le template supporte au moins :
  - 1 page A4 (et 2 pages si contenu long)
  - gestion basique des sauts de section

## Tests (scénarios)
- Appel sans token → 401/403.
- Appel avec token valide → HTML rendu.
