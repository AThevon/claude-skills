---
name: review
description: Review complète du code (PR ou branche)
allowed-tools: Bash, Read, Grep, Glob
---

Fais une code review complète :

1. Vérifie s'il existe une PR ouverte pour la branche actuelle :
   ```bash
   gh pr view --json number,title,body 2>/dev/null
   ```

2. Si une PR existe :
   - Récupère le diff de la PR : `gh pr diff`
   - Review ce diff

3. Si pas de PR :
   - Trouve la branche principale (main ou master)
   - Compare avec le point de création : `git log main..HEAD --oneline` et `git diff main...HEAD`
   - Review ces changements

4. Dans ta review, analyse :
   - La qualité du code
   - Les potentiels bugs ou edge cases
   - Les problèmes de performance
   - Les bonnes pratiques non respectées
   - La lisibilité et maintenabilité

5. Structure ta review avec des sections claires et donne des suggestions concrètes.
