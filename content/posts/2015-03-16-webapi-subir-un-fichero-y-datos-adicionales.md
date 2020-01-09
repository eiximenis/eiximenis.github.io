---
title: 'WebApi: Subir un fichero y datos adicionales'
author: eiximenis

date: 2015-03-16T12:03:00+00:00
geeks_url: /?p=1696
geeks_visits:
  - 771
geeks_ms_views:
  - 2107
categories:
  - Uncategorized

---
El modelo de _binding_ que tiene WebApi es bastante más simple que el de MVC y por ello algunas tareas que en MVC eran más simples en WebApi pasan a ser un poco más complejas. Una de esas tareas es la subida de ficheros y de datos adicionales.

A diferencia de MVC donde el contenido del cuerpo de la petición puede ser inspeccionado numerosas veces (según los _value providers_ que tengamos configurados) en WebApi el cuerpo de la petición es un stream que puede ser leído una sola vez. A nivel práctico eso implica que de todos los parámetros que pueda recibir el controlador tan solo uno puede ser puede ser enlazado con los datos del cuerpo de la petición. El resto deben ser enlazados desde otros sitios (usualmente la URL).

Así imagina que tienes el siguiente formulario que quieres enviar a un controlador:

<div style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px" class="wlWriterEditableSmartContent" id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:581f1a32-e7ee-424a-ae16-bb868e271990">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
    <div style="background: #ddd; max-height: 300px; overflow: auto">
      <ol style="background: #000000; margin: 0 0 0 2em; padding: 0 0 0 5px;">
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#808080"><</span><span style="background:#1e1e1e;color:#569cd6">form</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#9cdcfe">method</span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#c8c8c8">&#8220;post&#8221;</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#9cdcfe">action</span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#c8c8c8">&#8220;</span><span style="background:#ffffb3;color:#000000">@</span><span style="background:#1e1e1e;color:#dcdcdc">Url</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">Action(</span><span style="background:#1e1e1e;color:#d69d85">&#8220;Index&#8221;</span><span style="background:#1e1e1e;color:#dcdcdc">, </span><span style="background:#1e1e1e;color:#d69d85">&#8220;Upload&#8221;</span><span style="background:#1e1e1e;color:#dcdcdc">)</span><span style="background:#1e1e1e;color:#c8c8c8">&#8220;</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#9cdcfe">enctype</span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#c8c8c8">&#8220;multipart/form-data&#8221;</span><span style="background:#1e1e1e;color:#808080">></span>
        </li>
        <li>
          &nbsp;&nbsp;&nbsp;&nbsp;<span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#808080"><</span><span style="background:#1e1e1e;color:#569cd6">p</span><span style="background:#1e1e1e;color:#808080">></span>
        </li>
        <li>
          &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="background:#1e1e1e;color:#dcdcdc">Name: </span><span style="background:#1e1e1e;color:#808080"><</span><span style="background:#1e1e1e;color:#569cd6">input</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#9cdcfe">type</span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#c8c8c8">&#8220;text&#8221;</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#9cdcfe">name</span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#c8c8c8">&#8220;Beer.Name&#8221;</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#808080">/></span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#808080"><</span><span style="background:#1e1e1e;color:#569cd6">br</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#808080">/></span>
        </li>
        <li>
          &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="background:#1e1e1e;color:#dcdcdc">Rating: </span><span style="background:#1e1e1e;color:#808080"><</span><span style="background:#1e1e1e;color:#569cd6">input</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#9cdcfe">type</span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#c8c8c8">&#8220;number&#8221;</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#9cdcfe">name</span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#c8c8c8">&#8220;Beer.Rating&#8221;</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#808080">/><</span><span style="background:#1e1e1e;color:#569cd6">br</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#808080">/></span>
        </li>
        <li>
          &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="background:#1e1e1e;color:#dcdcdc">Image: </span><span style="background:#1e1e1e;color:#808080"><</span><span style="background:#1e1e1e;color:#569cd6">input</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#9cdcfe">type</span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#c8c8c8">&#8220;file&#8221;</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#9cdcfe">name</span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#c8c8c8">&#8220;image&#8221;</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#808080">/><</span><span style="background:#1e1e1e;color:#569cd6">br</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#808080">/></span>
        </li>
        <li>
          &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#808080"><</span><span style="background:#1e1e1e;color:#569cd6">input</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#9cdcfe">type</span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#c8c8c8">&#8220;submit&#8221;</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#9cdcfe">value</span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#c8c8c8">&#8220;send!&#8221;</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#808080">/></span>
        </li>
        <li>
          &nbsp;&nbsp;&nbsp;&nbsp;<span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#808080"></</span><span style="background:#1e1e1e;color:#569cd6">p</span><span style="background:#1e1e1e;color:#808080">></span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#808080"></</span><span style="background:#1e1e1e;color:#569cd6">form</span><span style="background:#1e1e1e;color:#808080">></span>
        </li>
      </ol>
    </div>
  </div>
