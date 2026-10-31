# QuoteFlow — index documentaire

**Référence active : Conception V2, proposition à valider.** L'index réunit la documentation fonctionnelle, technique et le backlog dans un seul dossier `docs/`. Les propositions des ADR et la matrice des décisions restent non validées tant qu'une décision explicite n'est pas inscrite.

## Ordre de lecture

`AGENTS.md` → `README.md` → cet index → documents V2 actifs utiles au ticket → documents historiques si un point de contexte l'exige.

## Documents actifs

| Document | Sujet |
|---|---|
| [06-vision-exigences.md](06-vision-exigences.md) | Vision, périmètre, exigences FR/NFR |
| [07-regles-rbac-workflow.md](07-regles-rbac-workflow.md) | Règles métier, rôles, états et transitions |
| [08-use-cases.md](08-use-cases.md) | Parcours nominaux et exceptions |
| [09-architecture-systeme.md](09-architecture-systeme.md) | Architecture, données, API, Odoo et sécurité |
| [10-backlog-sprints.md](10-backlog-sprints.md) | Sprints, dépendances et critères de sortie |
| [11-tests-recette.md](11-tests-recette.md) | Stratégie de test et recette manuelle |
| [12-frontend-agents.md](12-frontend-agents.md) | Adaptation du template et consignes aux agents |
| [13-decisions-risques.md](13-decisions-risques.md) | ADR proposées, risques et questions ouvertes initiales |
| [14-decisions-non-resolues.md](14-decisions-non-resolues.md) | Matrice consolidée des décisions non validées |
| [SPRINT-01-ODOO-API.md](SPRINT-01-ODOO-API.md) | Plan détaillé des tickets T1.1 à T1.6 |

## Historique conservé

- [`archive/blueprint-v1/`](archive/blueprint-v1/) contient les cinq documents initiaux V1. Ils sont conservés comme référence historique ; en cas de différence, les documents actifs V2 prévalent.
- [`archive/conception-v2-import/README-CONCEPTION-V2.md`](archive/conception-v2-import/README-CONCEPTION-V2.md) conserve la note d'import et de prudence du paquet V2.
- [`archive/README-TEMPLATE-SHADCN-ADMIN.md`](archive/README-TEMPLATE-SHADCN-ADMIN.md) conserve les informations importantes du README amont du template frontend.

## État connu et limites

- Les documents V2 restent une proposition à ratifier. Toute contradiction ou règle non décidée doit être signalée, pas déduite.
- Le frontend est situé à la racine du dépôt (`src/`), utilise TanStack Router et contient encore les exemples/mocks du template.
- Le backend n'est pas encore créé. `infra/compose.yaml` décrit l'environnement local Odoo et les bases PostgreSQL.
- La lecture de données et la création d'un devis brouillon par JSON-2 ne sont pas encore prouvées sur l'instance locale.
- Le Sprint 1 comprend une création de devis directe à des fins de laboratoire uniquement ; elle nécessite un accord explicite avant l'appel qui écrit dans Odoo.

## Règle de priorité

Les exigences et règles V2 sont la référence fonctionnelle active du projet. Les ADR marquées « proposée », les questions ouvertes et les points de la matrice restent à valider. Le Blueprint V1 est complémentaire et historique ; il ne doit pas introduire une règle contraire aux documents actifs.
