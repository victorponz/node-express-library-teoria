---
typora-copy-images-to: ../assets/img/
typora-root-url: ../../
layout: post
title: 'Tutorial Express - parte 5: Visualización de datos'
categories: parte1
conToc: true
permalink: visualizacion-de-datos
---

Ahora estamos listos para agregar las páginas que muestran los libros del sitio web de LocalLibrary y otros datos. Las páginas incluirán una página de inicio que muestra cuántos registros tenemos de cada tipo de modelo y una lista y páginas de detalles para todos nuestros modelos. En el camino, obtendremos experiencia práctica en la obtención de registros de la base de datos y en el uso de plantillas.

## Descripción general

En nuestros artículos tutoriales anteriores, definimos modelos Mongoose que podemos usar para interactuar con una base de datos y creamos algunos registros de biblioteca iniciales. Luego creamos todas las rutas necesarias para el sitio web de LocalLibrary, pero con funciones de "controlador ficticio" (estas son funciones de controlador de esqueleto que simplemente devuelven un mensaje "no implementado" cuando se accede a una página).

El siguiente paso es proporcionar implementaciones adecuadas para las páginas que muestran la información de nuestra biblioteca (veremos cómo implementar páginas con formularios para crear, actualizar o eliminar información en artículos posteriores). Esto incluye actualizar las funciones del controlador para obtener registros utilizando nuestros modelos y definir plantillas para mostrar esta información a los usuarios.

Comenzaremos brindando información general/temas básicos que explican cómo administrar operaciones asincrónas en funciones de controlador y cómo escribir plantillas usando Pug. Luego, proporcionaremos implementaciones para cada una de nuestras páginas principales de "solo lectura" con una breve explicación de cualquier característica especial o nueva que utilicen.

Al final de este artículo, debe tener una buena comprensión integral de cómo funcionan en la práctica las rutas, las funciones asincrónicas, las vistas y los modelos

## Control de flujo asíncrono usando async

