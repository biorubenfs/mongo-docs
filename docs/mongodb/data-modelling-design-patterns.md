---
sidebar_position: 1
---

# Modelado de Datos y Patrones de Diseño

## Relacional vs Documental (MongoDB)

Existe la tentación comprensible de diseñar los esquemas o modelos de MongoDB de la misma manera en que siempre hemos diseñado nuestros esquemas en modelos relacionales SQL. Sin embargo, hacer esto nos impedirá explotar algunas de las características y funcionalidades de MongoDB y del modelo documental.

### Relacional

En el modelo relacional, lo normal es diseñar los esquemas independientemente de las consultas (_queries_) que vayamos a emplear. Se hará uso de la normalización (proceso de organizar los datos de una base de datos) hasta llegar a la tercera forma normal. Básicamente lo que se hace es trocear los datos en tablas para no tener datos duplicados. Un ejemplo sería:

Tabla _Users_

| ID  | first_name | surname | cell         | city   | location_x | location_y |
|-----|------------|---------|--------------|--------|------------|------------|
| 1   | Paul       | Miller  | 447557505611 | London | 45.12      | 47.23      |

Tabla _Professions_

| ID  | user_id | profession |
|-----|---------|------------|
| 10  | 1       | banking    |
| 11  | 1       | finance    |
| 12  | 1       | trader     |

Tabla _Cars_

| ID  | user_id | model       | year |
|-----|---------|-------------|------|
| 20  | 1       | Bentley     | 1973 |
| 21  | 1       | Rolls Royce | 1965 |

Aquí hemos separado los datos en 3 tablas que pueden ser "unidas" empleando las claves foráneas de `user_id` en la tabla `Professions` y `Cars`.

### MongoDB (Documental)

En el diseño de esquemas de MongoDB:
- no existe proceso formal (proceso de normalización)
- no algoritmos
- no reglas

A la hora de diseñar el esquema en MongoDB, lo único que importa es que tu diseño de esquema funcione y trabaje bien con tu aplicación. **Dos aplicaciones distintas que utilicen exactamente los mismos datos pueden tener esquemas muy diferentes si las aplicaciones se utilizan de forma distinta.** A la hora de diseñar un esquema, tenemos que tener en consideración:
- almacenamiento de datos
- rendimiento en las consultas
- requerimientos de hardware

El modelo de usuario anterior, podría tener esta apariencia en un modelo documental:

```json
{
    "first_name": "Paul",
    "surname": "Miller",
    "cell": "447557505611",
    "city": "London",
    "location": [45.123, 47.232],
    "professions": ["banking", "finance", "trader"],
    "cars": [
        {
            "model": "Bentley",
            "year": 1973
        },
        {
            "model": "Rolls Royce",
            "year": 1965
        }
    ]
}
```

Esto nos permite recuperar todos los datos en una única query sencilla, ya que nos permite almacenar datos en arrays y objetos dentro del usuario.

## Fases de Diseño

Al contrario que en las bases de datos relacionales, donde la normalización es típicamente la primera fase, el modelado de datos en MongoDB tiene las siguientes fases:

- Identificación del workload

- Identificación y modelado de las relaciones entre entidades

- Aplicación de patrones de diseño

La respuesta 

### Identificación del workload

Debemos hacernos dos preguntas:

- ¿Cuáles son las principales entidades almacenadas?
- ¿Cuáles de estas entidades cambian con el tiempo?

Entender la respuesta a estas preguntas nos ayudará a aplicar la regla de oro del modelado de datos en MongoDB:

> **Regla de Oro del Modelado de Datos en MongoDB**: Los datos que se acceden juntos, se guardan juntos.

### Identificación y modelado de las relaciones entre entidades

Existen 3 tipos de relaciones:

- Relación 1:1 (uno a uno)
- Relacion 1:n (uno a muchos)
- Relación n:n (muchos a muchos)

Una vez identificadas las relaciones entre entidades debemos trasladas dichas relaciones al modelo documental de MongoDB. Así, para las entidades relacionadas debemos decidir si referenciar las entidades o incrustar los datos de una en otra.

#### Referenciar o incrustar
Ejemplo de libros y autores.

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


### Aplicación de patrones de diseño

## Patrones de diseño básicos

### Inheritance Pattern

### Computed Pattern

### Approximation Pattern

### Extended Reference Pattern

### Scheme Versioning Pattern


## Patrones de diseño avanzados


## Incrustar (Embedding) VS Referenciar (Referencing)


## Referencias
[MongoDB Best Practises](https://www.mongodb.com/developer/products/mongodb/mongodb-schema-design-best-practices/)