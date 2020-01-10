---
title: 'Azure Functions y SignalR: serverless push'

author: eiximenis

date: 2018-09-06T09:11:46+00:00
geeks_url: /?p=2178
geeks_ms_views:
  - 1077
categories:
  - .net
  - serverless
tags:
  - af
  - azure functions
  - signalr

---
El hecho de ofrecer SignalR como servicio PaaS en Azure y su integración con Azure Functions nos abre un escenario interesante: ahora es facilísimo hacer notificaciones _push_ desde una Azure Function (AF) a un cliente SignalR (p. ej. una Web).
  
<!--more-->


  
[SignalR][1] es una librería con bastante historia a sus espaldas. Cuando salió, su objetivo era dotar de un mecanismo de notificaciones _push_ a las aplicaciones web. La librería nos abstraía de toda la complejidad además de usar distintas técnicas según estuvieran disponibles (en navegadores y servidores modernos se usaban web sockets, pero en otros entornos se usaban otras técnicas como _long polling_). Todo era (casi) transparente para nosotros. Además SignalR ofrececía abstracciones por encima como los _Hubs_ que nos permitían fácilmente enviar notificaciones a grupos de clientes.
  
Como digo SignalR tiene ya bastante historia. La versión inicial era para ASP.NET, la que corre bajo el _Full Framework_, nada de Core. De hecho mi colega (y amigo) [José M. Aguilar][2] escribió un libro sobre ella que [todavía puedes encontrar en la tienda de Campus MVP][3].
  
Cuando salió ASP.NET Core que se portase SignalR a Core fue una de las reivindicaciones que se hicieron, pero en aquel momento... bueno, el equipo bastante tenía en intentar que Core funcionara y se estabilizara. Con Net Core 2.0 se empezó a ver la luz y [SignalR Core][4] salió de forma oficial.
  
Al cabo de relativamente poco tiempo se anunció que [Azure agregaba SignalR como servicio propio][5]. Antes tenías que crear tu propio servidor SignalR (en una aplicación web Net Core), y escalarla, que no era trivial. Y ahí es donde el servicio SignalR en Azure nos puede ayudar.
  
Puedes crear un servicio SignalR (hay una versión gratuita) desde el portal de Azure, es un proceso que te llevará un par de clicks 🙂
  
Vale, una vez lo tengas, en la pestaña &#8220;Keys&#8221; tienes todo lo que necesitas para acceder: la URL del servicio SignalR y las claves:
  
[<img class="alignnone size-large wp-image-2179" src="https://geeks.ms/etomas/wp-content/uploads/sites/154/2018/08/signalr-keys-1024x275.png" alt="Pantalla del portal mostrando las claves" width="660" height="177" />][6]
  
Ahora que tenemos un servicio de SignalR corriendo, podemos usar **el binding de Azure Functions con SignalR** para mandar mensajes desde una AF a los clientes de SignalR. Para ello, vamos a hacer:

  * Una AF que se dispare cuando se recibe un fichero en un blob
  * Dicha AF simulará un proceso asíncrono sobre dicho fichero (podría ser un análisis de visión, una transformación, una importación de datos,...) y usará SignalR para notificar cuando el proceso termina
  * Una Web que usará SignalR para recibir los mensajes de la AF

**Parte 1: La Azure Function**
  
**Nota MUY importante:** Como ya debes saber (y si no pues te informo xD) se ha desplegado **una nueva versión del motor v2 de Azure Functions**. Esa nueva versión es la 2.0.12050.0 y **es incompatible con versiones anteriores**. Eso, significa que una AF diseñada para versiones anteriores del motor no funcionará en ese... así **que ojo si tienes AF desplegadas que usen v2**, ¡si no haces nada dejarán de funcionar! [Tienes más información en esta issue][7].
  
Vamos a crear una AF usando C# y Net Core. Para ello necesitamos usar la v2 de Azure Functions (v1 no soporta Net Core). Cualquier VS2017 **con el tooling de AF actualizado** te servirá, aunque dependiendo de cuando sigas este post es posible que el tooling te genere una AF diseñada para **versiones anteriores** a la 2.0.12050.0. En **mi caso yo he usado Visual Studio 2017 15.9.0 Preview 1 con todo actualizado** y el resultado ha sido que:

  * Me ha generado una AF para una versión anterior del motor de v2
  * Al ejecutarla en local, me ha actualizado las AzureFunctionsTools a la versión 2.5.1 que usa el nuevo motor... por lo que la AF no me iba. Pero no sufras que, si te pasa, migrarlo es muy sencillo!

