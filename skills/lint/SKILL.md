---
name: lint
description: Vérifie et corrige les erreurs de lint importantes
allowed-tools: Bash, Read, Edit, Write, Grep, Glob
---

Vérifie le lint et corrige les erreurs importantes :

1. Détecte le package manager et lance le lint :
   ```bash
   pnpm lint  # ou npm run lint / yarn lint
   ```

2. Analyse les erreurs et corrige uniquement :
   - Les erreurs (pas les warnings mineurs)
   - Les problèmes qui pourraient causer des bugs
   - Les violations de sécurité

3. Ne corrige PAS :
   - Les préférences de style mineures
   - Les warnings sans impact

4. Relance le lint pour vérifier
   - **Max 3 tentatives.** Si les erreurs persistent après 3 cycles de correction, arrête et fais un récap des erreurs restantes à l'utilisateur.

5. Récap concis de ce qui a été corrigé
