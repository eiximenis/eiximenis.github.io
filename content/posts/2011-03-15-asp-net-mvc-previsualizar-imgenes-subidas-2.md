---
title: 'ASP.NET MVC: Previsualizar imágenes subidas (2)'
description: 'ASP.NET MVC: Previsualizar imágenes subidas (2)'
author: eiximenis

date: 2011-03-15T14:54:10+00:00
geeks_url: /?p=1561
geeks_visits:
  - 4334
geeks_ms_views:
  - 2145
categories:
  - Uncategorized

---
Buenas! Donde dije digo, digo Diego… Sí, ya sé que dije que el segundo post sería como hacerlo con Ajax, pero bueno… la culpa es de twitter, concretamente de <a href="http://twitter.com/pablonete" target="_blank" rel="noopener noreferrer">@pablonete</a> con el que hemos empezado a hablar sobre si es posible evitar el guardar la imágen físicamente en el servidor. Hay un mecanismo obvio, que es usar la sesión (guardar el array de bytes que conforman la imágen en la sesión). Pero… hay otra? Pues sí: usar data urls!

**Data urls**

Lo que mucha gente no conoce es que el formato de URL permite _incrustar_ datos que son _interpretados_ por el navegador como si se los hubiese descargado externamente. P.ej. la siguiente url es válida:

data:image/jpeg;base64,xxxxxxx

donde xxxxxxx es la codificación en base64 de la imágen.

P.ej. eso es totalmente correcto:

