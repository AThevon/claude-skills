---
name: push
description: Add, commit et push les changements
allowed-tools: Bash, Read, Grep
---

Commit et push les changements en cours :

1. Vérifie l'état :
   ```bash
   git status
   ```

2. S'il y a des changements :
   ```bash
   git add -A
   ```

3. Analyse les changements pour créer un message de commit :
   ```bash
   git diff --cached --stat
   git diff --cached
   ```

4. Commit avec un message clair et concis :
   - Format : type(scope): description (ex: "fix(auth): handle expired tokens")
   - Ou format simple si plus approprié : "add user avatar upload"
   - PAS de Co-Authored-By
   - Court et descriptif

5. Push :
   ```bash
   # Si la branche n'a pas d'upstream
   git push -u origin $(git branch --show-current)
   # Sinon
   git push
   ```

6. Confirme le push avec le lien si disponible
