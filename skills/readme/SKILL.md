---
name: readme
description: Met à jour le README selon les derniers changements
allowed-tools: Bash, Read, Edit, Grep, Glob
---

Met à jour le README en fonction des derniers changements :

1. Trouve le README :
   ```bash
   find . -maxdepth 1 -iname "readme*" -type f | head -1
   ```
   - Si aucun README trouvé → demander à l'utilisateur s'il veut en créer un. Si non → STOP.

2. Analyse les changements pertinents depuis la dernière modification du README :
   ```bash
   # Depuis le dernier commit qui a touché le README
   LAST_README_CHANGE=$(git log -1 --format=%H -- README* readme* 2>/dev/null)

   if [ -n "$LAST_README_CHANGE" ]; then
     git log --oneline "$LAST_README_CHANGE"..HEAD
     git diff "$LAST_README_CHANGE"..HEAD --stat
   else
     # README jamais commité ou repo neuf
     git log --oneline -20
     git diff --stat HEAD~$(git rev-list --count HEAD | xargs -I{} sh -c 'echo $(( {} < 10 ? {} : 10 ))')..HEAD 2>/dev/null
   fi
   ```

3. Lis le README actuel et compare avec les changements :
   - Identifie ce qui est obsolète
   - Identifie ce qui manque (nouvelles features, nouveaux fichiers, commandes, etc.)
   - Identifie ce qui doit être mis à jour

4. Met à jour le README :
   - Garde le style, le format et la langue existants
   - Ajoute/modifie uniquement les sections impactées par les changements
   - Ne supprime pas d'infos encore pertinentes
   - En cas de doute sur la pertinence d'une section, la laisser et signaler à l'utilisateur

5. Affiche un diff clair des modifications :
   ```bash
   git diff README* readme*
   ```

6. NE PAS commit — l'utilisateur fera `/push` s'il valide
