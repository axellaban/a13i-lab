# CLAUDE.md — A13I Lab

Landing del laboratorio de Axel Laban: experimentos, MVP funcionales y productos digitales, en formato portfolio.

## 🏁 Visión General

- **Rol:** vitrina, nada más. No es un funnel: no hay formularios, ni webhooks, ni CTA de venta.
- **Premisa de diseño:** **una sola pantalla, sin scroll.** Todo entra siempre: logo, título, bajada y las fichas de todos los proyectos. El detalle de cada uno está plegado y se abre al tocar la ficha.
- **Tecnología:** HTML/CSS/JS puros, un solo archivo, sin build, sin bundler, sin framework. Deploy en Vercel.
- **Contexto global:** las reglas de negocio (ICP, marca, voz, decisiones) viven en la raíz global, `../01-estrategia.md` a `../05-decisiones.md`. Leerlos antes de tocar copy.
- **Repos hermanos:** `a13i-accelerator` (DWY + lead magnets) y `a13i` / a13i-partner (DFY + conversión). De ahí sale el sistema de diseño.

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

## 🧱 Anatomía

```
logo A13I_LAB
título + bajada   |   meme        ← dos columnas en desktop, apiladas en mobile
[ 01 ] [ 02 ] [ 03 ] [ 04 ] [ 05 ]      ← grilla portfolio
```

La grilla es `repeat(auto-fit, minmax(12em, 1fr))`: reparte sola las fichas que haya, sin dejar huecos en la última fila. Con hasta 5 entran todas en una fila; a partir de ahí arma dos.

Al tocar una ficha se abre un panel que cubre **toda la grilla** (arte a la izquierda, detalle a la derecha). En mobile ese panel pasa a pantalla completa. Se cierra con la ✕, con Escape o tocando afuera.

No hay header fijo, nav, filtros ni footer. Agregar cualquiera de esas cosas rompe la premisa de la pantalla única.

## 🔒 Cómo se garantiza que entre todo sin scroll

Dos escalas, las dos manejadas por JS, las dos con búsqueda binaria:

- **`--s`** (en `:root`) escala la página entera. Todo el layout está en `em` sobre el `font-size` de `.screen`, así que bajar `--s` achica todo proporcionalmente. `fit()` lo baja hasta que `.inner` entra en el alto del viewport (piso 0.45).
- **`--so`** (en `.overlay`) hace lo mismo con el detalle abierto: `fitOverlay()` lo baja hasta que el contenido entra en el panel (piso 0.5), así nunca aparece scroll interno.

Las dos se recalculan en `resize`. El `font-size` base ya es responsive por su cuenta (`clamp(10.5px, min(3.1vw, 1.78vh), 17px)`); `--s` es el ajuste fino encima de eso.

**Consecuencia práctica:** cuantos más proyectos se sumen, más se achica todo. Con 5 o 6 fichas hay que pasar la grilla a dos filas (`grid-template-columns:repeat(3,1fr)` ya lo hace solo) y revisar que el tamaño resultante siga siendo legible.

## ➕ Cómo agregar un proyecto

Todo sale del array `PROYECTOS` en el `<script>` de `index.html`. Es la única fuente de verdad: la ficha, su numeración (`01`, `02`, …) y el panel de detalle se generan de ahí, en el orden del array.

```js
{
  titulo:  'Copiloto de Entrevistas con IA',      // obligatorio
  familia: 'Universo Loro',                       // opcional — etiqueta arriba del título
  tagline: 'La línea que se ve con la ficha cerrada.',  // obligatorio
  lead:    'Primera línea del detalle.',          // opcional
  items: [                                        // opcional — sub-productos o versiones
    { n: 'Nombre', d: 'Qué es, en una línea.' },
    { n: 'Otra versión', bullets: ['Detalle uno.', 'Detalle dos.'] }
  ],
  link:   'https://…',                            // opcional
  art:    'copiloto',                             // clave de ART (arte generado)
  imagen: '/assets/loro.webp'                     // opcional — si está, reemplaza al arte
}
```

Reglas:
- `items` admite `n`, `d` y `bullets`. Se pueden combinar o usar sueltos.
- `familia` agrupa productos que son parte de algo más grande sin necesidad de una ficha paraguas: los tres productos del Universo Loro tienen ficha propia y comparten esa etiqueta.
- El label del link se deriva de la URL (se le saca el protocolo y la barra final).
- Todo el contenido pasa por `esc()` antes de entrar al DOM. No romper eso al agregar campos.

## 🖼️ El arte de las fichas

El objeto `ART` tiene un SVG por proyecto, dibujado a mano en la paleta del sitio, que ilustra lo que hay adentro: `copiloto` (burbuja con waveform + anillo de puntaje), `lorito` (avión de papel con estela y reloj), `fitness` (HUD de juego), `eday` (radar chart + heatmap del fact checking), `astra` (bloques de transformers con líneas de atención) y `loro` (órbitas, quedó sin usar al separar los productos, sirve si vuelve una ficha paraguas). Todos usan `viewBox="0 0 200 125"` (16:10, igual que `.art`) y `preserveAspectRatio="xMidYMid meet"`: así llenan exacto la ficha en desktop y se ven enteros, sin recorte, en el thumbnail cuadrado de mobile. El fondo de `.art` y `.ov-art` es plano `#FFFDF8` para que el encuadre nunca se note.

