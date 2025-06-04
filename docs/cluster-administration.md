---
sidebar_position: 2
---

# Administración de Cluster

## Replica Set vs Cluster

Un replica set es un _conjunto de servidores de MongoDB que trabajan juntos_ para proporcionar alta disponibilidad y tolerancia a fallos. Consiste en varios nodos, donde uno de ellos actúa como nodo primario (primary) y los demás como nodos secundarios (secondary). El nodo primario es responsable de las operaciones de escritura y los nodos secundarios replican los datos del nodo primario y pueden aceptar operaciones de lectura. Si el nodo primario falla, se realiza una elección automática para seleccionar un nuevo nodo primario. Los replica sets se utilizan para garantizar la disponibilidad continua de los datos y la recuperación ante fallos en caso de que uno o más nodos se vuelvan inaccesibles.

Por otro lado, un clúster en MongoDB es un conjunto más amplio de componentes que incluye replica sets, servidores de configuración y servidores de consulta, utilizado para distribuir y escalar horizontalmente los datos en un entorno de base de datos a gran escala.

## Escalado horizontal vs vertical
- **Escalado vertical**: Se refiere a aumentar la potencia de procesamiento de un solo servidor o clúster (habitualmente CPU o memoria). Tanto las bases de datos relacionales como las no relacionales pueden escalar verticalmente, pero eventualmente habrá un límite en términos de potencia de procesamiento máxima y rendimiento. Además, hay costos adicionales al escalar hacia hardware de alto rendimiento, ya que los costos no se escalan de manera lineal. Su principal ventaja es que es fácil de llevar a cabo.

- **Escalado horizontal**: Hace referencia a agregar nodos adicionales para compartir la carga. Esto es difícil de lograr con bases de datos relacionales debido a la dificultad de distribuir datos relacionados entre nodos. Con las bases de datos no relacionales, esto se simplifica ya que las colecciones son autosuficientes y no están acopladas de manera relacional. Esto les permite distribuirse entre nodos de manera más sencilla, ya que las consultas no tienen que "unirlas" entre nodos. La escalabilidad horizontal en MongoDB se logra a través del uso de fragmentación (sharding) (preferible) y conjuntos de réplicas (replica sets).

## Sharding

En MongoDB, el _sharding_ es una técnica de escalado horizontal que se utiliza para distribuir datos en múltiples servidores o nodos. Consiste en dividir los datos y distribuirlos en diferentes máquinas o clústeres, lo que permite manejar grandes volúmenes de datos y cargas de trabajo intensivas de manera más eficiente.
El sharding en MongoDB se basa en una colección, que es una agrupación lógica de documentos. En lugar de almacenar todos los documentos de una colección en un solo servidor, se dividen en fragmentos (shards) y se distribuyen entre varios servidores. Cada fragmento contiene un subconjunto de los datos totales.
El proceso de sharding consta de los siguientes componentes principales:

- Servidores de configuración (config servers): Son nodos que almacenan los metadatos y la configuración del clúster. Mantienen un mapa de los datos y su ubicación en los fragmentos. Los servidores de configuración permiten que los clientes y los servidores de consulta (query routers) encuentren los fragmentos adecuados para ejecutar operaciones.

- Fragmentos (shards): Son los servidores que almacenan los datos reales. Cada fragmento es un conjunto independiente de datos almacenados en un servidor o réplica. Los fragmentos se replican en varios nodos para garantizar la alta disponibilidad y la tolerancia a fallos.

- Servidores de consulta (query routers): Son los puntos de entrada para las operaciones de lectura y escritura en el clúster de sharding. Los servidores de consulta enrutan las operaciones a los fragmentos correspondientes y devuelven los resultados al cliente.

El sharding en MongoDB se realiza mediante una clave de shard, que es el campo utilizado para particionar y distribuir los datos entre los fragmentos. Los datos se dividen en rangos o intervalos basados en la clave de shard, y cada fragmento se hace responsable de un rango específico de valores. Esto permite que las consultas se dirijan solo a los fragmentos relevantes y evita la necesidad de buscar en todo el clúster.
Al utilizar el sharding, MongoDB puede escalar horizontalmente añadiendo más servidores según sea necesario, lo que permite manejar grandes volúmenes de datos y altas cargas de trabajo de manera eficiente. Además, el sharding proporciona tolerancia a fallos y alta disponibilidad al replicar los fragmentos en varios nodos.

> En resumen, el sharding en MongoDB es una técnica de escalado horizontal que distribuye los datos en fragmentos en múltiples servidores, permitiendo un almacenamiento y una consulta eficientes de grandes conjuntos de datos.