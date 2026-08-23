# Static Build Specification

## Purpose

Pipeline de Tailwind CSS v4 con CLI standalone: `css/main.css` como única fuente `@theme`, `dist/main.css` generado, scripts npm, migración por swap atómico y rollback para un sitio estático que MUST permanecer servible en todo momento.

## Requirements

### Requirement: SB-1 — Pipeline de build CLI

El proyecto MUST proveer scripts npm usando `@tailwindcss/cli`: `npm run build` (compila `./css/main.css` → `./dist/main.css`, minificado) y `npm run watch` (lo mismo, en modo watch). `package.json` MUST declarar solo dos devDependencies: `tailwindcss` y `@tailwindcss/cli` (ambas `^4`). Sin bundler, sin framework de runtime, sin build CDN de navegador.

#### Scenario: El build emite CSS minificado

- GIVEN `package.json` con los scripts y devDeps declarados
- WHEN corre `npm run build`
- THEN se crea `dist/main.css`, minificado, y contiene las utilidades `@theme` generadas

#### Scenario: Watch reconstruye ante cambios

- GIVEN `npm run watch` corriendo
- WHEN cambia `css/main.css` o un archivo HTML fuente escaneado
- THEN `dist/main.css` se regenera

### Requirement: SB-2 — Estructura de la fuente

`css/main.css` MUST ser la única fuente de verdad: `@import "tailwindcss";` + `@theme` (tokens según la spec de design-tokens) + `@custom-variant dark` fijado a `data-theme`. Los archivos HTML MUST ser descubribles por el scanner de contenido (directiva `@source` si están fuera de los paths de scan por defecto).

#### Scenario: Todos los tokens en la fuente procesada

- GIVEN que `npm run build` ya corrió
- WHEN se inspecciona el `dist/main.css` generado
- THEN contiene los tokens definidos en el `@theme` de `css/main.css`
- AND cualquier `@theme` definido en un archivo que Tailwind nunca procesa está ausente (no existe ninguno)

### Requirement: SB-3 — Migración por swap atómico del link

La migración al design system MUST ser un swap único y atómico: el `<link>` en `index.html` apunta a `./dist/main.css` (reemplazando `css/main.css`). El commit del swap MUST NOT mezclar markup BEM legado con clases de utilidades — sin estados semi-migrados.

#### Scenario: El commit del swap es atómico

- GIVEN el commit de migración
- WHEN se inspecciona
- THEN contiene el swap del link más el markup totalmente migrado
- AND ningún commit mezcla clases BEM con utilidades

### Requirement: SB-4 — Plan de rollback

El sitio MUST permanecer servible sin el build. El rollback MUST ser un único `git revert` de los commits de swap/migración más el borrado del `dist/` generado; el `index.html` + `css/main.css` legados MUST permanecer intactos en el historial de git.

#### Scenario: El revert restaura el sitio legado

- GIVEN una regresión encontrada después de la migración
- WHEN se revierten los commits del cambio y se elimina `dist/`
- THEN el `index.html` + `css/main.css` legados sirven el dashboard original

#### Scenario: Sin estado de rollback semi-migrado

- GIVEN el punto de revert
- WHEN se ejecuta el rollback
- THEN el sitio está totalmente en estado legado o totalmente migrado — nunca ambos

### Requirement: SB-5 — Salida generada no commiteada

`dist/` MUST NOT commitearse (ya está en `.gitignore`). El `dist/main.css` generado es un artefacto de build. Los archivos de tooling de Node (`node_modules/`, `package-lock.json` si se genera) MUST seguir el manejo estándar de `.gitignore`.

#### Scenario: dist permanece sin trackear

- GIVEN un build que produjo `dist/main.css`
- WHEN se inspecciona `git status`
- THEN `dist/` no aparece como untracked ni staged

### Requirement: SB-6 — Íconos en regiones Alpine como contrato de build

Cualquier patrón de componente que renderice íconos dentro de regiones renderizadas por Alpine (`x-for`/`x-if`) MUST resolverlos como archivos SVG estáticos servidos desde `assets/icons/` y vincularlos vía `:src` (o SVG inline para esas regiones), de modo que aparezcan sin ejecutar JavaScript de íconos posterior al render. El contrato MUST NOT depender de `feather.replace()` — superseded: la librería Feather por CDN fue eliminada del proyecto por decisión del usuario.

#### Scenario: Los íconos de render tardío son SVGs

- GIVEN una sección poblada por Alpine después de la carga
- WHEN la sección renderiza
- THEN los íconos aparecen como SVGs de `assets/icons/` vinculados vía `:src`
- AND no queda ningún tag `<i data-feather>` crudo visible en esa sección

## Acceptance Criteria

- `npm run build` emite `dist/main.css` minificado desde `css/main.css`.
- El swap del link es un commit atómico; el rollback es un `git revert` + borrado de `dist/`.
- `dist/` nunca staged; el sitio es servible con y sin el build.
- Las regiones de íconos renderizadas por Alpine muestran SVGs (contrato `assets/icons/` + `:src` implementado).
