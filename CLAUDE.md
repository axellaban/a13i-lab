# CLAUDE.md — A13I Lab

Landing del laboratorio de Axel Laban: experimentos, MVP funcionales y productos digitales.

## 🏁 Visión General

- **Rol:** vitrina, nada más. No es un funnel: no hay formularios, ni webhooks, ni CTA de venta. Una sola página con el logo, el título y el listado de proyectos.
- **Tecnología:** HTML/CSS/JS puros, un solo archivo, sin build, sin bundler, sin framework. Deploy en Vercel.
- **Contexto global:** las reglas de negocio (ICP, marca, voz, decisiones) viven en la raíz global, `../01-estrategia.md` a `../05-decisiones.md`. Leerlos antes de tocar copy.
- **Repos hermanos:** `a13i-accelerator` (DWY + lead magnets) y `a13i` / a13i-partner (DFY + conversión). De ahí sale el sistema de diseño, acá está en su versión más despojada.

## 📂 Estructura

```
a13i-lab/
├── index.html      # la página entera: estilos en <head>, lógica al final del <body>
├── vercel.json     # cleanUrls + cache + headers de seguridad
├── assets/
│   └── og-image.jpg   # copiado de accelerator — pendiente uno propio del lab
├── favicon/        # mismo set que accelerator
└── README.md
```

## 🧱 Anatomía de la página

De arriba a abajo, y no hay nada más: **logo** (`A13I_LAB`) → **h1** → **subtítulo** → **listado de proyectos**. Sin header fijo, sin nav, sin filtros, sin footer. Cualquier cosa que se agregue rompe la premisa: mantenerla vacía.

## ➕ Cómo agregar un proyecto

Todo el listado sale del array `PROYECTOS` en el `<script>` de `index.html`. Es la única fuente de verdad: las entradas y su numeración (`01`, `02`, …) se generan a partir de ahí, en el orden del array. Para sumar uno, se agrega un objeto y listo — no hay que tocar HTML ni CSS.

```js
{
  titulo: 'Universo Loro',              // obligatorio
  lead:   'Una o dos líneas de qué es.',  // opcional
  items: [                               // opcional — sub-productos o versiones
    { n: 'Nombre del sub-producto', d: 'Qué es, en una línea.' },
    { n: 'Otra versión', bullets: ['Detalle uno.', 'Detalle dos.'] }
  ],
  link: 'https://…'                      // opcional — hace clickeable la entrada entera
}
```

Reglas:
- `items` admite `n` (nombre), `d` (descripción) y `bullets` (lista). Se pueden combinar o usar sueltos.
- El label del link se deriva solo de la URL (se le saca el protocolo y la barra final). No hace falta escribirlo.
- Todo el contenido pasa por `esc()` antes de entrar al DOM. No romper eso al agregar campos.
- Un proyecto sin `link` se renderiza como `<div>` en vez de `<a>`, sin la línea de URL.

## 🎨 Sistema de diseño

Mismos tokens que `a13i-accelerator`, en versión reducida. Tokens en `:root`, nunca hardcodear hex.

```css
--bg: #FEFBF2   --border: #E8E3DE   --border-2: #EAD9C8
--text: #1A1410  --muted: #6B5A4A   --faint: #8C7A68
--accent: #D4612A  --accent-2: #E15E3F  --accent-deep: #A04510
```

- **Tipografía:** `Geist` para el texto, `Geist Mono` para la numeración y las URLs, `Space Grotesk` 700 solo para el wordmark.
- **Wordmark:** `A13I_LAB` con `13` y `_` en naranja y `LAB` en itálica — mismo patrón que `A13I_ACCELERATOR`.
- **Fondo:** textura de ruido SVG al 1.5% + un radial cálido arriba a la izquierda.
- **Listado:** sin tarjetas ni cajas. Entradas separadas por líneas de 1px, con la numeración en un riel a la izquierda que colapsa arriba en mobile.
- **H1:** subrayado de "curiosidad" con [rough-notation](https://github.com/rough-stuff/rough-notation) `@0.5.1` (pinneada, no usar `@latest`), con los parámetros exactos del hero de accelerator: `#D4612A`, strokeWidth 2, 2 iteraciones. Si el CDN no carga, el listener no hace nada y el texto queda legible igual.

## ⚙️ Convenciones

1. Sin build tooling: HTML/CSS/JS plano.
2. Un solo archivo: estilos en `<style>` dentro del `<head>`, lógica en `<script>` al final del `<body>`.
3. Variables CSS para todo el color.
4. Español rioplatense, voz de Axel. Respetar el "filtro anti-IA" de `../03-marca-y-voz.md`.
5. JS vanilla. Sin frameworks, sin dependencias más allá de rough-notation.
6. Responsive real hasta 390px; `prefers-reduced-motion` respetado.
7. Sin tests ni linter: seguir el estilo existente por convención.

## 📊 Analítica

Solo **GA4** (`G-FV5WCTFBC3`, la misma propiedad que accelerator y partner). Sin Meta Pixel: acá no hay conversión que trackear ni ads apuntando al sitio.

## 🧪 Desarrollo

```bash
python3 -m http.server 8080
# http://localhost:8080/index.html
```

Push a `main` → auto-deploy en Vercel.

## 📌 Pendientes

1. `og:image` propio del lab (hoy reusa el genérico de accelerator).
2. Universo Loro: Axel mencionó cuatro productos y quedaron listados tres (Copiloto de Entrevistas, Envía un Lorito, Loro Fitness). Falta confirmar el cuarto.
3. Confirmar el dominio final.
