# Plan de trabajo — NimbusDesk v1

Objetivo de v1: reproducir, con un caso de uso inventado, una arquitectura de microservicios + hexagonal fiel a una implementada en producción en 2022 — mismas versiones, mismas decisiones, mismas limitaciones. Nada de mensajería, Config Server, Resilience4j ni tracing en esta fase: eso es v2.

Convenciones para todos los repos del sistema (no solo este):
- Rama `main`: snapshots estables. Rama `develop`: rama de trabajo por defecto. `feature/<nombre>` sobre `develop`, integradas vía PR.
- Commits en formato [Conventional Commits](https://www.conventionalcommits.org/) (`feat:`, `fix:`, `docs:`, `chore:`, `refactor:`, `test:`).
- Stack base v1: Java 17, Spring Boot 3.x, Spring Cloud 2023.0.1, Maven, Lombok + MapStruct, MySQL como base principal.

## Fase 0 — `nimbusdesk-infra` (este repo)
- [x] README con visión general y mapa de repos
- [x] ROADMAP.md (este archivo)
- [x] CONTRIBUTING.md con convención de ramas/commits
- [x] Diagramas de arquitectura (contexto + contenedores) en `docs/architecture/`
- [x] ADRs iniciales en `adr/`
- [x] `docker-compose.yml` base (MySQL + SQL Server) con bloques comentados para cada servicio

## Fase 1 — `nimbusdesk-discovery-server`
- [ ] Proyecto Spring Boot 3.x / Java 17, dependencia `spring-cloud-starter-netflix-eureka-server`
- [ ] `@EnableEurekaServer`, `application.yml` en modo standalone (sin auto-registro de sí mismo)
- [ ] Dockerfile + healthcheck
- [ ] Habilitar su bloque en `docker-compose.yml` de infra

## Fase 2 — `nimbusdesk-api-gateway`
- [ ] Spring Cloud Gateway (WebFlux) + `spring-cloud-starter-netflix-eureka-client`
- [ ] Filtro de autenticación JWT (`jjwt`) + `RouteValidator` con lista de rutas públicas/privadas
- [ ] Manejo centralizado de errores (`BadRequestException`, `UnauthorizedException`, `InternalServerException`)
- [ ] Rutas declarativas hacia `identity-service`, `booking-service`, `partner-bridge-service`
- [ ] Habilitar su bloque en `docker-compose.yml`

## Fase 3 — `nimbusdesk-identity-service`
- [ ] Arquitectura **por capas** (controller → service → persistence) — a propósito, ver [ADR-0003](adr/0003-identity-service-arquitectura-por-capas.md)
- [ ] Registro/login, emisión de JWT (`jjwt-api`/`impl`/`jackson`), hash de contraseñas
- [ ] Persistencia en MySQL (JPA)
- [ ] Registro en Eureka
- [ ] Habilitar su bloque en `docker-compose.yml`

## Fase 4 — `nimbusdesk-booking-service`
- [ ] Arquitectura **hexagonal**: `domain/{model,api,spi,usecase}`, `application/{dto,mapper,handler}`, `infrastructure/{input/rest,output/persistence,config}`
- [ ] Casos de uso: crear reserva, consultar disponibilidad, cancelar reserva, calcular precio
- [ ] Adaptador de salida JPA → MySQL
- [ ] Cliente Feign hacia `partner-bridge-service`
- [ ] Registro en Eureka
- [ ] Habilitar su bloque en `docker-compose.yml`

## Fase 5 — `nimbusdesk-partner-bridge-service`
- [ ] Arquitectura **hexagonal** con un puerto de salida (`domain/spi`) y **dos adaptadores intercambiables**: uno contra MySQL, otro contra SQL Server, seleccionables por perfil de Spring (`application-mysql.yml` / `application-mssql.yml`)
- [ ] Caso de uso: sincronizar disponibilidad de un "partner" externo
- [ ] Registro en Eureka
- [ ] Habilitar su bloque en `docker-compose.yml`

## Fase 6 — Integración end-to-end
- [ ] `docker compose up` levanta los 5 servicios backend + MySQL + SQL Server sin intervención manual
- [ ] Flujo completo probado manualmente: login en `identity-service` → JWT → reserva en `booking-service` vía Gateway → verificación cruzada en `partner-bridge-service`
- [ ] Colección Postman/Insomnia exportada a `docs/api/`

## Fase 7 — `nimbusdesk-web`
- [ ] Angular, consumo de la API únicamente a través del Gateway
- [ ] Login, listado de disponibilidad, creación/cancelación de reservas
- [ ] Servido con Nginx en Docker, habilitado en `docker-compose.yml`

## Fase 8 — v2 (repos nuevos, fuera de este set)
- [ ] Migración a Java 21
- [ ] `nimbusdesk-notification-service`: eventos de dominio vía mensajería (broker a definir)
- [ ] Resilience4j en llamadas síncronas entre servicios
- [ ] Config Server centralizado
- [ ] Tracing distribuido (OpenTelemetry)
- [ ] CI/CD (GitHub Actions) + tests automatizados
- [ ] `nimbusdesk-identity-service` migrado a hexagonal, documentando el cambio como ADR de seguimiento del ADR-0003
