# MongoDB Essential

[MongoDB Docs](https://docs.mongodb.com/)

MongoDB es útil para desarrollos rápidos.

### Intsalación en Windows

La instalación de MongoDB en Windows esta en la siguiente ubicación:

`Archivos de Programas/MongoDB/Server/4.2/bin`

En esta carpeta tenemos los archivos:

* `mongod`: Ejecuta el servidor de MongoDB
* `mongo`: Ejecuta el shell de Mongo

Desde el terminal verificamos la versión instalada de Mongo:

```sh
$ mongo --version
MongoDB shell version v4.2.2
```

<img src="/images/c1/diagrama-mongo.png">

El almacenamiento fisico de las bases de datos en MongoDB se hace en la carpeta:

`C:\data\db`

Si copio esa carpeta y la copio en otro ordenador no funcionará, debe hacerse la importación de las bases de datos a través de herramientas.

