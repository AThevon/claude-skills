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
   - Si aucun changement (working tree clean) → informer l'utilisateur et STOP
   - Si la branche courante est `main` ou `master` → avertir l'utilisateur "Tu es sur main, tu es sûr de vouloir push ici ?" et attendre confirmation avant de continuer

2. Stage les fichiers modifiés et nouveaux (pas de `git add -A`) :
   ```bash
   # Fichiers tracked modifiés
   git add -u
   # Fichiers untracked : les ajouter un par un après vérification (ignorer les fichiers suspects : *.log, *.tmp, .DS_Store, binaires lourds)
   git status --porcelain | grep '^??' | cut -c4-
   ```

3. Analyse les changements stagés pour créer un message de commit :
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
