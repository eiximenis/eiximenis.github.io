---
title: Monitorizando nuestros servicios en Kubernetes con Beatpulse
description: Monitorizando nuestros servicios en Kubernetes con Beatpulse
author: eiximenis

date: 2018-06-03T16:58:34+00:00
geeks_url: /?p=2065
geeks_ms_views:
  - 840
categories:
  - asp.net core
  - docker
  - kubernetes

---
**_Disclaimer_:** <span style="text-decoration: underline;">En ese post hablo de una librería (Beatpulse) de la que soy contribuidor</span> (lo aclaro, para que no haya ningún malentendido).
  
En todo sistema distribuído es importante disponer de un mecanismo que permita saber en todo momento **si un servicio está funcionando o no**. Es cierto que el concepto de &#8220;funcionando&#8221; es algo difuso de definir, pero yendo a mínimos deberíamos saber si un sistema se ha caído o no.
  
<!--more-->


  
**Toda imagen de Docker puede definir un _healthcheck_**, que no es nada más que un comando, que ejecutado periódicamente, indica si el servicio está funcionando correctamente o no. Docker ejecuta este comando como un proceso separado dentro del mismo contenedor. Si el comando devuelve un error, Docker considera que el contenedor &#8220;no está funcionando&#8221;.
  
Docker en sí mismo no hace nada más que marcar el estado del contenedor, pero otros sistemas más avanzados como Swarm pueden usar esa información para reiniciar aquellos contenedores una vez fallan.
  
En el _Dockerfile_ el _healthcheck_ como un comando y, lo típico (aunque no tiene porque ser así) es llamar a un _endpoint_ provisto por el propio contenedor: si ese responde correctamente se asume que el contenedor está funcionando:

<pre class="EnlighterJSRAW" data-enlighter-language="raw">HEALTHCHECK CMD curl --fail http://localhost/hc || exit 1</pre>

Ese es el ejemplo canónico de _healthcheck_ en Docker: se usa curl para navegar al endpoint (_/hc_) provisto por le contenedor. Usar curl es un mecanismo sencillo aunque viene con el precio de qué _curl_ debe existir en la imagen (las imágenes base del runtime de asp.net core vienen con curl instalado) lo que en según qué escenarios no te puedes permitir. [Aconsejo leer este post de Elton Stoneman][1] para más información al respecto.
  
Por supuesto si tu contenedor expone un _endpoint_ (_/hc_) para poder ser llamado por el _healthcheck_ ese endpoint debe estar levantado por tu API. En asp.net core eso es muy sencillo. Basta con agregar lo siguiente en el método _Configure_ de la clase _Startup _para que las peticiones a _/hc_ sean gestionadas por este pipeline (que se limita a devolver un 200):

<pre class="EnlighterJSRAW" data-enlighter-language="csharp">app.Map("/hc", hcapp =&gt;
{
    hcapp.Run(async ctx =&gt; ctx.Response.StatusCode = 200);
});</pre>

La **principal limitación de los _healthchecks_ de Docker es que solo hay uno** y el resultado es binario: o el &#8220;contenedor funciona&#8221; o no. Pero la vida real no es tan simple: imagina que tienes una API que depende de una base de datos. Si la base de datos está caída, es obvio que tu API no va a funcionar... Pero, ¿eso significa que el contenedor de tu API no funciona? Recuerda que antes he comentado que orquestradores como _Swarm_ usan esa información para reiniciar tu contenedor. Y, si el problema está en la base de datos... ¿de qué sirve reiniciar el contenedor de la API?
  
Por otro lado tener un _healthcheck_ en la API que verifique que ésta pueda conectarse con la base de datos es útil: nos permite saber qué la API está en un estado, pongamos &#8220;inusable&#8221;, por culpa de alguna de sus dependencias.
  
**Probes de Kubernetes**
  
