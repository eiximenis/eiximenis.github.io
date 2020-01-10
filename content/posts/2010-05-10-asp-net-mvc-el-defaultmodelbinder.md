---
title: 'ASP.NET MVC: El DefaultModelBinder'

author: eiximenis

date: 2010-05-10T17:01:00+00:00
geeks_url: /?p=1509
geeks_visits:
  - 3095
geeks_ms_views:
  - 2742
categories:
  - Uncategorized

---
En el <a target="_blank" href="/blogs/etomas/archive/2010/05/07/asp-net-mvc-valueproviders.aspx" rel="noopener noreferrer">post anterior vimos que eran los Value Providers</a> de ASP.NET MVC. En éste, lo que vamos a ver es el DefaultModelBinder y algunas de sus &ldquo;interioridades&rdquo;...

> **Disclaimer**: Al igual que el post anterior, este asume conocimientos básicos de ASP.NET MVC, así como de http en general.

<!--more-->

Antes que nada el repaso rápido: Cuando un controlador recibe en una acción un objeto del modelo, ASP.NET MVC es capaz de realizar el binding entre los datos contenidos en la request y el objeto que espera el controlador. Por un lado los value providers se encargan de leer los datos de la request y guardarlos &ldquo;en una estructura común&rdquo; y por otro los model binders crean el objeto del modelo a partir de dicha estructura común.

Hoy vamos a diseccionar el <a target="_blank" href="http://msdn.microsoft.com/en-us/library/system.web.mvc.defaultmodelbinder.aspx" rel="noopener noreferrer">DefaultModelBinder</a>__, el model binder que trae por defecto ASP.NET MVC. 

**Colecciones**

Vamos a suponer la siguiente clase del modelo:

