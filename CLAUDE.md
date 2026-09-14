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
│   ├── exploding-chemicals-dexter.gif   # el meme del encabezado
│   ├── mom.png        # la madre de Dexter, recortada — la usa la nota del Mom Test
│   ├── mom.jpg        # el original que subió Axel; de acá salió el .png, la página no lo usa
│   └── og-image.jpg   # copiado de accelerator — pendiente uno propio del lab
├── favicon/        # mismo set que accelerator
└── README.md
```

## 🧱 Anatomía

```
logo A13I_LAB
título + bajada   |   meme        ← dos columnas en desktop, apiladas en mobile
[ 01 ] [ 02 ] [ 03 ] [ 04 ] [ 05 ]      ← grilla portfolio
madre de Dexter + nota del Mom Test     ← al pie, explica la regla del pitch
```

Cada ficha es **arte + una sola frase**. Nada más: ni número, ni etiqueta, ni "ver el detalle" — repetidos seis veces no agregaban información. La única pista de que se abre es un `+` que aparece al pasar por encima (`.plus`, oculto en mobile).

La grilla es `repeat(auto-fit, minmax(12em, 1fr))`: reparte sola las fichas que haya, sin dejar huecos en la última fila. Con hasta 6 entran todas en una fila.

### Breakpoints

La ficha **nunca** se convierte en una fila horizontal con un thumbnail al costado: así el arte queda diminuto y la grilla deja de leerse como portfolio. En todos los tamaños es arte arriba, frase abajo; lo único que cambia es cuántas columnas hay.

| Ancho | Columnas | Qué más cambia |
|---|---|---|
| > 820px | auto-fit (6 en una fila) | el meme a la derecha del título |
| 560–820px | 3 | el meme se achica y sube al lado del título; el `+` queda fijo y tenue, porque en touch no hay hover |
| < 560px | 2 | |
| < 400px o alto < 700px | 2 | el h1 se achica más que el resto: es el bloque más caro y, si paga él, el texto de las fichas queda legible |
| landscape, alto < 540px | 6 en una fila | se oculta el meme; con 390px de alto no entran dos filas |

Otros detalles táctiles: `-webkit-tap-highlight-color` transparente con un `:active` que hunde la ficha, la ✕ del detalle flotando sobre el arte con `min-width/height:44px` (el mínimo de un target táctil), y `env(safe-area-inset-*)` en el padding de `.screen`, del `.ov-body` y de la ✕.

Al tocar una ficha se abre un modal centrado (`.overlay`, colgado del `<body>`) con el arte a la izquierda y el detalle a la derecha, sobre un fondo oscurecido (`.backdrop`). Mide `min(1080px, 92vw) × min(660px, 82vh)`; en mobile pasa a pantalla completa. Se cierra con la ✕, con Escape o tocando afuera.

Al pie hay una sola nota (`.momtest`): la madre de Dexter recortada más una línea que explica el Mom Test, la regla que gobierna el `pitch` de cada ficha. No es un footer — no lleva links, ni copyright, ni nav.

No hay header fijo, nav, filtros ni footer. Agregar cualquiera de esas cosas rompe la premisa de la pantalla única.

## 🔒 Cómo se garantiza que entre todo sin scroll

Dos escalas, las dos manejadas por JS, las dos con búsqueda binaria:

- **`--s`** (en `:root`) escala la página entera. Todo el layout está en `em` sobre el `font-size` de `.screen`, así que bajar `--s` achica todo proporcionalmente. `fit()` lo baja hasta que `.inner` entra en el alto del viewport (piso 0.45).
- **`--so`** (en `.overlay`) hace lo mismo con el detalle abierto: `fitOverlay()` lo baja hasta que el contenido entra en el modal (piso 0.5), así nunca aparece scroll interno. Con el modal a 660px de alto hoy no necesita bajar de 1 en ningún viewport probado; si un proyecto trae mucho más texto, escala en vez de recortar.

Si `fit()` llega al piso y **aun así** no entra (una pantalla rarísima, o muchos proyectos de golpe), habilita el scroll de `.screen`. Cortar contenido siempre es peor que scrollear; la premisa de la pantalla única no vale romper el contenido para sostenerla.

Las dos se recalculan en `resize`. El `font-size` base ya es responsive por su cuenta (`clamp(10.5px, min(3.1vw, 1.78vh), 17px)`); `--s` es el ajuste fino encima de eso.

**Consecuencia práctica:** cuantos más proyectos se sumen, más se achica todo. Al pasar de 6 conviene revisar a mano cómo queda en mobile chico (375×667 ya está en 0.82) y, si hace falta, bajar el `minmax` de la grilla o acortar los `pitch`.

## ➕ Cómo agregar un proyecto

Todo sale del array `PROYECTOS` en el `<script>` de `index.html`. Es la única fuente de verdad: la ficha, su numeración (`01`, `02`, …) y el panel de detalle se generan de ahí, en el orden del array.

```js
{
  titulo:  'Copiloto de Entrevistas',             // obligatorio — se ve solo en el detalle
  pitch:   'Te sopla qué responder mientras te hacen una entrevista.',  // obligatorio
  familia: 'Universo Loro',                       // opcional — etiqueta, solo en el detalle
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
- **`pitch` es lo único que se lee en la ficha cerrada.** Una sola frase de 8 a 10 palabras que se explique sola, sin jerga: tiene que pasar el Mom Test — si tu vieja la lee y entiende de qué va, sirve. El nombre del proyecto, la familia y el detalle aparecen recién al abrirla; en la ficha redundaban seis veces y no sumaban nada.
- `items` admite `n`, `d` y `bullets`. Se pueden combinar o usar sueltos.
- `familia` agrupa productos que son parte de algo más grande sin necesidad de una ficha paraguas: los cuatro productos del Universo Loro tienen ficha propia y comparten esa etiqueta, que se ve en el detalle.
- El label del link se deriva de la URL (se le saca el protocolo y la barra final).
- Todo el contenido pasa por `esc()` antes de entrar al DOM. No romper eso al agregar campos.

## 🖼️ El arte de las fichas

El objeto `ART` tiene un SVG por proyecto, dibujado a mano en la paleta del sitio. **Todos tienen movimiento propio, y el movimiento cuenta lo que el producto hace** — no es decoración:

| Clave | Qué muestra | Qué se mueve |
|---|---|---|
| `copiloto` | la onda de voz que entra y la respuesta que sale | la onda vibra, la flecha fluye y los bullets de la respuesta aparecen de a uno |
| `simulacro` | anillo de puntaje + los cinco indicadores | el anillo se dibuja, el tilde se traza y las barras crecen: el informe armándose |
| `lorito` | el ave y el reloj | el ave recorre la estela con `offset-path`, la estela corre y la aguja gira |
| `fitness` | figura con landmarks de pose | las alas aletean, el cuerpo rebota y los carriles se vienen encima |
| `eday` | radar chart + heatmap del fact checking | el radar se deforma con el pitch y el heatmap se va pintando bloque a bloque |
| `astra` | bloques de transformers | la atención corre entre los dos stacks y los bloques laten | Todos usan `viewBox="0 0 200 125"` (16:10, igual que `.art`) y `preserveAspectRatio="xMidYMid meet"`: así llenan exacto la ficha en desktop y se ven enteros, sin recorte, en el thumbnail cuadrado de mobile. El fondo de `.art` y `.ov-art` es plano `#FFFDF8` para que el encuadre nunca se note.

**Para poner una captura real** en lugar del SVG: dejar el archivo en `assets/` y agregarle `imagen: '/assets/loquesea.webp'` al proyecto. El SVG queda como fallback si algún día se saca la imagen.

⚠️ Dos trampas al dibujar el arte:

- Los `<style>` dentro de un SVG aplican a **todo el documento**, no solo a ese SVG. Por eso cada arte lleva su propio prefijo (`cop-`, `sim-`, `lor-`, `fit-`, `ed-`, `as-`). Dos artes con la misma clase se pisan entre sí.
- Cada arte apaga sus animaciones con su propio bloque `@media(prefers-reduced-motion:reduce)` adentro del SVG. Verificado: con la preferencia activada las seis quedan quietas.
- No poner `transform-box` / `transform-origin` en un elemento que ya trae un `transform="rotate(a cx cy)"` como atributo: el origen se aplica encima del que declara el rotate y el elemento se dibuja corrido. Solo hacen falta cuando la animación misma escala o rota, y el elemento no tiene transform propio.

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
- **Fichas:** blancas, radio `1.15em`, borde `--border-2`, hover con `translateY(-3px)`, un leve zoom del arte y el `+` que aparece en la esquina.

### El meme

El encabezado tiene un recuadro con un GIF del laboratorio de Dexter. La fuente sale de la constante `MEME_SRC`, arriba del script, y acepta las dos formas:

- un archivo del repo: `'/assets/exploding-chemicals-dexter.gif'` (el que está hoy, 220×165, justo la proporción 4:3 del recuadro)
- una URL directa a un gif externo: `'https://media.giphy.com/media/xxxx/giphy.gif'`

La nota del Mom Test usa `assets/mom.png`. El original que subió Axel (`mom.jpg`) venía con el tablero de ajedrez de la transparencia horneado en los píxeles, porque JPEG no soporta alpha. El `.png` se generó sacándolo con un flood fill desde los bordes: el tablero rodea a la figura y el blanco de la camisa queda encerrado por el contorno negro, así que no se toca. Si alguna vez hay que rehacerlo, ese es el método — un color-key directo le haría agujeros a la ropa.

**Si no carga ninguna, el `onerror` del `<img>` muestra un dibujo propio de matraces** (inline en el HTML, clase `.meme-fb`) y la página no se ve rota. Es material de Cartoon Network / Warner: uso de meme, decisión de Axel.

### Subrayado y resaltado del H1

Los dos están **portados del hero de `a13i-accelerator`**, que no usa rough-notation sino **roughjs directo** (`https://unpkg.com/roughjs@4.6.6/bundled/rough.js`, pinneada). Eso es lo que da el trazo dibujado a mano; rough-notation con parámetros parecidos queda bastante peor.

Se marcan en el HTML con `data-hl="underline"` o `data-hl="highlight"` más `data-hl-color`. Hoy: subrayado en "científico loco" (`#D4612A`) y resaltado en "el de Dexter" (`rgba(225,94,63,0.22)`). Para mover el efecto, se cambia de span — el JS toma todos los `[data-hl]` que encuentre.

- **Subrayado:** dos pasadas de línea, ida y vuelta.
- **Resaltado:** una sola línea del grosor del renglón (`r.height * 0.88`), `roughness 2.4`, dibujada en un SVG insertado **antes** del span para que quede detrás del texto.

Dos ajustes propios sobre la versión de accelerator, los dos por una razón concreta:

- El subrayado se dibuja a `r.height * 0.87`, en la zona del descendente, **no al pie de la caja del span**. Con `line-height` ajustado esa caja ya invade la línea de abajo — medido en mobile: el pie del span quedaba 2,4px por debajo del techo del renglón siguiente — y el trazo terminaba encima del texto.
- El grosor y el temblor acompañan al `font-size` (`strokeWidth` entre 1.5 y 2.5, `bowing` 0.7 abajo de 28px). Los valores fijos de accelerator son para un título de 50px; a los 23px de mobile quedan gruesos y se van de línea.

Se dibuja después de `document.fonts.ready` (si no, queda desalineado) y se redibuja en cada `resize` y después de cada `fit()`. Si el CDN no carga, no pasa nada: el texto queda legible sin la decoración.

**Para verlo en local:** unpkg puede estar bloqueado según el entorno. `npm pack roughjs@4.6.6`, sacar `package/bundled/rough.js` y apuntar el `<script>` ahí.

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
3. Confirmar el dominio final.

## 📚 De dónde salen las descripciones

Las de los cuatro productos del Universo Loro están escritas leyendo el código, no la landing:

| Ficha | Repo | Deploy |
|---|---|---|
| Copiloto de Entrevistas | `axellaban/loro` (ruta `/copiloto` → `/app`) | loreado.vercel.app/copiloto |
| Simulacro de Entrevistas | `axellaban/loro` (ruta `/mock` → `/simulador`) | loreado.vercel.app/mock |
| Envía un Lorito | `axellaban/Enviaunloro` | enviaunlorito.vercel.app |
| Loro Run | `axellaban/juego-fitness` | juego-fitness.vercel.app |

`axellaban/universo-loro` es la página índice que los agrupa. Copiloto y Simulacro son **dos productos distintos** del mismo repo, no dos nombres de lo mismo.
