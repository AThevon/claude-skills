---
name: bump
description: Bump la version du projet (analyse auto du type de bump)
allowed-tools: Bash, Read, Edit, Grep, Glob
---

Bump la version du projet selon le semantic versioning :

1. Analyse les changements depuis le dernier tag/release :
   ```bash
   # Trouve le dernier tag
   LAST_TAG=$(git describe --tags --abbrev=0 2>/dev/null || echo "")

   # Si pas de tag, regarde tous les commits
   if [ -z "$LAST_TAG" ]; then
     git log --oneline -30
     git log --stat -15
   else
     git log --oneline $LAST_TAG..HEAD
     git log --stat $LAST_TAG..HEAD
   fi
   ```

2. Détermine le type de bump selon les changements :
   - **MAJOR** (x.0.0) : Breaking changes, API incompatibles, refacto majeure
     - Mots-clés : BREAKING, breaking change, remove, suppression d'API
   - **MINOR** (0.x.0) : Nouvelles fonctionnalités rétro-compatibles
     - Mots-clés : feat, add, nouveau, nouvelle feature
   - **PATCH** (0.0.x) : Bug fixes, corrections, améliorations mineures
     - Mots-clés : fix, bugfix, correction, typo, refactor mineur

3. Trouve et lis les fichiers de version :
   ```bash
   # package.json (Node.js)
   cat package.json 2>/dev/null | grep '"version"'

   # Cargo.toml (Rust)
   cat Cargo.toml 2>/dev/null | grep '^version'

   # pyproject.toml (Python)
   cat pyproject.toml 2>/dev/null | grep '^version'

   # VERSION file
   cat VERSION 2>/dev/null
   ```

4. Calcule la nouvelle version :
   - Parse la version actuelle (ex: 1.2.3)
   - Applique le bump approprié
   - MAJOR: 1.2.3 → 2.0.0
   - MINOR: 1.2.3 → 1.3.0
   - PATCH: 1.2.3 → 1.2.4

5. Met à jour TOUS les fichiers contenant la version :
   - package.json (et package-lock.json si présent)
   - Cargo.toml
   - pyproject.toml
   - VERSION
   - README.md (badges, section version si présente)
   - Tout autre fichier pertinent trouvé

6. Génère/Met à jour le CHANGELOG.md si présent :
   - Ajoute une nouvelle section avec la date
   - Liste les changements depuis le dernier tag
   - Format : Keep a Changelog (https://keepachangelog.com)

7. Affiche un résumé :
   - Version précédente → Nouvelle version
   - Type de bump détecté et pourquoi
   - Fichiers modifiés
   ```bash
   git diff --stat
   ```

8. NE PAS commit automatiquement - l'utilisateur fera `/push` s'il valide

Note : Si l'utilisateur spécifie explicitement le type (ex: `/bump major`), utilise ce type au lieu de l'analyse automatique.
