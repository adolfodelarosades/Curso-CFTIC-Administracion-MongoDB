# Esquemas o Modelo

[Data Modeling Introduction](https://docs.mongodb.com/manual/core/data-modeling-introduction/)

### Flexibilidad de la estructura de datos en MongoDB.

* Los documentos de una misma colección no tienen por que tener la misma estructura ni tipos de datos.
* No es necesario especificar al crear la colección ningún tipo de estructura.
* Es posible modificar la estructura de los documentos a posteriori.

### Tipos de relaciones en MongoDB

1. Documento embebido o modelo **denormalizado**
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

2. Documentos referenciados de otra colección o modelo normalizado

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
   
