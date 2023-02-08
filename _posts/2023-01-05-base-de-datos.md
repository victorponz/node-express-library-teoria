---
typora-copy-images-to: ../assets/img/
typora-root-url: ../../
layout: post
title: 'Tutorial Express - parte 3: Base de datos - con Mongoose'
categories: parte1
conToc: true
permalink: base-de-datos
---

Este artículo presenta brevemente las bases de datos y cómo usarlas con las aplicaciones Node/Express. Luego pasa a mostrar cómo podemos usar Mongoose para proporcionar acceso a la base de datos para el sitio web de LocalLibrary. Explica cómo se declaran el esquema y los modelos de objetos, los principales tipos de campos y la validación básica. También muestra brevemente algunas de las formas principales en las que puede acceder a los datos del modelo.

## Descripción general

El personal de la biblioteca usará el sitio web de la biblioteca local para almacenar información sobre libros y prestatarios, mientras que los miembros de la biblioteca lo usarán para navegar y buscar libros, averiguar si hay copias disponibles y luego reservarlas o tomarlas prestadas. Para almacenar y recuperar información de manera eficiente, la almacenaremos en una base de datos.

Las aplicaciones Express pueden usar muchas bases de datos diferentes, y hay varios enfoques que puede usar para realizar operaciones de creación, lectura, actualización y eliminación (**CRUD**). Este tutorial proporciona una breve descripción general de algunas de las opciones disponibles y luego continúa mostrando en detalle los mecanismos particulares seleccionados.

## ¿Qué bases de datos se pueden usar?

Las aplicaciones Express pueden usar cualquier base de datos admitida por Node (Express en sí no define ningún comportamiento/requisito adicional específico para la administración de la base de datos). Hay muchas opciones populares, incluidas PostgreSQL, MySQL, Redis, SQLite y MongoDB.

Al elegir una base de datos, debe considerar aspectos como el tiempo de productividad/curva de aprendizaje, el rendimiento, la facilidad de replicación/copia de seguridad, el costo, el apoyo de la comunidad, etc. Si bien no existe una única base de datos "mejor", casi todas las soluciones populares debería ser más que aceptable para un sitio de tamaño pequeño a mediano como nuestra biblioteca local.

