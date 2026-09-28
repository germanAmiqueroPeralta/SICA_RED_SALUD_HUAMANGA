# Arquitectura inicial del sistema

Se documenta con el modelo **C4** (Simon Brown), desarrollando los tres primeros niveles: Contexto, Contenedores y Componentes. El nivel 4 (Código) no se desarrolla por corresponder a detalle de implementación.

## Nivel 1: Diagrama de Contexto

```mermaid
flowchart TD

Padre["Padre o Madre de Familia<br/>[Persona]<br/>Consulta el estado de atención integral de su hijo(a)"]
Admin["Administrador UEI<br/>[Persona]<br/>Gestiona catálogos, usuarios y reportes"]
Reg["Registradores<br/>[Persona]<br/>Digitan las atenciones en SQL Server (fuera de SICA)"]

SICA["Sistema SICA<br/>[Sistema de Software]<br/>Controla y alerta el cumplimiento de los planes de atención integral del niño en la Red de Salud Huamanga"]

SQLS["SQL Server<br/>[Base de Datos Externa]<br/>Fuente de los datos de atenciones de la Red de Salud Huamanga"]
RENIEC["RENIEC<br/>[Sistema Externo]<br/>Validación de identidad de los ciudadanos"]

Padre -->|"Consulta el estado de atención<br/>(escenario de alta concurrencia)"| SICA
Admin -->|"Administra"| SICA
Reg -->|"Digitan atenciones"| SQLS
SQLS -->|"Carga de datos de atenciones"| SICA
SICA -->|"Valida identidad (DNI)"| RENIEC

style Padre fill:#5b7c99,stroke:#333,color:#fff
style Admin fill:#5b7c99,stroke:#333,color:#fff
style Reg fill:#5b7c99,stroke:#333,stroke-dasharray: 5 5,color:#fff
style SICA fill:#2b5b8a,stroke:#333,color:#fff
style SQLS fill:#eee,stroke:#333,stroke-dasharray: 5 5,color:#222
style RENIEC fill:#eee,stroke:#333,stroke-dasharray: 5 5,color:#222
```

El sistema SICA se sitúa frente a dos tipos de usuario (padres o tutores y administrador de la UEI) y dos sistemas externos: **SQL Server**, fuente de las atenciones, y **RENIEC**, para validar identidad. Los registradores no usan SICA: digitan las atenciones directamente en SQL Server, y SICA las incorpora mediante una carga de datos. La relación entre el padre de familia y el sistema es el punto donde se concentra el escenario de alta concurrencia.

## Nivel 2: Diagrama de Contenedores

```mermaid
flowchart TD

Padre["Padre o Madre de Familia<br/>[Persona]<br/>Consulta desde su celular"]

subgraph SICA["Sistema SICA"]
    PortalC["Portal de Consulta Ciudadana<br/>[Aplicación Web]<br/>Consulta pública del estado de atención (alta concurrencia)"]
    GW["API Gateway / Balanceador de Carga<br/>[Contenedor]<br/>Enruta solicitudes y aplica límite de tasa (rate limiting)"]
    APIC["API de Consulta Pública<br/>[Servicio - solo lectura]<br/>Atiende las consultas masivas de los padres. Escalable horizontalmente"]
    Carga["Servicio de Carga de Datos<br/>[Servicio]<br/>Carga las atenciones desde SQL Server hacia la base de datos de SICA"]
    Redis[("Caché<br/>[Redis]<br/>Guarda resultados de consulta frecuentes")]
    BD[("Base de Datos de SICA<br/>[PostgreSQL]<br/>Padrón nominal y atenciones cargadas")]
    Replica[("Réplica de Solo Lectura<br/>[PostgreSQL - Read Replica]<br/>Atiende las consultas del portal ciudadano")]
end

SQLS["SQL Server<br/>[Base de Datos Externa]<br/>Atenciones digitadas por los registradores"]
RENIEC["RENIEC<br/>[Sistema Externo]<br/>Validación de identidad"]

Padre -->|"HTTPS"| PortalC
PortalC -->|"Solicitudes de consulta"| GW
GW --> APIC
APIC -->|"Lee / escribe"| Redis
Redis -->|"Si no hay dato en caché"| Replica
SQLS -->|"Carga de datos"| Carga
Carga -->|"Escribe"| BD
BD -->|"Replicación asíncrona"| Replica
APIC -->|"Valida identidad (opcional)"| RENIEC

style Padre fill:#5b7c99,stroke:#333,color:#fff
style PortalC fill:#4a7eb5,stroke:#333,color:#fff
style GW fill:#4a7eb5,stroke:#333,color:#fff
style APIC fill:#4a7eb5,stroke:#333,color:#fff
style Carga fill:#4a7eb5,stroke:#333,color:#fff
style Redis fill:#dce8f4,stroke:#333,color:#222
style BD fill:#dce8f4,stroke:#333,color:#222
style Replica fill:#dce8f4,stroke:#333,color:#222
style SQLS fill:#eee,stroke:#333,stroke-dasharray: 5 5,color:#222
style RENIEC fill:#eee,stroke:#333,stroke-dasharray: 5 5,color:#222
```

