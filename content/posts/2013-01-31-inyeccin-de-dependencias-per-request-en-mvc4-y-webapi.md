---
title: Inyección de dependencias per-request en MVC4 y WebApi
description: Inyección de dependencias per-request en MVC4 y WebApi
author: eiximenis

date: 2013-01-31T10:44:00+00:00
geeks_url: /?p=1629
geeks_visits:
  - 5395
geeks_ms_views:
  - 3134
categories:
  - Uncategorized

---
&iexcl;Muy buenas! Si desarrollais una aplicación web con MVC4 o bien una API REST con WebApi y usáis, pongamos, EF para acceder a la BBDD ya sabréis (y si no, os lo cuento ahora :P) que lo ideal es que el tiempo de vida del DbContext sea el de toda la petición web (lo mismo aplica a ISession si usáis NHibernate, por supuesto).

En muchos ejemplos y blogs se lee código parecido al siguiente (p.ej. en un repositorio/dao):

<div style="font-size: 10pt; font-family: consolas; background: #1e1e1e; color: #dcdcdc">
  <p style="margin: 0px">
    <span style="color: #569cd6">public</span> <span style="color: #b8d7a3">IEnumerable</span><span style="color: #b4b4b4"><</span><span style="color: #ff3333">MyEntity</span><span style="color: #b4b4b4">></span> <span style="color: white">GetAll</span>()
  </p>
  
  <p style="margin: 0px">
    {
  </p>
  
  <p style="margin: 0px">
    &nbsp;&nbsp;&nbsp; <span style="color: #569cd6">using</span> (<span style="color: white">_context</span> <span style="color: #b4b4b4">=</span> <span style="color: #569cd6">new</span> <span style="color: #ff3333">MyDbContext</span>())
  </p>
  
  <p style="margin: 0px">
    &nbsp;&nbsp;&nbsp; {
  </p>
  
  <p style="margin: 0px">
    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span style="color: #569cd6">return</span> <span style="color: white">_context</span><span style="color: #b4b4b4">.</span><span style="color: #ff3333">MyEntities</span><span style="color: #b4b4b4">.</span><span style="color: white">ToList</span>();
  </p>
  
  <p style="margin: 0px">
    &nbsp;&nbsp;&nbsp; }
  </p>
  
  <p style="margin: 0px">
    }
  </p>
</div>

Este código, aunque funciona, no es del todo correcto: Cada llamada a uno de esos métodos crea y destruye un contexto de Entity Framework. Sin entrar en consideraciones de rendimiento, esto hace que si dentro de una misma acción de un controlador llamamos a dos métodos de este repositorio (o bien de repositorios distintos) se crearán dos contextos de EF con todo lo que ello conlleva.

Por supuesto, irnos al extremo contrario, y guardar el contexto en una variable estática y abrirlo en el Application\_Start y cerrarlo en Application\_End es una idea terrible... para empezar todos los usuarios estarían compartiendo el mismo contexto de base de datos y además DbContext no es thread-safe (tampoco ISession de NHibernate lo es, si ya corrías a descargartelo).

¿Y guardar el contexto en una variable de sesión? &iexcl;Peor! Entonces tendrías un contexto abierto por cada usuario... idea terrible, pues si tu aplicación crece y tiene bastantes usuarios conectados a la vez, eso tumbaría tu base de datos. Además de que no hay nada que te garantice que todas las peticiones de un mismo usuario sean procesadas en un mismo thread.

¿La solución? Crear un contexto de BBDD por cada petición y cerrarlo una vez la petición se termina. De este modo, si dentro de la acción de un controlador llamas a 4 repositorios (por decir algo), esos 4 repositorios compartirán la instancia del contexto de bbdd. Al abrir y cerrar el contexto en cada petición podemos asegurar que no nos quedan contextos abiertos. A estos objetos que se crean y se destruyen una vez por cada petición, les decimos que tienen un tiempo de vida _per-request_.

Bien... veamos como podemos conseguir esto tanto en MVC4 como en WebApi, usando el contenedor IoC de Microsoft: Unity (en su versión 2.1).

Si no conoces nada acerca del funcionamiento de Unity, <a target="_blank" href="/blogs/etomas/archive/tags/unity/default.aspx" rel="noopener noreferrer">he escrito varios posts sobre él en este mismo blog</a>.

**A. Solución de MVC4**

MVC4 viene bastante bien preparada para IoC, de hecho hay varias maneras en como se puede configurar la inyección de dependencias en MVC4. Debemos tener presente que Unity, a diferencia de otros contenedores IoC no es capaz de gestionar automáticamente objetos con un tiempo de vida per-request, así que nos lo tendremos que currar nosotros, aunque no es excesivamente dificil. Para &ldquo;simular&rdquo; el tiempo de vida per-request, nos vamos a aprovechar de la característica de &ldquo;contenedor hijo&rdquo; que tiene Unity. En Unity, a partir de un contenedor padre, podemos crear un contenedor hijo, que obtiene todos los mapeos de tipo del padre y puede añadir los suyos. Cuando se destruye este contenedor hijo, todos los mapeos nuevos que tenga dicho contenedor son destruídos, y se llama a Dispose() de todos los objetos singleton gestionados por este controlador hijo (que sean IDisposable).

