# ADR-0001: Service discovery con Netflix Eureka

Fecha: 2022-03-01 (fecha de referencia, alineada al proyecto original que inspira este portafolio)
Estado: Aceptado

## Contexto

Con más de un servicio de negocio registrándose y comunicándose entre sí, hacía falta que el Gateway y los propios servicios pudieran ubicarse dinámicamente sin hardcodear hosts/puertos, incluso al escalar instancias.

## Decisión

Usar Netflix Eureka (`spring-cloud-starter-netflix-eureka-server` / `-client`) como registro de servicios. Cada servicio de negocio se registra al arrancar; el Gateway resuelve rutas contra el registro en vez de URLs fijas.

## Alternativas consideradas

- **DNS interno de un orquestador (Kubernetes Service Discovery)** — descartado porque el entorno de despliegue de referencia no usaba Kubernetes, sino contenedores Docker sueltos sobre redes definidas manualmente.
- **Consul** — funcionalmente equivalente, pero Eureka tiene integración de primera clase con Spring Cloud y era la opción con menos piezas nuevas que aprender en el momento.

## Consecuencias

Gana: los servicios pueden moverse de host/puerto sin tocar configuración del Gateway; base para escalar horizontalmente.
Resigna: Eureka es un punto de fallo adicional a operar (aunque los clientes cachean el registro y toleran caídas cortas); en 2022 no se evaluó una alternativa nativa de Kubernetes porque el despliegue no lo requería. Si el sistema migrara a Kubernetes, este ADR debería revisarse.
