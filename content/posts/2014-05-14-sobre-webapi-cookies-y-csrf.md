---
title: Sobre WebApi, cookies y CSRF
description: Sobre WebApi, cookies y CSRF
author: eiximenis

date: 2014-05-14T00:04:00+00:00
geeks_url: /?p=1667
geeks_visits:
  - 1985
geeks_ms_views:
  - 1903
categories:
  - Uncategorized

---
Buenas! Hace poco <a target="_blank" href="https://twitter.com/_pedrohurtado" rel="noopener noreferrer">Pedro Hurtado</a> ha escrito un post titulado &ldquo;<a target="_blank" href="/blogs/phurtado/archive/2014/05/13/una-evidencia-una-fricada-y-un-desprop-243-sito.aspx" rel="noopener noreferrer">Una evidencia, una frikada y un despropósito</a>&rdquo;. En él habla de varias cosas, relacionadas con la seguridad de aplicaciones web, pero quiero centrarme en un solo punto (el despropósito). Básicamente lo que dice Pedro es que si usas autenticación por cookies en WebApi eres vulnerable a ataques CSRF.

Vayamos por partes...

Antes que nada debemos aclarar que es un ataque CSRF. No es sencillo de definir, pero básicamente se trata de una vulnerabilidad que explota la confianza que un sitio web tiene en un usuario (previamente autenticado en dicho sitio web). La <a target="_blank" href="http://es.wikipedia.org/wiki/Cross_Site_Request_Forgery" rel="noopener noreferrer">explicación de la wikipedia</a> no está nada mal (aunque mejor la entrada <a target="_blank" href="http://en.wikipedia.org/wiki/Cross-site_request_forgery" rel="noopener noreferrer">inglesa</a>).

El ejemplo que pone Pedro es el siguiente, **suponiendo que tenemos habilitada la autenticación por cookies** y que el usuario se haya logado en la aplicación web (es decir, por lo tanto que la cookie esté emitida), una página web maliciosa puede hacer un ataque CSRF con algo tan simple como:

<div style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px" class="wlWriterEditableSmartContent" id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:ab9019e6-3cb9-4066-89f8-fd2ce5775ee0">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
    <div style="background: #ddd; max-height: 300px; overflow: auto">
      <ol style="background: #1d1d1d; margin: 0 0 0 2.5em; padding: 0 0 0 5px; white-space: nowrap" start="1">
        <li>
          <span style="background:#1e1e1e;color:#808080"><</span><span style="background:#1e1e1e;color:#569cd6">p</span><span style="background:#1e1e1e;color:#808080">></span><span style="background:#1e1e1e;color:#dcdcdc">[FROM ATTACKER] You have been hacked??? Who knows...</span><span style="background:#1e1e1e;color:#808080"></</span><span style="background:#1e1e1e;color:#569cd6">p</span><span style="background:#1e1e1e;color:#808080">></span>
        </li>
        <li style="background: #111111">
          &nbsp;
        </li>
        <li>
          <span style="background:#1e1e1e;color:#808080"><</span><span style="background:#1e1e1e;color:#569cd6">div</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#9cdcfe">style</span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#c8c8c8">&#8220;</span><span style="background:#1e1e1e;color:#9cdcfe">display</span><span style="background:#1e1e1e;color:#dcdcdc">:</span><span style="background:#1e1e1e;color:#c8c8c8">none&#8221;</span><span style="background:#1e1e1e;color:#808080">></span>
        </li>
        <li style="background: #111111">
          <span style="background:#1e1e1e;color:#dcdcdc">&nbsp;&nbsp;&nbsp;&nbsp;</span><span style="background:#1e1e1e;color:#808080"><</span><span style="background:#1e1e1e;color:#569cd6">form</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#9cdcfe">id</span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#c8c8c8">&#8220;attacker&#8221;</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#9cdcfe">method</span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#c8c8c8">&#8220;post&#8221;</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#9cdcfe">action</span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#c8c8c8">&#8220;http://localhost:35815/api/prueba&#8221;</span><span style="background:#1e1e1e;color:#808080">></span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</span><span style="background:#1e1e1e;color:#808080"><</span><span style="background:#1e1e1e;color:#569cd6">input</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#9cdcfe">type</span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#c8c8c8">&#8220;hidden&#8221;</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#9cdcfe">name</span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#c8c8c8">&#8220;valor&#8221;</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#9cdcfe">value</span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#c8c8c8">&#8220;hacked&#8221;</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#808080">/></span>
        </li>
        <li style="background: #111111">
          <span style="background:#1e1e1e;color:#dcdcdc">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</span><span style="background:#1e1e1e;color:#808080"><</span><span style="background:#1e1e1e;color:#569cd6">input</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#9cdcfe">type</span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#c8c8c8">&#8220;hidden&#8221;</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#9cdcfe">name</span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#c8c8c8">&#8220;valor2&#8221;</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#9cdcfe">value</span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#c8c8c8">&#8220;csrf&#8221;</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#808080">/></span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">&nbsp;&nbsp;&nbsp;&nbsp;</span><span style="background:#1e1e1e;color:#808080"></</span><span style="background:#1e1e1e;color:#569cd6">form</span><span style="background:#1e1e1e;color:#808080">></span>
        </li>
        <li style="background: #111111">
          <span style="background:#1e1e1e;color:#808080"></</span><span style="background:#1e1e1e;color:#569cd6">div</span><span style="background:#1e1e1e;color:#808080">></span>
        </li>
        <li>
          &nbsp;
        </li>
        <li style="background: #111111">
          <span style="background:#ffffb3;color:#000000">@section scripts {</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">&nbsp;&nbsp;&nbsp;&nbsp;</span><span style="background:#1e1e1e;color:#808080"><</span><span style="background:#1e1e1e;color:#569cd6">script</span><span style="background:#1e1e1e;color:#808080">></span>
        </li>
        <li style="background: #111111">
          <span style="background:#1e1e1e;color:#dcdcdc">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;$</span><span style="background:#1e1e1e;color:#b4b4b4">(</span><span style="background:#1e1e1e;color:#d69d85">&#8220;#attacker&#8221;</span><span style="background:#1e1e1e;color:#b4b4b4">).</span><span style="background:#1e1e1e;color:#dcdcdc">submit</span><span style="background:#1e1e1e;color:#b4b4b4">();</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">&nbsp;&nbsp;&nbsp;&nbsp;</span><span style="background:#1e1e1e;color:#808080"></</span><span style="background:#1e1e1e;color:#569cd6">script</span><span style="background:#1e1e1e;color:#808080">></span>
        </li>
        <li style="background: #111111">
          <span style="background:#1e1e1e;color:#dcdcdc">}</span>
        </li>
      </ol>
    </div>
  </div>
</div>

Este código estará en cualquier vista de **otra aplicación web** distinta a la que define el controlador web api. En mi caso en mi proyecto tengo:

[<img height="64" width="236" src="/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_569211A3.png" alt="image" border="0" style="border-top: 0px; border-right: 0px; background-image: none; border-bottom: 0px; padding-top: 0px; padding-left: 0px; border-left: 0px; display: inline; padding-right: 0px" title="image" />][1]

El proyecto web CSRFVictim define el controlador WebApi llamado Prueba con el método Post. Tengo pues las dos aplicaciones ejecutándose en IISExpress:

[<img height="82" width="321" src="/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_1422001E.png" alt="image" border="0" style="border-top: 0px; border-right: 0px; background-image: none; border-bottom: 0px; padding-top: 0px; padding-left: 0px; border-left: 0px; display: inline; padding-right: 0px" title="image" />][2]

La aplicación CSRFAttacker es la que tiene la vista anterior. Cuando navego a http://localhost:35594/Home/Attack (una URL de CSRFAttacker) se termina ejecutando el método Post del controlador Prueba de CSRFVictim. &iexcl;Ojo! **Con la condición de que ANTES el usuario se haya autenticado en la aplicación CSRFVictim**.

Esto tiene toda la lógica del mundo: al autenticarse en CSRFVictim se genera una cookie de autenticación. Cuando luego se navega a CSRFAttacker _desde el mismo_ navegador se hace el POST hacia CSRFVictim, el navegador envía la cookie de autenticación... Para el controlador WebApi es, a todos los efectos, como si la petición viniese de CSRFVictim. **Es una petición válida: tiene la cookie de autenticación**. Porque el navegador la envía. Insisto: es necesario que el **usuario esté autenticado antes en CSRFVictim**.

De esto podemos desprender un par de cosillas:

  1. Intenta estar logado en cualquier sitio web **el mínimo tiempo posible**. Y mientras estés logado en un sitio evita visitar otras URLs (especialmente las no confiables). **Evita siempre el uso de opciones tipo &ldquo;remember me&rdquo;.** Son cookies que carga el demonio.
  2. Vigila con las llamadas POST a tu aplicación. Si te autenticas por cookies eres vulnerable a CSRF. **Con independencia de si usas WebApi, un controlador MVC o cualquier otra cosa**. Si no haces nada más, eres por defecto vulnerable.

Si en lugar de un controlador WebApi usas un controlador MVC que responde a un POST, que se origina en un formulario HTML (p. ej. un formulario de login) puedes protegerte utilizando HtmlHelper.AntiForgeryToken(). Mira este post para los detalles: [http://blog.stevensanderson.com/2008/09/01/prevent-cross-site-request-forgery-csrf-using-aspnet-mvcs-antiforgerytoken-helper/][3]

Pero si usas WebApi no puedes usar la técnica de AntiForgeryToken, porque esa se basa en generar un input hidden... eso funciona para vistas HTML pero no para controladores que están pensados para ser llamados programáticamente. Esto nos lleva a otro consejo:

  * Evita, siempre que puedas, la autenticación por cookies en tus servicios web api.

La autenticación basada en cookies es la causa de la vulnerabilidad. Pero muchas veces se generan controladores web api para ser usados desde dentro de una aplicación web (p. ej. para soporte de llamadas Ajax). En este caso es muy útil que la misma cookie que autentique el usuario en la aplicación web, nos sirva para los servicios web api, ya que estos están pensados para ser llamados (via javascript) desde la propia aplicación web. En este caso, deberías añadir al menos algún mecanismo de seguridad adicional.

P. ej. podrías usar algo parecido a lo siguiente:

<div style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px" class="wlWriterEditableSmartContent" id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:ffb55a78-cbab-4629-b25d-7bdb2cd5c525">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
    <div style="background: #ddd; max-height: 300px; overflow: auto">
      <ol style="background: #1d1d1d; margin: 0 0 0 2.5em; padding: 0 0 0 5px; white-space: nowrap" start="1">
        <li>
          &nbsp;&nbsp;<span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">public</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">class</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#4ec9b0">AntiCSRFHandler</span><span style="background:#1e1e1e;color:#dcdcdc"> : </span><span style="background:#1e1e1e;color:#4ec9b0">DelegatingHandler</span>
        </li>
        <li style="background: #111111">
          &nbsp;&nbsp;<span style="background:#1e1e1e;color:#dcdcdc">{</span>
        </li>
        <li>
          &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">protected</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">override</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#4ec9b0">Task</span><span style="background:#1e1e1e;color:#b4b4b4"><</span><span style="background:#1e1e1e;color:#4ec9b0">HttpResponseMessage</span><span style="background:#1e1e1e;color:#b4b4b4">></span><span style="background:#1e1e1e;color:#dcdcdc"> SendAsync(</span><span style="background:#1e1e1e;color:#4ec9b0">HttpRequestMessage</span><span style="background:#1e1e1e;color:#dcdcdc"> request, </span><span style="background:#1e1e1e;color:#4ec9b0">CancellationToken</span><span style="background:#1e1e1e;color:#dcdcdc"> cancellationToken)</span>
        </li>
        <li style="background: #111111">
          &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="background:#1e1e1e;color:#dcdcdc">{</span>
        </li>
        <li>
          &nbsp;
        </li>
        <li style="background: #111111">
          &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">var</span><span style="background:#1e1e1e;color:#dcdcdc"> referrer </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> request</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">Headers</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">Referrer;</span>
        </li>
        <li>
          &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">var</span><span style="background:#1e1e1e;color:#dcdcdc"> uri </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> request</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">RequestUri;</span>
        </li>
        <li style="background: #111111">
          &nbsp;
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</span><span style="background:#1e1e1e;color:#569cd6">if</span><span style="background:#1e1e1e;color:#dcdcdc"> (referrer </span><span style="background:#1e1e1e;color:#b4b4b4">!=</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">null</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#b4b4b4">&&</span><span style="background:#1e1e1e;color:#dcdcdc"> uri </span><span style="background:#1e1e1e;color:#b4b4b4">!=</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">null</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#b4b4b4">&&</span><span style="background:#1e1e1e;color:#dcdcdc">referrer</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">Port </span><span style="background:#1e1e1e;color:#b4b4b4">==</span><span style="background:#1e1e1e;color:#dcdcdc"> uri</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">Port </span><span style="background:#1e1e1e;color:#b4b4b4">&&</span><span style="background:#1e1e1e;color:#dcdcdc"> referrer</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">Host </span><span style="background:#1e1e1e;color:#b4b4b4">==</span><span style="background:#1e1e1e;color:#dcdcdc"> uri</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">Host)</span>
        </li>
        <li style="background: #111111">
          &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="background:#1e1e1e;color:#dcdcdc">{</span>
        </li>
        <li>
          &nbsp;
        </li>
        <li style="background: #111111">
          &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">return</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">base</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">SendAsync(request, cancellationToken);</span>
        </li>
        <li>
          &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="background:#1e1e1e;color:#dcdcdc">}</span>
        </li>
        <li style="background: #111111">
          &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">var</span><span style="background:#1e1e1e;color:#dcdcdc"> response </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> request</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">CreateResponse(</span><span style="background:#1e1e1e;color:#b8d7a3">HttpStatusCode</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">InternalServerError);</span>
        </li>
        <li>
          &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="background:#1e1e1e;color:#dcdcdc">response</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">ReasonPhrase </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#d69d85">&#8220;Invalid referrer&#8221;</span><span style="background:#1e1e1e;color:#dcdcdc">;</span>
        </li>
        <li style="background: #111111">
          &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">return</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#4ec9b0">Task</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">FromResult</span><span style="background:#1e1e1e;color:#b4b4b4"><</span><span style="background:#1e1e1e;color:#4ec9b0">HttpResponseMessage</span><span style="background:#1e1e1e;color:#b4b4b4">></span><span style="background:#1e1e1e;color:#dcdcdc">(response);</span>
        </li>
        <li>
          &nbsp;
        </li>
        <li style="background: #111111">
          &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="background:#1e1e1e;color:#dcdcdc">}</span>
        </li>
        <li>
          &nbsp;&nbsp;<span style="background:#1e1e1e;color:#dcdcdc">}</span>
        </li>
      </ol>
    </div>
  </div>
</div>

Esto crea un message handler que _analiza_ todas las peticiones web a los servicios web api y si el referrer no es el mismo (port y host) que la url del servicio web, devuelve un HTTP 500.

Por supuesto tenemos que añadir el message handler a la configuración de Web Api:

<div style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px" class="wlWriterEditableSmartContent" id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:014a8b92-f253-4309-87a4-f801fd3eaa7c">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
    <div style="background: #ddd; max-height: 300px; overflow: auto">
      <ol style="background: #1d1d1d; margin: 0 0 0 2em; padding: 0 0 0 5px;" start="1">
        <li>
          <span style="background:#1e1e1e;color:#4ec9b0">GlobalConfiguration</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">Configuration</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">MessageHandlers</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">Add(</span><span style="background:#1e1e1e;color:#569cd6">new</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#4ec9b0">AntiCSRFHandler</span><span style="background:#1e1e1e;color:#dcdcdc">());</span>
        </li>
      </ol>
    </div>
  </div>
</div>

Ahora si la página atacante intenta realizar el POST recibirá un HTTP 500:

[<img height="101" width="496" src="/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_7CF6859F.png" alt="image" border="0" style="border-top: 0px; border-right: 0px; background-image: none; border-bottom: 0px; padding-top: 0px; padding-left: 0px; border-left: 0px; display: inline; padding-right: 0px" title="image" />][4]

Eso es porque el referrer contiene la URL que genera la petición (la url de la página Home/Attack de CSRFAttacker (en mi caso es <http://localhost:35594/Home/Attack> que es de distinto host y/o port que la URL del servicio web (<http://localhost:35815/api/prueba>). 

Con eso tan simple ya estamos protegidos de una página atacante. Por supuesto, esa protección no es infalible, pero es relativamente segura: Mediante código HTML no se puede establecer el referrer de una página y mediante JavaScript (usando XMLHttpRequest) en principio tampoco. E incluso si se pudiese mediante XMLHttpRequest no sería un problema muy grave ya que entonces la llamada fallaría por CORS (a menos que habilites CORS en WebApi lo que no debes hacer nunca en el caso de un servicio web api que tan solo se utiliza desde la propia aplicación web). 

Así pues recuerda: **solo debes permitir autenticación por cookies en tus servicios web api si y solo si, son servicios web api pensados para ser usados tan solo desde la propia aplicación web. Y si lo haces asegúrate de establecer algún mecanismo adicional de seguridad para evitar CSRF.**

Como siempre digo, debemos ser responsables de lo que hacemos y debemos entender lo que sucede. Es nuestro código y por lo tanto nuestra responsabilidad.

Un saludo!

 [1]: /cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_26D46924.png
 [2]: /cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_51AF5DE7.png
 [3]: http://blog.stevensanderson.com/2008/09/01/prevent-cross-site-request-forgery-csrf-using-aspnet-mvcs-antiforgerytoken-helper/ "http://blog.stevensanderson.com/2008/09/01/prevent-cross-site-request-forgery-csrf-using-aspnet-mvcs-antiforgerytoken-helper/"
 [4]: /cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_13B5CD29.png