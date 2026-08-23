# dashboard-grid

Panel administrativo estático que funciona como sistema de diseño **Monochrome Harmony**: una paleta monocromática con acento cálido, tokens centralizados en Tailwind CSS v4 y componentes portables sin dependencias de build pesadas. Sirve de base para futuras migraciones a frameworks (Next.js, Vue) manteniendo el lenguaje visual intacto.

## Inicio rápido

```bash
npm install        # instala tailwindcss + @tailwindcss/cli
npm run build      # genera dist/main.css (~20KB minificado)
npm run watch      # modo desarrollo con rebuild automático
```

Serví la carpeta con cualquier servidor estático (p. ej. Live Server en `:5500`) y abrí:

| Archivo | Qué es |
|---------|--------|
| `index.html` | Dashboard demostrativo: métricas, inventario, clientes, pedidos |
| `components.html` | Showcase del sistema: paleta, tipografía, botones, badges, tarjetas |

## Stack

| Capa | Elección | Por qué |
|------|----------|---------|
| Estilos | Tailwind CSS v4 (`@theme`) | Tokens nativos del framework, sin archivo de config |
| Comportamiento | Alpine.js 3 (CDN, `defer`) | Interactividad declarativa sin bundler |
| Iconos | SVGs locales (`assets/icons/`) | Sin JS de iconos ni red extra; stroke pre-coloreado por contexto |
| Build | `@tailwindcss/cli` | Un solo binario, output minificado a `dist/` |

## Paleta Monochrome Harmony

Seis colores crudos definidos como tokens en `css/main.css` (`@theme`) y expuestos vía aliases semánticos:

| Token crudo | Hex | Alias semántico | Rol |
|-------------|-----|-----------------|-----|
| Ivory | `#e8eddf` | `--color-surface` | Fondo principal de página |
| Dust Grey | `#cfdbd5` | `--color-surface-raised` | Superficies secundarias |
| Tuscan Sun | `#f5cb5c` | `--color-accent` | Acento: badges, contadores, destacados |
| Shadow Grey | `#242423` | `--color-ink` / fondos oscuros | Texto principal y superficies más oscuras |
| Graphite | `#333533` | `--color-ink-soft` | Texto suave, cabecera, hovers oscuros |
| White | `#ffffff` | `--color-on-dark` | Tarjetas y texto sobre fondos oscuros |

### Pares de contraste verificados (WCAG 2.1 AA)

| Par | Ratio | Estado |
|-----|-------|--------|
| Shadow Grey sobre Ivory | 13.03:1 | ✅ texto principal |
| Shadow Grey sobre Dust Grey | 10.91:1 | ✅ |
| Shadow Grey sobre Tuscan Sun | 10.05:1 | ✅ único par válido sobre acento |
| White sobre Shadow Grey | 15.54:1 | ✅ footer/header |
| White sobre Graphite | 12.37:1 | ✅ |
| **White sobre Tuscan Sun** | **1.55:1** | ❌ prohibido |

**Regla de oro**: texto oscuro sobre superficies claras, texto claro sobre oscuras, y el dorado solo admite texto oscuro.

## Tono del diseño

- **Monocromático y calmo**: la página vive en grises verdosos desaturados; el color llega solo por un acento.
- **Acento racionado**: Tuscan Sun aparece en pequeños momentos (badge de notificaciones, estados positivos) nunca en superficies grandes ni con texto blanco.
- **Enmarcado oscuro**: cabecera y pie en tonos profundos enmarcan contenido claro y dan jerarquía sin bordes.
- **Superficies en capas**: Ivory (página) → White/Dust Grey (tarjetas) → sombra sutil `shadow-card`, sin bordes duros entre capas.
- **Iconografía lineal**: trazos finos estilo Feather, coloreados por contexto (blanco sobre oscuro, grafito sobre claro).
- **Accesibilidad primero**: `aria-label` en botones solo-ícono, `aria-hidden` en decoración, roles de menú ARIA, anillo de foco visible de 3px en toda interacción teclado.

## Estructura

```
├── index.html          # dashboard (solo utilitarios Tailwind)
├── components.html     # showcase del sistema de diseño
├── css/main.css        # fuente Tailwind v4: @theme + @source (NO sirve al browser directo)
├── dist/main.css       # output compilado — lo que referencian los HTML
├── assets/icons/       # 31 SVGs Feather pre-coloreados (-white / -graphite)
├── openspec/           # specs del sistema (design-tokens, ui-components, static-build)
└── package.json        # scripts build/watch
```

## Convenciones

- **Solo utilitarios**: cero clases BEM o componentes CSS custom; todo estilizado con utilidades Tailwind.
- **Cero hex fuera de `css/main.css`**: los HTML usan únicamente aliases de token (`bg-surface`, `text-ink`, `bg-accent`...).
- **Alpine factory**: los dropdowns se registran con `Alpine.data('dropdown')` bajo `alpine:init`; cada instancia recibe estado independiente.
- **Commits en español**, formato convencional (`feat:`, `fix:`, `chore:`).

## Más documentación

Los requisitos formales del sistema (DT-1..6, UC-1..12, SB-1..6) viven en [`openspec/specs/`](openspec/specs/) y el historial del cambio fundacional en [`openspec/changes/archive/2026-08-23-tailwind-v4-design-system/`](openspec/changes/archive/2026-08-23-tailwind-v4-design-system/).
