# Esquemas o Modelo

[Data Modeling Introduction](https://docs.mongodb.com/manual/core/data-modeling-introduction/)

### Flexibilidad de la estructura de datos en MongoDB.

* Los documentos de una misma colección no tienen por que tener la misma estructura ni tipos de datos.
* No es necesario especificar al crear la colección ningún tipo de estructura.
* Es posible modificar la estructura de los documentos a posteriori.

### Tipos de relaciones en MongoDB

1. Documento embebido o modelo **denormalizado** [Embedded Data](https://docs.mongodb.com/manual/core/data-modeling-introduction/#embedded-data)

   Tipo de relación por defecto de MongoDB y al que se debe recurrir siempre que se pueda.
   
   Colección Productos
   ```
   {
     producto: "Nike TF55",
     marca: "Nike",
     distribuidores:[
       {nombre: "ServiZapas", contacto: "xxx", ....},
       {nombre: "Distr. Pérez", contacto: "xxx", ....}
     ],
     precio: ...
   }
   ```

2. Documentos referenciados de otra colección o modelo normalizado [References](https://docs.mongodb.com/manual/core/data-modeling-introduction/#references)

   Colección Productos
   ```
   {
     _id: "01",
     producto: "Nike TF55",
     marca: "Nike",
     distribuidores:["d03", "d06"],
     precio: ...
   }
   ```
  
   Colección Distribuidores
   ```
   {
    _id: "d03",
    nombre: "ServiZapas",
    contacto: ...
   }
  
   {
    _id: "d06",
   nombre: "Distr. Pérez",
   contacto: ...
   }
   ```
   
   ### MODELO ONE-TO-ONE
   
   [Model One-to-One Relationships with Embedded Documents](https://docs.mongodb.com/manual/tutorial/model-embedded-one-to-one-relationships-between-documents/)
   Em MongoDB usaremos el modelo denormalizado o embebido.
   
   ```
   {
     _id: "2343",
     nombre: "Juan",
     ...,
     direccion: {
       calle: "Gran Vía, 30",
       cp: "28001",
       localidad: "Madrid"
     }
   }
   ```

   No tengo dos colecciones (clientes, direcciones) en una sola tengo ambos datos.
   
   ### MODELO ONE-TO-MANY
   
   [Model One-to-Many Relationships with Embedded Documents¶](https://docs.mongodb.com/manual/tutorial/model-embedded-one-to-many-relationships-between-documents/)
   Modelo One-to-Few
   
   En MongoDB intentemos utilizar el modelo denormalizado o embebido.
   
   * El lado one es el que normalmente recibe más consultas
   * Los documentos del lado many normalmente no tendrán escrituras frecuentes.
   
    Colección Productos
    ```
    {
      _id: "01",
      producto: "Nike TF55",
      marca: "Nike",
      imagenes:["", ""],
      precio: ...
    }
    ```
   
   
