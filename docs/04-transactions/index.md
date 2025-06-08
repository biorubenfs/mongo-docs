---
sidebar_position: 4
---

# Transacciones

En MongoDB, una operación en un único documento es **atómica**. Debido a que puedes utilizar documentos embebidos y arrays para capturar las relaciones entre datos en una única estructura de documento en lugar de normalizarlos en varios documentos y colecciones, esta atomicidad de un solo documento obvia la necesidad de transacciones multidocumento para muchos casos de uso prácticos. Sin embargo, para situaciones que requieren atomicidad en lecturas y escrituras en múltiples documentos (en una colección o en varias), MongoDB soporta **transacciones multidocumentos**.

Las transacciones multidocumento son atómicas (i.e. “todo o nada”). Una transacción es un conjunto de operaciones de lectura y escritura que únicamente tiene éxito si todas las operaciones se ejecutan de forma exitosa.

La desventaja de utilizar una transacción es que la base de datos debe "bloquear" los recursos involucrados para evitar que las escrituras concurrentes interfieran entre sí. Esto significa que otros clientes que intenten escribir datos pueden quedar esperando a que la transacción se complete, lo que afecta a la latencia de la aplicación y, en última instancia, la experiencia del usuario.
