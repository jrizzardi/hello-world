## Creación de TdC en eOrder - Tipo Corte - Origen CANDELA
### Detalles del Proceso


1. Buscar en CANDELA todas las ordenes de CORTE (Suspensión) que aún no fueron exportadas a eOrder

> Nota: tener en cuenta al momento de la puesta en marcha, no deben quedar en CANDELA ordenes sin finalizar.

~~~
SELECT codigo_cuenta,
       correlativo,
       fec_solicit,
       saldo_actual
  FROM surep 
 WHERE tipo_evento = 'S' 
   AND estado = '...' 
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
SELECT ciclo,
       ruta_lectura,
       inf_adicional,
       sello1, sello2, sello3, sello4,
       antiguedad_saldo,
       codigo_postal,
       codigo_ciu,
       provincia,
       localidad,
       dir_adicional,
       nombre,
       dir_piso,
       dir_departamento,
       tarifa,
       telefono,
       dir_calle,
       dir_numero,
       dir_interseccion,
       dir_interseccion2
  FROM suscriptor 
 WHERE codigo_cuenta = <surep.codigo_cuenta> 
 
SELECT tipo_instalacion,
       coord_x_gm,
       coord_y_gm
  FROM cadena_electrica 
 WHERE codigo_cuenta = <surep.codigo_cuenta> 
 
SELECT marca_contador,
       modelo_contador,
       numero_contador
  FROM conta 
 WHERE codigo_cuenta = <surep.codigo_cuenta> 
 
 SELECT descripcion
   FROM codigo_ciiu
  WHERE codigo_ciiu = <suscriptor.codigo_ciu>
~~~


> Nota 3: Para completar el valor de **VALOR_NIVEL_OPERATIVO_3** se debe respetar la siguiente lógica enunciada en el documento InterfazCreacionTdC_Corte_MAC.md  

> Nota 4: Los clientes de CANDELA, a diferencia de MAC, pueden tener más de un medidor. 

> Nota 5: Completar el valor de **VALOR_TENSION_NOMINAL** de acuerdo al valor de ??????:  

| Valor de codigo_voltaje | Enviar |
|-----|------|
| 1 | 3x380/220V |
| 2 | 3x220V |
| 3 | 13200V |
| 4 | 33000V |
| 5 | 27500V |
| 6 | 6500V |
| 7 | 132000V |
| 8 | 220000V |
| 9 | 6600V |
| 10 | 5000V |

> Nota 6: Los clientes de CANDELA, a diferencia de MAC, tienen más de un precinto.




