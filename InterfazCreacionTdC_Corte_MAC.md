## Creación de TdC en eOrder - Tipo Corte - Origen MAC
### Detalles del Proceso


1. Buscar en MAC todas las ordenes de CORTE que aun no fueron exportadas a eOrder

> Nota: tener en cuenta al momento de la puesta en marcha, que se deben procesar en MAC todas las ordenes pendientes en SOLDIME antes de comenzar a trabajar con eOrder.

~~~
SELECT nro_servicio,
       fecha_solicitud,
       numero_cliente,
       sector,
       zona,
       sucursal,
       tarifa
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
       info_adic_lectura,
       pr_serie1_c, pr_numero1_c,
       pr_serie2_c, pr_numero2_c,
       pr_serie3_c, pr_numero3_c,
       antiguedad_saldo,
       deuda,
       piso_dir, 
       depto_dir,
       correlativo_ruta,
       telefono,
       nom_calle,
       nro_dir,
       nom_entre,
       nom_entre1
  FROM servicio_corte 
 WHERE nro_servicio = <servicio_cab.nro_servicio>
  
SELECT codigo_voltaje,
       tec_tipo_instala
  FROM tecni 
 WHERE numero_cliente = <servicio_cab.numero_cliente>
  
SELECT clave_montri
  FROM medid
 WHERE marca_medidor = <servicio_corte.marca_medidor> 
   AND modelo_medidor = <servicio_corte.modelo_medidor> 
   AND numero_medidor = <servicio_corte.numero_medidor>
   AND numero_cliente = <servicio_cab.numero_cliente>
   
SELECT cod_postal,
       provincia, 
       nom_comuna, 
       nom_barrio,
       nom_provincia
  FROM cliente 
 WHERE numero_cliente = <servicio_cab.numero_cliente>
 
SELECT lat, lon
  FROM ubica_geo_cliente 
 WHERE numero_cliente = <servicio_cab.numero_cliente>
 
~~~


> Nota 3: Para completar el valor de **VALOR_NIVEL_OPERATIVO_3** se debe respetar la siguiente lógica (en general):  

* Si servicio_cab.tarifa = T1:
    * Ver el tipo de cuadrilla asignada a la Orden: CONTRATADA o PROPIA

* Si servicio_cab.tarifa = T2:
    * Si es DIRECTO: Ver el tipo de cuadrilla asignada a la Orden: CONTRATADA o PROPIA
    * Si es INDIRECTO: PROPIA
       
* Si servicio_cab.tarifa = T3:
    * PROPIA


> Nota 4: Completar el valor de **VALOR_TENSION_NOMINAL** de acuerdo al valor de medid.clave_montri:  

| Valor de clave_montri | Enviar |
|-----|------|
| M | MONOFASICO | 
| T | TRIFASICO | 


> Nota 5: Completar el valor de **VALOR_TENSION_NOMINAL** de acuerdo al valor de tecni.codigo_voltaje:  

| Valor de codigo_voltaje | Enviar |
|-----|------|
| 1 | 220V | 
| 2 | 225V | 
| 3 | 380V | 

> Nota 6: En MAC, el precinto que se informa es el precinto_tapa. Para obtener la información del mismo, tomar el último instalado

~~~
SELECT serie, 
       numero_precinto, 
       fecha_estado
  FROM prt_precintos
 WHERE numero_cliente = <servicio_cab.numero_cliente>
   AND estado_actual ='08'
 ORDER BY fecha_estado DESC
 
