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

Entro y borro 5 documentos

````sh
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