</div>

En MVC te bastará con declarar un parámetro de tipo HttpPostedFileBase para el fichero y otro para el resto de valores (name y rating). El framework se encargará del resto:

[<img height="236" width="644" src="/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_6B1C1201.png" alt="image" border="0" style="border-top: 0px; border-right: 0px; background-image: none; border-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 10px 10px 0px 0px; border-left: 0px; display: inline; padding-right: 0px" title="image" />][1]

En WebApi eso no es posible, ya que aquí tenemos dos parámetros (image y beer) enlazados con datos provenientes del cuerpo de la petición. Por lo tanto, en WebApi tanto los datos del fichero subido como el resto de valores deberán venir juntos (asumiendo que están todos en el cuerpo de la petición como es este ejemplo).

Además la subida de ficheros en WebApi no se gestiona enlanzando ningún parámetro del controlador, si no instanciando y usando un objeto del tipo <a target="_blank" href="https://msdn.microsoft.com/es-es/library/system.net.http.multipartfilestreamprovider%28v=vs.118%29.aspx" rel="noopener noreferrer">MultipartFileStreamProvider</a>. Esta clase **lee el cuerpo de la petición y guarda el fichero en disco.** Pero claro, si este objeto lee el cuerpo de la petición ya nadie más puede hacerlo en WebApi, lo que implica que los datos asociados también debe leerlos. Por suerte existe la clase derivada <a target="_blank" href="https://msdn.microsoft.com/es-es/library/system.net.http.multipartformdatastreamprovider%28v=vs.118%29.aspx" rel="noopener noreferrer">MultiPartFormDataStreamProvider</a> que almacena los datos adicionales en la propiedad FormData. Así podemos tener el siguiente código en un ApiController:

<div style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px" class="wlWriterEditableSmartContent" id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:d5337797-0d0f-40f8-9551-1e4adaff1d79">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
    <div style="background: #ddd; max-height: 300px; overflow: auto">
      <ol style="background: #000000; margin: 0 0 0 2.5em; padding: 0 0 0 5px;">
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">[</span><span style="background:#1e1e1e;color:#4ec9b0">HttpPost</span><span style="background:#1e1e1e;color:#dcdcdc">]</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">[</span><span style="background:#1e1e1e;color:#4ec9b0">Route</span><span style="background:#1e1e1e;color:#dcdcdc">(</span><span style="background:#1e1e1e;color:#d69d85">&#8220;api/upload&#8221;</span><span style="background:#1e1e1e;color:#dcdcdc">)]</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">public</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">async</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#4ec9b0">Task</span><span style="background:#1e1e1e;color:#b4b4b4"><</span><span style="background:#1e1e1e;color:#b8d7a3">IHttpActionResult</span><span style="background:#1e1e1e;color:#b4b4b4">></span><span style="background:#1e1e1e;color:#dcdcdc"> Upload()</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">{</span>
        </li>
        <li>
          &nbsp;&nbsp;&nbsp;&nbsp;<span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">var</span><span style="background:#1e1e1e;color:#dcdcdc"> folder </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#4ec9b0">HostingEnvironment</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">MapPath(</span><span style="background:#1e1e1e;color:#d69d85">&#8220;~/Uploads&#8221;</span><span style="background:#1e1e1e;color:#dcdcdc">);</span>
        </li>
        <li>
          &nbsp;&nbsp;&nbsp;&nbsp;<span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">var</span><span style="background:#1e1e1e;color:#dcdcdc"> provider </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">new</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#4ec9b0">MultipartFormDataStreamProvider</span><span style="background:#1e1e1e;color:#dcdcdc">(folder);</span>
        </li>
        <li>
          &nbsp;&nbsp;&nbsp;&nbsp;<span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">var</span><span style="background:#1e1e1e;color:#dcdcdc"> data </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">await</span><span style="background:#1e1e1e;color:#dcdcdc"> Request</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">Content</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">ReadAsMultipartAsync(provider);</span>
        </li>
        <li>
          &nbsp;&nbsp;&nbsp;&nbsp;<span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">return</span><span style="background:#1e1e1e;color:#dcdcdc"> Ok();</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">}</span>
        </li>
      </ol>
    </div>
  </div>
