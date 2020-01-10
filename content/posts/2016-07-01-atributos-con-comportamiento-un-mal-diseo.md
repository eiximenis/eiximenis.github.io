---
title: 'Atributos con comportamiento: un mal diseño'

author: eiximenis

date: 2016-07-01T10:17:29+00:00
geeks_url: /?p=1775
geeks_ms_views:
  - 3018
categories:
  - asp.net MVC
  - patrones

---
Tarde o temprano, todo desarrollador ya se de ASP.NET MVC o WebApi necesita hacer sus propios filtros para validaciones propias de peticiones, logging, comprobación de precondiciones… En fin, lo habitual para lo que se usan los filtros, vamos.

Y tarde o temprano este desarrollador se da cuenta de que su filtro **debería acceder a un determinado servicio** de su aplicación: quizá necesita hacer una consulta a la bbdd, o a un determinado elemento de negocio, o acceder al sistema de logging o cualquier cosa más. Y este desarrollador, que conoce (y usa) la inyección de dependencias **se encuentra con que no es posible inyectar dependencias en un filtro**. Algunos desarrolladores buscaran cualquier otra alternativa, algunas mejores que otras, pero ninguna satisfactoria: crear un singleton, una clase estática o instanciar directamente el objeto en lugar de obtenerlo como una dependencia (y rompiendo cualquier abstracción realizada). Otros desarrolladores continuarán en su búsqueda. En esta fase de terquedad que nos caracteriza, pensaran “no, no es posible. Tiene que ver alguna manera”. 

<!--more -->

Y al final su búsqueda dará sus frutos y se encontrarán con artículos como <a href="http://www.variablenotfound.com/2013/07/inyeccion-de-dependencias-en-filtros.html" target="_blank" rel="noopener noreferrer">este de variable not found,</a> donde se cuenta como hacerlo. Hay otros mecanismos para conseguirlo, pero ninguno es del todo elegante. En asp.net mvc core han introducido ServiceFilter, que a pesar de su nombre es una factoría genérica de filtros y que proporciona una solución más limpia. De nuevo José M. Aguilar lo cuenta fenomenal o sea que miraros <a href="http://www.variablenotfound.com/2015/06/inyeccion-de-dependencias-en-filtros.html" target="_blank" rel="noopener noreferrer">este otro post de su blog</a>.

Pero pocos se plantearán el fondo de la cuestión, que es muy simple. Los filtros de ASP.NET MVC son atributos y **no debería haber motivos para tener inyectar nada a un atributo**.

Los atributos son especiales. Los crea el CLR y su objetivo es que contengan **metadatos** y los metadatos son datos. Los datos son eso datos. No tienen comportamiento. No deben tenerlo. Un filtro de ASP.NET MVC **tiene comportamiento**. Tomemos por ejemplo el, posiblemente, más famoso de todos ellos: [Authorize].

Podemos decorar una acción con este atributo y automáticamente ASP.NET MVC se encargará de asegurar que la petición está autenticada (p. ej. con la cookie) antes de permitir llamar a la acción. Si no lo está, devolverá un HTTP 401 como respuesta. Su uso es, realmente muy cómodo:

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:388f3155-6e70-4b21-8b26-aeefa4c93f20" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
    <div style="background: #ddd; max-height: 300px; overflow: auto">
      <ol start="1" style="background: #ffffff; margin: 0 0 0 2em; padding: 0 0 0 5px; white-space: nowrap">
        <li>
          <span style="color:#000000">[</span><span style="color:#2b91af">Authorize</span><span style="color:#000000">]</span>
        </li>
        <li>
          <span style="color:#000000"></span><span style="color:#0000ff">public</span><span style="color:#000000"> </span><span style="color:#2b91af">ActionResult</span><span style="color:#000000"> UserProfile()</span>
        </li>
        <li>
          <span style="color:#000000">{</span>
        </li>
        <li>
              <span style="color:#000000"></span><span style="color:#008000">// Codigo</span>
        </li>
        <li>
          <span style="color:#000000">}</span>
        </li>
      </ol>
    </div></p>
  </div></p>
