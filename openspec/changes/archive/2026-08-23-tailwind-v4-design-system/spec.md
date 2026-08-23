# Delta Spec — tailwind-v4-design-system

## Change Summary

Static design system with Tailwind CSS v4 (standalone CLI), Monochrome Harmony palette, Spanish UI, Alpine.js + Feather kept. Replaces the Bootstrap-gray palette, fixes verified WCAG failures (white on light-green 2.49:1, white-on-red badge 4.0:1), consolidates duplicated dropdowns into one accessible pattern, removes dead CSS, and introduces token-driven styling with an atomic link-swap migration and single-commit rollback.

## Delta Status

All 3 capabilities are **NEW** (`openspec/specs/` was empty before this change) — written as full specs in the main specs tree, not deltas.

| Capability | Type | Requirements | Status |
|---|---|---|---|
| `design-tokens` | NEW | DT-1 … DT-6 | Added |
| `ui-components` | NEW | UC-1 … UC-12 | Added |
| `static-build` | NEW | SB-1 … SB-6 | Added |

Modified: none. Removed: none. Renamed: none.

## Spec Listing

| Spec | Path |
|---|---|
| Design Tokens | `openspec/specs/design-tokens/spec.md` |
| UI Components | `openspec/specs/ui-components/spec.md` |
| Static Build | `openspec/specs/static-build/spec.md` |

## Key Contract Points (cross-referenced)

- WCAG pairing matrix (design-tokens DT-3) is the authority for all color decisions in ui-components (UC-5, UC-7, UC-9).
- Alpine × Feather mitigation (static-build SB-6 / ui-components UC-10) is a pattern-level requirement, not an implementation detail.
- `dist/` gitignore contract (static-build SB-5) and atomic swap (SB-3/SB-4) guard the rollback plan from the proposal.
- Delivery forecast ≈700–900 lines exceeds the 400-line review budget → chained/stacked PR slices resolved at sdd-tasks (ask-always).

## Next

Ready for sdd-design.
