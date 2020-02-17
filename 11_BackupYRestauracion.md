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

Realiza backup del servidor completo, base de datos, colecciones


Creamos las carpetas data2\server y data2\backup


Levantar nuestro nuevo servidor
````sh
C:\Users\manana>mongod --port 27300 --dbpath data2\server
```

Levanto la shell para ese servidor:

````sh
C:\Users\manana>mongo --port 27300
```

Creamos BD y le metemos 1000 registros.

````sh
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


````sh
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

````sh
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


````sh


```


   
