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