**Para poner una captura real** en lugar del SVG: dejar el archivo en `assets/` y agregarle `imagen: '/assets/loquesea.webp'` al proyecto. El SVG queda como fallback si algún día se saca la imagen.

⚠️ Los `<style>` dentro de un SVG aplican a **todo el documento**, no solo a ese SVG. Por eso las clases del arte van con prefijo (`art-spin`, `art-pulse`). No usar nombres genéricos ahí adentro.

## 🎨 Sistema de diseño

Mismos tokens que `a13i-accelerator`. Tokens en `:root`, nunca hardcodear hex.

```css
--bg: #FEFBF2   --card: #FFFFFF  --border: #E8E3DE  --border-2: #EAD9C8
--text: #1A1410  --muted: #6B5A4A  --faint: #8C7A68
--accent: #D4612A  --accent-2: #E15E3F  --accent-deep: #A04510
--green: #1E8E5A  --amber: #B5790A
```

- **Tipografía:** `Geist` para el texto, `Geist Mono` para numeración, chrome y URLs, `Space Grotesk` 700 solo para el wordmark.
- **Wordmark:** `A13I_LAB` con `13` y `_` en naranja y `LAB` en itálica — mismo patrón que `A13I_ACCELERATOR`.
- **Fondo:** textura de ruido SVG al 1.5% + un radial cálido arriba a la izquierda.
- **Fichas:** blancas, radio `1.15em`, borde `--border-2`, hover con `translateY(-3px)` y un leve zoom del arte.

### El meme

El encabezado tiene un recuadro con un GIF del laboratorio de Dexter. La fuente sale de la constante `MEME_SRC`, arriba del script, y acepta las dos formas:

- un archivo del repo: `'/assets/dexter-lab.gif'` (el default)
- una URL directa a un gif externo: `'https://media.giphy.com/media/xxxx/giphy.gif'`

**Si no carga ninguna, el `onerror` del `<img>` muestra un dibujo propio de matraces** (inline en el HTML, clase `.meme-fb`) y la página no se ve rota. Es material de Cartoon Network / Warner: uso de meme, decisión de Axel.

### Subrayado del H1

Está **portado tal cual del hero de `a13i-accelerator`**, que no usa rough-notation sino **roughjs directo** (`https://unpkg.com/roughjs@4.6.6/bundled/rough.js`, pinneada): dos pasadas de línea (ida y vuelta) sobre cada rect del texto, con `strokeWidth 2.5`, `roughness 1.3`, `bowing 1.2`, `disableMultiStroke`. Eso es lo que le da el trazo dibujado a mano; rough-notation con parámetros parecidos queda bastante peor. Se dibuja después de `document.fonts.ready` (si no, queda desalineado) y se redibuja en cada `resize` y después de cada `fit()`. Si el CDN no carga, no pasa nada: el texto queda legible sin la decoración.

## ⚙️ Convenciones

1. Sin build tooling: HTML/CSS/JS plano.
2. Un solo archivo: estilos en `<style>` dentro del `<head>`, lógica en `<script>` al final del `<body>`.
3. Variables CSS para todo el color; medidas en `em` para que `--s` pueda escalar todo.
4. Español rioplatense, voz de Axel. Respetar el "filtro anti-IA" de `../03-marca-y-voz.md`.
5. JS vanilla. Única dependencia externa: roughjs.
6. Responsive real hasta 390px; `prefers-reduced-motion` respetado (corta las animaciones del arte y las transiciones).
7. Sin tests ni linter: seguir el estilo existente por convención.

## 📊 Analítica

Solo **GA4** (`G-FV5WCTFBC3`, la misma propiedad que accelerator y partner). Dispara `lab_proyecto_abierto` con el título al abrir una ficha. Sin Meta Pixel: acá no hay conversión que trackear ni ads apuntando al sitio.

## 🧪 Desarrollo

```bash
python3 -m http.server 8080
# http://localhost:8080/index.html
```

Push a `main` → auto-deploy en Vercel.

## 📌 Pendientes

1. Capturas reales de los tres proyectos, para reemplazar el arte generado (campo `imagen`).
2. `og:image` propio del lab (hoy reusa el genérico de accelerator).
3. El GIF: dejar `assets/dexter-lab.gif` en el repo o apuntar `MEME_SRC` a una URL externa (hoy cae en el dibujo de fallback).
4. Universo Loro: Axel mencionó cuatro productos y hay ficha para tres (Copiloto de Entrevistas, Envía un Lorito, Loro Fitness). Falta el cuarto.
5. Confirmar el dominio final.
