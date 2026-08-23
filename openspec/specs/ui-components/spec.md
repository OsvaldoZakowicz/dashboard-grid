# UI Components Specification

## Purpose

Patrones de markup portables para el sistema de diseño Monochrome Harmony: contratos de estructura, estados, requisitos ARIA y comportamiento responsivo. El markup + las clases utilitarias de Tailwind son el contrato portable; los atributos `x-*` de Alpine son el seam de comportamiento documentado; la copia de UI es en español (`lang="es"`).

## Requirements

### Requirement: UC-1 — Contrato de markup portable

Cada componente MUST renderizar correctamente en HTML estático plano con el `dist/main.css` construido — sin JavaScript para mostrarse. Los atributos de Alpine MUST limitarse al comportamiento interactivo (dropdown, toggle del sidebar) y MUST poder intercambiarse por un componente de framework sin cambiar el markup. La raíz de la página MUST declarar `lang="es"`; la copia de UI MUST estar en español.

#### Scenario: Los componentes renderizan sin JS

- GIVEN un servidor estático sirviendo `index.html` con `dist/main.css`
- WHEN JavaScript está deshabilitado en el navegador
- THEN todos los componentes renderizan con estilos y layout correctos
- AND el comportamiento interactivo se degrada pero ningún contenido queda oculto de forma irrecuperable

#### Scenario: Elemento inválido eliminado

- GIVEN las páginas migradas
- WHEN se valida el markup
- THEN no existe ningún elemento `<id>` inválido (el ícono de mail usa un elemento válido)

### Requirement: UC-2 — Dropdown consolidado

El dropdown MUST existir como un patrón único (una definición de `x-data` de Alpine, una estructura de markup) reutilizado por todos los triggers (menú de usuario, notificaciones). MUST proveer: `role="menu"` en el panel, `role="menuitem"` en los ítems, `aria-expanded` + `aria-controls` en el trigger, cierre con Escape, cierre al hacer click fuera, y foco visible en los ítems. Ningún bloque de patrón duplicado.

#### Scenario: Patrón único, semántica correcta

- GIVEN el `index.html` reconstruido
- WHEN se inspeccionan los dropdowns de usuario y notificaciones
- THEN ambos usan la misma estructura de markup
- AND el panel tiene `role="menu"` con `aria-expanded`/`aria-controls` cableados

#### Scenario: Cierre con Escape y click fuera

- GIVEN un dropdown abierto
- WHEN el usuario presiona Escape o hace click fuera del panel
- THEN el dropdown se cierra
- AND `aria-expanded` vuelve a `false`

### Requirement: UC-3 — Sidebar drawer toggleable

El sidebar MUST comportarse como drawer por debajo de `md` (con backdrop) y como panel sticky con estilos de rail en `md+`. La visibilidad MUST resolverse enteramente vía `:class` (`menuOpen ? 'flex' : 'hidden'`) — sin `x-show`, cuyo `display:none` inline pisaría las clases responsive. El menú es toggleable en TODAS las resoluciones (decisión del usuario: la hamburguesa abre/cierra también en desktop; el rail NO es permanente). El botón de toggle MUST ser solo-ícono con `aria-label` (p. ej. `aria-label="Abrir menú"`). Abrir/cerrar MUST ser toggled por Alpine sin desplazamiento de layout en `md+`.

#### Scenario: Drawer responsivo a rail

- GIVEN un viewport de menos de 40rem
- WHEN se presiona el toggle de menú
- THEN el drawer se abre sobre el contenido con backdrop oscuro
- AND en ≥ 48rem el sidebar visible renderiza como panel sticky redondeado con sombra

#### Scenario: Toggle en desktop

- GIVEN un viewport de 48rem o más
- WHEN se presiona el toggle de hamburguesa
- THEN el sidebar alterna entre visible y oculto
- AND el contenido principal no sufre desplazamiento al cerrarse

### Requirement: UC-4 — Header

El header MUST contener brand, título de página y slots de acciones. Los botones solo-ícono MUST llevar `aria-label`. El header MUST ser visible y legible en todos los breakpoints (White sobre Shadow Grey o texto sobre superficies Ivory según DT-3).

#### Scenario: Los botones solo-ícono se anuncian

- GIVEN cualquier botón solo-ícono del header
- WHEN se inspecciona o enfoca el botón
- THEN tiene un `aria-label` con texto en español
- AND aparece un anillo de foco visible al enfocar por teclado

### Requirement: UC-5 — Badge y etiqueta de estado

Los badges de conteo MUST usar colores de token con contraste AA y MUST NOT usar 9px blanco-sobre-rojo (legado 4.0:1 FAIL). Badges: Tuscan Sun con texto Shadow Grey, o White sobre Shadow Grey/Graphite. La etiqueta de estado (`.role` revivida) MUST ser un elemento semántico con una etiqueta de texto, p. ej. anuncios estilo `role="status"` donde corresponda; etiquetas de ejemplo en español: `Disponible`, `En revisión`, `Agotado`.

#### Scenario: Conformidad de contraste del badge

- GIVEN un badge de conteo
- WHEN se muestrean su texto y fondo
- THEN el par es uno de los pares válidos de DT-3
- AND el tamaño de fuente es ≥ 10px

#### Scenario: La etiqueta de estado es semántica y rotulada

