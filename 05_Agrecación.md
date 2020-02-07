# 05 AGREGACION (06/02/2020)

:skull: Sólo una pregunta sobre todo el tema.


```sh
db.<coleccion>.aggregate(
  [
    { etapa },
    { <utilizará operadores> }
    ...
  ],
  { opciones }
)
```
`{allowDiskUse: true}` Es una OPCIÓN, Si la etapa de agrupamiento supera el uso de 100MB de memoria RAM

Operadores:

* `$project` // idem proyección `find()`
* `sort` // idem método `sort()`
* `group`
```
  $group:{
     _id: <expression>,
     <campo1>: {operador: expr},
     ...
  }
```
* `$unwind` (para arrays)


  
* ``
* ``
* ``
* ``
* ``



```sh
> use biblioteca
switched to db biblioteca
> show collections
foo
libros
> db.libros.aggregate([
... { $project: {titulo: 1, autor: 1, _id:0} }
... ])
{ "titulo" : "Cien Años de Soledad", "autor" : "Gabriel García Márquez" }
{ "titulo" : "El Otoño del Patriarca", "autor" : "Gabriel García Márquez" }
{ "titulo" : "La Ciudad y los Perros", "autor" : "Mario Vargas Llosa" }
{ "titulo" : "El Quijote", "autor" : { "nombre" : "Miguel", "apellidos" : "De Cervantes Saavedra", "pais" : "Italia" } }
{ "autor" : "Michael Ende", "titulo" : "La Historia Interminable" }
{ "autor" : "Michael Ende", "titulo" : "La Historia Interminable" }
{ "autor" : "Adolfito", "titulo" : "La Historia Interminable" }
{ "autor" : "William Shakespeare", "titulo" : "Otelo" }
{ "titulo" : "Otelo 2", "autor" : "Juan" }
{ "titulo" : "El Caso Fitgerald", "autor" : "John Grisam" }
{ "titulo" : "Cien Años de Soledad", "autor" : "Gabriel García Márquez" }
{ "titulo" : "The Firm", "autor" : "John Grisham" }
{ "titulo" : "El Sr. de los Anillos", "autor" : "J.R.R. Tolkin" }
{ "titulo" : "El Sr. de los Anillos", "autor" : "J.R.R. Tolkin" }
{ "titulo" : "El Sr. de los Anillos", "autor" : "J.R.R. Tolkin" }
{ "titulo" : "El Sr. de los Anillos", "autor" : "J.R.R. Tolkin" }
{ "titulo" : "El Coronel no tiene quién le escriba", "autor" : { "nombre" : "Gabriel García Márquez", "pais" : "Colombia" } }
{ "titulo" : "París era una Fiesta", "autor" : "Ernest Hemingway" }
{ "titulo" : "París, La Guía Completa", "autor" : "vv.aa." }
{ "titulo" : "La Era de Oro del Cine Americano", "autor" : "vv.aa." }
Type "it" for more
>  
```

Este hace lo mismo que un `.find({},  {titulo: 1, autor: 1, _id:0} )`

Puedo cambiar el nombre de los campos:

```sh
> db.libros.aggregate([
... { $project: { _id: 0, title: "$titulo"} }
... ])
{ "title" : "Cien Años de Soledad" }
{ "title" : "El Otoño del Patriarca" }
{ "title" : "La Ciudad y los Perros" }
{ "title" : "El Quijote" }
{ "title" : "La Historia Interminable" }
{ "title" : "La Historia Interminable" }
{ "title" : "La Historia Interminable" }
{ "title" : "Otelo" }
{ "title" : "Otelo 2" }
{ "title" : "El Caso Fitgerald" }
{ "title" : "Cien Años de Soledad" }
{ "title" : "The Firm" }
{ "title" : "El Sr. de los Anillos" }
{ "title" : "El Sr. de los Anillos" }
{ "title" : "El Sr. de los Anillos" }
{ "title" : "El Sr. de los Anillos" }
{ "title" : "El Coronel no tiene quién le escriba" }
{ "title" : "París era una Fiesta" }
{ "title" : "París, La Guía Completa" }
{ "title" : "La Era de Oro del Cine Americano" }
Type "it" for more
>
```

