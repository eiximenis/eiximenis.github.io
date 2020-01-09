---
title: Binding de colecciones en ASP.NET MVC (iii)
description: Binding de colecciones en ASP.NET MVC (iii)
author: eiximenis

date: 2011-08-03T11:50:38+00:00
geeks_url: /?p=1573
geeks_visits:
  - 2368
geeks_ms_views:
  - 1331
categories:
  - Uncategorized

---
Bueno… vamos a seguir viendo el tema de binding de colecciones con ASP.NET MVC. En los dos posts anteriores hemos visto:

  * <a href="http://geeks.ms/blogs/etomas/archive/2011/07/09/binding-de-colecciones-en-asp-net-mvc.aspx" target="_blank" rel="noopener noreferrer">Como se enlazan las colecciones en ASP.NET MVC</a> 
  * <a href="http://geeks.ms/blogs/etomas/archive/2011/07/15/binding-de-colecciones-en-asp-net-mvc-ii.aspx" target="_blank" rel="noopener noreferrer">El uso del parámetro de request index</a> 

En este post vamos a ver como enlazar una colección de N elementos, de los cuales sólo nos llegan un determinado número, pero queremos fácilmente saber cuales son. Es decir, si nos llega sólo el primer elemento, el segundo y el octavo, recibir una lista con los ocho elementos, todos ellos a “null” (o un valor por defecto) excepto los informados (el primero, el segundo y el octavo en nuestro caso).

Si usamos el DefaultModelBinder esto **no pasa:** en los posts anteriores hemos visto como en el mejor de los casos (usando el parámetro index), recibimos sólo una colección con los tres elementos, y debemos usar ModelState.Keys para saber cuales son los índices reales informados. Es decir, si la vista sólo nos informa del primer, segundo y octavo elementos en el controlador recibimos una colección de tres elementos (los tres informados). Para saber que el tercer elemento (p.ej.) de dicha colección se corresponde al octavo índice real debemos usar ModelState.Keys. Vamos a ver ahora como podemos hacerlo para recibir, en este caso, una colección con los ocho elementos. De estos ocho, tan sólo el primer, el segundo y el octavo tendrán valor (el resto, un valor por defecto).

La solución es simple, y pasa por crearnos un Custom Model Binder 🙂 Crear un model binder propio parece muy complejo, pero se trata de implementar una interfaz con un solo método (_BindModel_). Sí, si miras el código del DefaultModelBinder te parecerá enorme y complejo, pero piensa que el DefaultModelBinder está pensado para enlazar _cualquier cosa_, y nosotros vamos a hacer un model binder preparado para enlazar sólamente colecciones (IEnumerable<T> en nuestro caso).

Así pues, vamos a hacer este custom model binder, especializado en colecciones. Vamos a imitar en todo al Default Model Binder, excepto en que nosotros vamos a devolver una colección con el tamaño _real_ (no solo con los elementos informados).

Os pongo primero el código del model binder y lo discutimos (por supuesto, si queréis preguntar algo concreto sobre el código, adelante!):

