{{meta {code_links: "[\"code/file_server.js\"]"}}}

# Node.js

{{quote {author: "Maestro Yuan-Ma", title: "El Libro de la Programación", chapter: true}

Un estudiante preguntó, 'Los programadores de antaño usaban solo máquinas simples sin
lenguajes de programación, y sin embargo hacían hermosos programas. ¿Porque nosotros
usamos máquinas complicadas con lenguajes de programación?'. Fu-Tzu respondió,
'Los constructores de antaño usaban solo palos y barro, y sin embargo hacían
hermosas chozas.'

quote}}

{{index "Yuan-Ma", "Book of Programming"}}

{{figure {url: "img/chapter_picture_20.jpg", alt: "Picture of a telephone pole", chapter: "framed"}}}

{{index "command line"}}

Hasta ahora, hemos utilizado el lenguaje JavaScript en un solo entorno:
el navegador. Este capítulo y el [siguiente](skillsharing)
introduciran brevemente ((Node.js)), un programa que te permite aplicar tus
habilidades de JavaScript fuera del navegador. Con él, puedes construir
cualquier cosa, desde pequeñas herramientas de línea de comandos hasta
((servidores)) HTTP, con la habilidad de potenciar ((sitios web)) dinámicos.

Estos capítulos intentarán enseñarte los conceptos principales que Node.js utiliza
y darte la información suficiente como para que puedas escribir programas útiles en el.
Estos capítulos no pretenden ser un tratamiento completo, ni exhaustivo, de la
plataforma.

{{if interactive

Mientras que en capítulos anteriores podías ejecutar el código directamente
en estas páginas, ya que era JavaScript puro o escrito específicamente para el
navegador, los ejemplos de código en este capítulo están escritos para Node y
a menudo no podrán ser ejecutados en el navegador.

if}}

