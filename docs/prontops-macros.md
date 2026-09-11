# Macros ProntOps — Front

## ProntOps RM - Feedbacks

### Fonctionnement

- **Type d'action** : macro manuelle. Un teammate la déclenche lui-même depuis le
  menu contextuel (les trois petits points) du **dernier message** d'un ticket.
- **Effet** : elle lance un prompt de type *"Collect teammate answers"* qui
  affiche un formulaire au teammate avec un champ **"Qualité du draft
  généré"** — une note sur la qualité du draft généré par l'IA pour ce
  message.
- **Objectif** : collecter du feedback humain sur les drafts générés par l'IA
  (ProntOps) afin de suivre leur qualité dans le temps.

### Pourquoi elle n'apparaît pas sur les conversations fermées/résolues

En pratique, la macro n'est disponible que sur des conversations dont le
statut est **ouvert**. Sur une conversation archivée/résolue (ou en spam/à la
corbeille), l'action n'apparaît pas dans le menu du message.

→ Ce n'est pas quelque chose à diagnostiquer plus loin au cas par cas : si un
teammate ne voit pas la macro, la première chose à vérifier est simplement le
statut de la conversation (voir checklist ci-dessous) avant de chercher une
autre cause.

### Process de vérification

Quand un teammate signale que la macro **"ProntOps RM - Feedbacks"**
n'apparaît pas, vérifier dans l'ordre :

1. **Statut de la conversation**
   - La conversation doit être **ouverte** (`open`).
   - Si elle est archivée/résolue, spam, ou dans la corbeille : la macro ne
     s'affiche pas. → Rouvrir la conversation si un feedback est encore
     nécessaire, ou considérer que le feedback n'est plus collectable.

2. **Inbox scoping**
   - La macro est scopée à l'inbox **Gestion Locative** uniquement.
   - Si le ticket concerné est dans une autre inbox, la macro n'apparaîtra
     pas, quel que soit son statut ou le teammate connecté.

3. **Teammate access**
   - Aucune restriction d'équipe sur cette macro : **tout teammate** ayant
     accès à l'inbox Gestion Locative peut la voir et l'exécuter.
   - Si un teammate ne la voit toujours pas alors que les points 1 et 2 sont
     valides, vérifier son accès à l'inbox elle-même (plutôt qu'un problème
     de permission propre à la macro).

| Vérification      | Condition attendue                          |
|--------------------|----------------------------------------------|
| Statut conversation | `open`                                       |
| Inbox               | Gestion Locative                             |
| Teammate access     | Any teammate (pas de restriction spécifique) |

### Destination des réponses collectées

Les réponses soumises via le prompt "Collect teammate answers" (dont la note
"Qualité du draft généré") sont envoyées à un scénario Make, qui les logue à
son tour dans un Google Sheet :

- **Scénario Make** : [eu1.make.com/9872/scenarios/6461501/edit](https://eu1.make.com/9872/scenarios/6461501/edit)
- **Google Sheet de log** : [1pnAYoW_UZNvMOHS65usedwp8gFQtjgmf-6NE-V3gu8Y](https://docs.google.com/spreadsheets/d/1pnAYoW_UZNvMOHS65usedwp8gFQtjgmf-6NE-V3gu8Y/edit?gid=0#gid=0)
