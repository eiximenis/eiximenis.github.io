---
title: ASP.NET MVC–Traducir los mensajes de error de DataAnnotations… otra vez.
author: eiximenis

date: 2014-10-09T17:12:50+00:00
geeks_url: /?p=1683
geeks_visits:
  - 1805
geeks_ms_views:
  - 2603
categories:
  - Uncategorized

---
Pues sí… la verdad **es que esa es una cuestión recurrente** en ASP.NET MVC. Y es que con las distintas versiones de MVC han aparecido distintas maneras de conseguir este propósito.

**Nota 1:** Para tener **una idea de como eran las cosas en MVC2** echad un vistazo al post que publicó el Maestro hace tiempo: [Modificar los mensajes de validación por defecto en ASP.NET MVC 2][1]. **Por favor léete dicho post, pues en cierto modo mi post es una “continuación”**.

**Nota 2:** Una opción rápida es instalar paquetes de idioma de MVC. Esos paquetes vienen con los mensajes ya traducidos en varios idiomas. Podemos instalar tantos paquetes de idiomas como necesitemos y dependiendo de la cultura en el hilo del servidor se usará uno u otro. Eso nos permite tener los mensajes traducidos (aunque no podremos modificarlos, son los que son). De nuevo el Maestro publicó sobre ello: [Errores de ASP.NET MVC 4 en distintos idiomas][2]

El post de José María explicar muy bien como era la situación en MVC2. Pero en MVC3 y sobretodo en MVC4 hubieron algunos cambios significativos que voy a comentar en este post.

Por supuesto **podemos seguir usando la propiedad ErrorMessage** de los atributos de Data Annotations. Pero eso sigue sin ser multi-idioma y además es muy pesado. Otra opción **que sigue siendo válida y de hecho es la que se sigue (indirectamente) usando son las propiedades ErrorMessageResourceName y ErrorMessageResourceType**.

**Herencia de atributos**

