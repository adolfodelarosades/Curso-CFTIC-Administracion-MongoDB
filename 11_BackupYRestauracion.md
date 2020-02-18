# 11 Backup y Restauración

1. Backup con Atlas. (plataforma Cloud Computing MongoDB)
   * Continuous backup (herramienta propia de Atlas)
   * Cloud provider snapshop (herramienta del proveedor)

2. Backup con Cloud Manager o Ops Manager.
   
   Con cualquiera de las dos herramientas de operaciones de mongo.

3. Backup by copying underlying data files.
   
   Cualquier sistema de backup sobre los volúmenes.

4. Backup with MongoDB Tools

   Herramientas de MongoDB para proyectos no muy grandes (binarios):
   
   * mongodump
   * mongorestore
   

## Mongodump

Realiza backup del servidor completo, base de datos, colecciones.

Sintaxis:

```sh
mongodump <opciones>
--host <direccón del servidor>
--port <puerto>
--out= <ruta de salida>
--db <base-de-datos>
--collection <colección>
--query <json-consulta>
--oplog // Almacena un oplog.json por el tiempo que tarde para cosas que se generen mientras se hace el respaldo
```

## mongorestore

```sh
mongorestore <opciones>
--host <direccón del servidor>
--port <puerto>
  <ruta de archivos de backup>
  - directorio de todo
  - archivo .bson para colleccion
--db <base-de-datos>
--collection <colección>
--oplogReplay
```

Creamos las carpetas data2\server y data2\backup

Levantar nuestro nuevo servidor
```sh
C:\Users\manana>mongod --port 27300 --dbpath data2\server
```

Levanto la shell para ese servidor:

```sh
C:\Users\manana>mongo --port 27300
```

Creamos BD y le metemos 1000 registros.

```sh
> use getafe
switched to db getafe
> for(i=0; i < 1000; i++){
... db.foo.insert({a: i})
... }
WriteResult({ "nInserted" : 1 })
>

> use mostoles
switched to db mostoles
> for(i=0; i < 1000; i++){ db.foo2.insert({b: i}) }
WriteResult({ "nInserted" : 1 })
>
```

Creamos un indice para comprobar que tambieén se respaldan los indices:

```sh
> db.foo2.createIndex({ b: 1})
{
        "createdCollectionAutomatically" : false,
        "numIndexesBefore" : 1,
        "numIndexesAfter" : 2,
        "ok" : 1
}
>     
```

¿Cómo hacemos el respaldo?

Desde la terminal

```sh
C:\Users\manana>mongodump --port 27300 --out=data2\backup\bk17022020
2020-02-17T13:36:59.823+0100    writing admin.system.version to
2020-02-17T13:37:00.172+0100    done dumping admin.system.version (1 document)
2020-02-17T13:37:00.172+0100    writing getafe.foo to
2020-02-17T13:37:00.172+0100    writing mostoles.foo2 to
2020-02-17T13:37:00.176+0100    done dumping getafe.foo (1000 documents)
2020-02-17T13:37:00.479+0100    done dumping mostoles.foo2 (1000 documents)

C:\Users\manana>             
```

Reviso en la carpeta backup

<img src="/images/backup.png">



Borramos Colleccion con 
```sh
> show dbs
admin     0.000GB
config    0.000GB
getafe    0.000GB
local     0.000GB
mostoles  0.000GB
> use mostoles
switched to db mostoles
> show collections
foo2
> db.foo2.drop()
true
>      
```

Como Restauramos:

```sh
C:\Users\manana>mongorestore --port 27300 data2\backup\bk17022020 
...
2020-02-17T13:52:46.539+0100    no indexes to restore
2020-02-17T13:52:46.540+0100    finished restoring getafe.foo (0 documents, 1000 failures)
2020-02-17T13:52:46.540+0100    1000 document(s) restored successfully. 1000 document(s) failed to restore.
```

Vuelvo a entrar al shell.

```sh
C:\Users\manana>mongo --port 27300
```

Reviso si ya existen mis datos otra vez:

```sh
> show dbs
admin     0.000GB
config    0.000GB
getafe    0.000GB
local     0.000GB
mostoles  0.000GB
> use mostoles
switched to db mostoles
> show collections
foo2
> db.foo2.find().count()
1000
>       
```

Para hacerlos sobre una sola colección

```sh
C:\Users\manana>mongodump --port 27300 --out=data2\backup\bkgetafefoo17022020 --db "getafe" --collection "foo"
2020-02-17T13:58:38.622+0100    writing getafe.foo to
2020-02-17T13:58:38.705+0100    done dumping getafe.foo (1000 documents)
```

<img src="/images/backupcoleccion.png">

Entro y borro 5 documentos

```sh
> use getafe
switched to db getafe
> db.foo.deleteOne({})
{ "acknowledged" : true, "deletedCount" : 1 }
> db.foo.deleteOne({})
{ "acknowledged" : true, "deletedCount" : 1 }
>
> db.foo.deleteOne({})
{ "acknowledged" : true, "deletedCount" : 1 }
> db.foo.deleteOne({})
{ "acknowledged" : true, "deletedCount" : 1 }
> db.foo.deleteOne({})
{ "acknowledged" : true, "deletedCount" : 1 }
>            
```

Desde la consola restauro

```sh
C:\Users\manana>mongorestore --port 27300 data2\backup\bkgetafefoo17022020\getafe\foo.bson --db "getafe" --collection "foo" 
....
ex: _id_ dup key: { _id: ObjectId('5e4a8698f9571bc97a8de385') }
2020-02-17T14:03:06.022+0100    no indexes to restore
2020-02-17T14:03:06.023+0100    finished restoring getafe.foo (5 documents, 995 failures)
2020-02-17T14:03:06.023+0100    5 document(s) restored successfully. 995 document(s) failed to restore.
```

Me restaura los 5 archivos que yo había borrado.


### Mongodump y mongorestore
Exportación e importación de datos (binarios)

```sh
mongoexport` <opciones>
--host <direccion:puerto> |
--uri <formato-uri>
--db=<nombre-basededatos>
--collection=<nombre-coleccion>
--fields=<campo1, campo2> *Obligatorio en caso de exportar en .csv
--type=csv
--noHeaderLine
--out=ruta/<archivo>.json | .csv

```

#### URI Mongodb <string>
  
`"mongodb://<direccion>:<puerto>"`
`"mongodb://<direccion>:<puerto>/<basededatos>"`

Lo que usan los dives para conectarse

Levanto servidor

```sh
C:\Users\manana>mongod


