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

## Proyecciones

### Proyección de Todos los Campos de los Documentos Seleccionados.

Sintaxis.

```sh
db.<coleccion>.find({}, { campo1: 1, campo2: 1, ...} })
```

Valor | Descripción
------|------------
1 | Se visualiza el campo
-1 | Se oculta el campo

No se puede incluir y excluir en el mismo documento de proyección salvo el campo `_id`.

* No se pasan documentos de proyección 

Devuelve los campos especificados y el campo `_id`

Vamos a insertar un documento que solo tiene el campo `_id`:

```sh
> use gimnasio
switched to db gimnasio

> db.clientes.insert({ _id: 20 })
WriteResult({ "nInserted" : 1 })
```

Ahora si vamos a recuperar toda la colección:

```sh
> db.clientes.find({})
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
{ "_id" : ObjectId("5e42ed4390d86b85f5fda8f4"), "nombre" : "Pedro", "apellido" : "López", "actividades" : [ "yoga", "zumba" ] }
{ "_id" : ObjectId("5e42ef1590d86b85f5fda8f6"), "nombre" : "Paula", "apellido" : "García", "actividades" : [ "esgrima", "zumba", "padel" ] }
{ "_id" : ObjectId("5e42ef3b90d86b85f5fda8f7"), "nombre" : "Susana", "apellido" : "González", "actividades" : [ "esgrima", "natación", "step" ] }
{ "_id" : ObjectId("5e42f25490d86b85f5fda8f8"), "nombre" : "Rebeca", "puntuaciones" : [ 100, 34, 67 ] }
{ "_id" : ObjectId("5e42f26d90d86b85f5fda8f9"), "nombre" : "Rocio", "puntuaciones" : [ 95, 88, 21 ] }
{ "_id" : ObjectId("5e42f28b90d86b85f5fda8fa"), "nombre" : "Rosa", "puntuaciones" : [ 15, 49, 44 ] }
{ "_id" : ObjectId("5e42f2a290d86b85f5fda8fb"), "nombre" : "Rita", "puntuaciones" : [ 78, 92, 52 ] }
{ "_id" : ObjectId("5e442e4df6a56e42b753ca3a"), "nombre" : "Roberto", "apellido" : "García", "direcciones" : [ { "calle" : "Alcalá, 40", "cp" : "02800", "localidad" : "Madrid" }, { "calle" : "Zamora, 13", "cp" : "34005", "localidad" : "Vigo" } ] }
{ "_id" : ObjectId("5e442ec5f6a56e42b753ca3b"), "nombre" : "Carla", "apellido" : "López", "direcciones" : [ { "calle" : "Gran vía, 121", "cp" : "28025", "localidad" : "Madrid" }, { "calle" : "Bogota, 27", "cp" : "25052", "localidad" : "Valencia" } ] }
{ "_id" : ObjectId("5e443161f6a56e42b753ca3c"), "nombre" : "Roberto", "apellido" : "Pérez", "direcciones" : [ { "calle" : "Gran vía, 23", "cp" : "28025", "localidad" : "Madrid" }, { "calle" : "Toledo, 13", "cp" : "24122", "localidad" : "Madrid" } ] }
{ "_id" : 20 }
> 
```

Aquí se nos presentan todos los campos existentes, vamos a limitirar para que solo se muestren los campos `nombre` y `apellido`:

```sh
> db.clientes.find( {}, {nombre: 1, apellido:1} )
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
{ "_id" : 11, "nombre" : "Javier" }
{ "_id" : 12, "nombre" : "Salma" }
{ "_id" : 13, "nombre" : "Angelica" }
{ "_id" : 14, "nombre" : "Veronica" }
{ "_id" : 15, "nombre" : "José", "apellido" : "López" }
{ "_id" : 16, "nombre" : "Pedro", "apellido" : "Paramo" }
{ "_id" : ObjectId("5e41d25290d86b85f5fda8f0"), "nombre" : "Julio", "apellido" : "Cortez" }
{ "_id" : ObjectId("5e42d23890d86b85f5fda8f1"), "nombre" : "Luis", "apellido" : "González" }
{ "_id" : ObjectId("5e42d29a90d86b85f5fda8f2"), "nombre" : "Mariano", "apellido" : "Mejía" }
Type "it" for more
> it
{ "_id" : ObjectId("5e42d2c890d86b85f5fda8f3"), "nombre" : "Enrique", "apellido" : "Flores" }
{ "_id" : ObjectId("5e42ed4390d86b85f5fda8f4"), "nombre" : "Pedro", "apellido" : "López" }
{ "_id" : ObjectId("5e42ef1590d86b85f5fda8f6"), "nombre" : "Paula", "apellido" : "García" }
{ "_id" : ObjectId("5e42ef3b90d86b85f5fda8f7"), "nombre" : "Susana", "apellido" : "González" }
{ "_id" : ObjectId("5e42f25490d86b85f5fda8f8"), "nombre" : "Rebeca" }
{ "_id" : ObjectId("5e42f26d90d86b85f5fda8f9"), "nombre" : "Rocio" }
{ "_id" : ObjectId("5e42f28b90d86b85f5fda8fa"), "nombre" : "Rosa" }
{ "_id" : ObjectId("5e42f2a290d86b85f5fda8fb"), "nombre" : "Rita" }
{ "_id" : ObjectId("5e442e4df6a56e42b753ca3a"), "nombre" : "Roberto", "apellido" : "García" }
{ "_id" : ObjectId("5e442ec5f6a56e42b753ca3b"), "nombre" : "Carla", "apellido" : "López" }
{ "_id" : ObjectId("5e443161f6a56e42b753ca3c"), "nombre" : "Roberto", "apellido" : "Pérez" }
{ "_id" : 20 }
> 
```
Aquí vemos que se nos regresan los documentos mostrando solo los campos `nombre` y `apellido` y si no tiene uno o los dos de los campos también los devuelve como es el caso del último documento `{ "_id" : 20 }`. **Siempre se presentará el campo `_id`** a menos que le digamos lo contrario. 

