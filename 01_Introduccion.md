# Introducción

### Documentación Oficial

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

### Concepto de Documento en MongoDB

Tres ventajas:

* Correspondencia con tipos nativos (Objetos JS)
* Los arrays o documentos embebidos reducen las necesidades de joins.
* Esquemas dinámicos permiten polimorfismo

La base de datos ya no tiene la estructura de la base de datos, se hace desde las clases (**Mongoos**).

La velocidad de lectura es muy rápida.

### BSON (JSON Binario)

* Velocidad de lectura.
* Amplía tipos de JSON [BSON Types](https://docs.mongodb.com/manual/reference/bson-types/)

BSON es transparente para el administrador. No comprime tanto comparado con JSON.

Si solo usas JS todos los tipos de BSON sobrarán, pero para entornos como JAVA u otros si son muy útiles.

Limites:

* Documento BSON menor a 16 megabytes.
* Documentos anidados o embebidos menor de 100 niveles.

### Documentos, Colecciones y Bases de Datos

BD Relacional | MongoDB
--------------|--------
Registro | Documento
Tabla | Colección
BD | BD

Ademas tenenos:

* Vistas de sólo lectura
* Vistas On-demand

### Campo `_id`

Todos los documentos tendrán el campo `_id` t será único a nivel de colección.

Si el campo `_id` no es especificado cuando se inserta un documento MongoDB lo añade como un tipo [ObjectId](https://docs.mongodb.com/manual/reference/bson-types/#objectid).

<img src="/images/c1/objectid.png">

La mayoría de los drivers crean el `_id` de tipo `ObjectId` si no se especifica. :skull:











