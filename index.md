## 1.‐ ¡Bienvenido al Manual de Buenas Prácticas ABAP IMEGA VENTUS!

¡Bienvenido al Manual de Buenas Prácticas en ABAP! Este documento ha sido creado con el propósito de proporcionar una guía clara y estructurada para desarrollar aplicaciones y programas en ABAP de manera eficiente, segura y estandarizada.

En el desarrollo con ABAP (Advanced Business Application Programming), la calidad del código es crucial no solo para garantizar el correcto funcionamiento de las aplicaciones, sino también para mejorar la mantenibilidad, el rendimiento y la colaboración entre equipos de desarrollo. Este manual está diseñado para ayudarte a seguir un conjunto de recomendaciones que te permitirán escribir código más limpio, comprensible y eficaz.

A lo largo de este manual, encontrarás pautas detalladas sobre cómo estructurar tu código, realizar consultas de base de datos de forma óptima, manejar excepciones y modularizar correctamente tus programas. Nuestro objetivo es que estas prácticas no solo te guíen en tu día a día como desarrollador, sino que también te ayuden a mejorar continuamente tus habilidades, facilitando el mantenimiento de las soluciones a largo plazo.

Recuerda: El seguir estas buenas prácticas no solo beneficia tu propio trabajo, sino que también mejora la colaboración y el entendimiento entre los miembros de tu equipo. Un código bien estructurado y documentado es más fácil de comprender, extender y depurar.

Te invitamos a que consultes este manual en cada etapa de tu desarrollo, y esperamos que sea una herramienta valiosa para ti en la creación de aplicaciones robustas y de alto rendimiento en ABAP.

¡Gracias por tu dedicación al aprendizaje y la mejora continua!

## 2.‐ Estructura de Código

## Comentarios: 
Asegúrate de comentar el código donde sea necesario para que otros programadores puedan entenderlo fácilmente. Usa comentarios para describir la funcionalidad de bloques de código complejos o algoritmos importantes.

## Indentación y espaciado: 
Mantén una indentación coherente, utiliza los espacios o tabuladores para identificar diferentes niveles de código.

## Nombres de variables y objetos: 
Usa nombres descriptivos que representen el contenido o la funcionalidad. Evita usar abreviaturas a menos que sean comunes en el entorno de trabajo.

## Convención de nombres, Siguiendo las recomendaciones de SAP:
- Variables Globales: gt_, gv_, gwa_
- Variables locales: lv_, lt_, ls_
- Estructuras: ls_
- Tablas internas: lt_

## Funciones y métodos: 
Usa verbos para describir la acción (get_, set_, calculate_).

## 3.‐ Selección de Datos

## Evita el uso de __SELECT *__ : 
Especifica las columnas que realmente necesitas, ya que __SELECT *__ puede afectar el rendimiento al extraer datos innecesarios.

## Evitar el uso de __FOR ALL ENTRIES__: 
Esta sentencia puede causar duplicados si la tabla interna que se utiliza está vacía o si no se ha validado adecuadamente. En lugar de esto, usa otras alternativas como JOINS o subconsultas que sean más eficientes y seguras.


TEST
> **Advertencia:** Nunca uses SELECT * en producción sin una cláusula WHERE adecuada
> para evitar cargar toda la tabla de base de datos.
{: .note .note-warning}
TEST

> [!IMPORTANT]
> " Evitar __FOR ALL ENTRIES__: \
> __SELECT__ vbeln erdat \
>   __FROM__ vbak  \
>   __INTO TABLE__ lt_vbak \
>   __FOR ALL ENTRIES IN__ lt_kunnr 
>   __WHERE__ kunnr __EQ__ 'TEST'. 
>
> " Alternativa usando __INNER JOIN__: \
> __SELECT__ a.vbeln a.erdat \
>  __INTO TABLE__ lt_vbak \
>  __FROM__ vbak as a\
>  __INNER JOIN__ kna1 as b __ON__ a.kunnr = b.kunnr \
>  __WHERE__ b.kunnr __EQ__ 'TEST'. 
>

## No utilizar INTO CORRESPONDING FIELDS:

El uso de INTO CORRESPONDING FIELDS o INTO CORRESPONDING FIELDS TABLE añade una capa adicional de procesamiento que puede reducir el rendimiento y no es siempre necesario si se seleccionan explícitamente los campos que coinciden. Es preferible mapear las columnas directamente a las estructuras correspondientes.

> [!IMPORTANT]
> No usar: \
> __SELECT__ vbeln erdat __INTO CORRESPONDING FIELDS OF__ ls_vbak __FROM__ vbak __WHERE__ vbeln = '123456'.
>
> Usar en su lugar:\
> __SELECT__ vbeln erdat __INTO__ (ls_vbak-vbeln, ls_vbak-erdat) __FROM__ vbak __WHERE__ vbeln = '123456'.
>