#### Excluir el campo `_id`

Para excluir el campo `_id` hay que poner `_id: 0`:

```sh
> db.clientes.find( {}, {_id: 0, nombre: 1, apellido:1} )
{ "nombre" : "Juan", "apellido" : "Pérez" }
{ "nombre" : "María", "apellido" : "López" }
{ "nombre" : "Laura", "apellido" : "López" }
{ "nombre" : "Luis", "apellido" : "Pérez" }
{ "nombre" : "Pablo", "apellido" : "Gómez" }
{ "nombre" : "Sara", "apellido" : "García" }
{ "nombre" : "Carlos", "apellido" : "López" }
{ "nombre" : "José" }
{ "nombre" : "Lucia" }
{ "nombre" : "Luisa" }
{ "nombre" : "Carlos" }
{ "nombre" : "Javier" }
{ "nombre" : "Salma" }
{ "nombre" : "Angelica" }
{ "nombre" : "Veronica" }
{ "nombre" : "José", "apellido" : "López" }
{ "nombre" : "Pedro", "apellido" : "Paramo" }
{ "nombre" : "Julio", "apellido" : "Cortez" }
{ "nombre" : "Luis", "apellido" : "González" }
{ "nombre" : "Mariano", "apellido" : "Mejía" }
Type "it" for more
> it
{ "nombre" : "Enrique", "apellido" : "Flores" }
{ "nombre" : "Pedro", "apellido" : "López" }
{ "nombre" : "Paula", "apellido" : "García" }
{ "nombre" : "Susana", "apellido" : "González" }
{ "nombre" : "Rebeca" }
{ "nombre" : "Rocio" }
{ "nombre" : "Rosa" }
{ "nombre" : "Rita" }
{ "nombre" : "Roberto", "apellido" : "García" }
{ "nombre" : "Carla", "apellido" : "López" }
{ "nombre" : "Roberto", "apellido" : "Pérez" }
{  }
> 
```
Vemos que el documento lo presenta vacío, por que no tiene ni `nombre` ni `apellido` y el campo `_id` esta oculto por lo que el resultado es un documento vacio.

Veamos algunos otros ejemplos:

```sh
> db.clientes.find( {}, {_id:0,  apellido:1} )
{ "apellido" : "Pérez" }
{ "apellido" : "López" }
{ "apellido" : "López" }
{ "apellido" : "Pérez" }
{ "apellido" : "Gómez" }
{ "apellido" : "García" }
{ "apellido" : "López" }
{  }
{  }
{  }
{  }
{  }
{  }
{  }
{  }
{ "apellido" : "López" }
{ "apellido" : "Paramo" }
{ "apellido" : "Cortez" }
{ "apellido" : "González" }
{ "apellido" : "Mejía" }
Type "it" for more
> it
{ "apellido" : "Flores" }
{ "apellido" : "López" }
{ "apellido" : "García" }
{ "apellido" : "González" }
{  }
{  }
{  }
{  }
{ "apellido" : "García" }
{ "apellido" : "López" }
{ "apellido" : "Pérez" }
{  }
> 
```

Listemos los documentos mostrando solo su `direccion`:

```sh
> db.clientes.find( {}, {_id:0,  direccion:1} )
{  }
{  }
{  }
{  }
{  }
{  }
{  }
{  }
{  }
{  }
{  }
{  }
{  }
{  }
{  }
{  }
{  }
{  }
{ "direccion" : { "calle" : "Alcalá, 90", "localidad" : "Madrid" } }
{ "direccion" : { "calle" : "Gran vía, 100", "localidad" : "Madrid" } }
Type "it" for more
> it
{ "direccion" : { "calle" : "Plaza España, 50", "localidad" : "Sevilla" } }
{  }
{  }
{  }
{  }
{  }
{  }
{  }
{  }
{  }
{  }
{  }
> 
```