<span style="text-decoration: underline;">A.1 Crear un contenedor hijo en cada inicio de petición y destruirlo al final</span>

Ah... eso es fácil, ¿no? Nos basta usar Application_BeginRequest:

<div style="font-size: 10pt; font-family: consolas; background: #1e1e1e; color: #dcdcdc">
  <p style="margin: 0px">
    <span style="color: #569cd6">protected</span> <span style="color: #569cd6">void</span> <span style="color: white">Application_BeginRequest</span>()
  </p>
  
  <p style="margin: 0px">
    {
  </p>
  
  <p style="margin: 0px">
    &nbsp;&nbsp;&nbsp; <span style="color: #569cd6">var</span> <span style="color: white">childContainer</span> <span style="color: #b4b4b4">=</span> <span style="color: white">_container</span><span style="color: #b4b4b4">.</span><span style="color: white">CreateChildContainer</span>();
  </p>
  
  <p style="margin: 0px">
    &nbsp;&nbsp;&nbsp; <span style="color: #4ec9b0">HttpContext</span><span style="color: #b4b4b4">.</span><span style="color: white">Current</span><span style="color: #b4b4b4">.</span><span style="color: white">Items</span>[<span style="color: white">_unityGuid</span>] <span style="color: #b4b4b4">=</span> <span style="color: white">childContainer</span>;
  </p>
  
  <p style="margin: 0px">
    }
  </p>
</div>

Y Application_EndRequest, por supuesto:

<div style="font-size: 10pt; font-family: consolas; background: #1e1e1e; color: #dcdcdc">
  <p style="margin: 0px">
    <span style="color: #569cd6">protected</span> <span style="color: #569cd6">void</span> <span style="color: white">Application_EndRequest</span>()
  </p>
  
  <p style="margin: 0px">
    {
  </p>
  
  <p style="margin: 0px">
    &nbsp;&nbsp;&nbsp; <span style="color: #569cd6">var</span> <span style="color: white">childContainer</span> <span style="color: #b4b4b4">=</span> <span style="color: #4ec9b0">HttpContext</span><span style="color: #b4b4b4">.</span><span style="color: white">Current</span><span style="color: #b4b4b4">.</span><span style="color: white">Items</span>[<span style="color: white">_unityGuid</span>] <span style="color: #569cd6">as</span> <span style="color: #b8d7a3">IUnityContainer</span>;
  </p>
  
  <p style="margin: 0px">
    &nbsp;&nbsp;&nbsp; <span style="color: #569cd6">if</span> (<span style="color: white">childContainer</span> <span style="color: #b4b4b4">!=</span> <span style="color: #569cd6">null</span>)
  </p>
  
  <p style="margin: 0px">
    &nbsp;&nbsp;&nbsp; {
  </p>
  
  <p style="margin: 0px">
    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span style="color: white">childContainer</span><span style="color: #b4b4b4">.</span><span style="color: white">Dispose</span>();
  </p>
  
  <p style="margin: 0px">
    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span style="color: #4ec9b0">HttpContext</span><span style="color: #b4b4b4">.</span><span style="color: white">Current</span><span style="color: #b4b4b4">.</span><span style="color: white">Items</span><span style="color: #b4b4b4">.</span><span style="color: white">Remove</span>(<span style="color: white">_unityGuid</span>);
  </p>
  
  <p style="margin: 0px">
    &nbsp;&nbsp;&nbsp; }
  </p>
  
  <p style="margin: 0px">
    }
  </p>
</div>

Por si te lo preguntas \_container es el contendor de Unity principal y \_unityGuid no es nada más que un objeto para usar como clave en HttpContext. Ambos se inicializan en Application_Start y son estáticos:

<div style="font-size: 10pt; font-family: consolas; background: #1e1e1e; color: #dcdcdc">
  <p style="margin: 0px">
    <span style="color: #569cd6">private</span> <span style="color: #569cd6">static</span> <span style="color: #4ec9b0">Guid</span> <span style="color: white">_unityGuid</span>;
  </p>
  
  <p style="margin: 0px">
    <span style="color: #569cd6">private</span> <span style="color: #569cd6">static</span> <span style="color: #b8d7a3">IUnityContainer</span> <span style="color: white">_container</span>;
  </p>
  
  <p style="margin: 0px">
    &nbsp;
  </p>
  
  <p style="margin: 0px">
    <span style="color: #569cd6">protected</span> <span style="color: #569cd6">void</span> <span style="color: white">Application_Start</span>()
  </p>
  
  <p style="margin: 0px">
    {
  </p>
  
  <p style="margin: 0px">
    &nbsp;&nbsp;&nbsp; <span style="color: white">_container</span> <span style="color: #b4b4b4">=</span> <span style="color: #569cd6">new</span> <span style="color: #4ec9b0">UnityContainer</span>();
  </p>
  
  <p style="margin: 0px">
    &nbsp;&nbsp;&nbsp; <span style="color: white">_unityGuid</span> <span style="color: #b4b4b4">=</span> <span style="color: #4ec9b0">Guid</span><span style="color: #b4b4b4">.</span><span style="color: white">NewGuid</span>();
  </p>
</div>

<span style="text-decoration: underline;">A.2 Mapear los tipos en el contenedor de Unity</span>

Eso tampoco reviste especial complicación. La idea es:

  * Los singleton compartidos por todos los usuarios se declaran con el LifetimeManager _ContainerControlledLifetimeManager_ en el contenedor principal 
  * Los objetos de crear y destruir (p. ej. un repositorio o un controlador) se declaran con el LifetimeManager _TransientLifetimeManager_ en el contenedor princial 
  * Los objetos per-request (como el contexto de EF) se declaran con el LifetimeManager _HierarchicalLifetimeManager_ en el controlador principal. Otra opción sería declararlos como singletons (_ContainerControlledLifetimeManager_) en el contenedor hijo. 

> **Nota:** El lifetime manager HierarchicalLifetimeManager es equivalente al ContainerControlledLifetimeManager (es decir existe una sola instancia en el contenedor) pero con la salvedad de que los contenedors hijos **no comparten** la instancia del contenedor padre, si no que cada contenedor hijo puede tener la suya propia.

Sería algo así como:

<div style="font-size: 10pt; font-family: consolas; background: #1e1e1e; color: #dcdcdc">
  <p style="margin: 0px">
    <span style="color: #608b4e">// Objetos &#8220;de usar y tirar&#8221;:</span>
  </p>
  
  <p style="margin: 0px">
    <span style="color: white">container</span><span style="color: #b4b4b4">.</span><span style="color: white">RegisterType</span><span style="color: #b4b4b4"><</span><span style="color: #ff3333">IMyRepo</span>, <span style="color: #ff3333">MyRepository</span><span style="color: #b4b4b4">></span>();
  </p>
  
  <p style="margin: 0px">
    <span style="color: #608b4e">// Contexto EF</span>
  </p>
  
  <p style="margin: 0px">
    <span style="color: white">container</span><span style="color: #b4b4b4">.</span><span style="color: white">RegisterType</span><span style="color: #b4b4b4"><</span><span style="color: #ff3333">MyDbContext</span><span style="color: #b4b4b4">></span>(<span style="color: #569cd6">new</span> <span style="color: #4ec9b0">HierarchicalLifetimeManager</span>());
  </p>
</div>

<span style="text-decoration: underline;">A.3 Crear el &ldquo;activador de controladores&rdquo; (IControllerActivator)</span>

Una vez configurado el contenedor debemos decirle a MVC4 que lo use. Una de las formas de hacerlo es crear un _IControllerActivator_. Esto nos basta para la mayoría de casos (cuando tan solo vamos a inyectar dependencias en los controladores, lo que es ásí casi siempre). &acute;

La idea es crear los controladores **usando el controlador hijo** que hemos guardado en HttpContext.Items:

<div style="font-size: 10pt; font-family: consolas; background: #1e1e1e; color: #dcdcdc">
  <p style="margin: 0px">
    <span style="color: #569cd6">public</span> <span style="color: #569cd6">class</span> <span style="color: #4ec9b0">UnityControllerActivator</span> : <span style="color: #b8d7a3">IControllerActivator</span>
  </p>
  
  <p style="margin: 0px">
    {
  </p>
  
  <p style="margin: 0px">
    &nbsp;
  </p>
  
  <p style="margin: 0px">
    &nbsp;&nbsp;&nbsp; <span style="color: #569cd6">private</span> <span style="color: #4ec9b0">Guid</span> <span style="color: white">_containerGuid</span>;
  </p>
  
  <p style="margin: 0px">
    &nbsp;&nbsp;&nbsp; <span style="color: #569cd6">public</span> <span style="color: white">UnityControllerActivator</span>(<span style="color: #4ec9b0">Guid</span> <span style="color: white">containerGuid</span>)
  </p>
  
  <p style="margin: 0px">
    &nbsp;&nbsp;&nbsp; {
  </p>
  
  <p style="margin: 0px">
    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span style="color: white">_containerGuid</span> <span style="color: #b4b4b4">=</span> <span style="color: white">containerGuid</span>;
  </p>
  
  <p style="margin: 0px">
    &nbsp;&nbsp;&nbsp; }
  </p>
  
  <p style="margin: 0px">
    &nbsp;&nbsp;&nbsp; <span style="color: #569cd6">public</span> <span style="color: #b8d7a3">IController</span> <span style="color: white">Create</span>(<span style="color: #4ec9b0">RequestContext</span> <span style="color: white">requestContext</span>, <span style="color: #4ec9b0">Type</span> <span style="color: white">controllerType</span>)
  </p>
  
  <p style="margin: 0px">
    &nbsp;&nbsp;&nbsp; {
  </p>
  
  <p style="margin: 0px">
    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span style="color: #569cd6">var</span> <span style="color: white">container</span> <span style="color: #b4b4b4">=</span> <span style="color: white">requestContext</span><span style="color: #b4b4b4">.</span><span style="color: white">HttpContext</span><span style="color: #b4b4b4">.</span><span style="color: white">Items</span>[<span style="color: white">_containerGuid</span>] <span style="color: #569cd6">as</span> <span style="color: #b8d7a3">IUnityContainer</span>;
  </p>
  
  <p style="margin: 0px">
    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span style="color: #569cd6">if</span> (<span style="color: white">container</span> <span style="color: #b4b4b4">!=</span> <span style="color: #569cd6">null</span>)
  </p>
  
  <p style="margin: 0px">
    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; {
  </p>
  
  <p style="margin: 0px">
    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span style="color: #569cd6">return</span> <span style="color: white">container</span><span style="color: #b4b4b4">.</span><span style="color: white">Resolve</span>(<span style="color: white">controllerType</span>) <span style="color: #569cd6">as</span> <span style="color: #b8d7a3">IController</span>;
  </p>
  
  <p style="margin: 0px">
    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }
  </p>
  
  <p style="margin: 0px">
    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span style="color: #569cd6">return</span> <span style="color: #569cd6">null</span>;
  </p>
  
  <p style="margin: 0px">
    &nbsp;&nbsp;&nbsp; }
  </p>
  
  <p style="margin: 0px">
    }
  </p>
</div>

Fijaos que simplemente llamamos a &ldquo;Resolve&rdquo; del contenedor que obtenemos de HttpContext.Items.

<span style="text-decoration: underline;">A.4 Configurar MVC4 para que use nuestro activador de controladores</span>

Para que MVC4 use nuestro activador de controladores, debemos implementar un dependency resolver. A diferencia de la factoría de controladores p.ej, que se puede establecer a través de una clase estática), no hay ningún mecanismo para configurar que activador de controladores usará MVC. Si queremos usar uno específico (como es nuestro caso) debemos establecer un dependency resolver.

