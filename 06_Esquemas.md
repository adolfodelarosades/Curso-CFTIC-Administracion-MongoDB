# 6 Esquemas o Modelo

[Data Modeling Introduction](https://docs.mongodb.com/manual/core/data-modeling-introduction/)

### Flexibilidad de la estructura de datos en MongoDB.

* Los documentos de una misma colección no tienen por que tener la misma estructura ni tipos de datos.
* No es necesario especificar al crear la colección ningún tipo de estructura.
* Es posible modificar la estructura de los documentos a posteriori.

### Tipos de relaciones en MongoDB

1. Documento embebido o modelo **denormalizado** [Embedded Data](https://docs.mongodb.com/manual/core/data-modeling-introduction/#embedded-data)

   Tipo de relación por defecto de MongoDB y al que se debe recurrir siempre que se pueda.
   
   Colección Productos
   ```
   {
     producto: "Nike TF55",
     marca: "Nike",
     distribuidores:[
       {nombre: "ServiZapas", contacto: "xxx", ....},
       {nombre: "Distr. Pérez", contacto: "xxx", ....}
     ],
     precio: ...
   }
   ```

2. Documentos referenciados de otra colección o modelo normalizado [References](https://docs.mongodb.com/manual/core/data-modeling-introduction/#references)

   Colección Productos
   ```
   {
     _id: "01",
     producto: "Nike TF55",
     marca: "Nike",
     distribuidores:["d03", "d06"],
     precio: ...
   }
   ```
  
   Colección Distribuidores
   ```
   {
    _id: "d03",
    nombre: "ServiZapas",
    contacto: ...
   }
  
   {
    _id: "d06",
   nombre: "Distr. Pérez",
   contacto: ...
   }
   ```
   
   ### MODELO ONE-TO-ONE
   
   [Model One-to-One Relationships with Embedded Documents](https://docs.mongodb.com/manual/tutorial/model-embedded-one-to-one-relationships-between-documents/)
   Em MongoDB usaremos el modelo denormalizado o embebido.
   
   ```
   {
     _id: "2343",
     nombre: "Juan",
     ...,
     direccion: {
       calle: "Gran Vía, 30",
       cp: "28001",
       localidad: "Madrid"
     }
   }
   ```

   No tengo dos colecciones (clientes, direcciones) en una sola tengo ambos datos.
   
### MODELO ONE-TO-MANY
   
[Model One-to-Many Relationships with Embedded Documents¶](https://docs.mongodb.com/manual/tutorial/model-embedded-one-to-many-relationships-between-documents/)
[Model One-to-Many Relationships with Document References](https://docs.mongodb.com/manual/tutorial/model-referenced-one-to-many-relationships-between-documents/#model-one-to-many-relationships-with-document-references)
   
**Modelo One-to-Few**
   
En MongoDB intentemos utilizar el modelo denormalizado o embebido.
   
* El lado one es el que normalmente recibe más consultas
* Los documentos del lado many normalmente no tendrán escrituras frecuentes.
   
   Colección Productos (One)
   ```
    {
      _id: "01",
      producto: "Nike TF55",
      marca: "Nike",
      imagenes:[
        {url: "https://...", textoAlt: "Zapatillas ...."}
        {url: "https://...", textoAlt: "Zapatillas ...."}
        ...
      ],
      precio: ...
    }
   ```
   
   Las imagenes(Few) en teoría no cambian mucho, se embeben en el documento principal.
   
**Modelo One-to-Many**

En general, modelo denormalizado o embebido. Ver circunstancias concretas.

* Cada documento individual no puede superar 16MG,

**Modelo One-to-squillions** (muchísimos "tropecientos")

En general, modelo normalizado o referencia a otra colección.

* El lado many recibira muchas consultas.
* El lado many podría crecer hasta superar el límite de 16 MG del lado one.

 Colección Productos (One)
   ```
    {
      _id: "01",
      producto: "Nike TF55",
      marca: "Nike",
      opiniones:["265462", "254567" ],
      precio: ...
    }
   ```

Colección Opiniones (Many)

```
{
  _id: "265462",
  texto: "Buen producto",
  estrellas: 4,
  usuario: "..."
}
```

**Modelo Many-to-Many**

Con modelo denormalizado o documento embebido

* Mayor número de consultas en el lado del mayor número de registros.
* Redundancia de datos.

 Colección Productos (One)
   ```
    {
      _id: "01",
      producto: "Nike TF55",
      marca: "Nike",
      tiendas:[
        {nombre: "Alcorcón Store", direccion: "..."},
        {nombre: "Las Rozas Store", direccion: "..."},
        ...
      ],
      precio: ...
    },
    {
      _id: "09",
      producto: "Adidas AR57",
      marca: "Adidas",
      tiendas:[
        {nombre: "Gran Vía Store", direccion: "..."},
        {nombre: "Las Rosas Store", direccion: "..."},
        ...
      ],
      precio: ...
    }
   ```

La redundacia esta en las tiendas sin las inserto aquí y estas cambian puede haber problemas por que se estan poniendo para cada producto.

Con modelo normalizado o referencia a documentos.

* Consultas sobre los dos lados
* Alta Actulizada 

 Colección Productos (One)
   ```
    {
      _id: "01",
      producto: "Nike TF55",
      marca: "Nike",
      tiendas:["ALC", "LRZ"],
      precio: ...
    }
    
    {
      _id: "09",
      producto: "Adidas AR57",
      marca: "Adidas",
      tiendas:["ALC", "LRZ"],
      precio: ...
    }
   ```
   
   Colección Tiendas
   ```sh
   {
     _id: "GV",
     nombre: "Gran Vía Store",
     dir: ....
   }
    {
     _id: "LRZ",
     nombre: "Las Rosas Store",
     dir: ....
   }
    {
     _id: "ALC",
     nombre: "Alcorcón Store",
     dir: ....
   }
   ```

### Validación de Esquemas

[Schema Validation](https://docs.mongodb.com/manual/core/schema-validation/index.html)

Tu valida en aplicación. Llevate la validación a la aplicación no en la BD. Pero por si las dudas se puede hacer desde la propia BD.

Existen dos formas de validadr

* Esquema JSON
* 

**Esquema JSON** 
[JSON Schema](https://docs.mongodb.com/manual/core/schema-validation/index.html#json-schema)

[Operador `$jsonSchema`](https://docs.mongodb.com/manual/reference/operator/query/jsonSchema/)

```
db.createCollection(
  "<nombre-coleccion>",
  { validator:
    { $jsonSchema: {...} }
  }
)
```
`validationLevel`: "strct" (defecto) | "moderate"

* Si la colección ya existe con "strict" obliga a que los documentos de la coleccion cumplan con la validación cuando se actualicen. Con "moderate" los documentos antiguos a la validación pueden mantenerse sin validar cuando se actualicen.

`validationAction`: "error" (defecto) | "warn"

* La opción de "error" impide escrituras que no cumplan la validación y con la opcion "warn" permite la escritura pero emite un mensaje de advertencia.

Implementar un esquema de validación en una colección ya existente:

```
db.runCommand({
   collMod: "<coleccion>",
   validator: {
      //igual createCollection()
   }
})
```


Ejemplo:


```sh
db.createCollection(
    "pacientes", 
    { validator: {
          $jsonSchema: {
              bsonType: "object",
              required: ["nombre", "apellidos", "fechaNac", "direccion"],
              properties: {
                  nombre: {
                      bsonType: "string",
                      description: "Debe ser un string y es obligatorio"
                  },
                  apellidos: {
                    bsonType: "string",
                    description: "Debe ser un string y es obligatorio"
                  },
                  fechaNac: {
                    bsonType: "date",
                    description: "Debe ser un date y es obligatorio"
                  },
                  direccion: {
                    bsonType: "object",
                    required: ["calle", "cp","localidad"],
                    properties: {
                        calle: {
                            bsonType: "string",
                            description: "Debe ser un string y es obligatorio"
                        },
                        cp: {
                            bsonType: "string",
                            description: "Debe ser un string y es obligatorio"
                        },
                        localidad: {
                            bsonType: "string",
                            description: "Debe ser un string y es obligatorio"
                        }
                    }
                  }
                }
            }
        }
    }
)
```

```sh
> use clinica2
switched to db clinica2
> show collections
> db.createCollection(
...     "pacientes",
...     { validator: {
...           $jsonSchema: {
...               bsonType: "object",
...               required: ["nombre", "apellidos", "fechaNac", "direccion"],
...               properties: {
...                   nombre: {
...                       bsonType: "string",
...                       description: "Debe ser un string y es obligatorio"
...                   },
...                   apellidos: {
...                     bsonType: "string",
...                     description: "Debe ser un string y es obligatorio"
...                   },
...                   fechaNac: {
...                     bsonType: "date",
...                     description: "Debe ser un date y es obligatorio"
...                   },
...                   direccion: {
...                     bsonType: "object",
...                     required: ["calle", "cp","localidad"],
...                     properties: {
...                         calle: {
...                             bsonType: "string",
...                             description: "Debe ser un string y es obligatorio"
...                         },
...                         cp: {
...                             bsonType: "string",
...                             description: "Debe ser un string y es obligatorio"
...                         },
...                         localidad: {
...                             bsonType: "string",
...                             description: "Debe ser un string y es obligatorio"
...                         }
...                     }
...                   }
...                 }
...             }
...         }
...     }
... )
{ "ok" : 1 }

> db.pacientes.insert({
... nombre: "Juan",
... apellidos: "Pérez López",
... fechaNac: new Date(1980, 4, 5),
... direccion: {
... calle: "Gran Vía, 40",
... cp: "28001",
... localidad: "Madrid"
... }
... })
WriteResult({ "nInserted" : 1 })
```

`description` Mensaje de error que se puede usar desde los driver, la shell no lo muestra

**`Mongoose` tiene su propio esquema y es el que se suele usar no como en el ejemplo**.

Si trato de insertar un documento que no cumpla con tadas las restricciones que establecimos no permitira insertar el documento.
```sh
> db.pacientes.insert({ nombre: "Juan", apellidos: "Pérez López", fechaNac: new Date(1980, 4, 5) })
WriteResult({
        "nInserted" : 0,
        "writeError" : {
                "code" : 121,
                "errmsg" : "Document failed validation"
        }
})
```

Aplicar una validación a una colección ya eistente (medicos)
```sh
> db.medicos.insert({nombre: "Juan", apellidos: "Fernández"})
WriteResult({ "nInserted" : 1 })
> db.medicos.insert({nombre: "Laura", apellidos: "Pérez"})
WriteResult({ "nInserted" : 1 })

```

```sh
db.runCommand({
    collMod: "medicos", 
    validator: {
        $jsonSchema: {
            bsonType: "object",
            required: ["nombre", "apellidos", "fechaNac", "direccion"],
            properties: {
                nombre: {
                    bsonType: "string",
                    description: "Debe ser un string y es obligatorio"
                },
                apellidos: {
                bsonType: "string",
                description: "Debe ser un string y es obligatorio"
                },
                fechaNac: {
                bsonType: "date",
                description: "Debe ser un date y es obligatorio"
                },
                direccion: {
                bsonType: "object",
                required: ["calle", "cp","localidad"],
                properties: {
                    calle: {
                        bsonType: "string",
                        description: "Debe ser un string y es obligatorio"
                    },
                    cp: {
                        bsonType: "string",
                        description: "Debe ser un string y es obligatorio"
                    },
                    localidad: {
                        bsonType: "string",
                        description: "Debe ser un string y es obligatorio"
                    }
                }
                }
            }
        }
    }    
})


```

```sh
> db.medicos.find()
{ "_id" : ObjectId("5e414a3ad793860d1a933102"), "nombre" : "Juan", "apellidos" : "Fernández" }
{ "_id" : ObjectId("5e414a4dd793860d1a933103"), "nombre" : "Laura", "apellidos" : "Pérez" }
> db.medicos.insert({nombre: "Pedro"})
WriteResult({
        "nInserted" : 0,
        "writeError" : {
                "code" : 121,
                "errmsg" : "Document failed validation"
        }
})
> db.medicos.update({nombre: "Juan"}, {$set: {nombre: "Juan Pedro"}})
WriteResult({
        "nMatched" : 0,
        "nUpserted" : 0,
        "nModified" : 0,
        "writeError" : {
                "code" : 121,
                "errmsg" : "Document failed validation"
        }
})
>  
```


Si le añado un nivel de validación `moderate` me permite actualizar los que ya existian

```sh
db.createCollection(
    "pacientes", 
    { validator: {
          $jsonSchema: {
              bsonType: "object",
              required: ["nombre", "apellidos", "fechaNac", "direccion"],
              properties: {
                  nombre: {
                      bsonType: "string",
                      description: "Debe ser un string y es obligatorio"
                  },
                  apellidos: {
                    bsonType: "string",
                    description: "Debe ser un string y es obligatorio"
                  },
                  fechaNac: {
                    bsonType: "date",
                    description: "Debe ser un date y es obligatorio"
                  },
                  direccion: {
                    bsonType: "object",
                    required: ["calle", "cp","localidad"],
                    properties: {
                        calle: {
                            bsonType: "string",
                            description: "Debe ser un string y es obligatorio"
                        },
                        cp: {
                            bsonType: "string",
                            description: "Debe ser un string y es obligatorio"
                        },
                        localidad: {
                            bsonType: "string",
                            description: "Debe ser un string y es obligatorio"
                        }
                    }
                  }
                }
            }
        }
    }
)

//Aplicar validación a colección existente
db.runCommand({
    collMod: "medicos", 
    validator: {
        $jsonSchema: {
            bsonType: "object",
            required: ["nombre", "apellidos", "fechaNac", "direccion"],
            properties: {
                nombre: {
                    bsonType: "string",
                    description: "Debe ser un string y es obligatorio"
                },
                apellidos: {
                bsonType: "string",
                description: "Debe ser un string y es obligatorio"
                },
                fechaNac: {
                bsonType: "date",
                description: "Debe ser un date y es obligatorio"
                },
                direccion: {
                bsonType: "object",
                required: ["calle", "cp","localidad"],
                properties: {
                    calle: {
                        bsonType: "string",
                        description: "Debe ser un string y es obligatorio"
                    },
                    cp: {
                        bsonType: "string",
                        description: "Debe ser un string y es obligatorio"
                    },
                    localidad: {
                        bsonType: "string",
                        description: "Debe ser un string y es obligatorio"
                    }
                }
                }
            }
        }
    } ,
    validationLevel: "moderate"
})
```

```sh
> db.runCommand({
...     collMod: "medicos",
...     validator: {
...         $jsonSchema: {
...             bsonType: "object",
...             required: ["nombre", "apellidos", "fechaNac", "direccion"],
...             properties: {
...                 nombre: {
...                     bsonType: "string",
...                     description: "Debe ser un string y es obligatorio"
...                 },
...                 apellidos: {
...                 bsonType: "string",
...                 description: "Debe ser un string y es obligatorio"
...                 },
...                 fechaNac: {
...                 bsonType: "date",
...                 description: "Debe ser un date y es obligatorio"
...                 },
...                 direccion: {
...                 bsonType: "object",
...                 required: ["calle", "cp","localidad"],
...                 properties: {
...                     calle: {
...                         bsonType: "string",
...                         description: "Debe ser un string y es obligatorio"
...                     },
...                     cp: {
...                         bsonType: "string",
...                         description: "Debe ser un string y es obligatorio"
...                     },
...                     localidad: {
...                         bsonType: "string",
...                         description: "Debe ser un string y es obligatorio"
...                     }
...                 }
...                 }
...             }
...         }
...     } ,
...     validationLevel: "moderate"
... })
{ "ok" : 1 }

> db.medicos.update({nombre: "Juan"}, {$set: {nombre: "Juan Pedro"}})
WriteResult({ "nMatched" : 1, "nUpserted" : 0, "nModified" : 1 })
                                                                  
```

Como podemos ver nos permite modificar los registros antiguos.