```sh
> use gimnasio2
switched to db gimnasio2
> db.clientes.insert({ nombre: "Pedro", alta: new Date(2019, 11, 15), actividades: ["padel", "tenis","esgrima"]})
WriteResult({ "nInserted" : 1 })
> db.clientes.insert({ nombre: "Laura", alta: new Date(2019, 10, 9), actividades: ["aquagym", "tenis","step"]})
WriteResult({ "nInserted" : 1 })
> db.clientes.insert({ nombre: "Carlos", alta: new Date(2019, 9, 17), actividades: ["aquagym", "padel","cardio"]})
WriteResult({ "nInserted" : 1 })
> db.clientes.insert({ nombre: "Sara", alta: new Date(2019, 11, 20), actividades: ["pesas", "cardio", "step"]})
WriteResult({ "nInserted" : 1 })
>
```

```sh
> db.clientes.aggregate([
... {$project: {nombre: 1, _id: 0 } }
... ])
{ "nombre" : "Pedro" }
{ "nombre" : "Laura" }
{ "nombre" : "Carlos" }
{ "nombre" : "Sara" }
>      


> db.clientes.aggregate([
... { $project: { _id:0, cliente: { $toUpper: "$nombre"} } },
... { $sort: { cliente: 1 } }
... ])
{ "cliente" : "CARLOS" }
{ "cliente" : "LAURA" }
{ "cliente" : "PEDRO" }
{ "cliente" : "SARA" }

> db.clientes.aggregate([
... {$project: { cliente: {$toUpper: "$nombre"} , _id: 0 } },
... {$sort: { cliente: 1  } }
... ])
{ "cliente" : "CARLOS" }
{ "cliente" : "LAURA" }
{ "cliente" : "PEDRO" }
{ "cliente" : "SARA" }
```

Le he cambiado el nombre al campo `nombre` por `cliente` y posteriormente ordenamos a `cliente`, y no pintamos el campo `_id`, el orden en la proyección no importa.


 El campo `_id` es obligatorio.

**Cada etapa va haciendo referencia a la etapa anterior**

```sh
> db.clientes.aggregate([
... {$project: {mesAlta: {$month: "$alta" }, cliente: "$nombre", _id: 0}},
... {$sort: {mesAlta: -1, cliente: 1}}
... ])
{ "mesAlta" : 12, "cliente" : "Pedro" }
{ "mesAlta" : 12, "cliente" : "Sara" }
{ "mesAlta" : 11, "cliente" : "Laura" }
{ "mesAlta" : 10, "cliente" : "Carlos" }
```

Ordena descendentemente por mes y acendentemente por cliente. `$month` como usa la ISO regresa del 1-12 y lo asigna al campo `mesAlta`.

```sh
> db.clientes.aggregate([
... {$project: {mesAlta: {$month: "$alta"}}},
... {$group: {_id: "$mesAlta", numeroAltas: {$sum: 1}}}
... ])
{ "_id" : 12, "numeroAltas" : 2 }
{ "_id" : 11, "numeroAltas" : 1 }
{ "_id" : 10, "numeroAltas" : 1 }
> 
```
Agrupa por mesAlta, obliga a poner `_id` y los acumula sumandolos en el campo `numeroAltas`, el `1` del `$sum` indica que sume por el primer campo.

¿Cuando pongo o no el signo de $? A mi parecer aquí tiene lógica que lleve el `_id` por que lo convierte a `_id`

