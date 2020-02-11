# Capped Collections

[Capped Collections](https://docs.mongodb.com/manual/core/capped-collections/index.html)
Colecciones limitadas en espacio o número de documento.

Tambien elimina de manera automatica (como un indice que vimos), aqui lo borra como estructura FIFO el primero que entra el primero que sale.

Hay dops criterios por bytes o por número de documentos.

La utilidad de esto es para algo muy especifico no para almacenamiento convencional.

```
db.createCollection(
  "<nombre-coleccion>", 
  {capped: true, size: <bytes>, max: <entero>}
)
```

Ejemplo primero creamos la colección:

```sh
> use shop
switched to db shop
> show collections
productos
> db.createCollection(
... "foo",
... {capped: true, size:100000, max: 5 }
... )
{ "ok" : 1 }
>
```

Metemos datos

```sh
> db.foo.insert({ _id:1, a: 1})
WriteResult({ "nInserted" : 1 })
> db.foo.insert({ _id:2, a: 1})
WriteResult({ "nInserted" : 1 })
> db.foo.insert({ _id:3, a: 1})
WriteResult({ "nInserted" : 1 })
> db.foo.insert({ _id:4, a: 1})
WriteResult({ "nInserted" : 1 })
> db.foo.insert({ _id:5, a: 1})
WriteResult({ "nInserted" : 1 })
> db.foo.insert({ _id:6, a: 1})
WriteResult({ "nInserted" : 1 })
> db.foo.insert({ _id:7, a: 1})
WriteResult({ "nInserted" : 1 })
```

Como lo hemos limitado a un máximo de 5 documentos. Al listar la colección solo mostrara los últimos 5 registros que se metan.

```sh
> db.foo.find()
{ "_id" : 3, "a" : 1 }
{ "_id" : 4, "a" : 1 }
{ "_id" : 5, "a" : 1 }
{ "_id" : 6, "a" : 1 }
{ "_id" : 7, "a" : 1 }
>       
```

Para ver si es una colección capada:
```sh
> db.foo.isCapped()
true

Si tenemos una coleccion y luego le aplicamos la caoacion al max no le hace caso falta ver si por tamaño si.
```sh
> db.foo2.insert({ _id:1, a: 1})
WriteResult({ "nInserted" : 1 })
> db.foo2.insert({ _id:2, a: 1})
WriteResult({ "nInserted" : 1 })
> db.foo2.insert({ _id:3, a: 1})
WriteResult({ "nInserted" : 1 })
> db.foo2.insert({ _id:4, a: 1})
WriteResult({ "nInserted" : 1 })
> db.foo2.insert({ _id:5, a: 1})
WriteResult({ "nInserted" : 1 })
> db.foo2.isCapped()
false

> db.runCommand({convertToCapped: "foo2", size: 10000, max: 3})
{ "ok" : 1 }
> db.foo2.find()
{ "_id" : 1, "a" : 1 }
{ "_id" : 2, "a" : 1 }
{ "_id" : 3, "a" : 1 }
{ "_id" : 4, "a" : 1 }
{ "_id" : 5, "a" : 1 }
> db.foo2.insert({ _id:6, a: 1})
WriteResult({ "nInserted" : 1 })
> db.foo2.insert({ _id:7, a: 1})
WriteResult({ "nInserted" : 1 })
> db.foo2.find()
{ "_id" : 1, "a" : 1 }
{ "_id" : 2, "a" : 1 }
{ "_id" : 3, "a" : 1 }
{ "_id" : 4, "a" : 1 }
{ "_id" : 5, "a" : 1 }
{ "_id" : 6, "a" : 1 }
{ "_id" : 7, "a" : 1 }
>                                                                                                                                       ```                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                       

```