Si deseas seguir los ejemplos y ejecutar el código en este capítulo,
es necesario que instales la versión 10.1 o superior de Node.js. Para hacerlo, ve a
[_https://nodejs.org_](https://nodejs.org) y sigue las instrucciones de
instalación para tu sistema operativo. Allí también puedes encontrar más
((documentación)) para Node.js.

## Trasfondo

{{index responsiveness, input}}

Uno de los problemas más difíciles al escribir sistemas que se
comunican a través de la ((red)) es la gestión de entradas y ((salidas))—es
decir, la lectura y escritura de datos, desde y hacia la red y ((discos
duros)). Mover datos toma tiempo, y ((gestionar)) esto de manera inteligente
puede hacer una gran diferencia en cuanto a la rapidez con la que un sistema
responde al usuario o a solicitudes de red.

En tales programas, la ((programación asíncrona)) a menudo es útil. Esta
permite que el programa envíe y reciba datos desde y hacia múltiples
dispositivos al mismo tiempo sin una complicada gestión y
sincronización de [subprocesos](https://es.wikipedia.org/wiki/Hilo_(inform%C3%A1tica)).

{{index "programming language", "Node.js", standard}}

Node fue concebido inicialmente con el propósito de hacer que la programación
asíncrona fuera fácil y conveniente. JavaScript se presta bien a un
sistema como Node. Es uno de los pocos lenguajes de programación que no
tiene una forma predeterminada de realizar entradas y salidas. Por lo tanto,
JavaScript podía adaptarse al enfoque bastante excéntrico con el que Node
realiza entradas y salidas sin terminar con dos interfaces inconsistentes.
En 2009, cuando Node estaba siendo diseñado, las personas ya estaban haciendo
uso de la programación basada en devoluciones de llamadas en el navegador, por lo que
la ((comunidad)) alrededor del lenguaje ya estaba acostumbrada a
un estilo de ((programación asíncrona)).

## El comando node

{{index "node program"}}

Cuando ((Node.js)) está instalado en un sistema, este proporciona un programa
llamado `node`, que se puede utilizar para ejecutar archivos de JavaScript. Digamos
que tienes un archivo llamado `hola.js`, el cual contiene este código:

```
let mensaje = "Hola mundo";
console.log(mensaje);
```

Para ejecutar este programa, puedes hacer uso del comando `node` desde tu ((terminal)), de esta manera :

```{lang: null}
$ node hola.js
Hola mundo
```

{{index "console.log"}}

El método `console.log` en Node hace algo similar a lo que este
hace en el navegador. Este imprime una pieza de texto. Pero en Node, el
texto irá al flujo del proceso de ((salida estándar)) del computador, en lugar de
a la ((consola)) de un navegador. Esto significa, que cuando ejecutas `node`
desde tu línea de comandos, ves los valores escritos en tu
((terminal)).

{{index "node program", "read-eval-print loop"}}

Si ejecutas `node` sin darle un archivo, este te proporciona de una linea
de comandos en la cual puedes escribir código JavaScript e inmediatamente
ver los resultados.

```{lang: null}
$ node
> 1 + 1
2
> [-1, -2, -3].map(Math.abs)
[1, 2, 3]
> process.exit(0)
$
```

{{index "process object", "global scope", [binding, global], "exit method", "status code"}}

La vinculación `process`, al igual que la vinculación `console`, está disponible
de manera global en Node. Esta proporciona varias formas de inspeccionar y manipular
el programa actual. El método `exit` termina el proceso actual, y este puede recibir
un código de estado de salida, el cual le indica al programa que inició `node`
(en este caso, la línea de comandos de nuestra computadora) si el programa
fue completado con éxito (con el código cero) o si este encontró un error
(cualquier otro código).

{{index "command line", "argv property"}}

Para encontrar los argumentos dados en la línea de comando a tu script, puedes leer
`process.argv`, el cual es un array de strings. Ten en cuenta que este
tambien incluye el nombre del comando `node` y el nombre de tu script, por lo que
los argumentos reales comienzan a partir del índice 2. Si `mostrar_argumentos.js`
contuviera la declaración `console.log(process.argv)`, podrías ejecutarlo de
esta manera:

```{lang: null}
$ node mostrar_argumentos.js uno --y dos
["node", "/tmp/mostrar_argumentos.js", "uno", "--y", "dos"]
```

{{index [binding, global]}}

Todas las vinculaciones globales ((estándar)) de JavaScript, como `Array`,
`Math`, y `JSON`, están también presentes en el entorno de Node.
Por el otro lado, funcionalidades relacionadas con el navegador,
como `document` o `prompt`, no lo están.

## Módulos

{{index "Node.js", "global scope", "module loader"}}

Más allá de las vinculaciones que acaban de ser mencionadas, como `console` y `process`,
Node no proporciona de muchas vinculaciones adicionales en el alcance global.
Si quieres acceder a funcionalidades incorporadas, tendras que
pedírselas al sistema de módulos.

{{index "require function"}}

El sistema de módulos ((CommonJS)), basado en la función `require`, fue
descrito en el [Capítulo 10](modulos#commonjs). Este sistema está integrado en
Node y es utilizado para cargar cualquier cosa, desde ((módulo))s incorporados a
((paquete))s descargados a ((archivo))s que son parte de nuestros propios
programas.

{{index [path, "file system"], "relative path", resolution}}

Cuando `require` es llamado, Node tiene que convertir el string dado en un
((archivo)) real que este pueda cargar. Nombres de ruta que comienzan con `/`,
`./`, o `../` son convertidos en relación a la ruta del módulo actual,
donde `.` representa el directorio actual, `../` representa un
directorio arriba del actual, y `/` representa la raíz del sistema de archivos. Así que si pides
`"./grafo"` desde el archivo `/tmp/robot/robot.js`, Node intentará
cargar el archivo `/tmp/robot/grafo.js`.

{{index "index.js"}}

La ((extensión)) `.js` puede ser omitida, y Node la agregará si tal
archivo existe. Si la ruta requerida se refiere a un ((directorio)), Node
intentara cargar el archivo llamado `index.js` dentro de ese directorio.

{{index "node_modules directory", directory}}

Cuando un string que no parece ser una ruta relativa o absoluta es
dada a `require`, se asume que nos estamos refiriendo a un ((módulo)) incorporado de Node
o a un módulo instalado en el directorio `node_modules`. Por ejemplo,
`require("fs")` te dará acceso al módulo de sistema de archivos
integrado en Node. Y `require("robot")` podría intentar cargar una libreria
que se encuentre en `node_modules/robot/`. Una forma común de instalar tales
librerias es usando ((NPM)), al cual regresaremos en un momento.

{{index "require function", "Node.js", "garble example"}}

Vamos a armar un pequeño proyecto que consiste de dos archivos. El primero,
llamado `main.js`, define un script que puede ser llamado desde la
((línea de comandos)) para revertir un string.

```
const {revertir} = require("./revertir");

// El indice 2 contiene el primer argumento dado en la linea de comandos
let argumento = process.argv[2];

console.log(revertir(argumento));
```

{{index reuse, "Array.from function", "join method"}}

El archivo `revertir.js` define una libreria para revertir strings, la cual
puede ser utilizada tanto por esta herramienta de línea de comandos como por
otros scripts que necesiten acceso directo a una función para revertir strings.

```
exports.revertir = function(string) {
  return Array.from(string).reverse().join("");
};
```

{{index "exports object", CommonJS}}

Recuerda que cuando agregamos propiedades a `exports`, las estamos agregando a la
((interfaz)) del módulo. Dado que Node.js trata los archivos como
((modulos)) de ((CommonJS)), `main.js` puede tener acceso a la función
`revertir` la cual esta siendo exportada desde `revertir.js`.

Ahora podemos llamar a nuestra herramienta de esta manera:

```{lang: null}
$ node main.js JavaScript
tpircSavaJ
```

## Instalando con NPM

{{index NPM, "Node.js", "npm program", library}}

NPM, el cual fue introducido en el [Capítulo 10](modulos#modules_npm), es un
repositorio en línea de ((módulo))s de JavaScript, muchos de los cuales son
escritos específicamente para Node. Cuando instalas Node en tu computadora,
también obtienes el comando `npm`, el cual puedes utilizar para interactuar
con este repositorio.

{{index "ini package"}}

El principal uso de NPM es ((descargar)) paquetes. Vimos el paquete `ini` en el
[Capítulo 10](modulos#modules_ini). Podemos usar NPM para buscar e instalar
ese paquete en nuestra computadora

```{lang: null}
$ npm install ini
npm WARN enoent ENOENT: no such file or directory,
         open '/tmp/package.json'
+ ini@1.3.5
added 1 package in 0.552s

$ node
> const {parse} = require("ini");
> parse("x = 1\ny = 2");
{ x: '1', y: '2' }
```

{{index "require function", "node_modules directory", "npm program"}}

Después de correr `npm install`, ((NPM)) habrá creado un directorio
llamado `node_modules`. Dentro de ese directorio, habrá un directorio `ini`
el cual contiene la ((libreria)). Puedes abrirlo y leer el código.
Cuando llamamos a `require("ini")`, esta libreria es cargada, y
podemos llamar a su propiedad `parse` para analizar un archivo de configuración.

Por defecto, NPM instala paquetes en el directorio actual, en lugar de
en un lugar central. Si estás acostumbrado a otros gestores de paquetes,
esto puede parecer inusual, pero tiene sus ventajas—esto le da a cada aplicación
pleno control de los paquetes que instala y hace que sea más fácil
administrar versiones y limpiar luego de remover una aplicación.

### Archivos de paquetes

{{index "package.json", dependency}}

En el ejemplo de `npm install`, pudiste haber visto una ((advertencia)) acerca del
hecho de que el archivo `package.json` no existía. Es recomendado
crear un archivo de este tipo para cada proyecto, ya sea manualmente o ejecutando
el comando `npm init`. Este archivo contiene algo de información sobre el proyecto, como
su nombre y ((versión)), y enumera sus dependencias.

La simulación de robot del [Capítulo 7](robot), al ser modularizada en el
ejercicio del [Capítulo 10](modulos#modular_robot), podría tener un archivo
`package.json` como este:


```{lang: "application/json"}
{
  "author": "Marijn Haverbeke",
  "name": "eloquent-javascript-robot",
  "description": "Simulación de un robot de entrega de paquetes",
  "version": "1.0.0",
  "main": "run.js",
  "dependencies": {
    "dijkstrajs": "^1.0.1",
    "random-item": "^1.0.0"
  },
  "license": "ISC"
}
```

{{index "npm program", tool}}

Cuando ejecutas `npm install` sin nombrar un paquete a instalar, NPM
instalará las dependencias listadas en `package.json`. Cuando tú
instales un paquete específico que no esté listado como una dependencia,
NPM lo agregará a `package.json`.

### Versiones

{{index "package.json", dependency, evolution}}

Un archivo `package.json` enumera tanto la ((versión)) del propio proyecto al que pertenece
como a las versiones de sus dependencias. Las versiones son una manera de lidiar con el
hecho de que los ((paquete))s evolucionan por separado, y el código escrito para funcionar
con un paquete tal y como existía en un algun momento, podria no funcionar con una
versión posterior y modificada del mismo paquete.

{{index compatibility}}

NPM exige que sus paquetes sigan un esquema llamado _((versionamiento semántico))_,
que codifica cierta información acerca de qué versiones son
_compatibles_ (no rompen la interfaz antigua) en el número de versión. Una
versión semántica consiste de tres números, separados por puntos,
como `2.3.0`. Cada vez que una nueva funcionalidad es añadida, el número del medio
tiene que ser incrementado. Cada vez que se rompe la compatibilidad, por lo que
el código existente que utiliza el paquete podría no funcionar con la nueva
versión, el primer número tiene que ser incrementado.

{{index "caret character"}}

Un carácter de intercalación (`^`) delante del número de versión de una
dependencia en `package.json` indica que cualquier versión compatible
con el número dado puede ser instalada. Entonces, por ejemplo, `"^2.3.0"`
significaría que cualquier versión mayor o igual a 2.3.0 y menor
que 3.0.0 está permitida.

{{index publishing}}

El comando `npm` también se utiliza para publicar nuevos paquetes o versiones nuevas
de paquetes. Si ejecutas `npm publish` en un ((directorio)) que tiene un
archivo `package.json`, un paquete con el nombre y versión listada en el
archivo JSON sera publicado en el registro de NPM. Cualquier persona puede publicar
paquetes en NPM—aunque solo con un nombre de paquete que no este actualmente en uso, ya que sería
algo atemorizante si personas aleatorias pudieran actualizar paquetes ya existentes.

Dado que el programa `npm` es una pieza de software que se comunica con un
sistema abierto—el registro de paquetes—no hay nada único acerca de lo que
hace. Otro programa, `yarn`, el cual puede ser instalado desde el
registro de NPM, cumple el mismo rol que `npm`, pero utilizando una interfaz
algo diferente con una estrategia de instalación de paquetes distinta.

Este libro no profundizará mas en los detalles del uso de ((NPM)). Chequea [_https://npmjs.org_](https://npmjs.org) para obtener más documentación y una
manera de buscar paquetes.

## El módulo del sistema de archivos

{{index directory, "fs package", "Node.js"}}

Uno de los módulos incorporados más utilizados en Node es el módulo `fs`,
que significa _((file system))_ ("sistema de archivos" en español).
Este exporta funciones para trabajar con ((archivo))s y directorios.

{{index "readFile function", "callback function"}}

Por ejemplo, la función llamada `readFile` ("leerArchivo") lee un archivo
y luego ejecuta una devolución de llamada con el contenido del archivo.

```
let {readFile} = require("fs");
readFile("archivo.txt", "utf8", (error, texto) => {
  if (error) throw error;
  console.log("El archivo contiene:", texto);
});
```

{{index "Buffer class"}}

El segundo argumento dado a `readFile` indica la _((codificación de caracteres))_
a utilizar para decodificar el archivo en un string. Existen varias
formas en las que un ((texto)) puede codificarse en ((datos binarios)), pero la mayoría
de los sistemas modernos usan ((UTF-8)). Así que, a menos que tengas razones para creer
que otra codificación esta siendo utilizada, pasa el argumento `"utf8"` al leer
un archivo de texto. Si no pasas una codificación, Node asumirá que estás interesado en
los datos binarios y te dará un objeto `Buffer` en lugar de un
string. Este es un ((objeto similar a un array)), el cual contiene números
representando los bytes (trozos de datos de 8-bits) en los archivos.

```
const {readFile} = require("fs");
readFile("archivo.txt", (error, buffer) => {
  if (error) throw error;
  console.log("El archivo contiene", buffer.length, "bytes.",
              "El primer byte es:", buffer[0]);
});
```

{{index "writeFile function", "file system"}}

Una función similar, `writeFile` ("escribirArchivo"), es utilizada para escribir un ((archivo)) en el disco duro.

```
const {writeFile} = require("fs");
writeFile("graffiti.txt", "Node estuvo aqui", err => {
  if (err) console.log(`Error al escribir archivo: ${err}`);
  else console.log("Archivo escrito.");
});
```

{{index "Buffer class", "character encoding"}}

Aquí no fue necesario especificar la codificación—`writeFile` asumira que
cuando se le da un string a escribir, en lugar de un objeto `Buffer`,
este debe ser escrito como texto utilizando su codificación de caracteres
por defecto, la cual es ((UTF-8)).

{{index "fs package", "readdir function", "stat function", "rename function", "unlink function"}}

El módulo `fs` contiene muchas otras funciones útiles: `readdir` ("leerDirectorio")
retornara los ((archivo))s en un ((directorio)) como un array de strings, `stat` ("estatus")
te dara información acerca de un archivo, `rename` ("renombrar") cambiará el nombre de un archivo,
`unlink` ("desenlazar") eliminará uno, y así sucesivamente. Lee la documentación en
[_https://nodejs.org_](https://nodejs.org) para obtener detalles mas específicos.

{{index "asynchronous programming", "Node.js", "error handling", "callback function"}}

La mayoría de estas toman una función de devolución de llamada como el último parámetro, la
cual es llamada ya sea con un error (el primer argumento) o con un resultado exitoso
(el segundo). Como vimos en el [Capítulo 11](async), hay
desventajas con este estilo de programación—el más grande es que
el manejo de errores se vuelve mas verboso y propenso a errores.

{{index "Promise class", "promises package"}}

Aunque las promesas han sido parte de JavaScript por un tiempo, al momento
de escribir este libro, su integración con Node.js sigue siendo un trabajo en progreso.
Hay un objeto `promises` ("promesas") exportado desde el paquete `fs` a partir de
la versión 10.1 de Node, que contiene la mayoría de las mismas funciones que se encuentran
en `fs` pero utilizando promesas en lugar de funciones de devolución de llamada.

```
const {readFile} = require("fs").promises;
readFile("archivo.txt", "utf8")
  .then(text => console.log("El archivo contiene:", text));
```

{{index "synchronous programming", "fs package", "readFileSync function"}}

Algunas veces no se necesita asincronicidad, y esta solo se interpone en el camino.
Muchas de las funciones en `fs` también tienen una variante síncrona, la cual
tiene el mismo nombre pero con `Sync` agregado al final. Por ejemplo, la
versión síncrona de `readFile` es llamada `readFileSync`.

```
const {readFileSync} = require("fs");
console.log("El archivo contiene:",
            readFileSync("archivo.txt", "utf8"));
```

{{index optimization, performance, blocking}}

Ten en cuenta que mientras se realiza una operación síncrona,
tu programa se detiene por completo. Si este debería estar respondiendo a
usuarios o a otras máquinas en la red, estar atascado en una acción síncrona
puede producir retrasos molestos.

## El módulo HTTP

{{index "Node.js", "http package"}}

Otro módulo central es llamado `http`. Este proporciona funcionalidad para
ejecutar ((servidores)) ((HTTP)) y realizar ((solicitudes)) HTTP.

{{index "listening (TCP)", "listen method", "createServer function"}}

Esto es todo lo que se necesita para iniciar un servidor HTTP:

```
const {createServer} = require("http");
let servidor = createServer((solicitud, respuesta) => {
  respuesta.writeHead(200, {"Content-Type": "text/html"});
  respuesta.write(`
    <h1>Hola!</h1>
    <p>Pediste por <code>${solicitud.url}</code></p>`);
  respuesta.end();
});
servidor.listen(8000);
console.log("Escuchando! (en el puerto 8000)");
```

{{index port, localhost}}

Si ejecutas este script en tu propia máquina, puedes ingresar la
dirección [_http://localhost:8000/hola_](http://localhost:8000/hola) en tu
((navegador)) web para realizar una solicitud a tu servidor.
Este te responderá con una pequeña página HTML.

{{index "createServer function", HTTP}}

La función pasada como argumento a `createServer` ("crearServidor") es llamada
cada vez que un cliente se conecta al servidor. Las vinculaciones `solicitud` y `respuesta`
son objetos que representan los datos entrantes y salientes. El primero
contiene información acerca de la ((solicitud)), como su propiedad `url`,
lo que nos dice hacia qué URL se realizó la solicitud.

Asi que, cuando abres esa página en tu navegador, este envía una solicitud a tu
propia computadora. Esto causa que la función del servidor se ejecute y envíe una
respuesta, que luego puedes ver en el navegador.

{{index "200 (HTTP status code)", "Content-Type header", "writeHead method"}}

Para enviar algo de vuelta, llamas a los métodos en el objeto `respuesta`. El
primero, `writeHead` ("escribirCabeza"), escribirá los ((encabezado))s de respuesta
(ver [Capítulo 18](http#headers)). Le das el código de estado (200 para "OK"
en este caso) y un objeto que contiene los valores de encabezado. El ejemplo
establece el encabezado `Content-Type` para informarle al cliente que estaremos
retornando un documento HTML.

{{index "writable stream", "body (HTTP)", stream, "write method", "end method"}}

Luego, el cuerpo de la respuesta real (el documento en sí mismo) es enviado con
`respuesta.write`. Es permitido que llames a este método varias veces
si deseas enviar la respuesta pieza por pieza, por ejemplo, para transmitir
datos al cliente a medida que estos estén disponibles. Finalmente, `respuesta.end`
señala el final de la respuesta.

{{index "listen method"}}

La llamada a `servidor.listen` hace que el ((servidor)) comience a esperar por
conexiones en el ((puerto)) 8000. Esta es la razón por la cual tienes que conectarte
a _localhost:8000_ para comunicarte con este servidor, en lugar de solo
_localhost_, el cual usaría el puerto predeterminado 80.

{{index "Node.js", kill}}

Cuando ejecutas este script, el proceso simplemente se queda ahí y espera. Cuando
un script esta escuchando por eventos—en este caso, conexiones de
red—`node` no terminara automáticamente cuando llegue al final
del script. Para terminar el proceso, debes presionar [control]{keyname}-C.

Un ((servidor)) web real generalmente hace más que el del ejemplo—este
observa el metodo de la solicitud (la propiedad `method`) para ver qué
acción el cliente está intentando realizar y observa la ((URL)) de la solicitud para
averiguar en qué recurso se está realizando esta acción. Veremos un
servidor más avanzado [más adelante en este capítulo](node#file_server).

{{index "http package", "request function"}}

Para actuar como un _((cliente))_ ((HTTP)), podemos usar la función `request` ("solicitud")
en el módulo `http`.

```
const {request} = require("http");
let flujoSolicitud = request({
  hostname: "eloquentjavascript.net",
  path: "/20_node.html",
  method: "GET",
  headers: {Accept: "text/html"}
}, respuesta => {
  console.log("El servidor respondió con el código de estado",
              respuesta.statusCode);
});
flujoSolicitud.end();
```

{{index "Node.js", "callback function", "readable stream"}}

El primer argumento a `request` configura la solicitud, indicando a Node
con qué servidor hablar, qué ruta solicitar de ese servidor, cuál
método a utilizar, y así sucesivamente. El segundo argumento es la función que
debe llamarse cuando entra una respuesta. A esta se le da un objeto que
nos permite inspeccionar la respuesta, por ejemplo, para averiguar su código de estado.

{{index "GET method", "write method", "end method", "writable stream", "request function"}}

Al igual que el objeto `respuesta` que vimos en el servidor, el objeto
retornado por `request` nos permite ((transmitir)) datos hacia la
((solicitud)) con el método `write` y finalizar la solicitud con el
método `end`. El ejemplo no usa `write` ya que las solicitudes `GET`
no deben contener datos en su cuerpo de solicitud.

{{index HTTPS, "https package", "request function"}}

Hay una función similar de `request` en el módulo `https` que
se puede utilizar para realizar solicitudes a URLs que utilizen `https:`.

{{index "fetch function", "Promise class", "node-fetch package"}}

Hacer solicitudes con la funcionalidad cruda de Node es bastante verboso.
Hay paquetes disponibles en NPM para hacer esto de maneras mucho más convenientes.
Por ejemplo, `node-fetch` proporciona la interfaz `fetch` basada en promesas
que conocemos por el navegador.

## Flujos

{{index "Node.js", stream, "writable stream"}}

Hemos visto dos instancias de flujos escribibles en los ejemplos de HTTP—es
decir, el objeto de respuesta en el que el servidor podría escribir
y el objeto de solicitud que fue devuelto desde `request`.

{{index "callback function", "asynchronous programming", "write method", "end method", "Buffer class"}}

Los _flujos escribibles_ son un concepto ampliamente utilizado en Node. Tales objetos poseen
de un método `write` al que se le puede pasar un string o un objeto `Buffer` para
escribir algo hacia el flujo. Su método `end` cierra el flujo
y opcionalmente toma un valor para escribirlo en el flujo antes
del cierre. Ambos de estos métodos también pueden recibir una devolución de llamada como
un argumento adicional, al que llamarán cuando la escritura o el cierre
haya terminado.

{{index "createWriteStream function", "writeFile function"}}

Es posible crear un flujo escribible que apunte a un ((archivo))
con la función `createWriteStream` ("crearFlujoEscribible") del módulo `fs`. Tu entonces
puedes usar el método `write` en el objeto resultante para escribir en el archivo
una pieza a la vez, en lugar de hacerlo de una sola vez como se hace con `writeFile`.

{{index "createServer function", "request function", "event handling", "readable stream"}}

Los ((flujo))s legibles son un poco más complicados. Tanto la vinculación `solicitud`
que se pasó a la devolución de llamada del servidor HTTP, como la vinculación
`respuesta` que fue pasada a la devolución de llamada del cliente HTTP son flujos
legibles—un servidor lee las solicitudes y luego escribe las respuestas, mientras que
el cliente primero escribe una solicitud y luego lee una respuesta. Leer de
un flujo se hace utilizando controladores de eventos, en lugar de métodos.

{{index "on method", "addEventListener method"}}

Los objetos que emiten eventos en Node tienen un método llamado `on` que es
similar al método `addEventListener` en el navegador. Tu le das
un nombre de evento y luego una función, y registrará esa función
para que sea llamada siempre que el evento dado ocurra.

{{index "createReadStream function", "data event", "end event", "readable stream"}}

Los ((flujo))s legibles poseen los eventos `"data"` y `"end"`. El primero
se dispara cada vez que entran datos, y el segundo es llamado cada vez que
el flujo se encuentra en su final. Este modelo es el más adecuado para _transmitir_ datos que
puedan ser procesados inmediatamente, incluso cuando el documento completo no este
disponible aún. Un archivo puede ser leído como un flujo legible al usar
la función `createReadStream` ("crearFlujoLegible") de `fs`.

{{index "upcasing server example", capitalization, "toUpperCase method"}}

Este código crea un ((servidor)) que lee los cuerpos de solicitud y los transmite
de vuelta al cliente como texto en mayúsculas:

```
const {createServer} = require("http");
createServer((solicitud, respuesa) => {
  respuesa.writeHead(200, {"Content-Type": "text/plain"});
  solicitud.on("data", pedazo =>
    respuesa.write(pedazo.toString().toUpperCase()));
  solicitud.on("end", () => respuesa.end());
}).listen(8000);
```

{{index "Buffer class", "toString method"}}

El valor `pedazo` pasado al manejador de datos será un `Buffer` binario.
Podemos convertir esto en un string al decodificarlo como caracteres codificados
en UTF-8 al hacer uso de su método `toString`.

La siguiente pieza de código, cuando se ejecuta con el servidor de convertir
en mayúsculas activo, enviará una solicitud a ese servidor y escribirá la respuesta
que recibe:

```
const {request} = require("http");
request({
  hostname: "localhost",
  port: 8000,
  method: "POST"
}, respuesta => {
  respuesta.on("data", chunk =>
    process.stdout.write(chunk.toString()));
}).end("Hola servidor");
// → HOLA SERVIDOR
```

{{index "stdout property", "standard output", "writable stream", "console.log"}}

El ejemplo escribe a `process.stdout` (la salida estándar del proceso,
que es un flujo escribible) en lugar de usar `console.log`. No podemos
use `console.log` ya que este agrega un caracter de nueva linea adicional despues
de cada pieza de texto que escribe, lo que no es apropiado aquí ya que
la respuesta puede venir en múltiples pedazos.

{{id file_server}}

## Un servidor de archivos

{{index "file server example", "Node.js"}}

Vamos a combinar nuestro nuevo conocimiento acerca de como crear ((servidor))es ((HTTP))
con nuestro conocimiento acerca de como funciona el ((sistema de archivos))
para construir un puente entre los dos:
un servidor HTTP que permite ((acceso remoto)) a un sistema de archivos. Un
servidor como este tiene todo tipo de usos—permite que aplicaciones web almacenen y
compartan datos, o le puede dar a un grupo de personas acceso compartido a un montón de
archivos.

{{index [path, URL], "GET method", "PUT method", "DELETE method"}}

Cuando tratamos ((archivo))s como ((recurso))s HTTP, los métodos HTTP `GET`,
`PUT` y `DELETE` pueden ser usados para leer, escribir y eliminar archivos,
respectivamente. Interpretaremos la ruta en la solicitud como la ruta del
archivo al que la solicitud se esta intentado referir.

{{index [path, "file system"], "relative path"}}

Ya que probablemente no queremos compartir todo nuestro sistema de archivos,
interpretaremos estas rutas como los comienzos en el ((directorio))
en el cual funciona el servidor, que corresponderia al directorio en el que se inició
este. Si yo ejecutó el servidor desde `/tmp/public/`
(o `C:\tmp\public\` en Windows), entonces una solicitud por `/archivo.txt`
deberia referirse a `/tmp/public/archivo.txt` (o `C:\tmp\public\archivo.txt`).

{{index "file server example", "Node.js", "methods object", "Promise class"}}

Construiremos el programa pieza por pieza, usando un objeto llamado
`metodos` para almacenar las funciones que manejan los diversos métodos HTTP.
Los manejadores de métodos son funciones `async` que toman el objeto de
solicitud como su argumento y retornan una promesa que se resuelve en un
objeto que describe la respuesta.

```{includeCode: ">code/file_server.js"}
const {createServer} = require("http");

const metodos = Object.create(null);

createServer((solicitud, respuesta) => {
  let manejador = metodos[solicitud.method] || noPermitido;
  manejador(solicitud)
    .catch(error => {
      if (error.status != null) return error;
      return {body: String(error), status: 500};
    })
    .then(({body, status = 200, type = "text/plain"}) => {
       respuesta.writeHead(status, {"Content-Type": type});
       if (body && body.pipe) body.pipe(respuesta);
       else respuesta.end(body);
    });
}).listen(8000);

async function noPermitido(solicitud) {
  return {
    status: 405,
    body: `Metodo ${solicitud.method} no permitido.`
  };
}
```

{{index "405 (HTTP status code)"}}

Este código inicia un servidor que solo retorna respuestas de error con
el código 405, el cual es el código utilizado para indicar que el servidor
se niega a manejar un método determinado.

{{index "500 (HTTP status code)", "error handling", "error response"}}

Cuando se rechaza la promesa de un manejador de solicitud, la llamada a `catch`
traduce el error en un objeto de respuesta, si este no es un objeto aún, de manera
que el servidor pueda enviar una respuesta de error para informarle al cliente
que no pudo manejar la solicitud.

{{index "200 (HTTP status code)", "Content-Type header"}}

El campo `status` de la descripción de la respuesta puede ser omitido, en
cuyo caso se utiliza el valor predeterminado de 200 (OK). El tipo de contenido,
en la propiedad `type`, también se puede dejar fuera, en cuyo caso se asume que
la respuesta es texto simple ("text/plain" en inglés).

{{index "end method", "pipe method", stream}}

Cuando el valor de `body` es un ((flujo legible)), este tendrá un
método `pipe` que es usado para reenviar todo el contenido desde un flujo legible
a un ((flujo escribible)). Si no lo es, se asume que este es
`null` (sin cuerpo), un string o un búfer, y se pasa directamente al
método `end` de la respuesta.

{{index [path, URL], "urlToPath function", "url package", parsing, [escaping, "in URLs"], "decodeURIComponent function", "startsWith method"}}

Para averiguar cual es la ruta de archivo que le corresponde al URL de una solicitud, la
La función `rutaURL` utiliza el módulo `url` incorporado en Node para analizar
la URL. Esta toma su ruta de acceso, que será algo como `"/archivo.txt"`,
la decodifica para deshacerse de los códigos de escape al estilo `%20`, y
resuelve la ruta relativamente al directorio en donde el programa trabaja.

```{includeCode: ">code/file_server.js"}
const {parse} = require("url");
const {resolve, sep} = require("path");

const directorioBase = process.cwd();

function rutaURL(url) {
  let {pathname} = parse(url);
  let ruta = resolve(decodeURIComponent(pathname).slice(1));
  if (ruta != directorioBase &&
      !ruta.startsWith(directorioBase + sep)) {
    throw {status: 403, body: "Prohibido"};
  }
  return ruta;
}
```

Tan pronto como configuras un programa para aceptar solicitudes de red, tienes
que empezar a preocuparte por la ((seguridad)). En este caso, si no somos lo
suficientemente cuidadosos, es posible que expongamos accidentalmente
todo nuestro ((sistema de archivos)) en la red.

Las rutas de archivos son strings en Node. Para convertir tal string en un
archivo real, hay una cantidad no trivial de interpretación en marcha. Las rutas
pueden, por ejemplo, incluir `../` para referirse a un directorio padre. Asi que
una fuente obvia de problemas serían solicitudes por rutas como
`/../archivo_secreto`.

{{index "path package", "resolve function", "cwd function", "process object", "403 (HTTP status code)", "sep binding", "backslash character", "slash character"}}

Para evitar tales problemas, `rutaURL` utiliza la función `resolve` del
módulo `path`, que resuelve rutas relativas. Esta entonces verifica que
el resultado este _debajo_ del directorio de trabajo. La función `process.cwd`
(donde `cwd` significa "current working directory" o "directorio de trabajo actual")
puede ser utilizada para encontrar este directorio de trabajo. La variable `sep` del
paquete `path` es el separador de rutas en el sistema—una barra invertida ("\\") en Windows
y una barra diagonal hacia adelante ("/") en la mayoría de los otros sistemas.
Cuando la ruta no comienza con el directorio base, la función lanza un
objeto de respuesta de error, utilizando el código de estado HTTP que indica
que el acceso al recurso esta prohibido (403).

{{index "file server example", "Node.js", "GET method"}}

Configuraremos el método `GET` para retornar una lista de ((archivo))s cuando
sea lea un ((directorio)) y para retornar el contenido del archivo cuando se lea
un archivo regular.

{{index "media type", "Content-Type header", "mime package"}}

Una pregunta difícil, es decidir qué tipo de encabezado `Content-Type` deberíamos de
establecer al retornar el contenido de un archivo. Dado que los archivos
podrían ser cualquier cosa, nuestro servidor no puede simplemente devolver
el mismo tipo de contenido para todos ellos. ((NPM)) puede sernos de ayuda
nuevamente aquí. El paquete `mime` (los indicadores de tipo de contenido como
`text/plain` también son llamados _((tipos MIME))_) conoce el tipo correcto
para un gran número de ((extensiones de archivos)).

{{index "require function", "npm program"}}

El siguiente comando `npm`, en el directorio en donde el script del servidor
vive, instala una versión específica de `mime`:

```{lang: null}
$ npm install mime@2.2.0
```

{{index "404 (HTTP status code)", "stat function"}}

Cuando un archivo solicitado no existe, el código de estado HTTP correcto a
a retornar es 404. Usaremos la función `stat`, que busca
información acerca de un archivo, para averiguar si el ((archivo)) existe
y si este es un ((directorio)).

```{includeCode: ">code/file_server.js"}
const {createReadStream} = require("fs");
const {stat, readdir} = require("fs").promises;
const mime = require("mime");

metodos.GET = async function(solicitud) {
  let ruta = rutaURL(solicitud.url);
  let estatus;
  try {
    estatus = await stat(ruta);
  } catch (error) {
    if (error.code != "ENOENT") throw error;
    else return {status: 404, body: "Archivo no encontrado"};
  }
  if (estatus.isDirectory()) {
    return {body: (await readdir(ruta)).join("\n")};
  } else {
    return {body: createReadStream(ruta),
            type: mime.getType(ruta)};
  }
};
```

{{index "createReadStream function", "asynchronous programming", "error handling", "ENOENT (status code)", "Error type", inheritance}}

Dado que tiene que tocar el disco duro y, por lo tanto, podría tomar algo de tiempo,
`stat` es asíncrono. Ya que estamos usando promesas en lugar del estilo de
devolución de llamadas, esta función debe ser importada desde `promises` en
lugar de `fs` directamente.

Cuando el archivo no existe, `stat` lanzará un objeto de error con una
propiedad `code` de `"ENOENT"`. Estos códigos algo oscuros,
inspirados por ((Unix)), son la forma en que reconoces tipos de error en Node.

{{index "Stats type", "stat function", "isDirectory method"}}

El objeto `estatus` retornado por `stat` nos dice una serie de cosas
acerca de un ((archivo)), como su tamaño (propiedad `size`) y su
((fecha de modificación)) (propiedad `mtime`). Aquí estamos interesados ​​en
averiguar si este es un ((directorio)) o un archivo regular, para lo cual podemos
utilizar el método `isDirectory`.

{{index "readdir function"}}

Usamos `readdir` para leer el array de archivos en un ((directorio)) y
retornarselo al cliente. Para archivos normales, creamos un flujo legible
con `createReadStream` y devolvemos eso como el cuerpo, junto con el
tipo de contenido que el paquete `mime` nos da para el nombre del archivo.

{{index "Node.js", "file server example", "DELETE method", "rmdir function", "unlink function"}}

El código para manejar las solicitudes de `DELETE` es un poco más sencillo.

```{includeCode: ">code/file_server.js"}
const {rmdir, unlink} = require("fs").promises;

metodos.DELETE = async function(solicitud) {
  let ruta = rutaURL(solicitud.url);
  let estatus;
  try {
    estatus = await stat(ruta);
  } catch (error) {
    if (error.code != "ENOENT") throw error;
    else return {status: 204};
  }
  if (estatus.isDirectory()) await rmdir(ruta);
  else await unlink(ruta);
  return {status: 204};
};
```

{{index "204 (HTTP status code)", "body (HTTP)"}}

Cuando una ((respuesta)) ((HTTP)) no contiene ningún dato, el código de estado
204 ("sin contenido") puede ser usado para indicar esto. Dado que la respuesta
a la eliminación no necesita transmitir ninguna información más allá de
si la operación tuvo éxito, es algo razonable para retornar en este metodo.

{{index idempotence, "error response"}}

Puede que te estes preguntando por qué al intentar eliminar un archivo inexistente,
se devuelve un código de estado de éxito, en lugar de un error. Cuando el
archivo que está siendo eliminado no está allí, se podría decir que el
objetivo de la solicitud ya ha sido cumplido. El estándar ((HTTP)) nos anima a hacer
que las solicitudes sean _idempotentes_, lo que significa que hacer la misma solicitud
multiples veces produce siempre el mismo resultado que el hacerla una sola vez. En cierto sentido,
si intentas eliminar algo que ya ha sido eliminado, el efecto
que estas intentado hacer ya se ha logrado—el archivo ya no está allí.

{{index "file server example", "Node.js", "PUT method"}}

Este es el controlador para las solicitudes `PUT`:

```{includeCode: ">code/file_server.js"}
const {createWriteStream} = require("fs");

function conductoDeFlujo(desde, hacia) {
  return new Promise((resolver, rechazar) => {
    desde.on("error", rechazar);
    hacia.on("error", rechazar);
    hacia.on("finish", resolver);
    desde.pipe(hacia);
  });
}

metodos.PUT = async function(solicitud) {
  let ruta = rutaURL(solicitud.url);
  await conductoDeFlujo(solicitud, createWriteStream(ruta));
  return {status: 204};
};
```

{{index overwriting, "204 (HTTP status code)", "error event", "finish event", "createWriteStream function", "pipe method", stream}}

Esta vez no necesitaremos verificar si el archivo existe—si es así,
simplemente lo sobrescribiremos. Usamos nuevamente `pipe` para mover datos desde un
flujo legible a uno de escritura, en este caso desde la solicitud hacia
el archivo. Pero como `pipe` no está escrito para devolver una promesa, tenemos
que escribir una envoltura, `conductoDeFlujo`, que crea una promesa alrededor del
resultado de llamar a `pipe`.

{{index "error event", "finish event"}}

Cuando algo sale mal al abrir el archivo, `createWriteStream`
todavía retornara un flujo, pero ese flujo disparará un evento de `"error"`.
El flujo de salida a la solicitud también puede fallar, por ejemplo, si
la red se cae. Por lo tanto, configuramos los eventos de `"error"` de ambos
flujos para que rechazen la promesa. Cuando `pipe` finaliza, este cerrará el
flujo de salida, lo que hace que se dispare un evento de `"finish"` ("finalizar").
Ese es el punto en donde podemos resolver exitosamente la promesa (retornando nada).

{{index download, "file server example", "Node.js"}}

El script completo para el servidor está disponible en
[_https://eloquentjavascript.net/code/file_server.js_](https://eloquentjavascript.net/code/file_server.js).
Puedes descargarlo y, después de instalar sus dependencias, ejecutarlo
con Node para iniciar tu propio servidor de archivos. Y, por supuesto, puedes modificarlo
y extenderlo para resolver los ejercicios de este capítulo o simplemente para experimentar
por tu cuenta.

{{index "body (HTTP)", "curl program"}}

La herramienta de línea de comandos `curl`, ampliamente disponible en sistemas
((Unix)) (como macOS y Linux), se pueden utilizar para realizar ((peticiones))
((HTTP)). La siguiente sesión prueba brevemente nuestro servidor. La opción `-X`
se usa para establecer el ((método)) de la solicitud, y `-d` es usado para
incluir un cuerpo de solicitud.

```{lang: null}
$ curl http://localhost:8000/archivo.txt
Archivo no encontrado
$ curl -X PUT -d hola http://localhost:8000/archivo.txt
$ curl http://localhost:8000/archivo.txt
hola
$ curl -X DELETE http://localhost:8000/archivo.txt
$ curl http://localhost:8000/archivo.txt
Archivo no encontrado
```

La primera solicitud por `archivo.txt` falla ya que el archivo no existe
todavía. La solicitud `PUT` crea el archivo, y ¡sorpresa!, la siguiente solicitud
lo recupera con éxito. Después de borrarlo con una solicitud `DELETE`,
el archivo no es encontrado nuevamente.

## Resumen

{{index "Node.js"}}

Node es un buen y pequeño sistema que nos permite ejecutar JavaScript en un
contexto que no involucre navegadores. Este fue diseñado originalmente
para tareas de red para desempeñar el papel de un _nodo_ en una red.
Pero se presta bien para realizar cualquier tipo de tareas de scripting,
y si escribir JavaScript es algo que disfrutas, automatizar tareas con Node
es algo que funciona bien.

NPM proporciona paquetes para cualquier cosa que puedas imaginar (y bastantes
cosas en las que probablemente nunca habrías pensado), y te permite buscar e
instalar esos paquetes con el programa `npm`. Node viene con un
número de módulos incorporados, incluyendo el módulo `fs` para trabajar con
el sistema de archivos y el módulo `http` para ejecutar servidores HTTP y
hacer peticiones HTTP.

Todas las entradas y salidas en Node se realizan asíncronamente, a menos que
uses explícitamente una variante síncrona de una función, como
`readFileSync`. Al llamar a tales funciones asíncronas, tienes que proporcionarles
funciones de devolución de llamada, y Node las llamará con un valor de error y
(si está disponible) un resultado cuando la operación sea completada.

## Ejercicios

### Herramienta de búsqueda

{{index grep, search, "search tool (exercise)"}}

En los sistemas ((Unix)), hay una herramienta en la línea de comandos llamada `grep` que
puede ser usada para buscar archivos rápidamente utilizando una ((expresión regular)).

Escribe un script de Node que se pueda ejecutar desde la ((línea de comandos)), que funcione
similarmente a `grep`. Este script tratara su primer argumento en la línea de comandos como una
expresión regular y cualquier numero de argumentos adicionales seran tratados como archivos en donde buscar.
Deberas mostrar los nombres de cualquier archivo cuyo contenido coincida con la expresión regular.

Cuando ese script funcione, extiéndelo para que cuando uno de los argumentos sea un
((directorio)), busque a través de todos los archivos en ese directorio y sus
subdirectorios.

{{index "asynchronous programming", "synchronous programming"}}

Utiliza las funciones del sistema de archivos asíncronas o síncronas como mejor te parezca.
Configurar las cosas para que múltiples acciones asíncronas sean solicitadas
al mismo tiempo podría acelerar las cosas un poco, pero no por mucho,
debido a que la mayoría de los sistemas de archivos solo pueden leer una cosa a la vez.

{{hint

{{index "RegExp class", "search tool (exercise)"}}

Tu primer argumento en la línea de comandos, la ((expresión regular)), puede ser
encontrada en `process.argv[2]`. Los archivos de entrada vienen después de el.
Puedes usar el constructor `RegExp` para convertir un string en un objeto
de expresión regular.

{{index "readFileSync function"}}

Hacer esto de manera síncrona, con `readFileSync`, es más
directo, pero si usas `fs.promises` nuevamente para obtener
funciones que retornen promesas y escribiendo funciones `async`,
el código entre las dos maneras luce similar.

{{index "stat function", "statSync function", "isDirectory method"}}

Para averiguar si algo es un directorio, puedes usar de nuevo
`stat` (o `statSync`) y el método `isDirectory` del objeto de estatus.

{{index "readdir function", "readdirSync function"}}

Explorar un directorio es un proceso de ramificación. Puedes hacerlo ya sea
mediante el uso de una función recursiva o manteniendo un array de trabajo (archivos que
todavía tienen que ser explorados). Para encontrar los archivos en un directorio, puedes
llamar a `readdir` o `readdirSync`. El extraño uso de mayúsculas—la forma de nombrar
funciones del sistema de archivos en Node, esta basada en
funciones estándar de Unix, como `readdir`, que son todas en minúsculas,
pero luego agrega `Sync` con una letra mayúscula al comienzo de la palabra.

Para pasar de un nombre de archivo leído con `readdir` a un nombre de ruta completo,
tienes que combinarlo con el nombre del directorio, poniendo una ((barra
diagonal)) (`/`) entre ellos.

hint}}

### Creación de directorios

{{index "file server example", "directory creation (exercise)", "rmdir function"}}

Aunque el método `DELETE` en nuestro servidor de archivos es capaz de eliminar
directorios (usando `rmdir`), el servidor actualmente no proporciona ninguna
manera de _crear_ un ((directorio)).

{{index "MKCOL method", "mkdir function"}}

Añade soporte para el método `MKCOL` ("make column" o "crear columna"), que debería
de crear un directorio utilizando una llamada al metodo `mkdir` del módulo `fs`. `MKCOL` no es un
método HTTP ampliamente utilizado, pero existe para este mismo propósito en
el estándar _((WebDAV))_, que especifica un conjunto de convenciones arriba
de ((HTTP)) que lo hacen adecuado para crear documentos.

{{hint

{{index "directory creation (exercise)", "file server example", "MKCOL method", "mkdir function", idempotency, "400 (HTTP status code)"}}

```{hidden: false, includeCode: ">code/file_server.js"}
const {mkdir} = require("fs").promises;

metodos.MKCOL = async function(solicitud) {
  let ruta = rutaURL(solicitud.url);
  let estatus;
  try {
    estatus = await stat(ruta);
  } catch (error) {
    if (error.code != "ENOENT") throw error;
    await mkdir(ruta);
    return {status: 204};
  }
  if (estatus.isDirectory()) return {status: 204};
  else return {status: 400, body: "No es un directorio"};
};
```

Puedes usar la función que implementa el método `DELETE` como una base
para el método `MKCOL`. Cuando no se encuentra ningún archivo, intentamos crear
un directorio con `mkdir`. Cuando existe un directorio en esa ruta,
puedes devolver una respuesta 204 para que las solicitudes de creación de directorios
sean idempotentes. Si existe un archivo que no es un directorio aquí,
se retorna un código de error. El código 400 ("solicitud incorrecta") es apropiado
para esto.

hint}}

### Un espacio público en la web.

{{index "public space (exercise)", "file server example", "Content-Type header", website}}

Dado que el servidor de archivos puede servir cualquier tipo de archivo e incluso incluye el
encabezado `Content-Type` correcto, puedes usarlo para servir un sitio web. Ya
que le permite a cualquier persona eliminar y reemplazar archivos, sería un
tipo de sitio web interesante: uno que puede ser modificado, mejorado y
destrozado por todos los que se tomen el tiempo de crear la solicitud
HTTP correcta.

Escribe una página básica ((HTML)) que incluya un archivo de JavaScript simple.
Pon los archivos en un directorio servido por el servidor de archivos y ábrelos
en tu navegador.

A continuación, como un ejercicio avanzado o incluso como un
((proyecto de fin de semana)), combina todo el conocimiento que obtuviste
de este libro para construir una interfaz más fácil de usar, de manera
que se pueda modificar el sitio web—desde _adentro_ del sitio web.

Usa un ((formulario)) HTML para editar el contenido de los archivos que conforman el
sitio web, permitiéndole al usuario actualizarlos en el servidor utilizando
solicitudes HTTP, como aprendiste en el [Capítulo 18](http).

Comienza haciendo que un solo archivo sea editable. Entonces haz que el
usuario pueda seleccionar qué archivo editar. Utiliza el hecho de que
nuestro servidor de archivos retorna listas de archivos al leer un directorio.

{{index overwriting}}

No trabajes directamente en el código expuesto por el servidor de archivos,
ya que si cometes un error, es probable que dañes los archivos que estan allí.
En lugar de eso, manten tu trabajo afuera del directorio de acceso público y
copialo allí cuando hagas prueba.

{{hint

{{index "file server example", "textarea (HTML tag)", "fetch function", "relative path", "public space (exercise)"}}

Puedes crear un elemento `<textarea>` para mantener el contenido del archivo
que se esta editando. Una solicitud `GET`, usando `fetch`, puede recuperar el
contenido actual de un archivo. Puedes usar URLs relativas como
_index.html_, en lugar de
[_http://localhost:8000/index.html_](http://localhost:8000/index.html),
para referirte a los archivos en el mismo servidor que el script en ejecución.

{{index "form (HTML tag)", "submit event", "PUT method"}}

Entonces, cuando el usuario haga clic en un botón (puedes usar un elemento `<form>`
y el evento `"submit"`), realiza una solicitud `PUT` a la misma URL, con el
contenido del `<textarea>` como el cuerpo de la solicitud, para guardar el archivo.

{{index "select (HTML tag)", "option (HTML tag)", "change event"}}

Luego puedes agregar un elemento `<select>` que contenga todos los archivos en
el ((directorio)) superior del servidor al agregar elementos `<option>`
que contengan las líneas retornadas por una solicitud `GET` a la URL `/`. Cuando
el usuario seleccione otro archivo (un evento `"change"` en el campo), el
script debe buscar y mostrar ese archivo. Al guardar un archivo, usa el nombre
del archivo seleccionado actualmente.

hint}}
