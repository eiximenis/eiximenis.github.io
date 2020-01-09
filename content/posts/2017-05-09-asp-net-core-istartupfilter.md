---
title: ASP.NET Core – IStartupFilter
author: eiximenis

date: 2017-05-09T14:55:27+00:00
geeks_url: /?p=1874
geeks_ms_views:
  - 1532
categories:
  - asp.net 5
  - asp.net vNext
  - netcore

---
Buenas! Vamos a explorar en este post la interfaz **IStartupFilter**, por lo general un desconocido de ASP.NET Core, pero bueno… que está por ahí y no está de más conocerlo un poco. ¡Vamos allá!

<!--more-->


  
**Middlewares**
  
En este punto voy a asumir que todos conocemos el concepto de _middleware_ de asp.net core, ¿no? Si alguien no lo conoce, es de [lectura obligatoria este post][1] del maestro [Jose María Aguilar][2].
  
Bueno, pues eso… ahora ya sabemos que los middlewares en ASP.NET Core son los encargados de ir procesando las peticiones. La petición viaja por todos los middlewares y cada uno de ellos puede realizar distintas tareas (entre ellas modificar la petición o el contexto http) y luego o bien pasar la petición al siguiente middleware o bien cortocircuitar y devolver una respuesta… Respuesta que viaja otra vez por la cadena de middlewares (en orden inverso) donde de nuevo puede ser procesada por cada uno de ellos hasta ser enviada al navegador.
  
Veamos un ejemplo de middleware, que nos permite **cortocircuitar cualquier petición y mandar un 500** o bien no hacer nada. Esto nos permitiría de forma fácil poner un servicio “en modo de fallos” para probar distintas casuísticas. En github está el [código entero del middleware][3], por si queréis echarle un ojo. Hay tres ficheros. El primero es **FailingMiddleware.cs** que contiene el _middleware_ en sí. Este _middleware_ tiene una variable (__mustFail_) y si dicha variable value _true_, el middleware cortocircuita todas las peticiones y envía un 500:

<pre style="font-family: consolas; background: #1e1e1e; white-space: nowrap; overflow-x: scroll; color: gainsboro"><span style="color: #569cd6">public</span>&nbsp;<span style="color: #569cd6">async</span>&nbsp;<span style="color: #4ec9b0">Task</span> Invoke(<span style="color: #4ec9b0">HttpContext</span> context)<br />{<br />&nbsp;&nbsp;&nbsp; <span style="color: #569cd6">var</span> path <span style="color: #b4b4b4">=</span> context<span style="color: #b4b4b4">.</span>Request<span style="color: #b4b4b4">.</span>Path;<br />&nbsp;&nbsp;&nbsp; <span style="color: #569cd6">if</span> (path<span style="color: #b4b4b4">.</span>Equals(_options<span style="color: #b4b4b4">.</span>ConfigPath, <span style="color: #b8d7a3">StringComparison</span><span style="color: #b4b4b4">.</span>OrdinalIgnoreCase))<br />&nbsp;&nbsp;&nbsp; {<br />&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span style="color: #569cd6">await</span> ProcessConfigRequest(context);<br />&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span style="color: #569cd6">return</span>;<br />&nbsp;&nbsp;&nbsp; }<br />&nbsp;&nbsp;&nbsp; <span style="color: #569cd6">if</span> (_mustFail)<br />&nbsp;&nbsp;&nbsp; {<br />&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; context<span style="color: #b4b4b4">.</span>Response<span style="color: #b4b4b4">.</span>StatusCode <span style="color: #b4b4b4">=</span> (<span style="color: #569cd6">int</span>)System<span style="color: #b4b4b4">.</span>Net<span style="color: #b4b4b4">.</span><span style="color: #b8d7a3">HttpStatusCode</span><span style="color: #b4b4b4">.</span>InternalServerError;<br />&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; context<span style="color: #b4b4b4">.</span>Response<span style="color: #b4b4b4">.</span>ContentType <span style="color: #b4b4b4">=</span>&nbsp;<span style="color: #d69d85">"text/plain"</span>;<br />&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span style="color: #569cd6">await</span> context<span style="color: #b4b4b4">.</span>Response<span style="color: #b4b4b4">.</span>WriteAsync(<span style="color: #d69d85">"Failed due to FailingMiddleware enabled."</span>);<br />&nbsp;&nbsp;&nbsp; }<br />&nbsp;&nbsp;&nbsp; <span style="color: #569cd6">else</span><br />&nbsp;&nbsp;&nbsp; {<br />&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span style="color: #569cd6">await</span> _next<span style="color: #b4b4b4">.</span>Invoke(context);<br />&nbsp;&nbsp;&nbsp; }<br />}</pre>

