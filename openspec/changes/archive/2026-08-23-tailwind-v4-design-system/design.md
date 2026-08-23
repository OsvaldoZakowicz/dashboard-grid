# Design: Tailwind v4 Design System (Monochrome Harmony)

## Technical Approach

Static design system: `css/main.css` as single Tailwind v4 source (`@theme` + dark variant) compiled by standalone `@tailwindcss/cli` to gitignored `dist/main.css`; utility-class markup in rebuilt `index.html` + `components.html` showcase; Alpine for behavior only (dropdown + drawer); Feather in Alpine regions re-rendered via `x-effect`; migration = atomic `<link>` swap, single-commit rollback. Implements DT-1..6, UC-1..12, SB-1..6.

## Architecture Decisions

### D1 — Build tooling: standalone CLI
| Option | Tradeoff | Decision |
|---|---|---|
| `@tailwindcss/cli` standalone | 2 devDeps, no config file | ✅ (exploration + proposal) |
| PostCSS plugin | config + plugin chain | Rejected — no bundler per scope |
| Buildless CDN build | runtime cost, weak dark pin | Rejected — out of scope (SB-1) |

### D2 — Source & token architecture (SB-2, DT-1..6)
`css/main.css` only: `@import "tailwindcss";` + `@theme` + `@custom-variant dark (&:where([data-theme=dark], [data-theme=dark] *));` (DT-6) + `@source "../*.html";`. Rejected: separate `tokens.css` (SB-2), JS config.

```css
@theme {
  --color-ivory:#e8eddf; --color-dust-grey:#cfdbd5; --color-tuscan-sun:#f5cb5c;
  --color-shadow-grey:#242423; --color-graphite:#333533; --color-white:#ffffff;
  --color-surface:var(--color-ivory);            /* bg-surface */
  --color-surface-raised:var(--color-dust-grey); /* bg-surface-raised */
  --color-ink:var(--color-shadow-grey);          /* text-ink */
  --color-ink-soft:var(--color-graphite);        /* text-ink-soft */
  --color-accent:var(--color-tuscan-sun);        /* bg-accent — dark text only */
  --color-on-dark:var(--color-white);            /* text-on-dark */
  --shadow-card:0 1px 3px rgb(36 36 35/.1); --shadow-popover:0 10px 25px rgb(36 36 35/.18);
  --shadow-focus-ring:0 0 0 3px var(--color-graphite);
}
```
Raw hexes exactly per DT-2; aliases reference them via `var()` (no literals, no drift). WCAG by construction: accent⇒`text-ink*`; dark⇒`text-on-dark`; light⇒`text-ink*` (verify greps forbidden combos — Testing). Legacy `:root` mapping (config rule): `--bg-color-header→bg-graphite` `--bg-color-aside→bg-graphite` `--bg-color-body→bg-surface` `--bg-color-card→bg-surface-raised` `--bg-color-footer→bg-shadow-grey` `--bg-card-header→bg-accent+text-ink` `--bg-card-footer→bg-surface-raised` `--text-color-white→text-on-dark` `--text-color-dark→text-ink` `--color-ascent→bg-accent` `--hover-color→hover:bg-on-dark/80`.

