---
title: ASP.NET Core 2–Razor Pages
author: eiximenis

date: 2017-05-25T10:14:41+00:00
geeks_url: /?p=1884
geeks_ms_views:
  - 2957
categories:
  - asp.net 5
  - asp.net core

---
**Nota: Este post es sobre ASP.NET Core 2 Preview 1. Algunas cosas pueden cambiar en la versión final.**

¿Quien se acuerda de las ASP.NET Web Pages? Salieron tampoco hace tanto, más o menos junto con MVC3 y les acompañaba un producto propio (WebMatrix). Su objetivo era proporcionar un modelo de desarrollo basado en páginas (a lo Webforms) en contraposición del modelo basado en controladores de MVC.
  
He de reconocer que nunca les presté mucha atención y tengo la impresión que el resto del mundo tampoco. Su objetivo creo que era ofrecer una puerta de entrada rápida a ASP.NET ofreciendo un modelo sencillo de páginas. Su principal problema es que era difícil integrarlo en un proyecto “más grande” que estuviese hecho en ASP.NET MVC y así tener partes usando “Web Pages” y otras en MVC. Y, honestamente, montar un proyecto complejo en un modelo basado en páginas, no termino de verlo.
  
Personalmente las olvidé hace tiempo y por eso el anuncio de “Razor Pages” en ASP.NET Core 2 me sorprendió bastante. Pero la realidad es que Razor Pages es otra cosa bastante más interesante que las antiguas “Web Pages”…
  
<!--more-->


  
**Modelo basado en páginas**
  
Por supuesto la principal diferencia entre MVC y “Razor Pages” es que el primero es un modelo basado en controladores y el segundo basado en páginas: en MVC el punto de entrada de una petición es una acción de un controlador y en el segundo caso es una página Razor (un cshtml).
  
Reconozcámoslo: ¿cuantas acciones de este tipo tenemos?

<pre style="font-family: consolas; background: #1e1e1e; white-space: nowrap; overflow-x: scroll; color: gainsboro"><span style="color: #569cd6">public</span>&nbsp;<span style="color: #b8d7a3">IActionResult</span> Index()<br />{<br />&nbsp;&nbsp;&nbsp; <span style="color: #569cd6">return</span> View();<br />}</pre>

En este caso lo que únicamente queremos es devolver una vista cuando se invoque una URL. Oh, es cierto a veces queremos pasarle algún parámetro, pero la idea es la misma: devolver una vista y nada más.

