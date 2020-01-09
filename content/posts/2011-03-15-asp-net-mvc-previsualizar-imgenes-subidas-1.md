---
title: 'ASP.NET MVC: Previsualizar imágenes subidas (1)'
author: eiximenis

date: 2011-03-15T10:00:18+00:00
geeks_url: /?p=1560
geeks_visits:
  - 9126
geeks_ms_views:
  - 4329
categories:
  - Uncategorized

---
Buenas! Una pregunta que últimamente parece que se pregunta varias veces en los foros de ASP.NET MVC es como previsualizar una imagen que se quiere subir al servidor.

Antes que nada aclarar que, técnicamente, la pregunta está mal hecha: **no es posible previsualizar la imagen <u>antes</u> de que sea subida**. Antiguamente en algunos navegadores, y con un poco de javascript, eso era posible, pero ahora por suerte eso ya no funciona 🙂

Básicamente _previsualizar_ una imagen consiste en:

  1. Recibir los datos de la imagen
  2. Guardarla en algún sitio “temporal”
  3. Mandarla de _vuelta_ al navegador
  4. Borrar la imagen del sitio “temporal” si el usuario no la acepta

En este primer post vamos a ver como hacerlo sin usar Ajax. Luego habrá un segundo post y veremos como hacerlo usando Ajax (no es tan sencillo porque XMLHttpRequest no soporta el formato de datos multipart/form-data que es el que se usa cuando se mandan ficheros).

Bien, vamos a verlo rápidamente, ya veréis cuan sencillo es 🙂

Lo primero es tener la vista que tenga el formulario para enviar los datos. La siguiente servirá (si queréis más detalles de como hacer upload de ficheros en MVC mirad <a href="http://geeks.ms/blogs/etomas/archive/2010/09/08/subir-ficheros-al-servidor-en-asp-net-mvc.aspx" target="_blank" rel="noopener noreferrer">mi post al respecto</a>):

