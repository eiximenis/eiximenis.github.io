---
title: 'ASP.NET MVC, [Authorize] y jQuery.load'
description: 'ASP.NET MVC, [Authorize] y jQuery.load'
author: eiximenis

date: 2014-02-25T17:45:46+00:00
geeks_url: /?p=1661
geeks_visits:
  - 2143
geeks_ms_views:
  - 1725
categories:
  - Uncategorized

---
Muy buenas! Estreno el blog este 2014… dios a finales de Febrero! A ver, si empezamos a retomar el ritmo…

Este es un post sencillito, por si os encontráis con ello. La situación es la siguiente: Tenéis controladores que devuelven vistas parciales, las cuales desde JavaScript incluís dentro de vuestro DOM a través de una llamada Ajax, usando p. ej. el método load de jQuery.

Todo funciona correctamente, hasta que un día el usuario entra en el site, se va a comer y cuando vuelve pulsa uno de esos enlaces (o botones o lo que sea) que incluyen una de esas llamadas Ajax… Y ocurre que en lugar de aparecer la vista parcial, aparece la página de Login allí incrustada.

La razón, obviamente, es que la acción que devuelve la vista parcial está protegida con [Authorize] y al haber caducado la cookie de autorización, este atributo manda un HTTP 401 (no autorizado). Hasta ahí bien. Entonces entra en juego nuestro amigo FormsAuthentication, que “captura” este 401 y lo convierte en un 302 (redirección) que es lo que recibe el navegador. Desde el punto de vista del navegador, lo que llega es un 302 por lo que este, obendientemente, se redirige a la página de Login. Las peticiones Ajax hacen caso del HTTP 302 y por lo tanto el resultado de la redirección (la página de Login) se muestra.

Una alternativa sencilla y rápida para solucionar esto consiste en modificar la petición modificada por FormsAuthentication, de forma que cambiamos todos los 302 que sean resultado de una petición Ajax por un 401 y así revertir lo que FormsAuthentication hace.

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:f7a2af99-6048-40a0-a8fb-41c838ffe1c5" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
    <div style="background: #ddd; overflow: auto">
      <ol start="1" style="background: #1e1e1e; margin: 0 0 0 2.5em; padding: 0 0 0 5px;">
        <li>
          <span style="background:#1e1e1e;color:#569cd6">protected</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">void</span><span style="background:#1e1e1e;color:#dcdcdc"> Application_EndRequest()</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">{</span>
        </li>
        <li>
              <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">var</span><span style="background:#1e1e1e;color:#dcdcdc"> context </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">new</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#4ec9b0">HttpContextWrapper</span><span style="background:#1e1e1e;color:#dcdcdc">(</span><span style="background:#1e1e1e;color:#569cd6">this</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">Context);</span>
        </li>
        <li>
              <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">if</span><span style="background:#1e1e1e;color:#dcdcdc"> (</span><span style="background:#1e1e1e;color:#4ec9b0">FormsAuthentication</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">IsEnabled </span><span style="background:#1e1e1e;color:#b4b4b4">&&</span><span style="background:#1e1e1e;color:#dcdcdc"> context</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">Response</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">StatusCode </span><span style="background:#1e1e1e;color:#b4b4b4">==</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#b5cea8">302</span>
        </li>
        <li>
                  <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#b4b4b4">&&</span><span style="background:#1e1e1e;color:#dcdcdc"> context</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">Request</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">IsAjaxRequest())</span>
        </li>
        <li>
              <span style="background:#1e1e1e;color:#dcdcdc">{</span>
        </li>
        <li>
                  <span style="background:#1e1e1e;color:#dcdcdc">context</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">Response</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">Clear();</span>
        </li>
        <li>
                  <span style="background:#1e1e1e;color:#dcdcdc">context</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">Response</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">StatusCode </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#b5cea8">401</span><span style="background:#1e1e1e;color:#dcdcdc">;</span>
        </li>
        <li>
              <span style="background:#1e1e1e;color:#dcdcdc">}</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">}</span>
        </li>
      </ol>
    </div></p>
  </div></p>
