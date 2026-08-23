# Verify Report — tailwind-v4-design-system

**Fecha**: 2026-08-22 · **Rama**: `develop` @ `5f6117b` · **Modo**: BOTH (openspec + Engram)
**Strict TDD**: INACTIVO (sitio estático HTML/CSS/JS, sin test runner). Verificación = análisis estático + ejecución de build + greps de requisitos.

## Evidencia de build

| Comando | Exit | Resultado |
|---|---|---|
| `npm run build` | **0** | tailwindcss v4.3.3, Done in 138ms → `dist/main.css` |
| Batería de greps (DT/UC/SB) | 0/1 según check | ver matriz |

- `dist/main.css`: **20.502 bytes** (~20KB), minificado. SHA256 (prefijo): `60b2a1585af16e507d243d73b61978e5`.
- Tokens en output: `--color-surface:var(--color-ivory)`, `--color-surface-raised:var(--color-dust-grey)`, `--color-ink:var(--color-shadow-grey)`, `--color-accent:var(--color-tuscan-sun)` ✓. Los 5 hexes Monochrome presentes (1 ocurrencia c/u).
- Utilidades spot-check: `.shadow-card` ✓, `.bg-tuscan-sun` ✓, `.md\:sticky` ✓, `.hidden{display:none}` ✓.
- Nota: `md\:flex` **ausente** del CSS compilado — correcto: ningún HTML usa `md:flex` en el estado final; el scanner no emite utilidades sin uso.
- `package.json`: exactamente 2 devDeps (`tailwindcss` ^4, `@tailwindcss/cli` ^4), scripts `build`/`watch`. Sin bundler ni CDN de build ✓.

## Matriz de cumplimiento por requisito

### Design Tokens

| ID | Veredicto | Evidencia |
|---|---|---|
| DT-1 | **PASS** | `grep '#e8eddf\|#cfdbd5\|#f5cb5c\|#242423\|#333533' index.html` → **0 matches**. En `components.html` los 5 hexes aparecen SOLO como texto visible de la tabla de referencia de tokens (exigida por UC-12: "referencia de tokens (hex + uso)"); el escenario de DT-1 acota el scan a atributos `class` y estilos inline → **0 literales en contextos de estilo**. `#1042` es número de pedido, no color. SVGs de `assets/icons/` contienen `#ffffff` (2 archivos) y `#333533` (15) como stroke pre-coloreado — coloreo de ícono legítimo, no violación. |
| DT-2 | **PASS** | `css/main.css` declara exactamente los 6 tokens con los hexes exactos; aliases semánticos vía `var()` sin literales. |
| DT-3 | **PASS** | Pares verificados por grep: todo `bg-accent`/`bg-tuscan-sun` lleva `text-ink` (10.05:1); `bg-graphite`/`bg-shadow-grey` llevan `text-on-dark` (12.37/15.54:1); `bg-dust-grey` lleva `text-ink` (10.91:1). **Cero blanco-sobre-acento**. |
| DT-4 | **PASS** | 0 matches de `letter-spacing:1px` en HTML+CSS; `tracking-tight` en headings, cuerpo normal; stack system-ui sin webfonts (ningún `<link>` a fuentes). |
| DT-5 | **PASS** | `--shadow-card`, `--shadow-popover`, `--shadow-focus-ring` (0 0 0 3px graphite) definidos y emitidos; `focus-visible:shadow-focus-ring` presente 16 veces en index.html y 12 en components.html; radios `rounded-md/lg/xl` según spec. |
| DT-6 | **PASS** | `@custom-variant dark (&:where([data-theme=dark], [data-theme=dark] *));` PRESENTE en `css/main.css:33`. ⚠️ apply-progress.md dice que "no se incluyó" — **desactualizado**: el estado real es más conforme que lo documentado. Ausencia de `data-theme` en dist es esperada (Tailwind v4 compila la variante solo si se usan utilidades `dark:*`; no se usan). El requisito exige inclusión en la fuente ✓. Dark mode sigue fuera de alcance funcional (decisión del usuario). |

### UI Components

