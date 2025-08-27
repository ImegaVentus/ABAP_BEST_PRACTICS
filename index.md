# 1. ¡Bienvenido al Manual de Buenas Prácticas ABAP IMEGA VENTUS!

¡Bienvenido al Manual de Buenas Prácticas en ABAP! Este documento ha sido creado con el propósito de proporcionar una guía clara y estructurada para desarrollar aplicaciones y programas en ABAP de manera eficiente, segura y estandarizada.

En el desarrollo con ABAP (Advanced Business Application Programming), la calidad del código es crucial no solo para garantizar el correcto funcionamiento de las aplicaciones, sino también para mejorar la mantenibilidad, el rendimiento y la colaboración entre equipos de desarrollo. Este manual está diseñado para ayudarte a seguir un conjunto de recomendaciones que te permitirán escribir código más limpio, comprensible y eficaz.

A lo largo de este manual, encontrarás pautas detalladas sobre cómo estructurar tu código, realizar consultas de base de datos de forma óptima, manejar excepciones y modularizar correctamente tus programas. Nuestro objetivo es que estas prácticas no solo te guíen en tu día a día como desarrollador, sino que también te ayuden a mejorar continuamente tus habilidades, facilitando el mantenimiento de las soluciones a largo plazo.

Recuerda: El seguir estas buenas prácticas no solo beneficia tu propio trabajo, sino que también mejora la colaboración y el entendimiento entre los miembros de tu equipo. Un código bien estructurado y documentado es más fácil de comprender, extender y depurar.

Te invitamos a que consultes este manual en cada etapa de tu desarrollo, y esperamos que sea una herramienta valiosa para ti en la creación de aplicaciones robustas y de alto rendimiento en ABAP.

¡Gracias por tu dedicación al aprendizaje y la mejora continua!

---

# 2. Estructura de Código

### Comentarios
Asegúrate de comentar el código donde sea necesario para que otros programadores puedan entenderlo fácilmente. Usa comentarios para describir la funcionalidad de bloques de código complejos o algoritmos importantes.

### Indentación y espaciado
Mantén una indentación coherente, utiliza los espacios o tabuladores para identificar diferentes niveles de código.

### Nombres de variables y objetos
Usa nombres descriptivos que representen el contenido o la funcionalidad. Evita usar abreviaturas a menos que sean comunes en el entorno de trabajo.

### Convención de nombres (recomendaciones SAP)
- Variables Globales: `gt_`, `gv_`, `gwa_`
- Variables locales: `lv_`, `lt_`, `ls_`
- Estructuras: `ls_`
- Tablas internas: `lt_`

### Funciones y métodos
Usa verbos para describir la acción (`get_`, `set_`, `calculate_`).

---

# 3. Selección de Datos

### Evita el uso de `SELECT *`
Especifica las columnas que realmente necesitas, ya que `SELECT *` puede afectar el rendimiento al extraer datos innecesarios.

### Evita `FOR ALL ENTRIES`
Puede causar duplicados si la tabla interna está vacía. Usa **JOINS** o subconsultas.

```abap
" Evitar FOR ALL ENTRIES
SELECT vbeln erdat
  INTO TABLE lt_vbak
  FROM vbak
  FOR ALL ENTRIES IN lt_kunnr
  WHERE kunnr = 'TEST'.

" Alternativa con INNER JOIN
SELECT a~vbeln a~erdat
  INTO TABLE lt_vbak
  FROM vbak AS a
  INNER JOIN kna1 AS b ON a~kunnr = b~kunnr
  WHERE b~kunnr = 'TEST'.
