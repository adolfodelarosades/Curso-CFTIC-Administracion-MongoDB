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

