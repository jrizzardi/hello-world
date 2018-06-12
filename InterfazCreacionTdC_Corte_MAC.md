## Creación de TdC en eOrder - Tipo Corte - Origen MAC
### Detalles del Proceso


1. Buscar en MAC todas las ordenes de CORTE que aun no fueron exportadas a eOrder

> Nota: tener en cuenta al momento de la puesta en marcha, que se deben procesar en MAC todas las ordenes pendientes en SOLDIME antes de comenzar a trabajar con eOrder.

~~~
SELECT nro_servicio,
       fecha_solicitud,
       numero_cliente
   FROM servicio_cab 
  WHERE id_servicio = 1 
    AND estado = 'T' 
 ORDER BY fecha_solicitud
~~~

2. Por cada orden encontrada en la consulta anterior, armar una TdC con los siguientes datos:

> Nota: el resto de los campos que no aparecen en la tabla deben completarse con un valor nulo (o no completarse, de acuerdo a lo que permita el WS de eORder).

> Nota 1: Formato del código externo de Orden: M########### (para MAC: son 14 caracteres en total, comienzan con el caracterer M y luego el número de servicio completado con hasta 13 ceros a izquierda). 
  
> Nota 2: Para completar datos de la orden se deben consultar tablas adicionales a servicio_cab.

~~~
SELECT marca_medidor,
       modelo_medidor,
       numero_medidor,
       info_adic_lectura
   FROM servicio_corte 
  WHERE nro_servicio = <servicio_cab.nro_servicio>
  
SELECT codigo_voltaje
   FROM tecni 
  WHERE numero_cliente = <servicio_cab.numero_cliente>
~~~

| Elemento | Valor |
| --------- | --------- | 
| **Request.DATOS_CLAVE_PROCESOS_TDC** |  |
| CODIGO_DISTRIBUIDORA | ESU |
| CODIGO_EXTERNO_DEL_TDC | servicio_cab.nro_servicio con formato (ver Nota 1) |
| CODIGO_PROCESO | GI |
| CODIGO_SISTEMA_EXTERNO_DE_ORIGEN | ESUMAC |
| CODIGO_SUBPROCESO | SCR |
| CODIGO_TIPO_DE_TDC | SCR.01 |
| LLAVE_SECRETA | -*Definir*- |
| **Request.DATOS_COMUNES_PROCESOS_TDC** | |
| FECHA_Y_HORA_DE_CREACION_DE_LA_ORDEN | servicio_cab.fecha_solicitud |
| TIEMPO_DE_REFERENCIA_PARA_ANS_CONTRACTISTA | |
| TIEMPO_DE_REFERENCIA_PARA_ANS_INTERNO | |
| TIEMPO_DE_REFERENCIA_PARA_ANS_LEGAL | |
| SECTOR | |
| ZONA | |
| TIPO_DE_RED | |
| PARAMETRO_NIVEL_OPERATIVO_3 | |
| VALOR_NIVEL_OPERATIVO_3 | |
| **MEDIDORS: Request.DATOS_COMUNES_PROCESOS_TDC.MEDIDORS** | |
| MARCA_MEDIDOR | servicio_corte.marca_medidor |
| MODELO_MEDIDOR | servicio_corte.modelo_medidor |
| NUMERO_MEDIDOR | servicio_corte.numero_medidor |
| TIPO_MEDIDOR | |
| UBICACION_DEL_MEDIDOR | servicio_corte.info_adic_lectura |
| VALOR_TENSION_NOMINAL | |
| **SELLOS: Request.DATOS_COMUNES_PROCESOS_TDC.SELLOS** | |
| COLOR_DEL_SELLO | |
| SELLOS_EN_SISTEMA | |
| TIPO_DE_SELLO | |
| UBICACION_DEL_SELLO | |
| SERIE_SELLO | |
| **SUMINSTROS: Request.DATOS_COMUNES_PROCESOS_TDC.SUMINISTROS** | |
| ANTIGUEDAD  |
| CODIGO_CLIENTE | | 
| CODIGO_POSTAL | |
| DEUDA | |
| GIRO_DE_NEGOCIO | |
| LATITUD_CLIENTE | |
| LOCALIDAD | |
| LOCALIZACION_TERRENO | |
| LONGITUD_CLIENTE | |
| NOMBRE_Y_APELLIDO_CLIENTE | |
| PISO | |
| PROVINCIA | |
| RUTA_DE_LECTURA | |
| SUCURSAL | |
| TARIFA_EXISTENTE | |
| TELEFONO_CONTACTO | |
| TEXTO_DIRECCION | |


