---
name: review-short
description: Review concise du code (PR ou branche)
allowed-tools: Bash, Read, Grep, Glob
---

Fais une code review CONCISE :

1. Vérifie s'il existe une PR ouverte :
   ```bash
   gh pr view --json number 2>/dev/null
   ```

2. Si PR existe : `gh pr diff`
   Sinon : `git diff origin/main...HEAD`

3. Review en mode ultra-concis :
   - Liste uniquement les problèmes importants (bugs, sécu, perf)
   - Max 5-10 bullet points
   - Pas de blabla, que l'essentiel
   - Si tout est ok, dis juste "RAS"
