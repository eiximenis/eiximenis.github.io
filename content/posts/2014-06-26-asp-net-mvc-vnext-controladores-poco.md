---
title: ASP.NET MVC vNext – Controladores “POCO”
description: ASP.NET MVC vNext – Controladores “POCO”
author: eiximenis

date: 2014-06-26T10:12:29+00:00
geeks_url: /?p=1672
geeks_visits:
  - 1489
geeks_ms_views:
  - 1606
categories:
  - Uncategorized

---
Una de las novedades que presenta ASP.NET MVC6 (integrada dentro de vNext) es la posibilidad de **que los controladores ya no deban heredar de ninguna clase base**.

De hecho la clase Controller en MVC clásico (MVC5 y anteriores) proporcionaba básicamente dos cosas:

  1. Un conjunto de métodos de para devolver action results (p. ej. el método View() para devolver un <a href="http://msdn.microsoft.com/es-es/library/system.web.mvc.viewresult(v=vs.118).aspx" target="_blank" rel="noopener noreferrer">ViewResult</a> o el método Json para devolver un <a href="http://msdn.microsoft.com/es-es/library/system.web.mvc.jsonresult(v=vs.118).aspx" target="_blank" rel="noopener noreferrer">JsonResult</a>).
  2. Acceso a algunas propiedades para contexto (ControllerContext, ModelState y ViewBag básicamente).

Los métdos para devolver action results no son estríctamente necesarios (aunque ayudan) pero pueden encapsularse en alguna clase aparte y los objetos de contexto pueden añadirse por inyección de dependencias (ASP.NET vNext está montando desde la base usando inyección de dependencias).

