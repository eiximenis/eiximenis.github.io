---
title: 'ASP.NET MVC: Custom Model Binders vs ValueProviders y un ejemplo con JSON…'
author: eiximenis

date: 2010-06-01T12:49:00+00:00
geeks_url: /?p=1514
geeks_visits:
  - 3553
geeks_ms_views:
  - 1167
categories:
  - Uncategorized

---
Hola a todos!

Este post es el cuarto sobre la serie que podríamos llamar &ldquo;_el interior de ASP.NET MCV&rdquo;_ y viene a ser un resumen de los tres anteriores. Los anteriores posts fueron:

  * <a target="_blank" href="/blogs/etomas/archive/2010/05/07/asp-net-mvc-valueproviders.aspx" rel="noopener noreferrer">Value Providers</a>
  * <a target="_blank" href="/blogs/etomas/archive/2010/05/10/asp-net-mvc-el-defaultmodelbinder.aspx" rel="noopener noreferrer">El DefaultModelBinder</a>
  * <a target="_blank" href="/blogs/etomas/archive/2010/05/12/asp-net-mvc-custom-model-binders.aspx" rel="noopener noreferrer">Custom ModelBinders</a>

En este vamos a ver como podemos implementar una característica que no viene de serie en ASP.NET MVC y que es casi imprescindible si estáis implementando una API REST usando MVC: que los controladores MVC sean capaces de procesar peticiones POST que vengan con datos JSON.

El código de la vista que vamos a usar para probar que todo funciona es una vista con un solo botón con id=&rdquo;btnSend&rdquo; y el siguiente código javascript:

<div id="codeSnippetWrapper" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; width: 97.5%; font-family: 'Courier New', courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; cursor: text; border: silver 1px solid; padding: 4px;">
  <pre id="codeSnippet" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 0em; width: 100%; font-family: 'Courier New', courier, monospace; direction: ltr; color: black; font-size: 8pt; overflow: visible; border-style: none; padding: 0px;">&lt;script type=<span style="color: #006080">"text/javascript"</span>&gt;<br />    $(document).ready(<span style="color: #0000ff">function</span> () {<br />        $(<span style="color: #006080">"#btnSend"</span>).click(<span style="color: #0000ff">function</span> () {<br />            <span style="color: #0000ff">var</span> data = {<br />                Name: <span style="color: #006080">'edu'</span>,<br />                Urls: [<span style="color: #006080">'http://twitter.com/eiximenis'</span>, <span style="color: #006080">'http://geeks.ms/blogs/etomas'</span>]<br />            };<br />            $.ajax({<br />                type: <span style="color: #006080">"POST"</span>, data: $.toJSON(data), contentType: <span style="color: #006080">"application/json; charset=utf-8"</span>, <br />                dataType: <span style="color: #006080">"json"</span>, url: <span style="color: #006080">"/Home/Index"</span><br />        });<br />    });<br />});<br />&lt;/script&gt;</pre>
  
  <p>
    </div> 
    
    <p>
      Cuando se pulse en el botón se serializará el objeto &ldquo;data&rdquo; y se enviará via POST a la url /Home/Index. Evidentemente el controlador Home tiene una acción Index que espera datos via POST:
    </p>
    
    <div id="codeSnippetWrapper" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; width: 97.5%; font-family: 'Courier New', courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; cursor: text; border: silver 1px solid; padding: 4px;">
      <pre id="codeSnippet" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 0em; width: 100%; font-family: 'Courier New', courier, monospace; direction: ltr; color: black; font-size: 8pt; overflow: visible; border-style: none; padding: 0px;">[HttpPost]<br /><span style="color: #0000ff">public</span> ActionResult Index(UserData data)<br />{<br />    <span style="color: #0000ff">return</span> View();<br />}</pre>
      
      <p>
        </div> 
        
        <p>
          Y UserData es la clase del modelo que deberá contener los datos de la petición:
        </p>
        
        <div id="codeSnippetWrapper" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; width: 97.5%; font-family: 'Courier New', courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; cursor: text; border: silver 1px solid; padding: 4px;">
          <pre id="codeSnippet" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 0em; width: 100%; font-family: 'Courier New', courier, monospace; direction: ltr; color: black; font-size: 8pt; overflow: visible; border-style: none; padding: 0px;"><span style="color: #0000ff">public</span> <span style="color: #0000ff">class</span> UserData<br />{<br />    <span style="color: #0000ff">public</span> <span style="color: #0000ff">string</span> Name { get; set; }<br />    <span style="color: #0000ff">public</span> IEnumerable&lt;<span style="color: #0000ff">string</span>&gt; Urls { get; set; }<br />    <span style="color: #0000ff">public</span> <span style="color: #0000ff">int</span> Id { get; set; }<br />}</pre>
          
          <p>
            </div> 
            
            <p>
              Estamos listos para empezar... 🙂
            </p>
            
            <p>
              <strong>Usando un ModelBinder propio...</strong>
            </p>
            
            <p>
              La verdad es que usar un ModelBinder es casi, casi trivial: basta con derivar de DefaultModelBinder y comprobar si la Request tiene el content-type de application/json, y si es el caso usar JavascriptSerializer para deserializar la cadena json:
            </p>
            
            <div id="codeSnippetWrapper" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; width: 97.5%; font-family: 'Courier New', courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; cursor: text; border: silver 1px solid; padding: 4px;">
              <pre id="codeSnippet" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 0em; width: 100%; font-family: 'Courier New', courier, monospace; direction: ltr; color: black; font-size: 8pt; overflow: visible; border-style: none; padding: 0px;"><span style="color: #0000ff">public</span> <span style="color: #0000ff">class</span> JsonModelBinder : DefaultModelBinder<br />{<br />    <span style="color: #0000ff">public</span> <span style="color: #0000ff">override</span> <span style="color: #0000ff">object</span> BindModel(ControllerContext controllerContext, ModelBindingContext bindingContext)<br />    {<br />        <span style="color: #0000ff">if</span> (!IsJSONRequest(controllerContext))<br />        {<br />            <span style="color: #0000ff">return</span> <span style="color: #0000ff">base</span>.BindModel(controllerContext, bindingContext);<br />        }<br />        var request = controllerContext.HttpContext.Request;<br />        var jsonStringData = <span style="color: #0000ff">new</span> StreamReader(request.InputStream).ReadToEnd();<br />        <span style="color: #0000ff">return</span> <span style="color: #0000ff">new</span> JavaScriptSerializer()<br />            .Deserialize(jsonStringData, bindingContext.ModelMetadata.ModelType);<br />    }<br />    <span style="color: #0000ff">private</span> <span style="color: #0000ff">static</span> <span style="color: #0000ff">bool</span> IsJSONRequest(ControllerContext controllerContext)<br />    {<br />        var contentType = controllerContext.HttpContext.Request.ContentType;<br />        <span style="color: #0000ff">return</span> contentType.Contains(<span style="color: #006080">"application/json"</span>);<br />    }<br />}</pre>
              
              <p>
                </div> 
                
                <p>
                  Recordad de registrar este model binder como el model binder por defecto, colocando lo siguiente en el Application_Start:
                </p>
                
                <div id="codeSnippetWrapper" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; width: 97.5%; font-family: 'Courier New', courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; cursor: text; border: silver 1px solid; padding: 4px;">
                  <pre id="codeSnippet" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 0em; width: 100%; font-family: 'Courier New', courier, monospace; direction: ltr; color: black; font-size: 8pt; overflow: visible; border-style: none; padding: 0px;">ModelBinders.Binders.DefaultBinder = <span style="color: #0000ff">new</span> JsonModelBinder();</pre>
                  
                  <p>
                    </div> 
                    
                    <p>
                      Y listos... funciona! O más bien dicho.... <em>parece que funciona</em>...
                    </p>
                    
                    <p>
                      P.ej... <strong>&iexcl;hemos perdido las validaciones! </strong>P.ej. si añadís la siguiente validación en UserData:
                    </p>
                    
                    <div id="codeSnippetWrapper" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; width: 97.5%; font-family: 'Courier New', courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; cursor: text; border: silver 1px solid; padding: 4px;">
                      <pre id="codeSnippet" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 0em; width: 100%; font-family: 'Courier New', courier, monospace; direction: ltr; color: black; font-size: 8pt; overflow: visible; border-style: none; padding: 0px;">[Range(1,10)]<br /><span style="color: #0000ff">public</span> <span style="color: #0000ff">int</span> Id { get; set; }</pre>
                      
                      <p>
                        </div> 
                        
                        <p>
                          Y ejecutáis de nuevo vereis que el Id es 0 y el modelo sigue siendo válido (ModelState.IsValid vale true). ¿Y eso porque? Pues recordad que es el Model Binder quien las aplica, y nuestro Model Binder simplemente <em>está obviando todas las validaciones</em>.
                        </p>
                        
                        <p>
                          Pero no sólo esto... si modificamos la vista para que en lugar de enviar el post a /Home/Index lo envíe a /Home/Index?Id=2 cuando recibamos el objeto UserData, su propiedad Id seguirá valiendo 0. Es decir <em>hemos perdido la capacidad de ASP.NET MVC de crear modelos combinando elementos de la request que estén en POST y en querystring</em>.
                        </p>
                        
                        <p>
                          La razón de todo esto es simple: Un Model Binder <strong>no es la mejor manera para realizar esta tarea</strong>. Os acordáis cuando hablamos de los Value Providers? Comentamos que su responsabilidad era <em>recoger los datos de la request para después pasárselos a los model binders</em> que los usarán para crear los modelos.
                        </p>
                        
                        <p>
                          Aquí precisamente tenemos un caso clarísimo de uso de un Value Provider: Debemos inspeccionar los datos de la request y decodificarlos, pero no tenemos ninguna necesidad de redefinir las reglas de creación del modelo.
                        </p>
                        
                        <p>
                          <strong>Usando un Value Provider</strong>
                        </p>
                        
                        <p>
                          Si recordáis el post sobre los value providers, no damos de alta value providers directamente en el sistema sinó <em>factorías de value providers</em>. El siguiente código da de alta una factoría de value providers que leen datos JSON:
                        </p>
                        
                        <div id="codeSnippetWrapper" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; width: 97.5%; font-family: 'Courier New', courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; cursor: text; border: silver 1px solid; padding: 4px;">
                          <pre id="codeSnippet" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 0em; width: 100%; font-family: 'Courier New', courier, monospace; direction: ltr; color: black; font-size: 8pt; overflow: visible; border-style: none; padding: 0px;"><span style="color: #0000ff">public</span> <span style="color: #0000ff">class</span> JsonValueProviderFactory : ValueProviderFactory<br />{<br /><br />    <span style="color: #0000ff">public</span> <span style="color: #0000ff">override</span> IValueProvider GetValueProvider(ControllerContext controllerContext)<br />    {<br />        <span style="color: #0000ff">object</span> jsonData = GetDeserializedJson(controllerContext);<br />        <span style="color: #0000ff">if</span> (jsonData == <span style="color: #0000ff">null</span>)<br />        {<br />            <span style="color: #0000ff">return</span> <span style="color: #0000ff">null</span>;<br />        }<br /><br />        Dictionary&lt;<span style="color: #0000ff">string</span>, <span style="color: #0000ff">object</span>&gt; backingStore = <span style="color: #0000ff">new</span> Dictionary&lt;<span style="color: #0000ff">string</span>, <span style="color: #0000ff">object</span>&gt;(StringComparer.OrdinalIgnoreCase);<br /><br />        <span style="color: #008000">// El DefaultModelBinder es capaz de "bindear" colecciones si los elementos se llaman x[0], x[1], x[2], así</span><br />        <span style="color: #008000">// que si dentro del objeto json tenemos alguna propiedad que sea array vamos a crear una entrada por</span><br />        <span style="color: #008000">// cada elemento del array</span><br /><br />        AddToBackingStore(backingStore, String.Empty, jsonData);<br />        <span style="color: #0000ff">return</span> <span style="color: #0000ff">new</span> DictionaryValueProvider&lt;<span style="color: #0000ff">object</span>&gt;(backingStore, CultureInfo.CurrentCulture);<br />    }<br />    <span style="color: #0000ff">private</span> <span style="color: #0000ff">static</span> <span style="color: #0000ff">void</span> AddToBackingStore(Dictionary&lt;<span style="color: #0000ff">string</span>, <span style="color: #0000ff">object</span>&gt; backingStore, <span style="color: #0000ff">string</span> prefix, <span style="color: #0000ff">object</span> <span style="color: #0000ff">value</span>)<br />    {<br />        { <span style="color: #008000">// dictionary?</span><br />            IDictionary&lt;<span style="color: #0000ff">string</span>, <span style="color: #0000ff">object</span>&gt; d = <span style="color: #0000ff">value</span> <span style="color: #0000ff">as</span> IDictionary&lt;<span style="color: #0000ff">string</span>, <span style="color: #0000ff">object</span>&gt;;<br />            <span style="color: #0000ff">if</span> (d != <span style="color: #0000ff">null</span>)<br />            {<br />                <span style="color: #0000ff">foreach</span> (var entry <span style="color: #0000ff">in</span> d)<br />                {<br />                    AddToBackingStore(backingStore, MakePropertyKey(prefix, entry.Key), entry.Value);<br />                }<br />                <span style="color: #0000ff">return</span>;<br />            }<br />        }<br /><br />        { <span style="color: #008000">// list?</span><br />            IList l = <span style="color: #0000ff">value</span> <span style="color: #0000ff">as</span> IList;<br />            <span style="color: #0000ff">if</span> (l != <span style="color: #0000ff">null</span>)<br />            {<br />                <span style="color: #0000ff">for</span> (<span style="color: #0000ff">int</span> i = 0; i &lt; l.Count; i++)<br />                {<br />                    AddToBackingStore(backingStore, MakeArrayKey(prefix, i), l[i]);<br />                }<br />                <span style="color: #0000ff">return</span>;<br />            }<br />        }<br /><br />        <span style="color: #008000">// primitive</span><br />        backingStore[prefix] = <span style="color: #0000ff">value</span>;<br />    }<br /><br />    <span style="color: #008000">/// &lt;summary&gt;</span><br />    <span style="color: #008000">/// Deserializa el código json que se encuentra dentro del body de la request</span><br />    <span style="color: #008000">/// &lt;/summary&gt;</span><br />    <span style="color: #0000ff">private</span> <span style="color: #0000ff">static</span> <span style="color: #0000ff">object</span> GetDeserializedJson(ControllerContext controllerContext)<br />    {<br />        <span style="color: #0000ff">if</span> (!controllerContext.HttpContext.Request.ContentType.StartsWith(<span style="color: #006080">"application/json"</span>, StringComparison.OrdinalIgnoreCase))<br />        {<br />            <span style="color: #008000">// not JSON request</span><br />            <span style="color: #0000ff">return</span> <span style="color: #0000ff">null</span>;<br />        }<br /><br />        StreamReader reader = <span style="color: #0000ff">new</span> StreamReader(controllerContext.HttpContext.Request.InputStream);<br />        <span style="color: #0000ff">string</span> bodyText = reader.ReadToEnd();<br />        <span style="color: #0000ff">if</span> (String.IsNullOrEmpty(bodyText))<br />        {<br />            <span style="color: #008000">// no JSON data</span><br />            <span style="color: #0000ff">return</span> <span style="color: #0000ff">null</span>;<br />        }<br /><br />        JavaScriptSerializer serializer = <span style="color: #0000ff">new</span> JavaScriptSerializer();<br />        <span style="color: #0000ff">object</span> jsonData = serializer.DeserializeObject(bodyText);<br />        <span style="color: #0000ff">return</span> jsonData;<br />    }<br /><br /><br />    <span style="color: #0000ff">private</span> <span style="color: #0000ff">static</span> <span style="color: #0000ff">string</span> MakeArrayKey(<span style="color: #0000ff">string</span> prefix, <span style="color: #0000ff">int</span> index)<br />    {<br />        <span style="color: #0000ff">return</span> prefix + <span style="color: #006080">"["</span> + index.ToString(CultureInfo.InvariantCulture) + <span style="color: #006080">"]"</span>;<br />    }<br /><br />    <span style="color: #0000ff">private</span> <span style="color: #0000ff">static</span> <span style="color: #0000ff">string</span> MakePropertyKey(<span style="color: #0000ff">string</span> prefix, <span style="color: #0000ff">string</span> propertyName)<br />    {<br />        <span style="color: #0000ff">return</span> (String.IsNullOrEmpty(prefix)) ? propertyName : prefix + <span style="color: #006080">"."</span> + propertyName;<br />    }<br /><br />}</pre>
                          
                          <p>
                            </div> 
                            
                            <p>
                              Ya... el código es más largo y más complejo que en el caso anterior, pero <em>básicamente</em> hace lo siguiente:
                            </p>
                            
                            <ol>
                              <li>
                                Deserializa el contenido JSON de la request y obtiene un objeto .NET
                              </li>
                              <li>
                                Insepcciona via reflection dicho objeto y va creando entradas (clave, valor) para cada propiedad. Además trata arrays (colecciones) y subobjetos. P.ej. Si el objeto deserializado tiene una colección de dos elementos llamada Foo, creará dos entradas con claves Foo[0] y Foo[1]. Igualmente si el objeto tiene un subobjeto llamado Bar que tiene dos propiedades, pongamos Baz1 y Baz2 creará dos entradas llamadas Bar.Baz1 y Bar.Baz2.
                              </li>
                            </ol>
                            
                            <p>
                              P.ej. en el el caso que nos ocupa, creará las siguientes entradas:
                            </p>
                            
                            <ul>
                              <li>
                                Name
                              </li>
                              <li>
                                Urls[0]
                              </li>
                              <li>
                                Urls[1]
                              </li>
                            </ul>
                            
                            <p>
                              No crea entrada para la propiedad Id, porque dicha propiedad no la estamos enviando via POST en el JSON.
                            </p>
                            
                            <p>
                              Como vimos en el post sobre el DefaultModelBinder, éste entiende estos nombres de las entradas y con ellas es capaz de crear el modelo <strong>y aplicar las validaciones</strong>.
                            </p>
                            
                            <p>
                              Así ahora podemos observar que:
                            </p>
                            
                            <ol>
                              <li>
                                Si la vista manda los datos a Home/Index, el modelo <strong>no</strong> se valida correctamente, ya que Id vale 0.
                              </li>
                              <li>
                                Si la vista manda los datos a Home/Index?Id=2 el modelo <strong>se</strong> valida correctamente, ya que Id vale 2 (el DefaultModelBinder ha combinado los datos de todos los value providers).
                              </li>
                            </ol>
                            
                            <p>
                              Espero que este post os sirva para terminar de comprender cuando usar un Model Binder propio y cuando usar un Value Provider... Recordad: Si queréis modificar de <em>donde (y cómo) de la request se sacan los datos</em>, debéis usar un Value Provider. Si lo que queréis modificar es <em>cómo se interpretan esos datos</em> debéis usar un Model Binder.
                            </p>
                            
                            <p>
                              Referencias:
                            </p>
                            
                            <ul>
                              <li>
                                <a target="_blank" href="http://lozanotek.com/blog/archive/2010/04/16/simple_json_model_binder.aspx" rel="noopener noreferrer">Leer JSON via POST usando Model Binder (por Javier G. Lozano)</a>
                              </li>
                              <li>
                                ASP.NET MVC Futures: Json Value Provider <ul>
                                  <li>
                                    <a target="_blank" href="http://aspnet.codeplex.com/releases/view/41742#DownloadId=110347" rel="noopener noreferrer">Si te bajas el código fuente de ASP.NET MVC2</a>, verás que hay un proyecto llamado MvcFutures, que son aspectos que el equipo de ASP.NET MVC está evaluando para futuras versiones. Entre ellos hay precisamente el JsonValueProviderFactory que yo he mostrado en este post!
                                  </li>
                                </ul>
                              </li>
                            </ul>
                            
                            <p>
                              Un saludo!!!
                            </p>