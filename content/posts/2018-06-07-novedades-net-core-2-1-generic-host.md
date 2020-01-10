---
title: 'Novedades .NET Core 2.1: Generic host'

author: eiximenis

date: 2018-06-07T11:04:55+00:00
geeks_url: /?p=2073
geeks_ms_views:
  - 851
categories:
  - .net
  - asp.net 5

---
Ahora que .NET Core 2.1 ya es oficial ya podemos desgranar algunas de sus novedades más interesantes. La verdad es que, por fin, se vislumbra una madurez en la plataforma. Realmente a no ser que haya algún motivo de fuerza mayor (librería no disponible), .NET Core 2.1 debería ser la opción _por defecto_ a la hora de empezar cualquier proyecto nuevo.
  
<!--more-->


  
En este _post_ quiero hablaros del host genérico de .NET Core 2.1. En 2.0 usábamos el siguiente código de base para crear un _WebHost_ que hospedaba nuestra aplicación:

<pre class="EnlighterJSRAW" data-enlighter-language="csharp">public static IWebHost BuildWebHost(string[] args) =&gt;
    WebHost.CreateDefaultBuilder(args)
        .UseStartup&lt;Startup&gt;()
        .Build();</pre>

El método _CreateDefaultBuilder_ nos daba una instancia de _IWebHostBuilder_ preparada para añadir la configuración y servicios más habituales. Es este método el que hace la magia de que, entre otras cosas, el fichero de configuración sea appsettings.json y no otro o bien de que las variables de entorno también formen parte de la configuración o que el logging esté habilitado por defecto. Por supuesto, no estamos obligados a usar este método y usar un objeto _WebHostBuilder_ configurado a nuestro gusto, con nuestras necesidades concretas.
  
Pero al final lo que obtenemos es siempre un _IWebHost_ que es el objeto que termina hospedando nuestra aplicación. Este modelo funciona muy bien en aplicaciones web, pero **nos puede interesar tener un modelo similar de _hosting_ en aplicaciones que no sean web**.
  
Pongamos p. ej. una aplicación de consola que deba hacer algo hasta que alguien la pare. Si creamos una aplicación de consola usando NetCore 2.1 la plantilla por defecto no ha cambiado nada respecto a NetCore 2.0:

<pre class="EnlighterJSRAW" data-enlighter-language="csharp">class Program
{
    static void Main(string[] args)
    {
        Console.WriteLine("Hello World!");
    }
}</pre>

Y ya. Eso es todo lo que obtenemos. Si, como decía antes quiero hacer una aplicación de consola que realice cierta tarea hasta que sea parada, todo eso corre de mi parte. Pero, no sería interesante poder usar el modelo de _hosting_ y levantar un _host _que hospede mi aplicación? Vale, no expondrá ningún _endpoint_ http porque no es una aplicación web, pero el _host_ gestionará el ciclo de vida de mi aplicación.
  
Pues eso es, precisamente lo que nos permite usar el host genérico de NetCore 2.1:

<pre class="EnlighterJSRAW" data-enlighter-language="null">async static Task Main(string[] args)
{
    var builder = new HostBuilder();
    await builder.RunConsoleAsync();
}</pre>

Debes añadir el paquete [Microsoft.Extensions.Hosting][1]. Luego para que esto te compile asegúrate de que el compilador está en modo C# 7.1 como mínimo, ya que estamos usando _async Main_. Para ello edita el fichero .csproj y añade dentro de la etiqueta <PropertyGroup>:

<pre class="EnlighterJSRAW" data-enlighter-language="null">&lt;LangVersion&gt;7.1&lt;/LangVersion&gt;</pre>