```sh
> db.clientes.aggregate([ {$project: {mesAlta: {$month: "$alta"}}}, {$group: {_id: "$mesAlta", numeroAltas: {$sum: 1}}} , {$project:{ mes: "$_id", numeroAltas: 1 }} ])
{ "_id" : 12, "numeroAltas" : 2, "mes" : 12 }
{ "_id" : 11, "numeroAltas" : 1, "mes" : 11 }
{ "_id" : 10, "numeroAltas" : 1, "mes" : 10 }

> db.clientes.aggregate([ {$project: {mesAlta: {$month: "$alta"}}}, {$group: {_id: "$mesAlta", numeroAltas: {$sum: 1}}} , {$project:{ mes: "$_id", numeroAltas: 1, _id: 0 }} ])
{ "numeroAltas" : 2, "mes" : 12 }
{ "numeroAltas" : 1, "mes" : 11 }
{ "numeroAltas" : 1, "mes" : 10 }

> db.clientes.aggregate([ {$project: {mesAlta: {$month: "$alta"}}}, {$group: {_id: "$mesAlta", numeroAltas: {$sum: 1}}} , {$project:{ mes: "$_id", numeroAltas: 1, _id: 0 }}, {$sort:{mes: -1}} ])
{ "numeroAltas" : 2, "mes" : 12 }
{ "numeroAltas" : 1, "mes" : 11 }
{ "numeroAltas" : 1, "mes" : 10 }
```

```sh
> use maraton
switched to db maraton
> show collections
foo
participantes
> db.participantes.aggregate([
... {$group: {_id: "$nombre", cantidad: {$sum: 1} }},
... { $project: {nombre: "$_id", cantidad: 1, _id: 0 } }
... ])
{ "cantidad" : 250498, "nombre" : "Carlos" }
{ "cantidad" : 249888, "nombre" : "Juan" }
{ "cantidad" : 250110, "nombre" : "María" }
{ "cantidad" : 249504, "nombre" : "Lucía" }
>
```

```sh
> db.participantes.aggregate([ {$group: {_id: "$edad", cantidad: {$sum: 1} }},  { $project: {edad: "$_id", cantidad: 1, _id: 0 } } ])
{ "cantidad" : 9936, "edad" : 54 }
{ "cantidad" : 9988, "edad" : 68 }
{ "cantidad" : 10038, "edad" : 94 }
{ "cantidad" : 10190, "edad" : 63 }
{ "cantidad" : 9903, "edad" : 0 }
{ "cantidad" : 9968, "edad" : 51 }
{ "cantidad" : 9984, "edad" : 74 }
{ "cantidad" : 10073, "edad" : 26 }
{ "cantidad" : 9792, "edad" : 30 }
{ "cantidad" : 10089, "edad" : 45 }
{ "cantidad" : 9895, "edad" : 46 }
{ "cantidad" : 9756, "edad" : 14 }
{ "cantidad" : 10055, "edad" : 29 }
{ "cantidad" : 10004, "edad" : 31 }
{ "cantidad" : 9916, "edad" : 64 }
{ "cantidad" : 9886, "edad" : 49 }
{ "cantidad" : 10115, "edad" : 69 }
{ "cantidad" : 10035, "edad" : 25 }
{ "cantidad" : 10027, "edad" : 62 }
{ "cantidad" : 10108, "edad" : 22 }
Type "it" for more
>                                           
```

```sh
> use shop2
switched to db shop2
> db.pedidos.insert({item: "v101", cantidad:12, precio:20, fecha: ISODate("2020-01-19") })
WriteResult({ "nInserted" : 1 })
> db.pedidos.insert({item: "v101", cantidad:6, precio:20, fecha: ISODate("2020-01-11") })
WriteResult({ "nInserted" : 1 })
> db.pedidos.insert({item: "v101", cantidad:4, precio:20, fecha: ISODate("2020-01-21") })
WriteResult({ "nInserted" : 1 })
> db.pedidos.insert({item: "v102", cantidad:7, precio:10.3, fecha: ISODate("2020-01-21") })
WriteResult({ "nInserted" : 1 })
> db.pedidos.insert({item: "v102", cantidad:5, precio:10.9, fecha: ISODate("2020-01-22") })
WriteResult({ "nInserted" : 1 })
```

```sh
> db.pedidos.aggregate([
... {$group: {_id: {$dayOfWeek: "$fecha"}, total: {$sum:{$multiply:["$cantidad", "$precio"]}}}},
... {$sort: {total: -1}}
... ])
{ "_id" : 1, "total" : 240 }
{ "_id" : 3, "total" : 152.10000000000002 }
{ "_id" : 7, "total" : 120 }
{ "_id" : 4, "total" : 54.5 }
```

