---
title: ASP.NET vNext–Model Binding
author: eiximenis

date: 2014-07-14T14:02:20+00:00
geeks_url: /?p=1676
geeks_visits:
  - 1654
geeks_ms_views:
  - 1733
categories:
  - Uncategorized

---
Bien, en el post anterior comentamos cuatro cosillas sobre el model binding en ASP.NET MVC y WebApi, sus semejanzas y sus diferencias. En ASP.NET vNext ambos frameworks se unifican así que es de esperar que el model binding también lo haga… Veamos como funciona el model binding de vNext.

> **Nota:** Este post está realizado con la versión de ASP.NET vNext que viene con el VS14 CTP2. La mejor manera de probar dicha CTP es usando una VM en Azure creada a partir de una plantilla que ya la contiene instalada. Por supuesto todo lo dicho aquí puede contener cambios en la versión final 🙂

**Pruebas de caja negra**

Antes que nada he intentado hacer unas pruebas de “caja negra” para ver si el comportamiento era más parecido al de WebApi o al de MVC. He empezado con un proyecto web vNext vacío, y en el project.json he agregado la referencia a Microsoft.AspNet.Mvc. Luego me he creado un controlador como el siguiente:

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:a8828cd4-0b28-41af-abee-076eb75386e4" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
    <div style="background: #ddd; max-height: 300px; overflow: auto">
      <ol start="1" style="background: #1d1d1d; margin: 0 0 0 2.5em; padding: 0 0 0 5px;">
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">public</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">class</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#4ec9b0">HomeController</span><span style="background:#1e1e1e;color:#dcdcdc"> : </span><span style="background:#1e1e1e;color:#4ec9b0">Controller</span>
        </li>
        <li style="background: #111111">
          <span style="background:#1e1e1e;color:#dcdcdc">{</span>
        </li>
        <li>
              <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">public</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#b8d7a3">IActionResult</span><span style="background:#1e1e1e;color:#dcdcdc"> Index(</span><span style="background:#1e1e1e;color:#4ec9b0">Product</span><span style="background:#1e1e1e;color:#dcdcdc"> product, </span><span style="background:#1e1e1e;color:#4ec9b0">Customer</span><span style="background:#1e1e1e;color:#dcdcdc"> customer)</span>
        </li>
        <li style="background: #111111">
              <span style="background:#1e1e1e;color:#dcdcdc">{</span>
        </li>
        <li>
                  <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">return</span><span style="background:#1e1e1e;color:#dcdcdc"> View();</span>
        </li>
        <li style="background: #111111">
              <span style="background:#1e1e1e;color:#dcdcdc">}</span>
        </li>
        <li>
          &nbsp;
        </li>
        <li style="background: #111111">
              <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">public</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">bool</span><span style="background:#1e1e1e;color:#dcdcdc"> Post(</span><span style="background:#1e1e1e;color:#4ec9b0">Product</span><span style="background:#1e1e1e;color:#dcdcdc"> product, </span><span style="background:#1e1e1e;color:#4ec9b0">Customer</span><span style="background:#1e1e1e;color:#dcdcdc"> customer)</span>
        </li>
        <li>
              <span style="background:#1e1e1e;color:#dcdcdc">{</span>
        </li>
        <li style="background: #111111">
                  <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">return</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">true</span><span style="background:#1e1e1e;color:#dcdcdc">;</span>
        </li>
        <li>
              <span style="background:#1e1e1e;color:#dcdcdc">}</span>
        </li>
        <li style="background: #111111">
          <span style="background:#1e1e1e;color:#dcdcdc">}</span>
        </li>
      </ol>
    </div></p>
  </div></p>
</div>

