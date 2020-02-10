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

### Shell como Interprete JS

El shell se puede usar cono interprete de JS.

```sh
> let pacientes = [];
> let nombres = ["María", "Juan", "Fernando", "Lucia"];
> let apellidos = ["Gómez", "Pérez", "García", "López"];
> for(i=0; i < 100000; i++) {
... pacientes.push({
... nombre: nombres[Math.floor(Math.random()*nombres.length)],
... apellido1: apellidos[Math.floor(Math.random()*apellidos.length)],
... apellido2: apellidos[Math.floor(Math.random()*apellidos.length)] 
... })
... }
100000
```
Esto nos inserta 100 pacientes en el array `pacientes`. Vamos a insertarlos en la BD `clinica` en una colección `pacientes`:

```sh
> use clinica
switched to db clinica
> db.pacientes.insert(pacientes)
BulkWriteResult({
	"writeErrors" : [ ],
	"writeConcernErrors" : [ ],
	"nInserted" : 100000,
	"nUpserted" : 0,
	"nMatched" : 0,
	"nModified" : 0,
	"nRemoved" : 0,
	"upserted" : [ ]
})
> db.pacientes.find().count()
100000
```

### Formato Salida de Pantalla

* Por defecto a un cursor de 20 documentos iterable con `it`

```sh
> db.pacientes.find()
{ "_id" : ObjectId("5e418e55ce31e8b17f0c7e7f"), "nombre" : "Juan", "apellido1" : "García", "apellido2" : "López" }
{ "_id" : ObjectId("5e418e55ce31e8b17f0c7e80"), "nombre" : "María", "apellido1" : "Gómez", "apellido2" : "Pérez" }
{ "_id" : ObjectId("5e418e55ce31e8b17f0c7e81"), "nombre" : "Lucia", "apellido1" : "Pérez", "apellido2" : "García" }
{ "_id" : ObjectId("5e418e55ce31e8b17f0c7e82"), "nombre" : "Juan", "apellido1" : "Gómez", "apellido2" : "Pérez" }
{ "_id" : ObjectId("5e418e55ce31e8b17f0c7e83"), "nombre" : "María", "apellido1" : "Gómez", "apellido2" : "Gómez" }
{ "_id" : ObjectId("5e418e55ce31e8b17f0c7e84"), "nombre" : "Lucia", "apellido1" : "Pérez", "apellido2" : "López" }
{ "_id" : ObjectId("5e418e55ce31e8b17f0c7e85"), "nombre" : "Lucia", "apellido1" : "Pérez", "apellido2" : "García" }
{ "_id" : ObjectId("5e418e55ce31e8b17f0c7e86"), "nombre" : "Lucia", "apellido1" : "Gómez", "apellido2" : "Pérez" }
{ "_id" : ObjectId("5e418e55ce31e8b17f0c7e87"), "nombre" : "Juan", "apellido1" : "Gómez", "apellido2" : "García" }
{ "_id" : ObjectId("5e418e55ce31e8b17f0c7e88"), "nombre" : "Fernando", "apellido1" : "Pérez", "apellido2" : "Gómez" }
{ "_id" : ObjectId("5e418e55ce31e8b17f0c7e89"), "nombre" : "Juan", "apellido1" : "López", "apellido2" : "Gómez" }
{ "_id" : ObjectId("5e418e55ce31e8b17f0c7e8a"), "nombre" : "Lucia", "apellido1" : "García", "apellido2" : "Gómez" }
{ "_id" : ObjectId("5e418e55ce31e8b17f0c7e8b"), "nombre" : "Lucia", "apellido1" : "Pérez", "apellido2" : "Pérez" }
{ "_id" : ObjectId("5e418e55ce31e8b17f0c7e8c"), "nombre" : "Lucia", "apellido1" : "Pérez", "apellido2" : "López" }
{ "_id" : ObjectId("5e418e55ce31e8b17f0c7e8d"), "nombre" : "Lucia", "apellido1" : "López", "apellido2" : "Pérez" }
{ "_id" : ObjectId("5e418e55ce31e8b17f0c7e8e"), "nombre" : "Lucia", "apellido1" : "López", "apellido2" : "García" }
{ "_id" : ObjectId("5e418e55ce31e8b17f0c7e8f"), "nombre" : "María", "apellido1" : "López", "apellido2" : "García" }
{ "_id" : ObjectId("5e418e55ce31e8b17f0c7e90"), "nombre" : "María", "apellido1" : "García", "apellido2" : "López" }
{ "_id" : ObjectId("5e418e55ce31e8b17f0c7e91"), "nombre" : "Fernando", "apellido1" : "Pérez", "apellido2" : "López" }
{ "_id" : ObjectId("5e418e55ce31e8b17f0c7e92"), "nombre" : "María", "apellido1" : "Pérez", "apellido2" : "Gómez" }
Type "it" for more
> 
```

