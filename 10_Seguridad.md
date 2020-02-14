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


