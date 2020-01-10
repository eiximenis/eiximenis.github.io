---
title: Sirviendo ASP.NET vNext con Nowin

author: eiximenis

date: 2014-06-11T11:58:03+00:00
geeks_url: /?p=1669
geeks_visits:
  - 1374
geeks_ms_views:
  - 1666
categories:
  - Uncategorized

---
> **Nota:** Este post está realizado sobre la CTP de VS2014 y la alfa de ASP.NET vNext. Todo lo dicho puede cambiar cuando salgan las versiones RTM…

¿Conoces <a href="https://github.com/Bobris/Nowin" target="_blank" rel="noopener noreferrer">Nowin</a>? Es un servidor web <a href="http://owin.org/" target="_blank" rel="noopener noreferrer">OWIN</a>, es decir si construimos nuestra aplicación web basándonos en middleware OWIN (p. ej. WebApi 2 o NancyFx) podemos usar este servidor web para servirla a los clientes. Ya he hablado de OWIN varias veces antes, ya que OWIN ha sido la penúltima revolución en desarrollo de aplicaciones web en tecnología Microsot (la última es vNext, claro). Microsoft hizo un esfuerzo implementando middleware OWIN y permitiendo su integración con ASP.NET (el proyecto Katana), pero ahora sale ASP.NET vNext y es lógico preguntarse… ¿en qué queda todo?

Básicamente la pregunta es si ASP.NET vNext es compatible con OWIN y si lo es, en que terminos: ¿podemos añadir componentes OWIN al pipeline de vNext y podemos usar componentes vNext como si fuesen componentes OWIN?

La respuesta rápida a las tres preguntas es sí. ASP.NET vNext es compatible con OWIN y podemos tanto añadir componentes OWIN a nuestro pipeline de ASP.NET vNext como usar un componente vNext como si fuese middleware OWIN. De hecho cualquier otra respuesta sería totalmente incomprensible.

La idea de modularización de la aplicación en componentes que tiene ASP.NET vNext está sacada directamente de OWIN (en Katana se usa IAppBuilder lo que generó un <a href="https://github.com/owin/owin/issues/19" target="_blank" rel="noopener noreferrer">interesantísimo debate en la comunidad</a>).

Bueno, vayamos al tajo… intentemos ver como podemos servir una aplicación ASP.NET vNext (MVC6) pero usando un servidor OWIN como Nowin para ello. Lo primero es crear una “ASP.NET vNext **Console** Application” usando VS2014 CTP… Eso nos permitirá ver como hospedar ASP.NET vNext en una aplicación de consola 😉

El siguiente paso será instalar MVC6 en nuestra aplicacion. **No usaremos NuGet para ello**, en su lugar editaremos el fichero project.json que trae VS2104 y añadiremos las siguientes líneas en la sección _dependencies_:

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:083a97ee-f2f7-40ff-9196-6c2b9e6df997" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
    <div style="background: #ddd; max-height: 300px; overflow: auto">
      <ol start="1" style="background: #1d1d1d; margin: 0 0 0 2em; padding: 0 0 0 5px; white-space: nowrap">
        <li>
          <span style="background:#1e1e1e;color:#d7ba7d">"Microsoft.AspNet.Mvc"</span><span style="background:#1e1e1e;color:#b4b4b4">:</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#d69d85">"0.1-alpha-build-*"</span>
        </li>
      </ol>
    </div></p>
  </div></p>
</div>

Por suerte VS2014 nos ofrece intellisense al editar project.json (aunque parecen **no salir todas** las referencias de paquetes NuGet):

[<img title="image" style="border-left-width: 0px; border-right-width: 0px; background-image: none; border-bottom-width: 0px; padding-top: 0px; padding-left: 0px; display: inline; padding-right: 0px; border-top-width: 0px" border="0" alt="image" src="http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_02267DFC.png" width="504" height="135" />][1]