Finalmente en el Startup.cs he configurado una tabla de rutas que combine MVC y WebApi:

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:c1f8809e-da45-427d-b47c-0d01a96a31c8" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
    <div style="background: #ddd; max-height: 300px; overflow: auto">
      <ol start="1" style="background: #1d1d1d; margin: 0 0 0 2.5em; padding: 0 0 0 5px;">
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">public</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">class</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#4ec9b0">Startup</span>
        </li>
        <li style="background: #111111">
          <span style="background:#1e1e1e;color:#dcdcdc">{</span>
        </li>
        <li>
              <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">public</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">void</span><span style="background:#1e1e1e;color:#dcdcdc"> Configure(</span><span style="background:#1e1e1e;color:#b8d7a3">IBuilder</span><span style="background:#1e1e1e;color:#dcdcdc"> app)</span>
        </li>
        <li style="background: #111111">
              <span style="background:#1e1e1e;color:#dcdcdc">{</span>
        </li>
        <li>
                  <span style="background:#1e1e1e;color:#dcdcdc">app</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">UseServices(s </span><span style="background:#1e1e1e;color:#b4b4b4">=></span><span style="background:#1e1e1e;color:#dcdcdc"> s</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">AddMvc());</span>
        </li>
        <li style="background: #111111">
          &nbsp;
        </li>
        <li>
                  <span style="background:#1e1e1e;color:#dcdcdc">app</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">UseMvc(r </span><span style="background:#1e1e1e;color:#b4b4b4">=></span>
        </li>
        <li style="background: #111111">
                  <span style="background:#1e1e1e;color:#dcdcdc">{</span>
        </li>
        <li>
                      <span style="background:#1e1e1e;color:#dcdcdc">r</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">MapRoute(</span>
        </li>
        <li style="background: #111111">
                          <span style="background:#1e1e1e;color:#dcdcdc">name: </span><span style="background:#1e1e1e;color:#d69d85">"default"</span><span style="background:#1e1e1e;color:#dcdcdc">,</span>
        </li>
        <li>
                          <span style="background:#1e1e1e;color:#dcdcdc">template: </span><span style="background:#1e1e1e;color:#d69d85">"{controller}/{action}/{id?}"</span><span style="background:#1e1e1e;color:#dcdcdc">,</span>
        </li>
        <li style="background: #111111">
                          <span style="background:#1e1e1e;color:#dcdcdc">defaults: </span><span style="background:#1e1e1e;color:#569cd6">new</span><span style="background:#1e1e1e;color:#dcdcdc"> { controller </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#d69d85">"Home"</span><span style="background:#1e1e1e;color:#dcdcdc">, action </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#d69d85">"Index"</span><span style="background:
#1e1e1e;color:#dcdcdc"> });</span>
        </li>
        <li>
                      <span style="background:#1e1e1e;color:#dcdcdc">r</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">MapRoute(</span>
        </li>
        <li style="background: #111111">
                          <span style="background:#1e1e1e;color:#dcdcdc">name: </span><span style="background:#1e1e1e;color:#d69d85">"second"</span><span style="background:#1e1e1e;color:#dcdcdc">,</span>
        </li>
        <li>
                          <span style="background:#1e1e1e;color:#dcdcdc">template: </span><span style="background:#1e1e1e;color:#d69d85">"api/{Controller}/{id?}"</span>
        </li>
        <li style="background: #111111">
                          <span style="background:#1e1e1e;color:#dcdcdc">);</span>
        </li>
        <li>
                  <span style="background:#1e1e1e;color:#dcdcdc">});</span>
        </li>
        <li style="background: #111111">
              <span style="background:#1e1e1e;color:#dcdcdc">}</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">}</span>
        </li>
      </ol>
    </div></p>
  </div></p>
</div>

Con esa tabla de rutas un POST a /Home/Index debe enrutarme por la primera acción del controlador (al igual que un GET). Mientras que un POST a /api/Home debe enrutarme por la segunda acción del controlador (mientras que un GET a /api/Home debe devolverme un 404). <a href="http://geeks.ms/blogs/etomas/archive/2014/06/25/explorando-asp-net-vnext-routing.aspx" target="_blank" rel="noopener noreferrer">Para más información echa un vistazo a mi post sobre el routing en vNext</a>.

