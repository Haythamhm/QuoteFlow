# Sprint 1 — Intégration exploratoire de l'API Odoo

**But :** vérifier le contrat réel entre QuoteFlow et Odoo Community 19 avant de créer le backend ou un adapter. Ce sprint est manuel et exploratoire ; il ne développe aucune fonctionnalité QuoteFlow.

**État au démarrage :** aucune preuve locale de lecture JSON-2 ou de création d'un devis par API n'est consignée. Ne pas annoncer ces tests comme réussis avant de les avoir exécutés.

## Garde-fous

- Base Odoo de développement seulement ; ne jamais modifier le devis de référence `S00001`.
- T1.1 à T1.4 sont des actions de configuration locale et des lectures API. T1.5 crée un enregistrement Odoo : l'appel d'écriture attend un accord explicite du développeur.
- Ne jamais partager ni enregistrer la clé API dans Git, `.env.example`, le frontend, une capture, un ticket ou une conversation. Utiliser un environnement Postman local marqué secret et ne pas l'exporter avec la valeur.
- Ne pas envoyer, confirmer ou facturer le devis. Ne pas relancer une création après timeout ou réponse ambiguë avant d'avoir recherché le résultat dans Odoo.
- Ne pas exécuter de commande destructive Docker, ni réinitialiser une base.
- Les modèles, méthodes et champs doivent être découverts sur cette instance via `/doc` ou la documentation disponible ; ne pas inventer un schéma de création.

## Prérequis du sprint

1. Le développeur confirme manuellement que l'instance Odoo locale est disponible, que Sales et Contacts sont installés et que les données de recette sont présentes. Aucun redémarrage de conteneur n'est demandé par ce plan.
2. Identifier la base locale de développement. Les documents de recette citent `quoteflow_odoo_dev`, mais confirmer le nom dans l'instance avant la requête.
3. Avoir Postman ou un client HTTP déjà disponible. Ce ticket n'autorise pas l'installation d'outil ou de dépendance.
4. Avoir le client `Casablanca Tech SARL`, les produits `QF-LAP-001` et `QF-MON-001`, et le devis manuel `S00001` vérifiables dans l'interface Odoo. Ces noms et montants sont des données de recette documentées, pas une preuve qu'ils existent actuellement.
5. Lire les entrées D-05, D-06, D-11 et D-13 dans [`14-decisions-non-resolues.md`](14-decisions-non-resolues.md). Les incertitudes doivent être consignées, pas masquées.

## Tickets

### T1.1 — Compte de service Odoo et clé API

**Objectif :** disposer d'une identité Odoo distincte pour QuoteFlow, avec les droits minimaux nécessaires aux lectures prévues et au test de création contrôlé.

**Prérequis :** accès administrateur humain à l'Odoo local et confirmation du nom de la base. Ne pas donner la clé à un agent.

**Étapes manuelles :**

1. Dans Odoo, ouvrir **Paramètres → Utilisateurs et sociétés → Utilisateurs** (les libellés peuvent varier selon la langue/version).
2. Créer un utilisateur interne dédié, avec nom/login explicites pour l'intégration QuoteFlow ; ne pas réutiliser le compte personnel.
3. Dans les droits d'accès, attribuer uniquement les droits nécessaires pour lire les contacts et les produits. Le test T1.5 exigera également le droit de créer un devis brouillon et ses lignes ; identifier les groupes réellement nécessaires dans cette édition sans attribuer un rôle administrateur global par facilité.
4. Enregistrer les groupes/droits réellement attribués. Les vérifier ensuite par les lectures et erreurs d'accès des tickets suivants.
5. Ouvrir les préférences de cet utilisateur puis **Sécurité du compte → Nouvelle clé API** (ou l'écran équivalent de l'instance). Saisir un libellé reconnaissable et une durée conforme à la politique locale ; générer la clé et la copier une seule fois dans un stockage local protégé. La documentation Odoo 19 indique qu'une description et une durée sont demandées, que la valeur est montrée une seule fois et que la durée maximale documentée est de trois mois ; vérifier les règles réellement présentées par l'instance.
6. Dans Postman, créer un environnement local non exporté contenant `base_url`, `database` et `api_key`; marquer `api_key` comme secret. Ne pas sauvegarder une collection avec la valeur littérale de la clé.

**Tests attendus :** confirmer que le compte peut s'authentifier au T1.2 ; vérifier ensuite que les opérations hors de ses droits échouent. Ne pas tester de mutation à ce stade.

**Acceptation :** compte technique distinct créé ; droits documentés et limités ; clé stockée localement sans exposition ; base Odoo confirmée. La génération de la clé n'est pas faite dans ce sprint par l'agent.

### T1.2 — Disponibilité JSON-2 et authentification

