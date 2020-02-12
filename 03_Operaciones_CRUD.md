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
  <documento-query>,
  <documento-projection>
)
```
Ambos documentos son opcionales.

Parametro | Type | Descripción
----------|------|------------
query |	document | Opcional. Especifica el filtro de selección utilizando operadores de consulta. Para devolver todos los documentos de una colección, omita este parámetro o pase un documento vacío ({}).
projection | document |	Opcional. Especifica los campos para devolver en los documentos que coinciden con el filtro de consulta. Para devolver todos los campos en los documentos coincidentes, omita este parámetro.

#### Consulta de todos los documentos de la colección.

Tenemos tres formas de ejecutar el método `find()` para recuperar todos los documentos:

```sh
> db.<coleccion>.find()

> db.<coleccion>.find({})

> db.<coleccion>.find({}).pretty()
```

#### Especificar una condición de igualdad

Sintaxis.

```sh
> db.<coleccion>.find({ <campo>: <valor>, <campo>: <valor>, ... })
```

Veamos un ejemplo:

```sh
> use gimnasio
switched to db gimnasio

> db.clientes.find({ nombre: "José" })
{ "_id" : 7, "nombre" : "José" }
{ "_id" : 15, "nombre" : "José", "apellido" : "López" }
> 
```
Devuelve todos los documentos que tienen de nombre `José`.

* Cuando haya más de un par de campos, la coma que los separa implica un AND implicito.

```sh
> db.clientes.find({ nombre: "José", apellido: "López" })
{ "_id" : 15, "nombre" : "José", "apellido" : "López" }
```

* En el caso de `_id` el valor debe ser `ObjectId` cuando no se metio ese campo, si se metio se recupera normal.

```sh
> db.clientes.find({ _id: ObjectId("5e41d25290d86b85f5fda8f0") })
{ "_id" : ObjectId("5e41d25290d86b85f5fda8f0"), "nombre" : "Julio", "apellido" : "Cortez" }

> db.clientes.find({ _id: 10 })
{ "_id" : 10, "nombre" : "Carlos" }
```

**Debate** 

¿Cómo se debe guardar la información?
* En mayúsculas
* En minúsculas
* Como se inserte

#### Especificar condiciones con los Operadores de Consulta.

Sintaxis.

```sh
> db.<coleccion>.find({ <campo>: { $<operador>: <valor> } }, ... })
```

#### Especificar condiciones OR,  Operador $or

El operador `$or` realiza una operación lógica OR en un array de *dos o más* **<expresiones>** y selecciona los documentos que satisfacen al *menos* una de las <expresiones>. `$or` tiene la siguiente sintaxis:

```sh
> db.<coleccion>.find({ $or: [ { <expression1> }, { <expression2> }, ... , { <expressionN> } ] })

<expression1> = <campo1> : <valor>
```

Veamos un ejemplo:

```sh
> db.clientes.find({ $or: [
... { nombre: "José" },
... { nombre: "Carlos" },
... { apellido: "Pérez" }
... ] })
{ "_id" : ObjectId("5e419e9490d86b85f5fda8ef"), "nombre" : "Juan", "apellido" : "Pérez" }
{ "_id" : 3, "nombre" : "Luis", "apellido" : "Pérez" }
{ "_id" : 6, "nombre" : "Carlos", "apellido" : "López" }
{ "_id" : 7, "nombre" : "José" }
{ "_id" : 10, "nombre" : "Carlos" }
{ "_id" : 15, "nombre" : "José", "apellido" : "López" }
> 
```
En esta consulta recuperamos los documentos que tengan `nombre = José` o `nombre = Carlos` o `apellido = Pérez`.

#### Especificar tanto condiciones AND como OR

```sh
> db.<coleccion>.find({
    <campo>: <valor>,
    $or: [ { <expression1> }, { <expression2> }, ... , { <expressionN> } ] 
    })