</div>

Y podemos ver como en data tenemos tanto la ubicación del fichero en disco (del servidor) puesto que ya ha sido guardado, así como una propiedad llamada FormData que es un NameValueCollection con los datos adicionales:

[<img height="272" width="644" src="/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_0890F3BF.png" alt="image" border="0" style="border-top: 0px; border-right: 0px; background-image: none; border-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 10px 10px 0px 0px; border-left: 0px; display: inline; padding-right: 0px" title="image" />][2]

A partir de aquí, convertir esta NameValueCollection en un objeto tipado, ya depende de tí. P. ej. puedes usar este método de extensión:

<div style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px" class="wlWriterEditableSmartContent" id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:a07cf975-30bb-402e-af46-b86f2a3902a9">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
    <div style="background: #ddd; max-height: 300px; overflow: auto">
      <ol style="background: #000000; margin: 0 0 0 2.5em; padding: 0 0 0 5px;">
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">static</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">class</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#4ec9b0">NameValueCollectionExtensions</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">{</span>
        </li>
        <li>
          &nbsp;&nbsp;&nbsp;&nbsp;<span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">public</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">static</span><span style="background:#1e1e1e;color:#dcdcdc"> T AsObject</span><span style="background:#1e1e1e;color:#b4b4b4"><</span><span style="background:#1e1e1e;color:#dcdcdc">T</span><span style="background:#1e1e1e;color:#b4b4b4">></span><span style="background:#1e1e1e;color:#dcdcdc">(</span><span style="background:#1e1e1e;color:#569cd6">this</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#4ec9b0">NameValueCollection</span><span style="background:#1e1e1e;color:#dcdcdc"> source, </span><span style="background:#1e1e1e;color:#569cd6">string</span><span style="background:#1e1e1e;color:#dcdcdc"> prefix)</span>
        </li>
        <li>
          &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">where</span><span style="background:#1e1e1e;color:#dcdcdc"> T : </span><span style="background:#1e1e1e;color:#569cd6">class</span><span style="background:#1e1e1e;color:#dcdcdc">, </span><span style="background:#1e1e1e;color:#569cd6">new</span><span style="background:#1e1e1e;color:#dcdcdc">()</span>
        </li>
        <li>
          &nbsp;&nbsp;&nbsp;&nbsp;<span style="background:#1e1e1e;color:#dcdcdc">{</span>
        </li>
        <li>
          &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">var</span><span style="background:#1e1e1e;color:#dcdcdc"> result </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">new</span><span style="background:#1e1e1e;color:#dcdcdc"> T();</span>
        </li>
        <li>
          &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">string</span><span style="background:#1e1e1e;color:#dcdcdc"> fullPrefix </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">string</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">IsNullOrEmpty(prefix) </span><span style="background:#1e1e1e;color:#b4b4b4">?</span><span style="background:#1e1e1e;color:#dcdcdc"> prefix : prefix </span><span style="background:#1e1e1e;color:#b4b4b4">+</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#d69d85">&#8220;.&#8221;</span><span style="background:#1e1e1e;color:#dcdcdc">;</span>
        </li>
        <li>
          &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">foreach</span><span style="background:#1e1e1e;color:#dcdcdc"> (</span><span style="background:#1e1e1e;color:#569cd6">var</span><span style="background:#1e1e1e;color:#dcdcdc"> key </span><span style="background:#1e1e1e;color:#569cd6">in</span><span style="background:#1e1e1e;color:#dcdcdc"> source</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">AllKeys</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">Where(k </span><span style="background:#1e1e1e;color:#b4b4b4">=></span><span style="background:#1e1e1e;color:#dcdcdc"> k</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">StartsWith(fullPrefix))</span><span style="background:#1e1e1e;color:#b4b4b4">.</span>
        </li>
        <li>
          &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="background:#1e1e1e;color:#dcdcdc">Select(kwp </span><span style="background:#1e1e1e;color:#b4b4b4">=></span><span style="background:#1e1e1e;color:#dcdcdc"> kwp</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">Substring(fullPrefix</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">Length)))</span>
        </li>
        <li>
          &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="background:#1e1e1e;color:#dcdcdc">{</span>
        </li>
        <li>
          &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">var</span><span style="background:#1e1e1e;color:#dcdcdc"> prop </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">typeof</span><span style="background:#1e1e1e;color:#dcdcdc">(T)</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">GetProperty(key);</span>
        </li>
        <li>
          &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">if</span><span style="background:#1e1e1e;color:#dcdcdc"> (prop </span><span style="background:#1e1e1e;color:#b4b4b4">!=</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">null</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#b4b4b4">&&</span><span style="background:#1e1e1e;color:#dcdcdc"> prop</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">CanWrite)</span>
        </li>
        <li>
          &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="background:#1e1e1e;color:#dcdcdc">{</span>
        </li>
        <li>
          &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="background:#1e1e1e;color:#dcdcdc">prop</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">SetValue(result, </span><span style="background:#1e1e1e;color:#4ec9b0">Convert</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">ChangeType(source[fullPrefix </span><span style="background:#1e1e1e;color:#b4b4b4">+</span><span style="background:#1e1e1e;color:#dcdcdc"> key], prop</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">PropertyType));</span>
        </li>
        <li>
          &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="background:#1e1e1e;color:#dcdcdc">}</span>
        </li>
        <li>
          &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="background:#1e1e1e;color:#dcdcdc">}</span>
        </li>
        <li>
          &nbsp;
        </li>
        <li>
          &nbsp;
        </li>
        <li>
          &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">return</span><span style="background:#1e1e1e;color:#dcdcdc"> result;</span>
        </li>
        <li>
          &nbsp;&nbsp;&nbsp;&nbsp;<span style="background:#1e1e1e;color:#dcdcdc">}</span>
        </li>
      </ol>
    </div>
  </div>