Para obtener más información sobre las opciones, consulta [Integración de la base de datos](https://expressjs.com/en/guide/database-integration.html)

### ¿Cuál es la mejor manera de interactuar con una base de datos?

Hay dos enfoques comunes para interactuar con una base de datos:

1. Usar el lenguaje de consulta nativo de las bases de datos (por ejemplo, SQL)
2. Usando un modelo de datos de objetos ("ODM") o un modelo relacional de objetos ("ORM"). Un ODM/ORM representa los datos del sitio web como objetos JavaScript, que luego se asignan a la base de datos subyacente. Algunos ORM están vinculados a una base de datos específica, mientras que otros proporcionan un backend independiente de la base de datos.

Se puede obtener el mejor rendimiento utilizando SQL o cualquier lenguaje de consulta que admita la base de datos. Los ODM suelen ser más lentos porque usan código de traducción para mapear entre objetos y el formato de la base de datos, lo que puede no usar las consultas de base de datos más eficientes (esto es particularmente cierto si el ODM admite diferentes backends de bases de datos y debe comprometerse más en términos de qué base de datos características son compatibles).

El beneficio de usar un ORM es que los programadores pueden seguir pensando en términos de objetos de JavaScript en lugar de la semántica de la base de datos; esto es particularmente cierto si necesita trabajar con diferentes bases de datos (ya sea en el mismo sitio web o en sitios diferentes). También proporcionan un lugar obvio para realizar la validación de datos.

> -info- ¡El uso de ODM/ORM a menudo resulta en menores costos de desarrollo y mantenimiento! A menos que esté muy familiarizado con el lenguaje de consulta nativo o el rendimiento sea primordial, debería considerar seriamente el uso de un ODM.

### ¿Qué ORM/ODM debo usar?

Hay muchas soluciones ODM/ORM disponibles en el sitio del administrador de paquetes `npm` (consulta las etiquetas [odm](https://www.npmjs.com/search?q=keywords:odm) y [orm](https://www.npmjs.com/search?q=keywords:orm) para ver un subconjunto).

Algunas soluciones que eran populares en el momento de escribir este artículo son:

* [Mongoose](https://www.npmjs.com/package/mongoose): Mongoose es una herramienta de modelado de objetos [MongoDB](https://www.mongodb.com/) diseñada para trabajar en un entorno asíncrono.
* [Waterline](https://www.npmjs.com/package/waterline): un ORM extraído del marco web de [Sails](https://sailsjs.com/) basado en Express. Proporciona una API uniforme para acceder a numerosas bases de datos diferentes, incluidas Redis, MySQL, LDAP, MongoDB y Postgres.
* [Bookshelf](https://www.npmjs.com/package/bookshelf): presenta interfaces de devolución de llamadas tradicionales y basadas en promesas, que brindan soporte de transacciones, carga de relaciones ansiosas/anidadas, asociaciones polimórficas y soporte para relaciones uno a uno, uno a muchos y muchos a muchos. Funciona con PostgreSQL, MySQL y SQLite3.
* [Objection](https://www.npmjs.com/package/objection): Facilita al máximo el uso de toda la potencia de SQL y el motor de base de datos subyacente (compatible con SQLite3, Postgres y MySQL).
* [Sequelize](https://www.npmjs.com/package/sequelize) es un ORM basado en promesas para Node.js e io.js. Es compatible con los dialectos PostgreSQL, MySQL, MariaDB, SQLite y MSSQL y presenta un sólido soporte de transacciones, relaciones, replicación de lectura y más.
* [Node ORM2](https://node-orm.readthedocs.io/en/latest/) es un administrador de relaciones de objetos para NodeJS. Es compatible con MySQL, SQLite y Progress, lo que ayuda a trabajar con la base de datos utilizando un enfoque orientado a objetos.
* [GraphQL](https://graphql.org/): principalmente un lenguaje de consulta para API tranquilas, GraphQL es muy popular y tiene funciones disponibles para leer datos de bases de datos.

Como regla general, debes considerar tanto las funciones proporcionadas como la "actividad de la comunidad" (descargas, contribuciones, informes de errores, calidad de la documentación, etc.) al seleccionar una solución. Al momento de escribir, Mongoose es, con mucho, el ODM más popular y es una opción razonable si estás utilizando MongoDB para su base de datos.

### Usar Mongoose y MongoDB para la biblioteca

Para el ejemplo de la biblioteca local (y el resto de este tema), vamos a utilizar Mongoose ODM para acceder a los datos de nuestra biblioteca. Mongoose actúa como interfaz para MongoDB, una base de datos NoSQL de código abierto que utiliza un modelo de datos orientado a documentos. Una "colección" de "documentos" en una base de datos MongoDB es análoga a una "tabla" de "filas" en una base de datos relacional.

Esta combinación de ODM y base de datos es extremadamente popular en la comunidad de Node, en parte porque el sistema de consulta y almacenamiento de documentos se parece mucho a JSON y, por lo tanto, es familiar para los desarrolladores de JavaScript.

> -info-No necesitas conocer MongoDB para usar Mongoose, aunque partes de la documentación de Mongoose son más fáciles de usar y comprender si ya estás familiarizado con MongoDB.

El resto de este tutorial muestra cómo definir y acceder al esquema y los modelos de Mongoose para el ejemplo del sitio web LocalLibrary.

## Diseñar los modelos

Antes de saltar y comenzar a codificar los modelos, vale la pena tomarse unos minutos para pensar qué datos necesitamos almacenar y las relaciones entre los diferentes objetos.

Sabemos que necesitamos almacenar información sobre libros (título, resumen, autor, género, ISBN) y que podemos tener varias copias disponibles (con identificaciones únicas a nivel mundial, estados de disponibilidad, etc.). Es posible que necesitemos almacenar más información sobre el autor que solo su nombre, y puede haber varios autores con el mismo nombre o nombres similares. Queremos poder clasificar la información según el título del libro, el autor, el género y la categoría.

Al diseñar sus modelos, tiene sentido tener modelos separados para cada "objeto" (un grupo de información relacionada). En este caso, algunos candidatos obvios para estos modelos son los libros, las instancias de libros y los autores.

También es posible que desees utilizar modelos para representar las opciones de la lista de selección (por ejemplo, como una lista desplegable de opciones), en lugar de codificar las opciones en el propio sitio web; esto se recomienda cuando no se conocen todas las opciones por adelantado. o puede cambiar. Un buen ejemplo es un género (por ejemplo, fantasía, ciencia ficción, etc.).

Una vez que hemos decidido nuestros modelos y campos, debemos pensar en las relaciones entre ellos.

Con eso en mente, el siguiente diagrama de asociación UML muestra los modelos que definiremos en este caso (como cuadros). Como se discutió anteriormente, hemos creado modelos para el libro (los detalles genéricos del libro), la instancia del libro (estado de las copias físicas específicas del libro disponibles en el sistema) y el autor. También hemos decidido tener un modelo para el género para que los valores se puedan crear dinámicamente. Hemos decidido no tener un modelo para `BookInstance:status`; codificaremos los valores aceptables porque no esperamos que cambien. Dentro de cada uno de las cajas, puedes ver el nombre del modelo, los nombres y tipos de campo, y también los métodos y sus tipos de devolución.

El diagrama también muestra las relaciones entre los modelos, incluidas sus multiplicidades. Las multiplicidades son los números en el diagrama que muestra los números (máximo y mínimo) de cada modelo que puede estar presente en la relación. Por ejemplo, la línea de conexión entre las cajas muestra que el `Book` y `Genre` están relacionados. Los números cerca del modelo `Boolk` muestran que un `Genre` debe tener cero o más Libros (tantos como desee), mientras que los números en el otro extremo de la línea al lado del `Genre` muestran que un libro puede tener cero o más `Genre` asociados. .

![](/node-express-library-teoria/assets/img/library_website_-_mongoose_express.png)

## Primeros pasos con Mongoose

Esta sección proporciona una descripción general de cómo conectar Mongoose a una base de datos MongoDB, cómo definir un esquema y un modelo, y cómo realizar consultas básicas.

> -info- Este manual está fuertemente influenciado por el [inicio rápido de Mongoose](https://www.npmjs.com/package/mongoose) en npm y la [documentación oficial](https://mongoosejs.com/docs/guide.html).

## Instalación de Mongoose y MongoDB

Mongoose se instala en el proyecto (`package.json`) como cualquier otra dependencia, usando `npm`. Para instalarlo, use el siguiente comando dentro de la carpeta de su proyecto:

```
npm instalar mongoose
```

La instalación de Mongoose agrega todas sus dependencias, incluido el controlador de la base de datos MongoDB, pero no instala MongoDB en sí. Si deseas instalar un servidor MongoDB, puede descargar [instaladores](https://www.mongodb.com/try/download/community) desde aquí para varios sistemas operativos e instalarlo localmente. También puedes utilizar instancias de MongoDB basadas en la nube.

> -info-Para este tutorial, usaremos la base de datos basada en la nube de MongoDB Atlas como un nivel gratuito de servicio para proporcionar la base de datos. Esto es adecuado para el desarrollo y tiene sentido para el tutorial porque hace que la "instalación" del sistema operativo sea independiente (la base de datos como servicio también es un enfoque que puede usar para su base de datos de producción).

## Conexión a MongoDB

Mongoose requiere una conexión a una base de datos MongoDB. Puedes usar `require()` y conectarte a una base de datos alojada localmente con `mongoose.connect()` como se muestra a continuación (para el tutorial, en su lugar, nos conectaremos a una base de datos alojada en Internet).

```javascript
// Import the mongoose module
const mongoose = require("mongoose");

// Set `strictQuery: false` to globally opt into filtering by properties that aren't in the schema
// Included because it removes preparatory warnings for Mongoose 7.
// See: https://mongoosejs.com/docs/migrating_to_6.html#strictquery-is-removed-and-replaced-by-strict
mongoose.set('strictQuery', false);

// Define the database URL to connect to.
const mongoDB = "mongodb://127.0.0.1/my_database";

// Wait for database to connect, logging an error if there is a problem
main().catch(err => console.log(err));
async function main() {
  await mongoose.connect(mongoDB);
}
```

Puedes obtener el objeto `Connection` predeterminado con `mongoose.connection`. Si necesitas crear conexiones adicionales, puedes usar `mongoose.createConnection()`. Toma la misma forma de URI de base de datos (con host, base de datos, puerto, opciones, etc.) que `connect()` y devuelve un objeto `Connection`). Ten en cuenta que `createConnection()` regresa inmediatamente; si necesitas esperar a que se establezca la conexión, puede llamarla con `asPromise()` para devolver una promesa (`mongoose.createConnection(mongoDB).asPromise()`).

## Definir y crear modelo

Los modelos se definen mediante la interfaz `Schema`. El `Schema` te permite definir los campos almacenados en cada documento junto con sus requisitos de validación y valores predeterminados. Además, puedes definir métodos auxiliares estáticos y de instancia para facilitar el trabajo con sus tipos de datos, y también propiedades virtuales que puede usar como cualquier otro campo, pero que en realidad no están almacenadas en la base de datos (hablaremos un poco más abajo).

Luego, los esquemas se "compilan" en modelos utilizando el método `mongoose.model()`. Una vez que tengas un modelo, puedes usarlo para buscar, crear, actualizar y eliminar objetos del tipo dado.

> -info-Cada modelo se asigna a una colección de documentos en la base de datos MongoDB. Los documentos contendrán los campos/tipos de esquema definidos en el esquema modelo.

### Definir schemas

El siguiente fragmento de código muestra cómo se puede definir un esquema simple. Primero usas `requiere(mongoose)` , luego usas el constructor de esquema para crear una nueva instancia de esquema, definiendo los diversos campos dentro de él en el parámetro de objeto del constructor.

```javascript
// Require Mongoose
const mongoose = require("mongoose");

// Define a schema
const Schema = mongoose.Schema;

const SomeModelSchema = new Schema({
  a_string: String,
  a_date: Date,
});

```

En el caso anterior, solo tenemos dos campos, una cadena y una fecha. En las siguientes secciones, mostraremos algunos de los otros tipos de campos, validación y otros métodos.

### Crear un modelo

Los modelos se crean a partir de esquemas utilizando el método `mongoose.model()`:

```javascript
// Define schema
const Schema = mongoose.Schema;

const SomeModelSchema = new Schema({
  a_string: String,
  a_date: Date,
});

// Compile model from schema
const SomeModel = mongoose.model("SomeModel", SomeModelSchema);
```

El primer argumento es el nombre singular de la colección que se creará para su modelo (Mongoose creará la colección de la base de datos para el modelo `SomeModel` anterior), y el segundo argumento es el esquema que desea usar para crear el modelo.

> -info-Una vez que hayas definido tus clases de modelo, puedes usarlas para crear, actualizar o eliminar registros y ejecutar consultas para obtener todos los registros o subconjuntos particulares de registros. Te mostraremos cómo hacer esto en la sección Uso de modelos, y cuando creamos nuestras vistas.

### Schema types (fields)

Un esquema puede tener una cantidad arbitraria de campos; cada uno representa un campo en los documentos almacenados en MongoDB. A continuación se muestra un esquema de ejemplo que muestra muchos de los tipos de campo comunes y cómo se declaran.

```javascript
const schema = new Schema({
  name: String,
  binary: Buffer,
  living: Boolean,
  updated: { type: Date, default: Date.now() },
  age: { type: Number, min: 18, max: 65, required: true },
  mixed: Schema.Types.Mixed,
  _someId: Schema.Types.ObjectId,
  array: [],
  ofString: [String], // You can also have an array of each of the other types too.
  nested: { stuff: { type: String, lowercase: true, trim: true } },
});
```

La mayoría de los SchemaTypes (los descriptores después de "tipo:" o después de los nombres de campo) se explican por sí mismos. Las excepciones son:

* `ObjectId`: representa instancias específicas de un modelo en la base de datos. Por ejemplo, un libro podría usar esto para representar su objeto de autor. Esto realmente contendrá la ID única (_id) para el objeto especificado. Podemos usar el método `populate()` para extraer la información asociada cuando sea necesario.
* [Mixed](https://mongoosejs.com/docs/schematypes.html#mixed): un tipo de esquema arbitrario.
* `[]`: una matriz de elementos. Puedes realizar operaciones de matriz de JavaScript en estos modelos (`push`, `pop`, `unshift`, etc.). Los ejemplos  anteriores muestran una matriz de objetos sin un tipo específico y una matriz de objetos String, pero puede tener una matriz de cualquier tipo de objeto.

El código también muestra ambas formas de declarar un campo:

* Nombre y tipo de campo como un par clave-valor (es decir, como se hizo con los campos `name`, `binary` y `living`).
* Nombre de campo seguido de un objeto que define el tipo y cualquier otra opción para el campo. Las opciones incluyen cosas como:
  * valores predeterminados.
  * validadores incorporados (por ejemplo, valores máximos/mínimos) y funciones de validación personalizadas.
  * Si el campo es obligatorio
  * Si los campos de cadena deben configurarse automáticamente en minúsculas, mayúsculas o recortados (por ejemplo, `{ type: String, lowercase: true, trim: true }`))

Para obtener más información sobre las opciones, consulta [SchemaTypes](https://mongoosejs.com/docs/schematypes.html).

### Validación

Mongoose proporciona validadores integrados y personalizados, y validadores síncronos y asíncronos. Te permite especificar tanto el rango aceptable de valores como el mensaje de error por fallo de validación en todos los casos.

Los validadores integrados incluyen:

* Todos los [SchemaTypes](https://mongoosejs.com/docs/schematypes.html) tienen el validador [required](https://mongoosejs.com/docs/api.html#schematype_SchemaType-required) incorporado. Se utiliza para especificar si se debe proporcionar el campo para guardar un documento.
* Los [números](https://mongoosejs.com/docs/api.html#schema-number-js) tienen validadores [min](https://mongoosejs.com/docs/api.html#schema_number_SchemaNumber-min) y [max](https://mongoosejs.com/docs/api.html#schema_number_SchemaNumber-max).
* Las [cadenas](https://mongoosejs.com/docs/api.html#schema-string-js) tienen:
  * [enum](https://mongoosejs.com/docs/api.html#schema_string_SchemaString-enum): especifica el conjunto de valores permitidos para el campo.
  * [match](https://mongoosejs.com/docs/api.html#schema_string_SchemaString-match): especifica una expresión regular con la que debe coincidir la cadena.
  * [maxLength](https://mongoosejs.com/docs/api.html#schema_string_SchemaString-maxlength) y [minLength](https://mongoosejs.com/docs/api.html#schema_string_SchemaString-minlength) para la cadena.

El siguiente ejemplo (ligeramente modificado de los documentos de Mongoose) muestra cómo se pueden especificar algunos de los tipos de validadores y mensajes de error:

```javascript
const breakfastSchema = new Schema({
  eggs: {
    type: Number,
    min: [6, "Too few eggs"],
    max: 12,
    required: [true, "Why no eggs?"],
  },
  drink: {
    type: String,
    enum: ["Coffee", "Tea", "Water"],
  },
});
```

Para obtener información completa sobre la validación de campos, consulta [Validación](https://mongoosejs.com/docs/validation.html)

### Propiedades virtuales

Las propiedades virtuales son propiedades de documentos que se pueden obtener y establecer, pero **que no se conservan en MongoDB**. Los getters son útiles para dar formato o combinar campos, mientras que los setters son útiles para descomponer un solo valor en varios valores para su almacenamiento. El ejemplo de la documentación construye (y deconstruye) una propiedad virtual de nombre completo a partir de un campo de nombre y apellido, que es más fácil y limpio que construir un nombre completo cada vez que se usa uno en una plantilla.

> -info-Usaremos una propiedad virtual en la biblioteca para definir una URL única para cada registro de modelo usando una ruta y el valor _id del registro.

Para obtener más información, consulta [Virtuals](https://mongoosejs.com/docs/guide.html#virtuals)

### Métodos y asistentes de consulta

 Un esquema también puede tener métodos de instancia, métodos estáticos y asistentes de consulta. Los métodos de instancia y estáticos son similares, pero con la diferencia obvia de que un método de instancia está asociado con un registro en particular y tiene acceso al objeto actual. Los asistentes de consulta te permiten ampliar la API de generador de consultas encadenable de mongoose (por ejemplo, permitiéndole agregar una consulta "`byName`" además de los métodos `find()`, `findOne()` y `findById()`).

## Usar modelos

Una vez que hayas creado un esquema, puedes usarlo para crear modelos. El modelo representa una colección de documentos en la base de datos que puedes buscar, mientras que las instancias del modelo representan documentos individuales que puede guardar y recuperar.

Ofrecemos una breve descripción a continuación. Para obtener más información, consulta: [Modelos](https://mongoosejs.com/docs/models.html)

### Crear y modificar documentos

Para crear un registro, puedes definir una instancia del modelo y luego llamar a `save().` Los ejemplos a continuación asumen que `SomeModel` es un modelo (con un solo campo "name") que hemos creado a partir de nuestro esquema.

```java
// Create an instance of model SomeModel
const awesome_instance = new SomeModel({ name: "awesome" });

// Save the new model instance, passing a callback
awesome_instance.save((err) => {
  if (err) return handleError(err);
  // saved!
});
```

La creación de registros (junto con actualizaciones, eliminaciones y consultas) son operaciones asincrónicas: proporciona una devolución de llamada que se llama cuando se completa la operación. La API utiliza la convención de primer argumento de error, por lo que el primer argumento para la devolución de llamada siempre será un valor de error (o nulo). Si la API devuelve algún resultado, este se proporcionará como segundo argumento.

También puedes usar `create()` para definir la instancia del modelo al mismo tiempo que la guarda. La devolución de llamada devolverá un error para el primer argumento y la instancia de modelo recién creada para el segundo argumento.

```javascript
SomeModel.create({ name: "also_awesome" }, function (err, awesome_instance) {
  if (err) return handleError(err);
  // saved!
});
```

Cada modelo tiene una conexión asociada (esta será la conexión predeterminada cuando uses `mongoose.model()`). Creas una nueva conexión y llamas a `.model()` para crear los documentos en una base de datos diferente.

Puedes acceder a los campos en este nuevo registro utilizando la sintaxis de puntos y cambiar los valores. Debes llamar a `save()` o `update()` para almacenar los valores modificados en la base de datos.

```javascript
// Access model field values using dot notation
console.log(awesome_instance.name); //should log 'also_awesome'

// Change record by modifying the fields, then calling save().
awesome_instance.name = "New cool name";
awesome_instance.save((err) => {
  if (err) return handleError(err); // saved!
});
```

###  Buscar registros

Puedes buscar registros utilizando métodos de consulta, especificando las condiciones de consulta como un documento JSON. El fragmento de código a continuación muestra cómo puedes encontrar a todos los atletas en una base de datos que juegan a  tenis, devolviendo solo los campos para el nombre y la edad del atleta. Aquí solo especificamos un campo coincidente (deporte), pero puedes agregar más criterios, especificar criterios de expresión regular o eliminar las condiciones por completo para devolver a todos los atletas.

```java
const Athlete = mongoose.model("Athlete", yourSchema);

// find all athletes who play tennis, selecting the 'name' and 'age' fields
Athlete.find({ sport: "Tennis" }, "name age", (err, athletes) => {
  if (err) return handleError(err);
  // 'athletes' contains the list of athletes that match the criteria.
});
```

Si especificas una devolución de llamada, como se muestra arriba, la consulta se ejecutará de inmediato. La devolución de llamada se invocará cuando se complete la búsqueda.

> -info-Todas las devoluciones de llamada en Mongoose usan la devolución de llamada de patrón (error, resultado). Si ocurre un error al ejecutar la consulta, el parámetro de error contendrá un documento de error y el resultado será nulo. Si la consulta es exitosa, el parámetro de error será nulo y el resultado se completará con los resultados de la consulta.

> -info-Es importante recordar que no encontrar ningún resultado no es un error para una búsqueda, pero puede ser un caso fallido en el contexto de su aplicación. Si su aplicación espera que una búsqueda encuentre un valor, puedes verificar el resultado en la devolución de llamada (resultados == null) o conectar en cadena el método `orFail()` en la consulta.

Si no especificas una devolución de llamada, la API devolverá una variable de tipo [Query](https://mongoosejs.com/docs/api.html#query-js). Puedes usar este objeto de consulta para crear tu consulta y luego ejecutarla (con una devolución de llamada) más tarde usando el método `exec()`.

```javascript
// find all athletes that play tennis
const query = Athlete.find({ sport: "Tennis" });

// selecting the 'name' and 'age' fields
query.select("name age");

// limit our results to 5 items
query.limit(5);

// sort by age
query.sort({ age: -1 });

// execute the query at a later time
query.exec((err, athletes) => {
  if (err) return handleError(err);
  // athletes contains an ordered list of 5 athletes who play Tennis
});
```

Arriba hemos definido las condiciones de consulta en el método `find()`. También podemos hacer esto usando una función `where()`, y podemos encadenar todas las partes de nuestra consulta usando el operador de punto (.) en lugar de agregarlas por separado. El fragmento de código a continuación es el mismo que nuestra consulta anterior, con una condición adicional para la edad.

```javascript
Athlete.find()
  .where("sport")
  .equals("Tennis")
  .where("age")
  .gt(17)
  .lt(50) // Additional where query
  .limit(5)
  .sort({ age: -1 })
  .select("name age")
  .exec(callback); // where callback is the name of our callback function.
```

El método [find()](https://mongoosejs.com/docs/api.html#query_Query-find) obtiene todos los registros coincidentes, pero a menudo solo deseas obtener una coincidencia. Los siguientes métodos consultan por un solo registro:

* [findById()](https://mongoosejs.com/docs/api.html#model_Model.findById): encuentra el documento con la identificación especificada (cada documento tiene una identificación única).
*  [findOne()](https://mongoosejs.com/docs/api.html#query_Query-findOne): busca un solo documento que coincida con los criterios especificados.
* [findByIdAndRemove()](https://mongoosejs.com/docs/api.html#model_Model.findByIdAndRemove), [findByIdAndUpdate()](https://mongoosejs.com/docs/api.html#model_Model.findByIdAndUpdate), [findOneAndRemove()](https://mongoosejs.com/docs/api.html#query_Query-findOneAndRemove), [findOneAndUpdate()](https://mongoosejs.com/docs/api.html#query_Query-findOneAndUpdate): busca un solo documento por ID o criterio y lo actualiza o lo elimina. Estas son funciones de conveniencia útiles para actualizar y eliminar registros.

> -hint-También hay un método [count()](https://mongoosejs.com/docs/api.html#model_Model.count) que puedes usar para obtener la cantidad de elementos que coinciden con las condiciones. Esto es útil si desea realizar un conteo sin obtener realmente los registros.

Hay mucho más que puedes hacer con las consultas. Para más información ver: [Queries](https://mongoosejs.com/docs/queries.html)

### Trabajar con documentos relacionados: population

Puedes crear referencias de una instancia de documento/modelo a otra usando el campo de esquema ObjectId, o de un documento a muchos usando una matriz de ObjectIds. El campo almacena el id del modelo relacionado. Si necesitas el contenido real del documento asociado, puedes usar el método [populate()](https://mongoosejs.com/docs/api.html#query_Query-populate) en una consulta para reemplazar la identificación con los datos reales.

Por ejemplo, el siguiente esquema define autores e historias. Cada autor puede tener varias historias, que representamos como una matriz de ObjectId. Cada historia puede tener un solo autor. La propiedad ref le dice al esquema qué modelo se puede asignar a este campo.

```javascript
const mongoose = require("mongoose");

const Schema = mongoose.Schema;

const authorSchema = Schema({
  name: String,
  stories: [{ type: Schema.Types.ObjectId, ref: "Story" }],
});

const storySchema = Schema({
  author: { type: Schema.Types.ObjectId, ref: "Author" },
  title: String,
});

const Story = mongoose.model("Story", storySchema);
const Author = mongoose.model("Author", authorSchema);
```

Podemos guardar nuestras referencias al documento relacionado asignando el valor _id. A continuación, creamos un autor, luego una historia y asignamos la identificación del autor al campo de autor de nuestra historia.

```javascript
const bob = new Author({ name: "Bob Smith" });

bob.save((err) => {
  if (err) return handleError(err);

  // Bob now exists, so lets create a story
  const story = new Story({
    title: "Bob goes sledding",
    author: bob._id, // assign the _id from our author Bob. This ID is created by default!
  });

  story.save((err) => {
    if (err) return handleError(err);
    // Bob now has his story
  });
});
```

Nuestro documento de historia ahora tiene un autor al que se hace referencia mediante el ID del documento de autor. Para obtener la información del autor en los resultados de la historia, usamos `populate()`, como se muestra a continuación.

```javascript
Story.findOne({ title: "Bob goes sledding" })
  .populate("author") // This populates the author id with actual author information!
  .exec((err, story) => {
    if (err) return handleError(err);
    console.log("The author is %s", story.author.name);
    // prints "The author is Bob Smith"
  });
```

> -info- Los lectores astutos habrán notado que agregamos un autor a nuestra historia, pero no hicimos nada para agregar nuestra historia a la matriz de historias de nuestro autor. Entonces, ¿cómo podemos obtener todas las historias de un autor en particular? Una forma sería agregar nuestra historia a la matriz de historias, pero esto daría como resultado que tengamos dos lugares donde se debe mantener la información relacionada con los autores y las historias.
>
> Una mejor manera es obtener el _id de nuestro autor, luego usar `find()` para buscarlo en el campo del autor en todas las historias.
>
> ```javascript
> Story.find({ author: bob._id }).exec((err, stories) => {
>   if (err) return handleError(err);
>   // returns all stories that have Bob's id as their author.
> });
> ```

Esto es casi todo lo que necesitas saber sobre cómo trabajar con elementos relacionados para este tutorial. Para obtener información más detallada, consulta [Population](https://mongoosejs.com/docs/populate.html)

### Un esquema/modelo por fichero

Si bien puedes crear esquemas y modelos utilizando cualquier estructura de archivo que desees, te recomiendo que definas cada esquema de modelo en su propio módulo (archivo) y luego exportes el método para crear el modelo. Esto se muestra a continuación:

```javascript
// File: ./models/somemodel.js

// Require Mongoose
const mongoose = require("mongoose");

// Define a schema
const Schema = mongoose.Schema;

const SomeModelSchema = new Schema({
  a_string: String,
  a_date: Date,
});

// Export function to create "SomeModel" model class
module.exports = mongoose.model("SomeModel", SomeModelSchema);
```

A continuación, puede hacer `require` y utilizar el modelo inmediatamente en otros archivos. A continuación, mostramos cómo puedes usarlo para obtener todas las instancias del modelo.

```javascript
// Create a SomeModel model just by requiring the module
const SomeModel = require("../models/somemodel");

// Use the SomeModel object (model) to find all SomeModel records
SomeModel.find(callback_function);
```

## Configuración de la base de datos MongoDB

Ahora que entendemos algo de lo que Mongoose puede hacer y cómo queremos diseñar nuestros modelos, es hora de comenzar a trabajar en el sitio web de LocalLibrary. Lo primero que queremos hacer es configurar una base de datos MongoDB que podamos usar para almacenar los datos de nuestra biblioteca.

### Usar localhost

Primero hemos de instalar MongoDB. En el momento de escribir este tutorial, la versión que existe en el repositorio de Ubuntu es incompatible con algunas librerías del sistema por lo que no se puede instalar desde esos repos

Usa los siguientes comandos:

```bash
wget -qO - https://www.mongodb.org/static/pgp/server-6.0.asc |  gpg --dearmor | sudo tee /usr/share/keyrings/mongodb.gpg > /dev/null
echo "deb [ arch=amd64,arm64 signed-by=/usr/share/keyrings/mongodb.gpg ] https://repo.mongodb.org/apt/ubuntu jammy/mongodb-org/6.0 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-6.0.list
sudo apt update
sudo apt install mongodb-org
sudo chown -R mongodb:mongodb /var/lib/mongodb
```

Luego inicia el servicio mediante 

```
sudo systemctl start mongod
```

Y haz que inicie automáticamente como un servicio al arrancar la máquina.

```
sudo systemctl enable mongod
```

Para conectar desde la línea de comandos usa:

```
mongosh
```

Y para crear una Colección, usa

```
use local_library
```

### Usar MongoDB Atlas

Para este tutorial, vamos a utilizar la base de datos sandbox alojada en la nube de [MongoDB Atlas](https://www.mongodb.com/atlas/database). Este nivel de base de datos no se considera adecuado para sitios web de producción porque no tiene redundancia, pero es excelente para el desarrollo y la creación de prototipos. Lo estamos usando aquí porque es gratis y fácil de configurar, y porque MongoDB Atlas es una base de datos popular como proveedor de servicios que podría elegir razonablemente para su base de datos de producción (otras opciones populares en el momento de escribir este artículo incluyen [Compose](https://www.compose.com/), [ScaleGrid](https://scalegrid.io/pricing.html) y [ObjectRocket](https://www.objectrocket.com/)).

Primero deberás [crear una cuenta con MongoDB Atlas](https://www.mongodb.com/cloud/atlas/register) (esto es gratis y solo requiere que ingreses los datos de contacto básicos y reconozcas sus términos de servicio).

Después de iniciar sesión, accederá a la pantalla de [inicio](https://cloud.mongodb.com/v2):

1. Haz clic en el botón Crear una base de datos en la sección Implementaciones de base de datos.

![](/node-express-library-teoria/assets/img/mongodb_atlas_-_createdatabase.jpg)

2. Esto abrirá la pantalla Deploy una base de datos en la nube. Haga clic en el botón Crear debajo de la opción Shared Deployment

   ![](/node-express-library-teoria/assets/img/mongodb_atlas_-_deploy.jpg)

3. Esto abrirá la pantalla  *Create a Shared Cluster*

   ![](/node-express-library-teoria/assets/img/mongodb_atlas_-_createsharedcluster.jpg)

* Selecciona cualquier proveedor de la sección Proveedor de la nube y región. Diferentes regiones ofrecen diferentes proveedores.
* No es necesario cambiar el nivel de clúster y la configuración adicional. Puedes cambiar el nombre de su clúster en Nombre del clúster. Lo estamos nombrando Cluster0 para este tutorial.
* Haz clic en el botón Crear clúster (la creación del clúster tardará unos minutos).

4. Esto abrirá la sección *Security Quickstart*.    

   ![](/node-express-library-teoria/assets/img/mongodb_atlas_-_securityquickstart.jpg)

   Introduce un nombre de usuario y contraseña. Recuerda copiar y almacenar las credenciales de forma segura, ya que las necesitaremos más adelante. Haz clic en el botón Crear usuario.

   > -alert-Evita usar caracteres especiales en tu contraseña de usuario de MongoDB, ya que es posible que mongoose no analice la cadena de conexión correctamente.
   > Introduce 0.0.0.0/0 en el campo Dirección IP. Esto le dice a MongoDB que queremos permitir el acceso desde cualquier lugar. Haga clic en el botón Agregar entrada.

   > -info-Es una buena práctica limitar las direcciones IP que pueden conectarse a su base de datos y otros recursos. Aquí permitimos una conexión desde cualquier lugar porque no sabemos de dónde vendrá la solicitud después de la implementación.
   >
   >  Haga clic en el botón Finalizar y cerrar.

5. Esto abrirá la siguiente pantalla. Haga clic en el botón Ir a bases de datos.

   ![](/node-express-library-teoria/assets/img/mongodb_atlas_-_accessrules.jpg)

6. Volverás a la pantalla  *Database Deployments* . Haz clic en **Browse Collections**

   ![](/node-express-library-teoria/assets/img/mongodb_atlas_-_createcollection.jpg)

7. Esto abrirá la sección *Collections*. Haz clic en **Add My Own Data**.        

   ![](/node-express-library-teoria/assets/img/mongodb_atlas_-_adddata.jpg)

8. Esto abrirá la pantalla *Create Database*

   ![](/node-express-library-teoria/assets/img/mongodb_atlas_-_databasedetails.jpg)

   * Introducir el nombre de la nueva base de datos como `local_library`.
   * Introduce el nombre de la colección como Collection0.
   * Haga clic en el botón Crear para crear la base de datos.

9. Volverás a la pantalla de colecciones con la base de datos creada

   ![](/node-express-library-teoria/assets/img/mongodb_atlas_-_databasecreated.jpg)

   Haz clic en la pestaña *Overview* para volver al clúster

10. Haz clic en el botón **Connect**

    ![](/node-express-library-teoria/assets/img/mongodb_atlas_-_connectbutton.jpg)

11. Esto abrirá la pantalla *Connect to Cluster* screen. Haz clic en **Connect your application** .        

    ![](/node-express-library-teoria/assets/img/mongodb_atlas_-_chooseaconnectionmethod.jpg)

12. Se mostrará la pantalla **Connect**
    ![](/node-express-library-teoria/assets/img/mongodb_atlas_-_connectforshortsrv.jpg)

    * Selecciona el controlador y la versión de node como se muestra.
    * Haz clic en el icono Copiar para copiar la cadena de conexión.
    * Pega esto en su editor de texto local.
    * Actualiza el nombre de usuario y la contraseña con tu contraseña de usuario.
    * Inserte el nombre de la base de datos "`local_library`" en la ruta antes de las opciones (...mongodb.net/local_library?retryWrites...)/
    * Guarda el archivo que contiene esta cadena en un lugar seguro.

Ahora has creado la base de datos y tienes una URL (con nombre de usuario y contraseña) que se puede usar para acceder a ella. Esto se verá así:

```
mongodb+srv://tu_nombre_de_usuario:tu_contraseña@cluster0.lz91hw2.mongodb.net/local_library?retryWrites=true&w=majority
```

## Instalar mongoose

Abre un símbolo del sistema y navega hasta el directorio donde creaste el esqueleto de sitio web  de la biblioteca local. Introduce el siguiente comando para instalar Mongoose (y sus dependencias) y se agregará a tu archivo `package.json`, a menos que ya lo hayas hecho.

```
npm install mongoose
```

### Conectar a la base de datos

Abre `/app.js` (en la raíz de tu proyecto) y copia el siguiente texto debajo donde declara el objeto de la aplicación Express (después de la línea `var app = express();`). Reemplaza la cadena de URL de la base de datos ('insert_your_database_url_here') con la ubicación URL que representa su propia base de datos (es decir, utilizando la información de mongoDB Atlas o de localhost).

```javascript
// Set up mongoose connection
const mongoose = require("mongoose");
mongoose.set('strictQuery', false);
const mongoDB = "insert_your_database_url_here";

main().catch(err => console.log(err));
async function main() {
  await mongoose.connect(mongoDB);
}
```

En nuestro caso la url de la base de datos en localhost es:

```
mongodb://127.0.0.1:27017/library
```

y en Atlas

```
mongodb+srv://tu_nombre_de_usuario:tu_contraseña@tu-nombre-de-cluster/local_library?retryWrites=true&w=majority
```



## Definir el esquema

Definiremos un módulo separado para cada modelo, como se discutió anteriormente. Comienza creando una carpeta para nuestros modelos en la raíz del proyecto (`/models`) y luego cree archivos separados para cada uno de los modelos:

```
/express-locallibrary-tutorial  // the project root
  /models
    author.js
    book.js
    bookinstance.js
    genre.js
```

### Author model

Copia el código del esquema de autor que se muestra a continuación y pégalo en el archivo `./models/author.js`. El esquema define que un autor tiene tipos de esquema de cadena para el nombre y el apellido (obligatorio, con un máximo de 100 caracteres) y campos de fecha para las fechas de nacimiento y muerte.

```javascript
const mongoose = require("mongoose");

const Schema = mongoose.Schema;

const AuthorSchema = new Schema({
  first_name: { type: String, required: true, maxLength: 100 },
  family_name: { type: String, required: true, maxLength: 100 },
  date_of_birth: { type: Date },
  date_of_death: { type: Date },
});

// Virtual for author's full name
AuthorSchema.virtual("name").get(function () {
  // To avoid errors in cases where an author does not have either a family name or first name
  // We want to make sure we handle the exception by returning an empty string for that case
  let fullname = "";
  if (this.first_name && this.family_name) {
    fullname = `${this.family_name}, ${this.first_name}`;
  }
  if (!this.first_name || !this.family_name) {
    fullname = "";
  }
  return fullname;
});

// Virtual for author's URL
AuthorSchema.virtual("url").get(function () {
  // We don't use an arrow function as we'll need the this object
  return `/catalog/author/${this._id}`;
});

// Export model
module.exports = mongoose.model("Author", AuthorSchema);
```

También declaramos un virtual para `AuthorSchema` llamado "url" que devuelve la URL absoluta requerida para obtener una instancia particular del modelo; usaremos la propiedad en nuestras plantillas siempre que necesitemos obtener un enlace a un autor en particular.

> -info-Declarar nuestras URL como virtuales en el esquema es una buena idea porque entonces la URL de un elemento solo necesita cambiarse en un lugar. En este punto, un enlace que use esta URL no funcionaría, porque no tenemos ningún código de manejo de rutas para instancias de modelos individuales. ¡Los configuraremos en un artículo posterior!

Al final del módulo, exportamos el modelo.

### Book model

Copa el código de esquema del libro que se muestra a continuación y pégalo en su archivo `./models/book.js`. La mayor parte de esto es similar al modelo de autor: hemos declarado un esquema con varios campos de cadena y un virtual para obtener la URL de registros de libros específicos, y hemos exportado el modelo.

```javascript
const mongoose = require("mongoose");

const Schema = mongoose.Schema;

const BookSchema = new Schema({
  title: { type: String, required: true },
  author: { type: Schema.Types.ObjectId, ref: "Author", required: true },
  summary: { type: String, required: true },
  isbn: { type: String, required: true },
  genre: [{ type: Schema.Types.ObjectId, ref: "Genre" }],
});

// Virtual for book's URL
BookSchema.virtual("url").get(function () {
  // We don't use an arrow function as we'll need the this object
  return `/catalog/book/${this._id}`;
});

// Export model
module.exports = mongoose.model("Book", BookSchema);
```

La principal diferencia aquí es que hemos creado dos referencias a otros modelos:

* `author` es una referencia a un solo objeto de modelo Author y es obligatorio.
* `genre` es una referencia a una matriz de objetos de modelo Género. ¡Aún no hemos declarado este objeto!

### Bookinstance model

Finalmente, copia el código de esquema de `BookInstance` que se muestra a continuación y pégalo en el archivo `./models/bookinstance.js`. BookInstance representa una copia específica de un libro que alguien podría tomar prestada e incluye información sobre si la copia está disponible, en qué fecha se espera que se devuelva y detalles de "impresión" (o versión).

```java
const mongoose = require("mongoose");

const Schema = mongoose.Schema;

const BookInstanceSchema = new Schema({
  book: { type: Schema.Types.ObjectId, ref: "Book", required: true }, // reference to the associated book
  imprint: { type: String, required: true },
  status: {
    type: String,
    required: true,
    enum: ["Available", "Maintenance", "Loaned", "Reserved"],
    default: "Maintenance",
  },
  due_back: { type: Date, default: Date.now },
});

// Virtual for bookinstance's URL
BookInstanceSchema.virtual("url").get(function () {
  // We don't use an arrow function as we'll need the this object
  return `/catalog/bookinstance/${this._id}`;
});

// Export model
module.exports = mongoose.model("BookInstance", BookInstanceSchema);
```

Las cosas nuevas que mostramos aquí son las opciones de campo:

* `enum`: Esto nos permite establecer los valores permitidos de una cadena. En este caso, lo usamos para especificar el estado de disponibilidad de nuestros libros (usar una enumeración significa que podemos evitar errores ortográficos y valores arbitrarios para nuestro estado).
*  `default`: usamos `default` para establecer el estado predeterminado para las instancias de libros recién creadas en mantenimiento y la fecha de vencimiento predeterminada para ahora (¡observa cómo puedes llamar a la función `Date` al configurar la fecha!).

Todo lo demás debe ser familiar de nuestro esquema anterior.

### Genre model

> -reto-Abre tu archivo `./models/genre.js` y crea un esquema para almacenar géneros (la categoría del libro, por ejemplo, si es ficción o no ficción, romance o historia militar, etc.).
>
> La definición será muy similar a los otros modelos:
>
> * El modelo debe tener un campo de tipo String llamado  `name` para describir el género.
>
> * Este nombre debe ser obligatorio y tener entre 3 y 100 caracteres.
>
> * Declara una virtual para la URL del género, llamada url.
>
> Exporta el modelo.

### Testeo. Crear unos cuantos ítems

Eso es. ¡Ya tenemos todos los modelos para el sitio configurados!

Para probar los modelos (y crear algunos libros de ejemplo y otros elementos que podemos usar en nuestros próximos artículos), ahora ejecutaremos un script independiente para crear elementos de cada tipo:

1. Descarga el archivo [populatedb.js](/node-express-library-teoria/assets/populatedb.js) dentro de tu directorio express-locallibrary-tutorial (en el mismo nivel que `package.json`).

2. Introduce los siguientes comandos en la raíz del proyecto para instalar el módulo `async` que requiere el script

   ```
   npm install async
   ```

3. Ejecuta el script usando `node` en el símbolo del sistema, pasando la URL de su base de datos MongoDB (la misma con la que reemplazaste el marcador de posición `insert_your_database_url_here`, dentro de `app.js` anteriormente):

   ```
   node populatedb <tu-url-de-mongo>
   ```

4. El script debe ejecutarse hasta su finalización, mostrando los elementos a medida que los crea en la terminal.

> -info-Ve a tu base de datos en mongoDB Atlas (en la pestaña Colecciones). Ahora deberías poder profundizar en colecciones individuales de libros, autores, géneros e instancias de libros, y consultar documentos individuales.

## Véase también

- [Database integration](https://expressjs.com/en/guide/database-integration.html)
- [Mongoose website](https://mongoosejs.com/)
- [Mongoose Guide](https://mongoosejs.com/docs/guide.html)
- [Validation](https://mongoosejs.com/docs/validation.html)
- [Schema Types](https://mongoosejs.com/docs/schematypes.html)
- [Models](https://mongoosejs.com/docs/models.html)
- [Queries](https://mongoosejs.com/docs/queries.html)
- [Population](https://mongoosejs.com/docs/populate.html)