Las clases Customer y Product contienen simplemente propiedades:

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:01b2f6e3-e122-424c-9282-df64badbeae8" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
    <div style="background: #ddd; max-height: 300px; overflow: auto">
      <ol start="1" style="background: #1d1d1d; margin: 0 0 0 2em; padding: 0 0 0 5px;">
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">public</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">class</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#4ec9b0">Customer</span>
        </li>
        <li style="background: #111111">
          <span style="background:#1e1e1e;color:#dcdcdc">{</span>
        </li>
        <li>
              <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">public</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">int</span><span style="background:#1e1e1e;color:#dcdcdc"> Id { </span><span style="background:#1e1e1e;color:#569cd6">get</span><span style="background:#1e1e1e;color:#dcdcdc">; </span><span style="background:#1e1e1e;color:#569cd6">set</span><span style="background:#1e1e1e;color:#dcdcdc">; }</span>
        </li>
        <li style="background: #111111">
              <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">public</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">string</span><span style="background:#1e1e1e;color:#dcdcdc"> Name { </span><span style="background:#1e1e1e;color:#569cd6">get</span><span style="background:#1e1e1e;color:#dcdcdc">; </span><span style="background:#1e1e1e;color:#569cd6">set</span><span style="background:#1e1e1e;color:#dcdcdc">; }</span>
        </li>
        <li>
              <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">public</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">string</span><span style="background:#1e1e1e;color:#dcdcdc"> Gender { </span><span style="background:#1e1e1e;color:#569cd6">get</span><span style="background:#1e1e1e;color:#dcdcdc">; </span><span style="background:#1e1e1e;color:#569cd6">set</span><span style="background:#1e1e1e;color:#dcdcdc">; }</span>
        </li>
        <li style="background: #111111">
          <span style="background:#1e1e1e;color:#dcdcdc">}</span>
        </li>
      </ol>
    </div></p>
  </div></p>
