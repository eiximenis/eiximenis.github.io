---
title: "Procesar SIGTSTP en netcore (Linux)"
author: eiximenis
description: "Como procesar una señal del kernel de Linux en netcore"
date: 2020-11-04
draft: false
categories:
  - netcore
tags: ["linux", "terminal"]
---

Una de las cosas que he visto más veces en gente que está dando sus primeros pasos en Linux es usar Vi, equivocarse y darle a Ctrl+Z para hacer undo... con el "catastrófico" resultado de que Vi desaparecen y se vuelve al terminal. Además, para más horror eso sucede sin ninguna confirmación y da igual si había cambios por guardar. No solo eso, si no que si luego se inicia otra vez Vi para continuar editando el mismo fichero, se recibe un mensjae de error de que el fichero está siendo editado en otro editor y varias opciones a elegir sobre qué hacer, en las que uno tiene la sospecha que haga lo que haga va a perder algún dato.

¡Que no cunda el pánico! Antes de que Windows estandarizada el Ctrl+Z como _undo_, en Unix esta combinación de teclas manda una señal que suspende el proceso. El proceso está en estado suspendido hasta que no reciba la señal de que debe ser continuado. Un proceso suspendido mantiene su PID y su estado, pero no ocupa CPU ya que no ejecuta instrucción alguna.

Bien, si has llegado hasta aquí por alguna búsqueda tipo "Ctrl+Z ha matado/desparecido/terminado Vi" te ahorro el sufrimiento: teclea `fg` en el terminal y Vi volverá a aparecer tal cual estaba y listos, ya puedes continuar editando tu fichero.

## SIGTSTP y SIGSTOP

Cuando el Kernel de Linux debe informar a un proceso de un determinado evento, le manda una señal. Hay un porrón de señales (las puedes ver con `kill -l`), algunas de ellas son interceptables y cancelables por el proceso, otras no. Las que nos interesan para este post son `SIGTSTP` y `SIGSTOP`. Ambas son para poner un proceso en suspensión, pero la primera es cancelable por el proceso, mientras que la segunda no. Cuando se pulsa `^Z` el terminal manda un carácter que se interpreta como `SIGTSTP`, así que el kernel mandará esa señal al proceso que tiene la terminal enlazada. Sin hacer nada especial cualquier proceso ya "hace caso" a esas señales. Observa ese programa de consola:

```csharp
using System;
using System.Threading.Tasks;

namespace Test
{
    class Program
    {
        static async Task Main(string[] args)
        {
            var i = 0;
            while (true)
            {
                Console.Write($".{i}.");
                i++;
                await Task.Delay(500);
            }
        }
    }
}
```

Este programa imprimirá la secuencia `.0..1..2..3..4.` y así sucesivamente con un número nuevo cada medio segundo. Si mientras se está ejecutando pulsas Ctrl+Z, volverás a tu terminal y podrás hacer lo que quieras. Luego, cuando desees, teclea `fg` y verás como se continua imprimiendo la secuencia a partir del valor donde terminó cuando pulsaste Ctrl+Z. Has pausado el proceso (con Ctrl+Z) y luego lo has vuelto a poner en marcha (con `fg`). Realmente, por debajo el proceso ha recibido la señal `SIGTSTP` para pausarse y la `SIGCONT` para continuar.

Puedes simular el mismo efecto abriendo dos terminales y usando `kill`. A pesar de su nombre `kill` no solo mata procesos, si no que manda cualquier señal a un proceso. En un terminal ejecutas el proyecto de consola, y en el otro terminal le mandas la señal de `SIGTSTP` usando `kill -20 <pid-proceso>` (el `-20` es la señal a mandar, en este caso `SIGTSTP`). Si ahora vuelves al otro terminal verás que el proceso de netcore ha "terminado". Pero lo puedes continuar en cualquier momento, ya sea tecleando `fg` en el shell, o mandando la señal de `SIGCONT` usando `kill -18 <pid-proceso>`. Ahora bien, si usas el `kill -18` vas a tener "problemas", porque el proceso se continuará ejecutando en la terminal donde se estaba ejecutando antes... pero esa terminal está ejecutando el shell (bash), y el shell "no sabe" que el proceso ha sido despertado (a diferencia de cuando usas `fg`). El proceso se sigue ejecutando, pero en segundo plano. El resultado es que tendrás la secuencia de números y puntos, monstrándose a la vez que el shell. Si tecleas `ls` o cualquier otro comando este funciona, pero toda la salida se ve empañada con la secuencia de puntos y números. Un lío, vamos xD. Si lo quieres solucionar, teclea `fg` para llevar el proceso al primer plano (así el shell "desparece").

