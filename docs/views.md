---
sidebar_position: 3
---

# Vistas
Las vistas en MongoDB son objetos que se crean en base al resultado de un pipeline de procesamiento. Son básicamente una forma de presentar datos almacenados en una o más colecciones de manera personalizada, sin duplicar ni modificar datos originales. Poseen tres características fundamentales:

- Únicamente permiten operaciones de lectura.
- Son dinámicas. No persisten datos en disco sino que son que se calculan cada vez que se consulta la vista, por lo que si los datos de las colecciones que se consultan cambian, también lo harán los datos de la vista.
- No soportan índices, por lo que su rendimiento depende de los índices de las colecciones subyacentes.

Una de sus principales utilidades es que permite simplificar consultas complejas que se realizan de forma recurrente, lo que permite ahorrar tiempo al crear una capa de abstracción que nos evita tener que pensar cada vez en la estructura de datos subyacente.

Cada vez que consultamos una vista, MongoDB ejecuta una consulta con el pipeline que hayamos definido durante la creación de la vista.

## Creación de vistas

Para crear una vista:

```js
db.createView(<nombre_vista>, <colección>, <[pipeline]>)
```

Supongamos una colección `books` con el siguiente modelo de datos:

```json
{
    title: string,
    authors: array<string>,
    year: number
    published: number
    editor: string
}
```

Podemos crear una vista:

```js
db.createView(group_editors, books, [
    {$group: {_id: "$editor", total: {$sum: 1}}}
])
```

Esto crea una vista en la que hemos realizado una agrupación para ver el total de libros publicados por cada editorial. Podemos consultar la vista de la siguiente manera:

```js
db.group_editors.find({})
```

Al ejecutar esta consulta, MongoDB internamente ejecuta el pipeline de agregación definido en la vista, de forma que nos devolverá los datos agrupados por editor. Pero además, podemos establecer condiciones cuando consultamos la vista:

```js
db.group_editors.find({year: {$gte: 2004}})
```

Existe otra forma de crear vistas:

```js
db.createCollection(
  "<viewName>",
  {
    "viewOn" : "<source>",
    "pipeline" : [<pipeline>],
  }
)
```

Es posible crear vistas sobre otras vistas, simplemente utilizando la vista como si fuera una colección más:

```js
db.createView(view_2, view_1, [
    {
        $match: {field: value}
    }
])
```

Cuando se ejecute la vista MongoDB ejecutará a su vez primero el pipeline de la vista nueva (view_2) y sobre esos resultados, el pipeline de la vista creada anteriormente (view_1).

## Beneficios de utilizar vistas

- Simplifica queries complejas creando una capa de abstracción sobre el modelo de datos.

- Permite reutilizar queries complejas.

- Proporciona un mayor control de acceso y seguridad, ya que permite limitar los datos de una colección que deseamos que un usuario de base de datos pueda consultar.

- Encapsulamiento de cambios: Si cambias la lógica de cómo se definen los datos (por ejemplo, agregar un nuevo campo a la vista o cambiar los criterios de filtrado), solo necesitas actualizar la vista y no todos los lugares donde se realiza la consulta en el código.

## Vistas materializadas

Las vistas materializadas pueden considerar vistas precomputadas. Una vista materializada es una colección con resultados ya calculados de un pipeline de agregación. Los datos calculados se almacenan en una colección, con lo cual el acceso a los datos de la vista es mucho más rápido que en las vistas estándar. Están pensadas para almacenar el resultado de consultas complejas con alto costo de computación. Su principal problema es que no se actualizan de forma automática cuando las colecciones sobre las que se ejecuta el pipeline se modifican.

- Persisten datos en disco
- No se actualizan de forma dinámica
- Es posible crear índices sobre ella

Para crear una vista materializada:

```js
db.collection.aggregate(
    [
        {$stage1: {}}, 
        {$stage2: {}}, 
        ..., 
        {$out: <new_collection_name>}
    ]
)
```

En las vistas materializadas, el paso clave es el último, ya que esta etapa debe reflejar la creación o modificación de la colección.Existen dos operadores para escribir en una colección el resultado de un pipeline: `$out` y `$merge`.

El operador `$out` creará la colección indicada o sobreescribirá los datos existentes.

El operador `$merge` también permite crear la colección si esta no existe, pero además permite la modificación de los documentos existentes. Este operador se utiliza con más frecuencia que `$out`.

```js
db.collection.aggregate(
    [
        {$stage1: {}}, 
        {$stage2: {}}, 
        ..., 
        {$merge: {
            into: <new_collection_name>,
            whenMatched: <options>
            whenNotMatched: <options>
        }}
    ]
)
```

Según la documentación de MongoDB:

```js
db.collection.aggregate(
    [
        { $merge: {
            into: <collection> -or- { db: <db>, coll: <collection> },
            on: <identifier field> -or- [ <identifier field1>, ...],  // Optional
            let: <variables>,                                         // Optional
            whenMatched: <replace|keepExisting|merge|fail|pipeline>,  // Optional
            whenNotMatched: <insert|discard|fail>                     // Optional
        }}
    ]
)
```

El operador `$merge` ofrece opciones sobre qué hacer cuando se encuentra ya el documento en la colección o cuando no se encuentra. Por ejemplo, es posible reemplazar el documento (`replace`), mantener el existente (`keepExisting`), fusionar los documentos (`merge`, que es la opción por defecto), de

El gran inconveniente de las vistas materializadas es que no son dinámicas, lo que quiere decir que cuando los datos de la colección subyacentes cambian, los datos de la vista no lo hacen de forma automática, sino que su actualización queda a nuestra merced. Es posible actualizar bajo demanda la vista, diciéndole a MongoDB que documentos tiene que actualizar. Por ejemplo, suponiendo que se inserta un nuevo documento en una colección que es utiliza por la vista, podemos utilizar este documento para excluir todos aquellos que no deseamos actualizar en la vista, de forma que solo se actualice la vista con el nuevo. Si tenemos un campo `createdAt`, podemos actualizar la vista solo con los documentos que tengan una fecha de `createdAt` posterior a la fecha en la que se insertó el nuevo documento:

```js
db.mat_view.find({createdAt: {$gt: ISODate(<date>)}})
```