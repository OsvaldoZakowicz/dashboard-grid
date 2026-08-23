# Archive Report — tailwind-v4-design-system

**Fecha de archivo**: 2026-08-23
**Ciclo SDD**: completo — planificado, implementado, verificado y archivado.
**Modo de artefactos**: hybrid (openspec + Engram).

## Qué se archivó

El change `tailwind-v4-design-system` (Tailwind v4 Design System — Monochrome Harmony) se movió desde `openspec/changes/tailwind-v4-design-system/` hacia:

```
openspec/changes/archive/2026-08-23-tailwind-v4-design-system/
```

Contenido archivado (audit trail inalterable a partir de esta fecha):

- `proposal.md` — propuesta del cambio.
- `exploration.md` — exploración previa.
- `spec.md` — spec resumen del change.
- `specs/` — deltas consolidados (`design-tokens/`, `ui-components/`, `static-build/`).
- `design.md` — diseño técnico.
- `tasks.md` — 13/13 tareas marcadas completas (checkboxes de Fase 3/4 reconciliados por apply tras el W3 del verify; audit trail limpio al momento del archivo).
- `apply-progress.md` — registro de ejecución por work units (13 commits).
- `verify-report.md` — verificación independiente.

Nota de proceso: este archivo es un reintento correctivo. El intento previo ya había sincronizado los deltas hacia los specs globales (verificado sin duplicados); esta pasada solo movió la carpeta, escribió este reporte y persistió el reporte en Engram. No se modificó nada bajo `openspec/specs/`.

## Specs globales (fuente de verdad)

Los deltas fueron sincronizados previamente y quedaron aterrizados en:

| Dominio | Ruta | Requisitos |
|---|---|---|
| Design Tokens | `openspec/specs/design-tokens/spec.md` | 6 (DT-1..DT-6) |
| UI Components | `openspec/specs/ui-components/spec.md` | 12 (UC-1..UC-12) |
| Static Build | `openspec/specs/static-build/spec.md` | 6 (SB-1..SB-6) |

**Recuento final: 24 requisitos (6 DT + 12 UC + 6 SB)**, sin duplicados, con el wording toggleable de UC-3 incorporado.

## Desviaciones capturadas

1. **UC-3 — Menú lateral toggleable en todas las resoluciones.** La spec original contemplaba drawer <48rem + rail estático ≥48rem. La implementación final (commit `5f59299`) dejó el sidebar toggleable también en desktop (`:class="menuOpen ? 'flex' : 'hidden'"`). Desviación intencional del usuario, documentada en el delta sincronizado a `openspec/specs/ui-components/spec.md`.
2. **Feather JS → SVGs locales con `:src` (UC-10 / SB-6).** La mitigación planeada para Feather icons × Alpine x-for quedó **superseded**: se eliminó la dependencia CDN/JS y los íconos son SVG estáticos autohospedados en `assets/icons/` (variantes white/graphite). El requisito de re-render en x-for se resuelve vinculando la ruta vía `:src="n.icon"`, sin necesidad de re-scan.
3. **Dark mode declarado pero sin UI dark.** `@custom-variant dark (&:where([data-theme=dark], [data-theme=dark] *));` quedó declarado en `css/main.css` (DT-6 PASS), pero la implementación visual dark no se construyó — fuera de scope confirmado por el usuario. Tailwind v4 no emite la variante al dist mientras no existan utilidades `dark:*`.

## Estado de verificación

- Veredicto del verify-report: **PASS WITH WARNINGS** — **22/24 requisitos PASS** post-fixes.
- Excepciones documentadas: UC-8 (PARTIAL por `aria-hidden` faltante en el `<img>` del x-for de index.html, corregido post-verificación) y UC-3 (FAIL estricto solo bajo la lectura original del escenario; la desviación toggleable fue aceptada y absorbida por la spec).
- CRITICAL: ninguno. Build verde (`npm run build`, exit 0), `dist/main.css` ~20KB minificado, SHA256 prefijo `60b2a1585af16e507d243d73b61978e5`.
- HEAD al verificar: `develop @ 5f6117b`.

## Trazabilidad Engram

- Reporte de archivo persistido con topic_key `sdd/tailwind-v4-design-system/archive-report` (proyecto `dashboard-grid`).