> use maraton
switched to db maraton
> show collections
foo
nombresSenior
participantes
> db.participantes.find().count()
1000000
>
```
solo comprobe que maraton tenga datos en la colección `participantes`.

Desde la terminal

Hago la exportación

```sh
C:\Users\manana>mongoexport --db="maraton" --collection="participantes" --out=maraton\participantes.json
2020-02-18T09:22:19.634+0100    connected to: mongodb://localhost/
2020-02-18T09:22:20.715+0100    [#.......................]  maraton.participantes  48000/1000000  (4.8%)
2020-02-18T09:22:21.714+0100    [###.....................]  maraton.participantes  128000/1000000  (12.8%)
2020-02-18T09:22:22.716+0100    [####....................]  maraton.participantes  192000/1000000  (19.2%)
2020-02-18T09:22:23.715+0100    [######..................]  maraton.participantes  256000/1000000  (25.6%)
2020-02-18T09:22:24.716+0100    [#######.................]  maraton.participantes  328000/1000000  (32.8%)
2020-02-18T09:22:25.715+0100    [#########...............]  maraton.participantes  392000/1000000  (39.2%)
2020-02-18T09:22:26.716+0100    [###########.............]  maraton.participantes  464000/1000000  (46.4%)
2020-02-18T09:22:27.715+0100    [############............]  maraton.participantes  528000/1000000  (52.8%)
2020-02-18T09:22:28.716+0100    [##############..........]  maraton.participantes  608000/1000000  (60.8%)
2020-02-18T09:22:32.021+0100    [###################.....]  maraton.participantes  816000/1000000  (81.6%)
2020-02-18T09:22:32.714+0100    [####################....]  maraton.participantes  872000/1000000  (87.2%)
2020-02-18T09:22:33.715+0100    [######################..]  maraton.participantes  936000/1000000  (93.6%)
2020-02-18T09:22:34.461+0100    [########################]  maraton.participantes  1000000/1000000  (100.0%)
2020-02-18T09:22:34.461+0100    exported 1000000 records

C:\Users\manana>
```

Si solo quiero algunos campos:

```sh
C:\Users\manana>mongoexport --db="maraton" --collection="participantes" --fields="nombre,dni"  --out=maraton\participantes-nombre-dni.json
2020-02-18T09:23:57.713+0100    connected to: mongodb://localhost/
2020-02-18T09:23:58.797+0100    [#.......................]  maraton.participantes  56000/1000000  (5.6%)
2020-02-18T09:23:59.796+0100    [###.....................]  maraton.participantes  160000/1000000  (16.0%)
2020-02-18T09:24:00.796+0100    [#####...................]  maraton.participantes  248000/1000000  (24.8%)
2020-02-18T09:24:01.797+0100    [#######.................]  maraton.participantes  328000/1000000  (32.8%)
2020-02-18T09:24:02.798+0100    [##########..............]  maraton.participantes  432000/1000000  (43.2%)
2020-02-18T09:24:03.796+0100    [###########.............]  maraton.participantes  496000/1000000  (49.6%)
2020-02-18T09:24:04.797+0100    [##############..........]  maraton.participantes  600000/1000000  (60.0%)
2020-02-18T09:24:05.796+0100    [################........]  maraton.participantes  704000/1000000  (70.4%)
2020-02-18T09:24:06.797+0100    [##################......]  maraton.participantes  776000/1000000  (77.6%)
2020-02-18T09:24:07.796+0100    [#####################...]  maraton.participantes  880000/1000000  (88.0%)
2020-02-18T09:24:08.797+0100    [#######################.]  maraton.participantes  984000/1000000  (98.4%)
2020-02-18T09:24:08.887+0100    [########################]  maraton.participantes  1000000/1000000  (100.0%)
2020-02-18T09:24:08.887+0100    exported 1000000 records

C:\Users\manana>   
```

Si quiero en CSV:

```sh
C:\Users\manana>mongoexport --db="maraton" --collection="participantes" --fields="nombre,dni" --type=csv  --out=maraton\participantes-nombre-dni.csv
2020-02-18T09:25:37.513+0100    connected to: mongodb://localhost/
2020-02-18T09:25:38.591+0100    [####....................]  maraton.participantes  176000/1000000  (17.6%)
2020-02-18T09:25:39.591+0100    [########................]  maraton.participantes  352000/1000000  (35.2%)
2020-02-18T09:25:40.592+0100    [############............]  maraton.participantes  528000/1000000  (52.8%)
2020-02-18T09:25:41.591+0100    [#################.......]  maraton.participantes  744000/1000000  (74.4%)
2020-02-18T09:25:42.562+0100    [########################]  maraton.participantes  1000000/1000000  (100.0%)
2020-02-18T09:25:42.562+0100    exported 1000000 records

C:\Users\manana>
```

Sin Header(Cabecera):

```sh
C:\Users\manana>mongoexport --db="maraton" --collection="participantes" --fields="nombre,dni" --type=csv --noHeaderLine --out=maraton\participantes-nombre-dni.csv
2020-02-18T09:27:04.749+0100    connected to: mongodb://localhost/
2020-02-18T09:27:05.829+0100    [####....................]  maraton.participantes  176000/1000000  (17.6%)
2020-02-18T09:27:06.830+0100    [########................]  maraton.participantes  352000/1000000  (35.2%)
2020-02-18T09:27:07.829+0100    [############............]  maraton.participantes  536000/1000000  (53.6%)
2020-02-18T09:27:08.829+0100    [#################.......]  maraton.participantes  744000/1000000  (74.4%)
2020-02-18T09:27:09.818+0100    [########################]  maraton.participantes  1000000/1000000  (100.0%)
2020-02-18T09:27:09.819+0100    exported 1000000 records

```

<img src="/images/export.png">


Usando el HOST

```sh
C:\Users\manana>mongoexport --host="localhost:27017" --db="maraton" --collection="participantes" --out=maraton\participantes-host.json
2020-02-18T09:42:20.944+0100    connected to: mongodb://localhost:27017/
2020-02-18T09:42:22.027+0100    [#.......................]  maraton.participantes  56000/1000000  (5.6%)
2020-02-18T09:42:23.028+0100    [###.....................]  maraton.participantes  128000/1000000  (12.8%)
2020-02-18T09:42:24.026+0100    [####....................]  maraton.participantes  200000/1000000  (20.0%)
2020-02-18T09:42:25.028+0100    [######..................]  maraton.participantes  264000/1000000  (26.4%)
2020-02-18T09:42:26.026+0100    [########................]  maraton.participantes  336000/1000000  (33.6%)
2020-02-18T09:42:27.028+0100    [#########...............]  maraton.participantes  400000/1000000  (40.0%)
2020-02-18T09:42:28.026+0100    [###########.............]  maraton.participantes  480000/1000000  (48.0%)
2020-02-18T09:42:34.706+0100    [######################..]  maraton.participantes  952000/1000000  (95.2%)
2020-02-18T09:42:35.026+0100    [#######################.]  maraton.participantes  976000/1000000  (97.6%)
2020-02-18T09:42:35.330+0100    [########################]  maraton.participantes  1000000/1000000  (100.0%)
2020-02-18T09:42:35.330+0100    exported 1000000 records

C:\Users\manana>
```

Usando URI:

```sh
C:\Users\manana>mongoexport --uri="mongodb://localhost:27017/maraton" --collection="participantes" --out=maraton\participantes-uri.json
2020-02-18T09:51:48.160+0100    connected to: mongodb://localhost:27017/maraton
2020-02-18T09:51:49.270+0100    [#.......................]  maraton.participantes  64000/1000000  (6.4%)
2020-02-18T09:51:50.272+0100    [###.....................]  maraton.participantes  128000/1000000  (12.8%)
2020-02-18T09:51:51.270+0100    [####....................]  maraton.participantes  208000/1000000  (20.8%)
2020-02-18T09:51:52.272+0100    [######..................]  maraton.participantes  272000/1000000  (27.2%)
2020-02-18T09:51:53.270+0100    [########................]  maraton.participantes  352000/1000000  (35.2%)
2020-02-18T09:51:54.271+0100    [#########...............]  maraton.participantes  416000/1000000  (41.6%)
2020-02-18T09:51:55.272+0100    [###########.............]  maraton.participantes  496000/1000000  (49.6%)
2020-02-18T09:51:56.270+0100    [#############...........]  maraton.participantes  568000/1000000  (56.8%)
2020-02-18T09:51:57.272+0100    [###############.........]  maraton.participantes  640000/1000000  (64.0%)
2020-02-18T09:51:58.270+0100    [#################.......]  maraton.participantes  712000/1000000  (71.2%)
2020-02-18T09:51:59.272+0100    [##################......]  maraton.participantes  776000/1000000  (77.6%)
2020-02-18T09:52:00.272+0100    [####################....]  maraton.participantes  848000/1000000  (84.8%)
2020-02-18T09:52:01.270+0100    [######################..]  maraton.participantes  920000/1000000  (92.0%)
2020-02-18T09:52:02.235+0100    [########################]  maraton.participantes  1000000/1000000  (100.0%)
2020-02-18T09:52:02.235+0100    exported 1000000 records

C:\Users\manana>
```

Usando usuario/password:

```sh
C:\Users\manana>mongoexport --uri="mongodb://usuario:password@localhost:27017/maraton" --collection="participantes" --out=maraton\participantes-uri.json
```

### Mongoimport

```sh
mongoimport <opciones>
--host <direccion:puerto> | --uri <formato-uri>
--db=<nombre-basededatos>
--collection=<nombre-coleccion>
--type=<json|csv|tsv>
--file=<archivo>.json | .csv | .tsv
-mode=<insert|upsert|merge>
  
  En todos los casos el campo de comparación es _id
  insert => Inserta con operación de escritura desordenada
  upsert => Modifica el documento que encuentra o lo crea si no existe ese _id.
  merge => Fusiona los campos de los registros que encuentra o crea el registro si no existe ese _id
  
```

Entro al Shell y creo BD y colección

```sh
> use colegio
switched to db colegio
> db.createCollection("alumnos")
{ "ok" : 1 }
>

```

Tengo un JSON que es el que quiero importar esta en la carpeta `colegio`
```sh
{"_id": 1, "nombre": "Laura"}
{"_id": 2, "nombre": "Juan"}
{"_id": 3, "nombre": "Lucía"}
{"_id": 4, "nombre": "Carlos"}
{"_id": 5, "nombre": "Sara"}
```

Importar el JSON en la BD colegio colección alumnos:

```sh
C:\Users\manana>mongoimport --db="colegio" --collection="alumnos" --file="colegio\alumnos.json"
2020-02-18T10:18:40.077+0100    connected to: mongodb://localhost/
2020-02-18T10:18:40.158+0100    5 document(s) imported successfully. 0 document(s) failed to import.

C:\Users\manana>
```

Si me meto en la shell puedo comprobar:

```sh
> use colegio
switched to db colegio
> db.alumnos.find()
{ "_id" : 2, "nombre" : "Juan" }
{ "_id" : 5, "nombre" : "Sara" }
{ "_id" : 4, "nombre" : "Carlos" }
{ "_id" : 1, "nombre" : "Laura" }
{ "_id" : 3, "nombre" : "Lucía" }
>
```


Que pasa si tengo un id repetido

```sh
{"_id": 6, "nombre": "María"}
{"_id": 7, "nombre": "Pedro"}
{"_id": 3, "nombre": "Luis"}
{"_id": 8, "nombre": "Fernando"}
{"_id": 9, "nombre": "Raquel"}
```

Cargo este archivo

```sh
C:\Users\manana>mongoimport --db="colegio" --collection="alumnos" --file="colegio\alumnos.json"
2020-02-18T10:28:11.263+0100    connected to: mongodb://localhost/
2020-02-18T10:28:11.344+0100    continuing through error: E11000 duplicate key error collection: colegio.alumnos index: _id_ dup key: { _id: 3 }
2020-02-18T10:28:11.348+0100    4 document(s) imported successfully. 1 document(s) failed to import.

C:\Users\manana>

```

Me marca un error del id repetido, pero continua con el resto.

Si lo verifico en la shell:

```sh
> db.alumnos.find()
{ "_id" : 2, "nombre" : "Juan" }
{ "_id" : 5, "nombre" : "Sara" }
{ "_id" : 4, "nombre" : "Carlos" }
{ "_id" : 1, "nombre" : "Laura" }
{ "_id" : 3, "nombre" : "Lucía" }
{ "_id" : 6, "nombre" : "María" }
{ "_id" : 8, "nombre" : "Fernando" }
{ "_id" : 7, "nombre" : "Pedro" }
{ "_id" : 9, "nombre" : "Raquel" }
>
```

Modo upsert, cambia el documento o lo crea si no existe

```sh
{"_id": 1, "nombre": "María Rosa"}
{"_id": 2, "nombre": "Pedro José"}
{"_id": 3, "nombre": "Luis Carlos"}
{"_id": 4, "nombre": "Jose Luis"}
{"_id": 20, "nombre": "José María"}
```

Upser los cambia y si no existe lo crea.

```sh
C:\Users\manana>mongoimport --db="colegio" --collection="alumnos" --file="colegio\alumnos.json" --mode=upsert
2020-02-18T10:36:02.660+0100    connected to: mongodb://localhost/
2020-02-18T10:36:02.742+0100    5 document(s) imported successfully. 0 document(s) failed to import.

C:\Users\manana>
```

En la shell

```sh
> use colegio
switched to db colegio
> db.alumnos.find()
{ "_id" : 2, "nombre" : "Pedro José" }
{ "_id" : 5, "nombre" : "Sara" }
{ "_id" : 4, "nombre" : "Jose Luis" }
{ "_id" : 1, "nombre" : "María Rosa" }
{ "_id" : 3, "nombre" : "Luis Carlos" }
{ "_id" : 6, "nombre" : "María" }
{ "_id" : 8, "nombre" : "Fernando" }
{ "_id" : 7, "nombre" : "Pedro" }
{ "_id" : 9, "nombre" : "Raquel" }
{ "_id" : 20, "nombre" : "José María" }
>
```
Me cambio los que existian y me ingreso los que no existian.

Merge


```sh
{"_id": 1, "apellidos": "Pérea"}
{"_id": 2, "apellidos": "Gómez"}
{"_id": 3, "apellidos": "López"}
{"_id": 4, "apellidos": "García"}
{"_id": 30, "apellidos": "López"}

```

```sh

C:\Users\manana>mongoimport --db="colegio" --collection="alumnos" --file="colegio\alumnos.json" --mode=merge
2020-02-18T10:43:30.499+0100    connected to: mongodb://localhost/
2020-02-18T10:43:30.580+0100    5 document(s) imported successfully. 0 document(s) failed to import.
```

Comprobamos en el Shell
```sh
> use colegio
switched to db colegio
> db.alumnos.find()
{ "_id" : 2, "nombre" : "Pedro José", "apellidos" : "Gómez" }
{ "_id" : 5, "nombre" : "Sara" }
{ "_id" : 4, "nombre" : "Jose Luis", "apellidos" : "García" }
{ "_id" : 1, "nombre" : "María Rosa", "apellidos" : "Pérea" }
{ "_id" : 3, "nombre" : "Luis Carlos", "apellidos" : "López" }
{ "_id" : 6, "nombre" : "María" }
{ "_id" : 8, "nombre" : "Fernando" }
{ "_id" : 7, "nombre" : "Pedro" }
{ "_id" : 9, "nombre" : "Raquel" }
{ "_id" : 20, "nombre" : "José María" }
{ "_id" : 30, "apellidos" : "López" }
>

```

Los que ya existian les agrega lo que no tenia y si no existia lo inserta.

Cargo restaurants.json


```sh
C:\Users\manana>mongoimport --db="newyork" --collection="restaurants" --file="restaurant.json"
2020-02-18T10:53:26.366+0100    connected to: mongodb://localhost/
2020-02-18T10:53:27.309+0100    25359 document(s) imported successfully. 0 document(s) failed to import.

C:\Users\manana>
```


```sh
> use newyork
switched to db newyork
> show collections
restaurants
>
```

<img src="/images/compass-restaurantes.png">

<img src="/images/compass-schema.png">

<img src="/images/compass-circulo.png">

Me genera una consulta:

```sh
{'address.coord': {$geoWithin: { $centerSphere: [ [ -73.99612354056683, 40.74378514302997 ], 0.00038464636632964445 ]}}}
```

Si la analizo me restringe la vista

<img src="/images/compass-circulo2.png">

<img src="/images/compass-rombo.png">

<img src="/images/compas-rombo2.png">


## Motor de Almacenamiento en MongoDB

Existen dos motores de almacenamiento

* WiredTiger (Default)
* in-memory

### WiredTiger :skull:

WireTiger MVCC( Control de concurrencia de conversión múltiple)

¿Como realiza WiredTiger las operaciones de escritura?

1. Sin tener activado el journal (desaconsejado)
   mongod... --nojournal*
   Si el servidor forma parte de un replica set no podemos levantarlo sin journal
2. Con Jounal (por defecto)
 
 **explicacion apuntes 18022020**
 
 
 #### Uso de la memoria de WiredTiger :skull:
 
 * WiredTiger internal cache
 * Cantidad de memoria usa por defecto, mayor de los siguientes valores:
    * 50%(RAM - 1GB)
    * 256 MB
 
**explicacion apuntes 18022020-2**
 
 Por ejemplo, RAM de 4 GB sería:
 
 0,5 * (4 - 1) = 1,5 GB
 
 
 ## RENDIMIENTO
 
 Mongo da herramientas básicas que puedo consumir para crea r una aplicación. Conviene contratar sus herramientas graficas.
 
 
 ### Herramienta Mongostat 
 (Binario)
 
 Pequeño resumen del status del servidor, devuelve las operaciones cada 1s por defecto.
 
 ```sh
 C:\Users\manana>mongostat
insert query update delete getmore command dirty used flushes vsize  res qrw arw net_in net_out conn                time
    *0    *0     *0     *0       0     0|0  0.0% 1.1%       0 5.54G 346M 0|0 1|0   111b   35.5k    3 Feb 18 12:40:36.838
    *0    *0     *0     *0       0     0|0  0.0% 1.1%       0 5.54G 346M 0|0 1|0   111b   35.4k    3 Feb 18 12:40:37.840
    *0    *0     *0     *0       0     1|0  0.0% 1.1%       0 5.54G 346M 0|0 1|0   112b   35.6k    3 Feb 18 12:40:38.837
    *0    *0     *0     *0       0     1|0  0.0% 1.1%       0 5.54G 346M 0|0 1|0   112b   35.5k    3 Feb 18 12:40:39.837
insert query update delete getmore command dirty used flushes vsize  res qrw arw net_in net_out conn                time
    *0    *0     *0     *0       0     0|0  0.0% 1.1%       0 5.54G 346M 0|0 1|0   111b   35.5k    3 Feb 18 12:41:27.838
    *0    *0     *0     *0       0     1|0  0.0% 1.1%       0 5.54G 346M 0|0 1|0   112b   35.5k    3 Feb 18 12:4
    *0    *0     *0     *0       0     1|0  0.0% 1.1%       0 5.54G 346M 0|0 1|0   112b   35.5k    3 Feb 18 12:42:04.838
2020-02-18T12:42:05.275+0100    signal 'interrupt' received; forcefully terminating
 ```
 
### Herramienta Mongotop

Devuelve a nivel de colección el tiempo empleado en las operaciones de lectura escritura.

```sh
C:\Users\manana>mongotop 5
2020-02-18T12:48:51.823+0100    connected to: mongodb://localhost/

                    ns    total    read    write    2020-02-18T12:48:56+01:00
    admin.system.roles      0ms     0ms      0ms
  admin.system.version      0ms     0ms      0ms
       colegio.alumnos      0ms     0ms      0ms
config.system.sessions      0ms     0ms      0ms
   config.transactions      0ms     0ms      0ms
        local.oplog.rs      0ms     0ms      0ms
  local.system.replset      0ms     0ms      0ms
 maraton.participantes      0ms     0ms      0ms
   newyork.restaurants      0ms     0ms      0ms

```
  
### Server Status ( a nivel de BD).

[serverStatus](https://docs.mongodb.com/manual/reference/command/serverStatus/index.html)

db.serverStatus()

[Output](https://docs.mongodb.com/manual/reference/command/serverStatus/index.html#output)
```sh
> db.serverStatus()
{
        "host" : "G24-EQ09",
        "version" : "4.2.2",
        "process" : "mongod",
        "pid" : NumberLong(9976),
        "uptime" : 12931,
        "uptimeMillis" : NumberLong(12930528),
        "uptimeEstimate" : NumberLong(12930),
        "localTime" : ISODate("2020-02-18T11:52:11.481Z"),
        "asserts" : {
                "regular" : 0,
                "warning" : 0,
                "msg" : 0,
                "user" : 10,
                "rollovers" : 0
        },
        "connections" : {
                "current" : 2,
                "available" : 999998,
                "totalCreated" : 278,
                "active" : 1
        },
        "electionMetrics" : {
                "stepUpCmd" : {
                        "called" : NumberLong(0),
                        "successful" : NumberLong(0)
                },
                "priorityTakeover" : {
                        "called" : NumberLong(0),
                        "successful" : NumberLong(0)
                },
                "catchUpTakeover" : {
                        "called" : NumberLong(0),
                        "successful" : NumberLong(0)
                },
                "electionTimeout" : {
                        "called" : NumberLong(0),
                        "successful" : NumberLong(0)
                },
                "freezeTimeout" : {
                        "called" : NumberLong(0),
                        "successful" : NumberLong(0)
                },
                "numStepDownsCausedByHigherTerm" : NumberLong(0),
                "numCatchUps" : NumberLong(0),
                "numCatchUpsSucceeded" : NumberLong(0),
                "numCatchUpsAlreadyCaughtUp" : NumberLong(0),
                "numCatchUpsSkipped" : NumberLong(0),
                "numCatchUpsTimedOut" : NumberLong(0),
                "numCatchUpsFailedWithError" : NumberLong(0),
                "numCatchUpsFailedWithNewTerm" : NumberLong(0),
                "numCatchUpsFailedWithReplSetAbortPrimaryCatchUpCmd" : NumberLong(0),
                "averageCatchUpOps" : 0
        },
        "extra_info" : {
                "note" : "fields vary by platform",
                "page_faults" : 255771,
                "usagePageFileMB" : 367,
                "totalPageFileMB" : 37384,
                "availPageFileMB" : 29549,
                "ramMB" : 32520
        },
        "flowControl" : {
                "enabled" : true,
                "targetRateLimit" : 1000000000,
                "timeAcquiringMicros" : NumberLong(402),
                "locksPerOp" : 0,
                "sustainerRate" : 0,
                "isLagged" : false,
                "isLaggedCount" : 0,
                "isLaggedTimeMicros" : NumberLong(0)
        },
        "freeMonitoring" : {
                "state" : "undecided"
        },
        "globalLock" : {
                "totalTime" : NumberLong("12930526000"),
                "currentQueue" : {
                        "total" : 0,
                        "readers" : 0,
                        "writers" : 0
                },
                "activeClients" : {
                        "total" : 0,
                        "readers" : 0,
                        "writers" : 0
                }
        },
        "locks" : {
                "ParallelBatchWriterMode" : {
                        "acquireCount" : {
                                "r" : NumberLong(2024)
                        }
                },
                "ReplicationStateTransition" : {
                        "acquireCount" : {
                                "w" : NumberLong(91298)
                        }
                },
                "Global" : {
                        "acquireCount" : {
                                "r" : NumberLong(90160),
                                "w" : NumberLong(1134),
                                "W" : NumberLong(4)
                        }
                },
                "Database" : {
                        "acquireCount" : {
                                "r" : NumberLong(64237),
                                "w" : NumberLong(1129),
                                "W" : NumberLong(5)
                        }
                },
                "Collection" : {
                        "acquireCount" : {
                                "r" : NumberLong(64461),
                                "w" : NumberLong(1127),
                                "W" : NumberLong(2)
                        }
                },
                "Mutex" : {
                        "acquireCount" : {
                                "r" : NumberLong(40564)
                        }
                },
                "oplog" : {
                        "acquireCount" : {
                                "r" : NumberLong(12955)
                        }
                }
        },
        "logicalSessionRecordCache" : {
                "activeSessionsCount" : 1,
                "sessionsCollectionJobCount" : 44,
                "lastSessionsCollectionJobDurationMillis" : 1,
                "lastSessionsCollectionJobTimestamp" : ISODate("2020-02-18T11:51:43.174Z"),
                "lastSessionsCollectionJobEntriesRefreshed" : 3,
                "lastSessionsCollectionJobEntriesEnded" : 0,
                "lastSessionsCollectionJobCursorsClosed" : 0,
                "transactionReaperJobCount" : 44,
                "lastTransactionReaperJobDurationMillis" : 1,
                "lastTransactionReaperJobTimestamp" : ISODate("2020-02-18T11:51:43.171Z"),
                "lastTransactionReaperJobEntriesCleanedUp" : 0,
                "sessionCatalogSize" : 0
        },
        "network" : {
                "bytesIn" : NumberLong(10825260),
                "bytesOut" : NumberLong(593333422),
                "physicalBytesIn" : NumberLong(10825260),
                "physicalBytesOut" : NumberLong(593333422),
                "numRequests" : NumberLong(1229),
                "compression" : {
                        "snappy" : {
                                "compressor" : {
                                        "bytesIn" : NumberLong(0),
                                        "bytesOut" : NumberLong(0)
                                },
                                "decompressor" : {
                                        "bytesIn" : NumberLong(0),
                                        "bytesOut" : NumberLong(0)
                                }
                        },
                        "zstd" : {
                                "compressor" : {
                                        "bytesIn" : NumberLong(0),
                                        "bytesOut" : NumberLong(0)
                                },
                                "decompressor" : {
                                        "bytesIn" : NumberLong(0),
                                        "bytesOut" : NumberLong(0)
                                }
                        },
                        "zlib" : {
                                "compressor" : {
                                        "bytesIn" : NumberLong(0),
                                        "bytesOut" : NumberLong(0)
                                },
                                "decompressor" : {
                                        "bytesIn" : NumberLong(0),
                                        "bytesOut" : NumberLong(0)
                                }
                        }
                },
                "serviceExecutorTaskStats" : {
                        "executor" : "passthrough",
                        "threadsRunning" : 2
                }
        },
        "opLatencies" : {
                "reads" : {
                        "latency" : NumberLong(9136201),
                        "ops" : NumberLong(95)
                },
                "writes" : {
                        "latency" : NumberLong(316786),
                        "ops" : NumberLong(31)
                },
                "commands" : {
                        "latency" : NumberLong(357529),
                        "ops" : NumberLong(1102)
                },
                "transactions" : {
                        "latency" : NumberLong(0),
                        "ops" : NumberLong(0)
                }
        },
        "opReadConcernCounters" : {
                "available" : NumberLong(0),
                "linearizable" : NumberLong(0),
                "local" : NumberLong(0),
                "majority" : NumberLong(0),
                "snapshot" : NumberLong(0),
                "none" : NumberLong(64)
        },
        "opcounters" : {
                "insert" : NumberLong(25369),
                "query" : NumberLong(64),
                "update" : NumberLong(53),
                "delete" : NumberLong(19),
                "getmore" : NumberLong(48),
                "command" : NumberLong(1219)
        },
        "opcountersRepl" : {
                "insert" : NumberLong(0),
                "query" : NumberLong(0),
                "update" : NumberLong(0),
                "delete" : NumberLong(0),
                "getmore" : NumberLong(0),
                "command" : NumberLong(0)
        },
        "storageEngine" : {
                "name" : "wiredTiger",
                "supportsCommittedReads" : true,
                "oldestRequiredTimestampForCrashRecovery" : Timestamp(0, 0),
                "supportsPendingDrops" : true,
                "dropPendingIdents" : NumberLong(0),
                "supportsSnapshotReadConcern" : true,
                "readOnly" : false,
                "persistent" : true,
                "backupCursorOpen" : false
        },
        "tcmalloc" : {
                "generic" : {
                        "current_allocated_bytes" : 254102168,
                        "heap_size" : 266993664
                },
                "tcmalloc" : {
                        "pageheap_free_bytes" : 8232960,
                        "pageheap_unmapped_bytes" : 0,
                        "max_total_thread_cache_bytes" : NumberLong(1073741824),
                        "current_total_thread_cache_bytes" : 353656,
                        "total_free_bytes" : 4658536,
                        "central_cache_free_bytes" : 1264752,
                        "transfer_cache_free_bytes" : 3040128,
                        "thread_cache_free_bytes" : 353656,
                        "aggressive_memory_decommit" : 0,
                        "pageheap_committed_bytes" : 266993664,
                        "pageheap_scavenge_count" : 0,
                        "pageheap_commit_count" : 217,
                        "pageheap_total_commit_bytes" : 266993664,
                        "pageheap_decommit_count" : 0,
                        "pageheap_total_decommit_bytes" : 0,
                        "pageheap_reserve_count" : 217,
                        "pageheap_total_reserve_bytes" : 266993664,
                        "spinlock_total_delay_ns" : 2700,
                        "formattedString" : "------------------------------------------------\nMALLOC:      254102168 (  242.3 MiB) Bytes in use by application\nMALLOC: +      8232960 (    7.9 MiB) Bytes in page heap freelist\nMALLOC: +      1264752 (    1.2 MiB) Bytes in central cache freelist\nMALLOC: +      3040128 (    2.9 MiB) Bytes in transfer cache freelist\nMALLOC: +       353656 (    0.3 MiB) Bytes in thread cache freelists\nMALLOC: +      5111808 (    4.9 MiB) Bytes in malloc metadata\nMALLOC:   ------------\nMALLOC: =    272105472 (  259.5 MiB) Actual memory used (physical + swap)\nMALLOC: +            0 (    0.0 MiB) Bytes released to OS (aka unmapped)\nMALLOC:   ------------\nMALLOC: =    272105472 (  259.5 MiB) Virtual address space used\nMALLOC:\nMALLOC:           5234              Spans in use\nMALLOC:             13              Thread heaps in use\nMALLOC:           4096              Tcmalloc page size\n------------------------------------------------\nCall ReleaseFreeMemory() to release freelist memory to the OS (via madvise()).\nBytes released to the OS take up virtual address space but no physical memory.\n"
                }
        },
        "trafficRecording" : {
                "running" : false
        },
        "transactions" : {
                "retriedCommandsCount" : NumberLong(0),
                "retriedStatementsCount" : NumberLong(0),
                "transactionsCollectionWriteCount" : NumberLong(0),
                "currentActive" : NumberLong(0),
                "currentInactive" : NumberLong(0),
                "currentOpen" : NumberLong(0),
                "totalAborted" : NumberLong(0),
                "totalCommitted" : NumberLong(0),
                "totalStarted" : NumberLong(0),
                "totalPrepared" : NumberLong(0),
                "totalPreparedThenCommitted" : NumberLong(0),
                "totalPreparedThenAborted" : NumberLong(0),
                "currentPrepared" : NumberLong(0)
        },
        "transportSecurity" : {
                "1.0" : NumberLong(0),
                "1.1" : NumberLong(0),
                "1.2" : NumberLong(0),
                "1.3" : NumberLong(0),
                "unknown" : NumberLong(0)
        },
        "twoPhaseCommitCoordinator" : {
                "totalCreated" : NumberLong(0),
                "totalStartedTwoPhaseCommit" : NumberLong(0),
                "totalAbortedTwoPhaseCommit" : NumberLong(0),
                "totalCommittedTwoPhaseCommit" : NumberLong(0),
                "currentInSteps" : {
                        "writingParticipantList" : NumberLong(0),
                        "waitingForVotes" : NumberLong(0),
                        "writingDecision" : NumberLong(0),
                        "waitingForDecisionAcks" : NumberLong(0),
                        "deletingCoordinatorDoc" : NumberLong(0)
                }
        },
        "wiredTiger" : {
                "uri" : "statistics:",
                "async" : {
                        "current work queue length" : 0,
                        "maximum work queue length" : 0,
                        "number of allocation state races" : 0,
                        "number of flush calls" : 0,
                        "number of operation slots viewed for allocation" : 0,
                        "number of times operation allocation failed" : 0,
                        "number of times worker found no work" : 0,
                        "total allocations" : 0,
                        "total compact calls" : 0,
                        "total insert calls" : 0,
                        "total remove calls" : 0,
                        "total search calls" : 0,
                        "total update calls" : 0
                },
                "block-manager" : {
                        "blocks pre-loaded" : 620,
                        "blocks read" : 2033,
                        "blocks written" : 786,
                        "bytes read" : 50147328,
                        "bytes written" : 9809920,
                        "bytes written for checkpoint" : 9809920,
                        "mapped blocks read" : 0,
                        "mapped bytes read" : 0
                },
                "cache" : {
                        "application threads page read from disk to cache count" : 1887,
                        "application threads page read from disk to cache time (usecs)" : 318657,
                        "application threads page write from cache to disk count" : 523,
                        "application threads page write from cache to disk time (usecs)" : 41663,
                        "bytes belonging to page images in the cache" : 160576675,
                        "bytes belonging to the cache overflow table in the cache" : 182,
                        "bytes currently in the cache" : 183524517,
                        "bytes dirty in the cache cumulative" : 2711108,
                        "bytes not belonging to page images in the cache" : 22947842,
                        "bytes read into cache" : 148682067,
                        "bytes written from cache" : 14686301,
                        "cache overflow cursor application thread wait time (usecs)" : 0,
                        "cache overflow cursor internal thread wait time (usecs)" : 0,
                        "cache overflow score" : 0,
                        "cache overflow table entries" : 0,
                        "cache overflow table insert calls" : 0,
                        "cache overflow table max on-disk size" : 0,
                        "cache overflow table on-disk size" : 0,
                        "cache overflow table remove calls" : 0,
                        "checkpoint blocked page eviction" : 0,
                        "eviction calls to get a page" : 281,
                        "eviction calls to get a page found queue empty" : 270,
                        "eviction calls to get a page found queue empty after locking" : 0,
                        "eviction currently operating in aggressive mode" : 0,
                        "eviction empty score" : 0,
                        "eviction passes of a file" : 0,
                        "eviction server candidate queue empty when topping up" : 0,
                        "eviction server candidate queue not empty when topping up" : 0,
                        "eviction server evicting pages" : 0,
                        "eviction server slept, because we did not make progress with eviction" : 454,
                        "eviction server unable to reach eviction goal" : 0,
                        "eviction server waiting for a leaf page" : 3,
                        "eviction state" : 128,
                        "eviction walk target pages histogram - 0-9" : 0,
                        "eviction walk target pages histogram - 10-31" : 0,
                        "eviction walk target pages histogram - 128 and higher" : 0,
                        "eviction walk target pages histogram - 32-63" : 0,
                        "eviction walk target pages histogram - 64-128" : 0,
                        "eviction walk target strategy both clean and dirty pages" : 0,
                        "eviction walk target strategy only clean pages" : 0,
                        "eviction walk target strategy only dirty pages" : 0,
                        "eviction walks abandoned" : 0,
                        "eviction walks gave up because they restarted their walk twice" : 0,
                        "eviction walks gave up because they saw too many pages and found no candidates" : 0,
                        "eviction walks gave up because they saw too many pages and found too few candidates" : 0,
                        "eviction walks reached end of tree" : 0,
                        "eviction walks started from root of tree" : 0,
                        "eviction walks started from saved location in tree" : 0,
                        "eviction worker thread active" : 4,
                        "eviction worker thread created" : 0,
                        "eviction worker thread evicting pages" : 3,
                        "eviction worker thread removed" : 0,
                        "eviction worker thread stable number" : 0,
                        "files with active eviction walks" : 0,
                        "files with new eviction walks started" : 0,
                        "force re-tuning of eviction workers once in a while" : 0,
                        "forced eviction - pages evicted that were clean count" : 0,
                        "forced eviction - pages evicted that were clean time (usecs)" : 0,
                        "forced eviction - pages evicted that were dirty count" : 3,
                        "forced eviction - pages evicted that were dirty time (usecs)" : 2989,
                        "forced eviction - pages selected because of too many deleted items count" : 4,
                        "forced eviction - pages selected count" : 4,
                        "forced eviction - pages selected unable to be evicted count" : 0,
                        "forced eviction - pages selected unable to be evicted time" : 0,
                        "hazard pointer blocked page eviction" : 0,
                        "hazard pointer check calls" : 7,
                        "hazard pointer check entries walked" : 0,
                        "hazard pointer maximum array length" : 0,
                        "in-memory page passed criteria to be split" : 2,
                        "in-memory page splits" : 1,
                        "internal pages evicted" : 0,
                        "internal pages queued for eviction" : 0,
                        "internal pages seen by eviction walk" : 0,
                        "internal pages seen by eviction walk that are already queued" : 0,
                        "internal pages split during eviction" : 0,
                        "leaf pages split during eviction" : 2,
                        "maximum bytes configured" : 16512974848,
                        "maximum page size at eviction" : 0,
                        "modified pages evicted" : 6,
                        "modified pages evicted by application threads" : 0,
                        "operations timed out waiting for space in cache" : 0,
                        "overflow pages read into cache" : 0,
                        "page split during eviction deepened the tree" : 0,
                        "page written requiring cache overflow records" : 0,
                        "pages currently held in the cache" : 1919,
                        "pages evicted by application threads" : 0,
                        "pages queued for eviction" : 0,
                        "pages queued for eviction post lru sorting" : 0,
                        "pages queued for urgent eviction" : 4,
                        "pages queued for urgent eviction during walk" : 0,
                        "pages read into cache" : 1901,
                        "pages read into cache after truncate" : 12,
                        "pages read into cache after truncate in prepare state" : 0,
                        "pages read into cache requiring cache overflow entries" : 0,
                        "pages read into cache requiring cache overflow for checkpoint" : 0,
                        "pages read into cache skipping older cache overflow entries" : 0,
                        "pages read into cache with skipped cache overflow entries needed later" : 0,
                        "pages read into cache with skipped cache overflow entries needed later by checkpoint" : 0,
                        "pages requested from the cache" : 230805,
                        "pages seen by eviction walk" : 0,
                        "pages seen by eviction walk that are already queued" : 0,
                        "pages selected for eviction unable to be evicted" : 0,
                        "pages selected for eviction unable to be evicted as the parent page has overflow items" : 0,
                        "pages selected for eviction unable to be evicted because of active children on an internal page" : 0,
                        "pages selected for eviction unable to be evicted because of failure in reconciliation" : 0,
                        "pages selected for eviction unable to be evicted due to newer modifications on a clean page" : 0,
                        "pages walked for eviction" : 0,
                        "pages written from cache" : 524,
                        "pages written requiring in-memory restoration" : 1,
                        "percentage overhead" : 8,
                        "tracked bytes belonging to internal pages in the cache" : 161705,
                        "tracked bytes belonging to leaf pages in the cache" : 183362812,
                        "tracked dirty bytes in the cache" : 112786,
                        "tracked dirty pages in the cache" : 4,
                        "unmodified pages evicted" : 0
                },
                "capacity" : {
                        "background fsync file handles considered" : 0,
                        "background fsync file handles synced" : 0,
                        "background fsync time (msecs)" : 0,
                        "bytes read" : 49606656,
                        "bytes written for checkpoint" : 7294837,
                        "bytes written for eviction" : 0,
                        "bytes written for log" : 1261937280,
                        "bytes written total" : 1269232117,
                        "threshold to call fsync" : 0,
                        "time waiting due to total capacity (usecs)" : 0,
                        "time waiting during checkpoint (usecs)" : 0,
                        "time waiting during eviction (usecs)" : 0,
                        "time waiting during logging (usecs)" : 0,
                        "time waiting during read (usecs)" : 0
                },
                "connection" : {
                        "auto adjusting condition resets" : 441,
                        "auto adjusting condition wait calls" : 78424,
                        "detected system time went backwards" : 0,
                        "files currently open" : 22,
                        "memory allocations" : 626516,
                        "memory frees" : 474932,
                        "memory re-allocations" : 53838,
                        "pthread mutex condition wait calls" : 214255,
                        "pthread mutex shared lock read-lock calls" : 194618,
                        "pthread mutex shared lock write-lock calls" : 13554,
                        "total fsync I/Os" : 433,
                        "total read I/Os" : 3354,
                        "total write I/Os" : 1046
                },
                "cursor" : {
                        "cached cursor count" : 9,
                        "cursor bulk loaded cursor insert calls" : 0,
                        "cursor close calls that result in cache" : 1255,
                        "cursor create calls" : 397,
                        "cursor insert calls" : 50988,
                        "cursor insert key and value bytes" : 11229086,
                        "cursor modify calls" : 0,
                        "cursor modify key and value bytes affected" : 0,
                        "cursor modify value bytes modified" : 0,
                        "cursor next calls" : 6859540,
                        "cursor operation restarted" : 0,
                        "cursor prev calls" : 52395,
                        "cursor remove calls" : 83,
                        "cursor remove key bytes removed" : 2038,
                        "cursor reserve calls" : 0,
                        "cursor reset calls" : 131894,
                        "cursor search calls" : 6058092,
                        "cursor search near calls" : 52925,
                        "cursor sweep buckets" : 2604,
                        "cursor sweep cursors closed" : 0,
                        "cursor sweep cursors examined" : 48,
                        "cursor sweeps" : 434,
                        "cursor truncate calls" : 0,
                        "cursor update calls" : 0,
                        "cursor update key and value bytes" : 0,
                        "cursor update value size change" : 0,
                        "cursors reused from cache" : 1194,
                        "open cursor count" : 21
                },
                "data-handle" : {
                        "connection data handle size" : 432,
                        "connection data handles currently active" : 36,
                        "connection sweep candidate became referenced" : 0,
                        "connection sweep dhandles closed" : 0,
                        "connection sweep dhandles removed from hash list" : 77,
                        "connection sweep time-of-death sets" : 1089,
                        "connection sweeps" : 1292,
                        "session dhandles swept" : 0,
                        "session sweep attempts" : 173
                },
                "lock" : {
                        "checkpoint lock acquisitions" : 216,
                        "checkpoint lock application thread wait time (usecs)" : 0,
                        "checkpoint lock internal thread wait time (usecs)" : 0,
                        "dhandle lock application thread time waiting (usecs)" : 0,
                        "dhandle lock internal thread time waiting (usecs)" : 0,
                        "dhandle read lock acquisitions" : 52271,
                        "dhandle write lock acquisitions" : 190,
                        "durable timestamp queue lock application thread time waiting (usecs)" : 0,
                        "durable timestamp queue lock internal thread time waiting (usecs)" : 0,
                        "durable timestamp queue read lock acquisitions" : 0,
                        "durable timestamp queue write lock acquisitions" : 0,
                        "metadata lock acquisitions" : 58,
                        "metadata lock application thread wait time (usecs)" : 0,
                        "metadata lock internal thread wait time (usecs)" : 0,
                        "read timestamp queue lock application thread time waiting (usecs)" : 0,
                        "read timestamp queue lock internal thread time waiting (usecs)" : 0,
                        "read timestamp queue read lock acquisitions" : 0,
                        "read timestamp queue write lock acquisitions" : 0,
                        "schema lock acquisitions" : 92,
                        "schema lock application thread wait time (usecs)" : 0,
                        "schema lock internal thread wait time (usecs)" : 0,
                        "table lock application thread time waiting for the table lock (usecs)" : 0,
                        "table lock internal thread time waiting for the table lock (usecs)" : 0,
                        "table read lock acquisitions" : 0,
                        "table write lock acquisitions" : 27,
                        "txn global lock application thread time waiting (usecs)" : 0,
                        "txn global lock internal thread time waiting (usecs)" : 0,
                        "txn global read lock acquisitions" : 361,
                        "txn global write lock acquisitions" : 198
                },
                "log" : {
                        "busy returns attempting to switch slots" : 0,
                        "force archive time sleeping (usecs)" : 0,
                        "log bytes of payload data" : 3983805,
                        "log bytes written" : 4031232,
                        "log files manually zero-filled" : 0,
                        "log flush operations" : 128516,
                        "log force write operations" : 141736,
                        "log force write operations skipped" : 141631,
                        "log records compressed" : 454,
                        "log records not compressed" : 47,
                        "log records too small to compress" : 181,
                        "log release advances write LSN" : 64,
                        "log scan operations" : 6,
                        "log scan records requiring two reads" : 0,
                        "log server thread advances write LSN" : 129,
                        "log server thread write LSN walk skipped" : 14782,
                        "log sync operations" : 164,
                        "log sync time duration (usecs)" : 4261893,
                        "log sync_dir operations" : 1,
                        "log sync_dir time duration (usecs)" : 0,
                        "log write operations" : 682,
                        "logging bytes consolidated" : 4030720,
                        "maximum log file size" : 104857600,
                        "number of pre-allocated log files to create" : 2,
                        "pre-allocated log files not ready and missed" : 1,
                        "pre-allocated log files prepared" : 2,
                        "pre-allocated log files used" : 0,
                        "records processed by log scan" : 13,
                        "slot close lost race" : 0,
                        "slot close unbuffered waits" : 0,
                        "slot closures" : 193,
                        "slot join atomic update races" : 0,
                        "slot join calls atomic updates raced" : 0,
                        "slot join calls did not yield" : 682,
                        "slot join calls found active slot closed" : 0,
                        "slot join calls slept" : 0,
                        "slot join calls yielded" : 0,
                        "slot join found active slot closed" : 0,
                        "slot joins yield time (usecs)" : 0,
                        "slot transitions unable to find free slot" : 0,
                        "slot unbuffered writes" : 0,
                        "total in-memory size of compressed records" : 11507261,
                        "total log buffer size" : 33554432,
                        "total size of compressed records" : 3966568,
                        "written slots coalesced" : 0,
                        "yields waiting for previous log file close" : 0
                },
                "perf" : {
                        "file system read latency histogram (bucket 1) - 10-49ms" : 12,
                        "file system read latency histogram (bucket 2) - 50-99ms" : 1,
                        "file system read latency histogram (bucket 3) - 100-249ms" : 0,
                        "file system read latency histogram (bucket 4) - 250-499ms" : 0,
                        "file system read latency histogram (bucket 5) - 500-999ms" : 0,
                        "file system read latency histogram (bucket 6) - 1000ms+" : 0,
                        "file system write latency histogram (bucket 1) - 10-49ms" : 10,
                        "file system write latency histogram (bucket 2) - 50-99ms" : 0,
                        "file system write latency histogram (bucket 3) - 100-249ms" : 0,
                        "file system write latency histogram (bucket 4) - 250-499ms" : 0,
                        "file system write latency histogram (bucket 5) - 500-999ms" : 0,
                        "file system write latency histogram (bucket 6) - 1000ms+" : 0,
                        "operation read latency histogram (bucket 1) - 100-249us" : 0,
                        "operation read latency histogram (bucket 2) - 250-499us" : 0,
                        "operation read latency histogram (bucket 3) - 500-999us" : 1365,
                        "operation read latency histogram (bucket 4) - 1000-9999us" : 250,
                        "operation read latency histogram (bucket 5) - 10000us+" : 6,
                        "operation write latency histogram (bucket 1) - 100-249us" : 0,
                        "operation write latency histogram (bucket 2) - 250-499us" : 0,
                        "operation write latency histogram (bucket 3) - 500-999us" : 56,
                        "operation write latency histogram (bucket 4) - 1000-9999us" : 3,
                        "operation write latency histogram (bucket 5) - 10000us+" : 0
                },
                "reconciliation" : {
                        "fast-path pages deleted" : 0,
                        "page reconciliation calls" : 352,
                        "page reconciliation calls for eviction" : 4,
                        "pages deleted" : 24,
                        "split bytes currently awaiting free" : 0,
                        "split objects currently awaiting free" : 0
                },
                "session" : {
                        "open session count" : 18,
                        "session query timestamp calls" : 0,
                        "table alter failed calls" : 0,
                        "table alter successful calls" : 0,
                        "table alter unchanged and skipped" : 0,
                        "table compact failed calls" : 0,
                        "table compact successful calls" : 0,
                        "table create failed calls" : 0,
                        "table create successful calls" : 5,
                        "table drop failed calls" : 0,
                        "table drop successful calls" : 0,
                        "table import failed calls" : 0,
                        "table import successful calls" : 0,
                        "table rebalance failed calls" : 0,
                        "table rebalance successful calls" : 0,
                        "table rename failed calls" : 0,
                        "table rename successful calls" : 0,
                        "table salvage failed calls" : 0,
                        "table salvage successful calls" : 0,
                        "table truncate failed calls" : 0,
                        "table truncate successful calls" : 0,
                        "table verify failed calls" : 0,
                        "table verify successful calls" : 0
                },
                "thread-state" : {
                        "active filesystem fsync calls" : 0,
                        "active filesystem read calls" : 0,
                        "active filesystem write calls" : 0
                },
                "thread-yield" : {
                        "application thread time evicting (usecs)" : 0,
                        "application thread time waiting for cache (usecs)" : 0,
                        "connection close blocked waiting for transaction state stabilization" : 0,
                        "connection close yielded for lsm manager shutdown" : 0,
                        "data handle lock yielded" : 0,
                        "get reference for page index and slot time sleeping (usecs)" : 0,
                        "log server sync yielded for log write" : 0,
                        "page access yielded due to prepare state change" : 0,
                        "page acquire busy blocked" : 0,
                        "page acquire eviction blocked" : 0,
                        "page acquire locked blocked" : 0,
                        "page acquire read blocked" : 0,
                        "page acquire time sleeping (usecs)" : 0,
                        "page delete rollback time sleeping for state change (usecs)" : 0,
                        "page reconciliation yielded due to child modification" : 0
                },
                "transaction" : {
                        "Number of prepared updates" : 0,
                        "Number of prepared updates added to cache overflow" : 0,
                        "durable timestamp queue entries walked" : 0,
                        "durable timestamp queue insert to empty" : 0,
                        "durable timestamp queue inserts to head" : 0,
                        "durable timestamp queue inserts total" : 0,
                        "durable timestamp queue length" : 0,
                        "number of named snapshots created" : 0,
                        "number of named snapshots dropped" : 0,
                        "prepared transactions" : 0,
                        "prepared transactions committed" : 0,
                        "prepared transactions currently active" : 0,
                        "prepared transactions rolled back" : 0,
                        "query timestamp calls" : 12925,
                        "read timestamp queue entries walked" : 0,
                        "read timestamp queue insert to empty" : 0,
                        "read timestamp queue inserts to head" : 0,
                        "read timestamp queue inserts total" : 0,
                        "read timestamp queue length" : 0,
                        "rollback to stable calls" : 0,
                        "rollback to stable updates aborted" : 0,
                        "rollback to stable updates removed from cache overflow" : 0,
                        "set timestamp calls" : 0,
                        "set timestamp durable calls" : 0,
                        "set timestamp durable updates" : 0,
                        "set timestamp oldest calls" : 0,
                        "set timestamp oldest updates" : 0,
                        "set timestamp stable calls" : 0,
                        "set timestamp stable updates" : 0,
                        "transaction begins" : 52498,
                        "transaction checkpoint currently running" : 0,
                        "transaction checkpoint generation" : 59,
                        "transaction checkpoint max time (msecs)" : 353,
                        "transaction checkpoint min time (msecs)" : 60,
                        "transaction checkpoint most recent time (msecs)" : 159,
                        "transaction checkpoint scrub dirty target" : 0,
                        "transaction checkpoint scrub time (msecs)" : 0,
                        "transaction checkpoint total time (msecs)" : 7140,
                        "transaction checkpoints" : 216,
                        "transaction checkpoints skipped because database was clean" : 158,
                        "transaction failures due to cache overflow" : 0,
                        "transaction fsync calls for checkpoint after allocating the transaction ID" : 58,
                        "transaction fsync duration for checkpoint after allocating the transaction ID (usecs)" : 104583,
                        "transaction range of IDs currently pinned" : 0,
                        "transaction range of IDs currently pinned by a checkpoint" : 0,
                        "transaction range of IDs currently pinned by named snapshots" : 0,
                        "transaction range of timestamps currently pinned" : 0,
                        "transaction range of timestamps pinned by a checkpoint" : 0,
                        "transaction range of timestamps pinned by the oldest active read timestamp" : 0,
                        "transaction range of timestamps pinned by the oldest timestamp" : 0,
                        "transaction read timestamp of the oldest active reader" : 0,
                        "transaction sync calls" : 0,
                        "transactions committed" : 507,
                        "transactions rolled back" : 51993,
                        "update conflicts" : 0
                },
                "concurrentTransactions" : {
                        "write" : {
                                "out" : 0,
                                "available" : 128,
                                "totalTickets" : 128
                        },
                        "read" : {
                                "out" : 1,
                                "available" : 127,
                                "totalTickets" : 128
                        }
                },
                "snapshot-window-settings" : {
                        "cache pressure percentage threshold" : 95,
                        "current cache pressure percentage" : NumberLong(0),
                        "total number of SnapshotTooOld errors" : NumberLong(0),
                        "max target available snapshots window size in seconds" : 5,
                        "target available snapshots window size in seconds" : 5,
                        "current available snapshots window size in seconds" : 0,
                        "latest majority snapshot timestamp available" : "Jan  1 01:00:00:0",
                        "oldest majority snapshot timestamp available" : "Jan  1 01:00:00:0"
                }
        },
        "mem" : {
                "bits" : 64,
                "resident" : 346,
                "virtual" : 5674,
                "supported" : true
        },
        "metrics" : {
                "commands" : {
                        "aggregate" : {
                                "failed" : NumberLong(0),
                                "total" : NumberLong(20)
                        },
                        "buildInfo" : {
                                "failed" : NumberLong(0),
                                "total" : NumberLong(29)
                        },
                        "collStats" : {
                                "failed" : NumberLong(0),
                                "total" : NumberLong(4)
                        },
                        "connectionStatus" : {
                                "failed" : NumberLong(0),
                                "total" : NumberLong(1)
                        },
                        "count" : {
                                "failed" : NumberLong(0),
                                "total" : NumberLong(8)
                        },
                        "create" : {
                                "failed" : NumberLong(0),
                                "total" : NumberLong(1)
                        },
                        "dbStats" : {
                                "failed" : NumberLong(0),
                                "total" : NumberLong(19)
                        },
                        "delete" : {
                                "failed" : NumberLong(0),
                                "total" : NumberLong(12)
                        },
                        "endSessions" : {
                                "failed" : NumberLong(0),
                                "total" : NumberLong(19)
                        },
                        "find" : {
                                "failed" : NumberLong(0),
                                "total" : NumberLong(64)
                        },
                        "getCmdLineOpts" : {
                                "failed" : NumberLong(0),
                                "total" : NumberLong(9)
                        },
                        "getFreeMonitoringStatus" : {
                                "failed" : NumberLong(0),
                                "total" : NumberLong(8)
                        },
                        "getLog" : {
                                "failed" : NumberLong(0),
                                "total" : NumberLong(8)
                        },
                        "getMore" : {
                                "failed" : NumberLong(0),
                                "total" : NumberLong(48)
                        },
                        "hostInfo" : {
                                "failed" : NumberLong(0),
                                "total" : NumberLong(1)
                        },
                        "insert" : {
                                "failed" : NumberLong(0),
                                "total" : NumberLong(28)
                        },
                        "isMaster" : {
                                "failed" : NumberLong(0),
                                "total" : NumberLong(825)
                        },
                        "listCollections" : {
                                "failed" : NumberLong(0),
                                "total" : NumberLong(31)
                        },
                        "listDatabases" : {
                                "failed" : NumberLong(0),
                                "total" : NumberLong(3)
                        },
                        "listIndexes" : {
                                "failed" : NumberLong(0),
                                "total" : NumberLong(90)
                        },
                        "ping" : {
                                "failed" : NumberLong(0),
                                "total" : NumberLong(15)
                        },
                        "replSetGetStatus" : {
                                "failed" : NumberLong(8),
                                "total" : NumberLong(8)
                        },
                        "serverStatus" : {
                                "failed" : NumberLong(0),
                                "total" : NumberLong(89)
                        },
                        "top" : {
                                "failed" : NumberLong(0),
                                "total" : NumberLong(23)
                        },
                        "update" : {
                                "failed" : NumberLong(0),
                                "total" : NumberLong(14)
                        },
                        "whatsmyuri" : {
                                "failed" : NumberLong(0),
                                "total" : NumberLong(8)
                        }
                },
                "cursor" : {
                        "timedOut" : NumberLong(0),
                        "open" : {
                                "noTimeout" : NumberLong(0),
                                "pinned" : NumberLong(0),
                                "total" : NumberLong(0)
                        }
                },
                "document" : {
                        "deleted" : NumberLong(4),
                        "inserted" : NumberLong(25368),
                        "returned" : NumberLong(6012304),
                        "updated" : NumberLong(23)
                },
                "getLastError" : {
                        "wtime" : {
                                "num" : 0,
                                "totalMillis" : 0
                        },
                        "wtimeouts" : NumberLong(0)
                },
                "operation" : {
                        "scanAndOrder" : NumberLong(0),
                        "writeConflicts" : NumberLong(0)
                },
                "query" : {
                        "planCacheTotalSizeEstimateBytes" : NumberLong(0),
                        "updateOneOpStyleBroadcastWithExactIDCount" : NumberLong(0)
                },
                "queryExecutor" : {
                        "scanned" : NumberLong(6054671),
                        "scannedObjects" : NumberLong(6433605)
                },
                "record" : {
                        "moves" : NumberLong(0)
                },
                "repl" : {
                        "executor" : {
                                "pool" : {
                                        "inProgressCount" : 0
                                },
                                "queues" : {
                                        "networkInProgress" : 0,
                                        "sleepers" : 0
                                },
                                "unsignaledEvents" : 0,
                                "shuttingDown" : false,
                                "networkInterface" : "DEPRECATED: getDiagnosticString is deprecated in NetworkInterfaceTL"
                        },
                        "apply" : {
                                "attemptsToBecomeSecondary" : NumberLong(0),
                                "batchSize" : NumberLong(0),
                                "batches" : {
                                        "num" : 0,
                                        "totalMillis" : 0
                                },
                                "ops" : NumberLong(0)
                        },
                        "buffer" : {
                                "count" : NumberLong(0),
                                "maxSizeBytes" : NumberLong(0),
                                "sizeBytes" : NumberLong(0)
                        },
                        "initialSync" : {
                                "completed" : NumberLong(0),
                                "failedAttempts" : NumberLong(0),
                                "failures" : NumberLong(0)
                        },
                        "network" : {
                                "bytes" : NumberLong(0),
                                "getmores" : {
                                        "num" : 0,
                                        "totalMillis" : 0
                                },
                                "notMasterLegacyUnacknowledgedWrites" : NumberLong(0),
                                "notMasterUnacknowledgedWrites" : NumberLong(0),
                                "ops" : NumberLong(0),
                                "readersCreated" : NumberLong(0)
                        },
                        "stepDown" : {
                                "userOperationsKilled" : NumberLong(0),
                                "userOperationsRunning" : NumberLong(0)
                        }
                },
                "ttl" : {
                        "deletedDocuments" : NumberLong(20),
                        "passes" : NumberLong(215)
                }
        },
        "ok" : 1
}
>

```


```sh
```


```sh
```

