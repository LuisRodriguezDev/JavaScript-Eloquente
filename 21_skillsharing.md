{{meta {code_links: "[\"code/skillsharing.zip\"]"}}}

# Proyecto: Sitio Web de Skill-Sharing

{{quote {author: "Margaret Fuller", chapter: true}

Si tienes conocimiento, deja que otros se iluminen con él.

quote}}

{{index "skill-sharing project", meetup, "project chapter"}}

{{figure {url: "img/chapter_picture_21.jpg", alt: "Foto de dos monociclos", chapter: "framed"}}}

Una reunión de _((intercambio de habilidades))_ ("skill-sharing" en inglés)
es un evento en donde personas con un interés compartido
se juntan y dan pequeñas presentaciones informales acerca de
cosas que conocen. En una reunión de intercambio de habilidades de ((jardinería)), alguien
podría explicar cómo cultivar ((apio)). O en un grupo de intercambio de
habilidades de programación, podrias pasar y hablarle a la gente acerca de Node.js.

{{index learning, "users' group"}}

Dichas reuniones—también a menudo llamadas _grupos de usuarios_, cuando hay
computadoras de por medio—son una excelente manera de ampliar tus horizontes, aprender acerca de nuevos desarrollos, o simplemente para conocer gente con intereses similares a los tuyos. Muchas
ciudades tienen reuniones de JavaScript. Son típicamente de libre
asistencia, y las que he visitado me han parecido bastante amables y
acogedoras.

En este capítulo final de proyecto, nuestro objetivo es construir un
((sitio web)) para administrar ((charla))s dadas en una reunión de
intercambio de habilidades. Imagina un pequeño grupo de personas que se reúnen
regularmente en la oficina de uno de los miembros para hablar de ((monociclismo)).
El organizador anterior de las reuniones se translado a otra ciudad, y
nadie quizo dar un paso adelante para convertirse en el nuevo organizador.
Queremos crear un sistema que le permita a los participantes proponer y discutir
charlas entre sí mismos, sin un organizador central.

