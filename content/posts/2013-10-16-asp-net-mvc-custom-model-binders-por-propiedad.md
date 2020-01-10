---
title: ASP.NET MVC – Custom model binders por propiedad

author: eiximenis

date: 2013-10-16T11:51:55+00:00
geeks_url: /?p=1653
geeks_visits:
  - 1849
geeks_ms_views:
  - 1566
categories:
  - Uncategorized

---
Muy buenas! En este post vamos a ver como habilitar un custom model binder para una propiedad en concreto de un viewmodel.

De serie es posible configurar un Custom Model Binder de dos maneras:

  1. Añadiéndolo a la colección Binders de la clase ModelBinders. Con esto conseguimos que **todos** los valores de un tipo concreto se enlacen usando nuestro ModelBinder. 
  2. Usando el atributo [ModelBinder]. Con este atributo podemos especificar un Model Binder a usar para un viewmodel en concreto o para un parámetro en concreto de un controlador. Pero desgraciadamente no para una propiedad concreta de un viewmodel: 

[<img title="image" style="border-left-width: 0px; border-right-width: 0px; background-image: none; border-bottom-width: 0px; padding-top: 0px; padding-left: 0px; display: inline; padding-right: 0px; border-top-width: 0px" border="0" alt="image" src="http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_2FED8F68.png" width="504" height="54" />][1]

Por suerte conseguir usar un custom model binder para una propiedad en concreto de un viewmodel no es tan complicado 🙂