Vale... ¡empecemos ya! Para ello creamos un proyecto de tipo &#8220;_Azure Functions_&#8221; y nos saldrá el wizard:
  
[<img class="alignnone size-full wp-image-2180" src="https://geeks.ms/etomas/wp-content/uploads/sites/154/2018/08/new-af.png" alt="Wizard de AF" width="728" height="420" />][8]
  
Observa que tenemos marcado &#8220;Azure Funcions v2 Preview&#8221; en el selector y que hemos seleccionado &#8220;Blob Trigger&#8221; para que la AF se dispare cuando un determinado storage reciba un elemento. Finalmente entramos dos settings mas:

  1. Connection string setting: El nombre de la cadena de conexión que contiene la cadena de conexión al blob storage
  2. Path: La carpeta en el blob storage a monitorizar

Esto nos habrá generado el siguiente código:

<pre class="EnlighterJSRAW" data-enlighter-language="csharp">[FunctionName("SignalRDemoFunc")]
public static void Run([BlobTrigger("stuff/{name}", Connection = "blobconstr")]Stream myBlob, string name, ILogger log)
{
    log.LogInformation($"C# Blob trigger function Processed blob\n Name:{name} \n Size: {myBlob.Length} Bytes");
}</pre>

Observa que usamos atributos para definir el nombre de la funcion (FunctionName) y el enlace con el blob storage (BlobTrigger).
  
**¿Como saber si el código te ha generado una AF para el nuevo motor 2.0.12050.0 o no? **Es muy fácil, mira la versión del paquete Microsoft.NET.Sdk.Functions. Si es ANTERIOR a 1.0.19 (p. ej. 1.0.14) entonces el código está pensado para una versión anterior del motor. ¿**Cómo actualizarlo? Muy fácil**:

  1. Actualiza la versión de Microsoft.NET.Sdk.Functions a la 1.0.19
  2. Añade una referencia a Microsoft.Azure.WebJobs.Extensions.Storage a la versión 3.0.0-beta8
  3. Edita el fichero _host.json_ (que debe ser un json vacío) y modifícalo para que sea:

<pre class="EnlighterJSRAW" data-enlighter-language="js">{
  "version":  "2.0"
}</pre>

Con esas actualizaciones los paquetes de NuGet pasan a depender de Microsoft.Azure.Webjobs (3.0.0-beta8) que es el necesario para ese nuevo motor. Por supuesto **eso implica que cualquier paquete que añada bindings de AF debe estar preparado para esa versión de Microsoft.Azure.Webjobs** (si no... problemas al canto).
  
Ahora **debemos conectar nuestra AF al servicio SignalR, de forma que esta función pueda enviar mensajes por él**. Para ello debemos usar un paquete de NuGet llamado [AzureAdvocates.WebJobs.Extensions.SignalRService][9], que es el que contiene los _bindings_. **Actualmente este paquete está en _pre-release_ así que debes usar el modificador -Pre si usas Install-Package**. **Si usas el nuevo motor 2.0.12050.0 asegúrate de usar la versión 0.3.0-alpha** de dicho paquete. Si usas una versión anterior del motor, entonces debes usar la versión 0.2.0-alpha.
  
Ahora tenemos disponible el atributo [SignalR] para crear el binding con un Hub de SignalR y poder enviar mensajes. Así pues modificamos el código:

<pre class="EnlighterJSRAW" data-enlighter-language="csharp">[FunctionName("SignalRDemoFunc")]
public static void Run([BlobTrigger("stuff/{name}", Connection = "blobconstr")]Stream myBlob,
    [SignalR(HubName ="BlobDone", ConnectionStringSetting = "signalrconstr")] IAsyncCollector&lt;SignalRMessage&gt; sender,
    string name, ILogger log)
{
    log.LogInformation($"C# Blob trigger function Processed blob\n Name:{name} \n Size: {myBlob.Length} Bytes");
}</pre>

