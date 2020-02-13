# 8. Sharding

[Sharding](https://docs.mongodb.com/manual/sharding/)

Mecanismo de escalado horizontal.

Si mi primario se cuelga por que no es capaz de abastecer el volumen de trafico en ciertos momentos, una alternativa es tener varias maquinas con menos requisitos por que el Escalado Vertical es más caro.

En lugar de tener un replica set permite tener colecciones fragmentadas en varios clusters a las que les da una clave y las peticiones las reparte a todos los que tienen una clave.