Así, que ya ves, por defecto si no haces nada, tus procesos de consola de netcore ya se suspenden al recibir `SIGTSTP`. Entonces, ¿para qué este post?

## Interceptar SIGTSTP

A veces puede ser interesante poder hacer algo cuando se reciba `SIGTSTP`. De hecho, a veces es "obligatorio" hacer algo cuando recibes `SIGTSTP`: si modificas el funcionamiento del terminal, debes restaurarlo a su configuración "original" antes de suspenderte, ya que si no el terminal estará en otra configuración y nada bueno saldrá de ahí. Lo de las configuraciones de terminal en Linux, daría para varios posts, pero básicamente decir que el terminal tiene varios modos de funcionamiento que afectan, básicamente, al pre-procesamiento de ciertas teclas y al uso de ciertos carácteres de control. De nuevo Vi es el mejor ejemplo para ello. Cuando se ejecuta Vi, este toma el control entero del terminal. Por ejemplo, por defecto el terminal siempre muestra la tecla pulsada (es el comportamiento esperado: cuando pulsas una letra, esperas que esa letra aparezca en el terminal). Pero ese comportamiento no le sirve a Vi. A veces Vi si que te mostrará la tecla pulsada, pero muchas otras veces (p. ej. en el modo de comando) Vi no muestra la tecla pulsada si no que hace otras cosas. Así Vi quiere controlar "cuando se muestra un carácter". Para ello establece el modo _noecho_ del terminal. Además Vi desea controlar otras cosas del terminal (para empezar todo su contenido, para mostrar su interfaz). También desea obtener combinaciones de teclas sin procesar, así que establece el modo _raw_. Esos modos se establecen sobre el terminal actual, así que cuando salimos de Vi, este debe establecer los modos anteriores de terminal, ya que en caso contrario tendríamos comportamientos raros (p. ej. el shell espera que el terminal esté en modo _echo_). Así pues, Vi tiene la "obligación" de restaurar el terminal cuando recibe la señal de `SIGTSTP`. De hecho, por lo general, si haces aplicaciones de terminal "de pantalla completa" (vamos que tengan una [TUI](https://en.wikipedia.org/wiki/Text-based_user_interface)) entonces debes modificar la configuración del terminal.

En Linux el modo habitual de interceptar una señal es la llamada de sistema [signal(2)](https://man7.org/linux/man-pages/man2/signal.2.html). Este método nos permite registrar un manejador que se ejecutará cuando el proceso reciba una determinada señal del Kernel. Es tentador usar P/Invoke para llamar a `signal` pero el problema es que el handler de la señal no puede ejecutar "cualquier código". En particular no puede llamar a métodos no reentrantes. Eso, si desarrollas en C/C++ te limita un poco (en [signal-safety(7)](https://man7.org/linux/man-pages/man7/signal-safety.7.html) tienes la lista de funciones que puedes llamaar), pero es asumible. Pero si estás en netcore el inconveniente principal es que muchos de los mecanismos internos de P/Invoke usan funciones del kernel que no son reentrantes, por lo que terminarás teniendo problemas. De hecho, si usas P/Invoke para llamar a `signal`, tu manejador puede hacer poco más que establecer variables globales. Cualquier otra cosa te pone en "zona peligrosa".

Bueno, al turrón: en mi caso tengo un programa en netcore que hace varias cosas con el terminal, así que me interesaba que al ser suspendido, pudiera restaurar el terminal y que se comportase en definitiva como lo hace Vi. Estuve buscando bastante, y de hecho di con una issue en la cual [se discutía como exponer señales de Unix a netcore](https://github.com/dotnet/runtime/issues/19958). Tras varios comentarios [había una respuesta de Miguel de Icaza](https://github.com/dotnet/runtime/issues/19958#issuecomment-274116942) en la que mencionaba que lo mejor sería usar lo mismo que ya hacía Mono: usar la librería `Mono.Posix`. Esa librería [tal cual está](https://www.nuget.org/packages/Mono.Posix-4.5/) no sirve para netcore, pero [hay un port de esa misma librería a netstandard que sí que sirve](https://www.nuget.org/packages/Mono.Posix.NETStandard/).

Esa librería ofrece un wrapper en C# a muchas llamadas de Linux... Es cierto que está un poco "desorganizada", llena de métodos decorados con `[Obsolete]` y que es muy monolítica, pero funciona y, lo más importante, está ampliamente probada. Y en el caso de señales tiene una pequeña joya que es la clase [`UnixSignal`](http://docs.go-mono.com/index.aspx?link=T%3AMono.Unix.UnixSignal). Esta clase es **simplemente un semáforo que permite esperar hasta que se reciba una señal determinada**. Nada más. Lo bueno es que tanto la espera como "lo que se haga después de la espera" es código manejado que no requiere P/Invoke ni está sujeto a las limitaciones de un manejador establecido por `signal`. Con esa librería en mente podemos empezar a articular nuestra función. La idea básica consiste en:

1. Crear una tarea nueva, encolada en un hilo aparte que escuche por la señal `SIGTSTP`. Eso, usando `UnixSignal` son aproximadamente un par de líneas de código.
2. Cuando se reciba la señal, llamar a un delegado que realice las tareas que se deseen.

En resumen, y a lo bruto algo como:

```csharp
// dotnet add package Mono.Posix.netstandard --prerelease

var sigtstp = new UnixSignal(Signum.SIGTSTP);       
while (true) {
    sigtstp.WaitOne();
    sigtstp.Reset();
    OnSigtsp();       // OnSigtstp es un delegate
}
```

Vamos a ver como integrarlo con una pequeña app de consola de verdad.

## Integrándolo en una app de consola con el generic host

Vamos a empezar con una pequeña aplicación de consola usando el host genérico de netcore:

```csharp
class Program
{
    static async Task Main(string[] args)
    {
        await CreateHostBuilder(args)
            .RunConsoleAsync();
    }

    private static IHostBuilder  CreateHostBuilder(string[] args)
    {
            return Host.CreateDefaultBuilder(args)
            .ConfigureServices(sc => sc.AddHostedService<BgService>());
    }
}
class BgService : BackgroundService
{
    protected override async Task ExecuteAsync(CancellationToken stoppingToken)
    {
        var i = 0;
        while (!stoppingToken.IsCancellationRequested) {
            await Task.Delay(1000);
            Console.WriteLine("Iter: " + ++i);
        }
        Console.WriteLine("BgService ended");
    }
}
```

Este código va imprimiendo líneas con el formato `Iter: i` (siendo `i` un entero que se va incrementando) a razón de una línea por segundo. Ahora vamos a añadirle el soporte para procesar `SIGTSTP`. Para ello haremos un método de extensión sobre `IHostBuilder` para que se pueda hacer algo como eso:

```csharp
await CreateHostBuilder(args)
    .AddSigtstp(options =>
        options.HandleWith(sp =>  Console.WriteLine("SIGTSTP received!! ole!!")))
    .RunConsoleAsync();
```

El método `AddSigtstp` es el método de extensión. Dicho método tiene la siguiente firma:

```csharp
public static IHostBuilder AddSigtstp(this IHostBuilder hostBuilder, Action<SigtstpProcessorOptions> optionsAction)
```

La clase `SigtstpProcessorOptions` permite definir qué se debe hacer al recibir la señal de `SIGTSTP`. En este ejemplo solo tiene el método `HandleWith` que acepta un delegado (`Action<IServiceProvider>`) con la acción a realizar:

```csharp
public class SigtstpProcessorOptions
{
    internal Action<IServiceProvider> Handler {get; private set;}
    public SigtstpProcessorOptions()
    {
        Handler = (sp => {});
    }
    public void HandleWith(Action<IServiceProvider> handler) 
    {
        Handler = handler ?? (sp => {});
    }
}
```

Pues ya solo nos queda el método de extensión propiamente dicho. Este método registrará un _hosted service_ que será el que escuchará por la señal de `SIGTSTP`. Este _hosted service_ es la clase `SigtstpProcessor` y hace lo siguiente:

1. En `StartAsync`: Inicia un thread nuevo (usando `Task.Run`) que usa `UnixSignal` para escuchar por la señal de `SIGTSTP`. Observa como **no se devuelve la tarea devuelta por `Task.Run`** si no que se devuelve otra tarea completada. Eso es porque la tarea de `Task.Run`nunca termina y eso bloquearía el host. Cuando se lea la señal `SIGTSTP` se invocará al método `OnSigtstp`.
2. El método `ExecuteAsync` no hace nada.
3. El método `OnSigtstp` se limita a ejecutar la función que se haya indicado.

```csharp
public class SigtstpProcessor : BackgroundService
{
    private readonly IServiceProvider _serviceProvider;

    private  UnixSignal _sigtstp;
    private readonly SigtstpProcessorOptions _options;

    public SigtstpProcessor(IOptions<SigtstpProcessorOptions> options, IServiceProvider serviceProvider) 
    {
        _options = options.Value;
        _serviceProvider = serviceProvider;
    }


    public override Task StartAsync(CancellationToken cancellationToken)
    {
        Task.Run(() => {
            _sigtstp = new UnixSignal(Mono.Unix.Native.Signum.SIGTSTP);
            while (true) {
                _sigtstp.WaitOne();
                _sigtstp.Reset();
                OnSigtstp();                 
            }
        });

        return Task.CompletedTask;
    }

    protected override Task ExecuteAsync(CancellationToken stoppingToken)
    {
        return Task.CompletedTask;
    }

    private void OnSigtstp() 
    {
        using var scope = _serviceProvider.CreateScope();
        _options.Handler.Invoke(scope.ServiceProvider);
        var currentPID = Mono.Unix.Native.Syscall.getpid();
        Mono.Unix.Native.Syscall.kill(currentPID, Signum.SIGSTOP);            
    }
}
```

Vale, **el método `OnSigtstp` hace algo más**, y es mandar una señal de `SIGSTOP` al propio proceso. Eso es porque, al gestionar la señal `SIGTSTP` luego el proceso se quedaba en marcha igualmente. Así, al pulsar Ctrl+Z, aparecía el shell otra vez, pero cada segundo continuaba imprimiéndose una línea con el contenido `Iter: i`. No tengo claro porque cuando se espera por la señal de `SIGTSTP` usando `UnixSignal` luego el proceso no se suspende, pero también usé `signal` y me pasaba lo mismo. Así que sospecho que el problema es que al establecer un manejador propio para la señal (lo que `UnixSignal` hará internamente de algún modo), el manejador por defecto (que debe ser el que suspende el proceso) deja de ejecutarse. De todos modos, tampoco conozco en demasiado detalle ni el funcionamiento de las señales en Linux, ni la implementación de Mono, así que quizá se me escapa algo. Pero bueno, la solución es sencilla: enviar una señal de `SIGSTOP` al propio proceso. Esta señal es equivalente a `SIGTSTP` salvo que no puede ser manejada ni ignorada. Así pues, usar esta señal, suspende el proceso.

¡Y listos! Con esto, nuestro programa ya reconoce la señal `SIGTSTP` y responde adecuadamente a la pulsación de Ctrl+Z :)

> **Bonus track** Ah! Por cierto, si has llegado hasta aquí.... En Vi para deshacer la última acción, en lugar de pulsar Ctrl+Z, pulsas `ESC` para entrar en modo comando y luego `u`. ;)