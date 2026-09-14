# A13I Lab

Landing del laboratorio de Axel Laban: experimentos, MVP funcionales y productos digitales, en formato portfolio.

Una sola página estática (`index.html`), sin build ni dependencias de compilación. Deploy en Vercel.

**Premisa:** entra todo en una pantalla, sin scroll. El detalle de cada proyecto está plegado y se abre al tocar la ficha.

## Correrlo local

```bash
python3 -m http.server 8080
# http://localhost:8080/index.html
```

## Agregar un proyecto

Todo sale del array `PROYECTOS`, en el `<script>` al final de `index.html`. Se agrega un objeto y listo — la ficha, la numeración y el panel de detalle se generan solos:

```js
{
  titulo:  'Universo Loro',
  tagline: 'La línea que se ve con la ficha cerrada.',
  lead:    'Primera línea del detalle.',    // opcional
  items: [                                   // opcional
    { n: 'Sub-producto', d: 'Qué es, en una línea.' },
    { n: 'Otra versión', bullets: ['Detalle uno.', 'Detalle dos.'] }
  ],
  link:   'https://…',                       // opcional
  art:    'loro',                            // arte generado (objeto ART)
  imagen: '/assets/loro.webp'                // opcional: captura real en vez del arte
}
```

Detalle completo, sistema de diseño y convenciones: [CLAUDE.md](./CLAUDE.md).
