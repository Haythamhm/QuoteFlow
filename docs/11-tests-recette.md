# 11 — Stratégie de tests et recette manuelle

## 1. Pyramide de tests

| Niveau | Objectif | Exemples |
|---|---|---|
| Domaine (`xUnit`) | Transitions et invariants purs | quantité >0, rejets, approbation |
| Application | Cas d'usage avec dépendances simulées | permissions, job créé exactement une fois |
| API integration (`WebApplicationFactory` ou équivalent) | Auth/RBAC/status/validation | 401, 403, 404, 409, 422 |
| Infrastructure | PostgreSQL réel et Odoo de labo | EF migrations, JSON-2, mapping, timeout |
| Frontend (`Vitest`) | Composants et formulaires | lignes invalides, états d'erreur, filtre |
| E2E manuel / Playwright plus tard | Navigation complète | demande -> validation -> devis Odoo |

Éviter une grande suite fragile end-to-end dès Sprint 2. Ajouter tests pertinents à chaque ticket.

## 2. Référence de recette Odoo existante

- Base : `quoteflow_odoo_dev`.
- Société : AtlasTech Distribution.
- Client : Casablanca Tech SARL.
- Produits : `QF-LAP-001` prix 8 500 DH; `QF-MON-001` prix 1 400 DH.
- Devis manuel **S00001** avec 2 × 8 500 + 3 × 1 400 = **21 200 DH HT**; TVA 20 % = **4 240 DH**; total **25 440 DH TTC**.
- **Ne pas envoyer ou confirmer S00001.** Le test API créera un devis distinct, non une modification destructive.

## 3. Scénarios AT minimaux

| ID | Référence | Action | Attendu |
|---|---|---|---|
| AT-01 | FR-01 / UC-01 | API demande sans session | HTTP 401 |
| AT-02 | FR-03 / UC-08 | Commercial A lit demande de B | 403 ou 404 cohérent |
| AT-03 | FR-06 / BR-03 | Ajouter quantité 0/-1 | 422, aucune modification |
| AT-04 | FR-07 / BR-04 | Soumettre sans client | 422, statut inchangé |
| AT-05 | FR-07 / BR-04 | Soumettre sans ligne confirmée | 422, statut inchangé |
| AT-06 | FR-08 / BR-07 | Commercial appelle `/approve` | 403 |
| AT-07 | FR-08 / BR-07 | Responsable approuve sa propre demande | 403 |
| AT-08 | FR-08 / BR-08 | Rejet sans motif | 422 |
| AT-09 | FR-08 / BR-06 | Modifier demande en attente d'approbation | 409/403 selon politique documentée |
| AT-10 | FR-09 / BR-09 | Approbation valide | État Approved + 1 seul job dans même transaction |
| AT-11 | FR-09 / BR-18 | Double clic/double requête approbation | Pas de double job |
| AT-12 | FR-09 / BR-11 | Worker crée devis | Odoo `sale.order` brouillon, jamais envoyé ou confirmé |
| AT-13 | FR-10 / BR-15 | Création Odoo réussie | ID Odoo local enregistré, affiché aux personnes autorisées |
| AT-14 | NFR-03 / BR-16 | Timeout après appel Odoo ambigu | NeedsReconciliation, pas de réessai aveugle |
| AT-15 | NFR-02 | Inspecter bundle, Git et logs | Pas de secret Odoo/DB/API exposé |
| AT-16 | NFR-07 | Tester mobile 360 px, clavier, erreurs API | UI accessible, aucun formulaire bloqué |
| AT-17 | FR-04 / UC-03 | Recherche client inconnu | Aucune création automatique dans Odoo |
| AT-18 | FR-17 / UC-10 | Modèle IA propose référence inexistante | Proposition rejetée ou corrigée avant validation |

## 4. Recette manuelle par sprint

### Sprint 0 — déjà majoritairement réalisé
- Contrôler Odoo :8069, Sales/Contacts, société et données fictives.
- Vérifier S00001 en brouillon et totaux.
- Tester `docker compose stop` puis `up -d`, revérifier que S00001 existe. Ne pas exécuter `down -v`.

### Sprint 1 — prochain jalon manuel
- Générer clé d'API technique sur Odoo local, stocker hors Git.
- Postman : `res.partner/search_read`, `product.product/search_read`.
- Vérifier réponse HTTP + champs utiles.
- Découvrir endpoint et schéma de `sale.order` via `/doc` ou la documentation de version installée.
- Créer un second devis brouillon via API; comparer client, lignes, quantités, taxes et total dans l'interface.
- Archiver **captures expurgées** et réponses sans jetons ni coordonnées personnelles.

### Sprint 3 — contrôle du RBAC
- Créer trois comptes locaux différents : Commercial, Responsable, Admin.
- Tester les endpoints avec chacun des comptes, y compris les essais d'accès interdits par simple HTTP.

### Sprint 5–6 — contrôle E2E et résilience
- Demande textuelle -> produits reconnus manuellement -> approbation -> un brouillon Odoo.
- Couper Odoo pendant une opération, vérifier message, job, possibilité de diagnostic.
- Simuler réponse incertaine; vérifier que la création n'est pas réessayée automatiquement.

## 5. Journal de preuve de sprint

Template à copier dans une description de ticket ou `docs/progress/` :

```text
Sprint/Ticket:
Git commit/branche:
Configuration: .NET __ / Node __ / Odoo __
Exigences FR/BR/UC:
Commandes réellement exécutées:
Résultats obtenus:
Tests manuels (date, résultat, captures sans secrets):
Bugs restants:
Décision de validation (par humain):
```

Une compilation réussie n'est **pas** une preuve d'intégration Odoo réussie.
