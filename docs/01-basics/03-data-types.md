---
sidebar_position: 3
---

# Tipos de Datos

MongoDB soporta diferentes tipos de datos como valores en sus documentos.

Los documentos en MongoDB pueden verse como J"SON-like" en el sentido de que son conceptualmente similares a los objetos en JavaScript. JSON es una representación sencilla de los datos. En teoría JSON solo admite 6 tipos de datos. Aunque esto facilita la comprensión, también lo limita a `null`, `boolean`, `string`, `array` y `object`.

JSON no tiene un tipo date, solo hay un tipo para trabajar con números (no distingue entre floats y enteros y tampoco entre números 32 bits y 64 bits). No hay manera de representar otros tipos habituales, como expresiones regulares o funciones.

- **Null**: representa tanto un valor `null` como un campo inexistente.
- **Boolean**
- **Number**: la shell por defecto emplea numeros de coma flotante 64 bits. En la shell se visualizan como números normales
```
{"x" : 3.14}
{"x" : 3}
```     
Para enteros, se emplean las clases `NumberInt` o `NumberLong` que representan enteros de 4 y 8 bytes respectivamente (4 bytes es 32-bits y 8 bytes es 64-bits):
```
{"x" : NumberInt("3")}
{"x" : NumberLong("3")}
```
- **String**
- **Date**: MongoDB almacena las fechas como enteros de 64-bits que representan los milisegundos desde la Unix epoch (1 Enero de 1970). No guarda la zona horaria.
```
{"x" : new Date()}
```
- Regular expression: 
```
{"x" : /foobar/i}
```
- Array
- Documentos incrustados
- **ObjectID**: Es un Id de 12-bytes para documentos.
```      
{"x" : ObjectId()}
```
- **Binary data**: Es un string de bytes arbitrarios. No se puede manipular desde la shell y es la única forma de guardar cadenas de texto no-UTF-8.
- **Code**: Se puede guardar JS en queries y documentos:
```
{"x" : function() { /* ... */ }}
```

## _id y ObjectId

Cada documento de MongoDB debe contener un campo `_id`. El valor puede ser cualquier tipo, pero por defecto es un `ObjectId`. En una colección, cada documento debe tener un valor único para `_id` que asegura que un documento en esa colección puede ser identificado de manera unívoca. Si el documento a insertar no tiene un `_id`, automaticamente se le proporcionará uno en el momento de la inserción.

La clase `ObjectId` está diseñada para ser liviana y que a la vez sea capaz de generar un identificador único entre diferentes máquinas. La naturaleza distribuida de MongoDB es la principal razón para adoptar este enfoque en lugar de otras aproximaciones más tradicionales como autoincrementales, que hacen que sea más lento y dificil sincronizar las claves primarias entre múltiples servidores. Es importante que MongoDB tenga la capacidad de generar identificadores en entornos fragmentados (sharded).

Un `ObjectId` emplea 12 bytes de almacenamiento, lo que hace una cadena de texto de 24 digitos hexadecimales, 2 dígitos por cada byte. Aunque un `ObjectId` se representa a menudo como un string hexadecimal grande, el string es únicamente el doble de grande que el dato que está almacenando.

Si creas varios ObjectIds de forma rápida y sucesiva, podrás ver que únicamente los digitos del final cambian. Si espacias la creación algunos segundos, verás que cambian algunos digitos del medio. Esto es debido a la forma en la que se crean los ObjectsIds.

La estructura del `ObjectId` es la siguiente:
- A 4-byte timestamp, representing the ObjectId's creation, measured in seconds since the Unix epoch.
El conjunto de los primeros 9 bytes, proporciona unicidad con granularidad de un segundo. Puesto que los timestamps vienen primero, los ObjectIds se ordenaran en orden de inserción y hace que los _ids sean un buen campo a utilizar como índice. Como los 4 bytes iniciales son un timestamp implícito de cuando se creó cada documento. La mayoria de los drivers poseen un método para extraer la información del ObjectId.
- A 5-byte random value generated once per process. This random value is unique to the machine and process.
- A 3-byte incrementing counter, initialized to a random value.

Los 9 primeros bytes de un `ObjectId` garantizan unicidad enter maquinas y procesos durante un  solo segundo. Los ultimos tres bytes son simplemente un contador incremental que es responsable de la unicidad dentro de un segundo en un solo proceso. Esto permite generar hasta 256³ ObjectsIds únicos por proceso en un sólo segundo.