Se puede ver que si __mustFail_ vale true se envía un 500 y si no, entonces se pasa la request al siguiente middleware. Por supuesto se ofrece un endpoint (en __options.ConfigPath_) que permite habilitar y deshabilitar este middleware.
  
El fichero **FailingMiddlewareAppBuilderExtensions.cs** contiene los métodos de extensión que nos permiten “enchufar” este middleware de forma fácil:

<pre style="font-family: consolas; background: #1e1e1e; white-space: nowrap; overflow-x: scroll; color: gainsboro"><span style="color: #569cd6">public</span>&nbsp;<span style="color: #569cd6">static</span>&nbsp;<span style="color: #b8d7a3">IApplicationBuilder</span> UseFailingMiddleware(<span style="color: #569cd6">this</span>&nbsp;<span style="color: #b8d7a3">IApplicationBuilder</span> builder)<br />{<br />&nbsp;&nbsp;&nbsp; <span style="color: #569cd6">return</span> UseFailingMiddleware(builder, <span style="color: #569cd6">null</span>);<br />}<br /><span style="color: #569cd6">public</span>&nbsp;<span style="color: #569cd6">static</span>&nbsp;<span style="color: #b8d7a3">IApplicationBuilder</span> UseFailingMiddleware(<span style="color: #569cd6">this</span>&nbsp;<span style="color: #b8d7a3">IApplicationBuilder</span> builder, <span style="color: #4ec9b0">Action</span>&lt;<span style="color: #4ec9b0">FailingOptions</span>&gt; action)<br />{<br />&nbsp;&nbsp;&nbsp; <span style="color: #569cd6">var</span> options <span style="color: #b4b4b4">=</span>&nbsp;<span style="color: #569cd6">new</span>&nbsp;<span style="color: #4ec9b0">FailingOptions</span>();<br />&nbsp;&nbsp;&nbsp; action<span style="color: #b4b4b4">?.</span>Invoke(options);<br />&nbsp;&nbsp;&nbsp; builder<span style="color: #b4b4b4">.</span>UseMiddleware&lt;<span style="color: #4ec9b0">FailingMiddleware</span>&gt;(options);<br />&nbsp;&nbsp;&nbsp; <span style="color: #569cd6">return</span> builder;<br />}</pre>

El último fichero es **FailingOptions.cs** que contiene la clase _FailingOptions_ que se usa para configurar el middleware&nbsp; (a través del método de extensión) de una forma como la siguiente (y indicar el endpoint para configurar el middleware):

<pre style="font-family: consolas; background: #1e1e1e; white-space: nowrap; overflow-x: scroll; color: gainsboro">app<span style="color: #b4b4b4">.</span>UseFailingMiddleware(options <span style="color: #b4b4b4">=&gt;</span><br />{<br />&nbsp;&nbsp;&nbsp; options<span style="color: #b4b4b4">.</span>ConfigPath <span style="color: #b4b4b4">=</span>&nbsp;<span style="color: #d69d85">"/FailingEndpoint"</span>;<br />});</pre>

Si enchufamos este middleware antes de cualquier otro (y lo habilitamos llamando a su endpoint) cualquier otra petición devolverá siempre un 500 (hasta que lo deshabilitemos otra vez).
  
**IStartupFilter**
  
Vale, veamos ahora que rol juega esta interfaz. Empezaremos por su declaración:

<pre style="font-family: consolas; background: #1e1e1e; white-space: nowrap; overflow-x: scroll; color: gainsboro"><span style="color: #569cd6">namespace</span> Microsoft<span style="color: #b4b4b4">.</span>AspNetCore<span style="color: #b4b4b4">.</span>Hosting<br />{<br />&nbsp;&nbsp;&nbsp; <span style="color: #569cd6">public</span>&nbsp;<span style="color: #569cd6">interface</span>&nbsp;<span style="color: #b8d7a3">IStartupFilter</span><br />&nbsp;&nbsp;&nbsp; {<br />&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span style="color: #4ec9b0">Action</span>&lt;<span style="color: #b8d7a3">IApplicationBuilder</span>&gt; Configure(<span style="color: #4ec9b0">Action</span>&lt;<span style="color: #b8d7a3">IApplicationBuilder</span>&gt; next);<br />&nbsp;&nbsp;&nbsp; }<br />}</pre>

Tiene un solo método que toma un Action<IApplicationBuilder> y devuelve un Action<IApplicationBuilder>. Vale, eso no nos dice mucho, pero la idea es la siguiente: **un filtro de startup es un elemento que nos permite hacer “algo” (lo que queramos) con el IApplicationBuilder _<u>incluso antes que se ejecute el código de Startup.</u>_**
  
Solemos pensar que el punto de entrada de una aplicación ASP.NET Core, es la clase _Startup_. Esa percepción viene seguramente de los tiempos de OWIN, donde eso era así (el código que llamaba a Startup formaba parte del framework y no lo teníamos en nuestro proyecto) pero en ASP.NET Core eso es falso. En efecto, las aplicaciones ASP.NET Core son, en el fondo, aplicaciones de consola y si buscas en tu proyecto encontrarás el fichero Program.cs:

<pre style="font-family: consolas; background: #1e1e1e; white-space: nowrap; overflow-x: scroll; color: gainsboro"><span style="color: #569cd6">public</span>&nbsp;<span style="color: #569cd6">class</span>&nbsp;<span style="color: #4ec9b0">Program</span><br />{<br />&nbsp;&nbsp;&nbsp; <span style="color: #569cd6">public</span>&nbsp;<span style="color: #569cd6">static</span>&nbsp;<span style="color: #569cd6">void</span> Main(<span style="color: #569cd6">string</span>[] args)<br />&nbsp;&nbsp;&nbsp; {<br />&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span style="color: #569cd6">var</span> host <span style="color: #b4b4b4">=</span>&nbsp;<span style="color: #569cd6">new</span>&nbsp;<span style="color: #4ec9b0">WebHostBuilder</span>()<br />&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span style="color: #b4b4b4">.</span>UseKestrel()<br />&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span style="color: #b4b4b4">.</span>UseContentRoot(<span style="color: #4ec9b0">Directory</span><span style="color: #b4b4b4">.</span>GetCurrentDirectory())<br />&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span style="color: #b4b4b4">.</span>UseIISIntegration()<br />&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span style="color: #b4b4b4">.</span>UseStartup&lt;<span style="color: #4ec9b0">Startup</span>&gt;()<br />&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span style="color: #b4b4b4">.</span>Build();<br /> <br />&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; host<span style="color: #b4b4b4">.</span>Run();<br />&nbsp;&nbsp;&nbsp; }<br />}</pre>

Este **es el verdadero punto de entrada** de una aplicación ASP.NET Core. ¿Ves el método “UseStartup”? Pues ese es el método que cede el control a la clase _Startup_ que tenemos en nuestro proyecto. **Observa que realmente, además del pipeline de middlewares que tenemos en _Startup_ existe “otro pipeline” construído sobre WebHostBuilder**. Este otro pipeline es el que establecemos usando IStartupFilter. La diferencia es que no es un pipeline de middlewares si no un pipeline de… configuraciones 🙂
  
Veamos un ejemplo de ello. Para ello vamos a crear un filtro de startup:

<pre style="font-family: consolas; background: #1e1e1e; white-space: nowrap; overflow-x: scroll; color: gainsboro"><span style="color: #569cd6">public</span>&nbsp;<span style="color: #569cd6">class</span>&nbsp;<span style="color: #4ec9b0">FailingStartupFilter</span> : <span style="color: #b8d7a3">IStartupFilter</span><br />{<br />&nbsp;&nbsp;&nbsp; <span style="color: #569cd6">private</span>&nbsp;<span style="color: #569cd6">readonly</span>&nbsp;<span style="color: #569cd6">string</span> _path;<br />&nbsp;&nbsp;&nbsp; <span style="color: #569cd6">public</span> FailingStartupFilter(<span style="color: #569cd6">string</span> path)<br />&nbsp;&nbsp;&nbsp; {<br />&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; _path <span style="color: #b4b4b4">=</span> path;<br />&nbsp;&nbsp;&nbsp; }<br /> <br />&nbsp;&nbsp;&nbsp; <span style="color: #569cd6">public</span>&nbsp;<span style="color: #4ec9b0">Action</span>&lt;<span style="color: #b8d7a3">IApplicationBuilder</span>&gt; Configure(<span style="color: #4ec9b0">Action</span>&lt;<span style="color: #b8d7a3">IApplicationBuilder</span>&gt; next)<br />&nbsp;&nbsp;&nbsp; {<br />&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span style="color: #569cd6">return</span> app <span style="color: #b4b4b4">=&gt;</span><br />&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; {<br />&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; app<span style="color: #b4b4b4">.</span>UseFailingMiddleware(options <span style="color: #b4b4b4">=&gt;</span><br />&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; {<br />&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; options<span style="color: #b4b4b4">.</span>ConfigPath <span style="color: #b4b4b4">=</span> _path;<br />&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; });<br />&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; next(app);<br />&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; };<br />&nbsp;&nbsp;&nbsp; }<br />}</pre>

Vale, podemos ver lo simple que es. Al final simplemente debemos colocar el código en el método Configure. **La clave es entender que este método Configure hace lo mismo que el método Configure de la clase Startup**. En este caso, lo que hacemos es agregar el _FailingMiddleware_ que hemos visto antes, pero podríamos agregar varios. Este es uno de los usos habituales de los filtros de startup: agregar N middlewares que van todos ellos juntos, lo que puede ser útil si desarrollas librerías.
  
