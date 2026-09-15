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
│   └── og-lab.jpg     # la preview de compartir, dibujada acá (ver abajo)
├── favicon/        # mismo set que accelerator
└── README.md
```

## 🧱 Anatomía

```
logo A13I_LAB
título + bajada   |   meme        ← dos columnas en desktop, apiladas en mobile
[ 01 ] [ 02 ] [ 03 ] [ 04 ] [ 05 ] [ 06 ]   ← grilla portfolio (hoy son 6)
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

Al tocar una ficha se abre un modal centrado (`.overlay`, colgado del `<body>`) con el arte a la izquierda y el detalle a la derecha, sobre un fondo oscurecido (`.backdrop`). Mide `min(1080px, 92vw) × min(660px, 82vh)`; en mobile pasa a pantalla completa. Se cierra con la ✕, con Escape, tocando afuera **o con el botón atrás del navegador**.

### El detalle vive en el historial

Abrir una ficha hace `pushState` con el slug del proyecto (`#copiloto-de-entrevistas`) y cerrarla hace `history.back()`, que consume esa entrada. **Esto no es un extra, es lo que hace que el modal funcione en mobile:** ahí ocupa toda la pantalla y el gesto natural para volver es el botón atrás — sin esto te sacaba del sitio en vez de cerrar la ficha.

Dos cosas que hay que respetar si se toca esto:

- **Cerrar nunca toca el DOM directo, llama a `history.back()`** y deja que el `popstate` haga el cierre visual (`cerrarUI`). Si cerrás a mano además de navegar, la entrada queda colgada y el siguiente atrás no hace nada.
- **Entrando por link directo hay que armar el par base → detalle.** Un `replaceState` solo no deja entrada a la cual volver, y la ✕ termina sacando al visitante del sitio. Por eso el `boot()` hace `replaceState` a la URL sin hash y después `pushState` al detalle.

De regalo, cada proyecto queda con URL propia y se puede compartir el link a uno solo. Un hash que no matchea ningún proyecto se ignora.

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
  link:    'https://…',                           // opcional
  // — ficha técnica, todo opcional —
  modelos: [{ n:'Opus 5', p:60 }, { n:'Sonnet 5', p:40 }],  // ⚠️ objetos {n,p}, no strings.
                                                  //    p es el porcentaje y tiene que sumar 100
  tech:    ['Computer Vision', 'MediaPipe'],      // 2 o 3 palabras clave
  ritmo:   '67 commits en 13 días',               // de git (ver la salvedad abajo)
  tokens:  '~3 M',                                // estimación
  costo:   '~US$ 2',                              // los tokens a precio de lista de la API
  prompts: '1',                                   // solo si fue uno o dos
  nota:    'Es un fork de…',                      // aclaración al pie del detalle
  art:    'copiloto',                             // clave de ART (arte generado)
  imagen: '/assets/loro.webp'                     // opcional — si está, reemplaza al arte
}
```

Reglas:
- **`pitch` es lo único que se lee en la ficha cerrada.** Una sola frase de 8 a 10 palabras que se explique sola, sin jerga: tiene que pasar el Mom Test — si tu vieja la lee y entiende de qué va, sirve. El nombre del proyecto, la familia y el detalle aparecen recién al abrirla; en la ficha redundaban seis veces y no sumaban nada.
- `items` admite `n`, `d` y `bullets`. Se pueden combinar o usar sueltos.
- `familia` agrupa productos que son parte de algo más grande sin necesidad de una ficha paraguas: los cuatro productos del Universo Loro tienen ficha propia y comparten esa etiqueta, que se ve en el detalle.
- El label del link se deriva de la URL (se le saca el protocolo y la barra final).
- `modelos` / `tech` / `ritmo` / `tokens` / `costo` arman la ficha técnica al pie del detalle: una barra con el reparto de modelos y debajo una grilla de celdas. **El orden va de lo medido a lo estimado** — tecnología y ritmo salen de los repos, tokens y equivalente se infieren de eso. La credibilidad se gana en ese orden: primero lo que se puede verificar, y recién ahí la estimación.
- `ritmo` son commits y días activos. Es el dato más legible de la ficha — cuando Axel describe un proyecto no dice "27 M de tokens", dice "estuve dos semanas". **Los días siempre salen de git; los commits también, salvo en Copiloto y Simulacro**, que comparten repo y llevan un prorrateo (está explicado en la sección de tokens). No lo presentes como "exacto de git" sin esa salvedad.
- **`modelos` es de memoria de Axel, y la página lo marca "aprox."** El patrón que él describe: casi siempre Opus 5, con Sonnet 5 alrededor del 40%, y Sonnet nunca falta cuando hay algún modelo de Claude. Simulacro y el Dashboard son solo Opus + Sonnet (sin Fable). Loro Run y Arquitectura Transformer son 100% GPT-6-Astra. El reparto exacto donde entra Fable (15%) es una suposición, no un dato que él haya dado.
- **`tokens` es una estimación y la página lo dice** (la etiqueta es "Tokens estimados"). Ver abajo de dónde sale.
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

### La preview de compartir (`og:image`)

`assets/og-lab.jpg`, 2400×1260 (el doble de los 1200×630 estándar, para que no se vea blando en pantallas retina). **Es ilustración propia, no un frame del dibujito:** un matraz explotando entre dos tubos de ensayo, en la paleta del sitio, con el wordmark, el título y el mismo subrayado de roughjs del H1. Dexter aparece nombrado en el texto, no dibujado — una cosa es un meme adentro de la página y otra es que la tarjeta que representa el sitio en WhatsApp o LinkedIn sea material de un tercero.

**Cómo se rehace.** No se edita el JPG, se regenera. La fuente está en el scratchpad de la sesión que lo creó, pero reconstruirlo es directo: un HTML de 1200×630 con el diseño, y una captura con Playwright a `deviceScaleFactor: 2`. Dos detalles que hacen falta si el entorno bloquea CDNs:

- Las tipografías salen de npm, no de Google Fonts: `npm pack geist` y `npm pack @fontsource/space-grotesk`, y se declaran con `@font-face` apuntando a los `.woff2` locales. Sin eso el render sale con la fuente del sistema y no matchea la marca.
- Para el subrayado, `npm pack roughjs@4.6.6` y se sirve el `bundled/rough.js` local.

Se exporta a JPEG progresivo con calidad 90: queda en ~180 KB, contra 1,6 MB del PNG. Ningún scraper se queja de eso.

### Subrayado y resaltado del H1

Los dos están **portados del hero de `a13i-accelerator`**, que no usa rough-notation sino **roughjs directo** (`https://unpkg.com/roughjs@4.6.6/bundled/rough.js`, pinneada). Eso es lo que da el trazo dibujado a mano; rough-notation con parámetros parecidos queda bastante peor.

