# 01 — Cadrage fonctionnel

## Problème
Une PME reçoit des demandes de devis par courriel ou message. Le commercial lit la demande, recherche les références dans Odoo, ressaisit les lignes et fait valider la proposition. Cela peut entraîner perte de temps et erreurs de sélection.

## Objectif démontrable
Permettre de saisir une demande textuelle, résoudre les références du catalogue Odoo, faire approuver la demande par un responsable et **créer un devis BROUILLON dans Odoo Sales**. Mesurer le temps de préparation avec et sans QuoteFlow sur un échantillon de tests fictifs. Ne pas inventer de gains sans mesure.

## Acteurs / RBAC

| Capacité | Commercial | Responsable commercial | Administrateur |
|---|---|---|---|
| Créer, voir, modifier sa demande avant soumission | Oui | Oui | Non par défaut |
| Chercher clients/produits Odoo | Oui | Oui | Non par défaut |
| Soumettre pour validation | Oui | Oui | Non par défaut |
| Voir toutes demandes de son périmètre | Non | Oui | Non par défaut |
| Approuver/rejeter | Non | Oui | Non |
| Créer un devis Odoo | Uniquement par processus automatisé suite à approbation | Uniquement par processus automatisé suite à approbation | Non |
| Gérer utilisateurs / intégration / logs techniques | Non | Non | Oui |

Appliquer chaque permission **sur le serveur**, avec vérification d'appartenance et de statut de la demande.

## États

- `Received` : demande saisie.
- `NeedsReview` : lignes extraites ou saisies, vérification client/produits requise.
- `PendingApproval` : soumise pour décision et figée pour le commercial.
- `Rejected` : rejetée ; motif obligatoire, puis correction et nouvelle soumission possible.
- `Approved` : approuvée, en attente de création Odoo ; immuable côté commercial.
- `IntegrationFailed` : création Odoo non confirmée ; reprise contrôlée.
- `OdooDraftCreated` : identifiant du devis Odoo persisté.

Transitions admises : Received → NeedsReview → PendingApproval → Approved → OdooDraftCreated. PendingApproval → Rejected → NeedsReview. Approved → IntegrationFailed → Approved (réconciliation/reprise). Aucune confirmation automatique de commande.

## Cas d'usage métier E2E

1. Admin configure une base Odoo 19 locale et un compte API dédié.
2. Commercial saisit : « 5 batteries 100Ah + 3 chargeurs 20A » et sélectionne le client.
3. QuoteFlow affiche la liste des produits Odoo correspondants ; en cas d'ambiguïté, l'utilisateur sélectionne.
4. Commercial vérifie les quantités, soumet.
5. Responsable approuve.
6. Une tâche persistante tente de créer un devis `sale.order` en statut brouillon via l'API Odoo.
7. QuoteFlow affiche son identifiant Odoo et un lien interne vers l'opération.
8. Commercial peut poursuivre le processus commercial *dans Odoo*.

## MVP — inclus
Connexion, RBAC, demandes et lignes, choix du client/catalogue Odoo, transitions d'état, approbation, création de devis brouillon, historique, statut de l'intégration, erreurs, tests automatisés et scénarios manuels.

## Hors MVP
Lecture automatique de Gmail/Outlook, multi-entreprise, IA, OCR, CRM complet, factures, paiement, stock, notifications SMS, application mobile, moteur de workflow distribué, microservices, orchestration Kubernetes, multitenancy, reporting avancé.

