---
name: fix-ci
description: Analyse et corrige les erreurs de CI
allowed-tools: Bash, Read, Edit, Write, Grep, Glob
---

Corrige les erreurs de CI sur la branche actuelle :

1. Récupère l'état de la CI :
   ```bash
   gh pr checks || gh run list --branch $(git branch --show-current) --limit 5
   ```

2. Si des checks échouent, récupère les logs :
   ```bash
   gh run view <run-id> --log-failed
   ```

3. Analyse les erreurs et corrige-les dans le code

4. Vérifie ton fix en local (build, lint, tests selon l'erreur)
   - **Max 5 tentatives.** Si les erreurs persistent après 5 cycles de correction, arrête et fais un récap des erreurs restantes à l'utilisateur.

5. Commit et push le fix :
   - Message de commit clair décrivant le fix
   - Ne pas inclure de Co-Authored-By
   - Push sur la branche actuelle
