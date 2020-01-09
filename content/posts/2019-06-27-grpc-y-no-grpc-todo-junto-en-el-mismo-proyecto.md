---
title: gRPC y "no gRPC" todo junto en el mismo proyecto
author: eiximenis

date: 2019-06-27T09:22:03+00:00
geeks_url: /?p=2378
geeks_ms_views:
  - 760
categories:
  - .net
  - asp.net core
tags:
  - grpc

---
Una de las novedades que incluye Net Core 3 es el soporte para gRPC. ¿No conoces gRPC? Bueno, pues básicamente se trata del RPC de toda la vida, pero vestido a la moda, duchado y perfumado. Vamos, si te lees los puntos principales de la [página oficial de gRPC][1] (definición de servicio independiente del lenguaje, soporte de muchos lenguajes, streaming bi-direccionales) es como si volvieses unos cuantos años atrás y [Don Box estuviese en la bañera][2] vendiéndote SOAP. Y de todos modos, cuando hablábamos de [SOAP][3] ya era difícil no acordarse de CORBA (ya fuese [el estándard][4] o [el de Microsoft][5] xD).
  
<!--more-->


  
En definitiva, que aunque parecía que REST se iba a comer el mundo (aunque en el fondo la [mayoría de los que dicen que hacen REST no hacen REST][6]), resulta que bueno... que RPC es un paradigma que funciona, así que tarde o temprano era obvio que se aparecería de nuevo. Y la iteración actual de RPC es gRPC. La g viene de Google, ya que a ellos les debemos el inicio del proyecto, aunque ahora forme parte de ese gran paraguas que es la [CNCF][7]. Y dado que la CNCF es quizá el organismo más influyente actualmente en el diseño de arquitecturas distribuídas y _cloud native_, habrá que tenerlo en cuenta.
  
A grandes rasgos gRPC es RPC + [protocol buffers][8] sobre HTTP/2.  Protocol buffers es un lenguaje ideado por Google para la serialización binaria de mensajes, de forma independiente del lenguaje, de forma que un mensaje serializado en proto buffer usando Java, lo puedo leer en C#, en Go, en Python o en cualquier otro lenguaje. Claro, esa misma independencia la tengo en JSON, pero el concepto clave aquí es &#8220;serialización binaria&#8221;. Es más o menos evidente que la serialización binaria debería ser más rápida que cualquier formato textual: para empezar muchos tipos de datos ocupan menos en su representación binaria que en su representación textual, y no es necesario añadir separadores ni nada parecido para que los humanos puedan leer y entender los mensajes.
  
La forma en como protocol buffers, consigue la independencia del lenguaje es... creando un [nuevo lenguaje][9] para definir los mensajes. Luego solo hay que crear enlaces entre este nuevo lenguaje y los lenguajes existentes y listos. Nada nuevo bajo el sol, SOAP usaba WSDL y CORBA/DCOM tenían el IDL. En fin, lo que los genios de [xkcd][10] ya dijeron hace tiempo:
  
<img class="alignnone size-large" src="https://imgs.xkcd.com/comics/standards.png" alt="Chiste de xkcd sobre los estándares" width="500" height="283" />
  
En una arquitectura distribuída, el uso de gRPC ofrece ventajas en la comunicación interna entre servicios, al ser más rápida y ocupar menos ancho de banda que un equivalente textual, tipo JSON. Pero para la comunicación hacia el mundo exterior, todavía necesitas usar algo &#8220;REST-like&#8221; ya que la mayoría de clientes no entiende de gRPC (por el momento claro, en un futuro ya se verá). Por suerte [la traducción entre gRPC y JSON es estándard][11], lo que ha favorecido el uso de tooling de herramientas que auto-traducen entre gRPC y HTTP/JSON, como el [grpc gateway][12] o [Envoy][13] entre otras.
  
**gRPC y Net Core 3**
  
La documentación oficial de Microsoft sobre gRPC es bastante escasa, aunque da para indicarnos [como crear un servicio y como crear un cliente][14]. Desafortunadamente esa documentación pasa por alto algunos conceptos clave y si **pretendes mezclar gRPC y &#8220;no gRPC&#8221; (p. ej. un controlador MVC) en un mismo proyecto, **empezaráa a tener problemas.
  
Voy a asumir **que te has leído la documentación oficial de Microsoft y de que tienes el GreeterService creado y el cliente funcionando**. Y si la documentación de Microsoft te parece compleja, siempre puedes leer [este post de mi colega Jorge, quien viene a contar lo mismo, pero magistralmente][15].
  