## Uso de Índices: 
Asegúrate de usar índices cuando hagas consultas en tablas grandes, para mejorar el rendimiento de las consultas.

## Control de Errores: 
Siempre verifica si las consultas devuelven resultados válidos antes de seguir con el proceso.

> [!IMPORTANT]
> __IF__ lt_vbak __IS INITIAL__. \
>  __MESSAGE__ 'No se encontraron pedidos de ventas para la fecha actual' __TYPE__ 'I'. \
> __ELSE__. \
>  "Procesar los resultados" \
> __ENDIF__.


## 4.‐ Declaración de Variables

## Declaración local: 
Siempre que sea posible, declara variables dentro de la función o el método donde se van a usar. Evita declarar variables globales innecesarias.

> [!TIP]
 **FORM** calculate_discount. \
  **DATA**: lv_discount **TYPE** p **DECIMALS** 2. \
  "... \
> **ENDFORM**.

## Inicialización: 
Evita asumir que las variables están inicializadas. Asegúrate de asignarles un valor inicial si es necesario.

> [!TIP]
 **REFRESH** lt_sales_orders.  \
> **CLEAR** ls_sales_orders.


## Evitar la declaración de variables fuera de contexto: 
Declara las variables solo en el lugar donde se necesiten. Mantener las variables dentro del ámbito necesario mejora la mantenibilidad del código y reduce el riesgo de errores inesperados.

> [!CAUTION]
> Usar variable lv_temp fuera de la instrucción **IF** puede ocacionar **DUMP**.
>
> **IF** SY-SUBRC **EQ** 0. \
> **DATA**: LV_TEMP **TYPE** CHAR1. \
> .... \
> **ENDIF**. \
>
> **CASE** LV_TEMP. \
> **WHEN** .... 
> **ENDCASE** . \

## Evitar la declaración de datos sin uso: 

Nunca declares variables que no vas a utilizar. La presencia de variables no utilizadas en el código complica la legibilidad y puede generar confusión.


## 5.‐ Performance de Código

## Modularización del Código:
Uso de funciones y métodos: Divide el código en bloques funcionales pequeños utilizando subrutinas (FORM), funciones (FUNCTION MODULES) y métodos (en clases ABAP). Esto permite una mejor organización, reutilización y mantenimiento del código.

> [!CAUTION]
>  **Ejemplo de mala práctica**: \
> Código extenso en una sola subrutina que realiza múltiples tareas:
>

~~~
FORM process_sales_order.

  SELECT (campos) FROM vbak INTO TABLE lt_vbak WHERE erdat = sy-datum.

  LOOP AT lt_vbak INTO ls_vbak. 
    " Cálculos complejos 
    " Actualización de datos 
  ENDLOOP.

ENDFORM.
~~~


> [!TIP]
> **Ejemplo de buena práctica:**
>
> Modularización en diferentes subrutinas o funciones para cada tarea específica:

~~~
FORM select_sales_orders. 
 SELECT (campos) FROM vbak INTO TABLE lt_vbak WHERE erdat = sy-datum. 
ENDFORM.

FORM process_sales_order. 
  LOOP AT lt_vbak INTO ls_vbak. 
    PERFORM calculate_order. 
   PERFORM update_order. 

  ENDLOOP.

ENDFORM. 

FORM calculate_order.
 " Lógica de cálculo
ENDFORM.

FORM update_order.
 " Lógica de actualización 
ENDFORM.

~~~


## Uso de Clases y Objetos: 
Implementar un enfoque orientado a objetos (OO) cuando sea posible, favorece la reutilización de código, encapsulamiento y escalabilidad. Las clases permiten agrupar atributos y comportamientos relacionados, facilitando el mantenimiento.

~~~
CLASS lcl_sales_order DEFINITION.
  PUBLIC SECTION.
    METHODS: calculate_total_amount, update_order_status.
ENDCLASS.
~~~


## Manejo de Excepciones y Errores
Uso adecuado de TRY...CATCH: Es fundamental que las excepciones se manejen de forma adecuada para evitar que el sistema falle sin control. Usa bloques TRY...CATCH para capturar errores y proporcionar mensajes claros.

~~~
TRY.
    CALL FUNCTION 'Z_CALCULATE_DISCOUNT'.
  CATCH cx_sy_arithmetic_error INTO lx_arith_error.
    MESSAGE 'Error en el cálculo del descuento' TYPE 'E'.
ENDTRY.
~~~


## Optimización y Rendimiento

### Optimización de consultas: 
Siempre busca optimizar las consultas de base de datos. Evita el uso de SELECT * y selecciona solo los campos necesarios. Usa JOINS en lugar de realizar múltiples consultas independientes cuando trabajes con múltiples tablas relacionadas.

~~~
SELECT a~vbeln, a~erdat, b~name1
  INTO TABLE lt_vbak_knas
  FROM vbak AS a
  INNER JOIN kna1 AS b ON a~kunnr = b~kunnr
  WHERE a~erdat = sy-datum.
