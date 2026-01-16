# SkillForge — Spécifications MVP (7 jours)

**Renommage** : La plateforme se nomme désormais **SkillForge** (anciennement CV Instant Pro).

## Objectif MVP

Livrer un MVP **production-ready** permettant à un utilisateur de :
1. **Se connecter** (email magique)
2. **Prompter son CV** en langage naturel (nouvelle approche : pas de formulaire structuré)
3. **Personnaliser** via un éditeur visuel (templates, couleurs, polices)
4. **Télécharger** un PDF professionnel haute qualité

## Innovation principale : Prompt-Driven CV Generation

**Rupture UX** : Au lieu de remplir un formulaire structuré, l'utilisateur **décrit son parcours en langage naturel** :
- Textarea libre : "Je suis développeur full-stack avec 5 ans d'expérience..."
- Option **CV ciblé** : coller une annonce LinkedIn/Indeed pour optimiser le CV pour ce poste
- Upload de CV existant pour amélioration/réformatage
- Saisie vocale (Web Speech API)

L'IA (OpenAI) structure automatiquement les données en sections CV (expériences, formation, compétences...).

## Architecture documentaire
- `01-architecture/` : stack, schémas, docker-compose (Traefik + app)
- `02-specifications/` : flows, règles métier, APIs, prompt generation
- `03-user-stories/` : sprint 1 & 2 + backlog
- `04-deployment/` : CI/CD, guide deploy, monitoring
- `05-design/` : maquette fonctionnelle v0 (référence visuelle)