Observa que hemos agregado el parámetro _sender_ de tipo IAsyncCollector<SignalRMessage> y decorado el parámetro con el atributo [SignalR] indicando a que Hub de SignalR mandaremos el mensaje y el nombre del setting que contiene la cadena de conexión a SignalR. Con eso tenemos ya la conexión realizada. Ya podemos meternos con el código.
  
En nuestro caso es trivial: cuando se nos invoque directamente mandaremos un mensaje via SignalR:

<pre class="EnlighterJSRAW" data-enlighter-language="csharp">[FunctionName("SignalRDemoFunc")]
public static async Task Run([BlobTrigger("stuff/{name}", Connection = "blobconstr")]Stream myBlob,
    [SignalR(HubName = "fileprocess", ConnectionStringSetting = "AzureSignalRConnectionString")] IAsyncCollector&lt;SignalRMessage&gt; sender,
    string name, ILogger log)
{
    log.LogInformation($"C# Blob trigger function Processed blob\n Name:{name} \n Size: {myBlob.Length} Bytes");
    await sender.AddAsync(new SignalRMessage()
    {
        Target = "ProcessDone",
        Arguments = new[] { new {
            processedAt = DateTime.UtcNow,
            length = myBlob.Length,
            name
        }}
    });
    log.LogInformation($"C# Blob trigger function Ended");
}</pre>

Recuerda de añadir las entradas _signalrconstr_ y _blobconstr_ en el fichero _local.settings.json_ con las cadenas de conexión a un blob storage y la cadena de conexión a tu servicio SignalR para poder probarlo en local 🙂
  
**Nos falta un tema importante: la seguridad**. El servicio de SignalR que usamos está protegido y es necesario tener un endpoint que permita al cliente obtener un token de acceso. Afortunadamente el paquete [AzureAdvocates.WebJobs.Extensions.SignalRService][9] lo hace trivial. Añade otra Azure Function como la siguiente:

<pre class="EnlighterJSRAW" data-enlighter-language="csharp">[FunctionName(nameof(SignalRInfo))]
 public static IActionResult SignalRInfo(
     [HttpTrigger(AuthorizationLevel.Anonymous, "post")]HttpRequestMessage req,
     [SignalRConnectionInfo(HubName = "fileprocess")] SignalRConnectionInfo info, ILogger logger)
 {
     return info != null
         ? (ActionResult)new OkObjectResult(info)
         : new NotFoundObjectResult("Failed to load SignalR Info.");
 }</pre>

Esta AF usa el binding _SignalRConnectionInfo_ que se encarga automáticamente de rellenarnos el parámetro de tipo _SignalRConnectionInfo _con un token que el cliente debe usar para autenticarse. De hecho si pones en marcha dicha AF puedes verificarla fácilmente:
  
[<img class="alignnone size-full wp-image-2182" src="https://geeks.ms/etomas/wp-content/uploads/sites/154/2018/08/token-signalr.png" alt="Terminal con cURL invocando a la AF y mostrando el token de salida" width="965" height="80" />][10]
  
Por lo tanto desde el cliente deberemos llamar a esta AF para poder obtener el token 🙂
  
**Bueno, ahora vamos a por el cliente**. Podría ser una web, pero para no añadir mucha complejidad, vamos a hacer que sea una app de línea de comandos que use el cliente JavaScript de SignalR usando node. Así create una carpeta llamada &#8220;SignalrClient&#8221; y usa &#8220;npm init&#8221; para crear un package.json por defecto. Ahora instala los siguientes paquetes npm:

  * @aspnet/signalr -> Cliente JS de SignalR
  * xmlhttprequest -> Polyfill de XMLHttpRequest requerido por el cliente
  * websocket -> Polyfill de Websocket, usado por el cliente
  * axios -> Vamos a usar axios para hacer la llamada Ajax a la AF que nos da el token para conectarnos a SignalR
  * tslib -> Yo he tenido que instalarlo (se supone que no debería ser necesario), pero si no recibía un error.

Como referencia esos son los paquetes que tengo en mi _package.json:_

<pre class="EnlighterJSRAW" data-enlighter-language="json">"dependencies": {
  "@aspnet/signalr": "^1.0.3",
  "axios": "^0.18.0",
  "tslib": "^1.9.3",
  "websocket": "^1.0.26",
  "xmlhttprequest": "^1.8.0"
}</pre>