Se marcan en el HTML con `data-hl="underline"` o `data-hl="highlight"` más `data-hl-color`. Hoy: subrayado en "científico loco" (`#D4612A`) y resaltado en "laboratorio como el de Dexter" (`rgba(225,94,63,0.22)`). Para mover el efecto, se cambia de span — el JS toma todos los `[data-hl]` que encuentre.

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

1. Capturas reales de los proyectos, para reemplazar el arte generado (campo `imagen`). Hoy las seis fichas usan el SVG.
2. **Confirmar el dominio final.** Hoy el `<head>` tiene `https://a13i-lab.vercel.app` hardcodeado en cuatro lugares — `og:url`, `og:image`, `twitter:image` y el `<link rel="canonical">` — porque **`og:image` tiene que ser absoluta o la preview no sale** (los scrapers de WhatsApp y Facebook no resuelven rutas relativas). Al confirmar el dominio hay que cambiar esas cuatro.

## 🔢 De dónde salen los tokens

No hay medición real del consumo de Axel en ningún lado, así que se estima desde la historia de cada repo. Los datos, medidos con `git log` sobre la historia completa (ojo: los clones con `--depth 1` mienten, hay que hacer `fetch --depth=1000`):

| Repo | Commits | Días activos | Churn (líneas) | Período |
|---|---|---|---|---|
| `loro` (Copiloto + Simulacro) | 308 | 24 | 41.399 | 10 jul → 21 ago |
| `enviaunloro` | 84 | 10 | 25.473 | 26 ago → 9 sep |
| `eday` | 67 | 13 | 28.847 | 6 jun → 27 ago |
| `juego-fitness` | 4 | 2 | 1.199 | 13 → 14 sep |
| `transformer-architecture` | 9 (2 son el fork) | 2 | 1.008 propias | 13 sep |

`loro` se reparte entre sus dos productos por los commits que tocan los archivos de cada uno. **Ojo con el solapamiento:** 106 commits tocan algo de Copiloto y 88 tocan algo de Simulacro, pero **21 tocan los dos**, así que sumarlos y restarlos del total cuenta esos 21 dos veces. El reparto correcto sale de los exclusivos:

