---
name: release-github
description: "Release GitHub complète et automatique : commit, readme, bump, PR, squash merge, tag, CI, GitHub Release. Zéro effort."
user_invocable: true
allowed-tools: Bash, Read, Edit, Write, Grep, Glob
---

# Release GitHub — Full Auto

Release complète depuis n'importe quel état (branche ou main, commité ou non). Le but : l'utilisateur n'a rien à faire.

## 1. Demander le type de bump

- Si l'utilisateur a passé un argument (ex: `/release-github minor`), l'utiliser
- Sinon, analyser les commits depuis le dernier tag et proposer un type :
  - **MAJOR** : breaking changes
  - **MINOR** : nouvelles fonctionnalités (feat)
  - **PATCH** (default) : fixes, améliorations mineures
- Demander confirmation rapide

```bash
LAST_TAG=$(git describe --tags --abbrev=0 2>/dev/null || echo "v0.0.0")
echo "Dernier tag : $LAST_TAG"
```

## 2. Commit si nécessaire

```bash
git status --porcelain
```

- S'il y a des changements non commités, les add et commit avec un message clair
- Format : `type(scope): description`

## 3. Mettre à jour le README

- Invoquer le skill `/readme` pour mettre à jour le README avec les derniers changements
- Commit le résultat : `docs: update README for vX.Y.Z`

## 4. Bump la version

- Invoquer le skill `/bump` avec le type déterminé à l'étape 1
- Chercher et mettre à jour la version dans tous les fichiers pertinents du projet (package.json, Cargo.toml, pyproject.toml, VERSION, etc.)
- Commit le résultat : `bump: vX.Y.Z`

## 5. Créer la PR

Si on est sur une branche (pas main) :

```bash
CURRENT=$(git branch --show-current)
MAIN=$(git remote show origin | grep 'HEAD branch' | cut -d' ' -f5)

# Rebase (NE PAS checkout main — worktrees)
git fetch origin $MAIN
git rebase origin/$MAIN

# Push
git push -u origin "$CURRENT" --force-with-lease
```

Créer la PR avec titre et description **en anglais** :

```bash
gh pr create --title "Release vX.Y.Z" --body "$(cat <<'EOF'
## Changes

- list of changes from commits since last tag

## Version

LAST_TAG → vX.Y.Z
EOF
)"
```

Si on est directement sur main, skip la PR et passer à l'étape 6.

## 6. Squash merge la PR

Demander confirmation avant de merger.

```bash
gh pr merge --squash --delete-branch
```

## 7. Créer le tag

```bash
# Fetch le merge dans main
git fetch origin $MAIN

# Créer le tag sur le commit mergé
git tag "vX.Y.Z" origin/$MAIN
git push origin "vX.Y.Z"
```

## 8. Vérifier la CI

```bash
# Vérifier si un workflow s'est déclenché
gh run list --limit 5
```

- Si aucun workflow ne s'est déclenché sur le tag, tenter un `workflow_dispatch` :

```bash
gh workflow list
# Demander confirmation avant de déclencher
gh workflow run <workflow_name> --ref "vX.Y.Z"
```

## 9. Créer la GitHub Release

Générer le changelog depuis le dernier tag (en anglais) :

```bash
git log "$LAST_TAG"..origin/$MAIN --oneline --no-merges
```

```bash
gh release create "vX.Y.Z" \
  --title "vX.Y.Z" \
  --notes "$(cat <<'NOTES'
## What's Changed

- change 1
- change 2

**Full Changelog**: https://github.com/OWNER/REPO/compare/LAST_TAG...vX.Y.Z
NOTES
)" \
  --latest
```

## 10. Résumé final

Afficher :
- Version : `LAST_TAG` → `vX.Y.Z`
- PR : lien (si créée)
- Release : lien
- CI : statut
