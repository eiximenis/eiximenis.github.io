---
title: WPF y netcore3 con custom host
author: eiximenis
description: Una de las ventajas de usar WPF bajo .NET Core 3 es la posibilidad de usar el custom host de .Net Core y así obtener todas sus ventajas (DI, configuración, logging, ...). En este post vemos como
date: 2020-01-20T18:20:00
categories:
  - netcore
  - wpf
---

¡Buf! Hacía un porrón que no escribía sobre WPF, pero bueno, creo que vale la pena hablar un poco de las ventajas de usar WPF junto con .NET Core 3. Sí, se habla mucho de los aumentos de rendimiento pero a mi me interesa enfocarlo más en como usar, fácilmente, las ventajas intrínsecas de .NET Core al usar un proyecto de WPF. En concreto hay tres puntos que creo que son interesantes: DI, Configuración y Logging.

## Usar el custom host

Vamos a ver como preparar nuestro código para usar el _custom host_ de .Net Core en aplicaciones WPF y así, luego, poder usar todas sus ventajas.

Lo primero que debemos hacer es lo siguiente en `App.xaml`:

* 1. Eliminar el `StartupUri` ya que vamos a crear nosotros la aplicación.
* 2. Agregar el evento `Startup`: `Startup="Application_Startup"`
* 3. Agregar el evento `Exit `: `Exit="Application_Exit"`

Ahora ya podemos poner el código en `App.xml.cs`:

```cs
private readonly IHost _host;

public App()
{
    _host = new HostBuilder().Build();
}

private async void Application_Startup(object sender, StartupEventArgs e)
{
    await _host.StartAsync();

    var mainWindow = new MainWindow();
    mainWindow.Show();
}

private async void Application_Exit(object sender, ExitEventArgs e)
{
    using (_host)
    {
        await _host.StopAsync(TimeSpan.FromSeconds(10));
    }
}
```

Eso levanta nuestra aplicación, exactamente igual que antes, pero ya podemos usar todas las ventajas del _custom host_. ¡Vamos allá!

## Inyección de dependencias

Desde siempre .NET Core ha soportado inyección de dependencias y podemos aprovecharnos de nuestro amigo, el `IServiceCollection` para nuestras aplicaciones .NET Core. Del mismo modo que en un aplicación web MVC registramos los controladores y todas sus dependencias, eso mismo podemos hacer en una aplicación WPF. Simplemente usamos el método `ConfigureServices` del `HostBuilder`:

```cs
public App()
{
    _host = new HostBuilder()
        .ConfigureServices(ConfigureServices)
        .Build();
}

private void ConfigureServices(IServiceCollection services)
{
    // Registramos todos los servicios y los view models. P. ej:
    services.AddTransient<MainWindowViewModel>();
    // Registramos WPF
    services.AddWpf();
}

```

El método `AddWpf` es un método de extensión chungo-pelotero que me he creado yo, que, básicamente, registra en el sistema de DI todas las ventanas:

```cs
public static IServiceCollection AddWpf(this IServiceCollection services)
{
    var assembly = Assembly.GetExecutingAssembly();
    var windows = assembly.GetTypes().
        Where(t => t.IsSubclassOf(typeof(Window)));
    foreach (var wtype in windows)
    {
        services.AddTransient(wtype);
    }

    return services;
}
```

En este caso solo registro las clases que hereden de `Window`, por supuesto lo puedes adaptar a tus necesidades :)

Con eso ya tenemos nuestro sistema de DI configurado y **podemos inyectar el _viewmodel_ a la ventana**:

```cs
// El sistema de DI inyectará el MainWindowViewModel
public MainWindow(MainWindowViewModel vm)
{
    InitializeComponent();
    this.DataContext = vm;
}
```

## Configuración

La gestión de la configuración es otra de las grandes ventajas que tenemos ya incorporadas en .NET Core. En este caso, simplemente llamamos al método `ConfigureAppConfiguration` de nuestro `HostBuilder`:

```cs
_host = new HostBuilder()
    .ConfigureAppConfiguration((ctx, cfg) =>
    {
        cfg.SetBasePath(ctx.HostingEnvironment.ContentRootPath);
        cfg.AddJsonFile("appsettings.json", optional: true);
        cfg.AddEnvironmentVariables();

    })
```

Solo ten presente de incluir el fichero `appsettings` con la opción "Copy to output", puedes usar la ventana de propiedades del fichero en Visual Studio, o añadir lo siguiente en el `.csproj`:

```xml
<ItemGroup>
<None Update="appsettings.json">
    <CopyToOutputDirectory>PreserveNewest</CopyToOutputDirectory>
</None>
</ItemGroup>
``` 

## Logging

Oye, ya puestos podemos configurar el _logging_ también, en este caso basta con llamar al método `ConfigureLogging`:

```cs
_host = new HostBuilder()
    .ConfigureAppConfiguration((ctx, cfg) =>
    {
        cfg.SetBasePath(ctx.HostingEnvironment.ContentRootPath);
        cfg.AddJsonFile("appsettings.json", optional: true);
        cfg.AddEnvironmentVariables();

    })
    .ConfigureServices(ConfigureServices)
    .ConfigureLogging(log =>
    {
        log.AddConsole();
    })
    .Build();
```

¡Y ya está! Tenemos una aplicación de WPF ejecutándose bajo el _custom host_ de .NET Core con todas sus ventajas, lo que es mucho mejor que la plantilla por defecto que incorpora Visual Studio, que usa "el estilo antiguo" del WPF clásico de .NET Framework :)
