# Decisiones arquitectónicas (ADR)

Un **ADR** (*Architecture Decision Record*, Registro de Decisión Arquitectónica) documenta una decisión importante del diseño, el driver que la motiva, su justificación y su resultado. Las decisiones responden a los drivers definidos en [06-driver-arquitectonicos.md](06-driver-arquitectonicos.md).

## Resumen de decisiones

| **ID** | **Decisión arquitectónica** | **Driver relacionado** | **Justificación** | **Resultado** |
|--------|-----------------------------|------------------------|-------------------|---------------|
| **ADR-001** | Monolito modular en capas | DA01 – Escalabilidad; DA08 – Evolución modular; DA09 – Portabilidad | Organizar las funcionalidades en módulos independientes dentro de una misma aplicación desplegable, fácil de construir y operar en un contexto académico. | Módulos de Identidad, Consulta de Paquetes, Alertas, Carga de Datos, Auditoría y Administración. |
| **ADR-002** | Clean Architecture | DA07 – Mantenibilidad; DA08 – Evolución modular | Separar las reglas del negocio (paquetes, plazos, alertas) de los detalles tecnológicos (Redis, PostgreSQL, SQL Server, RENIEC). | Capas de Dominio, Aplicación, Infraestructura y Presentación en cada módulo. |
| **ADR-003** | Estrategia de caché con Redis | DA02 – Rendimiento | Reducir consultas repetitivas a la base de datos durante los picos de campaña. | Caché *cache-aside* con TTL para el resultado de la consulta por DNI del niño. |
| **ADR-004** | Réplica de lectura y carga en proceso separado | DA03 – Disponibilidad; DA06 – Carga desde SQL Server | Evitar que las consultas masivas compitan con la escritura de la carga de datos. | Consultas sobre réplica PostgreSQL; la carga escribe en la base principal desde un proceso programado. |
| **ADR-005** | Integración con RENIEC mediante puerto y adaptador | DA05 – RENIEC | Desacoplar los casos de uso del proveedor de validación de identidad. | Contrato `ValidadorIdentidad` en el dominio y adaptador `ValidadorIdentidadReniec` en infraestructura. |
| **ADR-006** | Carga desde SQL Server mediante adaptador | DA06 – Carga desde SQL Server | Aislar el formato y la tecnología de la fuente externa de las reglas del negocio. | Contrato `FuenteAtenciones` y adaptador `FuenteAtencionesSqlServer`. |
| **ADR-007** | Instancias sin estado detrás de un balanceador | DA01 – Escalabilidad | Permitir agregar instancias del backend en campañas sin cambiar el código. | Backend *stateless*; el estado compartido vive en Redis y PostgreSQL. |
| **ADR-008** | Seguridad por capas: HTTPS, validación de identidad y limitador de tasa | DA04 – Seguridad | Proteger los datos de menores y el sistema ante abusos o picos de tráfico. | Middleware de *rate limiting*, HTTPS y validación tutor–niño antes de mostrar datos. |
| **ADR-009** | API REST entre portal web y backend | DA10 – API REST | Desacoplar el portal responsivo del backend y permitir otros clientes a futuro. | Endpoints REST (HTTPS/JSON) bajo `/api/v1`. |
| **ADR-010** | Auditoría transversal | DA11 – Auditoría | Registrar cada consulta pública sin mezclar esta responsabilidad con la lógica de consulta. | Módulo de Auditoría con contrato `RegistroAuditoria`, invocado desde los casos de uso. |
| **ADR-011** | Despliegue en contenedores Docker | DA09 – Portabilidad | Ejecutar el mismo artefacto en local, servidor de la universidad o nube. | Una imagen Docker del backend; configuración por variables de entorno. |

