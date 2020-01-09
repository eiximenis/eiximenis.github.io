---
title: ASP.NET–Comprueba la disponibilidad de tus servicios
author: eiximenis

date: 2017-03-23T15:51:42+00:00
geeks_url: /?p=1869
geeks_ms_views:
  - 1585
categories:
  - netcore

---
Si desarrollas una aplicación en ASP.NET y/o ASP.NET Core, te puede interesar una nueva librería que ha sacado el equipo de .NET: [HealthChecks][1]. Esa librería (muy sencilla) contiene lo que necesitas para poder validar que un determinado recurso externo (SQL Server, API remota, etc) está funcionando y también para que decidas lo que significa que un recurso “está funcionando”.

<!--more-->

Actualmente la librería está en un estado “alfa” por decirlo de algún modo, así que no hay paquete NuGet todavía (ni readme.md xD) pero es muy sencilla de entender. De todos modos veamos un ejemplo muy sencillo. Para ello vamos a crear una solución (asp.net core) con dos proyectos:

  1. Una aplicación MVC Core que contiene el frontend 
      * Una aplicación MVC Core que nos representa una API externa y que usa un SQL Server.</ol> 
    El objetivo es el siguiente:
    
      * Que la aplicación web tenga una página para ver el estado de los servicios externos (en este caso la API) 
          * La API funciona si es accesible via HTTP y además si el SQL está levantado.</ul> 
        Veamos como esa librería nos puede ayudar a construir ese ejemplo. Lo primero es crear la solución con dos aplicaciones MVC Core. En mi caso he creado una solución con dos proyectos MVC Core (WebClient&nbsp; usando la plantilla de aplicación web y Api usando la plantailla de aplicación webapi):
        
        [<img title="image" style="border-left-width: 0px; border-right-width: 0px; background-image: none; border-bottom-width: 0px; padding-top: 0px; padding-left: 0px; display: inline; padding-right: 0px; border-top-width: 0px" border="0" alt="image" src="https://geeks.ms/etomas/wp-content/uploads/sites/154/2017/03/image_thumb.png" width="363" height="136" />][2]
        
        Ahora toca agregar la librería. En algun momento, esto será añadir un paquete NuGet (quizá más de uno) pero por ahora es “manual”. Descárgate el código del repo y copia las siguientes carpetas en la solución:
        
          * src/common 
              * src/Microsoft.AspNet.HealthChecks 
                  * src/Microsoft.Extensions.HealthChecks 
                      * src/Microsoft.Extensions.HealthChecks.Data</ul> 
                    Ahora **añade a la solución** los proyectos que están en cada una de las carpetas añadidas (menos en common que no hay proyecto):
                    
                    [<img title="image" style="border-left-width: 0px; border-right-width: 0px; background-image: none; border-bottom-width: 0px; padding-top: 0px; padding-left: 0px; display: inline; padding-right: 0px; border-top-width: 0px" border="0" alt="image" src="https://geeks.ms/etomas/wp-content/uploads/sites/154/2017/03/image_thumb-1.png" width="319" height="208" />][3]
                    
                    Ya lo tenemos todo a punto. Ahora **vamos a modificar la aplicación web para que nos muestre un informe del estado de los servicios externos**. Por el momento el único servicio externo es la aplicación Api.
                    
                    Para ello, lo primero es agregar una referencia desde el proyecto _WebClient_ al proyecto _Microsoft.AspNetCore.HealthChecks_:
                    
                    Ahora en el método _ConfigureServices_ de la clase _Startup_ del proyecto _WebClient_ agregamos el siguiente código:
                    
                    <pre style="font-family: consolas; background: #1e1e1e; white-space: nowrap; overflow-x: scroll; color: gainsboro">services<span style="color: #b4b4b4">.</span>AddHealthChecks(checks <span style="color: #b4b4b4">=&gt;</span><br />{<br />&nbsp;&nbsp;&nbsp; checks<span style="color: #b4b4b4">.</span>AddUrlCheck(Configuration[<span style="color: #d69d85">"check_api_url"</span>]);<br />});</pre>
                    
                    Con eso creamos un _check_ de _url_. Este tipo de checks simplemente comprueban que una determinada URL devuelve un 200. Por supuesto necesitamos indicar (en este caso en el fichero de config) la url a usar:
                    
                    <pre style="font-family: consolas; background: #1e1e1e; white-space: nowrap; overflow-x: scroll; color: gainsboro"><span style="color: #d7ba7d">"check_api_url"</span><span style="color: #b4b4b4">:</span>&nbsp;<span style="color: #d69d85">"http://localhost:56661/api/ping"</span><br /></pre>
                    
                    ¿Sencillo, verdad? Simplemente establecemos que si /api/ping devuelve un 200, entonces significa que la API está levantada y funcionando.
                    
                    Ahora creamos una acción en el controlador _Home_ de nuestra _WebClient_ que vamos a llamar _Status_. Dentro de esta acción usaremos un servicio proporcionado por la librería, para ejecutar los _checks_ de forma manual:
                    
                    <pre style="font-family: consolas; background: #1e1e1e; white-space: nowrap; overflow-x: scroll; color: gainsboro"><span style="color: #569cd6">public</span>&nbsp;<span style="color: #569cd6">class</span>&nbsp;<span style="color: #4ec9b0">HomeController</span> : <span style="color: #4ec9b0">Controller</span><br />{<br />&nbsp;&nbsp;&nbsp; <span style="color: #569cd6">private</span>&nbsp;<span style="color: #569cd6">readonly</span>&nbsp;<span style="color: #b8d7a3">IHealthCheckService</span> _healthCheckSvc;<br />&nbsp;&nbsp;&nbsp; <span style="color: #569cd6">public</span> HomeController(<span style="color: #b8d7a3">IHealthCheckService</span> healthSvc) <span style="color: #b4b4b4">=&gt;</span> _healthCheckSvc <span style="color: #b4b4b4">=</span> healthSvc;<br /> <br />&nbsp;&nbsp;&nbsp; <span style="color: #569cd6">public</span>&nbsp;<span style="color: #569cd6">async</span>&nbsp;<span style="color: #4ec9b0">Task</span>&lt;<span style="color: #b8d7a3">IActionResult</span>&gt; Status()<br />&nbsp;&nbsp;&nbsp; {<br />&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span style="color: #569cd6">var</span> result <span style="color: #b4b4b4">=</span>&nbsp;<span style="color: #569cd6">await</span> _healthCheckSvc<span style="color: #b4b4b4">.</span>CheckHealthAsync();<br />&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span style="color: #569cd6">return</span> View(result);<br />&nbsp;&nbsp;&nbsp; }<br /> <br />&nbsp;&nbsp;&nbsp; <span style="color: #569cd6">public</span>&nbsp;<span style="color: #b8d7a3">IActionResult</span> Index()<br />&nbsp;&nbsp;&nbsp; {<br />&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span style="color: #569cd6">return</span> View();<br />&nbsp;&nbsp;&nbsp; }<br /> <br /> <br />&nbsp;&nbsp;&nbsp; <span style="color: #569cd6">public</span>&nbsp;<span style="color: #b8d7a3">IActionResult</span> Error()<br />&nbsp;&nbsp;&nbsp; {<br />&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span style="color: #569cd6">return</span> View();<br />&nbsp;&nbsp;&nbsp; }<br />}</pre>
                    
                    Observa como inyectamos el servicio _IHeralthCheckService_ y llamamos al método _CheckHealthAsync_. Eso ejecutará todos los _checks_ y nos devolverá un resultado para cada uno de ellos.
                    
                    **Ahora solo nos queda crear el endpoint de “ping” en el proyecto _Api_** (en este caso devolvemos siempre un 200):
                    
                    <pre style="font-family: consolas; background: #1e1e1e; white-space: nowrap; overflow-x: scroll; color: gainsboro">[<span style="color: #4ec9b0">Route</span>(<span style="color: #d69d85">"api/[controller]"</span>)]<br /><span style="color: #569cd6">public</span>&nbsp;<span style="color: #569cd6">class</span>&nbsp;<span style="color: #4ec9b0">PingController</span> : <span style="color: #4ec9b0">Controller</span><br />{<br />&nbsp;&nbsp;&nbsp; <span style="color: #57a64a">// GET api/values</span><br />&nbsp;&nbsp;&nbsp; [<span style="color: #4ec9b0">HttpGet</span>]<br />&nbsp;&nbsp;&nbsp; <span style="color: #569cd6">public</span>&nbsp;<span style="color: #b8d7a3">IActionResult</span> Ping()<br />&nbsp;&nbsp;&nbsp; {<br />&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span style="color: #569cd6">return</span> StatusCode(<span style="color: #b5cea8">200</span>);<br />&nbsp;&nbsp;&nbsp; }&nbsp;&nbsp;&nbsp; <br />}</pre>
                    
                    Si ahora colocamos un breakpoint en la acción _Status_ del controlador _Home_ de la web e inspeccionamos la variable _result_ veremos que nos indica que el estado global del sistema es saludable y luego nos lo desgrana por cada uno de los _checks_:
                    
                    [<img title="image" style="border-left-width: 0px; border-right-width: 0px; background-image: none; border-bottom-width: 0px; padding-top: 0px; padding-left: 0px; display: inline; padding-right: 0px; border-top-width: 0px" border="0" alt="image" src="https://geeks.ms/etomas/wp-content/uploads/sites/154/2017/03/image_thumb-2.png" width="754" height="376" />][4]
                    
                    **Tampoco es para tanto…**
                    
                    Sí, seguro que es lo que estás pensando ¿verdad? Hay dos cosillas que debemos considerar sobre lo que hemos visto:
                    
                      1. Hemos tenido que crear un endpoint específico (/api/ping) en nuestra API 
                          * ¿Qué ocurre con recursos externos, tales como BBDD u otros?</ol> 
                        Veamos que soluciones nos ofrece la librería para esos dos puntos 😉
                        
                        **Middleware de estado de salud**
                        
                        Una cosa interesante de la librería es que **ofrece un endpoint para comprobar el estado de salud de una api** (es decir el equivalente al /api/ping que hemos hecho). Vamos a modificar nuestra Api para usar este middleware en lugar del endpoint /api/ping que hemos creado. Para ello podemos eliminar el _PingController_ y luego debemos agregar referencias desde la Api hacia los proyectos Microsoft.AspNetCore.HealthChecks y&nbsp; Microsoft.Extensions.HealthChecks.
                        
                        Una vez hecho esto añadimos una llamada al método de extensión _UseHealthChecks_ **en el método main** de la clase Program. Justo después del _UseKestrel_.
                        
                        Este método espera un puerto (o un path). Este puerto (o path) será el endpoint a usar para comprobar el estado del sistema. P. ej. en mi caso he usado el puerto 5050. Por lo tanto si ahora me connecto a <http://localhost:5050> (con la API en marcha, claro) veré una página que me informa del estado:
                        
                        [<img title="image" style="border-left-width: 0px; border-right-width: 0px; background-image: none; border-bottom-width: 0px; padding-top: 0px; padding-left: 0px; display: inline; padding-right: 0px; border-top-width: 0px" border="0" alt="image" src="https://geeks.ms/etomas/wp-content/uploads/sites/154/2017/03/image_thumb-3.png" width="264" height="125" />][5]
                        
                        **Nota:** Si en lugar de usar un puerto prefieres usar un _path_ para comprobar el estado, también se puede hacer. En este caso debes llamar a _UseHealthChecks_ pasando el path a usar (p. ej. _UseHealthChecks(“/status”)_ ).
                        
                        **Nota 2:** Si usas un puerto (como he hecho yo), no uses IISExpress para ejecutar los proyectos, porque no tendrás el _binding_ a ese nuevo puerto configurado en IISExpress. Si usas IISExpress usa un path, mejor. Si quieres usar un puerto, levanta Kestrel directamente con _dotnet run_.
                        
                        Nos dice “Unknown” (además devuelve un HTTP 503) porque el middleware no sabe el estado del sistema actual (la API) porque no hemos metido ningún check que indique si la API está en buen estado. Vamos a suponer que queremos indicar que la API está en buen estado.
                        
                        Para ello agregamos lo siguiente en el método _ConfigureServices_ de la API:
                        
                        <pre style="font-family: consolas; background: #1e1e1e; white-space: nowrap; overflow-x: scroll; color: gainsboro"><span style="color: #569cd6">public</span>&nbsp;<span style="color: #569cd6">void</span> ConfigureServices(<span style="color: #b8d7a3">IServiceCollection</span> services)<br />{<br />&nbsp;&nbsp;&nbsp; services<span style="color: #b4b4b4">.</span>AddHealthChecks(checks <span style="color: #b4b4b4">=&gt;</span><br />&nbsp;&nbsp;&nbsp; {<br />&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; checks<span style="color: #b4b4b4">.</span>AddValueTaskCheck(<span style="color: #d69d85">"AlwaysOk"</span>, () <span style="color: #b4b4b4">=&gt;</span>&nbsp;<span style="color: #569cd6">new</span>&nbsp;<span style="color: #4ec9b0">ValueTask</span>&lt;<span style="color: #b8d7a3">IHealthCheckResult</span>&gt;(<span style="color: #4ec9b0">HealthCheckResult</span><span style="color: #b4b4b4">.</span>Healthy(<span style="color: #d69d85">"Always OK"</span>, <span style="color: #569cd6">null</span>)));<br />&nbsp;&nbsp;&nbsp; });<br />&nbsp;&nbsp;&nbsp; <span style="color: #57a64a">// Add framework services.</span><br />&nbsp;&nbsp;&nbsp; services<span style="color: #b4b4b4">.</span>AddMvc();<br />}</pre>
                        
                        Este es un _check_ que siempre devuelve el estado de _Healthy_.
                        
                        Si ahora navegamos de nuevo al endpoint de verificación (localhost:5050 en nuestro caso) veremos lo siguiente:
                        
                        [<img title="image" style="border-left-width: 0px; border-right-width: 0px; background-image: none; border-bottom-width: 0px; padding-top: 0px; padding-left: 0px; display: inline; padding-right: 0px; border-top-width: 0px" border="0" alt="image" src="https://geeks.ms/etomas/wp-content/uploads/sites/154/2017/03/image_thumb-4.png" width="297" height="138" />][6]
                        
                        ¡Fantástico! Ahora la API nos reporta que está en buen estado (y devuelve un HTTP 200). Observa que ahora localhost:5050 es el endpoint de verificación que vamos a usar en la web, en lugar del api/ping.
                        
                        En resúmen: **el middleware de verificación nos ofrece un endpoint que al ser llamado ejecuta todos los _checks_ definidos y devuelve el estado de salud del sistema en base a estos checks**.
                        
                        Así que ahora tenemos:
                        
                          1. La Web que tiene un check por url (al endpoint de verificación de la API) 
                              * La Api que, usando el middleware de verificación, devuelve su estado (actualmente siempre OK).</ol> 
                            **Comprobando la disponibilidad de recursos externos**
                            
                            Comprobar la disponibilidad de recursos externos es, simplemente, usar checks específicos. En nuestro caso teníamos que la API dependía de un SQL Server. Observa que no debemos añadir un _check_ para verificar el SQL Server en la web (no tiene sentido ya que la web no tiene porque conocer ni tener acceso a un recurso usado por la API). No, el check debemos agregarlo en la API. De esta manera si la BBDD está caída, el endpoint de verificación de la API indicará que el estado de la API es incorrecto.
                            
                            Es de esperar que se agreguen varios _checks_ a la librería en un futuro, pero de momento ya tenemos uno para verificar un SQL Server. Para ello debemos añadir una referencia al proyecto **Microsoft.Extensions.HealthChecks.Data**.
                            
                            Ahora tenemos disponible un nuevo _check_ que podemos agregar con _AddSqlCheck_. Así en el método _ConfigureServices_ ahora tendremos:
                            
                            <pre style="font-family: consolas; background: #1e1e1e; white-space: nowrap; overflow-x: scroll; color: gainsboro">services<span style="color: #b4b4b4">.</span>AddHealthChecks(checks <span style="color: #b4b4b4">=&gt;</span><br />{<br />&nbsp;&nbsp;&nbsp; checks<span style="color: #b4b4b4">.</span>AddSqlCheck(<span style="color: #d69d85">"Main_SQL"</span>, Configuration[<span style="color: #d69d85">"constr"</span>]);<br />});</pre>
                            
                            (Observa que hemos podido quitar el _check_ anterior que siempre devolvía Ok. Ahora si la BBDD funciona, la API está OK y si no, no).
                            
                            **En resúmen**
                            
                            Esta librería nos permite verificar el estado de distintas APIs (y que cada API determine lo que significa que su estado es correcto) de una forma sencilla y extendible. Esto permite informar fácilmente al usuario del estado de los distintos componentes que forman la aplicación y, por supuesto, tomar decisiones en base a estos estados.
                            
                            Espero que te haya resultado interesante y si quieres… anímate que están esperando feedback!

 [1]: https://github.com/aspnet/HealthChecks
 [2]: https://geeks.ms/etomas/wp-content/uploads/sites/154/2017/03/image.png
 [3]: https://geeks.ms/etomas/wp-content/uploads/sites/154/2017/03/image-1.png
 [4]: https://geeks.ms/etomas/wp-content/uploads/sites/154/2017/03/image-2.png
 [5]: https://geeks.ms/etomas/wp-content/uploads/sites/154/2017/03/image-3.png
 [6]: https://geeks.ms/etomas/wp-content/uploads/sites/154/2017/03/image-4.png