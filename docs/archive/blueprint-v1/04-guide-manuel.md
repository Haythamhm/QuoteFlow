# 04 — Guide manuel d'installation et de recette Windows

## Phase A — prérequis et diagnostic (à faire soi-même)

1. Vérifier Windows et la RAM via Paramètres > Système > Informations système.
2. Installer Git, Node.js LTS, .NET 10 SDK, Docker Desktop (mode Linux/WSL2) et un éditeur. Sources : sites officiels Git, Node, Microsoft, Docker.
3. Ouvrir **PowerShell**, puis :

```powershell
wsl --status
wsl -l -v
docker version
docker compose version
docker ps
dotnet --info
node -v
npm -v
git --version
```

**Interprétation :** `docker version` doit contenir un serveur et `docker ps` ne doit pas afficher d'erreur de pipe `dockerDesktopLinuxEngine`. Si c'est le cas, **ne pas passer à Docker Compose**. Ouvrir Docker Desktop, vérifier WSL2 et la virtualisation, puis résoudre le service. Ne pas effacer des volumes existants pour « essayer ».

## Phase B — démarrer l'ERP et les BDD

Depuis le répertoire `infra` :

```powershell
if (-not (Test-Path .env)) { Copy-Item .env.example .env }
# Éditer les DEUX mots de passe dans .env avant de continuer.
docker compose config --quiet
docker compose up -d
docker compose ps
docker compose logs odoo --tail 80
```

Les services sont exposés **sur localhost seulement**. Si les ports 8069/5433 sont occupés, choisir des ports libres et documenter le changement. Les données persistent dans des volumes Docker.

Ouvrir `http://localhost:8069`. Lors de l'initialisation Odoo, créer **une nouvelle base `quoteflow_odoo_dev`**. La protection du gestionnaire de bases Odoo doit être configurée avant toute exposition réseau. Pour un premier laboratoire local, ne jamais publier `8069` sur Internet.

## Phase C — préparer Odoo manuellement

1. Entreprise fictive : **AtlasTech Distribution**.
2. Installer **Sales** et **Contacts** ; vérifier que leurs fonctions existent bien dans cette édition.
3. Créer 3 clients fictifs et 10 produits test avec des codes internes distincts ; attribuer des prix de vente de démonstration.
4. Via **Sales → Quotations**, créer un brouillon de devis avec 2 produits, enregistrer et vérifier les quantités/taxes.
5. Utiliser un compte Odoo technique aux droits restreints (après avoir validé les permissions nécessaires) et une clé API ; relever le nom de base Odoo.

## Phase D — test API Odoo via Postman ou équivalent

```http
POST http://localhost:8069/json/2/res.partner/search_read
Authorization: bearer VOTRE_CLE_API
X-Odoo-Database: quoteflow_odoo_dev
Content-Type: application/json

{"domain": [], "fields": ["id", "name"], "limit": 5}
```

**Résultat attendu :** HTTP 200 et une liste JSON, pas une page HTML ou une redirection vers la connexion.

- 401 : clé absente/incorrecte.
- 403 : autorisations insuffisantes.
- 404 : endpoint ou modèle/méthode mal écrit, version incompatible ou route indisponible.
- Erreur base : vérifier `X-Odoo-Database`, les journaux et le nom de la base.

Consulter `http://localhost:8069/doc` si disponible pour explorer modèles et méthodes ; créer ensuite un devis brouillon **via API** sous contrôle du Sprint 1 (ne pas automatiser l'envoi).

## Phase E — tests manuels applicatifs (sprints suivants)

- Commercial connecté : crée une demande et modifie **la sienne** ; ne voit pas les demandes privées d'autrui.
- Le commercial reçoit 403 sur approbation, même en appelant l'API directement.
- Le responsable valide ; changement enregistré avec identifiant d'acteur et horodatage.
- Odoo reçoit **un brouillon**, pas une commande validée.
- Produit introuvable : demande marquée `NeedsReview`, jamais création par supposition.
- Panne API : `IntegrationFailed`, journal d'erreur, traitement réconciliable.
- Double clic sur « Approuver » : pas de double devis.

## Nettoyage et sauvegarde

`docker compose stop` pour stopper sans suppression ; `docker compose up -d` pour reprendre. **Éviter `docker compose down -v`** : cette commande supprime les volumes et peut détruire les bases d'apprentissage. Sauvegarder/exporter les données de test importantes avant les grosses évolutions.

## Si Docker Desktop est en panne

Plan B : installateur Odoo Community Windows (usage local de développement, pas production), PostgreSQL via installateur Windows, puis l'API Odoo par localhost. Mais choisir **une seule méthode** par environnement afin d'éviter conflits de ports et double démarrage de PostgreSQL.

