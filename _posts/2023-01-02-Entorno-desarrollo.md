---
typora-copy-images-to: ../assets/img/
typora-root-url: ../../
layout: post
title: Crear un entorno de desarrollo
categories: parte1
conToc: true
permalink: entorno-desarrollo-node-express
---

Ahora que sabes para que sirve Express,  vamos a mostrar como preparar y testear un entorno de desarrollo Node/Express en: Windows, Linux (Ubuntu), y macOS. Este artículo te va a dar todo lo que se necesita para poder empezar a desarrollar apps en Express, sin importar el sistema operativo que se use.

<table>
  <tbody>
    <tr>
      <th scope="row">Prerequisitos:</th>
      <td>
        Saber como abrir una terminal / línea de comando. Saber como instalar
        paquetes de software en su sistema operativo de su computadora de
        desarrollo.
      </td>
    </tr>
    <tr>
      <th scope="row">Objectivo:</th>
      <td>
        Configurar un entorno de desarrollo para Express (X.XX) en su
        computadora.
      </td>
    </tr>
  </tbody>
</table>

## Notas acerca del entorno de desarrollo

`Node` y `Express` hacen muy fácil configurar su computadora con el propósito de iniciar el desarrollo de aplicaciones web. Esta sección provee unas notas acerca de qué herramientas son necesarias, explica algunos de los métodos más simples para instalar `Node` (y `Express`) en **Ubuntu**, **macOS** y **Windows**, y muestra como puede probar su instalación.

### Qué es el entorno de desarrollo Express?

El entorno de desarrollo `Express` incluye una instalación de `Nodejs`, el `npm` administrador de paquetes, y (opcionalmente) el Generador de Aplicaciones de `Express` en su computadora local.

`Node` y el administrador de paquetes `npm` se instalan juntos desde paquetes binarios, instaladores, administradores de paquetes del sistema operativo o desde los fuentes (como se muestra en las siguientes secciones). `Express` es entonces instalado por npm como una dependencia individual de sus aplicaciones web `Express` (conjuntamente con otras librerías como motores de plantillas, controladores de bases de datos, `middleware` de autenticación, middleware para servir archivos estáticos, etc.)

