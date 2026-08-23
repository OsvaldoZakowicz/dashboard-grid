# Proposal: Tailwind v4 Design System (Monochrome Harmony)

## Intent

Turn the static dashboard into a portable design system: token-driven Tailwind v4 CSS with documented components, WCAG-safe. Current problems: Bootstrap-gray palette (`#343a40`, `#f8f9fa`) with no identity; contrast failures (white on light-green 2.49:1, red badge 4.0:1); duplicated dropdowns (two identical Alpine blocks); dead CSS (`.role`, `.user`); a11y gaps (`lang="en"` with Spanish copy, invalid `<id>` element, no menu semantics); 10 hardcoded `:root` vars, no token system.

## Scope

### In Scope
- Tailwind v4 standalone CLI build; source `css/main.css` → `dist/main.css` (dist/ gitignored)
- Monochrome Harmony tokens in `@theme`: Dust Grey, Ivory, Tuscan Sun, Shadow Grey, Graphite, White + WCAG pairing rules
- Rebuilt `index.html` (Spanish UI, `lang="es"`): overview (metrics), inventory (table + status tags), clients (table + badges)
- `components.html` showcase: every component state + token reference
- Consolidated single dropdown (`role="menu"`, `aria-*`), sidebar drawer, cards, buttons, badges, links, notifications
- Keep Alpine.js + Feather; mitigate `feather.replace()` not re-running for `x-if`/`x-for` (design decision)
- Cleanup: dead CSS, invalid `<id>`, global 1px letter-spacing

### Out of Scope
- Dark mode (later via pinned `@custom-variant`); committing dist/; CDN browser build; Next.js/Vue migration; runtime JS framework

## Capabilities

All NEW (`openspec/specs/` is empty):

- `design-tokens`: palette, WCAG pairing rules, typography/spacing/radius/shadow/breakpoint tokens in `@theme`
- `ui-components`: portable markup patterns (header, sidebar, dropdown, card, badge, button, link, notification) with documented Alpine seam + Feather constraint
- `static-build`: CLI pipeline, `@custom-variant` dark pin, atomic link swap, dist/ gitignored

Modified: None.

## Approach

Standalone CLI (exploration-recommended): 2 devDeps, no bundler; `@theme` tokens + component-layer CSS live in processed source. Atomic migration point: one `<link>` swap in `index.html`. Preflight reset is an intentional visual redesign (documented). Modules reduced from 6 placeholders to 3 rich sections. Delivery: forecast ≈700–900 changed lines exceeds 400-line budget → chained/stacked PR slices, resolved at sdd-tasks (ask-always).

## Affected Areas

| Area | Impact | Description |
|------|--------|-------------|
| `index.html` | Modified | Rebuild; link → `dist/main.css`; `lang="es"` |
| `css/main.css` | Modified | Becomes Tailwind source (`@import`, `@theme`, `@custom-variant`) |
| `components.html` | New | Showcase gallery |
| `package.json` | New | devDeps + build/watch scripts |
| `dist/main.css` | New | Generated, gitignored |

## Risks

| Risk | Likelihood | Mitigation |
|------|------------|------------|
| Preflight reset breaks visuals | High | Treat as redesign; verify per component in showcase |
| Tuscan Sun misuse (white on gold 1.55:1 FAIL) | Med | Enforce pairing rule: gold with dark text only; document in tokens spec |
| Feather icons not replaced in Alpine-rendered nodes | Med | Design mitigation (x-effect re-render or inline SVG) |
| Budget overflow 700–900 lines | High | Chained PR slices; ask-always at sdd-tasks |

## Rollback Plan

Legacy `index.html` + `css/main.css` stay intact in git history. Rollback = revert link tag + delete `dist/` (single `git revert`). No half-migrated states: never mix BEM + utilities in the swap commit.

## Dependencies

- Node/npm (dev-only, for CLI); Tailwind v4 + `@tailwindcss/cli` devDeps
- CDN Alpine 3 + Feather (unchanged); `dist/` already in `.gitignore`

## Success Criteria

- [ ] WCAG AA pairs: dark-on-light, light-on-dark, Tuscan Sun with dark text only
- [ ] Single dropdown pattern (no duplicate x-data)
- [ ] No hardcoded hex outside `@theme`; tokens drive all surfaces
- [ ] Portable markup: components render without framework logic
- [ ] `npm run build` emits `dist/main.css`; site servable after swap; `lang="es"`, invalid elements and menu semantics fixed
