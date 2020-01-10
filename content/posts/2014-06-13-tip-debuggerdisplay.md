---
title: 'TIP: DebuggerDisplay'

author: eiximenis

date: 2014-06-13T09:50:16+00:00
geeks_url: /?p=1670
geeks_visits:
  - 1553
geeks_ms_views:
  - 1157
categories:
  - Uncategorized

---
<p align="left">
  Muy buenas! Acabo de leer el post de Luis Ruiz Pavon sobre sobre el sobreescribir .ToString() para <a href="http://geeks.ms/blogs/lruiz/archive/2014/06/13/c-sobreescribir-tostring-en-nuestras-clases-para-mejorar-la-informaci-243-n-en-modo-depuraci-243-n.aspx" target="_blank" rel="noopener noreferrer">mejorar la información del modo de depuración de VS</a>.
</p>

<p align="left">
  Y ya puestos, para complementar su post, quería comentar un truco que no se si conoce mucha gente y que permite algo parecido sin necesidad de sobreescribir ToString (que se usa para <em>otras</em> cosas además de para mostrar la información en la ventana de depuración) y que es el uso del atributo DebuggerDisplay.
</p>

<p align="left">
  En este caso un ejemplo vale más que mil palabras:
</p>

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:0c38f989-28b6-48a4-9d19-851dfecb2a6e" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
    <div style="background: #ddd; max-height: 300px; overflow: auto">
      <ol start="1" style="background: #1d1d1d; margin: 0 0 0 2.5em; padding: 0 0 0 5px;">
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">   [</span><span style="background:#1e1e1e;color:#4ec9b0">DebuggerDisplay</span><span style="background:#1e1e1e;color:#dcdcdc"> (</span><span style="background:#1e1e1e;color:#d69d85">"Product {Name} ({Code}): {Description}"</span><span style="background:#1e1e1e;color:#dcdcdc">)]</span>
        </li>
        <li style="background: #111111">
          <span style="background:#1e1e1e;color:#dcdcdc">    </span><span style="background:#1e1e1e;color:#569cd6">public</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">class</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#4ec9b0">Product</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">    {</span>
        </li>
        <li style="background: #111111">
          <span style="background:#1e1e1e;color:#dcdcdc">        </span><span style="background:#1e1e1e;color:#569cd6">public</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">string</span><span style="background:#1e1e1e;color:#dcdcdc"> Code { </span><span style="background:#1e1e1e;color:#569cd6">get</span><span style="background:#1e1e1e;color:#dcdcdc">; </span><span style="background:#1e1e1e;color:#569cd6">set</span><span style="background:#1e1e1e;color:#dcdcdc">; }</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">        </span><span style="background:#1e1e1e;color:#569cd6">public</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">string</span><span style="background:#1e1e1e;color:#dcdcdc"> Name { </span><span style="background:#1e1e1e;color:#569cd6">get</span><span style="background:#1e1e1e;color:#dcdcdc">; </span><span style="background:#1e1e1e;color:#569cd6">set</span><span style="background:#1e1e1e;color:#dcdcdc">; }</span>
        </li>
        <li style="background: #111111">
          <span style="background:#1e1e1e;color:#dcdcdc">        </span><span style="background:#1e1e1e;color:#569cd6">public</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">string</span><span style="background:#1e1e1e;color:#dcdcdc"> Description { </span><span style="background:#1e1e1e;color:#569cd6">get</span><span style="background:#1e1e1e;color:#dcdcdc">; </span><span style="background:#1e1e1e;color:#569cd6">set</span><span style="background:#1e1e1e;color:#dcdcdc">; }</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">        </span><span style="background:#1e1e1e;color:#569cd6">public</span><span style="background:#1e1e1e;color:#dcdcdc"> Product(</span><span style="background:#1e1e1e;color:#569cd6">string</span><span style="background:#1e1e1e;color:#dcdcdc"> code, </span><span style="background:#1e1e1e;color:#569cd6">string</span><span style="background:#1e1e1e;color:#dcdcdc"> name, </span><span style="background:#1e1e1e;color:#569cd6">string</span><span style="background:#1e1e1e;color:#dcdcdc"> description)</span>
        </li>
        <li style="background: #111111">
          <span style="background:#1e1e1e;color:#dcdcdc">        {</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">            Code </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> code;</span>
        </li>
        <li style="background: #111111">
          <span style="background:#1e1e1e;color:#dcdcdc">            Name </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> name;</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">            Description </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> description;</span>
        </li>
        <li style="background: #111111">
          <span style="background:#1e1e1e;color:#dcdcdc">        }</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">        </span><span style="background:#1e1e1e;color:#569cd6">public</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">override</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">string</span><span style="background:#1e1e1e;color:#dcdcdc"> ToString()</span>
        </li>
        <li style="background: #111111">
          <span style="background:#1e1e1e;color:#dcdcdc">        {</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">            </span><span style="background:#1e1e1e;color:#569cd6">return</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#4ec9b0">String</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">Format(</span><span style="background:#1e1e1e;color:#d69d85">"Code &#8211; {</span><span style="background:#1e1e1e;color:#80ff80">0}</span><span style="background:#1e1e1e;color:#d69d85">"</span><span style="background:#1e1e1e;color:#dcdcdc">, Code);</span>
        </li>
        <li style="background: #111111">
          <span style="background:#1e1e1e;color:#dcdcdc">        }</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">    }</span>
        </li>
      </ol>
    </div></p>
  </div></p>
