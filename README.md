# A13I Lab

Landing del laboratorio de Axel Laban: experimentos, MVP funcionales y productos digitales.

Una sola página estática (`index.html`), sin build ni dependencias de compilación. Deploy en Vercel.

## Correrlo local

```bash
python3 -m http.server 8080
# http://localhost:8080/index.html
```

## Agregar un proyecto

Todo el listado sale del array `PROYECTOS`, en el `<script>` al final de `index.html`. Se agrega un objeto y listo — la entrada y su numeración se generan solas:

```js
{
  titulo: 'Universo Loro',
  lead:   'Una o dos líneas de qué es.',   // opcional
  items: [                                  // opcional
    { n: 'Sub-producto', d: 'Qué es, en una línea.' },
    { n: 'Otra versión', bullets: ['Detalle uno.', 'Detalle dos.'] }
  ],
  link: 'https://…'                         // opcional: hace clickeable la entrada
}
```

Detalle completo de campos, sistema de diseño y convenciones: [CLAUDE.md](./CLAUDE.md).
