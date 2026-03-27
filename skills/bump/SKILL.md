---
name: bump
description: Bump la version du projet (analyse auto du type de bump)
allowed-tools: Bash, Read, Edit, Grep, Glob
---

Bump la version du projet selon le semantic versioning :

1. Analyse les changements depuis le dernier tag/release :
   ```bash
   LAST_TAG=$(git describe --tags --abbrev=0 2>/dev/null || echo "")

   if [ -z "$LAST_TAG" ]; then
     git log --oneline -30
   else
     git log --oneline $LAST_TAG..HEAD
   fi
   ```

2. Détermine le type de bump selon les changements :
   - Si l'utilisateur a spécifié le type (ex: `/bump major`, `/bump minor`), l'utiliser directement
   - Sinon, analyser les commits avec les Conventional Commits :
     - **MAJOR** (x.0.0) : `BREAKING CHANGE:` dans le body, ou `!` après le type (ex: `feat!:`)
     - **MINOR** (0.x.0) : commits préfixés `feat:` ou `feat(scope):`
     - **PATCH** (0.0.x) : `fix:`, `perf:`, `refactor:`, ou tout le reste
   - En cas de doute, proposer PATCH et demander confirmation

3. Trouve la version actuelle — utiliser les bons parseurs :
   ```bash
   # package.json (Node.js) — parser avec jq si dispo
   jq -r '.version' package.json 2>/dev/null || grep '"version"' package.json | head -1 | sed 's/.*"version": *"\([^"]*\)".*/\1/'

   # Cargo.toml (Rust) — version racine uniquement
   grep '^version' Cargo.toml 2>/dev/null | head -1

   # pyproject.toml (Python)
   grep '^version' pyproject.toml 2>/dev/null | head -1

   # VERSION file
   cat VERSION 2>/dev/null
   ```
   Si plusieurs fichiers de version existent, vérifier qu'ils sont synchronisés. Si incohérence → alerter l'utilisateur.

4. Calcule la nouvelle version :
   - Parse la version actuelle (ex: 1.2.3)
   - Applique le bump :
     - MAJOR: 1.2.3 → 2.0.0
     - MINOR: 1.2.3 → 1.3.0
     - PATCH: 1.2.3 → 1.2.4

5. Met à jour les fichiers de version :
   - `package.json` : utiliser `Edit` pour modifier le champ `"version"`
   - `package-lock.json` : si présent, lancer `npm install --package-lock-only` (ne PAS éditer manuellement)
   - `pnpm-lock.yaml` : si présent, lancer `pnpm install --lockfile-only`
   - `Cargo.toml`, `pyproject.toml`, `VERSION` : modifier avec `Edit`
   - README.md : mettre à jour les badges ou sections version si présentes

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

8. NE PAS commit automatiquement — l'utilisateur fera `/push` s'il valide
