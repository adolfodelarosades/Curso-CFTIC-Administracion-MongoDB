# 05 AGREGACION (06/02/2020)

:skull: Sólo una pregunta sobre todo el tema.

**Posibles preguntas de agragación en la certificación.**
* Etapas sencillas `$match`, `$project`, `$sort`...
* Que nos pidan cuantos docs devuelven un `$unwind`
* Que nos pregunten acerca de `allowDiscUse: true` (para evitar error si una etapa supera 100MG de set de datos)


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
```sh
  $group:{
     _id: <expression>,
     <campo1>: {operador: expr},
     ...
  }
  
  $group:{_id: {
     <campo>: "$campo",
     <campo>: "$campo",...
     }, ...}}
  
  
```
* `$unwind` (para arrays) :skull:
* `$match`
* `$addFields` Añade campos
* `$kip`
   {$kip: <entero>}
* `$limit`
   {$limit: <entero>}
* `$lookup` // Soluciona algún tipo de Join
  ```sh
   { $lookup: {
      from: <coleccion>, // misma base de datos
      localField: <campo>,
      foreignField: <campo>,
      as: <nuevo-campo>
     }
  }
 ```
* `$merge` // Volcar la agregación en una colección (si no existe la crea)
  ```
   { $merge: "<coleccion>" | 
              {db: <base-datos>, coll: <coleccion> }
   }
 ```
  
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
Agrupa por los elementos de un array y me dice los totales.

```sh
> use maraton
switched to db maraton

> db.participantes.find().count()
1000000

> db.participantes.aggregate([
... {$match: {edad: {$gte: 50}, edad: {$lt: 60}}},
... {$group: {_id: "$nombre", total: {$sum: 1}} },
... {$sort: {total: -1}}
... ])
{ "_id" : "Juan", "total" : 150497 }
{ "_id" : "Carlos", "total" : 150184 }
{ "_id" : "María", "total" : 149852 }
{ "_id" : "Lucía", "total" : 149357 }
```

```sh
> use shop3
switched to db shop3
> db.opiniones.insert({producto: "v101", usuario: "123", opinion:"buen servicio pero producto en mal estado"})
WriteResult({ "nInserted" : 1 })
> db.opiniones.insert({producto: "v101", usuario: "1234", opinion:"muy satisfecho con la compra"})
WriteResult({ "nInserted" : 1 })
> db.opiniones.insert({producto: "v101", usuario: "129", opinion:"muy mal tuve que devolverlo"})
WriteResult({ "nInserted" : 1 })
> db.opiniones.insert({producto: "v101", usuario: "211", opinion:"perfecto en todos los sentidos"})
WriteResult({ "nInserted" : 1 })
> db.opiniones.insert({producto: "v101", usuario: "212", opinion:"mal, no volveré a comprar"})
WriteResult({ "nInserted" : 1 })
> db.opiniones.insert({producto: "v102", usuario: "129", opinion:"muy mal horrible"})
WriteResult({ "nInserted" : 1 })
> db.opiniones.insert({producto: "v102", usuario: "130", opinion:"perfecto"})
WriteResult({ "nInserted" : 1 })

> db.opiniones.createIndex({opinion: "text"})
{
        "createdCollectionAutomatically" : false,
        "numIndexesBefore" : 1,
        "numIndexesAfter" : 2,
        "ok" : 1
}
>    
```
Creo indice de texto para buscar palabras (malas o buenas)
```sh
> db.opiniones.find({}, {_id:0, producto: 1, opinion: 1})
{ "producto" : "v101", "opinion" : "buen servicio pero producto en mal estado" }
{ "producto" : "v101", "opinion" : "muy satisfecho con la compra" }
{ "producto" : "v101", "opinion" : "muy mal tuve que devolverlo" }
{ "producto" : "v101", "opinion" : "perfecto en todos los sentidos" }
{ "producto" : "v101", "opinion" : "mal, no volveré a comprar" }
{ "producto" : "v102", "opinion" : "muy mal horrible" }
{ "producto" : "v102", "opinion" : "perfecto" }

> db.opiniones.aggregate([ {$match: {$text: {$search: "mal"} } }, {$group: {_id: "$producto", opiNegativas: {$sum: 1} } } ])
{ "_id" : "v102", "opiNegativas" : 1 }
{ "_id" : "v101", "opiNegativas" : 3 }
```
Tengo tres opiniones con la palabra **mal** en un producto `v101` y una en `v102`.

