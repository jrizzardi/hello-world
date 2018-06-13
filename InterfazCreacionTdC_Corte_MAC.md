## Creación de TdC en eOrder - Tipo Corte - Origen CANDELA
### Detalles del Proceso


1. Buscar en CANDELA todas las ordenes de CORTE (Suspensión) que aún no fueron exportadas a eOrder

> Nota: tener en cuenta al momento de la puesta en marcha, no deben quedar en CANDELA ordenes sin finalizar.

~~~
SELECT codigo_cuenta,
       correlativo
  FROM surep 
 WHERE tipo_evento = 'S' 
   AND estado = 'T' 
 ORDER BY fecha_solicitud
~~~

2. Por cada orden encontrada en la consulta anterior, armar una TdC con los siguientes datos:

> Nota: el resto de los campos que no aparecen en la tabla deben completarse con un valor nulo (o no completarse, de acuerdo a lo que permita el WS de eOrder).

> Nota 1: Formato del código externo de Orden: C###########  

Para CANDELA: son 14 caracteres en total: 
* Caracter C +
* surep.codigo_cuenta completado con hasta 10 ceros a izquierda +
* surep.correlativo completado con hasta 3 ceros a izquierda +
  
> Nota 2: Para completar datos de la orden se deben consultar tablas adicionales a surep.

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
  

~~~


> Nota 3: Para completar el valor de **VALOR_NIVEL_OPERATIVO_3** se debe respetar la siguiente lógica enunciada en el documento InterfazCreacionTdC_Corte_MAC.md  


> Nota 4: Completar el valor de **VALOR_TENSION_NOMINAL** de acuerdo al valor de tecni.codigo_voltaje:  

| Valor de codigo_voltaje | Enviar |
|-----|------|
| 1 | 220V | 
| 2 | 225V | 
| 3 | 380V | 



| Elemento | Valor |
| --------- | --------- | 
| **Request.DATOS_CLAVE_PROCESOS_TDC** |  |
| CODIGO_DISTRIBUIDORA | ESU |
| CODIGO_EXTERNO_DEL_TDC | servicio_cab.nro_servicio con formato (ver Nota 1) |
| CODIGO_PROCESO | GI |
| CODIGO_SISTEMA_EXTERNO_DE_ORIGEN | ESUCAN |
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
| GIRO_DE_NEGOCIO | tabla.descripcion para nomtabla = "ACTECO" y codigo = <cliente.actividad_economic> |
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
* Si el Código es "ODL002" ("Clave de identificación del TdC ya presiente en eOrder") o "ODL016" ("Intento de insertar a la misma vez el mismo TDC 2 veces"): la orden ya existe en eOrder. Registrar el evento y actualizar el campo servicio_cab.estado a "W".
* En otro caso: error no manejable. Registrar el evento (enviar mail) pero no actualizar tablas en MAC.