```

```sh
> db.clientes.find({
... apellido: "López",
... $or : [
... { nombre: "María" },
... { nombre: "Laura" }
... ]
... })
{ "_id" : 1, "nombre" : "María", "apellido" : "López" }
{ "_id" : 2, "nombre" : "Laura", "apellido" : "López" }
> 
```

Esta consulta nos recupera los documentos que tienen `apellido = "López" AND ( nombre = "María" OR nombre = "Laura")`.

#### Consultas en Documentos Embebidos

Sintaxis.

```sh
db.<coleccion>.find({ <campo>: <subdocumento> })
```

Vamos a insertar algunos documentos con documento embebido:

```sh
> db.clientes.insert({
... nombre: "Luis",
... apellido: "González",
... direccion:
... { calle: "Alcalá, 90", localidad: "Madrid" }
... })
WriteResult({ "nInserted" : 1 })
> db.clientes.insert({ nombre: "Mariano", apellido: "Mejía", direccion: { calle: "Gran vía, 100", localidad: "Madrid" } })
WriteResult({ "nInserted" : 1 })
> db.clientes.insert({ nombre: "Enrique", apellido: "Flores", direccion: { calle: "Plaza España, 50", localidad: "Sevilla" } })
WriteResult({ "nInserted" : 1 })
>
```

Los documentos de la colección son:

```sh
> db.clientes.find({ })
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
{ "_id" : ObjectId("5e42d23890d86b85f5fda8f1"), "nombre" : "Luis", "apellido" : "González", "direccion" : { "calle" : "Alcalá, 90", "localidad" : "Madrid" } }
{ "_id" : ObjectId("5e42d29a90d86b85f5fda8f2"), "nombre" : "Mariano", "apellido" : "Mejía", "direccion" : { "calle" : "Gran vía, 100", "localidad" : "Madrid" } }
Type "it" for more
> it
{ "_id" : ObjectId("5e42d2c890d86b85f5fda8f3"), "nombre" : "Enrique", "apellido" : "Flores", "direccion" : { "calle" : "Plaza España, 50", "localidad" : "Sevilla" } }
>
```

Para hacer la consulta del documento embebido:

```sh
> db.clientes.find( { direccion: { calle: "Gran vía, 100", localidad: "Madrid" } } )
{ "_id" : ObjectId("5e42d29a90d86b85f5fda8f2"), "nombre" : "Mariano", "apellido" : "Mejía", "direccion" : { "calle" : "Gran vía, 100", "localidad" : "Madrid" } }
> 
```

Vemos como recuperamos el documento buscando en su documento embebido.

**El orden en que se mete el documento es importante, si se invierten los campos no encuentra nada**
(se se usa la notación de punto no importa)

```sh
> db.clientes.find(
...  { direccion: { localidad: "Madrid", calle: "Gran vía, 100" } }
... )
>
```

Tampoco encuentra nada si solo ponemos parte del documento embebido **debemos ponerlo completo**

```sh
> db.clientes.find( { direccion: { calle: "Gran vía, 100" } } )
> 
```

#### Consulta en Campos de Documentos Embebidos

Sintaxis.

```sh
db.<coleccion>.find({ "<campo>.<subdocumento>": <valor> })
```

**Las comillas son muy importantes.**

Aquí vamos a buscar en un campo que pertenece a un documento embebido.

```sh
> db.clientes.find({ "direccion.localidad": "Sevilla" })
{ "_id" : ObjectId("5e42d2c890d86b85f5fda8f3"), "nombre" : "Enrique", "apellido" : "Flores", "direccion" : { "calle" : "Plaza España, 50", "localidad" : "Sevilla" } }
```

Aquí vamos a buscar en un campo que pertenece a un documento embebido y un campo del documento principal.

```sh
> db.clientes.find({ "direccion.localidad": "Madrid", nombre: "Luis" })
{ "_id" : ObjectId("5e42d23890d86b85f5fda8f1"), "nombre" : "Luis", "apellido" : "González", "direccion" : { "calle" : "Alcalá, 90", "localidad" : "Madrid" } }
```

#### Consulta de Igualdad en Arrays

Sintaxis.

```sh
db.<coleccion>.find({ <campo>: [elementos] })
```

**Exige igualdad exacta en el array incluyendo el orden de los elementos.**

Vamos a insertar algunos documentos que contenga en sus campos un array:

```sh
> db.clientes.insert({
... nombre: "Pedro",
... apellido: "López",
... actividades: ["yoga", "zumba"]
... })
WriteResult({ "nInserted" : 1 })
> 
> db.clientes.insert({ nombre: "Paula", apellido: "García", actividades: ["esgrima", "zumba", "padel"] })
WriteResult({ "nInserted" : 1 })
> db.clientes.insert({ nombre: "Susana", apellido: "González", actividades: ["esgrima", "natación", "step"] })
WriteResult({ "nInserted" : 1 })
> 
```

Si los consultamos:

```sh
> db.clientes.find()
....
> it
{ "_id" : ObjectId("5e42d2c890d86b85f5fda8f3"), "nombre" : "Enrique", "apellido" : "Flores", "direccion" : { "calle" : "Plaza España, 50", "localidad" : "Sevilla" } }
{ "_id" : ObjectId("5e42ed4390d86b85f5fda8f4"), "nombre" : "Pedro", "apellido" : "López", "actividades" : [ "yoga", "zumba" ] }
{ "_id" : ObjectId("5e42ef1590d86b85f5fda8f6"), "nombre" : "Paula", "apellido" : "García", "actividades" : [ "esgrima", "zumba", "padel" ] }
{ "_id" : ObjectId("5e42ef3b90d86b85f5fda8f7"), "nombre" : "Susana", "apellido" : "González", "actividades" : [ "esgrima", "natación", "step" ] }
```

Para localizar este documento en la colección es **importante el orden de los elementos del array**, si lo cambio no lo encuentra, **IGUALDAD EXACTA** vamos a comprobarlo:

```sh
> db.clientes.find({ actividades: ["yoga", "zumba"] })
{ "_id" : ObjectId("5e42ed4390d86b85f5fda8f4"), "nombre" : "Pedro", "apellido" : "López", "actividades" : [ "yoga", "zumba" ] }
> db.clientes.find({ actividades: ["zumba", "yoga"] })
> db.clientes.find({ actividades: ["yoga"] })
> 
```

Vemos que solo encuentra el documento cuando ponemos el array exactamente igual que cuando lo insertamos en cuanto a orden y número de elementos, sino es así no encontrara el documento.

### Consulta de un Elemento en el Array

Sintaxis.

```sh
db.<coleccion>.find({ <campo>: <elemento> })
```

* Devuelve los documentos que en el array contenga al menos un valor como el especificado.

```sh
{ "_id" : ObjectId("5e42ef1590d86b85f5fda8f6"), "nombre" : "Paula", "apellido" : "García", "actividades" : [ "esgrima", "zumba", "padel" ] }
{ "_id" : ObjectId("5e42ef3b90d86b85f5fda8f7"), "nombre" : "Susana", "apellido" : "González", "actividades" : [ "esgrima", "natación", "step" ] }
> 
```

Recuepera dos registros que en el array `actividades` tienen el elemento buscado, `esgrima`.

```sh
> db.clientes.find({ $or: [ {actividades: "step"}, {actividades: "yoga"} ] })
{ "_id" : ObjectId("5e42ed4390d86b85f5fda8f4"), "nombre" : "Pedro", "apellido" : "López", "actividades" : [ "yoga", "zumba" ] }
{ "_id" : ObjectId("5e42ef3b90d86b85f5fda8f7"), "nombre" : "Susana", "apellido" : "González", "actividades" : [ "esgrima", "natación", "step" ] }
> 
```

Usando el operador `$or` recuperamos los documentos que tengan en sus actividades `yoga` o `step`:

```sh
> db.clientes.find({ $or: [ {actividades: "step"}, {actividades: "yoga"} ] })
{ "_id" : ObjectId("5e42ed4390d86b85f5fda8f4"), "nombre" : "Pedro", "apellido" : "López", "actividades" : [ "yoga", "zumba" ] }
{ "_id" : ObjectId("5e42ef3b90d86b85f5fda8f7"), "nombre" : "Susana", "apellido" : "González", "actividades" : [ "esgrima", "natación", "step" ] }
>
```

### Consulta de Múltiples Condiciones en Array.

Sintaxis.

```sh
db.<coleccion>.find({ <campo>: { condiciones } })
```

* Devuelve los documentos que en el array tengan elementos que cumplan las condiciones.

Vamos a insertar documentos con campos tipo arrays que contengan números:

```sh
> db.clientes.insert({
... nombre: "Rebeca",
... puntuaciones: [100, 34, 67]
... })
WriteResult({ "nInserted" : 1 })
> db.clientes.insert({ nombre: "Rocio", puntuaciones: [95, 88, 21] })
WriteResult({ "nInserted" : 1 })
> db.clientes.insert({ nombre: "Rosa", puntuaciones: [15, 49, 44] })
WriteResult({ "nInserted" : 1 })
> db.clientes.insert({ nombre: "Rita", puntuaciones: [78, 92, 52] })
WriteResult({ "nInserted" : 1 })
>
```

Si los consultamos: 

```sh
> db.clientes.find()
....
> it
{ "_id" : ObjectId("5e42d2c890d86b85f5fda8f3"), "nombre" : "Enrique", "apellido" : "Flores", "direccion" : { "calle" : "Plaza España, 50", "localidad" : "Sevilla" } }
{ "_id" : ObjectId("5e42ed4390d86b85f5fda8f4"), "nombre" : "Pedro", "apellido" : "López", "actividades" : [ "yoga", "zumba" ] }
{ "_id" : ObjectId("5e42ef1590d86b85f5fda8f6"), "nombre" : "Paula", "apellido" : "García", "actividades" : [ "esgrima", "zumba", "padel" ] }
{ "_id" : ObjectId("5e42ef3b90d86b85f5fda8f7"), "nombre" : "Susana", "apellido" : "González", "actividades" : [ "esgrima", "natación", "step" ] }
{ "_id" : ObjectId("5e42f25490d86b85f5fda8f8"), "nombre" : "Rebeca", "puntuaciones" : [ 100, 34, 67 ] }
{ "_id" : ObjectId("5e42f26d90d86b85f5fda8f9"), "nombre" : "Rocio", "puntuaciones" : [ 95, 88, 21 ] }
{ "_id" : ObjectId("5e42f28b90d86b85f5fda8fa"), "nombre" : "Rosa", "puntuaciones" : [ 15, 49, 44 ] }
{ "_id" : ObjectId("5e42f2a290d86b85f5fda8fb"), "nombre" : "Rita", "puntuaciones" : [ 78, 92, 52 ] }
> 
```

Vamos a recuperar documentos que cumplan la codición `puntuacion >= 50 AND puntuacion < 95`:

```sh
> db.clientes.find({ puntuaciones: { $gte: 90, $lt: 100 } })
{ "_id" : ObjectId("5e42f25490d86b85f5fda8f8"), "nombre" : "Rebeca", "puntuaciones" : [ 100, 34, 67 ] }
{ "_id" : ObjectId("5e42f26d90d86b85f5fda8f9"), "nombre" : "Rocio", "puntuaciones" : [ 95, 88, 21 ] }
{ "_id" : ObjectId("5e42f2a290d86b85f5fda8fb"), "nombre" : "Rita", "puntuaciones" : [ 78, 92, 52 ] }
> 
```
:-1: Observemos muy bien los resultados, **devuelve aquellos documentos que en el array `puntuaciones` tengan un elemento que cumpla todas las condiciones o varios elementos cuya combinación cumplan todas las condiciones**.

**No es que cumpla ambas condiciones**, al menos un elemento sea `>= 90`.

Otro ejemplo:

```sh
> db.clientes.find({ puntuaciones: { $gte: 10, $lt: 50 } })
{ "_id" : ObjectId("5e42f25490d86b85f5fda8f8"), "nombre" : "Rebeca", "puntuaciones" : [ 100, 34, 67 ] }
{ "_id" : ObjectId("5e42f26d90d86b85f5fda8f9"), "nombre" : "Rocio", "puntuaciones" : [ 95, 88, 21 ] }
{ "_id" : ObjectId("5e42f28b90d86b85f5fda8fa"), "nombre" : "Rosa", "puntuaciones" : [ 15, 49, 44 ] }
> 
> db.clientes.find({ puntuaciones: { $gte: 10, $lt: 30 } })
{ "_id" : ObjectId("5e42f26d90d86b85f5fda8f9"), "nombre" : "Rocio", "puntuaciones" : [ 95, 88, 21 ] }
{ "_id" : ObjectId("5e42f28b90d86b85f5fda8fa"), "nombre" : "Rosa", "puntuaciones" : [ 15, 49, 44 ] }
>
> db.clientes.find({ puntuaciones: { $gte: 90, $lt: 94 } })
{ "_id" : ObjectId("5e42f25490d86b85f5fda8f8"), "nombre" : "Rebeca", "puntuaciones" : [ 100, 34, 67 ] }
{ "_id" : ObjectId("5e42f26d90d86b85f5fda8f9"), "nombre" : "Rocio", "puntuaciones" : [ 95, 88, 21 ] }
{ "_id" : ObjectId("5e42f2a290d86b85f5fda8fb"), "nombre" : "Rita", "puntuaciones" : [ 78, 92, 52 ] }
>
````

