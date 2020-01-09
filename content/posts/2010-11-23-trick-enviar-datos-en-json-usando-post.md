---
title: 'Trick: Enviar datos en JSON usando POST'
author: eiximenis

date: 2010-11-23T11:36:00+00:00
geeks_url: /?p=1544
geeks_visits:
  - 19025
geeks_ms_views:
  - 5461
categories:
  - Uncategorized

---
Muy buenas!

Una de las preguntas que mucha gente se formula cuando empieza a hacer cosillas con ajax y jQuery es _¿Como enviar datos codificados en JSON usando POST_? 

La verdad es que es muy sencillo, aunque jQuery no proporciona ninguna función _por defecto_ que haga esto. Vamos a ver tres aproximaciones, las dos primeras incorrectas pero que nos acercarán para llegar al final a <span style="text-decoration: line-through;">la</span> una forma correcta de hacerlo.

**Aproximación 1: Usando $.post**

jQuery tiene una función específica para enviar datos usando post, llamada [$.post][1]. Así podriamos pensar que el siguiente método funcionaria:

<div id="codeSnippetWrapper" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; width: 97.5%; font-family: 'Courier New', courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; cursor: text; border: silver 1px solid; padding: 4px;">
  <pre id="codeSnippet" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 0em; width: 100%; font-family: 'Courier New', courier, monospace; direction: ltr; color: black; font-size: 8pt; overflow: visible; border-style: none; padding: 0px;"><span style="color: #0000ff;">function</span> JsonPost(data, url, handler)<br />{<br />    $.post(url, data, handler);<br />}</pre>
</div>

Y luego podríamos llamarlo de la siguiente manera:

<div id="codeSnippetWrapper" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; width: 97.5%; font-family: 'Courier New', courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; cursor: text; border: silver 1px solid; padding: 4px;">
  <pre id="codeSnippet" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 0em; width: 100%; font-family: 'Courier New', courier, monospace; direction: ltr; color: black; font-size: 8pt; overflow: visible; border-style: none; padding: 0px;"><span style="color: #008000;">// Envia un objeto con propiedades Col y Row a /Map/ViewPort y</span><br /><span style="color: #008000;">// llama a fillMap con el resultado devuelto</span><br />JsonPost({ Col: c, Row: r}, <span style="color: #006080;">"/Map/Viewport"</span>, <span style="color: #0000ff;">function</span>(data) {<br />    fillMap(data);<br />});</pre>
</div>

Pero eso no realiza una petición JSON. Si usamos, p.ej. Firebug para analizar la petición vemos que lo que se envía al controlador es:

[<img height="118" width="244" src="/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_7F20259B.png" alt="image" border="0" title="image" style="background-image: none; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" />][2]

Si os fijáis los datos no están codificados en JSON, sinó en el formato &ldquo;estándard&rdquo; (param=value&param=value&...)

Si se lee la documentación de $.post() se ve que acepta un último parámetro _datatype_, así que podríamos pensar que poniendo dicho parámetro a &ldquo;json&rdquo; funcionará, pero no. El parámetro &ldquo;datatype&rdquo; indica **el tipo de datos esperado de vuelta**, no el que se envia.

Resumiendo: $.post() **siempre envía los datos en el formato _tradicional_.**

**Aproximación 2: Usando $.post() y convertir los datos a Json**

De acuerdo, hemos visto que $.post() nos transforma nuestro objeto javascript en la cadena _tradicional_ de param=value&param=value&... Pero si a $.post() se le pasa una cadena la manda _tal cual_, por lo que si transformamos el objeto javascript a una cadena JSON parece que todo funcionará.

