---
name: rebase
description: Rebase la branche actuelle sur main
allowed-tools: Bash, Read, Edit, Write
---

Rebase la branche actuelle sur la branche principale. Gère automatiquement les worktrees.

1. Identifie le contexte :
   ```bash
   CURRENT=$(git branch --show-current)
   MAIN=$(git remote show origin | grep 'HEAD branch' | cut -d' ' -f5)
   COMMON_DIR=$(git rev-parse --git-common-dir)
   ```

2. Détermine si on est dans un worktree secondaire :
   - Si `$COMMON_DIR` ne vaut PAS `.git` → on est dans un worktree secondaire
   - Si `$COMMON_DIR` vaut `.git` → on est dans le main worktree

3. Met à jour main et rebase :

   **Cas worktree secondaire** (le plus fréquent) :
   ```bash
   # Trouve le chemin du main worktree à partir du git-common-dir
   MAIN_WORKTREE=$(dirname "$COMMON_DIR")
   CURRENT_DIR=$(pwd)

   # Va dans le main worktree, pull, puis revient
   cd "$MAIN_WORKTREE"
   git pull
   cd "$CURRENT_DIR"

   # Rebase sur main (local, pas origin/main)
   git rebase "$MAIN"
   ```

   **Cas main worktree (pas de worktree)** :
   ```bash
   git checkout "$MAIN"
   git pull
   git checkout "$CURRENT"
   git rebase "$MAIN"
   ```

4. Si conflits :
   - Analyse chaque conflit
   - Résous-les intelligemment (garde la logique correcte)
   - `git add` les fichiers résolus
   - `git rebase --continue`
   - Répète jusqu'à la fin du rebase

5. Confirme que le rebase est terminé avec succès

6. Propose de push force :
   - Si la branche a un upstream distant, demande à l'utilisateur s'il veut `git push --force-with-lease`
   - Si oui, exécute le push force
