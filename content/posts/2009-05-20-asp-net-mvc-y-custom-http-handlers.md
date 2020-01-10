---
title: ASP.NET MVC y Custom Http Handlers

author: eiximenis

date: 2009-05-20T14:57:01+00:00
geeks_url: /?p=1451
geeks_visits:
  - 1772
geeks_ms_views:
  - 994
categories:
  - Uncategorized

---
Hola! ¿Como va todo?

Un post rapidito… Imaginad que creais un Custom Handler para procesar determinadas peticiones en vuestra aplicación ASP.NET MVC:

<!--more-->

<pre><pre style="background-color: #fbfbfb; margin: 0em; width: 100%; font-family: consolas,&#39;Courier New&#39;,courier,monospace; font-size: 12px">    <span style="color: #0000ff">public</span> <span style="color: #0000ff">class</span> CardImagesHandler : IHttpHandler
</pre>


<pre style="background-color: #ffffff; margin: 0em; width: 100%; font-family: consolas,&#39;Courier New&#39;,courier,monospace; font-size: 12px">    {
</pre>


<pre style="background-color: #fbfbfb; margin: 0em; width: 100%; font-family: consolas,&#39;Courier New&#39;,courier,monospace; font-size: 12px">        #region IHttpHandler Members
</pre>


<pre style="background-color: #ffffff; margin: 0em; width: 100%; font-family: consolas,&#39;Courier New&#39;,courier,monospace; font-size: 12px">        <span style="color: #0000ff">public</span> <span style="color: #0000ff">bool</span> IsReusable
</pre>


<pre style="background-color: #fbfbfb; margin: 0em; width: 100%; font-family: consolas,&#39;Courier New&#39;,courier,monospace; font-size: 12px">        {
</pre>


<pre style="background-color: #ffffff; margin: 0em; width: 100%; font-family: consolas,&#39;Courier New&#39;,courier,monospace; font-size: 12px">            <span style="color: #0000ff">get</span> { <span style="color: #0000ff">return</span> <span style="color: #0000ff">false</span>; }
</pre>


<pre style="background-color: #fbfbfb; margin: 0em; width: 100%; font-family: consolas,&#39;Courier New&#39;,courier,monospace; font-size: 12px">        }
</pre>


<pre style="background-color: #ffffff; margin: 0em; width: 100%; font-family: consolas,&#39;Courier New&#39;,courier,monospace; font-size: 12px">        <span style="color: #0000ff">public</span> <span style="color: #0000ff">void</span> ProcessRequest(HttpContext context)
</pre>


<pre style="background-color: #fbfbfb; margin: 0em; width: 100%; font-family: consolas,&#39;Courier New&#39;,courier,monospace; font-size: 12px">        {
</pre>


<pre style="background-color: #ffffff; margin: 0em; width: 100%; font-family: consolas,&#39;Courier New&#39;,courier,monospace; font-size: 12px">            <span style="color: #008000">// ...</span>
</pre>


<pre style="background-color: #fbfbfb; margin: 0em; width: 100%; font-family: consolas,&#39;Courier New&#39;,courier,monospace; font-size: 12px">        }
</pre>


<pre style="background-color: #ffffff; margin: 0em; width: 100%; font-family: consolas,&#39;Courier New&#39;,courier,monospace; font-size: 12px">        #endregion
</pre>


<pre style="background-color: #fbfbfb; margin: 0em; width: 100%; font-family: consolas,&#39;Courier New&#39;,courier,monospace; font-size: 12px">    }</pre>


<p>
  Luego, lo añadís en el web.config, para que os procese todas las URLs con extensión .portrait:
</p>


<pre><pre style="background-color: #fbfbfb; margin: 0em; width: 100%; font-family: consolas,&#39;Courier New&#39;,courier,monospace; font-size: 12px"><span style="color: #0000ff">&lt;</span><span style="color: #800000">add</span> <span style="color: #ff0000">verb</span>=<span style="color: #0000ff">"GET"</span> <span style="color: #ff0000">path</span>=<span style="color: #0000ff">"*.portrait"</span> </pre>


<pre style="background-color: #ffffff; margin: 0em; width: 100%; font-family: consolas,&#39;Courier New&#39;,courier,monospace; font-size: 12px"><span style="color: #ff0000">type</span>=<span style="color: #0000ff">"WofWeb.Code.CardImagesHandler"</span><span style="color: #0000ff">/&gt;</span></pre>


<p>
  Finalmente probáis alguna URL con extensión portrait, tal y como http://localhost:8080/prueba.portrait i veis que vuestro Custom Http Handler es vilmente ignorado…
</p>


<p>
  ¿Por qué? Pues porque ASP.NET MVC toma control de todas las URLs, por lo que intentará procesar esta URL aunque hayamos definido un Custom Http Handler para ella. ¿La solución? En la tabla de enrutamiento indicar que queremos que se <em>ignoren</em> las URL con .potrait, de esta manera serán procesadas por nuestro Custom Http Handler…
</p>


<p>
  … tan sencillo como añadir:
</p>


<pre><pre style="background-color: #fbfbfb; margin: 0em; width: 100%; font-family: consolas,&#39;Courier New&#39;,courier,monospace; font-size: 12px">routes.IgnoreRoute("{resource}.portrait/{*pathInfo}");</pre>


<p>
  en global.asax donde configuremos la tabla de enrutamiento!
</p>


<p>
  Saludos!
</p>