<pre style="font-family: consolas; background: #1e1e1e; white-space: nowrap; overflow-x: scroll; color: gainsboro"><span style="color: #569cd6">public</span>&nbsp;<span style="color: #b8d7a3">IActionResult</span> Contact(<span style="color: #569cd6">string</span> name)<br />{<br />&nbsp;&nbsp;&nbsp; ViewData[<span style="color: #d69d85">"Message"</span>] <span style="color: #b4b4b4">=</span>&nbsp;<span style="color: #d69d85">$"Hi </span>{name}<span style="color: #d69d85"> this is the contact page"</span>;<br />&nbsp;&nbsp;&nbsp; <span style="color: #569cd6">return</span> View();<br />}</pre>

Para hacer esto hemos tenido que crear un controlador y una acción que apenas aportan valor: lo que queremos es devolver un vista.
  
¿No estaría bien tener un mecanismo para poder devolver la vista, sin necesidad de crear esos métodos que aportan bien poco?
  
Ahí es donde entra “Razor Pages”, que es un modelo **complementario a MVC**. Lo de complementario es importante: **en un mismo proyecto vamos a poder usar Razor Pages y MVC sin problemas**. Razor pages está integrado en MVC, no es un framework independiente como las antiguas “ASP.NET Web Pages”.
  
Si tienes el [SDK de NetCore 2][1] (Preview 1) instalado, puedes crear un proyecto MVC tradicional mediante “**dotnet new mvc**” (igual que con ASP.NET Core 1) o bien un proyecto de “Razor Pages” mediante “**dotnet new razor**”. Por supuesto si prefieres un IDE puedes usar [Visual Studio 2017 Update 3 Preview 1][2] (VS 15.3 para los amigos) que puedes instalar _side-by-side_ sin problemas con Visual Studio 2017. **Aunque tengas VS15.3 debes instalar el SDK de netcore 2**, pues no lo incluye de serie. Una ves lo hayas instalado, cuando crees un proyecto web nuevo netcore usando netcore 2, verás la nueva plantilla de Razor Pages:
  
[<img title="" style="border-left-width: 0px; border-right-width: 0px; background-image: none; border-bottom-width: 0px; padding-top: 0px; padding-left: 0px; display: inline; padding-right: 0px; border-top-width: 0px" border="0" alt="Visual Studio mostrando la plantilla de Razor Pages" src="https://geeks.ms/etomas/wp-content/uploads/sites/154/2017/05/image_thumb.png" width="644" height="422" />][3]
  
No te asustes por el hecho que una plantilla propia: Razor Pages forma parte de MVC y para demostrarlo **no vamos a usar la plantilla de Razor Pages si no que vamos a partir de un proyecto MVC tradicional** (plantilla Web Application en VS).
  
**Agregando Razor Pages al proyecto MVC**
  
Ahora que ya tenemos un proyecto MVC “tradicional” vamos a agregarle algunas páginas de Razor Pages en él. Para ello se sigue un modelo de convenciones muy sencillo: **del mismo modo que las vistas estan en una carpeta Views, las Razor Pages están en una carpeta Pages.** Así que creamos esa carpeta y ahora agregamos un fichero Razor en ella. Vamos a llamarlo Welcome.cshtml y va a tener el siguiente código:

<pre style="font-family: consolas; background: #1e1e1e; white-space: nowrap; overflow-x: scroll; color: gainsboro"><span style="background: #ffffb3; color: black">@page</span><br /><span style="color: gray">&lt;</span><span style="color: #569cd6">h1</span><span style="color: gray">&gt;</span>Welcome to Razor Pages<span style="color: gray">&lt;/</span><span style="color: #569cd6">h1</span><span style="color: gray">&gt;</span></pre>

Observa la directiva _“@page_”. Esa es la directiva que indica que este fichero debe tratarse como una “Razor Page”. Ya está, **no es necesario nada más**. Ya puedes poner en marcha tu proyecto MVC y navegar a /Welcome (sin el cshtml, por supuesto) y te aparecerá nuestra página. Y por supuesto si navegas a /Home/Index verás la vista Index servida por el controlador. Insisto **no se trata de MVC y Razor Pages trabajando juntos: se trata de que Razor Pages forma parte de MVC**.
  
**Compartiendo vistas de Layout**
  
Si observas la pestaña network y navegas a /Welcome, verás que el navegador ha recibido directamente el contenido de la página Welcome.cshtml: no se ha aplicado página de Layout alguna. ¿Como podemos compartir las vistas de Layout entre vistas MVC y páginas Razor?
  
Pues la respuesta es sorprendentemente simple:

<pre style="font-family: consolas; background: #1e1e1e; white-space: nowrap; overflow-x: scroll; color: gainsboro"><span style="background: #ffffb3; color: black">@page</span><br /><span style="background: #ffffb3; color: black">@{</span> Layout <span style="color: #b4b4b4">=</span>&nbsp;<span style="color: #d69d85">"~/Views/Shared/_Layout.cshtml"</span>; <span style="background: #ffffb3; color: black">}</span><br /><span style="color: gray">&lt;</span><span style="color: #569cd6">h1</span><span style="color: gray">&gt;</span>Welcome to Razor Pages<span style="color: gray">&lt;/</span><span style="color: #569cd6">h1</span><span style="color: gray">&gt;</span></pre>

En MVC no debes establecer el layout en cada vista, porque hay un fichero llamado _Viewstart.cshtml en la carpeta /Views que lo hace automáticamente. Lo mismo puede aplicarse en Razor Pages: **Puedes crear un fichero _Viewstart.cshtml en la carpeta /Pages** y establecer el Layout desde este fichero. Este fichero será aplicado automáticamente para cada llamada a una Razor Page (a imágen y semejanza del homónimo localizado en /Views).
  
**Enrutamiento en Razor Pages**
  
El enrutamiento es muy sencillo. Hay una regla que no podemos cambiar y es que **se enruta por el nombre de la página**. Eso significa que la página /Pages/Welcome.cshtml se enruta por /Welcome.cshtml. Eso no se puede cambiar (al menos de momento).
  
**Por supuesto podemos crear carpetas dentro de Pages** y eso se mapea en la URL. Es decir la página /Pages/Admin/Welcome.cshtml se enrutará a la URL /Admin/Welcome
  
**Lo que sí podemos es añadir route values** de forma muy sencilla, usando la directiva page:

<pre style="font-family: consolas; background: #1e1e1e; white-space: nowrap; overflow-x: scroll; color: gainsboro"><span style="background: #ffffb3; color: black">@page</span> "{name?}"<br /><span style="color: gray">&lt;</span><span style="color: #569cd6">h1</span><span style="color: gray">&gt;</span>Welcome to Razor Pages<span style="color: gray">&lt;/</span><span style="color: #569cd6">h1</span><span style="color: gray">&gt;</span></pre>

Hemos agregado el route value “name” (opcional) y ahora nuestra página es enrutable mediante /Welcome o bien /Welcome/xxxx (siendo xxxx cualquier cosa).
  
Por supuesto, tenemos que ver como obtener el valor del route&nbsp; value y eso nos lleva a hablar del…
  
**Model binding en Razor Pages**
  
A grandes rasgos el model binding en razor pages es el mismo que en MVC (no podía ser de otro modo). En MVC aplicamos el model binding en las acciones de los controladores… ¿pero donde lo aplicamos en Razor Pages? Pues bien, existen dos métodos especiales llamadas _OnGet_ y _OnPost_ que sirven para gestionar los GETs y los POSTs a una Razor Page. Son opcionales (obvio, pues hasta ahora no los hemos definido) pero si los creamos es ahí donde aplicamos el model binding:

<pre style="font-family: consolas; background: #1e1e1e; white-space: nowrap; overflow-x: scroll; color: gainsboro"><span style="background: #ffffb3; color: black">@page</span> "{name?}"<br /> <br /><span style="background: #ffffb3; color: black">@functions</span>&nbsp; <span style="background: #ffffb3; color: black">{</span><br />&nbsp;&nbsp;&nbsp; <span style="color: #569cd6">string</span> Name { <span style="color: #569cd6">get</span>; <span style="color: #569cd6">set</span>; }<br />&nbsp;&nbsp;&nbsp; <span style="color: #569cd6">public</span>&nbsp;<span style="color: #569cd6">void</span> OnGet(<span style="color: #569cd6">string</span> name)<br />&nbsp;&nbsp;&nbsp; {<br />&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; Name <span style="color: #b4b4b4">=</span> name <span style="color: #b4b4b4">??</span>&nbsp;<span style="color: #d69d85">"unknown user"</span>;<br />&nbsp;&nbsp;&nbsp; }<br /><span style="background: #ffffb3; color: black">}</span><br /> <br /><span style="color: gray">&lt;</span><span style="color: #569cd6">h1</span><span style="color: gray">&gt;</span>Welcome to Razor Pages <span style="background: #ffffb3; color: black">@</span>Name<span style="color: gray">&lt;/</span><span style="color: #569cd6">h1</span><span style="color: gray">&gt;</span></pre>

Observa como declaramos el método OnGet con el parámetro _name_ (que se enlaza al route value “name” gracias al model binding). Por lo tanto si ahora navego a /Welcome/eiximenis veré el mensaje “_Welcome to Razor Pages eiximenis_”.
  
Generalmente OnGet y OnPost se usan para obtener los datos de la petición y preparar el estado de la página antes de renderizarla. Observa que no usamos “Model” ni “ViewBag/ViewData”. En su lugar hemos creado una propiedad en la vista y usamos esta propiedad.
  
**Code Behind**
  
Si has arrugado la nariz por este bloque @functions y quieres tener las funciones (como OnGet, OnPost o cualquier adicional) en su propio fichero .cs, puedes hacerlo. Y tranquilo… si acabas de arrugar la nariz al leer “Code Behind” sigue leyendo, que ya verás que no es lo que parece.
  
Simplemente crea un fichero llamado como la página, pero con la extensión cshtml.cs. Asegúrate que esté en la misma carpeta que la página. En nuestro caso es Welcome.cshtml.cs:

<pre style="font-family: consolas; background: #1e1e1e; white-space: nowrap; overflow-x: scroll; color: gainsboro"><span style="color: #569cd6">public</span>&nbsp;<span style="color: #569cd6">class</span>&nbsp;<span style="color: #4ec9b0">WelcomeModel</span> : <span style="color: #4ec9b0">PageModel</span><br />{<br />&nbsp;&nbsp;&nbsp; <span style="color: #569cd6">public</span>&nbsp;<span style="color: #569cd6">string</span> Name { <span style="color: #569cd6">get</span>; <span style="color: #569cd6">set</span>; }<br />&nbsp;&nbsp;&nbsp; <span style="color: #569cd6">public</span>&nbsp;<span style="color: #569cd6">void</span> OnGet(<span style="color: #569cd6">string</span> name)<br />&nbsp;&nbsp;&nbsp; {<br />&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; Name <span style="color: #b4b4b4">=</span> name <span style="color: #b4b4b4">??</span>&nbsp;<span style="color: #d69d85">"unknown user"</span>;<br />&nbsp;&nbsp;&nbsp; }<br />}</pre>

Ahora ya podemos quitar el _functions_ de la página:

<pre style="font-family: consolas; background: #1e1e1e; white-space: nowrap; overflow-x: scroll; color: gainsboro"><span style="background: #ffffb3; color: black">@page</span> "{name?}"<br /><span style="background: #ffffb3; color: black">@model</span> WebApplication1<span style="color: #b4b4b4">.</span>Pages<span style="color: #b4b4b4">.</span><span style="color: #4ec9b0">WelcomeModel</span><br /> <br /><span style="color: gray">&lt;</span><span style="color: #569cd6">h1</span><span style="color: gray">&gt;</span>Welcome to Razor Pages <span style="background: #ffffb3; color: black">@</span>Model<span style="color: #b4b4b4">.</span>Name<span style="color: gray">&lt;/</span><span style="color: #569cd6">h1</span><span style="color: gray">&gt;</span></pre>

Observa dos cosas importantes:

  1. Hemos enlazado la página con su _code-behind_ mediante la directiva @model 
      * Ahora sí que usamos la propiedad Model</ol> 
    La clave es tener claro que el _code-behind_ no representa a la propia página (tal y como sucede en Webforms p. ej.) si no que representa al _ViewModel_ de la página. Pero es un _ViewModel_ especial, ya que tiene los métodos OnGet y OnPost_._
  
    Si tenemos que enlazar la página con su _viewmodel_ usando @model, igual te preguntas porque la clase debe estar en un fichero .cshtml.cs y con el mismo nombre que la página. Pues **la verdad es que no es necesario**. Puedes tener el modelo en el fichero que desees, en la carpeta que desees y con el nombre de clase y namespace que quieras. Así que realmente llamarle “code-behind” no es muy apropiado. Pero supongo que es lo que veremos habitualmente ya que la plantilla de VS lo crea de esta manera.
  
    **Métodos OnGet y OnPost**
  
    Hemos visto que estos métodos nos permiten aplicar el _model binding_ y que son invocados cuando se hace un GET (o un POST) a la página. Pero ¿sabes qué? En el fondo esos métodos son acciones de controlador camufladas, así que **pueden devolver un IActionResult**. Si no devuelven nada se asume que se quiere devolver la página, pero podemos devolver otras cosas:
    
    <pre style="font-family: consolas; background: #1e1e1e; white-space: nowrap; overflow-x: scroll; color: gainsboro"><span style="color: #569cd6">public</span>&nbsp;<span style="color: #569cd6">class</span>&nbsp;<span style="color: #4ec9b0">WelcomeModel</span> : <span style="color: #4ec9b0">PageModel</span><br />{<br />&nbsp;&nbsp;&nbsp; <span style="color: #569cd6">public</span>&nbsp;<span style="color: #569cd6">string</span> Name { <span style="color: #569cd6">get</span>; <span style="color: #569cd6">set</span>; }<br />&nbsp;&nbsp;&nbsp; <span style="color: #569cd6">public</span>&nbsp;<span style="color: #b8d7a3">IActionResult</span> OnGet(<span style="color: #569cd6">string</span> name)<br />&nbsp;&nbsp;&nbsp; {<br />&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span style="color: #569cd6">if</span> (<span style="color: #569cd6">string</span><span style="color: #b4b4b4">.</span>IsNullOrEmpty(name))<br />&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; {<br />&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span style="color: #569cd6">return</span> RedirectToAction(<span style="color: #d69d85">"Index"</span>, <span style="color: #d69d85">"Home"</span>);<br />&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }<br />&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; Name <span style="color: #b4b4b4">=</span> name;<br />&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span style="color: #569cd6">return</span> Page();<br />&nbsp;&nbsp;&nbsp; }<br />}</pre>
    
    Si ahora navegamos a /Welcome/edu se mostrará la página pero si navegamos a /Welcome como “name” será null, seremos redirigidos a la acción “Index” del controlador “Home”. Efectivamente, podemos redirigir desde una página a un controlador.
  
    Vale, observa el “_return Page()_” del final. Eso lo que hace es renderizar la página. Solo es necesario usar este método cuando OnGet (o OnPost) devuelven un IActionResult (como hemos comentado antes si devuelven void siempre se renderiza la página). Este método (a diferencia del “equivalente” MVC “return View()”) no acepta sobrecargas: recuerda que la clase _WelcomeModel_ es el propio view model de la vista, por lo que la vista siempre recibe la propia instancia.
  
    Los métodos _Page(), RedirectToAction_ y otros más estan proporcionados por la clase base PageModel.
  
    **Inyección de dependencias**
  
    Nada que destacar aquí. Podemos inyectar dependencias tanto en los _PageModels_ como en las páginas. Para los primeros podemos usar inyección en el constructor:
    
    <pre style="font-family: consolas; background: #1e1e1e; white-space: nowrap; overflow-x: scroll; color: gainsboro"><span style="color: #569cd6">public</span> WelcomeModel(<span style="color: #b8d7a3">IFoo</span> foo) <span style="color: #b4b4b4">=&gt;</span> _foo <span style="color: #b4b4b4">=</span> foo;<br /></pre>
    
    Eso funcionará (siempre y cuando tengamos el servicio registrado en el contenedor de DI, claro). Y para inyectar en las páginas, pues usamos la directiva @inject (tal y como podemos hacer con vistas MVC).
  
    **Conclusiones**
  
    Razor Pages **tiene muy poco que ver con las “ASP.NET Web Pages”**. Es más bien una versión ligera de ASP.NET MVC que elimina mucho del “boilerplate” (o código repetitivo) que se hace. En el fondo cada página Razor es un “mini controlador” con dos acciones (una enrutable por GET y otra por POST) y que actúa a la vez de modelo del código Razor.
  
    Además no es que se integre con MVC, es **que forma parte de ASP.NET MVC Core** (observa que hemos empezado con la plantilla de MVC y no hemos modificado nuestro Startup), con lo que no hay fricciones: siempre podemos empezar algo con Razor pages y pasar a MVC si vemos que necesitamos algo que Razor Pages no nos da. El hecho de que se puedan combinar en un mismo proyecto para mí marca una distinción clarísima y les da un ámbito de uso del que (en mi opinión) carecían las “ASP.NET Web Pages” antiguas.
  
    Por supuesto, hay que usarlas bien: son una herramienta más que tienen sus usos, pero personalmente considero que son una buena herramienta que nos puede ayudar a eliminar código “repetitivo” en nuestros proyectos!
  
    ¿Y tú, qué opinas? ¡Gracias!

 [1]: https://www.microsoft.com/net/core/preview#windowscmd
 [2]: https://www.visualstudio.com/vs/preview/
 [3]: https://geeks.ms/etomas/wp-content/uploads/sites/154/2017/05/image.png