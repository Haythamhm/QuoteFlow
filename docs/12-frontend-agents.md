# 12 — Adaptation du frontend existant et contrat de travail des agents

## 1. Frontend existant à préserver

Le template **shadcn-admin-v1.0.0** contient React, TypeScript, Vite, Tailwind/shadcn, **TanStack Router**, TanStack Query, TanStack Table, React Hook Form, Zod, Recharts, Axios, Zustand et Vitest. Dans le dépôt réel, le frontend est situé à la racine : `src/`, `package.json`, `vite.config.ts` et les fichiers TypeScript associés. Il n'existe pas de dossier `frontend/`. Les commandes de build/lint/dev ne sont pas revalidées dans ce dépôt lors de l'audit.

### Principes d'adaptation

1. **Conserver TanStack Router**; ne pas installer React Router.
2. Conserver les composants Shadcn/Radix, tables, formulaires et layout; ne pas dupliquer des composants UI.
3. Les dashboards, listes et utilisateurs de démonstration sont des **mocks**, jamais les données du backend.
4. Supprimer/remplacer l'authentification simulée avant usage métier. La présence de Clerk n'oblige pas à l'utiliser; Identity ASP.NET Core est le fournisseur choisi.
5. L'API React appelle uniquement `/api/...` vers QuoteFlow; jamais `http://localhost:8069` ni clé Odoo dans le navigateur.
6. Respecter la structure réelle du template à la racine (`src/routes`, `src/features`, `src/components`), l'explorer avant de déplacer des fichiers. Ne pas supposer l'existence de `frontend/`.
7. Diff limité par ticket; ne pas refactoriser l'intégralité du template par commodité.

## 2. Mapping fonctionnel UI

| Page fonctionnelle QuoteFlow | Source template potentielle | Nouveau comportement |
|---|---|---|
| Connexion | Sign-in | Vrai login Identity + `/auth/me`, états 401/erreur |
| Liste demandes | Tasks / DataTable | Colonnes propriétaire/client/date/statut, filtrées par API |
| Nouvelle demande | Form + Dialog/Input | Texte source, client, lignes et validations Zod + serveur |
| Détail demande | Cards/Tabs | Historique + produits Odoo + actions conditionnelles |
| Approbations | Tasks/table + AlertDialog | Approbation/rejet motif, accès responsable uniquement |
| Dashboard | Dashboard/Recharts | Quelques KPIs mesurables de QuoteFlow; commencer par compteurs |
| Admin utilisateurs | Users demo | Vue d'admin limitée, backend Identity, aucune clé exposée |
| Statut ERP | Badge / Status | Intégration pending/success/error/reconciliation |

## 3. Navigation suggérée

- Commercial : **Mes demandes**, **Nouvelle demande**.
- Responsable : **Demandes**, **À approuver**, **Tableau de bord**.
- Admin : **Utilisateurs**, **État des intégrations**, **Paramètres techniques**.
- Un route guard côté React améliore l'UX, **mais ne remplace jamais le RBAC de l'API**.

## 4. Préparer le contexte Codex correctement

1. Ouvrir **la racine du dépôt QuoteFlow** dans Codex Desktop. Le frontend est dans `src/` à cette racine ; aucun sous-dossier `frontend/` n'existe actuellement.
2. Confirmer que `AGENTS.md`, `README.md` et `docs/` sont dans cette racine.
3. Donner explicitement la liste des documents à lire; demander la confirmation de lecture en citant **chemins et décisions**, sans longs copier-coller.
4. Indiquer le sprint/ticket actif, l'objectif, les répertoires autorisés, les commandes de test attendues.
5. **Au premier démarrage : audit en lecture seule.** Pas de refactor, npm install, migration EF ou changement Docker sans autorisation.
6. Après validation du plan : implémentation par tickets; review Gemini sur diff en lecture seule.
7. Les agents n'inventent pas une preuve de test, un environnement ou une permission Odoo; ils signalent les limites d'accès.

## 5. Première commande à donner à Codex (audit seulement)

```text
Agis comme Lead Software Architect / Senior Full-Stack Engineer .NET + React + Odoo.

Nous construisons QuoteFlow, projet individuel d'apprentissage, avec Odoo Community 19 et le template frontend Shadcn Admin déjà importé.

Travaille dans le dépôt RACINE ; le frontend est dans src/ à cette racine.
Lis dans cet ordre : AGENTS.md, README.md, docs/00-INDEX.md, puis les documents actifs requis par le ticket ; pour un audit global, lire docs/06 à 14, docs/SPRINT-01-ODOO-API.md et consulter l'archive Blueprint V1 au besoin. Parcours la structure réelle de src/ et infra/ à la racine du dépôt.

Mission immédiate : AUDIT ET PLAN UNIQUEMENT. Ne modifie aucun fichier et ne lance aucune commande qui écrit des données ou installe des dépendances.

1. Résume en 15 lignes le but métier, les acteurs, la source de vérité Odoo, les états et les limites MVP.
2. Liste les composants présents du projet et les écarts entre documentation et fichiers réels.
3. Vérifie que le frontend utilise TanStack Router, identifie le login fictif et les mocks, sans les modifier.
4. Repère les contradictions ou les décisions ouvertes : note-les dans ton rapport, ne tranche pas seul.
5. Propose le plan du Sprint 1 (API Odoo) en tickets détaillés : dépendances, actions humaines, critères d'acceptation, tests Postman, limites d'accès.
6. Ne code rien. Attends mon autorisation explicite ticket par ticket.

Format : résumé métier / audit technique / incohérences / risques / Sprint 1 détaillé / questions bloquantes.
```

## 6. Prompt de revue Gemini

```text
Agis en reviewer senior indépendant et lecture seule du dépôt QuoteFlow.
Lis docs/00-INDEX.md puis les exigences, les règles BR, les UC, l'architecture et les tests actifs.
Analyse le rapport de Codex ou le diff du ticket demandé. Classe les constats P0 (sécurité/perte données), P1 (règle métier cassée), P2 (maintenabilité), P3 (cosmétique). Indique preuves (fichier/ligne ou scénario) et corrections minimales. N'écris pas de code, ne modifie pas de fichier, ne lance pas d'action sur Odoo.
```

## 7. DoD des agents

Rapport d'audit avant code; pas de secret fourni; une branche/ticket; logs de commandes réelles; explication pédagogique des changements; tests unitaires et manuels; validation finale humaine. Codex n'est pas source d'autorité métier.
