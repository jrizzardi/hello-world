## Creación de TdC en eOrder - Tipo Corte - Origen MAC
### Detalles del Proceso

1. Buscar en MAC todas las ordenes de CORTE que aun no fueron exportadas a eOrder

> Nota: tener en cuenta al momento de la puesta en marcha, que se deben procesar en MAC todas las ordenes pendientes en SOLDIME antes de comenzar a trabajar con eOrder.

`SELECT * FROM servicio_cab WHERE id_servicio = 1 AND estado = '' ORDER BY fecha_solicitud`

2. Por cada orden encontrada en la consulta anterior, armar una TdC con los siguientes datos:





