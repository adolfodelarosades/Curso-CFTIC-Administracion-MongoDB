# 6 Esquemas o Modelo

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
[Model One-to-Many Relationships with Document References](https://docs.mongodb.com/manual/tutorial/model-referenced-one-to-many-relationships-between-documents/#model-one-to-many-relationships-with-document-references)
   
**Modelo One-to-Few**
   
En MongoDB intentemos utilizar el modelo denormalizado o embebido.
   
* El lado one es el que normalmente recibe más consultas
* Los documentos del lado many normalmente no tendrán escrituras frecuentes.
   
   Colección Productos (One)
   ```
    {
      _id: "01",
      producto: "Nike TF55",
      marca: "Nike",
      imagenes:[
        {url: "https://...", textoAlt: "Zapatillas ...."}
        {url: "https://...", textoAlt: "Zapatillas ...."}
        ...
      ],
      precio: ...
    }
   ```
   
   Las imagenes(Few) en teoría no cambian mucho, se embeben en el documento principal.
   
**Modelo One-to-Many**

En general, modelo denormalizado o embebido. Ver circunstancias concretas.

* Cada documento individual no puede superar 16MG,

**Modelo One-to-squillions** (muchísimos "tropecientos")

En general, modelo normalizado o referencia a otra colección.

* El lado many recibira muchas consultas.
* El lado many podría crecer hasta superar el límite de 16 MG del lado one.

 Colección Productos (One)
   ```
    {
      _id: "01",
      producto: "Nike TF55",
      marca: "Nike",
      opiniones:["265462", "254567" ],
      precio: ...
    }
   ```

Colección Opiniones (Many)

```
{
  _id: "265462",
  texto: "Buen producto",
  estrellas: 4,
  usuario: "..."
}
```

**Modelo Many-to-Many**

Con modelo denormalizado o documento embebido

* Mayor número de consultas en el lado del mayor número de registros.
* Redundancia de datos.

 Colección Productos (One)
   ```
    {
      _id: "01",
      producto: "Nike TF55",
      marca: "Nike",
      tiendas:[
        {nombre: "Alcorcón Store", direccion: "..."},
        {nombre: "Las Rozas Store", direccion: "..."},
        ...
      ],
      precio: ...
    },
    {
      _id: "09",
      producto: "Adidas AR57",
      marca: "Adidas",
      tiendas:[
        {nombre: "Gran Vía Store", direccion: "..."},
        {nombre: "Las Rosas Store", direccion: "..."},
        ...
      ],
      precio: ...
    }
   ```

La redundacia esta en las tiendas sin las inserto aquí y estas cambian puede haber problemas por que se estan poniendo para cada producto.

Con modelo normalizado o referencia a documentos.

* Consultas sobre los dos lados
* Alta Actulizada 

 Colección Productos (One)
   ```
    {
      _id: "01",
      producto: "Nike TF55",
      marca: "Nike",
      tiendas:["ALC", "LRZ"],
      precio: ...
    }
    
    {
      _id: "09",
      producto: "Adidas AR57",
      marca: "Adidas",
      tiendas:["ALC", "LRZ"],
      precio: ...
    }
   ```
   
   Colección Tiendas
   ```sh
   {
     _id: "GV",
     nombre: "Gran Vía Store",
     dir: ....
   }
    {
     _id: "LRZ",
     nombre: "Las Rosas Store",
     dir: ....
   }
    {
     _id: "ALC",
     nombre: "Alcorcón Store",
     dir: ....
   }
   ```




