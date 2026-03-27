---
name: lint-all
description: Corrige TOUTES les erreurs de lint
allowed-tools: Bash, Read, Edit, Write, Grep, Glob
---

Corrige TOUTES les erreurs de lint sans exception :

1. Détecte le package manager et lance le lint :
   ```bash
   pnpm lint  # ou npm run lint / yarn lint
   ```

2. Corrige TOUT :
   - Erreurs
   - Warnings
   - Problèmes de style
   - Tout ce que le linter signale

3. Utilise aussi les auto-fix si disponibles :
   ```bash
   pnpm lint --fix  # ou équivalent
   ```

4. Corrige manuellement ce qui reste

5. Relance jusqu'à 0 erreurs/warnings
   - **Max 3 tentatives.** Si les erreurs persistent après 3 cycles de correction, arrête et fais un récap des erreurs restantes à l'utilisateur.

6. Récap de toutes les corrections
