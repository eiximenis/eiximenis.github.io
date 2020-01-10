---
title: ASP.NET MVC–Vigila los nombres de los parámetros de tus acciones

author: eiximenis

date: 2014-10-01T09:19:36+00:00
geeks_url: /?p=1681
geeks_visits:
  - 1244
geeks_ms_views:
  - 1078
categories:
  - Uncategorized

---
Muy buenas! Un post cortito para contaros un problemilla que nos hemos encontrado en un proyecto ASP.NET MVC5. Aunque seguro que aplica a todas las versiones de MVC desde la 2 al menos.

Es uno de aquellos casos en que, **evidentemente hay algo que está mal**, pero **a simple vista todo parece correcto**. Luego das con la causa del error puede que o bien no entiendas el porqué o bien digas “¡ah claro!” dependiendo de si conoces o no como funciona el Model Binder por defecto de MVC.

**Reproducción del error**

Es muy sencillo. Create una clase llamada Beer tal y como sigue:

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:fbad8be2-0ea1-4348-bfe4-ab4cdbc5ae6e" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
    <div style="background: #ddd; max-height: 300px; overflow: auto">
      <ol start="1" style="background: #1d1d1d; margin: 0 0 0 2em; padding: 0 0 0 5px;">
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">public</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">class</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#4ec9b0">Beer</span>
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
              <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">public</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">int</span><span style="background:#1e1e1e;color:#dcdcdc"> BeerTypeId { </span><span style="background:#1e1e1e;color:#569cd6">get</span><span style="background:#1e1e1e;color:#dcdcdc">; </span><span style="background:#1e1e1e;color:#569cd6">set</span><span style="background:#1e1e1e;color:#dcdcdc">; }</span>
        </li>
        <li style="background: #111111">
          <span style="background:#1e1e1e;color:#dcdcdc">}</span>
        </li>
      </ol>
    </div></p>
  </div></p>
</div>