```sh
> db.opiniones.aggregate([ {$match: {$text: {$search: "buen perfecto satisfecho"} } }, {$group: {_id: "$producto", opiPositivas: {$sum: 1} } } ])
{ "_id" : "v102", "opiPositivas" : 1 }
{ "_id" : "v101", "opiPositivas" : 3 }
```

```sh
> use maraton2
switched to db maraton2
> db.resultados.insert({nombre:"Juan", llegada: new Date("2020-01-30T16:31:40")})
WriteResult({ "nInserted" : 1 })
> db.resultados.insert({nombre:"Laura", llegada: new Date("2020-01-30T15:43:20")})
WriteResult({ "nInserted" : 1 })
> db.resultados.insert({nombre:"Sara", llegada: new Date("2020-01-30T17:13:22")})
WriteResult({ "nInserted" : 1 })
```


```sh
> db.resultados.aggregate([
... {$addFields: {tiempo: {$subtract: ["$llegada", new Date("2020-01-30T12:00:00")] } }}
... ])
{ "_id" : ObjectId("5e3d58cbd8b0fd4ac4789901"), "nombre" : "Juan", "llegada" : ISODate("2020-01-30T15:31:40Z"), "tiempo" : NumberLong(16300000) }
{ "_id" : ObjectId("5e3d58e3d8b0fd4ac4789902"), "nombre" : "Laura", "llegada" : ISODate("2020-01-30T14:43:20Z"), "tiempo" : NumberLong(13400000) }
{ "_id" : ObjectId("5e3d58ffd8b0fd4ac4789903"), "nombre" : "Sara", "llegada" : ISODate("2020-01-30T16:13:22Z"), "tiempo" : NumberLong(18802000) }
```

Añado una columna `tiempo` que esta dada en milisegundos.

```sh
db.resultados.aggregate([
    { $addFields: { tiempo: { $subtract: ["$llegada", new Date("2020-01-30T12:00:00")] } } },
    { $addFields:{
        segundos: {
            $mod: [{$divide: ["$tiempo", 1000]}, 60]
        }
      }
    },
    { $addFields:{
        minutos: {
            $floor: {
                $mod: [{$divide: ["$tiempo", 1000 * 60]}, 60]
            }
        }
      }
    },
    { $addFields:{
        horas: {
            $floor: {
                $mod: [{$divide: ["$tiempo", 1000 * 60 * 60]}, 24]
            }
        }
      }
    },
    {$project: {
        nombre: 1, horas : 1, minutos : 1, segundos : 1, _id: 0
      }
    },
    {$sort: {horas: 1, minutos: 1, segundos: 1} }
])
````

```sh
> db.resultados.aggregate([
...     { $addFields: { tiempo: { $subtract: ["$llegada", new Date("2020-01-30T12:00:00")] } } },
...     { $addFields:{
...         segundos: {
...             $mod: [{$divide: ["$tiempo", 1000]}, 60]
...         }
...       }
...     },
...     { $addFields:{
...         minutos: {
...             $floor: {
...                 $mod: [{$divide: ["$tiempo", 1000 * 60]}, 60]
...             }
...         }
...       }
...     },
...     { $addFields:{
...         horas: {
...             $floor: {
...                 $mod: [{$divide: ["$tiempo", 1000 * 60 * 60]}, 24]
...             }
...         }
...       }
...     },
...     {$project: {
...         nombre: 1, horas : 1, minutos : 1, segundos : 1, _id: 0
...       }
...     },
...     {$sort: {horas: 1, minutos: 1, segundos: 1} }
... ])
{ "nombre" : "Laura", "segundos" : 20, "minutos" : 43, "horas" : 3 }
{ "nombre" : "Juan", "segundos" : 40, "minutos" : 31, "horas" : 4 }
{ "nombre" : "Sara", "segundos" : 22, "minutos" : 13, "horas" : 5 }
>                              
```

Uso de `$limit`
```sh
> use maraton
switched to db maraton
> db.participantes.aggregate([ {$match: { edad: {$gte: 40}, edad: {$lt: 50} } } , {$group: {_id: "$nombre", total: {$sum: 1} } }, {$sort: {total: -1} }, {$limit: 2} ])
{ "_id" : "Juan", "total" : 125284 }
{ "_id" : "Carlos", "total" : 125113 }
>  
```


```sh
> use shop4
switched to db shop4
> db.inventario.insert({ _id: 1, sku: "AA01", descripcion: "Lorem ipsum...", stock: 120 })
WriteResult({ "nInserted" : 1 })
> db.inventario.insert({ _id: 2, sku: "BC03", descripcion: "dolor sit...", stock: 90 })
WriteResult({ "nInserted" : 1 })
> db.inventario.insert({ _id: 3, sku: "FG03", descripcion: "amet consectetur...", stock: 30 })
WriteResult({ "nInserted" : 1 })

