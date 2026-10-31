# 02 — Architecture technique

> Archive Blueprint V1. La mention de React Router ci-dessous est historique et incompatible avec le frontend actuel ; la documentation active choisit TanStack Router, déjà présent dans le template.

## Solution : monolithe modulaire, dépôt unique

```text
quoteflow/
  AGENTS.md
  GEMINI.md
  docs/
  infra/
    compose.yaml
    .env.example
  backend/
    QuoteFlow.sln
    src/
      QuoteFlow.Domain/
      QuoteFlow.Application/
      QuoteFlow.Infrastructure/
      QuoteFlow.Api/
    tests/
      QuoteFlow.Domain.Tests/
      QuoteFlow.Application.Tests/
      QuoteFlow.Api.IntegrationTests/
  frontend/
    src/
      app/
      features/auth/
      features/quote-requests/
      features/odoo-catalog/
      features/approvals/
      shared/
```

Les dossiers `backend/` et `frontend/` sont à créer **pendant les sprints**, pas dans ce dossier documentaire initial.

### Sens des dépendances

- **Domain** : `QuoteRequest`, `QuoteRequestItem`, `QuoteRequestStatus`, invariants et règles. Aucune référence EF Core, HTTP ou Odoo.
- **Application** : cas d'usage / handlers, DTOs, ports (`IQuoteRequestRepository`, `IOdooQuotationGateway`, `IQuoteExtractionService`, `IUnitOfWork` si nécessaire). Référence Domain.
- **Infrastructure** : EF Core, Npgsql, Identity, HttpClient typé Odoo, outbox/jobs, extraction externe. Référence Application + Domain.
- **API** : endpoints, auth, politiques d'autorisation, validation HTTP, DI, OpenAPI. Référence Application + Infrastructure à la composition.
- **React** : React Router + TanStack Query et formulaires typés ; pas de logique de permission secrète côté client.

## Entités minimales

- `AspNetUsers`, `AspNetRoles`, ... (ASP.NET Core Identity).
- `QuoteRequests`: Id UUID, OwnerId, CustomerOdooId nullable, RawText, Status, Version, OdooQuotationId nullable, CreatedAt, UpdatedAt.
- `QuoteRequestItems`: Id UUID, RequestId FK, RawProductText, OdooProductId nullable, Quantity numeric, UserConfirmed boolean.
- `QuoteRequestEvents`: Id, RequestId, ActorId, Type, Notes, OccurredAt.
- `IntegrationJobs`: Id UUID, RequestId FK, Kind, Status, Attempts, NextAttemptAt, CorrelationId, ErrorSummary, CreatedAt.

Contraintes : clés FK, status contrôlé côté domaine, `Quantity > 0`, index sur statut / propriétaire, unicité sur `IntegrationJobs` pour une création active par demande, `OdooQuotationId` unique quand non null. Les prix et taxes sont validés à partir d'Odoo et **non inventés par l'IA**.

## API interne suggérée

| Méthode | Chemin | Accès |
|---|---|---|
| POST | `/api/quote-requests` | Commercial |
| GET | `/api/quote-requests` | Commercial (les siennes), Responsable (toutes) |
| GET | `/api/quote-requests/{id}` | Autorisé par ownership/rôle |
| PUT | `/api/quote-requests/{id}/items` | Commercial propriétaire, statut éditable |
| POST | `/api/quote-requests/{id}/submit` | Commercial propriétaire |
| POST | `/api/quote-requests/{id}/approve` | Responsable |
| POST | `/api/quote-requests/{id}/reject` | Responsable |
| GET | `/api/odoo/customers?search=...` | Commercial, Responsable |
| GET | `/api/odoo/products?search=...` | Commercial, Responsable |
| GET | `/api/quote-requests/{id}/integration` | Utilisateur autorisé |

Le worker crée le devis après approbation. Ne pas fournir d'endpoint public permettant de contourner l'approbation.

## Contrat Odoo

- Odoo reste la source de vérité pour `res.partner`, `product.product`, les prix et `sale.order`.
- QuoteFlow conserve uniquement les IDs externes et son propre historique métier.
- Accès par API JSON-2 (`/json/2/<model>/<method>`) sur Odoo 19, avec utilisateur technique et clé API à droits minimaux. Consulter la page `/doc` de l'instance.
- Ne jamais écrire dans les tables Odoo depuis le PostgreSQL local. Une API HTTP n'est pas une transaction SQL distribuée : erreurs/réconciliation obligatoires.
- Après timeout de création, **rechercher dans Odoo la référence métier** avant tout nouvel essai. `client_order_ref` peut aider mais n'est pas une garantie générale d'unicité. Un module Odoo contenant un identifiant externe unique pourra être ajouté ultérieurement pour une idempotence stricte.
- Vérifier en recette le calcul des listes de prix/taxes, les valeurs par défaut Odoo et la différence entre création via formulaire et API.
- Isoler l'intégration dans un adapter HttpClient ; pas de SDK communautaire imposé.

## Flux

`React → API [auth/RBAC] → Application [Use Case] → Domain [invariants] → Infrastructure [EF Core] → PostgreSQL`.

Après approbation : `transaction locale (status=Approved + IntegrationJob) → worker → API Odoo → sauvegarde sale.order.id → QuoteRequest OdooDraftCreated`.

## Sécurité

Identity, cookie HttpOnly/Secure/SameSite recommandé si React et API sont en même origine ; protéger les actions mutantes contre CSRF. Si domaines distincts, concevoir CORS, CSRF et session explicitement. Jamais de clé API Odoo exposée dans React. Validation serveur (quantité, ownership, transitions). Rate limits et secrets via user-secrets/variables d'environnement. Logs sans contenu sensible ni clés.

## Stratégie de test

Domain : transitions valides/invalides, quantités, ownership. Application : double soumission, rejet, validation, dépendance Odoo simulée. API : 401/403/404, modèles invalides, réponses cohérentes. E2E manuel : demande → approbation → devis brouillon. Intégration : Odoo disponible/indisponible, timeout post-création, réconciliation et absence de doublon.

