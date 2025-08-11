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

## Beneficios de utilizar vistas

- Simplifica queries complejas creando una capa de abstracción sobre el modelo de datos.

- Permite reutilizar queries complejas.

- Proporciona un mayor control de acceso y seguridad, ya que permite limitar los datos de una colección que deseamos que un usuario de base de datos pueda consultar.

- Encapsulamiento de cambios: Si cambias la lógica de cómo se definen los datos (por ejemplo, agregar un nuevo campo a la vista o cambiar los criterios de filtrado), solo necesitas actualizar la vista y no todos los lugares donde se realiza la consulta en el código.