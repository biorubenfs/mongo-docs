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

## Approximation Pattern

## Extended Reference Pattern

## Scheme Versioning Pattern