Ahora crea un fichero client.js con el siguiente código:

<pre class="EnlighterJSRAW" data-enlighter-language="js">XMLHttpRequest = require("xmlhttprequest").XMLHttpRequest;
WebSocket = require("websocket").w3cwebsocket;
const signalR = require('@aspnet/signalr');
const axios = require('axios');
const apiBaseUrl = process.env.BASE_URL || 'http://localhost:7071';</pre>

Simplemente inicializamos variables y paquetes de npm. Usamos una variable de entorno para establcer la URL base de las dos AF.
  
A continuación debemos usar axios para realizar la llamada a la AF para obtener el token:

<pre class="EnlighterJSRAW" data-enlighter-language="js">const getConnectionInfo = () =&gt; axios.post(`${apiBaseUrl}/api/SignalRInfo`)
  .then(resp =&gt; resp.data);
getConnectionInfo().then(start);
// Esperamos que se pulse una tecla para salir
console.log('Press any key to exit');
process.stdin.setRawMode(true);
process.stdin.resume();
process.stdin.on('data', process.exit.bind(process, 0));
</pre>

Una vez tenemos el token, llamamos a la función _start_, que es la que se conectará a SignalR usando la información obtenida por _getConnectionInfo_():

<pre class="EnlighterJSRAW" data-enlighter-language="js">function start(info) {
    const options = {
      accessTokenFactory: () =&gt; info.accessKey
    };
    const connection = new signalR.HubConnectionBuilder()
      .withUrl(info.endpoint, options)
      .configureLogging(signalR.LogLevel.Information)
      .build();
    connection.on('ProcessDone', ProcessDone);
    connection.onclose(() =&gt; console.log('+++ server closed +++'));
    console.log('connecting to signalr')
    connection.start().then(() =&gt; console.log('connected!')).catch(console.error);
}
function ProcessDone (data) {
    console.log('Message received!');
    console.log(data);
}</pre>

Observa como usamos _connection.on(&#8216;ProcessDone&#8217;, ProcessDone)_ para que se llame a la función ProcessDone cuando la AF envíe un mensaje cuyo _target_ sea precisamente _ProcessDone_.
  
¡Y ya hemos terminado! Pon en marcha el proyecto de AF, pon en marcha el cliente node, sube un fichero al storage y verás como el cliente se entera:
  
[<img class="alignnone size-large wp-image-2184" src="https://geeks.ms/etomas/wp-content/uploads/sites/154/2018/09/signalr-af-all-1024x314.png" alt="Cliente y AF ejecutándose" width="660" height="202" />][11]
  
**Resumiendo...**
  
Gracias al _binding_ para SignalR de las Azure Functions, ahora podemos tener una infraestructura _push_ totalmente _serverless_. Esto es muy interesante en aquellos escenarios en que nuestras notificaciones _push_ son solo para clientes típicos SignalR (usualmente una aplicación web). Para escenarios donde una AF deba notificar a otro tipo de clientes, ya usaríamos otra aproximación como p. ej. Event Grid.

 [1]: https://github.com/SignalR/SignalR
 [2]: https://twitter.com/jmaguilar
 [3]: https://www.campusmvp.es/catalogo/Product-Programaci%C3%B3n-con-ASP.NET-SignalR-2.0-(PDF)_217.aspx
 [4]: https://github.com/aspnet/SignalR
 [5]: https://azure.microsoft.com/es-es/services/signalr-service/
 [6]: https://geeks.ms/etomas/wp-content/uploads/sites/154/2018/08/signalr-keys.png
 [7]: https://github.com/Azure/app-service-announcements/issues/129
 [8]: https://geeks.ms/etomas/wp-content/uploads/sites/154/2018/08/new-af.png
 [9]: https://www.nuget.org/packages/AzureAdvocates.WebJobs.Extensions.SignalRService/0.2.0-alpha
 [10]: https://geeks.ms/etomas/wp-content/uploads/sites/154/2018/08/token-signalr.png
 [11]: https://geeks.ms/etomas/wp-content/uploads/sites/154/2018/09/signalr-af-all.png