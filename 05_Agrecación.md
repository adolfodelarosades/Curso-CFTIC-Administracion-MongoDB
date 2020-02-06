# 05 AGREGACION (06/02/2020)

:skull: Sólo una pregunta sobre todo el tema.

Operadore:

* 

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
... { $project: { _id:0, cliente: { $toUpper: "$nombre"} } },
... { $sort: { cliente: 1 } }
... ])
{ "cliente" : "CARLOS" }
{ "cliente" : "LAURA" }
{ "cliente" : "PEDRO" }
{ "cliente" : "SARA" }
>       
```

Le he cambiado el nombre al campo `nombre` por `cliente` y posteriormente ordenamos a `cliente`.

**Cada etapa va haciendo referencia a la etapa anterior**

```sh

```
