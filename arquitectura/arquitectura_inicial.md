# Arquitectura inicial del sistema

## Diagrama de arquitectura

```mermaid
flowchart TD

%% =========================
%% ACTORES
%% =========================

subgraph ACTORES["ACTORES"]
    Padre["Padre / Tutor"]
    Admin["Administrador UEI"]
end

%% =========================
%% PRESENTACIÓN
%% =========================

subgraph PRESENTACION["PRESENTACIÓN"]
    Web["Aplicación Web → API REST"]
end

%% =========================
%% LÓGICA DE NEGOCIO
%% =========================

subgraph NEGOCIO["LÓGICA DE NEGOCIO"]
    Identidad["Validación de identidad"]
    Paquetes["Consulta de paquetes"]
    Alertas["Alertas"]
    Carga["Carga de datos"]
    Auditoria["Auditoría"]
end

%% =========================
%% DATOS
%% =========================

subgraph DATOS["DATOS"]
    Cache["Caché (Redis)"]
    BD["Base de datos (PostgreSQL)"]
    Replica["Réplica de solo lectura"]
end

%% =========================
%% SISTEMAS EXTERNOS
%% =========================

subgraph EXTERNOS["SISTEMAS EXTERNOS"]
    SQLS["SQL Server"]
    RENIEC["RENIEC"]
end

%% =========================
%% FLUJO PRINCIPAL
%% =========================

ACTORES --> PRESENTACION
PRESENTACION --> NEGOCIO
NEGOCIO --> DATOS

%% =========================
%% INTEGRACIONES
%% =========================

DATOS -->|"integraciones"| EXTERNOS

%% =========================
%% DISTRIBUCIÓN HORIZONTAL
%% =========================

Padre ~~~ Admin

Identidad ~~~ Paquetes
Paquetes ~~~ Alertas
Alertas ~~~ Carga
Carga ~~~ Auditoria

Cache ~~~ BD
BD ~~~ Replica

SQLS ~~~ RENIEC

%% =========================
%% ESTILOS
%% =========================

style ACTORES fill:#222,stroke:#fff,stroke-width:2px,color:#fff
style PRESENTACION fill:#222,stroke:#fff,stroke-width:2px,color:#fff
style NEGOCIO fill:#222,stroke:#fff,stroke-width:2px,color:#fff
style DATOS fill:#222,stroke:#fff,stroke-width:2px,color:#fff
style EXTERNOS fill:#222,stroke:#fff,stroke-width:2px,color:#fff

style Padre fill:#222,stroke:#fff,color:#fff
style Admin fill:#222,stroke:#fff,color:#fff

style Web fill:#222,stroke:#fff,color:#fff

style Identidad fill:#222,stroke:#fff,color:#fff
style Paquetes fill:#222,stroke:#fff,color:#fff
style Alertas fill:#222,stroke:#fff,color:#fff
style Carga fill:#222,stroke:#fff,color:#fff
style Auditoria fill:#222,stroke:#fff,color:#fff

style Cache fill:#222,stroke:#fff,color:#fff
style BD fill:#222,stroke:#fff,color:#fff
style Replica fill:#222,stroke:#fff,color:#fff

style SQLS fill:#222,stroke:#fff,color:#fff
style RENIEC fill:#222,stroke:#fff,color:#fff
```

## Descripción

La arquitectura inicial se organiza en tres capas principales:

* **Presentación:** permite la interacción de los usuarios con el sistema mediante la aplicación web y la API REST.
* **Lógica de negocio:** contiene los módulos responsables de las funcionalidades del sistema: validación de identidad, consulta de paquetes de atención, alertas de atenciones pendientes, carga de datos y auditoría de consultas.
* **Datos:** almacena y sirve la información mediante una base de datos PostgreSQL, una réplica de solo lectura para las consultas masivas y un caché (Redis) que reduce la carga sobre la base de datos.

Además, el módulo de **Validación de identidad** se integra con **RENIEC** para verificar el DNI, y el módulo de **Carga de datos** obtiene las atenciones desde **SQL Server**, donde los registradores digitan la información fuera de SICA.

El detalle de contenedores y componentes de esta arquitectura se documenta en [`arquitectura-c4.md`](arquitectura-c4.md) y el flujo de una consulta en [`flujo-consulta.md`](flujo-consulta.md).