Para ello lo suyo es crearse uno que use el propio contenedor de Unity:

<div style="font-size: 10pt; font-family: consolas; background: #1e1e1e; color: #dcdcdc">
  <p style="margin: 0px">
    <span style="color: #569cd6">public</span> <span style="color: #569cd6">class</span> <span style="color: #4ec9b0">MvcConfig</span>
  </p>
  
  <p style="margin: 0px">
    {
  </p>
  
  <p style="margin: 0px">
    &nbsp;&nbsp;&nbsp; <span style="color: #569cd6">public</span> <span style="color: #569cd6">static</span> <span style="color: #569cd6">void</span> <span style="color: white">Register</span>(<span style="color: #b8d7a3">IUnityContainer</span> <span style="color: white">container</span>, <span style="color: #4ec9b0">Guid</span> <span style="color: white">containerGuid</span>)
  </p>
  
  <p style="margin: 0px">
    &nbsp;&nbsp;&nbsp; {
  </p>
  
  <p style="margin: 0px">
    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span style="color: white">container</span><span style="color: #b4b4b4">.</span><span style="color: white">RegisterInstance</span>(<span style="color: #569cd6">typeof</span> (<span style="color: #b8d7a3">IControllerActivator</span>), <span style="color: #569cd6">new</span> <span style="color: #4ec9b0">UnityControllerActivator</span>(<span style="color: white">containerGuid</span>));
  </p>
  
  <p style="margin: 0px">
    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span style="color: #569cd6">foreach</span> (<span style="color: #569cd6">var</span> <span style="color: white">type</span> <span style="color: #569cd6">in</span>
  </p>
  
  <p style="margin: 0px">
    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span style="color: #4ec9b0">Assembly</span><span style="color: #b4b4b4">.</span><span style="color: white">GetExecutingAssembly</span>()<span style="color: #b4b4b4">.</span><span style="color: white">GetExportedTypes</span>()<span style="color: #b4b4b4">.</span>
  </p>
  
  <p style="margin: 0px">
    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span style="color: white">Where</span>(<span style="color: white">x</span> <span style="color: #b4b4b4">=></span> <span style="color: white">x</span><span style="color: #b4b4b4">.</span><span style="color: white">GetInterface</span>(<span style="color: #569cd6">typeof</span>(<span style="color: #b8d7a3">IController</span>)<span style="color: #b4b4b4">.</span><span style="color: white">Name</span>) <span style="color: #b4b4b4">!=</span> <span style="color: #569cd6">null</span>))
  </p>
  
  <p style="margin: 0px">
    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; {
  </p>
  
  <p style="margin: 0px">
    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span style="color: white">container</span><span style="color: #b4b4b4">.</span><span style="color: white">RegisterType</span>(<span style="color: white">type</span>);
  </p>
  
  <p style="margin: 0px">
    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }
  </p>
  
  <p style="margin: 0px">
    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span style="color: #4ec9b0">DependencyResolver</span><span style="color: #b4b4b4">.</span><span style="color: white">SetResolver</span>(
  </p>
  
  <p style="margin: 0px">
    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span style="color: white">t</span> <span style="color: #b4b4b4">=></span> <span style="color: white">container</span><span style="color: #b4b4b4">.</span><span style="color: white">IsRegistered</span>(<span style="color: white">t</span>) <span style="color: #b4b4b4">?</span> <span style="color: white">container</span><span style="color: #b4b4b4">.</span><span style="color: white">Resolve</span>(<span style="color: white">t</span>) : <span style="color: #569cd6">null</span>,
  </p>
  
  <p style="margin: 0px">
    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span style="color: white">t</span> <span style="color: #b4b4b4">=></span> <span style="color: white">container</span><span style="color: #b4b4b4">.</span><span style="color: white">IsRegistered</span>(<span style="color: white">t</span>) <span style="color: #b4b4b4">?</span> <span style="color: white">container</span><span style="color: #b4b4b4">.</span><span style="color: white">ResolveAll</span>(<span style="color: white">t</span>) : <span style="color: #4ec9b0">Enumerable</span><span style="color: #b4b4b4">.</span><span style="color: white">Empty</span><span style="color: #b4b4b4"><</span><span style="color: #569cd6">object</span><span style="color: #b4b4b4">></span>());
  </p>
  
  <p style="margin: 0px">
    &nbsp;&nbsp;&nbsp; }
  </p>
  
  <p style="margin: 0px">
    }
  </p>
