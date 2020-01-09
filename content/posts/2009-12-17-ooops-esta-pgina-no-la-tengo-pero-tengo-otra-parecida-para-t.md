---
title: Ooops.. esta página no la tengo, pero tengo otra parecida para tí…
author: eiximenis

date: 2009-12-17T11:53:22+00:00
geeks_url: /?p=1484
geeks_visits:
  - 1196
geeks_ms_views:
  - 580
categories:
  - Uncategorized

---
Este genial post de José M. Aguilar <a href="http://geeks.ms/blogs/jmaguilar/archive/2009/12/16/procesar-peticiones-a-acciones-inexistentes-en-asp-net-mvc.aspx" target="_blank" rel="noopener noreferrer">sobre como procesar peticiones existentes en ASP.NET MVC</a>, me ha dado una idea que quiero compartir con vosotros… El tema consiste en que si el usuario se equivoca y entra una URL errónea como /Home/Jindex (en lugar de /Home/Index) le podemos sugerir que quizá quería ir a /Home/Index. Vamos a ver como podríamos hacerlo…

La idea es que cuando recibamos una petición errónea en el _HandleUnknownAction_ miremos cuales son las acciones del controlador y miremos cual es la acción que más se aproxima a la acción que el usuario ha entrado.

**1. Obteniendo las acciones del controlador actual**

Si usamos el _ActionInvoker_ por defecto de ASP.NET MVC, las acciones están mapeadas a métodos públicos del controlador. El nombre del método define el nombre de la acción, excepto si el método está decorado con el atributo _ActionNameAttribute_ que especifica un nombre de acción distinto.

Así pues, una manera de obtener las acciones del controlador actual es recorrerse sus métodos públicos y obtener su nombre o bien el nombre del atributo _ActionNameAttribute_ que tuviese asociado:

