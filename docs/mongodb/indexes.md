---
sidebar_position: 4
---

# Índices

## Qué son los índices
Funcionan de manera similar al índice de un libro. La base de datos, en lugar de buscar en toda la colección, emplea una lista ordenada con referencias al contenido, lo que permite ir mucho más rápido.

De acuerdo a la documentación de MongoDB, los índices son *estructuras de datos especiales que almacenan una pequeña porción de los datos de la colección de tal forma que sea fácil de recorrer. El índice almacena el valor de un campo específico o de un conjunto de campos, ordenados por el valor de dichos campos. El orden de cada una de las entradas del índice permite realizar consultas de igualdad o rango de manera eficiente.*

Cuando ejecutamos una consulta, esta puede apoyarse en índices o no hacerlo. Si no existe ningún índice en la colección que Mongo pueda usar para ejecutar la consulta, tendrá que hacer un **collection scan** (COLLSCAN). Por el contrario, si existe algún índice y lo puede usar mongo llevará a cabo un **index scan** (IXSCAN).

## explain()

Es la herramienta básica de que disponemos en Mongo para obtener datos acerca del funcionamiento de los índices:

```mongodb
db.books.find({title: "Zinky Boys"}).explain("queryPlanner")    // no ejecuta la query
{
  explainVersion: '1',
  queryPlanner: {
    namespace: 'bibliopolis.books',
    indexFilterSet: false,
    parsedQuery: {
      title: {
        '$eq': 'Zinky Boys'
      }
    },
    queryHash: '244E9C29',
    planCacheKey: '244E9C29',
    maxIndexedOrSolutionsReached: false,
    maxIndexedAndSolutionsReached: false,
    maxScansToExplodeReached: false,
    winningPlan: {
      stage: 'COLLSCAN',
      filter: {
        title: {
          '$eq': 'Zinky Boys'
        }
      },
      direction: 'forward'
    },
    rejectedPlans: []
  },
  command: {
    find: 'books',
    filter: {
      title: 'Zinky Boys'
    },
    '$db': 'bibliopolis'
  },
  serverInfo: {
    host: '6d0297d6b417',
    port: 27017,
    version: '6.0.16',
    gitVersion: '1bbe71e91a41b097b19d036dee47b861b3f27181'
  },
  serverParameters: {
    internalQueryFacetBufferSizeBytes: 104857600,
    internalQueryFacetMaxOutputDocSizeBytes: 104857600,
    internalLookupStageIntermediateDocumentMaxSizeBytes: 104857600,
    internalDocumentSourceGroupMaxMemoryBytes: 104857600,
    internalQueryMaxBlockingSortMemoryUsageBytes: 104857600,
    internalQueryProhibitBlockingMergeOnMongoS: 0,
    internalQueryMaxAddToSetBytes: 104857600,
    internalDocumentSourceSetWindowFieldsMaxMemoryBytes: 104857600
  },
  ok: 1,
  '$clusterTime': {
    clusterTime: Timestamp({ t: 1745446658, i: 1 }),
    signature: {
      hash: Binary(Buffer.from("0000000000000000000000000000000000000000", "hex"), 0),
      keyId: 0
    }
  },
  operationTime: Timestamp({ t: 1745446658, i: 1 })
}
```

En lo que debemos fijarnos de este resultado es en el `winningPlan`. Cuando haces una consulta con `.explain()`, MongoDB analiza cómo va a ejecutar esa consulta. Si hay varias formas posibles de ejecutarla (por ejemplo, usar un índice A, un índice B, o escanear toda la colección), MongoDB elige la más eficiente. Esa estrategia ganadora es lo que aparece en `winningPlan`. En este caso vemos que tendrá que hacer un COLLSCAN, ya que no hay ningún índice en que pueda apoyarse mongodb para ejecutar la consulta. Es importante señalar que la estructura de datos devuelta en el `winningPlan` (en árbol) se lee de abajo hacia arriba. Por ejemplo:

```
winningPlan: {
  stage: 'SORT',
  sortPattern: {
    title: 1
  },
  memLimit: 104857600,
  type: 'simple',
  inputStage: {
    stage: 'COLLSCAN',
    direction: 'forward'
  }
},
```

El orden de ejecución es primero el COLLSCAN y luego el SORT.

Podemos obtener más información haciendo `explain("executionStats")`:

```mongodb
db.books.find({title: "Zinky Boys"}).explain("executonStats")    // sí ejecuta la query
{
  explainVersion: '1',
  queryPlanner: {
    namespace: 'bibliopolis.books',
    indexFilterSet: false,
    parsedQuery: {
      title: {
        '$eq': 'Zinky Boys'
      }
    },
    queryHash: '244E9C29',
    planCacheKey: '244E9C29',
    maxIndexedOrSolutionsReached: false,
    maxIndexedAndSolutionsReached: false,
    maxScansToExplodeReached: false,
    winningPlan: {
      stage: 'COLLSCAN',
      filter: {
        title: {
          '$eq': 'Zinky Boys'
        }
      },
      direction: 'forward'
    },
    rejectedPlans: []
  },
  executionStats: {
    executionSuccess: true,
    nReturned: 1,
    executionTimeMillis: 0,
    totalKeysExamined: 0,
    totalDocsExamined: 104,
    executionStages: {
      stage: 'COLLSCAN',
      filter: {
        title: {
          '$eq': 'Zinky Boys'
        }
      },
      nReturned: 1,
      executionTimeMillisEstimate: 0,
      works: 105,
      advanced: 1,
      needTime: 103,
      needYield: 0,
      saveState: 0,
      restoreState: 0,
      isEOF: 1,
      direction: 'forward',
      docsExamined: 104
    }
  },
  command: {
    find: 'books',
    filter: {
      title: 'Zinky Boys'
    },
    '$db': 'bibliopolis'
  },
  serverInfo: {
    host: '6d0297d6b417',
    port: 27017,
    version: '6.0.16',
    gitVersion: '1bbe71e91a41b097b19d036dee47b861b3f27181'
  },
  serverParameters: {
    internalQueryFacetBufferSizeBytes: 104857600,
    internalQueryFacetMaxOutputDocSizeBytes: 104857600,
    internalLookupStageIntermediateDocumentMaxSizeBytes: 104857600,
    internalDocumentSourceGroupMaxMemoryBytes: 104857600,
    internalQueryMaxBlockingSortMemoryUsageBytes: 104857600,
    internalQueryProhibitBlockingMergeOnMongoS: 0,
    internalQueryMaxAddToSetBytes: 104857600,
    internalDocumentSourceSetWindowFieldsMaxMemoryBytes: 104857600
  },
  ok: 1,
  '$clusterTime': {
    clusterTime: Timestamp({ t: 1745446738, i: 1 }),
    signature: {
      hash: Binary(Buffer.from("0000000000000000000000000000000000000000", "hex"), 0),
      keyId: 0
    }
  },
  operationTime: Timestamp({ t: 1745446738, i: 1 })
}
```
Vemos que tenemos más información sobre la ejecución de la query dentro de `executionStats`. Lo importante de esta sección es el ratio totalDocsExamined vs nReturned. En este caso podemos ver que Mongo ha tenido que escanear toda la colección, de 105 documentos para recuperar únicamente uno. Las queries más optimizadas son aquellas en las que el total de documentos examinados *totalDocsExamined* y el *nReturned* se acercan al mismo número. No siempre es posible, así que deberíamos priorizar que esos números sean similares en aquellas queries que usamos más frecuentemente.

## Propiedades de los índices

## Tipos de índices

## Índices simples
Son aquellos índices que utilizan únicamente un campo del documento:

```mongodb
db.collection.createIndex({"title": 1})
```

MongoDB coge todos los documentos de la colección y saca el campo que hayamos especificado (title). Si la entrada no está presente en el documento, se le asigna un valor null.
Es posible emplear la notación del punto para crear índices sobre campos en subdocumentos:

```mongodb
db.movies.createIndex({"data.subdocument_field": 1})
```

Sin embargo, deberíamos evitar crear un documento sobre el campo que es en si un documento entero, ya que entonces solo se podrá emplear el indice cuando estemos consultando por el subdocumento completo, y no por alguno de sus campos. Por ejemplo, para documentos según este esquema:

```json
{
    "name": "Alice",
    "gpa": 3.6,
    "location": { city: "Sacramento", state: "California" }
}
```

Si creamos el índice:
```mongodb
db.collection.createIndex({location: 1})
```

La siguiente query podrá usar el índice:
```
db.students.find( { location: { city: "Sacramento", state: "California" } } )
```

Pero estas no podrán usarlo:
```mongodb
db.students.find( { "location.city": "Sacramento" } )
db.students.find( { "location.state": "New York" } )
```

Es posible usar índices en campos cuantitativos:
```mongodb
db.listingAndReviews.find({bedrooms: {$gte: 2, $lt: 4}})
```

