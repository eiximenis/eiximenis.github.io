---
title: ASP.NET vNext – IBuilder
author: eiximenis

date: 2014-06-09T17:08:02+00:00
geeks_url: /?p=1668
geeks_visits:
  - 1354
geeks_ms_views:
  - 1292
categories:
  - Uncategorized

---
Muy buenas! Esos días he estado jugando con VS2014 CTP. Esta versión **no puede instalarse side by side** con cualquier versión anterior de VS, así que la he instalado en una máquina virtual en Azure... La verdad es que es el mecanismo más rápido para probarlo, puesto que ha hay una plantilla de MV en Azure que contien VS2014 CTP. Vamos, que en cinco minutos pasas de no tener nada a estar ya trasteando con el VS2014. ¡Genial!

Una de las grandes novedades de este VS2014 es el _tooling_ para ASP.NET vNext. Si bien ya hace algunos días que se podía descargar ASP.NET vNext y juguetear con ella, no era posible usar VS para ello, teniendo que recurrir a las herramientas de líneas de comandos.

Para ver de que se compone una aplicación ASP.NET vNext he abierto el VS2014 y he creado una nueva ASP.NET vNext web Application.

La diferencia más importante respecto a una web clásica **es que ya no existen ni global.asax ni web.config**. Ahora tanto los elementos de configuración que se incluían dentro de web.config como la inicialización de la aplicación que se realizaba en global.asax se unifican en un mismo archivo (que por defecto se llama Startup.cs).

Dicho archivo contiene una clase (llamada Startup) que es invocada automáticamente por el runtime de ASP.NET vNext para realizar la inicialización de la aplicación. Nos encontramos ante un nuevo ejemplo de _convention over configuration_. No es necesario configurar nada: crea una clase llamada _Startup_ y esa pasará a ser la clase que inicialice toda tu aplicación. Eso no es nuevo de vNext, ya en Katana ocurría algo muy similar.

Dicha clase Startup debe contener un método llamado _Configure_ que espera un parámetro de tipo _IBuilder._ Dicha interfaz está definida de la siguiente manera:

<pre>public interface IBuilder
    {
        IServiceProvider ApplicationServices { get; set; }
        IServerInformation Server { get; set; }
        RequestDelegate Build();
        IBuilder New();
        IBuilder Use(Func&lt;RequestDelegate, RequestDelegate&gt; middleware);
    }
</pre>

El método importante es el método Use que permite "enchufar" un componente al pipeline de tu aplicación. Ten presente que ASP.NET vNext es totalmente modular de forma que el primer paso al configurar una aplicación es enchufar todos los componentes. Así ahora, ASP.NET MVC6 es un componente, al igual que lo es WebApi 3 y al igual que lo es un sistema para autenticarnos con cookies. Si conoces Katana es exactamente la misma idea.

El método Use es muy genérico pero raras veces lo usarás directamente: lo normal es quc cada componente proporcione un método de extensión sobre IBuilder para enchufar dicho componente. Así para enchufar ASP.NET MVC usarás el método UseMvc. Pero al final todos esos métodos de extensión terminan llamando al genérico Use.

**Creación de un componente (middleware) ASP.NET vNext**

Vamos a ver como crear un componente (también se les llama middleware) ASP.NET vNext. Para ello agrega una clase cualquiera a tu proyecto y añádele el siguiente código:

<pre>public class UfoMiddleware
{
    private readonly RequestDelegate _next;
    public UfoMiddleware(RequestDelegate next)
    {
        this._next = next;
    }
    public async Task Invoke(HttpContext ctx)
    {
        Debug.WriteLine("UfoMiddleware::Entrada");
        await _next(ctx);
        Debug.WriteLine("UfoMiddleware::Salida");
    }
}
</pre>

Eso es todo lo que necesitas para crear un middleware (componente) de ASP.NET vNext. El parámetro _next_ del constructor te lo mandará ASP.NET vNext (y es el componente siguiente a invocar). En el método Invoke realizas las acciones que quieras y luego llamas de forma asíncrona al middleware siguiente. Por supuesto puedes colocar código cuando el middleware siguiente ha terminado su ejecución (pero ten presente que el middleware siguiente llamará al siguiente y así sucesivamente). Así, si tienes 3 componentes (A, B y C) encadenados en este orden se ejecutará:

  1. El código de A de antes del await
  2. El código de B de antes del await
  3. El código de C de antes del await
  4. El código de C de después del await
  5. El código de B de después del await
  6. El código de A de después del await

Si te preguntas **en qué orden se ejecutan los componentes**, pues es en el orden el que estén registrados en IBuilder. Es decir el orden en que se llamen los métodos Use.

Para añadir nuestro componente en el pipeline de ASP.NET vNext podemos llamar al método Use de IBuilder:

<pre>app.Use(next =&gt; new UfoMiddleware(next).Invoke);
</pre>

Aunque lo más habitual sería que UfoMiddleware proporcionase un método de extensión sobre IBuilder:

<pre>public static IBuilder UseUfo(this IBuilder builder)
{
    return builder.Use(n =&gt; new UfoMiddleware(n).Invoke);
}
</pre>

Y así podríamos llamar a app.UseUfo() en el Startup. La otra gran ventaja de los métodos de extensión es que permiten pasar parámetros de configuración a nuestro middleware. Así que bueno, lo normal es que cada componente venga con su método de extensión.

Ahora podemos modificar el método Invoke de nuestro UfoMiddleware para que haga "visible":

<pre>public async Task Invoke(HttpContext ctx)
{
    var path = ctx.Request.Path;
    if (ctx.Request.Path.Value == "/Help.ufo")
    {
        using (StreamWriter outfile = new StreamWriter(ctx.Response.Body))
        {
            outfile.Write("Generated by Ufo Middleware");
        }
    }
    await _next(ctx);
}
</pre>

Si ahora ejecutamos de nuevo nuestra aplicación y vamos a la URL /Help.ufo (ojo, que esa url es case-sensitive) veremos que por el navegador nos sale "Generated by Ufo Middleware". Perfecto! Ya hemos hecho un middleware de ASP.NET vNext funcional (vale, no es muy útil, pero funcional es).

En siguientes posts iré desgranando distintos aspectos de ASP.NET vNext porque la verdad... ¡es apasionante!