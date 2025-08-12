---
sidebar_position: 5
---

# Patrones de diseño avanzados

## Single Collection Pattern

Agrupa documentos relacionados de diferentes tipos en una única colección. Es útil para modelar relaciones m:m cuando embeber los documentos no es una opción. También se puede emplear para modelar relaciones 1:m. Este patrón puede implementarse de diversas formas.

Por ejemplo, supongamos tres entidades: reviews, books y users. Si estas tres entidades se van a consultar con mucha frecuencia, convendría embeber todos los datos es un mismo documento, pero esto no es un opción ya que daría lugar a los unbounded arrays (arrays muy grandes, sin limite). Pero si empleamos una colección para cada entidad, entonces tendremos 3 colecciones y tendremos que realizar frecuentes lookups que son costosos para recuperar la información e impactaría el rendimiento de la aplicación. Por ello, conviene evitar multiples queries para leer documentos relacionados no embebidos.

### Variante de relaciones m:m 

*reviews*
```json
{
  review_id: 378,
  docType: "review"
  relatedTo: [202356]
}
```

*users*
```json
{
  user_id: 1,
  docType: "review"
  relatedTo: [202356]
}
```

*books*
```json
{
  book_id: 202356,
  docType: "review"
  relatedTo: [202356]
}
```

Luego crearíamos un índice para `relatedTo`.

### Variante de relaciones 1:m

Añadiendo un campo overload `sc_id` 

*books*
```json
{
  book_id: 202356
  sc_id: 202356
}
```

*reviews*
```json
{
  review_id: 378
  sc_id: 202356/378
}
```

Que se consultaría con algún tipo de `regex`.


## Subset Pattern
Reduce el tamaño total de los documentos que son accedidos frecuentemente por las aplicaciones. El problema que soluciona es que es posible que al recuperar un documento estemos recuperando y sobrecargando la memoria con mucha información que en realidad no necesitamos.

Por ejemplo, si dentro de un documento *book*, tenemos un campo reviews que es un array de reviews. Este array puede alcanzar cientos o miles de elementos, pero en realidad solo unos poco son necesarios, ya que nuestra aplicación solo muestra 3. La aplicación del patrón conllevaría a dejar únicamente 3 reviews en *book* y situar todas las reviews en una colección separada.

```json
{
  book_id: "Loop Tales",
  ...
  reviews: [
    {
      userId: "",
      comment: ""
    }
    ...
  ]
}
```

## Bucket Pattern
Especialmente útil en el IoT cuando se recogen datos cada cierto tiempo a intervalos muy pequeños. Por ejemplo:

```json
[
  {
     sensor_id: 12345,
     timestamp: ISODate("2019-01-31T10:00:00.000Z"),
     temperature: 40
  },
  {
     sensor_id: 12345,
     timestamp: ISODate("2019-01-31T10:01:00.000Z"),
     temperature: 40
  },
  {
     sensor_id: 12345,
     timestamp: ISODate("2019-01-31T10:02:00.000Z"),
     temperature: 41
  }
]
```

Esto puede plantear algunos problemas a medida que nuestra aplicación escala en términos de datos y tamaño del índice. Por ejemplo, podríamos terminar teniendo que indexar sensor_idy timestamp para cada medición para permitir un acceso rápido, a costa de la RAM. Sin embargo, al aprovechar el modelo de datos del documento, podemos agrupar estos datos, por tiempo, en documentos que contienen las mediciones de un período específico. También podemos agregar información adicional mediante programación a cada uno de estos "grupos".


Al usar el bucket pattern pasaríamos a tener algo así:

```json
{
  sensor_id: 12345,
  start_date: ISODate("2019-01-31T10:00:00.000Z"),
  end_date: ISODate("2019-01-31T10:59:59.000Z"),
  measurements: [
    {
      timestamp: ISODate("2019-01-31T10:00:00.000Z"),
      temperature: 40
    },
    {
      timestamp: ISODate("2019-01-31T10:01:00.000Z"),
      temperature: 40
    },
    … 
    {
      timestamp: ISODate("2019-01-31T10:42:00.000Z"),
      temperature: 42
    }
 ],
 transaction_count: 42,
 sum_temperature: 2413
} 
```

En este caso podemos agrupar datos en bloques de 1 hora.

Al usar el Patrón Bucket, hemos agrupado nuestros datos, en este caso, en un bucket de una hora. Este flujo de datos en particular seguiría creciendo, ya que actualmente solo tiene 42 mediciones; aún quedan más mediciones para esa hora por agregar al cubo. Al agregarlas al array de `measurements`, `transaction_count` se incrementarán y `sum_temperature` también se actualizarán.

Con el valor preagregado `sum_temperature`, es posible extraer fácilmente un segmento específico y determinar su temperatura promedio ( `sum_temperature / transaction_count`). Al trabajar con datos de series temporales, suele ser más interesante e importante saber cuál fue la temperatura promedio entre las 14:00 y las 15:00 h en Corning, California, el 13 de julio de 2018 que saber cuál fue la temperatura a las 14:03 h. Al agrupar y preagregar, podemos proporcionar esa información con mayor facilidad.

Además, a medida que recopilamos más información, podemos determinar que mantener todos los datos fuente en un archivo es más eficaz. Por ejemplo, ¿con qué frecuencia necesitamos acceder a la temperatura de Corning de 1948? Poder trasladar esos datos a un archivo de datos puede ser una gran ventaja.


## Outlier Pattern

Trata documentos con características poco comunes de forma algo distinta. Al ser casos extremos y poco frecuentes, no está justificado cambiar el modelo de datos, dado que esto haría que el rendimiento generalizado de la aplicación se viera afectado negativamente

Para casos en los que puede existir un array sin límites en determinados documentos. Por ejemplo, dado el siguiente esquema que funciona bien para la mayoría de libros:

```json
{
    "_id": ObjectID("507f1f77bcf86cd799439011")
    "title": "A Genealogical Record of a Line of Alger",
    "author": "Ken W. Alger",
    …,
    "customers_purchased": ["user00", "user01", "user02"]
}
```

Pero que eventualmente, puede haber un superventas que trastoque todo el esquema y afecta al rendimiento, al tratarse de un array sin límites (*unbounded array*):

```json
{
    "_id": ObjectID("507f191e810c19729de860ea"),
    "title": "Harry Potter, the Next Chapter",
    "author": "J.K. Rowling",
    …,
   "customers_purchased": ["user00", "user01", "user02", …, "user999"],
   "has_extras": "true"
}
```

Se marca el documento con algún tipo de flag y se mueven la información "excedente" a otro documento separado vinculado con el id. Al recuperar el documento, cuando tiene el flag, la aplicación obtendría el resto de información de la otra colección.

## Archive Pattern
Pretende sacar de la base de datos principal (o de la colección) documentos que son consultados con muy poca frecuencia pero que debemos mantener por algún motivo. Su mantenimiento en una colección de consulta frecuente degrada el rendimiento generalizado de la aplicación.

Se puede hacer uso de servicios propios de Mongo Atlas.

## Recursos

- [Single Reference Pattern](https://www.mongodb.com/developer/products/mongodb/single-collection-pattern/)
- [Outlier Pattern](https://www.mongodb.com/blog/post/building-with-patterns-the-outlier-pattern)
- [Archived Pattern](https://www.mongodb.com/docs/manual/data-modeling/design-patterns/archive/)
- [Subset Pattern](https://www.mongodb.com/blog/post/building-with-patterns-the-subset-pattern)
- [Bucket Pattern](https://www.mongodb.com/blog/post/building-with-patterns-the-bucket-pattern)