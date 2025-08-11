---
sidebar_position: 3
---

# Fases de Diseño

Al contrario que en las bases de datos relacionales, donde la normalización es típicamente la primera fase, el modelado de datos en MongoDB tiene las siguientes fases:

- Identificación de la carga de trabajo

- Identificación y modelado de las relaciones entre entidades

- Aplicación de patrones de diseño

La respuesta 

## Identificación del workload

Debemos hacernos dos preguntas:

- ¿Cuáles son las principales entidades almacenadas?
- ¿Cuáles de estas entidades cambian con el tiempo?

Entender la respuesta a estas preguntas nos ayudará a aplicar la regla de oro del modelado de datos en MongoDB:

> **Regla de Oro** del Modelado de Datos en Mongo: Los datos que se acceden juntos, se guardan juntos.

## Identificación y modelado de las relaciones entre entidades

Existen 3 tipos de relaciones:

- Relación 1:1 (uno a uno)
- Relacion 1:n (uno a muchos)
- Relación n:n (muchos a muchos)
- Relación 1:m (1 a millones) como caso especial de 1:n

Una vez identificadas las relaciones entre entidades debemos trasladar dichas relaciones al modelo documental de MongoDB. Así, para las entidades relacionadas debemos decidir si referenciar las entidades o incrustar los datos de una en otra.

### Reglas generales del modelado

Podemos establecer 4 reglas generales a la hora de modelar una base de datos documental:

> #### Regla 1
> En general deberíamos priorizar incrustar los datos dentro de un documento. Existen razones para no hacerlo como por ejemplo si necesitamos acceder a ellos por sí solos, si son demasiado grandes o si los necesitamos con muy poca frecuencia.

> #### Regla 2
> La necesidad de acceder a un objeto por sí solo es una razón de peso para no incrustarlo.

> #### Regla 3
> Evita los joins/lookups si es posible, pero no te preocupes si usarlos puede hacer que tengamos un mejor diseño de esquema.

> #### Regla 4
> Los arrays no deberían crecer sin límites. Si son más de un par de cientos, no los embebas en el documentos. Si hay más de un par de miles, no uses siquiere referencias para guardar el objectId. Las matrices de alta cardinalidad son una razón de peso para no embeberlas.

### Referenciar vs incrustar (embedding)
Supongamos que tenemos una colección `books` y otra colección `authors`. Sabemos que todo libro tiene al menos un autor y que todo autor tiene al menos un libro.

Referenciar haría que los autores quedasen dentro un array en un documento books conteniendo únicamente los identificadores de esos autores en otra colección:

```json
{
    title: "book title",
    ...
    authors: [
        "author0001",
        "author0002",
        "author0003"
    ]
}
```
y la colección `authors`:

```json
[
    {
        author_id: "author001",
        name: "",
    },
    {
        author_id: "author002",
        name: "",
    },
    {
        author_id: "author003",
        name: "",
    }
]
```

Al alternativa a referenciar es la incrustación, en donde ambas entidades quedan almacenadas dentro del mismo documentos:

```json
{
    title: "",
    authots: [
        {
            author_id: "author001",
            name: ""
        },
        {
            author_id: "author002",
            name: ""
        },
        {
            author_id: "author003",
            name: ""
        }
    ]
}
```

A menudo será suficiente con seguir la regla de oro del modelado de datos de MongoDB. Por ejemplo, si los autores siempre se acceden junto con los libros -> incrustar

En general podemos seguir las siguientes directrices:

| Guideline Name   | Question                                                                                          | Embed | Reference |
|------------------|---------------------------------------------------------------------------------------------------|-------|-----------|
| Simplicity       | Would keeping the pieces of information together lead to a simpler data model and code?                                 | Yes    | No        |
| Go Together      | Do the pieces of information have a "has-a," "contains," or similar relationship?                                       | Yes    | No        |
| Query Atomicity  | Does the application query the pieces of information together?                                                          | Yes    | No        |
| Update           | Are the pieces of information updated together?                                                                         | Yes    | No        |
| Archival         | Should the pieces of information be archived at the same time?                                                          | Yes    | No        |
| Cardinality      | Is there a high cardinality (current or growing) in the child side of the relationship?                                 | No     | Yes       |
| Data Duplication | Would data duplication be too complicated to manage and undesired?                                                      | No     | Yes       |
| Document Size    | Would the combined size of the pieces of information take too much memory or transfer bandwidth for the application?    | No     | Yes       |
| Document Growth  | Would the embedded piece grow without bound?                                                                            | No     | Yes       |
| Workload         | Are the pieces of information written at different times in a write-heavy workload?                                     | No     | Yes       |
| Individuality    | For the children side of the relationship, can the pieces exist by themselves without a parent?                         | No     | Yes       |

