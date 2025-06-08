---
sidebar_position: 6
---

# Mongo shell

Para iniciar la shell:

```bash
$ mongo
```

La shell intentará de forma automática conectarse al servidor de MongoDB que se está ejecutando la máquina local.

## Cliente de MongoDB
Aunque permite ejecutar JS, la shell es importante en tanto que es un cliente de MongoDB. Al inicio, la shell se conectará a la base de datos test y asignará esta conexión a una variable global db. Esta variable es el punto de acceso primario a nuestro servidor de MongoDB a través de la shell.

Posee algunos add-ons por ser comunes en las shells de SQL (use, switch).

Para crear una base de datos, basta con el comando 

```
$ use database_name
```

Si la base de datos no existe, la crea, sin niguna colección ni documento.

## Operaciones básicas

- Crear: `insertOne()`
- Leer: `find()` y `findOne()`
- Actualizar: `updateOne()`
- Borrar: `deleteOne()` y `deleteMany()`