### Proyección de Todos los Campos Menos los que Excluyamos.

[Project Fields to Return from Query](https://docs.mongodb.com/manual/tutorial/project-fields-from-query-results/)

Sintaxis.

```sh
db.<coleccion>.find({}, { campo1: 0, campo2: 0, ...} })
```

Los campos deben ponerse en 0 ó en 1 salvo el campo `_id` en el documento de proyección.

Listamos los documentos excluyendo los campos `_id`, `nombre` y `apellido`:

```sh
> db.clientes.find( {}, {_id:0,  nombre: 0, apellido: 0} )
{  }
{  }
{  }
{  }
{  }
{  }
{  }
{  }
{  }
{  }
{  }
{ "createAt" : ISODate("2020-02-10T21:29:44.672Z") }
{  }
{  }
{  }
{  }
{  }
{  }
{ "direccion" : { "calle" : "Alcalá, 90", "localidad" : "Madrid" } }
{ "direccion" : { "calle" : "Gran vía, 100", "localidad" : "Madrid" } }
Type "it" for more
> it
{ "direccion" : { "calle" : "Plaza España, 50", "localidad" : "Sevilla" } }
{ "actividades" : [ "yoga", "zumba" ] }
{ "actividades" : [ "esgrima", "zumba", "padel" ] }
{ "actividades" : [ "esgrima", "natación", "step" ] }
{ "puntuaciones" : [ 100, 34, 67 ] }
{ "puntuaciones" : [ 95, 88, 21 ] }
{ "puntuaciones" : [ 15, 49, 44 ] }
{ "puntuaciones" : [ 78, 92, 52 ] }
{ "direcciones" : [ { "calle" : "Alcalá, 40", "cp" : "02800", "localidad" : "Madrid" }, { "calle" : "Zamora, 13", "cp" : "34005", "localidad" : "Vigo" } ] }
{ "direcciones" : [ { "calle" : "Gran vía, 121", "cp" : "28025", "localidad" : "Madrid" }, { "calle" : "Bogota, 27", "cp" : "25052", "localidad" : "Valencia" } ] }
{ "direcciones" : [ { "calle" : "Gran vía, 23", "cp" : "28025", "localidad" : "Madrid" }, { "calle" : "Toledo, 13", "cp" : "24122", "localidad" : "Madrid" } ] }
{  }
> 
```
Vemos que en los documentos que solo existan los campos excluidos nos regresa `{}`.

Podemos combinar filtros de lo que queremos consultar con lo que queremos mostrar. En el siguiente ejemplo listamos los documentos que tienen de nombre Juan sin mostrar el campo `_id`

```sh
> db.clientes.find( { nombre: "Juan" }, { _id: 0 } )
{ "nombre" : "Juan", "apellido" : "Pérez" }
```

### Devolución de Campos de Documentos Embebidos

Sintaxis.

```sh
db.<coleccion>.find({}, { ..., "<campo>.<campoDoc>": 1, ...} })
```

Mostremos la estructura de dirección pero solo el campo `localidad`

```sh
> db.clientes.find( {}, { _id: 0, nombre: 1, "direccion.localidad": 1 } )
{ "nombre" : "Juan" }
{ "nombre" : "María" }
{ "nombre" : "Laura" }
{ "nombre" : "Luis" }
{ "nombre" : "Pablo" }
{ "nombre" : "Sara" }
{ "nombre" : "Carlos" }
{ "nombre" : "José" }
{ "nombre" : "Lucia" }
{ "nombre" : "Luisa" }
{ "nombre" : "Carlos" }
{ "nombre" : "Javier" }
{ "nombre" : "Salma" }
{ "nombre" : "Angelica" }
{ "nombre" : "Veronica" }
{ "nombre" : "José" }
{ "nombre" : "Pedro" }
{ "nombre" : "Julio" }
{ "nombre" : "Luis", "direccion" : { "localidad" : "Madrid" } }
{ "nombre" : "Mariano", "direccion" : { "localidad" : "Madrid" } }
Type "it" for more
> it
{ "nombre" : "Enrique", "direccion" : { "localidad" : "Sevilla" } }
{ "nombre" : "Pedro" }
{ "nombre" : "Paula" }
{ "nombre" : "Susana" }
{ "nombre" : "Rebeca" }
{ "nombre" : "Rocio" }
{ "nombre" : "Rosa" }
{ "nombre" : "Rita" }
{ "nombre" : "Roberto" }
{ "nombre" : "Carla" }
{ "nombre" : "Roberto" }
{  }
> 
```

**En caso de arrays de documentos es igual** Veamos un ejemplo:

```sh
> db.clientes.find( {}, { _id: 0, nombre: 1, "direcciones.calle": 1 })
{ "nombre" : "Juan" }
{ "nombre" : "María" }
{ "nombre" : "Laura" }
{ "nombre" : "Luis" }
{ "nombre" : "Pablo" }
{ "nombre" : "Sara" }
{ "nombre" : "Carlos" }
{ "nombre" : "José" }
{ "nombre" : "Lucia" }
{ "nombre" : "Luisa" }
{ "nombre" : "Carlos" }
{ "nombre" : "Javier" }
{ "nombre" : "Salma" }
{ "nombre" : "Angelica" }
{ "nombre" : "Veronica" }
{ "nombre" : "José" }
{ "nombre" : "Pedro" }
{ "nombre" : "Julio" }
{ "nombre" : "Luis" }
{ "nombre" : "Mariano" }
Type "it" for more
> it
{ "nombre" : "Enrique" }
{ "nombre" : "Pedro" }
{ "nombre" : "Paula" }
{ "nombre" : "Susana" }
{ "nombre" : "Rebeca" }
{ "nombre" : "Rocio" }
{ "nombre" : "Rosa" }
{ "nombre" : "Rita" }
{ "nombre" : "Roberto", "direcciones" : [ { "calle" : "Alcalá, 40" }, { "calle" : "Zamora, 13" } ] }
{ "nombre" : "Carla", "direcciones" : [ { "calle" : "Gran vía, 121" }, { "calle" : "Bogota, 27" } ] }
{ "nombre" : "Roberto", "direcciones" : [ { "calle" : "Gran vía, 23" }, { "calle" : "Toledo, 13" } ] }
{  }
> 
```

Muestra los campos `nombre` y `direccion` pero en el array de direcciones solo nos muestra la `calle` que fue lo que le indicamos.

**La instrucción se escribe igual pero en uno es un documento y en otro es un array de documentos**.


### Devolución de Elementos Especificos de un Array.

Sintaxis.

```sh
db.<coleccion>.find({}, { ..., <campo>:  { $elemMatch: { <campo>: <valor>, ...} } })
```
Esto se aplica en un array de documentos.

Veamos nuestra colección monitores, donde tenemos el array de documentos `actividades`:

```sh
> db.monitores.find()
{ "_id" : ObjectId("5e4433b8f6a56e42b753ca3d"), "nombre" : "Pedro", "actividades" : [ { "clase" : "aerobics", "turno" : "mañana", "homologado" : false }, { "clase" : "pesas", "turno" : "tarde", "homologado" : false }, { "clase" : "zumba", "turno" : "mañana", "homologado" : true } ] }
{ "_id" : ObjectId("5e44343ef6a56e42b753ca3e"), "nombre" : "Susana", "actividades" : [ { "clase" : "aerobics", "turno" : "tarde", "homologado" : true }, { "clase" : "step", "turno" : "tarde", "homologado" : false }, { "clase" : "ciclismo", "turno" : "tarde", "homologado" : true } ] }
{ "_id" : ObjectId("5e443498f6a56e42b753ca3f"), "nombre" : "Alexa", "actividades" : [ { "clase" : "aerobics", "turno" : "mañana", "homologado" : false }, { "clase" : "pesas", "turno" : "mañana", "homologado" : true }, { "clase" : "zumba", "turno" : "mañana", "homologado" : true } ] }
> 
```

Listar los documentos que tengan la clase zumba dentro de ellas:

```sh
> db.monitores.find( {}, { actividades: { $elemMatch: { clase: "zumba" }}} )
{ "_id" : ObjectId("5e4433b8f6a56e42b753ca3d"), "actividades" : [ { "clase" : "zumba", "turno" : "mañana", "homologado" : true } ] }
{ "_id" : ObjectId("5e44343ef6a56e42b753ca3e") }
{ "_id" : ObjectId("5e443498f6a56e42b753ca3f"), "actividades" : [ { "clase" : "zumba", "turno" : "mañana", "homologado" : true } ] }
> 
```

En el caso de los clientes tenemos un array de actividades pero no es de documentos por lo que no podriamos hacer algo similar. ¿O SI?

### Consulta Para Valores `null` o campos inexistentes.
[Query for Null or Missing Fields](https://docs.mongodb.com/manual/tutorial/query-for-null-fields/)

Vamos  a insertar dos documentos:

```sh
> db.monitores.insert({ nombre: "Juan", apellido: "González" })
WriteResult({ "nInserted" : 1 })
> db.monitores.insert({ nombre: "Diego", apellido: "Pérez", actividades: null })
WriteResult({ "nInserted" : 1 })
> 
```
Uno sin actividades y otro con actividades `null`

Listemos la colección:

```sh
> db.monitores.find( {}, { _id: 0} )
{ "nombre" : "Pedro", "actividades" : [ { "clase" : "aerobics", "turno" : "mañana", "homologado" : false }, { "clase" : "pesas", "turno" : "tarde", "homologado" : false }, { "clase" : "zumba", "turno" : "mañana", "homologado" : true } ] }
{ "nombre" : "Susana", "actividades" : [ { "clase" : "aerobics", "turno" : "tarde", "homologado" : true }, { "clase" : "step", "turno" : "tarde", "homologado" : false }, { "clase" : "ciclismo", "turno" : "tarde", "homologado" : true } ] }
{ "nombre" : "Alexa", "actividades" : [ { "clase" : "aerobics", "turno" : "mañana", "homologado" : false }, { "clase" : "pesas", "turno" : "mañana", "homologado" : true }, { "clase" : "zumba", "turno" : "mañana", "homologado" : true } ] }
{ "nombre" : "Juan", "apellido" : "González" }
{ "nombre" : "Diego", "apellido" : "Pérez", "actividades" : null }
> 
```
Listemos los documentos que tengan actividades nulas:

```sh
> db.monitores.find({ actividades: null })
{ "_id" : ObjectId("5e458f398a695f2c39e6da7e"), "nombre" : "Juan", "apellido" : "González" }
{ "_id" : ObjectId("5e458f618a695f2c39e6da7f"), "nombre" : "Diego", "apellido" : "Pérez", "actividades" : null }
> 
```

Devuelve aquellos documentos donde la actidad es nula o no existe.


## Operadores de Consulta de Elementos

[Element Query Operators](https://docs.mongodb.com/manual/reference/operator/query-element/)

Nombre | Descripción
-------|------------
$exists | Coincide con documentos que tienen el campo especificado.
$type | Selecciona documentos si un campo es del tipo especificado.

### Comprobación de Tipos `$type`

Sintaxis.

```sh
{ <campo>: { $type: <BSON type> } }
```

El `BSON type` lo podemos expresar con un número o un alias, según la tabla BSON Types.

Cada tipo BSON tiene asociado un número:

[BSON Types] (https://docs.mongodb.com/manual/reference/bson-types/#bson-types)

Type | Number |	Alias |	Notes
-----|--------|-------|------
Double | 1 | “double” | 
String | 2 | “string” | 
Object | 3 | “object” |	 
Array | 4 | “array” | 
Binary data | 5 | “binData” |
Undefined | 6 |	“undefined” | Deprecated.
ObjectId | 7 | “objectId” | 
Boolean	| 8 | “bool” | 
Date | 9 | “date” |	 
Null | 10 | “null” | 	 
Regular Expression | 11 | “regex” | 	 
DBPointer | 12 | “dbPointer” | Deprecated.
JavaScript| 13 | “javascript”	 
Symbol | 14 | “symbol” | Deprecated.
JavaScript (with scope) | 15 | “javascriptWithScope” | 
32-bit integer | 16 | “int” | 
Timestamp | 17 | “timestamp” | 
64-bit integer | 18 | “long” | 	 
Decimal128 | 19	| “decimal” | New in version 3.4.
Min key | -1 | “minKey” |
Max key	| 127 | “maxKey” | 

Listemos los documentos que tengan actividades nulas:

```sh
> db.monitores.find({ actividades: { $type: 10 } })
{ "_id" : ObjectId("5e458f618a695f2c39e6da7f"), "nombre" : "Diego", "apellido" : "Pérez", "actividades" : null }
```
Aquí solo regresa las actividades que sean nulas

También lo podemos expresar así: 

```sh
> db.monitores.find({ actividades: { $type: "null" } })
{ "_id" : ObjectId("5e458f618a695f2c39e6da7f"), "nombre" : "Diego", "apellido" : "Pérez", "actividades" : null }
> 
```

El comando anterior es más presiso al que ya habiamos visto:

```sh
```sh
> db.monitores.find({ actividades: null })
{ "_id" : ObjectId("5e458f398a695f2c39e6da7e"), "nombre" : "Juan", "apellido" : "González" }
{ "_id" : ObjectId("5e458f618a695f2c39e6da7f"), "nombre" : "Diego", "apellido" : "Pérez", "actividades" : null }
> 
```
Donde también retorna los que no tienen el campo `actividades`.

Insertemos un documento con una fecha:

```sh
> db.monitores.insert({
... nombre: "Francisco",
... fechaAlta: ISODate("2020-01-20")
... })
WriteResult({ "nInserted" : 1 })

> db.monitores.insert({ nombre: "Francisco II", fechaAlta: "2020-01-20" })
WriteResult({ "nInserted" : 1 })
> 
```

Listemos la colección monitores:

```sh
> db.monitores.find({})
{ "_id" : ObjectId("5e4433b8f6a56e42b753ca3d"), "nombre" : "Pedro", "actividades" : [ { "clase" : "aerobics", "turno" : "mañana", "homologado" : false }, { "clase" : "pesas", "turno" : "tarde", "homologado" : false }, { "clase" : "zumba", "turno" : "mañana", "homologado" : true } ] }
{ "_id" : ObjectId("5e44343ef6a56e42b753ca3e"), "nombre" : "Susana", "actividades" : [ { "clase" : "aerobics", "turno" : "tarde", "homologado" : true }, { "clase" : "step", "turno" : "tarde", "homologado" : false }, { "clase" : "ciclismo", "turno" : "tarde", "homologado" : true } ] }
{ "_id" : ObjectId("5e443498f6a56e42b753ca3f"), "nombre" : "Alexa", "actividades" : [ { "clase" : "aerobics", "turno" : "mañana", "homologado" : false }, { "clase" : "pesas", "turno" : "mañana", "homologado" : true }, { "clase" : "zumba", "turno" : "mañana", "homologado" : true } ] }
{ "_id" : ObjectId("5e458f398a695f2c39e6da7e"), "nombre" : "Juan", "apellido" : "González" }
{ "_id" : ObjectId("5e458f618a695f2c39e6da7f"), "nombre" : "Diego", "apellido" : "Pérez", "actividades" : null }
{ "_id" : ObjectId("5e459a598a695f2c39e6da80"), "nombre" : "Francisco", "fechaAlta" : ISODate("2020-01-20T00:00:00Z") }
{ "_id" : ObjectId("5e459ac28a695f2c39e6da81"), "nombre" : "Francisco II", "fechaAlta" : "2020-01-20" }
> 
```

Busquemos los documentos de tipo fecha en el campo `fechaAlta`

```sh
> db.monitores.find({ fechaAlta: { $type: 9 } })
{ "_id" : ObjectId("5e459a598a695f2c39e6da80"), "nombre" : "Francisco", "fechaAlta" : ISODate("2020-01-20T00:00:00Z") }
```
Solo nos muetra un registro por que el otro que tiene fechaAlta la tiene de tipo string.

### Comprobación de Existencia `$exists`

Sintaxis.

```sh
{ <campo>: { $exists: <boolean> } }
```

Listar documentos que no tengan el campo `actividades`:

```sh
> db.monitores.find({ actividades: {$exists: false} })
{ "_id" : ObjectId("5e458f398a695f2c39e6da7e"), "nombre" : "Juan", "apellido" : "González" }
{ "_id" : ObjectId("5e459a598a695f2c39e6da80"), "nombre" : "Francisco", "fechaAlta" : ISODate("2020-01-20T00:00:00Z") }
{ "_id" : ObjectId("5e459ac28a695f2c39e6da81"), "nombre" : "Francisco II", "fechaAlta" : "2020-01-20" }
>
```
Listar documentos que tengan el campo `actividades`:

```sh
> db.monitores.find({ actividades: {$exists: true} })
{ "_id" : ObjectId("5e4433b8f6a56e42b753ca3d"), "nombre" : "Pedro", "actividades" : [ { "clase" : "aerobics", "turno" : "mañana", "homologado" : false }, { "clase" : "pesas", "turno" : "tarde", "homologado" : false }, { "clase" : "zumba", "turno" : "mañana", "homologado" : true } ] }
{ "_id" : ObjectId("5e44343ef6a56e42b753ca3e"), "nombre" : "Susana", "actividades" : [ { "clase" : "aerobics", "turno" : "tarde", "homologado" : true }, { "clase" : "step", "turno" : "tarde", "homologado" : false }, { "clase" : "ciclismo", "turno" : "tarde", "homologado" : true } ] }
{ "_id" : ObjectId("5e443498f6a56e42b753ca3f"), "nombre" : "Alexa", "actividades" : [ { "clase" : "aerobics", "turno" : "mañana", "homologado" : false }, { "clase" : "pesas", "turno" : "mañana", "homologado" : true }, { "clase" : "zumba", "turno" : "mañana", "homologado" : true } ] }
{ "_id" : ObjectId("5e458f618a695f2c39e6da7f"), "nombre" : "Diego", "apellido" : "Pérez", "actividades" : null }
> 
```

## Método `findOne()`

[db.collection.findOne()](https://docs.mongodb.com/manual/reference/method/db.collection.findOne/index.html)

Devuelve el primer documento que encuentra, ya lo regresa con formato `pretty()`.

Sintaxis.

```sh
db.<coleccion>.findOne(
   <documento-query>,
   <documento-projection>
)
```

Veamos un ejemplo:

```sh
> db.monitores.findOne()
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
> 
```

**Consideraciones**: 
* Si uso Mongoose(ODM) Si lo hago con `find` me regresa un solo registro en un array, con `findOne` me lo regresa como documento, por lo que es más directo ya que con `find` tengo que recuperar el elemnto [0] del array para  tener el documento.
* HTTP tiene limite de tamaño del Body, por lo que hay que limitar el número de registros que se retornan.

## Métodos Adicionales de Lectura

Etapa `$match` de agregación.

## Operadores de Comparación

[Comparison Query Operators](https://docs.mongodb.com/manual/reference/operator/query-comparison/)

Nombre | Descripción
-------|------------
$eq | Coincide con valores que son iguales a un valor especificado.
$gt | Coincide con valores que son mayores que un valor especificado.
$gte | Coincide con valores mayores o iguales que un valor especificado.
$in | Coincide con cualquiera de los valores especificados en una matriz.
$lt | Coincide con valores inferiores a un valor especificado.
$lte | Coincide con valores inferiores o iguales a un valor especificado.
$ne | Coincide con todos los valores que no son iguales a un valor especificado.
$nin | No coincide con ninguno de los valores especificados en una matriz.

### Operador de Igualdad [`$eq`](https://docs.mongodb.com/manual/reference/operator/query/eq/) 

Sintaxis.

```sh
{ <campo>: { $eq: <valor> } }
```

Documentos que tienen el nombre = "Susana":

```sh
> db.monitores.find({ nombre: { $eq: "Susana" } })
{ "_id" : ObjectId("5e44343ef6a56e42b753ca3e"), "nombre" : "Susana", "actividades" : [ { "clase" : "aerobics", "turno" : "tarde", "homologado" : true }, { "clase" : "step", "turno" : "tarde", "homologado" : false }, { "clase" : "ciclismo", "turno" : "tarde", "homologado" : true } ] }
> 
> db.monitores.find({ nombre: "Susana" })
{ "_id" : ObjectId("5e44343ef6a56e42b753ca3e"), "nombre" : "Susana", "actividades" : [ { "clase" : "aerobics", "turno" : "tarde", "homologado" : true }, { "clase" : "step", "turno" : "tarde", "homologado" : false }, { "clase" : "ciclismo", "turno" : "tarde", "homologado" : true } ] }
> 
```

### Operador de Negación [`$ne`](https://docs.mongodb.com/manual/reference/operator/query/ne/) 

Sintaxis.

```sh
{ <campo>: { $ne: <valor> } }
```
Documentos que tienen el nombre <> "Susana":

```sh
> db.monitores.find({ nombre: { $ne: "Susana" } }, { _id: 0, nombre: 1})
{ "nombre" : "Pedro" }
{ "nombre" : "Alexa" }
{ "nombre" : "Juan" }
{ "nombre" : "Diego" }
{ "nombre" : "Francisco" }
{ "nombre" : "Francisco II" }
> 
```
Incluiria también los que no tengan `nombre` que no es el caso.

### Operador [`$in`](https://docs.mongodb.com/manual/reference/operator/query/in/) 

Sintaxis.

```sh
{ <campo>: { $in: [ <valor>, <valor>, ... ] } }
```

* Devuelve los documentos que incluyan alguna de los valores contenidos en el array.

Listemos los documentos con actividades.

```sh
> db.clientes.find( {actividades: {$type: 4 }}, {_id:0, })
{ "nombre" : "Pedro", "apellido" : "López", "actividades" : [ "yoga", "zumba" ] }
{ "nombre" : "Paula", "apellido" : "García", "actividades" : [ "esgrima", "zumba", "padel" ] }
{ "nombre" : "Susana", "apellido" : "González", "actividades" : [ "esgrima", "natación", "step" ] }
> 
```

Documentos que en actividades tengan `"step"` o `"yoga"`.

```sh
> db.clientes.find({ actividades: { $in: ["step", "yoga"] } })
{ "_id" : ObjectId("5e42ed4390d86b85f5fda8f4"), "nombre" : "Pedro", "apellido" : "López", "actividades" : [ "yoga", "zumba" ] }
{ "_id" : ObjectId("5e42ef3b90d86b85f5fda8f7"), "nombre" : "Susana", "apellido" : "González", "actividades" : [ "esgrima", "natación", "step" ] }
> 
```

### Operador [`$all`](https://docs.mongodb.com/manual/reference/operator/query/all/index.html) 

Sintaxis.

```sh
{ <campo>: { $all: [ <valor>, <valor>, ... ] } }
```

* Devuelve los documentos que incluyan todos los valores contenidos en el array.

Documentos que en actividades tengan `"step"` y `"yoga"`.

```sh
> db.clientes.find({ actividades: { $all: ["step", "yoga"] } })
> 
```
No recuperamos ningún documento.

Documentos que en actividades tengan `"esgrima"` y `"padel"`.

```sh
> db.clientes.find({ actividades: { $all: ["esgrima", "padel"] } })
{ "_id" : ObjectId("5e42ef1590d86b85f5fda8f6"), "nombre" : "Paula", "apellido" : "García", "actividades" : [ "esgrima", "zumba", "padel" ] }
>
```
Debe tener ambos elementos, pero el array puede tener más elementos.


### Operador [`$nin`](https://docs.mongodb.com/manual/reference/operator/query/nin/index.html) 

Sintaxis.

```sh
{ <campo>: { $nin: [ <valor>, <valor>, ... ] } }
```

* Devuelve los documentos que no tengan ninguno de los valores en el array y también los que no tengan ese campo.

Documentos que no tengan las actividades `"esgrima", "padel", "step"`:

```sh
> db.clientes.find({ actividades: { $nin: ["esgrima", "padel", "step"]} }, {_id: 0})
{ "nombre" : "Juan", "apellido" : "Pérez" }
{ "nombre" : "María", "apellido" : "López" }
{ "nombre" : "Laura", "apellido" : "López" }
{ "nombre" : "Luis", "apellido" : "Pérez" }
{ "nombre" : "Pablo", "apellido" : "Gómez" }
{ "nombre" : "Sara", "apellido" : "García" }
{ "nombre" : "Carlos", "apellido" : "López" }
{ "nombre" : "José" }
{ "nombre" : "Lucia" }
{ "nombre" : "Luisa" }
{ "nombre" : "Carlos" }
{ "nombre" : "Javier", "createAt" : ISODate("2020-02-10T21:29:44.672Z") }
{ "nombre" : "Salma" }
{ "nombre" : "Angelica" }
{ "nombre" : "Veronica" }
{ "nombre" : "José", "apellido" : "López" }
{ "nombre" : "Pedro", "apellido" : "Paramo" }
{ "nombre" : "Julio", "apellido" : "Cortez" }
{ "nombre" : "Luis", "apellido" : "González", "direccion" : { "calle" : "Alcalá, 90", "localidad" : "Madrid" } }
{ "nombre" : "Mariano", "apellido" : "Mejía", "direccion" : { "calle" : "Gran vía, 100", "localidad" : "Madrid" } }
Type "it" for more
> it
{ "nombre" : "Enrique", "apellido" : "Flores", "direccion" : { "calle" : "Plaza España, 50", "localidad" : "Sevilla" } }
{ "nombre" : "Pedro", "apellido" : "López", "actividades" : [ "yoga", "zumba" ] }
{ "nombre" : "Rebeca", "puntuaciones" : [ 100, 34, 67 ] }
{ "nombre" : "Rocio", "puntuaciones" : [ 95, 88, 21 ] }
{ "nombre" : "Rosa", "puntuaciones" : [ 15, 49, 44 ] }
{ "nombre" : "Rita", "puntuaciones" : [ 78, 92, 52 ] }
{ "nombre" : "Roberto", "apellido" : "García", "direcciones" : [ { "calle" : "Alcalá, 40", "cp" : "02800", "localidad" : "Madrid" }, { "calle" : "Zamora, 13", "cp" : "34005", "localidad" : "Vigo" } ] }
{ "nombre" : "Carla", "apellido" : "López", "direcciones" : [ { "calle" : "Gran vía, 121", "cp" : "28025", "localidad" : "Madrid" }, { "calle" : "Bogota, 27", "cp" : "25052", "localidad" : "Valencia" } ] }
{ "nombre" : "Roberto", "apellido" : "Pérez", "direcciones" : [ { "calle" : "Gran vía, 23", "cp" : "28025", "localidad" : "Madrid" }, { "calle" : "Toledo, 13", "cp" : "24122", "localidad" : "Madrid" } ] }
{  }
> 
```

Documentos que no tengan las actividades `"esgrima", "padel", "step"` y que exista el campo:


```sh
> db.clientes.find({ actividades: { $nin: ["esgrima", "padel", "step"]}, actividades: { $exists: true } }, {_id: 0})
{ "nombre" : "Pedro", "apellido" : "López", "actividades" : [ "yoga", "zumba" ] }
{ "nombre" : "Paula", "apellido" : "García", "actividades" : [ "esgrima", "zumba", "padel" ] }
{ "nombre" : "Susana", "apellido" : "González", "actividades" : [ "esgrima", "natación", "step" ] }
> 
```
**Aquí no hay Projección (solo para quitar el `_id`) es todo consulta**

## Operadores Lógicos

[Logical Query Operators](https://docs.mongodb.com/manual/reference/operator/query-logical/)

Nombre | Descripción
-------|------------
$and | Une cláusulas de consulta con un AND lógico que devuelve todos los documentos que coinciden con las condiciones de ambas cláusulas.
$not | Invierte el efecto de una expresión de consulta y devuelve documentos que no coinciden con la expresión de consulta.
$nor | Joins con un NOR lógico devuelve todos los documentos que no coinciden con ambas cláusulas.
$or | Une cláusulas de consulta con un OR lógico devuelve todos los documentos que coinciden con las condiciones de cualquiera de las cláusulas

### Operador `$and`

Sintaxis.

```sh
{ $and: [ { <expression1> }, { <expression2> } , ... , { <expressionN> } ] }
```

```sh

```

### Operador `$or`

Sintaxis.

```sh
{ $or: [ { <expression1> }, { <expression2> } , ... , { <expressionN> } ] }
```

```sh
```

### Operador `$not`

Sintaxis.

```sh
{ $not: { <expression> }
```


```sh
```

```sh
```

### Operador `$nor`

Sintaxis.

```sh
{ $nor: [ { <expression1> }, { <expression2> }, ...  { <expressionN> } ] }
```


```sh
```

```sh
```
