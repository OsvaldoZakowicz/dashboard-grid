## ADDED Requirements

### Requirement: DT-1 — Fuente única de verdad de tokens

Todos los tokens de diseño MUST declararse dentro del bloque `@theme` en `css/main.css`. Los literales de color hardcodeados MUST NOT aparecer fuera de `@theme` en HTML ni en CSS fuente; las superficies MUST referenciar los tokens mediante utilidades de Tailwind o custom properties de `@theme`.

#### Scenario: Todos los colores resuelven desde tokens

- GIVEN el `index.html` y el `components.html` renderizados
- WHEN se escanean todos los atributos `class` y estilos inline en busca de valores hex crudos
- THEN no existe ningún literal `#hex` fuera de `css/main.css`
- AND cada superficie visible mapea a un token `--color-*`

#### Scenario: Agregar un token exige entrada en el tema

- GIVEN un nuevo color de superficie necesario en tiempo de diseño
- WHEN el color se incorpora al sistema
- THEN se declara un nuevo token `--color-*` en `@theme` y se referencia desde el markup

### Requirement: DT-2 — Paleta Monochrome Harmony

La paleta MUST definir exactamente estos tokens en `@theme`:

| Token | Hex | Rol |
|---|---|---|
| `--color-ivory` | `#e8eddf` | Superficie más clara (fondo de página) |
| `--color-dust-grey` | `#cfdbd5` | Superficie secundaria (tarjetas, bandas sutiles) |
| `--color-tuscan-sun` | `#f5cb5c` | Acento (badges, destacados) — solo texto oscuro |
| `--color-shadow-grey` | `#242423` | Superficie más oscura / texto |
| `--color-graphite` | `#333533` | Superficie oscura secundaria / texto |
| `--color-white` | `#ffffff` | Tarjetas sobre superficies oscuras, texto de contraste |

#### Scenario: Los nombres de tokens resuelven al hex verificado

- GIVEN el `dist/main.css` construido
- WHEN se consulta el valor de cada token `--color-*`
- THEN el valor resuelto coincide con el hex indicado arriba

#### Scenario: Token desconocido es rechazado

- GIVEN una clase que referencia un token de color no declarado
- WHEN corre el build
- THEN el build emite un error o la clase queda indefinida (sin color de fallback silencioso)

### Requirement: DT-3 — Reglas de emparejamiento WCAG

Los pares texto/fondo MUST cumplir WCAG 2.1 AA: ≥ 4.5:1 para texto normal, ≥ 3.0:1 para texto grande (≥ 18pt o 14pt bold). Los siguientes emparejamientos son REQUIRED (válidos):

- Texto sobre Tuscan Sun MUST ser Shadow Grey (10.05:1) o Graphite (8.00:1) — texto oscuro sobre dorado.
- White MUST NOT usarse como texto sobre Tuscan Sun (1.55:1 FAIL).
- Tuscan Sun MUST NOT usarse como fondo para otros colores de texto (Ivory sobre Tuscan Sun 1.30:1 FAIL).
- Shadow Grey y Graphite MUST NOT emparejarse entre sí como texto-sobre-fondo (1.26:1 FAIL).
- Shadow Grey sobre Ivory (13.03:1), Graphite sobre Ivory (10.37:1), Shadow Grey sobre Dust Grey (10.91:1), Graphite sobre Dust Grey (8.69:1), White sobre Shadow Grey (15.54:1), White sobre Graphite (12.37:1) MUST usarse para texto donde esas superficies coincidan.
- Los emparejamientos no listados en esta matriz MUST verificarse antes de usarse.

#### Scenario: Texto oscuro sobre acento dorado

- GIVEN un badge o destacado Tuscan Sun
- WHEN se coloca texto sobre él
- THEN el color del texto es Shadow Grey o Graphite
- AND el par computa ≥ 4.5:1

#### Scenario: Texto blanco sobre dorado es rechazado

- GIVEN una revisión de diseño de cualquier superficie Tuscan Sun
- WHEN aparece texto blanco o ivory sobre ella
- THEN el par computa 1.55:1 y se rechaza como no conforme

### Requirement: DT-4 — Tokens de tipografía

El sistema MUST usar el stack de fuentes del sistema por defecto de Tailwind (`ui-sans-serif, system-ui, ...`) — sin webfonts. El `letter-spacing: 1px` global MUST eliminarse; los headings usan `tracking-tight`, el cuerpo `tracking-normal`. Escala: `text-xs` (0.75rem) subtexto/footer, `text-sm` cuerpo, `text-lg` títulos de tarjeta, `text-xl/2xl` headings de marca; `leading-snug` headings, `leading-relaxed` cuerpo.

#### Scenario: Sin letter-spacing global

- GIVEN la hoja de estilos construida
- WHEN se busca `letter-spacing: 1px` en el selector universal
- THEN está ausente
- AND el espaciado se aplica por utilidad (`tracking-*`)

### Requirement: DT-5 — Tokens de espaciado, radios, sombra y breakpoints

El espaciado MUST usar la escala por defecto de `0.25rem` (0.5rem legado → `gap-2`, 1rem → `p-4`, 1.5rem → `p-6`). Radios: botones `rounded-sm/md`, tarjetas `rounded-lg`, paneles de dropdown `rounded-xl`. Los tokens de elevación MUST definirse: `--shadow-card` (reposo), `--shadow-popover` (dropdown/sidebar), `--shadow-focus-ring` (indicador de foco de 3px Graphite sobre Ivory). Breakpoints: 480px legado → `sm` (40rem), 768px → `md` (48rem), `lg` (64rem).

#### Scenario: Los tokens de elevación existen en la salida

- GIVEN la hoja de estilos construida
- WHEN se resuelven `shadow-card`, `shadow-popover`, `shadow-focus-ring`
- THEN cada uno resuelve a un valor de sombra definido

#### Scenario: El anillo de foco cumple visibilidad

- GIVEN un elemento enfocable en cualquier estado
- WHEN recibe foco por teclado
- THEN el indicador de foco visible es el token `--shadow-focus-ring` (≥ 3px) y contrasta con la superficie adyacente

### Requirement: DT-6 — Variante dark fijada

La custom variant de dark mode MUST fijarse a `data-theme` (`@custom-variant dark (&:where([data-theme=dark], [data-theme=dark] *));`) e incluirse en la fuente aunque dark mode esté fuera del alcance de este cambio. Ninguna utilidad `dark:` MAY depender de `prefers-color-scheme`.

#### Scenario: Variante dark basada en data-theme

- GIVEN la fuente procesada
- WHEN se lee la definición de la custom variant
- THEN coincide con el patrón de selector `[data-theme=dark]`
