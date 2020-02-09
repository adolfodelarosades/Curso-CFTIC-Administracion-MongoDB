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

La mayoría de los drivers crean el `_id` de tipo `ObjectId` si no se especifica. :japanese_ogre:

Restricciones del  `_id`:

* Puede ser de cualquier tipo incluso un documento, excepto de tipo array. :japanese_ogre:
* El `_id` es único a nivel de colección

¿Permite Objetos con arrays? Vamos a comprobarlo

```sh
> use foo
switched to db foo

> db.clientes.insert({_id: {a:1, b:1}})
WriteResult({ "nInserted" : 1 })

PERMITIDO

> db.clientes.insert({ _id: [1, 2, 3] })
WriteResult({
	"nInserted" : 0,
	"writeError" : {
		"code" : 2,
		"errmsg" : "can't use an array for _id"
	}
})

NO PERMITIDO

> db.clientes.insert({ _id: {[1, 2, 3]} })
2020-02-09T20:44:04.777+0100 E  QUERY    [js] uncaught exception: SyntaxError: missing ] in computed property name :
@(shell):1:29

SEGÚN PERMITIDO PERO AMI ME MARCA ESTE ERROR
```

### Arrancar Mongo

Para arrancar Mongo exitesn varias formas:

* `mongo` es como poner `mongo --host localhost --port 27017`
* `mongo --host`
* `mongo --host direccion.com --port 8080`
* `mongo --username <usuario> --password <pass>` 
   `--authenticationDatabase <nombreBD>`
   `--host <direccion>`
   `--port <no. puerto>`
   
Si el password se omite lo pregunta después.

### Comandos Básicos.

* `cls`: Limpia la pantalla
* `show dbs`: Muestra las BD existentes.
* `db`: Indica en que BD estoy trabajando
* `use <nombre>`: Cambiar a otra BD, si no existe la crea.
* `use gimnasio`: Cambia a la BD gimnasio, si no existe la crea.
* `show dbs`: Muestra todas las BD existentes, no aparece `gimnasio` por que no tiene contenido.
* `db`: Indica que estamos en la BD gimnasio.

<img src="/images/c1/1-comandos-basicos.png">

**Crear Colecciones**

Existen 2 formas:

* Insertar un registro (Indirecta)
* Crear una colección (Directa

**Crear Colección de Forma Indirecta**

`db.<nombreColeccion>.insert(<documento>)`

**Crear Colección de Forma Directa**

`db.createCollection.(<nombreColeccion>)`

**Listar las Colecciones**

`show collections`


```sh
> db.clientes.insert({nombre: "Juan"})
WriteResult({ "nInserted" : 1 })
> show dbs
Ejemplo    0.000GB
admin      0.000GB
config     0.000GB
cursenode  0.000GB
cursonode  0.000GB
foo        0.000GB
gimnasio   0.000GB
local      0.000GB
udemyDb    0.000GB
> show collections
clientes
> db.createCollection("inventario")
{ "ok" : 1 }
> show collections
clientes
inventario
```

**Las colecciones usan las mismas reglas que usa JS para declarar variables**

```sh
> db.3monitores.insert({ modelo: "HP" })
2020-02-09T21:24:47.123+0100 E  QUERY    [js] uncaught exception: SyntaxError: identifier starts immediately after numeric literal :
@(shell):1:2
> db.createCollection("3monitores")
{ "ok" : 1 }
> db.getCollection("3monitores").insert({ modelo: "HP" })
WriteResult({ "nInserted" : 1 })
> db.3monitores.insert({ modelo: "LG" })
2020-02-09T21:28:45.084+0100 E  QUERY    [js] uncaught exception: SyntaxError: identifier starts immediately after numeric literal :
@(shell):1:2
```

En teoría el nombre `3monitores` no sería valido para nombrar una colección, **PERO si la creamos directamente LO PERMITE, si la creamos de manera indirecta NO LO PERMITE**.

**Insertemos un documento más complejo**:

```sh
> db.clientes.insert(
... {
... nombre: "María",
... apellidos: "Pérez Gómez",
... edad: 46
... }
... )
WriteResult({ "nInserted" : 1 })
> 
```

Observamos que para nombres de los campos no es necesario que vayan entre comillas.

**Mostrar documentos de la colección**

```sh
> db.clientes.find()
{ "_id" : ObjectId("5e406738ce31e8b17f0c7e7c"), "nombre" : "Juan" }
{ "_id" : ObjectId("5e406d3ace31e8b17f0c7e7e"), "nombre" : "María", "apellidos" : "Pérez Gómez", "edad" : 46 }

> db.clientes.find().pretty()
{ "_id" : ObjectId("5e406738ce31e8b17f0c7e7c"), "nombre" : "Juan" }
{
	"_id" : ObjectId("5e406d3ace31e8b17f0c7e7e"),
	"nombre" : "María",
	"apellidos" : "Pérez Gómez",
	"edad" : 46
}
```

Para mostrar los documentos de una colección usamos el método `find()` y si lo queremos pintarlo bonito usamos además el método `pretty()`.

**Borrar Colecciones**

Existen dos formas de borrar las colecciones, pero siempre usando el método `drop()`:

```sh
> show collections
3monitores
clientes
inventario

> db.getCollection("3monitores").drop()
true

> db.inventario.drop()
true

> show collections
clientes
```

**Borrar la Base de Datos**

Para eliminar la BD se usa el método `dropDatabase()`:

```sh
> show dbs
Ejemplo    0.000GB
admin      0.000GB
config     0.000GB
cursenode  0.000GB
cursonode  0.000GB
foo        0.000GB
gimnasio   0.000GB
local      0.000GB
udemyDb    0.000GB

> db
gimnasio

> db.dropDatabase()
{ "dropped" : "gimnasio", "ok" : 1 }

> db
gimnasio

> show collections

> show dbs
Ejemplo    0.000GB
admin      0.000GB
config     0.000GB
cursenode  0.000GB
cursonode  0.000GB
foo        0.000GB
local      0.000GB
udemyDb    0.000GB
```

Como vemos con el comando `db.dropDatabase()` eliminamos la BD pero si pulsamos `db` nos indica que seguimos en `gimnasio`, si intentamos mostrar sus colecciones no muestra nada y si listamos las BDs no se muestra `gimnasio`.















