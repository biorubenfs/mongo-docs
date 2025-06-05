---
sidebar_position: 4
---

# Ordenación con índices

Es posible emplear índices para ordenar documentos en nuestras queries. Cualquier query puede ser ordenada luego en base a algún campo:

```js
db.people.find({first_name: "James"}).sort({first_name: 1})
```

Existen dos métodos de ordenación en MongoDB: *en memoria* y *empleando un índice*.

## Ordenación en memoria
Los documentos de nuestra colección están almacenados en disco en un orden desconocido. Cuando realizamos la petición al servidor para que nos devuelva los documentos, nos los devolverá en el mismo orden en el que los encuentra. Habitualmente, tenemos que agregar una etapa de ordenamiento para obtener los documentos en el orden que deseamos. Así, el servidor vuelca los documentos del disco a la RAM y es aquí en la RAM en donde se lleva a cabo la operación de ordenamiento basándose en algún algoritmo. En función del número de documentos que tengamos esta operación puede tardar.

```
{
 "stage": "SORT",
 "nReturned": 0,
 "executionTimeMillisEstimate": 1,
 "works": 1577,
 "advanced": 0,
 "needTime": 1576,
 "needYield": 0,
 "saveState": 1,
 "restoreState": 1,
 "failed": true,
 "isEOF": 0,
 "sortPattern": {
  "name": 1
 },
 "memLimit": 33554432,
 "type": "simple",
 "totalDataSizeSorted": 0,
 "usedDisk": false
}
```

Como esta operación de ordenación en memoria del servidor puede requerir una alta demanda de recursos, el servidor puede abortar la operación si se emplean más de 32Mb.

```
executionStats: {
    executionSuccess: false,
    errorMessage: 'Sort exceeded memory limit of 33554432 ' +
      'bytes, but did not opt in to external ' +
      'sorting.',
}
```

## Empleando un índice
Si una query realiza un index scan, el orden de los documentos devueltos está garantizado por el orden de la clave del índice (1 o -1). Esto significa que no hay necesidad de realizar una ordenamiento explícito ya que los documentos son recuperados del servidor en orden. Pero recuerda que únicamente estarán ordenados por aquellos campos que se emplearon a la hora de crear el índice. Si por ejemplo, creamos el índice:

```
db.people.createIndex({first_name: 1})
```

Cuando recuperemos los documentos empleando en nuestra query first_name, los documentos devueltos estarán ordenados únicamente por ese campo.

Los índices pueden usarse exclusivamente para ordenar, aunque no exista etapa de filtrado, si tenemos un índice `{name: 1}`:

```js
db.listingAndReviews.find({}).sort({name: 1})
```
Podemos verlo en el explain():
```
Query Performance Summary
Documents Returned:5555
Index Keys Examined:5555
Documents Examined:5555
Actual Query Execution Time (ms):173
Sorted in Memory:no
Query used the following index: name_1
```
