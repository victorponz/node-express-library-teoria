---
typora-copy-images-to: ../assets/img/
typora-root-url: ../../
layout: post
title: 'Tutorial Express - parte 4: Rutas y controladores'
categories: parte1
conToc: true
permalink: rutas-y-controladores
---

En este tutorial, configuraremos rutas (código de manejo de URL) con funciones de controlador "ficticias" para todos los puntos finales de recursos que eventualmente necesitaremos en el sitio web de LocalLibrary. Al finalizar, tendremos una estructura modular para nuestro código de manejo de rutas, que podemos ampliar con funciones de controlador reales en los siguientes artículos. ¡También comprenderemos muy bien cómo crear rutas modulares usando Express!

## Descripción general

En el último artículo del tutorial, definimos modelos Mongoose para interactuar con la base de datos y usamos un script (independiente) para crear algunos registros de biblioteca iniciales. Ahora podemos escribir el código para presentar esa información a los usuarios. Lo primero que debemos hacer es determinar qué información queremos poder mostrar en nuestras páginas y luego definir las URL apropiadas para devolver esos recursos. Luego, necesitaremos crear las rutas (controladores de URL) y las vistas (plantillas) para mostrar esas páginas.

El siguiente diagrama se proporciona como un recordatorio del flujo principal de datos y las cosas que deben implementarse al manejar una solicitud/respuesta HTTP. Además de las vistas y las rutas, el diagrama muestra "controladores", funciones que separan el código para enrutar las solicitudes del código que realmente procesa las solicitudes.

Como ya hemos creado los modelos, las cosas principales que necesitaremos crear son:

* "Rutas" para reenviar las solicitudes admitidas (y cualquier información codificada en las URL de solicitud) a las funciones de controlador correspondientes.
*  El controlador funciona para obtener los datos solicitados de los modelos, crea una página HTML que muestra los datos y se los devuelve al usuario para que los vea en el navegador.
*  Vistas (plantillas) utilizadas por los controladores para representar los datos.

![](/node-express-library-teoria/assets/img/mvc_express.png)

En última instancia, podríamos tener páginas para mostrar listas e información detallada de libros, géneros, autores e instancias de libros, junto con páginas para crear, actualizar y eliminar registros. Eso es mucho para documentar en un artículo. Por lo tanto, la mayor parte de este artículo se concentrará en configurar nuestras rutas y controladores para devolver contenido "ficticio". Ampliaremos los métodos del controlador en nuestros artículos posteriores para trabajar con datos del modelo.

