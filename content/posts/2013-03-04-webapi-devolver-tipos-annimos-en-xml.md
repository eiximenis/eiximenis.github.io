---
title: WebApi–Devolver tipos anónimos en XML
description: WebApi–Devolver tipos anónimos en XML
author: eiximenis

date: 2013-03-04T10:49:29+00:00
geeks_url: /?p=1633
geeks_visits:
  - 1488
geeks_ms_views:
  - 1930
categories:
  - Uncategorized

---
Bueno… Honestamente: este post viene a colación de que se estuvo hablando por Linkedin de dedicar hoy (4 de Marzo) un post o algo a WebApi. He de decir que personalmente, no suelo planificar de que escribo. Es decir, tengo varias series abiertas de posts, montones de artículos en borrador y luego voy escribiendo cosas según me van viniendo.

<a href="http://geeks.ms/blogs/etomas/archive/tags/webapi/default.aspx" target="_blank" rel="noopener noreferrer">WebApi es un tema que he tratado bastante</a> en mi blog. Uno de los temas que NO he tratado y que era un buen candidato era la exposición de servicios OData usando WebApi. Pero <a href="https://twitter.com/marc_rubino" target="_blank" rel="noopener noreferrer">Marc Rubiño</a> se me adelantó y <a href="http://mrubino.net/2013/02/25/webapi-odata-queries/" target="_blank" rel="noopener noreferrer">lo contó de manera fenomenal en su blog</a>. Así que pensando temas sobre los que poder hablar al final he optado por este:

[<img title="image" style="border-left-width: 0px; border-right-width: 0px; background-image: none; border-bottom-width: 0px; padding-top: 0px; padding-left: 0px; display: inline; padding-right: 0px; border-top-width: 0px" border="0" alt="image" src="http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_1372D1FD.png" width="504" height="152" />][1]

Este error se produce si haces algo como p. ej:

<div style="font-size: 10pt; font-family: consolas; background: #1e1e1e; color: #dcdcdc">
  <p style="margin: 0px">
    <span style="color: #569cd6">public</span> <span style="color: #569cd6">class</span> <span style="color: #4ec9b0">BeersController</span> : <span style="color: #4ec9b0">ApiController</span>
  </p>
  
  <p style="margin: 0px">
    {
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: #608b4e">// GET api/values</span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: #569cd6">public</span> <span style="color: #b8d7a3">IEnumerable</span> <span style="color: white">Get</span>()
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; {
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #569cd6">return</span> <span style="color: #569cd6">new</span> <span style="color: #4ec9b0">ArrayList</span>() {<span style="color: #569cd6">new</span> {<span style="color: white">Name</span> <span style="color: #b4b4b4">=</span> <span style="color: #d69d85">"ufo"</span>, <span style="color: white">Value</span> <span style="color: #b4b4b4">=</span> <span style="color: #b5cea8">10</span>}, <span style="color: #569cd6">new</span> {<span style="color: white">Age</span> <span style="color: #b4b4b4">=</span> <span style="color: #b5cea8">20</span>, <span style="color: white">Name</span> <span style="color: #b4b4b4">=</span> <span style="color: #d69d85">"Gag"</span>}};
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; }
  </p>
  
  <p style="margin: 0px">
    }
  </p></p>
</div>

Es decir **si devuelves un objeto anónimo** (en mi caso estoy devolviendo una colección de objetos anónimos pero si devuelves uno solo ocurre lo mismo).

La primera opción es pensar que WebApi no soporta objetos anónimos. Pero es falso. WebApi, como tal, no tiene problema alguno en devolver objetos anónimos. Si llamo al mismo servicio pero usando una cabecera Accept application/json para que me devuelva el resultado en json:

[<img title="image" style="border-left-width: 0px; border-right-width: 0px; background-image: none; border-bottom-width: 0px; padding-top: 0px; padding-left: 0px; display: inline; padding-right: 0px; border-top-width: 0px" border="0" alt="image" src="http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_6A1B89FE.png" width="244" height="143" />][2]

En json funciona correctamente. Y es que el problema no está en WebApi en sí… Si no en el serializador de XML que WebApi usa.

