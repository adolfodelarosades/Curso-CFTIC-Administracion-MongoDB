# 3. Operaciones CRUD

[MongoDB CRUD Operations](https://docs.mongodb.com/manual/crud/index.html)

## 3.1 Operaciones de Inserción

[Create Operations](https://docs.mongodb.com/manual/crud/index.html#create-operations)

[Insert Documents](https://docs.mongodb.com/manual/tutorial/insert-documents/)

### Método insert()

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

#### Insertar documento individual sin `_id`:

```sh
> use gimnasio
switched to db gimnasio

> show collections

> db.clientes.insert({ nombre: "Juan", apellido: "Pérez" })
WriteResult({ "nInserted" : 1 })


> db.clientes.find()
{ "_id" : ObjectId("5e419e9490d86b85f5fda8ef"), "nombre" : "Juan", "apellido" : "Pérez" }
```

#### Insertar documento individual con `_id`:

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


#### Insertar un array de documentos:

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
**Inserta cada uno de los documentos del array**. Si en el array existe un error, insertara los registros hasta antes del error.

#### Operación con `ordered` true (Default)

Si `ordered` es `true` que es el valor por default, realiza una inserción ordenada de los documentos en el arrar, y si ocurre un error con uno de los documentos, MongoDB regresará sin procesar los documentos restantes del array.

```sh
> db.clientes.insert([
... { _id: 4, nombre: "Pablo", apellido: "Gómez" },
... { _id: 5, nombre: "Sara", apellido: "García" },
... { _id: 6, nombre: "Carlos", apellido: "López" },
... { _id: 6, nombre: "Dolores", apellido: "Pérez" },
... { _id: 7, nombre: "Pedro", apellido: "García" }
... ])
BulkWriteResult({
	"writeErrors" : [
		{
			"index" : 3,
			"code" : 11000,
			"errmsg" : "E11000 duplicate key error collection: gimnasio.clientes index: _id_ dup key: { _id: 6.0 }",
			"op" : {
				"_id" : 6,
				"nombre" : "Dolores",
				"apellido" : "Pérez"
			}
		}
	],
	"writeConcernErrors" : [ ],
	"nInserted" : 3,
	"nUpserted" : 0,
	"nMatched" : 0,
	"nModified" : 0,
	"nRemoved" : 0,
	"upserted" : [ ]
})

```
Nos indica que hay un diplicado en la key `_id: 6`, nos indica que ha insertado 3 documentos (`"nInserted" : 3`), es decir los documentos con `_id` 4, 5 y 6 y la ejecución se detuvo al encontrar el segundo `_id` 6, si listamos los documentod podemos comprobarlo:

```sh
> db.clientes.find()
{ "_id" : ObjectId("5e419e9490d86b85f5fda8ef"), "nombre" : "Juan", "apellido" : "Pérez" }
{ "_id" : 1, "nombre" : "María", "apellido" : "López" }
{ "_id" : 2, "nombre" : "Laura", "apellido" : "López" }
{ "_id" : 3, "nombre" : "Luis", "apellido" : "Pérez" }
{ "_id" : 4, "nombre" : "Pablo", "apellido" : "Gómez" }
{ "_id" : 5, "nombre" : "Sara", "apellido" : "García" }
{ "_id" : 6, "nombre" : "Carlos", "apellido" : "López" }
> 
```

#### Operación con `ordered` false

Si `ordered` es `false`, realiza una inserción desordenada, y si ocurre un error con uno de los documentos, continúa procesando los documentos restantes en el array.

Vamos a ver el siguiente ejemplo, Mongo nos permite meter documentos aun que no tenga todos sus campos.

```sh
> db.clientes.insert([
... { _id: 7, nombre: "José" },
... { _id: 8, nombre: "Lucia" },
... { _id: 8, nombre: "Juan" },
... { _id: 9, nombre: "Luisa" },
... { _id: 10, nombre: "Carlos" }
... ],
... { ordered: false }
... )
BulkWriteResult({
	"writeErrors" : [
		{
			"index" : 2,
			"code" : 11000,
			"errmsg" : "E11000 duplicate key error collection: gimnasio.clientes index: _id_ dup key: { _id: 8.0 }",
			"op" : {
				"_id" : 8,
				"nombre" : "Juan"
			}
		}
	],
	"writeConcernErrors" : [ ],
	"nInserted" : 4,
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
{ "_id" : 4, "nombre" : "Pablo", "apellido" : "Gómez" }
{ "_id" : 5, "nombre" : "Sara", "apellido" : "García" }
{ "_id" : 6, "nombre" : "Carlos", "apellido" : "López" }
{ "_id" : 7, "nombre" : "José" }
{ "_id" : 8, "nombre" : "Lucia" }
{ "_id" : 9, "nombre" : "Luisa" }
{ "_id" : 10, "nombre" : "Carlos" }
```
Aquí encontro un `_id` duplicado el 8 pero continuo la inserción,, en total inserto 4 documentos `"nInserted" : 4,`, podemos listar los documentos:

```sh
> db.clientes.find()
{ "_id" : ObjectId("5e419e9490d86b85f5fda8ef"), "nombre" : "Juan", "apellido" : "Pérez" }
{ "_id" : 1, "nombre" : "María", "apellido" : "López" }
{ "_id" : 2, "nombre" : "Laura", "apellido" : "López" }
{ "_id" : 3, "nombre" : "Luis", "apellido" : "Pérez" }
{ "_id" : 4, "nombre" : "Pablo", "apellido" : "Gómez" }
{ "_id" : 5, "nombre" : "Sara", "apellido" : "García" }
{ "_id" : 6, "nombre" : "Carlos", "apellido" : "López" }
{ "_id" : 7, "nombre" : "José" }
{ "_id" : 8, "nombre" : "Lucia" }
{ "_id" : 9, "nombre" : "Luisa" }
{ "_id" : 10, "nombre" : "Carlos" }
```

### Método insertOne()

[db.collection.insertOne()](https://docs.mongodb.com/manual/reference/method/db.collection.insertOne/)

Inserta un documento en una coleción.

Sintaxis.

```sh
db.collection.insertOne(
  <document>,
  {
    writeConcern: <document>
  }
)
```

Insertar un documento con `insertOne()`

```sh
> db.clientes.insertOne(
... { _id: 11, nombre: "Javier", createAt: new Date() }
... )
{ "acknowledged" : true, "insertedId" : 11 }

> db.clientes.find()
{ "_id" : ObjectId("5e419e9490d86b85f5fda8ef"), "nombre" : "Juan", "apellido" : "Pérez" }
{ "_id" : 1, "nombre" : "María", "apellido" : "López" }
{ "_id" : 2, "nombre" : "Laura", "apellido" : "López" }
{ "_id" : 3, "nombre" : "Luis", "apellido" : "Pérez" }
{ "_id" : 4, "nombre" : "Pablo", "apellido" : "Gómez" }
{ "_id" : 5, "nombre" : "Sara", "apellido" : "García" }
{ "_id" : 6, "nombre" : "Carlos", "apellido" : "López" }
{ "_id" : 7, "nombre" : "José" }
{ "_id" : 8, "nombre" : "Lucia" }
{ "_id" : 9, "nombre" : "Luisa" }
{ "_id" : 10, "nombre" : "Carlos" }
{ "_id" : 11, "nombre" : "Javier", "createAt" : ISODate("2020-02-10T21:29:44.672Z") }
>
```

### Método insertMany()

[db.collection.insertMany()](https://docs.mongodb.com/manual/reference/method/db.collection.insertMany/)

Inserta multiples documentos en una colección.

Sintaxis.

```sh
db.collection.insertMany(
  [ <document 1> , <document 2>, ... ],
  {
    writeConcern: <document>,
    ordered: <boolean>
  }
)
```

Insertar varios documentos con `insertMany()`:

```sh
> db.clientes.insertMany([
... { _id: 12, nombre: "Salma"},
... { _id: 13, nombre: "Angelica"},
... { _id: 14, nombre: "Veronica"},
... { _id: 15, nombre: "Rocio"}
... ])
{ "acknowledged" : true, "insertedIds" : [ 12, 13, 14, 15 ] }

> db.clientes.find()
{ "_id" : ObjectId("5e419e9490d86b85f5fda8ef"), "nombre" : "Juan", "apellido" : "Pérez" }
{ "_id" : 1, "nombre" : "María", "apellido" : "López" }
{ "_id" : 2, "nombre" : "Laura", "apellido" : "López" }
{ "_id" : 3, "nombre" : "Luis", "apellido" : "Pérez" }
{ "_id" : 4, "nombre" : "Pablo", "apellido" : "Gómez" }
{ "_id" : 5, "nombre" : "Sara", "apellido" : "García" }
{ "_id" : 6, "nombre" : "Carlos", "apellido" : "López" }
{ "_id" : 7, "nombre" : "José" }
{ "_id" : 8, "nombre" : "Lucia" }
{ "_id" : 9, "nombre" : "Luisa" }
{ "_id" : 10, "nombre" : "Carlos" }
{ "_id" : 11, "nombre" : "Javier", "createAt" : ISODate("2020-02-10T21:29:44.672Z") }
{ "_id" : 12, "nombre" : "Salma" }
{ "_id" : 13, "nombre" : "Angelica" }
{ "_id" : 14, "nombre" : "Veronica" }
{ "_id" : 15, "nombre" : "Rocio" }

```

### Método save()

[db.collection.save()](https://docs.mongodb.com/manual/reference/method/db.collection.save/)

Actualiza un documento existente o inserta un nuevo documento, dependiendo de los parámetros del documento.

Sintaxis.

```sh
db.collection.save(
   <document>,
   {
     writeConcern: <document>
   }
)
```

Existen 3 opciones:

* Si incluye `_id` y existe, actualiza el documento.
* Si incluye `_id` y no existe, inserta el documento.
* si no incluye `_id` inserta el documento.

Veamos nuestra colección `clientes`:

```sh
> db.clientes.find()
{ "_id" : ObjectId("5e419e9490d86b85f5fda8ef"), "nombre" : "Juan", "apellido" : "Pérez" }
{ "_id" : 1, "nombre" : "María", "apellido" : "López" }
{ "_id" : 2, "nombre" : "Laura", "apellido" : "López" }
{ "_id" : 3, "nombre" : "Luis", "apellido" : "Pérez" }
{ "_id" : 4, "nombre" : "Pablo", "apellido" : "Gómez" }
{ "_id" : 5, "nombre" : "Sara", "apellido" : "García" }
{ "_id" : 6, "nombre" : "Carlos", "apellido" : "López" }
{ "_id" : 7, "nombre" : "José" }
{ "_id" : 8, "nombre" : "Lucia" }
{ "_id" : 9, "nombre" : "Luisa" }
{ "_id" : 10, "nombre" : "Carlos" }
{ "_id" : 11, "nombre" : "Javier", "createAt" : ISODate("2020-02-10T21:29:44.672Z") }
{ "_id" : 12, "nombre" : "Salma" }
{ "_id" : 13, "nombre" : "Angelica" }
{ "_id" : 14, "nombre" : "Veronica" }
{ "_id" : 15, "nombre" : "Rocio" }
```

Vamos a salvar un documento con un `_id` que ya existe.

```sh
> db.clientes.save(
... { _id: 15, nombre: "José", apellido: "López" }
... )
WriteResult({ "nMatched" : 1, "nUpserted" : 0, "nModified" : 1 })

> db.clientes.find()
{ "_id" : ObjectId("5e419e9490d86b85f5fda8ef"), "nombre" : "Juan", "apellido" : "Pérez" }
{ "_id" : 1, "nombre" : "María", "apellido" : "López" }
{ "_id" : 2, "nombre" : "Laura", "apellido" : "López" }
{ "_id" : 3, "nombre" : "Luis", "apellido" : "Pérez" }
{ "_id" : 4, "nombre" : "Pablo", "apellido" : "Gómez" }
{ "_id" : 5, "nombre" : "Sara", "apellido" : "García" }
{ "_id" : 6, "nombre" : "Carlos", "apellido" : "López" }
{ "_id" : 7, "nombre" : "José" }
{ "_id" : 8, "nombre" : "Lucia" }
{ "_id" : 9, "nombre" : "Luisa" }
{ "_id" : 10, "nombre" : "Carlos" }
{ "_id" : 11, "nombre" : "Javier", "createAt" : ISODate("2020-02-10T21:29:44.672Z") }
{ "_id" : 12, "nombre" : "Salma" }
{ "_id" : 13, "nombre" : "Angelica" }
{ "_id" : 14, "nombre" : "Veronica" }
{ "_id" : 15, "nombre" : "José", "apellido" : "López" }
>
```
Vemos que nos indica que a **actualizado un documento** `"nModified" : 1` y si listamos la colección lo comprobamos.

Vamos a salvar un documento con un `_id` que no existe.

```sh
> db.clientes.save(
... { _id: 16, nombre: "Pedro", apellido: "Paramo" }
... )
WriteResult({ "nMatched" : 0, "nUpserted" : 1, "nModified" : 0, "_id" : 16 })

> db.clientes.find()
{ "_id" : ObjectId("5e419e9490d86b85f5fda8ef"), "nombre" : "Juan", "apellido" : "Pérez" }
{ "_id" : 1, "nombre" : "María", "apellido" : "López" }
{ "_id" : 2, "nombre" : "Laura", "apellido" : "López" }
{ "_id" : 3, "nombre" : "Luis", "apellido" : "Pérez" }
{ "_id" : 4, "nombre" : "Pablo", "apellido" : "Gómez" }
{ "_id" : 5, "nombre" : "Sara", "apellido" : "García" }
{ "_id" : 6, "nombre" : "Carlos", "apellido" : "López" }
{ "_id" : 7, "nombre" : "José" }
{ "_id" : 8, "nombre" : "Lucia" }
{ "_id" : 9, "nombre" : "Luisa" }
{ "_id" : 10, "nombre" : "Carlos" }
{ "_id" : 11, "nombre" : "Javier", "createAt" : ISODate("2020-02-10T21:29:44.672Z") }
{ "_id" : 12, "nombre" : "Salma" }
{ "_id" : 13, "nombre" : "Angelica" }
{ "_id" : 14, "nombre" : "Veronica" }
{ "_id" : 15, "nombre" : "José", "apellido" : "López" }
{ "_id" : 16, "nombre" : "Pedro", "apellido" : "Paramo" }
> 
```
En este caso nos indica que ha **insertado el documento** `"nUpserted" : 1`, cosa que comprobamos al listar la colección.

Finalmente vamos a salvar un documento sin incluir el `_id`.

```sh
> db.clientes.save(
... { nombre: "Julio", apellido: "Cortez" }
... )
WriteResult({ "nInserted" : 1 })

> db.clientes.find()
{ "_id" : ObjectId("5e419e9490d86b85f5fda8ef"), "nombre" : "Juan", "apellido" : "Pérez" }
{ "_id" : 1, "nombre" : "María", "apellido" : "López" }
{ "_id" : 2, "nombre" : "Laura", "apellido" : "López" }
{ "_id" : 3, "nombre" : "Luis", "apellido" : "Pérez" }
{ "_id" : 4, "nombre" : "Pablo", "apellido" : "Gómez" }
{ "_id" : 5, "nombre" : "Sara", "apellido" : "García" }
{ "_id" : 6, "nombre" : "Carlos", "apellido" : "López" }
{ "_id" : 7, "nombre" : "José" }
{ "_id" : 8, "nombre" : "Lucia" }
{ "_id" : 9, "nombre" : "Luisa" }
{ "_id" : 10, "nombre" : "Carlos" }
{ "_id" : 11, "nombre" : "Javier", "createAt" : ISODate("2020-02-10T21:29:44.672Z") }
{ "_id" : 12, "nombre" : "Salma" }
{ "_id" : 13, "nombre" : "Angelica" }
{ "_id" : 14, "nombre" : "Veronica" }
{ "_id" : 15, "nombre" : "José", "apellido" : "López" }
{ "_id" : 16, "nombre" : "Pedro", "apellido" : "Paramo" }
{ "_id" : ObjectId("5e41d25290d86b85f5fda8f0"), "nombre" : "Julio", "apellido" : "Cortez" }
> 
```
Como esperabamos nos indica que ha **insertado el documento** `"nInserted" : 1`, lo comprobamos listando la colección.

## Operaciones de Lectura (Consultas)

[Read Operations](https://docs.mongodb.com/manual/crud/index.html#read-operations)

[Query Documents](https://docs.mongodb.com/manual/tutorial/query-documents/)

### Método find()

[db.collection.find()](https://docs.mongodb.com/manual/reference/method/db.collection.find/index.html)

Sintaxis.

```sh
db.collection.find(
  <documento-consulta>,
  <documento-projection>
)
```
Ambos documentos son opcionales.

Parametro | Type | Descripción
----------|------|------------
query |	document | Opcional. Especifica el filtro de selección utilizando operadores de consulta. Para devolver todos los documentos de una colección, omita este parámetro o pase un documento vacío ({}).
projection | document |	Opcional. Especifica los campos para devolver en los documentos que coinciden con el filtro de consulta. Para devolver todos los campos en los documentos coincidentes, omita este parámetro. 