</div>

<p align="left">
  Y el resultado en la ventana de watches es:
</p>

<p align="left">
  <a href="http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_34C012B4.png"><img title="image" style="border-top: 0px; border-right: 0px; background-image: none; border-bottom: 0px; padding-top: 0px; padding-left: 0px; border-left: 0px; display: inline; padding-right: 0px" border="0" alt="image" src="http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_3219E7B4.png" width="504" height="80" /></a>
</p>

<p align="left">
  Podemos conseguir cosas más interesantes utilizando las propiedades Name y TargetTypeName que modifican los valores de las columnas “Name” y “Type”. Así si uso el siguiente [DebuggerDisplay]:
</p>

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:fafd0aeb-8b97-4430-9fe1-228709ef7f6f" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
    <div style="background: #ddd; max-height: 300px; overflow: auto">
      <ol start="1" style="background: #1d1d1d; margin: 0 0 0 2em; padding: 0 0 0 5px;">
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">[</span><span style="background:#1e1e1e;color:#4ec9b0">DebuggerDisplay</span><span style="background:#1e1e1e;color:#dcdcdc"> (</span
><span style="background:#1e1e1e;color:#d69d85">"({Code}): {Description}"</span><span style="background:#1e1e1e;color:#dcdcdc">, Name </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#d69d85">"Product {Name}"</span><span style="background:#1e1e1e;color:#dcdcdc">, Type </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#d69d85">"Producto"</span><span style="background:#1e1e1e;color:#dcdcdc">)]</span>
        </li>
      </ol>
    </div></p>
  </div></p>
</div>

<p align="left">
  <p>
    Veo lo siguiente en la ventana de watches:
  </p>
  
  <p>
    <a href="http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_0D885A30.png"><img title="image" style="border-top: 0px; border-right: 0px; background-image: none; border-bottom: 0px; padding-top: 0px; padding-left: 0px; border-left: 0px; display: inline; padding-right: 0px" border="0" alt="image" src="http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_0D1C273B.png" width="504" height="46" /></a>
  </p>
  
  <p>
    Y en la ventana de Locals veo, ahora, lo siguiente:
  </p>
  
  <p>
    <a href="http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_08A5A674.png"><img title="image" style="border-top: 0px; border-right: 0px; background-image: none; border-bottom: 0px; padding-top: 0px; padding-left: 0px; border-left: 0px; display: inline; padding-right: 0px" border="0" alt="image" src="http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_160BB97A.png" width="504" height="51" /></a>
  </p>
  
  <p>
    El uso de DebuggerDisplay tiene sus limitaciones si se compara con redefinir ToString() (este último es un método y puede contener código como el que mostraba Luis en su post para convertir a JSON), mientras que DebuggerDisplay tan solo puede generar una pequeña cadena a partir de las propiedades.
  </p>
  
  <p>
    Si una clase tiene tanto ToString() redefinido como DebuggerDispay este último tiene precedencia para el depurador.
  </p>
  
  <p>
    ¡Y… eso es todo! 🙂
  </p>
  
  <p>
    Saludos!
  </p>
</p>