# ADR-0003: `identity-service` se implementa por capas, no hexagonal

Fecha: 2022-04-02
Estado: Aceptado (revisado en v2 — ver nota al final)

## Contexto

`booking-service` y `partner-bridge-service` se diseñaron con arquitectura hexagonal porque tienen reglas de negocio no triviales y múltiples integraciones de salida que se benefician de puertos/adaptadores intercambiables. `identity-service`, en cambio, resuelve un problema acotado: autenticar credenciales y emitir un JWT contra una única fuente de datos.

## Decisión

Implementar `identity-service` con capas clásicas (`controller` → `service` → `persistence`) en vez de hexagonal.

## Alternativas consideradas

- **Hexagonal también acá, por consistencia con el resto del sistema** — se descartó en el momento porque el servicio no tenía (ni se preveía que fuera a tener pronto) más de un adaptador de salida ni reglas de dominio complejas: el costo de puertos/interfaces no se pagaba con ningún beneficio visible.

## Consecuencias

Gana: menos código ceremonial para un servicio simple; onboarding más rápido para quien lo toque por primera vez.
Resigna (el costo real, sin maquillar): el sistema queda arquitectónicamente inconsistente — alguien que lea `booking-service` primero y `identity-service` después puede asumir, equivocadamente, que hay un criterio único. Esa inconsistencia nunca se documentó formalmente en el proyecto original; este ADR existe, en parte, para corregir esa omisión con las herramientas de las que se dispone ahora.

## Nota para v2

En la versión v2 (Java 21, ver `ROADMAP.md`), `identity-service` se migra a hexagonal, no porque lo justifique una nueva regla de negocio, sino para eliminar la inconsistencia y dejar un solo criterio arquitectónico en todo el sistema. Esa migración se documentará como un ADR de seguimiento que referencia a este.
