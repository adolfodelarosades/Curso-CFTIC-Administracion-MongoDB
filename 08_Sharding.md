# 8. Sharding

[Sharding](https://docs.mongodb.com/manual/sharding/)

Mecanismo de escalado horizontal.

Componentes:

* Mongos (Enrutadores)
* Cluster Config Servers
* Cluster Sharded

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







