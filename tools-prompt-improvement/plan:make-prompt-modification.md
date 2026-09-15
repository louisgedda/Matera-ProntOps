```markdown
# Pipeline Make — amélioration continue des prompts Pronto (RM)

## Objectif

Boucler chaque semaine les corrections du Care (CSM/CCR) sur les drafts générés par les agents Pronto vers des propositions concrètes de modification des prompts (`prompts/layers/...`), avec validation humaine avant tout commit.

## Prérequis

Pour chaque ticket traité dans la semaine, doivent être loggés :
- `domaine` (ex. `reglement-loyer`, `sinistres`)
- `draft_ia` — le draft généré par le prompt en cours (Week N)
- `version_envoyee` — ce que le CSM ou le CCR a réellement envoyé
- `qui` — CSM ou CCR
- `commentaire` (optionnel)

Sans `draft_ia` déjà loggé au moment de l'envoi, rien à comparer : c'est le seul chantier data préalable.

## Étapes du scénario

### 1. Trigger

Hebdomadaire (vendredi ou lundi). Pas de temps réel : il faut un volume de cas pour distinguer une tendance d'un cas isolé.

### 2. Comparaison & classification (par ticket)

Iterator sur les lignes de la semaine. Pour chaque ligne, un module IA compare `draft_ia` vs `version_envoyee` et classe l'écart dans une des 5 catégories, plus un tag de nature de règle (`métier` ou `vocabulaire`) :

| # | Catégorie | Couche cible | Nature |
|---|---|---|---|
| 1 | Ton / registre / vocabulaire | `Consignes.md` ou `{domaine}-message.md` | vocabulaire |
| 2 | Fait métier faux, absent, ou mal appliqué (process, exception, chiffre) | `{domaine}-metier.md` | métier |
| 3 | Concision / structure / format HTML | `Consignes.md` | vocabulaire |
| 4 | Escalade ou routage mal géré (engagement non tenable, mauvaise transmission) | `Consignes.md` ou `{domaine}-message.md` | métier |
| 5 | Donnée mal interprétée ou hallucinée (chiffre/date absente des inputs) | `{domaine}-metier.md` | métier |

Sortie par ticket : `categorie`, `couche_cible`, `nature`, `commentaire` court. Écrit dans le Sheet.

### 3. Agrégation

Module Aggregator : regroupe toutes les lignes classifiées de la semaine, par domaine, en un tableau exploitable (ticket, catégorie, couche, nature, draft_ia, version_envoyee, commentaire).

### 4. Agent d'amélioration de prompt

Module IA n°2. Reçoit :
- le tableau agrégé de la semaine (par domaine),
- le contenu actuel des 3 prompts du domaine (`Consignes.md`, `{domaine}-metier.md`, `{domaine}-message.md`).

Règles :
- Ne propose une modification que si une catégorie se répète sur ≥ 2 cas, ou s'il s'agit d'une erreur grave (fait inventé, engagement non tenable) même isolée. Sinon : observation, pas de proposition.
- Une règle = une seule couche (celle indiquée par la catégorie).
- Connaissance factuelle métier → rédigée au présent descriptif, non taggée. Consigne de formulation/comportement → préfixée `[Answering Guideline]`.
- 1-2 phrases par règle proposée, généralisée à la classe de cas (jamais scénario-spécifique).
- Avant d'ajouter, vérifie qu'une règle existante ne couvre pas déjà le cas — proposer une reformulation plutôt qu'un ajout si oui.

Sortie : pour chaque fichier concerné, la modification exacte proposée + justification (nb de cas) + la nouvelle version complète du prompt.

### 5. Draft 2 — vérification avant déploiement

Iterator sur les mêmes tickets de la semaine, avec le nouveau prompt proposé (Week N+1), pour régénérer un `draft_2` par ticket. Comparaison `draft_2` vs `version_envoyee` : permet de vérifier que le nouveau prompt corrige effectivement les cas identifiés, avant tout commit.

### 6. Validation humaine

Résumé envoyé à Louis (note Front ou email) : catégories détectées, propositions par fichier, comparaison draft_ia / draft_2 / version_envoyee. Rien n'est committé sur GitHub sans validation explicite — le prompt Week N+1 ne remplace le Week N qu'après relecture.

## Garde-fous

- Aucune proposition sans seuil de répétition (≥ 2 cas) sauf faute grave.
- Jamais de commit direct sur `main` par l'automatisation — sortie = proposition à valider.
- Une catégorie = une couche cible fixe, pour éviter que l'agent d'amélioration ait à deviner où placer la règle.
- Le format des prompts existants (1-2 phrases, prescriptif, factuel vs `[Answering Guideline]`) doit être respecté par l'agent — déjà documenté dans `Consignes.md`.
```
