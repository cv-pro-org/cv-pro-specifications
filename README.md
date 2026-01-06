# CV Instant Pro — Spécifications MVP (7 jours)

Objectif : livrer un MVP **production-ready** permettant à un utilisateur de se connecter (email magique), remplir un formulaire, lancer une génération, puis récupérer un **PDF très bien rendu**.

Décision clé (qualité PDF) : **HTML/CSS → PDF via Playwright (Chromium headless)**, généré **asynchrone** via un worker.

## Architecture documentaire
- `01-architecture/` : stack, schémas, docker-compose (Traefik + app + worker)
- `02-specifications/` : flows, règles métier, APIs
- `03-user-stories/` : sprint 1 & 2 + backlog
- `04-deployment/` : CI/CD, guide deploy, monitoring
