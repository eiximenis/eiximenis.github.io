---
title: Unobtrusive Ajax en MVC3
description: Unobtrusive Ajax en MVC3
author: eiximenis

date: 2010-11-09T11:54:22+00:00
geeks_url: /?p=1540
geeks_visits:
  - 5957
geeks_ms_views:
  - 2050
categories:
  - Uncategorized

---
Buenas! Una de las novedades más interesantes de MVC3 es el soporte para eso que se llama _Unobtrusive_ Ajax. La verdad es que no encuentro una buena traducción para Unobtrusive (discreto no me convence).

La idea del Unobtrusive Ajax es **evitar mezclar código script con código HTML**. De la misma manera que CSS nos permite separar completamente el código HTML de su representación, con Unobtrusive Ajax vamos a poder separar el código javascript del código HTML.

Pero mejor, veamoslo con un ejemplo, ultra sencillo 🙂

Imaginad que tengo una vista con este contenido:

<div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
  <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; background-color: #f4f4f4; margin: 0em; border-left-style: none; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; border-right-style: none; font-size: 8pt; overflow: visible; padding-top: 0px" id="codeSnippet">&lt;h2&gt;Normal Ajax&lt;/h2&gt;<br />&lt;% <span style="color: #0000ff">using</span> (Ajax.BeginForm(<span style="color: #006080">"PostData"</span>, <span style="color: #0000ff">new</span> AjaxOptions() { HttpMethod=<span style="color: #006080">"Post"</span>, UpdateTargetId=<span style="color: #006080">"datadiv"</span>})) { %&gt;<br /><br />    &lt;label <span style="color: #0000ff">for</span>=<span style="color: #006080">"name"</span>&gt;Name: &lt;/label&gt;<br />    &lt;input type=<span style="color: #006080">"text"</span> name=<span style="color: #006080">"name"</span> id=<span style="color: #006080">"name"</span>/&gt;<br />    &lt;input type=<span style="color: #006080">"submit"</span> value=<span style="color: #006080">"Send"</span> /&gt;<br />&lt;% } %&gt;<br />&lt;hr /&gt;<br />Aquí irá el resultado: &lt;p /&gt;<br />&lt;div id=<span style="color: #006080">"datadiv"</span>&gt;<br />&lt;/div&gt;</pre>
  
  <p>
    </div> 
    
    <p>
      Esta vista genera un <form> con un campo de texto y envía los datos a una acción llamada “PostData” e incrusta el resultado de dicha acción (que será una vista parcial) en el div cuyo id es “datadiv”.
    </p>
    
    <p>
      Este es el código HTML generado por esta vista en MVC2:
    </p>
    
    <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
      <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; background-color: #f4f4f4; margin: 0em; border-left-style: none; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; border-right-style: none; font-size: 8pt; overflow: visible; padding-top: 0px" id="codeSnippet"><span style="color: #0000ff">&lt;</span><span style="color: #800000">h2</span><span style="color: #0000ff">&gt;</span>Normal Ajax<span style="color: #0000ff">&lt;/</span><span style="color: #800000">h2</span><span style="color: #0000ff">&gt;</span><br /><span style="color: #0000ff">&lt;</span><span style="color: #800000">form</span> <span style="color: #ff0000">action</span><span style="color: #0000ff">="/Home/PostData"</span> <span style="color: #ff0000">method</span><span style="color: #0000ff">="post"</span> <br />   <span style="color: #ff0000">onclick</span><span style="color: #0000ff">="Sys.Mvc.AsyncForm.handleClick(this, new Sys.UI.DomEvent(event));"</span> <br />   <span style="color: #ff0000">onsubmit</span><span style="color: #0000ff">="Sys.Mvc.AsyncForm.handleSubmit(this, new Sys.UI.DomEvent(event), { insertionMode: Sys.Mvc.InsertionMode.replace, httpMethod: &#39;Post&#39;, updateTargetId: &#39;datadiv&#39; });"</span><span style="color: #0000ff">&gt;</span><br />    <span style="color: #0000ff">&lt;</span><span style="color: #800000">label</span> <span style="color: #ff0000">for</span><span style="color: #0000ff">="name"</span><span style="color: #0000ff">&gt;</span>Name: <span style="color: #0000ff">&lt;/</span><span style="color: #800000">label</span><span style="color: #0000ff">&gt;</span><br />    <span style="color: #0000ff">&lt;</span><span style="color: #800000">input</span> <span style="color: #ff0000">type</span><span style="color: #0000ff">="text"</span> <span style="color: #ff0000">name</span><span style="color: #0000ff">="name"</span> <span style="color: #ff0000">id</span><span style="color: #0000ff">="name"</span><span style="color: #0000ff">/&gt;</span><br />    <span style="color: #0000ff">&lt;</span><span style="color: #800000">input</span> <span style="color: #ff0000">type</span><span style="color: #0000ff">="submit"</span> <span style="color: #ff0000">value</span><span style="color: #0000ff">="Send"</span> <span style="color: #0000ff">/&gt;</span><br /><span style="color: #0000ff">&lt;/</span><span style="color: #800000">form</span><span style="color: #0000ff">&gt;</span><br /><span style="color: #0000ff">&lt;</span><span style="color: #800000">hr</span> <span style="color: #0000ff">/&gt;</span><br />Aquí irá el resultado: <span style="color: #0000ff">&lt;</span><span style="color: #800000">p</span> <span style="color: #0000ff">/&gt;</span><br /><span style="color: #0000ff">&lt;</span><span style="color: #800000">div</span> <span style="color: #ff0000">id</span><span style="color: #0000ff">="datadiv"</span><span style="color: #0000ff">&gt;</span><br /><span style="color: #0000ff">&lt;/</span><span style="color: #800000">div</span><span style="color: #0000ff">&gt;</span></pre>
      
      <p>
        </div> 
        
        <p>
          Fijaos en que en el tag <form> se le ha incrustado código javascript para gestionar el <em>onclick</em> y el <em>onsubmit</em> (para poder realizar el envío via ajax).
        </p>
        
        <p>
          Bien… y <strong>esta misma vista (idéntica) que código genera en MVC3?</strong> Pues el siguiente:
        </p>
        
        <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
          <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; background-color: #f4f4f4; margin: 0em; border-left-style: none; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; border-right-style: none; font-size: 8pt; overflow: visible; padding-top: 0px" id="codeSnippet"><span style="color: #0000ff">&lt;</span><span style="color: #800000">h2</span><span style="color: #0000ff">&gt;</span>Normal Ajax<span style="color: #0000ff">&lt;/</span><span style="color: #800000">h2</span><span style="color: #0000ff">&gt;</span><br /><span style="color: #0000ff">&lt;</span><span style="color: #800000">form</span> <span style="color: #ff0000">action</span><span style="color: #0000ff">="/Home/PostData"</span> <br />   <span style="color: #ff0000">data-ajax</span><span style="color: #0000ff">="true"</span> <span style="color: #ff0000">data-ajax-method</span><span style="color: #0000ff">="Post"</span> <span style="color: #ff0000">data-ajax-mode</span><span style="color: #0000ff">="replace"</span> <br />   <span style="color: #ff0000">data-ajax-update</span><span style="color: #0000ff">="#datadiv"</span> <span style="color: #ff0000">method</span><span style="color: #0000ff">="post"</span><span style="color: #0000ff">&gt;</span><br /><br />    <span style="color: #0000ff">&lt;</span><span style="color: #800000">label</span> <span style="color: #ff0000">for</span><span style="color: #0000ff">="name"</span><span style="color: #0000ff">&gt;</span>Name: <span style="color: #0000ff">&lt;/</span><span style="color: #800000">label</span><span style="color: #0000ff">&gt;</span><br />    <span style="color: #0000ff">&lt;</span><span style="color: #800000">input</span> <span style="color: #ff0000">type</span><span style="color: #0000ff">="text"</span> <span style="color: #ff0000">name</span><span style="color: #0000ff">="name"</span> <span style="color: #ff0000">id</span><span style="color: #0000ff">="name"</span><span style="color: #0000ff">/&gt;</span><br />    <span style="color: #0000ff">&lt;</span><span style="color: #800000">input</span> <span style="color: #ff0000">type</span><span style="color: #0000ff">="submit"</span> <span style="color: #ff0000">value</span><span style="color: #0000ff">="Send"</span> <span style="color: #0000ff">/&gt;</span><br /><span style="color: #0000ff">&lt;/</span><span style="color: #800000">form</span><span style="color: #0000ff">&gt;</span><br /><br /><span style="color: #0000ff">&lt;</span><span style="color: #800000">hr</span> <span style="color: #0000ff">/&gt;</span><br />Aquí irá el resultado: <span style="color: #0000ff">&lt;</span><span style="color: #800000">p</span> <span style="color: #0000ff">/&gt;</span><br /><span style="color: #0000ff">&lt;</span><span style="color: #800000">div</span> <span style="color: #ff0000">id</span><span style="color: #0000ff">="datadiv"</span><span style="color: #0000ff">&gt;</span><br /><span style="color: #0000ff">&lt;/</span><span style="color: #800000">div</span><span style="color: #0000ff">&gt;</span></pre>
          
          <p>
            </div> 
            
            <p>
              Fijaos que diferencia… No hay nada de javascript mezclado en el código. Todo es HTML. Simplemente al tag <form> se le añaden unos cuantos atributos (los que empiezan por data-ajax) que indican como se debe comportarse este formulario a nivel de Ajax.
            </p>
            
            <p>
              Y quien realiza “la magia”? Pues quien va a ser… nuestra amada jQuery, junto con una extensión de Microsoft (el fichero <em>jquery.unobtrusive-ajax.js</em>)! Para que esto funciona teneis que añadir tanto jQuery como la extensión de MS (yo los pongo en la master):
            </p>
            
            <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
              <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; background-color: #f4f4f4; margin: 0em; border-left-style: none; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; border-right-style: none; font-size: 8pt; overflow: visible; padding-top: 0px" id="codeSnippet">&lt;script src=<span style="color: #006080">"&lt;%: Url.Content("</span>~/Scripts/jquery-1.4.1.js<span style="color: #006080">") %&gt;"</span> type=<span style="color: #006080">"text/javascript"</span>&gt;&lt;/script&gt;<br />&lt;script src=<span style="color: #006080">"&lt;%: Url.Content("</span>~/Scripts/jquery.unobtrusive-ajax.js<span style="color: #006080">") %&gt;"</span> type=<span style="color: #006080">"text/javascript"</span>&gt;&lt;/script&gt;</pre>
              
              <p>
                </div> 
                
                <p>
                  Esta adaptación de los helpers en MVC3 para soportar esta característica es a lo que nos referimos cuando decimos que “ASP.NET MVC3 da soporte para <em>Unobtrusive</em> Ajax”, y es una doble gran noticia. Digo doble porque por un lado nos permite seguir usando los helpers con la garantía de que vamos a generar código “limpio” de javascript y por otro lado <strong>el helper de Ajax usa ¡por fin! jQuery</strong>. A diferencia de MVC2 donde el Helper Ajax usaba la Ajax Library de Microsoft. De hecho, aunque en los templates de proyecto se sigue poniendo, si me aceptas un consejo: bórrala y no la uses. Puedes borrarla con total tranquilidad porque en MVC3 ningún helper la usa.
                </p>
                
                <p>
                  Unobtrusive Ajax viene habilitado por defecto en los nuevos proyectos MVC3 pero lo podéis deshabilitar (y entonces generar el mismo código que en MVC2, usando la Microsoft Ajax Library). Podeis deshabilitarlo a nivel de vista o para todo el proyecto.
                </p>
                
                <p>
                  Para deshabilitarlo a nivel de vista, basta con incluir:
                </p>
                
                <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
                  <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; background-color: #f4f4f4; margin: 0em; border-left-style: none; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; border-right-style: none; font-size: 8pt; overflow: visible; padding-top: 0px" id="codeSnippet">&lt;% HtmlHelper.UnobtrusiveJavaScriptEnabled = <span style="color: #0000ff">false</span>; %&gt;</pre>
                  
                  <p>
                    </div> 
                    
                    <p>
                      Para deshabilitarlo para todo el proyecto, puedes incluir ese mismo código en el global.asax.cs o bien usar web.config:
                    </p>
                    
                    <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
                      <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; background-color: #f4f4f4; margin: 0em; border-left-style: none; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; border-right-style: none; font-size: 8pt; overflow: visible; padding-top: 0px" id="codeSnippet"><span style="color: #0000ff">&lt;</span><span style="color: #800000">configuration</span><span style="color: #0000ff">&gt;</span><br />   <span style="color: #0000ff">&lt;</span><span style="color: #800000">appSettings</span><span style="color: #0000ff">&gt;</span><br />      <span style="color: #0000ff">&lt;</span><span style="color: #800000">add</span> <span style="color: #ff0000">key</span><span style="color: #0000ff">="UnobtrusiveJavaScriptEnabled"</span> <span style="color: #ff0000">value</span><span style="color: #0000ff">="false"</span><span style="color: #0000ff">/&gt;</span><br />   <span style="color: #0000ff">&lt;/</span><span style="color: #800000">appSettings</span><span style="color: #0000ff">&gt;</span><br /><span style="color: #0000ff">&lt;/</span><span style="color: #800000">configuration</span><span style="color: #0000ff">&gt;</span> </pre>
                      
                      <p>
                        </div> 
                        
                        <p>
                          Lo mismo para habilitarlo. Si <strong>no</strong> aparece la entrada <em>UnobtrusiveJavaScriptEnabled</em> en el <appSettings> el valor por defecto es <strong>false</strong>. Es por eso que si haces un upgrade de un proyecto de MVC2 a MVC3, no tendrás esta entrada en el web.config y por eso Unobtrusive Ajax estará deshabilitado!
                        </p>
                        
                        <p>
                          Un saludo!
                        </p>
                        
                        <p>
                          PD: El hecho de que los atributos que se usan para que Unobtrusive Ajax funcione empiecen por “data-“ es porque HTML5 reserva estos atributos “para usos propios de los scripts del site”, tal y como podéis leer en la <a href="http://dev.w3.org/html5/spec/elements.html#custom-data-attribute">especificación de Custom Data Attributes</a>.
                        </p>