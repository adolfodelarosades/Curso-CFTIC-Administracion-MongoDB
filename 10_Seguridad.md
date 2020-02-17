# 10. Seguridad

[Security](https://docs.mongodb.com/manual/security/index.html)

## Autenticación

[Authentication](https://docs.mongodb.com/manual/core/authentication/)

Mecanismos de Autenticación (Community)

* SCRAM (Defecto)
* x.509

Adicionales en la versión Enterprise

* LDAP proxy authentication
* Kerberos auth

* La autenticación de los usuarios al sistema gestor de base de datos se lleva a cabo a nivel de base de datos.

1. Creación de usuarios.
   Método `createUser()`
   
   ```sh
   db.createUser({
      user: "<nombre-usuario>",
      pwd: "<contraseña>",
      customData: <documento-datos>,  // Opcional
      roles: [
        ... roles y permisos
      ]
    })
   ```

**MongoDB no te obliga a levantar servidores sin autorización.**


Abrimos el servidor convencional


```sh
C:\Users\manana>mongod

C:\Users\manana>mongo

> show dbs
admin           0.000GB
biblioteca      0.000GB
clinica         0.000GB
clinica2        0.000GB
config          0.000GB
gimnasio        0.059GB
gimnasio2       0.000GB
ingenieria      0.000GB
local           0.000GB
maraton         0.047GB
maraton2        0.000GB
shop            0.000GB
shop2           0.000GB
shop3           0.000GB
shop4           0.000GB
sweetscomplete  0.000GB
test            0.000GB
>
```

Vamos a crear un usuario que tenga acceso a la base de datos `maraton`.

[Add Users](https://docs.mongodb.com/manual/tutorial/create-users/)

```sh
> use maraton
switched to db maraton

> db.createUser({
... user: "adolfo",
... pwd: "adolfo1234",
... customData: {dni: "X883838S"},
... roles: [
... "readWrite"
... ]
... })
Successfully added user: {
        "user" : "adolfo",
        "customData" : {
                "dni" : "X883838S"
        },
        "roles" : [
                "readWrite"
        ]
}
>      
```

MongoDB centraliza esta información en la BD admin 


```sh
> use admin
switched to db admin

> show collections
system.users
system.version

> db.system.users.find().pretty()
{
        "_id" : "maraton.adolfo",
        "userId" : UUID("182196d1-9937-4064-9a05-34169e6cb811"),
        "user" : "adolfo",
        "db" : "maraton",
        "credentials" : {
                "SCRAM-SHA-1" : {
                        "iterationCount" : 10000,
                        "salt" : "kj6Du7fgSrNNzRB0akw8SQ==",
                        "storedKey" : "d3PujkUfp3Gqsycqr74TKsBUzK8=",
                        "serverKey" : "rNYmvZipC69mvD7CQz1rWnYmxOk="
                },
                "SCRAM-SHA-256" : {
                        "iterationCount" : 15000,
                        "salt" : "GHkL+no0vLlfmqQL7p+iqk3JhWlZJSQk7fPQoA==",
                        "storedKey" : "f6BCcaBRfpm/NsD/8rybFn/u6BYzcQNoMntNRYSXINI=",
                        "serverKey" : "cJ8TeVXuSq/LrAr/hTOzxf6xBDOQnw0DMLL87O0qRsI="
                }
        },
        "customData" : {
                "dni" : "X883838S"
        },
        "roles" : [
                {
                        "role" : "readWrite",
                        "db" : "maraton"
                }
        ]
}
> 
```
Puedo ver todas las caracteristicas del usuario creado.

2. Paso.

Crear un Super Usuario para que trabaje en todas las BD incluyendo las BD del sistema. Tengo que estar en la BD `admin`:

```sh
> use admin
switched to db admin
> db.createUser({
... user: "superAdmin",
... pwd: "NoHay2sin3",
... roles: [
... { role: "userAdminAnyDatabase", db: "admin" },
... "readWriteAnyDatabase"
... ]
... })
Successfully added user: {
        "user" : "superAdmin",
        "roles" : [
                {
                        "role" : "userAdminAnyDatabase",
                        "db" : "admin"
                },
                "readWriteAnyDatabase"
        ]
}
>               
```

Veo la colección:

```sh
> db.system.users.find().pretty()
{
        "_id" : "maraton.adolfo",
        "userId" : UUID("182196d1-9937-4064-9a05-34169e6cb811"),
        "user" : "adolfo",
        "db" : "maraton",
        "credentials" : {
                "SCRAM-SHA-1" : {
                        "iterationCount" : 10000,
                        "salt" : "kj6Du7fgSrNNzRB0akw8SQ==",
                        "storedKey" : "d3PujkUfp3Gqsycqr74TKsBUzK8=",
                        "serverKey" : "rNYmvZipC69mvD7CQz1rWnYmxOk="
                },
                "SCRAM-SHA-256" : {
                        "iterationCount" : 15000,
                        "salt" : "GHkL+no0vLlfmqQL7p+iqk3JhWlZJSQk7fPQoA==",
                        "storedKey" : "f6BCcaBRfpm/NsD/8rybFn/u6BYzcQNoMntNRYSXINI=",
                        "serverKey" : "cJ8TeVXuSq/LrAr/hTOzxf6xBDOQnw0DMLL87O0qRsI="
                }
        },
        "customData" : {
                "dni" : "X883838S"
        },
        "roles" : [
                {
                        "role" : "readWrite",
                        "db" : "maraton"
                }
        ]
}
{
        "_id" : "admin.superAdmin",
        "userId" : UUID("227cd5bc-7d73-4983-8965-3b80accd14b2"),
        "user" : "superAdmin",
        "db" : "admin",
        "credentials" : {
                "SCRAM-SHA-1" : {
                        "iterationCount" : 10000,
                        "salt" : "pwaGtp7fu+r/3rKwPwJvJQ==",
                        "storedKey" : "YGGYI7gPhlSUfWsySjwsTFs6g+Q=",
                        "serverKey" : "rVXzKhE+H/YxEXDCnUu1Ygw/ZdI="
                },
                "SCRAM-SHA-256" : {
                        "iterationCount" : 15000,
                        "salt" : "mG7SSBJiFxrBeRx35qaxEUpSM/1q2SOdS8iSsA==",
                        "storedKey" : "/1sYk7qqhJsiqMAitM588DXvR9OL0YUUE1qA5BWXEzE=",
                        "serverKey" : "c1xSAg2IAiINeIAY6+WRg1WAIn1aY9F7J2jFKepYdaQ="
                }
        },
        "roles" : [
                {
                        "role" : "userAdminAnyDatabase",
                        "db" : "admin"
                },
                {
                        "role" : "readWriteAnyDatabase",
                        "db" : "admin"
                }
        ]
}
>     
```

Ya tengo el nuevo usuario creado.


Paramos nuestro servidor y lo levantamos con la autenticación activada.


```sh
C:\Users\manana>mongod --auth
```
Con esto ya me pedira la autenticación.

```sh
C:\Users\manana>mongo
MongoDB shell version v4.2.2
connecting to: mongodb://127.0.0.1:27017/?compressors=disabled&gssapiServiceName=mongodb
Implicit session: session { "id" : UUID("7ea12432-9572-4ae3-a90f-bdfd623aa630") }
MongoDB server version: 4.2.2
>
```


```sh
C:\Users\manana>mongo
MongoDB shell version v4.2.2
connecting to: mongodb://127.0.0.1:27017/?compressors=disabled&gssapiServiceName=mongodb
Implicit session: session { "id" : UUID("7ea12432-9572-4ae3-a90f-bdfd623aa630") }
MongoDB server version: 4.2.2
> ^C
bye

C:\Users\manana>mongo --authenticationDatabase "maraton" -u "adolfo"
MongoDB shell version v4.2.2
Enter password:

connecting to: mongodb://127.0.0.1:27017/?authSource=maraton&compressors=disabled&gssapiServiceName=mongodb
Implicit session: session { "id" : UUID("ed104782-432f-4a23-a7ba-a766a6129c21") }
MongoDB server version: 4.2.2

> show dbs
maraton  0.047GB
>
```
Como entre con el usuario que solo tiene acceso a maraton.


Si entro solo con `mongo`
```sh
C:\Users\manana>mongo
MongoDB shell version v4.2.2
connecting to: mongodb://127.0.0.1:27017/?compressors=disabled&gssapiServiceName=mongodb
Implicit session: session { "id" : UUID("f67a0b80-8cc6-429b-907b-8b20e8efc9b8") }
MongoDB server version: 4.2.2
> show dbs
>
```
No veo nada

Si en la shell que me conecte bien intento cambiarme de BD me deja pero no puedo hacer nada sobre ella:

```sh
> use gimnasio
switched to db gimnasio
> show collections
Warning: unable to run listCollections, attempting to approximate collection names by parsing connectionStatus
>
```

**CUANDO ENTRO SIEMPRE ENTRA A test DEBO SIEMPRE CAMBAIRRME A MI BD `maraton`**

Estando en mi BD ya puedo hacer cualquier operación.

**Me puedo autenticar desde adentro:**

```sh
C:\Users\manana>mongo
MongoDB shell version v4.2.2
connecting to: mongodb://127.0.0.1:27017/?compressors=disabled&gssapiServiceName=mongodb
Implicit session: session { "id" : UUID("8c46c31a-9331-4c25-9f53-8fe27f0f7274") }
MongoDB server version: 4.2.2
> show dbs
> use maraton
switched to db maraton
> db.auth("adolfo","adolfo1234")
1
>       
```
He entrado solo con Mongo y desde dentro me Logeo.


**Conectarme al Super Usuario**

```sh
C:\Users\manana>mongo --authenticationDatabase "admin" -u "superAdmin"
MongoDB shell version v4.2.2
Enter password:
connecting to: mongodb://127.0.0.1:27017/?authSource=admin&compressors=disabled&gssapiServiceName=mongodb
Implicit session: session { "id" : UUID("c713ee44-7097-440d-bbab-6747d40b4aa0") }
MongoDB server version: 4.2.2

> show dbs
admin           0.000GB
biblioteca      0.000GB
clinica         0.000GB
clinica2        0.000GB
config          0.000GB
gimnasio        0.059GB
gimnasio2       0.000GB
ingenieria      0.000GB
local           0.000GB
maraton         0.047GB
maraton2        0.000GB
shop            0.000GB
shop2           0.000GB
shop3           0.000GB
shop4           0.000GB
sweetscomplete  0.000GB
test            0.000GB
>            
```

**Para logearme desde dentro para el superUsuario tengo que estar en la BD `admin`**


## ROLES

[Built-In Roles](https://docs.mongodb.com/manual/reference/built-in-roles/index.html)
-Colección systrem.users

Roles propios de MongoDB

Los usuarios se crean a nivel de base de datos

* rol "read"
* rol "readWrite"
* rol "dbAdmin" => Idem a readWrite con permisos de operaciones sobre el system.profile
* rol "userAdmin" = Permite al usuario administrar los usuarios de esa base de datos **A NIVEL DE BD** .
* rol "dbOwner" => Engloba los anteriores (Super usuario a nivel de BD)


Roles para todas las bases de datos(deben estar asociados a la base de datos admin)

* rol "readAnyDatabase" 
* rol "readWriteAnyDatabase"
* rol "dbAdminAnyDatabase" idem dbAdmin para todas las bases de datos
* rol "userAdminAnyDatabase" Administrar usuarios para todas las BD **A NIVEL DE TODAS LAS BD**

Tener claro que un usuario tiene permsos para su base de datos(escritura) y otras para admin (readAnyDatabase)


Roles relacionados con los Clusters


* rol "clusterMonitor" // Permisos de lectura de tareas de monitorización de cluster
* rol "hostManager" // Permisos de gestión de servidores del cluster
* rol "clusterManager" // Permisos de configuración del cluster 
* rol "clusterAdmin" // Todos los permisos relacionados con la administración del cluster.

Roles de Backup y Restauración

* rol "backup"
* rol "restore"

Rol con todos los permisos

* rol "root"

Roles User-defined

Método `createRole()`: Permite crear roles que se almacenan el la BD admin a los que le añadimos privilegios que actuan sobre recursos y acciones.

Eliminar Usuarios a nivel de Base de Datos.

````sh
db.dropUser("<nombre>")
db.dropAllUsers()
```

Métodos de Administración de Usarios.

[User Management Methods](https://docs.mongodb.com/manual/reference/method/js-user-management/)





Cargo mi servidor con `mongod --auth` **IMPORTANTE** Sino paso de los permisos.

Entro con mi usuario Super Admin

```sh
C:\Users\manana>mongo --authenticationDatabase "admin" -u "superAdmin"
MongoDB shell version v4.2.2
Enter password: NoHay2sin3
connecting to: mongodb://127.0.0.1:27017/?authSource=admin&compressors=disabled&gssapiServiceName=mongodb
Implicit session: session { "id" : UUID("c713ee44-7097-440d-bbab-6747d40b4aa0") }
MongoDB server version: 4.2.2
```

Creo usuario:

```sh
> db.createUser({ user: "juan73", pwd: "juan1234", roles: ["read"] })
Successfully added user: { "user" : "juan73", "roles" : [ "read" ] }
```

Ver privilegios del usuarrio creado:

```sh
> db.runCommand({ usersInfo: "juan73", showPrivileges: true})
{
        "users" : [
                {
                        "_id" : "gimnasio.juan73",
                        "userId" : UUID("3f3b060a-875a-4a67-ae4c-033f83670b41"),
                        "user" : "juan73",
                        "db" : "gimnasio",
                        "mechanisms" : [
                                "SCRAM-SHA-1",
                                "SCRAM-SHA-256"
                        ],
                        "roles" : [
                                {
                                        "role" : "read",
                                        "db" : "gimnasio"
                                }
                        ],
                        "inheritedRoles" : [
                                {
                                        "role" : "read",
                                        "db" : "gimnasio"
                                }
                        ],
                        "inheritedPrivileges" : [
                                {
                                        "resource" : {
                                                "db" : "gimnasio",
                                                "collection" : ""
                                        },
                                        "actions" : [
                                                "changeStream",
                                                "collStats",
                                                "dbHash",
                                                "dbStats",
                                                "find",
                                                "killCursors",
                                                "listCollections",
                                                "listIndexes",
                                                "planCacheRead"
                                        ]
                                },
                                {
                                        "resource" : {
                                                "db" : "gimnasio",
                                                "collection" : "system.js"
                                        },
                                        "actions" : [
                                                "changeStream",
                                                "collStats",
                                                "dbHash",
                                                "dbStats",
                                                "find",
                                                "killCursors",
                                                "listCollections",
                                                "listIndexes",
                                                "planCacheRead"
                                        ]
                                }
                        ],
                        "inheritedAuthenticationRestrictions" : [ ]
                }
        ],
        "ok" : 1
}
>
```


Entro en otra consola con el usuario de "juan73".

```sh
C:\Users\manana>mongo --authenticationDatabase "gimnasio" -u "juan73"
```

En teoría solo puedo leer en mi base de datos.
**COMPROBAR** Por que si pude. No había puesto `--auth`



Como le cambio el Rol al usuario:


```sh
> use gimnasio
switched to db gimnasio
> db.runCommand({
... updateUser: "juan73",
... roles: [{ role: "readWrite", db: "gimnasio"}]
... })
{ "ok" : 1 }
>       
```
A partir de aquí ya puedo Leer y Escribir con "juan73".




```sh
> db.runCommand({ usersInfo: "juan73", showPrivileges: true})
{
        "users" : [
                {
                        "_id" : "gimnasio.juan73",
                        "userId" : UUID("3f3b060a-875a-4a67-ae4c-033f83670b41"),
                        "user" : "juan73",
                        "db" : "gimnasio",
                        "mechanisms" : [
                                "SCRAM-SHA-1",
                                "SCRAM-SHA-256"
                        ],
                        "roles" : [
                                {
                                        "role" : "readWrite",
                                        "db" : "gimnasio"
                                }
                        ],
                        "inheritedRoles" : [
                                {
                                        "role" : "readWrite",
                                        "db" : "gimnasio"
                                }
                        ],
                        "inheritedPrivileges" : [
                                {
                                        "resource" : {
                                                "db" : "gimnasio",
                                                "collection" : ""
                                        },
                                        "actions" : [
                                                "changeStream",
                                                "collStats",
                                                "convertToCapped",
                                                "createCollection",
                                                "createIndex",
                                                "dbHash",
                                                "dbStats",
                                                "dropCollection",
                                                "dropIndex",
                                                "emptycapped",
                                                "find",
                                                "insert",
                                                "killCursors",
                                                "listCollections",
                                                "listIndexes",
                                                "planCacheRead",
                                                "remove",
                                                "renameCollectionSameDB",
                                                "update"
                                        ]
                                },
                                {
                                        "resource" : {
                                                "db" : "gimnasio",
                                                "collection" : "system.js"
                                        },
                                        "actions" : [
                                                "changeStream",
                                                "collStats",
                                                "convertToCapped",
                                                "createCollection",
                                                "createIndex",
                                                "dbHash",
                                                "dbStats",
                                                "dropCollection",
                                                "dropIndex",
                                                "emptycapped",
                                                "find",
                                                "insert",
                                                "killCursors",
                                                "listCollections",
                                                "listIndexes",
                                                "planCacheRead",
                                                "remove",
                                                "renameCollectionSameDB",
                                                "update"
                                        ]
                                }
                        ],
                        "inheritedAuthenticationRestrictions" : [ ]
                }
        ],
        "ok" : 1
}
>          
```

Creo usuario que me permite administrar los usuarios
```sh
> use maraton
switched to db maraton
> db.createUser({
... user: "laura73",
... pwd: "laura1234",
... roles: ["userAdmin"]
... })
Successfully added user: { "user" : "laura73", "roles" : [ "userAdmin" ] }

> db.createUser({ user: "laura73", pwd: "laura1234", roles: ["userAdmin"] })
Successfully added user: { "user" : "laura73", "roles" : [ "userAdmin" ] }
>   
```

En otra consola entro con "laura73":

```sh
C:\Users\manana>mongo --authenticationDatabase "maraton" -u "laura73"
```

Puedo crear un usuario desde `laura73`:

```sh
> use maraton
switched to db maraton
> db.createUser({
... user: "manuel73",
... pwd: "manuel1234",
... roles: ["readWrite"]
... })
Successfully added user: { "user" : "manuel73", "roles" : [ "readWrite" ] }
>
```

Roles Sobre BD

Añado roles a usuario existente:

```sh
> db.runCommand({ grantRolesToUser: "juan73", roles: [{role: "readAnyDatabase", db: "admin"}] })
{ "ok" : 1 }
>        
```

```sh
>  db.runCommand({ usersInfo: "juan73", showPrivileges: true})
{
        "users" : [
                {
                        "_id" : "gimnasio.juan73",
                        "userId" : UUID("3f3b060a-875a-4a67-ae4c-033f83670b41"),
                        "user" : "juan73",
                        "db" : "gimnasio",
                        "mechanisms" : [
                                "SCRAM-SHA-1",
                                "SCRAM-SHA-256"
                        ],
                        "roles" : [
                                {
                                        "role" : "readAnyDatabase",
                                        "db" : "admin"
                                },
                                {
                                        "role" : "readWrite",
                                        "db" : "gimnasio"
                                }
                        ],
                        "inheritedRoles" : [
                                {
                                        "role" : "readAnyDatabase",
                                        "db" : "admin"
                                },
                                {
                                        "role" : "readWrite",
                                        "db" : "gimnasio"
                                }
                        ],
                        "inheritedPrivileges" : [
                                {
                                        "resource" : {
                                                "db" : "",
                                                "collection" : ""
                                        },
                                        "actions" : [
                                                "changeStream",
                                                "collStats",
                                                "dbHash",
                                                "dbStats",
                                                "find",
                                                "killCursors",
                                                "listCollections",
                                                "listIndexes",
                                                "planCacheRead"
                                        ]
                                },
                                {
                                        "resource" : {
                                                "cluster" : true
                                        },
                                        "actions" : [
                                                "listDatabases"
                                        ]
                                },
                                {
                                        "resource" : {
                                                "db" : "",
                                                "collection" : "system.js"
                                        },
                                        "actions" : [
                                                "changeStream",
                                                "collStats",
                                                "dbHash",
                                                "dbStats",
                                                "find",
                                                "killCursors",
                                                "listCollections",
                                                "listIndexes",
                                                "planCacheRead"
                                        ]
                                },
                                {
                                        "resource" : {
                                                "db" : "gimnasio",
                                                "collection" : ""
                                        },
                                        "actions" : [
                                                "changeStream",
                                                "collStats",
                                                "convertToCapped",
                                                "createCollection",
                                                "createIndex",
                                                "dbHash",
                                                "dbStats",
                                                "dropCollection",
                                                "dropIndex",
                                                "emptycapped",
                                                "find",
                                                "insert",
                                                "killCursors",
                                                "listCollections",
                                                "listIndexes",
                                                "planCacheRead",
                                                "remove",
                                                "renameCollectionSameDB",
                                                "update"
                                        ]
                                },
                                {
                                        "resource" : {
                                                "db" : "gimnasio",
                                                "collection" : "system.js"
                                        },
                                        "actions" : [
                                                "changeStream",
                                                "collStats",
                                                "convertToCapped",
                                                "createCollection",
                                                "createIndex",
                                                "dbHash",
                                                "dbStats",
                                                "dropCollection",
                                                "dropIndex",
                                                "emptycapped",
                                                "find",
                                                "insert",
                                                "killCursors",
                                                "listCollections",
                                                "listIndexes",
                                                "planCacheRead",
                                                "remove",
                                                "renameCollectionSameDB",
                                                "update"
                                        ]
                                }
                        ],
                        "inheritedAuthenticationRestrictions" : [ ]
                }
        ],
        "ok" : 1
}
>              
```

Me logeo para comprobarlo:

```sh
C:\Users\manana>mongo --authenticationDatabase "gimnasio" -u "juan73"

```

En Gimnasio me dejara leer y escribir en las otras solo consultarlas.


```sh
> use gimnasio
switched to db gimnasio
> db.fii.insert({ a: 1})
WriteResult({ "nInserted" : 1 })
>
```


en shop no me deja escribir
```sh
> use shop
switched to db shop
> db.shop.insert({ a:1 })
WriteCommandError({
        "ok" : 0,
        "errmsg" : "not authorized on shop to execute command { insert: \"shop\", ordered: true, lsid: { id: UUID(\"712e14cd-e07c-472b-af3c-8676b2793f7f\") }, $db: \"shop\" }",
        "code" : 13,
        "codeName" : "Unauthorized"
})
>
```

Roles User-defined

Creación de un rol particular y especifico.

```sh
> use gimnasio
switched to db gimnasio
> shoe collections
2020-02-17T12:16:06.333+0100 E  QUERY    [js] uncaught exception: SyntaxError: unexpected token: identifier :
@(shell):1:5
> show collections
asociaciones
clientes
fii
foo2
fuu
games
interesados
monitores
> db.createRole({
... role: "soloEscrituraInteresados",
... privileges: [
... {resources: {db: "gimnasio", collection: "interesados"}, actions: ["find", "insert","update","remove"]}
... ],
... roles: [{role: "read", db: "gimnasio"}]
... })  
2020-02-17T12:27:36.261+0100 E  QUERY    [js] uncaught exception: Error: missing resource field :
_getErrorWithCode@src/mongo/shell/utils.js:25:13
DB.prototype.createRole@src/mongo/shell/db.js:1654:15
@(shell):1:1
```
**ME FALLO**

Creo usuario con rol personalizado

```sh
> db.createUser({
... user: "maria73",
... pwd: "maria1234",
... roles: ["soloEscrituraInteresados"]
... })
2020-02-17T12:29:36.774+0100 E  QUERY    [js] uncaught exception: Error: couldn't add user: No role named soloEscrituraInteresados@gimnasio :
_getErrorWithCode@src/mongo/shell/utils.js:25:13
DB.prototype.createUser@src/mongo/shell/db.js:1370:11
@(shell):1:1
>        
```
**Me falla**

En otra shell entro como "maria73":

y compruebo que 



```sh
mongo --authenticationDatabase "gimnasio" -u "maria73"
>use gimnasio

> db.interesados.insert({a:1})
WriteResult({ "nInserted" : 1 })
>
> db.foo.insert({a:1})
FALLA
```
En interesados puedo insertar, y en otras colecciones solo puedo leerlas.
**Comprobar cuando pueda crear el rol personalizado que me fallo**


Como Borrar un Usuario:
estando en SuperAdmin

```sh
> use admin
switched to db admin
> db.system.users.find().pretty()
{
        "_id" : "maraton.adolfo",
        "userId" : UUID("182196d1-9937-4064-9a05-34169e6cb811"),
        "user" : "adolfo",
        "db" : "maraton",
        "credentials" : {
                "SCRAM-SHA-1" : {
                        "iterationCount" : 10000,
                        "salt" : "kj6Du7fgSrNNzRB0akw8SQ==",
                        "storedKey" : "d3PujkUfp3Gqsycqr74TKsBUzK8=",
                        "serverKey" : "rNYmvZipC69mvD7CQz1rWnYmxOk="
                },
                "SCRAM-SHA-256" : {
                        "iterationCount" : 15000,
                        "salt" : "GHkL+no0vLlfmqQL7p+iqk3JhWlZJSQk7fPQoA==",
                        "storedKey" : "f6BCcaBRfpm/NsD/8rybFn/u6BYzcQNoMntNRYSXINI=",
                        "serverKey" : "cJ8TeVXuSq/LrAr/hTOzxf6xBDOQnw0DMLL87O0qRsI="
                }
        },
        "customData" : {
                "dni" : "X883838S"
        },
        "roles" : [
                {
                        "role" : "readWrite",
                        "db" : "maraton"
                }
        ]
}
{
        "_id" : "admin.superAdmin",
        "userId" : UUID("227cd5bc-7d73-4983-8965-3b80accd14b2"),
        "user" : "superAdmin",
        "db" : "admin",
        "credentials" : {
                "SCRAM-SHA-1" : {
                        "iterationCount" : 10000,
                        "salt" : "pwaGtp7fu+r/3rKwPwJvJQ==",
                        "storedKey" : "YGGYI7gPhlSUfWsySjwsTFs6g+Q=",
                        "serverKey" : "rVXzKhE+H/YxEXDCnUu1Ygw/ZdI="
                },
                "SCRAM-SHA-256" : {
                        "iterationCount" : 15000,
                        "salt" : "mG7SSBJiFxrBeRx35qaxEUpSM/1q2SOdS8iSsA==",
                        "storedKey" : "/1sYk7qqhJsiqMAitM588DXvR9OL0YUUE1qA5BWXEzE=",
                        "serverKey" : "c1xSAg2IAiINeIAY6+WRg1WAIn1aY9F7J2jFKepYdaQ="
                }
        },
        "roles" : [
                {
                        "role" : "userAdminAnyDatabase",
                        "db" : "admin"
                },
                {
                        "role" : "readWriteAnyDatabase",
                        "db" : "admin"
                }
        ]
}
{
        "_id" : "gimnasio.juan73",
        "userId" : UUID("3f3b060a-875a-4a67-ae4c-033f83670b41"),
        "user" : "juan73",
        "db" : "gimnasio",
        "credentials" : {
                "SCRAM-SHA-1" : {
                        "iterationCount" : 10000,
                        "salt" : "AT1iIkRRsac56h1THkdShA==",
                        "storedKey" : "LcwiObBvz1VDotCnKCjldR8KMnc=",
                        "serverKey" : "YWmdOwLEdIQiNkchuL82CCKDjkY="
                },
                "SCRAM-SHA-256" : {
                        "iterationCount" : 15000,
                        "salt" : "H7bqbqWmVf9CHXj587ILRk8TxcwEEPUHdhqmDw==",
                        "storedKey" : "UxVy5UgAi6xIFQ/+1M8Ftt7IRBW+qI6dt5hvM+PrD/s=",
                        "serverKey" : "6nvLVqwDOUzoNliIWtz+fhxd+kt34LrduQKP0NBk564="
                }
        },
        "roles" : [
                {
                        "role" : "readAnyDatabase",
                        "db" : "admin"
                },
                {
                        "role" : "readWrite",
                        "db" : "gimnasio"
                }
        ]
}
{
        "_id" : "maraton.laura73",
        "userId" : UUID("94bb8feb-7b22-40bb-b7f4-1481a0e12b78"),
        "user" : "laura73",
        "db" : "maraton",
        "credentials" : {
                "SCRAM-SHA-1" : {
                        "iterationCount" : 10000,
                        "salt" : "syonuVSSkvZSWKt3i7pWAA==",
                        "storedKey" : "2IPRmGfTPJJ6ESEwl1aK/TVcpXs=",
                        "serverKey" : "Oy6/7gNZaMtdkgf4tlNsIYhCe54="
                },
                "SCRAM-SHA-256" : {
                        "iterationCount" : 15000,
                        "salt" : "lEjTG0xxULYFCZqF6EvSheW3OSxEQ83GQD+k+A==",
                        "storedKey" : "0+63cIfvrPlLmPtN7T2d/FNiboSKkY2xONcoPXrhg5Q=",
                        "serverKey" : "5FyGojGM6rDflsdtyFGBzAQPQdZj+JgkHyzzdkpcpC4="
                }
        },
        "roles" : [
                {
                        "role" : "userAdmin",
                        "db" : "maraton"
                }
        ]
}
{
        "_id" : "maraton.manuel73",
        "userId" : UUID("28e35ee5-e7f3-4a12-ae9f-b16d02128506"),
        "user" : "manuel73",
        "db" : "maraton",
        "credentials" : {
                "SCRAM-SHA-1" : {
                        "iterationCount" : 10000,
                        "salt" : "R4utsddqdJIP2FCwQV9xmA==",
                        "storedKey" : "TAiKGhw0q9XsNM80StPQomVFdGU=",
                        "serverKey" : "DYA5sPRjoCWIT2LzaCJ/xswVFAs="
                },
                "SCRAM-SHA-256" : {
                        "iterationCount" : 15000,
                        "salt" : "aHVUt48yuT6JtYnMLn0OfuFQiYEENRJ8jA/Wdw==",
                        "storedKey" : "s077cv2rb1K2zdzWetHdsUXxsuzR6MqBCcPaWaIH4Qo=",
                        "serverKey" : "dMkJwqoaloeqmXl/4kIwbEFs2rr+bxRCUeY3mwaJHG8="
                }
        },
        "roles" : [
                {
                        "role" : "readWrite",
                        "db" : "maraton"
                }
        ]
}
{
        "_id" : "biblioteca.manuel73",
        "userId" : UUID("b8e127ff-168c-4ac7-bc62-66f87140d1f6"),
        "user" : "manuel73",
        "db" : "biblioteca",
        "credentials" : {
                "SCRAM-SHA-1" : {
                        "iterationCount" : 10000,
                        "salt" : "Lnm9QX6jcKeocUDE5x1ZkA==",
                        "storedKey" : "8TG8ocyW3Yd9NWufVQ9VvCufy6Q=",
                        "serverKey" : "4RH/crkxRcIC2bXrvBfBIZYpr70="
                },
                "SCRAM-SHA-256" : {
                        "iterationCount" : 15000,
                        "salt" : "7rUqox3bqyWBYJzKwsXKO1bega8PAzQrZ1HTlw==",
                        "storedKey" : "tcFrRelEIsUWvCxYjOJQNmQdxiYDPAQRmIiYvOR1KwM=",
                        "serverKey" : "KV4M7Soj4pIf9g3OdEYPojZuKn/2ZOZGwu2WHr4fQKw="
                }
        },
        "roles" : [
                {
                        "role" : "readWrite",
                        "db" : "biblioteca"
                }
        ]
}
{
        "_id" : "test.laura73",
        "userId" : UUID("3674d48b-a618-44b8-89b3-f92661104fe0"),
        "user" : "laura73",
        "db" : "test",
        "credentials" : {
                "SCRAM-SHA-1" : {
                        "iterationCount" : 10000,
                        "salt" : "0UgIeK/vkKtX9zL6q4Bgbw==",
                        "storedKey" : "xSoeRm1vfpmSq5zO3BwGFm4UB1w=",
                        "serverKey" : "iwZZhUtnLM2ilOHbR75mQQi+d94="
                },
                "SCRAM-SHA-256" : {
                        "iterationCount" : 15000,
                        "salt" : "YQSPSWHA75i3XaIveS4SJrD+lAlFSG6H5FIW9Q==",
                        "storedKey" : "LWLj0lrJY+F89S6J2PqQmgvT6AA8cCUJOcTN0FiNkgU=",
                        "serverKey" : "Ai2Iunj9/SChtIAi0exKGraxRK034rQOxR7Sai99tBE="
                }
        },
        "roles" : [
                {
                        "role" : "userAdmin",
                        "db" : "test"
                }
        ]
}
> use maraton
switched to db maraton
> db.dropUser("pedro")
false
>
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
```sh
```


```sh
```