<div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
  <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; background-color: #f4f4f4; margin: 0em; border-left-style: none; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; border-right-style: none; font-size: 8pt; overflow: visible; padding-top: 0px" id="codeSnippet"><span style="color: #0000ff">&lt;</span><span style="color: #800000">h2</span><span style="color: #0000ff">&gt;</span>Index<span style="color: #0000ff">&lt;/</span><span style="color: #800000">h2</span><span style="color: #0000ff">&gt;</span><br /><br /><span style="color: #0000ff">&lt;</span><span style="color: #800000">form</span> <span style="color: #ff0000">enctype</span><span style="color: #0000ff">="multipart/form-data"</span> <span style="color: #ff0000">method</span><span style="color: #0000ff">="post"</span> <span style="color: #ff0000">action</span><span style="color: #0000ff">="@Url.Action("</span><span style="color: #ff0000">SendImage</span><span style="color: #0000ff">")"</span><span style="color: #0000ff">&gt;</span><br />    <span style="color: #0000ff">&lt;</span><span style="color: #800000">label</span> <span style="color: #ff0000">for</span><span style="color: #0000ff">="img"</span><span style="color: #0000ff">&gt;</span>Seleccionar imagen:<span style="color: #0000ff">&lt;/</span><span style="color: #800000">label</span><span style="color: #0000ff">&gt;</span><br />    <span style="color: #0000ff">&lt;</span><span style="color: #800000">input</span> <span style="color: #ff0000">type</span><span style="color: #0000ff">="file"</span> <span style="color: #ff0000">name</span><span style="color: #0000ff">="img"</span> <span style="color: #0000ff">/&gt;</span> <span style="color: #0000ff">&lt;</span><span style="color: #800000">br</span> <span style="color: #0000ff">/&gt;</span><br />    <span style="color: #0000ff">&lt;</span><span style="color: #800000">input</span> <span style="color: #ff0000">type</span><span style="color: #0000ff">="submit"</span> <span style="color: #0000ff">/&gt;</span><br /><span style="color: #0000ff">&lt;/</span><span style="color: #800000">form</span><span style="color: #0000ff">&gt;</span></pre>
  
  <p>
    </div> 
    
    <p>
      Trivial: un form con multipart/form-data que enviará sus datos a la acción SendImage. La acción podría ser algo como:
    </p>
    
    <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
      <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; background-color: #f4f4f4; margin: 0em; border-left-style: none; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; border-right-style: none; font-size: 8pt; overflow: visible; padding-top: 0px" id="codeSnippet">[HttpPost]<br /><span style="color: #0000ff">public</span> ActionResult SendImage(HttpPostedFileBase img)<br />{<br />    var data = <span style="color: #0000ff">new</span> <span style="color: #0000ff">byte</span>[img.ContentLength];<br />    img.InputStream.Read(data, 0, img.ContentLength);<br />    var path = ControllerContext.HttpContext.Server.MapPath(<span style="color: #006080">"/"</span>);<br />    var filename = Path.Combine(path, Path.GetFileName(img.FileName));<br />    System.IO.File.WriteAllBytes(Path.Combine(path, filename), data);<br />    ViewBag.ImageUploaded = filename;<br />    <span style="color: #0000ff">return</span> View(<span style="color: #006080">"Index"</span>);<br />}</pre>
      
      <p>
        </div> 
        
        <p>
          Vale, fijaos que recibimos el HttpPostedFileBase (llamado img como el atributo <em>name</em> del <em>input type=”file”</em>). Lo que hacemos en la acción es muy simple:
        </p>
        
        <ol>
          <li>
            Guardamos la imagen en disco (en este caso por pura pereza uso el directorio raíz de la aplicación web. Por supuesto lo suyo es usar un directorio configurado en web.config y con los permisos NTFS correspondientes).
          </li>
          <li>
            Coloco en el <em>ViewBag</em> el nombre de la imagen que se ha guardado (luego vemos porque).
          </li>
          <li>
            Devuelvo la vista “Index” (la misma de la cual venimos)
          </li>
        </ol>
        
        <p>
          Ok, tal y como lo tenemos ahora, si lo probáis veréis que podéis hacer un upload de la imagen, y la imagen se graba en el disco (en el directorio raíz de la aplicación web), pero no se previsualiza nada. Vamos a modificar la vista Index para que haya esa <em>previsualización</em>.
        </p>
        
        <p>
          Para previsualizar la imagen basta con añadir el siguiente código, despues del </form> en la vista:
        </p>
        
        <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
          <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; background-color: #f4f4f4; margin: 0em; border-left-style: none; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; border-right-style: none; font-size: 8pt; overflow: visible; padding-top: 0px" id="codeSnippet">@if (!string.IsNullOrEmpty(ViewBag.ImageUploaded))<br />{<br />    <span style="color: #0000ff">&lt;</span><span style="color: #800000">img</span> <span style="color: #ff0000">src</span><span style="color: #0000ff">="@Url.Action("</span><span style="color: #ff0000">Preview</span><span style="color: #0000ff">", new {file=ViewBag.ImageUploaded})"</span> <span style="color: #0000ff">/&gt;</span><br />}</pre>
          
          <p>
            </div> 
            
            <p>
              Simple, no? Si en el ViewBag existe la entrada ImageUploaded generamos un tag <img> cuya dirección es la acción “Preview” y le pasamos el parámetro file con el nombre de la imagen. Y como es la acción Preview? Pues super sencilla:
            </p>
            
            <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
              <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; background-color: #f4f4f4; margin: 0em; border-left-style: none; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; border-right-style: none; font-size: 8pt; overflow: visible; padding-top: 0px" id="codeSnippet"><span style="color: #0000ff">public</span> ActionResult Preview(<span style="color: #0000ff">string</span> file)<br />{<br />    var path = ControllerContext.HttpContext.Server.MapPath(<span style="color: #006080">"/"</span>);<br />    <span style="color: #0000ff">if</span> (System.IO.File.Exists(Path.Combine(path, file)))<br />    {<br />        <span style="color: #0000ff">return</span> File(Path.Combine(path, file), <span style="color: #006080">"image/jpeg"</span>);<br />    }<br />    <span style="color: #0000ff">return</span> <span style="color: #0000ff">new</span> HttpNotFoundResult();<br />}</pre>
              
              <p>
                </div> 
                
                <p>
                  Simplemente leemos la imagen guardada y la devolvemos usando un FilePathResult. Simple, eh? 😉
                </p>
                
                <p>
                  Con eso, si subís una imagen, cuando le deis a submit, aparecerá de nuevo la vista, pero ahora previsualizando la imagen.
                </p>
                
                <p>
                  Ahora sólo nos queda el punto final: Que el usuario pueda <em>aceptar</em> esa imagen como correcta. Para ello lo más simple es hacer lo siguiente:
                </p>
                
                <ol>
                  <li>
                    Si el usuario está previsualizando una imagen y NO ha seleccionado otra, cuando hace el submit se entiende que acepta dicha imagen.
                  </li>
                  <li>
                    Si el usuario NO está previsualizando una imagen, o bien está previsualizando una pero selecciona otra, al hacer el submit se entiende que <em>descarta</em> la imagen anterior y quiere previsualizar la nueva.
                  </li>
                </ol>
                
                <p>
                  Para ello, tenemos que hacer que la vista le indique al controlador <em>si se está previsualizando una imagen, y cual és</em>. Por suerte eso es muy sencillo. Basta con modificar el <form> de la vista para que su atributo action quede como:
                </p>
                
                <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
                  <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; background-color: #f4f4f4; margin: 0em; border-left-style: none; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; border-right-style: none; font-size: 8pt; overflow: visible; padding-top: 0px" id="codeSnippet">&lt;form enctype=<span style="color: #006080">"multipart/form-data"</span> method=<span style="color: #006080">"post"</span> <br />action=<span style="color: #006080">"@Url.Action("</span>SendImage<span style="color: #006080">", new {previewed=ViewBag.ImageUploaded})"</span>&gt;</pre>
                  
                  <p>
                    </div> 
                    
                    <p>
                      Fijaos que añadimos un parámetro <em>previewed</em> que la vista mandará a la acción y que será el nombre de la imagen que se está previsualizando.
                    </p>
                    
                    <p>
                      Vamos a modificar la acción para que reciba ese parámetro y actúe en consecuencia:
                    </p>
                    
                    <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
                      <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; background-color: #f4f4f4; margin: 0em; border-left-style: none; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; border-right-style: none; font-size: 8pt; overflow: visible; padding-top: 0px" id="codeSnippet">[HttpPost]<br /><span style="color: #0000ff">public</span> ActionResult SendImage(HttpPostedFileBase img, <span style="color: #0000ff">string</span> previewed)<br />{<br />    <span style="color: #0000ff">if</span> (!<span style="color: #0000ff">string</span>.IsNullOrEmpty(previewed) && img == <span style="color: #0000ff">null</span>)<br />    {<br />        ViewBag.ImageName = previewed;<br />        <span style="color: #0000ff">return</span> View(<span style="color: #006080">"Step2"</span>);<br />    }<br /><br />    <span style="color: #008000">// Código tal cual estaba</span><br />}</pre>
                      
                      <p>
                        </div> 
                        
                        <p>
                          Añadimos ese if al inicio: Si el usuario está previsualizando una imagen y no ha seleccionado otra, entendemos que acepta esta imagen. Entonces guardamos el nombre en el ViewBag y lo mandamos a la siguiente vista (Step2).
                        </p>
                        
                        <p>
                          La vista es muy sencilla:
                        </p>
                        
                        <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
                          <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; background-color: #f4f4f4; margin: 0em; border-left-style: none; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; border-right-style: none; font-size: 8pt; overflow: visible; padding-top: 0px" id="codeSnippet">&lt;h2&gt;Step2&lt;/h2&gt;<br /><br />Siguiente paso. El usuario ha aceptado la imagen: &lt;br /&gt;<br /><br />&lt;img src=<span style="color: #006080">"@Url.Action("</span>Preview<span style="color: #006080">", new {file=ViewBag.ImageName})"</span> /&gt;</pre>
                          
                          <p>
                            </div> 
                            
                            <p>
                              Simplemente mostramos la imagen. Fijaos que de nuevo usamos la acción Preview, puesto que la imagen ya la tenemos guardada en el servidor.
                            </p>
                            
                            <p>
                              <strong>Notas finales</strong>
                            </p>
                            
                            <p>
                              Faltarían, al menos, dos cosas para que dicho proyecto funcionase “de forma aceptable”:
                            </p>
                            
                            <ol>
                              <li>
                                Borrar las imagenes descartadas por el usuario (en la acción SendImage si se recibe una imagen <em>nueva </em>y se estaba previsualizando una, borrar esta ya que el usuario la ha descartado).
                              </li>
                              <li>
                                La acción&#160; Preview siempre devuelve content-type a image/jpeg, eso debería hacerse según la extensión de la imagen, o guardarlo en algún sitio.
                              </li>
                            </ol>
                            
                            <p>
                              Ambas cosas son triviales y no las añado porque lo único que consiguiría es liar el código un poco más 😉
                            </p>
                            
                            <p>
                              En el próximo post… lo mismo pero usando Ajax.
                            </p>
                            
                            <p>
                              Saludos!
                            </p>