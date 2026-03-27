---
name: fix-ci
description: Analyse et corrige les erreurs de CI (boucle watch jusqu'à green)
allowed-tools: Bash, Read, Edit, Write, Grep, Glob
---

Corrige les erreurs de CI sur la branche actuelle. Boucle jusqu'à ce que la CI passe (max 5 cycles).

## Boucle de correction (max 5 cycles)

### 1. Récupérer l'état de la CI

```bash
gh pr checks 2>/dev/null || gh run list --branch $(git branch --show-current) --limit 5
```

Si tout est green → informer l'utilisateur et STOP.

### 2. Récupérer les logs du run en échec

```bash
# Trouver le dernier run échoué
RUN_ID=$(gh run list --branch $(git branch --show-current) --status failure --limit 1 --json databaseId --jq '.[0].databaseId')
gh run view $RUN_ID --log-failed
```

### 3. Analyser et corriger

- Identifier si l'erreur vient du code (fixable) ou de l'infra/config (pas fixable par du code)
- Si c'est un problème d'infra (runner down, timeout réseau, secret manquant) → informer l'utilisateur et STOP
- Sinon, corriger le code

### 4. Vérifier en local si possible

Selon le type d'erreur, lancer le check correspondant en local (build, lint, tests) avant de push.

### 5. Commit et push

```bash
git add -u
git commit -m "fix(ci): description du fix"
git push
```

- Ne pas inclure de Co-Authored-By

### 6. Attendre le résultat de la CI

```bash
# Attendre que le nouveau run démarre puis surveiller
sleep 10
RUN_ID=$(gh run list --branch $(git branch --show-current) --limit 1 --json databaseId --jq '.[0].databaseId')
gh run watch $RUN_ID
```

- **Si la CI passe** → afficher un résumé des corrections et STOP
- **Si la CI échoue encore** → retour à l'étape 2 (nouveau cycle)
- **Max 5 cycles.** Si ça échoue encore après 5 corrections, faire un récap complet des erreurs restantes et STOP.
