# ProntOps — Historique du projet

> Fichier vivant : à mettre à jour à chaque avancée (nouvelle décision, nouveau flow, changement d'architecture). Chaque entrée du journal est datée ; ne pas réécrire l'historique passé, seulement ajouter.

## 1. Objectif du projet

Atteindre 100 % des mails envoyés par les cares sans retouche : les drafts générés par IA sur Front doivent être directement exploitables. Le projet vise aussi à construire une boucle d'amélioration continue des prompts, pilotée par les retours des cares et la comparaison entre drafts et messages réellement envoyés.

## 2. Architecture actuelle (au 11/09/2026)

### 2.1 Sujets traités
- **Sinistres**
- **Règlements loyers**

### 2.2 Flow de création de draft (par sujet)
Pour chaque sujet, un scénario Make dédié :
1. Récupère l'historique de la conversation Front.
2. Applique un process défini via un tool.
3. Passe par un agent IA (Claude, module Make) qui prend en entrée le contexte de conversation + le contexte métier.
4. Génère un draft de réponse.
5. Tague 5 conversations par assignee pour qu'elles remontent dans une vue Front dédiée à la notation.
6. Met à jour :
   - le Gsheet de log des tickets ([lien](https://docs.google.com/spreadsheets/d/1pnAYoW_UZNvMOHS65usedwp8gFQtjgmf-6NE-V3gu8Y/edit?gid=0))
   - le Gsheet de suivi de notation du sujet ([sujet 1](https://docs.google.com/spreadsheets/d/1RDJvZdBM3_rj23fzSDFQZqIDb-hP3eTiyr9UohDqt94/edit?gid=0), [sujet 2](https://docs.google.com/spreadsheets/d/1EzpLuHUExxwHwmHqdHNvId3f7s_755h7-AcBl0Jioxc/edit?gid=0))

Déclencheur : [règle Front](https://app.frontapp.com/settings/tim:2307590/rules/edit/5651398).

### 2.3 Flow de feedback (global)
1. Le care ajoute la [macro](https://app.frontapp.com/settings/tim:2307590/rules/macros/edit/6086) sur la conversation.
2. Une [règle Front](https://app.frontapp.com/settings/tim:2307590/rules/edit/5651974) se déclenche et appelle un Make.
3. Le [Make de log des feedbacks](https://eu1.make.com/9872/scenarios/6461501/edit) écrit la note et la comparaison draft/version envoyée dans le Gsheet de log des tickets.

### 2.4 Inventaire Make actuel
- 1 Make « draft » par sujet (Sinistres, Règlements loyers) → 2 Makes
- 1 Make global de log des feedbacks
- **Total : 3 scénarios Make en production**

## 3. Chantiers en cours / à venir

Détail complet dans le Kanban Notion : [ProntOps - Kanban détaillé](https://app.notion.com/p/3d86513772f8810592aacfb678c2068a).

Résumé des axes :
1. **Data & comparaison** — construire un dataset massif draft vs message réellement envoyé, pour nourrir l'amélioration des prompts.
2. **Architecture agents** — scinder l'agent unique de chaque flow en 3 agents spécialisés : récupération historique Front, contexte métier, rédaction du draft.
3. **Amélioration des prompts** — un Make d'entraînement par sujet, exécuté chaque fin de semaine, alimenté par les données de la semaine, piloté par un agent « améliorateur de prompt » (MD + skill dédiés).
4. **Versioning des prompts** — logger le prompt utilisé chaque semaine (Excel ou Notion), avec sélection manuelle en début de semaine après analyse.
5. **Réutilisation Pronto** — trier et adapter les prompts fournis par Yann (équipe Data), utilisés sur le projet Pronto, en écartant ce qui est trop lié à leur contexte métier.
6. **Documentation** — MD de process par flow + ce fichier d'historique global.
7. **Optimisation MCP/tools** — connecter directement les agents Make à des MCP/tools plutôt que de la logique custom.
8. **Traçabilité GitHub** — ce repo, comme source d'historique complet du projet (prompts versionnés, MD, logs).
9. **Reporting** — reporting hebdomadaire du nombre de tickets traités et de la note moyenne, par sujet.

## 4. Décisions prises

| Date | Décision |
|------|----------|
| 11/09/2026 | Le rythme d'ajustement des prompts sera hebdomadaire (fin de semaine), pas à chaque envoi, pour éviter le sur-ajustement sur un seul feedback. |
| 11/09/2026 | Chaque flow de création de draft sera scindé en 3 agents spécialisés plutôt qu'un agent monolithique. |
| 11/09/2026 | Le choix du prompt à utiliser chaque semaine reste une décision manuelle, basée sur une analyse humaine en début de semaine. |
| 11/09/2026 | Le suivi historique du projet est stocké dans ce repo GitHub (louisgedda/Matera-ProntOps), fichier `docs/historique.md`. |

## 5. Journal d'avancement

### 11/09/2026
- Première formalisation du plan d'action et création d'un premier Kanban Notion (axes : feedback cares, qualité des prompts, suivi global).
- Kanban initial supprimé accidentellement, recréé à l'identique.
- Description complète de l'architecture réelle du projet (2 Makes de draft par sujet + 1 Make global de feedback, Gsheets associés, règles et macro Front).
- Définition du plan détaillé : dataset de comparaison draft/envoyé, séparation en 3 agents par flow, boucle hebdomadaire d'amélioration des prompts, versioning des prompts, réutilisation des prompts Pronto, documentation MD, optimisation MCP/tools, repo GitHub, reporting hebdomadaire.
- Nouveau Kanban Notion détaillé créé (22 cartes, 9 axes, propriétés Sujet/Axe/Priorité/Lien) : [lien](https://app.notion.com/p/3d86513772f8810592aacfb678c2068a).
- Connexion du repo GitHub `louisgedda/Matera-ProntOps` et migration de ce fichier d'historique dans `docs/historique.md`.

## 6. Liens de référence

- Kanban Notion détaillé : https://app.notion.com/p/3d86513772f8810592aacfb678c2068a
- Gsheet log tickets : https://docs.google.com/spreadsheets/d/1pnAYoW_UZNvMOHS65usedwp8gFQtjgmf-6NE-V3gu8Y/edit?gid=0
- Gsheet notation sujet 1 : https://docs.google.com/spreadsheets/d/1RDJvZdBM3_rj23fzSDFQZqIDb-hP3eTiyr9UohDqt94/edit?gid=0
- Gsheet notation sujet 2 : https://docs.google.com/spreadsheets/d/1EzpLuHUExxwHwmHqdHNvId3f7s_755h7-AcBl0Jioxc/edit?gid=0
- Règle Front (trigger draft) : https://app.frontapp.com/settings/tim:2307590/rules/edit/5651398
- Règle Front (trigger feedback) : https://app.frontapp.com/settings/tim:2307590/rules/edit/5651974
- Macro Front (feedback) : https://app.frontapp.com/settings/tim:2307590/rules/macros/edit/6086
- Make log feedback : https://eu1.make.com/9872/scenarios/6461501/edit
