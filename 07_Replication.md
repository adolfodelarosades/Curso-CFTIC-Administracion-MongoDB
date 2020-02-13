# 7 Replication :skull: :skull: :skull: :skull: :skull: :skull: :skull:

[Replication](https://docs.mongodb.com/manual/replication/index.html)

## Introducción

* Un **replica set** o **cluster** es un grupo de servidores de base de datos MongoDB que mantiene el mismo set de datos proporcionado:
   * Redundacia: Como hay varios servidores la información esta copiada en cada uno de ellos.
   * Alta disponibilidad : Si solo tengo un servidor no tendre alta disponibilidad cuando se caiga mi único servidor.
   
   Y siendo la bas de datos de todos los despliegues en producción.
   
* Provee un nivel de tolerancia a fallos por la pérdida de servidores.

* En algunos casos, el cluster puede proporcionar un incremento en la capacidad de lectura.

* Amplia la disponibilidad de datos en aplicaciones distribuidas geográficamente.

* El cluster puede tener miembros (servidores) con propósitos dedicados como recuperación de desastres, backup o reporting.

### Componentes

* Varios miembros (nodos de producción) o servidores.
* solamente uno será el primario.
* Algunos de los miembros pueden ser "árbitros".

* Heatbeat (ping cada 2s) para comunicarse entre los miembros del cluster y detectar cuando se pierda la comunicación con uno de ellos.
* Replicación asíncrona entre el primario y los secundarios a través de una colección oplog (log de operaciones).

<img src="/images/replicaset.svg">
(Apuntes)
Las Lecturas y Escrituras del cliente se hacen al Primary.


El primary replica todo en los secundarios, cada que escribo en el primario escribe en los secundarios con una conexión Asíncrona, para que en caso que el primario se caiga por cualquier razon, automaticamente uno de los dos secundarios se levanta como primario con algun criterio establecido tarda de 10 - 90 seg. Si se levanta el primario caido vuelve a ser el primario.

Planteamientos:

* Algunos implementan el primario y secundario en el mismo lugar fisico así la asíncronia mínima y el otro secundario lo hago en otra ubicación fisica o en la nube.

* Todo a la nube, Azure, Amazon. Puedo desplegar maquinas con Mongo con Comunity o Enterprise(Soporte, Ops Manager). No hay un despliegue 

* Atlas. Todo automatizado. Con soporte especifico. No tienen sus granjas propias.(Herramienta Clust Manager)

### Automatic failover (conmutación automática)

[Automatic Failover](https://docs.mongodb.com/manual/replication/index.html#automatic-failover)

1. Cuando el primario no se comunica con los otros miembros por más de 10s (valor por defecto de `electionTimeoutMillis`) uno de los secundarios elegibles lanza el proceso de elecciones nominandose a si mismo.

2. Mientras se producen las elecciones se interumpen las operaciones de escritura al cluster.

3. Las operaciones de lectura se podrían mantener durante las elecciones, pero para ello, alguno de los miembros secundarios debe estar configurado para aceptar operaciones de lectura.

4. El tiempo medio de elecciones será unos 12 segundos, pero se podría rebajar configurando el valor de `electionTimeoutMillis` pero el problema podría surgir si se producen pequeñas interrupciones del primario que provocarían elecciones innecesarias.

5. Una vez resueltas las elecciones, el nuevo primario establece las operaciones de escritura y lectura y también a los miembros vivos.

6. Una vez que se recupere el servidor caído se puede incorporar al cluster como secundario para recuperar la información generada desde su falla y posteriormente volver a ser primario o no en función de nuestra configuración.

7. La lógica de las aplicaciones que se conectan al cluster debera incorporar tolerancoa a este `automatic failover` así cp,p a las elecciones. Algunos drivers proporcionan esta tolerancia de manera nativa y configurable.


Notas:

* Si incorporamos un server vacio prmero le hago la copia del backup y luego lo incorporo.
* Se pueden llegar a perder datos cuando hay una caida del primario aun que sea de milisegundos si hay un alto nivel de escritura. (Existe un Rollback para hacerlo manualmente), hay que implementar reconocimiento de escritura (la escritura no es valida hasta estar escrito o replicado en los secundarios, el problema es que la operacion de escritura es más larga, por eso suele haber el primario y secundario en el mismo centro de datos)
* Vale más la percistencia del dato que la velocidad de su manejo.
* El cluster permite tener hasta 50 servidores, pero cuando solo hay 3 sus caracteristicas deberían ser las mismas.

### Despliegue de Replica Set

En local solo tenemos 1 instancia, lo simularemos con puertos.

1. Crear 3 directorios de datos para cada servidor del cluster

2. Levantar servidores miembros de un replicate set.

   ```
   mongod --replSet <nombre-cluster> --dbpath <ruta-directorio>
          --bind_ip <ip-o-dominio>
          --port <numero-puerto>
   ```
   
   AWS
   ip 142.23.7.12  => mejor Dominio
   port 27027
   
3. Conectar la shell a cualquiera de los servidores levantados:

   * Crear un objeto de configuración.
   * Pasar ese objeto con el método `rs.initiate(<objeto-config>)`


Notas:

* Si fueran 3 maquinas diferentes en las 3 tendría que instalar Mongo.


Archivo de configuración de MongoDB `mongod.conf`.

```
# mongod.conf

# for documentation of all options, see:
#   http://docs.mongodb.org/manual/reference/configuration-options/

# Where and how to store data.
storage:
  dbPath: %MONGO_DATA_PATH%
  journal:
    enabled: true
#  engine:
#  mmapv1:
#  wiredTiger:

# where to write logging data.
systemLog:
  destination: file
  logAppend: true
  path:  %MONGO_LOG_PATH%\mongod.log

# network interfaces
net:
  port: 27017
  bindIp: 127.0.0.1


#processManagement:

#security:

#operationProfiling:

#replication:

#sharding:

## Enterprise-Only Options:

#auditLog:

#snmp:

```

Vamos a crear 3 subdirectorios para levantarlos en otros puertos diferentes al de default.

Desde el prompt creamos los siguientes directorios:

```sh
C:\Users\manana> md data\server1
C:\Users\manana> md data\server2
C:\Users\manana> md data\server3
```

Levantar los servidores en 3 consolas diferentes:

```sh
C:\Users\manana>mongod --replSet clusterGetafe --dbpath data\server1 --port 27017

C:\Users\manana>mongod --replSet clusterGetafe --dbpath data\server2 --port 27018

C:\Users\manana>mongod --replSet clusterGetafe --dbpath data\server3 --port 27019
```

Al arrancar los servidores las carpetas se llenan `C:\Users\manana\data` para cada servidor `server1`, `server2` y `server3`.

Hacer la conección a alguno de ellos:

```sh
mongo --port 27017
```

Esto arranca la shell de este puerto, lo podemos ver en el título de la ventana de la shell `mongo --port 27017`:

<img src="/images/consola.png">

Si ejecutamos el comando `show dbs` nos marca un error, es por que no esta configurado aún.

Con JS establecemos los parametros de conexión, más sencillos que podemos tener:

```sh
> rsconfig = {
... _id: "clusterGetafe",
... members: [
... {_id: 0, host: "localhost:27017"},
... {_id: 1, host: "localhost:27018"},
... {_id: 2, host: "localhost:27019"}
... ]
... }
{
        "_id" : "clusterGetafe",
        "members" : [
                {
                        "_id" : 0,
                        "host" : "localhost:27017"
                },
                {
                        "_id" : 1,
                        "host" : "localhost:27018"
                },
                {
                        "_id" : 2,
                        "host" : "localhost:27019"
                }
        ]
}
>                                                                                     
```

Pasar este objeto al método `rs.initiate()`:

```sh
> rs.initiate(rsconfig)
{
        "ok" : 1,
        "$clusterTime" : {
                "clusterTime" : Timestamp(1581421304, 1),
                "signature" : {
                        "hash" : BinData(0,"AAAAAAAAAAAAAAAAAAAAAAAAAAA="),
                        "keyId" : NumberLong(0)
                }
        },
        "operationTime" : Timestamp(1581421304, 1)
}
clusterGetafe:SECONDARY>      
```

Observemos como el prompt a cambiado a `clusterGetafe:SECONDARY>`.
El algoritmo elige cual de ellos sera el Primario.
Esto se conecta con los otros dos servidores y le pasa la información de configuración y se conectan entre ellos.
Una vez conectados en función del algoritmo elige el Primario, debemos comprobar cual es el primario. 

Con el comando `rs.status()` vemos información de los miembros, entre ella el  `"stateStr"` que indica si es Primario o Secundario: 

```sh
clusterGetafe:SECONDARY> rs.status()
{
        "set" : "clusterGetafe",
        "date" : ISODate("2020-02-11T11:51:18.067Z"),
        "myState" : 1,
        "term" : NumberLong(1),
        "syncingTo" : "",
        "syncSourceHost" : "",
        "syncSourceId" : -1,
        "heartbeatIntervalMillis" : NumberLong(2000),
        "majorityVoteCount" : 2,
        "writeMajorityCount" : 2,
        "optimes" : {
                "lastCommittedOpTime" : {
                        "ts" : Timestamp(1581421876, 1),
                        "t" : NumberLong(1)
                },
                "lastCommittedWallTime" : ISODate("2020-02-11T11:51:16.784Z"),
                "readConcernMajorityOpTime" : {
                        "ts" : Timestamp(1581421876, 1),
                        "t" : NumberLong(1)
                },
                "readConcernMajorityWallTime" : ISODate("2020-02-11T11:51:16.784Z"),
                "appliedOpTime" : {
                        "ts" : Timestamp(1581421876, 1),
                        "t" : NumberLong(1)
                },
                "durableOpTime" : {
                        "ts" : Timestamp(1581421876, 1),
                        "t" : NumberLong(1)
                },
                "lastAppliedWallTime" : ISODate("2020-02-11T11:51:16.784Z"),
                "lastDurableWallTime" : ISODate("2020-02-11T11:51:16.784Z")
        },
        "lastStableRecoveryTimestamp" : Timestamp(1581421856, 1),
        "lastStableCheckpointTimestamp" : Timestamp(1581421856, 1),
        "electionCandidateMetrics" : {
                "lastElectionReason" : "electionTimeout",
                "lastElectionDate" : ISODate("2020-02-11T11:41:55.748Z"),
                "electionTerm" : NumberLong(1),
                "lastCommittedOpTimeAtElection" : {
                        "ts" : Timestamp(0, 0),
                        "t" : NumberLong(-1)
                },
                "lastSeenOpTimeAtElection" : {
                        "ts" : Timestamp(1581421304, 1),
                        "t" : NumberLong(-1)
                },
                "numVotesNeeded" : 2,
                "priorityAtElection" : 1,
                "electionTimeoutMillis" : NumberLong(10000),
                "numCatchUpOps" : NumberLong(0),
                "newTermStartDate" : ISODate("2020-02-11T11:41:56.735Z"),
                "wMajorityWriteAvailabilityDate" : ISODate("2020-02-11T11:41:57.618Z")
        },
        "members" : [
                {
                        "_id" : 0,
                        "name" : "localhost:27017",
                        "ip" : "127.0.0.1",
                        "health" : 1,
                        "state" : 1,
                        "stateStr" : "PRIMARY",
                        "uptime" : 2020,
                        "optime" : {
                                "ts" : Timestamp(1581421876, 1),
                                "t" : NumberLong(1)
                        },
                        "optimeDate" : ISODate("2020-02-11T11:51:16Z"),
                        "syncingTo" : "",
                        "syncSourceHost" : "",
                        "syncSourceId" : -1,
                        "infoMessage" : "",
                        "electionTime" : Timestamp(1581421315, 1),
                        "electionDate" : ISODate("2020-02-11T11:41:55Z"),
                        "configVersion" : 1,
                        "self" : true,
                        "lastHeartbeatMessage" : ""
                },
                {
                        "_id" : 1,
                        "name" : "localhost:27018",
                        "ip" : "127.0.0.1",
                        "health" : 1,
                        "state" : 2,
                        "stateStr" : "SECONDARY",
                        "uptime" : 573,
                        "optime" : {
                                "ts" : Timestamp(1581421876, 1),
                                "t" : NumberLong(1)
                        },
                        "optimeDurable" : {
                                "ts" : Timestamp(1581421876, 1),
                                "t" : NumberLong(1)
                        },
                        "optimeDate" : ISODate("2020-02-11T11:51:16Z"),
                        "optimeDurableDate" : ISODate("2020-02-11T11:51:16Z"),
                        "lastHeartbeat" : ISODate("2020-02-11T11:51:17.951Z"),
                        "lastHeartbeatRecv" : ISODate("2020-02-11T11:51:17.588Z"),
                        "pingMs" : NumberLong(0),
                        "lastHeartbeatMessage" : "",
                        "syncingTo" : "localhost:27017",
                        "syncSourceHost" : "localhost:27017",
                        "syncSourceId" : 0,
                        "infoMessage" : "",
                        "configVersion" : 1
                },
                {
                        "_id" : 2,
                        "name" : "localhost:27019",
                        "ip" : "127.0.0.1",
                        "health" : 1,
                        "state" : 2,
                        "stateStr" : "SECONDARY",
                        "uptime" : 573,
                        "optime" : {
                                "ts" : Timestamp(1581421876, 1),
                                "t" : NumberLong(1)
                        },
                        "optimeDurable" : {
                                "ts" : Timestamp(1581421876, 1),
                                "t" : NumberLong(1)
                        },
                        "optimeDate" : ISODate("2020-02-11T11:51:16Z"),
                        "optimeDurableDate" : ISODate("2020-02-11T11:51:16Z"),
                        "lastHeartbeat" : ISODate("2020-02-11T11:51:17.951Z"),
                        "lastHeartbeatRecv" : ISODate("2020-02-11T11:51:17.587Z"),
                        "pingMs" : NumberLong(0),
                        "lastHeartbeatMessage" : "",
                        "syncingTo" : "localhost:27017",
                        "syncSourceHost" : "localhost:27017",
                        "syncSourceId" : 0,
                        "infoMessage" : "",
                        "configVersion" : 1
                }
        ],
        "ok" : 1,
        "$clusterTime" : {
                "clusterTime" : Timestamp(1581421876, 1),
                "signature" : {
                        "hash" : BinData(0,"AAAAAAAAAAAAAAAAAAAAAAAAAAA="),
                        "keyId" : NumberLong(0)
                }
        },
        "operationTime" : Timestamp(1581421876, 1)
}
clusterGetafe:PRIMARY>                                                                                                
```
El prompt a cambiado a `clusterGetafe:PRIMARY>`.

Me indica el estado de los otros servidores indicando que son secundarios ` "stateStr" : "SECONDARY",`.

El objeto `rs` es un objeto que representa el `replicatSet`.

[Replication Reference](https://docs.mongodb.com/manual/reference/replication/)


**Si la cago en algún paso borro el contenido de las carpetas `server1, server2 server3` y comienzo de nuevo**

Otra manera de lanzar información es con el comando `rs.isMaster()`:

```sh
clusterGetafe:PRIMARY> rs.isMaster()
{
        "hosts" : [
                "localhost:27017",
                "localhost:27018",
                "localhost:27019"
        ],
        "setName" : "clusterGetafe",
        "setVersion" : 1,
        "ismaster" : true,
        "secondary" : false,
        "primary" : "localhost:27017",
        "me" : "localhost:27017",
        "electionId" : ObjectId("7fffffff0000000000000001"),
        "lastWrite" : {
                "opTime" : {
                        "ts" : Timestamp(1581423306, 1),
                        "t" : NumberLong(1)
                },
                "lastWriteDate" : ISODate("2020-02-11T12:15:06Z"),
                "majorityOpTime" : {
                        "ts" : Timestamp(1581423306, 1),
                        "t" : NumberLong(1)
                },
                "majorityWriteDate" : ISODate("2020-02-11T12:15:06Z")
        },
        "maxBsonObjectSize" : 16777216,
        "maxMessageSizeBytes" : 48000000,
        "maxWriteBatchSize" : 100000,
        "localTime" : ISODate("2020-02-11T12:15:16.474Z"),
        "logicalSessionTimeoutMinutes" : 30,
        "connectionId" : 2,
        "minWireVersion" : 0,
        "maxWireVersion" : 8,
        "readOnly" : false,
        "ok" : 1,
        "$clusterTime" : {
                "clusterTime" : Timestamp(1581423306, 1),
                "signature" : {
                        "hash" : BinData(0,"AAAAAAAAAAAAAAAAAAAAAAAAAAA="),
                        "keyId" : NumberLong(0)
                }
        },
        "operationTime" : Timestamp(1581423306, 1)
}
clusterGetafe:PRIMARY> 

```
En el fondo te da la misma información presentada de forma diferente.



Si me conecto al puerto 27019 e intento insertar algo me marca error:

```sh
clusterGetafe:SECONDARY> use getafeTest
switched to db getafeTest
clusterGetafe:SECONDARY> db.foo.insert({ a: 1})
WriteCommandError({
        "operationTime" : Timestamp(1581423616, 1),
        "ok" : 0,
        "errmsg" : "not master",
        "code" : 10107,
        "codeName" : "NotMaster",
        "$clusterTime" : {
                "clusterTime" : Timestamp(1581423616, 1),
                "signature" : {
                        "hash" : BinData(0,"AAAAAAAAAAAAAAAAAAAAAAAAAAA="),
                        "keyId" : NumberLong(0)
                }
        }
})
clusterGetafe:SECONDARY>
```

En el primario, creamos una BD y una colección con 1000 registros:

```sh
clusterGetafe:PRIMARY> use getafeTest
switched to db getafeTest

clusterGetafe:PRIMARY> for(i=0; i < 1000; i++){
... db.foo.insert({a: i})
... }
WriteResult({ "nInserted" : 1 })

clusterGetafe:PRIMARY> db.foo.find().count()
1000
clusterGetafe:PRIMARY>  
```

Las operaciones de escritura se escriben en forma de idenpotencia en los otros `oplog` de los otros servidores y las ejecuta. Esto lo hace así por si algún servidor no esta activo y posteriormente entra, lo ejecuta de forma Idempotente, esto es para que solo las ejecute igual. Esto garantiza que todos los servidores aun que sea con retraso.

`oplog` Log de operaciones almacena las instrucciones de forma idempotente y luego el servidor Secundario las toma de aquí y las ejecuta. Cada servidor tiene su `oplog`.

IDEMPOTENTE lo ejecuta la primera vez y después ya no hace nada.

Si me voy a mi secundario no puedo leerlo si quiero ejecutar ``show dbs` para ver si efectivamente se ha replicado.

La forma que tengo es exportar los datos.


Abrimos una consola nueva tengo `mongoexport`.

```
C:\Users\manana>mongoexport --port 27019 --collection=foo --db=getafeTest
....
{"_id":{"$oid":"5e429ce101c51fddb7887272"},"a":998.0}
{"_id":{"$oid":"5e429ce101c51fddb7887273"},"a":999.0}
2020-02-11T13:39:42.550+0100    exported 1000 records

C:\Users\manana>mongoexport --port 27019 --collection=foo --db=getafeTest --out=foo.json
2020-02-11T13:41:48.801+0100    connected to: mongodb://localhost:27019/
2020-02-11T13:41:48.895+0100    exported 1000 records
```
En el primer comando lo muestra en la pantalla. Me exporta los 1000 registros en un archivo foo.json en la ruta donde se ejecuto el comando `C:\Users\manana`.


El secundario no es para leer es para replicar.
El primario es el unico que puede recibir las peticiones de lectura y escritura.

**El sarding permite crecer horizontalmente, por que ahora solo podemos hacer un escalado vertical.** :skull:

* **Resumen**
   * Cluster
      * Un solo primario (recibe escrituras y lecturas)
      * Secundario donde se replican los datos a través de la colección `oplog`
      
  ```sh
  mongod --dbpath <ruta>
         --port <puerto>
         --replSet <nombre-cluster> //Esto indica que esta en un cluster (replicaset) sino lo pongo el servidor es independiente
  ```
  
  ```sh
  mongod --dbpath data\serve3 --port 27019
  ```
  
Levanto mis servidores:
  
```sh
C:\Users\manana>mongod --replSet clusterGetafe --dbpath data\server1 --port 27017

C:\Users\manana>mongod --replSet clusterGetafe --dbpath data\server2 --port 27018

C:\Users\manana>mongod --replSet clusterGetafe --dbpath data\server3 --port 27019
```

Hacer la conección a alguno de ellos:

```sh
mongo --port 27017
```

```sh
clusterGetafe:PRIMARY> rs.status()
{
        "set" : "clusterGetafe",
        "date" : ISODate("2020-02-12T08:41:21.706Z"),
        "myState" : 1,
        "term" : NumberLong(3),
        "syncingTo" : "",
        "syncSourceHost" : "",
        "syncSourceId" : -1,
        "heartbeatIntervalMillis" : NumberLong(2000),
        "majorityVoteCount" : 2,
        "writeMajorityCount" : 2,
        "optimes" : {
                "lastCommittedOpTime" : {
                        "ts" : Timestamp(1581496874, 1),
                        "t" : NumberLong(3)
                },
                "lastCommittedWallTime" : ISODate("2020-02-12T08:41:14.670Z"),
                "readConcernMajorityOpTime" : {
                        "ts" : Timestamp(1581496874, 1),
                        "t" : NumberLong(3)
                },
                "readConcernMajorityWallTime" : ISODate("2020-02-12T08:41:14.670Z"),
                "appliedOpTime" : {
                        "ts" : Timestamp(1581496874, 1),
                        "t" : NumberLong(3)
                },
                "durableOpTime" : {
                        "ts" : Timestamp(1581496874, 1),
                        "t" : NumberLong(3)
                },
                "lastAppliedWallTime" : ISODate("2020-02-12T08:41:14.670Z"),
                "lastDurableWallTime" : ISODate("2020-02-12T08:41:14.670Z")
        },
        "lastStableRecoveryTimestamp" : Timestamp(1581496834, 1),
        "lastStableCheckpointTimestamp" : Timestamp(1581496834, 1),
        "electionCandidateMetrics" : {
                "lastElectionReason" : "electionTimeout",
                "lastElectionDate" : ISODate("2020-02-12T08:32:42.190Z"),
                "electionTerm" : NumberLong(3),
                "lastCommittedOpTimeAtElection" : {
                        "ts" : Timestamp(0, 0),
                        "t" : NumberLong(-1)
                },
                "lastSeenOpTimeAtElection" : {
                        "ts" : Timestamp(1581425591, 1),
                        "t" : NumberLong(2)
                },
                "numVotesNeeded" : 2,
                "priorityAtElection" : 1,
                "electionTimeoutMillis" : NumberLong(10000),
                "numCatchUpOps" : NumberLong(0),
                "newTermStartDate" : ISODate("2020-02-12T08:32:44.629Z"),
                "wMajorityWriteAvailabilityDate" : ISODate("2020-02-12T08:32:44.742Z")
        },
        "members" : [
                {
                        "_id" : 0,
                        "name" : "localhost:27017",
                        "ip" : "127.0.0.1",
                        "health" : 1,
                        "state" : 1,
                        "stateStr" : "PRIMARY",
                        "uptime" : 583,
                        "optime" : {
                                "ts" : Timestamp(1581496874, 1),
                                "t" : NumberLong(3)
                        },
                        "optimeDate" : ISODate("2020-02-12T08:41:14Z"),
                        "syncingTo" : "",
                        "syncSourceHost" : "",
                        "syncSourceId" : -1,
                        "infoMessage" : "",
                        "electionTime" : Timestamp(1581496362, 1),
                        "electionDate" : ISODate("2020-02-12T08:32:42Z"),
                        "configVersion" : 1,
                        "self" : true,
                        "lastHeartbeatMessage" : ""
                },
                {
                        "_id" : 1,
                        "name" : "localhost:27018",
                        "ip" : "127.0.0.1",
                        "health" : 1,
                        "state" : 2,
                        "stateStr" : "SECONDARY",
                        "uptime" : 529,
                        "optime" : {
                                "ts" : Timestamp(1581496874, 1),
                                "t" : NumberLong(3)
                        },
                        "optimeDurable" : {
                                "ts" : Timestamp(1581496874, 1),
                                "t" : NumberLong(3)
                        },
                        "optimeDate" : ISODate("2020-02-12T08:41:14Z"),
                        "optimeDurableDate" : ISODate("2020-02-12T08:41:14Z"),
                        "lastHeartbeat" : ISODate("2020-02-12T08:41:20.351Z"),
                        "lastHeartbeatRecv" : ISODate("2020-02-12T08:41:21.334Z"),
                        "pingMs" : NumberLong(0),
                        "lastHeartbeatMessage" : "",
                        "syncingTo" : "localhost:27017",
                        "syncSourceHost" : "localhost:27017",
                        "syncSourceId" : 0,
                        "infoMessage" : "",
                        "configVersion" : 1
                },
                {
                        "_id" : 2,
                        "name" : "localhost:27019",
                        "ip" : "127.0.0.1",
                        "health" : 1,
                        "state" : 2,
                        "stateStr" : "SECONDARY",
                        "uptime" : 430,
                        "optime" : {
                                "ts" : Timestamp(1581496874, 1),
                                "t" : NumberLong(3)
                        },
                        "optimeDurable" : {
                                "ts" : Timestamp(1581496874, 1),
                                "t" : NumberLong(3)
                        },
                        "optimeDate" : ISODate("2020-02-12T08:41:14Z"),
                        "optimeDurableDate" : ISODate("2020-02-12T08:41:14Z"),
                        "lastHeartbeat" : ISODate("2020-02-12T08:41:21.631Z"),
                        "lastHeartbeatRecv" : ISODate("2020-02-12T08:41:21.087Z"),
                        "pingMs" : NumberLong(0),
                        "lastHeartbeatMessage" : "",
                        "syncingTo" : "localhost:27018",
                        "syncSourceHost" : "localhost:27018",
                        "syncSourceId" : 1,
                        "infoMessage" : "",
                        "configVersion" : 1
                }
        ],
        "ok" : 1,
        "$clusterTime" : {
                "clusterTime" : Timestamp(1581496874, 1),
                "signature" : {
                        "hash" : BinData(0,"AAAAAAAAAAAAAAAAAAAAAAAAAAA="),
                        "keyId" : NumberLong(0)
                }
        },
        "operationTime" : Timestamp(1581496874, 1)
}
clusterGetafe:PRIMARY>
```

Si me deja de funcionar el servidor primario (Lo cierro), ya no me deja hacer nada.

Enmtro al `mongo --27018` y este pasa a ser mi primario.

```sh
clusterGetafe:PRIMARY> rs.status()
{
        "set" : "clusterGetafe",
        "date" : ISODate("2020-02-12T08:43:46.217Z"),
        "myState" : 1,
        "term" : NumberLong(4),
        "syncingTo" : "",
        "syncSourceHost" : "",
        "syncSourceId" : -1,
        "heartbeatIntervalMillis" : NumberLong(2000),
        "majorityVoteCount" : 2,
        "writeMajorityCount" : 2,
        "optimes" : {
                "lastCommittedOpTime" : {
                        "ts" : Timestamp(1581497024, 1),
                        "t" : NumberLong(4)
                },
                "lastCommittedWallTime" : ISODate("2020-02-12T08:43:44.730Z"),
                "readConcernMajorityOpTime" : {
                        "ts" : Timestamp(1581497024, 1),
                        "t" : NumberLong(4)
                },
                "readConcernMajorityWallTime" : ISODate("2020-02-12T08:43:44.730Z"),
                "appliedOpTime" : {
                        "ts" : Timestamp(1581497024, 1),
                        "t" : NumberLong(4)
                },
                "durableOpTime" : {
                        "ts" : Timestamp(1581497024, 1),
                        "t" : NumberLong(4)
                },
                "lastAppliedWallTime" : ISODate("2020-02-12T08:43:44.730Z"),
                "lastDurableWallTime" : ISODate("2020-02-12T08:43:44.730Z")
        },
        "lastStableRecoveryTimestamp" : Timestamp(1581497004, 1),
        "lastStableCheckpointTimestamp" : Timestamp(1581497004, 1),
        "electionCandidateMetrics" : {
                "lastElectionReason" : "stepUpRequestSkipDryRun",
                "lastElectionDate" : ISODate("2020-02-12T08:42:54.288Z"),
                "electionTerm" : NumberLong(4),
                "lastCommittedOpTimeAtElection" : {
                        "ts" : Timestamp(1581496964, 1),
                        "t" : NumberLong(3)
                },
                "lastSeenOpTimeAtElection" : {
                        "ts" : Timestamp(1581496964, 1),
                        "t" : NumberLong(3)
                },
                "numVotesNeeded" : 2,
                "priorityAtElection" : 1,
                "electionTimeoutMillis" : NumberLong(10000),
                "priorPrimaryMemberId" : 0,
                "numCatchUpOps" : NumberLong(0),
                "newTermStartDate" : ISODate("2020-02-12T08:42:54.725Z"),
                "wMajorityWriteAvailabilityDate" : ISODate("2020-02-12T08:42:54.781Z")
        },
        "electionParticipantMetrics" : {
                "votedForCandidate" : true,
                "electionTerm" : NumberLong(3),
                "lastVoteDate" : ISODate("2020-02-12T08:32:42.232Z"),
                "electionCandidateMemberId" : 0,
                "voteReason" : "",
                "lastAppliedOpTimeAtElection" : {
                        "ts" : Timestamp(1581425591, 1),
                        "t" : NumberLong(2)
                },
                "maxAppliedOpTimeInSet" : {
                        "ts" : Timestamp(1581425591, 1),
                        "t" : NumberLong(2)
                },
                "priorityAtElection" : 1
        },
        "members" : [
                {
                        "_id" : 0,
                        "name" : "localhost:27017",
                        "ip" : "127.0.0.1",
                        "health" : 0,
                        "state" : 8,
                        "stateStr" : "(not reachable/healthy)",
                        "uptime" : 0,
                        "optime" : {
                                "ts" : Timestamp(0, 0),
                                "t" : NumberLong(-1)
                        },
                        "optimeDurable" : {
                                "ts" : Timestamp(0, 0),
                                "t" : NumberLong(-1)
                        },
                        "optimeDate" : ISODate("1970-01-01T00:00:00Z"),
                        "optimeDurableDate" : ISODate("1970-01-01T00:00:00Z"),
                        "lastHeartbeat" : ISODate("2020-02-12T08:43:43.727Z"),
                        "lastHeartbeatRecv" : ISODate("2020-02-12T08:42:54.874Z"),
                        "pingMs" : NumberLong(0),
                        "lastHeartbeatMessage" : "Error connecting to localhost:27017 (127.0.0.1:27017) :: caused by :: No se puede establecer una conexi�n ya que el equipo de destino deneg� expresamente dicha conexi�n.",
                        "syncingTo" : "",
                        "syncSourceHost" : "",
                        "syncSourceId" : -1,
                        "infoMessage" : "",
                        "configVersion" : -1
                },
                {
                        "_id" : 1,
                        "name" : "localhost:27018",
                        "ip" : "127.0.0.1",
                        "health" : 1,
                        "state" : 1,
                        "stateStr" : "PRIMARY",
                        "uptime" : 679,
                        "optime" : {
                                "ts" : Timestamp(1581497024, 1),
                                "t" : NumberLong(4)
                        },
                        "optimeDate" : ISODate("2020-02-12T08:43:44Z"),
                        "syncingTo" : "",
                        "syncSourceHost" : "",
                        "syncSourceId" : -1,
                        "infoMessage" : "",
                        "electionTime" : Timestamp(1581496974, 1),
                        "electionDate" : ISODate("2020-02-12T08:42:54Z"),
                        "configVersion" : 1,
                        "self" : true,
                        "lastHeartbeatMessage" : ""
                },
                {
                        "_id" : 2,
                        "name" : "localhost:27019",
                        "ip" : "127.0.0.1",
                        "health" : 1,
                        "state" : 2,
                        "stateStr" : "SECONDARY",
                        "uptime" : 574,
                        "optime" : {
                                "ts" : Timestamp(1581497014, 1),
                                "t" : NumberLong(4)
                        },
                        "optimeDurable" : {
                                "ts" : Timestamp(1581497014, 1),
                                "t" : NumberLong(4)
                        },
                        "optimeDate" : ISODate("2020-02-12T08:43:34Z"),
                        "optimeDurableDate" : ISODate("2020-02-12T08:43:34Z"),
                        "lastHeartbeat" : ISODate("2020-02-12T08:43:44.384Z"),
                        "lastHeartbeatRecv" : ISODate("2020-02-12T08:43:45.832Z"),
                        "pingMs" : NumberLong(0),
                        "lastHeartbeatMessage" : "",
                        "syncingTo" : "localhost:27018",
                        "syncSourceHost" : "localhost:27018",
                        "syncSourceId" : 1,
                        "infoMessage" : "",
                        "configVersion" : 1
                }
        ],
        "ok" : 1,
        "$clusterTime" : {
                "clusterTime" : Timestamp(1581497024, 1),
                "signature" : {
                        "hash" : BinData(0,"AAAAAAAAAAAAAAAAAAAAAAAAAAA="),
                        "keyId" : NumberLong(0)
                }
        },
        "operationTime" : Timestamp(1581497024, 1)
}
clusterGetafe:PRIMARY>
```
Como vemos el `27017` no goza de buena salud `"health" : 0,`.

Las peticiones escritura y lectura se haran sobre el `27018`.

Si inserto algo aquí:

```sh
clusterGetafe:PRIMARY> use getafeTest
switched to db getafeTest

clusterGetafe:PRIMARY> db.biblioteca.insert(
... {titulo: "El Quijote", autor: "Miguel de Cervantes" }
... )
WriteResult({ "nInserted" : 1 })
clusterGetafe:PRIMARY>
>
```

Si levanto el `27019` por defecto no dejan hacer operaciones de lectura, pero hay un comando


```sh
clusterGetafe:SECONDARY> use getafeTest
switched to db getafeTest
clusterGetafe:SECONDARY> show dbs
2020-02-12T09:50:53.803+0100 E  QUERY    [js] uncaught exception: Error: listDatabases failed:{
        "operationTime" : Timestamp(1581497444, 1),
        "ok" : 0,
        "errmsg" : "not master and slaveOk=false",
        "code" : 13435,
        "codeName" : "NotMasterNoSlaveOk",
        "$clusterTime" : {
                "clusterTime" : Timestamp(1581497444, 1),
                "signature" : {
                        "hash" : BinData(0,"AAAAAAAAAAAAAAAAAAAAAAAAAAA="),
                        "keyId" : NumberLong(0)
                }
        }
} :
_getErrorWithCode@src/mongo/shell/utils.js:25:13
Mongo.prototype.getDBs/<@src/mongo/shell/mongo.js:135:19
Mongo.prototype.getDBs@src/mongo/shell/mongo.js:87:12
shellHelper.show@src/mongo/shell/utils.js:906:13
shellHelper@src/mongo/shell/utils.js:790:15
@(shellhelp2):1:1


clusterGetafe:SECONDARY> db.setSlaveOk()
```

Con este comando ya puedo hacer operaciones de lectura como vemos:


```sh
clusterGetafe:SECONDARY> show collections
biblioteca
foo
clusterGetafe:SECONDARY> db.foo.find()
{ "_id" : ObjectId("5e429ce001c51fddb7886e8e"), "a" : 2 }
{ "_id" : ObjectId("5e429ce001c51fddb7886e8f"), "a" : 3 }
{ "_id" : ObjectId("5e429ce001c51fddb7886e97"), "a" : 11 }
{ "_id" : ObjectId("5e429ce001c51fddb7886ea3"), "a" : 23 }
{ "_id" : ObjectId("5e429ce001c51fddb7886ea2"), "a" : 22 }
{ "_id" : ObjectId("5e429ce001c51fddb7886ea8"), "a" : 28 }
{ "_id" : ObjectId("5e429ce001c51fddb7886e90"), "a" : 4 }
{ "_id" : ObjectId("5e429ce001c51fddb7886ea6"), "a" : 26 }
{ "_id" : ObjectId("5e429ce001c51fddb7886e95"), "a" : 9 }
{ "_id" : ObjectId("5e429ce001c51fddb7886e94"), "a" : 8 }
{ "_id" : ObjectId("5e429ce001c51fddb7886e8d"), "a" : 1 }
{ "_id" : ObjectId("5e429ce001c51fddb7886e98"), "a" : 12 }
{ "_id" : ObjectId("5e429ce001c51fddb7886ea9"), "a" : 29 }
{ "_id" : ObjectId("5e429ce001c51fddb7886ea0"), "a" : 20 }
{ "_id" : ObjectId("5e429ce001c51fddb7886e9b"), "a" : 15 }
{ "_id" : ObjectId("5e429ce001c51fddb7886ea1"), "a" : 21 }
{ "_id" : ObjectId("5e429ce001c51fddb7886eab"), "a" : 31 }
{ "_id" : ObjectId("5e429ce001c51fddb7886e9a"), "a" : 14 }
{ "_id" : ObjectId("5e429ce001c51fddb7886ea5"), "a" : 25 }
{ "_id" : ObjectId("5e429ce001c51fddb7886eaa"), "a" : 30 }
Type "it" for more
clusterGetafe:SECONDARY>      
```

Por que quisiera leer de un secundario.

* Situación geografica
* Trafico de Big Data
* Lecturas Quioscos Burguer King

Para quitarlo `db.setSlaveOk(false)`
```sh
clusterGetafe:SECONDARY> db.setSlaveOk
function(value) {
    if (value == undefined)
        value = true;
    this._slaveOk = value;
}
clusterGetafe:SECONDARY>  
```

Levanto nuevamente el `27017`

```sh
clusterGetafe:PRIMARY> rs.status()
{
        "set" : "clusterGetafe",
        "date" : ISODate("2020-02-12T09:05:02.481Z"),
        "myState" : 1,
        "term" : NumberLong(4),
        "syncingTo" : "",
        "syncSourceHost" : "",
        "syncSourceId" : -1,
        "heartbeatIntervalMillis" : NumberLong(2000),
        "majorityVoteCount" : 2,
        "writeMajorityCount" : 2,
        "optimes" : {
                "lastCommittedOpTime" : {
                        "ts" : Timestamp(1581498294, 1),
                        "t" : NumberLong(4)
                },
                "lastCommittedWallTime" : ISODate("2020-02-12T09:04:54.819Z"),
                "readConcernMajorityOpTime" : {
                        "ts" : Timestamp(1581498294, 1),
                        "t" : NumberLong(4)
                },
                "readConcernMajorityWallTime" : ISODate("2020-02-12T09:04:54.819Z"),
                "appliedOpTime" : {
                        "ts" : Timestamp(1581498294, 1),
                        "t" : NumberLong(4)
                },
                "durableOpTime" : {
                        "ts" : Timestamp(1581498294, 1),
                        "t" : NumberLong(4)
                },
                "lastAppliedWallTime" : ISODate("2020-02-12T09:04:54.819Z"),
                "lastDurableWallTime" : ISODate("2020-02-12T09:04:54.819Z")
        },
        "lastStableRecoveryTimestamp" : Timestamp(1581498274, 1),
        "lastStableCheckpointTimestamp" : Timestamp(1581498274, 1),
        "electionCandidateMetrics" : {
                "lastElectionReason" : "stepUpRequestSkipDryRun",
                "lastElectionDate" : ISODate("2020-02-12T08:42:54.288Z"),
                "electionTerm" : NumberLong(4),
                "lastCommittedOpTimeAtElection" : {
                        "ts" : Timestamp(1581496964, 1),
                        "t" : NumberLong(3)
                },
                "lastSeenOpTimeAtElection" : {
                        "ts" : Timestamp(1581496964, 1),
                        "t" : NumberLong(3)
                },
                "numVotesNeeded" : 2,
                "priorityAtElection" : 1,
                "electionTimeoutMillis" : NumberLong(10000),
                "priorPrimaryMemberId" : 0,
                "numCatchUpOps" : NumberLong(0),
                "newTermStartDate" : ISODate("2020-02-12T08:42:54.725Z"),
                "wMajorityWriteAvailabilityDate" : ISODate("2020-02-12T08:42:54.781Z")
        },
        "electionParticipantMetrics" : {
                "votedForCandidate" : true,
                "electionTerm" : NumberLong(3),
                "lastVoteDate" : ISODate("2020-02-12T08:32:42.232Z"),
                "electionCandidateMemberId" : 0,
                "voteReason" : "",
                "lastAppliedOpTimeAtElection" : {
                        "ts" : Timestamp(1581425591, 1),
                        "t" : NumberLong(2)
                },
                "maxAppliedOpTimeInSet" : {
                        "ts" : Timestamp(1581425591, 1),
                        "t" : NumberLong(2)
                },
                "priorityAtElection" : 1
        },
        "members" : [
                {
                        "_id" : 0,
                        "name" : "localhost:27017",
                        "ip" : "127.0.0.1",
                        "health" : 1,
                        "state" : 2,
                        "stateStr" : "SECONDARY",
                        "uptime" : 62,
                        "optime" : {
                                "ts" : Timestamp(1581498294, 1),
                                "t" : NumberLong(4)
                        },
                        "optimeDurable" : {
                                "ts" : Timestamp(1581498294, 1),
                                "t" : NumberLong(4)
                        },
                        "optimeDate" : ISODate("2020-02-12T09:04:54Z"),
                        "optimeDurableDate" : ISODate("2020-02-12T09:04:54Z"),
                        "lastHeartbeat" : ISODate("2020-02-12T09:05:02.299Z"),
                        "lastHeartbeatRecv" : ISODate("2020-02-12T09:05:01.665Z"),
                        "pingMs" : NumberLong(0),
                        "lastHeartbeatMessage" : "",
                        "syncingTo" : "localhost:27019",
                        "syncSourceHost" : "localhost:27019",
                        "syncSourceId" : 2,
                        "infoMessage" : "",
                        "configVersion" : 1
                },
                {
                        "_id" : 1,
                        "name" : "localhost:27018",
                        "ip" : "127.0.0.1",
                        "health" : 1,
                        "state" : 1,
                        "stateStr" : "PRIMARY",
                        "uptime" : 1955,
                        "optime" : {
                                "ts" : Timestamp(1581498294, 1),
                                "t" : NumberLong(4)
                        },
                        "optimeDate" : ISODate("2020-02-12T09:04:54Z"),
                        "syncingTo" : "",
                        "syncSourceHost" : "",
                        "syncSourceId" : -1,
                        "infoMessage" : "",
                        "electionTime" : Timestamp(1581496974, 1),
                        "electionDate" : ISODate("2020-02-12T08:42:54Z"),
                        "configVersion" : 1,
                        "self" : true,
                        "lastHeartbeatMessage" : ""
                },
                {
                        "_id" : 2,
                        "name" : "localhost:27019",
                        "ip" : "127.0.0.1",
                        "health" : 1,
                        "state" : 2,
                        "stateStr" : "SECONDARY",
                        "uptime" : 1850,
                        "optime" : {
                                "ts" : Timestamp(1581498294, 1),
                                "t" : NumberLong(4)
                        },
                        "optimeDurable" : {
                                "ts" : Timestamp(1581498294, 1),
                                "t" : NumberLong(4)
                        },
                        "optimeDate" : ISODate("2020-02-12T09:04:54Z"),
                        "optimeDurableDate" : ISODate("2020-02-12T09:04:54Z"),
                        "lastHeartbeat" : ISODate("2020-02-12T09:05:00.697Z"),
                        "lastHeartbeatRecv" : ISODate("2020-02-12T09:05:02.148Z"),
                        "pingMs" : NumberLong(0),
                        "lastHeartbeatMessage" : "",
                        "syncingTo" : "localhost:27018",
                        "syncSourceHost" : "localhost:27018",
                        "syncSourceId" : 1,
                        "infoMessage" : "",
                        "configVersion" : 1
                }
        ],
        "ok" : 1,
        "$clusterTime" : {
                "clusterTime" : Timestamp(1581498294, 1),
                "signature" : {
                        "hash" : BinData(0,"AAAAAAAAAAAAAAAAAAAAAAAAAAA="),
                        "keyId" : NumberLong(0)
                }
        },
        "operationTime" : Timestamp(1581498294, 1)
}
clusterGetafe:PRIMARY>
```

El `27017` queda ya como secundario, vamos a levantar una consola para ver si replico biblioteca en el `27017`.

```sh
NO ME LO HIZO
```

¿Como determina Mongo cual es el Primario.?

* Si no hace falta cambiarlo lo deja como esta, hay una perdida de tiempo en hacer el cambio
* Si necesitamos que siempre uno sea el Primario por ser mejor maquina (27017) hay que configurarlo.
* En teoria en todos los servidores deben tener la misma capacidad.

### Tolerancia a Fallos :skull:

* Concepto de mayoría (del cluster).
* La mayoría se define por más de la mitad de los miembros del cluster, esten vivos o no.

Para que se produzcan elecciones es necesario que exista mayoria. Esto es debido a que si hubiera una partición de red, la mayoría necesaria para que haya elecciones impedirá que se generen dos primarios al mismo tiempo.


Si tenemos 5 servidores.

3 Madrid
2 Lisboa
Si no se consideraria la mayoria y no hay comunicacion entre Madrid y Lisboa sino se tuviera esta restricción se tendrían 2 primarios.
Por eso esta la restricción de la mayoría para que solo en ese lado haya un Primario.

¿Cómo se determina entonces la tolerancia a fallos?

Tolerancia a fallos = No. Miembros - Mayoría.

Tolerancia a fallos | No. Miembros | Mayoría
--------------------|--------------|--------
1 | 3 | 2
1 | 4 | 3
2 | 5 | 3
2 | 6 | 4
3 | 7 | 4

**Sino hay mayoría funcionando se queda fuera de servio el Cluster**.

Tolerancia a fallos indica cuantos servidores como máximo se pueden caer si es más el cluster deja de funcionar.

Conviene tener impares para poder tener un arbitro.

#### Mayoría en la distribución de servidores en Data Centers :skull:

* Evitar que un Data Center tenga la mayoría de miembros.

Cluster único.
Un solo primario.

Madrid 3 Lisboa 2

Un DataCenter cae. (No es particion de Red) se cae un centro de datos Madrid, fuera de servicio por que no hay mayoria.

Trata que uno de los datacenters no tenga mayoria

En este caso si Madrid se cae
Madrid 2 Lisboa 2 Otro 1 (Nube)

Me quedan 3 servidores hay mayoría y sigue pudiendo haber un primario.

Puede ser una arquitectura MÁS costosa por que el 5 servidor implica gastos de nuevas instalaciones.


#### Configuración de los Miembros

[Member Configuration Tutorials](https://docs.mongodb.com/manual/administration/replica-set-member-configuration/)

**Prioridad** (Para llegar a ser primario, siempre un servidor sea primario) 

* Podemos conseguir la configuración actual del cluster con el método `rs.config()`.
* Modificar la configuración
* Relanzar con el método `rs.reconfig(<objeto>)`


```sh
clusterGetafe:PRIMARY> var configuracion = rs.config()
clusterGetafe:PRIMARY> configuracion
{
        "_id" : "clusterGetafe",
        "version" : 1,
        "protocolVersion" : NumberLong(1),
        "writeConcernMajorityJournalDefault" : true,
        "members" : [
                {
                        "_id" : 0,
                        "host" : "localhost:27017",
                        "arbiterOnly" : false,
                        "buildIndexes" : true,
                        "hidden" : false,
                        "priority" : 1,
                        "tags" : {

                        },
                        "slaveDelay" : NumberLong(0),
                        "votes" : 1
                },
                {
                        "_id" : 1,
                        "host" : "localhost:27018",
                        "arbiterOnly" : false,
                        "buildIndexes" : true,
                        "hidden" : false,
                        "priority" : 1,
                        "tags" : {

                        },
                        "slaveDelay" : NumberLong(0),
                        "votes" : 1
                },
                {
                        "_id" : 2,
                        "host" : "localhost:27019",
                        "arbiterOnly" : false,
                        "buildIndexes" : true,
                        "hidden" : false,
                        "priority" : 1,
                        "tags" : {

                        },
                        "slaveDelay" : NumberLong(0),
                        "votes" : 1
                }
        ],
        "settings" : {
                "chainingAllowed" : true,
                "heartbeatIntervalMillis" : 2000,
                "heartbeatTimeoutSecs" : 10,
                "electionTimeoutMillis" : 10000,
                "catchUpTimeoutMillis" : -1,
                "catchUpTakeoverDelayMillis" : 30000,
                "getLastErrorModes" : {

                },
                "getLastErrorDefaults" : {
                        "w" : 1,
                        "wtimeout" : 0
                },
                "replicaSetId" : ObjectId("5e4292f8cca7dd3e0ef26c77")
        }
}
clusterGetafe:PRIMARY>                                                    
```

Mongo hasta ahora coge como el primario sin esfuerzo.

Para cambiarlo y poner siempre uno como primario cambiamos la prioridad.


```sh
clusterGetafe:PRIMARY> configuracion.members[2].priority = 2  
```

```sh
clusterGetafe:PRIMARY> rs.reconfig(configucion)  
```

```sh
clusterGetafe:PRIMARY> rs.reconfig(configuracion)
{
        "ok" : 1,
        "$clusterTime" : {
                "clusterTime" : Timestamp(1581502878, 1),
                "signature" : {
                        "hash" : BinData(0,"AAAAAAAAAAAAAAAAAAAAAAAAAAA="),
                        "keyId" : NumberLong(0)
                }
        },
        "operationTime" : Timestamp(1581502878, 1)
}
clusterGetafe:PRIMARY>    
usterGetafe:SECONDARY> 
```
Esto me cambia el servidor a que el `27019` sea ahora el primario, el prompt cambia cuando lanza algo.

Con ` rs.status()` en el `27019` ya me sale como el primario.
```sh
clusterGetafe:PRIMARY> rs.status()
{
        "set" : "clusterGetafe",
        "date" : ISODate("2020-02-12T10:22:09.855Z"),
        "myState" : 1,
        "term" : NumberLong(5),
        "syncingTo" : "",
        "syncSourceHost" : "",
        "syncSourceId" : -1,
        "heartbeatIntervalMillis" : NumberLong(2000),
        "majorityVoteCount" : 2,
        "writeMajorityCount" : 2,
        "optimes" : {
                "lastCommittedOpTime" : {
                        "ts" : Timestamp(1581502929, 1),
                        "t" : NumberLong(5)
                },
                "lastCommittedWallTime" : ISODate("2020-02-12T10:22:09.133Z"),
                "readConcernMajorityOpTime" : {
                        "ts" : Timestamp(1581502929, 1),
                        "t" : NumberLong(5)
                },
                "readConcernMajorityWallTime" : ISODate("2020-02-12T10:22:09.133Z"),
                "appliedOpTime" : {
                        "ts" : Timestamp(1581502929, 1),
                        "t" : NumberLong(5)
                },
                "durableOpTime" : {
                        "ts" : Timestamp(1581502929, 1),
                        "t" : NumberLong(5)
                },
                "lastAppliedWallTime" : ISODate("2020-02-12T10:22:09.133Z"),
                "lastDurableWallTime" : ISODate("2020-02-12T10:22:09.133Z")
        },
        "lastStableRecoveryTimestamp" : Timestamp(1581502899, 1),
        "lastStableCheckpointTimestamp" : Timestamp(1581502899, 1),
        "electionCandidateMetrics" : {
                "lastElectionReason" : "priorityTakeover",
                "lastElectionDate" : ISODate("2020-02-12T10:21:28.195Z"),
                "electionTerm" : NumberLong(5),
                "lastCommittedOpTimeAtElection" : {
                        "ts" : Timestamp(1581502878, 1),
                        "t" : NumberLong(4)
                },
                "lastSeenOpTimeAtElection" : {
                        "ts" : Timestamp(1581502878, 1),
                        "t" : NumberLong(4)
                },
                "numVotesNeeded" : 2,
                "priorityAtElection" : 2,
                "electionTimeoutMillis" : NumberLong(10000),
                "priorPrimaryMemberId" : 1,
                "numCatchUpOps" : NumberLong(0),
                "newTermStartDate" : ISODate("2020-02-12T10:21:29.130Z"),
                "wMajorityWriteAvailabilityDate" : ISODate("2020-02-12T10:21:30.189Z")
        },
        "electionParticipantMetrics" : {
                "votedForCandidate" : true,
                "electionTerm" : NumberLong(4),
                "lastVoteDate" : ISODate("2020-02-12T08:42:54.356Z"),
                "electionCandidateMemberId" : 1,
                "voteReason" : "",
                "lastAppliedOpTimeAtElection" : {
                        "ts" : Timestamp(1581496964, 1),
                        "t" : NumberLong(3)
                },
                "maxAppliedOpTimeInSet" : {
                        "ts" : Timestamp(1581496964, 1),
                        "t" : NumberLong(3)
                },
                "priorityAtElection" : 1
        },
        "members" : [
                {
                        "_id" : 0,
                        "name" : "localhost:27017",
                        "ip" : "127.0.0.1",
                        "health" : 1,
                        "state" : 2,
                        "stateStr" : "SECONDARY",
                        "uptime" : 4689,
                        "optime" : {
                                "ts" : Timestamp(1581502919, 1),
                                "t" : NumberLong(5)
                        },
                        "optimeDurable" : {
                                "ts" : Timestamp(1581502919, 1),
                                "t" : NumberLong(5)
                        },
                        "optimeDate" : ISODate("2020-02-12T10:21:59Z"),
                        "optimeDurableDate" : ISODate("2020-02-12T10:21:59Z"),
                        "lastHeartbeat" : ISODate("2020-02-12T10:22:08.258Z"),
                        "lastHeartbeatRecv" : ISODate("2020-02-12T10:22:08.171Z"),
                        "pingMs" : NumberLong(0),
                        "lastHeartbeatMessage" : "",
                        "syncingTo" : "localhost:27019",
                        "syncSourceHost" : "localhost:27019",
                        "syncSourceId" : 2,
                        "infoMessage" : "",
                        "configVersion" : 2
                },
                {
                        "_id" : 1,
                        "name" : "localhost:27018",
                        "ip" : "127.0.0.1",
                        "health" : 1,
                        "state" : 2,
                        "stateStr" : "SECONDARY",
                        "uptime" : 6478,
                        "optime" : {
                                "ts" : Timestamp(1581502919, 1),
                                "t" : NumberLong(5)
                        },
                        "optimeDurable" : {
                                "ts" : Timestamp(1581502919, 1),
                                "t" : NumberLong(5)
                        },
                        "optimeDate" : ISODate("2020-02-12T10:21:59Z"),
                        "optimeDurableDate" : ISODate("2020-02-12T10:21:59Z"),
                        "lastHeartbeat" : ISODate("2020-02-12T10:22:08.259Z"),
                        "lastHeartbeatRecv" : ISODate("2020-02-12T10:22:09.117Z"),
                        "pingMs" : NumberLong(0),
                        "lastHeartbeatMessage" : "",
                        "syncingTo" : "localhost:27019",
                        "syncSourceHost" : "localhost:27019",
                        "syncSourceId" : 2,
                        "infoMessage" : "",
                        "configVersion" : 2
                },
                {
                        "_id" : 2,
                        "name" : "localhost:27019",
                        "ip" : "127.0.0.1",
                        "health" : 1,
                        "state" : 1,
                        "stateStr" : "PRIMARY",
                        "uptime" : 6481,
                        "optime" : {
                                "ts" : Timestamp(1581502929, 1),
                                "t" : NumberLong(5)
                        },
                        "optimeDate" : ISODate("2020-02-12T10:22:09Z"),
                        "syncingTo" : "",
                        "syncSourceHost" : "",
                        "syncSourceId" : -1,
                        "infoMessage" : "",
                        "electionTime" : Timestamp(1581502888, 1),
                        "electionDate" : ISODate("2020-02-12T10:21:28Z"),
                        "configVersion" : 2,
                        "self" : true,
                        "lastHeartbeatMessage" : ""
                }
        ],
        "ok" : 1,
        "$clusterTime" : {
                "clusterTime" : Timestamp(1581502929, 1),
                "signature" : {
                        "hash" : BinData(0,"AAAAAAAAAAAAAAAAAAAAAAAAAAA="),
                        "keyId" : NumberLong(0)
                }
        },
        "operationTime" : Timestamp(1581502929, 1)
}
clusterGetafe:PRIMARY>                                                                                          
```

#### Añadir un Nuevo Miembro

Añadimos la carpeta `server4` (en algo real sería una nueva instancia, instalar Mongo, asignarle puerto, etc.)

```sh
C:\Users\manana>mongod --dbpath data\server4 --port 27020 --replSet clusterGetafe
```

Me voy al Servidor primario **para añadir el nuevo servidor al cluster se usa `rs.add()`** le tenemos que decir que tenga prioridad 0 para que no pueda llegar a ser primario, por que primero tiene que sincronizar con los otros servidores.

```sh
clusterGetafe:PRIMARY> rs.add({ host: "localhost:27020", priority: 0, votes: 0 })
{
        "ok" : 1,
        "$clusterTime" : {
                "clusterTime" : Timestamp(1581507009, 1),
                "signature" : {
                        "hash" : BinData(0,"AAAAAAAAAAAAAAAAAAAAAAAAAAA="),
                        "keyId" : NumberLong(0)
                }
        },
        "operationTime" : Timestamp(1581507009, 1)
}
clusterGetafe:PRIMARY>       
```

Ejecuto el Status:

```sh
clusterGetafe:PRIMARY> rs.status()
{
        "set" : "clusterGetafe",
        "date" : ISODate("2020-02-12T11:34:00.131Z"),
        "myState" : 1,
        "term" : NumberLong(7),
        "syncingTo" : "",
        "syncSourceHost" : "",
        "syncSourceId" : -1,
        "heartbeatIntervalMillis" : NumberLong(2000),
        "majorityVoteCount" : 2,
        "writeMajorityCount" : 2,
        "optimes" : {
                "lastCommittedOpTime" : {
                        "ts" : Timestamp(1581507232, 1),
                        "t" : NumberLong(7)
                },
                "lastCommittedWallTime" : ISODate("2020-02-12T11:33:52.896Z"),
                "readConcernMajorityOpTime" : {
                        "ts" : Timestamp(1581507232, 1),
                        "t" : NumberLong(7)
                },
                "readConcernMajorityWallTime" : ISODate("2020-02-12T11:33:52.896Z"),
                "appliedOpTime" : {
                        "ts" : Timestamp(1581507232, 1),
                        "t" : NumberLong(7)
                },
                "durableOpTime" : {
                        "ts" : Timestamp(1581507232, 1),
                        "t" : NumberLong(7)
                },
                "lastAppliedWallTime" : ISODate("2020-02-12T11:33:52.896Z"),
                "lastDurableWallTime" : ISODate("2020-02-12T11:33:52.896Z")
        },
        "lastStableRecoveryTimestamp" : Timestamp(1581507232, 1),
        "lastStableCheckpointTimestamp" : Timestamp(1581507232, 1),
        "electionCandidateMetrics" : {
                "lastElectionReason" : "priorityTakeover",
                "lastElectionDate" : ISODate("2020-02-12T11:11:20.853Z"),
                "electionTerm" : NumberLong(7),
                "lastCommittedOpTimeAtElection" : {
                        "ts" : Timestamp(1581505870, 1),
                        "t" : NumberLong(6)
                },
                "lastSeenOpTimeAtElection" : {
                        "ts" : Timestamp(1581505880, 1),
                        "t" : NumberLong(6)
                },
                "numVotesNeeded" : 2,
                "priorityAtElection" : 2,
                "electionTimeoutMillis" : NumberLong(10000),
                "priorPrimaryMemberId" : 0,
                "numCatchUpOps" : NumberLong(0),
                "newTermStartDate" : ISODate("2020-02-12T11:11:22.795Z"),
                "wMajorityWriteAvailabilityDate" : ISODate("2020-02-12T11:11:24.037Z")
        },
        "electionParticipantMetrics" : {
                "votedForCandidate" : true,
                "electionTerm" : NumberLong(6),
                "lastVoteDate" : ISODate("2020-02-12T11:11:09.951Z"),
                "electionCandidateMemberId" : 0,
                "voteReason" : "",
                "lastAppliedOpTimeAtElection" : {
                        "ts" : Timestamp(1581503269, 1),
                        "t" : NumberLong(5)
                },
                "maxAppliedOpTimeInSet" : {
                        "ts" : Timestamp(1581503269, 1),
                        "t" : NumberLong(5)
                },
                "priorityAtElection" : 2
        },
        "members" : [
                {
                        "_id" : 0,
                        "name" : "localhost:27017",
                        "ip" : "127.0.0.1",
                        "health" : 1,
                        "state" : 2,
                        "stateStr" : "SECONDARY",
                        "uptime" : 1377,
                        "optime" : {
                                "ts" : Timestamp(1581507232, 1),
                                "t" : NumberLong(7)
                        },
                        "optimeDurable" : {
                                "ts" : Timestamp(1581507232, 1),
                                "t" : NumberLong(7)
                        },
                        "optimeDate" : ISODate("2020-02-12T11:33:52Z"),
                        "optimeDurableDate" : ISODate("2020-02-12T11:33:52Z"),
                        "lastHeartbeat" : ISODate("2020-02-12T11:33:59.214Z"),
                        "lastHeartbeatRecv" : ISODate("2020-02-12T11:33:58.291Z"),
                        "pingMs" : NumberLong(0),
                        "lastHeartbeatMessage" : "",
                        "syncingTo" : "localhost:27019",
                        "syncSourceHost" : "localhost:27019",
                        "syncSourceId" : 2,
                        "infoMessage" : "",
                        "configVersion" : 3
                },
                {
                        "_id" : 1,
                        "name" : "localhost:27018",
                        "ip" : "127.0.0.1",
                        "health" : 1,
                        "state" : 2,
                        "stateStr" : "SECONDARY",
                        "uptime" : 1377,
                        "optime" : {
                                "ts" : Timestamp(1581507232, 1),
                                "t" : NumberLong(7)
                        },
                        "optimeDurable" : {
                                "ts" : Timestamp(1581507232, 1),
                                "t" : NumberLong(7)
                        },
                        "optimeDate" : ISODate("2020-02-12T11:33:52Z"),
                        "optimeDurableDate" : ISODate("2020-02-12T11:33:52Z"),
                        "lastHeartbeat" : ISODate("2020-02-12T11:33:59.219Z"),
                        "lastHeartbeatRecv" : ISODate("2020-02-12T11:33:58.291Z"),
                        "pingMs" : NumberLong(0),
                        "lastHeartbeatMessage" : "",
                        "syncingTo" : "localhost:27019",
                        "syncSourceHost" : "localhost:27019",
                        "syncSourceId" : 2,
                        "infoMessage" : "",
                        "configVersion" : 3
                },
                {
                        "_id" : 2,
                        "name" : "localhost:27019",
                        "ip" : "127.0.0.1",
                        "health" : 1,
                        "state" : 1,
                        "stateStr" : "PRIMARY",
                        "uptime" : 10792,
                        "optime" : {
                                "ts" : Timestamp(1581507232, 1),
                                "t" : NumberLong(7)
                        },
                        "optimeDate" : ISODate("2020-02-12T11:33:52Z"),
                        "syncingTo" : "",
                        "syncSourceHost" : "",
                        "syncSourceId" : -1,
                        "infoMessage" : "",
                        "electionTime" : Timestamp(1581505882, 1),
                        "electionDate" : ISODate("2020-02-12T11:11:22Z"),
                        "configVersion" : 3,
                        "self" : true,
                        "lastHeartbeatMessage" : ""
                },
                {
                        "_id" : 3,
                        "name" : "localhost:27020",
                        "ip" : "127.0.0.1",
                        "health" : 1,
                        "state" : 2,
                        "stateStr" : "SECONDARY",
                        "uptime" : 230,
                        "optime" : {
                                "ts" : Timestamp(1581507232, 1),
                                "t" : NumberLong(7)
                        },
                        "optimeDurable" : {
                                "ts" : Timestamp(1581507232, 1),
                                "t" : NumberLong(7)
                        },
                        "optimeDate" : ISODate("2020-02-12T11:33:52Z"),
                        "optimeDurableDate" : ISODate("2020-02-12T11:33:52Z"),
                        "lastHeartbeat" : ISODate("2020-02-12T11:33:59.219Z"),
                        "lastHeartbeatRecv" : ISODate("2020-02-12T11:33:59.826Z"),
                        "pingMs" : NumberLong(0),
                        "lastHeartbeatMessage" : "",
                        "syncingTo" : "localhost:27019",
                        "syncSourceHost" : "localhost:27019",
                        "syncSourceId" : 2,
                        "infoMessage" : "",
                        "configVersion" : 3
                }
        ],
        "ok" : 1,
        "$clusterTime" : {
                "clusterTime" : Timestamp(1581507232, 1),
                "signature" : {
                        "hash" : BinData(0,"AAAAAAAAAAAAAAAAAAAAAAAAAAA="),
                        "keyId" : NumberLong(0)
                }
        },
        "operationTime" : Timestamp(1581507232, 1)
}
clusterGetafe:PRIMARY>                    
```
Ya vemos el servidor `27010` con ` "_id" : 3,`

Le cambiamos las dos propiedades para llegeado el caso pueda llegar a ser primario.

```sh
clusterGetafe:PRIMARY> var configuracion = rs.config()  

clusterGetafe:PRIMARY> configuracion.members[3].priority = 1
1
clusterGetafe:PRIMARY> configuracion.members[3].votes = 1
1
clusterGetafe:PRIMARY> rs.reconfig(configuracion)
{
        "ok" : 1,
        "$clusterTime" : {
                "clusterTime" : Timestamp(1581507695, 1),
                "signature" : {
                        "hash" : BinData(0,"AAAAAAAAAAAAAAAAAAAAAAAAAAA="),
                        "keyId" : NumberLong(0)
                }
        },
        "operationTime" : Timestamp(1581507695, 1)
}
clusterGetafe:PRIMARY>  
```

27019 Primario

27017, 27018 y 27020 como Secundarios.


Vamos a comprobar la Tolerancia a Fallos (1) 4 -3 

Solo se puede caer uno.

Apago el primario `27019` averiguamos cual de los 3 que quedan es primario


```sh
clusterGetafe:SECONDARY> rs.status()
{
        "set" : "clusterGetafe",
        "date" : ISODate("2020-02-12T11:46:12.003Z"),
        "myState" : 1,
        "term" : NumberLong(8),
        "syncingTo" : "",
        "syncSourceHost" : "",
        "syncSourceId" : -1,
        "heartbeatIntervalMillis" : NumberLong(2000),
        "majorityVoteCount" : 3,
        "writeMajorityCount" : 3,
        "optimes" : {
                "lastCommittedOpTime" : {
                        "ts" : Timestamp(1581507962, 2),
                        "t" : NumberLong(8)
                },
                "lastCommittedWallTime" : ISODate("2020-02-12T11:46:02.986Z"),
                "readConcernMajorityOpTime" : {
                        "ts" : Timestamp(1581507962, 2),
                        "t" : NumberLong(8)
                },
                "readConcernMajorityWallTime" : ISODate("2020-02-12T11:46:02.986Z"),
                "appliedOpTime" : {
                        "ts" : Timestamp(1581507962, 2),
                        "t" : NumberLong(8)
                },
                "durableOpTime" : {
                        "ts" : Timestamp(1581507962, 2),
                        "t" : NumberLong(8)
                },
                "lastAppliedWallTime" : ISODate("2020-02-12T11:46:02.986Z"),
                "lastDurableWallTime" : ISODate("2020-02-12T11:46:02.986Z")
        },
        "lastStableRecoveryTimestamp" : Timestamp(1581507939, 1),
        "lastStableCheckpointTimestamp" : Timestamp(1581507939, 1),
        "electionCandidateMetrics" : {
                "lastElectionReason" : "stepUpRequestSkipDryRun",
                "lastElectionDate" : ISODate("2020-02-12T11:46:01.962Z"),
                "electionTerm" : NumberLong(8),
                "lastCommittedOpTimeAtElection" : {
                        "ts" : Timestamp(1581507952, 1),
                        "t" : NumberLong(7)
                },
                "lastSeenOpTimeAtElection" : {
                        "ts" : Timestamp(1581507952, 1),
                        "t" : NumberLong(7)
                },
                "numVotesNeeded" : 3,
                "priorityAtElection" : 1,
                "electionTimeoutMillis" : NumberLong(10000),
                "priorPrimaryMemberId" : 2,
                "numCatchUpOps" : NumberLong(0),
                "newTermStartDate" : ISODate("2020-02-12T11:46:02.986Z"),
                "wMajorityWriteAvailabilityDate" : ISODate("2020-02-12T11:46:04.415Z")
        },
        "electionParticipantMetrics" : {
                "votedForCandidate" : true,
                "electionTerm" : NumberLong(7),
                "lastVoteDate" : ISODate("2020-02-12T11:11:22.055Z"),
                "electionCandidateMemberId" : 2,
                "voteReason" : "",
                "lastAppliedOpTimeAtElection" : {
                        "ts" : Timestamp(1581505880, 1),
                        "t" : NumberLong(6)
                },
                "maxAppliedOpTimeInSet" : {
                        "ts" : Timestamp(1581505880, 1),
                        "t" : NumberLong(6)
                },
                "priorityAtElection" : 1
        },
        "members" : [
                {
                        "_id" : 0,
                        "name" : "localhost:27017",
                        "ip" : "127.0.0.1",
                        "health" : 1,
                        "state" : 1,
                        "stateStr" : "PRIMARY",
                        "uptime" : 9734,
                        "optime" : {
                                "ts" : Timestamp(1581507962, 2),
                                "t" : NumberLong(8)
                        },
                        "optimeDate" : ISODate("2020-02-12T11:46:02Z"),
                        "syncingTo" : "",
                        "syncSourceHost" : "",
                        "syncSourceId" : -1,
                        "infoMessage" : "",
                        "electionTime" : Timestamp(1581507962, 1),
                        "electionDate" : ISODate("2020-02-12T11:46:02Z"),
                        "configVersion" : 4,
                        "self" : true,
                        "lastHeartbeatMessage" : ""
                },
                {
                        "_id" : 1,
                        "name" : "localhost:27018",
                        "ip" : "127.0.0.1",
                        "health" : 1,
                        "state" : 2,
                        "stateStr" : "SECONDARY",
                        "uptime" : 9731,
                        "optime" : {
                                "ts" : Timestamp(1581507962, 2),
                                "t" : NumberLong(8)
                        },
                        "optimeDurable" : {
                                "ts" : Timestamp(1581507962, 2),
                                "t" : NumberLong(8)
                        },
                        "optimeDate" : ISODate("2020-02-12T11:46:02Z"),
                        "optimeDurableDate" : ISODate("2020-02-12T11:46:02Z"),
                        "lastHeartbeat" : ISODate("2020-02-12T11:46:10.105Z"),
                        "lastHeartbeatRecv" : ISODate("2020-02-12T11:46:10.335Z"),
                        "pingMs" : NumberLong(0),
                        "lastHeartbeatMessage" : "",
                        "syncingTo" : "localhost:27017",
                        "syncSourceHost" : "localhost:27017",
                        "syncSourceId" : 0,
                        "infoMessage" : "",
                        "configVersion" : 4
                },
                {
                        "_id" : 2,
                        "name" : "localhost:27019",
                        "ip" : "127.0.0.1",
                        "health" : 0,
                        "state" : 8,
                        "stateStr" : "(not reachable/healthy)",
                        "uptime" : 0,
                        "optime" : {
                                "ts" : Timestamp(0, 0),
                                "t" : NumberLong(-1)
                        },
                        "optimeDurable" : {
                                "ts" : Timestamp(0, 0),
                                "t" : NumberLong(-1)
                        },
                        "optimeDate" : ISODate("1970-01-01T00:00:00Z"),
                        "optimeDurableDate" : ISODate("1970-01-01T00:00:00Z"),
                        "lastHeartbeat" : ISODate("2020-02-12T11:46:10.111Z"),
                        "lastHeartbeatRecv" : ISODate("2020-02-12T11:46:01.165Z"),
                        "pingMs" : NumberLong(0),
                        "lastHeartbeatMessage" : "Error connecting to localhost:27019 (127.0.0.1:27019) :: caused by :: No se puede establecer una conexi�n ya que el equipo de destino deneg� expresamente dicha conexi�n.",
                        "syncingTo" : "",
                        "syncSourceHost" : "",
                        "syncSourceId" : -1,
                        "infoMessage" : "",
                        "configVersion" : -1
                },
                {
                        "_id" : 3,
                        "name" : "localhost:27020",
                        "ip" : "127.0.0.1",
                        "health" : 1,
                        "state" : 2,
                        "stateStr" : "SECONDARY",
                        "uptime" : 962,
                        "optime" : {
                                "ts" : Timestamp(1581507962, 2),
                                "t" : NumberLong(8)
                        },
                        "optimeDurable" : {
                                "ts" : Timestamp(1581507962, 2),
                                "t" : NumberLong(8)
                        },
                        "optimeDate" : ISODate("2020-02-12T11:46:02Z"),
                        "optimeDurableDate" : ISODate("2020-02-12T11:46:02Z"),
                        "lastHeartbeat" : ISODate("2020-02-12T11:46:10.106Z"),
                        "lastHeartbeatRecv" : ISODate("2020-02-12T11:46:10.341Z"),
                        "pingMs" : NumberLong(0),
                        "lastHeartbeatMessage" : "",
                        "syncingTo" : "localhost:27017",
                        "syncSourceHost" : "localhost:27017",
                        "syncSourceId" : 0,
                        "infoMessage" : "",
                        "configVersion" : 4
                }
        ],
        "ok" : 1,
        "$clusterTime" : {
                "clusterTime" : Timestamp(1581507962, 2),
                "signature" : {
                        "hash" : BinData(0,"AAAAAAAAAAAAAAAAAAAAAAAAAAA="),
                        "keyId" : NumberLong(0)
                }
        },
        "operationTime" : Timestamp(1581507962, 2)
}
clusterGetafe:PRIMARY>    
```

Me indica que el `27017` es el primario, lo apago tambien. y verifico en la consola del `27018`


```sh
clusterGetafe:SECONDARY> rs.status()
{
        "set" : "clusterGetafe",
        "date" : ISODate("2020-02-12T11:48:23.835Z"),
        "myState" : 2,
        "term" : NumberLong(9),
        "syncingTo" : "",
        "syncSourceHost" : "",
        "syncSourceId" : -1,
        "heartbeatIntervalMillis" : NumberLong(2000),
        "majorityVoteCount" : 3,
        "writeMajorityCount" : 3,
        "optimes" : {
                "lastCommittedOpTime" : {
                        "ts" : Timestamp(1581508052, 1),
                        "t" : NumberLong(8)
                },
                "lastCommittedWallTime" : ISODate("2020-02-12T11:47:32.995Z"),
                "readConcernMajorityOpTime" : {
                        "ts" : Timestamp(1581508052, 1),
                        "t" : NumberLong(8)
                },
                "readConcernMajorityWallTime" : ISODate("2020-02-12T11:47:32.995Z"),
                "appliedOpTime" : {
                        "ts" : Timestamp(1581508052, 1),
                        "t" : NumberLong(8)
                },
                "durableOpTime" : {
                        "ts" : Timestamp(1581508052, 1),
                        "t" : NumberLong(8)
                },
                "lastAppliedWallTime" : ISODate("2020-02-12T11:47:32.995Z"),
                "lastDurableWallTime" : ISODate("2020-02-12T11:47:32.995Z")
        },
        "lastStableRecoveryTimestamp" : Timestamp(1581508042, 1),
        "lastStableCheckpointTimestamp" : Timestamp(1581508042, 1),
        "electionParticipantMetrics" : {
                "votedForCandidate" : true,
                "electionTerm" : NumberLong(8),
                "lastVoteDate" : ISODate("2020-02-12T11:46:02.072Z"),
                "electionCandidateMemberId" : 0,
                "voteReason" : "",
                "lastAppliedOpTimeAtElection" : {
                        "ts" : Timestamp(1581507952, 1),
                        "t" : NumberLong(7)
                },
                "maxAppliedOpTimeInSet" : {
                        "ts" : Timestamp(1581507952, 1),
                        "t" : NumberLong(7)
                },
                "priorityAtElection" : 1
        },
        "members" : [
                {
                        "_id" : 0,
                        "name" : "localhost:27017",
                        "ip" : "127.0.0.1",
                        "health" : 0,
                        "state" : 8,
                        "stateStr" : "(not reachable/healthy)",
                        "uptime" : 0,
                        "optime" : {
                                "ts" : Timestamp(0, 0),
                                "t" : NumberLong(-1)
                        },
                        "optimeDurable" : {
                                "ts" : Timestamp(0, 0),
                                "t" : NumberLong(-1)
                        },
                        "optimeDate" : ISODate("1970-01-01T00:00:00Z"),
                        "optimeDurableDate" : ISODate("1970-01-01T00:00:00Z"),
                        "lastHeartbeat" : ISODate("2020-02-12T11:48:22.148Z"),
                        "lastHeartbeatRecv" : ISODate("2020-02-12T11:47:42.132Z"),
                        "pingMs" : NumberLong(0),
                        "lastHeartbeatMessage" : "Error connecting to localhost:27017 (127.0.0.1:27017) :: caused by :: No se puede establecer una conexi�n ya que el equipo de destino deneg� expresamente dicha conexi�n.",
                        "syncingTo" : "",
                        "syncSourceHost" : "",
                        "syncSourceId" : -1,
                        "infoMessage" : "",
                        "configVersion" : -1
                },
                {
                        "_id" : 1,
                        "name" : "localhost:27018",
                        "ip" : "127.0.0.1",
                        "health" : 1,
                        "state" : 2,
                        "stateStr" : "SECONDARY",
                        "uptime" : 11756,
                        "optime" : {
                                "ts" : Timestamp(1581508052, 1),
                                "t" : NumberLong(8)
                        },
                        "optimeDate" : ISODate("2020-02-12T11:47:32Z"),
                        "syncingTo" : "",
                        "syncSourceHost" : "",
                        "syncSourceId" : -1,
                        "infoMessage" : "could not find member to sync from",
                        "configVersion" : 4,
                        "self" : true,
                        "lastHeartbeatMessage" : ""
                },
                {
                        "_id" : 2,
                        "name" : "localhost:27019",
                        "ip" : "127.0.0.1",
                        "health" : 0,
                        "state" : 8,
                        "stateStr" : "(not reachable/healthy)",
                        "uptime" : 0,
                        "optime" : {
                                "ts" : Timestamp(0, 0),
                                "t" : NumberLong(-1)
                        },
                        "optimeDurable" : {
                                "ts" : Timestamp(0, 0),
                                "t" : NumberLong(-1)
                        },
                        "optimeDate" : ISODate("1970-01-01T00:00:00Z"),
                        "optimeDurableDate" : ISODate("1970-01-01T00:00:00Z"),
                        "lastHeartbeat" : ISODate("2020-02-12T11:48:23.679Z"),
                        "lastHeartbeatRecv" : ISODate("2020-02-12T11:46:01.165Z"),
                        "pingMs" : NumberLong(0),
                        "lastHeartbeatMessage" : "Error connecting to localhost:27019 (127.0.0.1:27019) :: caused by :: No se puede establecer una conexi�n ya que el equipo de destino deneg� expresamente dicha conexi�n.",
                        "syncingTo" : "",
                        "syncSourceHost" : "",
                        "syncSourceId" : -1,
                        "infoMessage" : "",
                        "configVersion" : -1
                },
                {
                        "_id" : 3,
                        "name" : "localhost:27020",
                        "ip" : "127.0.0.1",
                        "health" : 1,
                        "state" : 2,
                        "stateStr" : "SECONDARY",
                        "uptime" : 1094,
                        "optime" : {
                                "ts" : Timestamp(1581508052, 1),
                                "t" : NumberLong(8)
                        },
                        "optimeDurable" : {
                                "ts" : Timestamp(1581508052, 1),
                                "t" : NumberLong(8)
                        },
                        "optimeDate" : ISODate("2020-02-12T11:47:32Z"),
                        "optimeDurableDate" : ISODate("2020-02-12T11:47:32Z"),
                        "lastHeartbeat" : ISODate("2020-02-12T11:48:23.621Z"),
                        "lastHeartbeatRecv" : ISODate("2020-02-12T11:48:23.620Z"),
                        "pingMs" : NumberLong(0),
                        "lastHeartbeatMessage" : "",
                        "syncingTo" : "",
                        "syncSourceHost" : "",
                        "syncSourceId" : -1,
                        "infoMessage" : "",
                        "configVersion" : 4
                }
        ],
        "ok" : 1,
        "$clusterTime" : {
                "clusterTime" : Timestamp(1581508052, 1),
                "signature" : {
                        "hash" : BinData(0,"AAAAAAAAAAAAAAAAAAAAAAAAAAA="),
                        "keyId" : NumberLong(0)
                }
        },
        "operationTime" : Timestamp(1581508052, 1)
}
clusterGetafe:SECONDARY>                                                                                                                
```

Los dos servidores que quedan estan como Secundarios, ya no tendre la oportunidad de hacer operaciones de Lectura y Escritura por que ya no hay Primario.

Levanto Todos y tumbo el primario `27019` averiguo cual es el primario y tumbo un secundario. 

**¿Que pasa? El primario se mantiene como Primario. NO EL PRIMARIO PASA A SER SECUNDARIO y SE PIERDE EL ACCESO AL CLUSTER.**


Vamos a añadir un 5 servidor para que funcione como arbitro para hacer el desempate, solo sirve para provocar el desempate, no mantendra los datos ni nada solo para votaciones, aportas robustes a tu sistema.



#### Añadir un nuevo miembro árbitro 

```sh
rs.addArb("<direccion:puerto>")
```
Creamos carpeta `arbitro` y lo levantamos en `27021`

```sh
C:\Users\manana>mongod --dbpath data\arbitro --port 27021 --replSet clusterGetafe
```

Lo levanto como arbitro, no tendra permisos de lectura, ni prioridad ni nada solo es para las elecciones. Se instalara en un servidor muy básico (Black Berry :) ).

```sh
clusterGetafe:PRIMARY> rs.addArb("localhost:27021")
{
        "ok" : 1,
        "$clusterTime" : {
                "clusterTime" : Timestamp(1581509536, 1),
                "signature" : {
                        "hash" : BinData(0,"AAAAAAAAAAAAAAAAAAAAAAAAAAA="),
                        "keyId" : NumberLong(0)
                }
        },
        "operationTime" : Timestamp(1581509536, 1)
}
clusterGetafe:PRIMARY>      
```
Veo el Status
```sh
clusterGetafe:PRIMARY> rs.status()
{
        "set" : "clusterGetafe",
        "date" : ISODate("2020-02-12T12:17:20.802Z"),
        "myState" : 1,
        "term" : NumberLong(11),
        "syncingTo" : "",
        "syncSourceHost" : "",
        "syncSourceId" : -1,
        "heartbeatIntervalMillis" : NumberLong(2000),
        "majorityVoteCount" : 3,
        "writeMajorityCount" : 3,
        "optimes" : {
                "lastCommittedOpTime" : {
                        "ts" : Timestamp(1581509831, 1),
                        "t" : NumberLong(11)
                },
                "lastCommittedWallTime" : ISODate("2020-02-12T12:17:11.427Z"),
                "readConcernMajorityOpTime" : {
                        "ts" : Timestamp(1581509831, 1),
                        "t" : NumberLong(11)
                },
                "readConcernMajorityWallTime" : ISODate("2020-02-12T12:17:11.427Z"),
                "appliedOpTime" : {
                        "ts" : Timestamp(1581509831, 1),
                        "t" : NumberLong(11)
                },
                "durableOpTime" : {
                        "ts" : Timestamp(1581509831, 1),
                        "t" : NumberLong(11)
                },
                "lastAppliedWallTime" : ISODate("2020-02-12T12:17:11.427Z"),
                "lastDurableWallTime" : ISODate("2020-02-12T12:17:11.427Z")
        },
        "lastStableRecoveryTimestamp" : Timestamp(1581509821, 1),
        "lastStableCheckpointTimestamp" : Timestamp(1581509821, 1),
        "electionCandidateMetrics" : {
                "lastElectionReason" : "priorityTakeover",
                "lastElectionDate" : ISODate("2020-02-12T11:51:11.013Z"),
                "electionTerm" : NumberLong(11),
                "lastCommittedOpTimeAtElection" : {
                        "ts" : Timestamp(1581508254, 2),
                        "t" : NumberLong(10)
                },
                "lastSeenOpTimeAtElection" : {
                        "ts" : Timestamp(1581508254, 2),
                        "t" : NumberLong(10)
                },
                "numVotesNeeded" : 3,
                "priorityAtElection" : 2,
                "electionTimeoutMillis" : NumberLong(10000),
                "priorPrimaryMemberId" : 3,
                "numCatchUpOps" : NumberLong(0),
                "newTermStartDate" : ISODate("2020-02-12T11:51:11.303Z"),
                "wMajorityWriteAvailabilityDate" : ISODate("2020-02-12T11:51:13.499Z")
        },
        "members" : [
                {
                        "_id" : 0,
                        "name" : "localhost:27017",
                        "ip" : "127.0.0.1",
                        "health" : 1,
                        "state" : 2,
                        "stateStr" : "SECONDARY",
                        "uptime" : 1580,
                        "optime" : {
                                "ts" : Timestamp(1581509831, 1),
                                "t" : NumberLong(11)
                        },
                        "optimeDurable" : {
                                "ts" : Timestamp(1581509831, 1),
                                "t" : NumberLong(11)
                        },
                        "optimeDate" : ISODate("2020-02-12T12:17:11Z"),
                        "optimeDurableDate" : ISODate("2020-02-12T12:17:11Z"),
                        "lastHeartbeat" : ISODate("2020-02-12T12:17:20.111Z"),
                        "lastHeartbeatRecv" : ISODate("2020-02-12T12:17:20.255Z"),
                        "pingMs" : NumberLong(0),
                        "lastHeartbeatMessage" : "",
                        "syncingTo" : "localhost:27019",
                        "syncSourceHost" : "localhost:27019",
                        "syncSourceId" : 2,
                        "infoMessage" : "",
                        "configVersion" : 5
                },
                {
                        "_id" : 1,
                        "name" : "localhost:27018",
                        "ip" : "127.0.0.1",
                        "health" : 1,
                        "state" : 2,
                        "stateStr" : "SECONDARY",
                        "uptime" : 1580,
                        "optime" : {
                                "ts" : Timestamp(1581509831, 1),
                                "t" : NumberLong(11)
                        },
                        "optimeDurable" : {
                                "ts" : Timestamp(1581509831, 1),
                                "t" : NumberLong(11)
                        },
                        "optimeDate" : ISODate("2020-02-12T12:17:11Z"),
                        "optimeDurableDate" : ISODate("2020-02-12T12:17:11Z"),
                        "lastHeartbeat" : ISODate("2020-02-12T12:17:20.116Z"),
                        "lastHeartbeatRecv" : ISODate("2020-02-12T12:17:20.256Z"),
                        "pingMs" : NumberLong(0),
                        "lastHeartbeatMessage" : "",
                        "syncingTo" : "localhost:27019",
                        "syncSourceHost" : "localhost:27019",
                        "syncSourceId" : 2,
                        "infoMessage" : "",
                        "configVersion" : 5
                },
                {
                        "_id" : 2,
                        "name" : "localhost:27019",
                        "ip" : "127.0.0.1",
                        "health" : 1,
                        "state" : 1,
                        "stateStr" : "PRIMARY",
                        "uptime" : 1582,
                        "optime" : {
                                "ts" : Timestamp(1581509831, 1),
                                "t" : NumberLong(11)
                        },
                        "optimeDate" : ISODate("2020-02-12T12:17:11Z"),
                        "syncingTo" : "",
                        "syncSourceHost" : "",
                        "syncSourceId" : -1,
                        "infoMessage" : "",
                        "electionTime" : Timestamp(1581508271, 1),
                        "electionDate" : ISODate("2020-02-12T11:51:11Z"),
                        "configVersion" : 5,
                        "self" : true,
                        "lastHeartbeatMessage" : ""
                },
                {
                        "_id" : 3,
                        "name" : "localhost:27020",
                        "ip" : "127.0.0.1",
                        "health" : 1,
                        "state" : 2,
                        "stateStr" : "SECONDARY",
                        "uptime" : 1580,
                        "optime" : {
                                "ts" : Timestamp(1581509831, 1),
                                "t" : NumberLong(11)
                        },
                        "optimeDurable" : {
                                "ts" : Timestamp(1581509831, 1),
                                "t" : NumberLong(11)
                        },
                        "optimeDate" : ISODate("2020-02-12T12:17:11Z"),
                        "optimeDurableDate" : ISODate("2020-02-12T12:17:11Z"),
                        "lastHeartbeat" : ISODate("2020-02-12T12:17:20.116Z"),
                        "lastHeartbeatRecv" : ISODate("2020-02-12T12:17:20.238Z"),
                        "pingMs" : NumberLong(0),
                        "lastHeartbeatMessage" : "",
                        "syncingTo" : "localhost:27019",
                        "syncSourceHost" : "localhost:27019",
                        "syncSourceId" : 2,
                        "infoMessage" : "",
                        "configVersion" : 5
                },
                {
                        "_id" : 4,
                        "name" : "localhost:27021",
                        "ip" : "127.0.0.1",
                        "health" : 1,
                        "state" : 7,
                        "stateStr" : "ARBITER",
                        "uptime" : 304,
                        "lastHeartbeat" : ISODate("2020-02-12T12:17:20.112Z"),
                        "lastHeartbeatRecv" : ISODate("2020-02-12T12:17:20.355Z"),
                        "pingMs" : NumberLong(0),
                        "lastHeartbeatMessage" : "",
                        "syncingTo" : "",
                        "syncSourceHost" : "",
                        "syncSourceId" : -1,
                        "infoMessage" : "",
                        "configVersion" : 5
                }
        ],
        "ok" : 1,
        "$clusterTime" : {
                "clusterTime" : Timestamp(1581509831, 1),
                "signature" : {
                        "hash" : BinData(0,"AAAAAAAAAAAAAAAAAAAAAAAAAAA="),
                        "keyId" : NumberLong(0)
                }
        },
        "operationTime" : Timestamp(1581509831, 1)
}
clusterGetafe:PRIMARY>                                                                                                                 
```
Veo que el `27021` es un `"ARBITER"`.

Le doy la configuración

```sh
clusterGetafe:PRIMARY> rs.config()
{
        "_id" : "clusterGetafe",
        "version" : 5,
        "protocolVersion" : NumberLong(1),
        "writeConcernMajorityJournalDefault" : true,
        "members" : [
                {
                        "_id" : 0,
                        "host" : "localhost:27017",
                        "arbiterOnly" : false,
                        "buildIndexes" : true,
                        "hidden" : false,
                        "priority" : 1,
                        "tags" : {

                        },
                        "slaveDelay" : NumberLong(0),
                        "votes" : 1
                },
                {
                        "_id" : 1,
                        "host" : "localhost:27018",
                        "arbiterOnly" : false,
                        "buildIndexes" : true,
                        "hidden" : false,
                        "priority" : 1,
                        "tags" : {

                        },
                        "slaveDelay" : NumberLong(0),
                        "votes" : 1
                },
                {
                        "_id" : 2,
                        "host" : "localhost:27019",
                        "arbiterOnly" : false,
                        "buildIndexes" : true,
                        "hidden" : false,
                        "priority" : 2,
                        "tags" : {

                        },
                        "slaveDelay" : NumberLong(0),
                        "votes" : 1
                },
                {
                        "_id" : 3,
                        "host" : "localhost:27020",
                        "arbiterOnly" : false,
                        "buildIndexes" : true,
                        "hidden" : false,
                        "priority" : 1,
                        "tags" : {

                        },
                        "slaveDelay" : NumberLong(0),
                        "votes" : 1
                },
                {
                        "_id" : 4,
                        "host" : "localhost:27021",
                        "arbiterOnly" : true,
                        "buildIndexes" : true,
                        "hidden" : false,
                        "priority" : 0,
                        "tags" : {

                        },
                        "slaveDelay" : NumberLong(0),
                        "votes" : 1
                }
        ],
        "settings" : {
                "chainingAllowed" : true,
                "heartbeatIntervalMillis" : 2000,
                "heartbeatTimeoutSecs" : 10,
                "electionTimeoutMillis" : 10000,
                "catchUpTimeoutMillis" : -1,
                "catchUpTakeoverDelayMillis" : 30000,
                "getLastErrorModes" : {

                },
                "getLastErrorDefaults" : {
                        "w" : 1,
                        "wtimeout" : 0
                },
                "replicaSetId" : ObjectId("5e4292f8cca7dd3e0ef26c77")
        }
}
clusterGetafe:PRIMARY>                                                                                          

```

Veo en la configuración que `27021` tiene prioridad `"priority" : 0` por que nunca puede llegar a ser Primario.


####Comando `db.isMaster()`

```sh
clusterGetafe:PRIMARY> db.isMaster()
{
        "hosts" : [
                "localhost:27017",
                "localhost:27018",
                "localhost:27019",
                "localhost:27020"
        ],
        "arbiters" : [
                "localhost:27021"
        ],
        "setName" : "clusterGetafe",
        "setVersion" : 5,
        "ismaster" : true,
        "secondary" : false,
        "primary" : "localhost:27019",
        "me" : "localhost:27019",
        "electionId" : ObjectId("7fffffff000000000000000b"),
        "lastWrite" : {
                "opTime" : {
                        "ts" : Timestamp(1581510571, 1),
                        "t" : NumberLong(11)
                },
                "lastWriteDate" : ISODate("2020-02-12T12:29:31Z"),
                "majorityOpTime" : {
                        "ts" : Timestamp(1581510571, 1),
                        "t" : NumberLong(11)
                },
                "majorityWriteDate" : ISODate("2020-02-12T12:29:31Z")
        },
        "maxBsonObjectSize" : 16777216,
        "maxMessageSizeBytes" : 48000000,
        "maxWriteBatchSize" : 100000,
        "localTime" : ISODate("2020-02-12T12:29:34.354Z"),
        "logicalSessionTimeoutMinutes" : 30,
        "connectionId" : 22,
        "minWireVersion" : 0,
        "maxWireVersion" : 8,
        "readOnly" : false,
        "ok" : 1,
        "$clusterTime" : {
                "clusterTime" : Timestamp(1581510571, 1),
                "signature" : {
                        "hash" : BinData(0,"AAAAAAAAAAAAAAAAAAAAAAAAAAA="),
                        "keyId" : NumberLong(0)
                }
        },
        "operationTime" : Timestamp(1581510571, 1)
}
clusterGetafe:PRIMARY>
```

**CREAR APLICACION ANGULAR PARA PINTAR TODOS LOS DATOS QUE ARRAGAN ESTOS COMANDOS DE db, (char.js) **

#### Configurar un miembro como `delayed`

Va con retraso

Vamos a hacer que la replica en un servidor se haga 60s despues, por si alguien borra registros, en el primario en otro servidor tiene los datos hasta una hora después.

No primario, oculto y retrasado 60s
```sh
clusterGetafe:PRIMARY> var configuracion = rs.config()

clusterGetafe:PRIMARY> configuracion.members[3].slaveDelay = 60
60
clusterGetafe:PRIMARY> configuracion.members[3].priority = 0
0
clusterGetafe:PRIMARY> configuracion.members[3].hidden = true
true
clusterGetafe:PRIMARY>
```

Aplico los cambios persistiendolos:

```sh
clusterGetafe:PRIMARY> rs.reconfig(configuracion)
{
        "ok" : 1,
        "$clusterTime" : {
                "clusterTime" : Timestamp(1581511361, 1),
                "signature" : {
                        "hash" : BinData(0,"AAAAAAAAAAAAAAAAAAAAAAAAAAA="),
                        "keyId" : NumberLong(0)
                }
        },
        "operationTime" : Timestamp(1581511361, 1)
}
clusterGetafe:PRIMARY>   
```
**Como se recupera de `rs.config()` recupero toda la configuración y cambio solo lo que quiero y la vuelvo a aplicar.**

Lanzamos el isMaster

```sh
clusterGetafe:PRIMARY> db.isMaster()
{
        "hosts" : [
                "localhost:27017",
                "localhost:27018",
                "localhost:27019"
        ],
        "arbiters" : [
                "localhost:27021"
        ],
        "setName" : "clusterGetafe",
        "setVersion" : 6,
        "ismaster" : true,
        "secondary" : false,
        "primary" : "localhost:27019",
        "me" : "localhost:27019",
        "electionId" : ObjectId("7fffffff000000000000000b"),
        "lastWrite" : {
                "opTime" : {
                        "ts" : Timestamp(1581511641, 1),
                        "t" : NumberLong(11)
                },
                "lastWriteDate" : ISODate("2020-02-12T12:47:21Z"),
                "majorityOpTime" : {
                        "ts" : Timestamp(1581511641, 1),
                        "t" : NumberLong(11)
                },
                "majorityWriteDate" : ISODate("2020-02-12T12:47:21Z")
        },
        "maxBsonObjectSize" : 16777216,
        "maxMessageSizeBytes" : 48000000,
        "maxWriteBatchSize" : 100000,
        "localTime" : ISODate("2020-02-12T12:47:31.153Z"),
        "logicalSessionTimeoutMinutes" : 30,
        "connectionId" : 22,
        "minWireVersion" : 0,
        "maxWireVersion" : 8,
        "readOnly" : false,
        "ok" : 1,
        "$clusterTime" : {
                "clusterTime" : Timestamp(1581511641, 1),
                "signature" : {
                        "hash" : BinData(0,"AAAAAAAAAAAAAAAAAAAAAAAAAAA="),
                        "keyId" : NumberLong(0)
                }
        },
        "operationTime" : Timestamp(1581511641, 1)
}
clusterGetafe:PRIMARY>
```
No lo pinta por que ya esta oculto `27020`

En mi primario 
```sh
clusterGetafe:PRIMARY> use getafeTest
switched to db getafeTest
clusterGetafe:PRIMARY> db.foo5.insert({ m: "hola", fecha: new Date() })
WriteResult({ "nInserted" : 1 })
clusterGetafe:PRIMARY>  
```

Y en la maquina hago .

```sh
C:\Users\manana>mongoexport --port 27020 -c foo5 -d getafeTest --out=delayed.json
2020-02-12T13:51:22.151+0100    connected to: mongodb://localhost:27020/
2020-02-12T13:51:22.236+0100    exported 0 records
```
Hasta despues de 60 segundos me debe dar datos


```sh
C:\Users\manana>mongoexport --port 27020 -c foo5 -d getafeTest --out=delayed.json
2020-02-12T13:52:08.063+0100    connected to: mongodb://localhost:27020/
2020-02-12T13:52:08.165+0100    exported 1 record
```
Despues de los 60s ya veo que tengo los datos.


* **Resumen**
   * Tolerancia a fallos.
   * Configuración de miembros. :skull:...:skull: Sencillas
      * Prioridad
      * Árbitro.
      * Miembro Oculto.
      * Miembro Delayed.
   
#### Votos de los miembros.

[Configure Non-Voting Replica Set Member](https://docs.mongodb.com/manual/tutorial/configure-a-non-voting-replica-set-member/index.html)

Propiedades de los votos de un miembro permite a este participar en la elección del nuevo primario cuando el actual quede fuera de servicio.

* No. Miembros totales < 50
* No. Miembros con derecho a voto < 7

En cluster con más de 7 miembros hay que asignar la propiedad votes como 0 para limitar a 7 miembros el número de miembros que pueden votar.


Abrimos nuestros 5 servidores y la consola del primario:

```sh
C:\Users\manana>mongod --dbpath data/server1 --port 27017 --replSet clusterGetafe
C:\Users\manana>mongod --dbpath data/server2 --port 27018 --replSet clusterGetafe
C:\Users\manana>mongod --dbpath data/server3 --port 27019 --replSet clusterGetafe
C:\Users\manana>mongod --dbpath data/server4 --port 27020 --replSet clusterGetafe
C:\Users\manana>mongod --dbpath data/arbitro --port 27021 --replSet clusterGetafe

mongod --port 27019

```

En mi servidor Primary vemos el Status

```sh
clusterGetafe:PRIMARY> rs.status()
{
        "set" : "clusterGetafe",
        "date" : ISODate("2020-02-13T08:29:29.828Z"),
        "myState" : 1,
        "term" : NumberLong(14),
        "syncingTo" : "",
        "syncSourceHost" : "",
        "syncSourceId" : -1,
        "heartbeatIntervalMillis" : NumberLong(2000),
        "majorityVoteCount" : 3,
        "writeMajorityCount" : 3,
        "optimes" : {
                "lastCommittedOpTime" : {
                        "ts" : Timestamp(1581582568, 1),
                        "t" : NumberLong(14)
                },
                "lastCommittedWallTime" : ISODate("2020-02-13T08:29:28.072Z"),
                "readConcernMajorityOpTime" : {
                        "ts" : Timestamp(1581582568, 1),
                        "t" : NumberLong(14)
                },
                "readConcernMajorityWallTime" : ISODate("2020-02-13T08:29:28.072Z"),
                "appliedOpTime" : {
                        "ts" : Timestamp(1581582568, 1),
                        "t" : NumberLong(14)
                },
                "durableOpTime" : {
                        "ts" : Timestamp(1581582568, 1),
                        "t" : NumberLong(14)
                },
                "lastAppliedWallTime" : ISODate("2020-02-13T08:29:28.072Z"),
                "lastDurableWallTime" : ISODate("2020-02-13T08:29:28.072Z")
        },
        "lastStableRecoveryTimestamp" : Timestamp(1581582528, 1),
        "lastStableCheckpointTimestamp" : Timestamp(1581582528, 1),
        "electionCandidateMetrics" : {
                "lastElectionReason" : "priorityTakeover",
                "lastElectionDate" : ISODate("2020-02-13T08:26:13.357Z"),
                "electionTerm" : NumberLong(14),
                "lastCommittedOpTimeAtElection" : {
                        "ts" : Timestamp(1581582364, 1),
                        "t" : NumberLong(13)
                },
                "lastSeenOpTimeAtElection" : {
                        "ts" : Timestamp(1581582364, 1),
                        "t" : NumberLong(13)
                },
                "numVotesNeeded" : 3,
                "priorityAtElection" : 2,
                "electionTimeoutMillis" : NumberLong(10000),
                "priorPrimaryMemberId" : 1,
                "numCatchUpOps" : NumberLong(0),
                "newTermStartDate" : ISODate("2020-02-13T08:26:18.058Z"),
                "wMajorityWriteAvailabilityDate" : ISODate("2020-02-13T08:26:19.599Z")
        },
        "electionParticipantMetrics" : {
                "votedForCandidate" : true,
                "electionTerm" : NumberLong(13),
                "lastVoteDate" : ISODate("2020-02-13T08:26:01.762Z"),
                "electionCandidateMemberId" : 1,
                "voteReason" : "",
                "lastAppliedOpTimeAtElection" : {
                        "ts" : Timestamp(1581512091, 1),
                        "t" : NumberLong(11)
                },
                "maxAppliedOpTimeInSet" : {
                        "ts" : Timestamp(1581512091, 1),
                        "t" : NumberLong(11)
                },
                "priorityAtElection" : 2
        },
        "members" : [
                {
                        "_id" : 0,
                        "name" : "localhost:27017",
                        "ip" : "127.0.0.1",
                        "health" : 1,
                        "state" : 2,
                        "stateStr" : "SECONDARY",
                        "uptime" : 209,
                        "optime" : {
                                "ts" : Timestamp(1581582568, 1),
                                "t" : NumberLong(14)
                        },
                        "optimeDurable" : {
                                "ts" : Timestamp(1581582568, 1),
                                "t" : NumberLong(14)
                        },
                        "optimeDate" : ISODate("2020-02-13T08:29:28Z"),
                        "optimeDurableDate" : ISODate("2020-02-13T08:29:28Z"),
                        "lastHeartbeat" : ISODate("2020-02-13T08:29:29.539Z"),
                        "lastHeartbeatRecv" : ISODate("2020-02-13T08:29:29.495Z"),
                        "pingMs" : NumberLong(0),
                        "lastHeartbeatMessage" : "",
                        "syncingTo" : "localhost:27019",
                        "syncSourceHost" : "localhost:27019",
                        "syncSourceId" : 2,
                        "infoMessage" : "",
                        "configVersion" : 6
                },
                {
                        "_id" : 1,
                        "name" : "localhost:27018",
                        "ip" : "127.0.0.1",
                        "health" : 1,
                        "state" : 2,
                        "stateStr" : "SECONDARY",
                        "uptime" : 209,
                        "optime" : {
                                "ts" : Timestamp(1581582568, 1),
                                "t" : NumberLong(14)
                        },
                        "optimeDurable" : {
                                "ts" : Timestamp(1581582568, 1),
                                "t" : NumberLong(14)
                        },
                        "optimeDate" : ISODate("2020-02-13T08:29:28Z"),
                        "optimeDurableDate" : ISODate("2020-02-13T08:29:28Z"),
                        "lastHeartbeat" : ISODate("2020-02-13T08:29:29.539Z"),
                        "lastHeartbeatRecv" : ISODate("2020-02-13T08:29:29.318Z"),
                        "pingMs" : NumberLong(0),
                        "lastHeartbeatMessage" : "",
                        "syncingTo" : "localhost:27019",
                        "syncSourceHost" : "localhost:27019",
                        "syncSourceId" : 2,
                        "infoMessage" : "",
                        "configVersion" : 6
                },
                {
                        "_id" : 2,
                        "name" : "localhost:27019",
                        "ip" : "127.0.0.1",
                        "health" : 1,
                        "state" : 1,
                        "stateStr" : "PRIMARY",
                        "uptime" : 216,
                        "optime" : {
                                "ts" : Timestamp(1581582568, 1),
                                "t" : NumberLong(14)
                        },
                        "optimeDate" : ISODate("2020-02-13T08:29:28Z"),
                        "syncingTo" : "",
                        "syncSourceHost" : "",
                        "syncSourceId" : -1,
                        "infoMessage" : "",
                        "electionTime" : Timestamp(1581582373, 1),
                        "electionDate" : ISODate("2020-02-13T08:26:13Z"),
                        "configVersion" : 6,
                        "self" : true,
                        "lastHeartbeatMessage" : ""
                },
                {
                        "_id" : 3,
                        "name" : "localhost:27020",
                        "ip" : "127.0.0.1",
                        "health" : 1,
                        "state" : 2,
                        "stateStr" : "SECONDARY",
                        "uptime" : 194,
                        "optime" : {
                                "ts" : Timestamp(1581582508, 1),
                                "t" : NumberLong(14)
                        },
                        "optimeDurable" : {
                                "ts" : Timestamp(1581582508, 1),
                                "t" : NumberLong(14)
                        },
                        "optimeDate" : ISODate("2020-02-13T08:28:28Z"),
                        "optimeDurableDate" : ISODate("2020-02-13T08:28:28Z"),
                        "lastHeartbeat" : ISODate("2020-02-13T08:29:29.534Z"),
                        "lastHeartbeatRecv" : ISODate("2020-02-13T08:29:29.454Z"),
                        "pingMs" : NumberLong(0),
                        "lastHeartbeatMessage" : "",
                        "syncingTo" : "localhost:27019",
                        "syncSourceHost" : "localhost:27019",
                        "syncSourceId" : 2,
                        "infoMessage" : "",
                        "configVersion" : 6
                },
                {
                        "_id" : 4,
                        "name" : "localhost:27021",
                        "ip" : "127.0.0.1",
                        "health" : 1,
                        "state" : 7,
                        "stateStr" : "ARBITER",
                        "uptime" : 147,
                        "lastHeartbeat" : ISODate("2020-02-13T08:29:28.545Z"),
                        "lastHeartbeatRecv" : ISODate("2020-02-13T08:29:28.278Z"),
                        "pingMs" : NumberLong(0),
                        "lastHeartbeatMessage" : "",
                        "syncingTo" : "",
                        "syncSourceHost" : "",
                        "syncSourceId" : -1,
                        "infoMessage" : "",
                        "configVersion" : 6
                }
        ],
        "ok" : 1,
        "$clusterTime" : {
                "clusterTime" : Timestamp(1581582568, 1),
                "signature" : {
                        "hash" : BinData(0,"AAAAAAAAAAAAAAAAAAAAAAAAAAA="),
                        "keyId" : NumberLong(0)
                }
        },
        "operationTime" : Timestamp(1581582568, 1)
}
clusterGetafe:PRIMARY>
```

#### Retirar un Miembro del Cluster

* Apagar el miembro.
* Eliminar con `rs.remove("<dominio-ip>:<puerto>")`

```sh
clusterGetafe:PRIMARY> db.isMaster()
{
        "hosts" : [
                "localhost:27017",
                "localhost:27018",
                "localhost:27019"
        ],
        "arbiters" : [
                "localhost:27021"
        ],
        "setName" : "clusterGetafe",
        "setVersion" : 6,
        "ismaster" : true,
        "secondary" : false,
        "primary" : "localhost:27019",
        "me" : "localhost:27019",
        "electionId" : ObjectId("7fffffff000000000000000e"),
        "lastWrite" : {
                "opTime" : {
                        "ts" : Timestamp(1581583255, 1),
                        "t" : NumberLong(14)
                },
                "lastWriteDate" : ISODate("2020-02-13T08:40:55Z"),
                "majorityOpTime" : {
                        "ts" : Timestamp(1581583255, 1),
                        "t" : NumberLong(14)
                },
                "majorityWriteDate" : ISODate("2020-02-13T08:40:55Z")
        },
        "maxBsonObjectSize" : 16777216,
        "maxMessageSizeBytes" : 48000000,
        "maxWriteBatchSize" : 100000,
        "localTime" : ISODate("2020-02-13T08:40:59.211Z"),
        "logicalSessionTimeoutMinutes" : 30,
        "connectionId" : 21,
        "minWireVersion" : 0,
        "maxWireVersion" : 8,
        "readOnly" : false,
        "ok" : 1,
        "$clusterTime" : {
                "clusterTime" : Timestamp(1581583255, 1),
                "signature" : {
                        "hash" : BinData(0,"AAAAAAAAAAAAAAAAAAAAAAAAAAA="),
                        "keyId" : NumberLong(0)
                }
        },
        "operationTime" : Timestamp(1581583255, 1)
}
clusterGetafe:PRIMARY>                                                                                                                                                                                                                                            
```
Recordemos que teniamos el `27020` Oculto

Para el eliminar los servodores `27020` y `27021` primero los apago y luego los elimino:

```sh
clusterGetafe:PRIMARY> rs.remove("localhost:27020")
{
        "ok" : 1,
        "$clusterTime" : {
                "clusterTime" : Timestamp(1581583397, 1),
                "signature" : {
                        "hash" : BinData(0,"AAAAAAAAAAAAAAAAAAAAAAAAAAA="),
                        "keyId" : NumberLong(0)
                }
        },
        "operationTime" : Timestamp(1581583397, 1)
}
clusterGetafe:PRIMARY>
clusterGetafe:PRIMARY> rs.remove("localhost:27021")
{
        "ok" : 1,
        "$clusterTime" : {
                "clusterTime" : Timestamp(1581583469, 1),
                "signature" : {
                        "hash" : BinData(0,"AAAAAAAAAAAAAAAAAAAAAAAAAAA="),
                        "keyId" : NumberLong(0)
                }
        },
        "operationTime" : Timestamp(1581583469, 1)
}
clusterGetafe:PRIMARY>       
```

Comprobamos que los dos servidores ya estan retirados:

```sh
clusterGetafe:PRIMARY> db.isMaster()
{
        "hosts" : [
                "localhost:27017",
                "localhost:27018",
                "localhost:27019"
        ],
        "setName" : "clusterGetafe",
        "setVersion" : 8,
        "ismaster" : true,
        "secondary" : false,
        "primary" : "localhost:27019",
        "me" : "localhost:27019",
        "electionId" : ObjectId("7fffffff000000000000000e"),
        "lastWrite" : {
                "opTime" : {
                        "ts" : Timestamp(1581583488, 1),
                        "t" : NumberLong(14)
                },
                "lastWriteDate" : ISODate("2020-02-13T08:44:48Z"),
                "majorityOpTime" : {
                        "ts" : Timestamp(1581583488, 1),
                        "t" : NumberLong(14)
                },
                "majorityWriteDate" : ISODate("2020-02-13T08:44:48Z")
        },
        "maxBsonObjectSize" : 16777216,
        "maxMessageSizeBytes" : 48000000,
        "maxWriteBatchSize" : 100000,
        "localTime" : ISODate("2020-02-13T08:44:56.355Z"),
        "logicalSessionTimeoutMinutes" : 30,
        "connectionId" : 21,
        "minWireVersion" : 0,
        "maxWireVersion" : 8,
        "readOnly" : false,
        "ok" : 1,
        "$clusterTime" : {
                "clusterTime" : Timestamp(1581583488, 1),
                "signature" : {
                        "hash" : BinData(0,"AAAAAAAAAAAAAAAAAAAAAAAAAAA="),
                        "keyId" : NumberLong(0)
                }
        },
        "operationTime" : Timestamp(1581583488, 1)
}
clusterGetafe:PRIMARY>
```

Veo el Status

```sh
clusterGetafe:PRIMARY> rs.status()
{
        "set" : "clusterGetafe",
        "date" : ISODate("2020-02-13T08:49:48.907Z"),
        "myState" : 1,
        "term" : NumberLong(14),
        "syncingTo" : "",
        "syncSourceHost" : "",
        "syncSourceId" : -1,
        "heartbeatIntervalMillis" : NumberLong(2000),
        "majorityVoteCount" : 2,
        "writeMajorityCount" : 2,
        "optimes" : {
                "lastCommittedOpTime" : {
                        "ts" : Timestamp(1581583788, 1),
                        "t" : NumberLong(14)
                },
                "lastCommittedWallTime" : ISODate("2020-02-13T08:49:48.181Z"),
                "readConcernMajorityOpTime" : {
                        "ts" : Timestamp(1581583788, 1),
                        "t" : NumberLong(14)
                },
                "readConcernMajorityWallTime" : ISODate("2020-02-13T08:49:48.181Z"),
                "appliedOpTime" : {
                        "ts" : Timestamp(1581583788, 1),
                        "t" : NumberLong(14)
                },
                "durableOpTime" : {
                        "ts" : Timestamp(1581583788, 1),
                        "t" : NumberLong(14)
                },
                "lastAppliedWallTime" : ISODate("2020-02-13T08:49:48.181Z"),
                "lastDurableWallTime" : ISODate("2020-02-13T08:49:48.181Z")
        },
        "lastStableRecoveryTimestamp" : Timestamp(1581583738, 1),
        "lastStableCheckpointTimestamp" : Timestamp(1581583738, 1),
        "electionCandidateMetrics" : {
                "lastElectionReason" : "priorityTakeover",
                "lastElectionDate" : ISODate("2020-02-13T08:26:13.357Z"),
                "electionTerm" : NumberLong(14),
                "lastCommittedOpTimeAtElection" : {
                        "ts" : Timestamp(1581582364, 1),
                        "t" : NumberLong(13)
                },
                "lastSeenOpTimeAtElection" : {
                        "ts" : Timestamp(1581582364, 1),
                        "t" : NumberLong(13)
                },
                "numVotesNeeded" : 3,
                "priorityAtElection" : 2,
                "electionTimeoutMillis" : NumberLong(10000),
                "priorPrimaryMemberId" : 1,
                "numCatchUpOps" : NumberLong(0),
                "newTermStartDate" : ISODate("2020-02-13T08:26:18.058Z"),
                "wMajorityWriteAvailabilityDate" : ISODate("2020-02-13T08:26:19.599Z")
        },
        "electionParticipantMetrics" : {
                "votedForCandidate" : true,
                "electionTerm" : NumberLong(13),
                "lastVoteDate" : ISODate("2020-02-13T08:26:01.762Z"),
                "electionCandidateMemberId" : 1,
                "voteReason" : "",
                "lastAppliedOpTimeAtElection" : {
                        "ts" : Timestamp(1581512091, 1),
                        "t" : NumberLong(11)
                },
                "maxAppliedOpTimeInSet" : {
                        "ts" : Timestamp(1581512091, 1),
                        "t" : NumberLong(11)
                },
                "priorityAtElection" : 2
        },
        "members" : [
                {
                        "_id" : 0,
                        "name" : "localhost:27017",
                        "ip" : "127.0.0.1",
                        "health" : 1,
                        "state" : 2,
                        "stateStr" : "SECONDARY",
                        "uptime" : 1428,
                        "optime" : {
                                "ts" : Timestamp(1581583778, 1),
                                "t" : NumberLong(14)
                        },
                        "optimeDurable" : {
                                "ts" : Timestamp(1581583778, 1),
                                "t" : NumberLong(14)
                        },
                        "optimeDate" : ISODate("2020-02-13T08:49:38Z"),
                        "optimeDurableDate" : ISODate("2020-02-13T08:49:38Z"),
                        "lastHeartbeat" : ISODate("2020-02-13T08:49:47.876Z"),
                        "lastHeartbeatRecv" : ISODate("2020-02-13T08:49:47.885Z"),
                        "pingMs" : NumberLong(0),
                        "lastHeartbeatMessage" : "",
                        "syncingTo" : "localhost:27019",
                        "syncSourceHost" : "localhost:27019",
                        "syncSourceId" : 2,
                        "infoMessage" : "",
                        "configVersion" : 8
                },
                {
                        "_id" : 1,
                        "name" : "localhost:27018",
                        "ip" : "127.0.0.1",
                        "health" : 1,
                        "state" : 2,
                        "stateStr" : "SECONDARY",
                        "uptime" : 1428,
                        "optime" : {
                                "ts" : Timestamp(1581583778, 1),
                                "t" : NumberLong(14)
                        },
                        "optimeDurable" : {
                                "ts" : Timestamp(1581583778, 1),
                                "t" : NumberLong(14)
                        },
                        "optimeDate" : ISODate("2020-02-13T08:49:38Z"),
                        "optimeDurableDate" : ISODate("2020-02-13T08:49:38Z"),
                        "lastHeartbeat" : ISODate("2020-02-13T08:49:47.876Z"),
                        "lastHeartbeatRecv" : ISODate("2020-02-13T08:49:47.904Z"),
                        "pingMs" : NumberLong(0),
                        "lastHeartbeatMessage" : "",
                        "syncingTo" : "localhost:27019",
                        "syncSourceHost" : "localhost:27019",
                        "syncSourceId" : 2,
                        "infoMessage" : "",
                        "configVersion" : 8
                },
                {
                        "_id" : 2,
                        "name" : "localhost:27019",
                        "ip" : "127.0.0.1",
                        "health" : 1,
                        "state" : 1,
                        "stateStr" : "PRIMARY",
                        "uptime" : 1435,
                        "optime" : {
                                "ts" : Timestamp(1581583788, 1),
                                "t" : NumberLong(14)
                        },
                        "optimeDate" : ISODate("2020-02-13T08:49:48Z"),
                        "syncingTo" : "",
                        "syncSourceHost" : "",
                        "syncSourceId" : -1,
                        "infoMessage" : "",
                        "electionTime" : Timestamp(1581582373, 1),
                        "electionDate" : ISODate("2020-02-13T08:26:13Z"),
                        "configVersion" : 8,
                        "self" : true,
                        "lastHeartbeatMessage" : ""
                }
        ],
        "ok" : 1,
        "$clusterTime" : {
                "clusterTime" : Timestamp(1581583788, 1),
                "signature" : {
                        "hash" : BinData(0,"AAAAAAAAAAAAAAAAAAAAAAAAAAA="),
                        "keyId" : NumberLong(0)
                }
        },
        "operationTime" : Timestamp(1581583788, 1)
}
clusterGetafe:PRIMARY>                                                                                             

```

#### Colección `oplog` :skull:

`oplog`: Log de operaciones. Cada servidor tiene su propio `oplog` que tambien nos sirve para replicar, las operaciones las hace `idempotente` evitando problemas en la BD.


```sh
clusterGetafe:PRIMARY> show dbs
admin       0.000GB
config      0.000GB
getafeTest  0.000GB
local       0.000GB
clusterGetafe:PRIMARY>      
```

En Local esta la coleccion que registra todas las operaciones


```sh
clusterGetafe:PRIMARY> use local
switched to db local
clusterGetafe:PRIMARY> show collections
oplog.rs
replset.election
replset.minvalid
replset.oplogTruncateAfterPoint
startup_log
system.replset
system.rollback.id
clusterGetafe:PRIMARY>     
```

La que nos importara es `oplog.rs`


```sh
clusterGetafe:PRIMARY> db.oplog.rs.find()
{ "ts" : Timestamp(1581421304, 1), "h" : NumberLong(0), "v" : 2, "op" : "n", "ns" : "", "wall" : ISODate("2020-02-11T11:41:44.446Z"), "o" : { "msg" : "initiating set" } }
{ "ts" : Timestamp(1581421316, 1), "t" : NumberLong(1), "h" : NumberLong(0), "v" : 2, "op" : "c", "ns" : "config.$cmd", "ui" : UUID("8fcdfa7e-8ad2-4020-9bd4-71fa4b3a58ab"), "wall" : ISODate("2020-02-11T11:41:56.735Z"), "o" : { "create" : "transactions", "idIndex" : { "v" : 2, "key" : { "_id" : 1 }, "name" : "_id_", "ns" : "config.transactions" } } }
{ "ts" : Timestamp(1581421316, 2), "t" : NumberLong(1), "h" : NumberLong(0), "v" : 2, "op" : "n", "ns" : "", "wall" : ISODate("2020-02-11T11:41:56.735Z"), "o" : { "msg" : "new primary" } }
{ "ts" : Timestamp(1581421316, 3), "t" : NumberLong(1), "h" : NumberLong(0), "v" : 2, "op" : "c", "ns" : "admin.$cmd", "ui" : UUID("06b01d66-1d0b-488c-b739-99b26c478ea3"), "wall" : ISODate("2020-02-11T11:41:56.906Z"), "o" : { "create" : "system.keys", "idIndex" : { "v" : 2, "key" : { "_id" : 1 }, "name" : "_id_", "ns" : "admin.system.keys" } } }
{ "ts" : Timestamp(1581421316, 4), "t" : NumberLong(1), "h" : NumberLong(0), "v" : 2, "op" : "i", "ns" : "admin.system.keys", "ui" : UUID("06b01d66-1d0b-488c-b739-99b26c478ea3"), "wall" : ISODate("2020-02-11T11:41:56.907Z"), "o" : { "_id" : NumberLong("6792152833417281538"), "purpose" : "HMAC", "key" : BinData(0,"N9T/mQ67HSW1fiv6Z0XL3VaPOAc="), "expiresAt" : Timestamp(1589197316, 0) } }
{ "ts" : Timestamp(1581421317, 1), "t" : NumberLong(1), "h" : NumberLong(0), "v" : 2, "op" : "i", "ns" : "admin.system.keys", "ui" : UUID("06b01d66-1d0b-488c-b739-99b26c478ea3"), "wall" : ISODate("2020-02-11T11:41:57.886Z"), "o" : { "_id" : NumberLong("6792152833417281539"), "purpose" : "HMAC", "key" : BinData(0,"2mclaIGXlCayIqnr3dCE3pPYIpI="), "expiresAt" : Timestamp(1596973316, 0) } }
{ "ts" : Timestamp(1581421336, 1), "t" : NumberLong(1), "h" : NumberLong(0), "v" : 2, "op" : "n", "ns" : "", "wall" : ISODate("2020-02-11T11:42:16.739Z"), "o" : { "msg" : "periodic noop" } }
{ "ts" : Timestamp(1581421346, 1), "t" : NumberLong(1), "h" : NumberLong(0), "v" : 2, "op" : "n", "ns" : "", "wall" : ISODate("2020-02-11T11:42:26.740Z"), "o" : { "msg" : "periodic noop" } }
{ "ts" : Timestamp(1581421356, 1), "t" : NumberLong(1), "h" : NumberLong(0), "v" : 2, "op" : "n", "ns" : "", "wall" : ISODate("2020-02-11T11:42:36.740Z"), "o" : { "msg" : "periodic noop" } }
{ "ts" : Timestamp(1581421360, 1), "t" : NumberLong(1), "h" : NumberLong(0), "v" : 2, "op" : "c", "ns" : "config.$cmd", "ui" : UUID("e9a1ee6b-cc7f-4e9f-beeb-40d2ea281db3"), "wall" : ISODate("2020-02-11T11:42:40.515Z"), "o" : { "create" : "system.sessions", "idIndex" : { "v" : 2, "key" : { "_id" : 1 }, "name" : "_id_", "ns" : "config.system.sessions" } } }
{ "ts" : Timestamp(1581421360, 2), "t" : NumberLong(1), "h" : NumberLong(0), "v" : 2, "op" : "n", "ns" : "", "wall" : ISODate("2020-02-11T11:42:40.760Z"), "o" : { "msg" : "Creating indexes. Coll: config.system.sessions" } }
{ "ts" : Timestamp(1581421360, 3), "t" : NumberLong(1), "h" : NumberLong(0), "v" : 2, "op" : "c", "ns" : "config.$cmd", "ui" : UUID("e9a1ee6b-cc7f-4e9f-beeb-40d2ea281db3"), "wall" : ISODate("2020-02-11T11:42:40.810Z"), "o" : { "createIndexes" : "system.sessions", "v" : 2, "key" : { "lastUse" : 1 }, "name" : "lsidTTLIndex", "expireAfterSeconds" : 1800 } }
{ "ts" : Timestamp(1581421360, 4), "t" : NumberLong(1), "h" : NumberLong(0), "v" : 2, "op" : "i", "ns" : "config.system.sessions", "ui" : UUID("e9a1ee6b-cc7f-4e9f-beeb-40d2ea281db3"), "wall" : ISODate("2020-02-11T11:42:40.832Z"), "o" : { "_id" : { "id" : UUID("3aac91d5-6218-43c9-a587-d89acc6837ff"), "uid" : BinData(0,"47DEQpj8HBSa+/TImW+5JCeuQeRkm5NMpJWZG3hSuFU=") }, "lastUse" : ISODate("2020-02-11T11:42:40.832Z") } }
{ "ts" : Timestamp(1581421376, 1), "t" : NumberLong(1), "h" : NumberLong(0), "v" : 2, "op" : "n", "ns" : "", "wall" : ISODate("2020-02-11T11:42:56.741Z"), "o" : { "msg" : "periodic noop" } }
{ "ts" : Timestamp(1581421386, 1), "t" : NumberLong(1), "h" : NumberLong(0), "v" : 2, "op" : "n", "ns" : "", "wall" : ISODate("2020-02-11T11:43:06.742Z"), "o" : { "msg" : "periodic noop" } }
{ "ts" : Timestamp(1581421396, 1), "t" : NumberLong(1), "h" : NumberLong(0), "v" : 2, "op" : "n", "ns" : "", "wall" : ISODate("2020-02-11T11:43:16.742Z"), "o" : { "msg" : "periodic noop" } }
{ "ts" : Timestamp(1581421406, 1), "t" : NumberLong(1), "h" : NumberLong(0), "v" : 2, "op" : "n", "ns" : "", "wall" : ISODate("2020-02-11T11:43:26.742Z"), "o" : { "msg" : "periodic noop" } }
{ "ts" : Timestamp(1581421416, 1), "t" : NumberLong(1), "h" : NumberLong(0), "v" : 2, "op" : "n", "ns" : "", "wall" : ISODate("2020-02-11T11:43:36.743Z"), "o" : { "msg" : "periodic noop" } }
{ "ts" : Timestamp(1581421426, 1), "t" : NumberLong(1), "h" : NumberLong(0), "v" : 2, "op" : "n", "ns" : "", "wall" : ISODate("2020-02-11T11:43:46.744Z"), "o" : { "msg" : "periodic noop" } }
{ "ts" : Timestamp(1581421436, 1), "t" : NumberLong(1), "h" : NumberLong(0), "v" : 2, "op" : "n", "ns" : "", "wall" : ISODate("2020-02-11T11:43:56.744Z"), "o" : { "msg" : "periodic noop" } }
Type "it" for more
clusterGetafe:PRIMARY>  

```

Hago una operación:

```sh
clusterGetafe:PRIMARY> use getafeTest
switched to db getafeTest
clusterGetafe:PRIMARY> show collections
biblioteca
foo
foo5
clusterGetafe:PRIMARY> db.foo2.insert({puntuacion: 0})
WriteResult({ "nInserted" : 1 })
clusterGetafe:PRIMARY>                       
```

Hago una operación "dinámica" que cada vez que la ejecutara incrementaria:

```sh
clusterGetafe:PRIMARY> db.foo2.update({ puntuacion: 0 }, { $inc: { puntuacion: 1} } )
WriteResult({ "nMatched" : 1, "nUpserted" : 0, "nModified" : 1 })
clusterGetafe:PRIMARY>
```
Regreso a `oplog`

```sh
clusterGetafe:PRIMARY> db.oplog.rs.find().sort({$natural: -1})
{ "ts" : Timestamp(1581585038, 1), "t" : NumberLong(14), "h" : NumberLong(0), "v" : 2, "op" : "n", "ns" : "", "wall" : ISODate("2020-02-13T09:10:38.277Z"), "o" : { "msg" : "periodic noop" } }
{ "ts" : Timestamp(1581585028, 1), "t" : NumberLong(14), "h" : NumberLong(0), "v" : 2, "op" : "n", "ns" : "", "wall" : ISODate("2020-02-13T09:10:28.276Z"), "o" : { "msg" : "periodic noop" } }
{ "ts" : Timestamp(1581585018, 1), "t" : NumberLong(14), "h" : NumberLong(0), "v" : 2, "op" : "n", "ns" : "", "wall" : ISODate("2020-02-13T09:10:18.276Z"), "o" : { "msg" : "periodic noop" } }
{ "ts" : Timestamp(1581585008, 1), "t" : NumberLong(14), "h" : NumberLong(0), "v" : 2, "op" : "n", "ns" : "", "wall" : ISODate("2020-02-13T09:10:08.274Z"), "o" : { "msg" : "periodic noop" } }
{ "ts" : Timestamp(1581584998, 1), "t" : NumberLong(14), "h" : NumberLong(0), "v" : 2, "op" : "n", "ns" : "", "wall" : ISODate("2020-02-13T09:09:58.274Z"), "o" : { "msg" : "periodic noop" } }
{ "ts" : Timestamp(1581584988, 1), "t" : NumberLong(14), "h" : NumberLong(0), "v" : 2, "op" : "n", "ns" : "", "wall" : ISODate("2020-02-13T09:09:48.273Z"), "o" : { "msg" : "periodic noop" } }
{ "ts" : Timestamp(1581584978, 1), "t" : NumberLong(14), "h" : NumberLong(0), "v" : 2, "op" : "n", "ns" : "", "wall" : ISODate("2020-02-13T09:09:38.272Z"), "o" : { "msg" : "periodic noop" } }
{ "ts" : Timestamp(1581584968, 1), "t" : NumberLong(14), "h" : NumberLong(0), "v" : 2, "op" : "n", "ns" : "", "wall" : ISODate("2020-02-13T09:09:28.272Z"), "o" : { "msg" : "periodic noop" } }
{ "ts" : Timestamp(1581584958, 1), "t" : NumberLong(14), "h" : NumberLong(0), "v" : 2, "op" : "n", "ns" : "", "wall" : ISODate("2020-02-13T09:09:18.271Z"), "o" : { "msg" : "periodic noop" } }
{ "ts" : Timestamp(1581584948, 1), "t" : NumberLong(14), "h" : NumberLong(0), "v" : 2, "op" : "n", "ns" : "", "wall" : ISODate("2020-02-13T09:09:08.270Z"), "o" : { "msg" : "periodic noop" } }
{ "ts" : Timestamp(1581584938, 1), "t" : NumberLong(14), "h" : NumberLong(0), "v" : 2, "op" : "n", "ns" : "", "wall" : ISODate("2020-02-13T09:08:58.269Z"), "o" : { "msg" : "periodic noop" } }
{ "ts" : Timestamp(1581584928, 1), "t" : NumberLong(14), "h" : NumberLong(0), "v" : 2, "op" : "n", "ns" : "", "wall" : ISODate("2020-02-13T09:08:48.269Z"), "o" : { "msg" : "periodic noop" } }
{ "ts" : Timestamp(1581584918, 1), "t" : NumberLong(14), "h" : NumberLong(0), "v" : 2, "op" : "n", "ns" : "", "wall" : ISODate("2020-02-13T09:08:38.268Z"), "o" : { "msg" : "periodic noop" } }
{ "ts" : Timestamp(1581584905, 1), "t" : NumberLong(14), "h" : NumberLong(0), "v" : 2, "op" : "u", "ns" : "getafeTest.foo2", "ui" : UUID("5b8b0548-73ae-4a1e-b62f-1ec885e8c1f8"), "o2" : { "_id" : ObjectId("5e451197f37a0bc48f6bcaca") }, "wall" : ISODate("2020-02-13T09:08:25.601Z"), "o" : { "$v" : 1, "$set" : { "puntuacion" : 1 } } }
{ "ts" : Timestamp(1581584898, 1), "t" : NumberLong(14), "h" : NumberLong(0), "v" : 2, "op" : "n", "ns" : "", "wall" : ISODate("2020-02-13T09:08:18.267Z"), "o" : { "msg" : "periodic noop" } }
{ "ts" : Timestamp(1581584888, 1), "t" : NumberLong(14), "h" : NumberLong(0), "v" : 2, "op" : "n", "ns" : "", "wall" : ISODate("2020-02-13T09:08:08.266Z"), "o" : { "msg" : "periodic noop" } }
{ "ts" : Timestamp(1581584878, 1), "t" : NumberLong(14), "h" : NumberLong(0), "v" : 2, "op" : "n", "ns" : "", "wall" : ISODate("2020-02-13T09:07:58.266Z"), "o" : { "msg" : "periodic noop" } }
{ "ts" : Timestamp(1581584868, 1), "t" : NumberLong(14), "h" : NumberLong(0), "v" : 2, "op" : "n", "ns" : "", "wall" : ISODate("2020-02-13T09:07:48.266Z"), "o" : { "msg" : "periodic noop" } }
{ "ts" : Timestamp(1581584858, 1), "t" : NumberLong(14), "h" : NumberLong(0), "v" : 2, "op" : "n", "ns" : "", "wall" : ISODate("2020-02-13T09:07:38.265Z"), "o" : { "msg" : "periodic noop" } }
{ "ts" : Timestamp(1581584848, 1), "t" : NumberLong(14), "h" : NumberLong(0), "v" : 2, "op" : "n", "ns" : "", "wall" : ISODate("2020-02-13T09:07:28.264Z"), "o" : { "msg" : "periodic noop" } }
Type "it" for more
clusterGetafe:PRIMARY>                                                                                                             
```

Puedo observar que mi operación me la convirtio en IDEMPOTENTE

```sh
{ "ts" : Timestamp(1581584905, 1), "t" : NumberLong(14), "h" : NumberLong(0), "v" : 2, "op" : "u", "ns" : "getafeTest.foo2", "ui" : UUID("5b8b0548-73ae-4a1e-b62f-1ec885e8c1f8"), "o2" : { "_id" : ObjectId("5e451197f37a0bc48f6bcaca") }, "wall" : ISODate("2020-02-13T09:08:25.601Z"), "o" : { "$v" : 1, "$set" : { "puntuacion" : 1 } } }
```

**El `oplog` recibe las operaciones de actualización y las converte en IDEMPOTENTES para pasarselas al secundario y este secundario una vez que la tiene las ejecuta de forma idempotente.**

**Hay algunas operaciones que para convertirlas IDEMPOTENTE las explota en varias condiciones.**



```sh
```

```sh
```




