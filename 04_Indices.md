# Jueves 06/02/2020

## Uso y rendimiento índices 

### Uso del índice

* El orden de los campos en la consulta no influye (por que Mongo parsea la consulta para colocarla de acuerdo al orden del índice)
* Para que la consulta use el índice debe contener campos que estén en los prefijos delimitados por el índice.
* Si tiene el primer campo, siempre usara el índice (la eficiencia ya es otra cosa)

### Elección del índice en caso de que varios índices se puedan utilizar.

* **Query Optimizer** que evalua para una determinada "forma" de la consulta cual de los índices es el más eficiente.
* Vuelca los planes ganadores en **Plan Cache**.

Podemos monitorizar los índices para saber cuales son los más usados y borrar los que menos se usen.

### Uso de los índices en las consultas ordenadas

Como se distribuye el uso de los recursos en MongoDB

Memoria Ram

* Indices 
* dataset "fuente de datos"

Disco Duro

* Datos almacenados

En el disco busco los datos que haga la consulta y la ordenación la hace en la RAM y es muy costoso, hay limete de 32 MB para ordenar.

¿Qué pasa si el orden del índice coincide con el orden de consulta? **Es más rápido** y se consume menos RAM.

**Tratar para las consultas ordenadas un índice**

Para crear la BD maraton
```js
db = db.getSiblingDB("maraton");

let nombres = ["Carlos", "Lucía", "Juan", "María"];
let apellidos = ["Rodríguez", "López", "Gómez", "García"];
let letras = ["A","C","F","U","J","K", "P"];
let participantes = [];

for( i=0; i < 1000000; i++){
    participantes.push({
        nombre: nombres[Math.floor(Math.random()* nombres.length)],
        apellidos1: apellidos[Math.floor(Math.random()* apellidos.length)],
        apellidos2: apellidos[Math.floor(Math.random()* apellidos.length)],
        edad: Math.floor(Math.random()*100),
        dni: Math.floor(Math.random() * 100000000) + letras[Math.floor(Math.random() * letras.length)]
    });
}

db.participantes.insert(participantes);
```

```sh
> use maraton
switched to db maraton
> db.participantes.dropIndexes()
{
        "nIndexesWas" : 1,
        "msg" : "non-_id indexes dropped for collection",
        "ok" : 1
}
>

```

1. Ordenación sin índices limite 32 MB

   Realizará la ordenación en memoría con límite.

```sh
> db.participantes.find().sort({apellido1: 1}).explain("allPlansExecution")
{
        "queryPlanner" : {
                "plannerVersion" : 1,
                "namespace" : "maraton.participantes",
                "indexFilterSet" : false,
                "parsedQuery" : {

                },
                "winningPlan" : {
                        "stage" : "SORT",
                        "sortPattern" : {
                                "apellido1" : 1
                        },
                        "inputStage" : {
                                "stage" : "SORT_KEY_GENERATOR",
                                "inputStage" : {
                                        "stage" : "COLLSCAN",
                                        "direction" : "forward"
                                }
                        }
                },
                "rejectedPlans" : [ ]
        },
        "executionStats" : {
                "executionSuccess" : false,
                "errorMessage" : "Exec error resulting in state FAILURE :: caused by :: Sort operation used more than the maximum 33554432 bytes of RAM. Add an index, or specify a smaller limit.",
                "errorCode" : 96,
                "nReturned" : 0,
                "executionTimeMillis" : 674,
                "totalKeysExamined" : 0,
                "totalDocsExamined" : 258348,
                "executionStages" : {
                        "stage" : "SORT",
                        "nReturned" : 0,
                        "executionTimeMillisEstimate" : 206,
                        "works" : 258351,
                        "advanced" : 0,
                        "needTime" : 258350,
                        "needYield" : 0,
                        "saveState" : 2021,
                        "restoreState" : 2021,
                        "isEOF" : 0,
                        "sortPattern" : {
                                "apellido1" : 1
                        },
                        "memUsage" : 33554508,
                        "memLimit" : 33554432,
                        "inputStage" : {
                                "stage" : "SORT_KEY_GENERATOR",
                                "nReturned" : 258348,
                                "executionTimeMillisEstimate" : 197,
                                "works" : 258350,
                                "advanced" : 258348,
                                "needTime" : 2,
                                "needYield" : 0,
                                "saveState" : 2021,
                                "restoreState" : 2021,
                                "isEOF" : 0,
                                "inputStage" : {
                                        "stage" : "COLLSCAN",
                                        "nReturned" : 258348,
                                        "executionTimeMillisEstimate" : 192,
                                        "works" : 258349,
                                        "advanced" : 258348,
                                        "needTime" : 1,
                                        "needYield" : 0,
                                        "saveState" : 2021,
                                        "restoreState" : 2021,
                                        "isEOF" : 0,
                                        "direction" : "forward",
                                        "docsExamined" : 258348
                                }
                        }
                },
                "allPlansExecution" : [ ]
        },
        "serverInfo" : {
                "host" : "G24-EQ09",
                "port" : 27017,
                "version" : "4.2.2",
                "gitVersion" : "a0bbbff6ada159e19298d37946ac8dc4b497eadf"
        },
        "ok" : 1
}
>         
```
Me marca el error que indica que nos estamos pasando de los 32 MB y no lo puede ordenar.