</div>

La llamada al método MvcConfig.Register iría en el Application_Start una vez creado el contenedor principal. Este método hace 3 cosas:

  1. Registra el activador de controladores como singleton en el contenedor. 
  2. Registra todos los controladores que haya 
  3. Finalmente crea el dependency resolver que usa el propio contenedor. 

Con esto ya podemos inyectar las dependencias en nuestros controladores y en nuestros repositorios (a los controladores les inyectaríamos los repositorios y a los repositorios el contexto de EF).

**B. Solución WebApi**

Aunque, por oscuras razones, WebApi se distribuye junto a MVC y parece mucho lo mismo en el fondo hay muchas diferencias entre MVC y WebApi. Y una de esas diferencias es como funciona el tema de inyección de dependencias. Por lo tanto lo que hemos dicho de MVC no funciona para WebApi.

Si queremos obtener lo mismo (es decir que el contexto de EF tenga un tiempo de vida per-request) con WebApi, debemos seguir una estrategia distinta...

Lo que sí es igual es la configuración del contenedor (se siguen las mismas directrices que el punto A.2).

<span style="text-decoration: underline;">B.1 Crear un DependencyResolver de WebApi</span>

Al igual que en MVC, necesitamos un dependency resolver, pero el de WebApi es distinto que el de MVC. Así si en MVC usábamos el método estático SetResolver de la clase DependencyResolver, en WebApi usaremos la propiedad DependencyResolver del objeto HttpConfiguration global, al que podemos acceder a través de _GlobalConfiguration.Configuration_.

A diferencia de MVC donde podemos pasar directamente dos delegates para crear el dependency resolver, en WebApi estamos obligados a crear una clase que implemente _IDependencyResolver_.

Además WebApi, a diferencia de MVC, ya viene &ldquo;más o menos&rdquo; preparado para el concepto de objetos per-request. Existe un método dentro de IDependencyResolver, llamado _BeginScope_. La idea es que cada vez que se necesite crear un &ldquo;scope hijo&rdquo; se llame a este método. Lo bonito sería que el framework lo hiciese por nosotros al crear un controlador, pero la realidad es que no lo hace. Nos tocará hacerlo nosotros.

Así para implementar _IDependencyResolver_ vamos a usar dos clases:

