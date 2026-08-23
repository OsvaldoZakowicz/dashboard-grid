# Exploration: Tailwind v4 Design System (dashboard-grid)

Date: 2026-08-10 · Phase: sdd-explore · Mode: hybrid (OpenSpec + Engram)
Branch: develop · Strict TDD: false · Review budget: 400 lines (ask-always)

## 1. Current State Audit

Single static dashboard: `index.html` (291 lines) + `css/main.css` (395 lines, BEM + CSS custom properties), Alpine.js 3.x (jsdelivr CDN, deferred), Feather icons (unpkg CDN, `feather.replace()`). No package.json, no build, no tests, no CI. Git: branch `develop`, commit style Spanish loosely conventional.

### Structure & tokens

- `:root` holds 10 custom properties: grays are literally Bootstrap palette values (`#343a40`, `#495057`, `#dee2e6`, `#f8f9fa` ≈ Bootstrap gray-800/700/300/100) + brand olive greens `#4f772d` / `#90a955` (hue ≈ 86–97°, NOT pure green).
- Layout: CSS Grid on `.panel` with areas `header/content/footer`; sidebar is an absolutely-positioned drawer toggled by Alpine (`menu--closed`), width 80vw → 50vw → 25vw across 480/768px breakpoints.
- Content grid: 1 → 2 → 3 columns with card span/order modifiers (`card--md`, `card--lg`).

### Components present (de-facto inventory)

| Component | Notes |
|---|---|
| Header (topbar) | brand h1, actions, menu toggle |
| Dropdown (user + notifications) | duplicated Alpine x-data block, `x-id` + `aria-expanded`/`aria-controls`, `@click.outside`, escape close — same pattern twice |
| Notification item | icon + message + action link |
| Sidebar drawer | Alpine `menuOpen` toggle, `menu--closed` transform |
| Card | base + `--md`/`--lg` grid modifiers, header/icon/subtitle/content/footer |
| Icon button (`btn-menu`) | hamburger / close |
| Role badge `.role` | unused in index.html (dead CSS) |
| Footer | subtext + links |

### Alpine interactions

1. Sidebar open/close (inline `x-data="{ menuOpen }"`).
2. User dropdown + notifications dropdown (identical inline x-data, `toggle()/close()`, `x-id` for ARIA ids, `@focusin.window` outside-close).
3. Footer `x-text="message"` ("I ❤️ Alpine").

### A11y issues (verified contrast ratios)

| Pair | Ratio | Verdict |
|---|---|---|
| `#212529` on `#90a955` (card header text) | 5.87:1 | AA OK |
| `#4f772d` on `#f8f9fa` (`.link`) | 4.97:1 | AA borderline (small text) |
| `#f8f9fa` on `#90a955` (white text on light-green header, e.g. role badge) | 2.49:1 | **FAIL** |
| `#4f772d` on `#90a955` | 1.99:1 | **FAIL** |
| white `1` on `#ff0000` badge, 9px font | 4.00:1 | FAIL for normal text (9px < 14pt large-text exemption) |
| `#212529` on `#f8f9fa` (body/card text) | 14.63:1 | AA OK |
| `#343a40`/`#495057` bg with white text | 10.91 / 7.76:1 | AA OK |

Other markup issues: `<id data-feather="mail">` is an invalid HTML element (line 119); `bottom: top 20px` dead declaration in `.card__icon`; `letter-spacing: 1px` global on 14px text is unnecessarily wide; dropdowns use `<article>` + links without `role="menu"`/`menuitem` semantics; no focus trap or `aria-label` on icon-only buttons (title attr only); menu items are `<li>` with icons + text, not links. No dark mode. No container queries (grid spans hard-coded). Responsive behavior is fixed-column based; cards have fixed `height: 320px`.

## 2. Tailwind v4 Integration Strategy

Research source: Tailwind CSS official docs (tailwindcss.com, v4 branch). All three options confirmed current:

| Approach | Pros | Cons | Effort |
|---|---|---|---|
| **A. Vite + `@tailwindcss/vite`** | Official recommended path; dev server + HMR; auto content detection; nearest to Next.js/Vue ecosystems; minified output | Adds package.json + bundler; overkill for a pure static HTML page with no JS module graph (Alpine stays CDN); more moving parts for rollback | Med |
| **B. Standalone CLI (`@tailwindcss/cli`)** | Thinnest footprint (2 devDeps); `npx @tailwindcss/cli -i css/main.css -o dist/main.css --watch`; output is a static CSS file; HTML untouched at source level; tokens in `@theme` are 100% portable to Next.js/Vue; vendor prefixing built-in (Lightning CSS); no dev-server coupling | No HMR out of the box (pair with any static server, e.g. `python -m http.server`); needs `@source` if HTML lives outside scanned paths | Low |
| **C. Browser CDN (`@tailwindcss/browser`)** | Zero install, instant | **Officially dev-only — "not intended for production"**; a design system IS a production artifact; runtime CSS generation (FOUC, no precompiled output to commit); tokens not portable to Next.js/Vue as built output; no minification | Low (rejected) |

### Recommendation: **B — Standalone CLI (`@tailwindcss/cli`)**

- Preserves the "no framework" contract: no bundler, no transpilation; the repo stays static and servable.
- The portability contract is tokens + markup, not tooling — `@theme` tokens and utility classes in source CSS migrate to Next.js/Vue unchanged; the CLI is the thinnest wrapper to produce them.
- Rollback story (config rule: site must remain servable without the build): old `index.html` + `css/main.css` remain in git history; rollback = `git revert` of the change commits + delete generated `dist/`. The change keeps the legacy files untouched until swap point.
- Documented future path: when this design system graduates into a React/Vue app, switch to `@tailwindcss/vite` (option A) — same source `css/main.css`, same `@theme` block, zero token rework.
- Vite (option A) deferred: no JS module graph exists today; HMR buys little for static HTML edited by hand. Decision recorded so proposal/design phases don't re-litigate it.

**Build setup to propose** (for design phase):
```json
{
  "devDependencies": { "tailwindcss": "^4", "@tailwindcss/cli": "^4" },
  "scripts": {
    "build": "@tailwindcss/cli -i ./css/main.css -o ./dist/main.css --minify",
    "watch": "@tailwindcss/cli -i ./css/main.css -o ./dist/main.css --watch"
  }
}
```
`dist/` stays gitignored (matches existing .gitignore intent); `index.html` re-points to `./dist/main.css`. Optional later: commit `dist/` for GH Pages/Netlify zero-build deploys — flag as a proposal-phase decision.

## 3. Design Token Proposal

Direction: **keep the olive-green brand anchor** (identity continuity — the green is the only non-Bootstrap thing in the current palette), **replace all Bootstrap grays with a custom green-tinted neutral scale**, and derive a full 11-step green scale in oklch (hue 105 ≈ current brand family 86–97°). All values verified below.

### Brand green scale (oklch, hue 105)

| Token | oklch | Hex | vs white | vs green-100 |
|---|---|---|---|---|
| green-50 | 0.975 0.045 105 | `#fbf9d7` | — | — |
| green-100 | 0.945 0.070 105 | `#f3f0ba` | — | surface for dark text (13.56:1 with 950) |
| green-200 | 0.885 0.105 105 | `#e3dd8a` | — | surface (8.66:1 with neutral-800) |
| green-300 | 0.800 0.135 105 | `#cbc250` | — | decorative only |
| green-400 | 0.700 0.155 105 | `#aea200` | AA-large only (2.96:1) | decorative only |
| green-500 | 0.610 0.160 105 | `#938600` | AA-large only (3.72:1) | avoid for small text |
| **green-600** | 0.530 0.150 105 | `#7a6e00` | **5.18:1 AA** | primary button bg (white text) |
| **green-700** | 0.460 0.135 105 | `#645a00` | **6.99:1 AA** | accent text on white (replaces `#4f772d` ≈ 4.97:1) |
| green-800 | 0.390 0.115 105 | `#4f4700` | 9.40:1 AA | hover/pressed, strong links |
| green-900 | 0.320 0.095 105 | `#3b3400` | 12.52:1 AA | headings |
| green-950 | 0.250 0.070 105 | `#272300` | — | darkest |

### Custom neutral scale (identity, green-tinted hue 105, low chroma — NOT Bootstrap grays)

50 `#f7f6f0` · 100 `#edede7` · 200 `#ddddd7` · 300 `#cbcbc5` · 400 `#b0b0aa` · 500 `#8d8c87` · 600 `#696964` · 700 `#4e4d48` · 800 `#363631` · 900 `#242421` · 950 `#151512`.