</div>

Con esto **convertimos todos los 302 en 401** cuando sean peticiones Ajax y estemos bajo FormsAuthentication. Ojo, que los convertimos todos, incluso aquellos 302 legítimos que podrían haber.

Ahora ya solo queda actualizar nuestro código JavaScript y comprobar que no recibimos un 402 😉

**Postdata 1: .NET Framework 4.5**

Si usas .NET Framework 4.5 (VS2012), ya no es necesario que hagas este truco. En su lugar puedes usar la propiedad <a href="http://msdn.microsoft.com/en-us/library/system.web.httpresponse.suppressformsauthenticationredirect(v=vs.110).aspx" target="_blank" rel="noopener noreferrer">SuppressFormsAuthenticationRedirect</a> de HttpResponse y ponerla a true. Si el valor de esa propiedad es true, pues FormsAuthentication no convierte los 401 en 302.

Es una propiedad que debes establecer cada vez, por lo que lo puedes hacer de nuevo en el Application_EndRequest de Global.asax si lo deseas.

Si alguien me pregunta porque narices esa propiedad es a nivel de Response (ya me dirás tu porque el objeto Response tiene que “entender” del framework de autorización), pues no lo sé… pero no me termina de gustar, la verdad.

**Postdata 2: Katana Cookie Middleware**

Ya lo decía el bueno de <a href="http://es.wikipedia.org/wiki/Andr%C3%A9s_Montes" target="_blank" rel="noopener noreferrer">Andrés Montes</a>: La vida puede ser maravillosa. Si en FormsAuthentication arreglaron esta situación con la propiedad SuppressFormsAuthenticationRequest en el middleware de autenticación por cookies de Katana volvemos a la situación anterior. Y si usas VS2013 o los nuevos templates de ASP.NET en VS2012 no estarás usando FormsAuthentication si no el middleware de Katana. 

Por suerte Katana está mejor pensado que FormsAuthentication y podemos configurar mucho mejor el middleware de autenticación basada en cookies.

Buscad donde se configura el middleware basado en cookies de Katana, que por defecto es en el fichero App_Start/StartupAuth.cs y sustutís:

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:3f50df18-bbb0-47c4-910e-08314601378e" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
    <div style="background: #ddd; max-height: 300px; overflow: auto">
      <ol start="1" style="background: #1e1e1e
