# 06 — Vision, contexte, besoins, périmètre et exigences

## 1. Problématique et utilisateurs

**Contexte fictif** : AtlasTech Distribution vend du matériel informatique B2B au Maroc. Elle utilise Odoo Sales et Contacts. Une demande de devis arrive souvent sous forme de texte, avec des noms produits imprécis, des quantités et parfois des données client incomplètes. La ressaisie et l'approbation manuelle peuvent provoquer délais, erreurs ou perte de traçabilité.

**Question métier** : comment passer d'une demande libre à un devis Odoo correct, avec une validation humaine et un historique traçable, sans dupliquer l'ERP ?

**Utilisateur final** : commercial (prépare), responsable commercial (décide), administrateur technique (gère accès et intégrations). Le client externe n'a pas de compte QuoteFlow.

## 2. Objectifs testables (pas de promesse commerciale)

- O1 : une demande approuvée engendre un **unique brouillon** Odoo ou une erreur traçable nécessitant résolution ; aucune confirmation/envoi automatique.
- O2 : un commercial identifie le client et les références exactes dans Odoo, sans saisir des prix arbitraires.
- O3 : le responsable voit les demandes soumises et prend une décision explicite, historisée.
- O4 : l'administrateur peut diagnostiquer les pannes d'intégration sans accéder inutilement au contenu commercial.
- O5 : comparer 20 demandes fictives, saisie manuelle vs QuoteFlow, pour évaluer temps de traitement, taux de correction, références exactes ; aucun gain chiffré supposé.

## 3. Périmètre MVP

**Inclus** : login et gestion de sessions; 3 rôles; création de demande texte; sélection client Odoo; recherche de produits Odoo par nom/référence; lignes et quantités; contrôle de complétude; soumission, rejet, correction, approbation; travail asynchrone de création du devis brouillon; statut d'intégration; événements d'audit; liste filtrable; messages d'erreur; tests.

**Exclus** : email entrant automatique, envoi email sortant, pièces jointes/OCR, paiement, facture, inventaire, commandes confirmées, modification d'Odoo depuis le navigateur, multitenant/multisociété, temps réel SignalR, notifications externes, IA par défaut, génération de prix par IA, statistiques avancées.

## 4. Exigences fonctionnelles prioritaires

| ID | Priorité | Exigence observable | Acteur |
|---|---|---|---|
| FR-01 | Must | Se connecter/déconnecter, retrouver son rôle dans `/auth/me` | Tous |
| FR-02 | Must | Créer une demande avec texte, propriétaire et date | Commercial |
| FR-03 | Must | Lister/voir uniquement les demandes autorisées | Commercial, Responsable |
| FR-04 | Must | Rechercher et sélectionner un client Odoo existant | Commercial, Responsable |
| FR-05 | Must | Rechercher des produits Odoo par nom/référence; afficher référence | Commercial, Responsable |
| FR-06 | Must | Ajouter/modifier/supprimer des lignes, quantités strictement positives | Commercial |
| FR-07 | Must | Soumettre seulement une demande complète et vérifiée | Commercial |
| FR-08 | Must | Approuver ou rejeter avec motif de rejet obligatoire | Responsable |
| FR-09 | Must | Créer automatiquement, après approbation, **un brouillon Odoo** | Système |
| FR-10 | Must | Afficher ID, lien Odoo et résultat de l'intégration | Commercial, Responsable |
| FR-11 | Must | Historiser les transitions, l'acteur et l'horodatage | Système |
| FR-12 | Must | Exposer les erreurs techniques sans fuite de secret | Admin |
| FR-13 | Must | Gérer rôles/comptes d'essai dans un processus d'administration sécurisé | Admin |
| FR-14 | Should | Filtrer les demandes par statut, date et propriétaire | Commercial, Responsable |
| FR-15 | Should | Détecter sélection produit ambiguë et exiger confirmation | Commercial |
| FR-16 | Could | Rappels via Node-RED, sans altérer le workflow | Système |
| FR-17 | Could | Extraction des lignes depuis texte par IA locale avec revue humaine | Commercial |

Les champs prix/taxes présentés avant approbation sont **indicatifs, issus d'Odoo**, et doivent être vérifiés au moment de créer le brouillon. Les taxes définitives sont celles appliquées par Odoo.

## 5. Exigences non fonctionnelles et méthodes de vérification

| ID | Exigence | Critère de recette prévu |
|---|---|---|
| NFR-01 | Isolation sécurité | Accès API sans authentification : 401 ; non autorisé : 403 ; propriétaire incorrect : 403/404 documenté |
| NFR-02 | Secrets protégés | Aucun secret dans Git, logs, bundle frontend ou fichiers `.env.example` |
| NFR-03 | Fiabilité externe | Une panne Odoo laisse l'événement et le travail d'intégration traçables ; aucune création silencieuse en double |
| NFR-04 | Validation et intégrité | Quantité <= 0, produit/client absent, transitions invalides rejetés côté serveur |
| NFR-05 | Développement reproductible | `docker compose config`, `dotnet build/test`, `npm run build/lint` selon le sprint |
| NFR-06 | Observabilité | Logs structurés avec correlation ID, sans identifiants secrets ni texte client intégral par défaut |
| NFR-07 | UX | États chargement/vide/erreur, labels, navigation clavier, affichage mobile >= 360 px vérifiés manuellement |
| NFR-08 | Performance locale | Sur machine de développement, médiane API QuoteFlow locale < 500 ms hors Odoo sur parcours simple; mesurer au lieu de promettre une SLA publique |
| NFR-09 | Maintenabilité | Domain sans EF/HTTP/Odoo; tests unitaires des transitions et tests API des permissions |
| NFR-10 | Confidentialité | Données clients fictives en développement; aucun email ou contact réel nécessaire; sauvegarde/restauration documentées |

**NFR-08 est une cible indicative**, à réviser après mesure selon matériel local. Disponibilité « 99.9% » ou latence Odoo garantie : non pertinentes pour un laboratoire solo.

## 6. Définition de la valeur ajoutée (mesures)

Jeu de test de 20 demandes fictives : temps total par demande, nombre de corrections de références, présence du bon client, taux de devis créés correctement, occurrences de doublons, reprises après interruption. Le rapport comparera les mesures et signalera limites (petit échantillon, données fictives, effet d'apprentissage).
