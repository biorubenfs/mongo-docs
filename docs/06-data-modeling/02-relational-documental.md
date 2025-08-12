---
sidebar_position: 2
---

# Relacional vs Documental

Existe la tentación comprensible de diseñar los esquemas o modelos de MongoDB de la misma manera en que siempre hemos diseñado nuestros esquemas en modelos relacionales SQL. Sin embargo, hacer esto nos impedirá explotar algunas de las características y funcionalidades de MongoDB y del modelo documental.

## Relacional

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

## Documental (mongodb)

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
}
```

Esto nos permite recuperar todos los datos en una única query sencilla, ya que nos permite almacenar datos en arrays y objetos dentro del usuario.
