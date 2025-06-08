---
sidebar_position: 4
---

# JSON vs BSON

## JSON

JSON es un formato de intercambio de datos legible por los humanos. Los objetos JSON son contenedores de información en donde una clave de tipo string almacena un valor (que puese ser number, string, boolean, array, vacío (null) o incluso otro objeto).

```json
{
  "_id": 1,
  "name": { "first" : "John", "last" : "Backus" },
  "contribs": [ "Fortran", "ALGOL", "Backus-Naur Form", "FP" ],
  "awards": [
    {
      "award": "W.W. McDowell Award",
      "year": 1967,
      "by": "IEEE Computer Society"
    }, {
      "award": "Draper Prize",
      "year": 1993,
      "by": "National Academy of Engineering"
    }
  ]
}
```

Desde los inicios de MongoDB emplea JSON para representar estructuras de datos en el modelo de documento de datos. Sin embargo el uso de JSON en el funcionamiento interno de una base de datos, presenta algunas desventajas:
- JSON solo soporta un número limitado de tipos de datos básicos. JSON no tiene soporte para fechas y datos binarios.
- Al ser un formato basado en texto, hace que el parseo sea muy lento.
- Al ser un formato basado en texto, consume más espacio.
- Los objetos JSON y sus propiedades, no tienen una longitud fija, lo que hace que recorrerlos sea más lento.

El formato BSON aparece para solucionar estos problemas, pero sin renunciar al alto rendimiento y el propósito general. BSON es una representación binaria para almacenar datos en formato JSON, que optimiza la velocidad, esl espacio y la eficiencia.

## BSON (Binary JSON)

En este formato:
- La estructura binaria codifica la información de tipo y longitud, lo que permite recorrerlo mucho más rápidamente. 
- Añade algunos tipos de datos no nativos de JSON, como fechas y datos binarios.
- Únicamente es legible para la máquina.

Aquí puedes ver algunos ejemplos de objetos JSON y su representación en BSON:

```json
{"hello": "world"} →
\x16\x00\x00\x00           // total document size
\x02                       // 0x02 = type String
hello\x00                  // field name
\x06\x00\x00\x00world\x00  // field value
\x00                       // 0x00 = type EOO ('end of object')
```

```json
{"BSON": ["awesome", 5.05, 1986]} →
\x31\x00\x00\x00
 \x04BSON\x00
 \x26\x00\x00\x00
 \x02\x30\x00\x08\x00\x00\x00awesome\x00
 \x01\x31\x00\x33\x33\x33\x33\x33\x33\x14\x40
 \x10\x32\x00\xc2\x07\x00\x00
 \x00
 \x00
```


| Característica     | JSON                                          | BSON                                                                                 |
|--------------------|-----------------------------------------------|--------------------------------------------------------------------------------------|
| Encoding           | UTF-8 String                                  | Binary                                                                               |
| Data Support       | String, Boolean, Number, Array, Object, null  | String, Boolean, Number (Integer, Float, Long, Decimal128...), Array, null, Date, BinData |
| Readability        | Human and Machine                             | Machine Only                                                                         |

## Almacenamiento de datos

MongoDB almacena los datos en formato `BSON` tanto de forma interna como a lo largo de la red. Sin embargo esto no significa que no podamos pensar en MongoDB como una base de datos JSON. Cualquier cosa representada en `JSON` puede ser nativamente almacenada en MongoDB y recuperado fácilmente como un simple `JSON`.

El driver de MongoDB para el lenguaje de programación que estemos usando es el encargado de convertir los datos a `BSON` y devolverlos de nuevo cuando realizamos peticiones a la base de datos.

Tipos de datos BSON: https://www.mongodb.com/docs/manual/reference/bson-types/?_ga=2.157655226.1402691322.1664900628-1238349932.1620977965