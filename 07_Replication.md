# Replication

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

Esto arranca la shell de este puerto:

<img 


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

Pasar este objeto a `rs.initiate()`:

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



