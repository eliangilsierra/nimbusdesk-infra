# ADR-0002: Validación de JWT centralizada en el API Gateway

Fecha: 2022-03-15
Estado: Aceptado

## Contexto

Con varios servicios de negocio detrás del Gateway, validar el token en cada uno por separado implica duplicar la misma lógica de seguridad (parseo, verificación de firma, expiración) N veces, con el riesgo de que diverjan.

## Decisión

El `api-gateway` valida el JWT en un filtro (`AuthenticationFilter`) antes de enrutar la petición, usando un `RouteValidator` que mantiene la lista de rutas públicas (login, health) exentas de validación. Los servicios de negocio confían en que, si la petición llegó, el token ya fue validado.

## Alternativas consideradas

- **Validar el JWT en cada microservicio** — descartado por duplicación de lógica de seguridad y mayor superficie de inconsistencia si un servicio queda desactualizado respecto a otro.
- **Un servicio de autorización externo por petición (ej. OAuth2 introspection remota)** — descartado por la latencia adicional de una llamada de red por request, quedándose con verificación local de firma JWT.

## Consecuencias

Gana: un solo punto donde cambiar la lógica de autenticación; los servicios de negocio quedan más simples.
Resigna: el Gateway se vuelve un componente crítico de seguridad — si tiene un bug, todo el sistema queda expuesto o bloqueado. Además, los servicios de negocio confían ciegamente en la cabecera que les llega del Gateway, lo cual asume que no hay forma de llegar a ellos sin pasar por el borde (en este portafolio, se refuerza no exponiendo puertos de los servicios de negocio fuera de la red interna de Docker).
