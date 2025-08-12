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

```javascript
db.books.find()
```

O según su tipo:
```javascript
db.books.find({product_type: "ebook"})
```

Y también podemos realizar consultas basándonos en campos específicos del tipo a que pertenece:

```javascript
db.books.find({"duration.hours": { $gt: 20 }})
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

En una aplicación en el que los usuarios realizan una reseña de un determinado libro, con cada nueva reseña, la media del libro cambia, pero no siempre lo hace de forma significativa. Podríamos incrementar el recuento de revisión y volver a calcular el número:

`(avgRating * reviewCount + newRating) / (reviewCount + 1)`

Esta visión es más precisa pero incrementa mucho el número de escrituras. Cuando hay solo unas pocas reseñas para un libro, ese costo adicional es asumible y necesario, ya que queremos que el dato sea preciso, pero cuando un libro tiene ya miles de reseñas una nueva reseña apenas modifica el valor previo. Entonces a partir de cierto número de reseñas, podríamos optar por recalcular la media periódicamente en vez de bajo demanda. Esto reduce drásticamente las operaciones de escritura a cambio de una pérdida de precisión en el campo calculado. Una forma de implementar esto es mediante la introducción de un número aleatorio en cada nueva revisión. Solo realizaremos el cálculo del campo computado cuando este valor sea 10. Entonces hay que hacer una modificación en como calculamos el `avgRating`. En lugar de incrementar el valor `reviewCount` en 1, lo haremos en 10 y la `newRating` también habría que multiplicarla por 10. Esto asume que las últimas 10 reseñas han tenido ese mismo valor de newRating (sería el equivalente a decir que ha habido 10 reseñas idénticas). Es estadísticamente correcto. Sacrifica algo de precisión, pero a cambio obtenemos un 90% menos de escrituras en nuestra coleccion.

## Extended Reference Pattern

Hay ocasiones en las que tiene sentido tener colecciones separadas para los datos. Si una entidad puede considerarse como una «cosa» independiente, a menudo tiene sentido tener una colección separada. Por ejemplo, en una aplicación de comercio electrónico, existe la idea de un `order`, al igual que la de un `customer` y la de un `inventory`. Son entidades lógicas independientes.

Sin embargo, desde el punto de vista del rendimiento, esto se convierte en un problema, ya que necesitamos reunir los datos para un pedido específico. Un cliente puede tener N pedidos, lo que crea una relación 1-N. Desde el punto de vista del pedido, si le damos la vuelta, tienen una relación N-1 con un cliente. Incorporar toda la información sobre un cliente para cada pedido solo para reducir la operación JOIN da como resultado una gran cantidad de información duplicada. Además, es posible que no toda la información del cliente sea necesaria para un pedido.

El patrón de referencia ampliada ofrece una forma excelente de gestionar estas situaciones. En lugar de duplicar toda la información sobre el cliente, solo copiamos los campos a los que accedemos con frecuencia. En lugar de incorporar toda la información o incluir una referencia para unir la información, solo incorporamos los campos de mayor prioridad y a los que se accede con más frecuencia, como el nombre y la dirección.

### Ejemplo

Colección `customers`:

```json
{
   _id: "123",
   name: "Katrina Pope",
   street: ""123 Main St",
   city: "Somewhere",
   country: "Someplace",
   date_of_birth: ISODate(),
   social_handles: [
      twitter: "@somethingamazing123"
   ],
   ...
}
```

Colección `orders`:

```json
{
   _id: ObjectId(""),
   data: ISODate(),
   customer_id: 123,
   shipping_address: {
      name: "Katrina Pope",
      street: "123 Main St",
      city: "Somewhere",
      country: "Someplace",
   },
   order: [
      {
         product: "widget",
         qty: 5,
         cost: {
            value: NumberDecimal("11.99")
            currency: "USD",
         }
      }
   ]
}
```

Algo que hay que tener en cuenta al utilizar este patrón es que los datos se duplican. Por lo tanto, funciona mejor si los datos que se almacenan en el documento principal son campos que no cambian con frecuencia. Algo como un user_id y el nombre de una persona son buenas opciones. Esos datos rara vez cambian.

Además, solo se deben importar y duplicar los datos que sean necesarios. Pensemos en una factura de pedido. Si importamos el nombre del cliente en una factura, ¿necesitamos su segundo número de teléfono y su dirección de entrega en ese momento? Probablemente no, por lo que podemos dejar esos datos fuera de la colección de facturas y hacer referencia a una colección de clientes.

Cuando se actualiza la información, también debemos pensar en cómo gestionarla. ¿Qué referencias ampliadas han cambiado? ¿Cuándo deben actualizarse? Si la información es una dirección de facturación, ¿necesitamos mantener esa dirección con fines históricos o está bien actualizarla? A veces, es mejor duplicar los datos porque así se conservan los valores históricos, lo que puede tener más sentido. La dirección en la que vivía nuestro cliente en el momento en que enviamos los productos tiene más sentido en el documento del pedido que obtener la dirección actual a través de la recopilación de clientes.

## Scheme Versioning Pattern

Dado que las aplicaciones se encuentran en continua evolución es probable que también nuestros modelos de datos cambien, añadiéndose campos, reestructurando información, etc... Esto suele ser más complicado de hacer en bases de datos relacionales que en un sistema con modelos flexibles como es MongoDB.

En MongoDB podemos utilizar el patrón de control de versiones del esquema para facilitar los cambios.

Como se ha mencionado, actualizar el esquema de una base de datos tabular puede ser complicado. Por lo general, es necesario detener la aplicación, migrar la base de datos para que sea compatible con el nuevo esquema y, a continuación, reiniciarla. Este tiempo de inactividad puede dar lugar a una mala experiencia del cliente. Además, ¿qué ocurre si la migración no se ha realizado con éxito? Volver al estado anterior suele ser un reto aún mayor.

El patrón Schema Versioning aprovecha la compatibilidad de MongoDB con documentos de diferentes formas que pueden coexistir en la misma colección de la base de datos. Este aspecto polimórfico de MongoDB es muy potente. Permite que documentos con campos diferentes o incluso con tipos de campos diferentes para el mismo campo coexistan sin problemas.

Para implementar este patrón basta con añadir una clave que determine qué versión de modelo se está usando, algo como un campo `schema_version`. Así, cuando nuestra aplicación reciba esta campo sabrá como gestionar la entidad.

### Ejemplo

Supongamos un modelo de datos `users` que en un principio fue así:

```
{
   "_id": "<ObjectId>",
   "name": "Anakin Skywalker",
   "home": "503-555-0000",
   "work": "503-555-0010"
}
```

Pero con el paso de los años se comenzó a necesitar un campo adicional para el número de teléfono móvil:

```
{
   "_id": "<ObjectId>",
   "name": "Darth Vader",
   "home": "503-555-0100",
   "work": "503-555-0110",
   "mobile": "503-555-0120"
}
```

Este cambio es relativamente y apenas afecta al modelo de datos, ya que simplemente es un campo que puede estar presente o ausente. Pero con el paso del tiempo, se generan nuevas formas de contacto, que han de ser tenidas en cuenta. Así, podemos llegar al siguiente modelo:

```
{
   "_id": "<ObjectId>",
   "schema_version": "2",
   "name": "Anakin Skywalker (Retired)",
   "contact_method": [
       { "work": "503-555-0210" },
       { "mobile": "503-555-0220" },
       { "twitter": "@anakinskywalker" },
       { "skype": "AlwaysWithYou" }
   ]
}
```

La flexibilidad del modelo de documentos de MongoDB permite que todo esto ocurra sin tiempo de inactividad de la base de datos. Desde el punto de vista de la aplicación, se puede diseñar para leer ambas versiones del esquema. Este cambio en la aplicación en cuanto a cómo gestionar la diferencia de esquemas tampoco debería requerir tiempo de inactividad, suponiendo que haya más de un servidor de aplicaciones involucrado.