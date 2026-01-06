# US-001 — En tant que visiteur, je consulte une landing claire pour comprendre le produit et démarrer

## Bénéfice
Comprendre la proposition de valeur et accéder rapidement au générateur.

## Points
2

## Dépendances
Aucune.

## Critères d’acceptation
- La landing est accessible sur `/`.
- Elle contient :
  - un titre + sous-titre (promesse)
  - un aperçu visuel (mock ou screenshot) du rendu CV
  - un CTA principal “Générer mon CV” menant vers `/generate` (ou login si non connecté)
  - un bloc “Comment ça marche” (3 étapes max)
- Responsive (mobile/desktop) sans débordement.
- Temps de chargement raisonnable (pas de dépendance lourde côté client pour le MVP).

## Tests (scénarios)
- Ouvrir `/` sur mobile : CTA visible sans scroll excessif.
- Cliquer CTA : redirection vers `/generate` (ou vers login si requis).