Presenta los primeros 20 documentos, si quiero ver los siguientes 20 presiono `it`:

```sh
> it
{ "_id" : ObjectId("5e418e55ce31e8b17f0c7e93"), "nombre" : "María", "apellido1" : "Gómez", "apellido2" : "Pérez" }
{ "_id" : ObjectId("5e418e55ce31e8b17f0c7e94"), "nombre" : "Fernando", "apellido1" : "López", "apellido2" : "López" }
{ "_id" : ObjectId("5e418e55ce31e8b17f0c7e95"), "nombre" : "Fernando", "apellido1" : "Gómez", "apellido2" : "López" }
{ "_id" : ObjectId("5e418e55ce31e8b17f0c7e96"), "nombre" : "Juan", "apellido1" : "Gómez", "apellido2" : "López" }
{ "_id" : ObjectId("5e418e55ce31e8b17f0c7e97"), "nombre" : "Fernando", "apellido1" : "López", "apellido2" : "García" }
{ "_id" : ObjectId("5e418e55ce31e8b17f0c7e98"), "nombre" : "María", "apellido1" : "Gómez", "apellido2" : "Gómez" }
{ "_id" : ObjectId("5e418e55ce31e8b17f0c7e99"), "nombre" : "María", "apellido1" : "Gómez", "apellido2" : "Pérez" }
{ "_id" : ObjectId("5e418e55ce31e8b17f0c7e9a"), "nombre" : "María", "apellido1" : "Pérez", "apellido2" : "Gómez" }
{ "_id" : ObjectId("5e418e55ce31e8b17f0c7e9b"), "nombre" : "Lucia", "apellido1" : "Gómez", "apellido2" : "Pérez" }
{ "_id" : ObjectId("5e418e55ce31e8b17f0c7e9c"), "nombre" : "Fernando", "apellido1" : "Gómez", "apellido2" : "López" }
{ "_id" : ObjectId("5e418e55ce31e8b17f0c7e9d"), "nombre" : "María", "apellido1" : "López", "apellido2" : "Pérez" }
{ "_id" : ObjectId("5e418e55ce31e8b17f0c7e9e"), "nombre" : "María", "apellido1" : "Pérez", "apellido2" : "García" }
{ "_id" : ObjectId("5e418e55ce31e8b17f0c7e9f"), "nombre" : "Fernando", "apellido1" : "García", "apellido2" : "Pérez" }
{ "_id" : ObjectId("5e418e55ce31e8b17f0c7ea0"), "nombre" : "María", "apellido1" : "Pérez", "apellido2" : "Gómez" }
{ "_id" : ObjectId("5e418e55ce31e8b17f0c7ea1"), "nombre" : "Lucia", "apellido1" : "Pérez", "apellido2" : "Gómez" }
{ "_id" : ObjectId("5e418e55ce31e8b17f0c7ea2"), "nombre" : "Lucia", "apellido1" : "García", "apellido2" : "Pérez" }
{ "_id" : ObjectId("5e418e55ce31e8b17f0c7ea3"), "nombre" : "Fernando", "apellido1" : "López", "apellido2" : "Pérez" }
{ "_id" : ObjectId("5e418e55ce31e8b17f0c7ea4"), "nombre" : "María", "apellido1" : "García", "apellido2" : "García" }
{ "_id" : ObjectId("5e418e55ce31e8b17f0c7ea5"), "nombre" : "Juan", "apellido1" : "García", "apellido2" : "Gómez" }
{ "_id" : ObjectId("5e418e55ce31e8b17f0c7ea6"), "nombre" : "Lucia", "apellido1" : "Gómez", "apellido2" : "García" }
Type "it" for more
> 
```
Puedo meter los resultados de una colección en una variable JS:

```sh
> let coleccionPacientes = db.pacientes.find()
> coleccionPacientes.length()
100000
> 
```
Usamos el método `length()` para conocer su tamaño en JS es una propiedad `length`.

Podemos usar el método `printjson` para imprimir su contenido en formato JSON.

```sh
> printjson(coleccionPacientes)
{
	"_id" : ObjectId("5e418e56ce31e8b17f0c9298"),
	"nombre" : "Fernando",
	"apellido1" : "López",
	"apellido2" : "Gómez"
},
{
	"_id" : ObjectId("5e418e56ce31e8b17f0c9299"),
	"nombre" : "Juan",
	"apellido1" : "Gómez",
	"apellido2" : "García"
},
{
	"_id" : ObjectId("5e418e56ce31e8b17f0c929a"),
	"nombre" : "María",
	"apellido1" : "Pérez",
	"apellido2" : "Pérez"
},
{
	"_id" : ObjectId("5e418e56ce31e8b17f0c929b"),
	"nombre" : "Juan",
	"apellido1" : "Gómez",
	"apellido2" : "Pérez"
},
....

```

Así hasta imprimir los 100000.


Puedo usar el método `pretty()` para mostrar la colección en formato JSON formateado de 20 en 20.

```sh
> db.pacientes.find().pretty()
{
	"_id" : ObjectId("5e418e55ce31e8b17f0c7e7f"),
	"nombre" : "Juan",
	"apellido1" : "García",
	"apellido2" : "López"
}
{
	"_id" : ObjectId("5e418e55ce31e8b17f0c7e80"),
	"nombre" : "María",
	"apellido1" : "Gómez",
	"apellido2" : "Pérez"
}
....
{
	"_id" : ObjectId("5e418e55ce31e8b17f0c7e92"),
	"nombre" : "María",
	"apellido1" : "Pérez",
	"apellido2" : "Gómez"
}
Type "it" for more
> 
```

Si quiero ver los siguientes 20 presiono `it`.

**Método `findOne()`**

Devuelve el primer documento.

```sh
> db.pacientes.findOne()
{
	"_id" : ObjectId("5e418e55ce31e8b17f0c7e7f"),
	"nombre" : "Juan",
	"apellido1" : "García",
	"apellido2" : "López"
}
> 
```

Veamos que no es necesario `pretty()` para que lo pinte bonito.

### Comprobación de Tipos

Existen dos formas de comprobar el tipo de un campo, igual que con JS:

* `typeof`: Nos devuelve el tipo de un campo.
* `instanceof` Compara un campo con un tipo, para ver si es o no de ese tipo.

**JSON no soporta `undefined`.

```sh
> let documento = db.pacientes.findOne()
> typeof documento.apellido1
string
> documento._id instanceof ObjectId
true
> typeof documento._id
object
```

### Ayudas

`Ctrl-C`: Cierra el Shell.
`$ mongo --help` Muestra la ayuda de Mongo.

