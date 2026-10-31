# 14 — Matrice des décisions non résolues

Cette matrice rassemble les questions présentes dans la Conception V2 et les écarts relevés pendant l'audit. **Aucune ligne ci-dessous n'est une décision validée.** Les choix proposés dans les documents V2 restent des hypothèses jusqu'à ratification explicite.

| ID | Sujet | État documenté / hypothèse V2 | À décider / échéance |
|---|---|---|---|
| D-01 | Approbation par l'auteur | V2 propose qu'un responsable ne puisse pas approuver sa propre demande ; séparation des tâches. | Ratifier cette règle avant S3 ; préciser le parcours pour les demandes créées par un responsable. |
| D-02 | Périmètre du responsable | V2 dit « toutes les demandes de son périmètre » sans définir ce périmètre. | Définir le périmètre MVP avant S3 (par exemple équipe unique ou règle explicitement choisie). |
| D-03 | États `Received` et `NeedsReview` | La machine V2 montre `Received → NeedsReview`, mais les UC ne fixent pas le déclencheur. | Définir les conditions et le moment du passage avant l'implémentation du domaine en S2/S3. |
| D-04 | Variation de prix/taxe Odoo après revue ou approbation | BR-14 exige de ne pas substituer silencieusement une proposition ; V2 propose de revenir à `NeedsReview` puis de re-soumettre. | Ratifier la politique et préciser si une nouvelle approbation est toujours nécessaire avant S5. |
| D-05 | Disponibilité JSON-2 sur l'instance Community locale | V2 choisit JSON-2 comme cible, mais exige une preuve par Postman. La documentation Odoo 19 décrit JSON-2 et indique une restriction de disponibilité liée au plan ; son applicabilité à l'instance auto-hébergée Community reste à vérifier. | Tester l'instance réelle au Sprint 1 avant tout adapter. La décision d'intégration dépend du résultat observé. |
| D-06 | Champs et méthodes Odoo | Les champs de `sale.order`, `sale.order.line` et les règles de prix/taxes sont inconnus pour cette base. | Découvrir via `/doc` ou documentation de l'instance au Sprint 1 ; consigner la preuve avant de préparer T1.5. |
| D-07 | Sessions, cookies et CSRF | V2 propose Identity avec cookie HttpOnly et un proxy Vite ; les réglages SameSite, CSRF et origines sont encore à concevoir. | Choisir et tester avant S4 ; aucune décision ne se déduit du faux login du template. |
| D-08 | Création des comptes initiaux | V2 propose un provisioning local sans mot de passe codé en dur. | Définir le processus et le rôle de l'administrateur avant S3. |
| D-09 | Administration des utilisateurs | FR-13 exige la gestion des rôles/comptes d'essai, mais il n'existe pas encore de cas d'usage ni de contrat API détaillé pour cette fonction. | Décider si le MVP utilise un provisioning hors application ou une fonction admin ; cadrer les permissions avant S3/S4. |
| D-10 | Exposition des demandes et des erreurs | Les documents autorisent parfois 403 ou 404 pour ownership, 403 ou 409 pour statut et alternent 400/422 pour validation. | Fixer un contrat cohérent avant les tests API S3. |
| D-11 | Contacts et produits « autorisés » | Les droits Odoo et règles d'accès du compte de service s'appliquent, mais le filtrage métier des contacts et produits n'est pas décrit. | Vérifier au Sprint 1 les enregistrements et champs accessibles ; définir le besoin de filtrage avant S3/S5 sans élargir l'accès. |
| D-12 | Réconciliation et reprise | V2 distingue échec récupérable et issue incertaine, mais l'état métier `IntegrationFailed` et les transitions admin sont à préciser. | Définir quelles preuves permettent de rattacher un devis retrouvé ou de relancer une création avant S6. |
| D-13 | Nombre d'enregistrements de recette client | Le Blueprint V1 dit lire cinq contacts ; V2 et le guide décrivent trois clients de test. | Confirmer le jeu de données présent et fixer un seul critère de lecture pour T1.3. |
| D-14 | Validation du parcours UI en S4 | Le backlog place la création réelle Odoo en S5, tandis que S4 valide le workflow navigateur. | Définir si S4 utilise un adaptateur simulé explicitement signalé ou limite son critère au workflow interne ; aucune donnée de démonstration ne doit apparaître comme ERP réel. |
| D-15 | Préfixe des endpoints d'authentification | FR-01 écrit `/auth/me`, tandis que l'UC-01 et le contrat d'architecture utilisent `/api/auth/me`. | Choisir un préfixe REST canonique avant de figer le contrat API en S3. |

## Règle de mise à jour

Pour chaque décision ratifiée, inscrire la date, la personne qui valide, le choix retenu et les exigences/règles/UC/tickets affectés. Mettre ensuite à jour les documents actifs concernés et déplacer l'entrée correspondante de « ouverte » à « décidée ». Ne pas modifier cette matrice pour faire passer une proposition V2 pour une validation.