Jose María menciona en su post la posibilidad de usar esas propiedades a cada atributo de DataAnnotations (lo que no es muy DRY) o bien **crearse un atributo de DataAnnotations derivado** que auto-asigne dichas propiedades. Es decir hacer algo como lo siguiente:

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:11a40aa2-cd79-41cc-b298-38a0ee2bdcd5" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
    <div style="background: #ddd; max-height: 300px; overflow: auto">
      <ol start="1" style="background: #1d1d1d; margin: 0 0 0 2.5em; padding: 0 0 0 5px;">
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">public</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">class</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#4ec9b0">LocalizedRangeAttribute</span><span style="background:#1e1e1e;color:#dcdcdc"> : </span><span style="background:#1e1e1e;color:#4ec9b0">RangeAttribute</span>
        </li>
        <li style="background: #111111">
          <span style="background:#1e1e1e;color:#dcdcdc">{</span>
        </li>
        <li>
          &nbsp;
        </li>
        <li style="background: #111111">
              <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">public</span><span style="background:#1e1e1e;color:#dcdcdc"> LocalizedRangeAttribute(</span><span style="background:#1e1e1e;color:#569cd6">int</span><span style="background:#1e1e1e;color:#dcdcdc"> min, </span><span style="background:#1e1e1e;color:#569cd6">int</span><span style="background:#1e1e1e;color:#dcdcdc"> max) : </span><span style="background:#1e1e1e;color:#569cd6">base</span><span style="background:#1e1e1e;color:#dcdcdc">(min, max)</span>
        </li>
        <li>
              <span style="background:#1e1e1e;color:#dcdcdc">{</span>
        </li>
        <li style="background: #111111">
                  <span style="background:#1e1e1e;color:#dcdcdc">InitProps();</span>
        </li>
        <li>
              <span style="background:#1e1e1e;color:#dcdcdc">}</span>
        </li>
        <li style="background: #111111">
          &nbsp;
        </li>
        <li>
              <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">public</span><span style="background:#1e1e1e;color:#dcdcdc"> LocalizedRangeAttribute(</span><span style="background:#1e1e1e;color:#569cd6">double</span><span style="background:#1e1e1e;color:#dcdcdc"> min, </span><span style="background:#1e1e1e;color:#569cd6">double</span><span style="background:#1e1e1e;color:#dcdcdc"> max) : </span><span style="background:#1e1e1e;color:#569cd6">base</span><span style="background:#1e1e1e;color:#dcdcdc">(min, max)</span>
        </li>
        <li style="background: #111111">
              <span style="background:#1e1e1e;color:#dcdcdc">{</span>
        </li>
        <li>
                  <span style="background:#1e1e1e;color:#dcdcdc">InitProps();</span>
        </li>
        <li style="background: #111111">
              <span style="background:#1e1e1e;color:#dcdcdc">}</span>
        </li>
        <li>
          &nbsp;
        </li>
        <li style="background: #111111">
              <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">public</span><span style="background:#1e1e1e;color:#dcdcdc"> LocalizedRangeAttribute(</span><span style="background:#1e1e1e;color:#4ec9b0">Type</span><span style="background:#1e1e1e;color:#dcdcdc"> type, </span><span style="background:#1e1e1e;color:#569cd6">string</span><span style="background:#1e1e1e;color:#dcdcdc"> min, </span><span style="background:#1e1e1e;color:#569cd6">string</span><span style="background:#1e1e1e;color:#dcdcdc"> max) : </span><span style="background:#1e1e1e;color:#569cd6">base</span><span style="background:#1e1e1e;color:#dcdcdc">(type, min, max)</span>
        </li>
        <li>
              <span style="background:#1e1e1e;color:#dcdcdc">{</span>
        </li>
        <li style="background: #111111">
                  <span style="background:#1e1e1e;color:#dcdcdc">InitProps();</span>
        </li>
        <li>
              <span style="background:#1e1e1e;color:#dcdcdc">}</span>
        </li>
        <li style="background: #111111">
          &nbsp;
        </li>
        <li>
              <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">private</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">void</span><span style="background:#1e1e1e;color:#dcdcdc"> InitProps()</span>
        </li>
        <li style="background: #111111">
              <span style="background:#1e1e1e;color:#dcdcdc">{</span>
        </li>
        <li>
                  <span style="background:#1e1e1e;color:#dcdcdc">ErrorMessageResourceName </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#d69d85">"Range"</span><span style="background:#1e1e1e;color:#dcdcdc">;</span>
        </li>
        <li style="background: #111111">
                  <span style="background:#1e1e1e;color:#dcdcdc">ErrorMessageResourceType </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">typeof</span><span style="background:#1e1e1e;color:#dcdcdc"> (Resources</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#4ec9b0">Messages</span><span style="background:#1e1e1e;color:#dcdcdc">);</span>
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

Por supuesto me he creado mi fichero Resources.resx dentro de la carpeta App_GlobalResources (tal y como cuenta José María en su post):

[<img title="image" style="border-top: 0px; border-right: 0px; background-image: none; border-bottom: 0px; padding-top: 0px; padding-left: 0px; border-left: 0px; display: inline; padding-right: 0px" border="0" alt="image" src="http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_5346C5
12.png" width="504" height="74" />][3]

Lamentablemente **eso rompe la validación en cliente de MVC3.** A modo de ejemplo tengo dicha entidad, con dos propiedades, una decorada con el Range de toda la vida y otra con mi LocalizedRange:

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:867fb380-d056-4b2c-ab3f-0d8fb91209c1" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
    <div style="background: #ddd; max-height: 300px; overflow: auto">
      <ol start="1" style="background: #1d1d1d; margin: 0 0 0 2em; padding: 0 0 0 5px;">
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">public</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">class</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#4ec9b0">Ufo</span>
        </li>
        <li style="background: #111111">
          <span style="background:#1e1e1e;color:#dcdcdc">{</span>
        </li>
        <li>
              <span style="background:#1e1e1e;color:#dcdcdc">[</span><span style="background:#1e1e1e;color:#4ec9b0">Range</span><span style="background:#1e1e1e;color:#dcdcdc">(</span><span style="background:#1e1e1e;color:#b5cea8">1</span><span style="background:#1e1e1e;color:#dcdcdc">,</span><span style="background:#1e1e1e;color:#b5cea8">90</span><span style="background:#1e1e1e;color:#dcdcdc">)]</span>
        </li>
        <li style="background: #111111">
              <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">public</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">int</span><span style="background:#1e1e1e;color:#dcdcdc"> Age { </span><span style="background:#1e1e1e;color:#569cd6">get</span><span style="background:#1e1e1e;color:#dcdcdc">; </span><span style="background:#1e1e1e;color:#569cd6">set</span><span style="background:#1e1e1e;color:#dcdcdc">; }</span>
        </li>
        <li>
          &nbsp;
        </li>
        <li style="background: #111111">
              <span style="background:#1e1e1e;color:#dcdcdc">[</span><span style="background:#1e1e1e;color:#4ec9b0">LocalizedRange</span><span style="background:#1e1e1e;color:#dcdcdc">(</span><span style="background:#1e1e1e;color:#b5cea8">1</span><span style="background:#1e1e1e;color:#dcdcdc">,</span><span style="background:#1e1e1e;color:#b5cea8">90</span><span style="background:#1e1e1e;color:#dcdcdc">)]</span>
        </li>
        <li>
              <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">public</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">int</span><span style="background:#1e1e1e;color:#dcdcdc"> LocalizedAge { </span><span style="background:#1e1e1e;color:#569cd6">get</span><span style="background:#1e1e1e;color:#dcdcdc">; </span><span style="background:#1e1e1e;color:#569cd6">set</span><span style="background:#1e1e1e;color:#dcdcdc">; }</span>
        </li>
        <li style="background: #111111">
          <span style="background:#1e1e1e;color:#dcdcdc">}</span>
        </li>
      </ol>
    </div></p>
  </div></p>
</div>

Creo una vista estándar para editar objetos de este modelo:

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:84826f11-77cf-466d-b98a-f7f25376c024" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
    <div style="background: #ddd; max-height: 300px; overflow: auto">
      <ol start="1" style="background: #1d1d1d; margin: 0 0 0 2em; padding: 0 0 0 5px;">
        <li>
          <span style="background:#ffffb3;color:#000000">@model </span><span style="background:#1e1e1e;color:#dcdcdc">WebApplication1</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">Models</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#4ec9b0">Ufo</span>
        </li>
        <li style="background: #111111">
          &nbsp;
        </li>
        <li>
          &nbsp;
        </li>
        <li style="background: #111111">
          <span style="background:#ffffb3;color:#000000">@</span><span style="background:#1e1e1e;color:#dcdcdc">Html</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">LabelFor(m</span><span style="background:#1e1e1e;color:#b4b4b4">=></span><span style="background:#1e1e1e;color:#dcdcdc">m</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">Age)</span>
        </li>
        <li>
          <span style="background:#ffffb3;color:#000000">@</span><span style="background:#1e1e1e;color:#dcdcdc">Html</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">TextBoxFor(m</span><span style="background:#1e1e1e;color:#b4b4b4">=></span><span style="background:#1e1e1e;color:#dcdcdc">m</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">Age)</span>
        </li>
        <li style="background: #111111">
          <span style="background:#1e1e1e;color:#808080"><</span><span style="background:#1e1e1e;color:#569cd6">br</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#808080">/></span>
        </li>
        <li>
          <span style="background:#ffffb3;color:#000000">@</span><span style="background:#1e1e1e;color:#dcdcdc">Html</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">LabelFor(m </span><span style="background:#1e1e1e;color:#b4b4b4">=></span><span style="background:#1e1e1e;color:#dcdcdc"> m</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">LocalizedAge)</span>
        </li>
        <li style="background: #111111">
          <span style="background:#ffffb3;color:#000000">@</span><span style="background:#1e1e1e;color:#dcdcdc">Html</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">TextBoxFor(m </span><span style="background:#1e1e1e;color:#b4b4b4">=></span><span style="background:#1e1e1e;color:#dcdcdc"> m</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">LocalizedAge)</span>
        </li>
      </ol>
    </div></p>
  </div></p>
</div>

Si nos vamos al código fuente de la página veremos lo siguiente:

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:d5d17b0f-93dc-4a4e-9563-e9eb16393cbc" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
    <div style="background: #ddd; max-height: 300px; overflow: auto">
      <ol start="1" style="background: #1d1d1d; margin: 0 0 0 2em; padding: 0 0 0 5px;">
        <li>
          <span style="background:#1e1e1e;color:#808080"><</span><span style="background:#1e1e1e;color:#569cd6">label</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#9cdcfe">for</span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#c8c8c8">"Age"</span><span style="background:#1e1e1e;color:#808080">></span><span style="background:#1e1e1e;color:#dcdcdc">Age</span><span style="background:#1e1e1e;color:#808080"></</span><span style="background:#1e1e1e;color:#569cd6">label</span><span style="background:#1e1e1e;color:#808080">></span>
        </li>
        <li style="background: #111111">
          <span style="background:#1e1e1e;color:#808080"><</span><span style="background:#1e1e1e;color:#569cd6">input</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#9cdcfe">data-val</span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e
1e;color:#c8c8c8">"true"</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#9cdcfe">data-val-number</span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#c8c8c8">"The field Age must be a number."</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#9cdcfe">data-val-range</span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#c8c8c8">"The field Age must be between 1 and 90."</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#9cdcfe">data-val-range-max</span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#c8c8c8">"90"</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#9cdcfe">data-val-range-min</span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#c8c8c8">"1"</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#9cdcfe">data-val-required</span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#c8c8c8">"The Age field is required."</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#9cdcfe">id</span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#c8c8c8">"Age"</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#9cdcfe">name</span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#c8c8c8">"Age"</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#9cdcfe">type</span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#c8c8c8">"text"</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#9cdcfe">value</span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#c8c8c8">""</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#808080">/></span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#808080"><</span><span style="background:#1e1e1e;color:#569cd6">br</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#808080">/></span>
        </li>
        <li style="background: #111111">
          <span style="background:#1e1e1e;color:#808080"><</span><span style="background:#1e1e1e;color:#569cd6">label</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#9cdcfe">for</span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#c8c8c8">"LocalizedAge"</span><span style="background:#1e1e1e;color:#808080">></span><span style="background:#1e1e1e;color:#dcdcdc">LocalizedAge</span><span style="background:#1e1e1e;color:#808080"></</span><span style="background:#1e1e1e;color:#569cd6">label</span><span style="background:#1e1e1e;color:#808080">></span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#808080"><</span><span style="background:#1e1e1e;color:#569cd6">input</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#9cdcfe">data-val</span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#c8c8c8">"true"</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#9cdcfe">data-val-number</span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#c8c8c8">"The field LocalizedAge must be a number."</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#9cdcfe">data-val-required</span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#c8c8c8">"The LocalizedAge field is required."</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#9cdcfe">id</span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#c8c8c8">"LocalizedAge"</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#9cdcfe">name</span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#c8c8c8">"LocalizedAge"</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#9cdcfe">type</span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#c8c8c8">"text"</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#9cdcfe">value</span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#c8c8c8">""</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#808080">/></span>
        </li>
      </ol>
    </div></p>
  </div></p>
</div>

Observa como el segundo input, que se corresponde a la propiedad LocalizedAge (decorada con mi LocalizedRangeAttribute) no tiene los atributos para validar en cliente el rango (los data-val-range-*). Por lo tanto la validación en cliente de dicho campo no funcionará.

En servidor por supuesto la validación funcionará y además se puede ver que en el segundo caso se usa el mensaje del fichero de recursos:

[<img title="image" style="border-top: 0px; border-right: 0px; background-image: none; border-bottom: 0px; padding-top: 0px; padding-left: 0px; border-left: 0px; display: inline; padding-right: 0px" border="0" alt="image" src="http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_60ACD818.png" width="504" height="64" />][4]

De todos aunque la herencia funcionase bien existen motivos para no usarla (p. ej. si quieres enviar a una vista una entidad de EF, deberías decorar dicha entidad con los atributos heredados, lo que no es muy bonito y no sé si puede causar efectos colaterales en el propio EF).

**Adaptadores de atributos**

Vale, queda claro que la herencia de atributos no funciona bien con la validación remota. Pero que no cunda el pánico, ASP.NET MVC nos da otro mecanismo: los adaptadores de atributos.

Para crear una adaptador de atributo, debemos derivar de una clase de MVC, que depende del tipo de atributo. P. ej. para crear un adaptador para el atributo de Range, debemos derivar de System.Mvc.RangeAttributeAdapter:

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:04634152-e74b-4548-9d84-5bc52f634b41" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
    <div style="background: #ddd; max-height: 300px; overflow: auto">
      <ol start="1" style="background: #1d1d1d; margin: 0 0 0 2em; padding: 0 0 0 5px;">
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">public</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">class</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#4ec9b0">LocalizedRangeAttributeAdatper</span><span style="background:#1e1e1e;color:#dcdcdc"> : </span><span style="background:#1e1e1e;color:#4ec9b0">RangeAttributeAdapter</span>
        </li>
        <li style="background: #111111">
          <span style="b
ackground:#1e1e1e;color:#dcdcdc">{</span>
        </li>
        <li>
              <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">public</span><span style="background:#1e1e1e;color:#dcdcdc"> LocalizedRangeAttributeAdatper(</span><span style="background:#1e1e1e;color:#4ec9b0">ModelMetadata</span><span style="background:#1e1e1e;color:#dcdcdc"> metadata, </span><span style="background:#1e1e1e;color:#4ec9b0">ControllerContext</span><span style="background:#1e1e1e;color:#dcdcdc"> context, </span><span style="background:#1e1e1e;color:#4ec9b0">RangeAttribute</span><span style="background:#1e1e1e;color:#dcdcdc"> attribute) : </span><span style="background:#1e1e1e;color:#569cd6">base</span><span style="background:#1e1e1e;color:#dcdcdc">(metadata, context, attribute)</span>
        </li>
        <li style="background: #111111">
              <span style="background:#1e1e1e;color:#dcdcdc">{</span>
        </li>
        <li>
                  <span style="background:#1e1e1e;color:#dcdcdc">attribute</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">ErrorMessageResourceName </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#d69d85">"Range"</span><span style="background:#1e1e1e;color:#dcdcdc">;</span>
        </li>
        <li style="background: #111111">
                  <span style="background:#1e1e1e;color:#dcdcdc">attribute</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">ErrorMessageResourceType </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">typeof</span><span style="background:#1e1e1e;color:#dcdcdc">(Resources</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#4ec9b0">Messages</span><span style="background:#1e1e1e;color:#dcdcdc">);</span>
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

El adaptador recibe en su constructor al propio RangeAttribute y allí aprovechamos para establecer las propiedades ErrorMessageResourceName y ErrorMessageResourceType.

Una vez hemos creado el adaptador, debemos declararlo en ASP.NET MVC. Para ello en el Application_Start metemos el siguiente código:

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:649b7c68-bcbd-49f5-be76-a75aa1edc020" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
    <div style="background: #ddd; max-height: 300px; overflow: auto">
      <ol start="1" style="background: #1d1d1d; margin: 0 0 0 2em; padding: 0 0 0 5px;">
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#4ec9b0">DataAnnotationsModelValidatorProvider</span><span style="background:#1e1e1e;color:#b4b4b4">.</span>
        </li>
        <li style="background: #111111">
              <span style="background:#1e1e1e;color:#dcdcdc">RegisterAdapter(</span><span style="background:#1e1e1e;color:#569cd6">typeof</span><span style="background:#1e1e1e;color:#dcdcdc"> (</span><span style="background:#1e1e1e;color:#4ec9b0">RangeAttribute</span><span style="background:#1e1e1e;color:#dcdcdc">), </span><span style="background:#1e1e1e;color:#569cd6">typeof</span><span style="background:#1e1e1e;color:#dcdcdc"> (</span><span style="background:#1e1e1e;color:#4ec9b0">LocalizedRangeAttributeAdatper</span><span style="background:#1e1e1e;color:#dcdcdc">));</span>
        </li>
      </ol>
    </div></p>
  </div></p>
</div>

La llamada RegisterAdapter acepta dos parámetros: el tipo del atributo a adaptar y el tipo del adaptador. **Una vez hecho esto, automáticamente todos los atributos Range pasarán a usar, los recursos indicados**. Ya no hay necesidad ninguna del LocalizedRange.

Otros adaptadores de atributos son los siguientes (todos en el namespace System.Web.Mvc):

[<img title="image" style="border-top: 0px; border-right: 0px; background-image: none; border-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; border-left: 0px; display: inline; padding-right: 0px" border="0" alt="image" src="http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_70BBA6CF.png" width="227" height="138" />][5]

¡Espero que os haya sido útil!

Saludos!

 [1]: http://www.variablenotfound.com/2010/02/modificar-los-mensajes-de-validacion.html
 [2]: http://www.variablenotfound.com/2012/11/errores-de-aspnet-mvc-4-en-distintos.html
 [3]: http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_1CD61310.png
 [4]: http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_07E7214E.png
 [5]: http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_154D3454.png