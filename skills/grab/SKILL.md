---
name: grab
description: Récupère les N derniers fichiers d'un dossier (Downloads par défaut) et les affiche/lit. Cross-platform (macOS, Linux, WSL). Supporte aliases (downloads, desktop, pictures, screenshots, documents, videos, music) et offset négatif pour skip les plus récents.
allowed-tools: Bash, Read
---

Récupère le ou les derniers fichiers d'un dossier et les affiche dans la conversation.

## Syntaxe

`/grab [folder] [count] [-offset]`

- `folder` (optionnel) : alias ou chemin (default: `downloads`)
- `count` (optionnel, entier positif) : nombre de fichiers à prendre (default: `1`)
- `-offset` (optionnel, entier négatif) : skip les N plus récents (default: `0`)

L'ordre des arguments numériques n'a pas d'importance, seul le signe compte.

## Exemples

| Commande | Folder | Count | Offset | Résultat |
|----------|--------|-------|--------|----------|
| `/grab` | downloads | 1 | 0 | dernier fichier |
| `/grab 2` | downloads | 2 | 0 | 2 derniers |
| `/grab -1` | downloads | 1 | 1 | avant-dernier |
| `/grab 1 -1` | downloads | 1 | 1 | avant-dernier |
| `/grab 3 -2` | downloads | 3 | 2 | skip 2 derniers, prendre 3 |
| `/grab desktop` | desktop | 1 | 0 | dernier du Desktop |
| `/grab desktop 2` | desktop | 2 | 0 | 2 derniers du Desktop |
| `/grab pictures 3 -1` | pictures | 3 | 1 | skip 1, prendre 3 dans Pictures |
| `/grab ~/projects/foo 5` | path | 5 | 0 | 5 derniers du path custom |

## Aliases de dossiers

| Alias | macOS / Linux | WSL |
|-------|---------------|-----|
| `dl`, `downloads` | `~/Downloads` | `C:\Users\<user>\Downloads` |
| `desktop` | `~/Desktop` | `C:\Users\<user>\Desktop` |
| `docs`, `documents` | `~/Documents` | `C:\Users\<user>\Documents` |
| `pics`, `pictures` | `~/Pictures` | `C:\Users\<user>\Pictures` |
| `ss`, `screenshots` | `~/Desktop` (macOS), `~/Pictures/Screenshots` (Linux) | `C:\Users\<user>\Pictures\Screenshots` |
| `videos` | `~/Movies` (macOS), `~/Videos` (Linux) | `C:\Users\<user>\Videos` |
| `movies` | `~/Movies` (macOS), sinon `~/Videos` | `C:\Users\<user>\Videos` |
| `music` | `~/Music` | `C:\Users\<user>\Music` |

Si l'argument n'est pas un alias, il est interprété comme chemin (absolu, relatif au home, ou relatif au cwd).

## Étape 1 - Parser les arguments

À partir des arguments du skill, déterminer trois valeurs :

- `FOLDER` : par défaut `downloads`. Si un arg n'est pas numérique, c'est lui.
- `COUNT` : par défaut `1`. Tout arg entier positif (sans signe) écrase cette valeur.
- `OFFSET` : par défaut `0`. Tout arg entier négatif écrase cette valeur (valeur absolue).

## Étape 2 - Lancer le script bash avec les valeurs parsées

Remplace `<FOLDER>`, `<COUNT>`, `<OFFSET>` par les valeurs déterminées à l'étape 1, puis exécute la commande ci-dessous **telle quelle**. Le `--noprofile --norc` et l'usage systématique de `command` sont là pour neutraliser tout alias/fonction shell de l'utilisateur (eza, etc.) qui pourrait polluer la sortie.