</div>

¡Eso está bien! Es fácil de usar para el desarrollador. El problema viene **con la implementación de esto**.&nbsp; Basta con ver el <a href="https://github.com/ASP-NET-MVC/aspnetwebstack/blob/master/src/System.Web.Mvc/AuthorizeAttribute.cs" target="_blank" rel="noopener noreferrer">código fuente de AuthorizeAttribute</a> para ver que es la propia clase que define el atributo la que hace todo el trabajo y que se encarga de verificar que la petición está autenticada.

**Y eso es un error**. El atributo [Authorize] deberían tan solo “marcar” que una acción debe estar protegida contra acceso anónimo (eso son metadatos asociados a la acción, está bien que estén en un atributo). Y el framework debería **ofrecer otro interfaz (y una o más posibles implementaciones) para ofrecer el comportamiento**.

De este modo **separamos los metadatos del comportamiento**. Los primeros se mantienen en los atributos y el comportamiento se traslada a clases normales.

Veamos un ejemplo de **como podría ser esta supuesta API.** Dado que tenemos varios tipos de filtro (autenticación, de acción) que reciben “contextos” distintos podríamos usar una clase base derivada de Attribute para diferenciarlos. Así MVC podría definir:

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:4d8f9ded-13cb-45d2-8100-bd5c9fe66eae" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
    <div style="background: #ddd; max-height: 300px; overflow: auto">
      <ol start="1" style="background: #ffffff; margin: 0 0 0 2em; padding: 0 0 0 5px; white-space: nowrap">
        <li>
          <span style="color:#000000"></span><span style="color:#0000ff">public</span><span style="color:#000000"> </span><span style="color:#0000ff">class</span><span style="color:#000000"> </span><span style="color:#2b91af">MvcActionAttribute</span><span style="color:#000000"> : </span><span style="color:#2b91af">Attribute</span><span style="color:#000000"> { }</span>
        </li>
        <li>
          <span style="color:#000000"></span><span style="color:#0000ff">public</span><span style="color:#000000"> </span><span style="color:#0000ff">class</span><span style="color:#000000"> </span><span style="color:#2b91af">MvcAuthAttribute</span><span style="color:#000000"> : </span><span style="color:#2b91af">Attribute</span><span style="color:#000000"> { }</span>
        </li>
      </ol>
    </div></p>
  </div></p>
</div>

Así AuthorizeAttribute heredaría de MvcAuthAttribute y solo añadiría los datos necesarios (p. ej. el nombre de los usuarios o los roles). Lo mismo para el resto de atributos: **solo contendrían datos** (propiedades).

Ahora el siguiente punto es asociar el comportamiento a cada tipo de atributo. Esto se puede hacer de varias maneras. Una podría ser que MVC nos expusiese una interfaz que fuese **una factoría para crear esas clases con comportamiento**:

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:6da77f98-a1f4-4c00-a7a0-3bcdf961694c" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
    <div style="background: #ddd; max-height: 300px; overflow: auto">
      <ol start="1" style="background: #ffffff; margin: 0 0 0 2em; padding: 0 0 0 5px; white-space: nowrap">
        <li>
          <span style="color:#000000"></span><span style="color:#0000ff">public</span><span style="color:#000000"> </span><span style="color:#0000ff">interface</span><span style="color:#000000"> </span><span style="color:#2b91af">IAuthFilterFactory</span><span style="color:#000000"> {</span>
        </li>
        <li>
          &nbsp;
        </li>
        <li>
              <span style="color:#000000"></span><span style="color:#2b91af">IAuthFilterHandler</span><span style="color:#000000"> CreateForType(</span><span style="color:#2b91af">Type</span><span style="color:#000000"> attribute);</span>
        </li>
        <li>
          <span style="color:#000000">}</span>
        </li>
      </ol>
    </div></p>
  </div></p>
</div>

Tendríamos una factoría para cada uno de los tipos de filtr
  