```sh
> ^C
bye
192:MongoDB adolfodelarosa$ mongo --help
MongoDB shell version v4.2.2
usage: mongo [options] [db address] [file names (ending in .js)]
db address can be:
  foo                   foo database on local machine
  192.168.0.5/foo       foo database on 192.168.0.5 machine
  192.168.0.5:9999/foo  foo database on 192.168.0.5 machine on port 9999
  mongodb://192.168.0.5:9999/foo  connection string URI can also be used
Options:
  --ipv6                               enable IPv6 support (disabled by 
                                       default)
  --host arg                           server to connect to
  --port arg                           port to connect to
  -h [ --help ]                        show this usage information
  --version                            show version information
  --verbose                            increase verbosity
  --shell                              run the shell after executing files
  --nodb                               don't connect to mongod on startup - no 
                                       'db address' arg expected
  --norc                               will not run the ".mongorc.js" file on 
                                       start up
  --quiet                              be less chatty
  --eval arg                           evaluate javascript
  --disableJavaScriptJIT               disable the Javascript Just In Time 
                                       compiler
  --enableJavaScriptJIT                enable the Javascript Just In Time 
                                       compiler
  --disableJavaScriptProtection        allow automatic JavaScript function 
                                       marshalling
  --retryWrites                        automatically retry write operations 
                                       upon transient network errors
  --disableImplicitSessions            do not automatically create and use 
                                       implicit sessions
  --jsHeapLimitMB arg                  set the js scope's heap size limit

TLS Options:
  --tls                                use TLS for all connections
  --tlsCertificateKeyFile arg          PEM certificate/key file for TLS
  --tlsCertificateKeyFilePassword arg  Password for key in PEM file for TLS
  --tlsCAFile arg                      Certificate Authority file for TLS
  --tlsCRLFile arg                     Certificate Revocation List file for TLS
  --tlsAllowInvalidHostnames           Allow connections to servers with 
                                       non-matching hostnames
  --tlsAllowInvalidCertificates        Allow connections to servers with 
                                       invalid certificates
  --tlsFIPSMode                        Activate FIPS 140-2 mode at startup
  --tlsCertificateSelector arg         TLS Certificate in system store
  --tlsDisabledProtocols arg           Comma separated list of TLS protocols to
                                       disable [TLS1_0,TLS1_1,TLS1_2]

Authentication Options:
  -u [ --username ] arg                username for authentication
  -p [ --password ] arg                password for authentication
  --authenticationDatabase arg         user source (defaults to dbname)
  --authenticationMechanism arg        authentication mechanism
  --gssapiServiceName arg (=mongodb)   Service name to use when authenticating 
                                       using GSSAPI/Kerberos
  --gssapiHostName arg                 Remote host name to use for purpose of 
                                       GSSAPI/Kerberos authentication

FLE AWS Options:
  --awsAccessKeyId arg                 AWS Access Key for FLE Amazon KMS
  --awsSecretAccessKey arg             AWS Secret Key for FLE Amazon KMS
  --awsSessionToken arg                Optional AWS Session Token ID
  --keyVaultNamespace arg              database.collection to store encrypted 
                                       FLE parameters
  --kmsURL arg                         Test parameter to override the URL for 
                                       KMS

file names: a list of files to run. files have to end in .js and will exit after unless --shell is specified
192:MongoDB adolfodelarosa$ 
```

**Ayuda del Shell**

`> help`


```sh
> help
	db.help()                    help on db methods
	db.mycoll.help()             help on collection methods
	sh.help()                    sharding helpers
	rs.help()                    replica set helpers
	help admin                   administrative help
	help connect                 connecting to a db help
	help keys                    key shortcuts
	help misc                    misc things to know
	help mr                      mapreduce

	show dbs                     show database names
	show collections             show collections in current database
	show users                   show users in current database
	show profile                 show most recent system.profile entries with time >= 1ms
	show logs                    show the accessible logger names
	show log [name]              prints out the last segment of log in memory, 'global' is default
	use <db_name>                set current database
	db.foo.find()                list objects in collection foo
	db.foo.find( { a : 1 } )     list objects in foo where a == 1
	it                           result of the last line evaluated; use to further iterate
	DBQuery.shellBatchSize = x   set default number of items to display on shell
	exit                         quit the mongo shell
> 

```

**Ayuda de métodos de la BD**

`> db.help()`

