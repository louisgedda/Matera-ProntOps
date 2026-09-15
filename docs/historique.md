# ProntOps — Historique du projet

Fichier vivant : à mettre à jour à chaque avancée (nouvelle décision, nouveau flow, changement d'architecture). Chaque entrée du journal est datée ; ne pas réécrire l'historique passé, seulement ajouter.

## 1. Objectif du projet

Atteindre 100 % des mails envoyés par les cares sans retouche : les drafts générés par IA sur Front doivent être directement exploitables. Le projet vise aussi à construire une boucle d'amélioration continue des prompts, pilotée par les retours des cares et la comparaison entre drafts et messages réellement envoyés.

## 2. Architecture actuelle (au 11/09/2026)

### 2.1 Sujets traités

- Sinistres
- Règlements loyers

### 2.2 Flow de création de draft (par sujet)

Pour chaque sujet, un scénario Make dédié :

1. Récupère l'historique de la conversation Front.
2. Applique un process défini via un tool.
3. Passe par un agent IA (Claude, module Make) qui prend en entrée le contexte de conversation + le contexte métier.
4. Génère un draft de réponse.
5. Tague 5 conversations par assignee pour qu'elles remontent dans une vue Front dédiée à la notation.
6. Met à jour :
   - le Gsheet de log des tickets (lien)
   - le Gsheet de suivi de notation du sujet (sujet 1, sujet 2)

Déclencheur : règle Front.

### 2.3 Flow de feedback (global)

- Le care ajoute la macro sur la conversation.
- Une règle Front se déclenche et appelle un Make.
- Le Make de log des feedbacks écrit la note et la comparaison draft/version envoyée dans le Gsheet de log des tickets.

### 2.4 Inventaire Make actuel

- 1 Make « draft » par sujet (Sinistres, Règlements loyers) → 2 Makes
- 1 Make global de log des feedbacks

Total : 3 scénarios Make en production.

## 3. Chantiers en cours / à venir

Détail complet dans le Kanban Notion : ProntOps - Kanban détaillé.

Résumé des axes :

- **Data & comparaison** — construire un dataset massif draft vs message réellement envoyé, pour nourrir l'amélioration des prompts.
- **Architecture agents** — scinder l'agent unique de chaque flow en 3 agents spécialisés : récupération historique Front, contexte métier, rédaction du draft.
- **Amélioration des prompts** — un Make d'entraînement par sujet, exécuté chaque fin de semaine, alimenté par les données de la semaine, piloté par un agent « améliorateur de prompt » (MD + skill dédiés). *(Mise à jour le 15/09/2026 : architecture en 3 agents conçue et rédigée dans `tools-prompt-improvement/` — voir Journal d'avancement ci-dessous. Le montage effectif du scénario Make reste à faire.)*
- **Versioning des prompts** — logger le prompt utilisé chaque semaine (Excel ou Notion), avec sélection manuelle en début de semaine après analyse. *(Mise à jour le 15/09/2026 : ce chantier n'est plus porté par Excel/Notion mais directement par le repo GitHub `Matera-ProntOps` — voir l'entrée du 15/09/2026 dans le Journal d'avancement ci-dessous pour le détail de l'implémentation.)*
- **Réutilisation Pronto** — trier et adapter les prompts fournis par Yann (équipe Data), utilisés sur le projet Pronto, en écartant ce qui est trop lié à leur contexte métier. *(Mise à jour le 15/09/2026 : premier tri fait — voir Journal d'avancement.)*
- **Documentation** — MD de process par flow + ce fichier d'historique global.
- **Optimisation MCP/tools** — connecter directement les agents Make à des MCP/tools plutôt que de la logique custom.
- **Traçabilité GitHub** — ce repo, comme source d'historique complet du projet (prompts versionnés, MD, logs).
- **Reporting** — reporting hebdomadaire du nombre de tickets traités et de la note moyenne, par sujet.

## 4. Décisions prises

| Date | Décision |
|---|---|
| 11/09/2026 | Le rythme d'ajustement des prompts sera hebdomadaire (fin de semaine), pas à chaque envoi, pour éviter le sur-ajustement sur un seul feedback. |
| 11/09/2026 | Chaque flow de création de draft sera scindé en 3 agents spécialisés plutôt qu'un agent monolithique. |
| 11/09/2026 | Le choix du prompt à utiliser chaque semaine reste une décision manuelle, basée sur une analyse humaine en début de semaine. |
| 11/09/2026 | Le suivi historique du projet est stocké dans ce repo GitHub (louisgedda/Matera-ProntOps), fichier docs/historique.md. |
| 15/09/2026 | Le repo GitHub `Matera-ProntOps` accueille désormais une arborescence `prompts/` dédiée, distincte de `docs/` : chaque sujet est découpé en 2 fichiers réutilisables (façon de parler + structuration métier), plus un fichier d'instructions globales partagé par tous les sujets. |
| 15/09/2026 | Convention de nommage figée : `prompts/layers/redaction-message/<sujet>-message.md` (façon de parler, spécifique au sujet), `prompts/layers/structuration-metier/<sujet>-metier.md` (connaissance métier factuelle, spécifique au sujet), `prompts/layers/consignes.generales/Consignes.md` (partagé, quasiment jamais modifié). |
| 15/09/2026 | Numérotation des semaines de suivi en `W1`, `W2`... calculée depuis une date de départ de projet (2026-09-14 = semaine 1), plutôt que le numéro de semaine calendaire ISO — pour que la numérotation démarre à 1 avec le projet, pas avec l'année. |
| 15/09/2026 | Un script d'assemblage (`assemble_weekly_prompt.py`) concatène chaque semaine, pour un sujet donné, les 3 fichiers (instructions globales + façon de parler + structuration métier) en un seul fichier figé `prompts/weekly/<sujet>-W<n>.md`, avec traçabilité des versions sources (SHA GitHub) dans l'en-tête. Relancer le script sur un sujet déjà traité la même semaine met à jour le fichier existant plutôt que d'en créer un doublon. |
| 15/09/2026 | La brique "historique de conversation" (3e agent envisagé initialement) est mise en pause : non intégrée à l'assemblage pour l'instant, réintégrable plus tard sans changer l'architecture. |
| 15/09/2026 | La boucle d'amélioration hebdomadaire des prompts est conçue en 3 agents distincts (classification, correction, assemblage), plutôt qu'un agent unique qui ferait tout — chaque agent a une seule responsabilité, dans l'esprit "une règle = une seule couche" déjà appliqué aux prompts eux-mêmes. |
| 15/09/2026 | Taxonomie figée de 5 axes de classification des écarts draft/envoi (ton-registre-vocabulaire, concision-structure-format, règle métier/process, donnée mal interprétée/hallucinée, comportement face au cas/escalade), chacun pré-mappé à une couche cible (`Consignes.md`, `-metier.md`, ou `-message.md`) et une nature (métier ou vocabulaire) — pour que l'agent de correction n'ait jamais à deviner où placer une règle. |
| 15/09/2026 | Une proposition de modification de prompt n'est retenue que si l'écart se répète sur au moins 2 cas dans la semaine, sauf faute grave (fait inventé, engagement non tenable) qui peut être retenue même isolée — pour éviter le sur-ajustement sur un cas unique. |
| 15/09/2026 | Aucune modification de prompt n'est appliquée ou committée automatiquement : chaque étape (classification → correction → assemblage → régénération de test) reste une proposition, validée par Louis avant tout commit sur `main`. |
| 15/09/2026 | Réutilisation ciblée de la méthodologie du projet Pronto (équipe Data) : la boucle de review en étapes, la taxonomie de causes racines, l'ordre de préférence des types de fix, et la philosophie d'écriture des prompts sont repris. Ce qui est spécifique à leur stack (Langfuse, CLI de replay, api-core, agents finance/comptabilité/AG/mutation propres à la Copropriété) n'est pas repris — hors périmètre de la Gestion Locative. |

## 5. Journal d'avancement

### 11/09/2026

- Première formalisation du plan d'action et création d'un premier Kanban Notion (axes : feedback cares, qualité des prompts, suivi global).
- Kanban initial supprimé accidentellement, recréé à l'identique.
- Description complète de l'architecture réelle du projet (2 Makes de draft par sujet + 1 Make global de feedback, Gsheets associés, règles et macro Front).
- Définition du plan détaillé : dataset de comparaison draft/envoyé, séparation en 3 agents par flow, boucle hebdomadaire d'amélioration des prompts, versioning des prompts, réutilisation des prompts Pronto, documentation MD, optimisation MCP/tools, repo GitHub, reporting hebdomadaire.
- Nouveau Kanban Notion détaillé créé (22 cartes, 9 axes, propriétés Sujet/Axe/Priorité/Lien) : lien.
- Connexion du repo GitHub louisgedda/Matera-ProntOps et migration de ce fichier d'historique dans docs/historique.md.

### 15/09/2026

- Mise en place de l'arborescence `prompts/` sur le repo GitHub, en complément de `docs/` : séparation claire entre les briques de prompt réutilisables (`prompts/layers/`) et leurs assemblages hebdomadaires figés (`prompts/weekly/`).
- Rédaction et intégration du contenu des deux premiers sujets :
  - **Sinistres & Travaux** (agent orienté BAILLEURS) : classification par tags Front (signalement, devis, garantie/assurance, relance), garde-fous financiers, et base de connaissance métier dédiée (répartition locataire/bailleur selon le décret n°87-712, degrés d'urgence Matera, garanties GPA/biennale/décennale-DO, règles dégât des eaux, process travaux).
  - **Solde locataire & règlement loyer** (agent orienté LOCATAIRES) : classification par cas (accusé de réception, question factuelle sur solde/quittance, litige/anomalie), garde-fous propres au solde locataire.
- Écriture du script `assemble_weekly_prompt.py` (Python, API GitHub Contents — pas de clone git local) : lit les 3 fichiers sources d'un sujet, les concatène avec un en-tête de traçabilité (SHA des sources), et commit le résultat dans `prompts/weekly/<sujet>-W<n>.md`. Testé et validé en conditions réelles sur les sujets Sinistres et Règlement loyer.
- Plusieurs itérations sur la convention de nommage des fichiers (suffixes `-message.md` / `-metier.md`, dossier `consignes.generales/Consignes.md`) pour que le script puisse construire les chemins de façon prévisible à partir du seul nom du sujet.
- Nettoyage du premier essai généré avec l'ancienne numérotation calendaire (`sinistres-2026-W38.md`), supprimé au profit de la convention `W1`.
- **Conception de la boucle d'amélioration hebdomadaire des prompts**, avec Claude : analyse de 3 MD internes du projet Pronto (équipe Data — `pronto_simulation_review`, `pronto_architecture`, `pronto_tool_authoring`), tri de ce qui est réutilisable (boucle de review en étapes, taxonomie de causes racines, ordre de préférence des fixs, philosophie d'écriture des prompts, distinction connaissance factuelle vs `[Answering Guideline]`, principe d'auto-amélioration, calibration de confiance) vs ce qui est spécifique à leur stack (Langfuse, CLI `pronto_replay`, api-core, agents métier Copropriété).
- Définition de la taxonomie des 5 axes de classification des écarts draft/envoi, chacun pré-mappé à une couche cible et une nature (métier/vocabulaire), et du seuil de répétition (≥ 2 cas, sauf faute grave) avant de transformer un écart en proposition de règle.
- Rédaction de 4 fichiers dans `tools-prompt-improvement/` :
  - `agent-comparaison-classification.md` — Agent 1 : compare `draft_ia` et `version_envoyee` par ticket, classe l'écart selon les 5 axes, filtre les cas non-actionnables.
  - `agent-correction-prompt.md` — Agent 2 : reçoit le tableau agrégé des écarts classifiés de la semaine + le contenu actuel des 3 prompts du domaine, propose des modifications ciblées (ajout/reformulation/suppression) avec justification et nombre de cas concernés.
  - `agent-assemblage.md` — Agent 2bis : applique les propositions validées au contenu actuel d'un fichier pour produire la version candidate complète, sans toucher à rien d'autre que les passages ciblés.
  - `plan-make-prompt-modification.md` — schéma global du scénario Make (trigger hebdo → comparaison/classification → agrégation → correction → assemblage → régénération de draft de test → validation humaine → commit).
- Reste à faire : monter concrètement le scénario Make (les 4 fichiers sont les prompts des modules IA, pas encore branchés), et l'agent de régénération de draft de test (Agent 3) n'est pas encore rédigé.

## 6. Liens de référence

- Kanban Notion détaillé : https://app.notion.com/p/3d86513772f8810592aacfb678c2068a
- Gsheet log tickets : https://docs.google.com/spreadsheets/d/1pnAYoW_UZNvMOHS65usedwp8gFQtjgmf-6NE-V3gu8Y/edit?gid=0
- Gsheet notation sujet 1 : https://docs.google.com/spreadsheets/d/1RDJvZdBM3_rj23fzSDFQZqIDb-hP3eTiyr9UohDqt94/edit?gid=0
- Gsheet notation sujet 2 : https://docs.google.com/spreadsheets/d/1EzpLuHUExxwHwmHqdHNvId3f7s_755h7-AcBl0Jioxc/edit?gid=0
- Règle Front (trigger draft) : https://app.frontapp.com/settings/tim:2307590/rules/edit/5651398
- Règle Front (trigger feedback) : https://app.frontapp.com/settings/tim:2307590/rules/edit/5651974
- Macro Front (feedback) : https://app.frontapp.com/settings/tim:2307590/rules/macros/edit/6086
- Make log feedback : https://eu1.make.com/9872/scenarios/6461501/edit
- Boucle d'amélioration des prompts (agents) : `tools-prompt-improvement/agent-comparaison-classification.md`, `tools-prompt-improvement/agent-correction-prompt.md`, `tools-prompt-improvement/agent-assemblage.md`, `tools-prompt-improvement/plan-make-prompt-modification.md`