~~~

### Reducir operaciones costosas dentro de bucles: 
Evita hacer operaciones costosas, como consultas a la base de datos o cálculos complejos, dentro de bucles grandes. Estas operaciones deben hacerse fuera del bucle siempre que sea posible para mejorar el rendimiento.
> [!CAUTION]
> Ejemplo de mala práctica:
~~~
LOOP AT lt_vbak INTO ls_vbak.
  SELECT * FROM vbap WHERE vbeln = ls_vbak-vbeln INTO TABLE lt_vbap.
ENDLOOP.
~~~

> [!TIP]
> Ejemplo de buena práctica:

~~~
SELECT (campos) FROM vbap INTO TABLE lt_vbap FOR ALL ENTRIES IN lt_vbak WHERE vbeln = lt_vbak-vbeln.

LOOP AT lt_vbak INTO ls_vbak.
  " Procesar los datos de la tabla interna lt_vbap en lugar de hacer múltiples SELECTs
ENDLOOP.

~~~

### Manejo eficiente de tablas internas: 
Usa las operaciones sobre tablas internas de manera eficiente. Ordena y busca en tablas internas con claves para mejorar la velocidad de búsqueda.

~~~
SORT lt_vbak BY vbeln.
READ TABLE lt_vbak WITH KEY vbeln = '123456'.
~~~

## Control de Versiones y Documentación
**Versiones de Transporte** : Documenta adecuadamente cada transporte (ETF) y mantén un control riguroso de las versiones del código para facilitar la identificación y corrección de errores. 

~~~
* INI - USUARIO- FECHA - TICKET - DESCRIPCION TICKET
SELECT vbeln, erdat INTO TABLE lt_vbak FROM vbak WHERE erdat = sy-datum.
* FIN - USUARIO- FECHA - TICKET - DESCRIPCION TICKET
~~~
~~~
SELECT vbeln, erdat 
  INTO TABLE lt_vbak 
  FROM vbak 
  WHERE erdat = sy-datum
  AND STORN NE 'X'. * ADD - USUARIO - FECHA - TICKET - DESCRIPCION TICKET
~~~


**Documentación en el código**: Asegúrate de que el código esté bien documentado, especialmente en los puntos más complejos. Usa comentarios claros para describir la lógica y el propósito del código. La documentación también debe incluir detalles funcionales y técnicos, facilitando su entendimiento a otros desarrolladores.

~~~
*&---------------------------------------------------------------------*
*&      Form  SEND_EMAIL_V2
*&---------------------------------------------------------------------*
*       descripción del performs.
*       envia mail a casilla enviada por parametro de entrada
*
*----------------------------------------------------------------------*
*      -->p_email type char30 -> elemento de datos maximo 30 caracteres
*----------------------------------------------------------------------*
FORM send_email_v2  USING    p_email.
  "logica para envio
ENDFORM.

~~~


## 6. Auditoría de Código ABAP

   Para garantizar la calidad del código ABAP, SAP proporciona varias transacciones que permiten auditar el código en busca de errores, malas prácticas, problemas de rendimiento y violaciones de los estándares de desarrollo. Estas herramientas son fundamentales para asegurar que el código sea mantenible, eficiente y cumpla con las normativas de seguridad.


## Code Inspector – Transacción SCI:
Code Inspector es otra herramienta importante que permite revisar el código en busca de errores, inconsistencias y mejoras. Permite realizar análisis de forma manual y programada, aplicando reglas definidas por el usuario o reglas estándar de SAP. Si bien SCI y ATC tienen funcionalidades similares, SCI es más configurable para casos específicos.

### Principales características de SCI:

- Verificación de la calidad del código según una serie de controles predefinidos o personalizados.
- Análisis de consultas de base de datos, uso de memoria y rendimiento.
- Análisis de seguridad para detectar vulnerabilidades potenciales.


## Extended Syntax Check – Transacción SLIN:
SLIN realiza una verificación extendida de la sintaxis en programas ABAP. Esta herramienta ayuda a identificar errores de código que pueden pasar desapercibidos en la verificación estándar de sintaxis.

### Principales características de SLIN:

- Detección de errores de sintaxis, incluso aquellos que no generan errores inmediatos.
- Verificación del cumplimiento de estándares de programación.
- Análisis de código obsoleto o redundante.

## Importancia de la Auditoría de Código
Realizar auditorías periódicas del código con estas herramientas es esencial para:

- Mejorar el rendimiento: Identificar problemas que afectan la velocidad de ejecución o que consumen recursos de manera ineficiente.
- Seguridad del código: Detectar vulnerabilidades que podrían comprometer la integridad del sistema o los datos.
- Cumplimiento de estándares: Garantizar que el código siga las mejores prácticas y estándares corporativos o de SAP.
- Mantenibilidad: Facilitar el mantenimiento a largo plazo del código al garantizar que sea limpio, legible y libre de problemas potenciales.

