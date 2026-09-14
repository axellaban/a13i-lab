# Instrucciones para Agentes (AGENTS.md) — A13I Lab

Toda herramienta de agente (Claude Code, Antigravity, Cursor, etc.) debe leer las instrucciones de desarrollo de este repositorio antes de tocar nada.

## 🏁 Inicio Rápido

La documentación técnica completa está consolidada en:
- [CLAUDE.md](./CLAUDE.md) — descripción del proyecto, cómo agregar proyectos al listado, sistema de diseño, convenciones y pendientes.

*Nota:* antes de proponer cambios de copy o de lógica comercial, leer el contexto global de la raíz (`../01-estrategia.md` a `../05-decisiones.md`) y el AGENTS.md de la raíz.

## Regla corta

El listado de proyectos sale del array `PROYECTOS` dentro de `index.html`. Para sumar o editar un proyecto se toca ese array y nada más. La página tiene que entrar siempre en una sola pantalla, sin scroll: logo, título, bajada y las fichas de los proyectos. No agregarle nav, filtros, CTA ni footer.
