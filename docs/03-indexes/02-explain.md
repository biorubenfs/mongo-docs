---
sidebar_position: 2
---

# explain()

Es la herramienta básica de que disponemos en Mongo para obtener datos acerca del funcionamiento de los índices:

```js
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

```js
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
