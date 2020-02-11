# 7 Replication

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