[Al igual que en el [capítulo anterior](node), parte del código en este
capítulo está escrito para Node.js y es poco probable que funcione
si lo ejecutas directamente en la página HTML que estás mirando.]{if interactive}
El código completo para el proyecto puede ser ((descargado)) desde
[_https://eloquentjavascript.net/code/skillsharing.zip_](https://eloquentjavascript.net/code/skillsharing.zip).

## Diseño

{{index "skill-sharing project", persistence}}

En este proyecto, hay una parte de _((servidor))_, escrita para ((Node.js)),
y una parte de _((cliente))_, escrita para el ((navegador)). El servidor
almacena los datos del sistema y se los proporciona al cliente. Este también sirve
los archivos que implementan el sistema del lado del cliente.

El servidor mantiene la lista de ((charla))s propuestas para la próxima reunión,
y el cliente muestra esta lista. Cada charla tiene un nombre de presentador, un
título, un resumen y un array de ((comentarios)) asociados con ella.
El cliente permite que los usuarios propongan nuevas charlas (agregándolas a la lista),
eliminen charlas y comenten en las charlas existentes. Cada vez que un usuario haga
un cambio de este tipo, el cliente realiza una ((solicitud)) ((HTTP)) para hacerle
saber al servidor acerca de el.

{{figure {url: "img/skillsharing.png", alt: "Captura de pantalla del sitio web para compartir habilidades",width: "10cm"}}}

{{index "live view", "user experience", "pushing data", connection}}

La ((aplicación)) estara configurada para mostrar una vista _en vivo_ de las
charlas propuestas actualmente y sus comentarios. Cada vez que alguien,
en algún lugar, agregue una nueva charla o añada un comentario, todas las personas
que tengan la página abierta en sus navegadores deberían de ver inmediatamente el cambio.
Esto plantea de un pequeño desafío—no hay manera de que un servidor web
abra una conexión con un cliente, ni hay una buena manera de saber cuáles
de los clientes están actualmente viendo un sitio web determinado.

{{index "Node.js"}}

Una solución común a este problema es llamada _((long polling))_, que
resulta ser una de las motivaciones para el diseño de Node.

## Long polling

{{index firewall, notification, "long polling", network}}

Para poderle notificar inmediatamente a un cliente que algo cambió,
necesitamos una ((conexión)) con ese cliente. Dado que tradicionalmente
los ((navegadores)) web no aceptan conexiones y que los clientes a menudo
están detrás de ((enrutadors)) que bloquearían tales conexiones de cualquier
manera, hacer que el servidor inicie estas conexiones no es algo práctico.

Podemos hacer que el cliente abra la conexión y la mantenga,
de manera que el servidor pueda usarla y asi enviar información
cuando necesite hacerlo.

{{index socket}}

Pero una solicitud ((HTTP)) solo permite un simple flujo de información:
el cliente envía una solicitud, el servidor responde con solo una respuesta,
y eso es todo. Existe una tecnología llamada _((WebSockets))_,
compatible con navegadores modernos, que hace posible abrir
((conexiones)) para un intercambio de datos arbitrarios.
Pero usar esta tecnología adecuadamente es algo complicado.

En este capítulo, usamos una técnica más simple—((long polling))—en donde
los clientes continuamente le piden al servidor nueva información, usando
solicitudes HTTP regulares, y el servidor detiene su respuesta cuando no tiene
nada nuevo que reportar.

{{index "live view"}}

Siempre y cuando el cliente se asegure de tener constantemente una solicitud
de long polling abierta, este recibirá información del servidor rápidamente después de que
se vuelva disponible. Por ejemplo, si Fatima tiene nuestra aplicación de intercambio de habilidades
abierta en su navegador, ese navegador habrá realizado una solicitud
por actualizaciones nuevas y estará esperando por una respuesta a esa solicitud. Cuando Omar
envía una charla acerca de Monociclismo Extremo en Bajadas, el servidor notará
que Fatima está esperando por actualizaciones y enviara una respuesta que contenga la
nueva charla a su solicitud pendiente. El navegador de Fatima recibirá los datos
y actualizara la pantalla para mostrar la charla.

{{index robustness, timeout}}

Para evitar que las conexiones se desactiven (sean canceladas debido a una
falta de actividad), las técnicas de ((long polling)) generalmente establecen un
tiempo máximo para cada solicitud, después del cual el servidor responderá de todos modos,
aunque no tenga nada nuevo que informar, después de lo cual el cliente
iniciara una nueva solicitud. Reiniciar periódicamente la solicitud también hace
que la técnica sea más robusta, permitiendo que los clientes se recuperen de
fallas temporales de ((conexión)) o de problemas con el servidor.

{{index "Node.js"}}

Un servidor ocupado que usa long polling puede tener miles solicitudes en espera,
y por lo tanto conexiones ((TCP)), abiertas. Node, que hace que sea
fácil de gestionar muchas conexiones sin crear un hilo de control por separado
para cada una, es un buen candidato un sistema como tal.

## Interfaz HTTP

{{index "skill-sharing project"}}

Antes de comenzar a diseñar el servidor o el cliente, pensemos acerca
del punto en donde se tocan: la ((interfaz)) ((HTTP)) a traves de la cual
se comunican.

{{index [path, URL]}}

Usaremos ((JSON)) como el formato para el cuerpo de nuestras solicitudes y respuestas.
Al igual que en el servidor de archivos del [Capítulo 20](node#file_server),
intentaremos hacer un buen uso de los metodos y encabezados HTTP. La interfaz
estara centrada alrededor de la ruta `/charlas`. Las rutas que no comiencen con
`/charlas` seran utilizadas para servir ((archivos estáticos))—el código HTML y
JavaScript para el sistema del lado del cliente.

{{index "GET method"}}

Una solicitud `GET` a `/charlas` retorna un documento JSON similar a este:

```{lang: "application/json"}
[{"titulo": "Unituning",
  "presentador": "Jamal",
  "resumen": "Como modificar tu uniciclo para tener más estilo",
  "comentarios": []}]}
```

{{index "PUT method", URL}}

La creación de una nueva charla se realiza al hacer una solicitud `PUT` a una URL como
`/charlas/Unituning`, donde la parte después de la segunda barra diagonal es el título
de la charla. El cuerpo de la solicitud `PUT` debe contener un objeto ((JSON))
que contenga las propiedades `presentador` y `resumen`.

{{index "encodeURIComponent function", [escaping, "in URLs"], whitespace}}

Dado que los títulos de las charlas pueden contener espacios y otros caracteres
que no podrian aparecer normalmente en una URL, los strings de los título deben estar
codificados con la función `encodeURIComponent` al crear una URL de este tipo.

```
console.log("/charlas/" + encodeURIComponent("Tus Primeros Trucos"));
// → /charlas/Tus%20Primeros%Trucos
```

Una solicitud para crear una charla de trucos básicos podría verse similar a
esto:

```{lang: http}
PUT /charlas/Tus%20Primeros%Trucos HTTP/1.1
Content-Type: application/json
Content-Length: 92

{"presentador": "Mauricio",
 "resumen": "Ejemplos de como realizar los trucos mas básicos en uniciclo"}
```

Dichas URLs también permiten solicitudes `GET` para obtener la representación en JSON
de una charla y solicitudes `DELETE` para eliminar una charla.

{{index "POST method"}}

Añadir un ((comentario)) a una charla se hace con una solicitud `POST` a una URL
como `/charlas/Unituning/comentarios`, con un cuerpo en JSON que contenga
las propiedades `autor` y `mensaje`.

```{lang: http}
POST /charlas/Unituning/comentarios HTTP/1.1
Content-Type: application/json
Content-Length: 72

{"author": "Omar",
 "message": "¿Hablarás de como cambiar los asientos?"}
```

{{index "query string", timeout, "ETag header", "If-None-Match header"}}

Para soportar ((long polling)), las solicitudes `GET` hacia `/charlas` podrian incluir
encabezados adicionales para informarle al servidor que retrase la respuesta
si no hay datos nuevos disponibles. Usaremos un par de encabezados normalmente
destinados a gestionar el almacenamiento en caché: `ETag` y `If-None-Match`.

{{index "304 (HTTP status code)"}}

Los servidores pueden incluir un encabezado `ETag` ("entity tag" o "etiqueta de entidad" en español) en una respuesta. Su valor es un string que identifica la versión actual del recurso.
Los clientes, cuando vuelvan a solicitar ese recurso nuevamente, pueden hacer una
_((solicitud condicional))_ al incluir un encabezado `If-None-Match` cuyo
valor contiene el mismo string. Si el recurso no ha cambiado, el
el servidor responderá con el código de estado 304, que significa "no modificado",
indicandole al cliente que su versión en caché sigue siendo la mas nueva. Cuando la
etiqueta no coincide, el servidor responde de la manera normal.

{{index "Prefer header"}}

Nosotros necesitamos algo como esto, donde el cliente puede decirle al servidor
qué versión de la lista de charlas tiene, y el servidor solamente le
responde cuando esa lista ha cambiado. Pero en lugar de inmediatamente
retornar una respuesta 304, el servidor debería detener la respuesta y
retornarla solo cuando haya algo nuevo disponible o una cantidad de tiempo determinada
haya transcurrido. Para distinguir las solicitudes de long polling con
solicitudes condicionales normales, le damos otro encabezado, `Prefer: wait=90`,
que le dice al servidor que el cliente está dispuesto a esperar hasta 90
segundos por la respuesta.

El servidor mantendrá un número de versión que sera actualizado cada vez que las
charlas cambien y usara eso como el valor `ETag`. Los clientes pueden hacer
solicitudes como esta para que se les notifique cuando las charlas cambien:

```{lang: null}
GET /charlas HTTP/1.1
If-None-Match: "4"
Prefer: wait=90

(algo de tiempo pasa)

HTTP/1.1 200 OK
Content-Type: application/json
ETag: "5"
Content-Length: 295

[....]
```

{{index security}}

El protocolo descrito aquí no hace nada para ((controlar accesos)).
Todo el mundo puede comentar, modificar charlas e incluso eliminarlas. (Dado
que el Internet está lleno de ((gamberros)), poner tal sistema en línea
sin protección adicional probablemente no terminaría nada bien.)

## El servidor

{{index "skill-sharing project"}}

Comencemos por construir la parte del ((servidor)) del programa.
El código en esta sección se ejecuta en ((Node.js)).

### Enrutamiento

{{index "createServer function", [path, URL]}}

Nuestro servidor utilizará `createServer` para iniciar un servidor HTTP. En la
función que maneja una nueva solicitud, debemos de distinguir entre los
diferentes tipos de solicitudes (determinadas a traves del ((método)) y la
ruta) que soportemos. Esto se puede lograr con una larga cadena de
declaraciones `if`, pero hay una manera más agradable.

{{index dispatching}}

Un _((enrutador))_ es un componente que ayuda a enviar una solicitud a la
función que pueda manejarla. Puedes decirle al enrutador, por ejemplo,
que las solicitudes `PUT` con una ruta que coincida con la expresión regular
`/^\/charlas\/([^\/]+)$/` (`/charlas/` seguido por el título de una charla) pueden ser
manejadas por una función dada. Además, puede ayudarte a extraer los
pedazos significativos de la ruta (en este caso, el título de la charla),
envuelementoos en paréntesis en la ((expresión regular)), y pasarlos a la
función manejadora.

Existe una serie de buenos paquetes de enrutadores en ((NPM)),
pero aquí vamos a escribir uno por nosotros mismos para ilustrar el principio.

{{index "require function", "enrutador class", module}}

Esto es `enrutador.js`, al que luego le haremos `require` desde
el módulo de nuestro servidor:

```{includeCode: ">code/skillsharing/enrutador.js"}
const {parse} = require("url");

module.exports = class Enrutador {
  constructor() {
    this.rutas = [];
  }
  añadir(metodo, url, manejador) {
    this.rutas.push({metodo, url, manejador});
  }
  resolver(contexto, solicitud) {
    let ruta = parse(solicitud.url).pathname;

    for (let {metodo, url, manejador} of this.rutas) {
      let coincidencia = url.exec(ruta);
      if (!coincidencia || solicitud.metodo != metodo) continue;
      let partesURL = coincidencia.slice(1).map(decodeURIComponent);
      return manejador(contexto, ...partesURL, solicitud);
    }
    return null;
  }
};
```

{{index "enrutador class"}}

El módulo exporta la clase `Enrutador`. Un objeto enrutador permite que nuevos
manejadores sean registrados con el método `añadir` y puede resolver
solicitudes con su método `resolver`.

{{index "some method"}}

Este último devolverá una respuesta cuando encuentre un manejador, y `null`
de lo contrario. Este prueba las rutas de una en una (en el orden en el que
fueron definidas) hasta que encuentre una que coincida.

{{index "capture group", "decodeURIComponent function", [escaping, "in URLs"]}}

Las funciones manejadoras son llamadas con el valor `contexto` (que
será la instancia del servidor en nuestro caso), coincidirá strings para cualquier grupo
que estas definieron en su ((expresión regular)), y el objeto de solicitud.
Los strings deben estar decodificadas por URL, ya que la URL en bruto
puede contener codigos de estilo `%20`.

### Sirviendo archivos

Cuando una solicitud no coincida con ninguno de los tipos de solicitud definidos
en nuestro enrutador, el servidor debe interpretarla como una solicitud por un
archivo en el directorio `public`. Sería posible utilizar el servidor de archivos
definido en el [Capítulo 20](Node#file_server) para servir tales archivos,
pero nosotros no necesitamos ni queremos admitir solicitudes `PUT` y `DELETE`
en nuestros archivos, y nos gustaría tener características avanzadas tales como
soporte para almacenamiento en caché. Así que vamos a usar un servidor de
((archivos estáticos)) sólido y bien probado de ((NPM)) en su lugar.

{{index "createServer function", "ecstatic package"}}

Yo elegi usar `ecstatic`. Este no es el único servidor de este tipo en NPM, pero
funciona bien y se ajusta a nuestros propósitos. El paquete `ecstatic` exporta una
función que puede ser llamada con un objeto de configuración para producir una
función manejadora de solicitudes. Usamos la opción `root` para decirle al servidor
en donde deberia de buscar los archivos. La función manejadora acepta los parámetros
`solicitud` y `respuesta` y pueden ser pasados directamente a `createServer`
para crear un servidor que _solamente_ sirva archivos. Sin embargo, primero queremos chequear
las solicitudes que deberíamos de manejar especialmente, así que lo envolveremos
en otra función.

```{includeCode: ">code/skillsharing/skillsharing_server.js"}
const {createServer} = require("http");
const Enrutador = require("./enrutador");
const ecstatic = require("ecstatic");

const enrutador = new Enrutador();
const headersPredeterminados = {"Content-Type": "text/plain"};

class ServidorDeIntercambioDeHabilidades {
  constructor(charlas) {
    this.charlas = charlas;
    this.version = 0;
    this.enEspera = [];

    let servidorDeArchivos = ecstatic({root: "./public"});
    this.servidor = createServer((solicitud, respuesta) => {
      let resuelementoo = enrutador.resolver(this, solicitud);
      if (resuelementoo) {
        resuelementoo.catch(error => {
          if (error.status != null) return error;
          return {body: String(error), status: 500};
        }).then(({body,
                  status = 200,
                  headers = headersPredeterminados}) => {
          respuesta.writeHead(status, headers);
          respuesta.end(body);
        });
      } else {
        servidorDeArchivos(solicitud, respuesta);
      }
    });
  }
  comenzar(puerto) {
    this.servidor.listen(puerto);
  }
  detener() {
    this.servidor.close();
  }
}
```

Lo que vemos aqui utiliza una convención similar a la del servidor de archivos del
[capítulo anterior](nodo) para las respuestas—los manejadores devuelven
promesas que se resuelven en objetos que describen la respuesta.
Este envuelve el servidor en un objeto que también mantiene su estado.

### Charlas como recursos

Las ((charlas)) que han sido propuestas son almacenadas en la propiedad
`charlas` del servidor, un objeto cuyos nombres de propiedad son los
títulos de las charlas. Estos serán expuestos como ((recurso))s HTTP bajo
`/charlas/[titulos]`, por lo que necesitaremos agregar manejadores a
nuestro enrutador que implementen los diversos métodos que los
clientes pueden usar para trabajar con ellos.

{{index "GET method", "404 (HTTP status code)"}}

El controlador para las solicitudes que utilizan el metodo `GET` en una sola
charla debe buscar la charla y responder ya sea con los datos JSON de la charla
o con una respuesta de error 404.

```{includeCode: ">code/skillsharing/skillsharing_server.js"}
const rutaCharla = /^\/charlas\/([^\/]+)$/;

enrutador.añadir("GET", rutaCharla, async (servidor, titulo) => {
  if (titulo in servidor.charlas) {
    return {body: JSON.stringify(servidor.charlas[titulo]),
            headers: {"Content-Type": "application/json"}};
  } else {
    return {status: 404, body: `No se encontro ninguna charla con el titulo '${titulo}'`};
  }
});
```

{{index "DELETE method"}}

Eliminar una charla se realiza eliminándola del objeto `charlas`.

```{includeCode: ">code/skillsharing/skillsharing_server.js"}
enrutador.añadir("DELETE", rutaCharla, async (servidor, titulo) => {
  if (titulo in servidor.charlas) {
    delete servidor.charlas[titulo];
    servidor.actualizar();
  }
  return {status: 204};
});
```

{{index "long polling", "updated method"}}

El método `actualizar`, que definiremos [más adelante](skillsharing#updated),
le notifica a las solicitudes de long polling en espera acerca del cambio.

{{index "leerFlujo function", "body (HTTP)", stream}}

Para obtener el contenido de un cuerpo de solicitud, definiremos una función
llamada `leerFlujo`, que lee todo el contenido de un ((flujo legible)) y
retorna una promesa que se resuelve en un string.

```{includeCode: ">code/skillsharing/skillsharing_server.js"}
function leerFlujo(flujo) {
  return new Promise((resolver, rechazar) => {
    let datos = "";
    flujo.on("error", rechazar);
    flujo.on("datos", pedazo => datos += pedazo.toString());
    flujo.on("end", () => resolver(datos));
  });
}
```

{{index validation, input, "PUT method"}}

Un manejador que necesita de leer los cuerpos de solicitud es el manejador `PUT`,
el cual es utilizado para crear nuevas ((charlas)). Este tiene que comprobar si
los datos que se le dieron tienen las propiedades `presentador` y `resumen`, que
son strings. Cualquier información provenga desde afuera del
sistema puede ser algo sin sentido, y no queremos corromper nuestro modelo de
datos interno o que nuestro programa se ((detenga)) cuando entren
malas peticiones.

{{index "updated method"}}

Si los datos parecen ser válidos, el manejador almacena un objeto que representa
la nueva charla en el objeto `charlas`, posiblemente ((sobrescribiendo)) una
charla existente con este título, y de nuevo se llama al metodo `actualizar`.

```{includeCode: ">code/skillsharing/skillsharing_server.js"}
enrutador.añadir("PUT", rutaCharla,
           async (servidor, titulo, solicitud) => {
  let cuerpoSolicitud = await leerFlujo(solicitud);
  let charla;
  try { charla = JSON.parse(cuerpoSolicitud); }
  catch (_) { return {status: 400, body: "JSON invalido"}; }

  if (!charla ||
      typeof charla.presentador != "string" ||
      typeof charla.resumen != "string") {
    return {status: 400, body: "Datos incorrectos de charla"};
  }
  servidor.charlas[titulo] = {titulo,
                         presentador: charla.presentador,
                         resumen: charla.resumen,
                         comentarios: []};
  servidor.actualizar();
  return {status: 204};
});
```

{{index validation, "leerFlujo function"}}

Agregar un ((comentario)) a una ((charla)) funciona de manera similar. Usamos
`leerFlujo` para obtener el contenido de la solicitud, validamos los datos
resultantes, y lo almacenamos como un comentario cuando parece ser válido.

```{includeCode: ">code/skillsharing/skillsharing_server.js"}
enrutador.añadir("POST", /^\/charlas\/([^\/]+)\/comentarios$/,
           async (servidor, titulo, solicitud) => {
  let cuerpoSolicitud = await leerFlujo(solicitud);
  let comentario;
  try { comentario = JSON.parse(cuerpoSolicitud); }
  catch (_) { return {status: 400, body: "JSON invalido"}; }

  if (!comentario ||
      typeof comentario.autor != "string" ||
      typeof comentario.mensaje != "string") {
    return {status: 400, body: "Datos incorrectos de comentario"};
  } else if (titulo in servidor.charlas) {
    servidor.charlas[titulo].comentarios.push(comentario);
    servidor.actualizar();
    return {status: 204};
  } else {
    return {status: 404, body: `No se encontro ninguna charla con el titulo '${titulo}'`};
  }
});
```

{{index "404 (HTTP status code)"}}

Intentar agregar un comentario a una conversación inexistente retorna un error 404..

### Soporte para long polling

El aspecto más interesante del servidor es la parte que maneja el
((long polling)). Cuando se recibe una solicitud `GET` para `/charlas`,
esta puede ser una solicitud regular o una solicitud de long polling.

{{index "talkResponse method", "ETag header"}}

Habrán múltiples instancias en las que tendremos que enviar un array de
charlas al cliente, por lo que primero definiremos un método que nos ayude
a construir dicho array e incluya un encabezado `ETag` en la respuesta.

```{includeCode: ">code/skillsharing/skillsharing_server.js"}
ServidorDeIntercambioDeHabilidades.prototype.respuestaCharlas = function() {
  let charlas = [];
  for (let titulo of Object.keys(this.charlas)) {
    charlas.push(this.charlas[titulo]);
  }
  return {
    body: JSON.stringify(charlas),
    headers: {"Content-Type": "application/json",
              "ETag": `"${this.version}"`}
  };
};
```

{{index "query string", "url module", parsing}}

El manejador en sí mismo necesita de mirar los encabezados de solicitud
para ver si los encabezados `If-None-Match` y `Prefer` están presentes.
Node almacena los encabezados, cuyos nombres se encuentran bajo una especificación
para que no existan distinciones entre mayúsculas y minúsculas, bajo sus nombres en minúsculas.

```{includeCode: ">code/skillsharing/skillsharing_server.js"}
enrutador.añadir("GET", /^\/charlas$/, async (servidor, solicitud) => {
  let etiqueta = /"(.*)"/.exec(solicitud.encabezados["if-none-match"]);
  let espera = /\bwait=(\d+)/.exec(solicitud.encabezados["prefer"]);
  if (!etiqueta || etiqueta[1] != servidor.version) {
    return servidor.respuestaCharlas();
  } else if (!espera) {
    return {status: 304};
  } else {
    return servidor.esperarPorCambios(Number(espera[1]));
  }
});
```

{{index "long polling", "waitForChanges method", "If-None-Match header", "Prefer header"}}

Si no se dio ninguna etiqueta o si se dio una etiqueta que no coincide con la
versión actual del servidor, el manejador responde con la lista de charlas.
Si la solicitud es condicional y las charlas no cambiaron, consultamos
el encabezado `Prefer` para ver si de debemos retrasar la respuesta o responder
inmediatamente.

{{index "304 (HTTP status code)", "setTimeout function", timeout, "callback function"}}

Las funciones de devolución de llamadas para solicitudes retrasadas son almacenadas
en el array `enEspera` del servidor para que estas puedan ser notificados cuando algo suceda.
El método `esperarPorCambios` también establece inmediatamente un temporizador para responder
con un estado 304 cuando la solicitud ha esperado por una cantidad lo suficientemente larga
de tiempo.

```{includeCode: ">code/skillsharing/skillsharing_server.js"}
ServidorDeIntercambioDeHabilidades.prototype.esperarPorCambios = function(tiempo) {
  return new Promise(resolver => {
    this.enEspera.push(resolver);
    setTimeout(() => {
      if (!this.enEspera.includes(resolver)) return;
      this.enEspera = this.enEspera.filter(r => r != resolver);
      resolver({status: 304});
    }, tiempo * 1000);
  });
};
```

{{index "updated method"}}

{{id updated}}

Registrar un cambio con `actualizar` aumenta la propiedad `version`
y despierta a todas las solicitudes en espera.

```{includeCode: ">code/skillsharing/skillsharing_server.js"}
ServidorDeIntercambioDeHabilidades.prototype.actualizar = function() {
  this.version++;
  let respuesta = this.respuestaCharlas();
  this.enEspera.forEach(resolver => resolver(respuesta));
  this.enEspera = [];
};
```

Con esto concluye el código del servidor. Si creamos una instancia de
`ServidorDeIntercambioDeHabilidades` y lo iniciamos en el puerto 8000,
el servidor ((HTTP)) resultante sirve archivos del subdirectorio `public`
junto a una interfaz de gestión de charlas en la URL `/charlas`.

```{includeCode: ">code/skillsharing/skillsharing_server.js"}
new SkillShareServer(Object.create(null)).start(8000);
```

## El cliente

{{index "skill-sharing project"}}

La parte del cliente para el sitio web de compartir habilidades consiste en
tres archivos: una pequeña página de HTML, una hoja de estilos y un archivo JavaScript.

### HTML

{{index "index.html"}}

Es una convención ampliamente utilizada en los servidores web tratar de servir un archivo
llamado `index.html` cuando se realiza una solicitud directamente a una ruta que
le corresponda a un directorio. El módulo del ((servidor de archivos)) que
estamos utilizando, `ecstatic`, soporta esta convención. Cuando se realiza
una solicitud hacia la ruta `/`, el servidor busca el archivo `./public/index.html`
(`./public` siendo la raíz que le dimos) y retorna ese archivo si es encontrado.

Por lo tanto, si queremos que aparezca una página cuando un navegador apunte a nuestro
servidor, deberíamos de ponerla en `public/index.html`. Este es nuestro
archivo index.html:

```{lang: "text/html", includeCode: ">code/skillsharing/public/index.html"}
<!doctype html>
<meta charset="utf-8">
<title>Intercambio de Habilidades</title>
<link rel="stylesheet" href="intercambio_de_habilidades.css">

<h1>Intercambio de Habilidades</h1>

<script src="intercambio_de_habilidades_cliente.js"></script>
```

Este define el ((título)) del documento e incluye una ((hoja de estilos)),
que define algunos estilos para, entre otras cosas, asegurarse de que haya
algo de espacio entre las charlas.

En la parte inferior, agrega un encabezado en la parte superior de la página y
carga el script que contiene la aplicación de la parte del ((cliente)).

### Acciones

El estado de la aplicación consiste en la lista de charlas y el nombre del
usuario, y lo almacenaremos en un objeto `{charlas, usuario}`. Nosotros no
le permitimos a la interfaz de usuario que manipule directamente el estado o
que envíe solicitudes HTTP. En lugar de eso, esta puede emitir _acciones_
que describen lo que el usuario está intentando hacer.

{{index "handleaccion function"}}

La función `manejarAccion` toma una acción como tal y hace que suceda.
Dado que nuestras actualizaciones de estado son bastante simples,
los cambios de estado seran manejados en la misma funcion.

```{includeCode: ">code/skillsharing/public/skillsharing_client.js", test: no}
function manejarAccion(estado, accion) {
  if (accion.tipo == "guardarUsuario") {
    localStorage.setItem("nombreDeUsuario", accion.usuario);
    return Object.assign({}, estado, {usuario: accion.usuario});
  } else if (accion.tipo == "guardarCharlas") {
    return Object.assign({}, estado, {charlas: accion.charlas});
  } else if (accion.tipo == "nuevaCharla") {
    fetchOK(charlaURL(accion.titulo), {
      method: "PUT",
      headers: {"Content-Type": "application/json"},
      body: JSON.stringify({
        presentador: estado.usuario,
        resumen: accion.resumen
      })
    }).catch(reportarError);
  } else if (accion.tipo == "eliminarCharla") {
    fetchOK(charlaURL(accion.charla), {method: "DELETE"})
      .catch(reportarError);
  } else if (accion.tipo == "nuevoComentario") {
    fetchOK(charlaURL(accion.charla) + "/comentarios", {
      method: "POST",
      headers: {"Content-Type": "application/json"},
      body: JSON.stringify({
        autor: estado.usuario,
        mensaje: accion.mensaje
      })
    }).catch(reportarError);
  }
  return estado;
}
```

{{index "localStorage object"}}

Almacenaremos el nombre del usuario en `localStorage` para que este
pueda ser restaurado cuando la página sea cargada.

{{index "fetch function", "status property"}}

Para las acciones que necesiten involucrar al servidor para realizar solicitudes de red,
usando `fetch`, a la interfaz HTTP descrita anteriormente. Usamos una
función de envoltura, `fetchOK`, que se asegura de que la promesa retornada sea
rechazada cuando el servidor retorne un código de error.

```{includeCode: ">code/skillsharing/public/skillsharing_client.js", test: no}
function fetchOK(url, opciones) {
  return fetch(url, opciones).then(respuesta => {
    if (respuesta.status < 400) return respuesta;
    else throw new Error(respuesta.statusText);
  });
}
```

{{index "charlaURL function", "encodeURIComponent function"}}

Esta función auxiliar es usada para crear una ((URL)) para una charla
con un título dado.

```{includeCode: ">code/skillsharing/public/skillsharing_client.js", test: no}
function charlaURL(titulo) {
  return "charlas/" + encodeURIComponent(titulo);
}
```

{{index "error handling", "user experience", "reportError function"}}

Cuando la solicitud falle, no queremos que nuestra página se quede allí,
haciendo nada sin ninguna explicación. Así que definimos una función llamada
`reportarError`, que al menos le muestra al usuario un diálogo que le dice
que algo salió mal.

```{includeCode: ">code/skillsharing/public/skillsharing_client.js", test: no}
function reportarError(error) {
  alert(String(error));
}
```

### Renderizando componentes

{{index "renderUserField function"}}

Usaremos un enfoque similar al que vimos en el [Capítulo 19](paint),
en donde dividimos la aplicación en componentes. Pero ya que algunos de los
componentes nunca necesitan ser actualizados o siempre son completamente redibujados
cuando son actualizados, definiremos aquellas no como clases, sino como funciones
que retornan directamente un nodo DOM. Por ejemplo, aquí hay un componente que
muestra el campo en donde el usuario puede ingresar su nombre:

```{includeCode: ">code/skillsharing/public/skillsharing_client.js", test: no}
function renderizarCampoDeUsuario(nombre, despachar) {
  return elemento("label", {}, "Tu nombre: ", elemento("input", {
    type: "text",
    value: nombre,
    onchange(event) {
      despachar({tipo: "guardarUsuario", usuario: event.target.value});
    }
  }));
}
```

{{index "elemento function"}}

La función `elemento` utilizada para construir elementos DOM es igual a la
que utilizamos en el [Capítulo 19](paint).

```{includeCode: ">code/skillsharing/public/skillsharing_client.js", test: no, hidden: true}
function elemento(tipo, propiedades, ...hijos) {
  let dom = document.createElement(tipo);
  if (propiedades) Object.assign(dom, propiedades);
  for (let hijo of hijos) {
    if (typeof hijo != "string") dom.appendChild(hijo);
    else dom.appendChild(document.createTextNode(hijo));
  }
  return dom;
}
```

{{index "renderTalk function"}}

Una función similar es utilizada para renderizar charlas, las cuales incluyen una lista de
comentarios y un formulario para agregar nuevos ((comentarios)).

```{includeCode: ">code/skillsharing/public/skillsharing_client.js", test: no}
function renderizarCharla(charla, despachar) {
  return elemento(
    "section", {className: "charla"},
    elemento("h2", null, charla.titulo, " ", elemento("button", {
      type: "button",
      onclick() {
        despachar({tipo: "eliminarCharla", charla: charla.titulo});
      }
    }, "Eliminar")),
    elemento("div", null, "por ",
        elemento("strong", null, charla.presentador)),
    elemento("p", null, charla.resumen),
    ...charla.comentarios.map(renderizarComentario),
    elemento("form", {
      onsubmit(evento) {
        evento.preventDefault();
        let formulario = evento.target;
        despachar({tipo: "nuevoComentario",
                  charla: charla.titulo,
                  mensaje: formulario.elements.comentario.value});
        formulario.reset();
      }
    }, elemento("input", {type: "text", name: "comentario"}), " ",
       elemento("button", {type: "submit"}, "añadir comentario")));
}
```

{{index "submit event"}}

El controlador de evento de `"submit"` llama a `form.reset` para borrar el
contenido del formulario después de crear una acción `"nuevoComentario"`.

Al crear piezas del DOM moderadamente complejas, este estilo de
programación empieza a verse algo desordenado. Hay una extensión de JavaScript
(no estándar) muy utilizada llamada _((JSX))_ que te permite
escribir HTML directamente en tus scripts, lo que puede hacer que dicho código
se vea más bonito (dependiendo de lo que consideres bonito). Antes de que puedas ejecutar
dicho código, primero debes de ejecutar un programa en tu script para convertir el
pseudo-HTML en llamadas a funciones de JavaScript similares a las que estamos usando
aquí.

Los comentarios son más simples de renderizar.

```{includeCode: ">code/skillsharing/public/skillsharing_client.js", test: no}
function renderizarComentario(comentario) {
  return elemento("p", {className: "comentario"},
             elemento("strong", null, comentario.autor),
             ": ", comentario.mensaje);
}
```

{{index "form (HTML tag)", "renderTalkForm function"}}

Finalmente, el formulario que el usuario puede usar para crear una nueva charla
es renderizado de esta manera:

```{includeCode: ">code/skillsharing/public/skillsharing_client.js", test: no}
function renderizarFormularioCharla(despachar) {
  let titulo = elemento("input", {type: "text"});
  let resumen = elemento("input", {type: "text"});
  return elemento("form", {
    onsubmit(evento) {
      evento.preventDefault();
      despachar({tipo: "nuevaCharla",
                titulo: titulo.value,
                resumen: resumen.value});
      evento.target.reset();
    }
  }, elemento("h3", null, "Agregar una Charla"),
     elemento("label", null, "Titulo: ", titulo),
     elemento("label", null, "Resumen: ", resumen),
     elemento("button", {type: "submit"}, "Añadir"));
}
```

### Polling

{{index "pollTalks function", "long polling", "If-None-Match header", "Prefer header", "fetch function"}}

Para iniciar la aplicación necesitamos la lista actual de charlas.
Dado que la carga inicial está estrechamente relacionada con el proceso de
long polling—el encabezado `ETag` de la carga debe usarse al sondear—escribiremos
una función que mantenga sondeando el servidor por `/charlas` y llame a una
((función de devolución de llamada)) cuando un nuevo conjunto de charlas
este disponible.

```{includeCode: ">code/skillsharing/public/skillsharing_client.js", test: no}
async function pollingCharlas(actualizacion) {
  let etiqueta = undefined;
  for (;;) {
    let respuesta;
    try {
      respuesta = await fetchOK("/charlas", {
        headers: etiqueta && {"If-None-Match": etiqueta,
                         "Prefer": "wait=90"}
      });
    } catch (error) {
      console.log("Fallo de solicitud: " + error);
      await new Promise(resolver => setTimeout(resolver, 500));
      continue;
    }
    if (respuesta.status == 304) continue;
    etiqueta = respuesta.headers.get("ETag");
    actualizacion(await respuesta.json());
  }
}
```

{{index "async function"}}

Esta es una función `async` para que se pueda realizar un ciclo y asi esperar por la
solicitud más fácilmente. Ejecuta un ciclo infinito que, en cada iteración,
obtiene la lista de charlas—ya sea normalmente o, si esta no es la
primera solicitud, con los encabezados incluidos que la convierten en una
solicitud de long polling.

{{index "error handling", "Promise class", "setTimeout function"}}

Cuando una solicitud falla, la función espera un momento y luego intenta
otra vez. De esta manera, si tu conexión de red se cae por un tiempo y
luego vuelve, la aplicación se puede recuperar y seguir actualizando.
La promesa resuelta a través de `setTimeout` es una forma de forzar a la
función `async` a esperar.

{{index "304 (HTTP status code)", "ETag header"}}

Cuando el servidor retorna una respuesta 304, eso significa que la solicitud
de long polling ha caducado, por lo que la función debería de comenzar
inmediatamente la proxima solicitud. Si la respuesta es una respuesta 200
normal, su cuerpo es leido como JSON y pasado a la devolución de llmadada, y
el valor de su encabezado `ETag` es almacenado para la proxima iteración.

### La aplicación

{{index "SkillShareApp class"}}

El siguiente componente acopla toda la interfaz de usuario completamente:

The following component ties the whole user interface together:

```{includeCode: ">code/skillsharing/public/skillsharing_client.js", test: no}
class AplicacionIntercambioDeHabilidades {
  constructor(estado, despachar) {
    this.despachar = despachar;
    this.charlaDOM = elemento("div", {className: "charlas"});
    this.dom = elemento("div", null,
                   renderizarCampoDeUsuario(estado.usuario, despachar),
                   this.charlaDOM,
                   renderizarFormularioCharla(despachar));
    this.sincronizarEstado(estado);
  }

  sincronizarEstado(estado) {
    if (estado.charlas != this.charlas) {
      this.charlaDOM.textContent = "";
      for (let charla of estado.charlas) {
        this.charlaDOM.appendChild(
          renderizarCharla(charla, this.despachar));
      }
      this.charlas = estado.charlas;
    }
  }
}
```

{{index synchronization, "live view"}}

Cuando las charlas cambian, este componente las redibuja a todas ellas. Esto es
simple pero tambien desperdicioso. Volveremos a esto en los ejercicios.

Podemos comenzar la aplicación de esta manera:

```{includeCode: ">code/skillsharing/public/skillsharing_client.js", test: no}
function ejecutarAplicacion() {
  let usuario = localStorage.getItem("nombreDeUsuario") || "Anonimo";
  let estado, aplicacion;
  function despachar(accion) {
    estado = manejarAccion(estado, accion);
    aplicacion.sincronizarEstado(estado);
  }

  pollingCharlas(charlas => {
    if (!aplicacion) {
      estado = {usuario, charlas};
      aplicacion = new AplicacionIntercambioDeHabilidades(estado, despachar);
      document.body.appendChild(aplicacion.dom);
    } else {
      despachar({tipo: "guardarCharlas", charlas});
    }
  }).catch(reportError);
}

ejecutarAplicacion();
```

Si corres el servidor, y abres dos ventanas de navegador para
[_http://localhost:8000_](http://localhost:8000/) una al lado de la otra,
puedes ver que las acciones que realizas en una ventana son inmediatamente
visibles en la otra.

## Ejercicios

{{index "Node.js", NPM}}

Los siguientes ejercicios haran que modifiques el sistema definido en
este capitulo. Para resolverlos, asegurate de que ((descargues)) el codigo
primero
([_https://eloquentjavascript.net/code/skillsharing.zip_](https://eloquentjavascript.net/code/skillsharing.zip)),
tengas Node instalado [_https://nodejs.org_](https://nodejs.org), y hayas
instalado las dependencias del proyecto con `npm install`.

### Persistencia en el disco

{{index "data loss", persistence}}

El servidor de intercambio de habilidades mantiene sus datos netamente en memoria. Esto
significa que cuando el programa se detenga completamente o sea reiniciado por
cualquier razon, todas las charlas y comentarios seran perdidos.

{{index "hard drive"}}

Extiende el servidor de manera que este almacene los datos de las charlas en el disco y
automaticamente recargue los datos cuando sea reiniciado. No te preocupes
acerca de la eficiencia—solo haz la cosa mas sencilla que funcione.

{{hint

{{index "file system", "writeFile function", "updated method", persistence}}

La solución mas simple que me ocurrio a mi, fue codificar todo el objeto `charlas`
como ((JSON)) y puse todo eso en un archivo con `writeFile`.
Ya existe un metodo (`actualizar`) que es llamado cada vez que los datos
en el servidor cambian. Este puede ser extendido para escribir nuevos datos
en el disco.

{{index "readFile function"}}

Elige un nombre de archivo, por ejemplo `./charlas.json`. Cuando el servidor
inicie, este puede intentar leer ese archivo con `readFile`, y si tiene exito
en esa tarea, el servidor puede usar el contenido del archivo como sus datos
de inicio.

{{index prototype, "JSON.parse function"}}

Aun asi, ten cuidado. El objeto `charlas` comenzo como un objeto sin prototipo,
de manera que el operador `in` pudiera ser usado con confianza. `JSON.parse`
retornara objetos normales con `Object.prototype` como su prototipo.
Si usas JSON como el formato de tu archivo, tendras que copiar las propiedades
del objeto retornado por `JSON.parse` hacia un nuevo objeto, sin prototipo.

hint}}

### Reseteo del campo de comentario

{{index "comment field reset (exercise)", template, estado}}

Todo el asunto de redibujar las charlas funciona bastante bien porque
usualmente no puedes notar la diferencia entre un nodo del DOM y su reemplazo
identico. Pero hay excepciones. Si tu comienzas a escribir algo en el campo
de comentario de una charla en una ventana del navegador y luego, en otra,
añades un comentario a esa charla, el campo de la primera ventana sera
redibujado, removiendo tanto su contenido como su foco.

En una discusión acalorada, en donde multiples personas esten añadiendo
comentarios al mismo tiempo, esto seria fastidioso. Podrias ingeniarte una
manera de resolver esto?

{{hint

{{index "comment field reset (exercise)", template, "syncestado method"}}

La mejor manera de hacer esto probablemente sea crear objetos componentes
para las charlas, con un metodo `sincronizarEstado`, para que de esa manera
estas puedan ser actualizadas para mostrar una version modificada de la charla.
Durante el funcionamiento normal, la unica forma en la que una charla puede
ser cambiada es al añadirle mas comentarios, por lo que el metodo `sincronizarEstado`
puede ser relativamente simple.

La parte dificil es que, cuando entra una lista de charlas que ha cambiado,
tenemos que reconciliar la lista de componentes DOM existentes con las charlas
de la nueva lista—eliminando componentes cuyas charlas hayan sido borradas y
actualizando los componentes de cuyas charlas hayan cambiado.

{{index synchronization, "live view"}}

Para hacer esto, podria ser util mantener una estructura de datos que
almacene los componentes de charlas bajo los titulos de las charlas para
que facilmentes puedas descubrir si un componente existe para una charla dada.
Entonces puedes realizar un ciclo a traves del nuevo array de charlas, y para cada
una de ellas, sincronizar el componente existente o crear uno nuevo.
Para eliminar componentes de charlas eliminadas, tambien tendras que hacer un
ciclo a traves de los componentes y chequear si la charla correspondiente
aun existe.

hint}}
