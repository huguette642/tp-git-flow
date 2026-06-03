# Branches du projet

Ce tableau résume les règles de Git Flow appliquées dans ce dépôt.

| Branche | Rôle | Part de | Fusionne dans |
|---|---|---|---|
| `main` | Code en production | — | — |
| `develop` | Code stabilisé en cours d'intégration | `main` (initialisation) | — |
| `feature/...` | Développement d'une fonctionnalité | `develop` | `develop` |
| `release/...` | Préparation d'une livraison | `develop` | `main` puis `develop` |
| `hotfix/...` | Correctif urgent en production | `main` | `main` puis `develop` |

## Règles

- On ne développe **jamais directement** sur `main`.
- On ne développe **jamais directement** sur `develop`.
- Chaque fonctionnalité vit sur sa propre branche `feature/...`.
- Une branche `release/...` permet de geler le périmètre d'une livraison et de préparer la version.
- Un `hotfix/...` corrige un bug critique en production sans attendre le cycle normal des features.

## Cycle de vie d'une feature

```
develop
  └─ feature/ma-fonctionnalite
       └─ (commits)
  ← merge feature  (fin du développement)
```

## Cycle de vie d'une release

```
develop
  └─ release/1.1.0
       └─ (mise à jour VERSION, correctifs mineurs)
main    ← merge release
develop ← merge release  (retour)
```

## Cycle de vie d'un hotfix

```
main
  └─ hotfix/fix-mon-bug
       └─ (correctif)
main    ← merge hotfix  (tag v1.x.x)
develop ← merge hotfix  (retour)
```
