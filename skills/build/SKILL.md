---
name: build
description: Lance le build et corrige les erreurs
allowed-tools: Bash, Read, Edit, Write, Grep, Glob
---

Lance le build et corrige les erreurs :

1. Détecte le package manager (pnpm, npm, yarn) :
   ```bash
   [ -f pnpm-lock.yaml ] && echo "pnpm" || ([ -f yarn.lock ] && echo "yarn" || echo "npm")
   ```

2. Lance le build :
   ```bash
   pnpm build  # ou npm run build / yarn build
   ```

3. Si erreurs :
   - Analyse chaque erreur
   - Corrige dans le code
   - Relance le build pour vérifier
   - **Max 3 tentatives.** Si les erreurs persistent après 3 cycles de correction, arrête et fais un récap des erreurs restantes à l'utilisateur.

4. Une fois le build OK :
   - Fais un récap concis des corrections effectuées
   - Ne commite PAS automatiquement (laisse l'utilisateur décider)
