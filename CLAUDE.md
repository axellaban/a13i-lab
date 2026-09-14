# CLAUDE.md — A13I Lab

Landing del laboratorio de IA aplicada de A13I. Muestra los experimentos y proyectos del lab: qué se probó, en qué estado está y con qué se armó.

## 🏁 Visión General

- **Rol:** vitrina pública del banco de pruebas. No es un funnel: no captura leads, no tiene formularios, no manda webhooks. El único CTA sale hacia `a13i-accelerator`.
- **Tecnología:** HTML/CSS/JS puros, un solo archivo, sin build, sin bundler, sin framework. Deploy en Vercel.
- **Contexto global:** las reglas de negocio (ICP, marca, voz, decisiones) viven en la raíz global, `../01-estrategia.md` a `../05-decisiones.md`. Leerlos antes de tocar copy.
- **Repos hermanos:** `a13i-accelerator` (DWY + lead magnets) y `a13i` / a13i-partner (DFY + conversión). El diseño de este sitio es el mismo que el de ellos.

## 📂 Estructura

```
a13i-lab/
├── index.html      # la landing entera: estilos en <head>, lógica al final del <body>
├── vercel.json     # cleanUrls + cache + headers de seguridad
├── assets/
│   └── og-image.jpg   # copiado de accelerator — pendiente uno propio del lab
├── favicon/        # mismo set que accelerator
└── README.md
```

## ➕ Cómo agregar un proyecto

Todo el listado sale del array `PROYECTOS` en el `<script>` de `index.html`. Es la única fuente de verdad: los filtros, los contadores del hero y las tarjetas se generan a partir de ahí. Para sumar uno, se agrega un objeto al array y listo — no hay que tocar HTML ni CSS.

```js
{
  titulo:    'Agente de posventa por WhatsApp',   // obligatorio
  resumen:   'Qué hace y qué resolvió, 1 o 2 líneas.',  // obligatorio
  estado:    'produccion',    // 'produccion' | 'curso' | 'experimento' | 'archivado'
  fecha:     '2026',          // texto libre: '2026', 'Q3 2026', etc.
  metrica:   '85% sin intervención humana',  // opcional — el número que importa
  stack:     ['RAG', 'HITL', 'n8n'],         // opcional — hasta 4 rinde mejor
  link:      'https://…',     // opcional — si está, la tarjeta entera es clickeable
  linkLabel: 'Ver el repo'    // opcional — default 'Ver el detalle'
}
```

Reglas del listado:
- El orden de las tarjetas lo define `ORDEN` (producción → en curso → experimento → archivado), no el orden del array.
- Los chips de filtro se arman solos: solo aparece el estado que tenga al menos un proyecto.
- `ULTIMA_ACTUALIZACION` es una constante: actualizarla a mano al sumar proyectos.
- Los nombres de stack se escriben como se escriben (`n8n`, no `N8N`): las pills no tienen `text-transform`.
- Todo el contenido pasa por `esc()` antes de entrar al DOM. No romper eso al agregar campos.

⚠️ Los 6 items que están hoy en `PROYECTOS` son **contenido de muestra** para ver el layout poblado (salen de casos ya documentados en `a13i-accelerator/client-wins.html` y `a13i/sesion.html`). Reemplazarlos por los proyectos reales del lab.

## 🎨 Sistema de diseño

Es el mismo de `a13i-accelerator`, en versión más reducida. Tokens en `:root`, nunca hardcodear hex.

```css
--bg: #FEFBF2   --card: #FFFFFF   --border: #E8E3DE   --border-2: #EAD9C8
--text: #1A1410  --muted: #6B5A4A  --dim: #6B6560   --faint: #8C7A68
--accent: #D4612A  --accent-2: #E15E3F  --accent-deep: #A04510
--green: #1E8E5A (en producción)   --amber: #B5790A (experimento)
--dark: #1F1410  --dark-2: #2E1A0E  --dark-fg: #FFF4EB  --dark-muted: #A89F92
```

- **Tipografía:** `Geist` para todo el texto, `Geist Mono` para kickers, estados, métricas y chrome (11px, `letter-spacing: .16em`, uppercase), `Space Grotesk` 700 solo para el wordmark.
- **Wordmark:** `A13I_LAB` con `13` y `_` en naranja y `LAB` en itálica — mismo patrón que `A13I_ACCELERATOR`.
- **Header:** fijo, 60px, `rgba(254,251,242,.96)` + `backdrop-filter: blur(16px)`, borde inferior `--border`.
- **CTA:** fondo `--dark-2`, radio 9px, con el *border beam* (cónica animada sobre `@property --ba`) igual que en accelerator.
- **Tarjetas:** blancas, radio 18px, borde `--border-2`, `box-shadow: 0 2px 12px rgba(31,20,16,.05)`, hover `translateY(-3px)`.
- **Fondo:** textura de ruido SVG al 1.5% + un radial cálido arriba a la izquierda.
- **H1:** subrayado y highlight con [rough-notation](https://github.com/rough-stuff/rough-notation) `@0.5.1` (pinneada, no usar `@latest`), con los parámetros exactos del hero de accelerator: underline `#D4612A` strokeWidth 2 / 2 iteraciones, highlight `rgba(225,94,63,0.22)` padding 4. Si el CDN no carga, `initNotation()` no hace nada y el texto queda legible igual.

## ⚙️ Convenciones

1. Sin build tooling: HTML/CSS/JS plano.
2. Un solo archivo: estilos en `<style>` dentro del `<head>`, lógica en `<script>` al final del `<body>`.
3. Variables CSS para todo el color.
4. Español rioplatense, voz de Axel. Respetar el "filtro anti-IA" de `../03-marca-y-voz.md`.
5. JS vanilla. Sin frameworks, sin dependencias más allá de rough-notation.
6. Responsive real hasta 390px; `prefers-reduced-motion` respetado.
7. Sin tests ni linter: seguir el estilo existente por convención.

## 📊 Analítica

Solo **GA4** (`G-FV5WCTFBC3`, la misma propiedad que accelerator y partner). Se dispara un evento `lab_filtro` al usar los chips.

**No** tiene Meta Pixel, a diferencia de las páginas de los otros dos repos: acá no hay conversión que trackear ni ads apuntando al sitio. Si en algún momento se le mandan ads, agregarlo (pixel `1274224524679737`) y mantener Pixel y GA4 en sync como en el resto.

## 🔗 Links salientes

Están hardcodeados en `index.html` (header, footer). Si cambian los dominios, tocar ahí:
- `https://a13i-accelerator.vercel.app/` y `/book` y `/client-wins`
- `https://a13i-partner.vercel.app/`

## 🧪 Desarrollo

```bash
python3 -m http.server 8080
# http://localhost:8080/index.html
```

Push a `main` → auto-deploy en Vercel.

## 📌 Pendientes

1. Reemplazar el contenido de muestra de `PROYECTOS` por los proyectos reales del lab.
2. `og:image` propio del lab (hoy reusa el genérico de accelerator).
3. Confirmar el dominio final y actualizar los links salientes si hace falta.