El código del controlador para algunas de nuestras páginas de LocalLibrary dependerá de los resultados de varias solicitudes asíncronas, que pueden ser necesarias para ejecutarse en un orden particular o en paralelo. Para administrar el control de flujo y mostrar páginas cuando tengamos toda la información requerida disponible, usaremos el popular módulo [async](https://www.npmjs.com/package/async) de node.

> -info-Hay otras formas de administrar el comportamiento asíncrono y el control de flujo en JavaScript, incluidas funciones relativamente recientes del lenguaje JavaScript como [Promises](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise).

Async tiene muchos métodos útiles. Algunas de las funciones más importantes son:

* `async.parallel()` para ejecutar cualquier operación que deba realizarse en paralelo.
* `async.series()` para cuando necesitamos asegurarnos de que las operaciones asíncronas se realicen en serie.
* `async.waterfall()` para operaciones que deben ejecutarse en serie, y cada operación depende de los resultados de las operaciones anteriores.

### ¿Por qué es necesario?

La mayoría de los métodos que usamos en Express son asíncronos: especificas una operación para realizar, pasando una devolución de llamada. El método regresa inmediatamente y la devolución de llamada se invoca cuando se completa la operación solicitada. Por convención en Express, las funciones de devolución de llamada pasan un valor de error como primer parámetro (o nulo en caso de éxito) y los resultados de la función (si los hay) como segundo parámetro.

Si un controlador solo necesita realizar una operación asíncrona para obtener la información requerida para representar una página, entonces la implementación es fácil: representamos la plantilla en la devolución de llamada. El siguiente fragmento de código muestra esto para una función que representa el conteo de un modelo SomeModel (usando el método Mongoose [countDocuments](https://mongoosejs.com/docs/api.html#model_Model-countDocuments)):

```javascript
exports.some_model_count = function (req, res, next) {
  SomeModel.countDocuments(
    { a_model_field: "match_value" },
    function (err, count) {
      // Do something if there is an err.
      // …

      // On success, render the result by passing count into the render function (here, as the variable 'data').
      res.render("the_template", { data: count });
    }
  );
};
```

¿Qué sucede si necesitas realizar varias consultas asíncronas y no puedes representar la página hasta que se hayan completado todas las operaciones? Una implementación ingenua podría "conectar en cadena" las solicitudes, iniciar solicitudes posteriores en la devolución de llamada de una solicitud anterior y presentar la respuesta en la devolución de llamada final. El problema con este enfoque es que nuestras solicitudes tendrían que ejecutarse en serie, aunque podría ser más eficiente ejecutarlas en paralelo. Esto también podría resultar en un código anidado complicado, comúnmente conocido como [callback hell](http://callbackhell.com/).

Una solución mucho mejor sería ejecutar todas las solicitudes en paralelo y luego tener una única devolución de llamada que se ejecute cuando se hayan completado todas las consultas. ¡Este es el tipo de operación de flujo que el módulo Async facilita!

### Operaciones asíncronas en paralelo

El método [async.parallel()](https://caolan.github.io/async/v3/docs.html#parallel) se usa para ejecutar varias operaciones asíncronas en paralelo.

El primer argumento para `async.parallel()` es una colección de funciones asíncronas para ejecutar (una matriz, un objeto u otro iterable). A cada función se le pasa una devolución de llamada (`err`, `result`) que debe llamar al finalizar con un error `err` (que puede ser nulo) y un valor `results` opcional.

El segundo argumento opcional de `async.parallel()` es una devolución de llamada que se ejecutará cuando se hayan completado todas las funciones del primer argumento. La devolución de llamada se invoca con un argumento de error y una colección de resultados que contiene los resultados de las operaciones asíncronas individuales. La colección de resultados es del mismo tipo que el primer argumento (es decir, si pasa una matriz de funciones asíncronas, la devolución de llamada final se invocará con una matriz de resultados). Si alguna de las funciones paralelas informa un error, la devolución de llamada se invoca antes (con el valor de error).

El siguiente ejemplo muestra cómo funciona esto cuando pasamos un objeto como primer argumento. Como puedes ver, los resultados se devuelven en un objeto con los mismos nombres de propiedad que las funciones originales que se pasaron.

```javascript
async.parallel(
  {
    one(callback) {
      /* … */
    },
    two(callback) {
      /* … */
    },
    // …
    something_else(callback) {
      /* … */
    },
  },
  // optional callback
  function (err, results) {
    // 'results' is now equal to: {one: 1, two: 2, …, something_else: some_value}
  }
);
```

Si, en cambio, pasas una matriz de funciones como primer argumento, los resultados serán una matriz (los resultados del orden de la matriz coincidirán con el orden original en que se declararon las funciones, no con el orden en que se completaron).

### Operaciones asíncronas en serie

El método [async.series()](https://caolan.github.io/async/v3/docs.html#series) se usa para ejecutar varias operaciones asíncronas en secuencia, cuando las funciones posteriores no dependen de la salida de funciones anteriores. Básicamente se declara y se comporta de la misma manera que `async.parallel()`.

```javascript
async.series(
  {
    one(callback) {
      // …
    },
    two(callback) {
      // …
    },
    // …
    something_else(callback) {
      // …
    },
  },
  // optional callback after the last asynchronous function completes.
  function (err, results) {
    // 'results' is now equal to: {one: 1, two: 2, /* …, */ something_else: some_value}
  }
);
```

> -info-La especificación del lenguaje ECMAScript (JavaScript) establece que el orden de enumeración de un objeto no está definido, por lo que es posible que las funciones no se llamen en el mismo orden en que las especifica en todas las plataformas. Si el orden es realmente importante, debes pasar una matriz en lugar de un objeto, como se muestra a continuación.

```javascript
async.series(
  [
    function (callback) {
      // do some stuff …
      callback(null, "one");
    },
    function (callback) {
      // do some more stuff …
      callback(null, "two");
    },
  ],
  // optional callback
  function (err, results) {
    // results is now equal to ['one', 'two']
  }
);
```

### Operaciones asíncronas dependientes en serie

El método [async.waterfall()](https://caolan.github.io/async/v3/docs.html#waterfall) se usa para ejecutar múltiples operaciones asíncronas en secuencia cuando cada operación depende del resultado de la operación anterior.

La devolución de llamada invocada por cada función asíncrona contiene un valor nulo para el primer argumento y da como resultado argumentos posteriores. Cada función de la serie toma los argumentos de resultados de la devolución de llamada anterior como los primeros parámetros y luego una función de devolución de llamada. Cuando se completan todas las operaciones, se invoca una devolución de llamada final con el resultado de la última operación. La forma en que esto funciona es más clara cuando considera el fragmento de código a continuación (este ejemplo es de la documentación de `async`):

```javascript
async.waterfall(
  [
    function (callback) {
      callback(null, "one", "two");
    },
    function (arg1, arg2, callback) {
      // arg1 now equals 'one' and arg2 now equals 'two'
      callback(null, "three");
    },
    function (arg1, callback) {
      // arg1 now equals 'three'
      callback(null, "done");
    },
  ],
  function (err, result) {
    // result now equals 'done'
  }
);
```

### Instalación de `async`

Instala el módulo asíncrono usando el administrador de paquetes `npm` para que podamos usarlo en nuestro código. Haz esto de la manera habitual, abriendo una consola en la raíz del proyecto LocalLibrary e introduciendo el siguiente comando:

```
npm install async
```

## Plantillas

Una plantilla es un archivo de texto que define la estructura o el diseño de un archivo de salida, con marcadores de posición que se utilizan para representar dónde se insertarán los datos cuando se represente la plantilla (en Express, las plantillas se denominan vistas).

### Opciones de plantilla Express

Express se puede utilizar con muchos motores de representación de plantillas diferentes. En este tutorial usamos Pug (anteriormente conocido como Jade) para nuestras plantillas. Este es el lenguaje de plantillas de Node más popular y se describe a sí mismo como una "sintaxis limpia y sensible a los espacios en blanco para escribir HTML, fuertemente influenciada por [Haml](https://haml.info/)".

Diferentes lenguajes de plantilla usan diferentes enfoques para definir el diseño y marcar marcadores de posición para los datos; algunos usan HTML para definir el diseño, mientras que otros usan diferentes formatos de marcado que se pueden transpilar a HTML. Pug es del segundo tipo; usa una representación de HTML donde la primera palabra en cualquier línea generalmente representa un elemento HTML, y la sangría en las líneas subsiguientes se usa para representar el anidamiento. El resultado es una definición de página que se traduce directamente a HTML, pero es más concisa y posiblemente más fácil de leer.

> -info-La desventaja de usar Pug es que es sensible a la sangría y los espacios en blanco (si agregas un espacio adicional en el lugar equivocado, puede obtener un código de error inútil). Sin embargo, una vez que tengas las plantillas en su lugar, son muy fáciles de leer y mantener.

### Configuración de plantillas

LocalLibrary se configuró para usar Pug cuando creamos el sitio web esqueleto. Deberías ver el módulo pug incluido como una dependencia en el archivo `package.json` del sitio web y los siguientes ajustes de configuración en el archivo `app.js`. La configuración nos dice que estamos usando `pug` como motor de visualización y que Express debe buscar plantillas en el subdirectorio `/views`.

```javascript
// View engine setup
app.set("views", path.join(__dirname, "views"));
app.set("view engine", "pug");
```

Si buscas en el directorio de vistas, verás los archivos `.pug` para las vistas predeterminadas del proyecto. Estos incluyen la vista de la página de inicio (`index.pug`) y la plantilla base (`layout.pug`) que necesitaremos reemplazar con nuestro propio contenido.

```
/express-locallibrary-tutorial  //the project root
  /views
    error.pug
    index.pug
    layout.pug
```

### Sintaxis de plantilla

El archivo de plantilla de ejemplo a continuación muestra muchas de las características más útiles de Pug.

Lo primero que debes notar es que el archivo mapea la estructura de un archivo HTML típico, con la primera palabra en (casi) cada línea siendo un elemento HTML, y la sangría se usa para indicar elementos anidados. Entonces, por ejemplo, el elemento del cuerpo está dentro de un elemento html y los elementos de párrafo (p) están dentro del elemento del cuerpo, etc. Los elementos no anidados (por ejemplo, párrafos individuales) están en líneas separadas.

```pug
doctype html
html(lang="en")
  head
    title= title
    script(type='text/javascript').
  body
    h1= title

    p This is a line with #[em some emphasis] and #[strong strong text] markup.
    p This line has un-escaped data: !{'<em> is emphasized</em>'} and escaped data: #{'<em> is not emphasized</em>'}.
      | This line follows on.
    p= 'Evaluated and <em>escaped expression</em>:' + title
    <!-- You can add HTML comments directly -->
    // You can add single line JavaScript comments and they are generated to HTML comments
    p A line with a link
      a(href='/catalog/authors') Some link text
      |  and some extra text.

    #container.col
      if title
        p A variable named "title" exists.
      else
        p A variable named "title" does not exist.
      p.
        Pug is a terse and simple template language with a
        strong focus on performance and powerful features.

    h2 Generate a list

    ul
      each val in [1, 2, 3, 4, 5]
        li= val
```

Los atributos de los elementos se definen entre paréntesis después de su elemento asociado. Dentro de los paréntesis, los atributos se definen en listas separadas por comas o espacios en blanco de los pares de nombres de atributos y valores de atributos, por ejemplo:

```jade
script(type='text/javascript'), link(rel='stylesheet', href='/stylesheets/style.css')
meta(name='viewport' content='width=device-width initial-scale=1')
```

Los valores de todos los atributos se escapan (por ejemplo, los caracteres como `>` se convierten en sus equivalentes de código HTML como `&gt;`) para evitar la inyección de JavaScript o cross-site scripting attacks.

Si una etiqueta va seguida del signo igual, el siguiente texto se trata como una expresión de JavaScript. Entonces, por ejemplo, en la primera línea a continuación, el contenido de la etiqueta `h1` será un título variable (ya sea definido en el archivo o pasado a la plantilla desde Express). En la segunda línea, el contenido del párrafo es una cadena de texto concatenada con la variable de título. En ambos casos, el comportamiento predeterminado es escapar de la línea.

```jade
h1= title
p= 'Evaluated and <em>escaped expression</em>:' + title
```

Si no hay un símbolo de igual después de la etiqueta, el contenido se trata como texto sin formato. Dentro del texto sin formato, puedes insertar datos con y sin escape usando la sintaxis `#{}` y `!{}` respectivamente, como se muestra a continuación. También puedes agregar HTML sin procesar dentro del texto sin formato.

```jade
p This is a line with #[em some emphasis] and #[strong strong text] markup.
p This line has an un-escaped string: !{'<em> is emphasized</em>'}, an escaped string: #{'<em> is not emphasized</em>'}, and escaped variables: #{title}.
```

> -info-Casi siempre querrás escapar de los datos de los usuarios (a través de la sintaxis `#{}`). Los datos en los que se puede confiar (por ejemplo, recuentos de registros generados, etc.) se pueden mostrar sin escapar de los valores.

Puedes utilizar el carácter de barra vertical (`|`) al principio de una línea para indicar "texto sin formato". Por ejemplo, el texto adicional que se muestra a continuación se mostrará en la misma línea que el ancla anterior, pero no estará vinculado.

```jade
a(href='http://someurl/') Link text
| Plain text
```

Pug te permite realizar operaciones condicionales usando `if`, `else`, `else if` y `unless`, por ejemplo:

```jade
if title
  p A variable named "title" exists
else
  p A variable named "title" does not exist
```

También puedes realizar operaciones de bucle/iteración utilizando la sintaxis `each-in` o `while`. En el fragmento de código a continuación, hemos recorrido una matriz para mostrar una lista de variables (ten en cuenta el uso de '`li =`' para evaluar el "`val`" como una variable a continuación. El valor que itera también se puede pasar al plantilla como una variable!

```jade
ul
  each val in [1, 2, 3, 4, 5]
    li= val
```

La sintaxis también admite comentarios (que se pueden representar en la salida, o no, según elija), `mixins` para crear bloques de código reutilizables, declaraciones de casos y muchas otras características. Para obtener información más detallada, consulta los documentos de [The Pug](https://pugjs.org/api/getting-started.html).

### Extender plantillas

En un sitio, es habitual que todas las páginas tengan una estructura común, incluido el marcado HTML estándar para el encabezado, el pie de página, la navegación, etc. En lugar de obligar a los desarrolladores a duplicar este "repetitivo" en cada página, Pug te permite declarar un plantilla base y luego se extiende, reemplazando solo lo que es diferente para cada página específica.

Por ejemplo, la plantilla base `layout.pug` creada en nuestro proyecto de esqueleto se ve así:

```jade
doctype html
html
  head
    title= title
    link(rel='stylesheet', href='/stylesheets/style.css')
  body
    block content
```

La etiqueta `block` se usa para marcar secciones de contenido que se pueden reemplazar en una plantilla derivada (si el bloque no se redefine, se usa su implementación en la clase base).

El `index.pug` predeterminado (creado para nuestro proyecto de esqueleto) muestra cómo anulamos la plantilla base. La etiqueta `extends` identifica la plantilla base a usar y luego usamos el bloque `section_name` para indicar el nuevo contenido de la sección que anularemos.

```jade
extends layout

block content
  h1= title
  p Welcome to #{title}
```

## La plantilla base

Ahora que entendemos cómo extender plantillas usando Pug, comencemos creando una plantilla base para el proyecto. Tendrá una barra lateral con enlaces para las páginas que esperamos crear en los artículos del tutorial (por ejemplo, para mostrar y crear libros, géneros, autores, etc.) y un área de contenido principal que anularemos en cada una de nuestras páginas individuales.

Abre `/views/layout.pug` y reemplace el contenido con el siguiente código.

```jade
doctype html
html(lang='en')
  head
    title= title
    meta(charset='utf-8')
    meta(name='viewport', content='width=device-width, initial-scale=1')
    link(rel="stylesheet", href="https://stackpath.bootstrapcdn.com/bootstrap/4.5.2/css/bootstrap.min.css", integrity="sha384-JcKb8q3iqJ61gNV9KGb8thSsNjpSL0n8PARn9HuZOnIxN0hoP+VmmDGMN5t9UJ0Z", crossorigin="anonymous")
    script(src="https://code.jquery.com/jquery-3.5.1.slim.min.js", integrity="sha384-DfXdz2htPH0lsSSs5nCTpuj/zy4C+OGpamoFVy38MVBnE+IbbVYUew+OrCXaRkfj", crossorigin="anonymous")
    script(src="https://stackpath.bootstrapcdn.com/bootstrap/4.5.2/js/bootstrap.min.js", integrity="sha384-B4gt1jrGC7Jh4AgTPSdUtOBvfO8shuf57BaghqFfPlYxofvL8/KUEfYiJOMMV+rV", crossorigin="anonymous")
    link(rel='stylesheet', href='/stylesheets/style.css')
  body
    div(class='container-fluid')
      div(class='row')
        div(class='col-sm-2')
          block sidebar
            ul(class='sidebar-nav')
              li
                a(href='/catalog') Home
              li
                a(href='/catalog/books') All books
              li
                a(href='/catalog/authors') All authors
              li
                a(href='/catalog/genres') All genres
              li
                a(href='/catalog/bookinstances') All book-instances
              li
                hr
              li
                a(href='/catalog/author/create') Create new author
              li
                a(href='/catalog/genre/create') Create new genre
              li
                a(href='/catalog/book/create') Create new book
              li
                a(href='/catalog/bookinstance/create') Create new book instance (copy)

        div(class='col-sm-10')
          block content
```

La plantilla usa (e incluye) JavaScript y CSS de Bootstrap para mejorar el diseño y la presentación de la página HTML. El uso de Bootstrap u otro marco web del lado del cliente es una forma rápida de crear una página atractiva que puede escalar bien en diferentes tamaños de navegador, y también nos permite manejar la presentación de la página sin tener que entrar en detalles, simplemente quiero centrarme en el código del lado del servidor aquí!

El diseño debería ser bastante obvio si has leído nuestro manual de plantilla anterior. Ten en cuenta el uso de `block content` como marcador de posición para el lugar donde se colocará el contenido de nuestras páginas individuales.

La plantilla base también hace referencia a un archivo CSS local (`style.css`) que proporciona un poco de estilo adicional. Abre `/public/stylesheets/style.css` y reemplace su contenido con el siguiente código CSS:

```css
.sidebar-nav {
  margin-top: 20px;
  padding: 0;
  list-style: none;
}
```

Ahora tenemos una plantilla base para crear páginas con una barra lateral. En las próximas secciones lo usaremos para definir las páginas individuales.

## Página de inicio

La primera página que crearemos será la página de inicio del sitio web, a la que se puede acceder desde la raíz del sitio ('`/`') o del catálogo (`catalog/`). Esto mostrará un texto estático que describe el sitio, junto con "recuentos" calculados dinámicamente de diferentes tipos de registros en la base de datos.

Ya hemos creado una ruta para la página de inicio. Para completar la página, necesitamos actualizar nuestra función de controlador para obtener "recuentos" de registros de la base de datos y crear una vista (plantilla) que podamos usar para representar la página.

### Ruta

Creamos nuestras rutas de página de índice en un tutorial anterior. Como recordatorio, todas las funciones de ruta están definidas en `/routes/catalog.js`:

```javascript
// GET catalog home page.
router.get("/", book_controller.index); //This actually maps to /catalog/ because we import the route with a /catalog prefix
```

Donde el parámetro de la función de devolución de llamada (`book_controller.index`) se define en `/controllers/bookController.js`:

Es esta función de controlador la que ampliamos para obtener información de nuestros modelos y luego representarla usando una plantilla (vista).

### Controlador

La función de controlador de índice necesita obtener información sobre cuántos registros `Book`, `BookInstance`, `BookInstance` disponibles, `Author` y `Genre` tenemos en la base de datos, representar estos datos en una plantilla para crear una página HTML y luego devolverlos en una respuesta HTTP.

> -info-Usamos el método [countDocuments()](https://mongoosejs.com/docs/api.html#model_Model-countDocuments) para obtener el número de instancias de cada modelo. Esto se llama en un modelo, con un conjunto opcional de condiciones para comparar en el primer argumento y una devolución de llamada en el segundo argumento (como se explica en Uso de una base de datos (con Mongoose), y también puede devolver una `Query` y luego ejecutar con una devolución de llamada más tarde). La devolución de llamada se invocará cuando la base de datos devuelva el recuento, con un valor de error como primer parámetro (o nulo) y el recuento de documentos como segundo parámetro (o nulo si hubo un error).
>
> ```javascript
> SomeModel.countDocuments({ a_model_field: "match_value" }, (err, count) => {
>   // Do something if there is an err
>   // Do something with the count if there was no error
> });
> ```

Abre `/controllers/bookController.js`. Cerca de la parte superior del archivo, deberías ver la función `index()` exportada.

```javascript
const Book = require("../models/book");

exports.index = (req, res, next) => {
  res.send("NOT IMPLEMENTED: Site Home Page");
};
```

Reemplaza todo el código anterior con el siguiente fragmento de código. Lo primero que hace es importar (`require()`) todos los modelos. Necesitamos hacer esto porque los usaremos para obtener nuestros conteos de documentos. Luego importa el módulo `async` (que discutimos anteriormente en Control de flujo asíncrono usando `async`).

```javascript
const Book = require("../models/book");
const Author = require("../models/author");
const Genre = require("../models/genre");
const BookInstance = require("../models/bookinstance");

const async = require("async");

exports.index = (req, res) => {
  async.parallel(
    {
      book_count(callback) {
        Book.countDocuments({}, callback); // Pass an empty object as match condition to find all documents of this collection
      },
      book_instance_count(callback) {
        BookInstance.countDocuments({}, callback);
      },
      book_instance_available_count(callback) {
        BookInstance.countDocuments({ status: "Available" }, callback);
      },
      author_count(callback) {
        Author.countDocuments({}, callback);
      },
      genre_count(callback) {
        Genre.countDocuments({}, callback);
      },
    },
    (err, results) => {
      res.render("index", {
        title: "Local Library Home",
        error: err,
        data: results,
      });
    }
  );
};
```

Al método `async.parallel()` se le pasa un objeto con funciones para obtener los recuentos de cada uno de nuestros modelos. Todas estas funciones se inician al mismo tiempo. Cuando todos han completado, se invoca la devolución de llamada final con los recuentos en el parámetro de resultados (o un error).

En caso de éxito, la función de devolución de llamada llama a `res.render()`, especificando una vista (plantilla) llamada `index` y un objeto que contiene los datos que se insertarán en él (esto incluye el objeto de resultados que contiene nuestro modelo). Los datos se proporcionan como pares clave-valor y se puede acceder a ellos en la plantilla mediante la clave.

### View

Abre `/views/index.pug` y reemplaza su contenido con el texto a continuación.

```jade
extends layout

block content
  h1= title
  p Welcome to #[em LocalLibrary], a very basic Express website developed as a tutorial example on the Mozilla Developer Network.

  h1 Dynamic content

  if error
    p Error getting dynamic content.
  else
    p The library has the following record counts:

    ul
      li #[strong Books:] !{data.book_count}
      li #[strong Copies:] !{data.book_instance_count}
      li #[strong Copies available:] !{data.book_instance_available_count}
      li #[strong Authors:] !{data.author_count}
      li #[strong Genres:] !{data.genre_count}
```

La vista es sencilla. Extendemos la plantilla base de `layout.pug`, anulando el bloque llamado `content`. El primer encabezado `h1` será el texto escapado para la variable de título que se pasó a la función `render()`; ten en cuenta el uso de `h1=` para que el siguiente texto se trate como una expresión de JavaScript. Luego incluimos un párrafo que presenta LocalLibrary.

Bajo el encabezado Dynamic content, verificamos si la variable de error que se pasó desde la función `render()` se ha definido. Si es así, anotamos el error. Si no, obtenemos y enumeramos el número de copias de cada modelo de la variable de datos.

> -info-No escapamos de los valores de conteo (es decir, usamos la sintaxis `!{}`) porque los valores de conteo se calculan. Si la información fue proporcionada por los usuarios finales, escaparíamos de la variable para mostrarla.

### Cómo se ve?

En este punto, deberíamos haber creado todo lo necesario para mostrar la página de índice. Ejecute la aplicación y abra su navegador en [http://localhost:3000/](http://localhost:3000/). Si todo está configurado correctamente, su sitio debería parecerse a la siguiente captura de pantalla.

![image-20230124184324539](/node-express-library-teoria/assets/img/image-20230124184324539.png)

## Página de lista de libros

A continuación, implementaremos nuestra página de lista de libros. Esta página debe mostrar una lista de todos los libros en la base de datos junto con su autor, y cada título de libro es un hipervínculo a su página de detalles de libro asociada.

### Controlador

La función del controlador de la lista de libros necesita obtener una lista de todos los objetos `Book` en la base de datos, ordenarlos y luego pasarlos a la plantilla para su representación.

Abre `/controllers/bookController.js`. Busca el método del controlador `book_list()` exportado y reemplázalo con el siguiente código.

```javascript
// Display list of all Books.
exports.book_list = function (req, res, next) {
  Book.find({}, "title author")
    .sort({ title: 1 })
    .populate("author")
    .exec(function (err, list_books) {
      if (err) {
        return next(err);
      }
      //Successful, so render
      res.render("book_list", { title: "Book List", book_list: list_books });
    });
};
```

El método usa la función `find()` del modelo para devolver todos los objetos `Book`, seleccionando devolver solo el `title` y el `author` ya que no necesitamos los otros campos (también devolverá el `_id` y los campos virtuales), y luego ordena los resultados por `title` alfabéticamente usando el método `sort()`. Aquí también llamamos `populate()` en `Book`, especificando el campo de `author`; esto reemplazará la identificación del autor del libro almacenado con los detalles completos del autor.

En caso de éxito, la devolución de llamada pasada a la consulta representa la plantilla `book_list` (.pug), pasando el título y `book_list` (lista de libros con autores) como variables.

### Vista

Crea `/views/book_list.pug` y copie el texto a continuación.

```jade
extends layout

block content
  h1= title

  ul
    each book in book_list
      li
        a(href=book.url) #{book.title}
        |  (#{book.author.name})

    else
      li There are no books.
```

La vista extiende la plantilla base de `layout.pug` y anula el bloque llamado `contect`. Muestra el título que le pasamos desde el controlador (a través del método `render()`) e itera a través de la variable `book_list` usando la sintaxis `each-in-else`. Se crea un elemento de lista para cada libro que muestra el título del libro como un enlace a la página de detalles del libro seguido del nombre del autor. Si no hay libros en `book_list`, se ejecuta la cláusula `else` y se muestra el texto 'There are no books'.

> -info-Usamos `book.url` para proporcionar el enlace al registro detallado de cada libro (hemos implementado esta ruta, pero aún no la página). Esta es una propiedad virtual del modelo `Book` que utiliza el campo `_id` de la instancia del modelo para producir una ruta URL única.

De interés aquí es que cada libro se define como dos líneas, utilizando la tubería para la segunda línea. Este enfoque es necesario porque si el nombre del autor estuviera en la línea anterior, sería parte del hipervínculo.

![image-20230124185318470](/node-express-library-teoria/assets/img/image-20230124185318470.png)

## Página de instancias de libros

> -reto-Crea el controlador que muestre todas las instancias de libros. La plantilla es la siguiente:
>
> ```jade
> extends layout
> 
> block content
>   h1= title
> 
>   ul
>     each val in bookinstance_list
>       li
>         a(href=val.url) #{val.book.title} : #{val.imprint} -
>         if val.status=='Available'
>           span.text-success #{val.status}
>         else if val.status=='Maintenance'
>           span.text-danger #{val.status}
>         else
>           span.text-warning #{val.status}
>         if val.status!='Available'
>           span  (Due: #{val.due_back} )
> 
>     else
>       li There are no book copies in this library.
> 
> ```

![image-20230124185718573](/node-express-library-teoria/assets/img/image-20230124185718573.png)

## Formateo de fechas usando luxon

El renderizado de las fechas es bastante feo:

```
(Due: Sun Jan 22 2023 19:11:04 GMT+0100 (hora estándar de Europa central) )
```

Vamos a usar la librería [luxon](https://moment.github.io/luxon/#/)

Para instalarla, usamos `npm`

```
npm install luxon
```

### Crear una propiedad virtual

1. Abre `modles/bookinstance.js`

2. Al principio de la página, importa luxon

   ```java
   const { DateTime } = require("luxon");
   ```

3. Añade la propiedad virtual `due_back_formatted` al modelo después de la propiedad URL

   ```javascript
   BookInstanceSchema.virtual("due_back_formatted").get(function () {
     return    DateTime.fromJSDate(this.due_back).toLocaleString(DateTime.DATE_MED);
   });
   ```

### Actualizar la vista

Modifica la vista para que ahora renderice esta propiedad virtual.

```java
      if val.status != 'Available'
        //span  (Due: #{val.due_back} )
        span  (Due: #{val.due_back_formatted} )

```



## Página de libros

> -reto-Crea la página para listar los libros. Modifica las fechas de nacimiento y defunción al igual que hicimos en bookinstance

![image-20230124191604572](/node-express-library-teoria/assets/img/image-20230124191604572.png)

## Página de géneros

> -reto-Crea la página para listar los géneros

![image-20230124192352341](/node-express-library-teoria/assets/img/image-20230124192352341.png)

## Página detalle de género

La página de detalles del género debe mostrar la información de una instancia de género en particular, utilizando su valor de campo `_id` generado automáticamente como identificador. La página debe mostrar el nombre del género y una lista de todos los libros del género con enlaces a la página de detalles de cada libro.

### Controlador

Abre `/controllers/genreController.js` e importa los módulos `async` y `Book` en la parte superior del archivo.

```javascript
const Book = require("../models/book");
const async = require("async");
```

Encuentra el método `genre_detail()` y sustitúyelo por el siguiente código:

```javascript
// Display detail page for a specific Genre.
exports.genre_detail = (req, res, next) => {
  async.parallel(
    {
      genre(callback) {
        Genre.findById(req.params.id).exec(callback);
      },

      genre_books(callback) {
        Book.find({ genre: req.params.id }).exec(callback);
      },
    },
    (err, results) => {
      if (err) {
        return next(err);
      }
      if (results.genre == null) {
        // No results.
        const err = new Error("Genre not found");
        err.status = 404;
        return next(err);
      }
      // Successful, so render
      res.render("genre_detail", {
        title: "Genre Detail",
        genre: results.genre,
        genre_books: results.genre_books,
      });
    }
  );
};
```

El método usa `async.parallel()` para consultar el nombre del género y sus libros asociados en paralelo, y la devolución de llamada muestra la página cuando (`if`) ambas solicitudes se completan correctamente.

El ID del registro de género requerido se codifica al final de la URL y se extrae automáticamente según la definición de la ruta (`/genre/:id`). Se accede a la ID dentro del controlador a través de los parámetros de solicitud: `req.params.id`. Se usa en `Genre.findById()` para obtener el género actual. También se utiliza para obtener todos los objetos `Book` que tienen el ID de género en su campo de género: `Book.find({ 'genre': req.params.id })`.

> -info-Si el género no existe en la base de datos (es decir, es posible que se haya eliminado), `findById()` volverá correctamente sin resultados. En este caso, queremos mostrar una página "no encontrada", por lo que creamos un objeto de error y lo pasamos a la siguiente función de middleware en la cadena.
>
> ```javascript
> if (results.genre == null) {
>   // No results.
>   const err = new Error("Genre not found");
>   err.status = 404;
>   return next(err);
> }
> ```
>
> Luego, el mensaje se propagará a través de nuestro código de manejo de errores (esto se configuró cuando generamos el esqueleto de la aplicación; para obtener más información, consulta [Manejo de errores](https://developer.mozilla.org/en-US/docs/Learn/Server-side/Express_Nodejs/Introduction#handling_errors)).

La vista renderizada es `gender_detail` y se le pasan variables para el título, el género y la lista de libros de este género (`genre_books`).

### Vista

Crea `views/genre_detail.pug` y pega el siguiente código:

```jade
extends layout

block content

  h1 Genre: #{genre.name}

  div(style='margin-left:20px;margin-top:20px')

    h4 Books

    dl
      each book in genre_books
        dt
          a(href=book.url) #{book.title}
        dd #{book.summary}

      else
        p This genre has no books
```

### ¿Cómo se ve?

Ejecuta la aplicación y abre el  navegador en [http://localhost:3000/](http://localhost:3000/). Seleccione el enlace `All genres`, luego selecciona uno de los géneros (por ejemplo, "Fantasy"). Si todo está configurado correctamente, la página debería verse como la siguiente captura de pantalla.

![](/node-express-library-teoria/assets/img/locallibary_express_genre_detail.png)

## Página detalle del libro

La página de detalles del libro debe mostrar la información de un libro específico (identificado mediante su valor de campo `_id` generado automáticamente), junto con información sobre cada copia asociada en la biblioteca (`BookInstance`). Dondequiera que mostremos un autor, género o instancia de libro, estos deben estar vinculados a la página de detalles asociada a ese elemento.

### Controlador

Abre `/controllers/bookController.js`. Busca el método de controlador exportado `book_detail()` y reemplácelo con el siguiente código.

```javascript
// Display detail page for a specific book.
exports.book_detail = (req, res, next) => {
  async.parallel(
    {
      book(callback) {
        Book.findById(req.params.id)
          .populate("author")
          .populate("genre")
          .exec(callback);
      },
      book_instance(callback) {
        BookInstance.find({ book: req.params.id }).exec(callback);
      },
    },
    (err, results) => {
      if (err) {
        return next(err);
      }
      if (results.book == null) {
        // No results.
        const err = new Error("Book not found");
        err.status = 404;
        return next(err);
      }
      // Successful, so render.
      res.render("book_detail", {
        title: results.book.title,
        book: results.book,
        book_instances: results.book_instance,
      });
    }
  );
};
```

### Vista

Crea `/views/book_detail.pug` y pega el siguiente código:

```jade
extends layout

block content
  h1 Title: #{book.title}

  p #[strong Author:]
    a(href=book.author.url) #{book.author.name}
  p #[strong Summary:] #{book.summary}
  p #[strong ISBN:] #{book.isbn}
  p #[strong Genre:]
    each val, index in book.genre
      a(href=val.url) #{val.name}
      if index < book.genre.length - 1
        |,

  div(style='margin-left:20px;margin-top:20px')
    h4 Copies

    each val in book_instances
      hr
      if val.status=='Available'
        p.text-success #{val.status}
      else if val.status=='Maintenance'
        p.text-danger #{val.status}
      else
        p.text-warning #{val.status}
      p #[strong Imprint:] #{val.imprint}
      if val.status!='Available'
        p #[strong Due back:] #{val.due_back}
      p #[strong Id:]
        a(href=val.url) #{val._id}

    else
      p There are no copies of this book in the library.
```

> -info-La lista de géneros asociados con el libro se implementa en la plantilla como se muestra a continuación. Esto agrega una coma después de cada género asociado con el libro excepto el último.
>
> ```jade
>   p #[strong Genre:]
>     each val, index in book.genre
>       a(href=val.url) #{val.name}
>       if index < book.genre.length - 1
>         |,
> ```

### ¿Cómo se ve?

Ejecuta la aplicación y abre el navegador en [http://localhost:3000/](http://localhost:3000/). Selecciona el enlace  `All books`, luego selecciona uno de los libros. Si todo está configurado correctamente, su página debería verse como la siguiente captura de pantalla.

![Book Detail Page - Express Local Library site](/node-express-library-teoria/assets/img/locallibary_express_book_detail.png)

## Página detalle de autor

La página de detalles del autor debe mostrar la información sobre el autor especificado, identificado mediante su valor de campo `_id` (generado automáticamente), junto con una lista de todos los objetos `Book` asociados con ese autor.

### Controlador 

Abre `controllers/authorControler.js` y pega lo siguiente al principio:

```javascript
const async = require("async");
const Book = require("../models/book");
```

Encuentra el método `author_detail` y sustitúyelo por el siguiente código:

```javascript
// Display detail page for a specific Author.
exports.author_detail = (req, res, next) => {
  async.parallel(
    {
      author(callback) {
        Author.findById(req.params.id).exec(callback);
      },
      authors_books(callback) {
        Book.find({ author: req.params.id }, "title summary").exec(callback);
      },
    },
    (err, results) => {
      if (err) {
        // Error in API usage.
        return next(err);
      }
      if (results.author == null) {
        // No results.
        const err = new Error("Author not found");
        err.status = 404;
        return next(err);
      }
      // Successful, so render.
      res.render("author_detail", {
        title: "Author Detail",
        author: results.author,
        author_books: results.authors_books,
      });
    }
  );
};
```

El método usa `async.parallel()` para consultar al autor y sus instancias de libro asociadas en paralelo, y la devolución de llamada muestra la página cuando (`if`) ambas solicitudes se completan correctamente. El enfoque es exactamente el mismo que se describe en la página de detalles del género anterior.

### Vista

Crea `views/authorDetail.pug` y pega el siguiente código:

```jade
extends layout

block content

  h1 Author: #{author.name}
  p #{author.date_of_birth} - #{author.date_of_death}

  div(style='margin-left:20px;margin-top:20px')

    h4 Books

    dl
      each book in author_books
        dt
          a(href=book.url) #{book.title}
        dd #{book.summary}

      else
        p This author has no books.
```

### ¿Cómo se ve?

Ejecuta la aplicación y abre el navegador en [http://localhost:3000/](http://localhost:3000/). Selecciona el enlace  `All authors`, luego selecciona uno de los autores. Si todo está configurado correctamente, la página debería verse como la siguiente captura de pantalla.

![Author Detail Page - Express Local Library site](/node-express-library-teoria/assets/img/locallibary_express_author_detail.png)

## Página detalle BookInstance

> -reto-Realiza la página de detalle para BookInstace

![BookInstance Detail Page - Express Local Library site](/node-express-library-teoria/assets/img/locallibary_express_bookinstance_detail.png)