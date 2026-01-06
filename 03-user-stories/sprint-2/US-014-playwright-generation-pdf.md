# US-014 — En tant que système, je génère un PDF via Playwright (Chromium) à partir du template HTML

## Bénéfice
Obtenir un PDF typographiquement stable et esthétique.

## Points
8

## Dépendances
- US-013

## Critères d’acceptation
- Le worker lance Playwright/Chromium (compatible ARM) et génère un PDF via `page.pdf()`.
- Paramètres minimum : A4, `printBackground: true`, marges contrôlées.
- Fonts : les polices nécessaires sont disponibles dans l’image docker (évite fallback moche).
- Timeout de génération configuré (ex: 60–120s).

## Tests (scénarios)
- Génération sur un payload simple → PDF non vide et lisible.
- Payload long → PDF 2 pages, pas de chevauchement.
