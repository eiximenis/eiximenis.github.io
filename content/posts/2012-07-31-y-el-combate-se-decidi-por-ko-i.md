---
title: Y el combate se decidió por KO (i)
description: Y el combate se decidió por KO (i)
author: eiximenis

date: 2012-07-31T15:33:00+00:00
geeks_url: /?p=1606
geeks_visits:
  - 5308
geeks_ms_views:
  - 1789
categories:
  - Uncategorized

---
Hace algunas semanas salió un post de Shaun Walker titulado &ldquo;<a target="_blank" href="http://weblogs.asp.net/sbwalker/archive/2012/06/17/microsoft-declares-the-future-of-asp-net-is-web-api.aspx" rel="noopener noreferrer">Microsoft Declares the future of ASP.NET is Web API</a>&rdquo;. La verdad es que el post es interesante. Yo no sé cuales serán las intenciones de Microsoft (creo que ni ellos las saben realmente) pero lo que si es cierto es que las aplicaciones web están realmente cambiando a un modelo donde cada vez se procesa más en cliente y menos en servidor. Es un modelo que deja totalmente obsoleto no solo a Webforms si no incluso a ASP.NET MVC.

En este modelo las páginas o vistas de una aplicación web, ya no son servidas desde el servidor. En todo caso, se sirve un &ldquo;bootstrapper&rdquo; inicial, que contiene el código javascript que realiza la petición inicial y luego todo es modificar el DOM en cliente a partir de datos recibidos por servicios REST, usando javascript. Por lo tanto dejamos de servir contenido de presentación (HTML generado dinámicamente) desde el servidor, para servir tan solo datos. Y la conversión de datos a algo visible (DOM) se realiza en el propio cliente, usando javascript.

A nivel tecnológico, salvando quizá cuestiones de rendimiento (que no son en absoluto despreciables, ahí está la [experiencia de Twitter][1]) estamos ya bastante preparados para dar este salto. Y aquí entra lo que os quería comentar en el post de hoy. Vamos a iniciar una serie, de longitud desconocida, hablando de Knockout, una librería javascript que resumiendo lo que hace es implementar el patrón MVVM en javascript.

