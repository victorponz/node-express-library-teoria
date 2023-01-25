---
typora-copy-images-to: ../assets/img/
typora-root-url: ../../
layout: post
title: 'Tutorial Express - parte 2: Esqueleto de la aplicación'
categories: parte1
conToc: true
permalink: esqueleto-de-la-aplicacion
---

Este segundo artículo de nuestro [Tutorial Express](/node-express-library-teoria/) muestra cómo puede crear un "esqueleto" para un proyecto de sitio web que luego puede completar con rutas, plantillas/vistas, y llamadas a base de datos especifícas del sitio.

<table>
  <tbody>
    <tr>
      <th scope="row">Prerequisitos:</th>
      <td>
        <a
          href="/en-US/docs/Learn/Server-side/Express_Nodejs/development_environment"
          >Configurar un entorno de desarrollo de Node</a
        >. Revise el Tutorial Express.
      </td>
    </tr>
    <tr>
      <th scope="row">Objetivo:</th>
      <td>
        Poder empezar sus nuevos proyectos web usando el
        <em>Generador de Aplicaciones Express</em>.
      </td>
    </tr>
  </tbody>
</table>

## Visión General

Este artículo muestra cómo puedes crear un sitio web "esqueleto" usando la herramienta [Generador de Aplicaciones Express](https://expressjs.com/en/starter/generator.html), que luego puedes completar con rutas, vistas/plantillas, y llamadas a base de datos específicas del sitio. En este caso usaremos la herramienta para crear el framework para nuestro sitio web, al que luego agregaremos todo el código que el sitio necesite. El proceso es extremadamente simple, requiriendo sólo que se invoque el generador en la línea de comandos con un nombre para el nuevo proyecto, opcionalmente especificando también el motor de plantillas y el generador de CSS a utilizar.

Las siguientes secciones muestran como puedes llamar al generador de aplicaciones, y proporcionan una pequeña explicación sobre las diferentes opciones para vistas y CSS. También explicaremos como está estructurado el esqueleto del sitio web. Al final, mostraremos cómo puedes ejecutar el sitio web para verificar que funciona.

> -info- El _Generador de Aplicaciones Express_ no es el único generador para aplicaciones Express, y el proyecto generado no es la única forma viable para estructurar sus archivos y directorios. El sitio generado, sin embargo, tiene una estructura modular que es fácil de extender y comprender. Para informacion sobre una _mínima_ aplicación Express, vea el [Ejemplo Hello world](https://expressjs.com/en/starter/hello-world.html).

## Usando el generador de aplicaciones

Ya debe haber instalado el generador como parte de [Configurar un entorno de desarrollo de Node](node-express-library-teoria/entorno-desarrollo-node-express). Como un rápido recordatorio, la herramienta generador se instala para todos los sitios usando el manejador de paquetes `npm`, como se muestra:

```bash
npm install express-generator -g
```

El generador tiene un número de opciones, las cuales puede observar en la línea de comandos usando el comando `--help` (o bien `-h`):

```bash
> express --help

  Usage: express [options] [dir]

  Options:

        --version        output the version number
    -e, --ejs            add ejs engine support
        --pug            add pug engine support
        --hbs            add handlebars engine support
    -H, --hogan          add hogan.js engine support
    -v, --view <engine>  add view <engine> support (dust|ejs|hbs|hjs|jade|pug|twig|vash) (defaults to jade)
        --no-view        use static html instead of view engine
    -c, --css <engine>   add stylesheet <engine> support (less|stylus|compass|sass) (defaults to plain css)
        --git            add .gitignore
    -f, --force          force on non-empty directory
    -h, --help           output usage information
```

Simplemente puedes especificar `express` para crear un proyecto dentro del directorio actual usando el motor de plantillas _Jade_ y CSS plano (si especifica un nombre de directorio entonces el proyecto será creado en un subdirectorio con ese nombre).

```bash
express
```

También puedes seleccionar el motor de plantillas para las vistas usando `--view` y/o un motor generador de CSS usando `--css`.

> -info- Las otras opciones para elegir motores de plantillas (e.g. `--hogan`, `--ejs`, `--hbs` etc.) están descontinuadas. Use `--view` (o bien `-v`)!

### ¿Qué motor de vistas debo usar?

El _Generador de Aplicaciones Express_ le permite configurar un número de populares motores de plantillas, incluyendo [EJS](https://www.npmjs.com/package/ejs), [Hbs](http://github.com/donpark/hbs), [Pug](https://pugjs.org/api/getting-started.html) (Jade), [Twig](https://www.npmjs.com/package/twig), y [Vash](https://www.npmjs.com/package/vash), aunque si no se especifica una opcion de vista, selecciona Jade por defecto. Express puede soportar un gran número de motores de plantillas [aquí una lista](https://github.com/expressjs/express/wiki#template-engines).

> -info- Si quieres usar un motor de plantillas que no es soportado por el generador entonces échale un vistazo al artículo [Usando motores de plantillas con Express](https://expressjs.com/en/guide/using-template-engines.html) (Express docs) y la documentación de su motor de plantillas.

Generalmente hablando debes seleccionar un motor de plantillas que le brinde toda la funcionalidad que necesite y le permita ser productivo rápidamente — o en otras palabras, en la misma forma en que selecciona cualquier otro componente. Alguna de las cosas a considerar cuando se comparan motores de plantillas:

- Tiempo de productividad — Si tu equipo ya tiene experiencia con un lenguaje de plantillas entonces es probable que sean más productivos usando ese lenguaje. Si no, debería considerar la curva de aprendizaje relativa del motor de plantillas candidato.
- Popularidad y actividad — Revise la popularidad del motor y si tiene una comunidad activa. Es importante obtener soporte para el motor cuando tenga problemas durante la vida útil del sitio web.
- Estilo — Algunos motores de plantillas usan marcas específicas para indicar inserción de contenido dentro del HTML "ordinario", mientras que otros construyen el HTML usando una sintaxis diferente (por ejemplo, usando indentación (sangría) y nombres de bloque).
- Tiempo Renderizado/desempeño.
- Características — debe considerar si los motores que elija poseen las siguientes características disponibles:

  - Herencia del diseño: Le permite definir una plantilla base y luego "heredar" sólo las partes que desea que sean diferentes para una página particular. Típicamente esto es un mejor enfoque que construir plantillas incluyendo un número de componentes requeridos, contruyéndolas desde cero cada vez.
  - Soporte para incluir: Le permite construir plantillas incluyendo otras plantillas.
  - Control conciso de la sintaxis de variables y ciclos.
  - Habilidad para filtrar valores de variables a nivel de las plantillas (e.g. convertir variables en mayúsculas, o darle formato a una fecha).
  - Habilidad para generar formatos de salida distintos al HTML (e.g. JSON o XML).
  - Soporte para operaciones asíncronas y de transmisión.
  - Pueden ser usadas tanto en el cliente como en el servidor. Si un motor de plantillas puede ser usado del lado del cliente esto da la posibilidad de servir datos y tener todo o la mayoría del renderizado del lado del cliente.

>-info- En Internet hay muchos recursos que le ayudarán a comparar diferentes opciones.

Para este proyecto usaremos el motor de plantillas [Pug](https://pugjs.org/api/getting-started.html) (este es el recientemente renombrado motor Jade), ya que es de los más populares lenguajes de plantillas Express/JavaScript y es soportado por el generador por defecto.

### ¿Qué motor de hojas de estilo CSS debería usar?

El _Generador de Aplicaciones Express_ le permite crear un proyecto que puede usar los más comunes motores de hojas de estilos CSS: [LESS](http://lesscss.org/), [SASS](http://sass-lang.com/), [Compass](http://compass-style.org/), [Stylus](http://stylus-lang.com/).

>-info- CSS tiene algunas limitaciones que dificultan ciertas tareas. Los motores de hojas de estilos CSS le permiten usar una sintaxis más poderosa para definir su CSS, y luego compilar la definición en texto plano para su uso en los navegadores .

Como los motores de plantillas, debería usar el motor CSS que le permita a su equipo ser más productivo. Para este proyecto usaremos CSS ordinario (opción por defecto) ya que nuestros requerimientos no son lo suficientemente complicados para justificar el uso de un motor CSS.

### ¿Qué base de datos debería usar?

El código generado no usa o incluye ninguna base de datos. Las aplicaciones _Express_ pueden usar cualquier [mecanismo de bases de datos](https://expressjs.com/en/guide/database-integration.html) soportado por _Node_ (_Express_ por si mismo no define ningún comportamiento o requerimiento para el manejo de bases de datos).

Discutiremos la integración con una base de datos en un posterior artículo.

## Creando el proyecto

Para el ejemplo que vamos a crear la app _Local Library_, crearemos un proyecto llamado _express-locallibrary-tutorial usando la librería de plantillas_ _Pug_ y ningún motor CSS.

Primero navega a donde quieras crear el proyecto y luego ejecute el _Generador de Aplicaciones Express en la línea de comandos como se muestra_:

```bash
express express-locallibrary-tutorial --view=pug
```

El generador creará (y listará) los archivos del proyecto.

```bash
   create : express-locallibrary-tutorial
   create : express-locallibrary-tutorial/package.json
   create : express-locallibrary-tutorial/app.js
   create : express-locallibrary-tutorial/public/images
   create : express-locallibrary-tutorial/public
   create : express-locallibrary-tutorial/public/stylesheets
   create : express-locallibrary-tutorial/public/stylesheets/style.css
   create : express-locallibrary-tutorial/public/javascripts
   create : express-locallibrary-tutorial/routes
   create : express-locallibrary-tutorial/routes/index.js
   create : express-locallibrary-tutorial/routes/users.js
   create : express-locallibrary-tutorial/views
   create : express-locallibrary-tutorial/views/index.pug
   create : express-locallibrary-tutorial/views/layout.pug
   create : express-locallibrary-tutorial/views/error.pug
   create : express-locallibrary-tutorial/bin
   create : express-locallibrary-tutorial/bin/www

   install dependencies:
     > cd express-locallibrary-tutorial && npm install

   run the app:
     > SET DEBUG=express-locallibrary-tutorial:* & npm start
```

Al final de la lista el generador mostrará instrucciones sobre como instalar las dependencias necesarias (mostradas en el archivo `package.json`) y luego como ejecutar la aplicación (las instrucciones anteriores son para windows; en Linux/macOS serán ligeramente diferentes).

## Ejecutando el esqueleto del sitio web

En este punto tenemos un esqueleto completo de nuestro proyecto. El sitio web no hace mucho actualmente, pero es bueno ejecutarlo para ver como funciona.

1. Primero instale las dependencias (el comando `install` recuperará todas las dependencias listadas en sel archivo `package.json` del proyecto).

    ```bash
    cd express-locallibrary-tutorial
    npm install
    ```

2. Luego ejecute la aplicación.

    - En Windows, use este comando:

      ```bash
      SET DEBUG=express-locallibrary-tutorial:* & npm start
      ```

    - En macOS o Linux, use este comando:

      ```bash
      DEBUG=express-locallibrary-tutorial:* npm start
      ```

3. Luego abre en el navegador `http://localhost:3000/` para acceder a la aplicación.

Deberías ver una página parecida a esta:

![image-20230122090754310](/node-express-library-teoria/assets/img/image-20230122090754310.png)

Tienes una aplicación Express funcional, ejecutándose en _localhost:3000_.

>-info- También podrías ejecutar la app usando el comando `npm start`. Especificado la variable `DEBUG` como se muestra para habilitar el logging/debugging por consola. Por ejemplo, cuando visites la página mostrada arriba verás la información de depuración como esta:
>
> ```bash
> $ SET DEBUG=express-locallibrary-tutorial:* &#x26; npm start
> 
> $ express-locallibrary-tutorial@0.0.0 start D:\express-locallibrary-tutorial
> $ node ./bin/www
> 
> express-locallibrary-tutorial:server Listening on port 3000 +0ms
> GET / 200 288.474 ms - 170
> GET /stylesheets/style.css 200 5.799 ms - 111
> GET /favicon.ico 404 34.134 ms - 1335
> ```

## Habilita el reinicio del servidor cuando los archivos sean modificados

Cualquier cambio que le haga a su sitio web Express no será visible hasta que reinicie el servidor. Tener que detener y reiniciar el servidor cada vez que hacemos un cambio, se vuelve irritante, así que es beneficioso tomarse un tiempo y automatizar el reinicio del servidor cuando sea necesario.

Una de las herramientas más sencillas para este propósito es [nodemon](https://github.com/remy/nodemon). Éste usualmente se instala globalmente (ya que es una "herramienta"), pero aquí lo instalaremos y usaremos localmente como una dependencia de desarrollo, así cualquier desarrollador que esté trabajando con el proyecto lo obtendrá automáticamente cuando instale la aplicación. Usa el siguiente comando en el directorio raíz del esqueleto del proyecto:

```bash
npm install --save-dev nodemon
```

Si abres el archivo `package.json` de tu proyecto verás una nueva sección con esta dependencia:

```json
  "devDependencies": {
    "nodemon": "^2.0.20"
  }
```

Debido a que la herramienta no fue instalada globalmente no podemos ejecutarla desde la línea de comandos (a menos que la agreguemos a la ruta) pero podemos llamarla desde un script `npm` porque `npm` sabe todo sobre los paquetes instalados. Busca la sección `scripts` de tu `package.json`. Inicialmente contendrá una línea, la cual comienza con `"start"`. Actualízala colocando una coma al final de la línea, y agregue la línea `"devstart"` mostrada abajo:

```json
  "scripts": {
    "start": "node ./bin/www",
    "devstart": "nodemon ./bin/www"
  },
```

Ahora podemos iniciar el servidor casi exactamente como antes, pero especificando el comando `devstart`:

- En Windows, use este comando:

  ```bash
  SET DEBUG=express-locallibrary-tutorial:* & npm run devstart
  ```

- En macOS or Linux, use este comando:

  ```bash
  DEBUG=express-locallibrary-tutorial:* npm run devstart
  ```

>-info Ahora si modificas cualquier archivo del proyecto el servidor se reiniciará. Aún necesitarás recargar el navegador para refrescar la página.
>
> Ahora tendremos que llamar "`npm run <nombre del script>`" en vez de `npm start`, porque "start" es actualmente un comando npm que es mapeado al nombre del script. Podríamos haber reemplazado el comando en el script _start_ pero sólo queremos usar _nodemon_ durante el desarrollo, así que tiene sentido crear un nuevo script para este comando.

## El proyecto generado

Observemos el proyecto que hemos creado.

### Estructura del directorio

El proyecto generado, ahora que has instalado las dependencias, tiene la siguiente estructura de archivos (los archivos son los elementos que **no** están precedidos con "/"). El archivo `package.json` define las dependencias de la aplicación y otra información. También define un script de inicio que es el punto de entrada de la aplicación, el archivo JavaScript `/bin/www`. Éste establece algunos de los manejadores de error de la aplicación y luego carga el archivo **app.js** para que haga el resto del trabajo. Las rutas se almacenan en módulos separados en el directorio `/routes`. las plantillas se almacenan en el directorio `/views`.

```
/express-locallibrary-tutorial
    app.js
    /bin
        www
    package.json
    /node_modules
        [about 4,500 subdirectories and files]
    /public
        /images
        /javascripts
        /stylesheets
            style.css
    /routes
        index.js
        users.js
    /views
        error.pug
        index.pug
        layout.pug
```

Las siguientes secciones describen los archivos con más detalle.

### package.json

El archivo `package.json` define las dependencias de la aplicación y otra información:

```json
{
  "name": "express-locallibrary-tutorial",
  "version": "0.0.0",
  "private": true,
  "scripts": {
    "start": "node ./bin/www",
    "devstart": "nodemon ./bin/www"
  },
  "dependencies": {
    "body-parser": "~1.18.2",
    "cookie-parser": "~1.4.3",
    "debug": "~2.6.9",
    "express": "~4.16.2",
    "morgan": "~1.9.0",
    "pug": "~2.0.0-rc.4",
    "serve-favicon": "~2.4.5"
  },
  "devDependencies": {
    "nodemon": "^1.14.11"
  }
}
```

Las dependencias incluyen el paquete _express_ y los paquetes para el motor de plantillas elegido (_pug_). Adicionalmente, tenemos los siguientes paquetes que son útiles en muchas aplicaciones web:

- [body-parser](https://www.npmjs.com/package/body-parser): Esto analiza la parte del cuerpo de una solicitud HTTP entrante y facilita la extracción de diferentes partes de la información contenida. Por ejemplo, puede usar esto para leer los parámetros POST.
- [cookie-parser](https://www.npmjs.com/package/cookie-parser): Se utiliza para analizar el encabezado de la cookie y rellenar `req.cookies` (esencialmente proporciona un método conveniente para acceder a la información de la cookie).
- [debug](https://www.npmjs.com/package/debug): Una pequeña utilidad de depuración de node modelada a partir de la técnica de depuración del núcleo de node.
- [morgan](https://www.npmjs.com/package/morgan): Un middleware registrador de solicitudes HTTP para node.
- [serve-favicon](https://www.npmjs.com/package/serve-favicon): Middleware de node para servir un favicon (este es el icono utilizado para representar el sitio dentro de la pestaña del navegador, marcadores, etc.).

La sección de scripts define un script de "_start_", que es lo que invocamos cuando llamamos a npm start para iniciar el servidor. Desde la definición del script, puede ver que esto realmente inicia el archivo JavaScript **./bin/www** con _node_. También define un script "_devstart_", que invocamos cuando llamamos a npm run devstart en su lugar. Esto inicia el mismo archivo **./bin/www**, pero con _nodemon_ en lugar de node.

```json
  "scripts": {
    "start": "node ./bin/www",
    "devstart": "nodemon ./bin/www"
  },
```
Este archivo es el equivalente a `composer.json` en Symfony 
### El archivo `www`

El archivo **/bin/www** es el punto de entrada de la aplicación. Lo primero que hace es `require()` del punto de entrada de la aplicación "real" (**app.js**, en la raíz del proyecto) que configura y devuelve el objeto de la aplicación express ().

```js
#!/usr/bin/env node

/**
 * Module dependencies.
 */

var app = require('../app');
```

>-info- `require()` es una función de node global que se usa para importar módulos en el archivo actual. Aquí especificamos el módulo app.js utilizando una ruta relativa y omitiendo la extensión de archivo opcional (.js).

El resto del código en este archivo configura un servidor HTTP de node con la aplicación configurada en un puerto específico (definido en una variable de entorno o 3000 si la variable no está definida), y comienza a escuchar e informar errores y conexiones del servidor. 

### app.js

Este archivo crea un objeto de aplicación rápida, configura la aplicación con varias configuraciones y middleware, y luego exporta la aplicación desde el módulo. El siguiente código muestra solo las partes del archivo que crean y exportan el objeto de la aplicación:

```js
var express = require('express');
var app = express();
...
module.exports = app;
```

De vuelta en el archivo de punto de entrada **www** anterior, es este objeto module.exports que se proporciona al llamante cuando se importa este archivo.

Permite trabajar a través del archivo **app.js** en detalle. Primero importamos algunas bibliotecas de node útiles en el archivo usando require (), incluyendo _express_, _serve-favicon_, _morgan_, _cookie-parser_ y _body-parser_ que previamente descargamos para nuestra aplicación usando npm; y _path_, que es una biblioteca central de nodos para analizar rutas de archivos y directorios.

```js
var express = require('express');
var path = require('path');
var favicon = require('serve-favicon');
var logger = require('morgan');
var cookieParser = require('cookie-parser');
var bodyParser = require('body-parser');
```

Luego `require()` carga los módulos de nuestro directorio de rutas. Estos modules/files contienen código para manejar conjuntos particulares de "routes" relacionadas (rutas URL). Cuando extendemos la aplicación esqueleto, por ejemplo, para enumerar todos los libros de la biblioteca, agregaremos un nuevo archivo para tratar las rutas relacionadas con los libros.

```js
var index = require('./routes/index');
var users = require('./routes/users');
```

>-warning-- En este punto, acabamos de importar el módulo; aún no hemos utilizado sus rutas (esto sucede un poco más abajo en el archivo).
A continuación, creamos el objeto `app` usando nuestro módulo _express_ importado y luego lo usamos para configurar el motor de vista (plantilla). Hay dos partes para configurar el motor. Primero establecemos el valor '`views`' para especificar la carpeta donde se almacenarán las plantillas (en este caso, la subcarpeta `/views`). Luego establecemos el valor de `view engine` para especificar la biblioteca de plantillas (en este caso, "pug").

```js
var app = express();

// view engine setup
app.set('views', path.join(__dirname, 'views'));
app.set('view engine', 'pug');
```
El siguiente conjunto de funciones llama a `app.use()` para agregar las bibliotecas _middleware_ a la cadena de manejo de solicitudes. Además de las bibliotecas de terceros que importamos anteriormente, usamos el middleware `express.static` para que _Express_ sirva todos los archivos estáticos en el directorio `/public` en la raíz del proyecto.

```js
// uncomment after placing your favicon in /public
//app.use(favicon(path.join(__dirname, 'public', 'favicon.ico')));
app.use(logger('dev'));
app.use(bodyParser.json());
app.use(bodyParser.urlencoded({ extended: false }));
app.use(cookieParser());
app.use(express.static(path.join(__dirname, 'public')));
```
Ahora que todo el otro middleware está configurado, agregamos nuestro código de manejo de rutas (previamente importado) a la cadena de manejo de solicitudes. El código importado definirá rutas particulares para las diferentes _partes_ del sitio:

```js
app.use('/', index);
app.use('/users', users);
```
El último middleware del archivo agrega métodos de controlador para errores y respuestas HTTP 404.

```js
// catch 404 and forward to error handler
app.use(function(req, res, next) {
  var err = new Error('Not Found');
  err.status = 404;
  next(err);
});

// error handler
app.use(function(err, req, res, next) {
  // set locals, only providing error in development
  res.locals.message = err.message;
  res.locals.error = req.app.get('env') === 'development' ? err : {};

  // render the error page
  res.status(err.status || 500);
  res.render('error');
});
```

El objeto de la aplicación Express (aplicación) ahora está completamente configurado. El último paso es agregarlo a las exportaciones del módulo (esto es lo que permite que sea importado por **/bin/www**).

```js
module.exports = app;
```

### Rutas

El archivo de ruta `/routes/users.js` se muestra a continuación (los archivos de ruta comparten una estructura similar, por lo que no necesitamos mostrar también **index.js**). Primero carga el módulo _express_ y lo usa para obtener un objeto `express.Router`. Luego, especifica una ruta en ese objeto y, por último, exporta el enrutador desde el módulo (esto es lo que permite importar el archivo a **app.js**).
```js
var express = require('express');
var router = express.Router();

/* GET users listing. */
router.get('/', function(req, res, next) {
  res.send('respond with a resource');
});

module.exports = router;
```
La ruta define una devolución de llamada que se invocará cada vez que se detecte una solicitud HTTP `GET` con el patrón correcto. El patrón coincidente es la ruta especificada cuando se importa el módulo ('`/users`') más lo que esté definido en este archivo ('`/`'). En otras palabras, esta ruta se utilizará cuando se reciba una URL de `/users/`.


>-info- Pruébalo ejecutando el servidor con node y visitando la URL en el navegador: `http://localhost:3000/users/`. Deberías ver un mensaje: 'respond with a resource'.
Una cosa de interés anterior es que la función de devolución de llamada tiene el tercer argumento `next` y, por lo tanto, es una función de middleware en lugar de una simple devolución de llamada de ruta. Si bien el código actualmente no usa el argumento `next`, puede ser útil en el futuro si desea agregar varios controladores de ruta a la ruta de ruta `'/'`.

### Vistas (templates)

Las vistas (plantillas) se almacenan en el directorio `/views` (como se especifica en **app.js**) y reciben la extensión de archivo **.pug**. El método [`Response.render()`](http://expressjs.com/en/4x/api.html#res.render) se usa para representar una plantilla específica junto con los valores de las variables con nombre pasadas en un objeto y, a continuación, envía el resultado como respuesta. En el siguiente código de `/routes/index.js`, puedes ver cómo esa ruta genera una respuesta usando la plantilla "index" pasando la variable de plantilla "title".

```js
/* GET home page. */
router.get('/', function(req, res) {
  res.render('index', { title: 'Express' });
});
```
La plantilla correspondiente para la ruta anterior se proporciona a continuación (`index.pug`). Hablaremos más sobre la sintaxis más adelante. Todo lo que necesitas saber por ahora es que la variable `title` (con el valor `'Express'`) se inserta donde se especifica en la plantilla.

```
extends layout

block content
  h1= title
  p Welcome to #{title}
```

## Reto

>-reto-Crea una nueva ruta en `routes/users.js` que mostrará el texto "_You are so cool"_ en la URL `/users/cool/`. Pruébalo ejecutando el servidor y visitando `http://localhost:3000/users/cool/` en tu navegador

