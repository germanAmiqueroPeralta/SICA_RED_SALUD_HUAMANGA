## Atributos de calidad

| **ID** | **Atributo de calidad** | **Escenario de calidad** |
|--------|-------------------------|--------------------------|
| **AC01** | **Rendimiento** | Las consultas repetidas deben responder con rapidez mediante caché (Redis), reduciendo la carga sobre la base de datos, incluso ante picos de hasta 10 000 consultas simultáneas. |
| **AC02** | **Escalabilidad** | La API de Consulta Pública debe ser un servicio independiente y sin estado (stateless), contenedorizado, para desplegar varias instancias detrás de un balanceador de carga. |
| **AC03** | **Disponibilidad** | Una caída o saturación del módulo de consulta pública no debe afectar el registro de atenciones del personal de salud, al tratarse de servicios independientes. |
| **AC04** | **Seguridad** | La comunicación debe ser cifrada (HTTPS), las contraseñas del personal se almacenan con hash Argon2 y se valida el DNI del padre/tutor y del niño antes de mostrar información. |
| **AC05** | **Mantenibilidad** | El backend debe seguir una arquitectura semihexagonal con separación por capas, de modo que se pueda reemplazar un componente (por ejemplo, el motor de caché) sin afectar a los demás. |
| **AC06** | **Portabilidad** | El sistema debe desplegarse mediante contenedores Docker para ejecutarse en distintos entornos (equipo local, servidor de la universidad o nube) sin cambios en el código. |
| **AC07** | **Usabilidad** | El portal de consulta ciudadana debe ser responsivo y mostrar mensajes claros ante esperas (cola virtual) o errores de validación, priorizando su uso desde dispositivos móviles. |