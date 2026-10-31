# QuoteFlow

QuoteFlow est un projet individuel d'apprentissage et de démonstration pour préparer des demandes de devis B2B et, après approbation humaine, créer un devis **brouillon** dans Odoo Community 19.

Odoo reste la source officielle des clients, produits, prix, taxes et devis. QuoteFlow n'accède jamais directement aux tables PostgreSQL d'Odoo. Aucune commande n'est envoyée ou confirmée et aucune facture n'est créée automatiquement.

## État du projet

- Le frontend est le template Shadcn Admin, situé à la racine du dépôt (`src/`), avec React, TypeScript, Vite et TanStack Router.
- L'infrastructure de développement Odoo et PostgreSQL se trouve dans `infra/`.
- Le backend ASP.NET Core n'est pas encore créé.
- La disponibilité de l'API JSON-2 et la création d'un devis brouillon par API ne sont pas encore démontrées. Le Sprint 1 est décrit dans [`docs/SPRINT-01-ODOO-API.md`](docs/SPRINT-01-ODOO-API.md).
- Le login, Clerk, les utilisateurs, les tâches et les graphiques présents dans le template sont des exemples ou des mocks ; ils ne constituent pas des fonctions QuoteFlow ni un contrôle d'accès.

## Documentation

Commencer par [`docs/00-INDEX.md`](docs/00-INDEX.md). Il identifie les documents actifs, les décisions encore ouvertes et les sources V1 conservées dans `docs/archive/`.

## Prérequis de développement

Le guide d'installation et de recette historique est [`archive/blueprint-v1/04-guide-manuel.md`](docs/archive/blueprint-v1/04-guide-manuel.md). Il couvre Windows, WSL2, Docker Desktop et la préparation manuelle d'Odoo. Ne pas supprimer de volume Docker ni réinitialiser une base.

## Démarrer l'infrastructure locale

Depuis PowerShell, après avoir vérifié le guide et configuré les secrets locaux :

```powershell
Set-Location infra
if (-not (Test-Path .env)) { Copy-Item .env.example .env }
# Remplacer localement les deux mots de passe d'exemple avant de continuer.
docker compose config --quiet
docker compose up -d
docker compose ps
```

Odoo est publié sur `http://localhost:8069`. PostgreSQL QuoteFlow est publié sur `localhost:5433`. Les bases et le filestore Odoo utilisent des volumes persistants. N'exécutez pas `docker compose down -v`.

## Frontend actuel

Le `package.json` racine fournit notamment ces commandes :

```powershell
npm run dev
npm run build
npm run lint
npm run test
```

Ces commandes concernent le template actuel, pas une interface QuoteFlow connectée à un backend. Les dépendances ne sont pas installées par ce README ; le développeur doit décider de leur installation au ticket concerné.

## Garde-fous

- Créer des devis brouillons dans Odoo uniquement après approbation humaine dans le futur workflow QuoteFlow.
- Le test manuel de création directe prévu au Sprint 1 est limité à la base de développement et nécessite un accord explicite avant son exécution.
- Ne jamais mettre de clé Odoo, mot de passe ou secret dans Git, le frontend ou les logs.
- Aucun développement métier sans ticket et accord explicite ; lire `AGENTS.md` avant le travail.

## Origine du template

Le frontend initial provient de Shadcn Admin. La démo, le dépôt source, l'attribution des auteurs et les informations de licence du README d'origine sont conservés dans [`docs/archive/README-TEMPLATE-SHADCN-ADMIN.md`](docs/archive/README-TEMPLATE-SHADCN-ADMIN.md).
