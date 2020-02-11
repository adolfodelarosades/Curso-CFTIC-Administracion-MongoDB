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

<img src="/images/replicaset.svg">
(Apuntes)


El primary replica todo en los secundarios, cada que escribo en el primario escribe en los secundarios con una conexión Asíncrona, para que en caso que el primario se caiga por cualquier razon, automaticamente uno de los dos secundarios se levanta como primario con algun criterio establecido tarda de 10 - 90 seg. Si se levanta el primario caido vuelve a ser el primario.

Planteamientos:

* Algunos implementan el primario y secundario en el mismo lugar fisico así la asíncronia mínima y el otro secundario lo hago en otra ubicación fisica o en la nube.

* Todo a la nube, Azure, Amazon. Puedo desplegar maquinas con Mongo con Comunity o Enterprise(Soporte, Ops Manager). No hay un despliegue 

* Atlas. Todo automatizado. Con soporte especifico. No tienen sus granjas propias.(Herramienta Clust Manager)