Key ratios: neutral-900 vs white 15.56:1, neutral-700 vs white 8.47:1, white vs neutral-800 12.15:1, neutral-950 vs neutral-100 15.57:1 — all AA.

### Legacy → token mapping

| Current | → | Token |
|---|---|---|
| `--bg-color-header` `#343a40` | → | neutral-800 |
| `--bg-color-aside` `#495057` | → | neutral-700 |
| `--bg-color-body` `#dee2e6` | → | neutral-200/300 |
| `--bg-color-card` `#f8f9fa` | → | neutral-50/white |
| `--bg-color-footer` `#343a40` | → | neutral-800/900 |
| `--bg-card-header` `#90a955` | → | green-200/300 as accent surface + **neutral-950 text** (fixes the 2.49:1 fail) |
| `--bg-card-footer` `#ced4da` | → | neutral-200 |
| `--color-ascent` `#4f772d` | → | green-700/800 |
| `--text-color-dark` `#212529` | → | neutral-900 |
| `--hover-color` rgba(255,255,255,.8) | → | `white/80` utility |

### Typography

- `--font-sans`: system stack (Tailwind v4 default is fine: `ui-sans-serif, system-ui, ...`); no webfont — zero-dependency contract. Fonts can be added later via `@fontsource` when porting (tokens travel).
- Type scale: Tailwind defaults (`text-xs` 0.75rem → `text-4xl` 2.25rem). Map: brand h1 → `text-xl/2xl font-semibold tracking-tight`; card title → `text-lg font-semibold`; body → `text-sm`; subtext/footer → `text-xs`.
- **Drop global `letter-spacing: 1px`** → `tracking-normal`/`tight` on headings only. Current 1px on 14px harms readability and is not a brand feature.
- Line-heights: `leading-snug` headings, `leading-relaxed` body.

### Spacing / Radii / Shadows / Breakpoints

- Spacing: use Tailwind default `--spacing: 0.25rem` multiplier (`p-1`…`p-8`, `gap-*`). Legacy mapping: 0.5rem → `gap-2`, 1rem → `p-4`/`gap-4`, 1.5rem → `p-6`.
- Radii: defaults — buttons `rounded-sm`/`md` (replaces 4px), cards `rounded-lg` (0.5rem), dropdown panels `rounded-xl` (0.75rem).
- Shadows: define 2–3 elevation tokens via `@theme` overrides: `--shadow-card` (resting), `--shadow-popover` (dropdown/sidebar), `--shadow-focus-ring` (WCAG focus indicator — 3px outline in green-700).
- Breakpoints: map current 480/768 → `sm` (40rem) / `md` (48rem); add `--breakpoint-xs: 30rem` if 480px behavior must survive. `lg` (64rem) added free.
- Dark mode: ship the variant now even if only light ships: `@custom-variant dark (&:where([data-theme=dark], [data-theme=dark] *));` — token swap via `@theme inline` + `[data-theme=dark]` overrides.
- Container queries: `@container` on cards — card content adapts to its grid slot instead of fixed heights (replaces `height: 320px` + order/spans).

## 4. Component Inventory

Portable markup patterns (classes + structure travel to Next.js/Vue; behavior documented as Alpine-owned):

| # | Component | Current source | Notes / portability |
|---|---|---|---|
| 1 | Sidebar / drawer menu | `.menu` + Alpine `menuOpen` | responsive drawer; becomes static rail at md+; behavior = Alpine now |
| 2 | Header / topbar | `.header` | brand + actions slot; pure markup |
| 3 | Dropdown menu | `.dropdown-wrapper` (×2, duplicated) | **consolidate to ONE pattern**; proper `role="menu"`/`menuitem`; behavior = Alpine |
| 4 | Notification item | `.notification` | list-item pattern, icon + text + action |
| 5 | Badge (count) | `::before` red circle | token bg green-600/neutral-950 text AA-correct, ≥10px |
| 6 | Card (base + variants) | `.card` + `--md`/`--lg` | header/icon/subtitle/content/footer slots; container-query aware |
| 7 | Button (primary/secondary/ghost/icon) | `.btn-menu` | new variants; focus ring required |
| 8 | Link (`.link`) | — | green-700/800 on white, AA verified |
| 9 | Footer | `.footer` | subtext + links |
| 10 | Icon (Feather) | `data-feather` | keep CDN for demo; porting = swap to framework icon or inline SVG; sizes: sm 16 / md 18 / lg 24 |
| 11 | Typography primitives | h1/h2/subtext | documented scale |
| 12 | Status tag (`.role`, dead) | unused | revive or drop — decide in design |