```sh
> db.help()
DB methods:
	db.adminCommand(nameOrDocument) - switches to 'admin' db, and runs command [just calls db.runCommand(...)]
	db.aggregate([pipeline], {options}) - performs a collectionless aggregation on this database; returns a cursor
	db.auth(username, password)
	db.cloneDatabase(fromhost) - will only function with MongoDB 4.0 and below
	db.commandHelp(name) returns the help for the command
	db.copyDatabase(fromdb, todb, fromhost) - will only function with MongoDB 4.0 and below
	db.createCollection(name, {size: ..., capped: ..., max: ...})
	db.createUser(userDocument)
	db.createView(name, viewOn, [{$operator: {...}}, ...], {viewOptions})
	db.currentOp() displays currently executing operations in the db
	db.dropDatabase(writeConcern)
	db.dropUser(username)
	db.eval() - deprecated
	db.fsyncLock() flush data to disk and lock server for backups
	db.fsyncUnlock() unlocks server following a db.fsyncLock()
	db.getCollection(cname) same as db['cname'] or db.cname
	db.getCollectionInfos([filter]) - returns a list that contains the names and options of the db's collections
	db.getCollectionNames()
	db.getLastError() - just returns the err msg string
	db.getLastErrorObj() - return full status object
	db.getLogComponents()
	db.getMongo() get the server connection object
	db.getMongo().setSlaveOk() allow queries on a replication slave server
	db.getName()
	db.getProfilingLevel() - deprecated
	db.getProfilingStatus() - returns if profiling is on and slow threshold
	db.getReplicationInfo()
	db.getSiblingDB(name) get the db at the same server as this one
	db.getWriteConcern() - returns the write concern used for any operations on this db, inherited from server object if set
	db.hostInfo() get details about the server's host
	db.isMaster() check replica primary status
	db.killOp(opid) kills the current operation in the db
	db.listCommands() lists all the db commands
	db.loadServerScripts() loads all the scripts in db.system.js
	db.logout()
	db.printCollectionStats()
	db.printReplicationInfo()
	db.printShardingStatus()
	db.printSlaveReplicationInfo()
	db.resetError()
	db.runCommand(cmdObj) run a database command.  if cmdObj is a string, turns it into {cmdObj: 1}
	db.serverStatus()
	db.setLogLevel(level,<component>)
	db.setProfilingLevel(level,slowms) 0=off 1=slow 2=all
	db.setVerboseShell(flag) display extra information in shell output
	db.setWriteConcern(<write concern doc>) - sets the write concern for writes to the db
	db.shutdownServer()
	db.stats()
	db.unsetWriteConcern(<write concern doc>) - unsets the write concern for writes to the db
	db.version() current version of the server
	db.watch() - opens a change stream cursor for a database to report on all  changes to its non-system collections.
> 
```

**Ayuda de Colecciones**

`db.<colección>.help()`

