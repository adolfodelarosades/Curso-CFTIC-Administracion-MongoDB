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