**Objectif :** déterminer si l'instance expose l'API JSON-2 et vérifier la forme des réponses.

**Prérequis :** T1.1 ; URL locale et base confirmées ; endpoint `/doc` consulté si disponible.

**Requête Postman de lecture exploratoire :**

```http
POST {{base_url}}/json/2/res.partner/search_read
Authorization: Bearer {{api_key}}
Content-Type: application/json
User-Agent: QuoteFlow-Sprint1
X-Odoo-Database: {{database}}

{"domain": [], "fields": ["id", "name"], "limit": 5}
```

L'en-tête `X-Odoo-Database` est à conserver si le déploiement le requiert ; la documentation Odoo le rend conditionnel au routage/base. Ne pas supposer que l'URL ou l'authentification est valide avant d'avoir observé la réponse de l'instance.

**Tests Postman :**

- Clé valide et lecture `search_read` : noter code HTTP, `Content-Type`, corps et durée.
- Clé invalide ou retirée : noter le code et le message, sans copier le secret.
- Base absente/incorrecte, seulement si cela peut être testé sans modifier l'instance : noter l'erreur observée.
- `/doc` indisponible ou JSON-2 refusée : arrêter le parcours d'écriture, garder les éléments de diagnostic expurgés et marquer le sprint bloqué par une décision/API à clarifier. Ne pas changer vers une autre API sans accord.

**Acceptation :** une requête valide retourne une réponse JSON lisible ; le format d'erreur observé pour les cas invalides est consigné ; la disponibilité de JSON-2 sur l'instance Community est établie ou déclarée non établie. Aucun succès anticipé.

### T1.3 — Lecture des clients Odoo

**Objectif :** rechercher le client fictif de recette sans créer ni modifier de contact.

**Prérequis :** T1.2 réussi et champs du modèle `res.partner` vérifiés dans `/doc`.

**Étapes manuelles :**

1. Dans une requête `res.partner/search_read`, filtrer le nom pour retrouver `Casablanca Tech SARL`.
2. Demander uniquement les champs nécessaires, au minimum `id` et `name` si disponibles ; tout autre champ doit être justifié et vérifié dans la documentation de cette base.
3. Comparer l'enregistrement et son ID avec la fiche ouverte dans l'interface Odoo.
4. Vérifier si plusieurs sociétés, contacts enfants ou règles d'accès affectent les résultats ; consigner le comportement, sans inventer un filtre métier.

**Test Postman :** recherche du nom attendu ; recherche d'un nom absent, qui doit retourner une liste vide ou le résultat documenté sans création automatique.

**Acceptation :** le client attendu et son ID Odoo sont identifiés ; aucune création ou modification n'a eu lieu. Le critère quantitatif est le jeu de recette effectivement présent : le Blueprint V1 parle de cinq contacts, tandis que le guide/V2 décrivent trois clients. Ce nombre doit être confirmé par le développeur, voir D-13.

### T1.4 — Lecture des produits Odoo

**Objectif :** retrouver les deux références de test et confirmer les données utilisées pour la recette.

**Prérequis :** T1.2 réussi ; méthode et champs du modèle `product.product` vérifiés dans `/doc`.

**Étapes manuelles :**

1. Rechercher les références internes `QF-LAP-001` et `QF-MON-001`.
2. Après vérification dans `/doc`, demander l'ID, le nom, la référence interne, les indicateurs d'activité/vendabilité et le prix utile au contrôle.
3. Comparer les références, l'activité et les prix avec les fiches dans l'interface Odoo.
4. Si l'instance distingue produit et variante, noter quel modèle et quel identifiant doivent être utilisés, sans déduire le contrat du template.

**Tests Postman :** recherche de chaque référence ; recherche d'une référence absente ; vérification des droits de lecture. Aucune mutation.

**Acceptation :** les deux articles de recette sont identifiés sans ambiguïté et leurs valeurs correspondent à Odoo. Les documents de recette donnent 8 500 DH et 1 400 DH comme prix de référence ; les valeurs présentes sur l'instance font foi et tout écart est consigné, pas corrigé par le client API.

### T1.5 — Création contrôlée d'un devis brouillon de laboratoire

**Objectif :** démontrer que l'API de l'instance peut créer un devis distinct de `S00001` au stade brouillon, avec le bon client et deux bonnes lignes.

**Prérequis :** T1.1 à T1.4 réussis ; `/doc` ou documentation de cette instance a confirmé la méthode, les champs, les relations, et les règles applicables à `sale.order` / `sale.order.line`. Le développeur a donné son accord explicite pour l'appel qui crée l'enregistrement Odoo.

**Étapes manuelles :**