```sh
> db.pacientes.help()
DBCollection help
	db.pacientes.find().help() - show DBCursor help
	db.pacientes.bulkWrite( operations, <optional params> ) - bulk execute write operations, optional parameters are: w, wtimeout, j
	db.pacientes.count( query = {}, <optional params> ) - count the number of documents that matches the query, optional parameters are: limit, skip, hint, maxTimeMS
	db.pacientes.countDocuments( query = {}, <optional params> ) - count the number of documents that matches the query, optional parameters are: limit, skip, hint, maxTimeMS
	db.pacientes.estimatedDocumentCount( <optional params> ) - estimate the document count using collection metadata, optional parameters are: maxTimeMS
	db.pacientes.convertToCapped(maxBytes) - calls {convertToCapped:'pacientes', size:maxBytes}} command
	db.pacientes.createIndex(keypattern[,options])
	db.pacientes.createIndexes([keypatterns], <options>)
	db.pacientes.dataSize()
	db.pacientes.deleteOne( filter, <optional params> ) - delete first matching document, optional parameters are: w, wtimeout, j
	db.pacientes.deleteMany( filter, <optional params> ) - delete all matching documents, optional parameters are: w, wtimeout, j
	db.pacientes.distinct( key, query, <optional params> ) - e.g. db.pacientes.distinct( 'x' ), optional parameters are: maxTimeMS
	db.pacientes.drop() drop the collection
	db.pacientes.dropIndex(index) - e.g. db.pacientes.dropIndex( "indexName" ) or db.pacientes.dropIndex( { "indexKey" : 1 } )
	db.pacientes.dropIndexes()
	db.pacientes.ensureIndex(keypattern[,options]) - DEPRECATED, use createIndex() instead
	db.pacientes.explain().help() - show explain help
	db.pacientes.reIndex()
	db.pacientes.find([query],[fields]) - query is an optional query filter. fields is optional set of fields to return.
	                                              e.g. db.pacientes.find( {x:77} , {name:1, x:1} )
	db.pacientes.find(...).count()
	db.pacientes.find(...).limit(n)
	db.pacientes.find(...).skip(n)
	db.pacientes.find(...).sort(...)
	db.pacientes.findOne([query], [fields], [options], [readConcern])
	db.pacientes.findOneAndDelete( filter, <optional params> ) - delete first matching document, optional parameters are: projection, sort, maxTimeMS
	db.pacientes.findOneAndReplace( filter, replacement, <optional params> ) - replace first matching document, optional parameters are: projection, sort, maxTimeMS, upsert, returnNewDocument
	db.pacientes.findOneAndUpdate( filter, <update object or pipeline>, <optional params> ) - update first matching document, optional parameters are: projection, sort, maxTimeMS, upsert, returnNewDocument
	db.pacientes.getDB() get DB object associated with collection
	db.pacientes.getPlanCache() get query plan cache associated with collection
	db.pacientes.getIndexes()
	db.pacientes.insert(obj)
	db.pacientes.insertOne( obj, <optional params> ) - insert a document, optional parameters are: w, wtimeout, j
	db.pacientes.insertMany( [objects], <optional params> ) - insert multiple documents, optional parameters are: w, wtimeout, j
	db.pacientes.mapReduce( mapFunction , reduceFunction , <optional params> )
	db.pacientes.aggregate( [pipeline], <optional params> ) - performs an aggregation on a collection; returns a cursor
	db.pacientes.remove(query)
	db.pacientes.replaceOne( filter, replacement, <optional params> ) - replace the first matching document, optional parameters are: upsert, w, wtimeout, j
	db.pacientes.renameCollection( newName , <dropTarget> ) renames the collection.
	db.pacientes.runCommand( name , <options> ) runs a db command with the given name where the first param is the collection name
	db.pacientes.save(obj)
	db.pacientes.stats({scale: N, indexDetails: true/false, indexDetailsKey: <index key>, indexDetailsName: <index name>})
	db.pacientes.storageSize() - includes free space allocated to this collection
	db.pacientes.totalIndexSize() - size in bytes of all the indexes
	db.pacientes.totalSize() - storage allocated for all data and indexes
	db.pacientes.update( query, <update object or pipeline>[, upsert_bool, multi_bool] ) - instead of two flags, you can pass an object with fields: upsert, multi, hint
	db.pacientes.updateOne( filter, <update object or pipeline>, <optional params> ) - update the first matching document, optional parameters are: upsert, w, wtimeout, j, hint
	db.pacientes.updateMany( filter, <update object or pipeline>, <optional params> ) - update all matching documents, optional parameters are: upsert, w, wtimeout, j, hint
	db.pacientes.validate( <full> ) - SLOW
	db.pacientes.getShardVersion() - only for use with sharding
	db.pacientes.getShardDistribution() - prints statistics about data distribution in the cluster
	db.pacientes.getSplitKeysForChunks( <maxChunkSize> ) - calculates split points over all chunks and returns splitter function
	db.pacientes.getWriteConcern() - returns the write concern used for any operations on this collection, inherited from server/db if set
	db.pacientes.setWriteConcern( <write concern doc> ) - sets the write concern for writes to the collection
	db.pacientes.unsetWriteConcern( <write concern doc> ) - unsets the write concern for writes to the collection
	db.pacientes.latencyStats() - display operation latency histograms for this collection
> 

```

**Código de Métodos**

Si quiero ver el código de un método, lo pongo sin parentesis.

`db.<coleccion>.<metodo>`

```sh
> db.pacientes.find
function(query, fields, limit, skip, batchSize, options) {
    var cursor = new DBQuery(this._mongo,
                             this._db,
                             this,
                             this._fullName,
                             this._massageObject(query),
                             fields,
                             limit,
                             skip,
                             batchSize,
                             options || this.getQueryOptions());

    {
        const session = this.getDB().getSession();

        const readPreference = session._getSessionAwareClient().getReadPreference(session);
        if (readPreference !== null) {
            cursor.readPref(readPreference.mode, readPreference.tags);
        }

        const readConcern = session._getSessionAwareClient().getReadConcern(session);
        if (readConcern !== null) {
            cursor.readConcern(readConcern.level);
        }
    }

    return cursor;
}
```

### Cierre del Shell

Puedo cerrar el shell con:

* `Ctrl + c`
* Método `quit()`

### Documentación Oficial de MongoDB

[MongoDB Docs](https://docs.mongodb.com/)

[The mongo Shell](https://docs.mongodb.com/manual/mongo/index.html)

[mongo Shell Methods](https://docs.mongodb.com/manual/reference/method/index.html)

