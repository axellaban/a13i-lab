# A13I Lab

Landing del laboratorio de IA aplicada de A13I: qué experimentos y proyectos se están probando, en qué estado está cada uno y con qué se armó.

Sitio estático de un solo archivo (`index.html`), sin build ni dependencias de compilación. Deploy en Vercel.

## Correrlo local

```bash
python3 -m http.server 8080
# http://localhost:8080/index.html
```

## Agregar un proyecto

Todo el listado sale del array `PROYECTOS`, en el `<script>` al final de `index.html`. Se agrega un objeto y listo — las tarjetas, los filtros y los contadores del hero se generan solos:

```js
{
  titulo:  'Agente de posventa por WhatsApp',
  resumen: 'Qué hace y qué resolvió, en una o dos líneas.',
  estado:  'produccion',                      // produccion | curso | experimento | archivado
  fecha:   '2026',
  metrica: '85% sin intervención humana',     // opcional
  stack:   ['RAG', 'HITL', 'n8n'],            // opcional
  link:    'https://…'                        // opcional: hace clickeable la tarjeta
}
```

Detalle completo de campos, sistema de diseño y convenciones: [CLAUDE.md](./CLAUDE.md).

## Estado

Los proyectos que están hoy en el array son **contenido de muestra** para ver el layout poblado. Falta reemplazarlos por los reales.