- GIVEN una etiqueta de estado en la tabla de inventario
- WHEN la etiqueta renderiza
- THEN lleva una etiqueta de texto en español no vacía y una superficie con color de token

### Requirement: UC-6 — Card

Las tarjetas MUST seguir una estructura de slots (header / icon / title / subtitle / content / footer) y MUST adaptarse vía container queries (el contenido se adapta a su slot de grilla; sin `height: 320px` fija). Las tarjetas sobre superficies Dust Grey o Ivory usan texto Shadow Grey/Graphite (DT-3).

#### Scenario: La tarjeta se adapta al slot

- GIVEN tarjetas en una grilla multicolumna en `lg`
- WHEN la grilla colapsa a una columna
- THEN el contenido de la tarjeta refluye sin alturas fijas ni scroll horizontal

### Requirement: UC-7 — Botones y links

Los botones MUST soportar estados: default, hover, active, focus, disabled. Variantes: primary (White sobre Graphite/Shadow Grey), secondary (Graphite sobre Dust Grey), ghost, icon. Todas las variantes MUST mostrar el token de anillo de foco en `:focus-visible`. Los links MUST usar Shadow Grey o Graphite sobre superficies claras con subrayado visible en hover; copy de ejemplo: `Ver detalle`.

#### Scenario: El foco por teclado es visible

- GIVEN cualquier botón o link
- WHEN recibe foco por teclado
- THEN el indicador `--shadow-focus-ring` es visible

#### Scenario: El estado disabled es distinguible

- GIVEN un botón disabled
- WHEN se inspecciona
- THEN es visualmente distinto (superficie de contraste reducido) pero sigue siendo descubrible, y no es enfocable

### Requirement: UC-8 — Ítem de notificación

El ítem de la lista de notificaciones MUST seguir el patrón: ícono + texto del mensaje + link de acción. Copy de ejemplo en español: `Tu informe semanal está listo.` con acción `Ver informe`. El ícono MUST ser decorativo (`aria-hidden="true"`).

#### Scenario: Íconos decorativos ocultos al AT

- GIVEN un ítem de notificación con ícono
- WHEN se inspecciona el ícono
- THEN tiene `aria-hidden="true"` y ningún rol enfocable/semántico

### Requirement: UC-9 — Footer

El footer MUST mostrar subtexto (en español) y links, con texto sobre superficie Shadow Grey usando texto White/Ivory/Dust Grey (pares válidos de DT-3).

#### Scenario: Contraste del footer

- GIVEN el footer sobre superficie oscura
- WHEN se muestrean los colores de texto
- THEN cada par cumple AA según DT-3

### Requirement: UC-10 — Íconos SVG locales en regiones Alpine

Los íconos del sistema son SVG estáticos autohospedados en `assets/icons/` — sin JavaScript de íconos ni CDN. La mitigación `feather.replace()` quedó SUPERSEDED al eliminarse la dependencia Feather por CDN por decisión del usuario. En regiones renderizadas por Alpine (`x-for`, `x-if` o nodos renderizados tardíamente), el ícono MUST servirse como archivo SVG vinculado vía `:src` (p. ej. `<img :src="n.icon">`) o como SVG inline — nunca dependiendo de un re-scan de JavaScript posterior al render. Los íconos decorativos MUST declarar `aria-hidden="true"`. Los tamaños de ícono MUST seguir la escala documentada (sm 16 / md 18 / lg 24).

#### Scenario: Los íconos renderizan en listas de Alpine

- GIVEN una lista de notificaciones poblada vía `x-for` de Alpine
- WHEN la lista renderiza
- THEN cada ícono aparece como SVG de `assets/icons/` resuelto vía `:src` (no un tag `<i>` crudo)
- AND los íconos decorativos permanecen `aria-hidden="true"`

### Requirement: UC-11 — Primitivas tipográficas

Escala documentada: brand h1 `text-xl/2xl font-semibold tracking-tight`, título de tarjeta `text-lg font-semibold`, cuerpo `text-sm leading-relaxed`, subtexto/footer `text-xs`. Todo el texto MUST estar en español (`lang="es"`).

#### Scenario: Escala tipográfica aplicada

- GIVEN la página showcase
- WHEN se muestrean headings y texto de cuerpo
- THEN coinciden con las clases de escala documentadas

### Requirement: UC-12 — Página showcase de componentes

`components.html` MUST existir y MUST documentar cada componente con todos sus estados (default/hover/active/focus/disabled), la referencia de tokens (hex + uso), el seam de Alpine (qué atributos `x-*` son comportamiento) y labels en español. Debe servirse con el mismo `dist/main.css`.

#### Scenario: El showcase cubre todos los estados

- GIVEN `components.html` servido
- WHEN se revisa cada sección de componente
- THEN muestra todos los estados y los tokens correspondientes
- AND la página es visualmente consistente con `index.html`

## Acceptance Criteria

- Patrón único de dropdown con semántica de menú; Escape + click fuera verificados.
- Todos los controles solo-ícono llevan `aria-label` en español; íconos decorativos `aria-hidden="true"`.
- Sin badge 9px blanco-sobre-rojo; pares de badge según DT-3.
- Los íconos aparecen en regiones renderizadas por Alpine (SVGs locales de `assets/icons/` vinculados vía `:src`).
- `lang="es"` en ambas páginas; sin elemento `<id>` inválido.
