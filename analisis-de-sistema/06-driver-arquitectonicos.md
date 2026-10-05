## Drivers arquitectónicos

| **ID** | **Driver arquitectónico** | **Origen** | **¿Por qué influye en la arquitectura?** |
|--------|---------------------------|------------|------------------------------------------|
| **DA01** | El sistema debe soportar picos de hasta aproximadamente 10 000 consultas simultáneas durante una campaña. | AC02 – Escalabilidad | Justifica una API de Consulta Pública sin estado, escalable horizontalmente detrás de un balanceador de carga. |
| **DA02** | El sistema debe mantener tiempos de respuesta adecuados ante consultas repetidas y concurrentes. | AC01 – Rendimiento | Justifica el uso de caché (Redis) con TTL antes de consultar la base de datos. |
| **DA03** | Un pico de consultas públicas no debe comprometer la carga de datos desde SQL Server. | AC03 – Disponibilidad | Justifica separar la API de Consulta Pública (solo lectura) del Servicio de Carga de Datos y usar una réplica de lectura. |
| **DA04** | El sistema debe proteger los datos del niño validando la identidad del padre o tutor y limitando solicitudes. | AC04 – Seguridad | Influye en la validación de identidad, el limitador de tasa, HTTPS y el log de auditoría. |
| **DA05** | El sistema debe validar la identidad de los ciudadanos con RENIEC. | RC04 – RENIEC | Condiciona la integración con un servicio externo y motiva reutilizar la validación dentro de la sesión. |
| **DA06** | El sistema debe actualizar sus datos mediante una carga desde SQL Server. | RC03 – Carga desde SQL Server | Condiciona el mecanismo de carga hacia la base de datos de SICA y la vigencia de los datos que consulta la API de Consulta Pública. |
| **DA07** | El backend debe permitir reemplazar componentes sin afectar a los demás. | AC05 – Mantenibilidad | Justifica aplicar Clean Architecture (dependencias hacia el dominio, puertos y adaptadores) con separación por capas. |
| **DA08** | El sistema debe permitir modificar o agregar funcionalidades (por ejemplo, un nuevo paquete de atención o un nuevo tipo de alerta) sin afectar innecesariamente a otros módulos. | AC05 – Mantenibilidad | Influye en la separación de responsabilidades, la modularidad y el control de dependencias internas entre módulos. |
| **DA09** | El sistema debe ejecutarse igual en el equipo local, el servidor de la universidad o la nube. | AC06 – Portabilidad; RC02 – Contenedores Docker | Condiciona que el backend sea una unidad desplegable empaquetada en una imagen Docker, con configuración externa (variables de entorno). |
| **DA10** | El portal ciudadano debe usarse desde el celular y comunicarse con el backend de forma desacoplada. | AC07 – Usabilidad; RC01 – Aplicación web responsiva | Justifica separar el frontend web responsivo del backend mediante una API REST (HTTPS/JSON). |
| **DA11** | Cada consulta pública debe quedar registrada para auditoría y trazabilidad. | RF-07; AC04 – Seguridad | Influye en un módulo de auditoría transversal que registre fecha, hora y resultado de cada consulta sin bloquear la respuesta al ciudadano. |

## Drivers y decisiones que responden

| **Driver** | **Problema que plantea** | **Decisión que responde** |
|------------|--------------------------|---------------------------|
| **DA01 – Escalabilidad** | Aumentarán las consultas durante las campañas de salud. | Monolito modular sin estado, replicable horizontalmente detrás de un balanceador. |
| **DA02 – Rendimiento** | Habrá alta concurrencia y consultas repetidas. | Caché Redis con TTL antes de consultar la base de datos. |
| **DA03 – Disponibilidad** | La consulta masiva puede competir con la carga de datos. | Réplica de solo lectura y ejecución de la carga en un proceso separado. |
| **DA04 – Seguridad** | Se exponen datos sensibles de menores de edad. | Validación de identidad, limitador de tasa y HTTPS. |
| **DA05 – RENIEC** | Hay que comunicarse con un sistema externo de identidad. | Integración mediante un puerto (interfaz) y un adaptador RENIEC. |
| **DA06 – Carga desde SQL Server** | Los datos provienen de una base externa. | Adaptador de carga (lectura de SQL Server) desacoplado del dominio. |
| **DA07 – Mantenibilidad** | Cambiar una tecnología no debe afectar al resto. | Clean Architecture. |
| **DA08 – Evolución modular** | Los cambios en un módulo no deben afectar a otros. | Modularidad por dominio funcional + Clean Architecture dentro de cada módulo. |
| **DA09 – Portabilidad** | El sistema debe correr en distintos entornos. | Una imagen Docker con configuración por variables de entorno. |
| **DA10 – API REST** | Frontend y backend deben comunicarse de forma desacoplada. | Separar el portal web responsivo del backend mediante una API REST. |
| **DA11 – Auditoría** | Toda consulta debe ser trazable. | Módulo de auditoría transversal invocado desde los casos de uso. |
