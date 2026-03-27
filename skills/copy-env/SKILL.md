---
name: copy-env
description: Copie les fichiers .env depuis le worktree principal vers le worktree courant
allowed-tools: Bash, Read, Glob
---

Copie les fichiers .env depuis le worktree principal vers le worktree courant.

1. Vérifie qu'on est dans un worktree (pas le principal) :
   ```bash
   CURRENT=$(git rev-parse --show-toplevel)
   # Le premier worktree listé est toujours le principal
   MAIN_WT=$(git worktree list | head -1 | awk '{print $1}')
   ```
   - Si `$CURRENT` == `$MAIN_WT` → "Tu es déjà dans le worktree principal." et STOP

2. Liste tous les fichiers `.env*` dans le worktree principal :
   ```bash
   find "$MAIN_WT" -maxdepth 1 -name ".env*" -type f
   ```
   - Si aucun fichier → informer l'utilisateur et STOP

3. Copie dynamiquement tous les `.env*` trouvés :
   ```bash
   for f in "$MAIN_WT"/.env*; do
     [ -f "$f" ] && cp "$f" "$CURRENT/$(basename "$f")"
   done
   ```
   Cela couvre `.env`, `.env.local`, `.env.development`, `.env.staging`, `.env.test`, etc.

4. Si le projet est un monorepo (sous-dossiers avec leurs propres `.env`) :
   ```bash
   # Chercher les .env dans les sous-dossiers apps/ et packages/
   find "$MAIN_WT" -path "*/apps/*/.env*" -o -path "*/packages/*/.env*" | grep -v node_modules
   ```
   Si trouvés → reproduire la même structure dans le worktree courant et copier.

5. Afficher un résumé des fichiers copiés.