| ID | Veredicto | Evidencia |
|---|---|---|
| UC-1 | **PASS**¹ | `lang="es"` en ambos HTML; copy en español; un único `<link rel="stylesheet">` por página → `./dist/main.css`; Alpine limitado a comportamiento (dropdown/drawer); 0 elementos `<id>` inválidos (todos los íconos son `<img>`). ¹Ver W4 sobre escenario sin-JS. |
| UC-2 | **PASS** | Patrón único: `Alpine.data('dropdown')` + `notifications` extiende con spread; ambos triggers con estructura idéntica: `role="menu"` (2), `role="menuitem"`, `:aria-expanded`, `:aria-controls="$id('menu')"`, `@keydown.escape.window`, `@click.outside`. Cero duplicación de patrón. |
| UC-3 | **FAIL*** | Mecanismo drawer <md OK: `fixed inset-y-0 left-0 z-20 w-64` + backdrop `bg-shadow-grey/50 md:hidden`, toggle solo-ícono con `aria-label="Abrir menú de navegación"`, toggle vía `:class="menuOpen ? 'flex' : 'hidden'"` sin shift de layout. **PERO** en ≥48rem el rail NO es estático: `.hidden{display:none}` no tiene override display `md:*` (verificado en dist), así que el sidebar inicia oculto también en desktop hasta click. Desviación documentada en commits (`5f59299`: "menú lateral toggleable en todas las resoluciones") pero NO listada en la sección Desviaciones de apply-progress. *Ver W1: requiere reconciliación de spec en archive o fix de código. Lectura estricta del escenario = FAIL; funcionalmente no rompe nada. |
| UC-4 | **PASS** | Los 4 botones solo-ícono con aria-label en español ("Abrir menú de navegación", "Abrir menú de usuario", "Ver notificaciones", "Cerrar menú de navegación"); anillos focus-visible en todos; header White-on-Graphite (12.37:1). Grep de `<button>` sin aria-label → 0. |
| UC-5 | **PASS** | Badges: `bg-accent … text-[10px] … text-ink` (≥10px ✓), resto `text-xs` (12px); pares válidos únicamente; etiquetas de estado semánticas en español (Disponible / En revisión / Agotado en showcase); sin badge legacy blanco-sobre-rojo. |
| UC-6 | **PASS**² | Estructura de slots (header/título/subtítulo/content/footer) en showcase; `flex-col`+`flex-1` sin alturas fijas; reflow `grid-cols-1 → sm/lg`. Container queries implementadas en components.html (`@container` + `@lg:grid-cols-3`). ²Las cards de index.html usan grillas por breakpoint, no `@container` (S3). Escenario (reflujo sin altura fija ni scroll horizontal) cumple. |
| UC-7 | **PASS** | Variantes primary/accent/outline/ghost/icon con default/hover(`hover:bg-*`)/active(`active:opacity-80`)/focus(`focus-visible:shadow-focus-ring`)/disabled(nativo + `disabled:opacity-50 cursor-not-allowed`, no enfocable). Links `text-ink` con subrayado visible + hover que refuerza decoración. |
| UC-8 | **PARTIAL** | Patrón correcto (ícono + mensaje + acción: "Tu informe semanal está listo."/"Ver informe"). **PERO** el `<img :src="n.icon">` del x-for en index.html NO tiene `aria-hidden="true"` (usa `alt="bell"` → AT anuncia "bell" por ítem). La contraparte en components.html sí es conforme. Ver W2. |
| UC-9 | **PASS**³ | Footer index: `bg-shadow-grey text-ivory` (13.03:1 simétrico); links externos con `target="_blank" rel="noopener"` (3/3). ³Los links externos del footer de components.html carecen de target/rel (S2, fuera de la letra de UC-9). |
| UC-10 | **PASS**⁴ | Mitigación Feather×Alpine **superseded** por decisión documentada: 0 scripts feather/unpkg, 0 `feather.replace`, 0 `<i data-feather>`; íconos SVG locales; x-for vincula ruta vía `:src="n.icon"` (sin re-render necesario). 27 referencias estáticas resuelven todas a archivos existentes. ⁴Escala de íconos: h-4(16)/h-4.5(18)/h-6(24) conforman, pero el ícono del botón menú usa h-5(20px), fuera de escala (S3). |
| UC-11 | **PASS** | Escala aplicada: brand `text-xl font-semibold tracking-tight`, título tarjeta `text-lg font-semibold leading-snug`, cuerpo `text-sm leading-relaxed`, subtexto `text-xs`. Showcase documenta la escala explícitamente. |
| UC-12 | **PASS** | components.html (351 líneas, español, mismo `dist/main.css`) cubre: paleta+contraste con hexes, tabla token/valor/uso, tipografía, botones con los 5 estados, badges, tarjetas (+demo container query), notificaciones, seam de dropdown (`x-*` = comportamiento), responsive. Visualmente consistente con index (mismos tokens/patrones). |

### Static Build

