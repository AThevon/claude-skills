---
name: design-excellence
description: "Pipeline design d'exception — orchestre Stitch (mockups), Nano Banana (assets visuels), 21st.dev Magic (composants UI), UX/UI Pro Max (design system), et frontend-design (code distinctif). Déclenché par : design, landing page, UI exceptionnelle, mockup, maquette, créer un site, interface premium."
allowed-tools: Bash, Read, Edit, Write, Grep, Glob, WebSearch, Agent
---

# Design d'Exception — Pipeline Complet

Tu orchestre tous les outils design disponibles pour produire des interfaces exceptionnelles. Suis ce pipeline dans l'ordre.

## Phase 1 — Comprendre le besoin

Avant tout, clarifie avec l'utilisateur :
- **Type de produit** : SaaS, landing, portfolio, e-commerce, dashboard, app mobile...
- **Audience cible** : B2B, B2C, dev, grand public...
- **Mots-clés visuels** : minimal, bold, luxe, playful, dark, corporate...
- **Stack technique** : inspecte `package.json` ou demande (React, Next.js, Vue, Svelte, HTML/Tailwind...)
- **Références** : des sites/apps que l'utilisateur aime ?
- **21st.dev Magic** : demande si l'utilisateur veut utiliser 21st.dev pour générer des composants UI de référence, et si oui, sur quel périmètre :
  - **Hero only** — uniquement le hero section (bon pour un coup de wow rapide)
  - **Composants clés** — hero, navbar, CTA, footer, cards (recommandé pour une landing)
  - **Full page** — tous les composants de la page passent par Magic
  - **Non** — skip complètement la Phase 5

## Phase 2 — Générer le Design System (UX/UI Pro Max)

Lance le design system generator pour obtenir les recommandations complètes :

```bash
python3 ~/.claude/plugins/athevon-skills/skills/ui-ux-pro-max/scripts/search.py "<product_type> <industry> <keywords>" --design-system -p "<Project Name>"
```

Complète avec des recherches ciblées si besoin :
```bash
# Styles spécifiques
python3 ~/.claude/plugins/athevon-skills/skills/ui-ux-pro-max/scripts/search.py "<keywords>" --domain style
# Typographie
python3 ~/.claude/plugins/athevon-skills/skills/ui-ux-pro-max/scripts/search.py "<keywords>" --domain typography
# Palettes couleur
python3 ~/.claude/plugins/athevon-skills/skills/ui-ux-pro-max/scripts/search.py "<keywords>" --domain color
# Guidelines stack
python3 ~/.claude/plugins/athevon-skills/skills/ui-ux-pro-max/scripts/search.py "<keywords>" --stack <stack>
```

## Phase 3 — Mockups visuels (Google Stitch)

Si le MCP Stitch est disponible, utilise-le pour :
- Générer des mockups UI complets de chaque page/écran
- Récupérer le HTML de référence avec `get_screen_code`
- Obtenir des captures avec `get_screen_image`

Utilise les résultats du design system (Phase 2) comme brief pour Stitch.

## Phase 4 — Assets visuels (Nano Banana)

Si le MCP Nano Banana est disponible, utilise-le pour :
- Générer des hero images, illustrations, backgrounds
- Créer des assets UI sur mesure (icônes illustrées, patterns, textures)
- Produire des visuels marketing (OG images, banners)

Donne des prompts précis qui reprennent la palette et le style du design system.

## Phase 5 — Composants UI de référence (21st.dev Magic)

**Skip cette phase si l'utilisateur a répondu "Non" en Phase 1.**

Si le MCP `magic` est disponible, utilise-le selon le périmètre choisi en Phase 1 :

| Périmètre | Composants à générer |
|---|---|
| Hero only | Hero section uniquement |
| Composants clés | Hero, navbar, CTA, footer, cards |
| Full page | Tous les composants/sections de la page |

