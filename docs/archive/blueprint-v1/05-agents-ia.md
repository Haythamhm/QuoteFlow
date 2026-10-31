# 05 — Mode de travail avec Codex Desktop et Gemini 3.8 Flash

## Deux usages distincts de l'IA

- **Agents de programmation** : Codex Desktop pour implémenter ; Gemini CLI/outil Gemini pour avis indépendant, revue et cas limites. Le modèle Gemini documenté est `gemini-3.8-flash`. Cela **ne signifie pas** que l'on peut automatiquement le choisir dans Codex Desktop.
- **IA métier à l'intérieur de QuoteFlow** : étape optionnelle de Sprint 8, par exemple LM Studio + Qwen3. Elle intervient seulement pour **proposer** des extractions.

## Attribution des rôles

| Participant | Mission |
|---|---|
| Développeur humain | Décisions métier, choix technologiques, accès Odoo, test manuel, revue du diff, validation de sprint |
| Codex Desktop | Implémentation par petit ticket, tests, documentation et explication des choix |
| Gemini 3.8 Flash | Revue critique indépendante, cas limites, identification de failles RBAC, préparation de scénarios de test |

N'autoriser **qu'un agent en écriture sur une branche à un moment donné**. Préférer Gemini en lecture seule pour les revues. Ne jamais accorder un accès illimité au shell/à des secrets.

## Règles du cycle de travail

1. Ouvrir une branche `sprint-N/ticket-court`.
2. Préparer un ticket : contexte, fichiers autorisés, critères d'acceptation, tests attendus.
3. Demander à Codex d'expliquer le plan avant de coder ; valider le plan.
4. Exécuter les tests et vérifier le diff.
5. Demander à Gemini une revue **du diff**, notamment sécurité, permissions, logique ERP et tests manquants.
6. Corriger dans Codex, re-tester et vérifier soi-même dans Odoo/React.
7. Commit avec un message décrivant le ticket ; marquer le sprint terminé uniquement après preuve.

## Prompt réutilisable pour Codex

> Tu développes QuoteFlow, un projet solo d'apprentissage. Lis AGENTS.md, README.md, docs/01-cadrage.md, docs/02-architecture.md et docs/03-sprints.md. Nous travaillons sur [SPRINT/TICKET]. Ne modifie que [FICHIERS/PÉRIMÈTRE]. Commence par proposer une approche et les tests, sans coder immédiatement. Après mon accord : implémente le minimum nécessaire, n'introduis aucun framework ou microservice non prévu, exécute build/tests, puis fournis le diff résumé, les commandes de validation et les limites restantes. Toute création Odoo doit rester au statut brouillon et nécessiter une approbation métier préalable.

## Prompt réutilisable pour Gemini

> Agis comme reviewer senior .NET, sécurité RBAC et intégration Odoo. Lis les contraintes de QuoteFlow dans AGENTS.md. Examine le diff du ticket [SPRINT/TICKET] **sans modifier les fichiers**. Classe les problèmes par criticité : sécurité/secret, règles métier, architecture, fiabilité Odoo, tests manquants, simplicité. Fournis des corrections vérifiables et évite les recommandations disproportionnées pour un développeur seul.

## Installation Gemini CLI, si nécessaire

```powershell
npm install -g @google/gemini-cli
# lancement et authentification :
gemini
# tester le modèle s'il est disponible sur le compte et la version installée :
gemini -m gemini-3.8-flash
```

Ne suppose pas que l'abonnement ChatGPT ou Gemini donne gratuitement accès à toutes les API ou tous les modèles : vérifier les droits et les quotas de chaque produit. Les instructions d'agent et les prompts peuvent être transmis sans fichier privé ni clé API.

