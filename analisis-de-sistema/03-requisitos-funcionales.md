## Requisitos funcionales

| **ID** | **Requisito funcional** |
|--------|-------------------------|
| **RF-01** | El sistema debe permitir que un padre o tutor ingrese su DNI y el DNI del niño para iniciar una consulta pública. |
| **RF-02** | El sistema debe validar la identidad del padre o tutor y la relación con el niño consultado antes de mostrar cualquier información. |
| **RF-03** | El sistema debe mostrar el estado de cumplimiento de cada paquete de atención integral del niño (CRED, vacunas, suplementación con hierro, tamizajes, visitas domiciliarias, entre otros). |
| **RF-04** | El sistema debe señalar las atenciones pendientes o próximas a vencer del niño consultado, a manera de alerta visible para el padre o tutor. |
| **RF-05** | El sistema debe limitar la cantidad de solicitudes que un mismo usuario puede realizar en una ventana de tiempo determinada, y mostrar un mensaje de espera cuando se alcance dicho límite. |
| **RF-06** | El sistema debe sincronizar la información de atenciones con el sistema HIS de la Red de Salud Huamanga. |
| **RF-07** | El sistema debe registrar un log de cada consulta pública realizada (fecha, hora y resultado), con fines de auditoría y trazabilidad. |
| **RF-08** | El sistema debe presentar la información de la consulta pública en una interfaz responsiva, utilizable desde un teléfono móvil. |

## Relación entre historias de usuario y requisitos funcionales

| **Historia de usuario** | **Requisitos funcionales relacionados** |
|-------------------------|-----------------------------------------|
| **HU01 – Consultar estado de atención** | RF-01, RF-02, RF-03 |
| **HU02 – Ver atenciones pendientes** | RF-04 |
| **HU03 – Consultar desde el móvil** | RF-08 |
| **HU04 – Sincronizar con HIS** | RF-06 |
| **HU05 – Auditar consultas** | RF-07 |
| **HU06 – Controlar tráfico** | RF-05 |