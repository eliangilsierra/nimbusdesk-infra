# Arquitectura — NimbusDesk v1

## Diagrama de contexto

Quiénes interactúan con el sistema y qué sistemas externos toca.

```mermaid
flowchart LR
    Employee(["Empleado<br/>(usuario final)"])
    Admin(["Administrador<br/>de espacios"])
    NimbusDesk["Sistema NimbusDesk<br/>(reservas de salas / coworking)"]
    Partner[["Sistema de un partner externo<br/>(disponibilidad de espacios asociados)"]]

    Employee -->|consulta disponibilidad<br/>crea/cancela reservas| NimbusDesk
    Admin -->|administra espacios<br/>y reglas de precio| NimbusDesk
    NimbusDesk -->|sincroniza disponibilidad| Partner
```

## Diagrama de contenedores

```mermaid
flowchart TB
    subgraph Client["Cliente"]
        Web["nimbusdesk-web<br/>Angular + Nginx"]
    end

    subgraph Edge["Borde"]
        GW["nimbusdesk-api-gateway<br/>Spring Cloud Gateway<br/>valida JWT, enruta"]
    end

    Eureka["nimbusdesk-discovery-server<br/>Netflix Eureka"]

    subgraph Core["Servicios de negocio"]
        Identity["nimbusdesk-identity-service<br/>arquitectura por capas<br/>login + emisión JWT"]
        Booking["nimbusdesk-booking-service<br/>arquitectura hexagonal<br/>reservas, disponibilidad, precio"]
        Bridge["nimbusdesk-partner-bridge-service<br/>arquitectura hexagonal<br/>puerto único, 2 adaptadores"]
    end

    MySQL[("MySQL")]
    MSSQL[("SQL Server")]

    Web --> GW
    GW --> Identity
    GW --> Booking
    GW --> Bridge

    Identity -.registro.-> Eureka
    Booking -.registro.-> Eureka
    Bridge -.registro.-> Eureka
    GW -.resuelve rutas.-> Eureka

    Booking -->|Feign, síncrono| Bridge

    Identity --> MySQL
    Booking --> MySQL
    Bridge --> MySQL
    Bridge --> MSSQL
```

## Por qué dos estilos internos conviven en v1

`identity-service` usa capas clásicas (`controller` → `service` → `persistence`) mientras `booking-service` y `partner-bridge-service` usan hexagonal (`domain` → `application` → `infrastructure`). No es un descuido de este repo: es la reproducción fiel de una decisión real — un servicio de autenticación relativamente simple no siempre justifica el costo de puertos/adaptadores. El razonamiento completo está en [ADR-0003](../../adr/0003-identity-service-arquitectura-por-capas.md).

## Detalle interno de un servicio hexagonal (`booking-service` / `partner-bridge-service`)

```mermaid
flowchart LR
    subgraph Infra_in["infrastructure/input"]
        Rest["REST Controller"]
    end

    subgraph App["application"]
        Handler["handler"]
        DTO["dto / mapper"]
    end

    subgraph Domain["domain"]
        API["api (puerto de entrada)"]
        UseCase["useCase"]
        Model["model"]
        SPI["spi (puerto de salida)"]
    end

    subgraph Infra_out["infrastructure/output"]
        AdapterA["adapter MySQL"]
        AdapterB["adapter SQL Server"]
    end

    Rest --> Handler --> API --> UseCase
    UseCase --> Model
    UseCase --> SPI
    SPI -.implementado por.-> AdapterA
    SPI -.implementado por.-> AdapterB
```

El punto central de la demo: `domain/useCase` depende de la interfaz `domain/spi`, nunca de un adaptador concreto. En `partner-bridge-service` ese único puerto tiene dos implementaciones (MySQL y SQL Server) seleccionables por perfil de Spring — la prueba de que el dominio no sabe (ni le importa) contra qué motor de base de datos corre.