Si queremos saltarnos esto será imprecindible tener un índice así ayudamos a la RAM.

Si ejecuto solo la consulta tambén da error:

```sh
> db.participantes.find().sort({apellido1: 1})
Error: error: {
        "ok" : 0,
        "errmsg" : "Executor error during find command :: caused by :: Sort operation used more than the maximum 33554432 bytes of RAM. Add an index, or specify a smaller limit.",
        "code" : 96,
        "codeName" : "OperationFailed"
}
>      
```

Si limitamos la consulta lo ejecutará, por que usa menos registros y por lo tanto menos RAM.

```sh
> db.participantes.find({apellido1:/^L/}).sort({apellido1: 1}).explain("allPlansExecution")
{
        "queryPlanner" : {
                "plannerVersion" : 1,
                "namespace" : "maraton.participantes",
                "indexFilterSet" : false,
                "parsedQuery" : {
                        "apellido1" : {
                                "$regex" : "^L"
                        }
                },
                "winningPlan" : {
                        "stage" : "SORT",
                        "sortPattern" : {
                                "apellido1" : 1
                        },
                        "inputStage" : {
                                "stage" : "SORT_KEY_GENERATOR",
                                "inputStage" : {
                                        "stage" : "COLLSCAN",
                                        "filter" : {
                                                "apellido1" : {
                                                        "$regex" : "^L"
                                                }
                                        },
                                        "direction" : "forward"
                                }
                        }
                },
                "rejectedPlans" : [ ]
        },
        "executionStats" : {
                "executionSuccess" : true,
                "nReturned" : 0,
                "executionTimeMillis" : 821,
                "totalKeysExamined" : 0,
                "totalDocsExamined" : 1000000,
                "executionStages" : {
                        "stage" : "SORT",
                        "nReturned" : 0,
                        "executionTimeMillisEstimate" : 210,
                        "works" : 1000004,
                        "advanced" : 0,
                        "needTime" : 1000003,
                        "needYield" : 0,
                        "saveState" : 7816,
                        "restoreState" : 7816,
                        "isEOF" : 1,
                        "sortPattern" : {
                                "apellido1" : 1
                        },
                        "memUsage" : 0,
                        "memLimit" : 33554432,
                        "inputStage" : {
                                "stage" : "SORT_KEY_GENERATOR",
                                "nReturned" : 0,
                                "executionTimeMillisEstimate" : 210,
                                "works" : 1000003,
                                "advanced" : 0,
                                "needTime" : 1000002,
                                "needYield" : 0,
                                "saveState" : 7816,
                                "restoreState" : 7816,
                                "isEOF" : 1,
                                "inputStage" : {
                                        "stage" : "COLLSCAN",
                                        "filter" : {
                                                "apellido1" : {
                                                        "$regex" : "^L"
                                                }
                                        },
                                        "nReturned" : 0,
                                        "executionTimeMillisEstimate" : 210,
                                        "works" : 1000002,
                                        "advanced" : 0,
                                        "needTime" : 1000001,
                                        "needYield" : 0,
                                        "saveState" : 7816,
                                        "restoreState" : 7816,
                                        "isEOF" : 1,
                                        "direction" : "forward",
                                        "docsExamined" : 1000000
                                }
                        }
                },
                "allPlansExecution" : [ ]
        },
        "serverInfo" : {
                "host" : "G24-EQ09",
                "port" : 27017,
                "version" : "4.2.2",
                "gitVersion" : "a0bbbff6ada159e19298d37946ac8dc4b497eadf"
        },
        "ok" : 1
}
>      
```