### Relación 1:1

Es el tipo más común y a menudo se representa como un atributo dentro del documento padre. Supongamos que la relación entre `publisher` y `headquarter` es de 1 a 1. Existen diferentes alternativas:

```json
{
    _id: "",
    name: "",
    street: "",     // headquarter info
    city: "",       // headquarter info
    state: "",      // headquarter info
    country: "",    // headquarter info
    zip: ""         // headquarter info
}
```

Esos campos tendrían la misma función que una fila en un modelo relacional. Pero podemos optar por agrupar dichos campos, para tener la información mejor organizada:

```json
{
    _id: "",
    name: "",
    headquarters: {
        street: "",  
        city: "",    
        state: "",   
        country: "", 
        zip: ""      
    }
}
```

Esto también tiene la ventaja de que se amolda mejor al caso en que las direcciones tengan diferente forma según el país.

Pero hay más alternativas. Podemos dividir los datos en varias colecciones y referenciarlas:

```json
{
    _id: "",
    name: "",
    hq_id: "" // referencia
}
```

y tener:
```json
{
    _id: "",
    street: "",     
    city: "",       
    state: "",      
    country: "",    
    zip: ""         
}
```

Esta opción es útil si principalmente leemos los datos de la editorial sin incluir los datos de sus headquarters.

Otra opción es referenciar los datos del padre desde el hijo:

```json
{
    _id: "0001",
    name: "",
}
```

y tener:
```json
{
    _id: "",
    publisher_id: "0001"
    street: "",     
    city: "",       
    state: "",      
    country: "",    
    zip: ""         
}
```

Finalmente, existe una opción más, que es la de referenciar de forma bidireccional.


### Relación 1:n

Supongamos el ejemplo de `reviews` y `printed_books` en donde reviews es el lado n de la relación y printed_books el lado 1.
La forma más inmediata de tratar esto es incrustar el lado n como un array dentro del lado 1. En este caso sería un array de reviews dentro de un documento de tipo `printed_books`.

```json
{
    title: "",
    author: "",
    reviews: [
        {
            user_id: "0001",
            title: "",
            review_text: "",
            rating: 0
        },
        {
            user_id: "0002",
            title: "",
            review_text: "",
            rating: 0
        },{
            user_id: "0003",
            title: "",
            review_text: "",
            rating: 0
        }
    ]
}
```

La segunda es mediante la creación de un subdocumento en el que cada review es a su vez un subdocumento en el que la clave es el identificador de la review:

```json
{
    title: "",
    author: "",
    reviews: {
        0001: {
            ...
        },
        0002: {
            ...
        },
        0003: {
            ...
        },
    }
}
```

La ventaja que tiene la incrustación es que evitamos tener datos duplicados. Y suele ser una buena oción cuando el lado n de la relación no puede existir por si misma, como una review, que no puede existir si un libro.

Para modelar las referencias existen varias opciones. La primera es crear una un array de referencias en el padre. La segunda es tener referenciado el book dentro de la review. De esta forma podemos recuperar todas las referencias para un libro dado con una única query. 

Finalmente es posible tener bidireccionalidad en la referenciación. En general incrustar el documento entero es la forma preferida de modelar esta relación pero debemos tener en cuenta la cardinalidad de la relación. Si existe la posibilidad de que ese array sea grande, es mejor considerar la opción de que el array solo contenga la referencia. Y si la cardinalidad es enorme, entonces es mejor referenciar desde el hijo al padre.

### Relación 1:millions
Si tenemos el caso de un esquema que pudiera tener millones de subdocumentos. Esto son los unbounded array (array sin limites) y podemos superar fácilmente el tamaño de 16Mb límite de MongoDB, incluso si únicamente almacenamos ObjectId para referenciar.
Pensemos por ejemplo que deseamos crear una aplicació de registro (logging) de servidores. Cada servidor podría potencialmente guardar una cantidad masiva de datos. En lugar de guardar la relación entre el host y el mensaje de registro en el documento de host, podemos hacer que cada mensaje de registro almacece el host al que está asociado:

*Hosts*:
```json
{
    "_id": ObjectID("AAAB"),
    "name": "goofy.example.com",
    "ipaddr": "127.66.66.66"
}
```

*Log Message*
```json
{
    "time": ISODate("2014-03-28T09:42:41.382Z"),
    "message": "cpu is on fire!",
    "host": ObjectID("AAAB")
}
```

## Aplicación de patrones de diseño

Sería la fase final del proceso de modelado.