### D3 — Dropdown: one pattern (UC-2)
`Alpine.data('dropdown', …)` registered once at `alpine:init`; user + notification triggers both use `x-data="dropdown"`. Rejected: duplicated inline `x-data` (today's defect), web components (overkill).
```html
<div x-data="dropdown" @keydown.escape.window="close()" @click.outside="close()">
  <button @click="toggle()" :aria-expanded="open" :aria-controls="$id('menu')">
  <div role="menu" x-show="open" :id="$id('menu')">
    <a role="menuitem" href="#">mi perfil</a>
```
Escape/click-outside close and reset `aria-expanded` (UC-2); menuitems are native anchors; focus ring (D2) gives visible focus.

### D4 — Alpine × Feather (UC-10, SB-6): `x-effect` re-render
One `x-effect` per Alpine region rendering icons, with a reactive read as dependency:
```html
<div x-data="notifications" x-effect="items; feather.replace()">
  <template x-for="n in items" :key="n.id">
    <a :href="n.href"><i data-feather="bell" aria-hidden="true"></i><span x-text="n.text"></span></a>
  </template>
</div>
```
Reading `items` registers the dependency; the effect re-runs after every x-for render → `feather.replace()` re-scans. x-if: `open; feather.replace()`. Static icons keep the load-time call. Rejected: inline SVG (bloat), x-transition end hook (no initial-render fire), imperative `$nextTick` (couples behavior into data).

### D5 — Styling: utilities only, zero `@apply`
Class lists in markup are the portable contract (UC-1). Rejected: `@apply` component classes (v4 utility-first; indirection). Showcase documents each pattern's classes.

### D6 — Work units (review budget 400; forecast 700–900)
| WU | PR target | Content | Est. lines |
|---|---|---|---|
| 1 | PR1 → develop | `package.json` (build/watch), `.gitignore` — site unchanged | ~35 |
| 2 | PR2 → PR1 branch | `css/main.css` → Tailwind source + `components.html` | ~300 |
| 3 | PR3 → PR2 branch | `index.html` rebuild + atomic link swap (SB-3) | ~350–450 |

PR3 is the migration commit (BEM→utilities, never mixed). After PR2 the homepage is temporarily unstyled (link still → now-Tailwind `css/main.css`) — accepted so the swap stays atomic (SB-3); alternative: fold swap into PR2 (~500 lines, `size:exception`). Resolve at sdd-tasks (ask-always).

## Data Flow

    css/main.css ──CLI──▶ dist/main.css (gitignored)
    HTML ──▶ browser ── Alpine (dropdown·drawer) ─ x-effect ─▶ feather.replace() re-render

## File Changes

| File | Action | Description |
|---|---|---|
| `package.json` | Create | `build`/`watch` scripts + devDeps (SB-1) |
| `css/main.css` | Modify | Legacy BEM → Tailwind source (SB-2) |
| `index.html` | Modify | Rebuilt utilities, `lang="es"`, link swap (UC-1, SB-3) |
| `components.html` | Create | Showcase: states, tokens, Alpine seam, Spanish labels (UC-12) |
| `.gitignore` | Modify | Keep `dist/`/`node_modules/`; commit `package-lock.json` |
| `dist/main.css` | — | Generated, never committed (SB-5) |

## Interfaces / Contracts

```json
{ "scripts": { "build": "tailwindcss -i ./css/main.css -o ./dist/main.css --minify",
               "watch": "tailwindcss -i ./css/main.css -o ./dist/main.css --watch" },
  "devDependencies": { "tailwindcss": "^4.0.0", "@tailwindcss/cli": "^4.0.0" } }
```
Components: header, drawer, dropdown ×2, badge/status tag, card slots (`@container`, no fixed heights), buttons primary/secondary/ghost/icon, notification, footer, type scale. `index.html`: overview+metrics, inventory+status tags, clients+badges; tables `overflow-x-auto`; badges ≥10px; icons 16/18/24.

## Testing Strategy

| Layer | What | How |
|---|---|---|
| Build | minified dist, tokens resolve, no raw hex outside source, no global `letter-spacing:1px` | `npm run build` + grep at verify |
| A11y/Manual | dropdown ESC/outside; focus ring; SVGs in x-for; badge pairs | serve + manual; Playwright/axe deferred |
| Unit/E2E | — | No runner; `strict_tdd:false` |

## Threat Matrix

All rows `N/A` — no routing, git/PR automation, executable classification, or process integration; npm scripts are fixed commands with zero user-controlled input. No RED tests required.

## Migration / Rollout

1. PR1: infra — site unchanged.
2. PR2: tokens + showcase — homepage degraded (D6); `npm run build` before serving.
3. PR3: atomic swap — link → `./dist/main.css` + migrated markup (SB-3).
4. Rollback: `git revert` of PR3 (+PR2) + `rm -rf dist/`; legacy intact in history (SB-4).

## Open Questions

- [ ] DT-2 wording vs semantic aliases — clarify at archive; verify asserts the 6 raw hexes only.
- [ ] D6: 3-PR chain (degraded PR2) vs 2-PR with `size:exception` — resolve at sdd-tasks (ask-always).
- [ ] Commit `package-lock.json`? Proposed: yes.
