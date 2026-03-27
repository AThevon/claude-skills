---
name: pr
description: Crée une PR complète (rebase, push, create)
allowed-tools: Bash, Read, Edit, Write, Grep
---

Crée une Pull Request propre :

1. Vérifie l'état actuel :
   ```bash
   git status
   CURRENT=$(git branch --show-current)
   MAIN=$(git remote show origin | grep 'HEAD branch' | cut -d' ' -f5)
   ```
   - Si la branche est `main` ou `master` → STOP, on ne crée pas de PR depuis main

2. Vérifie si une PR existe déjà pour cette branche :
   ```bash
   gh pr view 2>/dev/null
   ```
   - Si une PR existe → informer l'utilisateur et proposer `/update-pr` à la place. STOP.

3. Si changements non commités (working tree dirty) :
   ```bash
   git stash -u
   ```
   Note : stash uniquement si `git status --porcelain` retourne des fichiers. Les commits non pushés ne nécessitent PAS de stash.

4. Rebase sur main (NE PAS checkout main — worktrees) :
   ```bash
   git fetch origin $MAIN
   git rebase origin/$MAIN
   ```
   Si conflits → les résoudre

5. Si on avait stash :
   ```bash
   git stash pop
   ```
   Si conflits → les résoudre
   Puis commit et push (message clair, sans Co-Authored-By)

6. Push la branche :
   ```bash
   git push -u origin $CURRENT --force-with-lease
   ```

7. Prépare la PR — détecter les conventions du repo :
   ```bash
   # Template PR ?
   cat .github/PULL_REQUEST_TEMPLATE.md 2>/dev/null || cat .github/pull_request_template.md 2>/dev/null
   # Langue et style des PR récentes
   gh pr list --state merged --limit 5 --json title,body --jq '.[] | "Title: \(.title)\nBody: \(.body[:200])\n---"'
   ```
   - Titre : toujours en anglais, clair et concis
   - Description : si un template existe, l'utiliser comme structure. Adapter la langue à celle des PR existantes (français si les PR récentes sont en français, anglais sinon)
   - PAS de mention de Claude/AI dans la description
   - Explique le "quoi" et le "pourquoi"

8. Crée la PR :
   ```bash
   gh pr create --title "..." --body "..."
   ```

9. Affiche le lien de la PR créée