### Motores de Almacenamiento:

* Wired Tiger
* In-memory

MongoDB vende un sistema de InMemory que va todo en la memoria, en disco sólo los Backups, es muy caro pero muy rápido.

### Vizualizando el explain()

Viendo el explain() con Koda:

<img src="/images/koda1.png">

Viendo el explain() con Compass:

<img src="/images/compass.png">

Viendo el explain() con la Shell:

```sh
> db.participantes.find({apellido1:"López"}).sort({apellido1: 1}).explain("allPlansExecution")
{
        "queryPlanner" : {
                "plannerVersion" : 1,
                "namespace" : "maraton.participantes",
                "indexFilterSet" : false,
                "parsedQuery" : {
                        "apellido1" : {
                                "$eq" : "López"
                        }
                },
                "winningPlan" : {
                        "stage" : "SORT",
                        "sortPattern" : {
                                "apellido1" : 1
                        },
                        "inputStage" : {
                                "stage" : "SORT_KEY_GENERATOR",
                                "inputStage" : {
                                        "stage" : "COLLSCAN",
                                        "filter" : {
                                                "apellido1" : {
                                                        "$eq" : "López"
                                                }
                                        },
                                        "direction" : "forward"
                                }
                        }
                },
                "rejectedPlans" : [ ]
        },
        "executionStats" : {
                "executionSuccess" : true,
                "nReturned" : 0,
                "executionTimeMillis" : 430,
                "totalKeysExamined" : 0,
                "totalDocsExamined" : 1000000,
                "executionStages" : {
                        "stage" : "SORT",
                        "nReturned" : 0,
                        "executionTimeMillisEstimate" : 0,
                        "works" : 1000004,
                        "advanced" : 0,
                        "needTime" : 1000003,
                        "needYield" : 0,
                        "saveState" : 7812,
                        "restoreState" : 7812,
                        "isEOF" : 1,
                        "sortPattern" : {
                                "apellido1" : 1
                        },
                        "memUsage" : 0,
                        "memLimit" : 33554432,
                        "inputStage" : {
                                "stage" : "SORT_KEY_GENERATOR",
                                "nReturned" : 0,
                                "executionTimeMillisEstimate" : 0,
                                "works" : 1000003,
                                "advanced" : 0,
                                "needTime" : 1000002,
                                "needYield" : 0,
                                "saveState" : 7812,
                                "restoreState" : 7812,
                                "isEOF" : 1,
                                "inputStage" : {
                                        "stage" : "COLLSCAN",
                                        "filter" : {
                                                "apellido1" : {
                                                        "$eq" : "López"
                                                }
                                        },
                                        "nReturned" : 0,
                                        "executionTimeMillisEstimate" : 0,
                                        "works" : 1000002,
                                        "advanced" : 0,
                                        "needTime" : 1000001,
                                        "needYield" : 0,
                                        "saveState" : 7812,
                                        "restoreState" : 7812,
                                        "isEOF" : 1,
                                        "direction" : "forward",
                                        "docsExamined" : 1000000
                                }
                        }
                },
                "allPlansExecution" : [ ]
        },
        "serverInfo" : {
                "host" : "G24-EQ09",
                "port" : 27017,
                "version" : "4.2.2",
                "gitVersion" : "a0bbbff6ada159e19298d37946ac8dc4b497eadf"
        },
        "ok" : 1
}
>
```

