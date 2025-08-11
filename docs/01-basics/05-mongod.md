---
sidebar_position: 5
---

# mongod

Cuando se ejecuta sin argumentos, `mongod` usará el directorio de datos por defecto /data/db/). Si el directorio no existe o no tiene permisos de escritura, el servidor fallará por lo que es importante crear el directorio y darle los permisos necesarios.
Si no tienes tu directorio /data/db/ puedes buscar en /etc/mongod.conf la línea dbPath, donde aparecerá la ruta en donde se almacenan los ficheros.