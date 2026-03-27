---
name: update-pr
description: Met à jour le titre et la description d'une PR existante
allowed-tools: Bash, Read, Grep
---

Met à jour la PR existante sur la branche actuelle :

1. Vérifie qu'une PR existe :
   ```bash
   gh pr view --json number,title,body,baseRefName
   ```

2. Analyse tous les commits depuis la création de la PR :
   ```bash
   gh pr view --json commits --jq '.commits[].messageHeadline'
   git log $(gh pr view --json baseRefName --jq .baseRefName)..HEAD --oneline
   ```

3. Regarde aussi le diff complet pour comprendre les changements :
   ```bash
   gh pr diff
   ```

4. Génère un nouveau titre et une nouvelle description :
   - Titre : clair, concis, reflète l'ensemble des changements
   - Description : explique le quoi et le pourquoi, pas trop longue
   - PAS de mention de Claude/AI

5. Met à jour la PR :
   ```bash
   gh pr edit --title "nouveau titre" --body "nouvelle description"
   ```

6. Affiche le lien de la PR mise à jour
