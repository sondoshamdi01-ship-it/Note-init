# 🚀 TP Git pour débutants — Travail en équipe sur une « Application de Notes »

> **Public** : personnes qui découvrent Git pour la première fois.
> **Durée estimée** : 4 à 6 heures (peut être découpé en 2 séances).
> **Format** : TP à 2 personnes → un **Leader** et un **Member**.
> **Application fil rouge** : une application de notes (CRUD + recherche + catégories) en **React + Node.js/Express + PostgreSQL**, packagée avec **Docker** (architecture **monolithe**) et organisée en **MVC**.

---

## 📑 Sommaire

1. [Objectifs pédagogiques](#1-objectifs-pédagogiques)
2. [Prérequis et organisation](#2-prérequis-et-organisation)
3. [Comprendre Git en 10 minutes](#3-comprendre-git-en-10-minutes)
4. [Présentation de l'application et de l'architecture](#4-présentation-de-lapplication-et-de-larchitecture)
5. [Le scénario complet (vue d'ensemble)](#5-le-scénario-complet-vue-densemble)
6. [Étape 0 — Configuration de Git (Leader + Member)](#6-étape-0--configuration-de-git-leader--member)
7. [Étape 1 — Le Leader crée le projet et fait le premier push](#7-étape-1--le-leader-crée-le-projet-et-fait-le-premier-push)
8. [Étape 2 — Le Member clone et pousse une fonctionnalité sur `main`](#8-étape-2--le-member-clone-et-pousse-une-fonctionnalité-sur-main)
9. [Étape 3 — Le Leader oublie le `pull` → conflit !](#9-étape-3--le-leader-oublie-le-pull--conflit-)
10. [Étape 4 — La solution : travailler avec des branches](#10-étape-4--la-solution--travailler-avec-des-branches)
11. [Étape 5 — Les tags (versions)](#11-étape-5--les-tags-versions)
12. [Étape 6 — Commandes utiles en plus (bonus)](#12-étape-6--commandes-utiles-en-plus-bonus)
13. [Aide-mémoire (cheat sheet)](#13-aide-mémoire-cheat-sheet)
14. [Erreurs fréquentes et solutions](#14-erreurs-fréquentes-et-solutions)
15. [Exercices finaux et grille d'évaluation](#15-exercices-finaux-et-grille-dévaluation)

---

## 1. Objectifs pédagogiques

À la fin du TP, vous saurez :

| # | Compétence |
|---|-----------|
| 1 | Configurer Git (`user.name`, `user.email`, branche par défaut…) |
| 2 | Initialiser un dépôt, faire des commits clairs, faire un premier `push` |
| 3 | Utiliser `.gitignore` pour ne pas versionner les fichiers inutiles ou secrets |
| 4 | Cloner un dépôt existant |
| 5 | Créer, utiliser et fusionner des branches |
| 6 | Utiliser `fetch`, `pull`, `push` correctement |
| 7 | **Provoquer, comprendre et résoudre un conflit** |
| 8 | Utiliser les Pull Requests (ou un merge local) |
| 9 | Créer et pousser des tags (versions `v1.0.0`) |
| 10 | Utiliser quelques commandes de secours (`stash`, `restore`, `revert`, `reset`, `reflog`) |

---

## 2. Prérequis et organisation

### 2.1 Logiciels à installer (sur chaque poste)

| Outil | Version conseillée | Vérification |
|-------|--------------------|--------------|
| Git | ≥ 2.40 | `git --version` |
| Node.js | ≥ 20 (optionnel si tout tourne dans Docker) | `node -v` |
| Docker + Docker Compose | Docker Desktop récent | `docker --version` et `docker compose version` |
| Éditeur | VS Code (recommandé) | — |
| Compte GitHub | gratuit | https://github.com |

### 2.2 Les rôles

- 👑 **Leader** : crée le projet, le dépôt GitHub, et invite le Member.
- 🧑‍💻 **Member** : clone le projet et contribue.

> 💡 **Vous êtes seul(e) ?** Vous pouvez jouer les deux rôles avec **deux dossiers** différents sur votre machine (`~/tp/leader` et `~/tp/member`) et deux configurations Git locales différentes. Voir [l'option « sans GitHub »](#option-sans-github--dépôt-distant-local).

### 2.3 Créer le dépôt distant (Leader)

1. Sur GitHub : **New repository**.
2. Nom : `notes-app`.
3. **Ne cochez rien** (pas de README, pas de .gitignore, pas de licence) → dépôt **vide**. Pourquoi ? Pour éviter un historique créé côté GitHub qui compliquerait le premier push.
4. **Settings → Collaborators → Add people** : ajoutez le compte GitHub du Member.
5. Le Member accepte l'invitation (email ou notification GitHub).

### Option « sans GitHub » : dépôt distant local

Un dépôt « bare » joue le rôle de GitHub :

```bash
mkdir -p ~/tp && cd ~/tp
git init --bare notes-app.git      # le « serveur »
# Ensuite, remplacez partout l'URL GitHub par : ~/tp/notes-app.git
# (Les Pull Requests n'existent pas : on fusionne en local avec git merge)
```

---

## 3. Comprendre Git en 10 minutes

### 3.1 Git, c'est quoi ?

**Git** est un système de **gestion de versions décentralisé** : il enregistre l'historique de votre projet (qui a changé quoi, quand, pourquoi) et permet à plusieurs personnes de travailler en parallèle.

> ⚠️ **Git ≠ GitHub.** Git est l'outil installé sur votre PC. GitHub (ou GitLab, Bitbucket) est un **service en ligne** qui héberge des dépôts Git.

### 3.2 Les 4 zones à connaître

```
 Working Directory        Staging Area          Local Repository         Remote Repository
 (vos fichiers)           (index)               (.git, historique)       (GitHub = origin)
 ┌───────────────┐  add   ┌────────────┐ commit ┌──────────────────┐ push ┌───────────────┐
 │ Je modifie    │ ─────▶ │ Je prépare │ ─────▶ │ Je sauvegarde    │ ───▶ │ Je partage    │
 │ des fichiers  │        │ mon commit │        │ une version      │      │ avec l'équipe │
 └───────────────┘        └────────────┘        └──────────────────┘ ◀─── └───────────────┘
        ▲                                                              pull / fetch
        └────────────────────────── restore / checkout ────────────────────────┘
```

| Zone | Rôle | Commande pour y entrer |
|------|------|------------------------|
| **Working directory** | Vos fichiers du quotidien | (vous éditez) |
| **Staging area (index)** | « Panier » de ce qui ira dans le prochain commit | `git add` |
| **Local repository** | Historique complet, dans le dossier caché `.git` | `git commit` |
| **Remote repository** | Copie partagée (nom par défaut : `origin`) | `git push` |

### 3.3 Vocabulaire indispensable

| Mot | Définition simple |
|-----|-------------------|
| **Repository (repo)** | Le projet + tout son historique |
| **Commit** | Une « photo » du projet à un instant T, avec un message et un auteur |
| **Branch (branche)** | Une ligne de développement parallèle. `main` = la branche principale |
| **HEAD** | Un pointeur : « où suis-je actuellement ? » (en général : la branche courante) |
| **origin** | Nom par défaut du dépôt distant |
| **Clone** | Copie complète d'un dépôt distant |
| **Fetch** | Télécharge les nouveautés du distant **sans** modifier votre travail |
| **Pull** | `fetch` + `merge` : télécharge **et** fusionne dans votre branche |
| **Push** | Envoie vos commits vers le distant |
| **Merge** | Fusionne deux branches |
| **Conflict** | Git ne sait pas fusionner automatiquement : deux personnes ont modifié **les mêmes lignes** |
| **Tag** | Étiquette fixe sur un commit (ex : `v1.0.0`) |
| **Pull Request (PR)** | Demande de fusion sur GitHub, avec revue de code |

### 3.4 Le cycle de travail de base

```bash
git status            # 1. Où en suis-je ?
git add <fichier>     # 2. Je prépare
git commit -m "..."   # 3. Je sauvegarde localement
git pull              # 4. Je récupère le travail des autres
git push              # 5. J'envoie mon travail
```

---

## 4. Présentation de l'application et de l'architecture

### 4.1 Fonctionnalités

- ✅ Créer une note
- ✅ Modifier et supprimer une note
- ✅ Rechercher une note (titre ou contenu)
- ✅ Organiser les notes par catégorie (Général, Travail, Perso, Idées)

### 4.2 Technologies

| Couche | Technologie |
|--------|-------------|
| Frontend | **React** (avec Vite) |
| Backend | **Node.js + Express** (API REST) *— « backend JavaScript »* |
| Base de données | **PostgreSQL 16** |
| Conteneurs | **Docker + Docker Compose** |
| Architecture | **Monolithe** + **MVC** |

### 4.3 Qu'est-ce qu'un « monolithe » ici ?

Un seul **service applicatif** (un seul conteneur `app`) contient **tout** : l'API Express **et** le frontend React compilé (fichiers statiques servis par Express). Seule la base de données est dans un second conteneur (c'est normal : on ne met jamais la base dans le même conteneur que l'application).

```
                  Navigateur (http://localhost:3000)
                          │
        ┌─────────────────▼─────────────────┐
        │     Conteneur « app » (Node.js)   │
        │  ┌─────────────────────────────┐  │
        │  │ Express                     │  │
        │  │  /api/notes  → API REST     │  │
        │  │  /           → React (build)│  │
        │  └─────────────────────────────┘  │
        └─────────────────┬─────────────────┘
                          │ SQL (port 5432)
        ┌─────────────────▼─────────────────┐
        │   Conteneur « db » (PostgreSQL)   │
        │   volume : pgdata                 │
        └───────────────────────────────────┘
```

**Pourquoi un monolithe pour apprendre Git ?** Un seul dépôt, un seul `docker compose up`, pas de complexité réseau/microservices : on se concentre sur **Git**.

### 4.4 Qu'est-ce que MVC ?

**MVC = Model – View – Controller** : on sépare les responsabilités.

| Lettre | Rôle | Où dans le projet |
|--------|------|-------------------|
| **M**odel | Accès aux données (requêtes SQL) | `backend/src/models/noteModel.js` |
| **V**iew | Ce que voit l'utilisateur | `frontend/src/` (React) |
| **C**ontroller | Reçoit la requête HTTP, valide, appelle le modèle, renvoie la réponse | `backend/src/controllers/noteController.js` |
| *(Routes)* | Associe une URL à un contrôleur | `backend/src/routes/noteRoutes.js` |

Flux d'une requête « créer une note » :

```
React (View) ──POST /api/notes──▶ Route ──▶ Controller ──▶ Model ──▶ PostgreSQL
     ▲                                                                   │
     └────────────── JSON de la nouvelle note ◀──────────────────────────┘
```

### 4.5 Structure finale du projet

```
notes-app/
├── .gitignore
├── .env.example            ← modèle de configuration (versionné)
├── .env                    ← vraie configuration (NON versionné !)
├── .dockerignore
├── Dockerfile              ← construit l'image monolithe
├── docker-compose.yml      ← app + db
├── README.md
├── backend/
│   ├── package.json
│   └── src/
│       ├── server.js
│       ├── config/db.js
│       ├── models/noteModel.js
│       ├── controllers/noteController.js
│       └── routes/noteRoutes.js
└── frontend/
    ├── package.json
    ├── vite.config.js
    ├── index.html
    └── src/
        ├── main.jsx
        ├── App.jsx
        ├── api.js
        ├── categories.js
        ├── styles.css
        └── components/
            ├── NoteForm.jsx
            ├── NoteList.jsx
            ├── NoteItem.jsx
            ├── SearchBar.jsx
            └── CategoryFilter.jsx
```

### 4.6 API REST (finale)

| Méthode | URL | Description |
|---------|-----|-------------|
| GET | `/api/notes` | Liste des notes (`?search=mot` pour rechercher) |
| POST | `/api/notes` | Crée une note |
| PUT | `/api/notes/:id` | Modifie une note |
| DELETE | `/api/notes/:id` | Supprime une note |

> 🎯 **Important** : l'application est **construite progressivement** pendant le TP. Chaque fonctionnalité correspond à une étape Git.

---

## 5. Le scénario complet (vue d'ensemble)

| Étape | Qui | Action Git | Fonctionnalité |
|-------|-----|------------|----------------|
| 0 | Les deux | `git config` | — |
| 1 | 👑 Leader | `init`, `add`, `commit`, `.gitignore`, `remote`, `push` sur `main` | Base : créer + lister les notes, Docker, MVC |
| 2 | 🧑‍💻 Member | `clone`, `commit`, `push` sur `main` | Recherche + changement du titre |
| 3 | 👑 Leader | modifie `main` **sans `pull`** → push refusé → `pull` → **CONFLIT** → résolution | Changement du titre (même ligne !) |
| 4 | Les deux | `switch -c`, branches, PR, merge, **2ᵉ conflit guidé** | Leader : catégories — Member : modifier/supprimer |
| 5 | 👑 Leader | `tag v1.0.0`, `push --tags` | Version 1.0.0 |
| 6 | Les deux | `stash`, `revert`, `reset`, `reflog`… | Bonus |

---

## 6. Étape 0 — Configuration de Git (Leader + Member)

> **Les deux personnes** exécutent ces commandes **chacune sur son poste**, avec **leurs propres** nom et email.

### 6.1 Identité (obligatoire)

```bash
git config --global user.name  "Prénom Nom"
git config --global user.email "votre.email@exemple.com"
```

📖 **Explication** :
- Chaque commit est signé par un nom + un email : c'est ce que vous voyez dans `git log`.
- `--global` = valable pour **tous** vos projets sur ce PC.
- Utilisez **le même email que celui de votre compte GitHub** pour que vos commits soient reliés à votre profil.

### 6.2 Réglages recommandés

```bash
# Nom de la branche créée par "git init" (au lieu de "master")
git config --global init.defaultBranch main

# Comportement de "git pull" : fusionner (merge) — plus simple pour débuter
git config --global pull.rebase false

# Éditeur ouvert quand Git a besoin d'un message long (ex : merge)
git config --global core.editor "code --wait"      # VS Code
# git config --global core.editor "nano"           # alternative simple en terminal

# Couleurs (souvent déjà actif)
git config --global color.ui auto
```

### 6.3 Vérifier

```bash
git config --list --show-origin
git config user.name
git config user.email
```

### 6.4 Les 3 niveaux de configuration

| Niveau | Option | Fichier | Portée |
|--------|--------|---------|--------|
| Système | `--system` | `/etc/gitconfig` | Tous les utilisateurs de la machine |
| Global | `--global` | `~/.gitconfig` | Tous vos projets |
| Local | `--local` (défaut) | `.git/config` | Ce dépôt uniquement |

> Le niveau le plus précis **gagne**. Ex : pour utiliser un autre email dans un seul projet : `git config user.email "autre@mail.com"` (sans `--global`, dans le dossier du projet).

### 6.5 (Optionnel) Alias pratiques

```bash
git config --global alias.st "status -sb"
git config --global alias.lg "log --oneline --graph --decorate --all"
```
Ensuite : `git st` et `git lg`.

### 6.6 Authentification GitHub

GitHub n'accepte plus le mot de passe en HTTPS. Deux solutions :
- **HTTPS + Personal Access Token** (Settings → Developer settings → Tokens) ou via *Git Credential Manager* (installé avec Git pour Windows), ou `gh auth login` (GitHub CLI).
- **SSH** : `ssh-keygen -t ed25519 -C "email"` puis ajouter la clé publique sur GitHub (Settings → SSH keys).

✅ **Point de contrôle** : `git config user.name` et `git config user.email` affichent vos valeurs.

---

## 7. Étape 1 — Le Leader crée le projet et fait le premier push

> 👑 **Rôle : Leader.** Le Member attend (ou lit l'étape pour comprendre).

### 7.1 Créer le dossier et initialiser Git

```bash
mkdir notes-app
cd notes-app
git init
```

📖 `git init` crée le dossier caché `.git/` : c'est **lui** qui contient tout l'historique. Supprimer `.git` = perdre l'historique (les fichiers restent).

```bash
git status
```
Résultat attendu : `On branch main` / `No commits yet` / `nothing to commit`.

### 7.2 Commit n°1 — `.gitignore`, `.env.example`, `README`

#### 📌 Pourquoi `.gitignore` **en premier** ?

On ne veut **jamais** versionner :
- `node_modules/` (des centaines de Mo, régénérable avec `npm install`),
- `dist/` (résultat de compilation),
- `.env` (**mots de passe, secrets** !),
- fichiers de l'OS/éditeur (`.DS_Store`, `.vscode/`…).

Si on les commit par erreur, ils restent **dans l'historique** même après suppression → il vaut mieux les ignorer **dès le départ**.

**`.gitignore`**
```gitignore
# Dépendances
node_modules/

# Builds
dist/
build/

# Secrets et configuration locale
.env

# Logs
*.log
npm-debug.log*

# Fichiers système / éditeurs
.DS_Store
Thumbs.db
.vscode/
.idea/
```

📖 **Syntaxe** : une règle par ligne ; `dossier/` ignore un dossier ; `*.log` ignore par motif ; `#` = commentaire ; `!fichier` = exception.

**`.env.example`** (modèle sans secret réel, **versionné**)
```env
DB_USER=notes
DB_PASSWORD=change_me
DB_NAME=notesdb
```

**`.env`** (vraie config, **ignoré**)
```bash
cp .env.example .env
```

**`README.md`**
```markdown
# Notes App

Application de notes (React + Express + PostgreSQL) — projet de TP Git.

## Démarrage
1. `cp .env.example .env`
2. `docker compose up --build`
3. Ouvrir http://localhost:3000
```

Vérifions que `.env` est bien ignoré :

```bash
git status
git check-ignore -v .env
```
- `git status` ne doit **pas** lister `.env`.
- `git check-ignore -v .env` indique quelle règle l'ignore.

Premier commit :

```bash
git add .gitignore .env.example README.md
git status                  # fichiers en vert = dans le staging
git commit -m "chore: initialise le projet (gitignore, env example, readme)"
git log --oneline
```

📖 **Explications** :
- `git add <fichiers>` : place les fichiers dans la **staging area**.
- `git commit -m "..."` : enregistre un commit avec un message.
- **Convention de messages (Conventional Commits)** — fortement conseillée :
  - `feat:` nouvelle fonctionnalité · `fix:` correction · `docs:` documentation · `chore:` tâche technique · `refactor:` · `style:` · `test:`
- Un bon message : court (≤ 72 caractères), à l'impératif ou au présent, explique **quoi/pourquoi**. ❌ « modif », « test », « ça marche enfin ».

### 7.3 Commit n°2 — le Backend (MVC)

Créez l'arborescence :
```bash
mkdir -p backend/src/config backend/src/models backend/src/controllers backend/src/routes
```

**`backend/package.json`**
```json
{
  "name": "notes-backend",
  "version": "1.0.0",
  "type": "module",
  "scripts": {
    "start": "node src/server.js",
    "dev": "node --watch src/server.js"
  },
  "dependencies": {
    "express": "^4.19.2",
    "pg": "^8.12.0"
  }
}
```
> `"type": "module"` permet d'utiliser `import/export` (ES Modules).

**`backend/src/config/db.js`** — connexion PostgreSQL + création de la table
```js
import pg from 'pg';

// Pool = ensemble de connexions réutilisables vers PostgreSQL
export const pool = new pg.Pool({
  host: process.env.DB_HOST || 'localhost',
  port: Number(process.env.DB_PORT) || 5432,
  user: process.env.DB_USER || 'notes',
  password: process.env.DB_PASSWORD || 'notes',
  database: process.env.DB_NAME || 'notesdb',
});

// Crée la table au démarrage si elle n'existe pas encore
export async function initDb() {
  await pool.query(`
    CREATE TABLE IF NOT EXISTS notes (
      id         SERIAL PRIMARY KEY,
      title      VARCHAR(150) NOT NULL,
      content    TEXT NOT NULL DEFAULT '',
      created_at TIMESTAMP NOT NULL DEFAULT NOW()
    )
  `);
}
```

**`backend/src/models/noteModel.js`** — **M**odel : uniquement du SQL
```js
import { pool } from '../config/db.js';

export async function findAll() {
  const { rows } = await pool.query(
    'SELECT * FROM notes ORDER BY created_at DESC'
  );
  return rows;
}

export async function create({ title, content }) {
  const { rows } = await pool.query(
    'INSERT INTO notes (title, content) VALUES ($1, $2) RETURNING *',
    [title, content]
  );
  return rows[0];
}
```
> 🔒 `$1`, `$2` = **requêtes paramétrées** : protègent contre l'injection SQL. Ne concaténez jamais les valeurs dans la chaîne SQL.

**`backend/src/controllers/noteController.js`** — **C**ontroller : valide et orchestre
```js
import * as Note from '../models/noteModel.js';

export async function list(req, res, next) {
  try {
    res.json(await Note.findAll());
  } catch (err) {
    next(err);
  }
}

export async function create(req, res, next) {
  try {
    const { title, content } = req.body;
    if (!title || !title.trim()) {
      return res.status(400).json({ error: 'Le titre est obligatoire' });
    }
    const note = await Note.create({
      title: title.trim(),
      content: content || '',
    });
    res.status(201).json(note);
  } catch (err) {
    next(err);
  }
}
```

**`backend/src/routes/noteRoutes.js`**
```js
import { Router } from 'express';
import * as ctrl from '../controllers/noteController.js';

const router = Router();

router.get('/', ctrl.list);
router.post('/', ctrl.create);

export default router;
```

**`backend/src/server.js`** — point d'entrée (sert l'API **et** le frontend compilé : le monolithe)
```js
import express from 'express';
import path from 'path';
import { fileURLToPath } from 'url';
import noteRoutes from './routes/noteRoutes.js';
import { initDb } from './config/db.js';

const __dirname = path.dirname(fileURLToPath(import.meta.url));
const app = express();

app.use(express.json());

// API
app.use('/api/notes', noteRoutes);

// Frontend React compilé (copié dans /app/public par le Dockerfile)
const publicDir = path.join(__dirname, '../public');
app.use(express.static(publicDir));
app.get('*', (req, res) => res.sendFile(path.join(publicDir, 'index.html')));

// Gestion centralisée des erreurs
app.use((err, req, res, next) => {
  console.error(err);
  res.status(500).json({ error: 'Erreur serveur' });
});

const PORT = process.env.PORT || 3000;

initDb()
  .then(() => app.listen(PORT, () => console.log(`✅ Serveur sur le port ${PORT}`)))
  .catch((err) => {
    console.error('❌ Connexion base de données impossible', err);
    process.exit(1);
  });
```

Commit :
```bash
git status
git add backend/
git commit -m "feat(backend): API REST des notes (liste + création) en MVC"
```

### 7.4 Commit n°3 — le Frontend React

```bash
mkdir -p frontend/src/components
```

**`frontend/package.json`**
```json
{
  "name": "notes-frontend",
  "private": true,
  "version": "1.0.0",
  "type": "module",
  "scripts": {
    "dev": "vite",
    "build": "vite build"
  },
  "dependencies": {
    "react": "^18.3.1",
    "react-dom": "^18.3.1"
  },
  "devDependencies": {
    "@vitejs/plugin-react": "^4.3.1",
    "vite": "^5.4.0"
  }
}
```

**`frontend/vite.config.js`**
```js
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';

export default defineConfig({
  plugins: [react()],
  server: {
    // En développement local (npm run dev), redirige /api vers le backend
    proxy: { '/api': 'http://localhost:3000' },
  },
});
```

**`frontend/index.html`**
```html
<!doctype html>
<html lang="fr">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Notes App</title>
  </head>
  <body>
    <div id="root"></div>
    <script type="module" src="/src/main.jsx"></script>
  </body>
</html>
```

**`frontend/src/main.jsx`**
```jsx
import React from 'react';
import ReactDOM from 'react-dom/client';
import App from './App.jsx';
import './styles.css';

ReactDOM.createRoot(document.getElementById('root')).render(
  <React.StrictMode>
    <App />
  </React.StrictMode>
);
```

**`frontend/src/api.js`** — toutes les requêtes HTTP au même endroit
```js
const BASE = '/api/notes';

async function request(url, options = {}) {
  const res = await fetch(url, {
    headers: { 'Content-Type': 'application/json' },
    ...options,
  });
  if (!res.ok) {
    const data = await res.json().catch(() => ({}));
    throw new Error(data.error || 'Erreur réseau');
  }
  return res.status === 204 ? null : res.json();
}

export const api = {
  getNotes: () => request(BASE),
  createNote: (note) =>
    request(BASE, { method: 'POST', body: JSON.stringify(note) }),
};
```

**`frontend/src/App.jsx`**
```jsx
import { useEffect, useState } from 'react';
import { api } from './api.js';
import NoteForm from './components/NoteForm.jsx';
import NoteList from './components/NoteList.jsx';

export default function App() {
  const [notes, setNotes] = useState([]);

  async function loadNotes() {
    setNotes(await api.getNotes());
  }

  useEffect(() => {
    loadNotes();
  }, []);

  async function handleCreate(note) {
    await api.createNote(note);
    await loadNotes();
  }

  return (
    <div className="container">
      <h1>📝 Notes App</h1>
      <NoteForm onCreate={handleCreate} />
      <NoteList notes={notes} />
    </div>
  );
}
```

**`frontend/src/components/NoteForm.jsx`**
```jsx
import { useState } from 'react';

export default function NoteForm({ onCreate }) {
  const [title, setTitle] = useState('');
  const [content, setContent] = useState('');

  async function handleSubmit(e) {
    e.preventDefault();
    await onCreate({ title, content });
    setTitle('');
    setContent('');
  }

  return (
    <form onSubmit={handleSubmit} className="form">
      <input
        value={title}
        onChange={(e) => setTitle(e.target.value)}
        placeholder="Titre"
        required
      />
      <textarea
        value={content}
        onChange={(e) => setContent(e.target.value)}
        placeholder="Contenu de la note"
      />
      <button type="submit">Ajouter</button>
    </form>
  );
}
```

**`frontend/src/components/NoteList.jsx`**
```jsx
import NoteItem from './NoteItem.jsx';

export default function NoteList({ notes }) {
  if (notes.length === 0) return <p>Aucune note pour le moment.</p>;

  return (
    <ul className="notes">
      {notes.map((n) => (
        <NoteItem key={n.id} note={n} />
      ))}
    </ul>
  );
}
```

**`frontend/src/components/NoteItem.jsx`**
```jsx
export default function NoteItem({ note }) {
  return (
    <li className="note">
      <h3>{note.title}</h3>
      <p>{note.content}</p>
      <small>{new Date(note.created_at).toLocaleString()}</small>
    </li>
  );
}
```

**`frontend/src/styles.css`**
```css
* { box-sizing: border-box; }
body { font-family: system-ui, sans-serif; background: #f4f6f8; margin: 0; }
.container { max-width: 720px; margin: 2rem auto; padding: 0 1rem; }
.form { display: flex; flex-direction: column; gap: .5rem; margin-bottom: 1rem; }
input, textarea, select { padding: .6rem; border: 1px solid #ccd; border-radius: 6px; font: inherit; }
button { padding: .6rem 1rem; border: 0; border-radius: 6px; background: #3b82f6; color: #fff; cursor: pointer; }
button.danger { background: #ef4444; }
button.secondary { background: #64748b; }
.notes { list-style: none; padding: 0; display: grid; gap: .75rem; }
.note { background: #fff; padding: 1rem; border-radius: 8px; box-shadow: 0 1px 3px #0002; }
.note h3 { margin: 0 0 .4rem; }
.note small { color: #64748b; }
.badge { display: inline-block; background: #e0e7ff; color: #3730a3; padding: .1rem .5rem; border-radius: 999px; font-size: .75rem; }
.actions { display: flex; gap: .5rem; margin-top: .6rem; }
.toolbar { display: flex; gap: .5rem; margin-bottom: 1rem; }
.toolbar input { flex: 1; }
```

Commit :
```bash
git add frontend/
git status
git commit -m "feat(frontend): interface React pour créer et lister les notes"
```

### 7.5 Commit n°4 — Docker (monolithe)

**`Dockerfile`** — *multi-stage build*
```dockerfile
# ---------- Étape 1 : compiler le frontend React ----------
FROM node:20-alpine AS front-build
WORKDIR /app/frontend
COPY frontend/package*.json ./
RUN npm install
COPY frontend/ ./
RUN npm run build            # produit /app/frontend/dist

# ---------- Étape 2 : image finale (backend + frontend compilé) ----------
FROM node:20-alpine
WORKDIR /app
COPY backend/package*.json ./
RUN npm install --omit=dev
COPY backend/src ./src
COPY --from=front-build /app/frontend/dist ./public
EXPOSE 3000
CMD ["npm", "start"]
```

📖 **Explications** :
- **Multi-stage** : l'étape 1 contient Vite et toutes les dépendances de build (lourdes) ; l'étape 2 ne garde que le **résultat** (`dist`) → image finale légère.
- `COPY package*.json` puis `npm install` **avant** de copier le code : Docker met en cache cette couche → rebuilds plus rapides tant que les dépendances ne changent pas.
- `COPY --from=front-build` : récupère uniquement le dossier compilé de l'étape 1 et le place dans `public/`, que `server.js` sert.

**`docker-compose.yml`**
```yaml
services:
  db:
    image: postgres:16-alpine
    restart: unless-stopped
    environment:
      POSTGRES_USER: ${DB_USER}
      POSTGRES_PASSWORD: ${DB_PASSWORD}
      POSTGRES_DB: ${DB_NAME}
    volumes:
      - pgdata:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U ${DB_USER} -d ${DB_NAME}"]
      interval: 5s
      timeout: 5s
      retries: 10

  app:
    build: .
    restart: unless-stopped
    ports:
      - "3000:3000"
    environment:
      DB_HOST: db
      DB_PORT: 5432
      DB_USER: ${DB_USER}
      DB_PASSWORD: ${DB_PASSWORD}
      DB_NAME: ${DB_NAME}
    depends_on:
      db:
        condition: service_healthy

volumes:
  pgdata:
```

📖 **Explications** :
- `${DB_USER}` etc. sont lus depuis le fichier **`.env`** (qui n'est pas dans Git → pas de secret publié).
- `DB_HOST: db` : dans le réseau Docker Compose, le service s'appelle `db`, c'est aussi son nom d'hôte.
- `healthcheck` + `depends_on: condition: service_healthy` : l'app démarre **seulement quand PostgreSQL est prêt**.
- `volumes: pgdata` : les données survivent à l'arrêt des conteneurs.

**`.dockerignore`** (accélère le build, évite de copier `node_modules` ou `.env` dans l'image)
```
node_modules
**/node_modules
.git
.env
dist
```

### 7.6 Tester l'application **avant** de pousser

```bash
docker compose up --build
```
Ouvrez http://localhost:3000, créez une note. ✅ Si ça marche : `Ctrl+C` puis :

```bash
git add Dockerfile docker-compose.yml .dockerignore
git commit -m "chore(docker): ajoute Dockerfile monolithe et docker-compose"
git log --oneline
```

Résultat attendu (les identifiants changent) :
```
d4e5f6a chore(docker): ajoute Dockerfile monolithe et docker-compose
c3d4e5f feat(frontend): interface React pour créer et lister les notes
b2c3d4e feat(backend): API REST des notes (liste + création) en MVC
a1b2c3d chore: initialise le projet (gitignore, env example, readme)
```

### 7.7 Premier push vers GitHub

```bash
git remote add origin https://github.com/<leader-username>/notes-app.git
git remote -v
git branch -M main
git push -u origin main
```

📖 **Explications** :
- `git remote add origin <url>` : enregistre l'adresse du dépôt distant sous le nom `origin`.
- `git branch -M main` : renomme (force) la branche courante en `main` (sécurité).
- `git push -u origin main` : envoie la branche `main` vers `origin`. `-u` (`--set-upstream`) **mémorise** le lien entre `main` local et `origin/main` → ensuite, `git push` et `git pull` suffisent, sans arguments.

✅ **Point de contrôle** : rafraîchissez la page GitHub : vous voyez vos fichiers et **4 commits**. Vérifiez que `.env` et `node_modules` **n'apparaissent pas**.

> ⚠️ **Et si j'ai commité `.env` par erreur ?**
> ```bash
> git rm --cached .env          # retire du suivi Git, garde le fichier local
> echo ".env" >> .gitignore
> git commit -am "fix: retire .env du suivi"
> ```
> Mais le secret reste dans l'**historique** : **changez le mot de passe** concerné !

---

## 8. Étape 2 — Le Member clone et pousse une fonctionnalité sur `main`

> 🧑‍💻 **Rôle : Member.** Le Leader a envoyé le lien du dépôt.

### 8.1 Cloner

```bash
cd ~/tp   # un dossier de votre choix
git clone https://github.com/<leader-username>/notes-app.git
cd notes-app
```

📖 `git clone` : télécharge **tout l'historique**, crée le dossier, configure automatiquement `origin`, et se place sur `main`.

### 8.2 Explorer

```bash
git log --oneline
git remote -v
git branch -a          # -a = toutes les branches (locales + distantes)
git status
ls -a                  # remarquez : pas de .env ni de node_modules !
```

### 8.3 Préparer son environnement

Le fichier `.env` n'est pas versionné (c'est voulu) → on le recrée :
```bash
cp .env.example .env
docker compose up --build
```
Testez http://localhost:3000, puis `Ctrl+C`.

### 8.4 Développer la fonctionnalité « Recherche » **directement sur `main`**

> ℹ️ Dans ce TP, on travaille volontairement sur `main` pour provoquer un conflit à l'étape 3. En entreprise, on évite (voir étape 4).

**① Backend — Model** : remplacez `findAll` dans `backend/src/models/noteModel.js` :
```js
export async function findAll(search) {
  if (search) {
    const { rows } = await pool.query(
      `SELECT * FROM notes
       WHERE title ILIKE $1 OR content ILIKE $1
       ORDER BY created_at DESC`,
      [`%${search}%`]
    );
    return rows;
  }
  const { rows } = await pool.query(
    'SELECT * FROM notes ORDER BY created_at DESC'
  );
  return rows;
}
```
> `ILIKE` = `LIKE` insensible à la casse (spécifique PostgreSQL). `%mot%` = « contient mot ».

**② Backend — Controller** : dans `list`, remplacez `Note.findAll()` par :
```js
res.json(await Note.findAll(req.query.search));
```

**③ Frontend — `api.js`** : remplacez `getNotes` :
```js
  getNotes: (search = '') =>
    request(`${BASE}?search=${encodeURIComponent(search)}`),
```

**④ Frontend — nouveau composant `frontend/src/components/SearchBar.jsx`**
```jsx
export default function SearchBar({ value, onChange }) {
  return (
    <input
      className="search"
      type="search"
      placeholder="🔍 Rechercher une note..."
      value={value}
      onChange={(e) => onChange(e.target.value)}
    />
  );
}
```

**⑤ Frontend — `App.jsx`** : modifiez comme suit (4 changements, repérés par `// ← `) :
```jsx
import { useEffect, useState } from 'react';
import { api } from './api.js';
import NoteForm from './components/NoteForm.jsx';
import SearchBar from './components/SearchBar.jsx';        // ← import
import NoteList from './components/NoteList.jsx';

export default function App() {
  const [notes, setNotes] = useState([]);
  const [search, setSearch] = useState('');                 // ← état

  async function loadNotes() {
    setNotes(await api.getNotes(search));                   // ← passe la recherche
  }

  useEffect(() => {
    loadNotes();
  }, [search]);                                             // ← recharge si "search" change

  async function handleCreate(note) {
    await api.createNote(note);
    await loadNotes();
  }

  return (
    <div className="container">
      <h1>📝 Notes App Pro</h1>                             {/* ← titre modifié */}
      <NoteForm onCreate={handleCreate} />
      <SearchBar value={search} onChange={setSearch} />     {/* ← barre de recherche */}
      <NoteList notes={notes} />
    </div>
  );
}
```

> 🎯 **Note pour la suite** : le Member a modifié la ligne du `<h1>`. Le Leader va modifier **cette même ligne**. C'est ce qui créera le conflit.

### 8.3 Tester, puis voir ce qui a changé

```bash
docker compose up --build       # testez la recherche, puis Ctrl+C
git status
git diff                         # détail ligne par ligne des modifications non indexées
```

📖 **Lire un `git diff`** :
```diff
-      <h1>📝 Notes App</h1>      ← ligne supprimée (rouge)
+      <h1>📝 Notes App Pro</h1>  ← ligne ajoutée (vert)
```

### 8.4 Commit et push

```bash
git add .
git status
git commit -m "feat: ajoute la recherche de notes (titre et contenu)"
git pull           # bonne habitude AVANT de pousser (ici : "Already up to date")
git push
```

✅ **Point de contrôle** : le commit apparaît sur GitHub (branche `main`).

> 💡 **Bonne habitude** : toujours `git pull` **avant** `git push`. Le Leader va justement l'oublier…

---

## 9. Étape 3 — Le Leader oublie le `pull` → conflit !

> 👑 **Rôle : Leader.** ⚠️ Le Leader **n'a pas encore récupéré** le travail du Member. **Ne faites surtout pas `git pull` maintenant !**

### 9.1 Le Leader modifie `main` (sans savoir que le Member a poussé)

Dans `frontend/src/App.jsx`, changez **la même ligne** :
```jsx
      <h1>📝 Mes Notes</h1>
```

Ajoutez aussi une ligne dans `README.md` (cette modification-là ne posera aucun problème) :
```markdown
## Fonctionnalités
- Créer, lister des notes
```

```bash
git status
git add .
git commit -m "feat: personnalise le titre et documente les fonctionnalités"
git push
```

### 9.2 💥 Le push est refusé

```
To https://github.com/<leader-username>/notes-app.git
 ! [rejected]        main -> main (fetch first)
error: failed to push some refs to '...'
hint: Updates were rejected because the remote contains work that you do
hint: not have locally. This is usually caused by another repository pushing
hint: to the same ref. You may want to first integrate the remote changes
hint: (e.g., 'git pull ...') before pushing again.
```

📖 **Comprendre l'erreur** — l'historique a **divergé** :

```
                 Remote (origin/main) :   A ── B ── C ── D ── M   (M = commit du Member)
                                                          \
                 Local du Leader      :                    └── L   (L = commit du Leader)
```
Git refuse d'écraser le travail du Member. Il faut d'abord **intégrer** `M` chez le Leader.

### 9.3 Observer avant d'agir (bonne pratique)

```bash
git fetch
git status
git log --oneline --graph --all
git diff main origin/main
```

📖 **`fetch` vs `pull`** :
- `git fetch` : télécharge les nouveautés **sans toucher à vos fichiers** → on regarde d'abord.
- `git pull` = `git fetch` + `git merge origin/main`.

`git status` indique : *Your branch and 'origin/main' have diverged, and have 1 and 1 different commits each.*

### 9.4 Lancer le `pull` → CONFLIT

```bash
git pull
```
Résultat :
```
Auto-merging README.md
Auto-merging frontend/src/App.jsx
CONFLICT (content): Merge conflict in frontend/src/App.jsx
Automatic merge failed; fix conflicts and then commit the result.
```

📖 **Que s'est-il passé ?**
- `README.md` : modifié uniquement par le Leader → fusion **automatique** ✅.
- Fichiers de la recherche (modifiés uniquement par le Member) → fusion **automatique** ✅.
- `App.jsx` : **la même ligne** (`<h1>`) a été modifiée différemment par les deux → Git **ne peut pas deviner** lequel garder ❌ → **conflit**.

La branche est maintenant dans l'état **« merging »**.

```bash
git status
```
```
You have unmerged paths.
  (fix conflicts and run "git commit")
Unmerged paths:
        both modified:   frontend/src/App.jsx
```

### 9.5 Lire les marqueurs de conflit

Ouvrez `frontend/src/App.jsx`. Git y a inséré :

```jsx
<<<<<<< HEAD
      <h1>📝 Mes Notes</h1>
=======
      <h1>📝 Notes App Pro</h1>
>>>>>>> a1b2c3d4e5f6...
```

| Marqueur | Signification |
|----------|---------------|
| `<<<<<<< HEAD` | Début de **votre** version (celle du Leader, branche courante) |
| `=======` | Séparateur |
| `>>>>>>> a1b2c3…` | Fin de la version **entrante** (celle du Member, venant de `origin/main`) |

### 9.6 Résoudre le conflit

**Il faut décider** du résultat final. Trois choix possibles :
1. Garder la version du Leader,
2. Garder la version du Member,
3. **Écrire une version combinée** ← on choisit celle-ci (après discussion entre le Leader et le Member 😉).

Modifiez le fichier pour obtenir **exactement** (et **supprimez les 3 lignes de marqueurs**) :
```jsx
      <h1>📝 Mes Notes Pro</h1>
```

> 💡 **Dans VS Code** : au-dessus du conflit apparaissent les boutons *Accept Current Change*, *Accept Incoming Change*, *Accept Both Changes*, *Compare Changes*. Vous pouvez les utiliser, puis ajuster le texte.
>
> ⚠️ **Piège n°1 des débutants** : laisser les `<<<<<<<`, `=======`, `>>>>>>>` dans le fichier → l'application ne compile plus. Cherchez-les : `grep -rn "<<<<<<<" .`

Vérifiez qu'il ne reste aucun marqueur :
```bash
grep -rn "<<<<<<<\|>>>>>>>" frontend backend README.md
```
(aucune sortie attendue)

### 9.7 Terminer la fusion

```bash
git add frontend/src/App.jsx      # "j'indique que ce conflit est résolu"
git status                         # tout doit être en vert
git commit                         # ouvre l'éditeur avec un message "Merge branch..." pré-rempli
                                   # (ou : git commit -m "merge: résout le conflit sur le titre")
git log --oneline --graph
```

Résultat du graphe :
```
*   9f8e7d6 (HEAD -> main) Merge branch 'main' of https://github.com/...
|\
| * a1b2c3d feat: ajoute la recherche de notes (titre et contenu)
* | 5e4d3c2 feat: personnalise le titre et documente les fonctionnalités
|/
* d4e5f6a chore(docker): ajoute Dockerfile monolithe et docker-compose
```
C'est un **commit de merge** (deux parents).

Testez que tout fonctionne, puis poussez :
```bash
docker compose up --build     # vérifier titre + recherche, puis Ctrl+C
git push
```

### 9.8 Si vous paniquez : annuler la fusion

```bash
git merge --abort
```
Revient exactement à l'état d'avant le `pull`. Aucune perte.

### 9.9 Le Member récupère le résultat

🧑‍💻 **Member** :
```bash
git pull
git log --oneline --graph
```

### 9.10 🧠 Bilan : comment éviter ce conflit ?

| Bonne pratique | Pourquoi |
|----------------|----------|
| `git pull` **avant** de commencer à travailler | Partir de la dernière version |
| `git pull` **avant** de `push` | Résoudre les conflits localement, pas sur le serveur |
| Petits commits fréquents | Conflits plus petits et plus faciles |
| **Utiliser des branches** | Isoler le travail de chacun → étape suivante |
| Communiquer dans l'équipe | « Je touche à App.jsx » |

---

## 10. Étape 4 — La solution : travailler avec des branches

### 10.1 Pourquoi des branches ?

Une **branche** est une ligne de développement indépendante. `main` reste **stable** ; chaque fonctionnalité est développée dans **sa** branche, puis fusionnée une fois terminée et relue.

```
main        A ──── B ──────────────────── M ──── (stable, toujours fonctionnelle)
                    \                      /
feature/x            └── C ── D ── E ────┘
```

**Convention de nommage** : `feature/nom`, `fix/nom`, `hotfix/nom`, `docs/nom`.

### 10.2 Plan de cette étape

| Qui | Branche | Fonctionnalité |
|-----|---------|----------------|
| 👑 Leader | `feature/categories` | Catégories (organiser les notes) |
| 🧑‍💻 Member | `feature/edit-delete` | Modifier / supprimer une note |

> Les deux travaillent **en même temps**, sans se gêner. 🎉

### 10.3 Préparation (les deux)

```bash
git switch main
git pull
git status         # "nothing to commit, working tree clean"
```

### 10.4 Créer sa branche

👑 **Leader** :
```bash
git switch -c feature/categories
```
🧑‍💻 **Member** :
```bash
git switch -c feature/edit-delete
```

📖 `git switch -c <nom>` = **c**réer la branche **et** s'y placer. (Équivalent ancien : `git checkout -b <nom>`.)

Commandes utiles :
```bash
git branch             # liste des branches locales (* = courante)
git branch -a          # + branches distantes
git switch main        # changer de branche
```

---

### 10.5 👑 Leader — fonctionnalité « Catégories » (branche `feature/categories`)

**① `backend/src/config/db.js`** : ajoutez, dans `initDb()`, **après** le `CREATE TABLE` :
```js
  // Ajoute la colonne "category" si elle n'existe pas encore (migration simple)
  await pool.query(`
    ALTER TABLE notes
    ADD COLUMN IF NOT EXISTS category VARCHAR(50) NOT NULL DEFAULT 'General'
  `);
```

**② `backend/src/models/noteModel.js`** : remplacez `create` :
```js
export async function create({ title, content, category }) {
  const { rows } = await pool.query(
    'INSERT INTO notes (title, content, category) VALUES ($1, $2, $3) RETURNING *',
    [title, content, category || 'General']
  );
  return rows[0];
}
```

**③ `backend/src/controllers/noteController.js`** : dans `create` :
```js
    const { title, content, category } = req.body;      // ← ajout de category
    ...
    const note = await Note.create({
      title: title.trim(),
      content: content || '',
      category,                                          // ← ajout
    });
```

**Commit n°1** :
```bash
git add backend/
git commit -m "feat(backend): ajoute la catégorie aux notes"
```

**④ `frontend/src/categories.js`** (nouveau)
```js
export const CATEGORIES = ['General', 'Travail', 'Perso', 'Idées'];
```

**⑤ `frontend/src/components/NoteForm.jsx`** : ajoutez un `select`
```jsx
import { useState } from 'react';
import { CATEGORIES } from '../categories.js';          // ← import

export default function NoteForm({ onCreate }) {
  const [title, setTitle] = useState('');
  const [content, setContent] = useState('');
  const [category, setCategory] = useState('General');  // ← état

  async function handleSubmit(e) {
    e.preventDefault();
    await onCreate({ title, content, category });       // ← envoie la catégorie
    setTitle('');
    setContent('');
  }

  return (
    <form onSubmit={handleSubmit} className="form">
      <input value={title} onChange={(e) => setTitle(e.target.value)} placeholder="Titre" required />
      <select value={category} onChange={(e) => setCategory(e.target.value)}>   {/* ← */}
        {CATEGORIES.map((c) => (
          <option key={c}>{c}</option>
        ))}
      </select>
      <textarea value={content} onChange={(e) => setContent(e.target.value)} placeholder="Contenu de la note" />
      <button type="submit">Ajouter</button>
    </form>
  );
}
```

**⑥ `frontend/src/components/CategoryFilter.jsx`** (nouveau)
```jsx
import { CATEGORIES } from '../categories.js';

export default function CategoryFilter({ value, onChange }) {
  return (
    <select value={value} onChange={(e) => onChange(e.target.value)}>
      <option value="Toutes">Toutes les catégories</option>
      {CATEGORIES.map((c) => (
        <option key={c}>{c}</option>
      ))}
    </select>
  );
}
```

**⑦ `frontend/src/components/NoteItem.jsx`** : affichez un badge **entre** le `<h3>` et le `<p>` :
```jsx
      <h3>{note.title}</h3>
      <span className="badge">{note.category}</span>
      <p>{note.content}</p>
```

**⑧ `frontend/src/App.jsx`** : filtre par catégorie (côté client)
```jsx
import CategoryFilter from './components/CategoryFilter.jsx';     // ← import (après NoteList)
...
  const [search, setSearch] = useState('');
  const [category, setCategory] = useState('Toutes');             // ← état (après search)
...
      <SearchBar value={search} onChange={setSearch} />
      <CategoryFilter value={category} onChange={setCategory} />  {/* ← */}
      <NoteList notes={category === 'Toutes' ? notes : notes.filter((n) => n.category === category)} />
```

**Commit n°2** et push de la branche :
```bash
docker compose up --build        # tester, puis Ctrl+C
git add frontend/
git commit -m "feat(frontend): choix de catégorie, badge et filtre par catégorie"
git push -u origin feature/categories
```

📖 `git push -u origin feature/categories` : la **branche n'existe pas encore** sur GitHub, `-u` la crée côté distant et mémorise le lien.

---

### 10.6 🧑‍💻 Member — fonctionnalité « Modifier / Supprimer » (branche `feature/edit-delete`)

**① `backend/src/models/noteModel.js`** : **ajoutez à la fin du fichier**
```js
export async function update(id, { title, content }) {
  const { rows } = await pool.query(
    'UPDATE notes SET title = $1, content = $2 WHERE id = $3 RETURNING *',
    [title, content, id]
  );
  return rows[0];
}

export async function remove(id) {
  const { rowCount } = await pool.query('DELETE FROM notes WHERE id = $1', [id]);
  return rowCount > 0;
}
```

**② `backend/src/controllers/noteController.js`** : **ajoutez à la fin**
```js
export async function update(req, res, next) {
  try {
    const { title, content } = req.body;
    if (!title || !title.trim()) {
      return res.status(400).json({ error: 'Le titre est obligatoire' });
    }
    const note = await Note.update(req.params.id, {
      title: title.trim(),
      content: content || '',
    });
    if (!note) return res.status(404).json({ error: 'Note introuvable' });
    res.json(note);
  } catch (err) {
    next(err);
  }
}

export async function remove(req, res, next) {
  try {
    const deleted = await Note.remove(req.params.id);
    if (!deleted) return res.status(404).json({ error: 'Note introuvable' });
    res.status(204).end();
  } catch (err) {
    next(err);
  }
}
```

**③ `backend/src/routes/noteRoutes.js`** : après `router.post(...)`
```js
router.put('/:id', ctrl.update);
router.delete('/:id', ctrl.remove);
```

**Commit n°1** :
```bash
git add backend/
git commit -m "feat(backend): routes PUT et DELETE pour modifier/supprimer une note"
```

**④ `frontend/src/api.js`** : ajoutez dans l'objet `api` (après `createNote`)
```js
  updateNote: (id, note) =>
    request(`${BASE}/${id}`, { method: 'PUT', body: JSON.stringify(note) }),
  deleteNote: (id) => request(`${BASE}/${id}`, { method: 'DELETE' }),
```

**⑤ `frontend/src/components/NoteItem.jsx`** : version complète avec mode édition
```jsx
import { useState } from 'react';

export default function NoteItem({ note, onUpdate, onDelete }) {
  const [editing, setEditing] = useState(false);
  const [title, setTitle] = useState(note.title);
  const [content, setContent] = useState(note.content);

  async function save() {
    await onUpdate(note.id, { title, content });
    setEditing(false);
  }

  if (editing) {
    return (
      <li className="note">
        <input value={title} onChange={(e) => setTitle(e.target.value)} />
        <textarea value={content} onChange={(e) => setContent(e.target.value)} />
        <div className="actions">
          <button onClick={save}>Enregistrer</button>
          <button className="secondary" onClick={() => setEditing(false)}>Annuler</button>
        </div>
      </li>
    );
  }

  return (
    <li className="note">
      <h3>{note.title}</h3>
      <p>{note.content}</p>
      <small>{new Date(note.created_at).toLocaleString()}</small>
      <div className="actions">
        <button onClick={() => setEditing(true)}>Modifier</button>
        <button className="danger" onClick={() => onDelete(note.id)}>Supprimer</button>
      </div>
    </li>
  );
}
```

**⑥ `frontend/src/components/NoteList.jsx`** : transmettre les fonctions
```jsx
import NoteItem from './NoteItem.jsx';

export default function NoteList({ notes, onUpdate, onDelete }) {
  if (notes.length === 0) return <p>Aucune note pour le moment.</p>;

  return (
    <ul className="notes">
      {notes.map((n) => (
        <NoteItem key={n.id} note={n} onUpdate={onUpdate} onDelete={onDelete} />
      ))}
    </ul>
  );
}
```

**⑦ `frontend/src/App.jsx`** : après `handleCreate`, ajoutez :
```jsx
  async function handleUpdate(id, note) {
    await api.updateNote(id, note);
    await loadNotes();
  }

  async function handleDelete(id) {
    if (!window.confirm('Supprimer cette note ?')) return;
    await api.deleteNote(id);
    await loadNotes();
  }
```
et modifiez la ligne :
```jsx
      <NoteList notes={notes} onUpdate={handleUpdate} onDelete={handleDelete} />
```

**Commit n°2** et push :
```bash
docker compose up --build        # tester modification + suppression
git add frontend/
git commit -m "feat(frontend): édition et suppression des notes"
git push -u origin feature/edit-delete
```

---

### 10.7 Voir les branches sur GitHub

```bash
git branch -a
git log --oneline --graph --all
```
Sur GitHub : menu des branches → vous voyez `main`, `feature/categories`, `feature/edit-delete`.

### 10.8 Pull Request n°1 — la branche du Member

1. 🧑‍💻 Sur GitHub, bouton **Compare & pull request** pour `feature/edit-delete` → base : `main`.
2. Titre : *feat: modifier et supprimer une note* · description : ce qui a été fait.
3. 👑 Le Leader **relit** (onglet *Files changed*, peut commenter ligne par ligne), puis **Approve**.
4. **Merge pull request** (choisir *Create a merge commit*).

> 🔎 **Option sans GitHub (merge local)** :
> ```bash
> git switch main
> git pull
> git merge feature/edit-delete
> git push
> ```

### 10.9 Pull Request n°2 — la branche du Leader → **2ᵉ conflit (attendu)**

`main` a avancé (les modifications du Member y sont). La branche du Leader est donc **en retard**. GitHub affiche :
> ⚠️ *This branch has conflicts that must be resolved* — fichier `frontend/src/App.jsx`

C'est normal : les deux branches ont modifié **la ligne `<NoteList …/>`** de `App.jsx`. Le Leader résout **en local** :

```bash
git switch feature/categories
git fetch
git merge origin/main        # on ramène main dans la branche pour la mettre à jour
```
```
CONFLICT (content): Merge conflict in frontend/src/App.jsx
```

Dans `App.jsx` :
```jsx
<<<<<<< HEAD
      <NoteList notes={category === 'Toutes' ? notes : notes.filter((n) => n.category === category)} />
=======
      <NoteList notes={notes} onUpdate={handleUpdate} onDelete={handleDelete} />
>>>>>>> origin/main
```

**Résolution (on combine les deux idées)** :
```jsx
      <NoteList
        notes={category === 'Toutes' ? notes : notes.filter((n) => n.category === category)}
        onUpdate={handleUpdate}
        onDelete={handleDelete}
      />
```

Les autres fichiers (`NoteItem.jsx`, `noteModel.js`…) ont fusionné **automatiquement** car les deux personnes ont modifié des **zones différentes**. Pensez à contrôler `NoteItem.jsx` : le badge de catégorie **et** les boutons doivent coexister.

```bash
grep -rn "<<<<<<<\|>>>>>>>" frontend backend      # aucun résultat attendu
git add frontend/src/App.jsx
git commit -m "merge: intègre main (édition/suppression) dans feature/categories"
docker compose up --build                          # tester l'ensemble !
git push
```
Puis, sur GitHub : **Merge pull request** (la PR n°2 est maintenant sans conflit).

#### 📄 `App.jsx` final de référence

```jsx
import { useEffect, useState } from 'react';
import { api } from './api.js';
import NoteForm from './components/NoteForm.jsx';
import SearchBar from './components/SearchBar.jsx';
import NoteList from './components/NoteList.jsx';
import CategoryFilter from './components/CategoryFilter.jsx';

export default function App() {
  const [notes, setNotes] = useState([]);
  const [search, setSearch] = useState('');
  const [category, setCategory] = useState('Toutes');

  async function loadNotes() {
    setNotes(await api.getNotes(search));
  }

  useEffect(() => {
    loadNotes();
  }, [search]);

  async function handleCreate(note) {
    await api.createNote(note);
    await loadNotes();
  }

  async function handleUpdate(id, note) {
    await api.updateNote(id, note);
    await loadNotes();
  }

  async function handleDelete(id) {
    if (!window.confirm('Supprimer cette note ?')) return;
    await api.deleteNote(id);
    await loadNotes();
  }

  return (
    <div className="container">
      <h1>📝 Mes Notes Pro</h1>
      <NoteForm onCreate={handleCreate} />
      <SearchBar value={search} onChange={setSearch} />
      <CategoryFilter value={category} onChange={setCategory} />
      <NoteList
        notes={category === 'Toutes' ? notes : notes.filter((n) => n.category === category)}
        onUpdate={handleUpdate}
        onDelete={handleDelete}
      />
    </div>
  );
}
```

### 10.10 Nettoyage et synchronisation (les deux)

```bash
git switch main
git pull
git log --oneline --graph

# Supprimer les branches terminées
git branch -d feature/categories                 # (ou feature/edit-delete) — local
git push origin --delete feature/categories      # — distant (GitHub propose aussi un bouton "Delete branch")
git fetch --prune                                # nettoie les références distantes obsolètes
```

- `-d` refuse de supprimer une branche **non fusionnée** (sécurité) ; `-D` force.

✅ **Point de contrôle** : `docker compose up --build` sur `main` → application complète : créer, modifier, supprimer, rechercher, catégories.

### 10.11 Résumé du workflow professionnel (« GitHub Flow »)

```
1. git switch main && git pull
2. git switch -c feature/ma-fonctionnalite
3. (travail + commits fréquents)
4. git push -u origin feature/ma-fonctionnalite
5. Pull Request → relecture → merge
6. git switch main && git pull
7. Suppression de la branche
```

---

## 11. Étape 5 — Les tags (versions)

### 11.1 À quoi servent les tags ?

Une **branche** avance avec chaque commit ; un **tag** est une **étiquette fixe** posée sur un commit précis, typiquement pour marquer une **version livrée** (`v1.0.0`).

### 11.2 Versionnement sémantique (SemVer) : `MAJOR.MINOR.PATCH`

| Partie | Quand l'incrémenter | Exemple |
|--------|--------------------|---------|
| MAJOR | Changement incompatible | `1.0.0 → 2.0.0` |
| MINOR | Nouvelle fonctionnalité compatible | `1.0.0 → 1.1.0` |
| PATCH | Correction de bug | `1.0.0 → 1.0.1` |

### 11.3 Créer un tag (👑 Leader, sur `main` à jour)

```bash
git switch main
git pull

# Tag ANNOTÉ (recommandé) : contient auteur, date, message
git tag -a v1.0.0 -m "Version 1.0.0 : CRUD notes, recherche, catégories"

# Lister
git tag
git show v1.0.0
```

| Type | Commande | Usage |
|------|----------|-------|
| **Annoté** | `git tag -a v1.0.0 -m "msg"` | Releases (recommandé) |
| **Léger** | `git tag v1.0.0` | Simple marque-page local |

### 11.4 Envoyer le tag (⚠️ `git push` n'envoie PAS les tags !)

```bash
git push origin v1.0.0        # un seul tag
# git push origin --tags      # tous les tags
```

### 11.5 Le Member récupère le tag

```bash
git pull
git fetch --tags
git tag
```

### 11.6 Revenir voir le code d'une version

```bash
git checkout v1.0.0           # (ou : git switch --detach v1.0.0)
```
Message : *You are in 'detached HEAD' state.* — pas de panique : vous regardez l'ancien état, **sans être sur une branche**. Ne commitez pas ici ; revenez avec :
```bash
git switch main
```
Besoin de corriger cette version ? Créez une branche depuis le tag :
```bash
git switch -c hotfix/v1.0.1 v1.0.0
```

### 11.7 Créer un tag **après coup** sur un ancien commit

```bash
git log --oneline
git tag -a v0.1.0 -m "Première version minimale" a1b2c3d     # a1b2c3d = identifiant du commit
git push origin v0.1.0
```

### 11.8 Supprimer un tag

```bash
git tag -d v0.1.0                    # local
git push origin --delete v0.1.0      # distant
```

### 11.9 Release GitHub

GitHub → **Releases → Draft a new release** → choisissez le tag `v1.0.0` → **Generate release notes** → *Publish*.

### 11.10 Exercice : une version `v1.0.1`

1. 🧑‍💻 Member : `git switch -c fix/trim-title`, corrigez un petit défaut (ex : ajouter `maxLength={150}` à l'`input` du titre dans `NoteForm.jsx`), commit, push, PR.
2. 👑 Leader : relit, merge, puis `git pull` et crée le tag `v1.0.1`.

---

## 12. Étape 6 — Commandes utiles en plus (bonus)

### 12.1 Voir l'historique

```bash
git log --oneline
git log --oneline --graph --all --decorate
git log --author="Prénom"
git log -p -1                         # contenu du dernier commit
git show <id-commit>
git blame backend/src/server.js       # qui a écrit chaque ligne ?
```

### 12.2 Annuler des modifications (selon la situation)

| Situation | Commande |
|-----------|----------|
| J'ai modifié un fichier, je veux **annuler** ces modifs | `git restore <fichier>` |
| J'ai fait `git add` par erreur | `git restore --staged <fichier>` |
| Je veux corriger le **dernier commit** (message ou oubli) — *pas encore poussé* | `git commit --amend` |
| Annuler un commit **déjà poussé** en créant un commit inverse (**sûr**) | `git revert <id>` |
| Revenir en arrière **en local**, garder les modifs en staging | `git reset --soft HEAD~1` |
| Revenir en arrière, garder les modifs non indexées | `git reset HEAD~1` (= `--mixed`) |
| Revenir en arrière **et supprimer** les modifs ⚠️ | `git reset --hard HEAD~1` |

> ⚠️ **Règle d'or** : ne jamais réécrire (`reset`, `amend`, `rebase`) un historique **déjà poussé et partagé**. Utilisez `revert`.

**Mini-exercice `revert`** :
```bash
echo "test" >> README.md
git commit -am "docs: ajoute une ligne de test"
git push
git revert HEAD                 # crée un commit qui annule le précédent
git push
```

### 12.3 `git stash` : mettre de côté son travail en cours

Situation : vous codez, et on vous demande de changer de branche **immédiatement**.
```bash
git stash push -m "wip: formulaire"
git switch autre-branche
# ... travail ...
git switch ma-branche
git stash list
git stash pop            # récupère et supprime le stash
```

### 12.4 `git reflog` : le filet de sécurité

Même après un `reset --hard`, Git garde la trace des déplacements de HEAD pendant ~90 jours :
```bash
git reflog
git reset --hard HEAD@{2}        # retrouver un état précédent
```

### 12.5 `cherry-pick` : récupérer **un** commit d'une autre branche

```bash
git cherry-pick <id-commit>
```

### 12.6 `rebase` (introduction)

Alternative au merge qui « rejoue » vos commits au-dessus de la branche cible → historique **linéaire**.
```bash
git switch feature/x
git rebase main
```
> ⚠️ Pour avancés. Ne jamais rebaser une branche **déjà partagée** sans l'accord de l'équipe.

### 12.7 Comparer

```bash
git diff                    # working dir vs staging
git diff --staged           # staging vs dernier commit
git diff main..feature/x    # entre deux branches
```

### 12.8 Mettre à jour un nom de remote / voir les infos

```bash
git remote -v
git remote show origin
git remote set-url origin <nouvelle-url>
```

---

## 13. Aide-mémoire (cheat sheet)

| Objectif | Commande |
|----------|----------|
| Configurer l'identité | `git config --global user.name "X"` / `user.email "x@y.z"` |
| Initialiser | `git init` |
| Cloner | `git clone <url>` |
| État | `git status` |
| Indexer | `git add <fichier>` / `git add .` |
| Commit | `git commit -m "type: message"` |
| Historique | `git log --oneline --graph --all` |
| Lier au distant | `git remote add origin <url>` |
| Premier push | `git push -u origin main` |
| Push suivants | `git push` |
| Télécharger sans fusionner | `git fetch` |
| Télécharger + fusionner | `git pull` |
| Créer + aller sur une branche | `git switch -c <nom>` |
| Changer de branche | `git switch <nom>` |
| Fusionner | `git merge <branche>` |
| Annuler un merge en cours | `git merge --abort` |
| Supprimer branche locale / distante | `git branch -d <nom>` / `git push origin --delete <nom>` |
| Créer un tag annoté | `git tag -a v1.0.0 -m "msg"` |
| Pousser les tags | `git push origin --tags` |
| Mettre de côté | `git stash` / `git stash pop` |
| Annuler un fichier | `git restore <fichier>` |
| Annuler un commit poussé | `git revert <id>` |
| Retrouver un état perdu | `git reflog` |

### Messages de commit : rappel

```
feat: ajoute la recherche de notes
fix: corrige le crash quand le titre est vide
docs: met à jour le README
chore: met à jour les dépendances
refactor: extrait la logique de filtre
```

---

## 14. Erreurs fréquentes et solutions

| Message / problème | Cause | Solution |
|--------------------|-------|----------|
| `Author identity unknown` / `Please tell me who you are` | `user.name`/`user.email` non configurés | Étape 0 |
| `fatal: not a git repository` | Vous n'êtes pas dans le bon dossier | `cd` dans le projet ; `git init` si nouveau |
| `! [rejected] main -> main (fetch first)` | Le distant a des commits que vous n'avez pas | `git pull` puis `git push` |
| `CONFLICT (content): Merge conflict in …` | Mêmes lignes modifiées par deux personnes | Éditer, retirer marqueurs, `git add`, `git commit` |
| `error: Your local changes would be overwritten by merge` | Modifications non commitées qui gênent le `pull` | `git commit` ou `git stash` avant |
| `fatal: refusing to merge unrelated histories` | Dépôt GitHub créé avec un README + dépôt local indépendant | `git pull origin main --allow-unrelated-histories` |
| `Permission denied` / `Authentication failed` | Mauvais token / droits collaborateur | Vérifier l'invitation, le token ou la clé SSH |
| `.env` ou `node_modules` apparaissent dans `git status` | `.gitignore` absent, ou fichier déjà suivi | Créer `.gitignore` ; `git rm -r --cached node_modules` |
| `detached HEAD` | `checkout` d'un tag ou d'un commit | `git switch main` |
| Port 3000 / 5432 déjà utilisé | Autre application | Arrêter l'application ou changer le port dans `docker-compose.yml` |
| L'app affiche une erreur de connexion à la base | `.env` absent après un clone | `cp .env.example .env` |
| Anciennes données/colonne manquante après un changement | Volume Docker ancien | `docker compose down -v` (⚠️ supprime les données) puis `up --build` |
| Le build n'inclut pas mes changements | Cache Docker | `docker compose build --no-cache` |

---

## 15. Exercices finaux et grille d'évaluation

### 15.1 Exercices complémentaires

1. **Hotfix** : le Leader crée `hotfix/empty-content` pour corriger un bug, puis tag `v1.0.2`.
2. **Conflit volontaire n°3** : le Leader et le Member modifient la même ligne de `README.md` dans deux branches différentes ; fusion + résolution.
3. **Catégorie à l'édition** : permettre de modifier la catégorie d'une note (branche `feature/edit-category`).
4. **Revert** : poussez un commit défectueux sur une branche de test puis annulez-le avec `git revert`.
5. **Stash** : commencez une modification, changez de branche en urgence avec `stash`, revenez la récupérer.
6. **Release** : publiez une Release GitHub à partir du tag `v1.1.0`.
7. **Challenge** : ajoutez une règle `.gitignore` pour ignorer `*.local`, créez un fichier `test.local` et prouvez avec `git check-ignore -v` qu'il est ignoré.

### 15.2 Grille d'auto-évaluation

| Compétence | Acquis ✅ | À revoir 🔁 |
|-----------|:--------:|:----------:|
| J'ai configuré `user.name` et `user.email` | ☐ | ☐ |
| Je sais expliquer working dir / staging / repo local / distant | ☐ | ☐ |
| J'ai fait `init`, `add`, `commit` avec de bons messages | ☐ | ☐ |
| J'ai créé un `.gitignore` et vérifié qu'il fonctionne | ☐ | ☐ |
| J'ai poussé pour la première fois (`push -u`) | ☐ | ☐ |
| J'ai cloné un dépôt | ☐ | ☐ |
| Je comprends pourquoi un push est rejeté | ☐ | ☐ |
| Je sais distinguer `fetch` et `pull` | ☐ | ☐ |
| J'ai résolu un conflit de A à Z | ☐ | ☐ |
| J'ai créé, poussé et fusionné une branche (PR) | ☐ | ☐ |
| J'ai créé et poussé un tag annoté | ☐ | ☐ |
| Je sais utiliser `stash`, `restore`, `revert` | ☐ | ☐ |
| L'application Docker fonctionne de bout en bout | ☐ | ☐ |

### 15.3 Résultat final attendu

- Un dépôt GitHub `notes-app` avec :
  - une branche `main` stable,
  - un historique lisible (`git log --graph` montre des merges),
  - un fichier `.gitignore` correct, aucun secret versionné,
  - les tags `v1.0.0` (et `v1.0.1` si exercice fait),
  - au moins 2 Pull Requests fusionnées,
  - une application Notes fonctionnelle lancée avec **une seule commande** : `docker compose up --build`.

---

## 🎓 Conclusion

Vous avez vécu le **cycle réel** du travail en équipe avec Git :

1. 🔧 **Configurer** → 2. 📦 **Initialiser et committer** → 3. 🚫 **Ignorer** → 4. ☁️ **Pousser** → 5. 📥 **Cloner** → 6. 💥 **Conflit** (oubli de `pull`) → 7. 🌿 **Branches + PR** → 8. 🏷️ **Tags** → 9. 🛟 **Commandes de secours**.

> 🧭 **Les 3 réflexes à retenir** :
> 1. `git status` — toujours, tout le temps.
> 2. `git pull` avant de travailler et avant de pousser.
> 3. Une fonctionnalité = une branche = une Pull Request.

Bon courage et bon code ! 🚀