Por cada día de la semana nos indica las ventas.

```sh
> db.pedidos.aggregate([
... {$group: {_id: "$item", cantPromedio: {$avg: "$cantidad"}}},
... {$project: {item: "$_id", cantPromedio:1, _id: 0}}
... ])
{ "cantPromedio" : 6, "item" : "v102" }
{ "cantPromedio" : 7.333333333333333, "item" : "v101" }
```

Cantidad promedio por `item`.

```sh
> db.libros.aggregate([
... {$group: {_id: "$autor", libros: {$push: "$titulo"} } },
... {$project: {autor: "$_id", libros: 1, _id:0} },
... {$sort: {autor: 1}}
... ])
{ "libros" : [ "Futuro de la Ingeniería", "Grandes Ingenieros del siglo XX", "Grandes Ingenieros del siglo XX", "El Coronel no tiene quien le escriba", "El Amor en los tiempos del cólera" ], "autor" : null }
{ "libros" : [ "La Historia Interminable" ], "autor" : "Adolfito" }
{ "libros" : [ "París era una Fiesta" ], "autor" : "Ernest Hemingway" }
{ "libros" : [ "Cien Años de Soledad", "El Otoño del Patriarca", "Cien Años de Soledad" ], "autor" : "Gabriel García Márquez" }
{ "libros" : [ "El Sr. de los Anillos", "El Sr. de los Anillos", "El Sr. de los Anillos", "El Sr. de los Anillos" ], "autor" : "J.R.R. Tolkin" }
{ "libros" : [ "París siempre será París" ], "autor" : "John Doe" }
{ "libros" : [ "El Caso Fitgerald" ], "autor" : "John Grisam" }
{ "libros" : [ "The Firm" ], "autor" : "John Grisham" }
{ "libros" : [ "Otelo 2" ], "autor" : "Juan" }
{ "libros" : [ "La Ciudad y los Perros" ], "autor" : "Mario Vargas Llosa" }
{ "libros" : [ "La Historia Interminable", "La Historia Interminable" ], "autor" : "Michael Ende" }
{ "libros" : [ "Otelo" ], "autor" : "William Shakespeare" }
{ "libros" : [ "París, La Guía Completa", "La Era de Oro del Cine Americano", "The Very Best of Beatles", "Agile Consultants", "Consulting for Global Marktes", "París" ], "autor" : "vv.aa." }
{ "libros" : [ "El Coronel no tiene quién le escriba" ], "autor" : { "nombre" : "Gabriel García Márquez", "pais" : "Colombia" } }
{ "libros" : [ "El Quijote" ], "autor" : { "nombre" : "Miguel", "apellidos" : "De Cervantes Saavedra", "pais" : "Italia" } }
```

Crea un array de libros por cada Autor


```sh
> use shop2
switched to db shop2
> show collections
pedidos
> db.productos.insert({nombre: "Camiseta Nike", tallas: ["xs", "s", "m", "l", "xl"]})
WriteResult({ "nInserted" : 1 })
```



```sh
> db.productos.aggregate([
... {$unwind: "$tallas"}
... ])
{ "_id" : ObjectId("5e3d4910d8b0fd4ac47898f5"), "nombre" : "Camiseta Nike", "tallas" : "xs" }
{ "_id" : ObjectId("5e3d4910d8b0fd4ac47898f5"), "nombre" : "Camiseta Nike", "tallas" : "s" }
{ "_id" : ObjectId("5e3d4910d8b0fd4ac47898f5"), "nombre" : "Camiseta Nike", "tallas" : "m" }
{ "_id" : ObjectId("5e3d4910d8b0fd4ac47898f5"), "nombre" : "Camiseta Nike", "tallas" : "l" }
{ "_id" : ObjectId("5e3d4910d8b0fd4ac47898f5"), "nombre" : "Camiseta Nike", "tallas" : "xl" }
>    
```
De un array crea un documento por cada elemento del array