| | Commits | Churn |
|---|---|---|
| Solo Copiloto | 85 | 9.187 |
| Solo Simulacro | 67 | 7.510 |
| Los dos | 21 | — |
| Ninguno (lib, config, estilos) | 135 | 24.702 |
| **Total repo** | **308** | **41.399** |

Cada producto se queda con sus exclusivos, los 21 compartidos se parten al medio y los 135 de infraestructura se prorratean por la razón de exclusivos (85:67 = **55,9/44,1**). Da **Copiloto 171** y **Simulacro 137**, que suman 308.

El error anterior (114 restantes, prorrateo 55/45) se cancelaba solo: daba 169 y 139, que también sumaban 308 y movían los tokens menos de lo que redondea la ficha. Igual conviene tenerlo bien, porque el número que se muestra es el que alguien va a querer reproducir.

⚠️ **Y por eso el `ritmo` de estos dos no es reproducible con un comando.** En el resto de las fichas un repo es un producto y el número sale de `git log`; acá es un prorrateo. El que muestra la página es el de la tabla de arriba. Los días activos sí son exactos, contados por fechas distintas de los commits de cada producto: Copiloto 20 días (10 jul → 11 ago), Simulacro 12 (20 jul → 11 ago).

**La fórmula:** `commits × 120k + churn × 300`. Cada commit es una ronda con el agente, y lo que más pesa en una ronda no es el código que sale sino el contexto que entra una y otra vez.

**Qué valida la única medición real que hay, y qué no.** El README de `transformer-architecture` publica el consumo **medido** de la sesión original en Codex: 83,7 M de tokens — **80,9 M de lectura de caché**, 2,4 M de input sin cachear y 397 k de salida.

Eso valida dos cosas:

- **La forma del gasto.** 96,7% es contexto re-enviado, no generación. De ahí sale la mezcla con la que se valúa en dólares (ver la sección siguiente).
- **El orden de magnitud.** Un proyecto así cuesta decenas de millones de tokens. Los ~27 M de Copiloto no son una cifra inflada: son *menos* que ese único proyecto medido.

Y **no valida la constante de la fórmula.** Una versión anterior de este documento decía "~920 tokens por línea", dividiendo 83,7 M por las 91.046 líneas del commit de importación. **Ese número estaba mal.** De esas 91.046 líneas:

| | Líneas |
|---|---|
| Three.js vendorizado (`dist/vendor/`) | 85.209 |
| JSON generado por `scripts/build_spatial_architecture.py` | 4.670 |
| **Texto efectivamente escrito** | **~5.800** (200 KB) |

La razón real es ~14.400 tokens por línea, **15× la que decía acá**. Dividir por código que nadie escribió y que ningún agente leyó entero no calibra nada.

Tampoco se puede calibrar el `× 120k` por commit contra esa medición, porque el proyecto original entró al repo como **un solo commit squasheado**: no hay commits entre los cuales dividir. El 120k es un supuesto razonado — una ronda de agente con ~100 k de contexto re-enviado — no un dato.

**Consecuencia, y esto es lo que hay que contestar si alguien la discute:** la fórmula da un **orden de magnitud, no una cifra**. La banda honesta es de ±3×. La respuesta correcta es "es una estimación derivada de la historia de git", nunca "está medido". Por eso la etiqueta de la página es *Tokens estimados* y no un número pelado.

**Sanity check contra lo que Axel recuerda:** dice que Copiloto + Simulacro fue lo más caro (≈49 M juntos, 308 commits en 6 semanas) y que el Dashboard lo hizo mucho más rápido (17 M, 67 commits). Los números dan lo mismo que su memoria.

## 💵 De dónde sale el costo en dólares

`costo` son los tokens estimados valuados a **precio de lista de la API de Anthropic**. No es lo que Axel pagó: trabajó con suscripciones, que son un abono fijo y no cobran por token. Por eso la etiqueta en la página es **"Equivalente en API"** y no "Costo": es una valuación, no un gasto, y la palabra elegida no arrastra la implicación de haberlo pagado.

La página **no aclara nada sobre suscripciones**, y es deliberado: una etiqueta que ya es correcta no necesita defensa, y agregarla le daría entidad a una pregunta que nadie se hace. El precio de suscripción además mide el plan, no el trabajo — dividirlo entre proyectos daría un número arbitrario que no informa nada.

