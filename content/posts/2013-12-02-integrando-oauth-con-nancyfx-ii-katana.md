---
title: Integrando oAuth con NancyFx (ii) – Katana
description: Integrando oAuth con NancyFx (ii) – Katana
author: eiximenis

date: 2013-12-02T17:57:10+00:00
geeks_url: /?p=1659
geeks_visits:
  - 1113
geeks_ms_views:
  - 981
categories:
  - Uncategorized

---
En el <a href="http://geeks.ms/blogs/etomas/archive/2013/11/29/integrando-oauth-con-nancyfx-i.aspx" target="_blank" rel="noopener noreferrer">post anterior</a> vimos como autenticar una aplicación NancyFx usando oAuth a través del paquete WorldDomination (o SimpleAuthentication que es el aburrido nombre que tiene ahora :p).

Pero dado que NancyFx puede funcionar como un componente OWIN y la estructura modular de OWIN permite que haya módulos de autenticación que se ejecuten antes en el pipeline, porque no “eliminar” toda responsabilidad sobre autenticación de NancyFx? Y que sea algún módulo OWIN el que lo haga no? A fin de cuentas, esa es la gracia de OWIN. En este post vamos a ver como integrar los módulos OWIN de autenticación que tiene Katana con NancyFx. Repasa el post anterior y haz lo siguiente:

  1. Crea una aplicación web vacía y añade los paquetes de Nancy, Nancy.Owin y Microsoft.Host.SystemWeb. 
  2. Crea la clase de inicialización OWIN para que use Nancy. 
  3. Crea un&#160; módulo que redirija las peticiones a la URL / a una vista que muestre un enlace de login with twitter (que vaya p. ej. a /login/twitter). 
  4. Crea otro módulo que redirija las peticiones de /secured a otra vista (basta que muestre un contenido tipo “Esto es seguro”. 

Quédate aquí. En este punto puedes tanto acceder a / como a /secured obviamente, y pulsar en enlace “login with twitter” te generará un 404.

Pero ahora estamos listos para empezar.

> **Nota:** Para este post vamos a usar la misma aplicación en twitter (que tenía el callback a /authentication/authenticatecallback. Aunque ahora la URL de callback puede ser la que queramos.

**Dejando que Katana hable con Twitter…**

Katana incorpora varios componentes de autenticación y hay uno que se encarga precisamente de gestionar el flujo oAuth con twitter. Este componente se llama <a href="http://www.nuget.org/packages/Microsoft.Owin.Security.Twitter/" target="_blank" rel="noopener noreferrer">Microsoft.Owin.Security.Twitter</a> así que añádelo al proyecto. Para entendernos es el equivalente al WorldDomination pero en un mundo OWIN.

Una vez hayas añadido este paquete el primer paso es modificar la clase de inicio de OWIN para añadir el módulo de autenticación por Twitter:

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:abbdc9c3-e055-4929-9e80-ed2ef7087536" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
    <div style="background: #ddd; max-height: 300px; overflow: auto">
      <ol start="1" style="background: #1e1e1e; margin: 0 0 0 2em; padding: 0 0 0 5px; white-space: nowrap">
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">app</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">UseTwitterAuthentication(</span><span style="background:#1e1e1e;color:#569cd6">new</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#4ec9b0">TwitterAuthenticationOptions</span><span style="background:#1e1e1e;color:#dcdcdc">()</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">{</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">    ConsumerKey </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#d69d85">"TU COMSUMER KEY"</span><span style="background:#1e1e1e;color:#dcdcdc">,</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">    ConsumerSecret </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#d69d85">"TU CONSUMER SECRET"</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">}</span><span style="background:#1e1e1e;color:#dcdcdc">);</span>
        </li>
      </ol>
    </div></p>
  </div></p>
</div>

Por supuesto pon el consumer key y consumer secret de tu aplicacion.

En este punto si pulsas el enlace de “login with twitter” recibirás… un 404 de Nancy. Pues este enlace apunta a /login/twitter (o a la URL que tu hayas elegido, en el fondo da igual) y es una URL que no está enrutada. A diferencia del post anterior donde WorldDomination ya gestionaba la URL “/authentication/redirect/twitter” el módulo de Katana no gestiona ninguna URL. En su lugar “entra en acción” tan buen punto se recibe un 401.

Así que nada, vamos a añadir un&#160; módulo que enrute la URL /login/twitter y devuelva un 401:

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:4317c9f8-941c-427e-9743-3fac16597a2a" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
    <div style="background: #ddd; max-height: 300px; overflow: auto">
      <ol start="1" style="background: #1e1e1e; margin: 0 0 0 2.5em; padding: 0 0 0 5px; white-space: nowrap">
        <li>
          <span style="background:#1e1e1e;color:#569cd6">public</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">class</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#4ec9b0">AuthTwitterModule</span><span style="background:#1e1e1e;color:#dcdcdc"> : </span><span style="background:#1e1e1e;color:#4ec9b0">NancyModule</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">{</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">    </span><span style="background:#1e1e1e;color:#569cd6">public</span><span style="background:#1e1e1e;color:#dcdcdc"> AuthTwitterModule()</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">    {</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">        Get[</span><span style="background:#1e1e1e;color:#d69d85">"/login/twitter"</span><span style="background:#1e1e1e;color:#dcdcdc">] </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> _ </span><span style="background:#1e1e1e;color:#b4b4b4">=></span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">new</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#4ec9b0">Response</span><span style="background:#1e1e1e;color:#dcdcdc">()</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">        {</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">            StatusCode </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#b8d7a3">HttpStatusCode</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">Unauthorized</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">        };</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">    }</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">}</span>
        </li>
      </ol>
    </div></p>
  </div></p>
</div>

Ahora si navegas a /login/twitter lo que recibirás es… bueno un 401 😛 La razón es porque aunque el módulo de Katana entra en acción cuando recibe un 401, no basta solo con el 401. Antes requiere que se rellene el entorno de OWIN con cierta información.

El entorno de OWIN es un diccionario de objetos, literalmente un IDictionary<string, object> que contiene **toda** la información del pipeline de OWIN. En OWIN no hay objetos tipo Request, Response o HttpContext porque eso implicaría que existe alguna DLL principal de OWIN y OWIN no pretende es
  
o: se basa en tipos de .NET (Hay una sola excepción a este caso y es la interfaz IAppBuilder que está definida en el <a href="http://www.nuget.org/packages/Owin/" target="_blank" rel="noopener noreferrer">paquete Owin</a>). Así los módulos OWIN se pasan información entre ellos a través de ese diccionario compartido.

Por lo tanto _antes_ de devolver el 401 debemos meter cierta información en el entorno de OWIN para que el módulo de autenticación de Katana sepa que queremos autenticarnos via Twitter. ¿Qué método hay en NancyFx para meter código _antes_ del código que procesa la petición? Exacto, el module hook Before. Pero en este caso no añadiremos el código directamente en el Before (podríamos) pero lo haremos más reutilizable a través de métodos de extensión (así seria aplicable a más de un módulo).

Pero primero necesitamos un método de extensión que me permita obtener el entorno de OWIN. El paquete Nancy.Owin (que es quien gestiona la integración de NancyFx en OWIN) deja el entorno OWIN dentro de la clave NancyOwinHost.RequestEnvironmentKey del contexto de NancyFx:

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:68fe5351-a292-452b-9176-4234750c522b" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
    <div style="background: #ddd; max-height: 300px; overflow: auto">
      <ol start="1" style="background: #1e1e1e; margin: 0 0 0 2.5em; padding: 0 0 0 5px; white-space: nowrap">
        <li>
          <span style="background:#1e1e1e;color:#569cd6">public</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">static</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">class</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#4ec9b0">NancyContextExtensions</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">{</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">    </span><span style="background:#1e1e1e;color:#569cd6">public</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">static</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#4ec9b0">OwinContext</span><span style="background:#1e1e1e;color:#dcdcdc"> GetOwinContext(</span><span style="background:#1e1e1e;color:#569cd6">this</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#4ec9b0">NancyContext</span><span style="background:#1e1e1e;color:#dcdcdc"> context)</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">    {</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">        </span><span style="background:#1e1e1e;color:#569cd6">var</span><span style="background:#1e1e1e;color:#dcdcdc"> environment </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> (</span><span style="background:#1e1e1e;color:#b8d7a3">IDictionary</span><span style="background:#1e1e1e;color:#b4b4b4"><</span><span style="background:#1e1e1e;color:#569cd6">string</span><span style="background:#1e1e1e;color:#dcdcdc">, </span><span style="background:#1e1e1e;color:#569cd6">object</span><span style="background:#1e1e1e;color:#b4b4b4">></span><span style="background:#1e1e1e;color:#dcdcdc">)context</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">Items[</span><span style="background:#1e1e1e;color:#4ec9b0">NancyOwinHost</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">RequestEnvironmentKey];</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">        </span><span style="background:#1e1e1e;color:#569cd6">var</span><span style="background:#1e1e1e;color:#dcdcdc"> owinContext </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">new</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#4ec9b0">OwinContext</span><span style="background:#1e1e1e;color:#dcdcdc">(environment);</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">        </span><span style="background:#1e1e1e;color:#569cd6">return</span><span style="background:#1e1e1e;color:#dcdcdc"> owinContext;</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">    }</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">}</span>
        </li>
      </ol>
    </div></p>
  </div></p>
</div>

A partir de la información del entorno se crea una variable de tipo OwinContext. La clase OwinContext **no es estandard OWIN**. Esta clase es una clase de Katana. **Así que para que no queden dudas: la integración que estamos haciendo es entre NancyFx y los componentes de Katana.** De hecho no es posible una integración universal porque la especificación de OWIN no define el nombre de las claves del entorno que los módulos deben usar, salvo unas cuantas (que podéis encontrar en la <a href="http://owin.org/spec/owin-1.0.0.html#_3.2._Environment" target="_blank" rel="noopener noreferrer">especificación de OWIN</a>). Así pues la clase OwinContext no es nada más que el entorno de OWIN, pero visto a través de algo más tipado que un IDictionary<string, object> y que además entiende las claves que usan los módulos de Katana.

Vale, ahora que ya tenemos como obtener el entorno OWIN, vamos a añadir otro método de extensión, pero ahora contra la clase NancyModule:

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:ced42d23-4c32-4223-8db9-56bf97bbc4eb" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
    <div style="background: #ddd; max-height: 300px; overflow: auto">
      <ol start="1" style="background: #1e1e1e; margin: 0 0 0 2.5em; padding: 0 0 0 5px; white-space: nowrap">
        <li>
          <span style="background:#1e1e1e;color:#569cd6">static</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">class</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#4ec9b0">NancyModuleExtensions</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">{</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">    </span><span style="background:#1e1e1e;color:#569cd6">public</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">static</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">void</span><span style="background:#1e1e1e;color:#dcdcdc"> Challenge(</span><span style="background:#1e1e1e;color:#569cd6">this</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#4ec9b0">NancyModule</span><span style="background:#1e1e1e;color:#dcdcdc"> module, </span><span style="background:#1e1e1e;color:#569cd6">string</span><span style="background:#1e1e1e;color:#dcdcdc"> redirectUri, </span><span style="background:#1e1e1e;color:#569cd6">string</span><span style="background:#1e1e1e;color:#dcdcdc"> userId)</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">    {</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">        module</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">AddBeforeHookOrExecute(ctx </span><span style="background:#1e1e1e;color:#b4b4b4">=></span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">        {</span>
        </li>
        <li>
          <span st
yle="background:#1e1e1e;color:#dcdcdc">            </span><span style="background:#1e1e1e;color:#569cd6">var</span><span style="background:#1e1e1e;color:#dcdcdc"> properties </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">new</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#4ec9b0">AuthenticationProperties</span><span style="background:#1e1e1e;color:#dcdcdc">() { RedirectUri </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> redirectUri };</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">            </span><span style="background:#1e1e1e;color:#569cd6">if</span><span style="background:#1e1e1e;color:#dcdcdc"> (userId </span><span style="background:#1e1e1e;color:#b4b4b4">!=</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">null</span><span style="background:#1e1e1e;color:#dcdcdc">)</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">            {</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">                properties</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">Dictionary[</span><span style="background:#1e1e1e;color:#d69d85">"XsrfId"</span><span style="background:#1e1e1e;color:#dcdcdc">] </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> userId;</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">            }</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">            module</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">Context</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">GetOwinContext()</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">Authentication</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">Challenge(properties, </span><span style="background:#1e1e1e;color:#d69d85">"Twitter"</span><span style="background:#1e1e1e;color:#dcdcdc">);</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">            </span><span style="background:#1e1e1e;color:#569cd6">return</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">null</span><span style="background:#1e1e1e;color:#dcdcdc">;</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">        }, </span><span style="background:#1e1e1e;color:#d69d85">"Challenge"</span><span style="background:#1e1e1e;color:#dcdcdc">);</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">    }</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">}</span>
        </li>
      </ol>
    </div></p>
  </div></p>
</div>

Lo que estamos haciendo es añadir código al module hook Before del módulo de NancyFx al que se llame este método (sería lo más parecido a crear un filtro en ASP.NET MVC que hay en Nancy). Básicamente lo que hacemos en el Before es llamar al método Challenge que proporciona Katana que es el que se encarga de todo lo necesario.

Ahora tenemos que modificar el AuthTwitterModule para añadir la llamada a este método de extensión:

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:505f1395-574d-4a40-82de-0124346dc24b" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
    <div style="background: #ddd; max-height: 300px; overflow: auto">
      <ol start="1" style="background: #1e1e1e; margin: 0 0 0 2em; padding: 0 0 0 5px; white-space: nowrap">
        <li>
          <span style="background:#1e1e1e;color:#569cd6">public</span><span style="background:#1e1e1e;color:#dcdcdc"> AuthTwitterModule()</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">{</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">    </span><span style="background:#1e1e1e;color:#569cd6">this</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">Challenge(</span><span style="background:#1e1e1e;color:#d69d85">"/auth/redirect"</span><span style="background:#1e1e1e;color:#dcdcdc">, </span><span style="background:#1e1e1e;color:#569cd6">null</span><span style="background:#1e1e1e;color:#dcdcdc">);</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">    Get[</span><span style="background:#1e1e1e;color:#d69d85">"/login/twitter"</span><span style="background:#1e1e1e;color:#dcdcdc">] </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> _ </span><span style="background:#1e1e1e;color:#b4b4b4">=></span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">new</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#4ec9b0">Response</span><span style="background:#1e1e1e;color:#dcdcdc">()</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">    {</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">        StatusCode </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#b8d7a3">HttpStatusCode</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">Unauthorized</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">    };</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">}</span>
        </li>
      </ol>
    </div></p>
  </div></p>
</div>

Ahora sí, si navegas a /login/twitter empezará el flujo de oAuth y después serás redirigido a la URL que hemos usado como primer parámetro de la llamada a Challenge, es decir /auth/redirect **con independencia del valor de callback que teníamos especificado en la aplicación de twitter**.

En esta URL de callback (/auth/redirect) tenemos que recoger los valores que nos haya devuelto el proveedor de oAuth. Para ello nos vamos a apoyar en otro componente de Katana, el paquete <a href="http://www.nuget.org/packages/Microsoft.AspNet.Identity.Owin/" target="_blank" rel="noopener noreferrer">Microsoft.AspNet.Identity.Owin</a>, así que añade este paquete ahora. Una vez lo hayas añadido podemos usar el método GetExternalLoginInfoAsync.

Este método es asíncrono, así que lo invocaremos con _await_. Para poder usar await en NancyFx tenemos que declarar que la ruta que responde a /auth/redirect es asíncrona:

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:2733c1f6-04d4-4c3f-afc5-05d6c5b4c9b3" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
    <div style="background: #ddd; overflow: auto">
      <ol start="1" style="background: #1e1e1e; margin: 0 0 0 2em; padding: 0 0 0 5px; white-space: nowrap">
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">Get[</span><span style="background:#1e1e1e;color:#d69d85">"/auth/redirect"</span><span style="background:#1e1e1e;color:#dcdcdc">, </span><span style="background:#1e1e1e;color:#569cd6">true</span><span style="background:#1e1e1e;color:#dcdcdc">] </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">async</span><span style="background:#1e1e1e;color:#dcdcdc"> (_, ct) </s pan><span style="background:#1e1e1e;color:#b4b4b4">=></span></li> 
          
          <li>
            <span style="background:#1e1e1e;color:#dcdcdc">{</span>
          </li>
          <li>
            <span style="background:#1e1e1e;color:#dcdcdc">    </span><span style="background:#1e1e1e;color:#569cd6">var</span><span style="background:#1e1e1e;color:#dcdcdc"> authManager </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> Context</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">GetOwinContext()</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">Authentication;</span>
          </li>
          <li>
            <span style="background:#1e1e1e;color:#dcdcdc">    </span><span style="background:#1e1e1e;color:#569cd6">var</span><span style="background:#1e1e1e;color:#dcdcdc"> loginInfo </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">await</span><span style="background:#1e1e1e;color:#dcdcdc"> authManager</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">GetExternalLoginInfoAsync();</span>
          </li>
          <li>
            <span style="background:#1e1e1e;color:#dcdcdc">    </span><span style="background:#1e1e1e;color:#57a64a">// ...</span>
          </li>
          <li>
            <span style="background:#1e1e1e;color:#dcdcdc">}</span><span style="background:#1e1e1e;color:#dcdcdc">;</span>
          </li></ol></div> </p></div> </p></div> 
          
          <p>
            Este método se encarga de obtener los datos que nos envía el proveedor de oAuth. En este punto podemos recoger los datos del usuario y crear un ClaimsIdentity (esa clase es la nueva clase base de todas las identity en .NET). Lo más normal sería delegar en el Identity Membership para esto, pero, para no saturar, hagámoslo a mano. En el siguiente post veremos como integrarnos con&#160; el Identity Membership. El código podría ser algo como así:
          </p>
          
          <div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:6e0adf70-ce21-46e4-99be-2a666ce2fb1f" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
            <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
              <div style="background: #ddd; max-height: 300px; overflow: auto">
                <ol start="1" style="background: #1e1e1e; margin: 0 0 0 2.5em; padding: 0 0 0 5px; white-space: nowrap">
                  <li>
                    <span style="background:#1e1e1e;color:#dcdcdc">Get[</span><span style="background:#1e1e1e;color:#d69d85">"/auth/redirect"</span><span style="background:#1e1e1e;color:#dcdcdc">, </span><span style="background:#1e1e1e;color:#569cd6">true</span><span style="background:#1e1e1e;color:#dcdcdc">] </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">async</span><span style="background:#1e1e1e;color:#dcdcdc"> (_, ct) </span><span style="background:#1e1e1e;color:#b4b4b4">=></span>
                  </li>
                  <li>
                    <span style="background:#1e1e1e;color:#dcdcdc">{</span>
                  </li>
                  <li>
                    <span style="background:#1e1e1e;color:#dcdcdc">    </span><span style="background:#1e1e1e;color:#569cd6">var</span><span style="background:#1e1e1e;color:#dcdcdc"> authManager </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> Context</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">GetOwinContext()</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">Authentication;</span>
                  </li>
                  <li>
                    <span style="background:#1e1e1e;color:#dcdcdc">    </span><span style="background:#1e1e1e;color:#569cd6">var</span><span style="background:#1e1e1e;color:#dcdcdc"> loginInfo </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">await</span><span style="background:#1e1e1e;color:#dcdcdc"> authManager</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">GetExternalLoginInfoAsync();</span>
                  </li>
                  <li>
                    <span style="background:#1e1e1e;color:#dcdcdc">    </span><span style="background:#1e1e1e;color:#569cd6">if</span><span style="background:#1e1e1e;color:#dcdcdc"> (loginInfo </span><span style="background:#1e1e1e;color:#b4b4b4">!=</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">null</span><span style="background:#1e1e1e;color:#dcdcdc">)</span>
                  </li>
                  <li>
                    <span style="background:#1e1e1e;color:#dcdcdc">    {</span>
                  </li>
                  <li>
                    <span style="background:#1e1e1e;color:#dcdcdc">        authManager</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">SignOut(</span><span style="background:#1e1e1e;color:#4ec9b0">DefaultAuthenticationTypes</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">ExternalCookie);</span>
                  </li>
                  <li>
                    &nbsp;
                  </li>
                  <li>
                    <span style="background:#1e1e1e;color:#dcdcdc">        </span><span style="background:#1e1e1e;color:#569cd6">var</span><span style="background:#1e1e1e;color:#dcdcdc"> identity </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">new</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#4ec9b0">ClaimsIdentity</span><span style="background:#1e1e1e;color:#dcdcdc">(</span><span style="background:#1e1e1e;color:#4ec9b0">DefaultAuthenticationTypes</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">ApplicationCookie</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">ToString(),</span>
                  </li>
                  <li>
                    <span style="background:#1e1e1e;color:#dcdcdc">            </span><span style="background:#1e1e1e;color:#d69d85">"http://schemas.xmlsoap.org/ws/2005/05/identity/claims/nameidentifier"</span><span style="background:#1e1e1e;color:#dcdcdc">,</span>
                  </li>
                  <li>
                    <span style="background:#1e1e1e;color:#dcdcdc">            </span><span style="background:#1e1e1e;color:#d69d85">"http://schemas.microsoft.com/ws/2008/06/identity/claims/role"</span><span style="background:#1e1e1e;color:#dcdcdc">);</span>
                  </li>
                  <li>
                    <span style="background:#1e1e1e;color:#dcdcdc">        identity</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">AddClaim(</span><span style="background:#1e1e1e;color:#569cd6">new</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#4ec9b0">Claim</span><span style="background:#1e1e1e;color:#dcdcdc">(</span><span style="background:#1e1e1e;color:#d69d85">"Player"</span><span style="background:#1e1e1e;color:#dcdcdc">, </span><span style="background:#1e1e1e;color:#d69d85">"True"</span><span style="background:#1e1e1e;color:#dcdcdc">));</span>
                  </li>
                  <li>
                    <span style="background:#1e1e1e;color:#dcdcdc">        identity</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">AddClaim(</span><span style="background:#1e1e1e;color:#569cd6">new</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#4ec9b0">Claim</span><span style="background:#1e1e1e;color:#dcdcdc">(</span><span style="background:#1e1e1e;color:#d69d85">"http://schemas.xmlsoap.org/ws/2005/05/identity/claims/nameidentifier"</span><span style="background:#1e1e1e;color:#dcdcdc">,</span>
                  </li>
                  <li>
                    <span style="background:#1e1e1e;color:#dcdcdc">            loginInfo</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc"><br /> DefaultUserName,</span>
                  </li>
                  <li>
                    <span style="background:#1e1e1e;color:#dcdcdc">            </span><span style="background:#1e1e1e;color:#d69d85">"http://www.w3.org/2001/XMLSchema#string"</span><span style="background:#1e1e1e;color:#dcdcdc">));</span>
                  </li>
                  <li>
                    <span style="background:#1e1e1e;color:#dcdcdc">        authManager</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">SignIn(</span><span style="background:#1e1e1e;color:#569cd6">new</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#4ec9b0">AuthenticationProperties</span><span style="background:#1e1e1e;color:#dcdcdc">() { IsPersistent </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">false</span><span style="background:#1e1e1e;color:#dcdcdc"> }, identity);</span>
                  </li>
                  <li>
                    <span style="background:#1e1e1e;color:#dcdcdc">    }</span>
                  </li>
                  <li>
                    &nbsp;
                  </li>
                  <li>
                    <span style="background:#1e1e1e;color:#dcdcdc">    </span><span style="background:#1e1e1e;color:#569cd6">return</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">new</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#4ec9b0">RedirectResponse</span><span style="background:#1e1e1e;color:#dcdcdc">(</span><span style="background:#1e1e1e;color:#d69d85">"/secured"</span><span style="background:#1e1e1e;color:#dcdcdc">);</span>
                  </li>
                  <li>
                    <span style="background:#1e1e1e;color:#dcdcdc">};</span>
                  </li>
                </ol>
              </div></p>
            </div></p>
          </div>
          
          <p>
            No te preocupes si no entiendes exactamente el código (como digo eso suele delegarse en el Identity Membership), pero básicamente creamos el ClaimsIdentity y le añadimos un nombre de usuario, así como una claim personalizada (Player con valor True).
          </p>
          
          <p>
            Al final redirigimos al usuario a la URL /secured, una URL que se supone solo debe poder verse si el usuario no está autenticado.
          </p>
          
          <p>
            El código del módulo Nancy que contiene la ruta para dicha URL es el siguiente:
          </p>
          
          <div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:db1da936-e607-4415-922c-31db5f47900a" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
            <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
              <div style="background: #ddd; max-height: 300px; overflow: auto">
                <ol start="1" style="background: #1e1e1e; margin: 0 0 0 2em; padding: 0 0 0 5px; white-space: nowrap">
                  <li>
                    <span style="background:#1e1e1e;color:#569cd6">public</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">class</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#4ec9b0">SecuredModule</span><span style="background:#1e1e1e;color:#dcdcdc"> : </span><span style="background:#1e1e1e;color:#4ec9b0">NancyModule</span>
                  </li>
                  <li>
                    <span style="background:#1e1e1e;color:#dcdcdc">{</span>
                  </li>
                  <li>
                    <span style="background:#1e1e1e;color:#dcdcdc">    </span><span style="background:#1e1e1e;color:#569cd6">public</span><span style="background:#1e1e1e;color:#dcdcdc"> SecuredModule()</span>
                  </li>
                  <li>
                    <span style="background:#1e1e1e;color:#dcdcdc">    {</span>
                  </li>
                  <li>
                    <span style="background:#1e1e1e;color:#dcdcdc">        </span><span style="background:#1e1e1e;color:#569cd6">this</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">RequiresOwinAuth();</span>
                  </li>
                  <li>
                    <span style="background:#1e1e1e;color:#dcdcdc">        Get[</span><span style="background:#1e1e1e;color:#d69d85">"/secured"</span><span style="background:#1e1e1e;color:#dcdcdc">] </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> _ </span><span style="background:#1e1e1e;color:#b4b4b4">=></span><span style="background:#1e1e1e;color:#dcdcdc"> View[</span><span style="background:#1e1e1e;color:#d69d85">"secured.html"</span><span style="background:#1e1e1e;color:#dcdcdc">];</span>
                  </li>
                  <li>
                    <span style="background:#1e1e1e;color:#dcdcdc">    }</span>
                  </li>
                  <li>
                    <span style="background:#1e1e1e;color:#dcdcdc">}</span>
                  </li>
                </ol>
              </div></p>
            </div></p>
          </div>
          
          <p>
            El método RequiresOwinAuth es un método de extensión que nos hemos creado. Dicho método comprueba que existe una ClaimsIdentity en el contexto OWIN:
          </p>
          
          <div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:3380e0d3-1700-4696-be8a-de9f875ba4ee" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
            <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
              <div style="background: #ddd; max-height: 300px; overflow: auto">
                <ol start="1" style="background: #1e1e1e; margin: 0 0 0 2.5em; padding: 0 0 0 5px; white-space: nowrap">
                  <li>
                    <span style="background:#1e1e1e;color:#569cd6">public</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">static</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">void</span><span style="background:#1e1e1e;color:#dcdcdc"> RequiresOwinAuth(</span><span style="background:#1e1e1e;color:#569cd6">this</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#4ec9b0">NancyModule</span><span style="background:#1e1e1e;color:#dcdcdc"> module)</span>
                  </li>
                  <li>
                    <span style="background:#1e1e1e;color:#dcdcdc">{</span>
                  </li>
                  <li>
                    <span style="background:#1e1e1e;color:#dcdcdc">    module</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">AddBeforeHookOrExecute(ctx </span><span style="background:#1e1e1e;color:#b4b4b4">=></span>
                  </li>
                  <li>
                    <span style="background:#1e1e1e;color:#dcdcdc">    {</span>
                  </li>
                  <li>
                    <span style="background:#1e1e1e;color:#dcdcdc">        </span><span style="background:#1e1e1e;color:#569cd6">var</span><span style="background:#1e1e1e;color:#dcdcdc"> user </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> ctx</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">GetOwinUser();</span>
                  </li>
                  <li>
                    <span style="background:#1e1e1e;color:#dcdcdc">        </span><span style="background:#1e1e1e;color:#569cd6">return</span><span style="background:#1e1e1e;color:#dcdcdc"> user </span><span style="background:#1e1e1e;color:#b4b4b4">==</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">null</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#b4b4b4">||</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#b4b4b4">!</span><span style="background:#1e1e1e;color:#dcdcdc">user</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">Identity</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">IsAuthenticated </span><span style="background:#1e1e1e;color:#b4b4b4">?</span><span style="background:#1e1e1e;color:#dcdcdc"> </span>
                  </li>
                  <li>
                    <span style="background:#1e1e1e;color:#dcdcdc">            </span><span style="background:#1e1e1e;color:#569cd6">new</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#4ec9b0">Response</span><span style="background:#1e1e1e;color:#dcdcdc">() {StatusCode </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#b8d7a3">HttpStatusCode</ span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">Unauthorized} : </span></li> 
                    
                    <li>
                      <span style="background:#1e1e1e;color:#dcdcdc">            </span><span style="background:#1e1e1e;color:#569cd6">null</span><span style="background:#1e1e1e;color:#dcdcdc">;</span>
                    </li>
                    <li>
                      <span style="background:#1e1e1e;color:#dcdcdc">    }, </span><span style="background:#1e1e1e;color:#d69d85">"OwinUser Not Found"</span><span style="background:#1e1e1e;color:#dcdcdc">);</span>
                    </li>
                    <li>
                      <span style="background:#1e1e1e;color:#dcdcdc">}</span>
                    </li></ol></div> </p></div> </p></div> 
                    
                    <p>
                      (Este método de extensión sería el equivalente a aplicar [Authorize] en un controlador ASP.NET MVC).
                    </p>
                    
                    <p>
                      Vale… ya casi lo tenemos, ahora tan solo nos falta configurar el pipeline OWIN para añadir la seguridad por cookies:
                    </p>
                    
                    <div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:75fdbab2-c54f-4879-8128-cbc4fa20fc3b" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
                      <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
                        <div style="background: #ddd; max-height: 300px; overflow: auto">
                          <ol start="1" style="background: #1e1e1e; margin: 0 0 0 2em; padding: 0 0 0 5px; white-space: nowrap">
                            <li>
                              <span style="background:#1e1e1e;color:#dcdcdc">app</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">UseCookieAuthentication(</span><span style="background:#1e1e1e;color:#569cd6">new</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#4ec9b0">CookieAuthenticationOptions</span><span style="background:#1e1e1e;color:#dcdcdc">()</span>
                            </li>
                            <li>
                              <span style="background:#1e1e1e;color:#dcdcdc">{</span>
                            </li>
                            <li>
                              <span style="background:#1e1e1e;color:#dcdcdc">    AuthenticationType </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#4ec9b0">DefaultAuthenticationTypes</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">ApplicationCookie</span>
                            </li>
                            <li>
                              <span style="background:#1e1e1e;color:#dcdcdc">});</span>
                            </li>
                          </ol>
                        </div></p>
                      </div></p>
                    </div>
                    
                    <p>
                      Añadimos este código al principio del método de inicialización de OWIN, para que este módulo esté en el principio del pipeline.
                    </p>
                    
                    <p>
                      Y ¡voilá! hemos terminado. Si nada más iniciar la aplicación navegas a /secured recibirás un 401 (Unauthorized). Si entras en twitter, después de hacer el login verás como se te redirige a /secured y ahora si ves el contenido seguro.
                    </p>
                    
                    <p>
                      Por supuesto, puedes hacer que en lugar de ver un 401 el usuario sea redirigido a una página de Login, simplemente cambiando la configuración del proveedor de seguridad por cookies:
                    </p>
                    
                    <div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:5a7f19b4-b4f1-4a18-91d9-3f5d4deb04e3" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
                      <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
                        <div style="background: #ddd; max-height: 300px; overflow: auto">
                          <ol start="1" style="background: #1e1e1e; margin: 0 0 0 2em; padding: 0 0 0 5px; white-space: nowrap">
                            <li>
                              <span style="background:#1e1e1e;color:#dcdcdc">app</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">UseCookieAuthentication(</span><span style="background:#1e1e1e;color:#569cd6">new</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#4ec9b0">CookieAuthenticationOptions</span><span style="background:#1e1e1e;color:#dcdcdc">()</span>
                            </li>
                            <li>
                              <span style="background:#1e1e1e;color:#dcdcdc">{</span>
                            </li>
                            <li>
                              <span style="background:#1e1e1e;color:#dcdcdc">    AuthenticationType </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#4ec9b0">DefaultAuthenticationTypes</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">ApplicationCookie,</span>
                            </li>
                            <li>
                              <span style="background:#1e1e1e;color:#dcdcdc">    LoginPath </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">new</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#4ec9b0">PathString</span><span style="background:#1e1e1e;color:#dcdcdc">(</span><span style="background:#1e1e1e;color:#d69d85">"/login"</span><span style="background:#1e1e1e;color:#dcdcdc">)</span>
                            </li>
                            <li>
                              <span style="background:#1e1e1e;color:#dcdcdc">});</span>
                            </li>
                          </ol>
                        </div></p>
                      </div></p>
                    </div>
                    
                    <p>
                      Ahora si nada más empezar navegas a /secured verás que el usuario es redirigido a /login (en este caso verás un 404 ya que no hay ninguna ruta que responda a la URL /login).
                    </p>
                    
                    <p>
                      Bueno… en el post anterior vimos como configurar NancyFx junto con WorldDomination para soportar login por oAuth. En este post hemos ido un paso más allá sustituyendo toda la autenticación por componentes OWIN, en lugar de que toda la responsabilidad esté gestionada por NancyFx.
                    </p>
                    
                    <p>
                      Un saludo!
                    </p>
                    
                    <p>
                      PD: <a href="http://sdrv.ms/IoN4zo" target="_blank" rel="noopener noreferrer">Tenéis el código en mi carpeta de SkyDrive</a> (fichero NancyKatanaIIS2).
                    </p>