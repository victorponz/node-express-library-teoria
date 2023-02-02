---
typora-copy-images-to: ../assets/img/
typora-root-url: ../../
layout: post
title: 'Tutorial Express - parte 6: Trabajar con formularios'
categories: parte1
conToc: true
permalink: trabajar-con-formularios
---

## Introducción

Un [formulario HTML](https://developer.mozilla.org/en-US/docs/Learn/Forms) es un grupo de uno o más campos/widgets en una página web que se puede usar para recopilar información de los usuarios para enviarla a un servidor. Los formularios son un mecanismo flexible para recopilar información del usuario porque hay entradas de formulario adecuadas disponibles para ingresar muchos tipos diferentes de datos: cuadros de texto, casillas de verificación, botones de radio, selectores de fecha, etc. Los formularios también son una forma relativamente segura de compartir datos con el servidor. , ya que nos permiten enviar datos en solicitudes `POST` con protección contra falsificación de solicitudes entre sitios. (`CSRF` - Cross Site Request Forgery)

¡Trabajar con formularios puede ser complicado! Los desarrolladores deben escribir HTML para el formulario, validar y desinfectar adecuadamente los datos ingresados en el servidor (y posiblemente también en el navegador), volver a publicar el formulario con mensajes de error para informar a los usuarios sobre cualquier campo no válido, manejar los datos cuando se hayan enviado correctamente. y finalmente responder al usuario de alguna manera para indicar el éxito.

En este tutorial, te mostraremos cómo se pueden realizar las operaciones anteriores en Express. En el camino, ampliaremos el sitio web de LocalLibrary para permitir que los usuarios creen, editen y eliminen elementos de la biblioteca.

## Formularios HTML

Primero, una breve descripción general de los formularios HTML. Considera un formulario HTML simple, con un solo campo de texto para ingresar el nombre de algún "equipo" y su etiqueta asociada:

![HTML Form](/node-express-library-teoria/assets/img/form_example_name_field.png)

El formulario se define en HTML como una colección de elementos dentro de las etiquetas `<form>…</form>`, que contienen al menos un elemento de entrada de `type="submit"`.

```html
<form action="/team_name_url/" method="post">
  <label for="team_name">Enter name: </label>
  <input
    id="team_name"
    type="text"
    name="name_field"
    value="Default name for team." />
  <input type="submit" value="OK" />
</form>
```

Si bien aquí hemos incluido solo un campo (de texto) para ingresar el nombre del equipo, un formulario puede contener cualquier número de otros elementos de entrada y sus etiquetas asociadas. El atributo de tipo del campo define qué tipo de widget se mostrará. El nombre y la identificación del campo se utilizan para identificar el campo en JavaScript/CSS/HTML, mientras que el valor define el valor inicial del campo cuando se muestra por primera vez. La etiqueta del equipo coincidente se especifica mediante la etiqueta de `label` (consulta "Ingrese el nombre" más arriba), con un campo `for` que contiene el valor de identificación de la entrada asociada.

La entrada de envío se mostrará como un botón (de forma predeterminada); el usuario puede presionarlo para cargar los datos contenidos en los otros elementos de entrada al servidor (en este caso, solo el nombre del equipo). Los atributos del formulario definen el método HTTP utilizado para enviar los datos y el destino de los datos en el servidor (`action`):

* `action`: el recurso/URL donde se enviarán los datos para su procesamiento cuando se envíe el formulario. Si no se establece (o se establece en una cadena vacía), el formulario se enviará de nuevo a la URL de la página actual.
* `method: El método HTTP utilizado para enviar los datos: `POST` o `GET`.
  * El método `POST` siempre debe usarse si los datos van a resultar en un cambio en la base de datos del servidor, ya que esto puede hacerse más resistente a los ataques de solicitud de falsificación entre sitios.
  * El método `GET` solo debe usarse para formularios que no cambian los datos del usuario (por ejemplo, un formulario de búsqueda). Se recomienda para cuando desees poder marcar o compartir la URL.

### Proceso de manejo de formularios

El manejo de formularios utiliza las mismas técnicas que aprendimos para mostrar información sobre nuestros modelos: la ruta envía nuestra solicitud a una función de controlador que realiza las acciones de base de datos requeridas, incluida la lectura de datos de los modelos, luego genera y devuelve una página HTML. Lo que complica más las cosas es que el servidor también debe poder procesar los datos proporcionados por el usuario y volver a mostrar el formulario con información de error si hay algún problema.

A continuación se muestra un diagrama de flujo de proceso para procesar solicitudes de formularios, que comienza con una solicitud de una página que contiene un formulario (que se muestra en verde):

![](/node-express-library-teoria/assets/img/web_server_form_handling-1675182916187.png)

Como se muestra en el diagrama anterior, las cosas principales que debe hacer el código de manejo de formularios son:

1. Mostrar el formulario predeterminado la primera vez que lo solicite el usuario. 
   * El formulario puede contener campos en blanco (p. ej., si estás creando un nuevo registro), o se puede completar previamente con valores iniciales (p. ej., si estás cambiando un registro o tiene valores iniciales predeterminados útiles).
2.  Recibir datos enviados por el usuario, generalmente en una solicitud `HTTP POST`.
3.  Validar y desinfectar los datos.
4.  Si algún dato no es válido, volver a mostrar el formulario, esta vez con los valores completados por el usuario y los mensajes de error para los campos problemáticos.
5. Si todos los datos son válidos, realizar las acciones requeridas (p. ej., guardar los datos en la base de datos, enviar un correo electrónico de notificación, devolver el resultado de una búsqueda, cargar un archivo, etc. 
6. Una vez completadas todas las acciones, redirigir al usuario a otra página.

A menudo, el código de manejo de formularios se implementa utilizando una ruta `GET` para la visualización inicial del formulario y una ruta `POST` a la misma ruta para manejar la validación y el procesamiento de los datos del formulario. Este es el enfoque que se utilizará en este tutorial.

Express en sí mismo no proporciona ningún soporte específico para las operaciones de manejo de formularios, pero puedes usar middleware para procesar los parámetros `POST` y `GET` del formulario y para validar/desinfectar sus valores.

## Validación y sanitización

Antes de almacenar los datos de un formulario, se deben validar y sanitizar:

La validación verifica que los valores ingresados sean apropiados para cada campo (estén en el rango correcto, formato, etc.) y que se hayan proporcionado valores para todos los campos obligatorios.
La sanitización elimina/reemplaza caracteres en los datos que podrían usarse para enviar contenido malicioso al servidor.

Para este tutorial, usaremos el popular módulo [express-validator](https://www.npmjs.com/package/express-validator) para realizar tanto la validación como la sanitización de los datos de nuestro formulario.

### Instalación

Instala el módulo ejecutando el siguiente comando en la raíz del proyecto.

```
npm install express-validator
```

### Usando express-validator

> -info- La guía [express-validator](https://express-validator.github.io/docs/#basic-guide) en GitHub proporciona una buena descripción general de la API. Te recomiendo que leas eso para tener una idea de todas sus capacidades (incluido el uso de la [validación de esquemas](https://express-validator.github.io/docs/schema-validation.html) y la creación de https://express-validator.github.io/docs/custom-validators-sanitizers.html). A continuación, cubrimos solo un subconjunto que es útil para LocalLibrary.

Para usar el validador en nuestros controladores, especificamos las funciones particulares que queremos importar desde el módulo `express-validator`, como se muestra a continuación:

```javascript
const { body, validationResult } = require("express-validator");
```

Hay muchas funciones disponibles que te permiten verificar y sanitizar los datos de los parámetros de la solicitud, el cuerpo, los encabezados, las cookies, etc., o todos a la vez. Para este tutorial, usaremos principalmente el `body` y el `validationResult`.

Las funciones se definen como sigue:

* [`body([fields, message])`](https://express-validator.github.io/docs/check-api.html#bodyfields-message) Especifica un conjunto de campos en el cuerpo de la solicitud (un parámetro `POST`) para validar y/o sanitizar junto con un mensaje de error opcional que se puede mostrar si fallan las pruebas. Los criterios de validación y saneamiento están conectados en cadena con el método `body()`. Por ejemplo, la línea a continuación define primero que estamos revisando el campo `name` y que un error de validación generará un mensaje de error "Empty name". Luego llamamos al método de sanitización `trim()` para eliminar los espacios en blanco desde el principio y el final de la cadena, y luego `isLength()` para verificar que la cadena resultante no esté vacía. Finalmente, llamamos a `escape()` para eliminar los caracteres HTML de la variable que podrían usarse en los ataques de secuencias de comandos entre sitios de JavaScript.

  ```javascript
  [
    // …
    body("name", "Empty name").trim().isLength({ min: 1 }).escape(),
    // …
  ];
  ```

  Esta prueba verifica que el campo `age` sea una fecha válida y usa `optional()` para especificar que las cadenas nulas y vacías no fallarán en la validación.

  ```javascript
  [
    // …
    body("age", "Invalid age")
      .optional({ checkFalsy: true })
      .isISO8601()
      .toDate(),
    // …
  ];
  ```

  También puedes conectar en cadena diferentes validadores y agregar mensajes que se muestran si los validadores anteriores son verdaderos.

  ```javascript
  [
    // …
    body("name")
      .trim()
      .isLength({ min: 1 })
      .withMessage("Name empty.")
      .isAlpha()
      .withMessage("Name must be alphabet letters."),
    // …
  ];
  ```

*  [`validationResult(req)`](https://express-validator.github.io/docs/validation-result-api.html#validationresultreq) Ejecuta la validación, haciendo que los errores estén disponibles en forma de un objeto `validationResult`. Este se invoca en una devolución de llamada separada, como se muestra a continuación:

  ```javascript
  (req, res, next) => {
    // Extract the validation errors from a request.
    const errors = validationResult(req);
  
    if (!errors.isEmpty()) {
      // There are errors. Render form again with sanitized values/errors messages.
      // Error messages can be returned in an array using `errors.array()`.
    } else {
      // Data from form is valid.
    }
  };
  ```

   Usamos el método `isEmpty()` del resultado de la validación para verificar si hubo errores, y su método `array()` para obtener el conjunto de mensajes de error. Consulta la [API de validationResults](https://express-validator.github.io/docs/validation-result-api.html) para obtener más información.

Las cadenas de validación y sanitización son middleware que deben pasarse al controlador de ruta Express (lo hacemos indirectamente, a través del controlador). Cuando se ejecuta el middleware, cada validador/sanitizador se ejecuta en el orden especificado.

Cubriremos algunos ejemplos reales cuando implementemos los formularios LocalLibrary a continuación.

## Diseño de formulario

Muchos de los modelos de la biblioteca están relacionados o son dependientes; por ejemplo, un libro requiere un autor y también puede tener uno o más géneros. Esto plantea la cuestión de cómo debemos manejar el caso en el que un usuario desea:

* Crear un objeto cuando sus objetos relacionados aún no existan (por ejemplo, un libro donde el objeto de autor no se ha definido).
* Eliminar un objeto que todavía está siendo utilizado por otro objeto (por ejemplo, eliminando un género que todavía está siendo utilizado por un libro).

Para este proyecto, simplificaremos la implementación indicando que un formulario solo puede:

* Crear un objeto usando objetos que ya existen (por lo que los usuarios tendrán que crear las instancias de `author` y `genre` requeridas antes de intentar crear cualquier objeto de libro).
* Eliminar un objeto si otros objetos no hacen referencia a él (por ejemplo, no podrás eliminar un `book` hasta que se hayan eliminado todos los objetos `BookInstance` asociados).

> -info-Una implementación más "robusta" podría permitirte crear los objetos dependientes al crear un nuevo objeto y eliminar cualquier objeto en cualquier momento (por ejemplo, eliminando objetos dependientes o eliminando referencias al objeto eliminado de la base de datos).

## Rutas

Para implementar nuestro código de manejo de formularios, necesitaremos dos rutas que tengan el mismo patrón de URL. La primera ruta (`GET`) se usa para mostrar un nuevo formulario vacío para crear el objeto. La segunda ruta (`POST`) se utiliza para validar los datos ingresados por el usuario y luego guardar la información y redirigir a la página de detalles (si los datos son válidos) o volver a mostrar el formulario con errores (si los datos no son válidos).

Ya hemos creado las rutas para todas las páginas de creación de nuestro modelo en `/routes/catalog.js` (en un [tutorial anterior](/node-express-library-teoria/rutas-y-controladores)). Por ejemplo, las rutas de `genre` se muestran a continuación:

```javascript
// GET request for creating a Genre. NOTE This must come before route that displays Genre (uses id).
router.get("/genre/create", genre_controller.genre_create_get);

// POST request for creating Genre.
router.post("/genre/create", genre_controller.genre_create_post);
```

## Formulario Genre

Este subartículo muestra cómo definimos nuestra página para crear objetos `Genre` (este es un buen lugar para comenzar porque el `Genre` tiene solo un campo, su nombre y no tiene dependencias). Como cualquier otra página, necesitamos configurar rutas, controladores y vistas.

### Métodos de validación y sanitización de importaciones

Para usar express-validator en nuestros controladores, debemos solicitar las funciones que queremos usar del módulo 'express-validator'.

Abre `/controllers/genreController.js` y agrega la siguiente línea en la parte superior del archivo:

```javascript
const { body, validationResult } = require("express-validator");
```

> -info-Esta sintaxis nos permite usar el `body` y el `validationResult` como las funciones de middleware asociadas, como verá en la sección de ruta posterior a continuación. es equivalente a:
>
> ```javascript
> const validator = require("express-validator");
> const body = validator.body;
> const validationResult = validator.validationResult;
> ```



### Controlador ruta GET

En el método `genre_create_get` pega el siguiente código. Renderiza la plantilla  `genre_form.pug` pasándole `title` como parámetro:

```javascript
// Display Genre create form on GET.
exports.genre_create_get = (req, res, next) => {
  res.render("genre_form", { title: "Create Genre" });
};
```

### Controlador ruta POST

En el método `genre_create_post` pega el siguiente código. 

```javascript
// Handle Genre create on POST.
exports.genre_create_post = [
  // Validate and sanitize the name field.
  body("name", "Genre name required").trim().isLength({ min: 1 }).escape(),

  // Process request after validation and sanitization.
  (req, res, next) => {
    // Extract the validation errors from a request.
    const errors = validationResult(req);

    // Create a genre object with escaped and trimmed data.
    const genre = new Genre({ name: req.body.name });

    if (!errors.isEmpty()) {
      // There are errors. Render the form again with sanitized values/error messages.
      res.render("genre_form", {
        title: "Create Genre",
        genre,
        errors: errors.array(),
      });
      return;
    } else {
      // Data from form is valid.
      // Check if Genre with same name already exists.
      Genre.findOne({ name: req.body.name }).exec((err, found_genre) => {
        if (err) {
          return next(err);
        }

        if (found_genre) {
          // Genre exists, redirect to its detail page.
          res.redirect(found_genre.url);
        } else {
          genre.save((err) => {
            if (err) {
              return next(err);
            }
            // Genre saved. Redirect to genre detail page.
            res.redirect(genre.url);
          });
        }
      });
    }
  },
];
```

Lo primero que hay que tener en cuenta es que, en lugar de ser una única función de middleware (con argumentos `(req, res, next)`), el controlador especifica un array de funciones de middleware. La matriz se pasa a la función del enrutador y cada método se llama en orden.

> -warning-Este enfoque es necesario porque los validadores son funciones de middleware.

El primer método del array define un validador de cuerpo (`body()`) que valida y sanitiza el campo. Usa `trim()` para eliminar cualquier espacio en blanco al final/adelante, verifica que el campo de nombre no esté vacío y luego usa `escape()` para eliminar cualquier carácter HTML peligroso).

```javascript
[
  // Validate that the name field is not empty.
  body("name", "Genre name required").trim().isLength({ min: 1 }).escape(),
  // …
];
```

Después de especificar los validadores, creamos una función de middleware para extraer cualquier error de validación. Usamos `isEmpty()` para verificar si hay algún error en el resultado de la validación. Si los hay, renderizamos el formulario nuevamente, pasando nuestro objeto `Genre` sanitizado y la matriz de mensajes de error (`errors.array()`).

```javascript
// Process request after validation and sanitization.
(req, res, next) => {
  // Extract the validation errors from a request.
  const errors = validationResult(req);

  // Create a genre object with escaped and trimmed data.
  const genre = new Genre({ name: req.body.name });

  if (!errors.isEmpty()) {
    // There are errors. Render the form again with sanitized values/error messages.
    res.render("genre_form", {
      title: "Create Genre",
      genre,
      errors: errors.array(),
    });
    return;
  } else {
    // Form data is valid.
    // Save the result.
    // …
  }
};
```

Si los datos del nombre del género son válidos, verificamos si ya existe un género con el mismo nombre (ya que no queremos crear duplicados). Si es así, lo redirigimos a la página de detalles del género existente. Si no, guardamos el nuevo Género y redirigimos a su página de detalles.

```javascript
// Check if Genre with same name already exists.
Genre.findOne({ name: req.body.name }).exec((err, found_genre) => {
  if (err) {
    return next(err);
  }
  if (found_genre) {
    // Genre exists, redirect to its detail page.
    res.redirect(found_genre.url);
  } else {
    genre.save((err) => {
      if (err) {
        return next(err);
      }
      // Genre saved. Redirect to genre detail page.
      res.redirect(genre.url);
    });
  }
});
```

Este mismo patrón se usa en todos nuestros controladores de correos: ejecutamos validadores (con sanitizadores), luego verificamos si hay errores y volvemos a presentar el formulario con información de error o guardamos los datos.

### Vista

La misma vista se representa en los controladores/rutas `GET` y `POST` cuando creamos un nuevo género (y más adelante también se usa cuando actualizamos un género). En el caso `GET`, el formulario está vacío y solo pasamos una variable `title`. En el caso de `POST`, si el usuario ha introducido previamente datos no válidos; en la variable de género, devolvemos una versión limpia de los datos introducidos y en la variable de errores, devolvemos una serie de mensajes de error.

```javascript
res.render("genre_form", { title: "Create Genre" });
res.render("genre_form", {
  title: "Create Genre",
  genre,
  errors: errors.array(),
});
```

Crea `/views/genre_form.pug` y copia el texto a continuación.

```jade
extends layout

block content
  h1 #{title}

  form(method='POST' action='')
    div.form-group
      label(for='name') Genre:
      input#name.form-control(type='text', placeholder='Fantasy, Poetry etc.' name='name' value=(undefined===genre ? '' : genre.name))
    button.btn.btn-primary(type='submit') Submit

  if errors
   ul
    for error in errors
     li!= error.msg
```

Gran parte de esta plantilla te resultará familiar de nuestros tutoriales anteriores. Primero, extendemos la plantilla base `layout.pug` y anulamos el bloque llamado `content`. Luego tenemos un encabezado con el `title` que pasamos desde el controlador (a través del método `render()`).

A continuación, tenemos el código `pug` para nuestro formulario HTML que usa `method="POST"` para enviar los datos al servidor y, dado que la acción es una cadena vacía, enviará los datos a la misma URL que la página.

El formulario define un solo campo obligatorio de tipo `text` llamado `name`. El valor predeterminado del campo depende de si la variable de género está definida. Si se llama desde la ruta `GET`, estará vacío ya que se trata de un formulario nuevo. Si se llama desde una ruta `POST`, contendrá el valor (no válido) ingresado originalmente por el usuario.

La última parte de la página es el código de error. Esto imprime una lista de errores, si se ha definido la variable de error (en otras palabras, esta sección no aparecerá cuando la plantilla se represente en la ruta GET).

> -info-Esta es solo una forma de representar los errores. También puedea obtener los nombres de los campos afectados de la variable de error y utilizarlos para controlar dónde se representan los mensajes de error, si aplicar CSS personalizado, etc.

### ¿Cómo se ve?

Ejecuta la aplicación, abre el navegador en http://localhost:3000/, luego selecciona el enlace `Create new Genre`. Si todo está configurado correctamente, tu sitio debería parecerse a la siguiente captura de pantalla. Después de ingresar un valor, debe guardarse y accederás a la página de detalles del género.

![Genre Create Page - Express Local Library site](/node-express-library-teoria/assets/img/locallibary_express_genre_create_empty.png)

El único error que validamos contra el lado del servidor es que el campo de género no debe estar vacío. La siguiente captura de pantalla muestra cómo se vería la lista de errores si no proporcionaras un género (resaltado en rojo).

![](/node-express-library-teoria/assets/img/locallibary_express_genre_create_error.png)

## Formulario Author

Al igual que con el formulario de `Genre`, para usar express-validator debemos requerir las funciones que queremos usar.

Abre `/controllers/authorController.js` y agrega las siguientes líneas en la parte superior del archivo:

### Controlador ruta GET

En el método `author_create_get` pega el siguiente código. Renderiza la plantilla  `author_form.pug` pasándole `title` como parámetro:

```javascript
// Display Author create form on GET.
exports.author_create_get = (req, res, next) => {
  res.render("author_form", { title: "Create Author" });
};
```

### Controlador ruta POST

En el método `author_create_post` pega el siguiente código. 

```javascript
// Handle Author create on POST.
exports.author_create_post = [
  // Validate and sanitize fields.
  body("first_name")
    .trim()
    .isLength({ min: 1 })
    .escape()
    .withMessage("First name must be specified.")
    .isAlphanumeric()
    .withMessage("First name has non-alphanumeric characters."),
  body("family_name")
    .trim()
    .isLength({ min: 1 })
    .escape()
    .withMessage("Family name must be specified.")
    .isAlphanumeric()
    .withMessage("Family name has non-alphanumeric characters."),
  body("date_of_birth", "Invalid date of birth")
    .optional({ checkFalsy: true })
    .isISO8601()
    .toDate(),
  body("date_of_death", "Invalid date of death")
    .optional({ checkFalsy: true })
    .isISO8601()
    .toDate(),
  // Process request after validation and sanitization.
  (req, res, next) => {
    // Extract the validation errors from a request.
    const errors = validationResult(req);

    if (!errors.isEmpty()) {
      // There are errors. Render form again with sanitized values/errors messages.
      res.render("author_form", {
        title: "Create Author",
        author: req.body,
        errors: errors.array(),
      });
      return;
    }
    // Data from form is valid.

    // Create an Author object with escaped and trimmed data.
    const author = new Author({
      first_name: req.body.first_name,
      family_name: req.body.family_name,
      date_of_birth: req.body.date_of_birth,
      date_of_death: req.body.date_of_death,
    });
    author.save((err) => {
      if (err) {
        return next(err);
      }
      // Successful - redirect to new author record.
      res.redirect(author.url);
    });
  },
];
```

> -alert-Nunca valides nombres usando `isAlphanumeric()` (como hemos hecho anteriormente) ya que hay muchos nombres que usan otros conjuntos de caracteres. Lo hacemos aquí para demostrar cómo se usa el validador y cómo se puede conectar en cadena con otros validadores e informes de errores.

### Vista

Crea `/views/author_form.pug` y copia el texto a continuación.

```jade
extends layout

block content
  h1=title

  form(method='POST' action='')
    div.form-group
      label(for='first_name') First Name:
      input#first_name.form-control(type='text' placeholder='First name' name='first_name' required='true' value=(undefined===author ? '' : author.first_name) )
      label(for='family_name') Family Name:
      input#family_name.form-control(type='text' placeholder='Family name' name='family_name' required='true' value=(undefined===author ? '' : author.family_name))
    div.form-group
      label(for='date_of_birth') Date of birth:
      input#date_of_birth.form-control(type='date' name='date_of_birth' value=(undefined===author ? '' : author.date_of_birth) )
    button.btn.btn-primary(type='submit') Submit
  if errors
    ul
      for error in errors
        li!= error.msg
```

> -alert-Algunos navegadores no admiten el tipo de entrada `= "date"`, por lo que no obtendrás el widget selector de fecha o el marcador de posición predeterminado `dd/mm/yyyy`, sino que obtendrás un campo de texto sin formato vacío. Una solución es agregar explícitamente el placeholder `= 'dd/mm/yyyy'` para que en los navegadores menos capaces aún obtengaa información sobre el formato de texto deseado.

> -reto-A la plantilla anterior le falta un campo para ingresar `date-of_death` Crea el campo siguiendo el mismo patrón que el grupo de formulario de fecha de nacimiento!

### ¿Cómo se ve?

Ejecuta la aplicación, abra su navegador en [http://localhost:3000/](http://localhost:3000/), luego selecciona el enlace `Create new author`. Si todo está configurado correctamente, tu sitio debería parecerse a la siguiente captura de pantalla. Después de ingresar un valor, debe guardarse y accederás a la página de detalles del autor.

![](/node-express-library-teoria/assets/img/locallibary_express_author_create_empty.png)

## Formulario Book

Al igual que con los otros formularios, para usar express-validator debemos requerir las funciones que queremos usar.

Abre `/controllers/bookController.js` y agrega las siguientes líneas en la parte superior del archivo:

```javascript
const { body, validationResult } = require("express-validator");
```

### Controlador ruta GET

En el método `book_create_get` pega el siguiente código. 

```javascript
// Display book create form on GET.
exports.book_create_get = (req, res, next) => {
  // Get all authors and genres, which we can use for adding to our book.
  async.parallel(
    {
      authors(callback) {
        Author.find(callback);
      },
      genres(callback) {
        Genre.find(callback);
      },
    },
    (err, results) => {
      if (err) {
        return next(err);
      }
      res.render("book_form", {
        title: "Create Book",
        authors: results.authors,
        genres: results.genres,
      });
    }
  );
};
```

Esto usa el módulo `async` (descrito en el [Tutorial Express Parte 5: Visualización de datos](/node-express-library-teoria/visualizacion-de-datos)) para obtener todos los objetos `Author` y `Genre`. Estos luego se pasan a la vista `book_form.pug` como variables denominadas `authors` y `genres` (junto con el título de la página).

### Controlador ruta POST

Localiza el método `book_create_post` y pega este código:

```javascript
// Handle book create on POST.
exports.book_create_post = [
  // Convert the genre to an array.
  (req, res, next) => {
    if (!Array.isArray(req.body.genre)) {
      req.body.genre =
        typeof req.body.genre === "undefined" ? [] : [req.body.genre];
    }
    next();
  },

  // Validate and sanitize fields.
  body("title", "Title must not be empty.")
    .trim()
    .isLength({ min: 1 })
    .escape(),
  body("author", "Author must not be empty.")
    .trim()
    .isLength({ min: 1 })
    .escape(),
  body("summary", "Summary must not be empty.")
    .trim()
    .isLength({ min: 1 })
    .escape(),
  body("isbn", "ISBN must not be empty").trim().isLength({ min: 1 }).escape(),
  body("genre.*").escape(),

  // Process request after validation and sanitization.
  (req, res, next) => {
    // Extract the validation errors from a request.
    const errors = validationResult(req);

    // Create a Book object with escaped and trimmed data.
    const book = new Book({
      title: req.body.title,
      author: req.body.author,
      summary: req.body.summary,
      isbn: req.body.isbn,
      genre: req.body.genre,
    });

    if (!errors.isEmpty()) {
      // There are errors. Render form again with sanitized values/error messages.

      // Get all authors and genres for form.
      async.parallel(
        {
          authors(callback) {
            Author.find(callback);
          },
          genres(callback) {
            Genre.find(callback);
          },
        },
        (err, results) => {
          if (err) {
            return next(err);
          }

          // Mark our selected genres as checked.
          for (const genre of results.genres) {
            if (book.genre.includes(genre._id)) {
              genre.checked = "true";
            }
          }
          res.render("book_form", {
            title: "Create Book",
            authors: results.authors,
            genres: results.genres,
            book,
            errors: errors.array(),
          });
        }
      );
      return;
    }

    // Data from form is valid. Save book.
    book.save((err) => {
      if (err) {
        return next(err);
      }
      // Successful: redirect to new book record.
      res.redirect(book.url);
    });
  },
];
```

La estructura y el comportamiento de este código es casi exactamente el mismo que para crear un objeto Género o Autor. Primero validamos y desinfectamos los datos. Si los datos no son válidos, volvemos a mostrar el formulario junto con los datos ingresados originalmente por el usuario y una lista de mensajes de error. Si los datos son válidos, guardamos el nuevo registro del libro y redirigimos al usuario a la página de detalles del libro.

La principal diferencia con respecto al otro código de manejo de formularios es cómo desinfectamos la información del género. El formulario devuelve una matriz de elementos de Género (mientras que para otros campos devuelve una cadena). Para validar la información, primero convertimos la solicitud en una matriz (requerido para el siguiente paso).

```javascript
[
  // Convert the genre to an array.
  (req, res, next) => {
    if (!Array.isArray(req.body.genre)) {
      req.body.genre =
        typeof req.body.genre === "undefined" ? [] : [req.body.genre];
    }
    next();
  },
  // …
];
```

Luego usamos un comodín (`*`) en el desinfectante para validar individualmente cada una de las entradas de la matriz de género. El siguiente código muestra cómo: esto se traduce como "desinfectar todos los elementos debajo del con el `key` `genre`".

```javascript
[
  // …
  body("genre.*").escape(),
  // …
];
```

La última diferencia con respecto al otro código de manejo de formularios es que necesitamos pasar todos los géneros y autores existentes al formulario. Para marcar los géneros que el usuario verificó, iteramos a través de todos los géneros y agregamos el parámetro `checked='true'` a los que estaban en nuestros datos del POST (como se reproduce en el fragmento de código a continuación).

```javascript
// Mark our selected genres as checked.
for (const genre of results.genres) {
  if (book.genre.includes(genre._id)) {
    // Current genre is selected. Set "checked" flag.
    genre.checked = "true";
  }
}
```

### Vista

Crea el archivo `book_form.pug` y pega el siguiente código:

```jade
extends layout

block content
  h1= title

  form(method='POST' action='')
    div.form-group
      label(for='title') Title:
      input#title.form-control(type='text', placeholder='Name of book' name='title' required='true' value=(undefined===book ? '' : book.title) )
    div.form-group
      label(for='author') Author:
      select#author.form-control(type='select', placeholder='Select author' name='author' required='true' )
        - authors.sort(function(a, b) {let textA = a.family_name.toUpperCase(); let textB = b.family_name.toUpperCase(); return (textA < textB) ? -1 : (textA > textB) ? 1 : 0;});
        for author in authors
          if book
            option(value=author._id selected=(author._id.toString()===book.author._id.toString() ? 'selected' : false) ) #{author.name}
          else
            option(value=author._id) #{author.name}
    div.form-group
      label(for='summary') Summary:
      textarea#summary.form-control(type='textarea', placeholder='Summary' name='summary' required='true') #{undefined===book ? '' : book.summary}
    div.form-group
      label(for='isbn') ISBN:
      input#isbn.form-control(type='text', placeholder='ISBN13' name='isbn' value=(undefined===book ? '' : book.isbn) required='true')
    div.form-group
      label Genre:
      div
        for genre in genres
          div(style='display: inline; padding-right:10px;')
            input.checkbox-input(type='checkbox', name='genre', id=genre._id, value=genre._id, checked=genre.checked )
            label(for=genre._id) #{genre.name}
    button.btn.btn-primary(type='submit') Submit

  if errors
    ul
      for error in errors
        li!= error.msg
```

La estructura y el comportamiento de la vista son casi los mismos que para la plantilla `gender_form.pug`.

Las principales diferencias están en cómo implementamos los campos de tipo selección: `Author` y `Genre`.

* El conjunto de géneros se muestra como casillas de verificación, utilizando el valor marcado que configuramos en el controlador para determinar si la casilla debe seleccionarse o no.
* El conjunto de autores se muestra como una lista desplegable ordenada alfabéticamente de una sola selección. Si el usuario ha seleccionado previamente un autor de libro (es decir, al corregir valores de campo no válidos después del envío del formulario inicial o al actualizar los detalles del libro), el autor se volverá a seleccionar cuando se muestre el formulario. Aquí determinamos qué autor seleccionar comparando la identificación de la opción de autor actual con el valor ingresado previamente por el usuario (pasado a través de la variable `book`).

> -alert-Si hay un error en el formulario enviado, entonces, cuando se vuelva a procesar el formulario, la identificación del autor del nuevo libro y las identificaciones de los autores de los libros existentes son del tipo `Schema.Types.ObjectId`. Entonces, para compararlos, primero debemos convertirlos en cadenas.

### ¿Cómo se ve?

Ejecuta la aplicación, abre tu navegador en [http://localhost:3000/](http://localhost:3000/), luego selecciona el enlace `Create new book`. Si todo está configurado correctamente, tu sitio debería parecerse a la siguiente captura de pantalla. Después de enviar un libro válido, debe guardarse y accederás a la página de detalles del libro.

![](/node-express-library-teoria/assets/img/locallibary_express_book_create_empty.png)

## Formulario BookInstance

Este subartículo muestra cómo definir una página/formulario para crear objetos `BookInstance`. Esto es muy parecido a la forma que usamos para crear objetos `Libro`.

### Métodos de validación y sanitización de importaciones

Abre `/controllers/bookinstanceController.js` y agrega las siguientes líneas en la parte superior del archivo:

```javascript
const { body, validationResult } = require("express-validator");
```

### Controlador ruta GET

En la parte superior del archivo, importa el modelo `Book` (necesario porque cada `BookInstance` está asociado con un libro en particular).

```javascript
const Book = require("../models/book");
```

Busca el método de controlador exportado `bookinstance_create_get()` y reemplázalo con el siguiente código.

```javascript
// Display BookInstance create form on GET.
exports.bookinstance_create_get = (req, res, next) => {
  Book.find({}, "title").exec((err, books) => {
    if (err) {
      return next(err);
    }
    // Successful, so render.
    res.render("bookinstance_form", {
      title: "Create BookInstance",
      book_list: books,
    });
  });
};
```

El controlador obtiene una lista de todos los libros (`book_list`) y la pasa a la vista `bookinstance_form.pug` (junto con el título)

### Controlador ruta POST

> -reto-Encuentra el método de controlador exportado `bookinstance_create_post()` y crea al código necesario para guardar una Instancia

### Vista

Crea `/views/bookinstance_form.pug` y copie el texto a continuación.

```jade
extends layout

block content
  h1=title

  form(method='POST' action='')
    div.form-group
      label(for='book') Book:
      select#book.form-control(type='select' placeholder='Select book' name='book' required='true')
        - book_list.sort(function(a, b) {let textA = a.title.toUpperCase(); let textB = b.title.toUpperCase(); return (textA < textB) ? -1 : (textA > textB) ? 1 : 0;});
        for book in book_list
          option(value=book._id, selected=(selected_book==book._id.toString() ? 'selected' : false) ) #{book.title}

    div.form-group
      label(for='imprint') Imprint:
      input#imprint.form-control(type='text' placeholder='Publisher and date information' name='imprint' required='true' value=(undefined===bookinstance ? '' : bookinstance.imprint))
    div.form-group
      label(for='due_back') Date when book available:
      input#due_back.form-control(type='date' name='due_back' value=(undefined===bookinstance ? '' : bookinstance.due_back))

    div.form-group
      label(for='status') Status:
      select#status.form-control(type='select' placeholder='Select status' name='status' required='true')
        option(value='Maintenance') Maintenance
        option(value='Available') Available
        option(value='Loaned') Loaned
        option(value='Reserved') Reserved

    button.btn.btn-primary(type='submit') Submit

  if errors
    ul
      for error in errors
        li!= error.msg
```

### ¿Cómo se ve?

Ejecuta la aplicación y abre el navegador en [http://localhost:3000/](http://localhost:3000/). A continuación, selecciona el enlace `Create new instance (copy`) Si todo está configurado correctamente, tu sitio debería parecerse a la siguiente captura de pantalla. Después de enviar una `BookInstance` válida, debe guardarse y accederás a la página de detalles.

![](/node-express-library-teoria/assets/img/locallibary_express_bookinstance_create_empty.png)

## Formulario para borrar Author

Como se discutió en la sección de diseño de formularios, nuestra estrategia será permitir solo la eliminación de objetos a los que no hacen referencia otros objetos (en este caso, eso significa que no permitiremos que se elimine un Autor si está referenciado por un Libro). En términos de implementación, esto significa que el formulario debe confirmar que no hay libros asociados antes de que se elimine el autor. Si hay libros asociados, debe mostrarlos e indicar que deben eliminarse antes de que se pueda eliminar el objeto Autor.

### Controlador ruta GET

Abre /`controllers/authorController.js`. Busca el método de controlador exportado `author_delete_get()` y reemplázalo con el siguiente código.

```javascript
// Display Author delete form on GET.
exports.author_delete_get = (req, res, next) => {
  async.parallel(
    {
      author(callback) {
        Author.findById(req.params.id).exec(callback);
      },
      authors_books(callback) {
        Book.find({ author: req.params.id }).exec(callback);
      },
    },
    (err, results) => {
      if (err) {
        return next(err);
      }
      if (results.author == null) {
        // No results.
        res.redirect("/catalog/authors");
      }
      // Successful, so render.
      res.render("author_delete", {
        title: "Delete Author",
        author: results.author,
        author_books: results.authors_books,
      });
    }
  );
};
```

El controlador obtiene la identificación de la instancia de autor que se eliminará del parámetro de URL (`req.params.id`). Utiliza el método `async.parallel()` para obtener el registro del autor y todos los libros asociados en paralelo. Cuando se han completado ambas operaciones, presenta la vista `author_delete.pug`, pasando variables para el `title`, `author` y `author_books`.

> -info-Si `findById()` no devuelve resultados, el autor no está en la base de datos. En este caso, no hay nada que eliminar, por lo que mostramos inmediatamente la lista de todos los autores.
>
> ```javascript
>   (err, results) => {
>     if (err) {
>       return next(err);
>     }
>     if (results.author == null) { // No results.
>        res.redirect('/catalog/authors');
>     }
> ```

### Controlador ruta POST

Busca el método de controlador exportado `author_delete_post()` y reemplázalo con el siguiente código.

```javascript
// Handle Author delete on POST.
exports.author_delete_post = (req, res, next) => {
  async.parallel(
    {
      author(callback) {
        Author.findById(req.body.authorid).exec(callback);
      },
      authors_books(callback) {
        Book.find({ author: req.body.authorid }).exec(callback);
      },
    },
    (err, results) => {
      if (err) {
        return next(err);
      }
      // Success
      if (results.authors_books.length > 0) {
        // Author has books. Render in same way as for GET route.
        res.render("author_delete", {
          title: "Delete Author",
          author: results.author,
          author_books: results.authors_books,
        });
        return;
      }
      // Author has no books. Delete object and redirect to the list of authors.
      Author.findByIdAndRemove(req.body.authorid, (err) => {
        if (err) {
          return next(err);
        }
        // Success - go to author list
        res.redirect("/catalog/authors");
      });
    }
  );
};
```

Primero validamos que se haya proporcionado una identificación (esto se envía a través de los parámetros del cuerpo del formulario, en lugar de usar la versión en la URL). Luego obtenemos el autor y sus libros asociados de la misma manera que para la ruta GET. Si no hay libros, eliminamos el objeto de autor y redirigimos a la lista de todos los autores. Si todavía hay libros, simplemente volvemos a presentar el formulario, pasando el autor y la lista de libros que se eliminarán.

### Vista

Crea `/views/author_delete.pug` y copia el texto a continuación.

```jade
extends layout

block content
  h1 #{title}: #{author.name}
  p= author.lifespan

  if author_books.length

    p #[strong Delete the following books before attempting to delete this author.]

    div(style='margin-left:20px;margin-top:20px')

      h4 Books

      dl
      each book in author_books
        dt
          a(href=book.url) #{book.title}
        dd #{book.summary}

  else
    p Do you really want to delete this Author?

    form(method='POST' action='')
      div.form-group
        input#authorid.form-control(type='hidden',name='authorid', required='true', value=author._id )

      button.btn.btn-primary(type='submit') Delete
```

La vista extiende la plantilla `layout`, anulando el bloque denominado `content`. En la parte superior muestra los detalles del autor. Luego incluye una declaración condicional basada en el número de `author_books` (las cláusulas `if` y `else`).

* Si hay libros asociados con el autor, la página enumera los libros y establece que deben eliminarse antes de que se pueda eliminar este autor.
*  Si no hay libros, la página muestra un mensaje de confirmación.
*  Si se hace clic en el botón `Delete`, la identificación del autor se envía al servidor en una solicitud POST y se eliminará el registro de ese autor.

### Control para borrar autores

A continuación, agregaremos un control `Delete` a la vista de detalles del autor (la página de detalles es un buen lugar para eliminar un registro).

Abre la vista `author_detail.pug` y agrega las siguientes líneas en la parte inferior.

```jade
hr
p
  a(href=author.url+'/delete') Delete author
```

## Formulario para actualizar el libro

El manejo de formularios al actualizar un libro es muy similar al de la creación de un libro, excepto que debes completar el formulario en la ruta GET con valores de la base de datos.

### Controlador ruta GET

Abre `/controllers/bookController.js`. Busca el método de controlador exportado `book_update_get()` y reemplázalo con el siguiente código.

```javascript
// Display book update form on GET.
exports.book_update_get = (req, res, next) => {
  // Get book, authors and genres for form.
  async.parallel(
    {
      book(callback) {
        Book.findById(req.params.id)
          .populate("author")
          .populate("genre")
          .exec(callback);
      },
      authors(callback) {
        Author.find(callback);
      },
      genres(callback) {
        Genre.find(callback);
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
      // Success.
      // Mark our selected genres as checked.
      for (const genre of results.genres) {
        for (const bookGenre of results.book.genre) {
          if (genre._id.toString() === bookGenre._id.toString()) {
            genre.checked = "true";
          }
        }
      }
      res.render("book_form", {
        title: "Update Book",
        authors: results.authors,
        genres: results.genres,
        book: results.book,
      });
    }
  );
};
```

El controlador obtiene la identificación del libro que se actualizará desde el parámetro de URL (`req.params.id`). Utiliza el método `async.parallel()` para obtener el registro del Libro especificado (rellenando sus campos de género y autor) y listas de todos los objetos `Autor` y `Género`.

Cuando se completan las operaciones, comprueba si hay errores en la operación de búsqueda y también si se encontraron libros.

Luego marcamos los géneros actualmente seleccionados como marcados y luego representamos la vista `book_form.pug`, pasando variables para título, libro, todos los autores y todos los géneros.

### Controlador ruta POST

Busca el método de controlador exportado `book_update_post()` y reemplázalo con el siguiente código:

```javascript
// Handle book update on POST.
exports.book_update_post = [
  // Convert the genre to an array
  (req, res, next) => {
    if (!Array.isArray(req.body.genre)) {
      req.body.genre =
        typeof req.body.genre === "undefined" ? [] : [req.body.genre];
    }
    next();
  },

  // Validate and sanitize fields.
  body("title", "Title must not be empty.")
    .trim()
    .isLength({ min: 1 })
    .escape(),
  body("author", "Author must not be empty.")
    .trim()
    .isLength({ min: 1 })
    .escape(),
  body("summary", "Summary must not be empty.")
    .trim()
    .isLength({ min: 1 })
    .escape(),
  body("isbn", "ISBN must not be empty").trim().isLength({ min: 1 }).escape(),
  body("genre.*").escape(),

  // Process request after validation and sanitization.
  (req, res, next) => {
    // Extract the validation errors from a request.
    const errors = validationResult(req);

    // Create a Book object with escaped/trimmed data and old id.
    const book = new Book({
      title: req.body.title,
      author: req.body.author,
      summary: req.body.summary,
      isbn: req.body.isbn,
      genre: typeof req.body.genre === "undefined" ? [] : req.body.genre,
      _id: req.params.id, //This is required, or a new ID will be assigned!
    });

    if (!errors.isEmpty()) {
      // There are errors. Render form again with sanitized values/error messages.

      // Get all authors and genres for form.
      async.parallel(
        {
          authors(callback) {
            Author.find(callback);
          },
          genres(callback) {
            Genre.find(callback);
          },
        },
        (err, results) => {
          if (err) {
            return next(err);
          }

          // Mark our selected genres as checked.
          for (const genre of results.genres) {
            if (book.genre.includes(genre._id)) {
              genre.checked = "true";
            }
          }
          res.render("book_form", {
            title: "Update Book",
            authors: results.authors,
            genres: results.genres,
            book,
            errors: errors.array(),
          });
        }
      );
      return;
    }

    // Data from form is valid. Update the record.
    Book.findByIdAndUpdate(req.params.id, book, {}, (err, thebook) => {
      if (err) {
        return next(err);
      }

      // Successful: redirect to book detail page.
      res.redirect(thebook.url);
    });
  },
];
```

Esto es muy similar a la ruta `POST` utilizada al crear un libro. Primero validamos y desinfectamos los datos del libro del formulario y los usamos para crear un nuevo objeto Libro (estableciendo su valor `_id` en el id del objeto para actualizar). Si hay errores cuando validamos los datos, volvemos a renderizar el formulario, mostrando además los datos ingresados por el usuario, los errores y las listas de géneros y autores. Si no hay errores, llamamos a `Book.findByIdAndUpdate()` para actualizar el documento del libro y luego redirigir a su página de detalles.

### Vista

No es necesario cambiar la vista del formulario (`/views/book_form.pug`) ya que el mismo código funciona tanto para crear como para actualizar el libro.

### Agregar un botón de actualización

Abre la vista `book_detail.pug` y asegúrate de que haya enlaces para eliminar y actualizar libros en la parte inferior de la página, como se muestra a continuación.

```jade
  hr
  p
    a(href=book.url+'/delete') Delete Book
  p
    a(href=book.url+'/update') Update Book
```

### ¿Cómo se ve?

Ejecuta la aplicación, abre tu navegador en [http://localhost:3000/](http://localhost:3000/), luego selecciona el enlace `All books`. Si todo está configurado correctamente, tu sitio debería parecerse a la siguiente captura de pantalla.

![](/node-express-library-teoria/assets/img/locallibary_express_book_update_noerrors.png)

## Reto

> -info-Crea el resto de páginas para borrar y actualizar objetos

