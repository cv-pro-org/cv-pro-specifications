# US-020 — En tant qu’opérateur, je déploie automatiquement via GitHub Actions (build + SSH deploy)

## Bénéfice
Itérer vite et fiablement.

## Points
5

## Dépendances
- US-019

## Critères d’acceptation
- Un workflow GitHub Actions :
  - build images `app` et `worker`
  - push registry (GHCR)
  - SSH sur serveur et exécute `docker compose pull && docker compose up -d`
- Support rollback MVP : re-déployer un tag précédent.
- Secrets listés dans la doc.

## Tests (scénarios)
- Push sur `main` → pipeline s’exécute.
- Serveur unreachable → job échoue clairement.
