# Convención de trabajo — sistema NimbusDesk

Aplica a todos los repos `nimbusdesk-*`.

## Ramas

- `main` — snapshots estables. Solo recibe merges desde `develop` en hitos cerrados (ej. "fin de Fase 3").
- `develop` — rama de trabajo por defecto. Todo el desarrollo día a día ocurre acá (directo o vía `feature/*`).
- `feature/<descripcion-corta>` — para cambios puntuales que se quieran revisar en un PR antes de integrarse a `develop` (opcional en repos pequeños, recomendado en los de negocio).

## Commits

[Conventional Commits](https://www.conventionalcommits.org/):

```
<tipo>(<alcance opcional>): <descripción en imperativo>
```

Tipos usados en este sistema: `feat`, `fix`, `docs`, `chore`, `refactor`, `test`, `build`, `ci`.

Ejemplos:
- `docs(infra): agregar diagrama de contenedores y ADRs iniciales`
- `feat(booking): implementar caso de uso crear reserva`
- `fix(gateway): corregir validación de rutas públicas`

## Pull Requests

- Título en el mismo formato que los commits.
- Descripción breve: qué cambia y por qué (no "qué archivos toca", eso ya lo muestra el diff).
- Un PR = un cambio coherente. Evitar mezclar, por ejemplo, un ADR nuevo con una feature de negocio.

## ADRs

Toda decisión de arquitectura no trivial (elegir un patrón, un mecanismo de comunicación, una base de datos) se documenta en `adr/NNNN-titulo.md` usando `adr/0000-template.md` como base. Un ADR se agrega, no se reescribe: si una decisión cambia, se crea un ADR nuevo que referencia al anterior.
