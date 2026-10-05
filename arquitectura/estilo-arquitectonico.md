# Estilo arquitectónico

## Estilo seleccionado: Monolito modular en capas

| **Elemento** | **Descripción aplicada a SICA** |
|--------------|---------------------------------|
| **Estilo arquitectónico** | Monolito modular combinado con arquitectura en capas. |
| **Monolito** | Unidad de despliegue: todo el backend se construye y despliega como una sola aplicación (una imagen Docker). |
| **Modular** | Organización por dominio funcional: Identidad, Consulta de Paquetes, Alertas, Carga de Datos, Auditoría y Administración. |
| **En capas** | Organización lógica dentro de cada módulo: Presentación → Lógica de negocio → Datos. |
| **Decisión que lo respalda** | [ADR-001](../analisis-de-sistema/07-decisiones-arquitectonicas.md) |
| **Drivers que atiende** | DA01 – Escalabilidad, DA03 – Disponibilidad, DA08 – Evolución modular, DA09 – Portabilidad, DA10 – API REST. |

> **Capas** = organización lógica del código. **Monolito** = unidad de despliegue. Ambos conceptos coexisten: un monolito puede (y debe) estar organizado internamente en capas y módulos.

## Diagrama de arquitectura

```mermaid
flowchart TB

%% ===== ACTORES =====
Padre["Padre / Tutor"]
Admin["Administrador UEI"]

Web["Portal de Consulta Ciudadana<br/>[Aplicación web responsiva]"]
LB["Balanceador de carga<br/>[HTTPS]"]

Padre --> Web
Admin --> Web
Web -->|"HTTPS · JSON<br/>/api/v1/*"| LB

%% ===== MONOLITO =====
subgraph MONO["«monolito» SICA Backend · una aplicación · una imagen Docker · N instancias sin estado"]
    direction TB

    MW["Middlewares transversales<br/>HTTPS · CORS · limitador de tasa · validación de entrada · manejo de errores · logger"]

    subgraph PRES["1. CAPA DE PRESENTACIÓN · recibe peticiones HTTP y responde JSON"]
        direction LR
        C1["identidad.controller"]
        C2["consulta.controller"]
        C3["alertas.controller"]
        C4["carga.job<br/>(tarea programada)"]
        C5["auditoria.controller"]
        C6["admin.controller"]
    end

    subgraph NEG["2. CAPA DE LÓGICA DE NEGOCIO · reglas y coordinación entre módulos"]
        direction LR
        S1["identidad.service<br/>valida tutor y niño"]
        S2["consulta.service<br/>estado de paquetes"]
        S3["alertas.service<br/>pendientes y por vencer"]
        S4["carga.service<br/>importa atenciones"]
        S5["auditoria.service<br/>registra consultas"]
        S6["admin.service<br/>catálogos y usuarios"]
    end

    subgraph DAT["3. CAPA DE DATOS · persistencia, caché e integraciones"]
        direction LR
        R1["identidad.repository<br/>+ cliente RENIEC"]
        R2["consulta.repository<br/>+ caché"]
        R3["alertas.repository"]
        R4["carga.repository<br/>+ lector SQL Server"]
        R5["auditoria.repository"]
        R6["admin.repository"]
    end

    MW --> PRES
    C1 --> S1 --> R1
    C2 --> S2 --> R2
    C3 --> S3 --> R3
    C4 --> S4 --> R4
    C5 --> S5 --> R5
    C6 --> S6 --> R6

    S2 -.->|"usa"| S1
    S2 -.->|"usa"| S3
    S2 -.->|"usa"| S5
end

LB --> MW

%% ===== INFRAESTRUCTURA DE DATOS =====
Redis[("Redis<br/>caché con TTL")]
PG[("PostgreSQL<br/>base principal")]
Rep[("PostgreSQL<br/>réplica de solo lectura")]

R2 --> Redis
R2 --> Rep
R3 --> Rep
R1 --> Rep
R4 -->|"escribe"| PG
R5 -->|"escribe"| PG
R6 --> PG
PG -->|"replicación asíncrona"| Rep

%% ===== SISTEMAS EXTERNOS =====
RENIEC["«sistema externo»<br/>RENIEC"]
SQLS["«sistema externo»<br/>SQL Server<br/>atenciones digitadas"]

R1 -->|"HTTPS / REST"| RENIEC
SQLS -->|"lectura periódica"| R4

style MONO fill:#f4f8fc,stroke:#2b5b8a,stroke-width:2px,stroke-dasharray: 6 4,color:#222
style MW fill:#dce8f4,stroke:#2b5b8a,color:#222
style PRES fill:#e8f0fa,stroke:#4a7eb5,color:#222
style NEG fill:#e6f4e6,stroke:#4a8a4a,color:#222
style DAT fill:#fbf1e1,stroke:#b5874a,color:#222
style RENIEC fill:#eee,stroke:#333,stroke-dasharray: 5 5,color:#222
style SQLS fill:#eee,stroke:#333,stroke-dasharray: 5 5,color:#222
```

**Leyenda:** flecha continua = llamada síncrona entre capas (de arriba hacia abajo); flecha punteada = uso entre módulos (solo a través de su *service*); caja discontinua = límite del monolito; «sistema externo» = fuera de SICA.

## Módulos del monolito

| **Módulo** | **Responsabilidad** | **Requisitos que cubre** |
|------------|---------------------|--------------------------|
| **Identidad** | Validar el DNI del padre/tutor y del niño, y la relación entre ambos (con RENIEC). | RF-01, RF-02 |
| **Consulta de Paquetes** | Obtener el estado de cumplimiento de cada paquete de atención integral (CRED, vacunas, hierro, tamizajes, visitas). | RF-03 |
| **Alertas** | Calcular las atenciones pendientes o próximas a vencer. | RF-04 |
| **Carga de Datos** | Importar periódicamente las atenciones desde SQL Server hacia PostgreSQL. | RF-06 |
| **Auditoría** | Registrar fecha, hora y resultado de cada consulta pública. | RF-07 |
| **Administración** | Gestionar catálogos, usuarios y reportes. | Actor Administrador UEI |
| **Transversal (middlewares)** | Limitador de tasa, HTTPS, validación de entrada, manejo de errores, logs. | RF-05 |

