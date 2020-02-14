# 10. Seguridad

[Security](https://docs.mongodb.com/manual/security/index.html)

## Autenticación

[Authentication](https://docs.mongodb.com/manual/core/authentication/)

Mecanismos de Autenticación (Community)

* SCRAM (Defecto)
* x.509
Adicionales en la versión Enterprise)
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



