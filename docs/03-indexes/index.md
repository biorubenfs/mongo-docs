---
sidebar_position: 3
---

# Índices

## Qué son los índices
Funcionan de manera similar al índice de un libro. La base de datos, en lugar de buscar en toda la colección, emplea una lista ordenada con referencias al contenido, lo que permite ir mucho más rápido.

De acuerdo a la documentación de MongoDB, los índices son *estructuras de datos especiales que almacenan una pequeña porción de los datos de la colección de tal forma que sea fácil de recorrer. El índice almacena el valor de un campo específico o de un conjunto de campos, ordenados por el valor de dichos campos. El orden de cada una de las entradas del índice permite realizar consultas de igualdad o rango de manera eficiente.*

Cuando ejecutamos una consulta, esta puede apoyarse en índices o no hacerlo. Si no existe ningún índice en la colección que Mongo pueda usar para ejecutar la consulta, tendrá que hacer un **collection scan** (COLLSCAN). Por el contrario, si existe algún índice y lo puede usar mongo llevará a cabo un **index scan** (IXSCAN).