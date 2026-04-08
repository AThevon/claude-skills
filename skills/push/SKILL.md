---
name: push
description: Add, commit et push les changements. Ajouter "?" pour le mode garde-fou, "!" pour forcer sans confirmation même sur main.
allowed-tools: Bash, Read, Grep, Glob, AskUserQuestion
---

# Mode

Détecter le mode selon l'invocation :

- `/push` ou `/push <message>` → **mode CONFIANT** (comportement standard)
- `/push!` ou `/push! <message>` → **mode FORCE** (push même sur main sans demander confirmation)
- `/push?` → **mode GARDE-FOU** (analyse de légitimité avant push)

---

# Mode CONFIANT (`/push`)

Commit et push les changements en cours :

1. Vérifie l'état :
   ```bash
   git status
   ```
   - Si aucun changement (working tree clean) → informer l'utilisateur et STOP
   - Si la branche courante est `main` ou `master` → avertir l'utilisateur "Tu es sur main, tu es sûr de vouloir push ici ?" et attendre confirmation avant de continuer (utiliser `/push!` pour skip cette confirmation)

2. Stage les fichiers modifiés et nouveaux (pas de `git add -A`) :
   ```bash
   # Fichiers tracked modifiés
   git add -u
   # Fichiers untracked : les ajouter un par un après vérification (ignorer les fichiers suspects : *.log, *.tmp, .DS_Store, binaires lourds)
   git status --porcelain | grep '^??' | cut -c4-
   ```

3. Analyse les changements stagés pour créer un message de commit :
   ```bash
   git diff --cached --stat
   git diff --cached
   ```

4. Commit avec un message clair et concis :
   - Format : type(scope): description (ex: "fix(auth): handle expired tokens")
   - Ou format simple si plus approprié : "add user avatar upload"
   - PAS de Co-Authored-By
   - Court et descriptif

5. Push :
   ```bash
   # Si la branche n'a pas d'upstream
   git push -u origin $(git branch --show-current)
   # Sinon
   git push
   ```

6. Confirme le push avec le lien si disponible

---

# Mode FORCE (`/push!`)

Identique au mode CONFIANT mais **ne demande JAMAIS de confirmation**, même sur `main`/`master`. Push directement.

1. Vérifie l'état :
   ```bash
   git status
   ```
   - Si aucun changement (working tree clean) → informer l'utilisateur et STOP
   - Si sur `main`/`master` → continuer sans demander confirmation

2. Stage, commit et push (mêmes étapes 2 à 6 que le mode CONFIANT)

---

# Mode GARDE-FOU (`/push?`)

Version prudente. Analyse les changements en profondeur avant de décider si on push ou pas.

**Philosophie** : sceptique — vérifie que les changements sont cohérents, complets et intentionnels.

## 1. État des lieux

```bash
git status
git branch --show-current
```

- Si aucun changement → informer et STOP
- Si sur `main`/`master` → avertir "Tu es sur main, tu es sûr de vouloir push ici ?" et attendre confirmation avant de continuer

## 2. Inventaire des changements

Catégoriser TOUS les fichiers modifiés/ajoutés/supprimés :

```bash
git diff --cached --name-status   # staged
git diff --name-status             # unstaged
git status --porcelain | grep '^??' # untracked
```

## 3. Analyse de légitimité

Lire les diffs et évaluer chaque fichier :

```bash
git diff --cached
git diff
```

**Critères :**

| Check | Détail | Verdict |
|-------|--------|---------|
| **Fichiers suspects** | `.env`, credentials, secrets, tokens, clés API, `.pem`, `.key` | 🚫 BLOQUER |
| **Fichiers accidentels** | `.DS_Store`, `*.log`, `*.tmp`, `node_modules/`, `dist/`, `build/` | 🚫 BLOQUER |
| **Conflits non résolus** | `<<<<<<<`, `=======`, `>>>>>>>` | 🚫 BLOQUER |
| **Debug oublié** | `console.log`, `debugger`, `print()` ajoutés (hors tests) | ❓ DEMANDER |
| **Code commenté** | Gros blocs de code commenté ajoutés | ❓ DEMANDER |
| **TODO orphelins** | `TODO`, `FIXME`, `HACK`, `XXX` ajoutés | ❓ DEMANDER |
| **Fichiers hors scope** | Changements dans des fichiers sans rapport entre eux | ❓ DEMANDER |
| **Changements partiels** | Import sans utilisation, fonction déclarée jamais appelée | ❓ DEMANDER |

## 4. Décision

### ✅ PUSH — Tout est clean
Les changements sont cohérents, pas de fichier suspect, pas de debug oublié.
→ Procéder au staging, commit et push (même logique que le mode CONFIANT).

### ❓ DEMANDER — Doute sur certains points
Présenter un résumé structuré :

```
## Analyse des changements

### ✅ OK
- path/to/file.ts — description courte

### ⚠️ Points d'attention
- path/to/other.ts:42 — console.log() de debug encore présent
- path/to/thing.ts — semble hors scope du reste

### Verdict : tu veux que je push quand même ?
```

Attendre la réponse explicite avec AskUserQuestion.
- Si oui → stage, commit, push
- Si non → STOP, lister ce qu'il faudrait corriger

### 🚫 BLOQUER — Changements problématiques
Expliquer clairement pourquoi et STOP. Pas de question, pas de push.

```
## ⛔ Push refusé

- .env.local modifié — contient potentiellement des secrets
- Marqueurs de conflit dans src/lib/auth.ts

Corrige ces points et relance /push?
```

## 5. Commit et push (si autorisé)

Même étapes que le mode CONFIANT (staging sélectif, message conventionnel, push avec -u si besoin).