Ahora añade un controlador de MVC a tu aplicación (HomeController) con una acción Index que simplemente retorne la vista asociada:

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:287fa0a6-3872-4254-800c-aab081ecd796" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
    <div style="background: #ddd; max-height: 300px; overflow: auto">
      <ol start="1" style="background: #1d1d1d; margin: 0 0 0 2.5em; padding: 0 0 0 5px; white-space: nowrap">
        <li>
          <span style="background:#1e1e1e;color:#569cd6">using</span><span style="background:#1e1e1e;color:#dcdcdc"> Microsoft</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">AspNet</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">Mvc;</span>
        </li>
        <li style="background: #111111">
          &nbsp;
        </li>
        <li>
          <span style="background:#1e1e1e;color:#569cd6">namespace</span><span style="background:#1e1e1e;color:#dcdcdc"> NowinDemo</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">Controllers</span>
        </li>
        <li style="background: #111111">
          <span style="background:#1e1e1e;color:#dcdcdc">{</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">    </span><span style="background:#1e1e1e;color:#569cd6">public</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">class</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#4ec9b0">HomeController</span><span style="background:#1e1e1e;color:#dcdcdc"> : </span><span style="background:#1e1e1e;color:#4ec9b0">Controller</span>
        </li>
        <li style="background: #111111">
          <span style="background:#1e1e1e;color:#dcdcdc">    {</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">        </span><span style="background:#1e1e1e;color:#569cd6">public</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#b8d7a3">IActionResult</span><span style="background:#1e1e1e;color:#dcdcdc"> Index()</span>
        </li>
        <li style="background: #111111">
          <span style="background:#1e1e1e;color:#dcdcdc">        {</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">            </span><span style="background:#1e1e1e;color:#569cd6">return</span><span style="background:#1e1e1e;color:#dcdcdc"> View();</span>
        </li>
        <li style="background: #111111">
          <span style="background:#1e1e1e;color:#dcdcdc">        }</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">    }</span>
        </li>
        <li style="background: #111111">
          <span style="background:#1e1e1e;color:#dcdcdc">}</span>
        </li>
      </ol>
    </div></p>
  </div></p>
</div>

Una vez hecho esto añade (en la carpeta /Views/Home) la vista Index.chstml:

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:d39fd7d6-2a82-4f8e-a639-9018517df59f" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
    <div style="background: #ddd; max-height: 300px; overflow: auto">
      <ol start="1" style="background: #1d1d1d; margin: 0 0 0 2em; padding: 0 0 0 5px; white-space: nowrap">
        <li>
          <span style="background:#1e1e1e;color:#808080"><</span><span style="background:#1e1e1e;color:#569cd6">html</span><span style="background:#1e1e1e;color:#808080">></span>
        </li>
        <li style="background: #111111">
          <span style="background:#1e1e1e;color:#808080"><</span><span style="background:#1e1e1e;color:#569cd6">head</span><span style="background:#1e1e1e;color:#808080">><</span><span style="background:#1e1e1e;color:#569cd6">title</span><span style="background:#1e1e1e;color:#808080">></span><span style="background:#1e1e1e;color:#dcdcdc">Nowin Test</span><span style="background:#1e1e1e;color:#808080"></</span><span style="background:#1e1e1e;color:#569cd6">title</span><br /> <span style="background:#1e1e1e;color:#808080">></</span><span style="background:#1e1e1e;color:#569cd6">head</span><span style="background:#1e1e1e;color:#808080">></span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#808080"><</span><span style="background:#1e1e1e;color:#569cd6">body</span><span style="background:#1e1e1e;color:#808080">></span>
        </li>
        <li style="background: #111111">
          <span style="background:#1e1e1e;color:#dcdcdc">    </span><span style="background:#1e1e1e;color:#808080"><</span><span style="background:#1e1e1e;color:#569cd6">p</span><span style="background:#1e1e1e;color:#808080">></span><span style="background:#1e1e1e;color:#dcdcdc">Running on Nowin... </span><span style="background:#ffffb3;color:#000000">@</span><span style="background:#1e1e1e;color:#4ec9b0">DateTime</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">Now</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">ToShortDateString()</span><span style="background:#1e1e1e;color:#808080"></</span><span style="background:#1e1e1e;color:#569cd6">p</span><span style="background:#1e1e1e;color:#808080">></span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#808080"></</span><span style="background:#1e1e1e;color:#569cd6">body</span><span style="background:#1e1e1e;color:#808080">></span>
        </li>
        <li style="background: #111111">
          <span style="background:#1e1e1e;color:#808080"></</span><span style="background:#1e1e1e;color:#569cd6">html</span><span style="background:#1e1e1e;color:#808080">></span>
        </li>
      </ol>
    </div></p>
  </div></p>
</div>

Ya tenemos el controlador y la vista de MVC. Tan solo nos falta “configurar” nuestra aplicación para que use ASP.NET MVC. Para ello nos creamos una clase llamada Startup:

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:5619d716-fac0-4b0b-af77-861958622d63" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
    <div style="background: #ddd; max-height: 300px; overflow: auto">
      <ol start="1" style="background: #1d1d1d; margin: 0 0 0 2.5em; padding: 0 0 0 5px; white-space: nowrap">
        <li>
          <span style="background:#1e1e1e;color:#569cd6">public</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">class</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#4ec9b0">Startup</span>
        </li>
        <li style="background: #111111">
          <span style="background:#1e1e1e;color:#dcdcdc">{</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">    </span><span style="background:#1e1e1e;color:#569cd6">public</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">void</span><span style="background:#1e1e1e;color:#dcdcdc"> Configure(</span><span style="background:#1e1e1e;color:#b8d7a3">IBuilder</span><span style="background:#1e1e1e;color:#dcdcdc"> app)</span>
        </li>
        <li style="background: #111111">
          <span style="background:#1e1e1e;color:#dcdcdc">    {</span>
        </li>
        <li>
          &nbsp;
        </li>
        <li style="background: #111111">
          <span style="background:#1e1e1e;color:#dcdcdc">        app</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">UseServices(services </span><span style="background:#1e1e1e;color:#b4b4b4">=></span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">        {</span>
        </li>
        <li style="background: #111111">
          <span style="background:#1e1e1e;color:#dcdcdc">            services</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">AddMvc();</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">        });</span>
        </li>
        <li style="background: #111111">
          &nbsp;
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">        app</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">UseMvc();</span>
        </li>
        <li style="background: #111111">
          &nbsp;
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">    }</span>
        </li>
        <li style="background: #111111">
          <span style="background:#1e1e1e;color:#dcdcdc">}</span>
        </li>
      </ol>
    </div></p>
  </div></p>
</div>

El siguiente paso es añadir las referencias necesarias para hospedar nuestra aplicación web y a Nowin. Para ello editamos el archivo project.json para añadir en la sección “dependencies”:

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:6bd05fb0-d14b-426c-b3b4-51fd9c152901" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
    <div style="background: #ddd; max-height: 300px; overflow: auto">
      <ol start="1" style="background: #1d1d1d; margin: 0 0 0 2em; padding: 0 0 0 5px; white-space: nowrap">
        <li>
          <span style="background:#1e1e1e;color:#d7ba7d">"Nowin"</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#b4b4b4">:</span><span style="background:#1e1e1e;color:#dcdcdc">  </span><span style="background:#1e1e1e;color:#d69d85">"0.11.0"</span><span style="background:#1e1e1e;color:#b4b4b4">,</span>
        </li>
        <li style="background: #111111">
          <span style="background:#1e1e1e;color:#d7ba7d">"Microsoft.AspNet.Owin"</span><span style="background:#1e1e1e;color:#b4b4b4">:</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#d69d85">"0.1-alpha-build-*"</span><span style="background:#1e1e1e;color:#b4b4b4">,</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#d7ba7d">"Microsoft.AspNet.Hosting"</span><span style="background:#1e1e1e;color:#b4b4b4">:</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#d69d85">"0.1-alpha-build-*"</span>
        </li>
      </ol>
    </div></p>
  </div></p>