> db.pedidos.insert({ _id: "p01", codigo: "BC03", cantidad: 10, precio: 40 })
WriteResult({ "nInserted" : 1 })
> db.pedidos.insert({ _id: "p02", codigo: "AA01", cantidad: 5, precio: 80 })
WriteResult({ "nInserted" : 1 })
```

**Uso de `$lookup`**

```sh
> db.pedidos.aggregate([
... { $lookup: {
... from: "inventario",
... localField: "codigo",
... foreignField: "sku",
... as: "detalleProducto"
... }}
... ])
{ "_id" : "p01", "codigo" : "BC03", "cantidad" : 10, "precio" : 40, "detalleProducto" : [ { "_id" : 2, "sku" : "BC03", "descripcion" : "dolor sit...", "stock" : 90 } ] }
{ "_id" : "p02", "codigo" : "AA01", "cantidad" : 5, "precio" : 80, "detalleProducto" : [ { "_id" : 1, "sku" : "AA01", "descripcion" : "Lorem ipsum...", "stock" : 120 } ] }
```

Volca la información en un array con nombre `detalleProducto`

```sh
> use maraton
switched to db maraton
> db.participantes.aggregate([
... {$group: {_id: {nombre: "$nombre", edad: "$edad"}, total: {$sum: 1} } }
... ])
{ "_id" : { "nombre" : "Lucía", "edad" : 29 }, "total" : 2447 }
{ "_id" : { "nombre" : "María", "edad" : 95 }, "total" : 2610 }
{ "_id" : { "nombre" : "Lucía", "edad" : 11 }, "total" : 2455 }
{ "_id" : { "nombre" : "Juan", "edad" : 26 }, "total" : 2462 }
{ "_id" : { "nombre" : "Lucía", "edad" : 25 }, "total" : 2507 }
{ "_id" : { "nombre" : "Lucía", "edad" : 84 }, "total" : 2519 }
{ "_id" : { "nombre" : "Lucía", "edad" : 57 }, "total" : 2426 }
{ "_id" : { "nombre" : "Juan", "edad" : 8 }, "total" : 2467 }
{ "_id" : { "nombre" : "Carlos", "edad" : 16 }, "total" : 2406 }
{ "_id" : { "nombre" : "Lucía", "edad" : 82 }, "total" : 2521 }
{ "_id" : { "nombre" : "María", "edad" : 35 }, "total" : 2488 }
{ "_id" : { "nombre" : "Juan", "edad" : 22 }, "total" : 2634 }
{ "_id" : { "nombre" : "María", "edad" : 51 }, "total" : 2483 }
{ "_id" : { "nombre" : "Juan", "edad" : 88 }, "total" : 2548 }
{ "_id" : { "nombre" : "Juan", "edad" : 96 }, "total" : 2539 }
{ "_id" : { "nombre" : "María", "edad" : 78 }, "total" : 2543 }
{ "_id" : { "nombre" : "Lucía", "edad" : 6 }, "total" : 2448 }
{ "_id" : { "nombre" : "Carlos", "edad" : 44 }, "total" : 2516 }
{ "_id" : { "nombre" : "Lucía", "edad" : 41 }, "total" : 2477 }
{ "_id" : { "nombre" : "María", "edad" : 11 }, "total" : 2511 }
Type "it" for more
>                                                                    
```

Agrupa por nombre y edad y nos da su 


```sh
> db.participantes.aggregate([ {$group: {_id: {nombre: "$nombre", edad: "$edad"}, total: {$sum: 1} }}, {$sort: {total: -1}}, {$limit: 3}  ])
{ "_id" : { "nombre" : "Lucía", "edad" : 78 }, "total" : 2646 }
{ "_id" : { "nombre" : "Juan", "edad" : 22 }, "total" : 2634 }
{ "_id" : { "nombre" : "Lucía", "edad" : 91 }, "total" : 2617 }
```

```sh
> db.participantes.aggregate([ {$group: { _id: "$edad", registros:{$push: "$$ROOT" } }} ])
2020-02-10T10:10:20.738+0100 E  QUERY    [js] uncaught exception: Error: command failed: {
        "ok" : 0,
        "errmsg" : "Exceeded memory limit for $group, but didn't allow external sort. Pass allowDiskUse:true to opt in.",
        "code" : 16945,
        "codeName" : "Location16945"
} : aggregate failed :
_getErrorWithCode@src/mongo/shell/utils.js:25:13
doassert@src/mongo/shell/assert.js:18:14
_assertCommandWorked@src/mongo/shell/assert.js:583:17
assert.commandWorked@src/mongo/shell/assert.js:673:16
DB.prototype._runAggregate@src/mongo/shell/db.js:266:5
DBCollection.prototype.aggregate@src/mongo/shell/collection.js:1012:12
@(shell):1:1
>                                          
```
Esto no da error por que tiene limite de 100MB.

```sh
> db.participantes.aggregate([ {$group: { _id: "$edad", registros:{$push: "$$ROOT" } }} ], {allowDiskUse: true}) 
```

Con `allowDiskUse: true` ya nos permite usar memoria en disco y puede ejecutar el comando, salen muchos datos hayq que cortarlo. 

En el campo `registro` hace un array con todos los documentos agrupados por edad.
```
registro: [
 {_id, nombre ....}
 {...}
 ....
]
```

```sh
> db.games.aggregate([
... {$group: {_id: "$juego.nombre", registros: {$push: "$$ROOT"} } }
... ])
{ "_id" : null, "registros" : [ { "_id" : ObjectId("5e317d9351e2181056d76dde"), "jugador" : "pepe", "juego" : "Tetris", "puntos" : [ 79, 102, 89, 101 ] }, { "_id" : ObjectId("5e317daf51e2181056d76ddf"), "jugador" : "luisa", "juego" : "Tetris", "puntos" : [ 120, 99, 100, 120 ] }, { "_id" : ObjectId("5e317dcb51e2181056d76de0"), "jugador" : "luis", "juego" : "Pacman", "puntos" : [ 120, 99, 100, 120 ] } ] }
>   
```
En este ejemplo si puedo los resultados por ser menos registros. Me crea un array de documentos agrupados por lo que yo le indique y el `$push` mete todo el documento.

**Uso de `$merge`**

```sh
> use maraton
switched to db maraton
> db.participantes.aggregate([
... {$match: {edad: {$gte: 50} } },
... {$group: {_id: "$nombre", total: {$sum: 1} }},
... {$merge: "nombresSenior" }
... ])
> show collections
foo
nombresSenior
participantes
> db.nombresSenior.find().count()
4
> db.nombresSenior.find()
{ "_id" : "María", "total" : 125180 }
{ "_id" : "Carlos", "total" : 125385 }
{ "_id" : "Lucía", "total" : 124955 }
{ "_id" : "Juan", "total" : 124604 }
```

Me crea una nueva colección `nombresSenior`, **Si esa colección ya existiera AÑADE LO QUE SE GENERA** como vemos en el siguiente ejemplo:

```sh
> db.foo.find()
{ "_id" : ObjectId("5e3d203dd8b0fd4ac47898e6"), "nombre" : "Carlos", "apellidos1" : "Gómez" }
> db.participantes.aggregate([ {$match: {edad: {$gte: 50} } }, {$group: {_id: "$nombre", total: {$sum: 1} }}, {$merge: "foo" } ])
> db.foo.find()
{ "_id" : ObjectId("5e3d203dd8b0fd4ac47898e6"), "nombre" : "Carlos", "apellidos1" : "Gómez" }
{ "_id" : "Carlos", "total" : 125385 }
{ "_id" : "Lucía", "total" : 124955 }
{ "_id" : "María", "total" : 125180 }
{ "_id" : "Juan", "total" : 124604 }
>                                      
```

Antes existia el operador **`$out`** y este si machacaba la colección por lo que se genere, como podemos ver en el siguiente ejemplo.


```sh
> db.participantes.aggregate([ {$match: {edad: {$gte: 50} } }, {$group: {_id: "$nombre", total: {$sum: 1} }}, {$out: "foo" } ])
> db.foo.find()
{ "_id" : "Carlos", "total" : 125385 }
{ "_id" : "Lucía", "total" : 124955 }
{ "_id" : "María", "total" : 125180 }
{ "_id" : "Juan", "total" : 124604 }
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

