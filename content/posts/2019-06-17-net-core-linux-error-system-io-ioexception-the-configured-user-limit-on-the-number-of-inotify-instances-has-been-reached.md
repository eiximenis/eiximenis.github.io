---
title: 'Net Core (Linux) Error: System.IO .IOException: The configured user limit on the number of inotify instances has been reached'
description: 'Net Core (Linux) Error: System.IO .IOException: The configured user limit on the number of inotify instances has been reached'
author: eiximenis

date: 2019-06-17T09:57:08+00:00
geeks_url: /?p=2371
geeks_ms_views:
  - 627
categories:
  - asp.net core
  - docker

---
Buenas! Andaba yo preparando unas demos donde tenía varios contenedores ejecutándose en un Kubernetes, usando netcore y Linux. Todo funcionaba (más o menos) bien, hasta que de golpe y porrazo los contenedores empezaron a fallar:

<pre><span class="js-display-url">System.IO</span><span class="tco-ellipsis"><span class="invisible"> </span></span>.IOException: The configured user limit (1024) on the number of inotify instances has been reached</pre>

Este error apareció cuando escalé el número de contenedores y se daba en los nuevos contenedores creados (los iniciados seguían funcionando). ¿Qué podía estar sucendiendo?
  
<!--more-->


  
Bueno, los que llevéis un tiempo en el mundo del desarrollo, seguro que habéis desarrollado una especie de _sexto sentido_ para detectar errores: en muchos casos seguro que sois capaces de determinar más o menos donde puede estar el error. No siempre se acierta, por supuesto, pero en este caso tuve una ligera intiución que se demostró verdadera.
  
Como digo, era un código para unas demos, lo que significa que... bueno, a veces uno hace cosas _que no deberían hacerse_. Y eso es lo que ocurría en este caso. Tenía **dos contenedores distintos que se ejecutaban**. Uno era un contenedor con una Azure Function en netcore, el otro era una API, también en Net Core. En la AF tenía el siguiente código:

<pre class="EnlighterJSRAW" data-enlighter-language="csharp">var config = new ConfigurationBuilder()
.SetBasePath(ctx.FunctionAppDirectory)
.AddJsonFile("local.settings.json", optional: true, reloadOnChange: true)
.AddEnvironmentVariables()
.Build();</pre>

Este código (donde _ctx_ es el ExecutionContext) permite usar un IConfiguration para acceder a la configuración pasada a la función.
  
Por su parte en la web api, tenía el código &#8220;estándard&#8221; de inicialización:

<pre class="EnlighterJSRAW" data-enlighter-language="csharp">public class Program
{
    public static void Main(string[] args)
    {
        CreateWebHostBuilder(args).Build().Run();
    }
    public static IWebHostBuilder CreateWebHostBuilder(string[] args) =&gt;
        WebHost.CreateDefaultBuilder(args)
            .UseStartup&lt;Startup&gt;();
}</pre>

Pero (ya sabéis, es código para una demo xD), hacía **algo que NO debe hacerse. **Tenía un servicio (_StorageService)_ declarado como _transient_ en el sistema de DI. Este servicio era **inyectado** a un controlador de la API. Hasta ahí nada raro o incorrecto. La mala práctica estaba en el constructor del servicio _StorageService_:

<pre class="EnlighterJSRAW" data-enlighter-language="null">public StorageClient(IConfiguration config)
{
    _constr = config["StorageConnectionString"];
}</pre>

**Le inyectaba _IConfiguration _al servicio**. **<span style="text-decoration: underline;">NO HAGÁIS NUNCA ESO.</span>** Es una mala práctica reconocida (un servicio no debería depender de toda la configuración, solo de aquellas partes que le afectan). Net Core ofrece la interfaz [IOptions<T>][1] para esos casos. Bueno... lo que decía de &#8220;código demo&#8221;, para ahorrarme dos líneas y una clase extra inyecté IConfiguration directamente... Y eso es probable que contribuyese al error.
  
Vayamos ahora al error: se queja de que el número máximo de _inotify_ ha sido alcanzado. Bien, en Linux un [_inotify_][2] [es un objeto del kernel que se usa para monitorizar cambios en ficheros del disco][2]. Como todos los objetos del kernel hay un límite (en este caso 1024).
  
Seguramente sabes **que los contenedores permiten ejecutar tu aplicación en un entorno aislado**, pero también debes (o deberías) saber **que los contenedores no virtualizan: el kernel es compartido**. Eso significa que **es posible &#8220;auto ataques de DoS entre contenedores&#8221;**. Si el límite de instancias de _inotify_ es 1024, este límite es a nivel de kernel (por cada usuario). Si un contenedor usa 1023 _inotify_, queda uno solo para todos los demás contenedores. **Por eso, los nuevos contenedores fallaban al iniciarse: los _inotify_ estaban agotados, **y este es (por el momento) un recurso &#8220;global&#8221; compartido entre todos los contenedores. Ten presente siempre eso: el kernel se comparte y algunos de sus objetos tienen límites globales.
  
La pregunta, de **qué causaba que se agotasen 1024 _inotify_ con apenas 7-8 contenedores corriendo**, tiene que ver con esa línea:

<pre class="EnlighterJSRAW" data-enlighter-language="null">AddJsonFile("local.settings.json", optional: true, reloadOnChange: true)</pre>

Este _reloadOnChange: true_, es el culpable. Ese parámetro habilita la reconfiguración automática si el fichero se modifica (algo que, dicho sea de paso me da igual, ya que los contenedores los configuro con variables de entorno). Me dirás &#8220;_Ya, pero en la web api NO usas esa línea_&#8220;. Falso: Esa lína está metida dentro del método _WebHost.CreateDefaultBuilder_.
  
Quizá haya algún error en netcore que hace que no se liberen los _inotify_ o quizá eso es complicado/imposible de hacer automáticamente (hay una [issue donde se discute eso][3]). Pero **parece que inyectar/crear múltiples veces un IConfiguration creado con _reloadOnChange:true_ te llevará al desastre**.
  
Honestamente, en mi caso yo tenía dos casos donde lo hacía:

  * La Azure function (cada invocación crea un IConfiguration)
  * La Web Api (se inyectaba IConfiguration en un controlador), que se llamaba varias veces por segundo.

No sé si la causa es la AF, la Web Api o ambos (me inclino a pensar que la AF, pero hay gente que dice que el error se les da solo con una web api, así que pueden ser ambos los causantes del error).
  
Lo solucione obviamente haciendo:

  1. No inyectando IConfiguration si no usando IOptions<T> en la web api
  2. Usando **reloadOnChanges:false** **en la Azure Function**

Tras esos dos cambios, parece ser que todo está funcionando. **Si el error se me reproduce (ya informaré si se da el caso), eso significa que cualquier webapi puede generar ese error** (usar IOptions<T> no lo solventa) y que deberíamos dejar de usar _WebHost.CreateDefaultBuilder_, o si lo usamos, modificar el builder para quitar las entradas de configuración creadas con _reloadOnChange:true_.
  
En cualquier caso recordad: Incluso sin problemas de _inotify_, es una mala práctica inyectar IConfiguration directamente 🙂
  
Saludos!
  
&nbsp;

 [1]: https://docs.microsoft.com/en-us/dotnet/api/microsoft.extensions.options.ioptions-1?view=aspnetcore-2.2
 [2]: http://man7.org/linux/man-pages/man7/inotify.7.html
 [3]: https://github.com/dotnet/corefx/issues/32024