**NO ENTIENDO EL COMPORTAMIENTO**

#### Consulta de un Elemento que Cumpla Multiples Condiciones.

Sintaxis.

```sh
db.<coleccion>.find({ <campo>: {$elemMatch: { condiciones } } })
```

* Devuelve los documentos que tengan al menos un elemento que cumpla todas las condiciones.

```sh
> db.clientes.find({ puntuaciones: { $gte: 94, $lt: 99 } })
{ "_id" : ObjectId("5e42f25490d86b85f5fda8f8"), "nombre" : "Rebeca", "puntuaciones" : [ 100, 34, 67 ] }
{ "_id" : ObjectId("5e42f26d90d86b85f5fda8f9"), "nombre" : "Rocio", "puntuaciones" : [ 95, 88, 21 ] }

> db.clientes.find({ puntuaciones: { $elemMatch: { $gte: 94, $lt: 99 } } })
{ "_id" : ObjectId("5e42f26d90d86b85f5fda8f9"), "nombre" : "Rocio", "puntuaciones" : [ 95, 88, 21 ] }
> 
```
Vemos que en el segundo ejemplo que al menos un elemento que cumpla a la vez las dos condiciones, solo tenemos un documento.

### Consulta de array con índice.

Sintaxis.

```sh
db.<coleccion>.find({ "<campo>.i": <valor> })
```