En una aplicación real, para editar una cerveza quizá usaríamos un viewmodel que contendría la cerveza que estamos editando y datos adicionales, p. ej. una lista con los tipos válidos para que el usuario pueda seleccionarlos de una combo:

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:01c69076-58cb-40e2-b808-421fd23c5118" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
    <div style="background: #ddd; max-height: 300px; overflow: auto">
      <ol start="1" style="background: #1d1d1d; margin: 0 0 0 2em; padding: 0 0 0 5px;">
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">public</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">class</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#4ec9b0">BeerViewModel</span>
        </li>
        <li style="background: #111111">
          <span style="background:#1e1e1e;color:#dcdcdc">{</span>
        </li>
        <li>
              <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">public</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#4ec9b0">Beer</span><span style="background:#1e1e1e;color:#dcdcdc"> Beer { </span><span style="background:#1e1e1e;color:#569cd6">get</span><span style="background:#1e1e1e;color:#dcdcdc">; </span><span style="background:#1e1e1e;color:#569cd6">set</span><span style="background:#1e1e1e;color:#dcdcdc">; }</span>
        </li>
        <li style="background: #111111">
              <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">public</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#4ec9b0">SelectList</span><span style="background:#1e1e1e;color:#dcdcdc"> BeerTypes { </span><span style="background:#1e1e1e;color:#569cd6">get</span><span style="background:#1e1e1e;color:#dcdcdc">; </span><span style="background:#1e1e1e;color:#569cd6">set</span><span style="background:#1e1e1e;color:#dcdcdc">; }</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">}</span>
        </li>
      </ol>
    </div></p>
  </div></p>
</div>

En el controlador rellenaríamos un BeerViewModel y lo mandaríamos para la vista de edición:

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:52d213d5-215a-4a83-8efd-fe1c17b2bd8a" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
    <div style="background: #ddd; max-height: 300px; overflow: auto">
      <ol start="1" style="background: #1d1d1d; margin: 0 0 0 2.5em; padding: 0 0 0 5px;">
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">public</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#4ec9b0">ActionResult</span><span style="background:#1e1e1e;color:#dcdcdc"> Edit(</span><span style="background:#1e1e1e;color:#569cd6">int</span><span style="background:#1e1e1e;color:#dcdcdc"> id)</span>
        </li>
        <li style="background: #111111">
          <span style="background:#1e1e1e;color:#dcdcdc">{</span>
        </li>
        <li>
              <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#57a64a">// Cargaramos la cerveza de la BBDD</span>
        </li>
        <li style="background: #111111">
              <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">var</span><span style="background:#1e1e1e;color:#dcdcdc"> beer </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">new</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#4ec9b0">Beer</span><span style="background:#1e1e1e;color:#dcdcdc">() {Id </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> id, Name </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#d69d85">"Beer "</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#b4b4b4">+</span><span style="background:#1e1e1e;color:#dcdcdc"> id};</span>
        </li>
        <li>
              <span style="backg
round:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">var</span><span style="background:#1e1e1e;color:#dcdcdc"> model </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">new</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#4ec9b0">BeerViewModel</span><span style="background:#1e1e1e;color:#dcdcdc">()</span>
        </li>
        <li style="background: #111111">
              <span style="background:#1e1e1e;color:#dcdcdc">{</span>
        </li>
        <li>
                  <span style="background:#1e1e1e;color:#dcdcdc">Beer </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> beer,</span>
        </li>
        <li style="background: #111111">
                  <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#57a64a">// Cargaramos los tipos de cerveza de algn sitio</span>
        </li>
        <li>
                  <span style="background:#1e1e1e;color:#dcdcdc">BeerTypes </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">new</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#4ec9b0">SelectList</span><span style="background:#1e1e1e;color:#dcdcdc">(</span><span style="background:#1e1e1e;color:#569cd6">new</span><span style="background:#1e1e1e;color:#dcdcdc">[]</span>
        </li>
        <li style="background: #111111">
                  <span style="background:#1e1e1e;color:#dcdcdc">{</span>
        </li>
        <li>
                      <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">new</span><span style="background:#1e1e1e;color:#dcdcdc"> {Id </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#b5cea8">1</span><span style="background:#1e1e1e;color:#dcdcdc">, Name </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#d69d85">"Pilsen"}</span><span style="background:#1e1e1e;color:#dcdcdc">,</span>
        </li>
        <li style="background: #111111">
                      <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">new</span><span style="background:#1e1e1e;color:#dcdcdc"> {Id </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#b5cea8">2</span><span style="background:#1e1e1e;color:#dcdcdc">, Name </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#d69d85">"Bock"}</span><span style="background:#1e1e1e;color:#dcdcdc">,</span>
        </li>
        <li>
                      <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">new</span><span style="background:#1e1e1e;color:#dcdcdc"> {Id </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#b5cea8">3</span><span style="background:#1e1e1e;color:#dcdcdc">, Name </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#d69d85">"IPA"}</span>
        </li>
        <li style="background: #111111">
                  <span style="background:#1e1e1e;color:#dcdcdc">}, </span><span style="background:#1e1e1e;color:#d69d85">"Id"</span><span style="background:#1e1e1e;color:#dcdcdc">, </span><span style="background:#1e1e1e;color:#d69d85">"Name"</span><span style="background:#1e1e1e;color:#dcdcdc">)</span>
        </li>
        <li>
              <span style="background:#1e1e1e;color:#dcdcdc">};</span>
        </li>
        <li style="background: #111111">
              <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">return</span><span style="background:#1e1e1e;color:#dcdcdc"> View(model);</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">}</span>
        </li>
      </ol>
    </div></p>
  </div></p>
</div>

La vista de edición por su parte se limita a mostrar un formulario para editar el nombre y el tipo de cerveza:

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:dbb6ba51-3a98-4b34-bfd5-b947219964a7" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
    <div style="background: #ddd; max-height: 300px; overflow: auto">
      <ol start="1" style="background: #1d1d1d; margin: 0 0 0 2.5em; padding: 0 0 0 5px;">
        <li>
          <span style="background:#ffffb3;color:#000000">@model </span><span style="background:#1e1e1e;color:#dcdcdc">WebApplication17</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">Models</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#4ec9b0">BeerViewModel</span>
        </li>
        <li style="background: #111111">
                     <span style="background:#1e1e1e;color:#dcdcdc"></span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#808080"><</span><span style="background:#1e1e1e;color:#569cd6">h2</span><span style="background:#1e1e1e;color:#808080">></span><span style="background:#1e1e1e;color:#dcdcdc">Edit a Beer</span><span style="background:#1e1e1e;color:#808080"></</span><span style="background:#1e1e1e;color:#569cd6">h2</span><span style="background:#1e1e1e;color:#808080">></span>
        </li>
        <li style="background: #111111">
          &nbsp;
        </li>
        <li>
          <span style="background:#ffffb3;color:#000000">@</span><span style="background:#1e1e1e;color:#569cd6">using</span><span style="background:#1e1e1e;color:#dcdcdc"> (Html</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">BeginForm())</span>
        </li>
        <li style="background: #111111">
          <span style="background:#1e1e1e;color:#dcdcdc">{</span>
        </li>
        <li>
              <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#ffffb3;color:#000000">@</span><span style="background:#1e1e1e;color:#dcdcdc">Html</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">LabelFor(m </span><span style="background:#1e1e1e;color:#b4b4b4">=></span><span style="background:#1e1e1e;color:#dcdcdc"> m</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">Beer</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">Name)</span>
        </li>
        <li style="background: #111111">
              <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#ffffb3;color:#000000">@</span><span style="background:#1e1e1e;color:#dcdcdc">Html</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">TextBoxFor(m </span><span style="background:#1e1e1e;color:#b4b4b4">=></span><span style="background:#1e1e1e;color:#dcdcdc"> m</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">Beer</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">Name)</span>
        </li>
        <li>
              <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#808080"><</span><span style="background:#1e1e1e;color:#569cd6">br</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#808080">/></span>
        </li>
        <li style="background: #111111">
              <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#808080"><</span><span style="backgroun
d:#1e1e1e;color:#569cd6">p</span><span style="background:#1e1e1e;color:#808080">></span><span style="background:#1e1e1e;color:#dcdcdc">Choose beer type:</span>
        </li>
        <li>
                  <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#ffffb3;color:#000000">@</span><span style="background:#1e1e1e;color:#dcdcdc">Html</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">DropDownListFor(m </span><span style="background:#1e1e1e;color:#b4b4b4">=></span><span style="background:#1e1e1e;color:#dcdcdc"> m</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">Beer</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">BeerTypeId, Model</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">BeerTypes)</span>
        </li>
        <li style="background: #111111">
              <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#808080"></</span><span style="background:#1e1e1e;color:#569cd6">p</span><span style="background:#1e1e1e;color:#808080">></span>
        </li>
        <li>
              <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#808080"><</span><span style="background:#1e1e1e;color:#569cd6">input</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#9cdcfe">type</span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#c8c8c8">"submit"</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#9cdcfe">value</span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#c8c8c8">"edit"</span><span style="background:#1e1e1e;color:#808080">/></span>
        </li>
        <li style="background: #111111">
          <span style="background:#1e1e1e;color:#dcdcdc">}</span>
        </li>
      </ol>
    </div></p>
  </div></p>
</div>

El funcionamiento de la vista es, como era de esperar, correcto:

[<img title="image" style="border-top: 0px; border-right: 0px; background-image: none; border-bottom: 0px; padding-top: 0px; padding-left: 0px; border-left: 0px; display: inline; padding-right: 0px" border="0" alt="image" src="http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_01E2ECD5.png" width="504" height="223" />][1]

Ahora creamos la acción para recibir los datos de la cerveza y miramos que datos recibimos en el controlador:

[<img title="image" style="border-top: 0px; border-right: 0px; background-image: none; border-bottom: 0px; padding-top: 0px; padding-left: 0px; border-left: 0px; display: inline; padding-right: 0px" border="0" alt="image" src="http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_718F272E.png" width="504" height="89" />][2]

¡No se ha producido el binding! Los datos que envía el navegador en el POST son correctos (no podía ser de otra forma ya que he usado los helpers para formulario):

[<img title="image" style="border-top: 0px; border-right: 0px; background-image: none; border-bottom: 0px; padding-top: 0px; padding-left: 0px; border-left: 0px; display: inline; padding-right: 0px" border="0" alt="image" src="http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_2F43C0FE.png" width="504" height="125" />][3]

**¿Cuál es la causa del fallo?**

Pues que el **parámetro de la acción se llama “beer”.** Cámbialo para que tenga otro nombre y… _voilá_:

[<img title="image" style="border-top: 0px; border-right: 0px; background-image: none; border-bottom: 0px; padding-top: 0px; padding-left: 0px; border-left: 0px; display: inline; padding-right: 0px" border="0" alt="image" src="http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_71716C45.png" width="504" height="114" />][4]

Todos los datos enlazados (excepto el Id vale, al final lo comentamos).

**¿Porque no puede mi parámetro llamarse beer?**

Porque el ViewModel que estamos usando BeerViewModel **tiene una propiedad con ese nombre.** De hecho si cambias el nombre de la propiedad del BeerViewModel te funcionará todo de nuevo. Y eso tiene que ver en como funciona el Model Binder. Déjame que te lo cuente de forma simplificada para que tengas clara el porque eso falla.

El Model Binder es el encargado de enlazar los valores de la request con los parámetros del controlador. Los parámetros que recibe el Model Binder de la request (en el form data) son los siguientes:

_Beer.Name_ y _Beer.BeerTypeId_

Cuando el Model Binder va a enlazar Beer.Name hace lo siguiente:

  * Dado que “Beer.Name” tiene un punto **el model binder busca si existe algún parámetro en el controlador llamado “Beer” (case insensitive)**. Esto es porque un controlador **puede tener varios parámetros en la acción**.
  * Si lo encuentra entonces buscará una propiedad que se llame Name en dicho parámetro y la enlazará.
  * Si no lo encuentra buscará el primer parámetro posible que tenga una propiedad llamada Beer.
  * Dentro de la propiedad llamada Beer buscará una propiedad llamada Name para enlazarla.

Por eso cuando en la acción el parámetro se llamaba “beer”, el Model Binder al enlazar el parámetro “Beer.Name” intentaba enlazar la propiedad “Name” del propio parámetro. Pero esa propiedad **no existe**. La clase BeerViewModel solo tiene una propiedad “Beer” y otra “BeerTypes”. Lo mismo ocurre con el parámetro Beer.BeerTypeId (intenta enlazar la propiedad BeerTypeId del propio BeerViewModel).

Al final el Model Binder encuentra que no hay nada que enlazar, así que no hace nada y por eso no recibimos datos.

Cuando hemos cambiado el parámetro del controlador para que se llame “data” entonces el Model Binder al enlazar “Beer.Name” busca un parámetro llamado “beer” en la acción. Pero como NO lo encuentra, entonces busca un parámetro en el controlador que tenga una propiedad llamada “Beer”. Y lo encuentra, porque el parámetro “data” (el BeerViewModel) tiene una propiedad llamada Beer. Luego busca si el tipo de dicha propiedad (la clase Beer) tiene una propiedad llamada Name. Y la encuentra, y la enlaza.

Por eso en el segundo caso recibimos los datos.

**Bonus track**: ¿Por qué el Id no se enlaza? Esa es sencilla: porque el route value se llama “id” (el POST está hecho a /Beers/Edit/{id}. El Model Binder soporta enlazado de route values, pero el nombre id no lo puede enlazar porque:

  1. No existe ningún parámetro llamado Id en la acción
  2. Ningún parámetro de la acción tiene una propiedad llamada Id.

Espero este post os haya sido interesante y si alguna vez os pasa eso… pues bueno, ya sabéis la razón! 😀

Saludos!

 [1]: http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_19780997.png
 [2]: http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_1762BCE1.png
 [3]: http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_6E1326F5.png
 [4]: http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_0BAF44B9.png