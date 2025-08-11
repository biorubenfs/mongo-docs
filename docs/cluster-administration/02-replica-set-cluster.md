---
sidebar_position: 2
---

# Replica Set vs Cluster

Un replica set es un _conjunto de servidores de MongoDB que trabajan juntos_ para proporcionar alta disponibilidad y tolerancia a fallos. Consiste en varios nodos, donde uno de ellos actúa como nodo primario (primary) y los demás como nodos secundarios (secondary). El nodo primario es responsable de las operaciones de escritura y los nodos secundarios replican los datos del nodo primario y pueden aceptar operaciones de lectura. Si el nodo primario falla, se realiza una elección automática para seleccionar un nuevo nodo primario. Los replica sets se utilizan para garantizar la disponibilidad continua de los datos y la recuperación ante fallos en caso de que uno o más nodos se vuelvan inaccesibles.

Por otro lado, un clúster en MongoDB es un conjunto más amplio de componentes que incluye replica sets, servidores de configuración y servidores de consulta, utilizado para distribuir y escalar horizontalmente los datos en un entorno de base de datos a gran escala.