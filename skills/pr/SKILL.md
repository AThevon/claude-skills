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

2. Si changements non commités/non pushés :
   ```bash
   git stash -u
   ```

3. Rebase sur main (NE PAS checkout main — worktrees) :
   ```bash
   git fetch origin $MAIN
   git rebase origin/$MAIN
   ```
   Si conflits → les résoudre

4. Si on avait stash :
   ```bash
   git stash pop
   ```
   Si conflits → les résoudre
   Puis commit et push (message clair, sans Co-Authored-By)

5. Push la branche :
   ```bash
   git push -u origin $CURRENT --force-with-lease
   ```

6. Crée la PR :
   ```bash
   gh pr create --title "..." --body "..."
   ```
   - Titre clair et concis
   - Description explicite mais pas trop longue
   - PAS de mention de Claude/AI dans la description
   - Explique le "quoi" et le "pourquoi"

7. Affiche le lien de la PR créée
