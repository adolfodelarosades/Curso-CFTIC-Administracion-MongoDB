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









