# Veris Assets — contexte permanent pour Gemini

Ce fichier est lu automatiquement par Gemini CLI et Gemini Code Assist (fichier de contexte `GEMINI.md` à la racine du dépôt). Il remplace, pour Gemini, le fichier `CLAUDE.md` utilisé jusqu'ici avec Claude Code. Les deux disent la même chose : en cas de doute, `CLAUDE.md` et `docs/` restent valables.

État du code décrit ici : branche `main`, commit `ef8e6f2` (1er octobre 2026).

---

## 1. Le produit en une minute

**Veris Assets** est un logiciel SaaS **B2B** pour les **émetteurs et sociétés de gestion** qui émettent et administrent des **obligations privées** (notes, obligations vertes, dette d'infrastructure ou immobilière).

- Utilisateurs : l'émetteur (administrateur, opérateur, conformité, auditeur), la plateforme (administrateur Veris) et des **investisseurs professionnels uniquement**.
- Cycle couvert : investisseurs et KYC/KYB → éligibilité et whitelist → émission → souscriptions → allocation → registre des porteurs → transferts → échéancier, coupons et remboursement → tableaux de bord, exports CSV, documents PDF, journal d'audit.
- **Pas de blockchain** : registre interne en ajout seul (append-only) chaîné en SHA-256, « compatible avec une future intégration ERC-3643 », hors MVP.
- **Ce n'est pas** une plateforme de financement participatif grand public. Pas d'investisseur particulier, pas de bouton « investir » sur le site public.
- Démo en ligne : https://veris-assets.com (EN/FR), données fictives uniquement.

### Positionnement réglementaire (à respecter dans le code et les textes)

Ce n'est pas un avis juridique ; c'est la ligne retenue par le porteur de projet.

- Veris Assets reste un **prestataire technique**. Il ne **détient jamais de fonds** (les paiements vont au compte de l'émetteur, d'un séquestre ou d'un agent payeur agréé), ne **place** pas de titres et ne **conseille** pas.
- Le modèle de revenus est un abonnement facturé aux clients (mise en place, mensualité, frais par émission, petit pourcentage d'encours), **jamais un pourcentage des montants levés ni des frais aux investisseurs**.
- **Aucune revendication de conformité ou de certification** (pas de « conforme MiCA », « agréé AMF », etc.). Le script `pnpm check:claims` bloque ces mentions en CI.
- Paiements, KYC, signature et valeur juridique du registre sont **fictifs** dans la démo, et doivent être présentés comme tels.

---

## 2. Qui tu aides

Le porteur de projet, **Alexis**, n'est **pas développeur**. Il travaille sur un Chromebook, dans **GitHub Codespaces** (VS Code dans le navigateur).

- Réponds en **français simple** ; explique chaque terme technique en une phrase.
- Le **code**, les noms techniques et les **messages de commit** sont en **anglais** (Conventional Commits : `feat(api): …`, `fix(web): …`, `docs: …`, `test(api): …`).
- **Demande confirmation** avant toute action destructrice ou irréversible : suppression de fichiers ou de branches, `git push --force`, réécriture de l'historique, suppression d'une base autre que la démo, toute action sur Railway ou Cloudflare (production).
- Ne jamais **inventer** une API, une option ou une version de bibliothèque. Vérifie dans `node_modules` ou la documentation, sinon dis que tu n'es pas sûr.

---

## 3. Sources de vérité (à lire avant de décider)

| Fichier | Contenu |
|---|---|
| `docs/SPEC.md` | Spécification complète. **Prime sur tout**, sauf décision écrite du porteur de projet. |
| `docs/DECISIONS.md` | Décisions D-001 à D-110. Seules les décisions **« Acceptée »** s'appliquent. Toute nouvelle décision s'ajoute ici (D-111, D-112…), avec ce qu'elle modifie. |
| `docs/ARCHITECTURE.md` | Stack, modules, dépendances autorisées, isolation multi-tenant, registre, tests. |
| `docs/DATA_MODEL.md` | Tables, contraintes, invariants. |
| `docs/API.md` | Contrats d'API. |
| `docs/BACKLOG.md` | Les 16 phases du MVP et leur statut (toutes livrées). |
| `docs/AUDIT.md` | Audit du code du 1er octobre 2026 : constats A-1 à A-7 et ordre de la suite. |
| `docs/DEPLOYMENT.md`, `docs/HOSTING.md` | Démo en ligne sur Railway (Amsterdam), mise à jour et remise à zéro. |
| `docs/SECURITY_CHECKLIST.md` | Revue sécurité de la phase 16. |
| `apps/web/AGENTS.md` | **Important** : Next.js 16 a des changements incompatibles avec ce que tu connais. Lis la doc dans `node_modules/next/dist/docs/` avant d'écrire du code front. |

Signale toute déviation par rapport à la spec **avant** de l'appliquer, et propose une décision à ajouter dans `docs/DECISIONS.md`.

---

## 4. Règles non négociables (section 31.2 de la spec)

1. Aucune revendication de conformité ou de certification réglementaire.
2. Aucun paiement réel, aucun KYC réel, aucune clé privée, aucune blockchain obligatoire dans le MVP.
3. **Ledger interne append-only = source de vérité.** Jamais d'`UPDATE` ni de `DELETE` sur le registre : une correction se fait par **contre-écriture**. Des déclencheurs PostgreSQL l'imposent.
4. Montants et quantités en **décimal** (`decimal.js`, colonnes `numeric`), **jamais en float**. Une règle ESLint interdit `Number()`, `Math.*` et `parseFloat` dans le code métier.
5. Timestamps techniques en **UTC**.
6. **Tenant (organisation) déterminé côté serveur** à partir de l'identité. Permissions et isolation contrôlées côté serveur ; RLS PostgreSQL activée et forcée sur chaque table qui a une colonne d'organisation.
7. Opérations financières **transactionnelles et idempotentes** (clé d'idempotence).
8. **Secrets hors du code et hors de git** : `.env` ignoré, `.env.example` commité. Le dépôt est **public** (D-104).

Marque : nom **Veris Assets** (jamais « Virtus Assets » ni « Astraea »). Logo `apps/web/public/brand/veris-assets-logo.svg` via le composant `Logo`, sans déformation ni recoloration. Couleurs : fond `#0A1020`, primaire `#4F52D6`, accent `#45D6E6` (section 23.1 de la spec).

---

## 5. Stack technique (versions figées)

| Couche | Choix |
|---|---|
| Langage, monorepo | TypeScript 6 partout ; pnpm 12 (workspaces) ; **Node.js 24** obligatoire |
| Front (`apps/web`) | Next.js 16 (App Router), React 19, Tailwind CSS 4, shadcn/ui, next-intl (en-GB, fr-FR), TanStack Query, client typé généré depuis l'OpenAPI, React Hook Form + Zod |
| Back (`apps/api`) | NestJS 12, Zod (validation et OpenAPI), Better Auth + Argon2id (sessions, TOTP), pino |
| Données | PostgreSQL 18 (fonction `uuidv7()`), Drizzle ORM 0.45 + migrations SQL manuelles (RLS, déclencheurs) |
| Asynchrone | pg-boss (file de jobs dans PostgreSQL) + table `outbox_event` |
| Autres services | Redis (limites de débit, cache), Garage (stockage S3 des documents, D-005), Mailpit (emails de test) |
| Tests | Vitest (unitaires, intégration sur vrai PostgreSQL, propriétés), Playwright (navigateur) |
| Qualité | ESLint, Prettier, dependency-cruiser (frontières entre modules), TypeScript strict |
| CI | GitHub Actions (`.github/workflows/ci.yml`), Dependabot |
| Hébergement démo | Railway, région EU West (Amsterdam), domaine veris-assets.com chez Cloudflare |

### Organisation du dépôt

```
apps/web          Next.js : landing, page Technologie, portails émetteur, investisseur et plateforme
apps/api          NestJS : toute la logique métier
  src/core        base de données, auth, notifications, jobs, erreurs, observabilité
  src/modules     iam · investor-compliance · issuance · registry · servicing · reporting-audit
  drizzle/        migrations (dont 0001_security_foundation.sql, 0017_registry_ledger_security.sql)
  scripts/db/     setup, migrate, seed, reset, backup, restore
packages/shared   code partagé web/API (décimaux, statuts, schémas)
packages/config   configuration ESLint et TypeScript partagée
infra/            configuration Garage et Mailpit
scripts/          démarrage (dev.sh), contrôles, déploiement Railway
docs/             spec, décisions, architecture, audit
```

Chaque module back suit `domain/` (règles pures), `application/` (cas d'usage), `infrastructure/` (base, fournisseurs), `api/` (contrôleurs). Les dépendances autorisées entre modules sont vérifiées par `pnpm depcruise` (voir ARCHITECTURE.md §4.2). Les fournisseurs externes passent par des interfaces (`PaymentProvider`, `KycProvider`, `EmailProvider`…) avec des implémentations `Fake*` en démo.

---

## 6. Démarrer et tester

Dans un Codespace (machine 4 cœurs recommandée, D-008) :

```bash
pnpm dev            # crée .env, démarre PostgreSQL/Redis/Garage/Mailpit, lance l'API (4000) et le web (3000)
pnpm db:reset       # remet la démo à zéro (refuse de tourner en production)
pnpm lint           # ESLint + depcruise + check:claims + Prettier
pnpm typecheck
pnpm test           # ~380 tests unitaires
pnpm test:integration   # vrai PostgreSQL 18 (services Docker démarrés)
pnpm test:coverage      # ≥ 90 % exigés sur registry et servicing
pnpm test:e2e           # Playwright
pnpm api:generate       # après un changement d'API : régénère OpenAPI + client web
```

Comptes de démo : listés sur `/fr/login`, mot de passe commun = `DEMO_ACCOUNTS_PASSWORD` du `.env`. Les comptes admin et conformité ont la 2FA : en mode démo, le code à 6 chiffres est affiché sur la page de connexion.

Démo en ligne : `pnpm online:deploy` (publier), `pnpm online:reset` (remettre à zéro), `pnpm test:online` (vérifier). Ces commandes touchent Railway : **demande l'accord d'Alexis** avant de les lancer.

**Avant chaque commit**, lance au minimum `pnpm lint`, `pnpm typecheck` et `pnpm test`. La CI rejoue tout, y compris l'intégration, la restauration de sauvegarde et Playwright.

---

## 7. Où en est le projet

- **MVP terminé** : les 16 phases du `docs/BACKLOG.md` sont livrées (PR #1 à #3), la landing a été refaite (D-109) avec des photos de projets illustratifs (PR #4, D-110), et l'audit a ajouté des tests par propriétés sur le registre et les coupons (PR #5).
- **Audit du 1er octobre 2026** (`docs/AUDIT.md`) : tout ce que le site annonce est réellement codé et testé. Écarts restants :

| # | Gravité | Constat | Piste |
|---|:-:|---|---|
| A-1 | Moyenne | `position.acquisition_amount` est hors du registre, du hash et du contrôle nocturne | Écrire le montant dans chaque mouvement (couvert par le hash) et l'ajouter au contrôle nocturne |
| A-2 | Moyenne | La chaîne de hash vit dans la même base : un admin de la base peut tout réécrire | Publier chaque jour le dernier hash de chaque émission hors de la base (e-mail/relevé émetteur, idéalement horodatage RFC 3161) |
| A-3 | Haute | Un seul environnement Railway `production` sert la démo ; sauvegardes chez le même fournisseur | Séparer démo, recette, production ; copier les sauvegardes chez un 2ᵉ fournisseur européen (OVHcloud ou Scaleway) |
| A-4 | Moyenne | Aucune alerte d'exploitation | Sentry (région UE) + alertes : sauvegarde ratée, écart du contrôle nocturne, API en erreur |
| A-5 | Basse | 30E/360 : coupons inégaux pour une émission datée d'une fin de mois (178/182 jours) | **Décision d'Alexis à prendre** : formule stricte ou option « coupon fixe par période » |
| A-6 | Basse | 2FA non obligatoire pour l'opérateur émetteur | L'imposer à tous les rôles émetteur en production |
| A-7 | Basse | Analyse des secrets GitHub non activée (dépôt public) | Settings → Code security (action d'Alexis) |

- Points d'environnement connus : PostgreSQL **18** est requis (la fonction `uuidv7()` n'existe pas en 16) ; Node **24** est requis. L'hébergement Neon a été étudié et écarté (pas de région Amsterdam, A-3 resterait ouvert).
- Branches Dependabot ouvertes sur le dépôt (Mailpit, otel-lgtm, mises à jour npm mineures) : à relire et fusionner si la CI passe.

## 8. Prochaines étapes (ordre validé après l'audit)

1. **Environnements séparés** démo / recette / production et sauvegardes hors fournisseur (A-3).
2. **Alertes d'exploitation** et suivi des erreurs (A-4).
3. **Montant d'acquisition dans le registre** (A-1) et **ancrage quotidien du hash** (A-2).
4. **Import d'un registre existant depuis Excel/CSV** (absent du code, indispensable pour un premier client).
5. **Remplacer le fictif** selon les prestataires choisis par Alexis : KYC/KYB, signature électronique, rapprochement des paiements (relevés CAMT.053), e-mails réels.
6. Puis test d'intrusion externe, RGPD, DORA (procédures), pilote avec un premier client.

Le détail (qui fait quoi, prestataires possibles) est dans `contexte/feuille-de-route-production.md` du package.

---

## 9. Comment travailler

- **Une étape à la fois.** Avant de coder : reformule la demande, liste les fichiers touchés, signale tout écart à la spec, attends l'accord.
- À la fin de chaque étape : résumé en français, fichiers créés ou modifiés, commande pour tester, résultat des tests, commit. Puis **arrête-toi** et attends l'accord avant l'étape suivante.
- Travaille sur une **branche** (`feat/...`, `fix/...`) et ouvre une **pull request** vers `main` ; ne pousse jamais directement sur `main`.
- Toute logique métier va dans l'API, jamais dans le web. Toute règle financière a des tests (unitaires et, si elle touche la base, d'intégration).
- Toute migration de base : fichier Drizzle + SQL manuel si besoin (RLS, déclencheurs) ; vérifie que `security.int-spec.ts` passe encore.
- Mets à jour la documentation (`docs/`) dans la même PR que le code.