Para transformar un objeto javascript a notación JSON podemos usar el plugin [jQuery-Json][3]. Puede haber otros plugins por ahí, pero uséis el que uséis, **aseguraros que soporta [native json][4]**. Native json es una funcionalidad que los nuevos navegadores traen y que se basa en un método llamado JSON.stringify que transforma el objeto pasado a una cadena json. Evidentemente siempre será mucho más rápido si el propio navegador puede hacer esto de forma nativa que si es código javascript que construye la cadena. El plugin jQuery-json usa JSON.stringify si está disponible y sólo en caso de que no exista usa código javascript.

Así pues podríamos modificar nuestra JsonPost para que quede como:

<div id="codeSnippetWrapper" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; width: 97.5%; font-family: 'Courier New', courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; cursor: text; border: silver 1px solid; padding: 4px;">
  <pre id="codeSnippet" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 0em; width: 100%; font-family: 'Courier New', courier, monospace; direction: ltr; color: black; font-size: 8pt; overflow: visible; border-style: none; padding: 0px;"><span style="color: #0000ff;">function</span> JsonPost(data, url, handler)<br />{<br />    $.post(url,$.toJSON(data), handler);<br />}</pre>
</div>

El método $.toJSON es el método proporcionado por el plugin. Ahora sí que estamos usando JSON para enviar los datos:

[<img height="89" width="244" src="/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_4F82070F.png" alt="image" border="0" title="image" style="background-image: none; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" />][5]

Ok. Un parentesis: Recordad que MVC2 **no tiene soporte directo para realizar binding de datos enviados en JSON**. Aunque [es muy fácil crearse un Value Provider propio que de soporte a JSON en MVC2][6]. Y recordad que MVC3 ya viene con el soporte por defecto de JSON.

Bien, si usáis un value provider para JSON que os hayais encontrado por &ldquo;ahí afuera&rdquo; lo más probable es que no os funcione, es decir que en el controlador no recibáis los datos. Por que? Pues por lo que está marcado en rojo en la imagen antrior: aunque estamos enviando los datos en formato json, el content-type no es correcto. El content-type de JSON es _application/json_ y este es el content-type que suelen mirar los value providers de JSON.

**Aproximación 3: Usando $.ajax()**

Bien, ya que $.post() no permite especificar el content-type, la tercera y última aproximación es usar [$.ajax()][7], que es la función más personalizable que tiene jQuery para hacer peticiones ajax.

De hecho básicamente lo único que tenemos que cambiar respecto la aproximación anterior es el content-type:

<div id="codeSnippetWrapper" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; width: 97.5%; font-family: 'Courier New', courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; cursor: text; border: silver 1px solid; padding: 4px;">
  <pre id="codeSnippet" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 0em; width: 100%; font-family: 'Courier New', courier, monospace; direction: ltr; color: black; font-size: 8pt; overflow: visible; border-style: none; padding: 0px;"><span style="color: #0000ff;">function</span> JsonPost(data, url, handler)<br />{<br />    $.ajax(<br />    {<br />        url: url,<br />        type: <span style="color: #006080;">"POST"</span>,<br />        success: handler,<br />        data: $.toJSON(data),<br />        contentType: <span style="color: #006080;">"application/json"</span><br />    });<br />}</pre>
</div>

Y lo que nos muestra Firebug de la petición:

[<img height="83" width="244" src="/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_34693801.png" alt="image" border="0" title="image" style="background-image: none; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border: 0px;" />][8]

Fijaos que ahora el propio Firebug reconoce que la petición es en JSON y me muestra los datos en JSON.

Espero que os sea útil! 

Un saludo!

 [1]: http://api.jquery.com/jQuery.post/
 [2]: /cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_6E98507A.png
 [3]: http://code.google.com/p/jquery-json/
 [4]: http://www.west-wind.com/weblog/posts/729630.aspx
 [5]: /cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_5DC07FFF.png
 [6]: /blogs/etomas/archive/2010/06/01/asp-net-mvc-custom-model-binders-vs-valueproviders-y-un-ejemplo-con-json.aspx
 [7]: http://api.jquery.com/jQuery.ajax/
 [8]: /cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_1C45CDA6.png