; margin: 0 0 0 2em; padding: 0 0 0 5px;">
        </p> 
        
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">app</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">UseCookieAuthentication(</span><span style="background:#1e1e1e;color:#569cd6">new</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#4ec9b0">CookieAuthenticationOptions</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">{</span>
        </li>
        <li>
              <span style="background:#1e1e1e;color:#dcdcdc">AuthenticationType </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#4ec9b0">DefaultAuthenticationTypes</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">ApplicationCookie,</span>
        </li>
        <li>
              <span style="background:#1e1e1e;color:#dcdcdc">LoginPath </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">new</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#4ec9b0">PathString</span><span style="background:#1e1e1e;color:#dcdcdc">(</span><span style="background:#1e1e1e;color:#d69d85">"/Account/Login"</span><span style="background:#1e1e1e;color:#dcdcdc">)</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">});</span>
        </li>
      </ol>
    </div></p>
  </div></p>
</div>

por:

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:e96ec975-0217-4c37-a2c3-5c1365d48b94" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
    <div style="background: #ddd; max-height: 300px; overflow: auto">
      <ol start="1" style="background: #1e1e1e; margin: 0 0 0 2.5em; padding: 0 0 0 5px;">
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">app</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">UseCookieAuthentication(</span><span style="background:#1e1e1e;color:#569cd6">new</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#4ec9b0">CookieAuthenticationOptions</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">{</span>
        </li>
        <li>
              <span style="background:#1e1e1e;color:#dcdcdc">AuthenticationType </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#4ec9b0">DefaultAuthenticationTypes</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">ApplicationCookie,</span>
        </li>
        <li>
              <span style="background:#1e1e1e;color:#dcdcdc">LoginPath </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">new</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#4ec9b0">PathString</span><span style="background:#1e1e1e;color:#dcdcdc">(</span><span style="background:#1e1e1e;color:#d69d85">"/Account/Login"</span><span style="background:#1e1e1e;color:#dcdcdc">),</span>
        </li>
        <li>
              <span style="background:#1e1e1e;color:#dcdcdc">Provider </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">new</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#4ec9b0">CookieAuthenticationProvider</span>
        </li>
        <li>
              <span style="background:#1e1e1e;color:#dcdcdc">{</span>
        </li>
        <li>
                  <span style="background:#1e1e1e;color:#dcdcdc">OnApplyRedirect </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> ctx </span><span style="background:#1e1e1e;color:#b4b4b4">=></span>
        </li>
        <li>
                  <span style="background:#1e1e1e;color:#dcdcdc">{</span>
        </li>
        <li>
                      <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">if</span><span style="background:#1e1e1e;color:#dcdcdc"> (</span><span style="background:#1e1e1e;color:#b4b4b4">!</span><span style="background:#1e1e1e;color:#dcdcdc">IsAjaxRequest(ctx</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">Request))</span>
        </li>
        <li>
                      <span style="background:#1e1e1e;color:#dcdcdc">{</span>
        </li>
        <li>
                          <span style="background:#1e1e1e;color:#dcdcdc">ctx</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">Response</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">Redirect(ctx</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">RedirectUri);</span>
        </li>
        <li>
                      <span style="background:#1e1e1e;color:#dcdcdc">}</span>
        </li>
        <li>
                  <span style="background:#1e1e1e;color:#dcdcdc">}</span>
        </li>
        <li>
              <span style="background:#1e1e1e;color:#dcdcdc">}</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">});</span>
        </li>
      </ol>
    </div></p>
  </div></p>
</div>

De esta manera tomamos el control del redirect por 401 y lo hacemos solo si la request no es Ajax. Bueno, bonito y barato. 

Ah si! El método IsAjaxResponse… Este método **no** es el método IsAjaxResponse clásico (ctx.Request es una IOwinRequest) así que os lo tendréis que crear vosotros. Aquí os pongo una implementación:

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:84312346-708d-4400-8fa8-464216877041" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
    <div style="background: #ddd; max-height: 300px; overflow: auto">
      <ol start="1" style="background: #1e1e1e; margin: 0 0 0 2.5em; padding: 0 0 0 5px;">
        <li>
          <span style="background:#1e1e1e;color:#569cd6">public</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">static</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">bool</span><span style="background:#1e1e1e;color:#dcdcdc"> IsAjaxRequest(</span><span style="background:#1e1e1e;color:#b8d7a3">IOwinRequest</span><span style="background:#1e1e1e;color:#dcdcdc"> request)</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">{</span>
        </li>
        <li>
              <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#b8d7a3">IReadableStringCollection</span><span style="background:#1e1e1e;color:#dcdcdc"> query </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> request</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">Query;</span>
        </li>
        <li>
              <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">if</span><span style="background:#1e1e1e;color:#dcdcdc"> (query </span><span style="background:#1e1e1e;color:#b4b4b4">!=</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">null</span><span style="background:#1e1e1e;color:#dcdcdc">)</span>
        </li>
        <li>
              <span style="background:#1e1e1e;color:#dcdcdc">{</span>
        </li>
        <li>
                  <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">if</span><span style="backgrou
nd:#1e1e1e;color:#dcdcdc"> (query[</span><span style="background:#1e1e1e;color:#d69d85">"X-Requested-With"</span><span style="background:#1e1e1e;color:#dcdcdc">] </span><span style="background:#1e1e1e;color:#b4b4b4">==</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#d69d85">"XMLHttpRequest"</span><span style="background:#1e1e1e;color:#dcdcdc">)</span>
        </li>
        <li>
                  <span style="background:#1e1e1e;color:#dcdcdc">{</span>
        </li>
        <li>
                      <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">return</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">true</span><span style="background:#1e1e1e;color:#dcdcdc">;</span>
        </li>
        <li>
                  <span style="background:#1e1e1e;color:#dcdcdc">}</span>
        </li>
        <li>
              <span style="background:#1e1e1e;color:#dcdcdc">}</span>
        </li>
        <li>
          &nbsp;
        </li>
        <li>
              <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#b8d7a3">IHeaderDictionary</span><span style="background:#1e1e1e;color:#dcdcdc"> headers </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> request</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">Headers;</span>
        </li>
        <li>
              <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">if</span><span style="background:#1e1e1e;color:#dcdcdc"> (headers </span><span style="background:#1e1e1e;color:#b4b4b4">!=</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">null</span><span style="background:#1e1e1e;color:#dcdcdc">)</span>
        </li>
        <li>
              <span style="background:#1e1e1e;color:#dcdcdc">{</span>
        </li>
        <li>
                  <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">if</span><span style="background:#1e1e1e;color:#dcdcdc"> (headers[</span><span style="background:#1e1e1e;color:#d69d85">"X-Requested-With"</span><span style="background:#1e1e1e;color:#dcdcdc">] </span><span style="background:#1e1e1e;color:#b4b4b4">==</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#d69d85">"XMLHttpRequest"</span><span style="background:#1e1e1e;color:#dcdcdc">)</span>
        </li>
        <li>
                  <span style="background:#1e1e1e;color:#dcdcdc">{</span>
        </li>
        <li>
                      <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">return</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">true</span><span style="background:#1e1e1e;color:#dcdcdc">;</span>
        </li>
        <li>
                  <span style="background:#1e1e1e;color:#dcdcdc">}</span>
        </li>
        <li>
              <span style="background:#1e1e1e;color:#dcdcdc">}</span>
        </li>
        <li>
              <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">return</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">false</span><span style="background:#1e1e1e;color:#dcdcdc">;</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">}</span>
        </li>
      </ol>
    </div></p>
  </div></p>
</div>

No,&#160; no me deis las gracias… Si el método IsAjaxRequest os funciona las dais al equipo de Katana, ya que está copiado del código fuente de Katana (concretamente de <a href="https://katanaproject.codeplex.com/SourceControl/latest#src/Microsoft.Owin.Security.Cookies/Provider/DefaultBehavior.cs" target="_blank" rel="noopener noreferrer">aquí</a>). Si, si… yo también me pregunto porque es privado este método y no un método de extensión público.

En fin… eso es todo! Espero que os sea útil!

Saludos!

**Fuentes usadas:**

  * [http://jvance.com/blog/2012/09/21/FixFormsAuthentication302.xhtml][1] –> Muestra la solución aplicable a .NET Framework 4 y anteriores.
  * [http://brockallen.com/2013/10/27/using-cookie-authentication-middleware-with-web-api-and-401-response-codes/][2] –> Muestra la solución para Katana
  * [http://haacked.com/archive/2011/10/04/prevent-forms-authentication-login-page-redirect-when-you-donrsquot-want.aspx/][3] –> Phil Haack siempre es interesante de leer…

 [1]: http://jvance.com/blog/2012/09/21/FixFormsAuthentication302.xhtml "http://jvance.com/blog/2012/09/21/FixFormsAuthentication302.xhtml"
 [2]: http://brockallen.com/2013/10/27/using-cookie-authentication-middleware-with-web-api-and-401-response-codes/ "http://brockallen.com/2013/10/27/using-cookie-authentication-middleware-with-web-api-and-401-response-codes/"
 [3]: http://haacked.com/archive/2011/10/04/prevent-forms-authentication-login-page-redirect-when-you-donrsquot-want.aspx/ "http://haacked.com/archive/2011/10/04/prevent-forms-authentication-login-page-redirect-when-you-donrsquot-want.aspx/"