<div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
  <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; background-color: #f4f4f4; margin: 0em; border-left-style: none; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; border-right-style: none; font-size: 8pt; overflow: visible; padding-top: 0px" id="codeSnippet"><span style="color: #0000ff">&lt;</span><span style="color: #800000">img</span> <span style="color: #ff0000">src</span>=”<span style="color: #ff0000">data:image</span>/<span style="color: #ff0000">jpeg</span>;<span style="color: #ff0000">base64</span>,<span style="color: #ff0000">xxxxxx</span>” <span style="color: #0000ff">/&gt;</span></pre>
  
  <p>
    </div> 
    
    <p>
      Veamos como podríamos modificar nuestro proyecto anterior, para usar data urls en lugar de un fichero temporal en el servidor… Fijaos que eso <strong>sólo evita guardar el fichero, la imagen debe ser subida al servidor</strong>.
    </p>
    
    <p>
      Las modificaciones son muy simples, por un lado primero vamos a modificar la acción del controlador, para que quede así:
    </p>
    
    <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
      <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; background-color: #f4f4f4; margin: 0em; border-left-style: none; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; border-right-style: none; font-size: 8pt; overflow: visible; padding-top: 0px" id="codeSnippet">[HttpPost]<br /><span style="color: #0000ff">public</span> ActionResult SendImage(HttpPostedFileBase img, <span style="color: #0000ff">string</span> base64, <span style="color: #0000ff">string</span> contenttype)<br />{<br />    <span style="color: #0000ff">if</span> (base64 != <span style="color: #0000ff">null</span> && contenttype != <span style="color: #0000ff">null</span> && img==<span style="color: #0000ff">null</span>)<br />    {<br />        <span style="color: #008000">// Aquí podríamos guardar la imagen (en base64 tenemos los datos)</span><br />        <span style="color: #0000ff">return</span> View(<span style="color: #006080">"Step2"</span>);<br />    }<br />    var data = <span style="color: #0000ff">new</span> <span style="color: #0000ff">byte</span>[img.ContentLength];<br />    img.InputStream.Read(data, 0, img.ContentLength);<br />    var base64Data = Convert.ToBase64String(data);<br />    ViewBag.ImageData = base64Data;<br />    ViewBag.ImageContentType = img.ContentType;<br />    <span style="color: #0000ff">return</span> View(<span style="color: #006080">"Index"</span>);<br />}</pre>
      
      <p>
        </div> 
        
        <p>
          La acción recibe tres parámetros:
        </p>
        
        <ol>
          <li>
            img: El fichero seleccionado (contenido del <input type=”file”).
          </li>
          <li>
            base64: Codificación en base64 de la imagen <strong>que se está previsualizándo.</strong>
          </li>
          <li>
            contenttype: Content-type de la imagen <strong>que se está previsualizando.</strong>
          </li>
        </ol>
        
        <p>
          Por supuesto base64 y contenttype sólo se envían si se está previsualizando una imagen. En caso contrario valen null.
        </p>
        
        <p>
          Fijémonos lo que hace la acción, si base64 y contenttype valen null, o bien el usuario ha seleccionado una imagen nueva (img != null):
        </p>
        
        <ol>
          <li>
            Obtenemos los datos en base64 de la imagen enviada por el usuario (base64Data).
          </li>
          <li>
            Pasamos esos datos (junto con el content-type) a la vista, usando el ViewBag.
          </li>
        </ol>
        
        <p>
          Y la vista que hace? Pues poca cosa:
        </p>
        
        <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
          <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; background-color: #f4f4f4; margin: 0em; border-left-style: none; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; border-right-style: none; font-size: 8pt; overflow: visible; padding-top: 0px" id="codeSnippet"><span style="color: #0000ff">&lt;</span><span style="color: #800000">form</span> <span style="color: #ff0000">enctype</span><span style="color: #0000ff">="multipart/form-data"</span> <span style="color: #ff0000">method</span><span style="color: #0000ff">="post"</span> <span style="color: #ff0000">action</span><span style="color: #0000ff">="@Url.Action("</span><span style="color: #ff0000">SendImage</span><span style="color: #0000ff">")"</span><span style="color: #0000ff">&gt;</span><br />    <span style="color: #0000ff">&lt;</span><span style="color: #800000">label</span> <span style="color: #ff0000">for</span><span style="color: #0000ff">="img"</span><span style="color: #0000ff">&gt;</span>Seleccionar imagen:<span style="color: #0000ff">&lt;/</span><span style="color: #800000">label</span><span style="color: #0000ff">&gt;</span><br />    <span style="color: #0000ff">&lt;</span><span style="color: #800000">input</span> <span style="color: #ff0000">type</span><span style="color: #0000ff">="file"</span> <span style="color: #ff0000">name</span><span style="color: #0000ff">="img"</span> <span style="color: #0000ff">/&gt;</span> <span style="color: #0000ff">&lt;</span><span style="color: #800000">br</span> <span style="color: #0000ff">/&gt;</span><br />    <span style="color: #0000ff">&lt;</span><span style="color: #800000">input</span> <span style="color: #ff0000">type</span><span style="color: #0000ff">="submit"</span> <span style="color: #0000ff">/&gt;</span><br /><br />    @if (ViewBag.ImageData != null)<br />    {   <br />        <span style="color: #0000ff">&lt;</span><span style="color: #800000">img</span> <span style="color: #ff0000">src</span><span style="color: #0000ff">="data:@ViewBag.ImageContentType;base64,@ViewBag.ImageData"</span> <span style="color: #0000ff">/&gt;</span><br />        <span style="color: #0000ff">&lt;</span><span style="color: #800000">input</span> <span style="color: #ff0000">type</span><span style="color: #0000ff">="hidden"</span> <span style="color: #ff0000">value</span><span style="color: #0000ff">="@ViewBag.ImageData"</span> <span style="color: #ff0000">name</span><span style="color: #0000ff">="base64"</span> <span style="color: #0000ff">/&gt;</span><br />        <span style="color: #0000ff">&lt;</span><span style="color: #800000">input</span> <span style="color: #ff0000">type</span><span style="color: #0000ff">="hidden"</span> <span style="color: #ff0000">value</span><span style="color: #0000ff">="@ViewBag.ImageContentType"</span> <span style="color: #ff0000">name</span><span style="color: #0000ff">="contenttype"</span> <span style="color: #0000ff">/&gt;</span><br />    }<br /><span style="color: #0000ff">&lt;/</span><span style="color: #800000">form</span><span style="color: #0000ff">&gt;</span></pre>
          
          <p>
            </div> 
            
            <p>
              Tenemos el formulario con el input type=”submit”, y luego si en el ViewBag vienen los datos de la imagen que se debe previsualizar, genera 3 campos más:
            </p>
            
            <ol>
              <li>
                Una imagen, cuyo src es <strong>una data url </strong>(formato <em>data:content-type;base64,[datos en base64]</em>)
              </li>
              <li>
                Dos campos hidden (para guardar el content-type y los datos para mandarlos de vuelta al servidor).
              </li>
            </ol>
            
            <p>
              Y listos! Con eso tenemos la previsualización de las imágenes <strong>sin necesidad de generar fichero temporal alguno</strong>.
            </p>
            
            <p>
              Finalmente, tres aclaraciones:
            </p>
            
            <ol>
              <li>
                Fijaos que la codificación en base64 <strong>se incrusta en la página</strong> (en este caso se incrusta dos veces, aunque con un poco de javascript podría incrustarse solo una), por lo que esto puede generar páginas <strong>muy grandes</strong> si la imagen ocupa mucho.
              </li>
              <li>
                Si no voy errado, los navegadores sólo están obligados a soportar URLs de hasta 1024 bytes. Todo lo que exceda de aquí… depende del navegador. Firefox p.ej. acepta URLs muy largas, pero recordad que Base64 genera fácilmente URLs no muy largas, sinó descomunalmente largas (si la imágen ocupa 100Ks, la URL en Base64 ocupará <strong>más</strong> de 100Ks). Así que para imágenes pequeñas igual tiene sentido, pero para imágenes largas, honestamente no creo que sea una buena solución.
              </li>
              <li>
                La mayoría de navegadores modernos soportan data urls (IE lo empezó a hacer con su versión 8).
              </li>
            </ol>
            
            <p>
              En fin… bueno, como curiosidad no está mal, eh? 😉
            </p>
            
            <p>
              Gracias a <a href="http://twitter.com/pablonete" target="_blank" rel="noopener noreferrer">Pablo Nuñez</a> por la divertida e interesante conversación en twitter!
            </p>
            
            <p>
              Saludos!
            </p>
            
            <p>
              PD: Enlace al post anterior de esa serie: <a href="http://geeks.ms/blogs/etomas/archive/2011/03/15/asp-net-mvc-previsualizar-im-225-genes-subidas-1.aspx" target="_blank" rel="noopener noreferrer">http://geeks.ms/blogs/etomas/archive/2011/03/15/asp-net-mvc-previsualizar-im-225-genes-subidas-1.aspx</a>
            </p>