El **Portal de Consulta Ciudadana** envía las solicitudes al **API Gateway**, que las enruta a la **API de Consulta Pública** (solo lectura, escalable horizontalmente). Esta se apoya en **Redis** y en una **réplica de solo lectura** de la base de datos. Por otro lado, el **Servicio de Carga de Datos** trae las atenciones desde **SQL Server** y las escribe en la base de datos de SICA, que se replica a la réplica de lectura. Así, los picos de consulta no compiten por recursos con la carga de datos.

## Nivel 3: Diagrama de Componentes (API de Consulta Pública)

```mermaid
flowchart TD

GW["API Gateway<br/>[Contenedor]<br/>Enruta la solicitud entrante"]

subgraph API["API de Consulta Pública"]
    Ctrl["Controlador de Consulta<br/>[Componente REST]<br/>Expone el endpoint GET /consultas/{dni}"]
    RL["Limitador de Tasa<br/>[Rate Limiter]<br/>Controla el número de solicitudes por usuario en cada ventana de tiempo"]
    Val["Servicio de Validación de Identidad<br/>[Componente]<br/>Verifica el DNI del padre/tutor y del niño"]
    Paq["Servicio de Consulta de Paquetes<br/>[Componente]<br/>Obtiene el estado de los paquetes de atención integral del niño"]
    Cache["Gestor de Caché<br/>[Componente]<br/>Consulta y actualiza el caché Redis"]
    Repo["Repositorio de Datos<br/>[Componente]<br/>Acceso de solo lectura a la réplica de la base de datos"]
end

RENIEC["RENIEC<br/>[Sistema Externo]<br/>Validación de identidad"]
Redis[("Caché<br/>[Redis]")]
Replica[("Réplica de Solo Lectura<br/>[PostgreSQL]")]

GW --> Ctrl
Ctrl -->|"1. Verifica límite"| RL
RL -->|"2. Solicitud dentro del límite"| Val
Val -->|"3. Consulta opcional"| RENIEC
Val -->|"4. Identidad válida"| Paq
Paq -->|"5. Busca en caché"| Cache
Cache --> Redis
Cache -->|"6. Si hay cache miss"| Repo
Repo --> Replica

style GW fill:#4a7eb5,stroke:#333,color:#fff
style Ctrl fill:#dce8f4,stroke:#333,color:#222
style RL fill:#dce8f4,stroke:#333,color:#222
style Val fill:#dce8f4,stroke:#333,color:#222
style Paq fill:#dce8f4,stroke:#333,color:#222
style Cache fill:#dce8f4,stroke:#333,color:#222
style Repo fill:#dce8f4,stroke:#333,color:#222
style Redis fill:#dce8f4,stroke:#333,color:#222
style Replica fill:#dce8f4,stroke:#333,color:#222
style RENIEC fill:#eee,stroke:#333,stroke-dasharray: 5 5,color:#222
```

Se hace zoom sobre la API de Consulta Pública por ser el contenedor que concentra el escenario de alta concurrencia. Ante un incremento de tráfico solo es necesario escalar este contenedor, sin afectar el resto del sistema.