2. Ordenación con índice simple.

   Si la consulta usa el índice no ordenará en memoria tanto si la ordenación es ascendente o descendente.

```sh
> db.participantes.createIndex({apellido1: 1})
{
        "createdCollectionAutomatically" : false,
        "numIndexesBefore" : 1,
        "numIndexesAfter" : 2,
        "ok" : 1
}
>    

```

Si ejecuto de nuevo la consulta es más rapido

<img src="/images/koda2.png">

<img src="/images/compass2.png">

**CON ESTO NO ORDENAS EN MEMORIA** 

Da lo mismo que la consulta sea ascendente o descendentes, da igual en el sentido de que se aplica el índice, tardan más o menos lo mismo.


3. Ordenación con índice compuesto

   No tendrá que ordenar en memoria si la ordenación usa el mismo sentido de todos los campos del índice o el sentido inverso de todos los campos del índice.
   
   

```sh
> db.participantes.dropIndexes()
{
        "nIndexesWas" : 3,
        "msg" : "non-_id indexes dropped for collection",
        "ok" : 1
}
> db.participantes.createIndex({apellidos1: 1, edad: -1})
{
        "createdCollectionAutomatically" : false,
        "numIndexesBefore" : 1,
        "numIndexesAfter" : 2,
        "ok" : 1
}
>
```

¿Cual es el crierio para usar la ordenación?

Existen 4 casos dos son posibles.

{apellido1: 1, edad: -1} 

No tendrá que ordenar en memoria las operaciones 
 ```sh
 .sort({apellido1: 1, edad: -1})
 .sort({apellido1: -1, edad: 1})
 ```
Si tendrá que ordenar en memoria las operaciones 
 ```sh
 .sort({apellido1: 1, edad: 1})
 .sort({apellido1: 1, edad: 1})
 ```
 
<img src="/images/koda3.png">

<img src="/images/koda4.png">

<img src="/images/compass3.png">

<img src="/images/compass4.png">

Lee de arriba abajo o abajo arriba según necesite

<img src="/images/koda5.png">

<img src="/images/koda6.png">

<img src="/images/compass5.png">

<img src="/images/compass6.png">
 
Cuando cambia un sentido del orden, si necesita ordenar en memoria.

**En Compass me indica si necesita usar la memoria con un texto**

4. Ordenación con índice compuesto de tres o más campos

   No tendrá que ordenar en memoria si usa el índice y en la operación sort() contiene alguno de los prefijos del índice con el anterior criterio de ordenación.

```sh
> db.participantes.dropIndexes()
{
        "nIndexesWas" : 2,
        "msg" : "non-_id indexes dropped for collection",
        "ok" : 1
}
> db.participantes.createIndex({apellidos1: 1, apellidos2: 1, edad: -1})
{
        "createdCollectionAutomatically" : false,
        "numIndexesBefore" : 1,
        "numIndexesAfter" : 2,
        "ok" : 1
}
>      
```

Por ejemplo:
`{apellidos1: 1, apellidos2: 1, edad: -1}`

No tendríamos que ordenar en memoria con los siguientes prefijos:

```sh
.sort({apellidos1: 1})
.sort({apellidos1: -1})
.sort({apellidos1: 1, apellidos2: 1})
.sort({apellidos1: -1, apellidos2: -1})
.sort({apellidos1: 1, apellidos2: 1, edad: -1})
.sort({apellidos1: -1, apellidos2: -1, edad: 1})
```

**EL ORDEN EN QUE LOS PONGA LOS CAMPOS EN EL SORT SI INFLUYE** 

Comprobación

<img src="/images/kodak7.png">

<img src="/images/kodak8.png">

<img src="/images/kodak9.png">

<img src="/images/kodak10.png">

<img src="/images/kodak11.png">

<img src="/images/kodak12.png">

 
Si hago .sort({apellidos1: 1, edad: -1}) necesitara usar la memoria por que en medio falta apellidos2:

<img src="/images/kodak15.png">

