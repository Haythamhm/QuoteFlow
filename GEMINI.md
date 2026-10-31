# GEMINI.md — contexte de revue pour QuoteFlow

## Documents de référence

Lire `AGENTS.md`, `README.md`, puis `docs/00-INDEX.md` et les documents actifs qu'il référence. Pour le Sprint 1, lire `docs/SPRINT-01-ODOO-API.md` et `docs/14-decisions-non-resolues.md` en plus des exigences, de l'architecture et du backlog associés. Les fichiers de `docs/archive/` sont historiques ; ils ne remplacent pas les documents actifs.

Les ADR marqués « proposé » et les entrées de la matrice marquées « ouvert » ne sont pas des décisions validées. Signaler les contradictions sans inventer de règle métier.

## Principes de revue

- Priorités : sécurité, RBAC, frontières Clean Architecture, intégration Odoo, idempotence, preuve des tests et simplicité pour un développeur seul.
- Ne pas modifier de fichier pendant une revue en lecture seule.
- Ne pas exposer de secret, appeler d'API Odoo, créer de devis, ni agir sur une base ou un conteneur sans autorisation explicite.
- Ne jamais inventer un résultat de commande, un test, une permission Odoo ou la disponibilité d'une API/modèle.
- Le frontend existant utilise TanStack Router ; ne pas recommander React Router comme remplacement.
