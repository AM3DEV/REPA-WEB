# Gestion de Fiches de Maintenance

Application web professionnelle de **suivi et gestion des fiches de maintenance / réparation**, avec interface 100 % en français. Conçue pour les ateliers de réparation informatique et électronique (ordinateurs, imprimantes, DVR/CCTV, smartphones, tablettes...).

![PHP](https://img.shields.io/badge/PHP-8.0%2B-777BB4?logo=php&logoColor=white)
![Supabase](https://img.shields.io/badge/Supabase-PostgreSQL%20REST-3ECF8E?logo=supabase&logoColor=white)
![Tailwind CSS](https://img.shields.io/badge/Tailwind_CSS-CDN-06B6D4?logo=tailwindcss&logoColor=white)
![Licence](https://img.shields.io/badge/Licence-MIT-blue)

---

## Sommaire

- [Fonctionnalités](#fonctionnalités)
- [Aperçu](#aperçu)
- [Stack technique](#stack-technique)
- [Structure du projet](#structure-du-projet)
- [Prérequis](#prérequis)
- [Installation](#installation)
- [Identifiants par défaut](#identifiants-par-défaut)
- [Utilisation](#utilisation)
- [API interne](#api-interne)
- [Schéma de la base de données](#schéma-de-la-base-de-données)
- [Sécurité](#sécurité)
- [Dépannage](#dépannage)
- [Licence](#licence)

---

## Fonctionnalités

| Fonctionnalité | Description |
|---|---|
| Tableau de bord | 4 cartes KPI (Total, En attente, En cours, Terminés) cliquables pour filtrer le tableau |
| Fiches de maintenance | Création, modification et suivi complet des fiches (client, téléphone, matériel, modèle, panne, prix, statut) |
| Numérotation automatique | Numéros générés au format `#00XXX` basés sur le dernier numéro existant |
| Recherche instantanée | Filtrage en direct du tableau à la frappe (insensible à la casse et aux accents) sur toutes les colonnes |
| Changement de statut en ligne | PATCH immédiat via AJAX depuis la liste (`En attente` / `En cours` / `Terminé`) |
| Notification WhatsApp | Ouverture de `wa.me` avec numéro au format international Maroc (`06`/`07`/`05` → `212...`) et message pré-rempli en français |
| Reçu imprimable | Ticket 80 mm / A5 (`recu.php`) avec en-tête, garantie et impression automatique (`window.print()`) |
| Mode sombre | Bascule clair/sombre persistante (localStorage) |
| Statistiques | Revenu total (DH) des fiches terminées, répartition par statut |
| Gestion des utilisateurs | Liste, ajout d'utilisateurs avec rôles `admin` / `technicien` |
| Authentification | Sessions PHP, mots de passe hachés (bcrypt), `login.php` / `logout.php` |

---

## Aperçu

| Tableau de bord (clair) | Tableau de bord (sombre) |
|---|---|
| ![Dashboard clair](docs/dashboard-light.png) | ![Dashboard sombre](docs/dashboard-dark.png) |

| Connexion |
|---|
| ![Page de connexion](docs/login.png) |

---

## Stack technique

- **Backend** : PHP 8+ modulaire (aucune dépendance externe, aucun Composer requis)
- **Base de données** : Supabase (PostgreSQL) via l'API REST PostgREST (`/rest/v1/...`) consommée par cURL
- **Frontend** : HTML5 sémantique, Tailwind CSS (CDN), Font Awesome 6, JavaScript Vanilla (ES6+)
- **Police** : Inter (Google Fonts)
- **Serveur** : compatible XAMPP / Apache / Nginx / serveur intégré PHP (aucun driver `pdo_pgsql` nécessaire)

---

## Structure du projet

```
K/
├── config.php            # Configuration (URL + clé Supabase, helpers, sessions, icône)
├── SupabaseService.php   # Client REST Supabase réutilisable (GET / POST / PATCH / DELETE)
├── index.php             # Tableau de bord : fiches, statistiques, utilisateurs
├── api.php               # Endpoints JSON (AJAX : statut, création, édition, KPI...)
├── login.php             # Page de connexion
├── logout.php            # Déconnexion (destruction de session)
├── recu.php              # Reçu imprimable (80 mm / A5, impression automatique)
├── schema.sql            # Script SQL complet (tables, RLS, données de démonstration)
├── icon.png              # Icône de l'application (remplaçable)
└── docs/                 # Captures d'écran pour la documentation
```

---

## Prérequis

| Composant | Version minimale | Remarque |
|---|---|---|
| PHP | 8.0 | Extensions `curl`, `mbstring`, `json` activées (par défaut sous XAMPP) |
| Projet Supabase | — | Compte gratuit suffisant sur [supabase.com](https://supabase.com) |
| Navigateur | Moderne | Chrome, Edge, Firefox, Safari |

---

## Installation

### 1. Base de données Supabase

1. Créez un projet sur [supabase.com](https://supabase.com).
2. Ouvrez **SQL Editor** et exécutez l'intégralité du fichier [`schema.sql`](schema.sql).
   Cela crée les tables `utilisateurs` et `fiches_maintenance`, les politiques RLS, le compte `admin` et 10 fiches de démonstration.

### 2. Configuration

Copiez le dossier `K` dans `C:\xampp\htdocs\`, puis renseignez vos identifiants dans [`config.php`](config.php) :

```php
define('SUPABASE_URL',      'https://VOTRE_PROJET.supabase.co');
define('SUPABASE_ANON_KEY', 'sb_publishable_XXXXXXXX');
```

> Utilisez la **clé publishable** (ou `anon`). Les variables d'environnement `SUPABASE_URL` et `SUPABASE_ANON_KEY` sont également prises en charge et ont priorité si définies.

Si les identifiants ne sont pas encore configurés, l'application affiche un message d'erreur explicite au lieu de planter.

### 3. Icône de l'application (optionnel)

Déposez votre propre logo (format carré) à la racine du projet sous l'un des noms suivants — il sera détecté automatiquement comme favicon et logo :

`icon.png` → `icon.svg` → `icon.ico` → `icon.jpg/jpeg/webp` → `favicon.png/ico` → `logo.png/jpg`

Un `icon.png` par défaut (clé à molette bleue) est fourni.

### 4. Lancement

Démarrez Apache depuis le panneau XAMPP puis ouvrez :

```
http://localhost/K/login.php
```

---

## Identifiants par défaut

| Nom d'utilisateur | Mot de passe | Rôle |
|---|---|---|
| `admin` | `admin` | admin |

> **Important** : changez ce mot de passe après la première connexion (voir [Sécurité](#sécurité)).

---

## Utilisation

1. **Créer une fiche** — bouton `+ Nouvelle fiche` (barre de navigation ou en-tête du tableau). Le numéro `#00XXX` est généré automatiquement.
2. **Modifier** — icône crayon orange ou lien `Modifier` sur la ligne concernée.
3. **Changer le statut** — menu déroulant dans la colonne `STATUT` : la mise à jour est envoyée immédiatement (AJAX) et les compteurs KPI sont rafraîchis.
4. **Rechercher** — tapez directement dans la barre de recherche : filtrage instantané. Les boutons `×` ou `Échap` effacent la recherche.
5. **Filtrer par statut** — cliquez sur une carte KPI (`Total`, `En attente`, `En cours`, `Terminés`).
6. **Imprimer un reçu** — bouton `Reçu` : le ticket s'ouvre et la boîte d'impression se lance automatiquement.
7. **Contacter le client** — bouton vert WhatsApp : message pré-rempli avec le n° de fiche, l'appareil, le statut et le prix en DH.
8. **Statistiques** — onglet affichant le revenu total (fiches terminées) et la répartition par statut.
9. **Utilisateurs** — onglet permettant d'ajouter des comptes `admin` ou `technicien`.

---

## API interne

Toutes les requêtes exigent une session authentifiée et répondent en JSON (`{"ok": true|false, ...}`).

| Endpoint (`api.php`) | Méthode | Paramètres | Description |
|---|---|---|---|
| `?action=next_number` | GET | — | Retourne le prochain numéro de fiche (`#00XXX`) |
| `?action=save` | POST | `id` (vide = création), `client_nom`, `telephone`, `materiel`, `modele_serie`, `description_panne`, `prix`, `statut` | Crée ou met à jour une fiche |
| `?action=status` | POST | `id`, `statut` | Change le statut d'une fiche |
| `?action=delete` | POST | `id` | Supprime une fiche |
| `?action=kpi` | GET | — | Compteurs par statut + revenu total |
| `?action=utilisateurs` | GET | — | Liste des utilisateurs (sans les hachages) |
| `?action=add_user` | POST | `username`, `password`, `role` | Crée un utilisateur (mot de passe haché bcrypt) |

Le client REST (`SupabaseService.php`) expose `get()`, `post()`, `patch()`, `delete()` et `query()`, et gère les en-têtes `apikey`, `Authorization: Bearer` et `Prefer: return=representation`.

---

## Schéma de la base de données

### `utilisateurs`

| Colonne | Type | Contrainte |
|---|---|---|
| `id` | bigint | PK, identité |
| `username` | text | unique, non nul |
| `password_hash` | text | non nul (bcrypt) |
| `role` | text | `admin` / `technicien` |
| `created_at` | timestamptz | `now()` |

### `fiches_maintenance`

| Colonne | Type | Contrainte |
|---|---|---|
| `id` | bigint | PK, identité |
| `numero_fiche` | text | unique (`#00XXX`) |
| `date_depot` | timestamptz | `now()` |
| `client_nom` | text | non nul |
| `telephone` | text | non nul |
| `materiel` | text | non nul |
| `modele_serie` | text | — |
| `description_panne` | text | défaut `-` |
| `prix` | numeric(10,2) | défaut `0.00` |
| `statut` | text | `En attente` / `En cours` / `Terminé` |
| `created_at` | timestamptz | `now()` |

---

## Sécurité

- **Clé API** : n'utilisez que la clé **publishable/anon** côté application. Ne placez **jamais** la clé `sb_secret_...` (ou `service_role`) dans le code — elle contourne les politiques RLS.
- **Row Level Security** : `schema.sql` active RLS sur les deux tables avec des politiques CRUD explicites. Pour un environnement multi-tenant, remplacez-les par des politiques basées sur `auth.uid()`.
- **Mots de passe** : hachés avec `password_hash()` / `password_verify()` (bcrypt). Changez le mot de passe `admin` par défaut via l'onglet **Utilisateurs** (nouveau compte) puis suppression/rotation de l'ancien.
- **Sessions** : l'accès au tableau de bord, au reçu et aux endpoints API est refusé sans session valide (redirection vers `login.php`).
- **HTTPS** : en production, servez l'application derrière HTTPS pour protéger les sessions.

---

## Dépannage

| Symptôme | Cause probable | Solution |
|---|---|---|
| « Supabase non configuré » | Identifiants non renseignés dans `config.php` | Renseignez `SUPABASE_URL` et `SUPABASE_ANON_KEY` |
| Identifiants corrects refusés | `schema.sql` non exécuté (table vide) | Exécutez `schema.sql` dans le SQL Editor Supabase |
| Erreur `401` ou `403` sur les requêtes REST | Politiques RLS absentes | Vérifiez que les policies de `schema.sql` ont bien été créées |
| Le favicon ne change pas | Cache navigateur | Videz le cache ou forcez le rafraîchissement (`Ctrl+F5`) |
| Caractères accentués mal affichés | Ancienne version de fichier | Les fichiers sont encodés en UTF-8 — vérifiez votre éditeur |
| Extension cURL manquante | `php_curl` désactivée | Activez `extension=curl` dans `php.ini` puis redémarrez Apache |

---

## Licence

Distribué sous licence **MIT**. Vous êtes libre de l'utiliser, le modifier et le distribuer, y compris à des fins commerciales.
