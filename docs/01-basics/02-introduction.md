---
sidebar_position: 2
---

# Conceptos generales

MongoDB es una base de datos orientada a documentos. Su ventaja principal es que es más fácilmente escalable. 

Una base de datos orientada a documentos sustituye el concepto de "fila" por un modelo más flexible, el "documento". La posibilidad de embeber o incrustar (del inglés *embedding*) documentos y arrays hace posible la representación de relaciones jerárquicas complejas dentro de un único documento, lo que se ajusta más a la forma en la que trabajan hoy en día los desarrolladores con lenguajes orientados a objetos.

No existen esquemas predefinidos. No hay claves o valores con tipos o tamaños fijos. Esto hace que sea mucho más sencillo añadir o eliminar campos según las necesidades.

## Características

MongoDB es una base de datos de propósito general. Además de crear, leer, actualizar y borrar datos, permite también:

- **Indexación**: Soporta índices secundarios genéricos (índices primarios son aquellos creados sobre la clave automática `_id`), y ofrece capacidades de indexacion única, compuesta, geospacial y de texto completo. También admite índices secundarios en estructuras jerárquicas, como documentos anidados y arrays.
- **Agregación**: Framework de agregación para el procesamiento de datos basado en el concepto de pipelines de procesamiento de datos.
- **Tipos especiales de colecciones e índices**: Proporcionan colecciones de tamaño fijo (`capped`) y colecciones con índices TTL, con datos que se eliminan pasado cierto tiempo. También soporta índices parciales o condicionados.
- **Almacenamiento de ficheros**: Es posible almacenar ficheros en Mongo.

## Documentos

Un documento es un conjunto ordenado de claves asociadas con valores. Es el equivalente a un fila de una tabla SQL.
- **claves**: son siempre strings. Cualquier carácter UTF-8 se admite como clave a excepción de:
    - El carácter `\0` (caracter `null`). Se emplea para indicar el fin del string.
    - El punto `.` y símbolo del dólar `$` tienen propiedades especiales y deberían ser usados únicamente en determinadas circunstancias.
- **valores**: Pueden contener distintos tipos de datos.

MongoDB es *type-sensitive* and *case-sensitive* y tampoco se permiten claves duplicadas.

## Colecciones

Es un conjunto de documentos. Análogo a una tabla de SQL.

Las colecciones tienen esquemas dinámicos. Esto significa que los documentos dentro de una colección pueden tener diferente forma (diferentes numero de claves, diferentes claves y diferentes tipos de valores).

A la hora de dar nombre a las colecciones debemos tener en cuenta:
- El string vacio `""` no es un válido.
- No pueden contener el carácter `\0` (el carácter `null`) porque éste indica el fin del nombre de la colección.
- No se deberían crear colecciones con el nombre que empiece por system. MongoDB tiene colecciones internas que empiezan por este nombre.
- Las colecciones creadas por el usuaria no deberían contener en su nombre el carácter reservado $.

## Bases de datos

Las colecciones se agrupan en bases de datos. Una instancia de MongoDB pueden albergar varias bases de datos, cada una de ella con cero o más colecciones.

La base de datos puede nombrarse con UTF-8 son las siguientes restricciones:
- El string vacio `""` no es un válido.
- No pueden contener los caracteres `/`, `\`, `.`, `"`, `*`, `<`, `>`, `:`, `|`, `?`, `$`, (a single space), or `\0` (the null character).
- Los nombres de las bases de datos son case-insensitive.
- Los nombres están limitados a un máximo de 64 bytes.
- Existen algunos nombre de bases de datos que están reservados:
  - *admin*: para autenticación y autorización. Se necesita acceso a esta base de datos para tareas administrativas.
  - *local*: almacena datos específicos de un único servidor. En los replica sets, guarda los datos empleados en el proceso de replicación.
  - *config*: los clusters fragmentados emplean esta base de datos para almacenar la información de cada fragmento.