5. Uso del índice para no tener que ordenar en memoria aunque en la consulta no haya campos del índice. 

Por ejemplo para el índice:
{apellidos1: 1, apellidos2: 1, edad: -1}
No tendría que ordenar en memoria con:

`find({nombre:"María}).sort({apellidos1: 1, apellidos2: 1, edad: -1})`
 
    5.1. Usa el índice para escanear todos los registros de la colección en el orden de sort().
    5.2 Al escaneo le aplica el filtro de la consulta nombre igual a "María".
     
Aplica el índice aunque en nombre no este en el indice, para ordernar y sobre todos los registros ordenados ya recupera los que cumplen la consulta.

<img src="/images/kodak16.png">

6. Complementar el prefijo de la operación de ordenación con un campo en la consulta.
   Siempre que:
   
   * La condición de filtro sea de igauldad.
   * La consulta (filtro) utilice el índice.
   
`find().sort({ apellidos2: 1, edad: -1})`

<img src="/images/kodak17.png">

`find({apellidos1:"López"}).sort({apellidos2: 1, edad: -1})` Siempre con criterio de igualdad

<img src="/images/kodak18.png">

Que pasa si el campo es el de enmedio en la consulta 

<img src="/images/kodak19.png">

No funciona por que en la consulta no usa el indice.

Otro caso dos campos en la consulta y uno en el Sort.

<img src="/images/kodak20.png">


### "Asegúrate que los índices caben en la RAM"

```sh
db.<coleccion>.totalIndexSize()

> db.participantes.totalIndexSize()
16781312
```
Equivale a 16 MB

Los índices estan en disco.

MongoDB los carga en memoria.

Si hay espacio en RAM sube todos los indices y sino sube algunos.

### :skull: Consultas totalmente cubiertas (Covered Querys)

* Una  consulta totalmente cubierta sera aquella que sólo examinrá un índice y no examinará ningún documento.

   * Todos los campos de la consulta (filtro) estén contenidos en el mismo índice.
   * Todos los campos devueltos por la consulta estén en ese mismo índice.
   
<img src="/images/kodak21.png">

**Todos los campos en la consulta deben pertenecer al índice y todos los campos de la proyección deben pertenecer al índice**

Debo excluir el `_id` por que no se encuentra en el índice, si lo dejo ya no es consulta totalmente cubierta.

<img src="/images/kodak22.png">

## Uso del índice con colación

* Crear índices con colación.
* Pra que se use el índice con colación la consulta tiene que especificar exactamente la misma colación, si cambio el valor ya no es la misma.

La colación tiene como objetivo:

* Ordena según el lenguaje que especifiquemos.
* Controlar se en las consultas se tienen en cuenta o no
   * Mayus/ Minus
   * Diacríticos
   