**Kubernetes**, a diferencia de Swarm, **no usa los _healthchecks_ de Docker**. A _priori_ puede parecer un paso atrás: ¿quién mejor que la propia imagen (en el _Dockerfile_) puede saber lo qué singifica qué un contenedor &#8220;no funciona&#8221;? La razón de que Kubernetes no use los _healthchecks_ de Docker (usa el modificador _&#8211;no-healthcheck_ de &#8220;docker run&#8221; al lanzar los contenedores) es debido a qué Kubernetes ofrece su propio mecanismo, llamado _probes_. En esencia un _probe_ es lo mismo que un _healthcheck_: un comando que ejecuta sobre un contenedor y que devuelvo un estado (ok o error). Pero **Kubernetes soporta dos tipos de _probes_:**

  1. Liveness probes: Son aquellos que cuando fallan **la solución pasa por reiniciar el contenedor**. En el ejemplo anterior, sería el _healthcheck_ que hemos implementado. Si llamar a un _endpoint_ que devuelve un 200 siempre, te da un error, és que hay algo mal en el propio contenedor (quizá la API no se ha levantado). En estos casos, reiniciar es la solución.
  2. Readiness probes: Son aquellos que cuando fallan **la solución NO pasa por reiniciar, si no por &#8220;esperar&#8221;** a que se solucione el problema. Durante esta espera Kubernetes se asegura de que el contenedor (realmente el _pod_) no reciba tráfico. En el ejemplo anterior es lo que ocurre si lo que falla es la conexión a la BBDD: en este caso reiniciar la API sirve de poco. La solución es reiniciar la BBDD (si está caída) y esperar a qué la conexión se restablezca (puede ocurrir sin reiniciar la BBDD por supuesto, si era p. ej. un error de red).

Por lo tanto en Kubernetes, en un sistema con dos _pods_ uno de los cuales ejecuta una API conectada a una BBDD y el segundo ejecuta la BBDD, sería interesante definir tres probes:

  * Un liveness probe en la API (endpoint que devuelve siempre 200)
  * Un readiness probe en la API (endpoint que intenta conectarse al SQL Server y si puede devuelve un 200)
  * Un liveness probe en el _pod_ de la BBDD (que lance un comando para intentar conectarse a la BBDD).

De este modo el primer probe garantiza que la API se reiniciará si, por cualquier razón, no se levanta o se queda &#8220;bloqueada&#8221;. El segundo se asegura de que no se enrute tráfico a la API mientras la BBDD no está accesible y el tercero se encarga de reiniciar la BBDD en caso de que esta se cuelgue.
  
En Docker/Swarm los _healthchecks_ los ejecutaba el propio contenedor, mientras que en Kubernetes los ejecuta el _pod_ (realmente _kubelet_) que ejecuta el contenedor. Eso tiene la ventaja de que el _kubelet_ ya sabe hacer determinadas tareas sin necesidad de que la imagen tenga nada especial. P. ej. no necesitas tener curl en la imagen del contenedor para crear un probe que sea una llamada HTTP GET: kubelet ya sabe hacer llamadas HTTP GET. Eso permite que las imágenes puedan ser más reducidas.
  
De los tres probes mencionados antes me interesa el segundo, el readiness: un readiness probe generalmente debe validar todas las dependencias. Vale, quizá solo sea una BBDD, pero pueden ser más sistemas. Puedes depender también de un Redis, o de un storage de Azure... todas aquellas dependencias que impacten en el correcto funcionamiento de tu API deberían ser validadas en el _readiness_ probe. Por supuesto que puedes hacerlo &#8220;a mano&#8221; (de echo estás usando esas dependencias en tu API, así que los SDKs para acceder a ellos ya los estás usando), pero es código bastante repetitivo. Por eso **existen librerías para automatizarlo, y una de ellas, que quiero presentarte es [Beatpulse][2]**.
  
Hace tiempo [escribí sobre Healthchecks][3], una librería diseñada por (parte de) el equipo de ASP.NET para eso mismo, pero la realidad es que esta librería ni ha salido y sus características distan mucho de ser las necesarias. El principal problema de esa librería es que solo soporta un _healthcheck_ para tu API: suficiente para _healthchecks_ de Docker/Swarm, pero no para probes de Kubernetes.
  
Beatpulse, por otro lado se alinea con la mentalidad de Kubernetes: se expone un endpoint en tu API que, básicamente, devuelve siempre un 200 (para ser usado como _liveness_ probe) y luego otro endpoint donde puedes comprobar las dependencias. Para que no tengas que teclear mucho códgo, se incluyen muchos &#8220;verificadores&#8221; de dependencias.
  
Para empezar a usar _BeatPulse_ debes instalar el paquete de NuGet ([Install-Package Beatpulse][4]) y usar el método _UseBeatPulse_ de _IWebHostBuilder_:

<pre class="EnlighterJSRAW" data-enlighter-language="csharp">public static IWebHost BuildWebHost(string[] args) =&gt;
    WebHost.CreateDefaultBuilder(args)
        .UseBeatPulse()
        .UseStartup&lt;Startup&gt;()
        .Build();</pre>

Para que te funcione debes también llamar al método _AddBeatPulse_()_ _en _ConfigureServices_:

<pre class="EnlighterJSRAW" data-enlighter-language="csharp">public void ConfigureServices(IServiceCollection services)
{
    services.AddBeatPulse();
    services.AddMvc();
}</pre>

¡Listos! Con esto ya tienes tu _endpoint (/hc/self)_ que devuelve un 200, pensado para que lo uses como _liveness probe_ en Kubernetes.
  
El siguiente paso es crear el endpoint de _readiness probe_: imagina que dependes de un Sql Server. El primer paso es instalar el paquete [Beatpulse.SqlServer][5] que contiene el &#8220;verificador&#8221; de Sql Server. Y agregarlo a la llamada a _AddBeatPulse_:

<pre class="EnlighterJSRAW" data-enlighter-language="null">services.AddBeatPulse(opt =&gt;
{
    opt.AddSqlServer(Configuration["connectionstring"]);
});</pre>

Con esto ahora tienes lo siguiente:

  1. El endpoint /hc/self devuelve siempre 200 y es para ser usado como _liveness_ probe
  2. El endpoint /hc/sqlserver verificará solamente el Sql Server
  3. El endpoint /hc verificará todas tus dependencias (en este caso solo tenemos el Sql Serve) y es el que debes usar como _readiness _probe

A partir de ahí si tenemos más dependencias, agregaríamos su &#8220;verificador&#8221; correspondiente. Hay &#8220;verificadores&#8221; para varias de las dependencias más clásicas (y si no, el modelo es extensible para que te puedas crear las tuyas propias).
  
**La filosofía de Beatpulse es muy simple**: cada &#8220;verificador&#8221; que añadas, te añade un _endpoint_ (por debajo del endpoint raíz que por defecto es /hc) que te indica si esa dependencia (solo esa) está funcionando. Luego **el endpoint raíz (/hc) te verifica todas las dependencias** (es el que debes usar para _readiness_) y **/hc/self es un endpoint &#8220;especial&#8221; que devuelve siempre 200** pensado para _liveness_.
  
En este caso, los _probes_ que definiríamos en la _spec_ del _pod_ de Kubernetes serian (en este ejemplo uso el probe _httpGet_ que es el más habitual y que significa, como debes imaginar, realizar una llamada HTTP GET):

<pre class="EnlighterJSRAW" data-enlighter-language="json">readinessProbe:
  httpGet:
    path: /hc
    port: 80
    scheme: HTTP
  periodSeconds: 60
livenessProbe:
  httpGet:
    path: /hc/self
    port: 80
    scheme: HTTP
  periodSeconds: 60</pre>

De este modo, si se cae la conexión entre el Sql Server y nuestra API, fallará el _readiness _probe (ya que /hc ejecuta todos los &#8220;verificadores&#8221;) y Kubernetes no nos mandará tráfico al _pod_. Pero el _pod_ no se reiniciará (a no ser que también falle el /hc/self que es un _endpoint_ que, recuerda, devuelve siempre 200).
  
Luego, otra de las ventajas de usar una librería como Beatpulse para eso, son los &#8220;servicios añadidos&#8221; que esta te ofrece respecto a &#8220;hacértelo tu&#8221;. En concreto Beatpulse ofrece integraciones a través de Webhooks (para usar con Microsoft Teams o Slack), con Application Insights y tiene también servicios de valor añadido como cache, información para saber cual de los &#8220;verificadores&#8221; ha fallado e incluso... ¡una web para mostrar de forma gráfica el estado de tus verificaciones!
  
No sé, si usas ASP.NET Core en escenarios distribuídos [échale un vistazo a Beatpulse][2] y por supuesto: ¡usa las issues de Github para cualquier duda o sugerencia!

 [1]: https://blog.sixeyed.com/docker-healthchecks-why-not-to-use-curl-or-iwr/
 [2]: https://github.com/Xabaril/BeatPulse
 [3]: https://geeks.ms/etomas/2017/03/23/asp-net-comprueba-la-disponibilidad-de-tus-servicios/
 [4]: https://www.nuget.org/packages/BeatPulse/
 [5]: https://www.nuget.org/packages/BeatPulse.SqlServer/