Lo primero es crearnos un atributo propio ya que el ModelBinderAttribute no podemos usarlo, así que vamos a ello:

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:2fb8ec5b-222d-4a6f-9fcb-457d8d880b47" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
    <div style="background: #000080; color: #fff; font-family: Verdana, Tahoma, Arial, sans-serif; font-weight: bold; padding: 2px 5px">
      Atributo [PropertyModelBinder]
    </div>
    
    <div style="background: #ddd; max-height: 300px; overflow: auto">
      <ol start="1" style="background: #1e1e1e; margin: 0 0 0 2.5em; padding: 0 0 0 5px; white-space: nowrap">
        <li>
          <span style="background:#1e1e1e;color:#4ec9b0">AttributeUsage</span><span style="background:#1e1e1e;color:#dcdcdc">(</span><span style="background:#1e1e1e;color:#b8d7a3">AttributeTargets</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">Property)]</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">    </span><span style="background:#1e1e1e;color:#569cd6">public</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">class</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#4ec9b0">PropertyModelBinderAttribute</span><span style="background:#1e1e1e;color:#dcdcdc"> : </span><span style="background:#1e1e1e;color:#4ec9b0">CustomModelBinderAttribute</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">    {</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">        </span><span style="background:#1e1e1e;color:#569cd6">private</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">readonly</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#4ec9b0">Type</span><span style="background:#1e1e1e;color:#dcdcdc"> _typeToUse;</span>
        </li>
        <li>
          &nbsp;
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">        </span><span style="background:#1e1e1e;color:#569cd6">public</span><span style="background:#1e1e1e;color:#dcdcdc"> PropertyModelBinderAttribute(</span><span style="background:#1e1e1e;color:#4ec9b0">Type</span><span style="background:#1e1e1e;color:#dcdcdc"> typeToUse)</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">        {</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">            _typeToUse </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> typeToUse;</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">        }</span>
        </li>
        <li>
          &nbsp;
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">        </span><span style="background:#1e1e1e;color:#569cd6">public</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">override</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#b8d7a3">IModelBinder</span><span style="background:#1e1e1e;color:#dcdcdc"> GetBinder()</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">        {</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">            </span><span style="background:#1e1e1e;color:#569cd6">var</span><span style="background:#1e1e1e;color:#dcdcdc"> binder </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#4ec9b0">DependencyResolver</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">Current</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">GetService(_typeToUse) </span><span style="background:#1e1e1e;color:#569cd6">as</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#b8d7a3">IModelBinder</span><span style="background:#1e1e1e;color:#dcdcdc">;</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">            </span><span style="background:#1e1e1e;color:#569cd6">return</span><span style="background:#1e1e1e;color:#dcdcdc"> binder </span><span style="background:#1e1e1e;color:#b4b4b4">??</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">new</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#4ec9b0">DefaultModelBinder</span><span style="background:#1e1e1e;color:#dcdcdc">();</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">        }</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">    }</span>
        </li>
      </ol>
    </div></p>
  </div></p>
</div>

Una vez lo tenemos creado, lo aplicamos a nuestro viewmodel:

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:55b6d6d7-e58e-4d37-8ce7-049a89f23843" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
    <div style="background: #000080; color: #fff; font-family: Verdana, Tahoma, Arial, sans-serif; font-weight: bold; padding: 2px 5px">
      ViewModel de prueba
    </div>
    
    <div style="background: #ddd; max-height: 300px; overflow: auto">
      <ol start="1" style="background: #1e1e1e; margin: 0 0 0 2em; padding: 0 0 0 5px; white-space: nowrap">
        <li>
          <span style="background:#1e1e1e;color:#569cd6">public</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">class</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#4ec9b0">SomeViewModel</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">{</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">    [</span><span style="background:#1e1e1e;color:#4ec9b0">PropertyModelBinder</span><span style="background:#1e1e1e;color:#dcdcdc">(</span><span style="background:#1e1e1e;color:#569cd6">typeof</span><span style="background:#1e1e1e;color:#dcdcdc">(</span><span style="background:#1e1e1e;color:#4ec9b0">RomanNumberModelBinder</span><span style="background:#1e1e1e;color:#dcdcdc">))]</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">    [</span><span style="background:#1e1e1e;color:#4ec9b0">Range</span><span style="background
:#1e1e1e;color:#dcdcdc">(</span><span style="background:#1e1e1e;color:#b5cea8">1</span><span style="background:#1e1e1e;color:#dcdcdc">,</span><span style="background:#1e1e1e;color:#b5cea8">21</span><span style="background:#1e1e1e;color:#dcdcdc">)]</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">    </span><span style="background:#1e1e1e;color:#569cd6">public</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">int</span><span style="background:#1e1e1e;color:#dcdcdc"> Century { </span><span style="background:#1e1e1e;color:#569cd6">get</span><span style="background:#1e1e1e;color:#dcdcdc">; </span><span style="background:#1e1e1e;color:#569cd6">set</span><span style="background:#1e1e1e;color:#dcdcdc">; }</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">    </span><span style="background:#1e1e1e;color:#569cd6">public</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">string</span><span style="background:#1e1e1e;color:#dcdcdc"> Name { </span><span style="background:#1e1e1e;color:#569cd6">get</span><span style="background:#1e1e1e;color:#dcdcdc">; </span><span style="background:#1e1e1e;color:#569cd6">set</span><span style="background:#1e1e1e;color:#dcdcdc">; }</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">}</span>
        </li>
      </ol>
    </div></p>
  </div></p>
</div>

Ahora debemos implementar el custom model binder. En este caso he implementado un model binder que convierte cadenas en formato de número romano (tal como XX o VIII) al entero correspondiente. 

> **Nota:** El código de conversión lo he sacado de <http://vadivel.blogspot.com.es/2011/09/how-to-convert-roman-numerals-to.html>. Tan solo lo he corregido para usar un SortedDictionary y así asegurarme de que el orden por el que se itera sobre las claves es el esperado (en un Dictionary<,> el orden de iteración no está definido).

El código entero de mi ModelBinder es el siguiente:

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:36da1c39-fab9-4973-9c79-ba9e80e1471d" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
    <div style="background: #000080; color: #fff; font-family: Verdana, Tahoma, Arial, sans-serif; font-weight: bold; padding: 2px 5px">
      RomanNumberModelBinder
    </div>
    
    <div style="background: #ddd; max-height: 300px; overflow: auto">
      <ol start="1" style="background: #1e1e1e; margin: 0 0 0 2.5em; padding: 0 0 0 5px; white-space: nowrap">
        <li>
          <span style="background:#1e1e1e;color:#569cd6">public</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">class</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#4ec9b0">RomanNumberModelBinder</span><span style="background:#1e1e1e;color:#dcdcdc"> : </span><span style="background:#1e1e1e;color:#b8d7a3">IModelBinder</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">{</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">    </span><span style="background:#1e1e1e;color:#569cd6">private</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">static</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#4ec9b0">List</span><span style="background:#1e1e1e;color:#b4b4b4"><</span><span style="background:#1e1e1e;color:#569cd6">string</span><span style="background:#1e1e1e;color:#b4b4b4">></span><span style="background:#1e1e1e;color:#dcdcdc"> _romanTokens </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">new</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#4ec9b0">List</span><span style="background:#1e1e1e;color:#b4b4b4"><</span><span style="background:#1e1e1e;color:#569cd6">string</span><span style="background:#1e1e1e;color:#b4b4b4">></span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">    {</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">        </span><span style="background:#1e1e1e;color:#d69d85">"M"</span><span style="background:#1e1e1e;color:#dcdcdc">,</span><span style="background:#1e1e1e;color:#d69d85">"CM"</span><span style="background:#1e1e1e;color:#dcdcdc">,</span><span style="background:#1e1e1e;color:#d69d85">"D"</span><span style="background:#1e1e1e;color:#dcdcdc">,</span><span style="background:#1e1e1e;color:#d69d85">"CD"</span><span style="background:#1e1e1e;color:#dcdcdc">,</span><span style="background:#1e1e1e;color:#d69d85">"C"</span><span style="background:#1e1e1e;color:#dcdcdc">,</span><span style="background:#1e1e1e;color:#d69d85">"XC"</span><span style="background:#1e1e1e;color:#dcdcdc">,</span><span style="background:#1e1e1e;color:#d69d85">"L"</span><span style="background:#1e1e1e;color:#dcdcdc">,</span><span style="background:#1e1e1e;color:#d69d85">"XL"</span><span style="background:#1e1e1e;color:#dcdcdc">,</span><span style="background:#1e1e1e;color:#d69d85">"X"</span><span style="background:#1e1e1e;color:#dcdcdc">,</span><span style="background:#1e1e1e;color:#d69d85">"IX"</span><span style="background:#1e1e1e;color:#dcdcdc">,</span><span style="background:#1e1e1e;color:#d69d85">"V"</span><span style="background:#1e1e1e;color:#dcdcdc">,</span><span style="background:#1e1e1e;color:#d69d85">"IV"</span><span style="background:#1e1e1e;color:#dcdcdc">,</span><span style="background:#1e1e1e;color:#d69d85">"I"</span><span style="background:#1e1e1e;color:#dcdcdc">,</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">    };</span>
        </li>
        <li>
          &nbsp;
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">    </span><span style="background:#1e1e1e;color:#569cd6">private</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">class</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#4ec9b0">RomanKeyComparer</span><span style="background:#1e1e1e;color:#dcdcdc"> : </span><span style="background:#1e1e1e;color:#b8d7a3">IComparer</span><span style="background:#1e1e1e;color:#b4b4b4"><</span><span style="background:#1e1e1e;color:#569cd6">string</span><span style="background:#1e1e1e;color:#b4b4b4">></span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">    {</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">        </span><span style="background:#1e1e1e;color:#569cd6">public</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">int</span><span style="background:#1e1e1e;color:#dcdcdc"> Compare(</span><span style="background:#1e1e1e;color:#569cd6">string</span><span style="background:#1e1e1e;color:#dcdcdc"> x, </span><span style="background:#1e1e1e;color:#569cd6">string</span><span style="background:#1e1e1e;color:#dcdcdc"> y)</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">        {</span>
        </li>
        <li>
          &nbsp;
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">            </span><span style="background:#1e1e1e;color:#569cd6">var</span><span style="background:#1e1e1e;color:#dcdcdc"> idx </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> _romanTokens</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">IndexOf(x);</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">            </span><br /> <span style="background:#1e1e1e;color:#569cd6">var</span><span style="background:#1e1e1e;color:#dcdcdc"> idx2 </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> _romanTokens</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">IndexOf(y);</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">            </span><span style="background:#1e1e1e;color:#569cd6">if</span><span style="background:#1e1e1e;color:#dcdcdc"> (idx </span><span style="background:#1e1e1e;color:#b4b4b4">==</span><span style="background:#1e1e1e;color:#dcdcdc"> idx2) </span><span style="background:#1e1e1e;color:#569cd6">return</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#b5cea8"></span><span style="background:#1e1e1e;color:#dcdcdc">;</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">            </span><span style="background:#1e1e1e;color:#569cd6">return</span><span style="background:#1e1e1e;color:#dcdcdc"> idx </span><span style="background:#1e1e1e;color:#b4b4b4"><</span><span style="background:#1e1e1e;color:#dcdcdc"> idx2 </span><span style="background:#1e1e1e;color:#b4b4b4">?</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#b4b4b4">&#8211;</span><span style="background:#1e1e1e;color:#b5cea8">1</span><span style="background:#1e1e1e;color:#dcdcdc"> : </span><span style="background:#1e1e1e;color:#b5cea8">1</span><span style="background:#1e1e1e;color:#dcdcdc">;</span>
        </li>
        <li>
          &nbsp;
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">        }</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">    }</span>
        </li>
        <li>
          &nbsp;
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">    </span><span style="background:#1e1e1e;color:#569cd6">private</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">static</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#4ec9b0">SortedDictionary</span><span style="background:#1e1e1e;color:#b4b4b4"><</span><span style="background:#1e1e1e;color:#569cd6">string</span><span style="background:#1e1e1e;color:#dcdcdc">, </span><span style="background:#1e1e1e;color:#569cd6">int</span><span style="background:#1e1e1e;color:#b4b4b4">></span><span style="background:#1e1e1e;color:#dcdcdc"> RomanNumbers </span><span style="background:#1e1e1e;color:#b4b4b4">=</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">        </span><span style="background:#1e1e1e;color:#569cd6">new</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#4ec9b0">SortedDictionary</span><span style="background:#1e1e1e;color:#b4b4b4"><</span><span style="background:#1e1e1e;color:#569cd6">string</span><span style="background:#1e1e1e;color:#dcdcdc">, </span><span style="background:#1e1e1e;color:#569cd6">int</span><span style="background:#1e1e1e;color:#b4b4b4">></span><span style="background:#1e1e1e;color:#dcdcdc">(</span><span style="background:#1e1e1e;color:#569cd6">new</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#4ec9b0">RomanKeyComparer</span><span style="background:#1e1e1e;color:#dcdcdc">());</span>
        </li>
        <li>
          &nbsp;
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">    </span><span style="background:#1e1e1e;color:#569cd6">public</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">object</span><span style="background:#1e1e1e;color:#dcdcdc"> BindModel(</span><span style="background:#1e1e1e;color:#4ec9b0">ControllerContext</span><span style="background:#1e1e1e;color:#dcdcdc"> controllerContext, </span><span style="background:#1e1e1e;color:#4ec9b0">ModelBindingContext</span><span style="background:#1e1e1e;color:#dcdcdc"> bindingContext)</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">    {</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">        </span><span style="background:#1e1e1e;color:#569cd6">if</span><span style="background:#1e1e1e;color:#dcdcdc"> (bindingContext</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">ModelType </span><span style="background:#1e1e1e;color:#b4b4b4">!=</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">typeof</span><span style="background:#1e1e1e;color:#dcdcdc"> (</span><span style="background:#1e1e1e;color:#569cd6">int</span><span style="background:#1e1e1e;color:#dcdcdc">)) </span><span style="background:#1e1e1e;color:#569cd6">return</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">null</span><span style="background:#1e1e1e;color:#dcdcdc">;</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">        </span><span style="background:#1e1e1e;color:#569cd6">var</span><span style="background:#1e1e1e;color:#dcdcdc"> value </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> bindingContext</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">ValueProvider</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">GetValue(bindingContext</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">ModelName)</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">AttemptedValue;</span>
        </li>
        <li>
          &nbsp;
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">        </span><span style="background:#1e1e1e;color:#569cd6">return</span><span style="background:#1e1e1e;color:#dcdcdc"> RomanToInt(value);</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">    }</span>
        </li>
        <li>
          &nbsp;
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">    </span><span style="background:#1e1e1e;color:#569cd6">private</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">static</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">int</span><span style="background:#1e1e1e;color:#dcdcdc"> RomanToInt(</span><span style="background:#1e1e1e;color:#569cd6">string</span><span style="background:#1e1e1e;color:#dcdcdc"> roman)</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">    {</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">        </span><span style="background:#1e1e1e;color:#569cd6">if</span><span style="background:#1e1e1e;color:#dcdcdc"> (</span><span style="background:#1e1e1e;color:#b4b4b4">!</span><span style="background:#1e1e1e;color:#dcdcdc">RomanNumbers</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">Any())</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">        {</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">            RomanNumbers</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">Add(_romanTokens[</span><span style="background:#1e1e1e;color:#b5cea8"></span><span style="background:#1e1e1e;color:#dcdcdc">], </span><span style="background:#1e1e1e;color:#b5cea8">1000</span><span style="background:#1e1e1e;color:#dcdcdc">);</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">            RomanNumbers</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">Add(_romanTokens[</span><span style="background:#1e1e1e;color:#b5cea8">1</span><span style="background:#1e1e1e;color:#dcdcdc">], </span><span style="background:#1e1e1e;color:#b5cea8">900</span><span style="background:#1e1e1e;color:#dcdcdc">);</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">            RomanNumbers</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">Add(_romanTokens[</span><span style="background:#1e1e1e;color:#b5cea8">2</span><span style="background:#1e1e1e;color:#dcdcdc">], </span><span style="background:#1e1e1e;color:#b5cea8">500</span><span style="background:#1e1e1e;color:#dcdcdc">);</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">            RomanNumbers</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">Add(_romanTokens[</span><span style="background:#1e1e1e;color:#b5cea8">3</span><span style="background:#1e1e1e;color:#dcdcdc">], </span><span style="background:#1e1e1e;color:#b5cea8">400</span><span style="background:#1e1e1e;color:#dcdcdc">);</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">            RomanNumbers</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">Add(_romanTokens[</span><span style="background:#1e1e1e;color:#b5cea8">4</span><span style="background:#1e1e1e;color:#dcdcdc">], </span><span style="background:#1e1e1e;color:#b5cea8">100</span><span style="background:#1e1e1e;color:#dcdcdc">);</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">            RomanNumbers</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">Add(_romanTokens[</span><span style="background:#1e1e1e;color:#b5cea8">5</span><span style="background:#1e1e1e;color:#dcdcdc">], </span><span style="background:#1e1e1e;color:#b5cea8">90</span><span style="background:#1e1e1e;color:#dcdcdc">);</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">            RomanNumbers</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">Add(_romanTokens[</span><span style="background:#1e1e1e;color:#b5cea8">6</span><span style="background:#1e1e1e;color:#dcdcdc">], </span><span style="background:#1e1e1e;color:#b5cea8">50</span><span style="background:#1e1e1e;color:#dcdcdc">);</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">            RomanNumbers</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">Add(_romanTokens[</span><span style="background:#1e1e1e;color:#b5cea8">7</span><span style="background:#1e1e1e;color:#dcdcdc">], </span><span style="background:#1e1e1e;color:#b5cea8">40</span><span style="background:#1e1e1e;color:#dcdcdc">);</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">            RomanNumbers</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">Add(_romanTokens[</span><span style="background:#1e1e1e;color:#b5cea8">8</span><span style="background:#1e1e1e;color:#dcdcdc">], </span><span style="background:#1e1e1e;color:#b5cea8">10</span><span style="background:#1e1e1e;color:#dcdcdc">);</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">            RomanNumbers</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">Add(_romanTokens[</span><span style="background:#1e1e1e;color:#b5cea8">9</span><span style="background:#1e1e1e;color:#dcdcdc">], </span><span style="background:#1e1e1e;color:#b5cea8">9</span><span style="background:#1e1e1e;color:#dcdcdc">);</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">            RomanNumbers</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">Add(_romanTokens[</span><span style="background:#1e1e1e;color:#b5cea8">10</span><span style="background:#1e1e1e;color:#dcdcdc">], </span><span style="background:#1e1e1e;color:#b5cea8">5</span><span style="background:#1e1e1e;color:#dcdcdc">);</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">            RomanNumbers</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">Add(_romanTokens[</span><span style="background:#1e1e1e;color:#b5cea8">11</span><span style="background:#1e1e1e;color:#dcdcdc">], </span><span style="background:#1e1e1e;color:#b5cea8">4</span><span style="background:#1e1e1e;color:#dcdcdc">);</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">            RomanNumbers</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">Add(_romanTokens[</span><span style="background:#1e1e1e;color:#b5cea8">12</span><span style="background:#1e1e1e;color:#dcdcdc">], </span><span style="background:#1e1e1e;color:#b5cea8">1</span><span style="background:#1e1e1e;color:#dcdcdc">);</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">        }</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">        </span><span style="background:#1e1e1e;color:#569cd6">int</span><span style="background:#1e1e1e;color:#dcdcdc"> result </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#b5cea8"></span><span style="background:#1e1e1e;color:#dcdcdc">;</span>
        </li>
        <li>
          &nbsp;
        </li>
        <li>
          &nbsp;
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">        </span><span style="background:#1e1e1e;color:#569cd6">foreach</span><span style="background:#1e1e1e;color:#dcdcdc"> (</span><span style="background:#1e1e1e;color:#569cd6">var</span><span style="background:#1e1e1e;color:#dcdcdc"> pair </span><span style="background:#1e1e1e;color:#569cd6">in</span><span style="background:#1e1e1e;color:#dcdcdc"> RomanNumbers)</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">        {</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">            </span><span style="background:#1e1e1e;color:#569cd6">while</span><span style="background:#1e1e1e;color:#dcdcdc"> (roman</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">IndexOf(pair</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">Key</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">ToString()) </span><span style="background:#1e1e1e;color:#b4b4b4">==</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#b5cea8"></span><span style="background:#1e1e1e;color:#dcdcdc">)</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">            {</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">                result </span><span style="background:#1e1e1e;color:#b4b4b4">+=</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">int</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">Parse(pair</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">Value</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">ToString());</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">                roman </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> roman</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">Substring(pair</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">Key</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">ToString()</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">Length);</span>
        </li>
        <p>
          <l i><span style="background:#1e1e1e;color:#dcdcdc">            }</span></li> 
          
          <li>
            <span style="background:#1e1e1e;color:#dcdcdc">        }</span>
          </li>
          <li>
            <span style="background:#1e1e1e;color:#dcdcdc">        </span><span style="background:#1e1e1e;color:#569cd6">return</span><span style="background:#1e1e1e;color:#dcdcdc"> result;</span>
          </li>
          <li>
            <span style="background:#1e1e1e;color:#dcdcdc">    }</span>
          </li>
          <li>
            <span style="background:#1e1e1e;color:#dcdcdc">}</span>
          </li></ol></div>
        </p></div> </p></div> 
        
        <p>
          Teóricamente lo tenemos todo montado. Pero si lo probamos veremos que NO funciona. Para la prueba me he generado una pequeña vista como la siguiente:
        </p>
        
        <div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:12d51033-3cb5-40ec-a6ae-96e71d7585ef" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
          <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
            <div style="background: #000080; color: #fff; font-family: Verdana, Tahoma, Arial, sans-serif; font-weight: bold; padding: 2px 5px">
              Vista de prueba
            </div>
            
            <div style="background: #ddd; max-height: 300px; overflow: auto">
              <ol start="1" style="background: #1e1e1e; margin: 0 0 0 2.5em; padding: 0 0 0 5px; white-space: nowrap">
                <li>
                  <span style="background:#ffffb3;color:#000000">@model </span><span style="background:#1e1e1e;color:#dcdcdc">WebApplication1</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">Models</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#4ec9b0">SomeViewModel</span>
                </li>
                <li>
                  &nbsp;
                </li>
                <li>
                  <span style="background:#ffffb3;color:#000000">@</span><span style="background:#1e1e1e;color:#569cd6">using</span><span style="background:#1e1e1e;color:#dcdcdc"> (Html</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">BeginForm())</span>
                </li>
                <li>
                  <span style="background:#1e1e1e;color:#dcdcdc">{</span>
                </li>
                <li>
                  <span style="background:#1e1e1e;color:#dcdcdc">    </span><span style="background:#1e1e1e;color:#808080"><</span><span style="background:#1e1e1e;color:#569cd6">p</span><span style="background:#1e1e1e;color:#808080">></span>
                </li>
                <li>
                  <span style="background:#1e1e1e;color:#dcdcdc">        </span><span style="background:#ffffb3;color:#000000">@</span><span style="background:#1e1e1e;color:#dcdcdc">Html</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">LabelFor(m </span><span style="background:#1e1e1e;color:#b4b4b4">=></span><span style="background:#1e1e1e;color:#dcdcdc"> m</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">Name)</span>
                </li>
                <li>
                  <span style="background:#1e1e1e;color:#dcdcdc">        </span><span style="background:#ffffb3;color:#000000">@</span><span style="background:#1e1e1e;color:#dcdcdc">Html</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">EditorFor(m </span><span style="background:#1e1e1e;color:#b4b4b4">=></span><span style="background:#1e1e1e;color:#dcdcdc"> m</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">Name)</span>
                </li>
                <li>
                  <span style="background:#1e1e1e;color:#dcdcdc">    </span><span style="background:#1e1e1e;color:#808080"></</span><span style="background:#1e1e1e;color:#569cd6">p</span><span style="background:#1e1e1e;color:#808080">></span>
                </li>
                <li>
                  <span style="background:#1e1e1e;color:#dcdcdc">    </span><span style="background:#1e1e1e;color:#808080"><</span><span style="background:#1e1e1e;color:#569cd6">p</span><span style="background:#1e1e1e;color:#808080">></span>
                </li>
                <li>
                  <span style="background:#1e1e1e;color:#dcdcdc">        </span><span style="background:#ffffb3;color:#000000">@</span><span style="background:#1e1e1e;color:#dcdcdc">Html</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">LabelFor(m </span><span style="background:#1e1e1e;color:#b4b4b4">=></span><span style="background:#1e1e1e;color:#dcdcdc"> m</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">Century)</span>
                </li>
                <li>
                  <span style="background:#1e1e1e;color:#dcdcdc">        </span><span style="background:#ffffb3;color:#000000">@</span><span style="background:#1e1e1e;color:#dcdcdc">Html</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">TextBoxFor(m </span><span style="background:#1e1e1e;color:#b4b4b4">=></span><span style="background:#1e1e1e;color:#dcdcdc"> m</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">Century, </span><span style="background:#1e1e1e;color:#569cd6">new</span><span style="background:#1e1e1e;color:#dcdcdc"> {placeholder </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#d69d85">"In Romna numbers, like XX or XIX"}</span><span style="background:#1e1e1e;color:#dcdcdc">)</span>
                </li>
                <li>
                  <span style="background:#1e1e1e;color:#dcdcdc">    </span><span style="background:#1e1e1e;color:#808080"></</span><span style="background:#1e1e1e;color:#569cd6">p</span><span style="background:#1e1e1e;color:#808080">></span>
                </li>
                <li>
                  <span style="background:#1e1e1e;color:#dcdcdc">    </span>
                </li>
                <li>
                  <span style="background:#1e1e1e;color:#dcdcdc">    </span><span style="background:#1e1e1e;color:#808080"><</span><span style="background:#1e1e1e;color:#569cd6">input</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#9cdcfe">type</span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#c8c8c8">"submit"</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#9cdcfe">value</span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#c8c8c8">"send"</span><span style="background:#1e1e1e;color:#808080">/></span>
                </li>
                <li>
                  <span style="background:#1e1e1e;color:#dcdcdc">}</span>
                </li>
              </ol>
            </div></p>
          </div></p>
        </div>
        
        <p>
          Y luego las acciones correspondientes en el controlador para mostrar la vista y recibir los resultados. Si lo probáis veréis que nuestro RomanNumberModelBinder no se invoca 🙁
        </p>
        
        <p>
          El “culpable” de que no funcione es el ModelBinder por defecto (DefaultModelBinder). Dado que el atributo [ModelBinder] origina <strong>no</strong> puede aplicarse a propiedades, el DefaultModelBinder no tiene en cuenta la posibilidad de que una propiedad en concreto use un model binder distinto. Así pues nos toca reescribir parte del DefaultModelBinder.
        </p>
        
        <p>
          Para reescribir parte del ModelBinder lo más sencillo es heredar de él y redefinir el método que necesitemos. En este caso el método necesario es BindProperty que enlaza una propiedad. Veamos como queda el código:
        </p>
        
        <div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:ff559daf-a03c-452a-a2df-7aff51339978" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
          <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
            <div style="background: #000080; color: #fff; font-family: Verdana, Tahoma, Arial, sans-serif; font-weight: bold; padding: 2px 5px">
              DefaultModelBinderEx
            </div>
            
            <div style="background: #ddd; max-height: 300px; overflow: auto">
              <ol start="1" style="background: #1e1e1e; margin: 0 0 0 2.5em; padding: 0 0 0 5px; white-space: nowrap">
                <li>
                  <span style="background:#1e1e1e;color:#569cd6">public</span><span style="background:#1e1e1e;color:#dcdcdc"> </s pan><span style="background:#1e1e1e;color:#569cd6">class</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#4ec9b0">DefaultModelBinderEx</span><span style="background:#1e1e1e;color:#dcdcdc"> : </span><span style="background:#1e1e1e;color:#4ec9b0">DefaultModelBinder</span></li> 
                  
                  <li>
                    <span style="background:#1e1e1e;color:#dcdcdc">{</span>
                  </li>
                  <li>
                    <span style="background:#1e1e1e;color:#dcdcdc">    </span><span style="background:#1e1e1e;color:#569cd6">protected</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">override</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">void</span><span style="background:#1e1e1e;color:#dcdcdc"> BindProperty(</span><span style="background:#1e1e1e;color:#4ec9b0">ControllerContext</span><span style="background:#1e1e1e;color:#dcdcdc"> controllerContext, </span><span style="background:#1e1e1e;color:#4ec9b0">ModelBindingContext</span><span style="background:#1e1e1e;color:#dcdcdc"> bindingContext,</span>
                  </li>
                  <li>
                    <span style="background:#1e1e1e;color:#dcdcdc">        </span><span style="background:#1e1e1e;color:#4ec9b0">PropertyDescriptor</span><span style="background:#1e1e1e;color:#dcdcdc"> propertyDescriptor)</span>
                  </li>
                  <li>
                    <span style="background:#1e1e1e;color:#dcdcdc">    {</span>
                  </li>
                  <li>
                    &nbsp;
                  </li>
                  <li>
                    <span style="background:#1e1e1e;color:#dcdcdc">        </span><span style="background:#1e1e1e;color:#569cd6">var</span><span style="background:#1e1e1e;color:#dcdcdc"> cmbattr </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> propertyDescriptor</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">Attributes</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">OfType</span><span style="background:#1e1e1e;color:#b4b4b4"><</span><span style="background:#1e1e1e;color:#4ec9b0">CustomModelBinderAttribute</span><span style="background:#1e1e1e;color:#b4b4b4">></span><span style="background:#1e1e1e;color:#dcdcdc">()</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">FirstOrDefault();</span>
                  </li>
                  <li>
                    <span style="background:#1e1e1e;color:#dcdcdc">        </span><span style="background:#1e1e1e;color:#b8d7a3">IModelBinder</span><span style="background:#1e1e1e;color:#dcdcdc"> binder;</span>
                  </li>
                  <li>
                    <span style="background:#1e1e1e;color:#dcdcdc">        </span><span style="background:#1e1e1e;color:#569cd6">if</span><span style="background:#1e1e1e;color:#dcdcdc"> (cmbattr </span><span style="background:#1e1e1e;color:#b4b4b4">!=</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">null</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#b4b4b4">&&</span><span style="background:#1e1e1e;color:#dcdcdc"> (binder </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> cmbattr</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">GetBinder()) </span><span style="background:#1e1e1e;color:#b4b4b4">!=</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">null</span><span style="background:#1e1e1e;color:#dcdcdc">)</span>
                  </li>
                  <li>
                    <span style="background:#1e1e1e;color:#dcdcdc">        {</span>
                  </li>
                  <li>
                    <span style="background:#1e1e1e;color:#dcdcdc">            </span><span style="background:#1e1e1e;color:#569cd6">var</span><span style="background:#1e1e1e;color:#dcdcdc"> subPropertyName </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#4ec9b0">DefaultModelBinder</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">CreateSubPropertyName(bindingContext</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">ModelName, propertyDescriptor</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">Name);</span>
                  </li>
                  <li>
                    <span style="background:#1e1e1e;color:#dcdcdc">            </span><span style="background:#1e1e1e;color:#569cd6">var</span><span style="background:#1e1e1e;color:#dcdcdc"> obj </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> propertyDescriptor</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">GetValue(bindingContext</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">Model);</span>
                  </li>
                  <li>
                    <span style="background:#1e1e1e;color:#dcdcdc">            </span><span style="background:#1e1e1e;color:#569cd6">var</span><span style="background:#1e1e1e;color:#dcdcdc"> modelMetadata </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> bindingContext</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">PropertyMetadata[propertyDescriptor</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">Name];</span>
                  </li>
                  <li>
                    <span style="background:#1e1e1e;color:#dcdcdc">            modelMetadata</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">Model </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> obj;</span>
                  </li>
                  <li>
                    <span style="background:#1e1e1e;color:#dcdcdc">            </span><span style="background:#1e1e1e;color:#569cd6">var</span><span style="background:#1e1e1e;color:#dcdcdc"> bindingContext1 </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">new</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#4ec9b0">ModelBindingContext</span><span style="background:#1e1e1e;color:#dcdcdc">()</span>
                  </li>
                  <li>
                    <span style="background:#1e1e1e;color:#dcdcdc">            {</span>
                  </li>
                  <li>
                    <span style="background:#1e1e1e;color:#dcdcdc">                ModelMetadata </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> modelMetadata,</span>
                  </li>
                  <li>
                    <span style="background:#1e1e1e;color:#dcdcdc">                ModelName </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> subPropertyName,</span>
                  </li>
                  <li>
                    <span style="background:#1e1e1e;color:#dcdcdc">                ModelState </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> bindingContext</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">ModelState,</span>
                  </li>
                  <li>
                    <span style="background:#1e1e1e;color:#dcdcdc">                ValueProvider </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> bindingContext</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">ValueProvider</span>
                  </li>
                  <li>
                    <span style="background:#1e1e1e;color:#dcdcdc">            };</span>
                  </li>
                  <li>
                    <span style="background:#1e1e1e;color:#dcdcdc">            </span><span style="background:#1e1e1e;color:#569cd6">var</span><span style="background:#1e1e1e;color:#dcdcdc"> propertyValue </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;col
or:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">this</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">GetPropertyValue(controllerContext, bindingContext1, propertyDescriptor, binder);</span>
                  </li>
                  <li>
                    <span style="background:#1e1e1e;color:#dcdcdc">            SetProperty(controllerContext, bindingContext, propertyDescriptor, propertyValue);</span>
                  </li>
                  <li>
                    <span style="background:#1e1e1e;color:#dcdcdc">            </span><span style="background:#1e1e1e;color:#569cd6">return</span><span style="background:#1e1e1e;color:#dcdcdc">;</span>
                  </li>
                  <li>
                    <span style="background:#1e1e1e;color:#dcdcdc">        }</span>
                  </li>
                  <li>
                    &nbsp;
                  </li>
                  <li>
                    <span style="background:#1e1e1e;color:#dcdcdc">        </span><span style="background:#1e1e1e;color:#569cd6">base</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">BindProperty(controllerContext, bindingContext, propertyDescriptor);</span>
                  </li>
                  <li>
                    <span style="background:#1e1e1e;color:#dcdcdc">    }</span>
                  </li>
                  <li>
                    <span style="background:#1e1e1e;color:#dcdcdc">}</span>
                  </li></ol></div> </p></div> </p></div> 
                  
                  <p>
                    Vale… el código luce un “pelín” complicado, pero básicamente hace lo siguiente:
                  </p>
                  
                  <ol>
                    <li>
                      Mira si la propiedad a enlazar está decorada con algún atributo que sea [ModelBinder] o derivado (en este caso será precisamente el [PropertyModelBinder]).
                    </li>
                    <li>
                      Si dicha propiedad está enlazada obtiene el binder (mediante el método GetBinder() del atributo) y: <ol>
                        <li>
                          Crea un “subcontexto” de binding, que contenga tan solo esta propiedad.
                        </li>
                        <li>
                          Obtiene el valor a bindea, mediante la llamada a GetPropertyValue, un método definido en el propio DefaultModelBinder, pasándole el subcontexto de binding y el binder a usar.
                        </li>
                        <li>
                          Llama a SetProperty (otro método de DefaultModelBinder) para establecer el valor de la propiedad
                        </li>
                      </ol>
                    </li>
                    
                    <li>
                      En caso de que la propiedad NO esté decorada con ningún atributo [ModelBinder] o derivado, llama a base.BindProperty para procesarla de forma normal.
                    </li>
                  </ol>
                  
                  <blockquote>
                    <p>
                      <strong>Nota:</strong> Este es el MÍNIMO código para que funcione, pero ¡ojo! no estamos tratando todas las casuísticas posibles 😉
                    </p>
                  </blockquote>
                  
                  <p>
                    Por supuesto para que funcione debemos decirle a ASP.NET MVC que el nuevo ModelBinder por defecto es nuestro DefaultModelBinderEx:
                  </p>
                  
                  <div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:d30324d7-84c1-444a-98ef-63ec9396d898" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
                    <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
                      <div style="background: #ddd; max-height: 300px; overflow: auto">
                        <ol start="1" style="background: #1e1e1e; margin: 0 0 0 2em; padding: 0 0 0 5px;">
                          <li>
                            <span style="background:#1e1e1e;color:#4ec9b0">ModelBinders</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">Binders</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">DefaultBinder </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">new</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#4ec9b0">DefaultModelBinderEx</span><span style="background:#1e1e1e;color:#dcdcdc">();</span>
                          </li>
                        </ol>
                      </div></p>
                    </div></p>
                  </div>
                  
                  <p>
                    Y con esto ya tendríamos habilitados nuestros ModelBinders por propiedad!
                  </p>
                  
                  <p>
                    <strong>Una nota final…</strong>
                  </p>
                  
                  <p>
                    ¿Y como tratar todas las casuísticas posibles que antes he dicho que nuestro método BindProperty no trataba y tener así un código “más completo”? Bien, pues la forma más sencilla y rápida es <strong>coger el código fuente de ASP.NET MVC y COPIAR TODO el método BindProperty</strong> del DefaultModelBinder en el DefaultModelBinderEx. Y luego una vez lo habéis echo localizad la línea:
                  </p>
                  
                  <div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:5b393b18-a68f-45d2-9f5f-0a47948d38d3" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
                    <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
                      <div style="background: #ddd; overflow: auto">
                        <ol start="1" style="background: #1e1e1e; margin: 0 0 0 2em; padding: 0 0 0 5px;">
                          <li>
                            <span style="background:#1e1e1e;color:#dcdcdc">IModelBinder binder </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">this</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">Binders</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">GetBinder(propertyDescriptor</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">PropertyType);</span>
                          </li>
                        </ol>
                      </div></p>
                    </div></p>
                  </div>
                  
                  <p>
                    Y modificadla por:
                  </p>
                  
                  <div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:b1e413d0-719a-406d-a020-7a8939dd04e1" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
                    <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
                      <div style="background: #ddd; max-height: 300px; overflow: auto">
                        <ol start="1" style="background: #1e1e1e; margin: 0 0 0 2em; padding: 0 0 0 5px;">
                          <li>
                            <span style="background:#1e1e1e;color:#569cd6">var</span><span style="background:#1e1e1e;color:#dcdcdc"> cmbattr </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> propertyDescriptor</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">Attributes</span><span style="background:#1e1e1e;color:#b4b4b4">.</span>
                          </li>
                          <li>
                            <span style="background:#1e1e1e;color:#dcdcdc">    OfType</span><span style="background:#1e1e1e;color:#b4b4b4"><</span><span style="background:#1e1e1e;color:#4ec9b0">CustomModelBinderAttribute</span><span style="background:#1e1e1e;color:#b4b4b4">></span><span style="background:#1e1e1e;color:#dcdcdc">()</span><span style="background:#1e1e1e;color:#b4b4b4">.</span>
                          </li>
                          <li>
                            <span style="background:#1e1e1e;color:#dcdcdc">    FirstOrDefault();</span>
                          </li>
                          <li>
                            <span style="background:#1e1e1e;color:#b8d7a3">IModelBinder</span><span style="background:#1e1e1e;color:#dcdcdc"> binder </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> cmbattr </span><span style="background:#1e1e1e;color:#b4b4b4">!=</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">null</span><span style="background:#1e1e1e;color:#dcdcdc"> </span>
                          </li>
                          <li>
                            <span style="background:#1e1e1e;color:#dcdcdc">    </span><span style="background:#1e1e1e;color:#b4b4b4">?</span><span style="background:#1e1e1e;color:#dcdcdc"> cmbattr</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">GetBinder() </span>
                          </li>
                          <li>
                            <span style="background:#1e1e1e;color:#dcdcdc">    : </span><span style="background:#1e1e1e;color:#569cd6">this</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">Binders</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">GetBinder(propertyDescriptor</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">PropertyType);</span>
                          </li>
                        </ol>
                      </div></p>
                    </div></p>
                  </div>
                  
                  <p>
                    Y con esto ya tendréis el soporte completo. La verdad es una pena que el DefaultModelBinder no ofrezca un método virtual GetBinder(PropertyDescriptor) ya que si lo tuviese tan solo redefiniendo este método hubiese sido suficiente… Pero en fin… ¡esto es lo que hay!
                  </p>
                  
                  <p>
                    Espero que os haya resultado interesante!
                  </p>

 [1]: http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_074F2361.png