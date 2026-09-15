# Pipeline Make — amélioration continue des prompts Pronto

Objectif : boucler automatiquement les corrections du Care (CSM/CCR) sur les drafts générés par Pronto vers des propositions concrètes d'amélioration des prompts (`prompts/layers/...`), sans jamais modifier un prompt sans validation humaine.

## Vue d'ensemble

```
Front (drafts IA + envoi final)
   │
   ▼
Google Sheet de log (déjà existant : logging macro)
   │  → ajouter : draft_ia, version_envoyee, domaine, commentaire_csm
   ▼
Scénario Make "Prompt Improver" (hebdomadaire)
   │
   ├─ 1. Récupère les lignes de la semaine (Google Sheets)
   ├─ 2. Regroupe par domaine (reglement-loyer, sinistres, ...)
   ├─ 3. Récupère le contenu actuel des prompts (module GitHub)
   ├─ 4. Appelle l'agent IA "amélioration" (meta-prompt dédié)
   ├─ 5. Parse la sortie JSON (résumé + propositions)
   ├─ 6. Crée une branche + commit des propositions (module GitHub)
   └─ 7. Notifie Louis (Front note interne / email) avec résumé + lien
   ▼
Louis relit et merge (ou rejette) sur GitHub — aucune règle n'entre en prod sans lui
```

## Détail des modules Make

### 1. Trigger

Schedule hebdomadaire (ex. lundi 8h), ou déclenché manuellement. Pas de trigger temps réel : l'agent a besoin d'un volume de cas pour distinguer une tendance d'un cas isolé (règle du meta-prompt : ≥2 occurrences).

### 2. Source des cas corrigés

Il faut que le Sheet de log existant contienne, par ticket :
- `domaine` (tag du ticket)
- `draft_ia` (ce que Pronto a généré)
- `version_envoyee` (le texte réellement envoyé par le CSM/CCR)
- `commentaire_csm` (optionnel — champ libre si le CSM veut expliquer sa correction)

Si ces colonnes n'existent pas encore, c'est le premier chantier avant de brancher ce scénario : sans `draft_ia` vs `version_envoyee`, l'agent n'a rien à comparer.

### 3. Lecture des prompts actuels

Module GitHub "Get a file" sur les 3 couches concernées par domaine :
- `prompts/layers/consignes.generales/Consignes.md`
- `prompts/layers/structuration-metier/{domaine}-metier.md`
- `prompts/layers/redaction-message/{domaine}-message.md`

### 4. Agent IA

Module OpenAI/Claude (celui déjà utilisé pour Pronto). Prompt système = `prompts/meta/agent-amelioration-prompts.md`. Variables injectées : les 3 contenus de prompts + la liste des cas groupés par domaine.

### 5. Sortie

JSON structuré (`resume`, `observations_mineures`, `propositions[]`) — cf. format défini dans le meta-prompt. Un module "Parse JSON" Make le rend directement exploitable pour les étapes suivantes.

### 6. Publication des propositions (jamais direct sur `main`)

Module GitHub :
- Crée une branche `prompt-improvements/{domaine}-W{numero_semaine}`.
- Pour chaque proposition avec `type: ajout | reformulation`, applique le texte proposé dans le fichier cible sur cette branche (pas sur main).
- Ouvre une Pull Request avec en description le `resume` + le détail des `propositions` (fichier, justification, nb de cas).

### 7. Notification

Note interne Front ou email à Louis : résumé + lien vers la PR GitHub. Il relit, ajuste si besoin, merge.

## Garde-fous à respecter

- L'agent ne touche jamais `main` directement — toujours une PR.
- Aucune proposition sans justification chiffrée (nb de cas).
- Si le volume de cas de la semaine est trop faible, le scénario doit pouvoir sortir sans PR (juste "rien à proposer cette semaine").
- Le format de style des prompts existants (1-2 phrases, prescriptif, une règle = une couche) doit être respecté par l'agent — c'est déjà documenté dans `Consignes.md`, on le rappelle dans le meta-prompt pour éviter la dérive.

## Prochaine étape

1. Ajouter les colonnes manquantes au Sheet de log (`draft_ia`, `version_envoyee`, `domaine`, `commentaire_csm`) si absentes.
2. Monter le scénario Make module par module (trigger → Sheets → GitHub read → IA → parse → GitHub PR → notification).
3. Tester sur un seul domaine (`reglement-loyer`) avant de généraliser à `sinistres` et aux futurs domaines.