SELECT color,
       ubicacion,
       numero_precinto,
       serie
  FROM pr_precintos
 WHERE serie = <prt_precintos.serie> 
  AND numero_precinto = <prt_precintos.numero_precinto>  
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
| TIEMPO_DE_REFERENCIA_PARA_ANS_CONTRACTISTA | servicio_cab.fecha_solicitud |
| TIEMPO_DE_REFERENCIA_PARA_ANS_INTERNO | servicio_cab.fecha_solicitud |
| TIEMPO_DE_REFERENCIA_PARA_ANS_LEGAL | servicio_cab.fecha_solicitud |
| SECTOR | servicio_cab.sector |
| ZONA | servicio_cab.zona |
| TIPO_DE_RED | tecni.tec_tipo_instala |
| PARAMETRO_NIVEL_OPERATIVO_3 | TIPO_CUADRILLA |
| VALOR_NIVEL_OPERATIVO_3 | PROPIA o CONTRATADA (ver Nota 3) |
| **MEDIDORS: Request.DATOS_COMUNES_PROCESOS_TDC.MEDIDORS** | |
| MARCA_MEDIDOR | servicio_corte.marca_medidor |
| MODELO_MEDIDOR | servicio_corte.modelo_medidor |
| NUMERO_MEDIDOR | servicio_corte.numero_medidor |
| TIPO_MEDIDOR | medid.clave_montri (ver Nota 4) |
| UBICACION_DEL_MEDIDOR | servicio_corte.info_adic_lectura |
| VALOR_TENSION_NOMINAL | Valor según tecni.codigo_voltaje (Ver Nota 5) |
| **SELLOS: Request.DATOS_COMUNES_PROCESOS_TDC.SELLOS** |  |
| COLOR_DEL_SELLO | pr_precintos.color (Ver Nota 6) |
| SELLOS_EN_SISTEMA | pr_precintos.numero_precinto (Ver Nota 6) |
| TIPO_DE_SELLO | **???????** |
| UBICACION_DEL_SELLO | pr_precintos.ubicacion (Ver Nota 6) |
| SERIE_SELLO | pr_precintos.serie (Ver Nota 6) |
| **SUMINSTROS: Request.DATOS_COMUNES_PROCESOS_TDC.SUMINISTROS** | |
| ANTIGUEDAD | servicio_corte.antiguedad_saldo |
| CODIGO_CLIENTE | servicio_cab.numero_cliente | 
| CODIGO_POSTAL | cliente.cod_postal |
| DEUDA | servicio_corte.deuda |
| GIRO_DE_NEGOCIO | **???????** |
| LATITUD_CLIENTE | ubica_geo_cliente.lat  |
| LONGITUD_CLIENTE | ubica_geo_cliente.lat |
| LOCALIDAD | cliente.nom_comuna si cliente.provincia = "B"; cliente.nom_barrio si cliente.provincia = "C" |
| LOCALIZACION_TERRENO | servicio_corte.obs_dir |
| NOMBRE_Y_APELLIDO_CLIENTE | servicio_corte.nombre |
| PISO | servicio_corte.piso_dir + " " + servicio_corte.piso_depto |
| PROVINCIA | cliente.nom_provincia |
| RUTA_DE_LECTURA | servicio_corte.correlativo_ruta |
| SUCURSAL | servicio_cab.sucursal |
| TARIFA_EXISTENTE | servicio_cab.tarifa |
| TELEFONO_CONTACTO | servicio_corte.telefono |
| TEXTO_DIRECCION | servicio_corte.nom_calle + " " + servicio_corte.nro_dir + " (entre " + servicio_corte.nom_entre + " y " + servicio_corte.nom_entre1 + ")"|


3. Con la estructura cargada, consumir el WS de TIBCO P001C_PeticionCreacionTDC_ARG.wsdl. 

4. Evaluar el resultado que retorna el WS: 

* Si el Código de Error/Resultado es "00" ("Operación ejecutada con éxito"): actualizar el campo servicio_cab.estado a "W".
* Si el Código es "ODL002" ("Clave de identificación del TdC ya presiente en eOrder") o "ODL016" ("Intento de insertar a la misma vez el mismo TDC 2 veces"): la orden ya existe en eOrder. Registrar el evento pero no actualizar tablas en MAC.
* En otro caso: error no manejable. Registrar el evento (enviar mail) pero no actualizar tablas en MAC.