<div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
  <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px" id="codeSnippet"><span style="color: #0000ff">namespace</span> System.Web.Mvc<br />{<br />    <span style="color: #0000ff">public</span> <span style="color: #0000ff">static</span> <span style="color: #0000ff">class</span> ControllerExtensions<br />    {<br />        <span style="color: #0000ff">public</span> <span style="color: #0000ff">static</span> IEnumerable&lt;<span style="color: #0000ff">string</span>&gt; GetAllActions(<span style="color: #0000ff">this</span> Controller self)<br />        {<br />            <span style="color: #0000ff">var</span> methods = self.GetType().GetMethods(BindingFlags.Instance | BindingFlags.DeclaredOnly | BindingFlags.Public);<br />            <span style="color: #0000ff">return</span> methods.Select(x =&gt;<br />                x.GetCustomAttributes(<span style="color: #0000ff">typeof</span>(ActionNameAttribute), <span style="color: #0000ff">true</span>).Length == 1 ?<br />                    ((ActionNameAttribute)x.GetCustomAttributes(<span style="color: #0000ff">typeof</span>(ActionNameAttribute), <span style="color: #0000ff">true</span>)[0]).Name :<br />                    x.Name);<br />        }<br />    }<br />}</pre>
  
  <p>
    </div> 
    
    <p>
      <strong>2. Calculando que acción es la más parecida a la que ha entrado el usuario</strong>
    </p>
    
    <p>
      El siguiente paso es ver cual de todas las acciones se parece más a la acción que ha entrado el usuario. Hay varios algoritmos para calcular la <em>distancia</em> entre dos cadenas, uno conocido es la <a href="http://en.wikipedia.org/wiki/Levenshtein_distance" target="_blank" rel="noopener noreferrer"><em>distancia de Levenshtein</em></a> que es el que yo he usado. En el artículo de la wikipedia enlazado tenéis el pseudo-código del algoritmo… para los vagos <a href="http://www.merriampark.com/ldcsharp.htm" target="_blank" rel="noopener noreferrer">aquí tenéis una implementación de la distancia de Levenshtein en C#</a>.
    </p>
    
    <p>
      Yo he implementado el método cómo un método de extensión de la clase string:
    </p>
    
    <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
      <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px" id="codeSnippet"><span style="color: #0000ff">namespace</span> MvcApplication1.Extension<br />{<br />    <span style="color: #0000ff">public</span> <span style="color: #0000ff">static</span> <span style="color: #0000ff">class</span> StringExtensions<br />    {<br />        <span style="color: #0000ff">public</span> <span style="color: #0000ff">static</span> <span style="color: #0000ff">int</span> LevenshteinDistance(<span style="color: #0000ff">this</span> <span style="color: #0000ff">string</span> s, <span style="color: #0000ff">string</span> t)<br />        {<br />           <span style="color: #008000">// Ver una implementación en http://www.merriampark.com/ldcsharp.htm</span><br />        }<br />    }<br />}</pre>
      
      <p>
        </div> 
        
        <p>
          Finalmente en el método <em>HandleUnknownAction</em> sólo nos queda recorrer el enumerable de acciones devuelto por <em>GetAllActions </em>y para cada acción calcular la distancia de Levenshtein entre esta acción y el nombre que ha entrado el usuario… y cojer la menor:
        </p>
        
        <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
          <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px" id="codeSnippet"><span style="color: #0000ff">protected</span> <span style="color: #0000ff">override</span> <span style="color: #0000ff">void</span> HandleUnknownAction(<span style="color: #0000ff">string</span> actionName)<br />{<br />    var actions =  <span style="color: #0000ff">this</span>.GetAllActions();<br />    <span style="color: #0000ff">int</span> min = <span style="color: #0000ff">int</span>.MaxValue;<br />    <span style="color: #0000ff">string</span> newAction = <span style="color: #0000ff">null</span>;<br />    <span style="color: #0000ff">foreach</span> (var action <span style="color: #0000ff">in</span> actions)<br />    {<br />        <span style="color: #0000ff">int</span> ld = action.LevenshteinDistance(actionName);<br />        <span style="color: #0000ff">if</span> (ld &lt; min)<br />        {<br />            min = ld;<br />            newAction = action;<br />        }<br />    }<br />    <span style="color: #0000ff">if</span> (min &lt; <span style="color: #0000ff">int</span>.MaxValue)<br />    {<br />        View(<span style="color: #006080">"RedirectView"</span>, <span style="color: #0000ff">new</span> RedirectModel(newAction, <span style="color: #006080">"Home"</span>, actionName)).<br />            ExecuteResult(<span style="color: #0000ff">this</span>.ControllerContext);<br />    }<br />    <span style="color: #0000ff">else</span><br />    {<br />        <span style="color: #0000ff">base</span>.HandleUnknownAction(actionName);<br />    }<br />}</pre>
          
          <p>
            </div> 
            
            <p>
              La clase RedirectModel es una clase que tiene tres propiedades: Acción a donde pensamos que el usuario quería ir, controlador de dicha acción, y acción tecleada por el usuario:
            </p>
            
            <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
              <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px" id="codeSnippet"><span style="color: #0000ff">public</span> <span style="color: #0000ff">class</span> RedirectModel<br />{<br />    <span style="color: #0000ff">public</span> RedirectModel(<span style="color: #0000ff">string</span> action, <span style="color: #0000ff">string</span> controller, <span style="color: #0000ff">string</span> originalAction)<br />    {<br />        <span style="color: #0000ff">this</span>.Action = action;<br />        <span style="color: #0000ff">this</span>.Controller = controller;<br />        <span style="color: #0000ff">this</span>.OriginalAction = originalAction;<br />    }<br /><br />    <span style="color: #0000ff">public</span> <span style="color: #0000ff">string</span> Action { get; <span style="color: #0000ff">private</span> set; }<br />    <span style="color: #0000ff">public</span> <span style="color: #0000ff">string</span> Controller { get; <span style="color: #0000ff">private</span> set; }<br />    <span style="color: #0000ff">public</span> <span style="color: #0000ff">string</span> OriginalAction { get; <span style="color: #0000ff">private</span> set; }<br />}</pre>
              
              <p>
                </div> 
                
                <p>
                  Finalmente, sólo nos queda la vista “RedirectView”, que yo he puesto en Shared (para que pueda ser reutilizada por varios controladores):
                </p>
                
                <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
                  <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px" id="codeSnippet">&lt;%@ Page Title=<span style="color: #006080">""</span> Language=<span style="color: #006080">"C#"</span> MasterPageFile=<span style="color: #006080">"~/Views/Shared/Site.Master"</span> Inherits=<span style="color: #006080">"System.Web.Mvc.ViewPage&lt;MvcApplication1.Models.RedirectModel&gt;"</span> %&gt;<br /><br />&lt;asp:Content ID=<span style="color: #006080">"Content1"</span> ContentPlaceHolderID=<span style="color: #006080">"TitleContent"</span> runat=<span style="color: #006080">"server"</span>&gt;<br />    ViewUserControl1<br />&lt;/asp:Content&gt;<br /><br />&lt;asp:Content ID=<span style="color: #006080">"Content2"</span> ContentPlaceHolderID=<span style="color: #006080">"MainContent"</span> runat=<span style="color: #006080">"server"</span>&gt;<br /><br />    &lt;h2&gt;Pos por &lt;%= Model.OriginalAction%&gt; no me viene nada...&lt;/h2&gt;<br /><br />     OOppps... esta página no existe... ¿seguro que no querías ir a<br />     &lt;a href=<span style="color: #006080">"&lt;%= Url.Action(Model.Action, Model.Controller)%&gt;"</span>&gt;&lt;%= Model.Action %&gt;&lt;/a&gt;?<br />&lt;/asp:Content&gt;</pre>
                  
                  <p>
                    </div> 
                    
                    <p>
                      Y este es el resultado, si el usuario teclea /Home/Jindex esto es lo que se le muestra:
                    </p>
                    
                    <p>
                      <a href="http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_1D13D5EB.png"><img style="border-bottom: 0px; border-left: 0px; display: inline; border-top: 0px; border-right: 0px" title="image" border="0" alt="image" src="http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_2FB8A9D5.png" width="244" height="135" /></a>
                    </p>
                    
                    <p>
                      Ya véis, qué fácil 🙂 Un saludo!!!
                    </p>
                    
                    <p>
                      PD: Os dejo un <a href="http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas.17122009/MvcSugerirPagina.zip" target="_blank" rel="noopener noreferrer">zip con la solución completa</a>!!!
                    </p>