### Stratégie d'utilisation :
1. Pour chaque composant du périmètre, décris-le en reprenant le design system (Phase 2) et les mockups (Phase 3)
2. Utilise le code généré comme point de départ, puis adapte-le au design system du projet
3. Ne copie jamais aveuglément — ajuste palette, typo, spacing selon les décisions des phases précédentes

## Phase 6 — Implémentation (Frontend Design + Creative)

Applique les principes du skill `frontend-design` d'Anthropic :
- **Refuse le design générique AI** — pas de gradient violet/bleu par défaut, pas d'Inter/Roboto sans raison
- **Direction artistique forte** — chaque interface a une personnalité
- **Typographie expressive** — Google Fonts, pairings intentionnels
- **Micro-interactions** — hover, focus, scroll, apparition
- **Performance** — GPU-accelerated, transform/opacity, lazy loading

### Contraste couleurs — CRITIQUE :
- **Chaque combinaison texte/fond doit être lisible dans TOUS les états** (default, hover, focus, active, disabled)
- Quand un hover change le background, le texte DOIT changer aussi pour rester lisible (ex: bg sombre → texte blanc, bg clair → texte sombre)
- Ratio minimum : **4.5:1** pour le texte normal, **3:1** pour le texte large (≥18px bold ou ≥24px)
- Vérifier systématiquement : boutons, liens, badges, tags, cards cliquables, nav items — tout élément interactif
- Ne jamais poser du texte sombre sur un bg coloré saturé au hover (erreur fréquente)
- En cas de doute, utiliser `color-contrast()` CSS ou tester manuellement chaque état

### Standards non négociables (du skill creative) :
- Travaille avec les technos déjà installées dans le projet
- Palette intentionnelle, pas de gris tièdes
- Contrastes forts, espaces blancs intentionnels
- Le mouvement guide l'attention (150-300ms pour les micro-interactions)

## Phase 7 — Validation

Avant de livrer, vérifie la checklist UX/UI Pro Max :

### Qualité visuelle
- [ ] Aucun emoji utilisé comme icône (SVG uniquement)
- [ ] Icons d'un set cohérent (Heroicons/Lucide)
- [ ] Hover states sans layout shift
- [ ] Couleurs du thème utilisées directement

### Interaction
- [ ] `cursor-pointer` sur tous les éléments cliquables
- [ ] Transitions smooth (150-300ms)
- [ ] Focus states visibles pour navigation clavier

### Accessibilité couleurs (passe en revue CHAQUE composant interactif)
- [ ] Contraste texte/fond minimum 4.5:1 à l'état **default**
- [ ] Contraste texte/fond minimum 4.5:1 à l'état **hover**
- [ ] Contraste texte/fond minimum 4.5:1 à l'état **focus**
- [ ] Contraste texte/fond minimum 4.5:1 à l'état **active**
- [ ] Si le hover change le background, le texte change aussi pour rester lisible
- [ ] Pas de texte sombre sur fond coloré saturé (vérifier boutons, badges, nav items, cards)
- [ ] Texte sur images/gradients : text-shadow ou overlay semi-transparent pour garantir la lisibilité

### Accessibilité générale
- [ ] Alt text sur toutes les images
- [ ] Labels sur les inputs
- [ ] `prefers-reduced-motion` respecté

### Responsive
- [ ] Testé à 375px, 768px, 1024px, 1440px
- [ ] Pas de scroll horizontal sur mobile
- [ ] Touch targets minimum 44x44px

## Ordre de priorité des outils

1. **UX/UI Pro Max** — toujours en premier pour le design system
2. **Stitch** — si disponible, pour les mockups de référence
3. **Nano Banana** — si disponible, pour les assets visuels
4. **21st.dev Magic** — si disponible, pour les composants UI de référence
5. **frontend-design + creative** — pour l'implémentation du code

Si un MCP n'est pas disponible, skip la phase correspondante et continue avec les autres.

$ARGUMENTS
