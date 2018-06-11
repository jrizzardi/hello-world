# Básico de markdown

Markdown dispone de cuatro niveles de encabezados definidos por el número de `#` antes del texto del encabezado.
# Primer nivel de encabezado
## Segundo nivel de encabezado
### Tercer nivel de encabezado
#### Cuarto nivel de encabezado

El primer y segundo nivel de encabezado también se pueden escribir como sigue:

Primer nivel de encabezado
==========================

Segundo nivel de encabeado
--------------------------

Los saltos de línea sencillos deben indicarse con dos espacios en blanco en algunas implementaciones de Markdown.  
Los párrafos deben estar separados por una línea vacía.

El texto se puede poner en *cursivas* encerrándolo entre los símbolos * o -. De la misma forma, el texto en **negritas** se escribe encerrando la palabra entre doble asterisco o doble guión bajo. 

Lista de items (usar asteriscos y sangrías)
---------------
* Frutas
  * Manzanas
  * Naranjas
  * Uvas
* Lácteos
  * Leche
  * Queso
  
Las listas ordenadas se escriben numerando cada línea.  

Para generar listas anidadas dentro de otras, simplemente hay que añadir cuatro espacios en blanco antes del siguiente asterisco.

Lista de pendientes
------------------
1. Terminar el tutorial de Markdown
2. Ir a la tienda de abarrotes
3. Preparar el almuerzo

Representamos los fragmentos de código encerrados entre dos signos de acento grave. Por ejemplo: `<br/>`.

> Los bloques de citas se redactan con el signo mayor al comienzo. Le deja una sangría con respecto al margen izquierdo.
>> Se pueden anidar citas utilzando más de un signo mayor.

El título del enlace se encierra primero entre corchetes y después se incluye la dirección completa del URL entre paréntesis. O sea si querés ir a Google, tenés que presionar [acá](http:\\google.com).

# Tablas

| Encabezado 1 | Encabezado 2 | Encabezado 3 |
| --------- | --------- | --------- |
| renglón 1, columna 1 | renglón 1, columna 2 | renglón 1, columna 3|
| renglón 2, columna 1 | renglón 2, columna 2 | renglón 2, columna 3|
| renglón 3, columna 1 | renglón 3, columna 2 | renglón 3, columna 3|


Para especificar la alineación del contenido de cada columna se pueden agregar dos puntos al renglón de los encabezados como sigue:

| Alineado-izquierda | Centrado | Alineado-derecha |
| :-------- | :-------: | --------: |
| Manzanas | rojo | 5000 |
| Plátanos | amarillo | 75 |

### La barra invertida es el caracter de escape.