</div>

Este método es muy limitado (no admite propiedades complejas) pero si los datos que envías junto a los ficheros son simples, te puede servir. En este caso se usaría de la siguiente manera:

<div style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px" class="wlWriterEditableSmartContent" id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:eaa5ad2f-8685-494a-bde4-091553709f9e">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
    <div style="background: #ddd; max-height: 300px; overflow: auto">
      <ol style="background: #000000; margin: 0 0 0 2em; padding: 0 0 0 5px;">
        <li>
          <span style="background:#1e1e1e;color:#569cd6">var</span><span style="background:#1e1e1e;color:#dcdcdc"> beer </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> data</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">FormData</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">AsObject</span><span style="background:#1e1e1e;color:#b4b4b4"><</span><span style="background:#1e1e1e;color:#4ec9b0">Beer</span><span style="background:#1e1e1e;color:#b4b4b4">></span><span style="background:#1e1e1e;color:#dcdcdc">(</span><span style="background:#1e1e1e;color:#d69d85">&#8220;Beer&#8221;</span><span style="background:#1e1e1e;color:#dcdcdc">);</span>
        </li>
      </ol>
    </div>
  </div>
</div>

Y por supuesto, si necesitas soluciones más complejas que esas evalúa soluciones como <a target="_blank" href="http://randyburden.com/Slapper.AutoMapper/" rel="noopener noreferrer">Slapper.AutoMapper</a> o similares que ya te lo dan todo hecho... no vayas por ahí reinventando la rueda (o si, vamos... eso ya es cosa tuya :P)!

Saludos!

 [1]: /cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_20924376.png
 [2]: /cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_5F8654C2.png