```sh
> db.productos.insert({nombre: "Camiseta Puma", tallas: ["m", "l"]})
WriteResult({ "nInserted" : 1 })
> db.productos.find({}, {_id: 0})
{ "nombre" : "Camiseta Nike", "tallas" : [ "xs", "s", "m", "l", "xl" ] }
{ "nombre" : "Camiseta Puma", "tallas" : [ "m", "l" ] }
```
:skull: Pregunta clasica ¿Cuantos documentos regresa si uso el `$unwind`

```sh
> db.productos.aggregate([ {$unwind: "$tallas"}, {$count: "documentos"} ])
{ "documentos" : 7 }

```

```sh
> db.productos.insert({nombre: "Camiseta Adidas", tallas: null})
WriteResult({ "nInserted" : 1 })
> db.productos.insert({nombre: "Camiseta Zara"})
WriteResult({ "nInserted" : 1 })

> db.productos.aggregate([ {$unwind: "$tallas"} ])
{ "_id" : ObjectId("5e3d4910d8b0fd4ac47898f5"), "nombre" : "Camiseta Nike", "tallas" : "xs" }
{ "_id" : ObjectId("5e3d4910d8b0fd4ac47898f5"), "nombre" : "Camiseta Nike", "tallas" : "s" }
{ "_id" : ObjectId("5e3d4910d8b0fd4ac47898f5"), "nombre" : "Camiseta Nike", "tallas" : "m" }
{ "_id" : ObjectId("5e3d4910d8b0fd4ac47898f5"), "nombre" : "Camiseta Nike", "tallas" : "l" }
{ "_id" : ObjectId("5e3d4910d8b0fd4ac47898f5"), "nombre" : "Camiseta Nike", "tallas" : "xl" }
{ "_id" : ObjectId("5e3d49cad8b0fd4ac47898f6"), "nombre" : "Camiseta Puma", "tallas" : "m" }
{ "_id" : ObjectId("5e3d49cad8b0fd4ac47898f6"), "nombre" : "Camiseta Puma", "tallas" : "l" }


> db.productos.aggregate([ {$unwind: {path: "$tallas", preserveNullAndEmptyArrays: true}} ])
{ "_id" : ObjectId("5e3d4910d8b0fd4ac47898f5"), "nombre" : "Camiseta Nike", "tallas" : "xs" }
{ "_id" : ObjectId("5e3d4910d8b0fd4ac47898f5"), "nombre" : "Camiseta Nike", "tallas" : "s" }
{ "_id" : ObjectId("5e3d4910d8b0fd4ac47898f5"), "nombre" : "Camiseta Nike", "tallas" : "m" }
{ "_id" : ObjectId("5e3d4910d8b0fd4ac47898f5"), "nombre" : "Camiseta Nike", "tallas" : "l" }
{ "_id" : ObjectId("5e3d4910d8b0fd4ac47898f5"), "nombre" : "Camiseta Nike", "tallas" : "xl" }
{ "_id" : ObjectId("5e3d49cad8b0fd4ac47898f6"), "nombre" : "Camiseta Puma", "tallas" : "m" }
{ "_id" : ObjectId("5e3d49cad8b0fd4ac47898f6"), "nombre" : "Camiseta Puma", "tallas" : "l" }
{ "_id" : ObjectId("5e3d4b7fd8b0fd4ac47898f8"), "nombre" : "Camiseta Adidas", "tallas" : null }
{ "_id" : ObjectId("5e3d4ba1d8b0fd4ac47898f9"), "nombre" : "Camiseta Zara" }
```
Ya presenta los que tienen array `null` o sin array.

```sh
> db.clientes.aggregate([ {$unwind: "$actividades"}, {$group: {_id: "$actividades", total: {$sum: 1} }}, {$project: {actividad: "$_id", total: 1, _id:0} }, {$sort: {total: -1}} ])
{ "total" : 2, "actividad" : "cardio" }
{ "total" : 2, "actividad" : "step" }
{ "total" : 2, "actividad" : "aquagym" }
{ "total" : 2, "actividad" : "tenis" }
{ "total" : 2, "actividad" : "padel" }
{ "total" : 1, "actividad" : "pesas" }
{ "total" : 1, "actividad" : "esgrima" }
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
