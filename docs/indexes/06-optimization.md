---
sidebar_position: 5
---

# Optimización de índices

## Regla ESR (Equality, Sorting, Range)

Este principio te guía sobre cómo MongoDB aprovecha los índices compuestos y en qué orden debes organizar los campos dentro del índice para obtener el mejor rendimiento en las consultas.

Al crear un índice compuesto, los campos deben ordenarse según cómo se utilizan en las consultas, en este orden:

1. E - Equality (Igualdad):
Campos que se consultan con operadores de igualdad (`{ campo: valor }`).
Ejemplo: `{ status: "activo" }`

2. S - Sort (Ordenamiento):
Campos que se usan para ordenar los resultados (sort()).
Ejemplo: `.sort({ fecha: -1 })`

3. R - Range (Rango):
Campos que se consultan usando operadores de rango (`$gt`, `$lt`, `$gte`, `$lte`).
Ejemplo: `{ edad: { $gte: 18, $lte: 30 } }`

MongoDB solo puede usar la parte del índice hasta que encuentra el primer campo que no sigue el orden ESR.
Si no sigues ESR, podrías terminar con un índice que no se usa eficientemente.

Por ejemplo, si queremos hacer la consulta:
```js
db.usuarios.find(
  { estado: "activo", edad: { $gt: 25 } }
).sort({ fechaRegistro: -1 })
```

El índice óptimo para esta query sería:
```js
{ estado: 1, fechaRegistro: -1, edad: 1 }
```
## Covered Queries