La primera sección a continuación proporciona una breve "instrucción" sobre cómo usar el middleware Express [Router](https://expressjs.com/en/4x/api.html#router). Luego usaremos ese conocimiento en las siguientes secciones cuando configuremos las rutas de LocalLibrary.

## Introducción a las rutas

Una ruta es una sección de código Express que asocia un verbo HTTP (GET, POST, PUT, DELETE, etc.), una ruta/patrón de URL y una función que se llama para manejar ese patrón.

Hay varias formas de crear rutas. Para este tutorial, vamos a utilizar el middleware express.Router, ya que nos permite agrupar los controladores de ruta para una parte particular de un sitio y acceder a ellos usando un prefijo de ruta común. Mantendremos todas nuestras rutas relacionadas con la biblioteca en un módulo de "catálogo" y, si agregamos rutas para manejar cuentas de usuario u otras funciones, podemos mantenerlas agrupadas por separado.

### Definición y uso de módulos de ruta separados

El siguiente código proporciona un ejemplo concreto de cómo podemos crear un módulo de ruta y luego usarlo en una aplicación Express.

Primero creamos rutas para una wiki en un módulo llamado `wiki.js`. El código primero importa el objeto de la aplicación Express, lo usa para obtener un objeto de enrutador y luego le agrega un par de rutas usando el método get(). Por último, el módulo exporta el objeto Router.

```javascript
// wiki.js - Wiki route module.

const express = require("express");
const router = express.Router();

// Home page route.
router.get("/", function (req, res) {
  res.send("Wiki home page");
});

// About page route.
router.get("/about", function (req, res) {
  res.send("About this wiki");
});

module.exports = router;
```

> -info-Arriba, estamos definiendo las devoluciones de llamada de nuestro controlador de ruta directamente en las funciones del enrutador. En LocalLibrary, definiremos estas devoluciones de llamada en un módulo de controlador separado.

Para usar el módulo de enrutador en nuestro archivo de aplicación principal, primero hacemos `require()` el módulo de ruta (wiki.js). Luego llamamos a use() en la aplicación Express para agregar el enrutador a la ruta de manejo del middleware, especificando una ruta URL de 'wiki'.

```java
const wiki = require("./wiki.js");
// …
app.use("/wiki", wiki);
```

Las dos rutas definidas en nuestro módulo de ruta wiki son accesibles desde `/wiki/` y `/wiki/about/`.

## Funciones del router

Nuestro módulo anterior define un par de funciones de ruta típicas. La ruta "acerca de" (reproducida a continuación) se define mediante el método `Router.get()`, que responde solo a las solicitudes `HTTP GET`. El primer argumento de este método es la ruta de la URL, mientras que el segundo es una función de devolución de llamada que se invocará si se recibe una solicitud `HTTP GET` con la ruta.

```javascript
router.get("/about", function (req, res) {
  res.send("About this wiki");
});
```

La devolución de llamada toma tres argumentos (generalmente denominados como se muestra: `req`, `res`, `next`), que contendrán el objeto de solicitud HTTP, la respuesta HTTP y la siguiente función en la cadena de middleware.

> -info-Las funciones del enrutador son [middleware Express](https://developer.mozilla.org/en-US/docs/Learn/Server-side/Express_Nodejs/Introduction#using_middleware), lo que significa que deben completar (responder) la solicitud o llamar a la siguiente función en la cadena. En el caso anterior, completamos la solicitud usando send(), por lo que no se usa el siguiente argumento (y elegimos no especificarlo).

La función de enrutador anterior toma una sola devolución de llamada, pero puede especificar tantos argumentos de devolución de llamada como desee, o una matriz de funciones de devolución de llamada. Cada función es parte de la cadena de middleware y se llamará en el orden en que se agrega a la cadena (a menos que una función anterior complete la solicitud).

La función de devolución de llamada aquí llama a [`send()`](https://expressjs.com/en/4x/api.html#res.send) en la respuesta para devolver la cadena "Acerca de este wiki" cuando recibimos una solicitud GET con la ruta ('/acerca de'). Hay una serie de [otros métodos de respuesta](https://expressjs.com/en/guide/routing.html#response-methods) para finalizar el ciclo de solicitud/respuesta. Por ejemplo, podría llamar a [`res.json()`](https://expressjs.com/en/4x/api.html#res.json) para enviar una respuesta JSON o [`res.sendFile()`](https://expressjs.com/en/4x/api.html#res.sendFile) para enviar un archivo. El método de respuesta que usaremos con más frecuencia a medida que construimos la biblioteca es [`render()`](https://expressjs.com/en/4x/api.html#res.render), que crea y devuelve archivos HTML usando plantillas y datos. ¡Hablaremos mucho más sobre eso en un artículo posterior!

## Verbos HTTP

El ejemplo anterior hace uso de `Router.get()` para responder a peticiones con el método `GET`

También se pueden usar los siguientes métodos: `post()`, `put()`, `delete()`, `options()`, `trace()`, `copy()`, `lock()`, `mkcol()`, `move()`, `purge()`, `propfind()`, `proppatch()`, `unlock()`, `report()`, `mkactivity()`, `checkout()`, `merge()`, `m-search()`, `notify()`, `subscribe()`, `unsubscribe()`, `patch()`, `search()`, y `connect()`.

Por ejemplo, el siguiente código se comporta como la ruta `/about` anterior, pero solo responde a las solicitudes HTTP POST.

```javascript
router.post("/about", (req, res) => {
  res.send("About this wiki");
});
```

## Paths en las rutas

Los paths de ruta definen los puntos finales en los que se pueden realizar solicitudes. Los ejemplos que hemos visto hasta ahora son solo cadenas y se usan exactamente como están escritos: '/', '/about', '/book', '/any-random.path'.

Los paths de ruta también pueden ser patrones de cadenas. Los patrones de cadena utilizan una forma de sintaxis de expresión regular para definir patrones de puntos finales que coincidirán. La sintaxis se enumera a continuación (ten en cuenta que el guión (-) y el punto (.) se interpretan literalmente mediante rutas basadas en cadenas):

* `?` : El endpoint debe tener 0 o 1 del carácter anterior (o grupo). Un path de ruta de `'/ab?cd'` coincidirá con los endpoint `acd` o `abcd`.
* `+` : El endpoint debe tener 1 o más del carácter anterior (o grupo). Un path de ruta de `'/ab+cd'` coincidirá con los endpoints `abcd`, `abbcd`, `abbbcd`, etc.
* `*`: el punto final puede tener una cadena arbitraria donde se coloca el carácter. Un path de ruta `'/ab*cd'` coincidirá con los endpoints `abcd`, `abXcd`, `abSOMErandomTEXTcd`, etc.
* `()` : Coincidencia de agrupación en un conjunto de caracteres para realizar otra operación. `'/ab(cd)?e'` realizará una coincidencia `?` en el grupo (`cd`) — coincidirá con `abe` y `abcde`.

Los paths de ruta también pueden ser [expresiones regulares de JavaScript](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Regular_Expressions). Por ejemplo, el siguiente path de ruta coincidirá con `catfish` y `dogfish`, pero no con `catflap`, `catfishhead`, etc. Ten en cuenta que la ruta de una expresión regular utiliza la sintaxis de expresiones regulares (no es una cadena entre comillas como en los casos anteriores).

```javascript
app.get(/.*fish$/, function (req, res) {
  // …
});
```

## Parámetros en las rutas

Los parámetros de ruta son segmentos de URL con nombre que se utilizan para capturar valores en posiciones específicas de la URL. Los segmentos nombrados tienen el prefijo de dos puntos y luego el nombre (por ejemplo, `/:your_parameter_name/`). Los valores capturados se almacenan en el objeto `req.params` usando los nombres de los parámetros como claves (por ejemplo, `req.params.your_parameter_name`).

Entonces, por ejemplo, considere una URL codificada para contener información sobre usuarios y libros: [http://localhost:3000/users/34/books/8989](http://localhost:3000/users/34/books/8989). Podemos extraer esta información como se muestra a continuación, con los parámetros de ruta `userId` y `bookId`:

```javascript
app.get("/users/:userId/books/:bookId", (req, res) => {
  // Access userId via: req.params.userId
  // Access bookId via: req.params.bookId
  res.send(req.params);
});
```

Los nombres de los parámetros de ruta deben estar formados por "caracteres de palabra" (A-Z, a-z, 0-9 y _).

> -info-La URL `/book/create` coincidirá con una ruta como `/book/:bookId` (que extraerá un valor "bookid" de 'create'). Se utilizará la primera ruta que coincida con una URL entrante, por lo que si deseas procesar `/book/create` URL por separado, su controlador de ruta debes definirse antes de tu ruta `/book/:bookId`.

Eso es todo lo que necesitas para comenzar con las rutas; si es necesario, puedes encontrar más información en los documentos [Express: Enrutamiento básico](https://expressjs.com/en/starter/basic-routing.html) y [Guía de enrutamiento](https://expressjs.com/en/guide/routing.html). Las siguientes secciones muestran cómo configuraremos nuestras rutas y controladores para LocalLibrary.Los nombres de los parámetros de ruta deben estar formados por "caracteres de palabra" (A-Z, a-z, 0-9 y _).

## Rutas necesarias para LocalLibrary

Las URL que finalmente necesitaremos para nuestras páginas se enumeran a continuación, donde objeto se reemplaza por el nombre de cada uno de nuestros modelos (book, bookinstance, genre, author), objects es el plural de object e id es el campo de instancia único (_id) que se otorga a cada instancia del modelo Mongoose de forma predeterminada.

* `catalog/` — La página de inicio/índice.
* `catalog/<objects>/` — La lista de todos los libros, instancias de libros, géneros o autores (por ejemplo, `/catalog/books/`, `/catalog/genres/,` etc.)
* `catalog/<object>/<id>` — La página de detalles para un libro específico, instancia de libro, género o autor con el valor de campo _id dado (por ejemplo, /catalog/book/584493c1f4887f06c0e67d37).
* `catalog/<object>/create` — El formulario para crear un nuevo libro, instancia de libro, género o autor (por ejemplo, `/catalog/book/create`).
* `catalog/<object>/<id>/update` — El formulario para actualizar un libro específico, instancia de libro, género o autor con el valor de campo _id dado (por ejemplo, `/catalog/book/584493c1f4887f06c0e67d37/update`).
* `catalog/<object>/<id>/delete`: el formulario para eliminar un libro específico, una instancia de libro, un género o un autor con el valor de campo _id dado (por ejemplo, `/catalog/book/584493c1f4887f06c0e67d37/delete`).

La primera página de inicio y las páginas de lista no codifican ninguna información adicional. Si bien los resultados devueltos dependerán del tipo de modelo y el contenido de la base de datos, las consultas que se ejecutan para obtener la información siempre serán las mismas (de manera similar, el código que se ejecuta para la creación de objetos siempre será similar).

Por el contrario, las otras URL se utilizan para actuar en una instancia específica de documento/modelo; estas codifican la identidad del elemento en la URL (que se muestra como `<id>` arriba). Usaremos parámetros de ruta para extraer la información codificada y pasarla al controlador de ruta (y en un artículo posterior usaremos esto para determinar dinámicamente qué información obtener de la base de datos). Al codificar la información en nuestra URL, solo necesitamos una ruta para cada recurso de un tipo particular (por ejemplo, una ruta para manejar la visualización de cada elemento de libro).

> -info-Express te permite construir sus URL de la forma que desees; puedes codificar información en el cuerpo de la URL como se muestra arriba o usar parámetros GET de URL (por ejemplo, `/book/?id=6`). Independientemente del enfoque que utilices, las URL deben mantenerse **limpias, lógicas y legibles** (consulta los [consejos del W3C](https://www.w3.org/Provider/Style/URI)).

A continuación, creamos nuestras funciones de devolución de llamada del controlador de ruta y el código de ruta para todas las URL anteriores.

## Crear las callback del controlador de ruta

Antes de definir nuestras rutas, primero crearemos todas las funciones de devolución de llamada ficticias/esqueléticas que invocarán. Las devoluciones de llamada se almacenarán en módulos de "controlador" separados para libros, instancias de libros, géneros y autores (puedes usar cualquier estructura de archivo/módulo, pero parece una granularidad apropiada para este proyecto).

Comienza creando una carpeta para nuestros controladores en la raíz del proyecto (`/controllers`) y luego crea archivos/módulos de controlador separados para manejar cada uno de los modelos:

```
/express-locallibrary-tutorial  //the project root
  /controllers
    authorController.js
    bookController.js
    bookinstanceController.js
    genreController.js
```

### Author controller

Abre el controlador `/controller/authorcontroller.js` e introduce el siguiente código:

```javascript
const Author = require("../models/author");

// Display list of all Authors.
exports.author_list = (req, res) => {
  res.send("NOT IMPLEMENTED: Author list");
};

// Display detail page for a specific Author.
exports.author_detail = (req, res) => {
  res.send(`NOT IMPLEMENTED: Author detail: ${req.params.id}`);
};

// Display Author create form on GET.
exports.author_create_get = (req, res) => {
  res.send("NOT IMPLEMENTED: Author create GET");
};

// Handle Author create on POST.
exports.author_create_post = (req, res) => {
  res.send("NOT IMPLEMENTED: Author create POST");
};

// Display Author delete form on GET.
exports.author_delete_get = (req, res) => {
  res.send("NOT IMPLEMENTED: Author delete GET");
};

// Handle Author delete on POST.
exports.author_delete_post = (req, res) => {
  res.send("NOT IMPLEMENTED: Author delete POST");
};

// Display Author update form on GET.
exports.author_update_get = (req, res) => {
  res.send("NOT IMPLEMENTED: Author update GET");
};

// Handle Author update on POST.
exports.author_update_post = (req, res) => {
  res.send("NOT IMPLEMENTED: Author update POST");
};
```

El módulo primero requiere el **modelo** que luego usaremos para acceder y actualizar nuestros datos. Luego **exporta** funciones para cada una de las URL que deseamos manejar (las operaciones de creación, actualización y eliminación usan formularios y, por lo tanto, también tienen métodos adicionales para manejar solicitudes de publicación de formularios; discutiremos esos métodos en el "artículo de formularios" más adelante).

Todas las funciones tienen la forma estándar de una función de middleware Express, con argumentos para la solicitud y la respuesta. También podríamos incluir la siguiente función a llamar si el método no completa el ciclo de solicitud, pero en todos estos casos lo hace, por lo que la hemos omitido. Los métodos devuelven una cadena que indica que la página asociada aún no se ha creado. Si se espera que una función de controlador reciba parámetros de ruta, estos se emiten en la cadena de mensaje (ver `req.params.id` arriba).

### Bookinstance Controller

Abre el archivo `/controller/bookinstanceController.js` y escribe el siguiente código. Sigue el mismo patrón que el controlador para Autores

```javascript
const BookInstance = require("../models/bookinstance");

// Display list of all BookInstances.
exports.bookinstance_list = (req, res) => {
  res.send("NOT IMPLEMENTED: BookInstance list");
};

// Display detail page for a specific BookInstance.
exports.bookinstance_detail = (req, res) => {
  res.send(`NOT IMPLEMENTED: BookInstance detail: ${req.params.id}`);
};

// Display BookInstance create form on GET.
exports.bookinstance_create_get = (req, res) => {
  res.send("NOT IMPLEMENTED: BookInstance create GET");
};

// Handle BookInstance create on POST.
exports.bookinstance_create_post = (req, res) => {
  res.send("NOT IMPLEMENTED: BookInstance create POST");
};

// Display BookInstance delete form on GET.
exports.bookinstance_delete_get = (req, res) => {
  res.send("NOT IMPLEMENTED: BookInstance delete GET");
};

// Handle BookInstance delete on POST.
exports.bookinstance_delete_post = (req, res) => {
  res.send("NOT IMPLEMENTED: BookInstance delete POST");
};

// Display BookInstance update form on GET.
exports.bookinstance_update_get = (req, res) => {
  res.send("NOT IMPLEMENTED: BookInstance update GET");
};

// Handle bookinstance update on POST.
exports.bookinstance_update_post = (req, res) => {
  res.send("NOT IMPLEMENTED: BookInstance update POST");
};
```

### Genre Controller

Abre el archivo `/controller/genreController.js` y escribe el siguiente código. Sigue el mismo patrón que los otros controladores:

```javascript
const Genre = require("../models/genre");

// Display list of all Genre.
exports.genre_list = (req, res) => {
  res.send("NOT IMPLEMENTED: Genre list");
};

// Display detail page for a specific Genre.
exports.genre_detail = (req, res) => {
  res.send(`NOT IMPLEMENTED: Genre detail: ${req.params.id}`);
};

// Display Genre create form on GET.
exports.genre_create_get = (req, res) => {
  res.send("NOT IMPLEMENTED: Genre create GET");
};

// Handle Genre create on POST.
exports.genre_create_post = (req, res) => {
  res.send("NOT IMPLEMENTED: Genre create POST");
};

// Display Genre delete form on GET.
exports.genre_delete_get = (req, res) => {
  res.send("NOT IMPLEMENTED: Genre delete GET");
};

// Handle Genre delete on POST.
exports.genre_delete_post = (req, res) => {
  res.send("NOT IMPLEMENTED: Genre delete POST");
};

// Display Genre update form on GET.
exports.genre_update_get = (req, res) => {
  res.send("NOT IMPLEMENTED: Genre update GET");
};

// Handle Genre update on POST.
exports.genre_update_post = (req, res) => {
  res.send("NOT IMPLEMENTED: Genre update POST");
};
```

### Book Controller

Abre el archivo `/controller/bookController.js` y escribe el siguiente código. Sigue el mismo patrón que los otros controladores pero además tiene un método `index()` para mostrar la página de inicio:

```javascript
const Book = require("../models/book");

exports.index = (req, res) => {
  res.send("NOT IMPLEMENTED: Site Home Page");
};

// Display list of all books.
exports.book_list = (req, res) => {
  res.send("NOT IMPLEMENTED: Book list");
};

// Display detail page for a specific book.
exports.book_detail = (req, res) => {
  res.send(`NOT IMPLEMENTED: Book detail: ${req.params.id}`);
};

// Display book create form on GET.
exports.book_create_get = (req, res) => {
  res.send("NOT IMPLEMENTED: Book create GET");
};

// Handle book create on POST.
exports.book_create_post = (req, res) => {
  res.send("NOT IMPLEMENTED: Book create POST");
};

// Display book delete form on GET.
exports.book_delete_get = (req, res) => {
  res.send("NOT IMPLEMENTED: Book delete GET");
};

// Handle book delete on POST.
exports.book_delete_post = (req, res) => {
  res.send("NOT IMPLEMENTED: Book delete POST");
};

// Display book update form on GET.
exports.book_update_get = (req, res) => {
  res.send("NOT IMPLEMENTED: Book update GET");
};

// Handle book update on POST.
exports.book_update_post = (req, res) => {
  res.send("NOT IMPLEMENTED: Book update POST");
};
```

## Crear el módulo de ruta del catálogo

A continuación, creamos rutas para todas las URL que necesita el sitio web de LocalLibrary, que llamará a las funciones del controlador que definimos en la sección anterior.

El esqueleto ya tiene una carpeta `./routes` que contiene rutas para el índice y los usuarios. Creea otro archivo de ruta, `catalog.js`, dentro de esta carpeta, como se muestra.

```
/express-locallibrary-tutorial //the project root
  /routes
    index.js
    users.js
    catalog.js
```

Abre `routes/catalog.js` y pega el siguiente código:

```java
const express = require("express");
const router = express.Router();

// Require controller modules.
const book_controller = require("../controllers/bookController");
const author_controller = require("../controllers/authorController");
const genre_controller = require("../controllers/genreController");
const book_instance_controller = require("../controllers/bookinstanceController");

/// BOOK ROUTES ///

// GET catalog home page.
router.get("/", book_controller.index);

// GET request for creating a Book. NOTE This must come before routes that display Book (uses id).
router.get("/book/create", book_controller.book_create_get);

// POST request for creating Book.
router.post("/book/create", book_controller.book_create_post);

// GET request to delete Book.
router.get("/book/:id/delete", book_controller.book_delete_get);

// POST request to delete Book.
router.post("/book/:id/delete", book_controller.book_delete_post);

// GET request to update Book.
router.get("/book/:id/update", book_controller.book_update_get);

// POST request to update Book.
router.post("/book/:id/update", book_controller.book_update_post);

// GET request for one Book.
router.get("/book/:id", book_controller.book_detail);

// GET request for list of all Book items.
router.get("/books", book_controller.book_list);

/// AUTHOR ROUTES ///

// GET request for creating Author. NOTE This must come before route for id (i.e. display author).
router.get("/author/create", author_controller.author_create_get);

// POST request for creating Author.
router.post("/author/create", author_controller.author_create_post);

// GET request to delete Author.
router.get("/author/:id/delete", author_controller.author_delete_get);

// POST request to delete Author.
router.post("/author/:id/delete", author_controller.author_delete_post);

// GET request to update Author.
router.get("/author/:id/update", author_controller.author_update_get);

// POST request to update Author.
router.post("/author/:id/update", author_controller.author_update_post);

// GET request for one Author.
router.get("/author/:id", author_controller.author_detail);

// GET request for list of all Authors.
router.get("/authors", author_controller.author_list);

/// GENRE ROUTES ///

// GET request for creating a Genre. NOTE This must come before route that displays Genre (uses id).
router.get("/genre/create", genre_controller.genre_create_get);

//POST request for creating Genre.
router.post("/genre/create", genre_controller.genre_create_post);

// GET request to delete Genre.
router.get("/genre/:id/delete", genre_controller.genre_delete_get);

// POST request to delete Genre.
router.post("/genre/:id/delete", genre_controller.genre_delete_post);

// GET request to update Genre.
router.get("/genre/:id/update", genre_controller.genre_update_get);

// POST request to update Genre.
router.post("/genre/:id/update", genre_controller.genre_update_post);

// GET request for one Genre.
router.get("/genre/:id", genre_controller.genre_detail);

// GET request for list of all Genre.
router.get("/genres", genre_controller.genre_list);

/// BOOKINSTANCE ROUTES ///

// GET request for creating a BookInstance. NOTE This must come before route that displays BookInstance (uses id).
router.get(
  "/bookinstance/create",
  book_instance_controller.bookinstance_create_get
);

// POST request for creating BookInstance.
router.post(
  "/bookinstance/create",
  book_instance_controller.bookinstance_create_post
);

// GET request to delete BookInstance.
router.get(
  "/bookinstance/:id/delete",
  book_instance_controller.bookinstance_delete_get
);

// POST request to delete BookInstance.
router.post(
  "/bookinstance/:id/delete",
  book_instance_controller.bookinstance_delete_post
);

// GET request to update BookInstance.
router.get(
  "/bookinstance/:id/update",
  book_instance_controller.bookinstance_update_get
);

// POST request to update BookInstance.
router.post(
  "/bookinstance/:id/update",
  book_instance_controller.bookinstance_update_post
);

// GET request for one BookInstance.
router.get("/bookinstance/:id", book_instance_controller.bookinstance_detail);

// GET request for list of all BookInstance.
router.get("/bookinstances", book_instance_controller.bookinstance_list);

module.exports = router;
```

El módulo requiere Express y luego lo usa para crear un objeto de enrutador. Todas las rutas se configuran en el enrutador, que luego se exporta.

Las rutas se definen utilizando los métodos `.get()` o `.post()` en el objeto del enrutador. Todas las rutas se definen mediante cadenas (no utilizamos patrones de cadenas ni expresiones regulares). Las rutas que actúan sobre algún recurso específico (por ejemplo, un libro) usan parámetros de ruta para obtener la identificación del objeto de la URL.

Todas las funciones del controlador se importan de los módulos del controlador que creamos en la sección anterior.

### Actualiza el módulo de ruta de índice

Hemos configurado todas nuestras rutas nuevas, pero todavía tenemos una ruta a la página original. En su lugar, redirijamos esto a la nueva página de índice que hemos creado en la ruta `/catalog`.

Abre `/routes/index.js` y reemplaza la ruta existente con la función a continuación.

```javascript
// GET home page.
router.get("/", function (req, res) {
  res.redirect("/catalog");
});
```

> -info-Este es nuestro primer uso del método de respuesta `redirect()`. Esto redirige a la página especificada, enviando de forma predeterminada el código de estado HTTP "302 Encontrado". Puede cambiar el código de estado devuelto si es necesario y proporcionar rutas absolutas o relativas.

### Actualizar app.js

El último paso es agregar las rutas a la cadena de middleware. Hacemos esto en `app.js`.

Abre `app.js` y solicite la ruta del catálogo debajo de las otras rutas (agrega la tercera línea que se muestra a continuación, debajo de las otras dos):

```javascript
var indexRouter = require("./routes/index");
var usersRouter = require("./routes/users");
const catalogRouter = require("./routes/catalog"); //Import routes for "catalog" area of site
```

A continuación, agrega la ruta del catálogo a la pila de middleware debajo de las otras rutas (agrega la tercera línea que se muestra a continuación, debajo de las otras dos):

```javascript
app.use("/", indexRouter);
app.use("/users", usersRouter);
app.use("/catalog", catalogRouter); // Add catalog routes to middleware chain.
```

> -info-Hemos agregado nuestro módulo de catálogo en una ruta `/catalog`. Esto se antepone a todas las rutas definidas en el módulo de catálogo. Entonces, por ejemplo, para acceder a una lista de libros, la URL será: `/catalog/books/`.

Eso es. Ahora deberíamos tener rutas y funciones básicas habilitadas para todas las URL que admitiremos eventualmente en el sitio web de LocalLibrary.

## Testear las rutas

Primero lanzamos node

```
DEBUG=express-locallibrary-tutorial:* npm run devstart
```

Luego navega a varias URL de LocalLibrary y verifica que no obtengas una página de error (HTTP 404). A continuación se incluye un pequeño conjunto de direcciones URL para tu comodidad:

- `http://localhost:3000/`
- `http://localhost:3000/catalog`
- `http://localhost:3000/catalog/books`
- `http://localhost:3000/catalog/bookinstances/`
- `http://localhost:3000/catalog/authors/`
- `http://localhost:3000/catalog/genres/`
- `http://localhost:3000/catalog/book/5846437593935e2f8c2aa226`
- `http://localhost:3000/catalog/book/create`