Como siempre no soy el primero en hablar de Knockout en geeks. Concretamente el <a target="_blank" href="http://twitter.com/jmaguilar" rel="noopener noreferrer">Maestro</a> se me ha avanzado con un post ([http://geeks.ms/blogs/jmaguilar/archive/2012/05/07/knockout-i-pongamos-orden-en-el-lado-cliente.aspx][2]).

También está <a target="_blank" href="http://twitter.com/Marc_Rubino" rel="noopener noreferrer">Marc Rubiño</a> que tiene una presentación en slideshare: <http://www.slideshare.net/webcatbcn/javascript-con-mvvm-knockout-por-marcrubino>&nbsp;

Pero por encima de todos <a target="_blank" href="http://twitter.com/smarrerof" rel="noopener noreferrer">Sergio Marrero</a> que tiene una serie **impresionante** sobre Knockout en su blog: <http://smarrerof.blogspot.com.es/search/label/knockout>

Pero bueno... Yo intentaré aportar mi grano de arena y intentaré acercaros un poco Knockout para que los que no lo conozcáis os echeis las manos a la cabeza y os preguntéis como narices habeis podido hacer aplicaciones web todo este tiempo 🙂

&iexcl;Espero que esta sea una larga serie de posts, porque realmente hay mucho que contar!

**Introducción a Knockout**

Knockout es una librería javascript que _básicamente_ nos da soporte para el patrón MVVM. Este patrón, bien conocido para los que desarrollan en WPF, Silverlight o WinRT, es en realidad un MVP [Supervising Controller][3] con dos peculiaridades:

  1. Se crea un modelo específicamente para presentación. Es decir, no se usa el &ldquo;Modelo&rdquo; real para tareas de presentación, si no que se crea _otro modelo_ específicamente para ello. En muchos casos este otro modelo está adaptado y optimizado para cada vista. En MVVM conocemos a este otro modelo con el nombre de ViewModel. 
  2. Se promueve que la comunicación entre la vista y su viewmodel sea a través de bindings (bidireccionales). 

Lo que Knockout nos permite hacer de forma muy rápida es, realizar estos bindings entre elementos del DOM y nuestro viewmodel así como crear elementos del DOM a partir del propio viewmodel (lo que antes hacíamos con jquery_tmpl p.ej.)

**Hello Knockout**

Vamos a empezar con el primer proyecto usando Knockout. En este caso vamos a montar una tabla que muestre información de cervezas. Para ello, nos creamos un nuevo proyecto de tipo WebApi, donde vamos a añadir un controlador:

<div style="overflow: auto; cursor: text; font-size: 8pt; font-family: 'Courier New', courier, monospace; direction: ltr; text-align: left; margin: 20px 0px 10px; line-height: 12pt; max-height: 200px; width: 97.5%; background-color: #f4f4f4; border: silver 1px solid; padding: 4px;" id="codeSnippetWrapper">
  <div style="overflow: visible; font-size: 8pt; font-family: 'Courier New', courier, monospace; color: black; direction: ltr; text-align: left; line-height: 12pt; width: 100%; background-color: #f4f4f4; border-style: none; padding: 0px;" id="codeSnippet">
    <pre style="overflow: visible; font-size: 8pt; font-family: 'Courier New', courier, monospace; color: black; direction: ltr; text-align: left; margin: 0em; line-height: 12pt; width: 100%; background-color: white; border-style: none; padding: 0px;"><span style="color: #606060" id="lnum1">   1:</span> <span style="color: #0000ff">public</span> <span style="color: #0000ff">class</span> ApiBeersController : ApiController</pre>
    
    <p>
      <!--CRLF-->
    </p>
    
    <pre style="overflow: visible; font-size: 8pt; font-family: 'Courier New', courier, monospace; color: black; direction: ltr; text-align: left; margin: 0em; line-height: 12pt; width: 100%; background-color: #f4f4f4; border-style: none; padding: 0px;"><span style="color: #606060" id="lnum2">   2:</span> {</pre>
    
    <p>
      <!--CRLF-->
    </p>
    
    <pre style="overflow: visible; font-size: 8pt; font-family: 'Courier New', courier, monospace; color: black; direction: ltr; text-align: left; margin: 0em; line-height: 12pt; width: 100%; background-color: white; border-style: none; padding: 0px;"><span style="color: #606060" id="lnum3">   3:</span>     <span style="color: #0000ff">public</span> IEnumerable&lt;Beer&gt; GetAll()</pre>
    
    <p>
      <!--CRLF-->
    </p>
    
    <pre style="overflow: visible; font-size: 8pt; font-family: 'Courier New', courier, monospace; color: black; direction: ltr; text-align: left; margin: 0em; line-height: 12pt; width: 100%; background-color: #f4f4f4; border-style: none; padding: 0px;"><span style="color: #606060" id="lnum4">   4:</span>     {</pre>
    
    <p>
      <!--CRLF-->
    </p>
    
    <pre style="overflow: visible; font-size: 8pt; font-family: 'Courier New', courier, monospace; color: black; direction: ltr; text-align: left; margin: 0em; line-height: 12pt; width: 100%; background-color: white; border-style: none; padding: 0px;"><span style="color: #606060" id="lnum5">   5:</span>         <span style="color: #0000ff">yield</span> <span style="color: #0000ff">return</span> <span style="color: #0000ff">new</span> Beer { Name = <span style="color: #006080">"Estrella Damm"</span>, Abv = 5.4M, Ibu = 21 };</pre>
    
    <p>
      <!--CRLF-->
    </p>
    
    <pre style="overflow: visible; font-size: 8pt; font-family: 'Courier New', courier, monospace; color: black; direction: ltr; text-align: left; margin: 0em; line-height: 12pt; width: 100%; background-color: #f4f4f4; border-style: none; padding: 0px;"><span style="color: #606060" id="lnum6">   6:</span>         <span style="color: #0000ff">yield</span> <span style="color: #0000ff">return</span> <span style="color: #0000ff">new</span> Beer { Name = <span style="color: #006080">"Heineken"</span>, Abv = 5.0M, Ibu = 10 };</pre>
    
    <p>
      <!--CRLF-->
    </p>
    
    <pre style="overflow: visible; font-size: 8pt; font-family: 'Courier New', courier, monospace; color: black; direction: ltr; text-align: left; margin: 0em; line-height: 12pt; width: 100%; background-color: white; border-style: none; padding: 0px;"><span style="color: #606060" id="lnum7">   7:</span>     }</pre>
    
    <p>
      <!--CRLF-->
    </p>
    
    <pre style="overflow: visible; font-size: 8pt; font-family: 'Courier New', courier, monospace; color: black; direction: ltr; text-align: left; margin: 0em; line-height: 12pt; width: 100%; background-color: #f4f4f4; border-style: none; padding: 0px;"><span style="color: #606060" id="lnum8">   8:</span> }</pre>
    
    <p>
      <!--CRLF--></div> </div> 
      
      <p>
        La clase Beer simplemente contiene las propiedades Name, Abv e Ibu.
      </p>
      
      <p>
        Si ahora abrimos una URL y navegamos a /api/apibeers deberíamos recibir un json:
      </p>
      
      <pre>[{"Name":"Estrella Damm","Abv":5.4,"Ibu":21},{"Name":"Heineken","Abv":5.0,"Ibu":10}]</pre>
      
      <blockquote>
        <p>
          <strong>Nota:</strong> Si recibes un XML en lugar de un JSON eso es debido a la cabecera Accept que envía el navegador. Una solución para ello es indicarle a WebApi que nunca devuelva resultados en XML, lo que se consigue colocando la siguiente línea en el Application_Start: <em>GlobalConfiguration.Configuration.Formatters.Remove(GlobalConfiguration.Configuration.Formatters.XmlFormatter);</em>
        </p>
      </blockquote>
      
      <p>
        &nbsp;
      </p>
      
      <blockquote>
        <p>
          Ahora vamos a incluir a Knockout en nuestra página master.
        </p>
      </blockquote>
      
      <p>
        Para ello damos de alta un nuevo bundle para incluir Knockout (en el método RegisterBundles de la clase BundleConfig):
      </p>
      
      <div style="overflow: auto; cursor: text; font-size: 8pt; font-family: 'Courier New', courier, monospace; direction: ltr; text-align: left; margin: 20px 0px 10px; line-height: 12pt; max-height: 200px; width: 97.5%; background-color: #f4f4f4; border: silver 1px solid; padding: 4px;" id="codeSnippetWrapper">
        <div style="overflow: visible; font-size: 8pt; font-family: 'Courier New', courier, monospace; color: black; direction: ltr; text-align: left; line-height: 12pt; width: 100%; background-color: #f4f4f4; border-style: none; padding: 0px;" id="codeSnippet">
          <pre style="overflow: visible; font-size: 8pt; font-family: 'Courier New', courier, monospace; color: black; direction: ltr; text-align: left; margin: 0em; line-height: 12pt; width: 100%; background-color: white; border-style: none; padding: 0px;"><span style="color: #606060" id="lnum1">   1:</span> bundles.Add(<span style="color: #0000ff">new</span> ScriptBundle(<span style="color: #006080">"~/bundles/ko"</span>).Include(</pre>
          
          <p>
            <!--CRLF-->
          </p>
          
          <pre style="overflow: visible; font-size: 8pt; font-family: 'Courier New', courier, monospace; color: black; direction: ltr; text-align: left; margin: 0em; line-height: 12pt; width: 100%; background-color: #f4f4f4; border-style: none; padding: 0px;"><span style="color: #606060" id="lnum2">   2:</span>                         <span style="color: #006080">"~/Scripts/knockout-2*"</span>));</pre>
          
          <p>
            <!--CRLF--></div> </div> 
            
            <p>
              Ok, ahora vamos a crear un controlador &ldquo;normal&rdquo; que nos devuelva la vista que debe mostrar las cervezas:
            </p>
            
            <div style="overflow: auto; cursor: text; font-size: 8pt; font-family: 'Courier New', courier, monospace; direction: ltr; text-align: left; margin: 20px 0px 10px; line-height: 12pt; max-height: 200px; width: 97.5%; background-color: #f4f4f4; border: silver 1px solid; padding: 4px;" id="codeSnippetWrapper">
              <div style="overflow: visible; font-size: 8pt; font-family: 'Courier New', courier, monospace; color: black; direction: ltr; text-align: left; line-height: 12pt; width: 100%; background-color: #f4f4f4; border-style: none; padding: 0px;" id="codeSnippet">
                <pre style="overflow: visible; font-size: 8pt; font-family: 'Courier New', courier, monospace; color: black; direction: ltr; text-align: left; margin: 0em; line-height: 12pt; width: 100%; background-color: white; border-style: none; padding: 0px;"><span style="color: #606060" id="lnum1">   1:</span> &lt;h2&gt;Index de cervezas&lt;/h2&gt;</pre>
                
                <p>
                  <!--CRLF-->
                </p>
                
                <pre style="overflow: visible; font-size: 8pt; font-family: 'Courier New', courier, monospace; color: black; direction: ltr; text-align: left; margin: 0em; line-height: 12pt; width: 100%; background-color: #f4f4f4; border-style: none; padding: 0px;"><span style="color: #606060" id="lnum2">   2:</span>&nbsp; </pre>
                
                <p>
                  <!--CRLF-->
                </p>
                
                <pre style="overflow: visible; font-size: 8pt; font-family: 'Courier New', courier, monospace; color: black; direction: ltr; text-align: left; margin: 0em; line-height: 12pt; width: 100%; background-color: white; border-style: none; padding: 0px;"><span style="color: #606060" id="lnum3">   3:</span> &lt;script type=<span style="color: #006080">"text/javascript"</span>&gt;</pre>
                
                <p>
                  <!--CRLF-->
                </p>
                
                <pre style="overflow: visible; font-size: 8pt; font-family: 'Courier New', courier, monospace; color: black; direction: ltr; text-align: left; margin: 0em; line-height: 12pt; width: 100%; background-color: #f4f4f4; border-style: none; padding: 0px;"><span style="color: #606060" id="lnum4">   4:</span>     $(document).ready(<span style="color: #0000ff">function</span> () {</pre>
                
                <p>
                  <!--CRLF-->
                </p>
                
                <pre style="overflow: visible; font-size: 8pt; font-family: 'Courier New', courier, monospace; color: black; direction: ltr; text-align: left; margin: 0em; line-height: 12pt; width: 100%; background-color: white; border-style: none; padding: 0px;"><span style="color: #606060" id="lnum5">   5:</span>         <span style="color: #0000ff">var</span> url=<span style="color: #006080">"@Url.RouteUrl("</span>DefaultApi<span style="color: #006080">", new {httproute="</span><span style="color: #006080">", controller="</span>ApiBeers<span style="color: #006080">"})"</span>;</pre>
                
                <p>
                  <!--CRLF-->
                </p>
                
                <pre style="overflow: visible; font-size: 8pt; font-family: 'Courier New', courier, monospace; color: black; direction: ltr; text-align: left; margin: 0em; line-height: 12pt; width: 100%; background-color: #f4f4f4; border-style: none; padding: 0px;"><span style="color: #606060" id="lnum6">   6:</span>         $.getJSON(url, <span style="color: #0000ff">function</span>(data) {</pre>
                
                <p>
                  <!--CRLF-->
                </p>
                
                <pre style="overflow: visible; font-size: 8pt; font-family: 'Courier New', courier, monospace; color: black; direction: ltr; text-align: left; margin: 0em; line-height: 12pt; width: 100%; background-color: white; border-style: none; padding: 0px;"><span style="color: #606060" id="lnum7">   7:</span>             ko.applyBindings({beers: data});</pre>
                
                <p>
                  <!--CRLF-->
                </p>
                
                <pre style="overflow: visible; font-size: 8pt; font-family: 'Courier New', courier, monospace; color: black; direction: ltr; text-align: left; margin: 0em; line-height: 12pt; width: 100%; background-color: #f4f4f4; border-style: none; padding: 0px;"><span style="color: #606060" id="lnum8">   8:</span>         });</pre>
                
                <p>
                  <!--CRLF-->
                </p>
                
                <pre style="overflow: visible; font-size: 8pt; font-family: 'Courier New', courier, monospace; color: black; direction: ltr; text-align: left; margin: 0em; line-height: 12pt; width: 100%; background-color: white; border-style: none; padding: 0px;"><span style="color: #606060" id="lnum9">   9:</span>     });</pre>
                
                <p>
                  <!--CRLF-->
                </p>
                
                <pre style="overflow: visible; font-size: 8pt; font-family: 'Courier New', courier, monospace; color: black; direction: ltr; text-align: left; margin: 0em; line-height: 12pt; width: 100%; background-color: #f4f4f4; border-style: none; padding: 0px;"><span style="color: #606060" id="lnum10">  10:</span> &lt;/script&gt;</pre>
                
                <p>
                  <!--CRLF-->
                </p>
                
                <pre style="overflow: visible; font-size: 8pt; font-family: 'Courier New', courier, monospace; color: black; direction: ltr; text-align: left; margin: 0em; line-height: 12pt; width: 100%; background-color: white; border-style: none; padding: 0px;"><span style="color: #606060" id="lnum11">  11:</span>&nbsp; </pre>
                
                <p>
                  <!--CRLF-->
                </p>
                
                <pre style="overflow: visible; font-size: 8pt; font-family: 'Courier New', courier, monospace; color: black; direction: ltr; text-align: left; margin: 0em; line-height: 12pt; width: 100%; background-color: #f4f4f4; border-style: none; padding: 0px;"><span style="color: #606060" id="lnum12">  12:</span>&nbsp; </pre>
                
                <p>
                  <!--CRLF-->
                </p>
                
                <pre style="overflow: visible; font-size: 8pt; font-family: 'Courier New', courier, monospace; color: black; direction: ltr; text-align: left; margin: 0em; line-height: 12pt; width: 100%; background-color: white; border-style: none; padding: 0px;"><span style="color: #606060" id="lnum13">  13:</span> &lt;div id=<span style="color: #006080">"beers"</span>&gt;</pre>
                
                <p>
                  <!--CRLF-->
                </p>
                
                <pre style="overflow: visible; font-size: 8pt; font-family: 'Courier New', courier, monospace; color: black; direction: ltr; text-align: left; margin: 0em; line-height: 12pt; width: 100%; background-color: #f4f4f4; border-style: none; padding: 0px;"><span style="color: #606060" id="lnum14">  14:</span>     &lt;table&gt;</pre>
                
                <p>
                  <!--CRLF-->
                </p>
                
                <pre style="overflow: visible; font-size: 8pt; font-family: 'Courier New', courier, monospace; color: black; direction: ltr; text-align: left; margin: 0em; line-height: 12pt; width: 100%; background-color: white; border-style: none; padding: 0px;"><span style="color: #606060" id="lnum15">  15:</span>         &lt;tbody data-bind=<span style="color: #006080">"foreach:beers"</span>&gt;</pre>
                
                <p>
                  <!--CRLF-->
                </p>
                
                <pre style="overflow: visible; font-size: 8pt; font-family: 'Courier New', courier, monospace; color: black; direction: ltr; text-align: left; margin: 0em; line-height: 12pt; width: 100%; background-color: #f4f4f4; border-style: none; padding: 0px;"><span style="color: #606060" id="lnum16">  16:</span>             &lt;tr&gt;</pre>
                
                <p>
                  <!--CRLF-->
                </p>
                
                <pre style="overflow: visible; font-size: 8pt; font-family: 'Courier New', courier, monospace; color: black; direction: ltr; text-align: left; margin: 0em; line-height: 12pt; width: 100%; background-color: white; border-style: none; padding: 0px;"><span style="color: #606060" id="lnum17">  17:</span>                 &lt;td data-bind=<span style="color: #006080">"text: Name"</span> /&gt;</pre>
                
                <p>
                  <!--CRLF-->
                </p>
                
                <pre style="overflow: visible; font-size: 8pt; font-family: 'Courier New', courier, monospace; color: black; direction: ltr; text-align: left; margin: 0em; line-height: 12pt; width: 100%; background-color: #f4f4f4; border-style: none; padding: 0px;"><span style="color: #606060" id="lnum18">  18:</span>                 &lt;td data-bind=<span style="color: #006080">"text: Abv"</span> /&gt;</pre>
                
                <p>
                  <!--CRLF-->
                </p>
                
                <pre style="overflow: visible; font-size: 8pt; font-family: 'Courier New', courier, monospace; color: black; direction: ltr; text-align: left; margin: 0em; line-height: 12pt; width: 100%; background-color: white; border-style: none; padding: 0px;"><span style="color: #606060" id="lnum19">  19:</span>                 &lt;td data-bind=<span style="color: #006080">"text: Ibu"</span> /&gt;</pre>
                
                <p>
                  <!--CRLF-->
                </p>
                
                <pre style="overflow: visible; font-size: 8pt; font-family: 'Courier New', courier, monospace; color: black; direction: ltr; text-align: left; margin: 0em; line-height: 12pt; width: 100%; background-color: #f4f4f4; border-style: none; padding: 0px;"><span style="color: #606060" id="lnum20">  20:</span>             &lt;/tr&gt;</pre>
                
                <p>
                  <!--CRLF-->
                </p>
                
                <pre style="overflow: visible; font-size: 8pt; font-family: 'Courier New', courier, monospace; color: black; direction: ltr; text-align: left; margin: 0em; line-height: 12pt; width: 100%; background-color: white; border-style: none; padding: 0px;"><span style="color: #606060" id="lnum21">  21:</span>         &lt;/tbody&gt;</pre>
                
                <p>
                  <!--CRLF-->
                </p>
                
                <pre style="overflow: visible; font-size: 8pt; font-family: 'Courier New', courier, monospace; color: black; direction: ltr; text-align: left; margin: 0em; line-height: 12pt; width: 100%; background-color: #f4f4f4; border-style: none; padding: 0px;"><span style="color: #606060" id="lnum22">  22:</span>     &lt;/table&gt;</pre>
                
                <p>
                  <!--CRLF-->
                </p>
                
                <pre style="overflow: visible; font-size: 8pt; font-family: 'Courier New', courier, monospace; color: black; direction: ltr; text-align: left; margin: 0em; line-height: 12pt; width: 100%; background-color: white; border-style: none; padding: 0px;"><span style="color: #606060" id="lnum23">  23:</span> &lt;/div&gt;</pre>
                
                <p>
                  <!--CRLF--></div> </div> 
                  
                  <p>
                    Analicemos el código:
                  </p>
                  
                  <ol>
                    <li>
                      Al cargarse la página usamos $.getJSON para obtener los datos de las cervezas
                    </li>
                    <li>
                      Una vez hemos obtenido el objeto JSON con las cervezas (recordad que era un IEnumerable<Beer> en C# y un array puro en javascript) creamos un objeto con una propiedad llamada &ldquo;beers&rdquo; cuyo contenido es precisamente este array de cervezas.
                    </li>
                    <li>
                      Llamamos al método applyBindings, para Knockout haga su magia. A applyBindings se le pasa el viewmodel a usar.
                    </li>
                  </ol>
                  
                  <p>
                    ¿Y como hace el enlace Knockout? Pues, y ahí radica su gracia, usa un atributo propio llamado <strong>data-bind</strong>. Recordad que HTML5 nos permite definir nuestros propios atributos siempre y cuando empiecen por data-. Knockout buscará los elementos que estén marcados con este atributo data-bind y realizará distintas tareas según el valor de dicho atributo y del viewmodel definido. En este primer ejemplo vemos dos usos de data-bind:
                  </p>
                  
                  <ul>
                    <li>
                      foreach: propiedad &ndash;> Itera sobre todos los elementos de la propiedad especificada. En nuestro caso usamos foreach:beers, ya que beers es el nombre de la propiedad de nuestro viewmodel que contiene el array de cervezas. Por cada elemento de la propiedad beers, knockout repetirá todos los elementos que estén <em>dentro</em> del elemento que contiene el foreach. En nuestro caso creará un <tr> por cada elemento.
                    </li>
                    <li>
                      text: propiedad &ndash;> Muestra el texto de la propiedad especificada. Como estamos dentro de un foreach muestra la propiedad especificada del elemento que se está &ldquo;renderizando&rdquo;.
                    </li>
                  </ul>
                  
                  <p>
                    El resultado de crear esta vista es, tal y como seguro que esperabais:
                  </p>
                  
                  <p>
                    <img height="186" width="296" src="/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_5674A3E7.png" alt="image" border="0" style="border-top: 0px; border-right: 0px; border-bottom: 0px; border-left: 0px; display: inline" title="image" />
                  </p>
                  
                  <p>
                    Bueno... ¿no está nada mal por un par de líneas, no? ¿A que ya os está picando la curiosidad?
                  </p>
                  
                  <p>
                    En los próximos posts iremos ampliando y viendo muchas otras cosas que Knockout nos ofrece! 😀
                  </p>

 [1]: http://engineering.twitter.com/2012/05/improving-performance-on-twittercom.html
 [2]: /blogs/jmaguilar/archive/2012/05/07/knockout-i-pongamos-orden-en-el-lado-cliente.aspx
 [3]: http://martinfowler.com/eaaDev/SupervisingPresenter.html