`npm` puede ser usado también para (globalmente) instalar el Generador de Aplicaciones de `Express`, una herramienta manual para crear la estructura de las web apps de `Express` que siguen el [patrón MVC](https://developer.mozilla.org/en-US/docs/Glossary/MVC). El generador de aplicaciones es opcional porque no necesita utilizar esta herramienta para crear apps que usan Express, o construir apps Express que tienen el mismo diseño arquitectónico o dependencias. No obstante estaremos usandolo, porque hace mucho más fácil, y promueve una estructura modular de aplicación.

> -info-A diferencia de otros frameworks web , el entorno de desarrollo no incluye un servidor web independiente. Una aplicación web `Node`/`Express` crea y ejecuta su propio servidor web!

Hay otras herramientas periféricas que son parte de un entorno de desarrollo típico, incluyendo editores de texto o IDEs para edición de código, y herramientas de administración de control de fuentes como [Git](https://git-scm.com/) para administrar con seguridad diferentes versiones de su código. Asumimos que usted ya tiene instaladas esta clase de herramientas (en particular un editor de texto).

### Qué sistemas operativos son soportados?

`Node` puede ser ejecutado en Windows, macOS, varias "versiones" de Linux, Docker, etc. (hay una lista completa de paginas de [Downloads](https://nodejs.org/en/download/) de nodejs). Casi cualquier computadora personal podría tener el desempeño necesario para ejecutar Node durante el desarrollo. `Express` es ejecutado en un entorno `Node`, y por lo tanto puede ejecutarse en cualquier plataforma que ejecute `Node`.

En este articulo proveemos instrucciones para configurarlo para Windows, macOS, and Ubuntu Linux.

### ¿Qué versión de Node/Express puedo usar?

Hay varias [versiones de Node](https://nodejs.org/en/blog/release/) — recientes que contienen reparaciones de bugs, soporte para versiones mas recientes de ECMAScript (JavaScript) estándares, y mejoras a las APIs de Node .

Generalmente se debe usar la versión más reciente LTS (Long Term Support), una versión como esta es más estable que la versión "actual", mientras que sigue teniendo características relativamente recientes (y continua siendo activamente actualizado). Debería utilizar la versión _Actual_ si necesita una característica que no esta presente en la versión LTS.

Para `Express` siempre se debe utilizar la versión más reciente.

### ¿Qué pasa con bases de datos y otras dependencias?

Otras dependencias, tales como los controladores de bases de datos, motores de plantillas, motores de autenticación, etc. son parte de la aplicación, y son importadas dentro del entorno de la aplicación utilizando el administrador de paquetes npm.

## Instalar Node

Para poder utilizar `Express` primero tienes que instalar _Nodejs_ y el [Administrador de Paquetes de Node (npm)](https://docs.npmjs.com/) en su sistema operativo. Las siguientes secciones explican la forma más fácil de instalar la versión Soporte de Largo-Plazo (LTS) de Nodejs en Ubuntu Linux 16.04, macOS, y Windows 10.

> -info-Las secciones de abajo muestran la forma más fácil de instalar `Node` y `npm` en nuestras plataformas de sistemas operativo a elegir. Si esta utilizando otro SO o solo quiere ver alguna de otros enfoques para las plataformas actuales entonce vea [Instalando Node.js via administrador de paquetes](https://nodejs.org/en/download/package-manager/) (nodejs.org).

### macOS y Windows

Instalar `Node` y `npm` en Windows y macOS es sencillo, porque simplemente debe utilizar el instalador provisto:

1. Descargue el instalador requerido:

    1. Vaya a [https://nodejs.org/es/](https://nodejs.org/en/)
    2. Seleccione el botón para descargar la versión LTS que es "Recomendada la mayoría de los usuarios".

2. Instala Node al dar doble-clic en el archivo de descarga y en seguida la instalación inicia.

### Ubuntu 20.04

La forma más fácil de instalar la versión LTS de Node es la usar el [administrador de paquetes](https://nodejs.org/en/download/package-manager/#debian-and-ubuntu-based-linux-distributions) para obtenerlo del repositorio de distribuciones _binarias_ de Ubuntu. Esto puede ser hecho muy simple al ejecutar los siguientes dos comandos en tu terminal:

```bash
curl -sL https://deb.nodesource.com/setup_lts.x | sudo -E bash -
sudo apt-get install -y nodejs
```

> -warning-No instales directamente desde los repositorios normales de Ubuntu porque pueden contener versiones muy antiguas de Node.

### Probar su instalación de Nodejs y npm

La forma más fácil de probar que Node está instalado es ejecutar el comando "version" en su prompt de terminal/command y checar que una cadena de versión es devuelta:

```bash
>node -v
v16.17.1
```

El administrador de paquetes `npm` de _Nodejs_ también debería haber sido instalado y puede ser probado de la misma forma:

```bash
>npm -v
8.19.2
```

Como una prueba un poco más emocionante creemos un muy básico "servidor node" que simplemente imprima "Hola Mundo" en el navegador cuando visite la URL correcta en él:

1. Copia el siguiente texto en un archivo llamado **holanode.js**. Este utiliza características básicas de Node (nada desde Express) y algo de sintaxis ES6:

    ```js
    //Load HTTP module
    const http = require("http");
    const hostname = '127.0.0.1';
    const port = 3000;
    
    //Create HTTP server and listen on port 3000 for requests
    const server = http.createServer((req, res) => {
    
      //Set the response HTTP header with HTTP status and Content type
      res.statusCode = 200;
      res.setHeader('Content-Type', 'text/plain');
      res.end('Hello World\n');
    });
    
    //listen for request on port 3000, and as a callback function have the port listened on logged
    server.listen(port, hostname, () => {
      console.log(`Server running at http://${hostname}:${port}/`);
    });
    ```

    El código importa el módulo "http" y lo usa para crear un servidor (`createServer()`) que escucha las solicitudes HTTP en el puerto 3000. Luego, el script imprime un mensaje en la consola con la URL del navegador que se puede usar para probar el servidor. La función `createServer()` toma como argumento una función `callback` que se invocará cuando se reciba una solicitud HTTP — esto simplemente devuelve una respuesta con un código de estado HTTP de 200 ("OK") y el texto sin formato "Hello World".

    > -info-¡No te preocupes si aún no comprendes exactamente lo que está haciendo este código! ¡Explicaremos nuestro código con mayor detalle una vez que comencemos a usar Express!

2. Inicia el servidor navegando en el mismo directorio que su archivo `hellonode.js` en su símbolo del sistema, y llamando a `node` junto con el nombre del script, así:

    ```bash
    >node hellonode.js
    Server running at http://127.0.0.1:3000/
    ```

3. Navega a la URL `http://127.0.0.1:3000`. Sí todo esta funciona, el navegador simplemente debe mostrar la cadena de texto "Hello World".

## Usando npm

Junto al propio node, [npm](https://docs.npmjs.com/) es la herramienta más importante para trabajar con aplicaciones de node. npm se usa para obtener los paquetes (bibliotecas de JavaScript) que una aplicación necesita para el desarrollo, las pruebas y/o la producción, y también se puede usar para ejecutar pruebas y herramientas utilizadas en el proceso de desarrollo.

> -info-Desde la perspectiva de Node, Express es solo otro paquete que necesita instalar usando npm y luego requerir en su propio código.

Se puede usar npm manualmente para buscar por separado cada paquete necesario. Por lo general, administramos las dependencias utilizando un archivo de definición de texto plano llamado [package.json](https://docs.npmjs.com/files/package.json). Este archivo enumera todas las dependencias para un "paquete" de JavaScript específico, incluido el nombre del paquete, la versión, la descripción, el archivo inicial a ejecutar, las dependencias de producción, las dependencias de desarrollo, las versiones de Node con las que puede trabajar, etc. El archivo `package.json` debería contener todo lo que npm necesita para buscar y ejecutar su aplicación (si estuviera escribiendo una biblioteca reutilizable, podría usar esta definición para cargar su paquete en el repositorio npm y ponerlo a disposición de otros usuarios).

### Agregando dependencias

Los siguientes pasos muestran cómo puede usar npm para descargar un paquete, guardarlo en las dependencias del proyecto y luego requerirlo en una aplicación Node.

> -info-Aquí mostramos las instrucciones para buscar e instalar el paquete `Express`. Más adelante mostraremos cómo este paquete y otros ya están especificados para nosotros utilizando el _Generador de aplicaciones Express_. Esta sección se proporciona porque es útil para comprender cómo funciona npm y qué está creando el generador de aplicaciones.

1. Primero crea un directorio para su nueva aplicación y acceda a él:

    ```bash
    mkdir myapp
    cd myapp
    ```

2. Usa el comando `npm init` para crear un archivo `package.json` para su aplicación. Este comando te solicita varias cosas, incluido el nombre y la versión de su aplicación y el nombre del archivo de punto de entrada inicial (de forma predeterminada, esto es `index.js`). Por ahora, solo acepta los valores predeterminados:

    ```bash
    npm init
    ```

    Si muestras el archivo `package.json` (`cat package.json`), verás los valores predeterminados que aceptó, que finalizarán con la licencia.

    ```json
    {
      "name": "myapp",
      "version": "1.0.0",
      "description": "",
      "main": "index.js",
      "scripts": {
        "test": "echo \"Error: no test specified\" && exit 1"
      },
      "author": "",
      "license": "ISC"
    }
    ```

3. Ahora instala `Express` en el directorio `myapp` y guárdalo en la lista de dependencias de su archivo `package.json`

    ```bash
    npm install express --save
    ```

    La sección de dependencias de su `package.json` ahora aparecerá al final del archivo `package.json` e incluirá `Express`.

    ```json
    {
      "name": "myapp",
      "version": "1.0.0",
      "description": "",
      "main": "index.js",
      "scripts": {
        "test": "echo \"Error: no test specified\" && exit 1"
      },
      "author": "",
      "license": "ISC",
      "dependencies": {
        "express": "^4.16.3"
      }
    }
    ```

4. Para usar la biblioteca, llama a la función `require()` como se muestra a continuación en tu archivo `index.js`.
    Crea un archivo llamado `index.js` en la raíz del directorio de la aplicación "myapp" con el contenido que se muestra a continuación.

    ```js
    const express = require('express')
    const app = express();

    app.get('/', (req, res) => {
      res.send('Hello World!')
    });

    app.listen(8000, () => {
      console.log('Example app listening on port 8000!')
    });
    ```

    Este código muestra una aplicación web mínima "HelloWorld" Express. Esto importa el módulo "express" y lo usa para crear un servidor (`app`) que escucha las solicitudes HTTP en el puerto 8000 e imprime un mensaje en la consola que indica qué URL del navegador puede usar para probar el servidor. La función `app.get ()` solo responde a las solicitudes HTTP `GET` con la ruta URL especificada (`'/'`), en este caso llamando a una función para enviar nuestro mensaje Hello World! .

5. Puedes iniciar el servidor llamando a `node` con el script en su símbolo del sistema:

    ```bash
    >node index.js
    Example app listening on port 8000
    ```

6. Navega a la URL (`http://127.0.0.1:8000/`). Sí todo esta funciona, el navegador simplemente debe mostrar la cadena de texto "Hello World".

### Dependencias de Desarrollo

Si una dependencia solo se usa durante el desarrollo, debes guardarla como una "dependencia de desarrollo" (para que los usuarios de su paquete no tengan que instalarla en producción). Por ejemplo, para usar la popular herramienta Linting JavaScript [eslint](http://eslint.org/) llamarías a `npm` como se muestra a continuación:

```bash
npm install eslint --save-dev
```

La siguiente entrada se agregaría al **paquete.json** de su aplicación:

```js
  "devDependencies": {
    "eslint": "^4.12.1"
  }
```

> -info-"[Linters](<https://en.wikipedia.org/wiki/Lint_(software)>)" son herramientas que realizan análisis estáticos en el software para reconocer e informar la adhesión / no adhesión a algún conjunto de mejores prácticas de codificación.

### Ejecutando tareas

Además de definir y buscar dependencias, también puedes definir scripts con nombre en sus archivos `package.json` y llamar a `npm` para ejecutarlos con el comando [run-script](https://docs.npmjs.com/cli/run-script). Este enfoque se usa comúnmente para automatizar las pruebas en ejecución y partes de la cadena de herramientas de desarrollo o construcción (por ejemplo, ejecutar herramientas para minimizar JavaScript, reducir imágenes, LINT/analizar su código, etc.).

> -info-Los ejecutadores de tareas como [Gulp](http://gulpjs.com/) y [Grunt](http://gruntjs.com/) también se pueden usar para ejecutar pruebas y otras herramientas externas.

Por ejemplo, para definir un script para ejecutar la dependencia de desarrollo de `eslint` que especificamos en la sección anterior, podríamos agregar el siguiente bloque de script a nuestro archivo `package.json` (suponiendo que el origen de nuestra aplicación esté en una carpeta `/src/js`):

```js
"scripts": {
  ...
  "lint": "eslint src/js"
  ...
}
```

Para explicar un poco más, `eslint src/js` es un comando que podríamos ingresar en nuestra línea de terminal/linea de comandos para ejecutar `eslint` en archivos JavaScript contenidos en el directorio `src/js` dentro de nuestro directorio de aplicaciones. Incluir lo anterior dentro del archivo `package.json` de nuestra aplicación proporciona un acceso directo para este comando: `lint`.

Entonces podríamos ejecutar `eslint` usando `npm` llamando a:

```bash
npm run-script lint
# OR (using the alias)
npm run lint
```

Es posible que este ejemplo no parezca más corto que el comando original, pero puede incluir comandos mucho más grandes dentro de sus scripts `npm`, incluidas cadenas de comandos múltiples. Puede identificar un solo script `npm` que ejecute todas sus pruebas a la vez.

## Instalando Express Application Generator

La herramienta [Express Application Generator](https://expressjs.com/en/starter/generator.html) genera un "esqueleto" de la aplicación Express. Instala el generador usando `npm` como se muestra (el indicador `-g` instala la herramienta globalmente para que pueda llamarla desde cualquier lugar):

```
npm install express-generator -g
```

Para crear una aplicación `Express` llamada "helloworld" con la configuración predeterminada, navegue hasta donde desea crearla y ejecute la aplicación como se muestra:

```bash
express helloworld
```

> -info-También puedes especificar la biblioteca de plantillas para usar y una serie de otras configuraciones. Usa el comando `--help` para ver todas las opciones:
>
> ```
> express --help
> ```

`npm` creará la nueva aplicación Express en una subcarpeta de su ubicación actual, mostrando el progreso de la compilación en la consola. Al finalizar, la herramienta mostrará los comandos que necesita ingresar para instalar las dependencias de Node e iniciar la aplicación.

> -info-La nueva aplicación tendrá un archivo `package.json` en su directorio raíz. Puede abrir esto para ver qué dependencias están instaladas, incluidas `Express` y la biblioteca de plantillas Jade (pug):
>
> ```json
> {
>   "name": "helloworld",
>   "version": "0.0.0",
>   "private": true,
>   "scripts": {
>     "start": "node ./bin/www"
>   },
>   "dependencies": {
>     "body-parser": "~1.18.2",
>     "cookie-parser": "~1.4.3",
>     "debug": "~2.6.9",
>     "express": "~4.15.5",
>     "jade": "~1.11.0",
>     "morgan": "~1.9.0",
>     "serve-favicon": "~2.4.5"
>   }
> }
> ```

Instala todas las dependencias para la aplicación helloworld usando npm como se muestra:

```bash
cd helloworld
npm install
```

Luego ejecuta la aplicación (los comandos son ligeramente diferentes para Windows y Linux/macOS), como se muestra a continuación:

```bash
#  Ejecute helloworld en Windows con símbolo del sistema
SET DEBUG=helloworld:* & npm start

#  Ejecute helloworld en Windows con PowerShell
SET DEBUG=helloworld:* | npm start

#  Ejecute helloworld en Linux/macOS
DEBUG=helloworld:* npm start
```

El comando `DEBUG` crea registros útiles, lo que resulta en una salida como la que se muestra a continuación.

```bash
>SET DEBUG=helloworld:* & npm start

> helloworld@0.0.0 start D:\Github\expresstests\helloworld
> node ./bin/www

  helloworld:server Listening on port 3000 +0ms
```

Abre un navegador y navegue a `http://127.0.0.1:3000/` para ver la página de bienvenida `Express` predeterminada.

![Express - Generated App Default Screen](https://developer.mozilla.org/en-US/docs/Learn/Server-side/Express_Nodejs/development_environment/express_default_screen.png)

Hablaremos más sobre la aplicación generada cuando lleguemos al artículo sobre la generación de una aplicación esqueleto.

## Resumen

Ahora tiene un entorno de desarrollo de Node en funcionamiento en su computadora que puede usarse para crear aplicaciones web Express. También ha visto cómo se puede usar npm para importar Express en una aplicación, y también cómo puede crear aplicaciones usando la herramienta Express Application Generator y luego ejecutarlas.

En el siguiente artículo, comenzaremos a trabajar a través de un tutorial para crear una aplicación web completa utilizando este entorno y las herramientas asociadas.

## Ver también

- [Downloads](https://nodejs.org/en/download/) page (nodejs.org)
- [Installing Node.js via package manager](https://nodejs.org/en/download/package-manager/) (nodejs.org)
- [Installing Express](http://expressjs.com/en/starter/installing.html) (expressjs.com)
- [Express Application Generator](https://expressjs.com/en/starter/generator.html) (expressjs.com)