Vamos a modificar el servidor, para añadir un endpoint (&#8220;/test&#8221;) que nos devuelva cualquier cosa. Como agregar un controlador de MVC da pereza y han &#8220;expressizado&#8221; un poco más a asp.net core, pues usamos el nuevo MapGet:

<pre class="EnlighterJSRAW" data-enlighter-language="csharp">app.UseEndpoints(endpoints =&gt;
{
    endpoints.MapGet("/test", async ctx =&gt;
    {
        ctx.Response.ContentType = "text/plain";
        await ctx.Response.WriteAsync("I am not gRPC!");
    });
    endpoints.MapGrpcService&lt;GreeterService&gt;();
});</pre>

Perfecto, ahora este servidor nos expone el endpoint (&#8220;/test&#8221;) que devuelve una cadena y expone nuestro amigo, el GreeterService via gRPC.
  
Vale, ahora ponemos en marcha el servidor (con dotnet run, vamos a usar kestrel, no IIS Express, aunque si has usado la plantilla de VS2019 no hay IIS Express disponible) y navegamos a &#8220;/test&#8221; usando un navegador:
  
[<img class="alignnone size-full wp-image-2379" src="https://geeks.ms/etomas/wp-content/uploads/sites/154/2019/06/error-no-grpc.png" alt="Error en el navegador y excepción en el servidor (HTTP/2 connection error)" width="953" height="680" />][16]
  
Ups! Tu servidor gRPC **no es capaz de servir ningún endpoint que no sea gRPC**. Lo mismo ocurriría en un controlador MVC o una Razor Page. Vale, técnicamente eso no es cierto, **lo que ocurre es que cuando usamos la plantilla de gRPC se nos configura Kestrel para usar solo HTTP/2**. Ya, quizá estás pensando, y con razón, que donde está el problema, ya que estoy usando Chrome y no IE10 y se supone que Chrome soporta HTTP/2 ¿verdad?
  
Cierto, pero la realidad es que **hay dos maneras en las que un cliente se puede conectar a un servidor HTTP/2**. La que conocemos como _prior knowledge_ y la, bueno, la normal. La diferencia está en que usando _prior knowledge_ nos conectamos directamente al servidor usando HTTP/2, porque sabemos que dicho servidor soporta HTTP/2. Por otro lado la forma digamos más estándard, es conectarse al servidor usando HTTP1.1 y se establece una negociación entre cliente y servidor, en la cual si el servidor soporta HTTP/2 se lo indica al cliente y este si quiere y puede, pasa a usarlo. Los **navegadores no soportan _prior knowledge_, así que por eso fallaba la petición usando Chrome**. Chrome se conectaba usando HTTP1.1, pero Kestrel solo admite HTTP/2.
  
Pero, para que veas que realmente sí que Kestrel sirve contenido no-gRPC, puedes usar cURL:
  
[<img class="alignnone size-full wp-image-2380" src="https://geeks.ms/etomas/wp-content/uploads/sites/154/2019/06/curl-prior-knowledge.png" alt="Usar curl con la opción --http2-prior-knowledge nos permite llamar al endpoint /test" width="744" height="146" />][17]
  
Como puedes observar la primera llamada falla (y en el servidor aparece la excepción otra vez), pero la segunda llamada donde se fuerza el uso de HTTP/2 con el modificador _&#8211;http2-prior-knowledge_ funciona correctamente.
  
Así pues recuerda: **El template de VS2019 de gRPC configura Kestrel solo para HTTP/2 lo que te impide acceder a recursos no-gRPC usando un navegador** (o cualquier otro cliente que no soporte prior knowledge).
  
Donde está esa configuración? Pues el template la mete en el _appsettings.json_:

<pre class="EnlighterJSRAW" data-enlighter-language="json">"Kestrel": {
  "EndpointDefaults": {
    "Protocols": "Http2"
  }
}</pre>

El [valor de Protocols, es un enumerado][18] y lo bueno es que hay una opción llamada Http1AndHttp2. ¿Qué puede salir mal con esa opción? Veamoslo... Modifica el _appsettings_ y lanza el servidor otra vez. Si ahora usas cURL verás como puedes llamar sin problemas el endpoint /test. Pero... ¡sorprendentemente ahora falla cuando usas el _&#8211;http2-prior-knowledge_!:
  
[<img class="alignnone size-full wp-image-2381" src="https://geeks.ms/etomas/wp-content/uploads/sites/154/2019/06/curl-http2-error.png" alt="Ahora curl da error al usar --http2-prior-knowledge y aparece una excepción en el servidor (Unrecognized HTTP version: 'HTTP/2.0)" width="977" height="371" />][19]
  
Y observa como el servidor peta con un &#8220;_Unrecognized HTTP version: &#8216;HTTP/2.0_&#8220;. Por otro lado si usas Chrome y navegas a /test, usando las &#8220;Developer Tools&#8221; verás que se accede al recurso usando &#8220;http1.1&#8221; en lugar de &#8220;http2&#8221;:
  
[<img class="alignnone size-full wp-image-2382" src="https://geeks.ms/etomas/wp-content/uploads/sites/154/2019/06/chrome-protocol-http1.png" alt="Chrome mostrando en la pestaña network como el protocolo es http1.1" width="971" height="488" />][20]
  
¿Y el cliente de gRPC? Bueno... igual ya te lo imaginas, pero... da un error. Habilitar &#8220;Http1AndHttp2&#8221; no es la solución, ya que intentar acceder directamente usando HTTP/2 no funciona. Vale, miento de nuevo 😛 Realmente **usar Http1AndHttp2 es _parte_ de la solución**, pero por si solo no basta. Y la razón se llama ALPN.
  
Hay &#8220;dos variantes&#8221; de como saltar de HTTP1 a HTTP/2. La primera, la más usada, es conocida como h2 y se trata de usar HTTP/2 **bajo HTTPS y toda esa negociación que he comentado antes se realiza bajo TLS** y recibe el nombre de [ALPN][21]. ALPN es una extensión a TLS que permite negociar protocolos de forma segura y es la opción estándard para &#8220;dar el salto a HTTP/2&#8221;. Es posible usar HTTP/2 sin ALPN y por lo tanto sin TLS y se conoce bajo el nombre de h2c, pero **no hay navegador que lo soporte**. Por lo que se ve hay varios problemas técnicos en h2c y no parece que se esté usando mucho.
  
Así que ya sabes: **Chrome se conectaba usando http1.1 porque al no haber TLS en el servidor no podía establecer una negociación con ALPN.**
  
Y por qué curl o el cliente gRPC, que usan HTTP/2 directamente fallan? Pues muy fácil. **Cuando usamos Http1AndHttp2, Kestrel espera siempre que se use ALPN**. Por lo tanto cuando usamos _prior knwoledge_ con curl falla. El cliente gRPC de netcore es otra historia: si detecta TLS intenta ALPN y en caso contrario usa _prior knowledge_. En este caso, al no detectar TLS (es un endpoint HTTP), intenta usar HTTP/2 directamente, pero Kestrel espera ALPN y por eso falla.
  
**Qué soluciones tengo, entonces?**
  
Recapitulemos: Si configuras kestrel para usar sólo HTTP/2, entonces solo te funcionan los clientes que usen _prior knowledge_, como el cliente de gRPC pero descartamos a los navegadores. Si usas Http1AndHttp2 como protocolos soportados, entonces Kestrel fuerza el uso de ALPN. Eso permite que funcionen tanto clientes que soporten HTTP1.1 como HTTP/2 pero no se permite el uso de _prior knowledge_, debe existir siempre la negociación. Y debe ser usando ALPN, es decir bajo TLS.
  
Por lo tanto, para poder dar soporte a todo tipo de clientes tienes **tres soluciones**:

  1. Usar **un solo endpoint con soporte para TLS**. En este caso, debes usar https en este endpoint y Http1AndHttp2 como protocolos. Esto soporta cualquier cliente HTTP/2 que use ALPN (es decir (casi absolutamente) todos). Si usas _prior knowledge_ fallará, pero vamos, casi no hay (apenas ningún) cliente que lo use. **En este escenario, con un solo endpoint soportas cualquier cliente**. Pero requiere el uso de TLS y por lo tanto de un certificado válido.
  2. Usar **dos endpoints (ambos sin TLS)**: 
      1. Uno **sin TLS y configurado como HTTP2**: Los clientes gRPC pueden conectarse a este endpoint y llamar a los servicios gRPC. También un cliente que use _prior knowledge_ puede usar este endpoint para cualquier tipo de recurso.
      2. Otro **sin TLS y configurado como Http1AndHttp2**. Los clientes que no soportan _prior knowledge_ (navegadores p. ej.) deben usar este endpoint. **Al no haber TLS no habrá ALPN y por lo tanto estos clientes usarán HTTP1.1**.
  3. Usar **dos endpoints** **uno con TLS y el otro sin**: 
      1. Uno **sin TLS y configurado como HTTP2**: Los clientes gRPC pueden conectarse a este endpoint y llamar a los servicios gRPC. También los clientes que no soporten ALPN y usen _prior knowledge_ deben usar este endpoint para cualquier tipo de recurso.
      2. Otro **con TLS y configurado como Http1AndHttp2**, para ser usado por los clientes que no soporten _prior knowledge_. Al haber TLS existirá ALPN y los clientes que soporten HTTP/2 lo usarán (los que no, pues usarán HTTP1).

Por lo tanto dependiendo de tus necesidades deberás usar una opción u otra:

  * Si no quieres/puedes usar TLS, entonces debes optar por la opción 2, aunque eso impide el uso de HTTP/2 a clientes que no soporten _prior knowledge._
  * Si puedes usar TLS, entonces puedes optar por la primera opción (un solo endpoint) a no ser que por algún motivo quieras soportar clientes con _prior knowledge_ (en este caso deberás optar por la opción 3).

Para definir esos endpoints debes usar el método **ConfigureKestrel**:

<pre class="EnlighterJSRAW" data-enlighter-language="csharp">private static IHostBuilder CreateHostBuilder(IConfiguration configuration, string[] args) =&gt;
     Host.CreateDefaultBuilder(args)
         .ConfigureWebHostDefaults(builder =&gt;
         {
             builder.ConfigureKestrel(options =&gt;
             {
                     options.ListenAnyIP(5000, listenOptions =&gt;
                     {
                             listenOptions.UseHttps("my-cert.pfx");    // Habilitamos TLS usando el certificado indicado
                             listenOptions.Protocols = HttpProtocols.Http1AndHttp2;   // Protocolos Http1AndHttp2
                     });
                     options.ListenAnyIP(5001, listenOptions =&gt;
                     {
                             listenOptions.Protocols = HttpProtocols.Http2;   // Solo HTTP/2 para prior knwoledge. No hay https aquí.
                     });
                 });
             });</pre>

(En este caso, ya que configuras por código puedes quitar la sección &#8220;Kestrel&#8221; del fichero _appsettings.json_).
  
Si usas TLS en desarrollo con un certificado auto-firmado, entonces debes configurar los clientes para que &#8220;acepten&#8221; dicho certificado. Para los navegadores puedes aceptar todos los avisos de seguridad. Para el cliente gRPC puedes indicarle que no valide el certificado:

<pre class="EnlighterJSRAW" data-enlighter-language="csharp">var handler = new HttpClientHandler();
handler.ClientCertificateOptions = ClientCertificateOption.Manual;
handler.ServerCertificateCustomValidationCallback =
    (httpRequestMessage, cert, cetChain, policyErrors) =&gt;
    {
        return true;
    };
var httpClient = new HttpClient(handler);
httpClient.BaseAddress = new Uri(_grpcServerUrl);
var client = GrpcClient.Create&lt;GreeterClient&gt;(httpClient);
</pre>

Y listos! Esto es todo 🙂 Espero que te haya sido útil y ya sabes: no hay problema alguno para combinar gRPC y MVC o Razor Pages o lo que sea, en un mismo servidor. Al final, todo es HTTP 😉

 [1]: https://grpc.io/
 [2]: https://blog.mattmags.com/2011/05/19/don-box-the-bathtub-lecture/
 [3]: https://es.wikipedia.org/wiki/Simple_Object_Access_Protocol
 [4]: https://es.wikipedia.org/wiki/CORBA
 [5]: https://es.wikipedia.org/wiki/Modelo_de_Objetos_de_Componentes_Distribuidos
 [6]: https://aboullaite.me/http-rest-apis/
 [7]: https://www.cncf.io/
 [8]: https://developers.google.com/protocol-buffers/
 [9]: https://developers.google.com/protocol-buffers/docs/proto3
 [10]: https://xkcd.com
 [11]: https://developers.google.com/protocol-buffers/docs/proto3#json
 [12]: https://github.com/grpc-ecosystem/grpc-gateway
 [13]: https://www.envoyproxy.io/docs/envoy/latest/configuration/http_filters/grpc_json_transcoder_filter
 [14]: https://docs.microsoft.com/en-us/aspnet/core/tutorials/grpc/grpc-start?view=aspnetcore-3.0&tabs=visual-studio
 [15]: https://geeks.ms/jorge/2019/06/25/grpc-hello-world/
 [16]: https://geeks.ms/etomas/wp-content/uploads/sites/154/2019/06/error-no-grpc.png
 [17]: https://geeks.ms/etomas/wp-content/uploads/sites/154/2019/06/curl-prior-knowledge.png
 [18]: https://docs.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.server.kestrel.core.httpprotocols
 [19]: https://geeks.ms/etomas/wp-content/uploads/sites/154/2019/06/curl-http2-error.png
 [20]: https://geeks.ms/etomas/wp-content/uploads/sites/154/2019/06/chrome-protocol-http1.png
 [21]: https://en.wikipedia.org/wiki/Application-Layer_Protocol_Negotiation