Precios por millón de tokens:

| Modelo | Input | Output | Lectura de caché |
|---|---|---|---|
| Opus 5 | $5 | $25 | $0,50 (0,1×) |
| Sonnet 5 | $2 | $10 | $0,20 (0,1×) |
| Fable 5.1 | $10 | $50 | $0,25 (0,025×) |

**Lo que domina el número no es el precio, es la caché.** La mezcla sale de la única medición real que hay (el README de `transformer-architecture`): **96,7% lectura de caché, 2,9% input sin cachear, 0,47% salida**. Con esa mezcla, un millón de tokens totales cuesta US$ 0,75 en Opus 5, US$ 0,30 en Sonnet 5 y US$ 0,77 en Fable 5.1 — Fable termina casi igual que Opus porque su caché es 4× más barata y compensa su input más caro.

La diferencia es enorme: Copiloto son ~27 M de tokens, que **sin caché** serían US$ 135 en Opus; con la mezcla medida son ~US$ 16.

**La escritura de caché no está en la tabla, y da igual.** Anthropic cobra el *write* a 1,25× el input, y la valuación de arriba trata todo el input sin cachear como input común. Es la aproximación correcta: el write es un cargo de una sola vez sobre el 2,9% del volumen, y aunque *todo* ese 2,9% fuese write, el millón sube de US$ 0,75 a US$ 0,78 en Opus (+4,8%; +9,4% en Fable, que tiene la caché más barata). Está muy por debajo del ±3× de los tokens — no vale la pena modelarlo.

Los dos proyectos de GPT-6-Astra **no llevan costo**: es un modelo de OpenAI y no hay precio que se pueda verificar desde acá. Mejor no poner número que poner uno inventado.

**Dónde está lo flojo de la ficha, por si alguien la discute** — de lo más firme a lo más blando:

1. **Los precios** son exactos: están publicados.
2. **Los días activos** son exactos: salen de `git log` en todas las fichas.
3. **Los commits** son exactos en cuatro de las seis fichas. En Copiloto y Simulacro son un **prorrateo** de un repo compartido (ver la sección de tokens) — no se reproducen con un comando.
4. **Los tokens** son inferidos, de datos reales pero con una constante supuesta: orden de magnitud, ±3×.
5. **El dólar** hereda la incertidumbre de los tokens y no agrega nada propio: los precios son exactos y la mezcla está medida. Es exactamente tan sólido como los tokens, ni más ni menos — por eso no tiene sentido sacar uno y dejar el otro.
6. **El reparto de modelos** es el eslabón más débil: sale de la memoria de Axel, y el 15% de Fable en Copiloto y Envía un Lorito es directamente una suposición.

Si alguna vez hay que recortar la ficha, el orden de salida es de abajo hacia arriba: primero el reparto de modelos, no el dólar.

## 📚 De dónde salen las descripciones

Las de los cuatro productos del Universo Loro están escritas leyendo el código, no la landing:

| Ficha | Repo | Deploy |
|---|---|---|
| Copiloto de Entrevistas | `axellaban/loro` (ruta `/copiloto` → `/app`) | loreado.vercel.app/copiloto |
| Simulacro de Entrevistas | `axellaban/loro` (ruta `/mock` → `/simulador`) | loreado.vercel.app/mock |
| Envía un Lorito | `axellaban/Enviaunloro` | enviaunlorito.vercel.app |
| Loro Run | `axellaban/juego-fitness` | juego-fitness.vercel.app |
| Arquitectura Transformer | `axellaban/transformer-architecture` (fork) | transformer-architecture.vercel.app |
| Dashboard eCommerce Day 2026 | `axellaban/eday-argentina-2026-eCommerce-StartUp-Competition` | eday-2026-argentina-demo-day.vercel.app |

`axellaban/universo-loro` es la página índice que los agrupa. Copiloto y Simulacro son **dos productos distintos** del mismo repo, no dos nombres de lo mismo.

⚠️ **`transformer-architecture` es un fork**, y la ficha ahora lo dice con el campo `nota`. El commit `8fad2fb` importa 91.046 líneas del proyecto original de Peter Gostev; lo de Axel son 7 commits del 13 de septiembre (~1.000 líneas): traducción al español, la narración y una página de diagnóstico.

**"Astra 6" es un modelo, no un proyecto.** El nombre real del proyecto es "Arquitectura Transformer"; GPT-6-Astra es con lo que se construyó — por eso aparece en `modelos` y no en `titulo`.