<div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
  <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; background-color: #f4f4f4; margin: 0em; border-left-style: none; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; border-right-style: none; font-size: 8pt; overflow: visible; padding-top: 0px" id="codeSnippet"><span style="color: #0000ff">public</span> <span style="color: #0000ff">class</span> CollectionBinder : IModelBinder<br />{<br />    <span style="color: #0000ff">public</span> <span style="color: #0000ff">object</span> BindModel(ControllerContext controllerContext, ModelBindingContext bindingContext)<br />    {<br /><br />        var model = CreateModel(bindingContext) <span style="color: #0000ff">as</span> IList;<br />        var prefix = bindingContext.ModelName;<br />        var indexesKey = bindingContext.FallbackToEmptyPrefix ?<br />            bindingContext.ValueProvider.GetValue(<span style="color: #006080">"index"</span>) :<br />            bindingContext.ValueProvider.GetValue(<span style="color: #0000ff">string</span>.Format(<span style="color: #006080">"{0}.index"</span>, prefix));<br />        var indexes = indexesKey == <span style="color: #0000ff">null</span> ? AllIndexes() : EnumerableFromIndexes(indexesKey.RawValue <span style="color: #0000ff">as</span> <span style="color: #0000ff">string</span>[]);<br />        var genericType = GetGenericTypeOfModel(bindingContext);<br /><br /><br />        <span style="color: #0000ff">foreach</span> (var index <span style="color: #0000ff">in</span> indexes)<br />        {<br />            var <span style="color: #0000ff">value</span> = bindingContext.FallbackToEmptyPrefix ?<br />                bindingContext.ValueProvider.GetValue(<span style="color: #0000ff">string</span>.Format(<span style="color: #006080">"[{0}]"</span>, index)) :<br />                bindingContext.ValueProvider.GetValue(<span style="color: #0000ff">string</span>.Format(<span style="color: #006080">"{0}[{1}]"</span>, prefix, index));<br />            <br />            <span style="color: #0000ff">if</span> (<span style="color: #0000ff">value</span> != <span style="color: #0000ff">null</span>)<br />            {<br />                var valueConverted = Convert.ChangeType(<span style="color: #0000ff">value</span>.AttemptedValue, genericType);<br />                model.Add(valueConverted);<br />            }<br />            <span style="color: #0000ff">else</span><br />            {<br />                <span style="color: #0000ff">if</span> (indexesKey == <span style="color: #0000ff">null</span>) <span style="color: #0000ff">break</span>;<br />                <span style="color: #0000ff">else</span><br />                {<br />                    model.Add(genericType.IsValueType<br />                                  ? Activator.CreateInstance(genericType)<br />                                  : <span style="color: #0000ff">null</span>);<br />                }<br />            }<br />        }<br /><br /><br />        <span style="color: #0000ff">return</span> model;<br />    }<br /><br />    <span style="color: #0000ff">private</span> <span style="color: #0000ff">object</span> CreateModel(ModelBindingContext bindingContext)<br />    {<br />        var genericType = GetGenericTypeOfModel(bindingContext);<br />        var listOfTType = <span style="color: #0000ff">typeof</span>(List&lt;&gt;).MakeGenericType(<span style="color: #0000ff">new</span> Type[] { genericType });<br />        <span style="color: #0000ff">return</span> Activator.CreateInstance(listOfTType);<br />    }<br /><br />    <span style="color: #0000ff">private</span> Type GetGenericTypeOfModel(ModelBindingContext bindingContext)<br />    {<br />        var type = bindingContext.ModelType;<br />        var genericTypes = type.GetGenericArguments();<br />        <span style="color: #0000ff">return</span> genericTypes.FirstOrDefault();<br />    }<br /><br />    <span style="color: #0000ff">private</span> IEnumerable&lt;<span style="color: #0000ff">int</span>&gt; AllIndexes()<br />    {<br />        <span style="color: #0000ff">for</span> (<span style="color: #0000ff">int</span> i = 0; i &lt; Int32.MaxValue; i++)<br />        {<br />            <span style="color: #0000ff">yield</span> <span style="color: #0000ff">return</span> i;<br />        }<br />    }<br /><br /><br />    <span style="color: #0000ff">private</span> IEnumerable&lt;<span style="color: #0000ff">int</span>&gt; EnumerableFromIndexes(<span style="color: #0000ff">string</span>[] indexesToUse)<br />    {<br />        <span style="color: #0000ff">if</span> (indexesToUse != <span style="color: #0000ff">null</span>)<br />        {<br />            <span style="color: #0000ff">foreach</span> (var token <span style="color: #0000ff">in</span> indexesToUse)<br />            {<br />                <span style="color: #0000ff">yield</span> &lt;s