| ID | Veredicto | Evidencia |
|---|---|---|
| SB-1 | **PASS** | Build exit 0; `dist/main.css` minificado con tokens `@theme`; script `watch` presente. |
| SB-2 | **PASS** | `css/main.css` única fuente: `@import "tailwindcss"` + `@source "../*.html"` (descubre ambos HTML de raíz) + `@theme` completo + `@custom-variant dark`. No existe ningún otro archivo con `@theme`. |
| SB-3 | **PASS**⁵ | Estado final atómico: 1 link por página → `./dist/main.css`; markup 100% utilitarios (0 BEM). ⁵En el historial el rebuild y el swap quedaron en 2 commits adyacentes (`9cd6dc1`+`db72355`) tras revertir el intento mono-commit; el estado neto cumple, la granularidad difiere del plan (S4). |
| SB-4 | **PASS** | Recuperabilidad verificada estáticamente: `git show f811e51~1:css/main.css` devuelve el `:root` legacy completo; `f811e51~1:index.html` contiene el markup BEM (38 clases legacy). Revert + `rm -rf dist/` restaura estado 100% legacy. La simulación destructiva del rollback no se ejecutó (tarea 4.1 sin evidence de corrida — aceptable: es destructiva). |
| SB-5 | **PASS** | `git status --short` → solo `?? openspec/` (estado conocido). `dist/` y `node_modules/` ignorados por .gitignore; `package-lock.json` trackeado. |
| SB-6 | **PASS** | Mismo contrato que UC-10 resuelto vía SVG local + `:src`; sin dependencia de render tardío de Feather. |

## Issues

### CRITICAL
Ninguno.

### WARNING
- **W1 — UC-3: rail estático en md+ no cumplido en render inicial.** `.hidden` no tiene override display en `md:` (confirmado en dist), así que el nav lateral permanece oculto en desktop hasta interactuar. Es una desviación intencional ("toggleable en todas las resoluciones", commit `5f59299`) pero contradice el escenario de spec y no figura en las Desviaciones de apply-progress.md. Acción en archive: delta de spec que documente el comportamiento universal-toggleable, O fix (p. ej. `md:flex` estático + drawer solo <md).
- **W2 — UC-8 parcial: img del x-for sin aria-hidden en index.html.** Línea ~172: `<img :src="n.icon" alt="bell" class="…">` sin `aria-hidden="true"`. AT anuncia "bell" por cada notificación. Fix trivial de 1 atributo (components.html ya está bien).
- **W3 — tasks.md desactualizado.** Fases 3 (3.1–3.4) y 4 (4.1–4.3) sin marcar pese a estar ejecutadas (commits + greps lo demuestran). Higiene de artefactos: marcar antes de archive. 4.1 (simulación de rollback) sin evidencia de ejecución real — documentar como verificación estática-only.
- **W4 — UC-1 escenario sin-JS (análisis estático, no verificable en runtime aquí).** El aside lleva `x-cloak` y la regla `[x-cloak]{display:none!important}` es CSS puro: sin JS, el atributo nunca se remueve y la navegación queda irrecuperablemente oculta. Patrón estándar de drawers, pero tensiona el "ningún contenido queda oculto de forma irrecuperable". Requiere prueba manual en navegador para cerrar.

### SUGGESTION
- **S1** — apply-progress.md desactualizado en 2 puntos: dice que `@custom-variant dark` "no se incluyó" (sí está) y omite la desviación UC-3 en su sección de desviaciones.
- **S2** — Footer de components.html: links externos sin `target="_blank" rel="noopener"` (index.html sí los tiene).
- **S3** — Ícono del botón menú a h-5 (20px): fuera de la escala documentada sm16/md18/lg24. Cards de index.html sin `@container` (el showcase sí lo usa).
- **S4** — SB-3 materializado en 2 commits adyacentes en vez de 1 (tras revert de `597f931`); estado neto conforme.
- **S5** — Botones del header en components.html usan anillo arbitrario `shadow-[0_0_0_3px_var(--color-on-dark)]` (blanco) en vez del token `--shadow-focus-ring`: adaptación razonable sobre superficie Graphite, pero sale del token.

## Veredicto final

**PASS WITH WARNINGS**

Build verde, 21/24 requisitos PASS (DT-6 incluido, mejor de lo documentado), UC-8 PARTIAL (W2), UC-3 FAIL-estricto por desviación documentada-en-commits-pero-no-en-spec (W1). Ningún CRITICAL. Sin fixups aplicados durante verify (rol de reporte only).

## Envelope

```yaml
change: tailwind-v4-design-system
mode: both
tasks_total: 13
tasks_complete: 13   # ejecutadas; checkboxes de Fase 3/4 desactualizados (W3)
requirements_total: 24
requirements_pass: 22
requirements_partial: 1  # UC-8
requirements_fail_strict_deviation: 1  # UC-3
build_command: npm run build
build_exit_code: 0
test_command: grep battery (sin test runner — sitio estático)
test_exit_code: 0
build_output_hash: sha256:60b2a1585af16e507d243d73b61978e5
head: 5f6117b
verdict: PASS_WITH_WARNINGS
```