<div style="font-size: 10pt; font-family: consolas; background: #1e1e1e; color: #dcdcdc">
  <p style="margin: 0px">
    <span style="color: #569cd6">public</span> <span style="color: #569cd6">class</span> <span style="color: #4ec9b0">UnityDependencyScope</span> : <span style="color: #b8d7a3">IDependencyScope</span>
  </p>
  
  <p style="margin: 0px">
    {
  </p>
  
  <p style="margin: 0px">
    &nbsp;&nbsp;&nbsp; <span style="color: #569cd6">private</span> <span style="color: #b8d7a3">IUnityContainer</span> <span style="color: white">_container</span>;
  </p>
  
  <p style="margin: 0px">
    &nbsp;&nbsp;&nbsp; <span style="color: #569cd6">protected</span> <span style="color: #b8d7a3">IUnityContainer</span> <span style="color: white">Container</span> { <span style="color: #569cd6">get</span> { <span style="color: #569cd6">return</span> <span style="color: white">_container</span>; } }
  </p>
  
  <p style="margin: 0px">
    &nbsp;&nbsp;&nbsp; <span style="color: #569cd6">public</span> <span style="color: white">UnityDependencyScope</span>(<span style="color: #b8d7a3">IUnityContainer</span> <span style="color: white">container</span>)
  </p>
  
  <p style="margin: 0px">
    &nbsp;&nbsp;&nbsp; {
  </p>
  
  <p style="margin: 0px">
    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span style="color: white">_container</span> <span style="color: #b4b4b4">=</span> <span style="color: white">container</span>;
  </p>
  
  <p style="margin: 0px">
    &nbsp;&nbsp;&nbsp; }
  </p>
  
  <p style="margin: 0px">
    &nbsp;&nbsp;&nbsp; <span style="color: #569cd6">public</span> <span style="color: #569cd6">void</span> <span style="color: white">Dispose</span>()
  </p>
  
  <p style="margin: 0px">
    &nbsp;&nbsp;&nbsp; {
  </p>
  
  <p style="margin: 0px">
    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span style="color: white">_container</span><span style="color: #b4b4b4">.</span><span style="color: white">Dispose</span>();
  </p>
  
  <p style="margin: 0px">
    &nbsp;&nbsp;&nbsp; }
  </p>
  
  <p style="margin: 0px">
    &nbsp;&nbsp;&nbsp; <span style="color: #569cd6">public</span> <span style="color: #569cd6">object</span> <span style="color: white">GetService</span>(<span style="color: #4ec9b0">Type</span> <span style="color: white">serviceType</span>)
  </p>
  
  <p style="margin: 0px">
    &nbsp;&nbsp;&nbsp; {
  </p>
  
  <p style="margin: 0px">
    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span style="color: #569cd6">return</span> <span style="color: white">_container</span><span style="color: #b4b4b4">.</span><span style="color: white">IsRegistered</span>(<span style="color: white">serviceType</span>)
  </p>
  
  <p style="margin: 0px">
    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span style="color: #b4b4b4">?</span> <span style="color: white">_container</span><span style="color: #b4b4b4">.</span><span style="color: white">Resolve</span>(<span style="color: white">serviceType</span>)
  </p>
  
  <p style="margin: 0px">
    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; : <span style="color: #569cd6">null</span>;
  </p>
  
  <p style="margin: 0px">
    &nbsp;&nbsp;&nbsp; }
  </p>
  
  <p style="margin: 0px">
    &nbsp;&nbsp;&nbsp; <span style="color: #569cd6">public</span> <span style="color: #b8d7a3">IEnumerable</span><span style="color: #b4b4b4"><</span><span style="color: #569cd6">object</span><span style="color: #b4b4b4">></span> <span style="color: white">GetServices</span>(<span style="color: #4ec9b0">Type</span> <span style="color: white">serviceType</span>)
  </p>
  
  <p style="margin: 0px">
    &nbsp;&nbsp;&nbsp; {
  </p>
  
  <p style="margin: 0px">
    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span style="color: #569cd6">return</span> <span style="color: white">_container</span><span style="color: #b4b4b4">.</span><span style="color: white">ResolveAll</span>(<span style="color: white">serviceType</span>);
  </p>
  
  <p style="margin: 0px">
    &nbsp;&nbsp;&nbsp; }
  </p>
  
  <p style="margin: 0px">
    }
  </p>
</div>

Esta primera clase implementa _IDependencyScope_. Esta interfaz es la interfaz base de _IDependencyResolver_. De hecho _IDependencyResolver_ añade tan solo el método BeginScope. La segunda clase es la que implementa _IDependencyResolver_:

<div style="font-size: 10pt; font-family: consolas; background: #1e1e1e; color: #dcdcdc">
  <p style="margin: 0px">
    <span style="color: #569cd6">public</span> <span style="color: #569cd6">class</span> <span style="color: #4ec9b0">UnityDependencyResolver</span> : <span style="color: #4ec9b0">UnityDependencyScope</span>, <span style="color: #b8d7a3">IDependencyResolver</span>
  </p>
  
  <p style="margin: 0px">
    {
  </p>
  
  <p style="margin: 0px">
    &nbsp;&nbsp;&nbsp; <span style="color: #569cd6">public</span> <span style="color: white">UnityDependencyResolver</span>(<span style="color: #b8d7a3">IUnityContainer</span> <span style="color: white">container</span>) : <span style="color: #569cd6">base</span> (<span style="color: white">container</span>)
  </p>
  
  <p style="margin: 0px">
    &nbsp;&nbsp;&nbsp; {
  </p>
  
  <p style="margin: 0px">
    &nbsp;&nbsp;&nbsp; }
  </p>
  
  <p style="margin: 0px">
    &nbsp;&nbsp;&nbsp; <span style="color: #569cd6">public</span> <span style="color: #b8d7a3">IDependencyScope</span> <span style="color: white">BeginScope</span>()
  </p>
  
  <p style="margin: 0px">
    &nbsp;&nbsp;&nbsp; {
  </p>
  
  <p style="margin: 0px">
    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span style="color: #569cd6">return</span> <span style="color: #569cd6">new</span> <span style="color: #4ec9b0">UnityDependencyScope</span>(<span style="color: white">Container</span><span style="color: #b4b4b4">.</span><span style="color: white">CreateChildContainer</span>());
  </p>
  
  <p style="margin: 0px">
    &nbsp;&nbsp;&nbsp; }
  </p>
  
  <p style="margin: 0px">
    }
  </p>
</div>

Implementamos el método BeginScope retornando un contenedor hijo de nuestro contenedor. De esta manera ya podemos &ldquo;encadenar&rdquo; scopes hijos.

<span style="text-decoration: underline;">B.2 Crear el activador de controladores</span>

