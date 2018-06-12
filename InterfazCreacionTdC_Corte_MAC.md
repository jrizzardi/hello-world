## Creación de TdC en eOrder - Tipo Corte - Origen MAC
### Detalles del Proceso


1. Buscar en MAC todas las ordenes de CORTE que aun no fueron exportadas a eOrder

> Nota: tener en cuenta al momento de la puesta en marcha, que se deben procesar en MAC todas las ordenes pendientes en SOLDIME antes de comenzar a trabajar con eOrder.

~~~
SELECT nro_servicio,
       fecha_solicitud,
       numero_cliente,
       sector,
       zona
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
  
SELECT codigo_voltaje,
       tec_tipo_instala
  FROM tecni 
  WHERE numero_cliente = <servicio_cab.numero_cliente>
~~~

> Nota 3: Completar el valor de **VALOR_TENSION_NOMINAL** de acuerdo al valor de tecni.codigo_voltaje:  

| Valor de codigo_voltaje | Enviar |
|-----|------|
| 1 | 220V | 
| 2 | 225V | 
| 3 | 380V | 


> Nota 4: Para completar el valor de **VALOR_NIVEL_OPERATIVO_3** se debe respetar la siguiente lógica (en general):  

>> Si servicio_cab.tarifa = T1:
       >>> Ver el tipo de cuadrilla asignada a la Orden: CONTRATADA o PROPIA

>> Si servicio_cab.tarifa = T2:
       >>> Si es DIRECTO: Ver el tipo de cuadrilla asignada a la Orden: CONTRATADA o PROPIA
       >>> Si es INDIRECTO: PROPIA
       
>> Si servicio_cab.tarifa = T3:
       >>> Si es INDIRECTO: PROPIA


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
| TIEMPO_DE_REFERENCIA_PARA_ANS_CONTRACTISTA | servicio_cab.fecha_solicitud |
| TIEMPO_DE_REFERENCIA_PARA_ANS_INTERNO | servicio_cab.fecha_solicitud |
| TIEMPO_DE_REFERENCIA_PARA_ANS_LEGAL | servicio_cab.fecha_solicitud |
| SECTOR | servicio_cab.sector |
| ZONA | servicio_cab.zona |
| TIPO_DE_RED | tecni.tec_tipo_instala |
| PARAMETRO_NIVEL_OPERATIVO_3 | TIPO_CUADRILLA |
| VALOR_NIVEL_OPERATIVO_3 | |
| **MEDIDORS: Request.DATOS_COMUNES_PROCESOS_TDC.MEDIDORS** | |
| MARCA_MEDIDOR | servicio_corte.marca_medidor |
| MODELO_MEDIDOR | servicio_corte.modelo_medidor |
| NUMERO_MEDIDOR | servicio_corte.numero_medidor |
| TIPO_MEDIDOR | |
| UBICACION_DEL_MEDIDOR | servicio_corte.info_adic_lectura |
| VALOR_TENSION_NOMINAL | Valor según tecni.codigo_voltaje (Ver Nota 3) |
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