Ahora nos falta ver como lo invocamos. Pues es muy sencillo, para ello creamos un método de extensión sobre IWebHostBuilder:

<pre style="font-family: consolas; background: #1e1e1e; white-space: nowrap; overflow-x: scroll; color: gainsboro"><span style="color: #569cd6">public</span>&nbsp;<span style="color: #569cd6">static</span>&nbsp;<span style="color: #569cd6">class</span>&nbsp;<span style="color: #4ec9b0">WebHostBuildertExtensions</span><br />{<br />&nbsp;&nbsp;&nbsp; <span style="color: #569cd6">public</span>&nbsp;<span style="color: #569cd6">static</span>&nbsp;<span style="color: #b8d7a3">IWebHostBuilder</span> UseFailing(<span style="color: #569cd6">this</span>&nbsp;<span style="color: #b8d7a3">IWebHostBuilder</span> builder, <span style="color: #569cd6">string</span> path)<br />&nbsp;&nbsp;&nbsp; {<br />&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; builder<span style="color: #b4b4b4">.</span>ConfigureServices(services <span style="color: #b4b4b4">=&gt;</span><br />&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; {<br />&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span style="color: #57a64a">// Registrar el propio filtro</span><br />&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; services<span style="color: #b4b4b4">.</span>AddSingleton&lt;<span style="color: #b8d7a3">IStartupFilter</span>&gt;(<span style="color: #569cd6">new</span>&nbsp;<span style="color: #4ec9b0">FailingStartupFilter</span>(path));<br />&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; });<br />&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span style="color: #569cd6">return</span> builder;<br />&nbsp;&nbsp;&nbsp; }<br /> <br />}</pre>

Básicamente registramos el filtro como singleton, con el tipo **IStartupFilter** (observa que aquí podrías registrar más elementos si quisieras). Y ahora por supuesto ya puedes añadir tu filtro en Program.cs:

<pre style="font-family: consolas; background: #1e1e1e; white-space: nowrap; overflow-x: scroll; color: gainsboro"><span style="color: #569cd6">public</span>&nbsp;<span style="color: #569cd6">static</span>&nbsp;<span style="color: #569cd6">void</span> Main(<span style="color: #569cd6">string</span>[] args)<br />{<br />&nbsp;&nbsp;&nbsp; <span style="color: #569cd6">var</span> host <span style="color: #b4b4b4">=</span>&nbsp;<span style="color: #569cd6">new</span>&nbsp;<span style="color: #4ec9b0">WebHostBuilder</span>()<br />&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span style="color: #b4b4b4">.</span>UseKestrel()<br />&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span style="color: #b4b4b4">.</span>UseFailing(<span style="color: #d69d85">"/failing"</span>)<br />&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span style="color: #b4b4b4">.</span>UseContentRoot(<span style="color: #4ec9b0">Directory</span><span style="color: #b4b4b4">.</span>GetCurrentDirectory())<br />&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span style="color: #b4b4b4">.</span>UseIISIntegration()<br />&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span style="color: #b4b4b4">.</span>UseStartup&lt;<span style="color: #4ec9b0">Startup</span>&gt;()<br />&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span style="color: #b4b4b4">.</span>UseApplicationInsights()<br />&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span style="color: #b4b4b4">.</span>Build();<br /> <br />&nbsp;&nbsp;&nbsp; host<span style="color: #b4b4b4">.</span>Run();<br />}</pre>

Observa como lo hemos añadido después del “UseKestrel”. ¿Qué conseguimos con esto? Pues **instalar el FailingMiddleware delante de cualquier otro middleware que el desarrollador ponga en su Startup**. Algo que en algunos escenarios puede ser muy interesante.
  
Si te preguntas quien es el encargado de “procesar todos los IStartupFilter” que tenemos, pues… es el método Build(). Es este método quien recoje todos los IStartupFilter y va llamando a sus métodos _Configure_ y configura de esa manera la aplicación.
  
**¿Cuando usar IStartupFilter?**
  
Pues por norma general diría que no veo muchos escenarios para hacerlo, a no ser que construyas una librería que requiera realizar ciertas tareas antes del Startup (o bien después). En aplicaciones normales no veo que sea necesario bajar a este nivel, cuando la configuración tradicional mediante Startup es más que suficiente.
  
Espero que te haya resultado interesante 😉

 [1]: http://www.variablenotfound.com/2015/12/custom-middlewares-en-aspnet-5.html
 [2]: https://twitter.com/jmaguilar
 [3]: https://github.com/dotnet-architecture/eShopOnContainers/tree/master/src/Services/Ordering/Ordering.API/Infrastructure/Middlewares