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

```


```sh

```