</div>

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:28de7b67-fed7-4360-87ec-7c35740870c0" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
    <div style="background: #ddd; max-height: 300px; overflow: auto">
      <ol start="1" style="background: #1d1d1d; margin: 0 0 0 2em; padding: 0 0 0 5px;">
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">public</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">class</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#4ec9b0">Product</span>
        </li>
        <li style="background: #111111">
          <span style="background:#1e1e1e;color:#dcdcdc">{</span>
        </li>
        <li>
              <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">public</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">int</span><span style="background:#1e1e1e;color:#dcdcdc"> Id { </span><span style="background:#1e1e1e;color:#569cd6">get</span><span style="background:#1e1e1e;color:#dcdcdc">; </span><span style="background:#1e1e1e;color:#569cd6">set</span><span style="background:#1e1e1e;color:#dcdcdc">; }</span>
        </li>
        <li style="background: #111111">
              <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">public</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">string</span><span style="background:#1e1e1e;color:#dcdcdc"> Name { </span><span style="background:#1e1e1e;color:#569cd6">get</span><span style="background:#1e1e1e;color:#dcdcdc">; </span><span style="background:#1e1e1e;color:#569cd6">set</span><span style="background:#1e1e1e;color:#dcdcdc">; }</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">}</span>
        </li>
      </ol>
    </div></p>
  </div></p>
</div>

Luego he usado <a href="http://curl.haxx.se/" target="_blank" rel="noopener noreferrer">cURL</a> para realizar unos posts y ver que es lo que tenía:

curl &#8211;data "Id=1&Name=eiximenis&Gender=Male" <http://localhost:49228/>&#160; &#8211;header "Content-type:application/x-www-form-urlencoded"

Con esto simulo un post a que contenga los datos Id, Name y Gender y eso es lo que recibo en el controlador (en el método Index):

[<img title="image" style="border-left-width: 0px; border-right-width: 0px; background-image: none; border-bottom-width: 0px; padding-top: 0px; padding-left: 0px; display: inline; padding-right: 0px; border-top-width: 0px" border="0" alt="image" src="http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_476C218D.png" width="393" height="140" />][1]

Este comportamiento es el mismo que en ASP.NET MVC. Ahora cambio la petición de cURL para enviar la misma petición pero a /api/Home para que se me enrute al método Post (estilo WebApi). Mi idea era ver si para enrutamiento tipo MVC se usaba un binding parecido a MVC y para enrutamiento tipo WebApi (sin acción y basado en verbo HTTP) se usaba un binding parecido al de WebApi:

curl –data "Id=1&Name=eiximenis&Gender=Male" <http://localhost:49228/api/Home>&#160; &#8211;header "Content-type:application/x-www-form-urlencoded"

El resultado es que se me llama al m
  
étodo Post del controlador **pero recibo exactamente los mismos valores que antes**. Recordad que en WebApi eso NO era así. Así a simple vista parece que se ha elegido el modelo de model binding de ASP.NET MVC antes que el de web api.

Otra prueba ha sido realizar un POST contra /api/Home/10 (el parámetro 10 se corresponde al route value id) y dado que estamos pasando el id por URL quitarlo del cuerpo de la petición:

curl &#8211;data "Name=eiximenis&Gender=Male" <http://localhost:49228/api/Home/10>&#160; &#8211;header "Content-type:application/x-www-form-urlencoded"

El resultado es el mismo que en el caso anterior (y coincide con ASP.NET MVC donde el model binder ni se preocupa de _donde_ vienen los datos).

Por lo tanto estas pruebas **parecen sugerir que en vNext el model binding que se sigue es el de ASP.NET MVC**.

Claro que cuando uno pruebas de caja negra debe tener presente el máximo número de opciones… Porque resulta que si hago algo parecido a:

curl –data "{&#8216;Name&#8217;:&#8217;eiximenis&#8217;,&#8217;Gender&#8217;:&#8217;Male&#8217;}"<http://localhost:49228/api/Home>&#160; &#8211;header "Content-type:application/json"

Entonces resulta que ambos parámetros son null. Parece ser que vNext **no enlaza por defecto datos en JSON, solo en www-form-urlencoded**. Además mandar datos en JSON hace que los parámetros no se enlacen. Aunque mande datos a través de la URL (p. ej. como route values) esos no se usan.

Por supuesto vNext soporta JSON, pero es que nos falta probar una cosilla…

**Atributo [FromBody]**

De momento en vNext existe el atributo \[FromBody\] (pero no existe el [FromUri]). Ni corto ni perezoso he aplicado el FromBody a uno de los parámetros del controlador:

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:875d346c-1af9-446b-ba8d-5799d209f36a" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
    <div style="background: #ddd; max-height: 300px; overflow: auto">
      <ol start="1" style="background: #1d1d1d; margin: 0 0 0 2em; padding: 0 0 0 5px;">
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">public</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">bool</span><span style="background:#1e1e1e;color:#dcdcdc"> Post(</span><span style="background:#1e1e1e;color:#4ec9b0">Product</span><span style="background:#1e1e1e;color:#dcdcdc"> product, [</span><span style="background:#1e1e1e;color:#4ec9b0">FromBody</span><span style="background:#1e1e1e;color:#dcdcdc">] </span><span style="background:#1e1e1e;color:#4ec9b0">Customer</span><span style="background:#1e1e1e;color:#dcdcdc"> customer)</span>
        </li>
        <li style="background: #111111">
          <span style="background:#1e1e1e;color:#dcdcdc">{</span>
        </li>
        <li>
              <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">return</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">true</span><span style="background:#1e1e1e;color:#dcdcdc">;</span>
        </li>
        <li style="background: #111111">
          <span style="background:#1e1e1e;color:#dcdcdc">}</span>
        </li>
      </ol>
    </div></p>
  </div></p>
</div>

Y he repetido la última petición (el POST a /api/Home/10). Y el resultado ha sido… **un error:**

<font size="2" face="Consolas">System.InvalidOperationException: 415: Unsupported content type Microsoft.AspNet.Mvc.ModelBinding.ContentTypeHeaderValue</font>

He modificado la petición cURL para usar JSON en lugar de form-urlencoded:

curl &#8211;data "{&#8216;Name&#8217;:&#8217;eiximenis&#8217;,&#8217;Gender&#8217;:&#8217;Male&#8217;}" <http://localhost:49228/api/Home/10>&#160; &#8211;header "Content-type:application/json"

Y el resultado ha sido muy interesante:

[<img title="image" style="border-left-width: 0px; border-right-width: 0px; background-image: none; border-bottom-width: 0px; padding-top: 0px; padding-left: 0px; display: inline; padding-right: 0px; border-top-width: 0px" border="0" alt="image" src="http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_149C1B19.png" width="393" height="116" />][2]

El parámetro _customer_ se ha enlazado a partir de los datos en JSON del cuerpo (el Id está a 0 porque es un route value y no está en el cuerpo de la petición) pero el parámetro _product_ está a null. **Por lo tanto el uso de [FromBody] modifica el model binding a un modelo más parecido al de WebApi**.

WebApi solo permite un solo parámetro enlazado desde el cuerpo de la petición. Mi duda ahora era si vNext tiene la misma restricción. Mirando el código fuente de la clase JsonInputFormatter intuía que sí… y efectivamente. Aunque a diferencia de WebApi no da error si no que **tan solo enlaza el primer parámetro**. Así si tengo el método:

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:ecf028ed-3aa5-444a-bf8e-7c43f2619995" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
    <div style="background: #ddd; max-height: 300px; overflow: auto">
      <ol start="1" style="background: #1d1d1d; margin: 0 0 0 2em; padding: 0 0 0 5px;">
        <li>
          <span style="background:#1e1e1e;color:#569cd6">public</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">bool</span><span style="background:#1e1e1e;color:#dcdcdc"> Post([</span><span style="background:#1e1e1e;color:#4ec9b0">FromBody</span><span style="background:#1e1e1e;color:#dcdcdc">] </span><span style="background:#1e1e1e;color:#4ec9b0">Product</span><span style="background:#1e1e1e;color:#dcdcdc"> product, [</span><span style="background:#1e1e1e;color:#4ec9b0">FromBody</span><span style="background:#1e1e1e;color:#dcdcdc">] </span><span style="background:#1e1e1e;color:#4ec9b0">Customer</span><span style="background:#1e1e1e;color:#dcdcdc"> customer)</span>
        </li>
      </ol>
    </div></p>
  </div></p>
</div>

Y repito la llamada cURL anterior, los datos recibidos son:

[<img title="image" style="border-left-width: 0px; border-right-width: 0px; background-image: none; border-bottom-width: 0px; padding-top: 0px; padding-left: 0px; display: inline; padding-right: 0px; border-top-width: 0px" border="0" alt="image" src="http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_2FD4741A.png" width="399" height="99" />][3]

El parámetro _product_ (el primero) se ha enlazado a partir del cuerpo de la petición y el segundo vale _null_.

**¿Y como funciona todo (más o menos)?**

Recordad que ASP.NET vNext es open source y que nos podemos bajar libremente el código de <a href="https://github.com/aspnet" target="_blank" rel="noopener noreferrer">su repositorio de GitHub</a>. Con este vistazo al código he visto algunas cosillas.

El método interesante es el método GetActionArguments de la clase ReflectedActionInvoker. Dicho método es el encargado de obtener los argumentos de la acción (por tanto de todo el proceso de model binding). Dicho método hace lo siguiente:

  * Obtiene el BindingContext. El BindingContext es un objeto que tiene varias propiedad
  
    es, entre ellas 3 que nos interesan: </p> 
      1. El InputFormatterProvider a usar 
      2. El ModelBinder a usar 
      3. Los Value providers a usar 
  * Obtiene los parámetros de la acción. Cada parametro viene representado por un objeto ParameterDescriptor. Si el controlador acepta dos parámtetros (_customer_ y _product_) existen dos objetos ParameterDescriptor, uno representando a cada parámetro de la acción. Dicha clase tiene una propiedad llamada BodyParameterInfo. **Si el valor de dicha propiedad es null se usa un binding más tipo MVC (basado en value providers y model binders). Si el valor no es null se usa un binding más tipo WebApi (basado en InputFormatters).** 

Por defecto vNext viene con los siguientes Value Providers:

  1. Uno para query string (se crea siempre) 
  2. Uno para form data (se crea solo si el content type es application/x-www-form-urlencoded 
  3. Otro para route values (se crea siempre) 

La clave está en el uso del atributo [FromBody] cuando tenemos un parámetro enlazado mediante este atributo entonces no se usan los value providers si no los InputFormatters. Pueden haber dado de alta varios InputFormatters pero solo se aplicará uno (basado en el content-type). Por defecto vNext incluye un solo InputFormatter para application/json.

Ahora bien… qué pasa si tengo un controlador como el siguiente:

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:4f8fdbf4-9cca-48d9-aec0-ecae01603ccb" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
    <div style="background: #ddd; max-height: 300px; overflow: auto">
      <ol start="1" style="background: #1d1d1d; margin: 0 0 0 2em; padding: 0 0 0 5px;">
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">public</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#b8d7a3">IActionResult</span><span style="background:#1e1e1e;color:#dcdcdc"> Index([</span><span style="background:#1e1e1e;color:#4ec9b0">FromBody</span><span style="background:#1e1e1e;color:#dcdcdc">] </span><span style="background:#1e1e1e;color:#4ec9b0">Customer</span><span style="background:#1e1e1e;color:#dcdcdc"> customer, </span><span style="background:#1e1e1e;color:#4ec9b0">Product</span><span style="background:#1e1e1e;color:#dcdcdc"> product)</span>
        </li>
        <li style="background: #111111">
          <span style="background:#1e1e1e;color:#dcdcdc">{</span>
        </li>
        <li>
              <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">return</span><span style="background:#1e1e1e;color:#dcdcdc"> View();</span>
        </li>
        <li style="background: #111111">
          <span style="background:#1e1e1e;color:#dcdcdc">}</span>
        </li>
      </ol>
    </div></p>
  </div></p>
</div>

Y hago la siguiente petición?

C:UsersetomasDesktopcurl>curl –data "{&#8216;Name&#8217;:&#8217;eiximenis&#8217;,&#8217;Gender&#8217;:&#8217;Male&#8217;}" <http://localhost:38820/Home/Index/100?Name=pepe> &#8211;header "Content-type:application/json"

Pues el valor de los parámetros será como sigue:

[<img title="image" style="border-left-width: 0px; border-right-width: 0px; background-image: none; border-bottom-width: 0px; padding-top: 0px; padding-left: 0px; display: inline; padding-right: 0px; border-top-width: 0px" border="0" alt="image" src="http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_6D646294.png" width="394" height="114" />][4]

Se puede ver como el parámetro enlazado con el [FromBody] se enlaza con los parámetros del cuerpo (en JSON) mientras que el parámetro enlazado sin [FromBody] se enlaza con el resto de parámetros (de la URL, routevalues y querystring). En vNext el [FromUri] no es necesario: si hay un [FromBody] el resto de elementos deben ser enlazados desde la URL. Si no hay [FromBody] los elementos serán enlazados desde cualquier parte de la request.

Bueno… en este post hemos visto un poco el funcionamiento de ASP.NET vNext en cuanto a model binding. El resumen es que estamos ante un modelo mixto del de ASP.NET MVC y WebApi. 

En futuros posts veremos como podemos añadir InputFormatters y ValueProviders para configurar el sistema de model binding de vNext.

Saludos!

 [1]: http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_77936C50.png
 [2]: http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_22DA9409.png
 [3]: http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_3E12ED0A.png
 [4]: http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_2B5DF353.png