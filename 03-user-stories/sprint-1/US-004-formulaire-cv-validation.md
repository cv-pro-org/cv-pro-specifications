# US-006 — En tant qu’utilisateur, je remplis un formulaire CV avec validation pour lancer une génération

## Bénéfice
Collecter des données propres pour générer un CV de qualité.

## Points
5

## Dépendances
- US-005

## Critères d’acceptation
- Le formulaire (sur `/generate`) collecte au minimum :
  - identité : prénom/nom, titre (ex: “Product Manager”), email affiché (readonly)
  - résumé (texte)
  - expériences (liste : entreprise, rôle, dates, bullets)
  - formation (liste)
  - compétences (liste)
- Validation front : champs requis, limites de taille (anti payload énorme).
- Validation back (API) via schéma (ex: zod) ; refuse `400` en cas de payload invalide.
- Le submit déclenche la création d’un job (US-007) et affiche un écran “Génération en cours”.

## Tests (scénarios)
- Submit avec champs requis manquants → erreurs UI + `400` si bypass.
- Submit valide → job créé et UI passe en “en cours”.
