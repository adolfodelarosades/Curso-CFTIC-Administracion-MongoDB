# 8. Sharding

[Sharding](https://docs.mongodb.com/manual/sharding/)

[Shard Keys](https://docs.mongodb.com/manual/core/sharding-shard-key/)

Mecanismo de escalado horizontal. Para proyectos muy grandes.

Componentes:

* Mongos (Enrutadores)
* Cluster Config Servers
* Cluster Sharded

Despliegue:

1. Desplegar el cluster Config Servers 
   idem cluster replica set
   * Al iniciar añadir `--configsvr`
   * En `rs.initiate()` añadir `configsvr: true`

2. Desplegar los shards(2)



**DIAGRAMA APUNTES**

<img src="/images/sharded.svg">

Si mi primario se cuelga por que no es capaz de abastecer el volumen de trafico en ciertos momentos, una alternativa es tener varias maquinas con menos requisitos por que el Escalado Vertical es más caro.

En lugar de tener un replica set permite tener colecciones fragmentadas en varios clusters a las que les da una clave y las peticiones las reparte a todos los que tienen una clave.

¿Cómo fracmento los documentos?

Los A-M a un cluster y de la N-Z a otro. (Para la aplicación esto es transparente).

**La pega es que la clave la tengo que elegir al principio y despues ya no la puedo modificar.**

Si aumentara A-M mucho ya no puedo cambiarlo por lo anterior.

Se pueden establecer zonas geograficas para que atiendan trafico.

Es muy flexible solo la clave es la pega (En caliente).

La clave va por colecciones.


PRACTICA

Creo en `c:\Usuarios\manana` creo las carpetas `datash` y dentro `configServer1`, `configServer2` y `configServer3`
Los levantamos en los servers `27100`, `27101` y `27102` 

Nombre del cluster `configServerGetafe`

Creamos los cluster pertenecientes al Sarding:
```sh
C:\Users\manana>mongod --dbpath datash\configServer1 --port 27100 -replSet configServerGetafe --configsvr
C:\Users\manana>mongod --dbpath datash\configServer2 --port 27101 -replSet configServerGetafe --configsvr
C:\Users\manana>mongod --dbpath datash\configServer3 --port 27102 -replSet configServerGetafe --configsvr
```
Con `--configsvr` sabe que este cluster va a pertenecer a un Sarding

Levanto el `27100`
```sh
C:\Users\manana>mongo --port 27100
```

Y cargo la configuración:

```sh
> rs.initiate({
... _id: "configServerGetafe",
... configsvr: true,
... members: [
... {_id: 0, host: "localhost:27100"},
... {_id: 1, host: "localhost:27101"},
... {_id: 2, host: "localhost:27102"}
... ]
... })
```
Esto hace que las carpetas creadas se llenen de información.

Esto se hace cada que quiero levantar el Replica Set lo puedo automatizar.

Crear dos carpetas en la carpeta `datash` para los dos sarders `shServer1` y `shServer2` darles los puertos `27200`, `27201`

Hay que levantarlos 

```sh
C:\Users\manana>mongod --dbpath datash\shServer1 --port 27200 --shardsvr
C:\Users\manana>mongod --dbpath datash\shServer2 --port 27201 --shardsvr
```
`--shardsvr` Esto indica que pertenece al Sarding



Levantar mis servidores:

```sh
C:\Users\manana>mongod --dbpath datash\configServer1 --port 27100 -replSet configServerGetafe --configsvr
C:\Users\manana>mongod --dbpath datash\configServer2 --port 27101 -replSet configServerGetafe --configsvr
C:\Users\manana>mongod --dbpath datash\configServer3 --port 27102 -replSet configServerGetafe --configsvr
```

Me conecto con una shell a uno de ellos:

```sh
C:\Users\manana>mongo --port 27100
>
>configServerGetafe:PRIMARY> rs.status()
{
        "set" : "configServerGetafe",
        "date" : ISODate("2020-02-14T08:24:38.593Z"),
        "myState" : 1,
        "term" : NumberLong(2),
        "syncingTo" : "",
        "syncSourceHost" : "",
        "syncSourceId" : -1,
        "configsvr" : true,
        "heartbeatIntervalMillis" : NumberLong(2000),
        "majorityVoteCount" : 2,
        "writeMajorityCount" : 2,
        "optimes" : {
                "lastCommittedOpTime" : {
                        "ts" : Timestamp(1581668675, 1),
                        "t" : NumberLong(2)
                },
                "lastCommittedWallTime" : ISODate("2020-02-14T08:24:35.690Z"),
                "readConcernMajorityOpTime" : {
                        "ts" : Timestamp(1581668675, 1),
                        "t" : NumberLong(2)
                },
                "readConcernMajorityWallTime" : ISODate("2020-02-14T08:24:35.690Z"),
                "appliedOpTime" : {
                        "ts" : Timestamp(1581668675, 1),
                        "t" : NumberLong(2)
                },
                "durableOpTime" : {
                        "ts" : Timestamp(1581668675, 1),
                        "t" : NumberLong(2)
                },
                "lastAppliedWallTime" : ISODate("2020-02-14T08:24:35.690Z"),
                "lastDurableWallTime" : ISODate("2020-02-14T08:24:35.690Z")
        },
        "lastStableRecoveryTimestamp" : Timestamp(1581668625, 1),
        "lastStableCheckpointTimestamp" : Timestamp(1581668625, 1),
        "electionCandidateMetrics" : {
                "lastElectionReason" : "electionTimeout",
                "lastElectionDate" : ISODate("2020-02-14T08:23:31.143Z"),
                "electionTerm" : NumberLong(2),
                "lastCommittedOpTimeAtElection" : {
                        "ts" : Timestamp(0, 0),
                        "t" : NumberLong(-1)
                },
                "lastSeenOpTimeAtElection" : {
                        "ts" : Timestamp(1581598511, 1),
                        "t" : NumberLong(1)
                },
                "numVotesNeeded" : 2,
                "priorityAtElection" : 1,
                "electionTimeoutMillis" : NumberLong(10000),
                "numCatchUpOps" : NumberLong(0),
                "newTermStartDate" : ISODate("2020-02-14T08:23:35.613Z"),
                "wMajorityWriteAvailabilityDate" : ISODate("2020-02-14T08:23:35.954Z")
        },
        "members" : [
                {
                        "_id" : 0,
                        "name" : "localhost:27100",
                        "ip" : "127.0.0.1",
                        "health" : 1,
                        "state" : 1,
                        "stateStr" : "PRIMARY",
                        "uptime" : 106,
                        "optime" : {
                                "ts" : Timestamp(1581668675, 1),
                                "t" : NumberLong(2)
                        },
                        "optimeDate" : ISODate("2020-02-14T08:24:35Z"),
                        "syncingTo" : "",
                        "syncSourceHost" : "",
                        "syncSourceId" : -1,
                        "infoMessage" : "could not find member to sync from",
                        "electionTime" : Timestamp(1581668611, 1),
                        "electionDate" : ISODate("2020-02-14T08:23:31Z"),
                        "configVersion" : 1,
                        "self" : true,
                        "lastHeartbeatMessage" : ""
                },
                {
                        "_id" : 1,
                        "name" : "localhost:27101",
                        "ip" : "127.0.0.1",
                        "health" : 1,
                        "state" : 2,
                        "stateStr" : "SECONDARY",
                        "uptime" : 77,
                        "optime" : {
                                "ts" : Timestamp(1581668675, 1),
                                "t" : NumberLong(2)
                        },
                        "optimeDurable" : {
                                "ts" : Timestamp(1581668675, 1),
                                "t" : NumberLong(2)
                        },
                        "optimeDate" : ISODate("2020-02-14T08:24:35Z"),
                        "optimeDurableDate" : ISODate("2020-02-14T08:24:35Z"),
                        "lastHeartbeat" : ISODate("2020-02-14T08:24:37.314Z"),
                        "lastHeartbeatRecv" : ISODate("2020-02-14T08:24:38.437Z"),
                        "pingMs" : NumberLong(0),
                        "lastHeartbeatMessage" : "",
                        "syncingTo" : "localhost:27100",
                        "syncSourceHost" : "localhost:27100",
                        "syncSourceId" : 0,
                        "infoMessage" : "",
                        "configVersion" : 1
                },
                {
                        "_id" : 2,
                        "name" : "localhost:27102",
                        "ip" : "127.0.0.1",
                        "health" : 1,
                        "state" : 2,
                        "stateStr" : "SECONDARY",
                        "uptime" : 58,
                        "optime" : {
                                "ts" : Timestamp(1581668675, 1),
                                "t" : NumberLong(2)
                        },
                        "optimeDurable" : {
                                "ts" : Timestamp(1581668675, 1),
                                "t" : NumberLong(2)
                        },
                        "optimeDate" : ISODate("2020-02-14T08:24:35Z"),
                        "optimeDurableDate" : ISODate("2020-02-14T08:24:35Z"),
                        "lastHeartbeat" : ISODate("2020-02-14T08:24:38.090Z"),
                        "lastHeartbeatRecv" : ISODate("2020-02-14T08:24:37.208Z"),
                        "pingMs" : NumberLong(0),
                        "lastHeartbeatMessage" : "",
                        "syncingTo" : "localhost:27101",
                        "syncSourceHost" : "localhost:27101",
                        "syncSourceId" : 1,
                        "infoMessage" : "",
                        "configVersion" : 1
                }
        ],
        "ok" : 1,
        "$gleStats" : {
                "lastOpTime" : Timestamp(0, 0),
                "electionId" : ObjectId("7fffffff0000000000000002")
        },
        "lastCommittedOpTime" : Timestamp(1581668675, 1),
        "$clusterTime" : {
                "clusterTime" : Timestamp(1581668675, 1),
                "signature" : {
                        "hash" : BinData(0,"AAAAAAAAAAAAAAAAAAAAAAAAAAA="),
                        "keyId" : NumberLong(0)
                }
        },
        "operationTime" : Timestamp(1581668675, 1)
}
configServerGetafe:PRIMARY>

```


Levantar los dos servidores sharding

```sh
C:\Users\manana>mongod --dbpath datash\shServer1 --port 27200 --shardsvr
C:\Users\manana>mongod --dbpath datash\shServer2 --port 27201 --shardsvr
```


Tenemos levantado el config Server Cluster (Tres Servidores Conectados entre si) en los puertos 27100, 27101 y 27102 que solo tendran metadatos (son más pequeños).

Y tenemos dos servidores más grandes levantados en el 27200 y 27201 estos van a recibir los datos.

Falta un tercer componente `mongos` (un binario) el enrutador que tambien es un servidor, que es el encargado de repartir los datos ente mis dos sardins. No le hace falta su carpeta. 


Levantar Mongos

```sh
C:\Users\manana>mongos --configdb configServerGetafe/localhost:27100,localhost:27101,localhost:27102 --port 27300
```

Levantamos una shell y levantamos la `27300`

```sh
C:\Users\manana>mongo --port 27300
...
mongos>
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