```bash
FOLDER="<FOLDER>" COUNT=<COUNT> OFFSET=<OFFSET> bash --noprofile --norc <<'GRAB_EOF'
set -eu
unalias -a 2>/dev/null || true

detect_os() {
  if [[ "${OSTYPE:-}" == "darwin"* ]]; then echo "mac"
  elif [[ -f /proc/version ]] && command grep -qi microsoft /proc/version 2>/dev/null; then echo "wsl"
  else echo "linux"; fi
}
OS=$(detect_os)

if [[ "$OS" == "wsl" ]]; then
  WIN_USER=$(cmd.exe /C "echo %USERNAME%" 2>/dev/null | command tr -d '\r\n')
  BASE="/mnt/c/Users/$WIN_USER"
else
  BASE="$HOME"
fi

resolve_folder() {
  local input="${1:-downloads}"
  local lower
  lower=$(command printf '%s' "$input" | command tr '[:upper:]' '[:lower:]')
  case "$lower" in
    dl|downloads)   command printf '%s\n' "$BASE/Downloads" ;;
    desktop)        command printf '%s\n' "$BASE/Desktop" ;;
    docs|documents) command printf '%s\n' "$BASE/Documents" ;;
    pics|pictures)  command printf '%s\n' "$BASE/Pictures" ;;
    ss|screenshots)
      if   [[ "$OS" == "mac" ]]; then command printf '%s\n' "$HOME/Desktop"
      else                            command printf '%s\n' "$BASE/Pictures/Screenshots"; fi ;;
    videos|movies)
      if   [[ "$OS" == "mac" ]]; then command printf '%s\n' "$BASE/Movies"
      else                            command printf '%s\n' "$BASE/Videos"; fi ;;
    music) command printf '%s\n' "$BASE/Music" ;;
    *)
      if   [[ -d "$input" ]];        then command printf '%s\n' "$input"
      elif [[ -d "$BASE/$input" ]];  then command printf '%s\n' "$BASE/$input"
      elif [[ -d "$HOME/$input" ]];  then command printf '%s\n' "$HOME/$input"
      elif [[ -d "$PWD/$input" ]];   then command printf '%s\n' "$PWD/$input"
      else command printf '\n'; fi ;;
  esac
}

DIR=$(resolve_folder "$FOLDER")
if [[ -z "$DIR" || ! -d "$DIR" ]]; then
  command printf 'ERROR: dossier introuvable: %s\n' "$FOLDER" >&2
  exit 1
fi

get_mtime() {
  if [[ "$OS" == "mac" ]]; then command stat -f '%m' "$1"
  else                          command stat -c '%Y' "$1"; fi
}

TMP=$(command mktemp)
trap 'command rm -f "$TMP"' EXIT

while IFS= read -r -d '' f; do
  mt=$(get_mtime "$f" 2>/dev/null) || continue
  command printf '%s\t%s\n' "$mt" "$f" >> "$TMP"
done < <(command find "$DIR" -maxdepth 1 -type f ! -name '.*' -print0 2>/dev/null)

TOTAL=$(command wc -l < "$TMP" | command tr -d ' ')
if [[ "$TOTAL" -eq 0 ]]; then
  command printf 'ERROR: aucun fichier dans %s\n' "$DIR" >&2
  exit 1
fi

START=$((OFFSET + 1))
RESULTS=$(command sort -rn "$TMP" | command tail -n +"$START" | command head -n "$COUNT" | command cut -f2-)

if [[ -z "$RESULTS" ]]; then
  command printf 'ERROR: offset trop grand (%s) pour %s fichier(s) dans %s\n' "$OFFSET" "$TOTAL" "$DIR" >&2
  exit 1
fi

command printf 'Dossier: %s\n' "$DIR"
command printf 'Total: %s fichier(s) | count=%s offset=%s\n' "$TOTAL" "$COUNT" "$OFFSET"
command printf '===GRAB_PATHS===\n'
command printf '%s\n' "$RESULTS"
command printf '===GRAB_END===\n'
GRAB_EOF
```

## Étape 3 - Lire et présenter les fichiers

1. Récupère les chemins entre les marqueurs `===GRAB_PATHS===` et `===GRAB_END===` (un chemin par ligne, dans l'ordre du plus récent au plus ancien).
2. Pour chaque fichier, utilise **directement** le tool `Read` avec le chemin absolu. **Ne lance JAMAIS de commande shell auxiliaire** (`ls`, `file`, `cat`, etc.) pour vérifier le fichier - ces commandes peuvent être aliasées (eza, bat, ...) et polluer la conversation. Le `Read` tool suffit (il gère images, PDFs, texte).
3. Si plusieurs fichiers : décris brièvement chacun.
4. Demande à l'utilisateur ce qu'il veut en faire, ou enchaîne avec le contexte de la conversation s'il y en a un.

## Notes techniques

- `bash --noprofile --norc` + `unalias -a` + `command <bin>` : isolation totale contre les alias/fonctions shell de l'utilisateur (eza remplaçant `ls`, etc.).
- Marqueurs `===GRAB_PATHS===` / `===GRAB_END===` : parsing déterministe même si la sortie a du bruit avant/après.
- Tri par `stat` cross-platform (BSD `stat -f` sur macOS, GNU `stat -c` sur Linux/WSL) - évite le bug `ls -t` sur NTFS via WSL.
- `maxdepth 1` : pas de récursion, uniquement le dossier cible.
- Ignore les fichiers cachés (`.DS_Store`, `.localized`, etc.).
- Sur WSL, le username Windows est auto-détecté via `cmd.exe /C "echo %USERNAME%"`.
