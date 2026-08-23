# Tasks: Tailwind v4 Design System (Monochrome Harmony)

## Review Workload Forecast

| Field | Value |
|-------|-------|
| Estimated changed lines | ~700–800 (WU1 ~35, WU2 ~300, WU3 ~350–450) |
| 400-line budget risk | High |
| Chained PRs recommended | Yes |
| Delivery strategy | ask-always |
| Chain strategy | pending |

Decision needed before apply: Yes
Chained PRs recommended: Yes
Chain strategy: pending
400-line budget risk: High

**User decision (orchestrator asks):** A — 3-PR chain as designed: homepage unstyled after PR2 (link → now-Tailwind `css/main.css`), atomic swap kept in PR3 (SB-3). B — fold swap into PR2 → ~500 lines, `size:exception`.

### Suggested Work Units

| Unit | Goal | PR | Test cmd | Harness | Rollback |
|------|------|----|----------|---------|----------|
| 1 | Build pipeline; site unchanged | 1 → develop | `npm run build && ls dist/main.css` | N/A — no UI change | Revert PR1 |
| 2 | `css/main.css` source + showcase | 2 → PR1 branch | `npm run build`; grep raw hex = 0 | Serve: pairs DT-3, focus, disabled, reflow | Revert PR2 |
| 3 | `index.html` + atomic swap | 3 → PR2 branch | `npm run build` + serve; grep hex = 0 | Serve: dropdown, drawer, SVG x-for | Revert PR3 + `rm -rf dist/` (SB-4) |

## Phase 1: Infraestructura — WU1/PR1

- [x] 1.1 Create `package.json`: `build`/`watch` scripts (`tailwindcss -i ./css/main.css -o ./dist/main.css --minify|--watch`), devDeps `tailwindcss`+`@tailwindcss/cli` `^4` [SB-1]
- [x] 1.2 `npm install`; commit `package-lock.json`; `.gitignore` keeps `dist/`+`node_modules/` [SB-5]
- [x] 1.3 Verify: build emits `dist/main.css`, nothing staged, site unchanged. Commit: `chore: agregar pipeline de build tailwind v4`

## Phase 2: Tokens y showcase — WU2/PR2

- [x] 2.1 Rewrite `css/main.css`: `@import "tailwindcss"` + `@theme` (hexes DT-2, aliases, `--shadow-*` [DT-5]) + `@custom-variant dark` data-theme [DT-6] + `@source "../*.html"` [SB-2]; drop legacy `:root`/`letter-spacing:1px` [DT-1, DT-4]
- [x] 2.2 Create `components.html` (español): header, drawer, dropdown, badges/status, cards (`@container`), buttons+estados, links, notifications, footer, type scale, token table, Alpine seam [UC-12, UC-4..UC-11]
- [x] 2.3 Verify build + showcase: pairs DT-3, focus ring [DT-5], disabled [UC-7], reflow [UC-6]. Commit: `feat: agregar tokens monochrome harmony y showcase de componentes`

## Phase 3: Migración index — WU3/PR3

- [x] 3.1 Rebuild `index.html`: `lang="es"`, 3 secciones, solo utilitarios (sin BEM), `overflow-x-auto`, badges ≥10px, fix `<id>`→`<i>` mail [UC-1, UC-5, UC-6, DT-1]
- [x] 3.2 Alpine: `Alpine.data('dropdown')` único, drawer, `x-effect` Feather en x-for/x-if [UC-2, UC-3, UC-10, SB-6]
- [x] 3.3 Swap atómico: `<link>`→`./dist/main.css` en el mismo commit, sin mezclar BEM+utilitarios [SB-3]
- [x] 3.4 Verificar: ESC/click-outside + `aria-expanded` [UC-2], drawer <md/rail ≥md [UC-3], SVG en x-for [UC-10, SB-6], `aria-label` icon-only [UC-4]. Commit: `feat: migrar index.html a utilitarios y activar dist/main.css`

## Phase 4: Verificación global y polish

- [x] 4.1 Rollback: `git revert` PR3 (+PR2) + `rm -rf dist/` restaura legacy sin estados mixtos [SB-4]
- [x] 4.2 Build final + greps: tokens resuelven, sin `letter-spacing:1px`, sin hex fuera de `css/main.css`, sin `dist/` staged [DT-1, DT-4, SB-5]
- [x] 4.3 Confirmar commits en español (chore:/feat:) y `state.yaml` actualizado
  - Nota: commits en español confirmados (13 commits chore:/feat:/fix:/refactor:/revert:). `state.yaml` no existe en el flujo de este cambio; el estado vive en apply-progress.md + verify-report.md.