(Si tienes VS puedes ir a las &#8220;propiedades del proyecto -> Build -> Advanced -> Language Version&#8221; y seleccionar al menos 7.1. El resultado es el mismo).
  
Si ahora ejecutas este código verás que la aplicación se pone en marcha y se queda &#8220;eternamente&#8221; esperando:
  
[<img class="alignnone size-medium wp-image-2075" src="https://geeks.ms/etomas/wp-content/uploads/sites/154/2018/06/host-console-en-marcha-300x37.png" alt="Aplicación en marcha" width="300" height="37" />][2]
  
En este caso hemos usado el método de extensión _RunConsoleAsync_ que levanta el _host_ con soporte de consola y lo mantiene levantado hasta que llega [SIGTERM][3] o [SIGINT][3] (Ctrl+C).
  
Por supuesto la aplicación no hace nada porque hemos levantado un _host_ vacío. Veamos ahora como configurarlo, y como verás es casi idéntico a como configuramos el _IWebHost_ usando el _WebHostBuilder_:

<pre class="EnlighterJSRAW" data-enlighter-language="csharp">var builder = new HostBuilder()
    .ConfigureAppConfiguration(cfg =&gt;
    {
        cfg.AddJsonFile("appsettings.json", optional: true);
        cfg.AddEnvironmentVariables();
    })
    .ConfigureServices(sc =&gt;
    {
        sc.AddTransient&lt;IGuidProvider, GuidProvider&gt;();
    })
    .ConfigureLogging(lb =&gt;
    {
        lb.AddConsole();
    });</pre>

Seguramente te sonaran todos esos métodos, ya que son iguales a los que venimos usando en ASP.NET Core. En este caso hemos registrado un servicio (IGuidProvider), hemos configurado el login y la configuración usando _appsettings.json_ y las variables de entorno. Pongo, por completitud, el código de _IGuidProvider_, aunque por el nombre ya debes deducir lo qué hace 😛

<pre class="EnlighterJSRAW" data-enlighter-language="csharp">public interface IGuidProvider
{
    Guid Id { get; }
}
public class GuidProvider : IGuidProvider
{
    public Guid Id { get; }
    public GuidProvider() =&gt; Id = Guid.NewGuid();
}</pre>

Solo comentar que deberás añadir algunos paquetes adicionales:

  * [Microsoft.Extensions.Configuration.EnvironmentVariables][4]
  * [Microsoft.Extensions.Configuration.Json][5]
  * [Microsoft.Extensions.Logging.Console][6]

Si pones en marcha la aplicación, de nuevo sigue sin hacer nada. Claro, hemos configurado el _host_ pero no le hemos dicho qué hospeda. Del mismo modo que una aplicación ASP.NET Core hospeda _middlewares_, una aplicación con el host genérico hospeda objetos _IHostedService. _**Como parte de proceso de inicialización del _host_ se inician (uno tras otro) los _IHostedService_ que tengamos:**

<pre class="EnlighterJSRAW" data-enlighter-language="csharp">public class GuidConsumerService : IHostedService
{
    private readonly Guid _myId;
    private readonly ILogger&lt;GuidConsumerService&gt; _logger;
    public GuidConsumerService(IGuidProvider gp, ILogger&lt;GuidConsumerService&gt; logger)
    {
        _myId = gp.Id;
        _logger = logger;
    }
    public Task StartAsync(CancellationToken cancellationToken)
    {
        _logger.LogInformation("Starting worker " + _myId);
        return Task.CompletedTask;
    }
    public Task StopAsync(CancellationToken cancellationToken)
    {
        _logger.LogInformation("Stopping worker " + _myId);
        return Task.CompletedTask;
    }
}</pre>

Una vez tenemos el _IHostedService_ creado lo añadimos al host usando el método _AddHostedService_ dentro de _ConfigureServices_:

<pre class="EnlighterJSRAW" data-enlighter-language="csharp">.ConfigureServices(sc =&gt;
{
    sc.AddTransient&lt;IGuidProvider, GuidProvider&gt;();
    sc.AddHostedService&lt;GuidConsumerService&gt;();
    sc.AddHostedService&lt;GuidConsumerService&gt;();
})</pre>

Si ahora iniciamos la aplicación de nuevo veremos lo siguiente:
  
[<img class="alignnone size-medium wp-image-2076" src="https://geeks.ms/etomas/wp-content/uploads/sites/154/2018/06/host-console-en-marcha2-300x53.png" alt="hosted services en marcha" width="300" height="53" />][7]
  
(La inicialización de cada _IHostedService_ es asíncrona, así que el orden de los mensajes puede variar).
  
Un ejemplo típico es tener un _IHostedService _que realice algo cada cierto tiempo, entonces en este caso puedes usar un temporizador:

<pre class="EnlighterJSRAW" data-enlighter-language="csharp">public class GuidConsumerService : IHostedService, IDisposable
{
    private readonly Guid _myId;
    private readonly ILogger&lt;GuidConsumerService&gt; _logger;
    private Timer _timer;
    public GuidConsumerService(IGuidProvider gp, ILogger&lt;GuidConsumerService&gt; logger)
    {
        _myId = gp.Id;
        _logger = logger;
    }
    public Task StartAsync(CancellationToken cancellationToken)
    {
        _logger.LogInformation("Starting worker " + _myId);
        _timer = new Timer(DoSomething, null, TimeSpan.Zero, TimeSpan.FromSeconds(5));
        return Task.CompletedTask;
    }
    private void DoSomething(object state)
    {
        _logger.LogInformation($"I am {_myId} doing stuff!");
    }
    public Task StopAsync(CancellationToken cancellationToken)
    {
        _logger.LogInformation("Stopping worker " + _myId);
        _timer?.Change(Timeout.Infinite, 0);
        return Task.CompletedTask;
    }
    public void Dispose() =&gt; _timer?.Dispose();
}</pre>

Otros usos de IHostedService son, p. ej. levantar un servicio que escuche en un RabbitMQ o en un Azure Service Bus y haga algo cada vez que se recibe un mensaje. En este caso en _StartAsync_ habría el código para suscribirnos a la cola o al topic deseado y en _StopAsync_ nos desuscribiríamos.
  
En este post hemos usado el método de extensión _[RunConsoleAsync][8] _pero lo mismo podríamos haber conseguido usando directamente _IHost_:

<pre class="EnlighterJSRAW" data-enlighter-language="csharp">var builder = new HostBuilder()
    // TODA LA CONFIGURACIÓN DE ANTES
    .UseConsoleLifetime();
  await builder.Build().RunAsync()</pre>

En resúmen: el _host_ genérico de Net Core 2.1 es un mecanismo ideal que nos permite nuevos escenarios que antes eran más complicados: ahora hacer aplicaciones _self-hosted_  que interaccionen con servicios de mensajería, de eventos o que tengan que realizar una tarea periódica ¡es más fácil que nunca! La gran ventaja es que toda la infraestructura que ya conocemos de ASP.NET Core (configuración, DI, logging, etc) la reaprovechamos completamente.

 [1]: https://www.nuget.org/packages/Microsoft.Extensions.Hosting/
 [2]: https://geeks.ms/etomas/wp-content/uploads/sites/154/2018/06/host-console-en-marcha.png
 [3]: https://www.gnu.org/software/libc/manual/html_node/Termination-Signals.html
 [4]: https://www.nuget.org/packages/Microsoft.Extensions.Configuration.EnvironmentVariables/
 [5]: https://www.nuget.org/packages/Microsoft.Extensions.Configuration.Json/
 [6]: https://www.nuget.org/packages/Microsoft.Extensions.Logging.Console/
 [7]: https://geeks.ms/etomas/wp-content/uploads/sites/154/2018/06/host-console-en-marcha2.png
 [8]: https://docs.microsoft.com/en-us/dotnet/api/microsoft.extensions.hosting.hostinghostbuilderextensions.runconsoleasync?view=aspnetcore-2.1