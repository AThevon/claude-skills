---
name: copy-env
description: Use when working in a git worktree and needing to copy .env or .env.local files from the main worktree
allowed-tools: Bash, Read, Glob
---

Copie les fichiers .env depuis le worktree principal vers le worktree courant.

1. Vérifie qu'on est dans un worktree (pas le main) :
   ```bash
   CURRENT=$(git rev-parse --show-toplevel)
   MAIN_WT=$(git worktree list | grep '\[main\]' | awk '{print $1}')
   ```
   - Si `$CURRENT` == `$MAIN_WT` → erreur : "Tu es déjà dans le worktree principal."
   - Si `$MAIN_WT` est vide → essayer avec `master` : `git worktree list | grep '\[master\]' | awk '{print $1}'`

2. Liste les fichiers .env dans le worktree principal :
   ```bash
   ls -a "$MAIN_WT"/.env* 2>/dev/null
   ```

3. S'il n'y a aucun fichier .env → informe l'utilisateur et stop.

4. S'il y a des fichiers, copie-les :
   ```bash
   cp "$MAIN_WT/.env" "$CURRENT/.env" 2>/dev/null
   cp "$MAIN_WT/.env.local" "$CURRENT/.env.local" 2>/dev/null
   ```
   Copier uniquement les fichiers qui existent côté main.

5. Confirme les fichiers copiés avec un résumé.