Básicamente y resumiendo: El serializador de XML de WebApi **no** soporta objetos anónimos. 

**¿Y hay solución para ello?**

¡Pues claro! Crearte tu propio serializador de XML y luego un media type formatter asociado que lo use.

Lo primero es eliminar el media type formatter de XML que viene por defecto. Para ello, en Application_Start:

<div style="font-size: 10pt; font-family: consolas; background: #1e1e1e; color: #dcdcdc">
  <p style="margin: 0px">
    <span style="color: white">config</span><span style="color: #b4b4b4">.</span><span style="color: white">Formatters</span><span style="color: #b4b4b4">.</span><span style="color: white">Remove</span>(<span style="color: white">config</span><span style="color: #b4b4b4">.</span><span style="color: white">Formatters</span><span style="color: #b4b4b4">.</span><span style="color: white">XmlFormatter</span>);
  </p></p>
</div>

Con esto eliminamos el media type formatter de XML y por lo tanto WebApi deja de dar soporte a XML. Este media type formatter que viene por defecto utiliza o bien DataContractSerializer o bien XmlSerializer para serializar los datos y ninguno de los dos tiene soporte para tipos anónimos. Lo primero es crearnos un serializador de XML propio que tenga soporte para tipos anónimos.

Yo he usado uno (muy sencillo y nada “personalizable”) que he encontrado en [http://stackoverflow.com/questions/2404685/can-i-serialize-anonymous-types-as-xml][3]. Simplemente lo he completado para que tenga soporte para colecciones de elementos. El código del serializador es:

<div style="font-size: 10pt; font-family: consolas; background: #1e1e1e; color: #dcdcdc">
  <p style="margin: 0px">
    <span style="color: #569cd6">public</span> <span style="color: #569cd6">static</span> <span style="color: #569cd6">class</span> <span style="color: #4ec9b0">CustomXmlSerializer</span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; {
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #569cd6">private</span> <span style="color: #569cd6">static</span> <span style="color: #569cd6">readonly</span> <span style="color: #4ec9b0">Type</span>[] <span style="color: white">WriteTypes</span> <span style="color: #b4b4b4">=</span> <span style="color: #569cd6">new</span>[] {
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #569cd6">typeof</span>(<span style="color: #569cd6">string</span>), <span style="color: #569cd6">typeof</span>(<span style="color: #4ec9b0">DateTime</span>), <span style="color: #569cd6">typeof</span>(<span style="color: #4ec9b0">Enum</span>),
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #569cd6">typeof</span>(<span style="color: #569cd6">decimal</span>), <span style="color: #569cd6">typeof</span>(<span style="color: #4ec9b0">Guid</span>) };
  </p>
  
  <p style="margin: 0px">
    &#160;
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #569cd6">public</span> <span style="color: #569cd6">static</span> <span style="color: #569cd6">bool</span> <span style="color: white">IsEnumerable</span>(<span style="color: #569cd6">this</span> <span style="color: #4ec9b0">Type</span> <span style="color: white">type</span>)
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; {
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #569cd6">return</span> <span style="color: white">type</span><span style="color: #b4b4b4">.</span><span style="color: white">GetInterface</span>(<span style="color: #d69d85">"IEnumerable"</sp an>) <span style="color: #b4b4b4">!=</span> <span style="color: #569cd6">null</span>;</p> 
    
    <p style="margin: 0px">
      &#160;&#160;&#160;&#160;&#160;&#160;&#160; }
    </p>
    
    <p style="margin: 0px">
      &#160;
    </p>
    
    <p style="margin: 0px">
      &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #569cd6">public</span> <span style="color: #569cd6">static</span> <span style="color: #569cd6">bool</span> <span style="color: white">IsSimpleType</span>(<span style="color: #569cd6">this</span> <span style="color: #4ec9b0">Type</span> <span style="color: white">type</span>)
    </p>
    
    <p style="margin: 0px">
      &#160;&#160;&#160;&#160;&#160;&#160;&#160; {
    </p>
    
    <p style="margin: 0px">
      &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #569cd6">return</span> <span style="color: white">type</span><span style="color: #b4b4b4">.</span><span style="color: white">IsPrimitive</span> <span style="color: #b4b4b4">||</span> <span style="color: white">WriteTypes</span><span style="color: #b4b4b4">.</span><span style="color: white">Contains</span>(<span style="color: white">type</span>);
    </p>
    
    <p style="margin: 0px">
      &#160;&#160;&#160;&#160;&#160;&#160;&#160; }
    </p>
    
    <p style="margin: 0px">
      &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #569cd6">public</span> <span style="color: #569cd6">static</span> <span style="color: #4ec9b0">XElement</span> <span style="color: white">ToXml</span>(<span style="color: #569cd6">this</span> <span style="color: #569cd6">object</span> <span style="color: white">input</span>)
    </p>
    
    <p style="margin: 0px">
      &#160;&#160;&#160;&#160;&#160;&#160;&#160; {
    </p>
    
    <p style="margin: 0px">
      &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #569cd6">return</span> <span style="color: white">input</span><span style="color: #b4b4b4">.</span><span style="color: white">ToXml</span>(<span style="color: #569cd6">null</span>);
    </p>
    
    <p style="margin: 0px">
      &#160;&#160;&#160;&#160;&#160;&#160;&#160; }
    </p>
    
    <p style="margin: 0px">
      &#160;
    </p>
    
    <p style="margin: 0px">
      &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #569cd6">public</span> <span style="color: #569cd6">static</span> <span style="color: #4ec9b0">XElement</span> <span style="color: white">ToXml</span>(<span style="color: #569cd6">this</span> <span style="color: #569cd6">object</span> <span style="color: white">input</span>, <span style="color: #569cd6">string</span> <span style="color: white">element</span>)
    </p>
    
    <p style="margin: 0px">
      &#160;&#160;&#160;&#160;&#160;&#160;&#160; {
    </p>
    
    <p style="margin: 0px">
      &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #569cd6">if</span> (<span style="color: white">input</span> <span style="color: #b4b4b4">==</span> <span style="color: #569cd6">null</span>)
    </p>
    
    <p style="margin: 0px">
      &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #569cd6">return</span> <span style="color: #569cd6">null</span>;
    </p>
    
    <p style="margin: 0px">
      &#160;
    </p>
    
    <p style="margin: 0px">
      &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #569cd6">if</span> (<span style="color: #569cd6">string</span><span style="color: #b4b4b4">.</span><span style="color: white">IsNullOrEmpty</span>(<span style="color: white">element</span>))
    </p>
    
    <p style="margin: 0px">
      &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: white">element</span> <span style="color: #b4b4b4">=</span> <span style="color: #d69d85">"object"</span>;
    </p>
    
    <p style="margin: 0px">
      &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: white">element</span> <span style="color: #b4b4b4">=</span> <span style="color: #4ec9b0">XmlConvert</span><span style="color: #b4b4b4">.</span><span style="color: white">EncodeName</span>(<span style="color: white">element</span>);
    </p>
    
    <p style="margin: 0px">
      &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #569cd6">var</span> <span style="color: white">ret</span> <span style="color: #b4b4b4">=</span> <span style="color: #569cd6">new</span> <span style="color: #4ec9b0">XElement</span>(<span style="color: white">element</span>);
    </p>
    
    <p style="margin: 0px">
      &#160;
    </p>
    
    <p style="margin: 0px">
      &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #569cd6">if</span> (<span style="color: white">input</span> <span style="color: #b4b4b4">!=</span> <span style="color: #569cd6">null</span>)
    </p>
    
    <p style="margin: 0px">
      &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; {
    </p>
    
    <p style="margin: 0px">
      &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #569cd6">var</span> <span style="color: white">type</span> <span style="color: #b4b4b4">=</span> <span style="color: white">input</span><span style="color: #b4b4b4">.</span><span style="color: white">GetType</span>();
    </p>
    
    <p style="margin: 0px">
      &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #569cd6">var</span> <span style="color: white">props</span> <span style="color: #b4b4b4">=</span> <span style="color: white">type</span><span style="color: #b4b4b4">.</span><span style="color: white">GetProperties</span>();
    </p>
    
    <p style="margin: 0px">
      &#160;
    </p>
    
    <p style="margin: 0px">
      &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #569cd6">var</span> <span style="color: white">elements</span> <span style="color: #b4b4b4">=</span> <span style="color: #569cd6">from</span> <span style="color: white">prop</span> <span style="color: #569cd6">in</span> <span style="color: white">props</span>
    </p>
    
    <p style="margin: 0px">
      &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #569cd6">let</span> <span style="color: white">name</span> <span style="color: #b4b4b4">=</span> <span style="color: #4ec9b0">XmlConvert</span><span style="color: #b4b4b4">.</span><span style="color: white">EncodeName</span>(<span style="color: white">prop</span><span style="color: #b4b4b4">.</span><span style="color: white">Name</span>)
    </p>
    
    <p style="margin: 0px">
      &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #569cd6">let</span> <span style="color: white">val</span> <span style="color: #b4b4b4">=</span> <span style="color: white">prop</span><span style="color: #b4b4b4">.</span><span style="color: white">GetValue</span>(<span style="color: white">input</span>, <span style="color: #569cd6">null</span>)
    </p>
    
    <p style="margin: 0px">
      &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #569cd6">let</span> <span style="color: white">value</span> <span style="color: #b4b4b4">=</span> <span style="color: white">prop</span><span style="color: #b4b4b4">.</span><span style="color: white">PropertyType</span><span style="color: #b4b4b4">.</span><span style="color: white">IsSimpleType</span>()
    </p>
    
    <p style="margin: 0px">
      &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #b4b4b4">?</span> <span style="color: #569cd6">new</span> <span style="color: #4ec9b0">XElement</span>(<span style="color: white">name</span>, <span style="color: white">val</span>)
    </p>
    
    <p style="margin: 0px">
      &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160<br /> ;&#160; : <span style="color: white">val</span><span style="color: #b4b4b4">.</span><span style="color: white">ToXml</span>(<span style="color: white">name</span>)
    </p>
    
    <p style="margin: 0px">
      &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #569cd6">where</span> <span style="color: white">value</span> <span style="color: #b4b4b4">!=</span> <span style="color: #569cd6">null</span>
    </p>
    
    <p style="margin: 0px">
      &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #569cd6">select</span> <span style="color: white">value</span>;
    </p>
    
    <p style="margin: 0px">
      &#160;
    </p>
    
    <p style="margin: 0px">
      &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: white">ret</span><span style="color: #b4b4b4">.</span><span style="color: white">Add</span>(<span style="color: white">elements</span>);
    </p>
    
    <p style="margin: 0px">
      &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; }
    </p>
    
    <p style="margin: 0px">
      &#160;
    </p>
    
    <p style="margin: 0px">
      &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #569cd6">return</span> <span style="color: white">ret</span>;
    </p>
    
    <p style="margin: 0px">
      &#160;&#160;&#160;&#160;&#160;&#160;&#160; }
    </p>
    
    <p style="margin: 0px">
      &#160;
    </p>
    
    <p style="margin: 0px">
      &#160;&#160;&#160; }
    </p></p></div> 
    
    <p>
      El código es bastante simple no? Básicamente itera sobre todas las propiedades del elemento y usa Linq to XML para convertirlas a XML.
    </p>
    
    <p>
      Una vez tenemos un serializador que nos soporta tipos anónimos, ha llegado el momento de crear un media type formatter que lo use. Recuerda que los media type formatters son las clases que usa WebApi para leer/escribir desde/el stream de la petición/respuesta.
    </p>
    
    <p>
      En nuestro caso nuestro media type formatter soportará escritura solamente. Eso signfica que podremos devolver tipos anónimos en XML pero NO soportaremos la lectura de XML en las peticiones (es decir no aceptaremos peticiones cuyo cuerpo sea un XML (al menos no con este media type formatter)).
    </p>
    
    <p>
      El código es realmente simple:
    </p>
    
    <div style="font-size: 10pt; font-family: consolas; background: #1e1e1e; color: #dcdcdc">
      <p style="margin: 0px">
        <span style="color: #569cd6">public</span> <span style="color: #569cd6">class</span> <span style="color: #4ec9b0">CustomXmlFormatter</span> : <span style="color: #4ec9b0">BufferedMediaTypeFormatter</span>
      </p>
      
      <p style="margin: 0px">
        {
      </p>
      
      <p style="margin: 0px">
        &#160;&#160;&#160; <span style="color: #569cd6">public</span> <span style="color: white">CustomXmlFormatter</span>()
      </p>
      
      <p style="margin: 0px">
        &#160;&#160;&#160; {
      </p>
      
      <p style="margin: 0px">
        &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: white">SupportedMediaTypes</span><span style="color: #b4b4b4">.</span><span style="color: white">Add</span>(<span style="color: #569cd6">new</span> <span style="color: #4ec9b0">MediaTypeHeaderValue</span>(<span style="color: #d69d85">"application/xml"</span>));
      </p>
      
      <p style="margin: 0px">
        &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: white">SupportedMediaTypes</span><span style="color: #b4b4b4">.</span><span style="color: white">Add</span>(<span style="color: #569cd6">new</span> <span style="color: #4ec9b0">MediaTypeHeaderValue</span>(<span style="color: #d69d85">"text/xml"</span>));
      </p>
      
      <p style="margin: 0px">
        &#160;&#160;&#160; }
      </p>
      
      <p style="margin: 0px">
        &#160;
      </p>
      
      <p style="margin: 0px">
        &#160;&#160;&#160; <span style="color: #569cd6">public</span> <span style="color: #569cd6">override</span> <span style="color: #569cd6">bool</span> <span style="color: white">CanReadType</span>(<span style="color: #4ec9b0">Type</span> <span style="color: white">type</span>)
      </p>
      
      <p style="margin: 0px">
        &#160;&#160;&#160; {
      </p>
      
      <p style="margin: 0px">
        &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #569cd6">return</span> <span style="color: #569cd6">false</span>;
      </p>
      
      <p style="margin: 0px">
        &#160;&#160;&#160; }
      </p>
      
      <p style="margin: 0px">
        &#160;
      </p>
      
      <p style="margin: 0px">
        &#160;&#160;&#160; <span style="color: #569cd6">public</span> <span style="color: #569cd6">override</span> <span style="color: #569cd6">bool</span> <span style="color: white">CanWriteType</span>(<span style="color: #4ec9b0">Type</span> <span style="color: white">type</span>)
      </p>
      
      <p style="margin: 0px">
        &#160;&#160;&#160; {
      </p>
      
      <p style="margin: 0px">
        &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #569cd6">return</span> <span style="color: #569cd6">true</span>;
      </p>
      
      <p style="margin: 0px">
        &#160;&#160;&#160; }
      </p>
      
      <p style="margin: 0px">
        &#160;
      </p>
      
      <p style="margin: 0px">
        &#160;&#160;&#160; <span style="color: #569cd6">public</span> <span style="color: #569cd6">override</span> <span style="color: #569cd6">void</span> <span style="color: white">WriteToStream</span>(<span style="color: #4ec9b0">Type</span> <span style="color: white">type</span>, <span style="color: #569cd6">object</span> <span style="color: white">value</span>, <span style="color: #4ec9b0">Stream</span> <span style="color: white">writeStream</span>, <span style="color: #4ec9b0">HttpContent</span> <span style="color: white">content</span>)
      </p>
      
      <p style="margin: 0px">
        &#160;&#160;&#160; {
      </p>
      
      <p style="margin: 0px">
        &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #569cd6">using</span> (<span style="color: #569cd6">var</span> <span style="color: white">writer</span> <span style="color: #b4b4b4">=</span> <span style="color: #569cd6">new</span> <span style="color: #4ec9b0">StreamWriter</span>(<span style="color: white">writeStream</span>))
      </p>
      
      <p style="margin: 0px">
        &#160;&#160;&#160;&#160;&#160;&#160;&#160; {
      </p>
      
      <p style="margin: 0px">
        &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: white">WriteAny</span>(<span style="color: white">writer</span>, <span style="color: white">type</span>, <span style="color: white">value</span>);
      </p>
      
      <p style="margin: 0px">
        &#160;&#160;&#160;&#160;&#160;&#160;&#160; }
      </p>
      
      <p style="margin: 0px">
        &#160;
      </p>
      
      <p style="margin: 0px">
        &#160;&#160;&#160; }
      </p>
      
      <p style="margin: 0px">
        &#160;
      </p>
      
      <p style="margin: 0px">
        &#160;&#160;&#160; <span style="color: #569cd6">private</span> <span style="color: #569cd6">void</span> <span style="color: white">WriteAny</span>(<span style="color: #4ec9b0">StreamWriter</span> <span style="color: white">writer</span>, <span style="color: #4ec9b0">Type</span> <span style="color: white">type</span>, <span style="color: #569cd6">object</span> <span style="color: white">value</span>)
      </p>
      
      <p style="margin: 0px">
        &#160;&#160;&#160; {
      </p>
      
      <p style="margin: 0px">
        &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #569cd6">if</span> (<span style="color: white">type</span><span style="color: #b4b4b4">.</span><span style="color: white">IsEnumerable</span>()) <span style="color: white">WriteCollection</span>(<span style="color: white">writer</span>, <span style="color: white">type</span>, <span style="color: white">value</span>);
      </p>
      
      <p style="margin: 0px">
        &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #569cd6">else</span> <span style="color: white">WriteObject</span>(<span style="color: white">writer</span>, <span style="color: white">type</span>, <span style="color: white">value</span>);
      </p>
      
      <p style="margin: 0px">
        &#160;&#160;&#160; }
      </p>
      
      <p style="margin: 0px">
        &#160;
      </p>
      
      <p style="margin: 0px">
        &#160;&#160;&#160; <span style="color: #569cd6">private</span> <span style="color: #569cd6">void</span> <span style="color: white">WriteObject</span>(<span style="color: #4ec9b0">StreamWriter</span> <span style="color: white">writer</span>, <span style="color: #4ec9b0">Type</span> <span style="color: white">type</span>, <span style="color: #569cd6">object</span> <span style="color: white">va<br /> lue</span>)
      </p>
      
      <p style="margin: 0px">
        &#160;&#160;&#160; {
      </p>
      
      <p style="margin: 0px">
        &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #569cd6">var</span> <span style="color: white">xml</span> <span style="color: #b4b4b4">=</span> <span style="color: white">value</span><span style="color: #b4b4b4">.</span><span style="color: white">ToXml</span>();
      </p>
      
      <p style="margin: 0px">
        &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: white">writer</span><span style="color: #b4b4b4">.</span><span style="color: white">Write</span>(<span style="color: white">xml</span><span style="color: #b4b4b4">.</span><span style="color: white">ToString</span>());
      </p>
      
      <p style="margin: 0px">
        &#160;&#160;&#160; }
      </p>
      
      <p style="margin: 0px">
        &#160;
      </p>
      
      <p style="margin: 0px">
        &#160;&#160;&#160; <span style="color: #569cd6">private</span> <span style="color: #569cd6">void</span> <span style="color: white">WriteCollection</span>(<span style="color: #4ec9b0">StreamWriter</span> <span style="color: white">writer</span>, <span style="color: #4ec9b0">Type</span> <span style="color: white">type</span>, <span style="color: #569cd6">object</span> <span style="color: white">value</span>)
      </p>
      
      <p style="margin: 0px">
        &#160;&#160;&#160; {
      </p>
      
      <p style="margin: 0px">
        &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #569cd6">var</span> <span style="color: white">collection</span> <span style="color: #b4b4b4">=</span> <span style="color: white">value</span> <span style="color: #569cd6">as</span> <span style="color: #b8d7a3">IEnumerable</span>;
      </p>
      
      <p style="margin: 0px">
        &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: white">writer</span><span style="color: #b4b4b4">.</span><span style="color: white">Write</span>(<span style="color: #d69d85">"<collection>"</span>);
      </p>
      
      <p style="margin: 0px">
        &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #569cd6">foreach</span> (<span style="color: #569cd6">var</span> <span style="color: white">item</span> <span style="color: #569cd6">in</span> <span style="color: white">collection</span>)
      </p>
      
      <p style="margin: 0px">
        &#160;&#160;&#160;&#160;&#160;&#160;&#160; {
      </p>
      
      <p style="margin: 0px">
        &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #569cd6">if</span> (<span style="color: white">item</span> <span style="color: #b4b4b4">!=</span> <span style="color: #569cd6">null</span>)
      </p>
      
      <p style="margin: 0px">
        &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; {
      </p>
      
      <p style="margin: 0px">
        &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: white">WriteAny</span>(<span style="color: white">writer</span>, <span style="color: white">item</span><span style="color: #b4b4b4">.</span><span style="color: white">GetType</span>(), <span style="color: white">item</span>);
      </p>
      
      <p style="margin: 0px">
        &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; }
      </p>
      
      <p style="margin: 0px">
        &#160;&#160;&#160;&#160;&#160;&#160;&#160; }
      </p>
      
      <p style="margin: 0px">
        &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: white">writer</span><span style="color: #b4b4b4">.</span><span style="color: white">Write</span>(<span style="color: #d69d85">"</collection>"</span>);
      </p>
      
      <p style="margin: 0px">
        &#160;&#160;&#160; }
      </p>
      
      <p style="margin: 0px">
        }
      </p></p>
    </div>
    
    <p>
      ¡Listos! Simplemente redefinimos los métodos CanWriteType para indicar que podemos serializar cualquier tipo y CanReadType para indicar que NO podemos leer ninguno.
    </p>
    
    <p>
      Luego redefinimos el método WriteToStream y usamos los métodos extensores definidos en nuestro serializador de xml propio para obtener el XML y escribirlo en el stream de salida.
    </p>
    
    <p>
      Ahora tan solo nos queda registrar este media type formatter para que WebApi lo use:
    </p>
    
    <div style="font-size: 10pt; font-family: consolas; background: #1e1e1e; color: #dcdcdc">
      <p style="margin: 0px">
        <span style="color: white">config</span><span style="color: #b4b4b4">.</span><span style="color: white">Formatters</span><span style="color: #b4b4b4">.</span><span style="color: white">Add</span>(<span style="color: #569cd6">new</span> <span style="color: #4ec9b0">CustomXmlFormatter</span>());
      </p></p>
    </div>
    
    <p>
      Y ahora ya estamos. Si ejecutamos de nuevo la llamada a nuestro servicio con una cabecera Accept que prefiera XML:
    </p>
    
    <p>
      <a href="http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_68D6F11F.png"><img title="image" style="border-top: 0px; border-right: 0px; background-image: none; border-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; border-left: 0px; display: inline; padding-right: 0px" border="0" alt="image" src="http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_4E9687FB.png" width="244" height="217" /></a>
    </p>
    
    <p>
      <strong>Resumen</strong>
    </p>
    
    <ul>
      <li>
        WebApi usa el concepto de media type formatter para decidir que clase usar para serializar los datos devueltos por los controladores (también para leer los datos enviados a los controladores)
      </li>
      <li>
        Un media type formatter básicamente se asocia a uno o varios tipos mime (eso se hace en el constructor de cada media type formatter)
      </li>
      <li>
        El media type formatter por defecto que viene en WebApi para serializar XML usa DataContractSerializer o XmlSerializer y NO tiene soporte para tipos anónimos
      </li>
      <li>
        Para habilitar el soporte de tipos anónimos debemos pues crearnos un media type formatter propio que use un serializador de XML que admita tipos anónimos.
      </li>
      <li>
        Para JSON no es necesario hacer nada de esto: los tipos anónimos están soportados de serie.
      </li>
    </ul>
    
    <p>
      Espero que te haya resultado interesante.
    </p>
    
    <p>
      Saludos!
    </p>

 [1]: http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_0895F0B5.png
 [2]: http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_6440E665.png
 [3]: http://stackoverflow.com/questions/2404685/can-i-serialize-anonymous-types-as-xml "http://stackoverflow.com/questions/2404685/can-i-serialize-anonymous-types-as-xml"