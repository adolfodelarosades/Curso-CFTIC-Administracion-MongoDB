# 3. Operaciones CRUD

[MongoDB CRUD Operations](https://docs.mongodb.com/manual/crud/index.html)

## 3.1 Operaciones de Inserción

[Insert Documents](https://docs.mongodb.com/manual/tutorial/insert-documents/)

### Método insert()

[Insert](https://docs.mongodb.com/manual/reference/command/insert/index.html)
[db.collection.insert()](https://docs.mongodb.com/manual/reference/method/db.collection.insert/index.html)

Sintaxis.

```sh
db.collection.insert(
   <document or array of documents>,
   {
     writeConcern: <document>,
     ordered: <boolean>
   }
)
```

`writeConcern` y `ordered` Son opcionales


* Crea la colección sino existe.
* Crea `_id` si no se especifica para cada documento.
* Si se especifica `_id` debe ser único.

Insertar documento individual sin `_id`:

```sh
> use gimnasio
switched to db gimnasio

> show collections

> db.clientes.insert({ nombre: "Juan", apellido: "Pérez" })
WriteResult({ "nInserted" : 1 })


> db.clientes.find()
{ "_id" : ObjectId("5e419e9490d86b85f5fda8ef"), "nombre" : "Juan", "apellido" : "Pérez" }
```

Insertar documento individual con `_id`:

```sh
> db.clientes.insert({ _id: 1, nombre: "María", apellido: "López" })
WriteResult({ "nInserted" : 1 })
> db.clientes.insert({ _id: 1, nombre: "Rosa", apellido: "López" })
WriteResult({
	"nInserted" : 0,
	"writeError" : {
		"code" : 11000,
		"errmsg" : "E11000 duplicate key error collection: gimnasio.clientes index: _id_ dup key: { _id: 1.0 }"
	}
})
> 

```
Como se ve en el ejemplo si intento meter otro documento con el mismo `_id` me indica que no lo puedo hacer.


Insertar un array de documentos:

```sh
> db.clientes.insert([
... {_id: 2, nombre: "Laura", apellido: "López"},
... {_id: 3, nombre: "Luis", apellido: "Pérez"}
... ])
BulkWriteResult({
	"writeErrors" : [ ],
	"writeConcernErrors" : [ ],
	"nInserted" : 2,
	"nUpserted" : 0,
	"nMatched" : 0,
	"nModified" : 0,
	"nRemoved" : 0,
	"upserted" : [ ]
})
> db.clientes.find()
{ "_id" : ObjectId("5e419e9490d86b85f5fda8ef"), "nombre" : "Juan", "apellido" : "Pérez" }
{ "_id" : 1, "nombre" : "María", "apellido" : "López" }
{ "_id" : 2, "nombre" : "Laura", "apellido" : "López" }
{ "_id" : 3, "nombre" : "Luis", "apellido" : "Pérez" }
> 
```
**Inserta cada uno de los documentos del array** Si en el array existe un error, insertara los registros hasta antes del error.



```sh

```
```sh

```

```sh

```

```sh

```