1. Dans Odoo, vérifier une dernière fois que `S00001` est intact et brouillon. Ne pas le modifier.
2. Préparer la requête Postman de création en utilisant uniquement le schéma réellement documenté pour cette instance. Ne pas reprendre un payload trouvé pour une autre version/base sans le vérifier.
3. Utiliser `Casablanca Tech SARL`, `QF-LAP-001` et `QF-MON-001` avec les quantités choisies et consignées avant l'appel. Inscrire un identifiant de test facilement reconnaissable dans le champ métier approprié uniquement si ce champ a été validé dans `/doc`.
4. Envoyer la requête une seule fois. Noter le code HTTP et l'ID renvoyé, sans enregistrer la clé.
5. Ouvrir le nouvel enregistrement dans Odoo ; vérifier client, produits, quantités, prix, taxes et totaux calculés par Odoo. Confirmer visuellement qu'il est au stade devis/brouillon.
6. Ne jamais appeler une méthode d'envoi, de confirmation, de validation de commande ou de facturation.
7. Si le client HTTP expire ou si la réponse ne permet pas de savoir si l'écriture a abouti, rechercher dans Odoo par l'identifiant de test avant toute action. Ne pas renvoyer la création tant que le résultat n'est pas établi.

**Tests Postman :** une seule création autorisée ; inspection de la réponse ; aucun second envoi automatique ; vérification dans l'interface Odoo. Les cas d'échec de permission sont documentés avec la réponse observée. Ne pas tester un retry en envoyant plusieurs créations.

**Acceptation :** un nouveau brouillon, distinct de `S00001`, apparaît dans Odoo avec le client, les deux produits et les quantités convenus ; Odoo calcule les prix/taxes/totaux cohérents ; aucune commande n'est envoyée ni confirmée. Si l'API n'est pas disponible ou le contrat demeure ambigu, le ticket reste non accepté avec le blocage documenté.

### T1.6 — Preuves expurgées et clôture du sprint

**Objectif :** rendre les résultats vérifiables sans conserver de secret ni de donnée personnelle inutile.

**Prérequis :** résultats réels de T1.1 à T1.5, y compris les échecs ou étapes non exécutées.

**Étapes manuelles :**

1. Consigner version Odoo affichée, date locale, base (nom seulement), tickets exécutés, commandes Postman, codes HTTP et résultats observés.
2. Archiver les corps de requête/réponse sans bearer token, cookies, email réel, coordonnées ou traceback interne sensible.
3. Ajouter une capture expurgée du nouveau brouillon et de `S00001` si nécessaire pour démontrer qu'il s'agit de deux devis distincts.
4. Consigner les droits du compte de service, les champs/méthodes découverts, les écarts de prix/taxes, les limitations de l'API et les questions ouvertes. Ne pas inscrire la clé.

**Acceptation :** un compte rendu vérifiable distingue les tests réussis, échoués et non exécutés ; aucune clé ou donnée sensible n'apparaît dans les preuves ; le développeur décide explicitement si Sprint 1 est terminé.

## Tableau récapitulatif des tests manuels

| ID | Requête/scénario | Résultat attendu ou preuve |
|---|---|---|
| S1-API-01 | Auth valide + `res.partner/search_read` | Réponse JSON et code HTTP observés ; lecture permise |
| S1-API-02 | Clé invalide | Échec d'authentification ; secret absent des preuves |
| S1-API-03 | Recherche client connu | ID et nom comparés à l'interface Odoo |
| S1-API-04 | Recherche client absent | Aucun client créé automatiquement |
| S1-API-05 | Recherche des deux références produit | IDs/références/activité/prix comparés à l'interface |
| S1-API-06 | Utilisateur sans droit d'une opération de lecture | Refus observé et consigné, sans élargir les droits automatiquement |
| S1-API-07 | Création du devis de laboratoire, après accord explicite seulement | Un brouillon distinct de `S00001`, contrôlé dans Odoo |
| S1-API-08 | Timeout/réponse d'écriture incertaine | Recherche de réconciliation avant tout nouvel appel ; aucune répétition aveugle |
| S1-API-09 | Vérification finale | Pas d'envoi ni de confirmation ; preuves expurgées |

## Compte rendu de sprint

```text
Sprint/Ticket :
Odoo/version observée :
Base de développement (nom uniquement) :
Tickets exécutés / non exécutés :
Requêtes et résultats HTTP réellement observés :
Champs et méthodes confirmés par /doc :
Résultat visuel Odoo :
Écarts / risques / questions ouvertes :
Secrets expurgés : oui/non
Décision humaine de clôture :
```

## Référence API

La [documentation officielle Odoo 19 JSON-2](https://www.odoo.com/documentation/19.0/developer/misc/api/odoo.html) décrit `POST /json/2/<model>/<method>`, l'authentification bearer, l'en-tête `X-Odoo-Database` conditionnel et la documentation des modèles propre à chaque base. La disponibilité effective doit être vérifiée sur l'instance locale.
