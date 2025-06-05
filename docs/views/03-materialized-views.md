---
sidebar_position: 3
---

# Vistas materializadas

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