os (autenticación, autorización, acción, error) ya que el tipo del objeto que gestiona esos filtros es distinto (reciben contextos distintos). O podría haber una sola factoría con varios métodos.

La siguiente interfaz que MVC nos tiene que proveer es la **IAuthFilterHandler** (de nuevo habría una interfaz para cada tipo de filtro):

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:985ad695-9048-4d00-b0d2-95d1e9da30eb" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
    <div style="background: #ddd; max-height: 300px; overflow: auto">
      <ol start="1" style="background: #ffffff; margin: 0 0 0 2em; padding: 0 0 0 5px; white-space: nowrap">
        <li>
          <span style="color:#000000"></span><span style="color:#0000ff">public</span><span style="color:#000000"> </span><span style="color:#0000ff">interface</span><span style="color:#000000"> </span><span style="color:#2b91af">IAuthFilterHandler</span>
        </li>
        <li>
          <span style="color:#000000">{</span>
        </li>
        <li>
              <span style="color:#000000"></span><span style="color:#0000ff">void</span><span style="color:#000000"> ApplyFilter (</span><span style="color:#2b91af">MvcAuthAttribute</span><span style="color:#000000"> filter, </span><span style="color:#2b91af">AuthorizationContext</span><span style="color:#000000"> context);</span>
        </li>
        <li>
          <span style="color:#000000">}</span>
        </li>
      </ol>
    </div></p>
  </div></p>
</div>

Ahora, podríamos crear unas versiones genéricas, para que el método “ApplyFilter” no reciba la clase base, si no realmente el atributo con su tipo y evitarnos castings. Primero, la interfaz genérica:

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:cda6e579-201b-456b-bd6f-b06ab5ca3504" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
    <div style="background: #ddd; max-height: 300px; overflow: auto">
      <ol start="1" style="background: #ffffff; margin: 0 0 0 2em; padding: 0 0 0 5px; white-space: nowrap">
        <li>
          <span style="color:#000000"></span><span style="color:#0000ff">public</span><span style="color:#000000"> </span><span style="color:#0000ff">interface</span><span style="color:#000000"> </span><span style="color:#2b91af">IAuthFilterAttribute</span><span style="color:#000000"><</span><span style="color:#2b91af">TA</span><span style="color:#000000">> : </span><span style="color:#2b91af">IAuthFilterHandler</span>
        </li>
        <li>
              <span style="color:#000000"></span><span style="color:#0000ff">where</span><span style="color:#000000"> </span><span style="color:#2b91af">TA</span><span style="color:#000000"> : </span><span style="color:#2b91af">MvcAuthAttribute</span>
        </li>
        <li>
          <span style="color:#000000">{</span>
        </li>
        <li>
              <span style="color:#000000"></span><span style="color:#0000ff">void</span><span style="color:#000000"> ApplyFilter(</span><span style="color:#2b91af">TA</span><span style="color:#000000"> filter, </span><span style="color:#2b91af">AuthorizationContext</span><span style="color:#000000"> context);</span>
        </li>
        <li>
          <span style="color:#000000">}</span>
        </li>
      </ol>
    </div></p>
  </div></p>
</div>