Recuperemos los documentos que tengan en el primer elemento del campo `actividades` sea `yoga`:

```sh
> db.clientes.find({ "actividades.0": "yoga" })
{ "_id" : ObjectId("5e42ed4390d86b85f5fda8f4"), "nombre" : "Pedro", "apellido" : "López", "actividades" : [ "yoga", "zumba" ] }
> 
```
Otro ejemplo:

```sh
> db.clientes.find({ "puntuaciones.0": { $gte: 50 }  })
{ "_id" : ObjectId("5e42f25490d86b85f5fda8f8"), "nombre" : "Rebeca", "puntuaciones" : [ 100, 34, 67 ] }
{ "_id" : ObjectId("5e42f26d90d86b85f5fda8f9"), "nombre" : "Rocio", "puntuaciones" : [ 95, 88, 21 ] }
{ "_id" : ObjectId("5e42f2a290d86b85f5fda8fb"), "nombre" : "Rita", "puntuaciones" : [ 78, 92, 52 ] }
>

> db.clientes.find({ "puntuaciones.0": { $lte: 50 }  })
{ "_id" : ObjectId("5e42f28b90d86b85f5fda8fa"), "nombre" : "Rosa", "puntuaciones" : [ 15, 49, 44 ] }
> 

> db.clientes.find({ "puntuaciones.2": { $lte: 50 }  })
{ "_id" : ObjectId("5e42f26d90d86b85f5fda8f9"), "nombre" : "Rocio", "puntuaciones" : [ 95, 88, 21 ] }
{ "_id" : ObjectId("5e42f28b90d86b85f5fda8fa"), "nombre" : "Rosa", "puntuaciones" : [ 15, 49, 44 ] }
>
```