Al igual que en MVC debemos crear un activador de controladores propio. Y por supuesto es distinto del de MVC 🙂

En este caso debemos implementar IHttpControllerActivator:

<div style="font-size: 10pt; font-family: consolas; background: #1e1e1e; color: #dcdcdc">
  <p style="margin: 0px">
    <span style="color: #569cd6">public</span> <span style="color: #569cd6">class</span> <span style="color: #4ec9b0">UnityHttpControllerActivator</span> : <span style="color: #b8d7a3">IHttpControllerActivator</span>
  </p>
  
  <p style="margin: 0px">
    {
  </p>
  
  <p style="margin: 0px">
    &nbsp;&nbsp;&nbsp; <span style="color: #569cd6">private</span> <span style="color: #b8d7a3">IUnityContainer</span> <span style="color: white">_container</span>;
  </p>
  
  <p style="margin: 0px">
    &nbsp;&nbsp;&nbsp; <span style="color: #569cd6">public</span> <span style="color: white">UnityHttpControllerActivator</span>(<span style="color: #b8d7a3">IUnityContainer</span> <span style="color: white">container</span>)
  </p>
  
  <p style="margin: 0px">
    &nbsp;&nbsp;&nbsp; {
  </p>
  
  <p style="margin: 0px">
    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span style="color: white">_container</span> <span style="color: #b4b4b4">=</span> <span style="color: white">container</span>;
  </p>
  
  <p style="margin: 0px">
    &nbsp;&nbsp;&nbsp; }
  </p>
  
  <p style="margin: 0px">
    &nbsp;&nbsp;&nbsp; <span style="color: #569cd6">public</span> <span style="color: #b8d7a3">IHttpController</span> <span style="color: white">Create</span>(<span style="color: #4ec9b0">HttpRequestMessage</span> <span style="color: white">request</span>, <span style="color: #4ec9b0">HttpControllerDescriptor</span> <span style="color: white">controllerDescriptor</span>, <span style="color: #4ec9b0">Type</span> <span style="color: white">controllerType</span>)
  </p>
  
  <p style="margin: 0px">
    &nbsp;&nbsp;&nbsp; {
  </p>
  
  <p style="margin: 0px">
    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span style="color: #569cd6">var</span> <span style="color: white">scope</span> <span style="color: #b4b4b4">=</span> <span style="color: white">request</span><span style="color: #b4b4b4">.</span><span style="color: white">GetDependencyScope</span>();
  </p>
  
  <p style="margin: 0px">
    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span style="color: #569cd6">return</span> <span style="color: white">scope</span><span style="color: #b4b4b4">.</span><span style="color: white">GetService</span>(<span style="color: white">controllerType</span>) <span style="color: #569cd6">as</span> <span style="color: #b8d7a3">IHttpController</span>;
  </p>
  
  <p style="margin: 0px">
    &nbsp;&nbsp;&nbsp; }
  </p>
  
  <p style="margin: 0px">
    }
  </p>
</div>

La clave está en la llamada a GetDependencyScope del parámetro request. Esto creará un scope (llamando a BeginScope) por lo que el controlador que creemos se creará con el contenedor Unity hijo.

<span style="text-decoration: underline;">B.3 Configurar WebApi para que use nuestro activador de controladores</span>

Eso si que es realmente sencillo. Simplemente registramos el activador de controladores y el dependency resolver en WebApi. Esto lo hacemos en el Application_Start, p.ej. en el propio método Register de la clase WebApiConfig que crea Visual Studio:

<div style="font-size: 10pt; font-family: consolas; background: #1e1e1e; color: #dcdcdc">
  <p style="margin: 0px">
    <span style="color: #569cd6">public</span> <span style="color: #569cd6">static</span> <span style="color: #569cd6">void</span> <span style="color: white">Register</span>(<span style="color: #4ec9b0">HttpConfiguration</span> <span style="color: white">config</span>, <span style="color: #b8d7a3">IUnityContainer</span> <span style="color: white">container</span>)
  </p>
  
  <p style="margin: 0px">
    {
  </p>
  
  <p style="margin: 0px">
    &nbsp;&nbsp;&nbsp; <span style="color: white">config</span><span style="color: #b4b4b4">.</span><span style="color: white">DependencyResolver</span> <span style="color: #b4b4b4">=</span> <span style="color: #569cd6">new</span> <span style="color: #4ec9b0">UnityDependencyResolver</span>(<span style="color: white">container</span>);
  </p>
  
  <p style="margin: 0px">
    &nbsp;&nbsp;&nbsp; <span style="color: white">container</span><span style="color: #b4b4b4">.</span><span style="color: white">RegisterInstance</span><span style="color: #b4b4b4"><</span><span style="color: #b8d7a3">IHttpControllerActivator</span><span style="color: #b4b4b4">></span>(<span style="color: #569cd6">new</span> <span style="color: #4ec9b0">UnityHttpControllerActivator</span>(<span style="color: white">container</span>));
  </p>
  
  <p style="margin: 0px">
    &nbsp;&nbsp;&nbsp; <span style="color: #569cd6">foreach</span> (<span style="color: #569cd6">var</span> <span style="color: white">type</span> <span style="color: #569cd6">in</span>
  </p>
  
  <p style="margin: 0px">
    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span style="color: #4ec9b0">Assembly</span><span style="color: #b4b4b4">.</span><span style="color: white">GetExecutingAssembly</span>()<span style="color: #b4b4b4">.</span><span style="color: white">GetExportedTypes</span>()<span style="color: #b4b4b4">.</span>
  </p>
  
  <p style="margin: 0px">
    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span style="color: white">Where</span>(<span style="color: white">x</span> <span style="color: #b4b4b4">=></span> <span style="color: white">x</span><span style="color: #b4b4b4">.</span><span style="color: white">GetInterface</span>(<span style="color: #569cd6">typeof</span>(<span style="color: #b8d7a3">IHttpController</span>)<span style="color: #b4b4b4">.</span><span style="color: white">Name</span>) <span style="color: #b4b4b4">!=</span> <span style="color: #569cd6">null</span>))
  </p>
  
  <p style="margin: 0px">
    &nbsp;&nbsp;&nbsp; {
  </p>
  
  <p style="margin: 0px">
    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span style="color: white">container</span><span style="color: #b4b4b4">.</span><span style="color: white">RegisterType</span>(<span style="color: white">type</span>);
  </p>
  
  <p style="margin: 0px">
    &nbsp;&nbsp;&nbsp; }
  </p>
  
  <p style="margin: 0px">
    }
  </p>
