# ADR-0004: Puerto único con dos adaptadores de persistencia en `partner-bridge-service`

Fecha: 2022-05-20
Estado: Aceptado

## Contexto

`partner-bridge-service` sincroniza disponibilidad contra un sistema externo. Durante su desarrollo convivieron dos escenarios de despliegue: uno con el dato replicado en MySQL (entorno propio) y otro donde el dato de referencia vivía en una base SQL Server ya existente (del lado del partner/legado). Se necesitaba que el dominio no dependiera de cuál de las dos estuviera activa en cada entorno.

## Decisión

Definir un único puerto de salida (`domain/spi`) para la operación de sincronización, con dos adaptadores concretos en `infrastructure/output`: uno contra MySQL y otro contra SQL Server, seleccionados por perfil de Spring (`application-mysql.yml` / `application-mssql.yml`). El caso de uso en `domain/useCase` no conoce cuál de los dos está activo.

## Alternativas consideradas

- **Dos servicios separados, uno por motor de base de datos** — descartado por duplicar toda la lógica de negocio (que es idéntica) solo para variar el mecanismo de persistencia.
- **Un único adaptador con lógica condicional interna (`if mysql ... else ...`)** — descartado porque mezclar el detalle de infraestructura dentro de un mismo adaptador rompe la idea de que un adaptador conoce una sola tecnología, y complica las pruebas de cada uno por separado.

## Consecuencias

Gana: es la demostración más directa del valor de hexagonal en este sistema — cambiar de motor de base de datos es una cuestión de configuración (perfil activo), no de tocar el dominio.
Resigna: mantener dos adaptadores implica duplicar el esfuerzo de pruebas de integración (una suite contra MySQL, otra contra SQL Server) y a futuro, dos migraciones de esquema en paralelo si el modelo cambia.
