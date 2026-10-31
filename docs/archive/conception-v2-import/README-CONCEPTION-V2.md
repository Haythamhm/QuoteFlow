# Archive — note d'import de la Conception V2

Les documents V2 ont été intégrés dans `docs/` et sont maintenant les documents actifs décrits par `docs/00-INDEX.md`. Cette note est conservée pour son contexte d'origine ; ses anciennes instructions de copie ne s'appliquent plus.

Ce paquet **ne contient pas** de code backend, ni de frontend, ni de `.env`, ni de configuration Docker modifiée. Il contient seulement des **documents Markdown supplémentaires** à fusionner avec le dépôt QuoteFlow local existant.

## Installation SANS écraser le projet

1. Faire un commit ou une copie de sauvegarde du projet local actuel.
2. Décompresser ce paquet dans un répertoire temporaire.
3. Copier **uniquement le dossier `docs/`** du paquet dans le dossier `docs/` du projet QuoteFlow existant en fusionnant les nouveaux fichiers. Ce paquet ajoute les fichiers `00-INDEX-V2.md` et `06-...` à `13-...`, sans toucher aux fichiers `01` à `05`.
4. Garder les dossiers `frontend/` et `infra/` existants exactement comme ils sont. Aucun changement à `.env` ou aux bases.
5. Ouvrir le dépôt **racine** dans Codex Desktop, transmettre la consigne du fichier `docs/12-frontend-agents.md`.
6. Demander l'**audit en lecture seule** d'abord. Ne rien implémenter tant que le cadrage n'est pas validé par le développeur.

## Note sur AGENTS.md

Conserver l'`AGENTS.md` existant, puis ajouter à la fin les lignes suivantes (ou les fournir dans le premier prompt Codex) :

```markdown
## Référence V2
Avant toute analyse ou modification, lire `docs/00-INDEX-V2.md` et les documents `docs/06-...` à `docs/13-...`. Le backlog V2 `docs/10-backlog-sprints.md` précise les tickets. Relever et signaler les contradictions avec les documents 01–05 plutôt que de choisir seul. Aucun changement de code sans ticket et validation humaine.
```

## Prudence

L'état du dépôt utilisateur n'a pas été ouvert depuis son PC. Les fichiers V2 sont des **propositions** établies à partir du ZIP Blueprint précédemment partagé, du template frontend transmis et des captures Odoo. Leur contenu doit être revu et approuvé, et les résultats de builds/tests annoncés sur le PC restent à confirmer si nécessaires. La création API d'un devis brouillon n'est **pas encore démontrée** : c'est le prochain objectif du Sprint 1.