<div id="codeSnippetWrapper" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; width: 97.5%; font-family: 'Courier New', courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; cursor: text; border: silver 1px solid; padding: 4px;">
  <pre id="codeSnippet" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 0em; width: 100%; font-family: 'Courier New', courier, monospace; direction: ltr; color: black; font-size: 8pt; overflow: visible; border-style: none; padding: 0px;"><span style="color: #0000ff">public</span> <span style="color: #0000ff">class</span> Persona<br />{<br />    <span style="color: #0000ff">public</span> IEnumerable&lt;<span style="color: #0000ff">string</span>&gt; Telefonos { get; set; }<br />    <span style="color: #0000ff">public</span> <span style="color: #0000ff">string</span> Nombre { get; set; }<br />    <span style="color: #0000ff">public</span> <span style="color: #0000ff">int</span> Edad { get; set; }<br />}</pre>
  
  <p>
    </div> 
    
    <p>
      Asumid que tenemos un controlador <em>Personas</em> con una acción <em>Crear</em> que recibe un objeto <em>Persona</em>. Vamos a ver como el DefaultModelBinder es capaz de mapear las propiedades, incluída la colección <em>Telefonos. </em>Para ello tengo simplemente la siguiente vista:
    </p>
    
    <div id="codeSnippetWrapper" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; width: 97.5%; font-family: 'Courier New', courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; cursor: text; border: silver 1px solid; padding: 4px;">
      <pre id="codeSnippet" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 0em; width: 100%; font-family: 'Courier New', courier, monospace; direction: ltr; color: black; font-size: 8pt; overflow: visible; border-style: none; padding: 0px;"><span style="color: #0000ff">&lt;</span><span style="color: #800000">form</span> <span style="color: #ff0000">action</span><span style="color: #0000ff">="/Personas/Crear"</span> <span style="color: #ff0000">method</span><span style="color: #0000ff">="post"</span> <span style="color: #0000ff">&gt;</span><br />    <span style="color: #0000ff">&lt;</span><span style="color: #800000">fieldset</span><span style="color: #0000ff">&gt;</span><br />        <span style="color: #0000ff">&lt;</span><span style="color: #800000">legend</span><span style="color: #0000ff">&gt;</span>Fields<span style="color: #0000ff">&lt;/</span><span style="color: #800000">legend</span><span style="color: #0000ff">&gt;</span><br />        <span style="color: #0000ff">&lt;</span><span style="color: #800000">div</span> <span style="color: #ff0000">class</span><span style="color: #0000ff">="editor-label"</span><span style="color: #0000ff">&gt;</span><br />            <span style="color: #0000ff">&lt;</span><span style="color: #800000">label</span> <span style="color: #ff0000">for</span><span style="color: #0000ff">="Nombre"</span><span style="color: #0000ff">&gt;</span>Nombre<span style="color: #0000ff">&lt;/</span><span style="color: #800000">label</span><span style="color: #0000ff">&gt;</span><br />            <span style="color: #0000ff">&lt;</span><span style="color: #800000">input</span> <span style="color: #ff0000">type</span><span style="color: #0000ff">="text"</span> <span style="color: #ff0000">name</span><span style="color: #0000ff">="Nombre"</span> <span style="color: #0000ff">/&gt;</span><br />        <span style="color: #0000ff">&lt;/</span><span style="color: #800000">div</span><span style="color: #0000ff">&gt;</span><br />        <span style="color: #0000ff">&lt;</span><span style="color: #800000">div</span> <span style="color: #ff0000">class</span><span style="color: #0000ff">="editor-label"</span><span style="color: #0000ff">&gt;</span><br />            <span style="color: #0000ff">&lt;</span><span style="color: #800000">label</span> <span style="color: #ff0000">for</span><span style="color: #0000ff">="Edad"</span><span style="color: #0000ff">&gt;</span>Edad<span style="color: #0000ff">&lt;/</span><span style="color: #800000">label</span><span style="color: #0000ff">&gt;</span><br />            <span style="color: #0000ff">&lt;</span><span style="color: #800000">input</span> <span style="color: #ff0000">type</span><span style="color: #0000ff">="text"</span> <span style="color: #ff0000">name</span><span style="color: #0000ff">="Edad"</span> <span style="color: #0000ff">/&gt;</span><br />        <span style="color: #0000ff">&lt;/</span><span style="color: #800000">div</span><span style="color: #0000ff">&gt;</span><br />        <span style="color: #0000ff">&lt;</span><span style="color: #800000">div</span> <span style="color: #ff0000">class</span><span style="color: #0000ff">="editor-label"</span><span style="color: #0000ff">&gt;</span><br />            <span style="color: #0000ff">&lt;</span><span style="color: #800000">label</span> <span style="color: #ff0000">for</span><span style="color: #0000ff">="Telefonos"</span><span style="color: #0000ff">&gt;</span>Telf 1<span style="color: #0000ff">&lt;/</span><span style="color: #800000">label</span><span style="color: #0000ff">&gt;</span><br />            <span style="color: #0000ff">&lt;</span><span style="color: #800000">input</span> <span style="color: #ff0000">type</span><span style="color: #0000ff">="text"</span> <span style="color: #ff0000">name</span><span style="color: #0000ff">="Telefonos"</span> <span style="color: #0000ff">/&gt;</span><br />        <span style="color: #0000ff">&lt;/</span><span style="color: #800000">div</span><span style="color: #0000ff">&gt;</span><br />        <span style="color: #0000ff">&lt;</span><span style="color: #800000">div</span> <span style="color: #ff0000">class</span><span style="color: #0000ff">="editor-label"</span><span style="color: #0000ff">&gt;</span><br />            <span style="color: #0000ff">&lt;</span><span style="color: #800000">label</span> <span style="color: #ff0000">for</span><span style="color: #0000ff">="Telefonos"</span><span style="color: #0000ff">&gt;</span>Telf 2<span style="color: #0000ff">&lt;/</span><span style="color: #800000">label</span><span style="color: #0000ff">&gt;</span><br />            <span style="color: #0000ff">&lt;</span><span style="color: #800000">input</span> <span style="color: #ff0000">type</span><span style="color: #0000ff">="text"</span> <span style="color: #ff0000">name</span><span style="color: #0000ff">="Telefonos"</span> <span style="color: #0000ff">/&gt;</span><br />        <span style="color: #0000ff">&lt;/</span><span style="color: #800000">div</span><span style="color: #0000ff">&gt;</span><br />        <span style="color: #0000ff">&lt;</span><span style="color: #800000">div</span> <span style="color: #ff0000">class</span><span style="color: #0000ff">="editor-label"</span><span style="color: #0000ff">&gt;</span><br />            <span style="color: #0000ff">&lt;</span><span style="color: #800000">label</span> <span style="color: #ff0000">for</span><span style="color: #0000ff">="Telefonos"</span><span style="color: #0000ff">&gt;</span>Telf 3<span style="color: #0000ff">&lt;/</span><span style="color: #800000">label</span><span style="color: #0000ff">&gt;</span><br />            <span style="color: #0000ff">&lt;</span><span style="color: #800000">input</span> <span style="color: #ff0000">type</span><span style="color: #0000ff">="text"</span> <span style="color: #ff0000">name</span><span style="color: #0000ff">="Telefonos"</span> <span style="color: #0000ff">/&gt;</span><br />        <span style="color: #0000ff">&lt;/</span><span style="color: #800000">div</span><span style="color: #0000ff">&gt;</span><br />        <span style="color: #0000ff">&lt;</span><span style="color: #800000">input</span> <span style="color: #ff0000">type</span><span style="color: #0000ff">="submit"</span> <span style="color: #0000ff">/&gt;</span><br /><span style="color: #0000ff">&lt;/</span><span style="color: #800000">form</span><span style="color: #0000ff">&gt;</span></pre>
      
      <p>
        </div> 
        
        <p>
          Fijaos que hay 3 <input type=&rdquo;text&rdquo; name=&rdquo;Telefono&rdquo;>. Esto <strong>no</strong> es incorrecto, el parámetro name de un <input> puede estar repetido para indicar campos con más de un valor... Si introduzco datos y envío el formulario los campos POST de la request son:
        </p>
        
        <p>
          <code>Content-Type: application/x-www-form-urlencoded &lt;br /></code><code>Content-Length: 90 &lt;br /></code><code>Nombre=Nombre&Edad=12&Telefonos=%2B34+555222&Telefonos=%2B34+666112&Telefonos=%2B34+777114</code>
        </p>
        
        <p>
          Honestamente... el navegador no es que haga gran cosa: simplemente tenemos 5 campos POST: Nombre, Edad y 3 campos Telefonos. Veamos ahora lo que ocurre en el lado del DefaultModelBinder.
        </p>
        
        <p>
          El método dentro del DefaultModelBinder que obtiene el valor de una propiedad se llama <em>GetPropertyValue</em> y tiene la siguiente firma:
        </p>
        
        <div id="codeSnippetWrapper" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; width: 97.5%; font-family: 'Courier New', courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; cursor: text; border: silver 1px solid; padding: 4px;">
          <pre id="codeSnippet" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 0em; width: 100%; font-family: 'Courier New', courier, monospace; direction: ltr; color: black; font-size: 8pt; overflow: visible; border-style: none; padding: 0px;"><span style="color: #0000ff">protected</span> <span style="color: #0000ff">virtual</span> <span style="color: #0000ff">object</span> GetPropertyValue(ControllerContext controllerContext, ModelBindingContext bindingContext, PropertyDescriptor propertyDescriptor, IModelBinder propertyBinder)</pre>
          
          <p>
            </div> 
            
            <p>
              De todos los parámetros que recibe este método nos interesan básicamente dos:
            </p>
            
            <ol>
              <li>
                bindingContext: Un objeto de la clase <a target="_blank" href="http://msdn.microsoft.com/en-us/library/system.web.mvc.modelbindingcontext.aspx" rel="noopener noreferrer">ModelBindingContext</a> que contiene la información necesaria para poder realizar el binding: Proporciona acceso a los value providers, al ModelState (donde se guarda información relativa a la <em>validación</em> del modelo) y a los metadatos del modelo.
              </li>
              <li>
                propertyDescriptor: Un objeto de la clase <a target="_blank" href="http://msdn.microsoft.com/es-es/library/system.componentmodel.propertydescriptor(VS.80).aspx" rel="noopener noreferrer">PropertyDescriptor</a> con información sobre la <em>propiedad</em> del modelo de la cual queremos obtener el valor (básicamente el <em>tipo</em> de la propiedad).
              </li>
            </ol>
            
            <p>
              A partir de la información del <em>bindingContext</em> el DefaultModelBinder puede preguntar a los value providers que le den el valor del campo que contiene el valor de la propiedad (por defecto si estamos rellenando la propiedad <em>Telefonos</em> el DefaultModelBinder preguntará a los value providers por el valor de <em>Telefonos</em>). El DefaultModelBinder usará la información del propertyDescriptor para convertir el valor devuelto por los value providers al tipo del que sea la propiedad.
            </p>
            
            <p>
              Luego el framework llama al método <em>SetPropertyValue</em> del propio model binder, que tiene esta firma:
            </p>
            
            <div id="codeSnippetWrapper" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; width: 97.5%; font-family: 'Courier New', courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; cursor: text; border: silver 1px solid; padding: 4px;">
              <pre id="codeSnippet" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 0em; width: 100%; font-family: 'Courier New', courier, monospace; direction: ltr; color: black; font-size: 8pt; overflow: visible; border-style: none; padding: 0px;"><span style="color: #0000ff">protected</span> <span style="color: #0000ff">virtual</span> <span style="color: #0000ff">void</span> SetProperty(ControllerContext controllerContext, ModelBindingContext bindingContext, System.ComponentModel.PropertyDescriptor propertyDescriptor, <span style="color: #0000ff">object</span> <span style="color: #0000ff">value</span>)</pre>
              
              <p>
                </div> 
                
                <p>
                  Los parámetros son más o menos los mismos que GetPropertyValue, con la salvedad de que recibimos también el objeto devuelto por GetPropertyValue. En este método &ldquo;simplemente&rdquo; asignamos el valor a la propiedad (podemos acceder al modelo a través del la propiedad Model del parámetro bindingContext).
                </p>
                
                <p>
                  Bueno, en el caso de la petición POST que nos ocupa esto es lo que devuelve el método GetPropertyValue de la propiedad Teléfonos:
                </p>
                
                <p>
                  <a href="/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_326BE3EE.png"><img height="78" width="166" src="/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_02CDC562.png" alt="image" border="0" title="image" style="border-bottom: 0px; border-left: 0px; display: inline; border-top: 0px; border-right: 0px" /></a>
                </p>
                
                <p>
                  El DefaultModelBinder, a partir de la información de los value providers, crea un array de cadenas con el valor de los tres teléfonos (recordad que la propiedad estava declarada como IEnumerable<string> en el modelo).
                </p>
                
                <p>
                  Hemos visto como a partir de una petición POST con varios campos &ldquo;Telefonos&rdquo; el DefaultModelBinder era capaz de enlazar esto a una variable IEnumerable<string> del modelo... Sigamos jugando un poco a ver que pasa... 🙂
                </p>
                
                <p>
                  Imaginad que hago la siguiente modificación en la vista:
                </p>
                
                <div id="codeSnippetWrapper" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; width: 97.5%; font-family: 'Courier New', courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; cursor: text; border: silver 1px solid; padding: 4px;">
                  <pre id="codeSnippet" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 0em; width: 100%; font-family: 'Courier New', courier, monospace; direction: ltr; color: black; font-size: 8pt; overflow: visible; border-style: none; padding: 0px;"><span style="color: #0000ff">&lt;</span><span style="color: #800000">div</span> <span style="color: #ff0000">class</span><span style="color: #0000ff">="editor-label"</span><span style="color: #0000ff">&gt;</span><br />    <span style="color: #0000ff">&lt;</span><span style="color: #800000">label</span> <span style="color: #ff0000">for</span><span style="color: #0000ff">="Telefonos"</span><span style="color: #0000ff">&gt;</span>Telf 1<span style="color: #0000ff">&lt;/</span><span style="color: #800000">label</span><span style="color: #0000ff">&gt;</span><br />    <span style="color: #0000ff">&lt;</span><span style="color: #800000">input</span> <span style="color: #ff0000">type</span><span style="color: #0000ff">="text"</span> <span style="color: #ff0000">name</span><span style="color: #0000ff">="Telefonos[0]"</span> <span style="color: #0000ff">/&gt;</span><br /><span style="color: #0000ff">&lt;/</span><span style="color: #800000">div</span><span style="color: #0000ff">&gt;</span><br /><span style="color: #0000ff">&lt;</span><span style="color: #800000">div</span> <span style="color: #ff0000">class</span><span style="color: #0000ff">="editor-label"</span><span style="color: #0000ff">&gt;</span><br />    <span style="color: #0000ff">&lt;</span><span style="color: #800000">label</span> <span style="color: #ff0000">for</span><span style="color: #0000ff">="Telefonos"</span><span style="color: #0000ff">&gt;</span>Telf 2<span style="color: #0000ff">&lt;/</span><span style="color: #800000">label</span><span style="color: #0000ff">&gt;</span><br />    <span style="color: #0000ff">&lt;</span><span style="color: #800000">input</span> <span style="color: #ff0000">type</span><span style="color: #0000ff">="text"</span> <span style="color: #ff0000">name</span><span style="color: #0000ff">="Telefonos[1]"</span> <span style="color: #0000ff">/&gt;</span><br /><span style="color: #0000ff">&lt;/</span><span style="color: #800000">div</span><span style="color: #0000ff">&gt;</span><br /><span style="color: #0000ff">&lt;</span><span style="color: #800000">div</span> <span style="color: #ff0000">class</span><span style="color: #0000ff">="editor-label"</span><span style="color: #0000ff">&gt;</span><br />    <span style="color: #0000ff">&lt;</span><span style="color: #800000">label</span> <span style="color: #ff0000">for</span><span style="color: #0000ff">="Telefonos"</span><span style="color: #0000ff">&gt;</span>Telf 3<span style="color: #0000ff">&lt;/</span><span style="color: #800000">label</span><span style="color: #0000ff">&gt;</span><br />    <span style="color: #0000ff">&lt;</span><span style="color: #800000">input</span> <span style="color: #ff0000">type</span><span style="color: #0000ff">="text"</span> <span style="color: #ff0000">name</span><span style="color: #0000ff">="Telefonos[2]"</span> <span style="color: #0000ff">/&gt;</span><br /><span style="color: #0000ff">&lt;/</span><span style="color: #800000">div</span><span style="color: #0000ff">&gt;</span></pre>
                  
                  <p>
                    </div> 
                    
                    <p>
                      Es decir, en lugar de tener tres campos cuyo <em>name</em> es Telefonos, tengo tres campos cuyo <em>name</em> es Telefonos[0], Telefonos[1] y Telefonos[2]. Si relleno los datos y envio la petición, ahora los datos POST tienen la forma:
                    </p>
                    
                    <p>
                      <code>Content-Type: application/x-www-form-urlencoded &lt;br /></code><code>Content-Length: 105 &lt;br /></code><code>Nombre=eiximenis&Edad=20&Telefonos%5B0%5D=%2B34+111&Telefonos%5B1%5D=%2B34+222&Telefonos%5B2%5D=%2B34</code><code>+333</code>
                    </p>
                    
                    <p>
                      Ahora en los datos POST tenemos (además de Nombre y Edad) tres campos más, llamados Telefonos%5B0%5D, Telefonos%5B1%5D, y Telefonos%5B2%5D (%5B es el código para [ y %5D es el código para ]).
                    </p>
                    
                    <p>
                      Pues bien: eso funciona correctamente... el DefaultModelBinder en el GetPropertyValue para la propiedad Telefonos hace lo siguiente (es una simplificación de lo que ocurre realmente, pero es suficiente para entender el concepto):
                    </p>
                    
                    <ol>
                      <li>
                        Pregunta a los value providers para el valor de Telefonos y el resultado es null (no hay ningún campo &ldquo;Telefonos&rdquo;)
                      </li>
                      <li>
                        Como el DefaultModelBinder sabe que está enlazando una colección, pregunta a los value providers por el valor de Telefonos[0] y si existe lo añade a la colección de la propiedad Telefonos y pregunta por Telefonos[1]... y así sucesivamente.
                      </li>
                    </ol>
                    
                    <p>
                      No me puteéis el pobre DefaultModelBinder, que es bastante inteligente pero tampoco es dios... eso <strong>no</strong> funciona:
                    </p>
                    
                    <div id="codeSnippetWrapper" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; width: 97.5%; font-family: 'Courier New', courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; cursor: text; border: silver 1px solid; padding: 4px;">
                      <pre id="codeSnippet" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 0em; width: 100%; font-family: 'Courier New', courier, monospace; direction: ltr; color: black; font-size: 8pt; overflow: visible; border-style: none; padding: 0px;"><span style="color: #0000ff">&lt;</span><span style="color: #800000">div</span> <span style="color: #ff0000">class</span><span style="color: #0000ff">="editor-label"</span><span style="color: #0000ff">&gt;</span><br />    <span style="color: #0000ff">&lt;</span><span style="color: #800000">label</span> <span style="color: #ff0000">for</span><span style="color: #0000ff">="Telefonos"</span><span style="color: #0000ff">&gt;</span>Telf 1<span style="color: #0000ff">&lt;/</span><span style="color: #800000">label</span><span style="color: #0000ff">&gt;</span><br />    <span style="color: #0000ff">&lt;</span><span style="color: #800000">input</span> <span style="color: #ff0000">type</span><span style="color: #0000ff">="text"</span> <span style="color: #ff0000">name</span><span style="color: #0000ff">="Telefonos[1]"</span> <span style="color: #0000ff">/&gt;</span><br /><span style="color: #0000ff">&lt;/</span><span style="color: #800000">div</span><span style="color: #0000ff">&gt;</span><br /><span style="color: #0000ff">&lt;</span><span style="color: #800000">div</span> <span style="color: #ff0000">class</span><span style="color: #0000ff">="editor-label"</span><span style="color: #0000ff">&gt;</span><br />    <span style="color: #0000ff">&lt;</span><span style="color: #800000">label</span> <span style="color: #ff0000">for</span><span style="color: #0000ff">="Telefonos"</span><span style="color: #0000ff">&gt;</span>Telf 2<span style="color: #0000ff">&lt;/</span><span style="color: #800000">label</span><span style="color: #0000ff">&gt;</span><br />    <span style="color: #0000ff">&lt;</span><span style="color: #800000">input</span> <span style="color: #ff0000">type</span><span style="color: #0000ff">="text"</span> <span style="color: #ff0000">name</span><span style="color: #0000ff">="Telefonos[2]"</span> <span style="color: #0000ff">/&gt;</span><br /><span style="color: #0000ff">&lt;/</span><span style="color: #800000">div</span><span style="color: #0000ff">&gt;</span></pre>
                      
                      <p>
                        </div> 
                        
                        <p>
                          En este caso NO tenemos ni campo &ldquo;Telefonos&rdquo;, ni campo &ldquo;Telefonos[0]&#8221; en la request, así que el DefaultModelBinder entenderá que la propiedad Telefonos no ha sido informado y la pondrá a null.
                        </p>
                        
                        <p>
                          <strong>Objetos compuestos</strong>
                        </p>
                        
                        <p>
                          Imaginad ahora que tenemos nuestro modelo definido del siguiente modo:
                        </p>
                        
                        <div id="codeSnippetWrapper" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; width: 97.5%; font-family: 'Courier New', courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; cursor: text; border: silver 1px solid; padding: 4px;">
                          <pre id="codeSnippet" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 0em; width: 100%; font-family: 'Courier New', courier, monospace; direction: ltr; color: black; font-size: 8pt; overflow: visible; border-style: none; padding: 0px;"><span style="color: #0000ff">public</span> <span style="color: #0000ff">class</span> Persona<br />{<br />    <span style="color: #0000ff">public</span> IEnumerable&lt;<span style="color: #0000ff">string</span>&gt; Telefonos { get; set; }<br />    <span style="color: #0000ff">public</span> <span style="color: #0000ff">string</span> Nombre { get; set; }<br />    <span style="color: #0000ff">public</span> <span style="color: #0000ff">int</span> Edad { get; set; }<br /><br />    <span style="color: #0000ff">public</span> Direccion Direccion { get; set; }<br />}<br /><br /><span style="color: #0000ff">public</span> <span style="color: #0000ff">class</span> Direccion<br />{<br />    <span style="color: #0000ff">public</span> <span style="color: #0000ff">string</span> Calle { get; set; }<br />    <span style="color: #0000ff">public</span> <span style="color: #0000ff">string</span> Ciudad { get; set; }<br />}</pre>
                          
                          <p>
                            </div> 
                            
                            <p>
                              Como se las apaña el DefaultModelBinder para enlazar la propiedad Direccion que es un objeto completo??
                            </p>
                            
                            <p>
                              Para ver como se las puede apañar el DefaultModelBinder, vamos a dejar que el propio framework nos ayude 🙂 Par ello añado las siguientes líneas dentro del <form> de la vista:
                            </p>
                            
                            <div id="codeSnippetWrapper" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; width: 97.5%; font-family: 'Courier New', courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; cursor: text; border: silver 1px solid; padding: 4px;">
                              <pre id="codeSnippet" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 0em; width: 100%; font-family: 'Courier New', courier, monospace; direction: ltr; color: black; font-size: 8pt; overflow: visible; border-style: none; padding: 0px;"><span style="color: #006080">&lt;</span><span style="color: #0000ff">div</span> class="editor-<span style="color: #0000ff">label</span>"<span style="color: #006080">&gt;</span><br />    <span style="color: #006080">&lt;</span>%: Html<span style="color: #cc6633">.LabelFor</span>(x=<span style="color: #006080">&gt;</span>x<span style="color: #cc6633">.Direccion</span>) %<span style="color: #006080">&gt;</span><br />    <span style="color: #006080">&lt;</span>%: Html<span style="color: #cc6633">.EditorFor</span>(x=<span style="color: #006080">&gt;</span>x<span style="color: #cc6633">.Direccion</span>) %<span style="color: #006080">&gt;</span><br /><span style="color: #006080">&lt;</span>/<span style="color: #0000ff">div</span><span style="color: #006080">&gt;</span></pre>
                              
                              <p>
                                </div> 
                                
                                <p>
                                  Recordad que Html.LabelFor lo que hace es crear una <label> vinculada a la propiedad del modelo que se le pasa vía la expresión lambda. Por otro lado Html.EditorFor lo que hace es crear un &ldquo;editor&rdquo; para la propiedad del modelo que se le indica... El framework usa los metadatos del modelo para saber que tipo de editor es mejor (además de que nosotros podemos definir el tipo editor que queramos, pero eso es <em>otra</em> historia). Cuál creeis que es el &ldquo;editor por defecto&rdquo; para la propiedad <em>Direccion</em> del modelo, que es un objeto de la clase <em>Direccion</em>?
                                </p>
                                
                                <p>
                                  Si habéis respondido &ldquo;dos textboxes, uno para la propiedad Calle y el otro para la propiedad Ciudad&rdquo; habeis dado en el clavo... este es el código HTML que me genera la llamada a Html.EditorFor:
                                </p>
                                
                                <div id="codeSnippetWrapper" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; width: 97.5%; font-family: 'Courier New', courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; cursor: text; border: silver 1px solid; padding: 4px;">
                                  <pre id="codeSnippet" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 0em; width: 100%; font-family: 'Courier New', courier, monospace; direction: ltr; color: black; font-size: 8pt; overflow: visible; border-style: none; padding: 0px;"><span style="color: #0000ff">&lt;</span><span style="color: #800000">div</span> <span style="color: #ff0000">class</span><span style="color: #0000ff">="editor-field"</span><span style="color: #0000ff">&gt;</span><br /><span style="color: #0000ff">&lt;</span><span style="color: #800000">input</span> <span style="color: #ff0000">class</span><span style="color: #0000ff">="text-box single-line"</span> <span style="color: #ff0000">id</span><span style="color: #0000ff">="Direccion_Calle"</span> <span style="color: #ff0000">name</span><span style="color: #0000ff">="Direccion.Calle"</span> <span style="color: #ff0000">type</span><span style="color: #0000ff">="text"</span> <span style="color: #ff0000">value</span><span style="color: #0000ff">=""</span> <span style="color: #0000ff">/&gt;</span><br /><span style="color: #0000ff">&lt;/</span><span style="color: #800000">div</span><span style="color: #0000ff">&gt;</span><br /><span style="color: #0000ff">&lt;</span><span style="color: #800000">div</span> <span style="color: #ff0000">class</span><span style="color: #0000ff">="editor-label"</span><span style="color: #0000ff">&gt;</span><br /><span style="color: #0000ff">&lt;</span><span style="color: #800000">label</span> <span style="color: #ff0000">for</span><span style="color: #0000ff">="Direccion_Ciudad"</span><span style="color: #0000ff">&gt;</span>Ciudad<span style="color: #0000ff">&lt;/</span><span style="color: #800000">label</span><span style="color: #0000ff">&gt;</span><br /><span style="color: #0000ff">&lt;/</span><span style="color: #800000">div</span><span style="color: #0000ff">&gt;</span><br /><span style="color: #0000ff">&lt;</span><span style="color: #800000">div</span> <span style="color: #ff0000">class</span><span style="color: #0000ff">="editor-field"</span><span style="color: #0000ff">&gt;</span><br /><span style="color: #0000ff">&lt;</span><span style="color: #800000">input</span> <span style="color: #ff0000">class</span><span style="color: #0000ff">="text-box single-line"</span> <span style="color: #ff0000">id</span><span style="color: #0000ff">="Direccion_Ciudad"</span> <span style="color: #ff0000">name</span><span style="color: #0000ff">="Direccion.Ciudad"</span> <span style="color: #ff0000">type</span><span style="color: #0000ff">="text"</span> <span style="color: #ff0000">value</span><span style="color: #0000ff">=""</span> <span style="color: #0000ff">/&gt;</span> <br /><span style="color: #0000ff">&lt;/</span><span style="color: #800000">div</span><span style="color: #0000ff">&gt;</span><br /></pre>
                                  
                                  <p>
                                    </div> 
                                    
                                    <p>
                                      Exacto, dos <input type=&rdquo;text&rdquo;> uno para la propiedad Calle y el otro para la propiedad Ciudad... pero os habéis fijado en los <em>name</em>?
                                    </p>
                                    
                                    <ul>
                                      <li>
                                        Direccion.Calle
                                      </li>
                                      <li>
                                        Direccion.Ciudad
                                      </li>
                                    </ul>
                                    
                                    <p>
                                      No vamos a entrar dentro del ciclo de vida entero del DefaultModelBinder (porque daría para <em>otro</em> post entero) y pienso que nos basta con esta idea: El DefaultModelBinder es capaz de procesar modelos complejos siempre y cuando los value providers tengan valores cuyo nombre sea propiedad.propiedad.propiedad...
                                    </p>
                                    
                                    <p>
                                      <strong>Validación del modelo</strong>
                                    </p>
                                    
                                    <p>
                                      El DefaultModelBinder no sólo crea objetos del modelo y los enlaza... también es el responsable de <strong>validar</strong> que el modelo cumpla con las restricciones que tiene. Las restricciones forman parte de los <em>metadatos</em> del modelo y por lo tanto están accesibles a través del parámetro <em>bindingContext</em> (propiedad ModelMetaData). El DefaultModelBinder comprueba que los valores de las propiedades satisfacen las restricciones que indican los metadatos. Si no es el caso, utiliza el <em><a target="_blank" href="http://msdn.microsoft.com/en-us/library/system.web.mvc.modelstatedictionary.aspx" rel="noopener noreferrer">ModelState</a></em> (accesible a través del bindingContext) y añade un error (llamando al método AddModelError) para indicar que la propiedad determinada no cumple la restricción indicada.
                                    </p>
                                    
                                    <p>
                                      El <em>proveedor de metadatos para el modelo</em> que usa ASP.NET MVC por defecto es DataAnnotations, pero nosotros podríamos crearnos nuestros propios proveedores de metadatos (sí, sí... ASP.NET MVC es sumamente extensible). Pero eso también ya es una historia para otro post... 🙂
                                    </p>
                                    
                                    <p>
                                      <strong>Resumiendo...</strong>
                                    </p>
                                    
                                    <p>
                                      El Model Binder es el encargado de crear un objeto del tipo correspondiente al que espera la acción de destino del controlador y de rellenarlo con los valores que provienen de la petición, aunque para ello <strong>no</strong> consulta los datos de la petición, sinó que consulta los datos de los value providers (que son quienes <em>previamente</em> han consultado la petición). Eso independiza al Model Binder de la forma en cómo los datos estén codificados en la petición. Finalmente el Model Binder también valida que el modelo cumpla las restricciones indicadas en los metadatos asociados.
                                    </p>
                                    
                                    <p>
                                      Hemos visto por encima como funciona el DefaultModelBinder, el Model Binder que viene por defecto en ASP.NET MVC... Como (casi) todo en ASP.NET MVC podemos cambiar el DefaultModelBinder por otro a nuestro antojo, o lo que suele ser más normal, asociar un Model Binder específico a un tipo de modelo... Pero eso lo veremos en otro post... 🙂
                                    </p>
                                    
                                    <p>
                                      Un saludo!
                                    </p>