</div>

También debemos registrar todos los controladores de WebApi (de una forma análoga a la que usábamos al registrar los controladores MVC).

**¿Y si tengo WebApi junto a MVC en un mismo proyecto?**

Pues te tocará hacer todo lo de MVC y todo lo de WebApi. Lo único que no haces dos veces es la configuración del contenedor (punto A.2) ya que es la misma.

Eso sí, en este caso puedes hacer que si la petición HTTP es una llamada a WebApi, en lugar de una llamada a un controlador MVC, no se cree el contenedor hijo en Application\_BeginRequest (ni se destruya en Application\_EndRequest). Algo como:

<div style="font-size: 10pt; font-family: consolas; background: #1e1e1e; color: #dcdcdc">
  <p style="margin: 0px">
    <span style="color: #569cd6">protected</span> <span style="color: #569cd6">void</span> <span style="color: white">Application_BeginRequest</span>()
  </p>
  
  <p style="margin: 0px">
    {
  </p>
  
  <p style="margin: 0px">
    &nbsp;&nbsp;&nbsp; <span style="color: #569cd6">if</span> (<span style="color: #b4b4b4">!</span><span style="color: white">Request</span><span style="color: #b4b4b4">.</span><span style="color: white">IsWebApiRequest</span>())
  </p>
  
  <p style="margin: 0px">
    &nbsp;&nbsp;&nbsp; {
  </p>
  
  <p style="margin: 0px">
    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span style="color: #569cd6">// Código para MVC</span>
  </p>
  
  <p style="margin: 0px">
    &nbsp;&nbsp;&nbsp; }
  </p>
  
  <p style="margin: 0px">
    }
  </p>
</div>

El método IsWebApiRequest es un método de extensión que me he inventado. La verdad, no tenía nada claro como diferenciar una petición de WebApi de una de MVC así que he cortado por lo sano:

<div style="font-size: 10pt; font-family: consolas; background: #1e1e1e; color: #dcdcdc">
  <p style="margin: 0px">
    <span style="color: #569cd6">public</span> <span style="color: #569cd6">static</span> <span style="color: #569cd6">class</span> <span style="color: #4ec9b0">HttpRequestExtensions</span>
  </p>
  
  <p style="margin: 0px">
    {
  </p>
  
  <p style="margin: 0px">
    &nbsp;&nbsp;&nbsp; <span style="color: #569cd6">public</span> <span style="color: #569cd6">static</span> <span style="color: #569cd6">bool</span> <span style="color: white">IsWebApiRequest</span>(<span style="color: #569cd6">this</span> <span style="color: #4ec9b0">HttpRequest</span> <span style="color: white">@this</span>)
  </p>
  
  <p style="margin: 0px">
    &nbsp;&nbsp;&nbsp; {
  </p>
  
  <p style="margin: 0px">
    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span style="color: #569cd6">return</span> <span style="color: white">@this</span><span style="color: #b4b4b4">.</span><span style="color: white">FilePath</span><span style="color: #b4b4b4">.</span><span style="color: white">StartsWith</span>(<span style="color: #d69d85">&#8220;/api/&#8221;</span>);
  </p>
  
  <p style="margin: 0px">
    &nbsp;&nbsp;&nbsp; }
  </p>
  
  <p style="margin: 0px">
    }
  </p>
</div>

Exacto... Si la URL empieza por /api/ es que es una llamada a WebApi. Ya... cutre, pero oye tu: funciona 😛 Por supuesot, dependiendo de la tabla de rutas que tengas eso puede no ser cierto en tu caso 😉

Y nada más... con eso conseguimos un tiempo de vida per-request tanto en ASP.NET MVC4 como con WebApi 🙂

Saludos!