JS-coupled (Alpine) components — dropdown, sidebar — must document the coupling boundary: markup + utility classes are the contract; `x-*` attributes are the explicit seam a React/Vue component later replaces.

## 5. Structure Proposal

```
dashboard-grid/
├── index.html              # rebuilt dashboard demo (points to dist/main.css)
├── components.html         # NEW: component showcase gallery (all states, tokens reference)
├── package.json            # only devDeps: tailwindcss + @tailwindcss/cli (NEW)
├── css/
│   ├── main.css            # SOURCE: @import "tailwindcss"; @theme; @custom-variant dark; @source
│   └── tokens.css          # optional split: pure @theme tokens (imported by main.css) — decide in design
├── dist/                   # generated output (gitignored)
├── openspec/
└── .atl/                   # unchanged
```

- `main.css` (source) holds the whole design system: `@import "tailwindcss";` + `@theme { ... }` (colors/fonts/radii/shadows/breakpoints) + `@custom-variant dark` + any component-layer classes (`@layer components` only for complex composites like card shells that read better as classes).
- `index.html` = the migrated dashboard (proves the system); `components.html` = gallery with every component state + token reference (hex + oklch + usage), serving as living documentation.
- Keep Spanish UI copy (config rule). Fix `<id>` element, drop dead CSS, restructure dropdowns into one pattern.
- Static-servable at every step: `index.html` swap from `css/main.css` → `dist/main.css` is the single atomic migration point.

## 6. Risks / Gotchas

1. **CSS-first config is the config**: `@theme` tokens must live in the processed CSS (or be `@import`-ed into it). Don't put tokens in a file Tailwind never sees. `@source` directive needed if HTML lands outside auto-scanned paths.
2. **Preflight reset is a visual break**: Tailwind v4 base reset will restyle everything (Verdana → system stack, 1px letter-spacing gone, margins zeroed). This is an intentional redesign, not a 1:1 migration — set expectations with the user.
3. **oklch + modern baseline**: v4 emits oklch colors (Chrome 111+, Safari 16.4+, FF 113+). Fine for a demo; document a hex fallback strategy if the showcase must run on older browsers.
4. **Alpine × Feather timing**: `feather.replace()` runs once at load; icons inside Alpine `x-if`/`x-for` (rendered later) stay as raw `<i>` tags. Current page only uses `x-show` (safe), but the dropdown/notification patterns must document the constraint. Options: call `feather.replace()` after Alpine render hooks, or move to inline SVG for the showcase.
5. **Dark mode variant**: default `dark:` maps to `prefers-color-scheme`; the design system should pin the `@custom-variant` to `data-theme` now for a future toggle — cheap now, costly retrofit later.
6. **Review budget**: index.html rewrite (~291 lines) + new components.html + main.css rewrite (~395 lines) + package.json ≈ 700–900 changed lines — **over the 400-line budget**; `delivery_strategy=ask-always` → proposal/tasks must plan chained or stacked PR slices.
7. **Rollback discipline**: keep the legacy `css/main.css` + `index.html` intact in history; the atomic swap (one link tag) is the revert point. Do not half-migrate (BEM + utilities mixed) in the same commit as the swap.
8. **Dropdown semantics**: current `<article>`-based dropdowns lack menu semantics; the consolidated pattern must ship `role="menu"`/`aria-*` + focus handling — a11y requirement, not a style nicety.
9. **`.role` dead component**: dead CSS (`.user`, `.role`) should be dropped or revived; don't carry it forward silently.
10. **Container queries scope**: `@container` requires `container-type: inline-size` on the card shell; the fixed `height: 320px` and `order`/`span` grid tricks go away — behavior change to document.

## Ready for Proposal

**Yes.** Exploration complete with verified evidence (contrast math, docs research, current-code audit). Next phase: sdd-propose with these pre-decided anchors: (a) standalone CLI integration, (b) olive-green identity palette (hue 105, oklch), (c) green-tinted neutral scale replacing Bootstrap grays, (d) single dropdown pattern, (e) index.html + components.html structure, (f) chained delivery expected (400-line budget risk). Open questions for the user in proposal: keep `.role` tag? commit `dist/` for zero-build deploys? dark mode in scope now or later?
