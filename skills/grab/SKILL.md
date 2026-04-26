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

Remplace `<FOLDER>`, `<COUNT>`, `<OFFSET>` par les valeurs déterminées à l'étape 1, puis exécute :

```bash
FOLDER="<FOLDER>" COUNT=<COUNT> OFFSET=<OFFSET> bash <<'GRAB_EOF'
set -eu

detect_os() {
  if [[ "${OSTYPE:-}" == "darwin"* ]]; then echo "mac"
  elif [[ -f /proc/version ]] && grep -qi microsoft /proc/version 2>/dev/null; then echo "wsl"
  else echo "linux"; fi
}
OS=$(detect_os)

if [[ "$OS" == "wsl" ]]; then
  WIN_USER=$(cmd.exe /C "echo %USERNAME%" 2>/dev/null | tr -d '\r\n')
  BASE="/mnt/c/Users/$WIN_USER"
else
  BASE="$HOME"
fi

resolve_folder() {
  local input="${1:-downloads}"
  local lower
  lower=$(printf '%s' "$input" | tr '[:upper:]' '[:lower:]')
  case "$lower" in
    dl|downloads)   echo "$BASE/Downloads" ;;
    desktop)        echo "$BASE/Desktop" ;;
    docs|documents) echo "$BASE/Documents" ;;
    pics|pictures)  echo "$BASE/Pictures" ;;
    ss|screenshots)
      if   [[ "$OS" == "mac" ]]; then echo "$HOME/Desktop"
      else                            echo "$BASE/Pictures/Screenshots"; fi ;;
    videos)
      if   [[ "$OS" == "mac" ]]; then echo "$BASE/Movies"
      else                            echo "$BASE/Videos"; fi ;;
    movies)
      if   [[ "$OS" == "mac" ]]; then echo "$BASE/Movies"
      else                            echo "$BASE/Videos"; fi ;;
    music) echo "$BASE/Music" ;;
    *)
      if   [[ -d "$input" ]];        then echo "$input"
      elif [[ -d "$BASE/$input" ]];  then echo "$BASE/$input"
      elif [[ -d "$HOME/$input" ]];  then echo "$HOME/$input"
      elif [[ -d "$PWD/$input" ]];   then echo "$PWD/$input"
      else echo ""; fi ;;
  esac
}

DIR=$(resolve_folder "$FOLDER")
if [[ -z "$DIR" || ! -d "$DIR" ]]; then
  echo "ERROR: dossier introuvable: $FOLDER" >&2
  exit 1
fi

get_mtime() {
  if [[ "$OS" == "mac" ]]; then stat -f '%m' "$1"
  else                          stat -c '%Y' "$1"; fi
}

TMP=$(mktemp)
trap 'rm -f "$TMP"' EXIT

while IFS= read -r -d '' f; do
  mt=$(get_mtime "$f" 2>/dev/null) || continue
  printf '%s\t%s\n' "$mt" "$f" >> "$TMP"
done < <(find "$DIR" -maxdepth 1 -type f ! -name '.*' -print0 2>/dev/null)

TOTAL=$(wc -l < "$TMP" | tr -d ' ')
if [[ "$TOTAL" -eq 0 ]]; then
  echo "ERROR: aucun fichier dans $DIR" >&2
  exit 1
fi

START=$((OFFSET + 1))
RESULTS=$(sort -rn "$TMP" | tail -n +"$START" | head -n "$COUNT" | cut -f2-)

if [[ -z "$RESULTS" ]]; then
  echo "ERROR: offset trop grand ($OFFSET) pour $TOTAL fichier(s) dans $DIR" >&2
  exit 1
fi

echo "Dossier: $DIR"
echo "Total: $TOTAL fichier(s) | count=$COUNT offset=$OFFSET"
echo "---"
echo "$RESULTS"
GRAB_EOF
```

## Étape 3 - Lire et présenter les fichiers

1. Récupère les chemins listés après `---` dans la sortie.
2. Pour chaque fichier, utilise le tool **Read** pour le lire/afficher (Claude Code est multimodal pour les images, PDFs, etc.).
3. Si plusieurs fichiers : décris brièvement chacun.
4. Demande à l'utilisateur ce qu'il veut en faire, ou enchaîne avec le contexte de la conversation s'il y en a un.

## Notes techniques

- Tri par `stat` cross-platform (BSD `stat -f` sur macOS, GNU `stat -c` sur Linux/WSL) - évite le bug `ls -t` sur NTFS via WSL.
- `maxdepth 1` : pas de récursion, uniquement le dossier cible.
- Ignore les fichiers cachés (`.DS_Store`, `.localized`, etc.).
- Sur WSL, le username Windows est auto-détecté via `cmd.exe /C "echo %USERNAME%"`.