</div>

(Fíjate que al guardar el fichero project.json y al compilar te saldrán ya las referencias en VS2014)

<div>
  <p>
    Ya tenemos Nowin agregado, ahora toca configurarlo todo. Lo primero es cargar nuestra clase Startup. Eso es necesario porque es una aplicación de consola. En este caso debemos añadir el siguiente código a Program.cs:
  </p>
  
  <div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:21e955df-5d7d-4a29-acc4-5fb58967bc3f" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
    <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
      <div style="background: #ddd; max-height: 300px; overflow: auto">
        <ol start="1" style="background: #1d1d1d; margin: 0 0 0 2.5em; padding: 0 0 0 5px; white-space: nowrap">
          <li>
            <span style="background:#1e1e1e;color:#569cd6">public</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">class</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#4ec9b0">Program</span>
          </li>
          <li style="background: #111111">
            <span style="background:#1e1e1e;color:#dcdcdc">{</span>
          </li>
          <li>
            <span style="background:#1e1e1e;color:#dcdcdc">    </span><span style="background:#1e1e1e;color:#569cd6">private</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">readonly</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#b8d7a3">IServiceProvider</span><span style="background:#1e1e1e;color:#dcdcdc"> _hostServiceProvider;</span>
          </li>
          <li style="background: #111111">
            &nbsp;
          </li>
          <li>
            <span style="background:#1e1e1e;color:#dcdcdc">    </span><br /> <span style="background:#1e1e1e;color:#569cd6">public</span><span style="background:#1e1e1e;color:#dcdcdc"> Program(</span><span style="background:#1e1e1e;color:#b8d7a3">IServiceProvider</span><span style="background:#1e1e1e;color:#dcdcdc"> hostServiceProvider)</span>
          </li>
          <li style="background: #111111">
            <span style="background:#1e1e1e;color:#dcdcdc">    {</span>
          </li>
          <li>
            <span style="background:#1e1e1e;color:#dcdcdc">        _hostServiceProvider </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> hostServiceProvider;</span>
          </li>
          <li style="background: #111111">
            <span style="background:#1e1e1e;color:#dcdcdc">    }</span>
          </li>
          <li>
            &nbsp;
          </li>
          <li style="background: #111111">
            <span style="background:#1e1e1e;color:#dcdcdc">    </span><span style="background:#1e1e1e;color:#569cd6">public</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#4ec9b0">Task</span><span style="background:#1e1e1e;color:#dcdcdc"><</span><span style="background:#1e1e1e;color:#569cd6">int</span><span style="background:#1e1e1e;color:#dcdcdc">> Main(</span><span style="background:#1e1e1e;color:#569cd6">string</span><span style="background:#1e1e1e;color:#dcdcdc">[] args)</span>
          </li>
          <li>
            <span style="background:#1e1e1e;color:#dcdcdc">    {</span>
          </li>
          <li style="background: #111111">
            <span style="background:#1e1e1e;color:#dcdcdc">        </span><span style="background:#1e1e1e;color:#569cd6">return</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#4ec9b0">Task</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">FromResult(</span><span style="background:#1e1e1e;color:#b5cea8"></span><span style="background:#1e1e1e;color:#dcdcdc">);</span>
          </li>
          <li>
            <span style="background:#1e1e1e;color:#dcdcdc">    }</span>
          </li>
          <li style="background: #111111">
            <span style="background:#1e1e1e;color:#dcdcdc">}</span>
          </li>
        </ol>
      </div></p>
    </div></p>
  </div>
  
  <p>
    <strong>Sí: es un código un poco raro</strong> para ser un programa de consola, verdad? Por un lado <em>Program</em> tiene un constructor que recibe un IServiceProvider y por otro el método Main no es estático si no que devuelve una Task<int>. Y esto? Esto es porque la aplicación de consola se pone en marcha a través de KRE (el runtime de ASP.NET vNext).
  </p>
  
  <p>
    Bien, ahora vamos a colocar código en Main para inicializar nuestr aplicación ASP.NET vNext:
  </p>
  
  <div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:6224d385-fbfa-4988-8aad-e3e9ce2593f8" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
    <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
      <div style="background: #ddd; max-height: 300px; overflow: auto">
        <ol start="1" style="background: #1d1d1d; margin: 0 0 0 2em; padding: 0 0 0 5px; white-space: nowrap">
          <li>
            <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">var</span><span style="background:#1e1e1e;color:#dcdcdc"> serviceCollection </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">new</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#4ec9b0">ServiceCollection</span><span style="background:#1e1e1e;color:#dcdcdc">();</span>
          </li>
          <li style="background: #111111">
            <span style="background:#1e1e1e;color:#dcdcdc">serviceCollection</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">Add(</span><span style="background:#1e1e1e;color:#4ec9b0">HostingServices</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">GetDefaultServices());</span>
          </li>
          <li>
            <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">var</span><span style="background:#1e1e1e;color:#dcdcdc"> services </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> serviceCollection</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">BuildServiceProvider(_hostServiceProvider);</span>
          </li>
        </ol>
      </div></p>
    </div></p>
  </div>
  
  <blockquote>
    <p>
      Bien, ahora que ya lo tenemos todo configurado, nos queda poner en marcha el host. Antes que nada, un pequeño recordatorio, sobre como establecíamos el servidor web usando OWIN y Katana en una aplicación de consola. En Katana usábamos el paquete Microsoft.Owin.Hosting, que define una clase llamada WebApp<T> y usábamos un código parecido a:
    </p>
    
    <pre>var options = new StartOptions
{
    ServerFactory = "Nowin",
    Port = 8080
};
using (WebApp.Start&lt;Startup&gt;(options))
{
    Console.WriteLine("Running a http server on port 8080");
    Console.ReadKey();
}</pre>
    
    <p>
      En este contexto la clase Startup (parámetro genérico de WebApp) era la clase de configuración de la aplicación (lo equivalente a <em>nuestra</em> clase Startup). Lo interesante es la la StartOptions que permitía establecer el ensamblado que contiene el servidor web (en este caso Nowin).
    </p>
  </blockquote>
  
  <p>
    En vNext la idea es muy similar (cambian clases y propiedades) pero es la misma idea:
  </p>
  
  <div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:acc6cfe3-515c-452b-9069-b4442eb1aa28" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
    <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
      <div style="background: #ddd; overflow: auto">
        <ol start="1" style="background: #1d1d1d; margin: 0 0 0 2.5em; padding: 0 0 0 5px; white-space: nowrap">
          <li>
            <span style="background:#1e1e1e;color:#569cd6">var</span><span style="background:#1e1e1e;color:#dcdcdc"> context </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">new</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#4ec9b0">HostingContext</span><span style="background:#1e1e1e;color:#dcdcdc">()</span>
          </li>
          <li style="background: #111111">
            <span style="background:#1e1e1e;color:#dcdcdc">{</span>
          </li>
          <li>
            <span style="background:#1e1e1e;color:#dcdcdc">    Services </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> services,</span>
          </li>
          <li style="background: #111111">
            <span style="background:#1e1e1e;color:#dcdcdc">    ServerName</span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#d69d85">"Nowin"</span><span style="background:#1e1e1e;color:#dcdcdc">,</span>
          </li>
          <li>
            <span style="background:#1e1e1e;color:#dcdcdc">    ApplicationName </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#d69d85">"NowinDemo"</span><span style="background:#1e1e1e;color:#dcdcdc">,</span>
          </li>
          <li style="background: #111111">
            <span style="background:#1e1e1e;color:#dcdcdc">};</span>
          </li>
          <li>
            <span style="background:#1e1e1e;color:#569cd6">var</span><span style="background:#1e1e1e;color:#dcdcdc"> engine </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> services</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">GetService<</span><span style="background:#1e1e1e;color:#b8d7a3">IHostingEngine</span><span style="background:#1e1e1e;color:#dcdcdc">>();</span>
          </li>
          <li style="background: #111111">
            <span style="background:#1e1e1e;color:#569cd6">using</span><span style="background:#1e1e1e;color:#dcdcdc"> (engine</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">Start(context))</span>
          </li>
          <li>
            <span style="background:#1e1e1e;color:#dcdcdc">{</span>
          </li>
          <li style="background: #111111">
            <span style="background:#1e1e1e;color:#dcdcdc">    </span><span style="background:#1e1e1e;color:#4ec9b0">Console</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">WriteLine(</span><span style="background:#1e1e1e;color:#d69d85">"Server waiting... "</span><span style="background:#1e1e1e;color:#dcdcdc">);</span>
          </li>
          <li>
            <span style="background:#1e1e1e;color:#dcdcdc">    </span><span style="background:#1e1e1e;color:#4ec9b0">Console</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">ReadLine();</span>
          </li>
          <li style="background: #111111">
            <span style="background:#1e1e1e;color:#dcdcdc">}</span>
          </li>
        </ol>
      </div>
    </div>
  </div>
  
  <p>
    <strong>Nota:</strong> NowinDemo es el nombre de mi ensamblado (el que contiene la clase Startup) y Nowin es obviamente el nombre del ensamblado de Nowin.
  </p>
  
  <p>
    Directamente establezco que el servidor está en el ensamblado Nowin. No obstante esto <strong>no funciona:</strong>
  </p>
  
  <p>
    <img title="image" style="border-left-width: 0px; border-right-width: 0px; background-image: none; border-bottom-width: 0px; padding-top: 0px; padding-left: 0px; display: inline; padding-right: 0px; border-top-width: 0px" border="0" alt="image" src="http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_7690C0BC.png" width="436" height="87" />
  </p>
  
  <p>
    La razón es que, efectivamente ASP.NET vNext intenta ir a este ensamblado para obtener un servidor, pero ahora espera un servidor vNext y no uno OWIN como Nowin. En concreto espera que haya una clase que implemente IServerFactory (interfaz de vNext).
  </p>
  
  <p>
    Bueno, por suerte, no es excesivamente complejo crear dicha clase. Así pues añadimos una clase que implemente IServerFactory:
  </p>
  
  <div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:c1515ae3-5877-472e-b9d0-23ded1e2d45c" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
    <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
      <div style="background: #ddd; max-height: 300px; overflow: auto">
        <ol start="1" style="background: #1d1d1d; margin: 0 0 0 2.5em; padding: 0 0 0 5px;">
          <li>
            <span style="background:#1e1e1e;color:#dcdcdc">    </span><span style="background:#1e1e1e;color:#569cd6">public</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">class</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#4ec9b0">NowinServerFactory</span><span style="background:#1e1e1e;color:#dcdcdc"> : </span><span style="background:#1e1e1e;color:#b8d7a3">IServerFactory</span>
          </li>
          <li style="background: #111111">
            <span style="background:#1e1e1e;color:#dcdcdc">    { </span>
          </li>
          <li>
            <span style="background:#1e1e1e;color:#dcdcdc">        </span><span style="background:#1e1e1e;color:#569cd6">public</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#b8d7a3">IServerInformation</span><span style="background:#1e1e1e;color:#dcdcdc"> Initialize(</span><span style="background:#1e1e1e;color:#b8d7a3">IConfiguration</span><span style="background:#1e1e1e;color:#dcdcdc"> configuration)</span>
          </li>
          <li style="background: #111111">
            <span style="background:#1e1e1e;color:#dcdcdc">        {</span>
          </li>
          <li>
            <span style="background:#1e1e1e;color:#dcdcdc">        }       </span>
          </li>
          <li style="background: #111111">
            <span style="background:#1e1e1e;color:#dcdcdc">        </span><span style="background:#1e1e1e;color:#569cd6">public</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#b8d7a3">IDisposable</span><span style="background:#1e1e1e;color:#dcdcdc"> Start(</span><span style="background:#1e1e1e;color:#b8d7a3">IServerInformation</span><span style="background:#1e1e1e;color:#dcdcdc"> serverInformation, </span><span style="background:#1e1e1e;color:#4ec9b0">Func</span><span style="background:#1e1e1e;color:#dcdcdc"><</span><span style="background:#1e1e1e;color:#569cd6">object</span><span style="background:#1e1e1e;color:#dcdcdc">, </span><span style="background:#1e1e1e;color:#4ec9b0">Task</span><span style="background:#1e1e1e;color:#dcdcdc">> application)</span>
          </li>
          <li>
            <span style="background:#1e1e1e;color:#dcdcdc">        {</span>
          </li>
          <li style="background: #111111">
            <span style="background:#1e1e1e;color:#dcdcdc">  </span>
          </li>
          <li>
            <span style="background:#1e1e1e;color:#dcdcdc">        }</span>
          </li>
          <li style="background: #111111">
            <span style="background:#1e1e1e;color:#dcdcdc">    }</span>
          </li>
        </ol>
      </div>
    </div>
  </div>
  
  <p>
    El método Initialize se llama una vez al principio de la aplicación y debemos devolver una clase que implemente IServerBuilder. Dicha interfaz <strong>tan solo define una propiedad llamada Name</strong>. Vamos a crearnos una clase que nos implemente dicha interfaz:
  </p>
  
  <div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:dba233d6-abdd-44a8-9e3d-b1a7d2e30b95" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
    <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
      <div style="background: #ddd; max-height: 300px; overflow: auto">
        <ol start="1" style="background: #1d1d1d; margin: 0 0 0 2em; padding: 0 0 0 5px;">
          <li>
            <span style="background:#1e1e1e;color:#569cd6">class</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#4ec9b0">NowinServerInformation</span><span style="background:#1e1e1e;color:#dcdcdc"> : </span><span style="background:#1e1e1e;color:#b8d7a3">IServerInformation</span>
          </li>
          <li style="background: #111111">
            <span style="background:#1e1e1e;color:#dcdcdc">{</span>
          </li>
          <li>
            <span style="background:#1e1e1e;color:#dcdcdc">    </span><span style="background:#1e1e1e;color:#569cd6">public</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">string</span><span style="background:#1e1e1e;color:#dcdcdc"> Name { </span><span style="background:#1e1e1e;color:#569cd6">get</span><span style="background:#1e1e1e;color:#dcdcdc"> { </span><span style="background:#1e1e1e;color:#569cd6">return</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#d69d85">"Nowin"</span><span style="background:#1e1e1e;color:#dcdcdc">; } }</span>
          </li>
          <li style="background: #111111">
            <span style="background:#1e1e1e;color:#dcdcdc">}</span>
          </li>
        </ol>
      </div>
    </div>
  </div>
  
  <p>
    En el método Initialize debemos instanciar el servidor Nowin y devolver un objeto IServerInformation:
  </p>
  
  <div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:867cd975-baeb-4eeb-bc5f-9b333d909edc" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
    <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
      <div style="background: #ddd; max-height: 500px; overflow: auto">
        <ol start="1" style="background: #1d1d1d; margin: 0 0 0 2em; padding: 0 0 0 5px;">
          <li>
            <span style="background:#1e1e1e;color:#569cd6">public</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#b8d7a3">IServerInformation</span><span style="background:#1e1e1e;color:#dcdcdc"> Initialize(</span><span style="background:#1e1e1e;color:#b8d7a3">IConfiguration</span><span style="background:#1e1e1e;color:#dcdcdc"> configuration)</span>
          </li>
          <li style="background: #111111">
            <span style="background:#1e1e1e;color:#dcdcdc">{</span>
          </li>
          <li>
            <span style="background:#1e1e1e;color:#dcdcdc">    </span><span style="background:#1e1e1e;color:#569cd6">var</span><span style="background:#1e1e1e;color:#dcdcdc"> builder </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#4ec9b0">ServerBuilder</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">New()</span>
          </li>
          <li style="background: #111111">
            <span style="background:#1e1e1e;color:#dcdcdc">        </span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">SetAddress(</span><span style="background:#1e1e1e;color:#4ec9b0">IPAddress</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">Any)</span>
          </li>
          <li>
            <span style="background:#1e1e1e;color:#dcdcdc">        </span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">SetPort(</span><span style="background:#1e1e1e;color:#b5cea8">8080</span><span style="background:#1e1e1e;color:#dcdcdc">)</span>
          </li>
          <li style="background: #111111">
            <span style="background:#1e1e1e;color:#dcdcdc">        </span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">SetOwinApp(HandleRequest);</span>
          </li>
          <li>
            <span style="background:#1e1e1e;color:#dcdcdc">    </span><span style="background:#1e1e1e;color:#569cd6">return</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">new</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#4ec9b0">NowinServerInformation</span><span style="background:#1e1e1e;color:#dcdcdc">();</span>
          </li>
          <li style="background: #111111">
            <span style="background:#1e1e1e;color:#dcdcdc">}</span>
          </li>
        </ol>
      </div>
    </div>
  </div>
  
  <p>
    Nos falta el método HandleRequest (un Func<IDictionary<string, object>, Task> que espera SetOwinApp:
  </p>
  
  <div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:ca44a1ab-8f0a-49d9-8e9d-04fa17f08a10" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
    <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
      <div style="background: #ddd; max-height: 300px; overflow: auto">
        <ol start="1" style="background: #1d1d1d; margin: 0 0 0 2em; padding: 0 0 0 5px;">
          <li>
            <span style="background:#1e1e1e;color:#569cd6">private</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#4ec9b0">Task</span><span style="background:#1e1e1e;color:#dcdcdc"> HandleRequest(</span><span style="background:#1e1e1e;color:#b8d7a3">IDictionary</span><span style="background:#1e1e1e;color:#dcdcdc"><</span><span style="background:#1e1e1e;color:#569cd6">string</span><span style="background:#1e1e1e;color:#dcdcdc">, </span><span style="background:#1e1e1e;color:#569cd6">object</span><span style="background:#1e1e1e;color:#dcdcdc">> env)</span>
          </li>
          <li style="background: #111111">
            <span style="background:#1e1e1e;color:#dcdcdc">{</span>
          </li>
          <li>
            <span style="background:#1e1e1e;color:#dcdcdc">    </span><span style="background:#1e1e1e;color:#569cd6">return</span><span style="background:#1e1e1e;color:#dcdcdc"> _callback(</span><span style="background:#1e1e1e;color:#569cd6">new</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#4ec9b0">OwinFeatureCollection</span><span style="background:#1e1e1e;color:#dcdcdc">(env));</span>
          </li>
          <li style="background: #111111">
            <span style="background:#1e1e1e;color:#dcdcdc">}</span>
          </li>
        </ol>
      </div>
    </div>
  </div>
  
  <p>
    La variable _callback es un Func<object, Task>. Lo interesante es el uso de la clase OwinFeatureCollection (definida en Microsoft.AspNet.Owin) que ofrece un “envoltorio” tipado al diccionario de entorno de Owin.
  </p>
  
  <p>
    Ahora en el método Start del objeto IServerFactory debemos devolver el componente que es el servidor. Este método recibe el IServerInformation y la definición de aplicación (es decir el conjunto de componentes del pipeline vNext) como una Func<object, Task>.&#160; El problema ahora lo tenemos en el hecho de que debemos obtener el servidor Nowin (INowinServer) a partir del objeto IServerInformation… pero allí no lo hemos guardado. Así pues debemos redefinir dicha clase, para que nos guarde la información necesaria para ello. La idea es que la clase que implementa IServerFactory debe ser, como su nombre indica, la factoría para permitir crear el servidor a cada petición. Así pues el código final queda como sigue:
  </p>
  
  <div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:fe0a441e-985a-47a5-8952-bcdc877f5446" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
    <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
      <div style="background: #ddd; max-height: 300px; overflow: auto">
        <ol start="1" style="background: #1d1d1d; margin: 0 0 0 2.5em; padding: 0 0 0 5px;">
          <li>
            <span style="background:#1e1e1e;color:#569cd6">public</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">class</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#4ec9b0">NowinServerFactory</span><span style="background:#1e1e1e;color:#dcdcdc"> : </span><span style="background:#1e1e1e;color:#b8d7a3">IServerFactory</span>
          </li>
          <li style="background: #111111">
            <span style="background:#1e1e1e;color:#dcdcdc">{</span>
          </li>
          <li>
            <span style="background:#1e1e1e;color:#dcdcdc">    </span><span style="background:#1e1e1e;color:#569cd6">private</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#4ec9b0">Func</span><span style="background:#1e1e1e;color:#dcdcdc"><</span><span style="background:#1e1e1e;color:#569cd6">object</span><span style="background:#1e1e1e;color:#dcdcdc">, </span><span style="background:#1e1e1e;color:#4ec9b0">Task</span><span style="background:#1e1e1e;color:#dcdcdc">> _callback;</span>
          </li>
          <li style="background: #111111">
            &nbsp;
          </li>
          <li>
            <span style="background:#1e1e1e;color:#dcdcdc">    </span><span style="background:#1e1e1e;color:#569cd6">private</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#4ec9b0">Task</span><span style="background:#1e1e1e;color:#dcdcdc"> HandleRequest(</span><span style="background:#1e1e1e;color:#b8d7a3">IDictionary</span><span style="background:#1e1e1e;color:#dcdcdc"><</span><span style="background:#1e1e1e;color:#569cd6">string</span><span style="background:#1e1e1e;color:#dcdcdc">, </span><span style="background:#1e1e1e;color:#569cd6">object</span><span style="background:#1e1e1e;color:#dcdcdc">> env)</span>
          </li>
          <li style="background: #111111">
            <span style="background:#1e1e1e;color:#dcdcdc">    {</span>
          </li>
          <li>
            <span style="background:#1e1e1e;color:#dcdcdc">        </span><span style="background:#1e1e1e;color:#569cd6">return</span><span style="background:#1e1e1e;color:#dcdcdc"> _callback(</span><span style="background:#1e1e1e;color:#569cd6">new</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#4ec9b0">OwinFeatureCollection</span><span style="background:#1e1e1e;color:#dcdcdc">(env));</span>
          </li>
          <li style="background: #111111">
            <span style="background:#1e1e1e;color:#dcdcdc">    }</span>
          </li>
          <li>
            &nbsp;
          </li>
          <li style="background: #111111">
            <span style="background:#1e1e1e;color:#dcdcdc">    </span><span style="background:#1e1e1e;color:#569cd6">public</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#b8d7a3">IServerInformation</span><span style="background:#1e1e1e;color:#dcdcdc"> Initialize(</span><span style="background:#1e1e1e;color:#b8d7a3">IConfiguration</span><span style="background:#1e1e1e;color:#dcdcdc"> configuration)</span>
          </li>
          <li>
            <span style="background:#1e1e1e;color:#dcdcdc">    {</span>
          </li>
          <li style="background: #111111">
            <span style="background:#1e1e1e;color:#dcdcdc">        </span><span style="background:#1e1e1e;color:#569cd6">var</span><span style="background:#1e1e1e;color:#dcdcdc"> builder </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#4ec9b0">ServerBuilder</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">New()</span>
          </li>
          <li>
            <span style="background:#1e1e1e;color:#dcdcdc">            </span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">SetAddress(</span><span style="background:#1e1e1e;color:#4ec9b0">IPAddress</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">Any)</span>
          </li>
          <li style="background: #111111">
            <span style="background:#1e1e1e;color:#dcdcdc">            </span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">SetPort(</span><span style="background:#1e1e1e;color:#b5cea8">8080</span><span style="background:#1e1e1e;color:#dcdcdc">)</span>
          </li>
          <li>
            <span style="background:#1e1e1e;color:#dcdcdc">            </span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">SetOwinApp(HandleRequest);</span>
          </li>
          <li style="background: #111111">
            <span style="background:#1e1e1e;color:#dcdcdc">        </span><span style="background:#1e1e1e;color:#569cd6">return</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">new</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#4ec9b0">NowinServerInformation</span><span style="background:#1e1e1e;color:#dcdcdc">(builder);</span>
          </li>
          <li>
            <span style="background:#1e1e1e;color:#dcdcdc">    }</span>
          </li>
          <li style="background: #111111">
            <span style="background:#1e1e1e;color:#dcdcdc">            </span>
          </li>
          <li>
            <span style="background:#1e1e1e;color:#dcdcdc">    </span><span style="background:#1e1e1e;color:#569cd6">public</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#b8d7a3">IDisposable</span><span style="background:#1e1e1e;color:#dcdcdc"> Start(</span><span style="background:#1e1e1e;color:#b8d7a3">IServerInformation</span><span style="background:#1e1e1e;color:#dcdcdc"> serverInformation, </span><span style="background:#1e1e1e;color:#4ec9b0">Func</span><span style="background:#1e1e1e;color:#dcdcdc"><</span><span style="background:#1e1e1e;color:#569cd6">object</span><span style="background:#1e1e1e;color:#dcdcdc">, </span><span style="background:#1e1e1e;color:#4ec9b0">Task</span><span style="background:#1e1e1e;color:#dcdcdc">> application)</span>
          </li>
          <li style="background: #111111">
            <span style="background:#1e1e1e;color:#dcdcdc">    {</span>
          </li>
          <li>
            <span style="background:#1e1e1e;color:#dcdcdc">        </span><span style="background:#1e1e1e;color:#569cd6">var</span><span style="background:#1e1e1e;color:#dcdcdc"> information </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> (</span><span style="background:#1e1e1e;color:#4ec9b0">NowinServerInformation</span><span style="background:#1e1e1e;color:#dcdcdc">)serverInformation;</span>
          </li>
          <li style="background: #111111">
            <span style="background:#1e1e1e;color:#dcdcdc">        _callback </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> application;</span>
          </li>
          <li>
            <span style="background:#1e1e1e;color:#dcdcdc">        </span><span style="background:#1e1e1e;color:#b8d7a3">INowinServer</span><span style="background:#1e1e1e;color:#dcdcdc"> server </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> information</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">Builder</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">Build();</span>
          </li>
          <li style="background: #111111">
            <span style="background:#1e1e1e;color:#dcdcdc">        server</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">Start();</span>
          </li>
          <li>
            <span style="background:#1e1e1e;color:#dcdcdc">        </span><span style="background:#1e1e1e;color:#569cd6">return</span><span style="background:#1e1e1e;color:#dcdcdc"> server;</span>
          </li>
          <li style="background: #111111">
            <span style="background:#1e1e1e;color:#dcdcdc">    }</span>
          </li>
          <li>
            &nbsp;
          </li>
          <li style="background: #111111">
            &nbsp;
          </li>
          <li>
            <span style="background:#1e1e1e;color:#dcdcdc">    </span><span style="background:#1e1e1e;color:#569cd6">private</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">class</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#4ec9b0">NowinServerInformation</span><span style="background:#1e1e1e;color:#dcdcdc"> : </span><span style="background:#1e1e1e;color:#b8d7a3">IServerInformation</span>
          </li>
          <li style="background: #111111">
            <span style="background:#1e1e1e;color:#dcdcdc">    {</span>
          </li>
          <li>
            <span style="background:#1e1e1e;color:#dcdcdc">        </span><span style="background:#1e1e1e;color:#569cd6">public</span><span style="background:#1e1e1e;color:#dcdcdc"> NowinServerInformation(</span><span style="background:#1e1e1e;color:#4ec9b0">ServerBuilder</span><span style="background:#1e1e1e;color:#dcdcdc"> builder)</span>
          </li>
          <li style="background: #111111">
            <span style="background:#1e1e1e;color:#dcdcdc">        {</span>
          </li>
          <li>
            <span style="background:#1e1e1e;color:#dcdcdc">            Builder </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> builder;</span>
          </li>
          <li style="background: #111111">
            <span style="background:#1e1e1e;color:#dcdcdc">        }</span>
          </li>
          <li>
            &nbsp;
          </li>
          <li style="background: #111111">
            <span style="background:#1e1e1e;color:#dcdcdc">        </span><span style="background:#1e1e1e;color:#569cd6">public</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#4ec9b0">ServerBuilder</span><span style="background:#1e1e1e;color:#dcdcdc"> Builder { </span><span style="background:#1e1e1e;color:#569cd6">get</span><span style="background:#1e1e1e;color:#dcdcdc">; </span><span style="background:#1e1e1e;color:#569cd6">private</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">set</span><span style="background:#1e1e1e;color:#dcdcdc">; }</span>
          </li>
          <li>
            &nbsp;
          </li>
          <li style="background: #111111">
            <span style="background:#1e1e1e;color:#dcdcdc">        </span><span style="background:#1e1e1e;color:#569cd6">public</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">string</span><span style="background:#1e1e1e;color:#dcdcdc"> Name</span>
          </li>
          <li>
            <span style="background:#1e1e1e;color:#dcdcdc">        {</span>
          </li>
          <li style="background: #111111">
            <span style="background:#1e1e1e;color:#dcdcdc">            </span><span style="background:#1e1e1e;color:#569cd6">get</span>
          </li>
          <li>
            <span style="background:#1e1e1e;color:#dcdcdc">            {</span>
          </li>
          <li style="background: #111111">
            <span style="background:#1e1e1e;color:#dcdcdc">                </span><span style="background:#1e1e1e;color:#569cd6">return</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#d69d85">"Nowin"</span><span style="background:#1e1e1e;color:#dcdcdc">;</span>
          </li>
          <li>
            <span style="background:#1e1e1e;color:#dcdcdc">            }</span>
          </li>
          <li style="background: #111111">
            <span style="background:#1e1e1e;color:#dcdcdc">        }</span>
          </li>
          <li>
            <span style="background:#1e1e1e;color:#dcdcdc">    }</span>
          </li>
          <li style="background: #111111">
            <span style="background:#1e1e1e;color:#dcdcdc">}</span>
          </li>
        </ol>
      </div>
    </div>
  </div>
  
  <p>
    <strong>Nota:</strong> El código de esta clase <a href="https://gist.github.com/davidfowl/8ea8be87fd5e1090f345" target="_blank" rel="noopener noreferrer">es mérito de David Fowler y lo tenéis en este Gist</a>.
  </p>
  
  <p>
    Ahora ya tan solo nos queda modificar el HostContext puesto que el ensamblado que contiene el IServerFactory ya no es Nowin sino nuestra propia aplicación (NowinDemo en mi caso):
  </p>
  
  <div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:c00b6f54-5ce7-4b8d-9f3e-aa79c447c7c5" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
    <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
      <div style="background: #ddd; max-height: 300px; overflow: auto">
        <ol start="1" style="background: #1d1d1d; margin: 0 0 0 2em; padding: 0 0 0 5px;">
          <li>
            <span style="background:#1e1e1e;color:#569cd6">var</span><span style="background:#1e1e1e;color:#dcdcdc"> context </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">new</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#4ec9b0">HostingContext</span><span style="background:#1e1e1e;color:#dcdcdc">()</span>
          </li>
          <li style="background: #111111">
            <span style="background:#1e1e1e;color:#dcdcdc">{</span>
          </li>
          <li>
            <span style="background:#1e1e1e;color:#dcdcdc">    ServerName</span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#d69d85">"NowinDemo"</span><span style="background:#1e1e1e;color:#dcdcdc">,</span>
          </li>
          <li style="background: #111111">
            <span style="background:#1e1e1e;color:#dcdcdc">    ApplicationName </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#d69d85">"NowinDemo"</span><span style="background:#1e1e1e;color:#dcdcdc">,</span>
          </li>
          <li>
            <span style="background:#1e1e1e;color:#dcdcdc">    Services </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> services,</span>
          </li>
          <li style="background: #111111">
            <span style="background:#1e1e1e;color:#dcdcdc">};</span>
          </li>
        </ol>
      </div>
    </div>
  </div>
  
  <p>
    ¡Y con esto ya hemos terminado! Tenemos <strong>una aplicación ASP.NET vNext MVC6</strong> servida desde línea de comandos y usando un servidor web OWIN como Nowin para servirla (en lugar del clásico Microsoft.AspNet.Server.WebListener de ASP.NET vNext).
  </p>
</div>

**¿Donde está el código compilado?**

Si compilas el proyecto en VS2014 CTP verás que en la carpeta Bin/Debug… **no hay nada**! No hay un NowinDemo.exe ni nada parecido. Las aplicaciones ASP.NET vNext, incluso si son de consola, se ponen en marcha a través del runtime de ASP.NET. Para ello se usa el comando K run “path\_de\_la_aplicación”.

El fichero K.cmd está donde tengas instalado el KRE. Por defecto está en %HOME%.krepackagesKRE-svrc50-x86.0.1-alpha-build-0446bin

Si no lo tienes debes añadir al path este directorio, irte donde tienes el proyecto de VS2014 (en el path raíz del proyecto donde tienes el project.json) y teclear:

> **K run**

¡Y listos! Tu aplicación de consola ASP.NET vNext ya está en marcha!

 [1]: http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_200501FD.png