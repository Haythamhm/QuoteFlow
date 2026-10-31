# 13 — Registre de décisions architecturales, risques, hypothèses

## 1. Décisions proposées (à ratifier)

| ADR | Décision | Justification |
|---|---|---|
| ADR-001 | Odoo Community 19 auto-hébergé | Contrôle de l'ERP et des intégrations; pas de dépendance à un essai Online |
| ADR-002 | Backend ASP.NET Core 10 LTS | Support actuel, base pédagogique propre |
| ADR-003 | Clean Architecture en 4 projets .NET | Règles métier et dépendances explicites |
| ADR-004 | Monolithe modulaire, sans microservices | Plus adapté à un développeur solo |
| ADR-005 | PostgreSQL QuoteFlow séparé du PostgreSQL Odoo | Frontière entre données métier de l'outil et ERP |
| ADR-006 | TanStack Router et composants du template Shadcn | Ne pas réécrire un frontend déjà fonctionnel |
| ADR-007 | Identity + cookies sécurisés, non authentification mock | RBAC fiable, couplage backend contrôlé |
| ADR-008 | Création brouillon Odoo uniquement après approbation | Sécurité métier et absence d'opération commerciale non demandée |
| ADR-009 | Queue persistante minimale dans PostgreSQL | Déclenchement robuste sans broker supplémentaire |
| ADR-010 | Gestion explicite des écritures Odoo incertaines | Ne pas créer de doublon lors d'un timeout |
| ADR-011 | Node-RED et LM Studio optionnels, après MVP | Pas de dépendance artificielle à l'IA |
| ADR-012 | Test de faisabilité JSON-2 au Sprint 1 | Ne pas concevoir sur une API supposée disponible |

## 2. Risques et mesures

| Risque | Impact | Prévention / plan B |
|---|---|---|
| API JSON-2 indisponible / droits insuffisants | Bloque création de devis | Test S1 dès maintenant; vérifier documentation de l'instance et permissions; adapter approche validée |
| Écart prix/taxes entre UI et API Odoo | Devis commercial inexact | Odoo source de vérité; recette S00001 et validation avant création |
| Création Odoo réussie mais timeout | Doublon potentiel | Référence métier et réconciliation manuelle; pas de retries aveugles |
| Auth fictive du template laissée en place | Accès non sécurisé | Remplacer par Identity avant Sprint 4 validé |
| Agent modifie trop de fichiers | Bugs et perte de compréhension | Ticket court + lecture du diff + revue indépendante |
| Machine locale insuffisante (Odoo + Docker + IA) | Instabilité et lenteur | IA plus tard, désactiver services facultatifs, test séparé LM Studio |
| Secrets publiés | Accès non autorisé | .gitignore + user secrets/env; scans avant commit |
| Perte des données Odoo par suppression de volume | Retard et reconstruction | Backups base + filestore, pas de `docker compose down -v` sans accord |

## 3. Questions ouvertes à résoudre avant tickets concernés

- **Q1 (avant S3)** : le responsable qui crée sa demande doit-il la faire approuver par un autre responsable ? Proposition : **oui**, séparation des tâches; configurer deux comptes responsables seulement si on teste ce cas.
- **Q2 (avant S5)** : si les prix ou taxes Odoo ont changé depuis l'approbation, comment gérer l'écart ? Proposition : retourner en `NeedsReview`, demander confirmation puis re-soumettre; pas d'ajustement silencieux.
- **Q3 (avant S5)** : quels champs exacts de `sale.order`/`sale.order.line` sont accessibles par JSON-2 sur **ton** Odoo 19 ? Réponse obtenue uniquement après S1.
- **Q4 (avant S3)** : comment créer les premiers comptes et les mots de passe de test sans valeur hardcodée ? Proposition : script de provisioning local avec secrets d'environnement.
- **Q5 (avant S4)** : quelle stratégie définitive de session/cookies/CSRF avec Vite dev proxy et serveur .NET ? Choisir et tester avant UI connectée.
- **Q6 (avant S7)** : faut-il vraiment Node-RED si les notifications internes .NET couvrent le cas ? Conserver optionnel jusqu'à démonstration de la valeur.
- **Q7 (avant S8)** : matériel PC et qualité réelle du modèle; comparer via jeu d'essai avant adoption.

## 4. Règle d'évolution documentaire

À chaque décision : mettre à jour l'ADR, l'exigence FR/NFR/BR concernée, le cas UC et le ticket. Garder **une seule vérité fonctionnelle** : ces documents de cadrage approuvés par le développeur, pas les choix implicites d'un agent.