| Elemento | Valor |
| --------- | --------- | 
| **Request.DATOS_CLAVE_PROCESOS_TDC** |  |
| CODIGO_DISTRIBUIDORA | ESU |
| CODIGO_EXTERNO_DEL_TDC | surep.codigo_cuenta + surep.correlativo con formato (ver Nota 1) |
| CODIGO_PROCESO | GI |
| CODIGO_SISTEMA_EXTERNO_DE_ORIGEN | ESUCAN |
| CODIGO_SUBPROCESO | SCR |
| CODIGO_TIPO_DE_TDC | SCR.01 |
| LLAVE_SECRETA | -*Definir*- |
| **Request.DATOS_COMUNES_PROCESOS_TDC** | |
| FECHA_Y_HORA_DE_CREACION_DE_LA_ORDEN | surep.fec_solicit (es de tipo number, pasar a datetime) |
| TIEMPO_DE_REFERENCIA_PARA_ANS_CONTRACTISTA | surep.fec_solicit (es de tipo number, pasar a datetime) |
| TIEMPO_DE_REFERENCIA_PARA_ANS_INTERNO | surep.fec_solicit (es de tipo number, pasar a datetime) |
| TIEMPO_DE_REFERENCIA_PARA_ANS_LEGAL | surep.fec_solicit (es de tipo number, pasar a datetime) |
| SECTOR | suscriptor.ciclo |
| ZONA | suscriptor.ruta_lectura[1, 2] |
| TIPO_DE_RED | cadena_electrica.tipo_instalacion |
| PARAMETRO_NIVEL_OPERATIVO_3 | TIPO_CUADRILLA |
| VALOR_NIVEL_OPERATIVO_3 | PROPIA o CONTRATADA (ver Nota 3) |
| **MEDIDORS: Request.DATOS_COMUNES_PROCESOS_TDC.MEDIDORS** | Pueden tener más de un medidor |
| MARCA_MEDIDOR | conta.marca_contador |
| MODELO_MEDIDOR | conta.modelo_contador |
| NUMERO_MEDIDOR | conta.numero_contador |
| TIPO_MEDIDOR | TRIFASICO |
| UBICACION_DEL_MEDIDOR | suscriptor.inf_adicional |
| VALOR_TENSION_NOMINAL | Valor según ?????? (Ver Nota 5) |
| **SELLOS: Request.DATOS_COMUNES_PROCESOS_TDC.SELLOS** | Pueden tener más de un precinto |
| COLOR_DEL_SELLO | **???????** |
| SELLOS_EN_SISTEMA | suscriptor.selloX (Ver Nota 6) |
| TIPO_DE_SELLO | **???????** |
| UBICACION_DEL_SELLO | No se envía nada |
| SERIE_SELLO | **???????** |
| **SUMINSTROS: Request.DATOS_COMUNES_PROCESOS_TDC.SUMINISTROS** | |
| ANTIGUEDAD | suscriptor.antiguedad_saldo * 30 |
| CODIGO_CLIENTE | surep.numero_cliente | 
| CODIGO_POSTAL | suscriptor.codigo_postal |
| DEUDA | surep.saldo_actual |
| GIRO_DE_NEGOCIO | codigo_ciiu.descripcion para codigo_ciiu = <suscriptor.codigo_ciu> |
| LATITUD_CLIENTE | cadena_electrica.coord_x_gm  |
| LONGITUD_CLIENTE | cadena_electrica.coord_y_gm |
| LOCALIDAD | suscriptor.localidad si suscriptor.provincia = "2"; ????????? si suscriptor.provincia = "1" |
| LOCALIZACION_TERRENO | suscriptor.dir_adicional |
| NOMBRE_Y_APELLIDO_CLIENTE | suscriptor.nombre |
| PISO | suscriptor.dir_piso + " " + suscriptor.dir_departamento |
| PROVINCIA | 'BUENOS AIRES' si suscriptor.provincia = "2"; 'CAPITAL FEDERAL' si suscriptor.provincia = "1"  |
| RUTA_DE_LECTURA | suscriptor.ruta_lectura |
| SUCURSAL | suscriptor.ruta_lectura[0,2] |
| TARIFA_EXISTENTE | suscriptor.tarifa |
| TELEFONO_CONTACTO | suscriptor.telefono |
| TEXTO_DIRECCION | suscriptor.dir_calle + " " + suscriptor.dir_numero + " (entre " + suscriptor.dir_interseccion + " y " + suscriptor.dir_interseccion2 + ")" -- Se podría cambiar si no vienen completos los campos de entre calle.|


3. Con la estructura cargada, consumir el WS de TIBCO P001C_PeticionCreacionTDC_ARG.wsdl. 

4. Evaluar el resultado que retorna el WS: 

* Si el Código de Error/Resultado es "00" ("Operación ejecutada con éxito"): actualizar el campo ... a "...".
* Si el Código es "ODL002" ("Clave de identificación del TdC ya presiente en eOrder") o "ODL016" ("Intento de insertar a la misma vez el mismo TDC 2 veces"): la orden ya existe en eOrder. Registrar el evento y actualizar el campo ... a "...".
* En otro caso: error no manejable. Registrar el evento (enviar mail) pero no actualizar tablas en CANDELA.