```sh
> use maraton
switched to db maraton
> db.participantes.dropIndexes()
{
        "nIndexesWas" : 2,
        "msg" : "non-_id indexes dropped for collection",
        "ok" : 1
}
> db.participantes.createIndex({ nombre: 1}, { collation: {locale: "es", strength: 1}})
{
        "createdCollectionAutomatically" : false,
        "numIndexesBefore" : 1,
        "numIndexesAfter" : 2,
        "ok" : 1
}
>      

> db.participantes.find({nombre: "juan"}).explain("allPlansExecution")
{
        "queryPlanner" : {
                "plannerVersion" : 1,
                "namespace" : "maraton.participantes",
                "indexFilterSet" : false,
                "parsedQuery" : {
                        "nombre" : {
                                "$eq" : "juan"
                        }
                },
                "winningPlan" : {
                        "stage" : "COLLSCAN",
                        "filter" : {
                                "nombre" : {
                                        "$eq" : "juan"
                                }
                        },
                        "direction" : "forward"
                },
                "rejectedPlans" : [ ]
        },
        "executionStats" : {
                "executionSuccess" : true,
                "nReturned" : 0,
                "executionTimeMillis" : 387,
                "totalKeysExamined" : 0,
                "totalDocsExamined" : 1000000,
                "executionStages" : {
                        "stage" : "COLLSCAN",
                        "filter" : {
                                "nombre" : {
                                        "$eq" : "juan"
                                }
                        },
                        "nReturned" : 0,
                        "executionTimeMillisEstimate" : 0,
                        "works" : 1000002,
                        "advanced" : 0,
                        "needTime" : 1000001,
                        "needYield" : 0,
                        "saveState" : 7812,
                        "restoreState" : 7812,
                        "isEOF" : 1,
                        "direction" : "forward",
                        "docsExamined" : 1000000
                },
                "allPlansExecution" : [ ]
        },
        "serverInfo" : {
                "host" : "G24-EQ09",
                "port" : 27017,
                "version" : "4.2.2",
                "gitVersion" : "a0bbbff6ada159e19298d37946ac8dc4b497eadf"
        },
        "ok" : 1
}

> db.participantes.find({nombre: "juan"}).collation({locale: "es", strength: 1}).explain("allPlansExecution")
{
        "queryPlanner" : {
                "plannerVersion" : 1,
                "namespace" : "maraton.participantes",
                "indexFilterSet" : false,
                "parsedQuery" : {
                        "nombre" : {
                                "$eq" : "juan"
                        }
                },
                "collation" : {
                        "locale" : "es",
                        "caseLevel" : false,
                        "caseFirst" : "off",
                        "strength" : 1,
                        "numericOrdering" : false,
                        "alternate" : "non-ignorable",
                        "maxVariable" : "punct",
                        "normalization" : false,
                        "backwards" : false,
                        "version" : "57.1"
                },
                "winningPlan" : {
                        "stage" : "FETCH",
                        "inputStage" : {
                                "stage" : "IXSCAN",
                                "keyPattern" : {
                                        "nombre" : 1
                                },
                                "indexName" : "nombre_1",
                                "collation" : {
                                        "locale" : "es",
                                        "caseLevel" : false,
                                        "caseFirst" : "off",
                                        "strength" : 1,
                                        "numericOrdering" : false,
                                        "alternate" : "non-ignorable",
                                        "maxVariable" : "punct",
                                        "normalization" : false,
                                        "backwards" : false,
                                        "version" : "57.1"
                                },
                                "isMultiKey" : false,
                                "multiKeyPaths" : {
                                        "nombre" : [ ]
                                },
                                "isUnique" : false,
                                "isSparse" : false,
                                "isPartial" : false,
                                "indexVersion" : 2,
                                "direction" : "forward",
                                "indexBounds" : {
                                        "nombre" : [
                                                "[\";Q)C\", \";Q)C\"]"
                                        ]
                                }
                        }
                },
                "rejectedPlans" : [ ]
        },
        "executionStats" : {
                "executionSuccess" : true,
                "nReturned" : 249888,
                "executionTimeMillis" : 286,
                "totalKeysExamined" : 249888,
                "totalDocsExamined" : 249888,
                "executionStages" : {
                        "stage" : "FETCH",
                        "nReturned" : 249888,
                        "executionTimeMillisEstimate" : 8,
                        "works" : 249889,
                        "advanced" : 249888,
                        "needTime" : 0,
                        "needYield" : 0,
                        "saveState" : 1952,
                        "restoreState" : 1952,
                        "isEOF" : 1,
                        "docsExamined" : 249888,
                        "alreadyHasObj" : 0,
                        "inputStage" : {
                                "stage" : "IXSCAN",
                                "nReturned" : 249888,
                                "executionTimeMillisEstimate" : 5,
                                "works" : 249889,
                                "advanced" : 249888,
                                "needTime" : 0,
                                "needYield" : 0,
                                "saveState" : 1952,
                                "restoreState" : 1952,
                                "isEOF" : 1,
                                "keyPattern" : {
                                        "nombre" : 1
                                },
                                "indexName" : "nombre_1",
                                "collation" : {
                                        "locale" : "es",
                                        "caseLevel" : false,
                                        "caseFirst" : "off",
                                        "strength" : 1,
                                        "numericOrdering" : false,
                                        "alternate" : "non-ignorable",
                                        "maxVariable" : "punct",
                                        "normalization" : false,
                                        "backwards" : false,
                                        "version" : "57.1"
                                },
                                "isMultiKey" : false,
                                "multiKeyPaths" : {
                                        "nombre" : [ ]
                                },
                                "isUnique" : false,
                                "isSparse" : false,
                                "isPartial" : false,
                                "indexVersion" : 2,
                                "direction" : "forward",
                                "indexBounds" : {
                                        "nombre" : [
                                                "[\";Q)C\", \";Q)C\"]"
                                        ]
                                },
                                "keysExamined" : 249888,
                                "seeks" : 1,
                                "dupsTested" : 0,
                                "dupsDropped" : 0
                        }
                },
                "allPlansExecution" : [ ]
        },
        "serverInfo" : {
                "host" : "G24-EQ09",
                "port" : 27017,
                "version" : "4.2.2",
                "gitVersion" : "a0bbbff6ada159e19298d37946ac8dc4b497eadf"
        },
        "ok" : 1
}
> 
> db.participantes.find({nombre: "juan"}).collation({locale: "en", strength: 1}).explain("allPlansExecution")
{
        "queryPlanner" : {
                "plannerVersion" : 1,
                "namespace" : "maraton.participantes",
                "indexFilterSet" : false,
                "parsedQuery" : {
                        "nombre" : {
                                "$eq" : "juan"
                        }
                },
                "collation" : {
                        "locale" : "en",
                        "caseLevel" : false,
                        "caseFirst" : "off",
                        "strength" : 1,
                        "numericOrdering" : false,
                        "alternate" : "non-ignorable",
                        "maxVariable" : "punct",
                        "normalization" : false,
                        "backwards" : false,
                        "version" : "57.1"
                },
                "winningPlan" : {
                        "stage" : "COLLSCAN",
                        "filter" : {
                                "nombre" : {
                                        "$eq" : "juan"
                                }
                        },
                        "direction" : "forward"
                },
                "rejectedPlans" : [ ]
        },
        "executionStats" : {
                "executionSuccess" : true,
                "nReturned" : 249888,
                "executionTimeMillis" : 431,
                "totalKeysExamined" : 0,
                "totalDocsExamined" : 1000000,
                "executionStages" : {
                        "stage" : "COLLSCAN",
                        "filter" : {
                                "nombre" : {
                                        "$eq" : "juan"
                                }
                        },
                        "nReturned" : 249888,
                        "executionTimeMillisEstimate" : 0,
                        "works" : 1000002,
                        "advanced" : 249888,
                        "needTime" : 750113,
                        "needYield" : 0,
                        "saveState" : 7812,
                        "restoreState" : 7812,
                        "isEOF" : 1,
                        "direction" : "forward",
                        "docsExamined" : 1000000
                },
                "allPlansExecution" : [ ]
        },
        "serverInfo" : {
                "host" : "G24-EQ09",
                "port" : 27017,
                "version" : "4.2.2",
                "gitVersion" : "a0bbbff6ada159e19298d37946ac8dc4b497eadf"
        },
        "ok" : 1
}
>                                                            

```

Otro caso si el indice contiene la colation no es necesario meterla 

```sh
> db.createCollection("foo", {collation: {locale: "es", strength: 1 }})
{ "ok" : 1 }

> db.foo.createIndex({apellidos1: 1}, {collation: {locale: "es", strangth: 1}})
{
        "ok" : 0,
        "errmsg" : "failed to add collation information to index spec for index creation: { ns: \"maraton.foo\", v: 2, key: { apellidos1: 1.0 }, name: \"apellidos1_1\", collation: { locale: \"es\", strangth: 1.0 } } :: caused by :: Collation spec contains unknown field. Collation spec: { locale: \"es\", strangth: 1.0 }",
        "code" : 9,
        "codeName" : "FailedToParse"
}
> db.foo.insert({nombre: "Carlos", apellidos1: "Gómez"})
WriteResult({ "nInserted" : 1 })
> db.foo.find({apellidos: "López"}) .explain("allPlansExecution")
```






