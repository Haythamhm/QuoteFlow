# AGENTS.md — règles obligatoires pour QuoteFlow

## Mission

Construire progressivement QuoteFlow en monolithe Clean Architecture (.NET 10, React TypeScript, PostgreSQL, Odoo Community 19). Le MVP doit créer un devis **brouillon** dans Odoo uniquement après approbation humaine.

## Documents prioritaires

Avant toute analyse ou modification, lire `README.md`, puis `docs/00-INDEX.md` et les documents qu'il désigne pour le ticket concerné. Pour le Sprint 1, lire au minimum `docs/06-vision-exigences.md`, `docs/09-architecture-systeme.md`, `docs/10-backlog-sprints.md`, `docs/13-decisions-risques.md`, `docs/14-decisions-non-resolues.md` et `docs/SPRINT-01-ODOO-API.md`.

Les documents actifs sont ceux de `docs/`. `docs/archive/` conserve les sources historiques et ne prévaut pas sur les documents actifs. Les choix des ADR et les décisions marquées « ouvertes » restent des propositions : ne pas les présenter comme validés.

## Règles non négociables

1. Une tâche ou un sprint limité à la fois. Expliquer l'approche avant le code ; attendre l'accord pour les décisions structurantes.
2. Architecture : Domain indépendant ; Application dépend de Domain ; Infrastructure implémente les ports ; API compose et expose. Aucun accès EF Core, HTTP ou Odoo dans Domain.
3. Odoo reste la source officielle pour clients, produits, prix et `sale.order` ; aucun accès SQL direct aux tables Odoo depuis QuoteFlow.
4. Ne jamais envoyer, confirmer une commande ou créer une facture automatiquement. Une approbation Responsable précède toute création de devis brouillon par QuoteFlow.
5. RBAC appliqué au backend, avec contrôle de propriété ; jamais seulement des boutons React.
6. Aucun secret (clés API, mots de passe, cookies, connexion PostgreSQL) dans Git ou les logs. Utiliser `.env` ignoré et user secrets / variables d'environnement.
7. Éviter microservices, message broker, cache distribué, DDD excessif, auto-mapping inutile ou dépendances lourdes dans le MVP.
8. Intégration Odoo : timeout, réconciliation et idempotence ; ne jamais relancer aveuglément une création incertaine.
9. Écrire des tests ciblés et fournir les commandes exécutées et leurs résultats. Ne jamais prétendre qu'un test a réussi sans l'avoir exécuté.
10. Ne pas supprimer de volumes Docker, réinitialiser de bases, déployer, envoyer d'emails ou effectuer de changements externes sans accord explicite du développeur.
11. Aucun développement métier sans ticket sélectionné et accord explicite. Un audit demandé en lecture seule n'autorise aucune modification.

## Critères de fin d'un ticket

- Le périmètre demandé seul a été modifié.
- Build et tests verts, ou limitations décrites précisément.
- Contrôles 401/403/404 et règles métier pris en compte.
- Documentation mise à jour ; diff court, compréhensible et sans secret.
