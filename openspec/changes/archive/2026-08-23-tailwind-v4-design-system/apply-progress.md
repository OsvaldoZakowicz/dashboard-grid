# Apply Progress — tailwind-v4-design-system

Estado: COMPLETO. Implementación finalizada en rama `develop` y pusheada a origin.

## Work Units ejecutados

### WU1/PR1 — Infraestructura de build [SB-1, SB-5]
- Commit `f811e51`: `package.json` (tailwindcss ^4 + @tailwindcss/cli ^4, scripts build/watch), `css/main.css` mínimo (`@import "tailwindcss"` + `@source "../*.html"`).
- `.gitignore`: `dist/` + `node_modules/` excluidos.

### WU2/PR2 — Tokens y showcase [DT-1..6]
- Commit `be7c3d6`: `css/main.css` con `@theme` completo (6 hexes crudos DT-2, aliases semánticos, `--shadow-card/popover/focus-ring` DT-5); `components.html` showcase (346 líneas, español).

### WU3/PR3 — Migración index [UC-1..12, SB-3, SB-6]
- Commit `9cd6dc1`: `index.html` reconstruido solo con utilitarios (0 clases BEM, 0 colores default Tailwind), Alpine factory pattern (`alpine:init` + `Alpine.data('dropdown'/'notifications')`), swap atómico del `<link>` a CSS compilado.
- Commit `db72355`: `<link>` apuntado a `dist/main.css` (el source no es procesable por browser) + regla `[x-cloak]`.
- Commit `7d1f227`: sidebar visible en desktop vía `:class="menuOpen ? 'flex' : 'hidden'"` (x-show generaba display:none inline que pisaba md:block); eliminado `x-effect` sobre el x-for de notificaciones.
- Commits `6bc38bf` + `28a4133` + `5f59299`: feather-icons JS reemplazado por SVGs locales autohospedados en `assets/icons/` (variantes white/graphite); aplicado a index.html y components.html; menú lateral toggleable en todas las resoluciones.
- Commit `a0ab362`: iconos agregados al dropdown de usuario del showcase (paridad con index.html).
- Commit `5f6117b`: footer de index linkea components.html.

## Desviaciones respecto al plan original
- Feather-icons por CDN JavaScript fue ELIMINADO por decisión del usuario: los iconos son SVG estáticos locales (stroke pre-coloreado). El requisito UC-10/SB-6 (re-render de iconos en x-for) se resuelve vinculando la ruta del SVG vía `:src` — ya no requiere re-scan de feather.
- Dark mode (DT-6): la implementación visual dark NO se construyó (fuera de scope confirmado por el usuario), pero el `@custom-variant dark` SÍ quedó declarado en css/main.css desde PR2 — DT-6 verificó PASS.
- Los specs de capacidad se escribieron directamente en `openspec/specs/{design-tokens,ui-components,static-build}/spec.md`; los deltas correspondientes se consolidaron luego bajo `openspec/changes/tailwind-v4-design-system/specs/`.

## Verificación ad-hoc realizada durante apply
- greps: 0 clases BEM, 0 colores default Tailwind, 0 referencias feather/unpkg, 0 SVGs rotos, target="_blank" presente, aria-labels/aria-hidden/role=menu presentes.
- `npm run build` exitoso en cada commit relevante (~20KB minificado).
