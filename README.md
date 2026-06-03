# cours-02-git-flow-api

API NestJS minimaliste — support du TP cours-02, Partie 2 (Git Flow).

---

## Branches du projet

| Branche | Rôle |
|---|---|
| `main` | Code en production — ne jamais développer directement dessus |
| `develop` | Code stabilisé en cours d'intégration |
| `feature/...` | Développement d'une fonctionnalité → fusionne dans `develop` |
| `release/...` | Préparation d'une livraison → fusionne dans `main` puis `develop` |
| `hotfix/...` | Correctif urgent → fusionne dans `main` puis `develop` |

> Détail des règles et cycles de vie → [BRANCHES.md](BRANCHES.md)

---

## Contexte des Pull Requests en TP

Par défaut, les Pull Requests sont ouvertes **dans ton fork** : par exemple `release/1.1.0` est proposée vers `main` de ton dépôt forké.

Si l'enseignant le demande explicitement, la Pull Request peut viser le dépôt central `GVI2026/tp-git-flow`. Sans droit d'écriture ou sans accès GitHub disponible, le scénario peut être simulé localement avec les merges indiqués dans ce README, en gardant la même logique de revue et de validation.

---

## Démarrage recommandé — DevContainer (Windows / Linux / macOS)

**Prérequis** : [VS Code](https://code.visualstudio.com/) + [Docker Desktop](https://www.docker.com/products/docker-desktop/) + extension [Dev Containers](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-containers)

1. **Forker** le dépôt sur GitHub, puis le cloner :
   ```bash
   git clone <url-de-ton-fork>
   cd cours-02-git-flow-api
   ```

2. Ouvrir dans VS Code et accepter "Reopen in Container" — ou via la palette de commandes : `Dev Containers: Reopen in Container`

3. Attendre l'initialisation (première fois ~2 min). Les dépendances npm et les migrations Prisma s'exécutent automatiquement.

4. Lancer l'application :
   ```bash
   npm run start:dev
   ```

5. Swagger disponible sur [http://localhost:3000/api](http://localhost:3000/api)

---

## Démarrage manuel (sans DevContainer)

**Prérequis** : Node.js 24, PostgreSQL 18

1. Copier et renseigner les variables d'environnement :
   ```bash
   cp .env.example .env
   # Éditer .env : renseigner DATABASE_URL
   ```

2. Installer les dépendances :
   ```bash
   npm install
   ```

3. Appliquer les migrations :
   ```bash
   npx prisma migrate dev --name init
   ```

4. Lancer l'application :
   ```bash
   npm run start:dev
   ```

---

## Lancer les tests

```bash
npm test
```

---

## Routes disponibles

| Méthode | Route | Description |
|---|---|---|
| `POST` | `/tasks` | Créer une tâche |
| `GET` | `/tasks` | Lister toutes les tâches |
| `GET` | `/tasks/:id` | Récupérer une tâche |
| `PATCH` | `/tasks/:id` | Mettre à jour une tâche |
| `DELETE` | `/tasks/:id` | Supprimer une tâche |
| `GET` | `/tasks/stats` | ⚠️ **À implémenter** — `{ total, done, pending }` |

---

## Scénario Git Flow

### Étape 0 — Mise en place

1. **Forker** le dépôt sur GitHub, puis le cloner.
2. Vérifier que les deux branches permanentes existent :
   ```bash
   git branch -a
   ```
   Tu dois voir `main` et `origin/develop`.
3. Se placer sur `develop` :
   ```bash
   git checkout develop
   ```

### Étape 1 — Créer une feature

4. Créer la branche de fonctionnalité depuis `develop` :
   ```bash
   git checkout -b feature/add-task-stats
   ```
5. Implémenter `GET /tasks/stats` dans `TasksService` et `TasksController`.
6. Décommenter le bloc `TODO` dans `tasks.service.spec.ts` et l'adapter.
7. Vérifier que tous les tests passent :
   ```bash
   npm test
   ```
8. Committer proprement (conventional commits) :
   ```bash
   git add .
   git commit -m "feat: add task stats endpoint"
   ```

### Étape 2 — Intégrer dans develop

9. Fusionner la feature dans `develop` :
   ```bash
   git checkout develop
   git merge feature/add-task-stats
   ```
10. Supprimer la branche locale :
    ```bash
    git branch -d feature/add-task-stats
    ```

### Étape 3 — Préparer une release

11. Créer la branche de release depuis `develop` :
    ```bash
    git checkout -b release/1.1.0
    ```
12. Mettre à jour le fichier `VERSION` :
    ```
    1.1.0
    ```
13. Committer cette préparation :
    ```bash
    git commit -am "chore: prepare release 1.1.0"
    ```

### Étape 4 — Finaliser la release

14. Pousser la branche de release :
    ```bash
    git push origin release/1.1.0
    ```
15. Ouvrir une **Pull Request** `release/1.1.0` → `main` sur GitHub.
16. Après merge, créer un tag sur `main` :
    ```bash
    git checkout main
    git pull
    git tag v1.1.0
    git push origin v1.1.0
    ```
17. Répercuter la release dans `develop` :
    ```bash
    git checkout develop
    git merge main
    git push origin develop
    ```

---

## Pour les plus rapides — Exercice optionnel (hotfix)

Après avoir terminé le scénario principal, simule un correctif urgent en production.

La validation du champ `title` est actuellement incomplète : un `POST /tasks` avec `{ "title": "" }` accepte une chaîne vide.

1. Créer une branche `hotfix/fix-title-validation` depuis `main` :
   ```bash
   git checkout main
   git checkout -b hotfix/fix-title-validation
   ```
2. Corriger le problème dans `CreateTaskDto`.
3. Écrire un test qui vérifie que le titre vide est rejeté.
4. Committer : `fix: reject empty task title`
5. Merger dans `main`, tagger `v1.1.1` :
   ```bash
   git checkout main
   git merge hotfix/fix-title-validation
   git tag v1.1.1
   git push origin main v1.1.1
   ```
6. Répercuter dans `develop` :
   ```bash
   git checkout develop
   git merge main
   git push origin develop
   ```

---

## Note de setup — enseignant

Avant la séance, s'assurer que :

- La branche `develop` existe et est poussée : `git push origin develop`
- `main` est protégée : PR obligatoire, CI doit passer avant merge
- `develop` est semi-protégée : push direct interdit depuis GitHub, merge local autorisé
- `npm test` est vert sur un dépôt fraîchement cloné
