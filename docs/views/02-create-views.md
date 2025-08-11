---
sidebar_position: 2
---

# Creación de vistas

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
