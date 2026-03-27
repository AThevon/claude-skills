---
name: clean-comments
description: Nettoie les commentaires trop situationnels ou liés à la conversation dans les changements de la branche
user-invocable: true
allowed-tools: Read, Grep, Glob, Bash, Edit
---

# Clean Comments

Détecte et nettoie les commentaires de code qui sont trop précis, trop situationnels, ou trop liés à une conversation spécifique dans les fichiers modifiés de la branche actuelle.

## Contexte

Quand on code avec un assistant IA, on génère parfois des commentaires qui :
- Expliquent POURQUOI on a fait un changement spécifique (contexte de conversation)
- Sont trop verbeux ou explicatifs
- Mentionnent des bugs spécifiques qu'on évitait
- Contiennent des warnings situationnels
- Ne seraient pas compréhensibles sans le contexte de la discussion

Ces commentaires doivent être nettoyés pour garder un code professionnel.

## Processus

### 1. Récupérer les fichiers modifiés

```bash
git diff --name-only main...HEAD
```

### 2. Analyser les commentaires ajoutés

Pour chaque fichier, regarder les lignes ajoutées (`+`) qui contiennent des commentaires :
- `//` pour JS/TS
- `/* */` pour JS/TS/CSS
- `#` pour Python/Shell/YAML
- `{/* */}` pour JSX

### 3. Détecter les commentaires problématiques

**Patterns à détecter** :

| Type | Exemple | Problème |
|------|---------|----------|
| **Commentaire en français** | `// Vérifie si l'utilisateur est connecté` | Doit être en anglais |
| **Explication de workaround** | `// On fait ça car sinon le state ne se met pas à jour` | Trop contextuel |
| **Référence à un bug** | `// Fix pour éviter le crash quand X` | Situationnel |
| **Warning trop précis** | `// IMPORTANT: ne pas utiliser Y ici car...` | Over-engineering |
| **Explication évidente** | `// On vérifie si l'utilisateur existe` | Redondant |
| **Historique** | `// Avant on faisait X mais maintenant...` | Pas pertinent |
| **Conversation** | `// Comme demandé, on gère ce cas` | Contextuel |
| **TODO vagues** | `// TODO: améliorer plus tard` | Inutile |

### 4. Actions

Pour chaque commentaire détecté :

1. **Supprimer** si :
   - Le code est auto-explicatif
   - Le commentaire est redondant
   - C'est un warning/note trop situationnel

2. **Traduire** si :
   - Le commentaire est en français (ou autre langue non-anglaise)
   - Traduire en anglais concis et professionnel

3. **Simplifier** si :
   - L'intention est bonne mais trop verbeuse
   - Garder uniquement l'essentiel

4. **Garder** si :
   - Explique une logique business non évidente
   - Documente une API publique
   - Warning légitime et permanent (ex: "Not implemented on iOS")

### 5. Format de sortie

Afficher les modifications proposées :

```markdown
## Fichier: `path/to/file.ts`

### Ligne 42 - SUPPRIMER
```typescript
// On évite de faire X car ça causait un bug avec le state
const value = computeValue();
```
**Raison** : Code auto-explicatif, commentaire situationnel

---

### Ligne 87 - SIMPLIFIER
**Avant** :
```typescript
// IMPORTANT: On doit absolument vérifier ce cas car sinon quand l'utilisateur
// arrive depuis la page précédente avec un état invalide ça crash
if (!isValid) return null;
```

**Après** :
```typescript
// Guard: invalid state
if (!isValid) return null;
```
```

### 6. Demander confirmation

Avant d'appliquer les modifications, lister tous les changements proposés et demander confirmation à l'utilisateur.

### 7. Appliquer les modifications

Une fois confirmé, utiliser l'outil Edit pour appliquer chaque modification.

## Règles de bon commentaire

Un bon commentaire :
- **Est toujours en anglais** (traduire si en français ou autre langue)
- Explique le **POURQUOI**, pas le **QUOI** (le code dit déjà quoi)
- Est intemporel (valide dans 6 mois sans contexte)
- Est concis (une ligne max si possible)
- Utilise un ton neutre et professionnel

## Output

1. Liste des commentaires problématiques avec proposition
2. Demande de confirmation
3. Application des modifications
4. Résumé final des changements effectués
