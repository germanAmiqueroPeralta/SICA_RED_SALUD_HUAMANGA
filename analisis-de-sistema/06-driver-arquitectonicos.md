## Drivers arquitectónicos

| **ID** | **Driver arquitectónico** | **Origen** | **¿Por qué influye en la arquitectura?** |
|--------|---------------------------|------------|------------------------------------------|
| **DA01** | El sistema debe soportar picos de hasta aproximadamente 10 000 consultas simultáneas durante una campaña. | AC02 – Escalabilidad | Justifica una API de Consulta Pública sin estado, escalable horizontalmente detrás de un balanceador de carga. |
| **DA02** | El sistema debe mantener tiempos de respuesta adecuados ante consultas repetidas y concurrentes. | AC01 – Rendimiento | Justifica el uso de caché (Redis) con TTL antes de consultar la base de datos. |
| **DA03** | Un pico de consultas públicas no debe comprometer la carga de datos desde SQL Server. | AC03 – Disponibilidad | Justifica separar la API de Consulta Pública (solo lectura) del Servicio de Carga de Datos y usar una réplica de lectura. |
| **DA04** | El sistema debe proteger los datos del niño validando la identidad del padre o tutor y limitando solicitudes. | AC04 – Seguridad | Influye en la validación de identidad, el limitador de tasa, HTTPS y el log de auditoría. |
| **DA05** | El sistema debe validar la identidad de los ciudadanos con RENIEC. | RC04 – RENIEC | Condiciona la integración con un servicio externo y motiva reutilizar la validación dentro de la sesión. |
| **DA06** | El sistema debe actualizar sus datos mediante una carga desde SQL Server. | RC03 – Carga desde SQL Server | Condiciona el mecanismo de carga hacia la base de datos de SICA y la vigencia de los datos que consulta la API de Consulta Pública. |
| **DA07** | El backend debe permitir reemplazar componentes sin afectar a los demás. | AC05 – Mantenibilidad | Justifica la arquitectura hexagonal con separación por capas. |