---
name: rebase
description: Rebase la branche actuelle sur main
allowed-tools: Bash, Read, Edit, Write
---

Rebase la branche actuelle sur la branche principale via origin (compatible worktrees).

1. Identifie le contexte :
   ```bash
   CURRENT=$(git branch --show-current)
   MAIN=$(git remote show origin | grep 'HEAD branch' | cut -d' ' -f5)
   ```

2. Fetch et rebase sur origin (même flow worktree ou non) :
   ```bash
   git fetch origin
   git rebase "origin/$MAIN"
   ```
   Ne JAMAIS faire `git checkout main` ou `git switch main` — main est toujours checked out dans un autre worktree.

3. Si conflits :
   - Analyse chaque conflit
   - Résous-les intelligemment (garde la logique correcte)
   - `git add` les fichiers résolus
   - `git rebase --continue`
   - Répète jusqu'à la fin du rebase

4. Confirme que le rebase est terminé avec succès

5. Propose de push force :
   - Si la branche a un upstream distant, demande à l'utilisateur s'il veut `git push --force-with-lease`
   - Si oui, exécute le push force