## Índices compuestos
Son aquellos que emplean más de una clave. Este tipo de índices son útiles cuando en nuestra query empleamos más de un campo. Por ejemplo:
```mongodb
db.users.createIndex({"age" : 1, "username" : 1})
```
Hace más rápida la ordenación de documentos, de hecho incluso pueden emplearse únicamente con este fin. Sin embargo, hay que tener en cuenta que las claves de ordenación deben figurar en el mismo orden en que aparecen en el índice. Por ejemplo, el indice: `{a: 1, b: 1}` puede emplearse para ordenar por `{a:1, b:1}` y `{a:-1, b:-1}` pero no para `{b:1, a:1}` ni `{b:-1, a:-1}`. La dirección, como has podido ver, sí puede ser la inversa al indice.

Si las claves de la ordenación corresponden a las claves del índice o a un prefijo, MongoDB puede emplear el índice para ordenar los resultados de la consulta. Un prefijo de un índice compuesto es un subconjunto que consiste en una o más claves del inicio de las claves del índice. Por ejemplo:

```mongodb
db.data.createIndex( { a:1, b: 1, c: 1, d: 1 } )
```

Serían prefijos para ese índice:

```
{ a: 1 }
{ a: 1, b: 1 }
{ a: 1, b: 1, c: 1 }
```

Por tanto, las siguientes operaciones pueden realizarse sin necesidad de un ordenamiento en memoria:

| Ejemplo                                                   | Prefijo de índice           |
|-----------------------------------------------------------|-----------------------------|
| `db.data.find().sort( { a: 1 } )`                         | `{ a: 1 }`                  |
| `db.data.find().sort( { a: -1 } )`                        | `{ a: 1 }`                  |
| `db.data.find().sort( { a: 1, b: 1 } )`                   | `{ a: 1, b: 1 }`            |
| `db.data.find().sort( { a: -1, b: -1 } )`                 | `{ a: 1, b: 1 }`            |
| `db.data.find().sort( { a: 1, b: 1, c: 1 } )`             | `{ a: 1, b: 1, c: 1 }`      |
| `db.data.find({a: 4, b: 3}).sort( { c: 1 } )`             | `{ a: 1, b: 1, c: 1 }`      |
| `db.data.find( { a: { $gt: 4 } } ).sort( { a: 1, b: 1 } )`| `{ a: 1, b: 1 }`            |

Es posible que un índice sea empleado en la etapa de ordenación sin necesidad de utilizar un subconjunto de prefijo siempre y cuando se incluya una condición de igualdad que contenga todas las claves que preceden las claves de ordenación. De este modo, si tenemos el indice:

```
{ a: 1, b: 1, c: 1, d: 1 }
```

Las siguientes operaciones pueden utilizar el índince para la etapa de ordenación:

| Consulta                                                   | Prefijo de índice           |
|-----------------------------------------------------------|-----------------------------|
| `db.data.find( { a: 5 } ).sort( { b: 1, c: 1 } )`         | `{ a: 1, b: 1, c: 1 }`      |
| `db.data.find( { b: 3, a: 4 } ).sort( { c: 1 } )`         | `{ a: 1, b: 1, c: 1 }`      |
| `db.data.find( { a: 5, b: { $lt: 3} } ).sort( { b: 1 } )` | `{ a: 1, b: 1 }`            |

Pero estas no:
| Consulta                                                  | Motivo                      |
|-----------------------------------------------------------|-----------------------------|
| `db.data.find( { a: { $gt: 2 } } ).sort( { c: 1 } )`      | Falta b                     |
| `db.data.find( { c: 5 } ).sort( { c: 1 } )`               | Falta a y b                 |
| `db.data.find( { a: 5 } ).sort( { c: 1 } )`               | Falta b            |


## Índices multiclave

## Ordenación con índices

Es posible emplear índices para ordenar documentos en nuestras queries. Cualquier query puede ser ordenada luego en base a algún campo:

```mongodb
db.people.find({first_name: "James"}).sort({first_name: 1})
```

Existen dos métodos de ordenación en MongoDB: *en memoria* y *empleando un índice*.

### Ordenación en memoria
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

### Empleando un índice
Si una query realiza un index scan, el orden de los documentos devueltos está garantizado por el orden de la clave del índice (1 o -1). Esto significa que no hay necesidad de realizar una ordenamiento explícito ya que los documentos son recuperados del servidor en orden. Pero recuerda que únicamente estarán ordenados por aquellos campos que se emplearon a la hora de crear el índice. Si por ejemplo, creamos el índice:

```
db.people.createIndex({first_name: 1})
```

Cuando recuperemos los documentos empleando en nuestra query first_name, los documentos devueltos estarán ordenados únicamente por ese campo.

Los índices pueden usarse exclusivamente para ordenar, aunque no exista etapa de filtrado, si tenemos un índice `{name: 1}`:

```mongodb
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

## Optimización de índices

### Regla ESR (Equality, Sorting, Range)

### Covered Queries