---
sidebar_position: 4
---

# Patrones de diseño básicos

## Inheritance Pattern

En MongoDB, no existe herencia nativa entre documentos. El Inheritance Pattern consiste en almacenar diferentes tipos de entidades relacionadas (por ejemplo, una jerarquía de clases) en una misma colección, diferenciándolas mediante un campo tipo (`type` o `kind`) y almacenando los atributos específicos de cada tipo en el mismo documento.


### Ejemplo

Una tienda de libros usa este patrón para almacenar diferentes tipos de medios en la misma colección. La entidad padre `book` almacena campos comunes, como `title`o `author` y múltiples entidades hijas heredan de la entidad `book`. Por ejemplo, audio books, printed books y ebooks tienen campos comunes y también tienen campos únicos específicos de su media type. La aplicación de este patrón en este caso almacena en la misma colección estas entidades ligeramente diferentes, lo que mejora el rendimiento de las queries que necesitan acceder a todos los `book` independientemente de su tipo.

```javascript
db.books.insertMany( [
   {
      product_type: "ebook",
      title: "Practical MongoDB Aggregations",
      author: "Paul Done",
      rating: 4.8,
      genres: [ "programming" ],
      pages: 338,
      download_url: "<url>"
   },
   {
      product_type: "audiobook",
      title: "Practical MongoDB Aggregations",
      author: "Paul Done",
      rating: 4.6,
      genres: [ "programming" ],
      narrators: [ "Paul Done" ],
      duration: {
         hours: 21,
         minutes: 8
      },
      time_by_chapter: [
         {
            chapter: 1,
            start: "00:00:00",
            end: "01:00:00"
         },
         {
            chapter: 2,
            start: "01:00:00",
            end: "01:55:00"
         }
      ]
   },
   {
      product_type: "physical_book",
      title: "Practical MongoDB Aggregations",
      author: "Paul Done",
      rating: 4.9,
      genres: [ "programming" ],
      pages: 338,
      stock: 12,
      delivery_time: 2
   }
] )
```

Los documentos anteriores comparten una serie de campos comunes. Otros dependen del tipo del campo `product_type`. Por ejemplo:

- documentos de tipo `ebook` tienen un campo `download_url`.

- documentos de tipo `audiobook` tienen un campo `time_by_chapter`.

- documentos de tipo `physical_book` tienen un campo `delivery_time`.

#### Queries

Podemos recuperar todos los libros:

```mongodb
db.books.find()
```

O según su tipo:
```mongodb
db.books.find({product_type: "ebook"})
```

Y también podemos realizar consultas basándonos en campos específicos del tipo a que pertenece:
```javascript
db.books.find(
   {
      "duration.hours": { $gt: 20 }
   }
)
```
En este caso solo se devolverían documentos con `product_type` igual a `audio_book`.

## Computed Pattern

A menudo ciertos datos de utilidad no son almacenados en base de datos. En su lugar se pueden calcular a partir de los datos almacenados. ¿Cuál es el ingreso total por ventas del último Amazon Alexa? ¿Cuántos espectadores vieron la última película taquillera? Este tipo de preguntas pueden responderse a partir de los datos almacenados en una base de datos, pero deben calcularse. Es lo que se conoce como un valor calculado (computed field). Sin embargo este cálculo conlleva cierto a nivel de CPU, especialmente en aquellos en los que el data set es grande, lo que puede conllevar una degradación del rendimiento de la aplicación debido al tiempo que tarda la consulta. Para estos casos disponemos del **computed pattern**.

Si las lecturas son significativamente más comunes que las escrituras, el patrón calculado reduce la frecuencia del cálculo de datos. En lugar de calcular los valores en cada lectura, la aplicación almacena el valor calculado y lo recalcula según sea necesario. La aplicación puede volver a calcular el valor con cada escritura que cambie los datos de origen del valor calculado, o como parte de una tarea periódica.

Con las actualizaciones periódicas, no se garantiza que el valor calculado devuelto sea exacto. Sin embargo, este enfoque puede valer la pena por la mejora en el rendimiento si la precisión exacta no es un requisito (ver Approximation Pattern)

### Ejemplo

Supongamos una aplicación que muestra información sobre películas e ingresos. Los usuarios pueden buscar una película concreta y ver cuánto dinero ha recaudado:

```javascript
db.screenings.insertMany( [
   {
      theater: "Alger Cinema",
      location: "Lakeview, OR",
      movie_title: "Lost in the Shadows",
      movie_id: 1,
      num_viewers: 344,
      revenue: 3440
   },
   {
      theater: "City Cinema",
      location: "New York, NY",
      movie_title: "Lost in the Shadows",
      movie_id: 1,
      num_viewers: 1496,
      revenue: 22440
   },
] )
```

Si los usuarios desean conocer cuánta gente a visto determinada película y cuánto recaudó, debemos calcular ambos campos a partir de una lectura de los cines que proyectaron dicha película y sumar los valores de dichos campos.

Para evitar que ese cálculo se realice cada vez que se necesita la inforamción, podemos calcular los totales y guardarlos en una colección `movies` 

```javascript
db.movies.insertOne(
   {
      _id: 1,
      title: "Lost in the Shadows",
      total_viewers: 1840,
      total_revenue: 25880
   }
)
```

Cuando añadimos un nuevo visionado:

```javascript
db.screenings.insertOne(
   {
      theater: "Overland Park Cinema",
      location: "Boise, ID",
      movie_title: "Lost in the Shadows",
      movie_id: 1,
      num_viewers: 760,
      revenue: 7600
   }
)
```

los campos calculados ya no reflejan la realidad y deberían ser recalculados. La frecuencia de esta operaicón ya depende de la aplicación:

- Si estamos en un entorno de pocas escrituras, la actualización puede hacerse con cada inserción de un nuevo documento en `screenings`. 

- Si existen más escrituras, el cálculo puede hacerse a determinados intervalos (por ejemplo cada hora, o cada x número de documentos nuevos insertados). 

Para actualizar los datos computados que almacenamos, podríamos hacer:

```javascript
db.screenings.aggregate( [
   {
      $group: {
         _id: "$movie_id",
         total_viewers: {
            $sum: "$num_viewers"
         },
         total_revenue: {
            $sum: "$revenue"
         }
      }
   },
   {
      $merge: {
         into: { db: "test", coll: "movies" },
         on: "_id",
         whenMatched: "merge"
      }
   }
] )
```

Ahora cuando hagamos de nuevo:

```javascript
db.movies.find()
```

Obtendremos los campos calculados actualizados.

```javascript
[
   {
      _id: 1,
      title: 'Lost in the Shadows',
      total_viewers: 2600,
      total_revenue: 33480
   }
]
```

## Approximation Pattern

Este patrón no deja de ser una extensión del `computed pattern`. 

**Computed Pattern**: Calcula y almacena valores derivados de otros datos, actualizándolos cada vez que los datos subyacentes cambian. Así, los valores computados siempre reflejan el estado actual y son precisos.

**Approximation Pattern**: También almacena valores calculados, pero no los actualiza en cada modificación de los datos base. En su lugar, los recalcula periódicamente (por ejemplo, cada cierto número de inserciones o cada intervalo de tiempo). Esto significa que los valores pueden estar desactualizados temporalmente, pero se gana en rendimiento y escalabilidad, especialmente cuando las escrituras son frecuentes.

### Ejemplo

En una aplicación en el que los usuarios realizan una reseña de un determinado libro, con cada nueva reseña, la media del libro cambia, pero no siempre lo hace de forma significativa. Podríamos incrementar el recuento de revisión y volver a calculad el número:

`(avgRating * reviewCount + newRating) / (reviewCount + 1)`

Esta visión es más precisa pero incrementa mucho el número de escrituras. Cuando hay solo unas pocas reseñas para un libro, ese costo adicional es asumible y necesario, ya que queremos que el dato sea preciso, pero cuando un libro tiene ya miles de reseñas una nueva reseña apenas modifica el valor previo. Entonces a partir de cierto número de reseñas, podríamos optar por recalcular la media periódicamente en vez de bajo demanda. Esto reduce drásticamente las operaciones de escritura a cambio de una pérdida de precisión en el campo calculado. Una forma de implementar esto es mediante la introducción de un número aleatorio en cada nueva revisión. Solo realizaremos el cálculo del campo computado cuando este valor sea 10. Entonces hay que hacer una modificación en como calculamos el `avgRating`. En lugar de incrementar el valor `reviewCount` en 1, lo haremos en 10 y la `newRating` también habría que multiplicarla por 10. Esto asume que las últimas 10 reseñas han tenido ese mismo valor de newRating (sería el equivalente a decir que ha habido 10 reseñas idénticas). Es estadísticamente correcto. Sacrifica algo de precisión, pero a cambio obtenemos un 90% menos de escrituras en nuestra coleccion.

## Extended Reference Pattern

## Scheme Versioning Pattern