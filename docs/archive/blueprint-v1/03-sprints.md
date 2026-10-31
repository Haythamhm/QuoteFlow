# 03 — Backlog priorisé par sprints

**Une sprint n'est pas une semaine.** Estimation flexible, projet solo. Aucune dépendance Node-RED/LLM dans le cœur du MVP.

| Sprint | Objectif et apprentissage | Actions manuelles | Agent IA (ticket unique à chaque fois) | Critère de sortie |
|---|---|---|---|---|
| 0 — Environnement | Docker, Odoo, PostgreSQL, outils de développement | Vérifier WSL et Docker, initialiser base Odoo, sélectionner Sales/Contacts | Générer/relire compose ; préparer `.gitignore` et README | Odoo :8069 répond ; bases persistent après redémarrage |
| 1 — ERP/contrat | Workflow Sales et JSON-2 | Créer clients/produits, devis manuel, clé API, tester lecture clients/produits | Petit client HTTP exploratoire uniquement | Lire 5 contacts et créer un devis brouillon via API validé dans Odoo |
| 2 — Fondations | Clean Architecture, Git, EF Core, migrations | Vérifier la structure et `dotnet test` | Solution .NET + migrations + tests de démarrage | `dotnet build`, `dotnet test`, base QuoteFlow migrée |
| 3 — Métier/RBAC | Identity, use cases, invariants de QuoteRequest | Créer profils de test ; appeler endpoints via Swagger/Postman | Cas d'usage + policies + tests unitaires | Commercial ne peut pas approuver, Manager oui ; transitions invalides rejetées |
| 4 — React | UI, React Query, formulaires, gestion des erreurs | Tester chaque rôle et responsive | Pages login, liste, formulaire, détail, écran approbation | Nouveau devis interne créé et approuvé depuis UI ; aucun faux bouton fonctionnel |
| 5 — E2E Odoo | Intégration externe et création réelle du devis | Examiner devis brouillon dans Odoo ; comparer produits et quantités | Odoo adapter + mappers + tests d'intégration | QuoteRequest approuvé → devis brouillon Odoo ; ID persistant |
| 6 — Robustesse | Tâche persistante, retry, audit, tests | Couper Odoo pendant intégration, relancer, analyser doublons | Worker + gestion états d'erreur + tests | Panne Odoo traçable ; reprise contrôlée sans doublon silencieux |
| 7 — Automatisation | Événements et Node-RED (optionnel) | Installer Node-RED local ; simuler une notification | Webhook protégé ou événement, exemple de flux | Notification/rappel sans modifier règles métier |
| 8 — IA | Adapter extraction (optionnel) | Tester Qwen3 avec LM Studio sur 20 demandes fictives | Port AI + adapter + validation JSON + score de performance | IA propose sans créer devis ni inventer référence ; correction humaine |

## DoD — Definition of Done (tous les sprints)

- Acceptance criteria expliqués ; un seul ticket actif par agent.
- Compilation et tests existants réussis.
- Tests nouveaux couvrent les règles ajoutées, notamment 401/403, statut, sécurité.
- Vérification humaine des comportements critiques dans le navigateur/Odoo.
- Diff relu, secrets absents, documentation révisée, commit Git traçable.
- Les problèmes non résolus sont consignés ; aucun sprint annoncé « terminé » sans preuve.

## Ordre de livraison

MVP fonctionnel : sprints 0 à 5. Niveau portfolio solide : + sprint 6. Automatisation et IA : sprints 7 à 8, **uniquement après** stabilisation.

