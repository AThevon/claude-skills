---
name: readme
description: Met à jour le README selon les derniers changements
allowed-tools: Bash, Read, Edit, Grep, Glob
---

Met à jour le README en fonction des derniers changements :

1. Trouve et lis le README actuel :
   ```bash
   ls README* readme* 2>/dev/null | head -1
   ```

2. Analyse les derniers commits sur la branche :
   ```bash
   git log --oneline -20
   git log --stat -10
   ```

3. Regarde le diff des fichiers modifiés récemment pour comprendre les changements :
   ```bash
   git diff HEAD~10..HEAD --stat
   git diff HEAD~10..HEAD
   ```

4. Compare avec le contenu actuel du README :
   - Identifie ce qui est obsolète
   - Identifie ce qui manque (nouvelles features, nouveaux fichiers, etc.)
   - Identifie ce qui doit être mis à jour

5. Met à jour le README :
   - Garde le style et le format existant
   - Ajoute/modifie uniquement ce qui est nécessaire
   - Ne supprime pas d'infos encore pertinentes

6. Affiche un diff clair des modifications effectuées :
   ```bash
   git diff README*
   ```

7. NE PAS commit - l'utilisateur fera `/push` s'il valide les changements
