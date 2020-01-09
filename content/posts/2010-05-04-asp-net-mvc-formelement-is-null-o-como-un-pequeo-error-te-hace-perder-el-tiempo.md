---
title: '[ASP.NET MVC] ‘formElement is null’ o como un pequeño error te hace perder el tiempo…'
description: '[ASP.NET MVC] ‘formElement is null’ o como un pequeño error te hace perder el tiempo…'
author: eiximenis

date: 2010-05-04T12:17:37+00:00
geeks_url: /?p=1506
geeks_visits:
  - 2041
geeks_ms_views:
  - 1866
categories:
  - Uncategorized

---
Hola! Esta es una breve historia de un _pequeño_ error que cometí y que quiero compartir con vosotros… por si acaso 🙂

Tenía una aplicación ASP.NET MVC que usaba <a href="http://msdn.microsoft.com/es-es/library/system.componentmodel.dataannotations(VS.95).aspx" target="_blank" rel="noopener noreferrer">DataAnnotations</a> para la validación de los modelos. El uso de DataAnnotations para permitir la validación de modelos en ASP.NET MVC es muy simple. El primer paso es decorar la clase modelo con los atributos que indican las validaciones a realizar:

<div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
  <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px" id="codeSnippet"><span style="color: #0000ff">public</span> <span style="color: #0000ff">class</span> FooModel<br />{<br />    [Required(ErrorMessage=<span style="color: #006080">"Debe entrar un valor!"</span>)]<br />    <span style="color: #0000ff">public</span> <span style="color: #0000ff">string</span> Name { get; set; }<br />}</pre>
  
  <p>
    </div> 
    
    <p>
      Aquí simplemente indicamos que la propiedad ‘Name’ debe ser obligatoria. Hay otros validadores como <a href="http://msdn.microsoft.com/es-es/library/system.componentmodel.dataannotations.regularexpressionattribute(v=VS.95).aspx" target="_blank" rel="noopener noreferrer">RegularExpressionAttribute</a> que valida contra una expresión regular o <a href="http://msdn.microsoft.com/es-es/library/system.componentmodel.dataannotations.rangeattribute(v=VS.95).aspx" target="_blank" rel="noopener noreferrer">RangeAttribute</a> que valida entre rangos, entre otros (y obviamente nos podemos crear los nuestros).
    </p>
    
    <p>
      La vista para editar elementos de tipo FooModel es también muy sencilla:
    </p>
    
    <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
      <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px" id="codeSnippet"><span style="color: #006080">&lt;</span>%@ Page Title="" Language="C#" MasterPageFile="~/Views/Shared/Site<span style="color: #cc6633">.Master</span>" Inherits="System<span style="color: #cc6633">.Web</span><span style="color: #cc6633">.Mvc</span><span style="color: #cc6633">.ViewPage</span><span style="color: #006080">&lt;</span>FooMvc<span style="color: #cc6633">.Models</span><span style="color: #cc6633">.Home</span><span style="color: #cc6633">.FooModel</span><span style="color: #006080">&gt;</span>" %<span style="color: #006080">&gt;</span><br /><br /><span style="color: #006080">&lt;</span>asp:Content ID="Content1" ContentPlaceHolderID="TitleContent" runat="server"<span style="color: #006080">&gt;</span><br />    Index<br /><span style="color: #006080">&lt;</span>/asp:Content<span style="color: #006080">&gt;</span><br /><br /><span style="color: #006080">&lt;</span>asp:Content ID="Content2" ContentPlaceHolderID="MainContent" runat="server"<span style="color: #006080">&gt;</span><br /><br />    <span style="color: #006080">&lt;</span><span style="color: #0000ff">h2</span><span style="color: #006080">&gt;</span>Index<span style="color: #006080">&lt;</span>/<span style="color: #0000ff">h2</span><span style="color: #006080">&gt;</span><br /><br />    <span style="color: #006080">&lt;</span>% Html<span style="color: #cc6633">.EnableClientValidation</span>(); %<span style="color: #006080">&gt;</span><br />    <span style="color: #006080">&lt;</span>% using (Html<span style="color: #cc6633">.BeginForm</span>()) {%<span style="color: #006080">&gt;</span><br />        <span style="color: #006080">&lt;</span>%: Html<span style="color: #cc6633">.ValidationSummary</span>(true) %<span style="color: #006080">&gt;</span><br /><br />        <span style="color: #006080">&lt;</span><span style="color: #0000ff">fieldset</span><span style="color: #006080">&gt;</span><br />            <span style="color: #006080">&lt;</span><span style="color: #0000ff">legend</span><span style="color: #006080">&gt;</span>Fields<span style="color: #006080">&lt;</span>/<span style="color: #0000ff">legend</span><span style="color: #006080">&gt;</span><br />            <span style="color: #006080">&lt;</span><span style="color: #0000ff">div</span> class="editor-<span style="color: #0000ff">label</span>"<span style="color: #006080">&gt;</span><br />                <span style="color: #006080">&lt;</span>%: Html<span style="color: #cc6633">.LabelFor</span>(model =<span style="color: #006080">&gt;</span> model<span style="color: #cc6633">.Name</span>) %<span style="color: #006080">&gt;</span><br />            <span style="color: #006080">&lt;</span>/<span style="color: #0000ff">div</span><span style="color: #006080">&gt;</span><br />            <span style="color: #006080">&lt;</span><span style="color: #0000ff">div</span> class="editor-field"<span style="color: #006080">&gt;</span><br />                <span style="color: #006080">&lt;</span>%: Html<span style="color: #cc6633">.TextBoxFor</span>(model =<span style="color: #006080">&gt;</span> model<span style="color: #cc6633">.Name</span>) %<span style="color: #006080">&gt;</span><br />                <span style="color: #006080">&lt;</span>%: Html<span style="color: #cc6633">.ValidationMessageFor</span>(model =<span style="color: #006080">&gt;</span> model<span style="color: #cc6633">.Name</span>) %<span style="color: #006080">&gt;</span><br />            <span style="color: #006080">&lt;</span>/<span style="color: #0000ff">div</span><span style="color: #006080">&gt;</span><br />            <span style="color: #006080">&lt;</span><span style="color: #0000ff">p</span><span style="color: #006080">&gt;</span><br />                <span style="color: #006080">&lt;</span><span style="color: #0000ff">input</span> type="submit" value="Create" /<span style="color: #006080">&gt;</span><br />            <span style="color: #006080">&lt;</span>/<span style="color: #0000ff">p</span><span style="color: #006080">&gt;</span><br />        <span style="color: #006080">&lt;</span>/<span style="color: #0000ff">fieldset</span><span style="color: #006080">&gt;</span><br />    <span style="color: #006080">&lt;</span>% } %<span style="color: #006080">&gt;</span><br /><span style="color: #006080">&lt;</span>/asp:Content<span style="color: #006080">&gt;</span><br /><br /></pre>
      
      <p>
        </div>
      </p>
      
      <p>
        Fijaos en la llamada a Html.EnableClientValidation() que lo que hace es habilitar las validaciones en cliente (además de las de servidor claro, recordad que <strong>siempre</strong> debe validarse en servidor con independencia de que también se valide en cliente). Si no usamos EnableClientValidation() tenemos sólo validación en servidor. El uso de la validación en cliente requiere el uso de la ajax library, así que en mi master page he incluído lo siguiente:
      </p>
      
      <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
        <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px" id="codeSnippet">&lt;script src=<span style="color: #006080">"../../Scripts/MicrosoftAjax.js"</span> type=<span style="color: #006080">"text/javascript"</span>&gt;&lt;/script&gt;<br />&lt;script src=<span style="color: #006080">"../../Scripts/MicrosoftMvcAjax.js"</span> type=<span style="color: #006080">"text/javascript"</span>&gt;&lt;/script&gt;<br />&lt;script src=<span style="color: #006080">"../../Scripts/MicrosoftMvcValidation.js"</span> type=<span style="color: #006080">"text/javascript"</span>&gt;&lt;/script&gt;</pre>
        
        <p>
          </div> 
          
          <p>
            Finalmente queda sólo el controlador… Es extremadamente simple:
          </p>
          
          <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
            <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px" id="codeSnippet"><span style="color: #0000ff">public</span> <span style="color: #0000ff">class</span> HomeController : Controller<br />{<br />    <span style="color: #0000ff">public</span> ActionResult Index()<br />    {<br />        <span style="color: #0000ff">return</span> View();<br />    }<br /><br />    [HttpPost()]<br />    <span style="color: #0000ff">public</span> ActionResult Index(FooModel model)<br />    {<br />        <span style="color: #0000ff">if</span> (ModelState.IsValid)<br />        {<br />            <span style="color: #008000">// Validación OK</span><br />        }<br />        <span style="color: #0000ff">else</span><br />        {<br />            <span style="color: #0000ff">return</span> View(model);<br />        }<br />    }<br /><br />}</pre>
            
            <p>
              </div> 
              
              <p>
                El método Index sin parámetros es el que responde a la petición GET /host/Home/Index y devuelve la vista. Cuando se hace submit del formulario, se llama a /host/Home/Index pero usando POST por lo que se usa el método Index que recibe un FooModel. ASP.NET MVC hace binding automáticamente entre los parámetros POST y las propiedades de FooModel, por lo que obtenemos el objeto con los datos del usuario. La propiedad ModelState.IsValid me devuelve si las validaciones son correctas o no (y si no lo son, generalmente lo que se hace es mostrar de nuevo la vista para que el usuario pueda corregir los datos).
              </p>
              
              <p>
                Hasta ahí todo normal… sólo que a mi no me funcionaba. Recordad que había habilitado las validaciones en cliente, así que si hacía submit del formulario sin entrar ningún valor para la propiedad Name, me debería mostrar el error <strong>sin</strong> hacer POST… pero me lo hacía. Es decir, me ignoraba las validaciones en cliente.
              </p>
              
              <p>
                Después de varios intentos y mirando que podía estar pasando, lo ejecuté con Firefox para ver si es que se daba algún error de javascript (internet explorer no me decía nada). Y efectivamente en la consola de javascript de Firefox me aparecía un error: <em>formElement is null</em>.
              </p>
              
              <p>
                Una búsqueda en Google no me dio muchos resultados… A <a href="http://stackoverflow.com/questions/2368912/formelement-is-null-with-mvc-client-validation" target="_blank" rel="noopener noreferrer">alguien más le pasaba lo mismo</a> pero ninguna de las respuestas eran aplicables a mi caso. Después de algunas paranoias (era mi primer proyecto asp.net mvc2 con el vs2010 rtm y a lo mejor algo no se había instalado bien), investigando el código html generado vi esto:
              </p>
              
              <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
                <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px" id="codeSnippet">&lt;input type=<span style="color: #006080">"hidden"</span> name=<span style="color: #006080">"__VIEWSTATE"</span> id=<span style="color: #006080">"__VIEWSTATE"</span> value=<span style="color: #006080">"/wEPDwUJNDEyOTQxNTc1ZGRm6ES0w4WmJi1Rd8IlwFU8wFLynCBA/haT9Dce2tSlxQ=="</span> /&gt;<br /></pre>
                
                <p>
                  </div> 
                  
                  <p>
                    Coooomo??? Un Viewstate??? Y que se supone que hace aquí? Bueno… Pues sabéis de donde venía? Pues de aquí:
                  </p>
                  
                  <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
                    <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px" id="codeSnippet"><span style="color: #0000ff">&lt;</span><span style="color: #800000">form</span> <span style="color: #ff0000">id</span><span style="color: #0000ff">="form1"</span> <span style="color: #ff0000">runat</span><span style="color: #0000ff">="server"</span><span style="color: #0000ff">&gt;</span><br /><span style="color: #0000ff">&lt;</span><span style="color: #800000">div</span><span style="color: #0000ff">&gt;</span><br />    <span style="color: #0000ff">&lt;</span><span style="color: #800000">asp:ContentPlaceHolder</span> <span style="color: #ff0000">ID</span><span style="color: #0000ff">="MainContent"</span> <span style="color: #ff0000">runat</span><span style="color: #0000ff">="server"</span><span style="color: #0000ff">&gt;</span><br />    <span style="color: #0000ff">&lt;/</span><span style="color: #800000">asp:ContentPlaceHolder</span><span style="color: #0000ff">&gt;</span><br /><span style="color: #0000ff">&lt;/</span><span style="color: #800000">div</span><span style="color: #0000ff">&gt;</span><br /><span style="color: #0000ff">&lt;/</span><span style="color: #800000">form</span><span style="color: #0000ff">&gt;</span></pre>
                    
                    <p>
                      </div> 
                      
                      <p>
                        Y donde estaba este maléfico <form runat=”server”>? Pues sí, sí: en la master page.
                      </p>
                      
                      <p>
                        Y por qué? Bueno… pues porque cuando añadí la master page me equivoqué y añadí una “Master Page” en lugar de una “MVC 2 View Master Page”:
                      </p>
                      
                      <p>
                        <a href="http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_34B9B799.png"><img style="border-bottom: 0px; border-left: 0px; display: inline; border-top: 0px; border-right: 0px" title="image" border="0" alt="image" src="http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_77D50FF9.png" width="244" height="154" /></a>
                      </p>
                    </p>
                    
                    <p>
                      Así que esa es mi historia de como un error tonto puede llegar a darte verdaderos quebraderos que cabeza… 🙂
                    </p>
                    
                    <p>
                      Un saludo!
                    </p>
                    
                    <p>
                      PD: Y ahora voy a escribir 100 veces “cuando añada una master page a mi proyecto, voy a fijarme que sea una master page de MVC 2”… 🙂
                    </p>