#### Consulta de campos en Arrays de Documentos en Cualquier Posición

Sintaxis.

```sh
db.<coleccion>.find({ "<campo>.<campoDocumento>": <valor> })
```

Vamos a insertar algunos documentos para poder hacer estas consultas:

```sh
> db.clientes.insert({
... nombre: "Roberto",
... apellido: "García",
... direcciones: [
... { calle: "Alcalá, 40", cp: "02800", localidad: "Madrid" },
... { calle: "Zamora, 13", cp: "34005", localidad: "Vigo"}
... ]
... })
WriteResult({ "nInserted" : 1 })
> db.clientes.insert({ nombre: "Carla", apellido: "López", direcciones: [ { calle: "Gran vía, 121", cp: "28025", localidad: "Madrid" }, { calle: "Bogota, 27", cp: "25052", localidad: "Valencia"} ] })
WriteResult({ "nInserted" : 1 })
> db.clientes.insert({ nombre: "Roberto", apellido: "Pérez", direcciones: [ { calle: "Gran vía, 23", cp: "28025", localidad: "Madrid" }, { calle: "Toledo, 13", cp: "24122", localidad: "Madrid"} ] })
WriteResult({ "nInserted" : 1 })
> 
```

Vamos a buscar los documentos que la localidad de su dirección sea Vigo:

```sh
> db.clientes.find({ "direcciones.localidad": "Vigo" })
{ "_id" : ObjectId("5e442e4df6a56e42b753ca3a"), "nombre" : "Roberto", "apellido" : "García", "direcciones" : [ { "calle" : "Alcalá, 40", "cp" : "02800", "localidad" : "Madrid" }, { "calle" : "Zamora, 13", "cp" : "34005", "localidad" : "Vigo" } ] }
> 
```
Recuperamos un documento.

#### Consulta de Campos en Arrays de Documentos en una Posición Determinada.

Sintaxis.

```sh
db.<coleccion>.find({ "<campo>.i.<campoDocumento>": <valor> })
```

Recuperemos aquellos documentos que en su segunda dirección tengan la localidad de Madrid.
```sh
> db.clientes.find({ "direcciones.1.localidad": "Madrid" })
{ "_id" : ObjectId("5e443161f6a56e42b753ca3c"), "nombre" : "Roberto", "apellido" : "Pérez", "direcciones" : [ { "calle" : "Gran vía, 23", "cp" : "28025", "localidad" : "Madrid" }, { "calle" : "Toledo, 13", "cp" : "24122", "localidad" : "Madrid" } ] }
> 
```

#### Multiples Condiciones en Array de Documentos

Sintaxis.

```sh
db.<coleccion>.find({ <campo>: { $elemMatch: {condiciones} } })
```

Insertemos algunos registros:

```sh
> db.monitores.insert({
... nombre: "Pedro",
... actividades: [
... { clase: "aerobics", turno: "mañana", homologado: false },
... { clase: "pesas", turno: "tarde", homologado: false },
... { clase: "zumba", turno: "mañana", homologado: true }
... ]
... })
WriteResult({ "nInserted" : 1 })
> db.monitores.insert({ nombre: "Susana", actividades: [ { clase: "aerobics", turno: "tarde", homologado: true }, { clase: "step", turno: "tarde", homologado: false }, { clase: "ciclismo", turno: "tarde", homologado: true } ] })
WriteResult({ "nInserted" : 1 })
> db.monitores.insert({ nombre: "Alexa", actividades: [ { clase: "aerobics", turno: "mañana", homologado: false }, { clase: "pesas", turno: "mañana", homologado: true }, { clase: "zumba", turno: "mañana", homologado: true } ] })
WriteResult({ "nInserted" : 1 })
> 
```
Listemos los monitores:

```sh
> db.monitores.find().pretty()
{
	"_id" : ObjectId("5e4433b8f6a56e42b753ca3d"),
	"nombre" : "Pedro",
	"actividades" : [
		{
			"clase" : "aerobics",
			"turno" : "mañana",
			"homologado" : false
		},
		{
			"clase" : "pesas",
			"turno" : "tarde",
			"homologado" : false
		},
		{
			"clase" : "zumba",
			"turno" : "mañana",
			"homologado" : true
		}
	]
}
{
	"_id" : ObjectId("5e44343ef6a56e42b753ca3e"),
	"nombre" : "Susana",
	"actividades" : [
		{
			"clase" : "aerobics",
			"turno" : "tarde",
			"homologado" : true
		},
		{
			"clase" : "step",
			"turno" : "tarde",
			"homologado" : false
		},
		{
			"clase" : "ciclismo",
			"turno" : "tarde",
			"homologado" : true
		}
	]
}
{
	"_id" : ObjectId("5e443498f6a56e42b753ca3f"),
	"nombre" : "Alexa",
	"actividades" : [
		{
			"clase" : "aerobics",
			"turno" : "mañana",
			"homologado" : false
		},
		{
			"clase" : "pesas",
			"turno" : "mañana",
			"homologado" : true
		},
		{
			"clase" : "zumba",
			"turno" : "mañana",
			"homologado" : true
		}
	]
}
> 
```

Listemos los monitores que impartan a clase aerobics por las mañanas.

```sh
> db.monitores.find({ actividades: { $elemMatch: { clase: "aerobics", turno: "mañana"} } })
{ "_id" : ObjectId("5e4433b8f6a56e42b753ca3d"), "nombre" : "Pedro", "actividades" : [ { "clase" : "aerobics", "turno" : "mañana", "homologado" : false }, { "clase" : "pesas", "turno" : "tarde", "homologado" : false }, { "clase" : "zumba", "turno" : "mañana", "homologado" : true } ] }
{ "_id" : ObjectId("5e443498f6a56e42b753ca3f"), "nombre" : "Alexa", "actividades" : [ { "clase" : "aerobics", "turno" : "mañana", "homologado" : false }, { "clase" : "pesas", "turno" : "mañana", "homologado" : true }, { "clase" : "zumba", "turno" : "mañana", "homologado" : true } ] }
> 
```
Ambos documentos cumplen ambas condiciones.

#### Multiples Condiciones en Array de Documentos con uno o Varios Documentos.

Sintaxis.

```sh
db.<coleccion>.find({ "<campo><campoSubDocumento>": <valor>, "<campo><campoSubDocumento>": <valor>, ...   })
```
Buscamos documentos con clase aerobics y turno mañana (Como el ejemplo anterior)

```sh
> db.monitores.find({
... "actividades.clase": "aerobics",
... "actividades.turno": "mañana"
... })
{ "_id" : ObjectId("5e4433b8f6a56e42b753ca3d"), "nombre" : "Pedro", "actividades" : [ { "clase" : "aerobics", "turno" : "mañana", "homologado" : false }, { "clase" : "pesas", "turno" : "tarde", "homologado" : false }, { "clase" : "zumba", "turno" : "mañana", "homologado" : true } ] }
{ "_id" : ObjectId("5e443498f6a56e42b753ca3f"), "nombre" : "Alexa", "actividades" : [ { "clase" : "aerobics", "turno" : "mañana", "homologado" : false }, { "clase" : "pesas", "turno" : "mañana", "homologado" : true }, { "clase" : "zumba", "turno" : "mañana", "homologado" : true } ] }
> 
```

### Proyecciones

#### Proyección de Todos los Campos de los Documentos Seleccionados.

Sintaxis.

```sh
db.<coleccion>.find({}, { campo1: 1, campo2: 1, ...} })
```

Valor | Descripción
------|------------
1 | Se visualiza el campo
-1 | Se oculta el campo


* No se pasan documentos de proyección 

Devuelve los campos especificados y el campo `_id`

```sh

```


```sh

```

```sh

```

```sh

```

```sh

```

```sh

```

```sh

```

```sh

```

```sh

```

```sh

```

```sh

```

```sh

```

```sh

```

```sh

```

```sh

```