pan style="color: #0000ff">return&lt;/span> Int32.Parse(token);<br />            }<br />        }<br />    }<br />}<br /></pre>
  
  <p>
    </div> 
    
    <p>
      Como funciona el siguiente código? Pues nuestro collection binder hace lo siguiente:
    </p>
    
    <ol>
      <li>
        Crea un objeto para representar el modelo. Dicho objeto será siempre una List<T>, siendo T el parámetro genérico del IEnumerable del modelo.
      </li>
      <li>
        Mira si existe el parámetro index. Si dicho parámetro existe, lo usa para saber los indices reales de la colección.&#160; Es decir, si indexes vale “0,1,2,3,4,5” (p.ej.) nuestro model binder va a devolver siempre una colección de 6 elementos (del 0 al 5) con independencia de los elementos reales informados en la vista. Esto es para <em>imitar</em> lo que hace el DefaultModelBinder y que vimos en el post anterior.
      </li>
      <li>
        Busca en los valueproviders los valores para todos los índices. Si el parámetro “index” no existía, todos los indices son literalmente “todos” (de 0 a Int32.MaxValue-1). Si el parámetro index no existe <strong>nos paramos cuando falta un elemento</strong> (porque si no, siempre devolveríamos una colección de Int32.MaxValue elementos!). Por su parte si el parametr index existe, iteramos sólo sobre sus valores, y si el valor no existe, lo añadimos al modelo con el valor por defecto del tipo genérico. Es decir, si index vale “0,1,2,3,4,5” y la vista no nos informa del valor del índice 3, pondremos el valor por defecto en el índice 3 y continuaremos hasta llegar a 5.
      </li>
    </ol>
    
    <p>
      El uso de los value providers para obtener los valores nos independiza de si dichos valores vienen por GET, POST o lo que sea. De esta manera el Model Binder es independiente de la request de http.
    </p>
    
    <p>
      Este CollectionBinder está preparado para trabajar con cualquier tipo de IEnumerable. Para usarlo, debemos registrarlo en global.asax:
    </p>
    
    <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
      <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; background-color: #f4f4f4; margin: 0em; border-left-style: none; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; border-right-style: none; font-size: 8pt; overflow: visible; padding-top: 0px" id="codeSnippet">ModelBinders.Binders[<span style="color: #0000ff">typeof</span>(IEnumerable&lt;<span style="color: #0000ff">string</span>&gt;)] = <span style="color: #0000ff">new</span> CollectionBinder();</pre>
      
      <p>
        </div> 
        
        <p>
          Con esto, lo hemos registrado para que los IEnumerable<string> se enlacen usando nuestro model binder!
        </p>
        
        <p>
          ¿Lo probamos? Para ello nos creamos un modelo:
        </p>
        
        <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
          <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; background-color: #f4f4f4; margin: 0em; border-left-style: none; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; border-right-style: none; font-size: 8pt; overflow: visible; padding-top: 0px" id="codeSnippet"><span style="color: #0000ff">public</span> <span style="color: #0000ff">class</span> FooModel<br />{<br />    <span style="color: #0000ff">public</span> <span style="color: #0000ff">string</span> Name { get; set; }<br />    <span style="color: #0000ff">public</span> <span style="color: #0000ff">int</span> Age { get; set; }<br />    <span style="color: #0000ff">public</span> IEnumerable&lt;<span style="color: #0000ff">string</span>&gt; Data { get; set; }<br />}<br /></pre>
          
          <p>
            </div> 
            
            <p>
              Y luego un controlador con un método para recibir un <em>FooModel</em>:
            </p>
            
            <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
              <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; background-color: #f4f4f4; margin: 0em; border-left-style: none; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; border-right-style: none; font-size: 8pt; overflow: visible; padding-top: 0px" id="codeSnippet"><span style="color: #0000ff">public</span> ActionResult Index()<br />{<br />    var model = <span style="color: #0000ff">new</span> FooModel();<br />    model.Age = 10;<br />    model.Name = <span style="color: #006080">"Nombre"</span>;<br />    model.Data = <span style="color: #0000ff">new</span> List&lt;<span style="color: #0000ff">string</span>&gt; {<span style="color: #006080">"cero"</span>, <span style="color: #006080">"uno"</span>, <span style="color: #006080">"dos"</span>, <span style="color: #006080">"tres"</span>, <span style="color: #006080">"cuatro"</span>};<br />    <span style="color: #0000ff">return</span> View(model);<br />}<br />[HttpPost]<br /><span style="color: #0000ff">public</span> ActionResult Index(FooModel model )<br />{<br />    <span style="color: #0000ff">int</span> i = 0;<br />    <span style="color: #008000">// Codigo...</span><br />}<br /></pre>
              
              <p>
                </div> 
                
                <p>
                  Vamos ahora a hacer una vista para editar nuestro FooModel:
                </p>
                
                <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
                  <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; background-color: #f4f4f4; margin: 0em; border-left-style: none; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; border-right-style: none; font-size: 8pt; overflow: visible; padding-top: 0px" id="codeSnippet">@using BindingColecciones3.Models<br />@model FooModel<br /><span style="color: #0000ff">&lt;!</span><span style="color: #800000">DOCTYPE</span> <span style="color: #ff0000">html</span><span style="color: #0000ff">&gt;</span><br /><span style="color: #0000ff">&lt;</span><span style="color: #800000">html</span><span style="color: #0000ff">&gt;</span><br /><span style="color: #0000ff">&lt;</span><span style="color: #800000">head</span><span style="color: #0000ff">&gt;</span><br />    <span style="color: #0000ff">&lt;</span><span style="color: #800000">title</span><span style="color: #0000ff">&gt;</span>title<span style="color: #0000ff">&lt;/</span><span style="color: #800000">title</span><span style="color: #0000ff">&gt;</span><br /><span style="color: #0000ff">&lt;/</span><span style="color: #800000">head</span><span style="color: #0000ff">&gt;</span><br /><span style="color: #0000ff">&lt;</span><span style="color: #800000">body</span><span style="color: #0000ff">&gt;</span><br />    <span style="color: #0000ff">&lt;</span><span style="color: #800000">div</span><span style="color: #0000ff">&gt;</span><br />        @using (Html.BeginForm())<br />        {<br />            <span style="color: #0000ff">&lt;</span><span style="color: #800000">div</span><span style="color: #0000ff">&gt;</span><br />                @Html.LabelFor(x =<span style="color: #0000ff">&gt;</span> x.Name)<br />                @Html.EditorFor(x =<span style="color: #0000ff">&gt;</span> x.Name)<br />                <span style="color: #0000ff">&lt;</span><span style="color: #800000">br</span> <span style="color: #0000ff">/&gt;</span><br />                @Html.LabelFor(x =<span style="color: #0000ff">&gt;</span> x.Age)<br />                @Html.EditorFor(x =<span style="color: #0000ff">&gt;</span> x.Age)<br />            <span style="color: #0000ff">&lt;/</span><span style="color: #800000">div</span><span style="color: #0000ff">&gt;</span><br />            <span style="color: #0000ff">&lt;</span><span style="color: #800000">ul</span><span style="color: #0000ff">&gt;</span><br />                @for (var idx = 0; idx <span style="color: #0000ff">&lt;</span> Model.Data.Count(); idx++)<br />                {<br />                    <span style="color: #0000ff">&lt;</span><span style="color: #800000">li</span><span style="color: #0000ff">&gt;</span>Checkbox #@idx:<br />                        <span style="color: #0000ff">&lt;</span><span style="color: #800000">input</span> <span style="color: #ff0000">type</span><span style="color: #0000ff">="checkbox"</span> <span style="color: #ff0000">name</span><span style="color: #0000ff">="Data[@idx]"</span> <span style="color: #ff0000">value</span><span style="color: #0000ff">="@Model.Data.Skip(idx).First()"</span><span style="color: #0000ff">/&gt;</span><br />                        <span style="color: #0000ff">&lt;</span><span style="color: #800000">input</span> <span style="color: #ff0000">type</span><span style="color: #0000ff">="hidden"</span> <span style="color: #ff0000">name</span><span style="color: #0000ff">="Data.index"</span> <span style="color: #ff0000">value</span><span style="color: #0000ff">="@idx"</span><span style="color: #0000ff">/&gt;</span><br />                    <span style="color: #0000ff">&lt;/</span><span style="color: #800000">li</span><span style="color: #0000ff">&gt;</span><br />                }<br />            <span style="color: #0000ff">&lt;/</span><span style="color: #800000">ul</span><span style="color: #0000ff">&gt;</span><br />            <span style="color: #0000ff">&lt;</span><span style="color: #800000">input</span> <span style="color: #ff0000">type</span><span style="color: #0000ff">="submit"</span> <span style="color: #0000ff">/&gt;</span><br />        }<br />    <span style="color: #0000ff">&lt;/</span><span style="color: #800000">div</span><span style="color: #0000ff">&gt;</span><br /><span style="color: #0000ff">&lt;/</span><span style="color: #800000">body</span><span style="color: #0000ff">&gt;</span><br /><span style="color: #0000ff">&lt;/</span><span style="color: #800000">html</span><span style="color: #0000ff">&gt;</span><br /></pre>
                  
                  <p>
                    </div> 
                    
                    <p>
                      Fijaos en como creamos las checkboxes: El atributo name de cada checkbox es Data[0], Data[1], Data[2]… Eso es porque Data es el nombre de la propiedad IEnumerable<string> de nuestro modelo. El atributo value de cada checkbox será la cadena que se enlazará en el modelo. Si p.ej. sólo marcamos la tercera checkbox (cuyo value es “dos”, eso es lo que recibiremos en el controlador:
                    </p>
                    
                    <p>
                      <a href="http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_7EA48627.png"><img style="background-image: none; border-right-width: 0px; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; border-top-width: 0px; border-bottom-width: 0px; border-left-width: 0px; padding-top: 0px" title="image" border="0" alt="image" src="http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_37DAED1D.png" width="196" height="177" /></a>
                    </p>
                    
                    <p>
                      Fijaos que, a diferencia del CustomModelBinder, lo que recibimos ahora es una colección de 6 elementos (0-5) y sabemos exactamente cual era la única checkbox marcada. Esa misma vista, pero usando el DefaultModelBinder para enlazar los datos, devolvería lo siguiente al controlador (tal y como vimos en el post anterior):
                    </p>
                    
                    <p>
                      <a href="http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_00452C2F.png"><img style="background-image: none; border-right-width: 0px; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; border-top-width: 0px; border-bottom-width: 0px; border-left-width: 0px; padding-top: 0px" title="image" border="0" alt="image" src="http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_45E9B64D.png" width="193" height="94" /></a>
                    </p>
                    
                    <p>
                      Y deberíamos usar ModelState.Keys para saber que este “dos” es el valor de la tercera checkbox marcada.
                    </p>
                    
                    <p>
                      Recordad que esto ocurre porque <strong>los navegadores no envían valores para una checkbox NO marcada.</strong> Es decir, en HTML las checkboxes no tienen el valor de true o false. Tienen sólo el valor que ponga en su <em>value</em> si están marcadas o no existen si no están marcadas.
                    </p>
                    
                    <p>
                      Y finalmente una consideración sobre el código de este CollectionModelBinder: Tiene algunas limitaciones, alguna que <em>otra cosa que se podría mejorar, alguna incongruencia (sobretodo en la gestión del parámetro index)</em> y cosas que se le podrían añadir… Os dejo que vayáas pensando cuales… y alguna de ellas las veremos en un siguiente post, que por hoy, es suficiente, no? 😉
                    </p>
                    
                    <p>
                      Un saludo!
                    </p>