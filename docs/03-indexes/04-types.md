---
sidebar_position: 3
---

# Tipos de índices

## Índices simples
Son aquellos índices que utilizan únicamente un campo del documento:

```js
db.collection.createIndex({"title": 1})
```

MongoDB coge todos los documentos de la colección y saca el campo que hayamos especificado (title). Si la entrada no está presente en el documento, se le asigna un valor null.
Es posible emplear la notación del punto para crear índices sobre campos en subdocumentos:

```js
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
```js
db.collection.createIndex({location: 1})
```

La siguiente query podrá usar el índice:
```
db.students.find( { location: { city: "Sacramento", state: "California" } } )
```

Pero estas no podrán usarlo:
```js
db.students.find( { "location.city": "Sacramento" } )
db.students.find( { "location.state": "New York" } )
```

Es posible usar índices en campos cuantitativos:
```js
db.listingAndReviews.find({bedrooms: {$gte: 2, $lt: 4}})
```

## Índices compuestos
Son aquellos que emplean más de una clave. Este tipo de índices son útiles cuando en nuestra query empleamos más de un campo. Por ejemplo:
```js
db.users.createIndex({"age" : 1, "username" : 1})
```
Hace más rápida la ordenación de documentos, de hecho incluso pueden emplearse únicamente con este fin. Sin embargo, hay que tener en cuenta que las claves de ordenación deben figurar en el mismo orden en que aparecen en el índice. Por ejemplo, el indice: `{a: 1, b: 1}` puede emplearse para ordenar por `{a:1, b:1}` y `{a:-1, b:-1}` pero no para `{b:1, a:1}` ni `{b:-1, a:-1}`. La dirección, como has podido ver, sí puede ser la inversa al indice.

Si las claves de la ordenación corresponden a las claves del índice o a un prefijo, MongoDB puede emplear el índice para ordenar los resultados de la consulta. Un prefijo de un índice compuesto es un subconjunto que consiste en una o más claves del inicio de las claves del índice. Por ejemplo:

```js
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