Así en MVC6 podemos crear un controlador como el siguiente:

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:214f9200-e90d-41e3-8d0a-9d8400ea94ef" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
    <div style="background: #ddd; max-height: 300px; overflow: auto">
      <ol start="1" style="background: #1d1d1d; margin: 0 0 0 2em; padding: 0 0 0 5px;">
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">public</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">class</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#4ec9b0">HomeController</span>
        </li>
        <li style="background: #111111">
          <span style="background:#1e1e1e;color:#dcdcdc">{</span>
        </li>
        <li>
              <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#57a64a">// GET: /<controller>/</span>
        </li>
        <li style="background: #111111">
              <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">public</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#b8d7a3">IActionResult</span><span style="background:#1e1e1e;color:#dcdcdc"> Index()</span>
        </li>
        <li>
              <span style="background:#1e1e1e;color:#dcdcdc">{</span>
        </li>
        <li style="background: #111111">
                  <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">return</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">null</span><span style="background:#1e1e1e;color:#dcdcdc">;</span>
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

Si (asumiendo la tabla de rutas tradicional) navegamos hacia /Home/Index veremos como se nos invoca dicho método. Por supuesto ahora hemos de ver como crear el action result necesario. P. ej. supongamos que queremos devolver la vista (lo que sería un return View() en un controlador tradicional). Vemos que el constructor del ViewResult nos pide dos parámetros:

[<img title="image" style="border-top: 0px; border-right: 0px; background-image: none; border-bottom: 0px; padding-top: 0px; padding-left: 0px; border-left: 0px; display: inline; padding-right: 0px" border="0" alt="image" src="http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_00D66B87.png" width="504" height="84" />][1]

Como he dicho antes ASP.NET vNext está montado basado en inyección de dependencias así que… deja que el propio framework te inyecte estos parámetros:

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:e825be7b-f0b8-4abb-9ec1-7dff86bcce23" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
    <div style="background: #ddd; max-height: 300px; overflow: auto">
      <ol start="1" style="background: #1d1d1d; margin: 0 0 0 2.5em; padding: 0 0 0 5px;">
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">public</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">class</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#4ec9b0">HomeController</span>
        </li>
        <li style="background: #111111">
          <span style="background:#1e1e1e;color:#dcdcdc">{</span>
        </li>
        <li>
              <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">private</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">readonly</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#b8d7a3">IServiceProvider</span><span style="background:#1e1e1e;color:#dcdcdc"> _serviceProvider;</span>
        </li>
        <li style="background: #111111">
              <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">private</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">readonly</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#b8d7a3">IViewEngine</span><span style="background:#1e1e1e;color:#dcdcdc"> _viewEngine;</span>
        </li>
        <li>
              <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">public</span><span style="background:#1e1e1e;color:#dcdcdc"> HomeController(</span><span style="background:#1e1e1e;color:#b8d7a3">IServiceProvider</span><span style="background:#1e1e1e;color:#dcdcdc"> serviceProvider, </span><span style="background:#1e1e1e;color:#b8d7a3">IViewEngine</span><span style="background:#1e1e1e;color:#dcdcdc"> viewEngine)</span>
        </li>
        <li style="background: #111111">
              <span style="background:#1e1e1e;color:#dcdcdc">{</span>
        </li>
        <li>
                  <span style="background:#1e1e1e;color:#dcdcdc">_serviceProvider </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> serviceProvider;</span>
        </li>
        <li style="background: #111111">
                  <span style="background:#1e1e1e;color:#dcdcdc">_viewEngine </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> viewEngine;</span>
        </li>
        <li>
              <span style="background:#1e1e1e;color:#dcdcdc">}</span>
        </li>
        <li style="background: #111111">
              <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">public</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#b8d7a3">IActionResult</span><span style="background:#1e1e1e;color:#dcdcdc"> Index()</span>
        </li>
        <li>
              <span style="background:#1e1e1e;color:#dcdcdc">{</span>
        </li>
        <li style="background: #111111">
                  <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">return</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">new</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#4ec9b0">ViewResult</span><span style="background:#1e1e1e;color:#dcdcdc
">(_serviceProvider, _viewEngine);</span>
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

Si ahora ejecutas y colocas un breakpoint en el constructor verás que ambos parámetros han sido inicializados por el framework de ASP.NET vNext:

[<img title="image" style="border-top: 0px; border-right: 0px; background-image: none; border-bottom: 0px; padding-top: 0px; padding-left: 0px; border-left: 0px; display: inline; padding-right: 0px" border="0" alt="image" src="http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_6BE779C4.png" width="504" height="57" />][2]

Verás como efectivamente esto devuelve la vista Index.cshtml localizada en Views/Home (exactamente lo mismo que hace return View()).

Pasar un modelo a la vista tampoco es excesivamente complicado:

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:80adafe1-6320-41f4-9e02-abd4d0ee01f7" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
    <div style="background: #ddd; max-height: 300px; overflow: auto">
      <ol start="1" style="background: #1d1d1d; margin: 0 0 0 2.5em; padding: 0 0 0 5px;">
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">public</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">class</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#4ec9b0">HomeController</span>
        </li>
        <li style="background: #111111">
          <span style="background:#1e1e1e;color:#dcdcdc">{</span>
        </li>
        <li>
              <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">private</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">readonly</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#b8d7a3">IServiceProvider</span><span style="background:#1e1e1e;color:#dcdcdc"> _serviceProvider;</span>
        </li>
        <li style="background: #111111">
              <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">private</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">readonly</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#b8d7a3">IViewEngine</span><span style="background:#1e1e1e;color:#dcdcdc"> _viewEngine;</span>
        </li>
        <li>
              <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">private</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">readonly</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#b8d7a3">IModelMetadataProvider</span><span style="background:#1e1e1e;color:#dcdcdc"> _modelMetadataProvider;</span>
        </li>
        <li style="background: #111111">
              <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">public</span><span style="background:#1e1e1e;color:#dcdcdc"> HomeController(</span><span style="background:#1e1e1e;color:#b8d7a3">IServiceProvider</span><span style="background:#1e1e1e;color:#dcdcdc"> serviceProvider, </span><span style="background:#1e1e1e;color:#b8d7a3">IViewEngine</span><span style="background:#1e1e1e;color:#dcdcdc"> viewEngine, </span><span style="background:#1e1e1e;color:#b8d7a3">IModelMetadataProvider</span><span style="background:#1e1e1e;color:#dcdcdc"> modelMetadataProvider)</span>
        </li>
        <li>
              <span style="background:#1e1e1e;color:#dcdcdc">{</span>
        </li>
        <li style="background: #111111">
                  <span style="background:#1e1e1e;color:#dcdcdc">_serviceProvider </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> serviceProvider;</span>
        </li>
        <li>
                  <span style="background:#1e1e1e;color:#dcdcdc">_viewEngine </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> viewEngine;</span>
        </li>
        <li style="background: #111111">
                  <span style="background:#1e1e1e;color:#dcdcdc">_modelMetadataProvider </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> modelMetadataProvider;</span>
        </li>
        <li>
              <span style="background:#1e1e1e;color:#dcdcdc">}</span>
        </li>
        <li style="background: #111111">
              <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">public</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#b8d7a3">IActionResult</span><span style="background:#1e1e1e;color:#dcdcdc"> Index()</span>
        </li>
        <li>
              <span style="background:#1e1e1e;color:#dcdcdc">{</span>
        </li>
        <li style="background: #111111">
                  <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">var</span><span style="background:#1e1e1e;color:#dcdcdc"> viewdata </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">new</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#4ec9b0">ViewDataDictionary</span><span style="background:#1e1e1e;color:#dcdcdc"><</span><span style="background:#1e1e1e;color:#4ec9b0">FooModel</span><span style="background:#1e1e1e;color:#dcdcdc">>(_modelMetadataProvider);</span>
        </li>
        <li>
                  <span style="background:#1e1e1e;color:#dcdcdc">viewdata</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">Model </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">new</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#4ec9b0">FooModel</span><span style="background:#1e1e1e;color:#dcdcdc">();</span>
        </li>
        <li style="background: #111111">
                  <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">return</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">new</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#4ec9b0">ViewResult</span><span style="background:#1e1e1e;color:#dcdcdc">(_serviceProvider, _viewEngine) { ViewData </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> viewdata };</span>
        </li>
        <li>
              <span style="background:#1e1e1e;color:#dcdcdc">}</span>
        </li>
      </ol>
    </div></p>
  </div></p>
</div>

Necesitamos un IModelMetadataProvider (que recibimos también por inyección de dependencias) ya que lo necesitamos para la construcción del ViewDataDictionary que pasamos a la vista.

Para evitar que nuestro controlador POCO deba tomar demasiadas depenencias en el constructor (dependencias que son requeridas básicamente para construir los action results), el equipo de ASP.NET ha creado la interfaz IActionResultHelper. Dicha interfaz contiene métodos para ayudarnos a crear más fácilmente los action results. Por supuesto, en el controlador recibimos un IActionResultHelper por inyección de dependencias. Así podemos modificar nuestro controlador para que quede de la siguiente forma:

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:f3b8fdd6-3d07-4281-b22f-
335a258c6b06" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
  </p> 
  
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
    <div style="background: #ddd; max-height: 300px; overflow: auto">
      <ol start="1" style="background: #1d1d1d; margin: 0 0 0 2.5em; padding: 0 0 0 5px;">
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">public</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">class</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#4ec9b0">HomeController</span>
        </li>
        <li style="background: #111111">
          <span style="background:#1e1e1e;color:#dcdcdc">{</span>
        </li>
        <li>
              <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">private</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">readonly</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#b8d7a3">IActionResultHelper</span><span style="background:#1e1e1e;color:#dcdcdc"> _actionHelper;</span>
        </li>
        <li style="background: #111111">
              <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">private</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">readonly</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#b8d7a3">IModelMetadataProvider</span><span style="background:#1e1e1e;color:#dcdcdc"> _modelMetadataProvider;</span>
        </li>
        <li>
              <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">public</span><span style="background:#1e1e1e;color:#dcdcdc"> HomeController(</span><span style="background:#1e1e1e;color:#b8d7a3">IActionResultHelper</span><span style="background:#1e1e1e;color:#dcdcdc"> actionHelper, </span><span style="background:#1e1e1e;color:#b8d7a3">IModelMetadataProvider</span><span style="background:#1e1e1e;color:#dcdcdc"> modelMetadataProvider)</span>
        </li>
        <li style="background: #111111">
              <span style="background:#1e1e1e;color:#dcdcdc">{</span>
        </li>
        <li>
                  <span style="background:#1e1e1e;color:#dcdcdc">_actionHelper </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> actionHelper;</span>
        </li>
        <li style="background: #111111">
                  <span style="background:#1e1e1e;color:#dcdcdc">_modelMetadataProvider </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> modelMetadataProvider;</span>
        </li>
        <li>
              <span style="background:#1e1e1e;color:#dcdcdc">}</span>
        </li>
        <li style="background: #111111">
              <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">public</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#b8d7a3">IActionResult</span><span style="background:#1e1e1e;color:#dcdcdc"> Index()</span>
        </li>
        <li>
              <span style="background:#1e1e1e;color:#dcdcdc">{</span>
        </li>
        <li style="background: #111111">
                  <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">var</span><span style="background:#1e1e1e;color:#dcdcdc"> viewdata </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">new</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#4ec9b0">ViewDataDictionary</span><span style="background:#1e1e1e;color:#dcdcdc"><</span><span style="background:#1e1e1e;color:#4ec9b0">FooModel</span><span style="background:#1e1e1e;color:#dcdcdc">>(_modelMetadataProvider);</span>
        </li>
        <li>
                  <span style="background:#1e1e1e;color:#dcdcdc">viewdata</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">Model </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">new</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#4ec9b0">FooModel</span><span style="background:#1e1e1e;color:#dcdcdc">();</span>
        </li>
        <li style="background: #111111">
                  <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">return</span><span style="background:#1e1e1e;color:#dcdcdc"> _actionHelper</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">View(</span><span style="background:#1e1e1e;color:#d69d85">"Index"</span><span style="background:#1e1e1e;color:#dcdcdc">, viewdata);</span>
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

Ahora el controlador solo toma una dependencia contra el IActionResultHelper y el IModelMetadataProvider. Las dependencias contra el IServiceProvider y el IViewEngine (que eran solo para crear el ViewResult) son gestionadas ahora por el IActionResultHelper.

¡Y listos! Hemos visto como podemos crear controladores POCO (que no hereden de Controller) y como a través de la inyección de dependencias ¡recibimos las dependencias necesarias de forma automática!

¡Saludos!

 [1]: http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_15C55D49.png
 [2]: http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_179843C1.png