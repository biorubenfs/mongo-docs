---
sidebar_position: 2
---

# Propiedades de las transacciones

Las transacciones poseen las siguientes propiedades, que se conocen como **ACID** (*atomicity*, *consistency*, *isolation*, *durability*):

## Atomicidad
La atomicidad garantiza que todos los comandos que conforman una transacción se traten como una unidad única y que tengan éxito o fracasen juntos. Esto es importante porque, en caso de un evento no deseado, como un fallo o un corte de energía, podemos estar seguros del estado de la base de datos. La transacción se habrá completado correctamente o se habrá revertido si alguna parte de la transacción falló.

## Consistencia
La consistencia garantiza que los cambios realizados dentro de una transacción sean consistentes con las restricciones de la base de datos. Esto incluye todas las reglas, restricciones y disparadores. Si los datos llegan a un estado no válido, la transacción completa fallará.

## Aislamiento
Asegura que todas las transacciones se ejecutan en un entorno aislado. Eso posibilita la ejecución de transacciones de forma concurrente puesto que las transacciones no interfieren entre sí.

## Durabilidad
La durabilidad garantiza que una vez que la transacción se completa y los cambios se escriben en la base de datos, se persisten. Esto asegura que los datos dentro del sistema persistan incluso en caso de fallos del sistema, como caídas o cortes de energía