Y luego una clase que se encargue de hacer el “puente” entre la interfaz genérica y la que no es genérica:

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:3b123008-ed71-48f2-9913-85262622b119" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
    <div style="background: #ddd; max-height: 300px; overflow: auto">
      <ol start="1" style="background: #ffffff; margin: 0 0 0 2.5em; padding: 0 0 0 5px; white-space: nowrap">
        <li>
          <span style="color:#000000"></span><span style="color:#0000ff">public</span><span style="color:#000000"> </span><span style="color:#0000ff">abstract</span><span style="color:#000000"> </span><span style="color:#0000ff">class</span><span style="color:#000000"> </span><span style="color:#2b91af">AuthFilterHandler</span><span style="color:#000000"><</span><span style="color:#2b91af">X</span><span style="color:#000000">> : </span><span style="color:#2b91af">IAuthFilterAttribute</span><span style="color:#000000"><</span><span style="color:#2b91af">X</span><span style="color:#000000">></span>
        </li>
        <li>
              <span style="color:#000000"></span><span style="color:#0000ff">where</span><span style="color:#000000"> </span><span style="color:#2b91af">X</span><span style="color:#000000"> : </span><span style="color:#2b91af">MvcAuthAttribute</span>
        </li>
        <li>
          <span style="color:#000000">{</span>
        </li>
        <li>
              <span style="color:#000000"></span><span style="color:#0000ff">void</span><span style="color:#000000"> </span><span style="color:#2b91af">IAuthFilterHandler</span><span style="color:#000000">.ApplyFilter(</span><span style="color:#2b91af">MvcAuthAttribute</span><span style="color:#000000"> filter, </span><span style="color:#2b91af">AuthorizationContext</span><span style="color:#000000"> context)</span>
        </li>
        <li>
              <span style="color:#000000">{</span>
        </li>
        <li>
                  <span style="color:#000000">ApplyFilter((</span><span style="color:#2b91af">X</span><span style="color:#000000">)filter, context);</span>
        </li>
        <li>
              <span style="color:#000000">}</span>
        </li>
        <li>
          &nbsp;
        </li>
        <li>
              <span style="color:#000000"></span><span style="color:#0000ff">public</span><span style="color:#000000"> </span><span style="color:#0000ff">abstract</span><span style="color:#000000"> </span><span style="color:#0000ff">void</span><span style="color:#000000"> ApplyFilter(</span><span style="color:#2b91af">X</span><span style="color:#000000"> filter, </span><span style="color:#2b91af">AuthorizationContext</span><span style="color:#000000"> context);</span>
        </li>
        <li>
          <span style="color:#000000">}</span>
        </li>
      </ol>
    </div></p>
  </div></p>
</div>

Toda esa infrastructura estaría proveída por el framework (ASP.NET MVC en nuestro ejemplo). Ahora **veamos como la podríamos usar**.

Primero nos definiríamos un filtro:

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:c3200a49-6c7b-46ed-96c9-8769072418a4" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
    <div style="background: #ddd; max-height: 300px; overflow: auto">
      <ol start="1" style="background: #ffffff; margin: 0 0 0 2em; padding: 0 0 0 5px; white-space: nowrap">
        <li>
          <span style="color:#0000ff">public</span><span style="color:#000000"> </span><span style="color:#0000ff">class</span><span style="color:#000000"> </span><span style="color:#2b91af">MyAuthorizeAttribute</span><span style="color:#000000"> : </span><span style="color:#2b91af">MvcAuthAttribute</span><span style="color:#000000"> { }</span>
        </li>
      </ol>
    </div></p>
  </div></p>
</div>

Luego la clase que contiene el **comportamiento** (el _handler_):

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:9b7e5ba2-96df-4164-8f07-fe50d2f6ff13" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
    <div style="background: #ddd; max-height: 300px; overflow: auto">
      <ol start="1" style="background: #ffffff; margin: 0 0 0 2em; padding: 0 0 0 5px; white-space: nowrap">
        <li>
          <span style="color:#000000"></span><span style="color:#0000ff">public</span><span style="color:#000000
"> </span><span style="color:#0000ff">class</span><span style="color:#000000"> </span><span style="color:#2b91af">MyAuthorizeFilterHandler</span><span style="color:#000000"> : </span><span style="color:#2b91af">AuthFilterHandler</span><span style="color:#000000"><</span><span style="color:#2b91af">MyAuthorizeAttribute</span><span style="color:#000000">></span>
        </li>
        <li>
          <span style="color:#000000">{</span>
        </li>
        <li>
              <span style="color:#000000"></span><span style="color:#0000ff">public</span><span style="color:#000000"> </span><span style="color:#0000ff">override</span><span style="color:#000000"> </span><span style="color:#0000ff">void</span><span style="color:#000000"> ApplyFilter(</span><span style="color:#2b91af">MyAuthorizeAttribute</span><span style="color:#000000"> filter, </span><span style="color:#2b91af">AuthorizationContext</span><span style="color:#000000"> context)</span>
        </li>
        <li>
              <span style="color:#000000">{</span>
        </li>
        <li>
                  <span style="color:#000000"></span><span style="color:#008000">// Hacer lo necesario</span>
        </li>
        <li>
              <span style="color:#000000">}</span>
        </li>
        <li>
          <span style="color:#000000">}</span>
        </li>
      </ol>
    </div></p>
  </div></p>
