# US-019 — En tant qu’opérateur, je déploie app + worker + Traefik via docker-compose

## Bénéfice
Déploiement reproductible en une commande.

## Points
3

## Dépendances
Aucune.

## Critères d’acceptation
- `docker-compose.yml` inclut : traefik, app, worker, volumes (pdfs, letsencrypt).
- Les variables d’environnement nécessaires sont documentées.
- Le routage Traefik HTTPS fonctionne (certresolver Let’s Encrypt).

## Tests (scénarios)
- `docker compose up -d` démarre les 3 services.
- Accès HTTPS au domaine → page OK.