</div>

Observa que heredamos de la AuthFilterHandler genérica que hemos creado antes por lo que tan solo debemos implementar el ApplyFilter que ya recibe el atributo del tipo correcto.

Y finalmente debemos implementar la factoría:

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:36c3bd1a-0f31-4104-ae5d-9cd32fc84e17" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
    <div style="background: #ddd; max-height: 300px; overflow: auto">
      <ol start="1" style="background: #ffffff; margin: 0 0 0 2em; padding: 0 0 0 5px; white-space: nowrap">
        <li>
          <span style="color:#000000"></span><span style="color:#0000ff">public</span><span style="color:#000000"> </span><span style="color:#0000ff">class</span><span style="color:#000000"> </span><span style="color:#2b91af">AuthFilterFactory</span><span style="color:#000000"> : </span><span style="color:#2b91af">IAuthFilterFactory</span>
        </li>
        <li>
          <span style="color:#000000">{</span>
        </li>
        <li>
              <span style="color:#000000"></span><span style="color:#0000ff">public</span><span style="color:#000000"> </span><span style="color:#2b91af">IAuthFilterHandler</span><span style="color:#000000"> CreateForType(</span><span style="color:#2b91af">Type</span><span style="color:#000000"> attribute)</span>
        </li>
        <li>
              <span style="color:#000000">{</span>
        </li>
        <li>
                  <span style="color:#000000"></span><span style="color:#008000">// Resolver el IAuthFilterHandler correspondiente segn el tipo del atributo</span>
        </li>
        <li>
                  <span style="color:#000000"></span><span style="color:#008000">// AQUI PODEMOS USAR DI</span>
        </li>
        <li>
              <span style="color:#000000">}</span>
        </li>
        <li>
          <span style="color:#000000">}</span>
        </li>
      </ol>
    </div></p>
  </div></p>
</div>

**Y esa es la clave**, la factoría es **nuestra** y **puede usar el contenedor de IoC para crear los handlers** si quiere y así **inyectarles cualquier dependencia**. Los atributos contienen solo datos y los handlers el comportamiento asociado.

Y “mágicamente” todas las fricciones con la inyección de dependencias se resuelven.

Cierto, faltaría como informamos al framework que factoría para crear los handlers, debe usar. Bueno, en el caso de MVC podría ser algo como:

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:e405381e-2568-4e22-b739-5bc2f7a8eb22" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
    <div style="background: #ddd; max-height: 300px; overflow: auto">
      <ol start="1" style="background: #ffffff; margin: 0 0 0 2em; padding: 0 0 0 5px; white-space: nowrap">
        <li>
          <span style="color:#000000">System.Web.Mvc.FilterCreator.Current.</span>
        </li>
        <li>
              <span style="color:#000000">SetAuthFactory(</span><span style="color:#0000ff">new</span><span style="color:#000000"> </span><span style="color:#2b91af">AuthFilterFactory</span><span style="color:#000000">());</span>
        </li>
      </ol>
    </div></p>
  </div></p>
</div>

(Siguiendo su estilo de tener _singletons_ para la configuración).

**Y eso es todo**. La idea de este post es que veas **que no es buena idea tener atributos con comportamiento**. Si desarrollas un framework y empiezas a crear atributos con comportamiento, yo te recomendaría que pararas y pensaras lo que haces. Para mi es ¡un error de diseño clarísimo!

Saludos!