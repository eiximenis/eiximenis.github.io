---
title: 'ASP.NET MVC: Mostrar datos en HTML o PDF (pero en el fondo vamos a hablar de la tabla de rutas).'
author: eiximenis

date: 2010-09-09T13:13:19+00:00
geeks_url: /?p=1533
geeks_visits:
  - 5587
geeks_ms_views:
  - 3400
categories:
  - Uncategorized

---
El otro día, [uno de los grandes de Webforms][1] (al que, aún que a veces despotrique un poco, MVC le está empezando a gustar :P), publico un [excelente artículo sobre como generar PDFs usando MVC][2]. En el artículo Marc mostraba como mostrar datos de un PDF físico en disco o bien usando una vista parcial para generar PDFs al vuelo.

Su artículo me sirve de excusa perfecta para escribir un artículo sobre como podríamos aplicar su solución a un caso más general: vamos a ver como podemos mostrar **una misma vista** ya sea en formato HTML o bien en PDF. De nuevo, **antes que nada leeros el artículo de Marc**, ya que este está basado en aquel.

El objetivo que pretendemos es que dada una url, p.ej. _http://host/geeks/list_ nos muestre la información en formato HTML o bien en formato PDF. La primera cosa a resolver es como indicar si queremos que el formato de salida sea pdf o html… Las dos maneras que quizá se os ocurran primero son:

  1. Añadiendo un parámetro de ruta, de forma que tendremos una URL tipo /geeks/list/pdf o /geeks/list/html. No me convence porque no queda claro que este parámetro no es de “negocio”. Me explico, una URL que para ver mis datos podría ser /geeks/view/pdf/edu. El parámetro ‘edu’ és claramente de negocio, pero el parámetro pdf, no… 
  2. Añadiendo un parámetro en querystring, de forma que tendremos una URL tipo /geeks/list?format=pdf. No me convence porque aunque MVC se defiende bien con parámetros en querystring, realmente son totalmente innecesarios (y rompen la “amigabilidad” de las URLs que promulga MVC).

Un dia navegando por la MSDN vi otra opción que me pareció interesante. P.ej. si vais a [http://msdn.microsoft.com/library/system.string(v=VS.90).aspx][3] obteneis la información de string para el framework 3.5, mientras que si vais a [http://msdn.microsoft.com/library/system.string(v=VS.80).aspx][4] la que obteneis es la información para el framework 2.0. El parámetro (v=xxx) indica la versión para el que mostrar información.

Podemos conseguir fácilmente algo parecido en MVC? Pues… por supuesto! 😉

**1. La tabla de rutas**

La tabla de rutas es uno de los aspectos más desconocidos de ASP.NET MVC. Mucha gente asume que la convención de URLs /controlador/accion/id es una _obligación_, pero ni mucho menos: las URLs en ASP.NET MVC pueden tener cualquier forma y se definen usando la tabla de rutas. La forma /controlador/accion/id es sólamente la configuración estándard de la tabla de rutas (y digo estándard y no _por defecto_ porque la tabla de rutas inicialmente está vacía, pero el wizard de VS nos genera el código para rellenarla usando dicha convención).

El código para configurar la tabla de rutas está en Global.asax.cs y el código que genera VS es tal y como sigue:

<div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
  <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px" id="codeSnippet">routes.IgnoreRoute(<span style="color: #006080">"{resource}.axd/{*pathInfo}"</span>);<br />routes.MapRoute(<br />    <span style="color: #006080">"Default"</span>, <span style="color: #008000">// Route name</span><br />    <span style="color: #006080">"{controller}/{action}/{id}"</span>, <span style="color: #008000">// URL with parameters</span><br />    <span style="color: #0000ff">new</span> { controller = <span style="color: #006080">"Home"</span>, action = <span style="color: #006080">"Index"</span>, id = UrlParameter.Optional }<br />);</pre>
  
  <p>
    </div> 
    
    <p>
      La primera sentencia es para que MVC ignore las rutas de tipo algo.axd/algomás (estas URLs pertenecen al sistema de trace de ASP.NET) y la segunda sentencia es la interesante. En ella se mapean las URLs con el formato /controlador/accion/id, y no sólo eso, sinó que se asignan valores por defecto (contrrolador vale Home, accion vale index y id es opcional y no tiene valor). Es por eso que la URL / en MVC equivale a /Home que equivale a /Home/Index.
    </p>
    
    <p>
      Lo que ponemos entre llaves ({}) en la definición de la URLs son las <em>variables</em> de la tabla de rutas. El resultado de procesar una URL mediante la tabla de rutas es un conjunto de valores, que conocemos con el nombre de <em>Route Values</em>. Hay dos <em>Route Values</em> que el sistema MVC debe ser capaz de extraer de cada URL: controller y action. El resto de route values son pasadas como parámetros al controlador. Cuando usamos <em>MapRoute</em> para añadir una ruta a la tabla de rutas, le pasamos la mayoría de veces tres parámetros:
    </p>
    
    <ol>
      <li>
        Un nombre de ruta
      </li>
      <li>
        El formato de las URLs que acepta dicha ruta. Lo que esté entre llaves es variable y se mapeará a una <em>route value</em>
      </li>
      <li>
        El valor de las route values por defecto cuando no se puedan mapear a partir de la URL.
      </li>
    </ol>
    
    <p>
      Las URLs que queremos obtener son URLs del siguiente tipo: /geeks/list(pdf) o /geeks/list(html). Obviamente quiero poder añadir parámetros después, es decir /geeks/list(pdf)/edu debe funcionar. Y también quiero que las URL clásicas (/geeks/list o bien /geeks/list/edu) funcionen bien (generando la salida en HTML).
    </p>
    
    <p>
      La tabla de rutas que hace posible esas URLs es tal y como sigue:
    </p>
    
    <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
      <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px" id="codeSnippet">routes.IgnoreRoute(<span style="color: #006080">"{resource}.axd/{*pathInfo}"</span>);<br />routes.MapRoute(<br />    <span style="color: #006080">"WithDevice"</span>, <span style="color: #008000">// Route name</span><br />    <span style="color: #006080">"{controller}/{action}({device})/{id}"</span>, <span style="color: #008000">// URL with parameters</span><br />    <span style="color: #0000ff">new</span> { controller = <span style="color: #006080">"Home"</span>, action = <span style="color: #006080">"Index"</span>, device = <span style="color: #006080">"html"</span>, id = UrlParameter.Optional }<br />);<br /><br />routes.MapRoute(<br />    <span style="color: #006080">"Default"</span>, <span style="color: #008000">// Route name</span><br />    <span style="color: #006080">"{controller}/{action}/{id}"</span>, <span style="color: #008000">// URL with parameters</span><br />    <span style="color: #0000ff">new</span> { controller = <span style="color: #006080">"Home"</span>, action = <span style="color: #006080">"Index"</span>, device = <span style="color: #006080">"html"</span>, id = UrlParameter.Optional }<br />);</pre>
      
      <p>
        </div> 
        
        <p>
          Fijaos en los dos <em>MapRoute</em>:
        </p>
        
        <ol>
          <li>
            El primero mapea URLs del tipo controlador/accion(formato)/param. Los paréntesis <strong>deben</strong> aparecer. Es decir una url del tipo /geeks/list(pdf)/edu será procesada por esa ruta y asignará los <em>route values</em>:
          </li>
          <ol>
            <li>
              controller = geeks
            </li>
            <li>
              action=list
            </li>
            <li>
              device=pdf
            </li>
            <li>
              id=edu
            </li>
          </ol>
          
          <li>
            Si una URL no tiene este formato (no tiene los paréntesis) no se procesará por la primera ruta y entrará por la segunda. Esta segunda ruta es la clásica de ASP.NET MVC. La única diferencia es que asigna el valor del route value a html. Así pues una url del tipo /geeks/list/edu será procesada por esta segunda ruta (y obtendremos los mismos route values que el caso anterior).
          </li>
        </ol>
        
        <p>
          Ahora que ya podemos procesar las URLs que nos interesan, ya podemos generar la salida en formato PDF o HTML, según el valor del route value <em>device</em>.
        </p>
        
        <p>
          <strong>2.El controlador</strong>
        </p>
        
        <p>
          El controlador es muy simple. En el ejemplo sólo tengo una acción (List) que devuelve un listado de geeks. No me interesa el parámetro <em>id</em>, pero si el parámetro <em>device </em>(para saber si debo generar la salida en&#160; pdf o html) así pues, declaro el controlador:
        </p>
        
        <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
          <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px" id="codeSnippet"><span style="color: #0000ff">public</span> ActionResult List(<span style="color: #0000ff">string</span> device)<br />{<br />    IEnumerable&lt;GeeksViewModel&gt; geeks = <span style="color: #0000ff">new</span> GeeksModel().GetAllGeeks();<br />    <span style="color: #0000ff">return</span> <span style="color: #006080">"pdf"</span>.Equals(device, StringComparison.InvariantCultureIgnoreCase) ? <br />        <span style="color: #0000ff">this</span>.Pdf(geeks) : View(geeks);<br />}</pre>
          
          <p>
            </div> 
            
            <p>
              Fijaos como la acción recibe el parámetro <em>device</em>, cuyo valor será el valor del route value <em>device</em>… Os he dicho que me encanta MVC? 🙂
            </p>
            
            <p>
              Lo que hace el controlador es poca cosa: obtiene una lista de objetos GeeksViewModel y luego o bien llama a View() para pasar dichos datos a la vista, o bien llama a Pdf que es un método extensor que he hecho, que devuelve los datos en pdf.
            </p>
            
            <p>
              El código del método extensor (totalmente basado en lo que publicó Marc) es el siguiente:
            </p>
            
            <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
              <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px" id="codeSnippet"><span style="color: #0000ff">public</span> <span style="color: #0000ff">static</span> <span style="color: #0000ff">class</span> ControllerExtensions<br />{<br />    <span style="color: #0000ff">public</span> <span style="color: #0000ff">static</span> ActionResult Pdf(<span style="color: #0000ff">this</span> ControllerBase @<span style="color: #0000ff">this</span>)<br />    {<br />        <span style="color: #0000ff">return</span> Pdf(@<span style="color: #0000ff">this</span>, <span style="color: #0000ff">null</span>);<br />    }<br /><br />    <span style="color: #0000ff">public</span> <span style="color: #0000ff">static</span> ActionResult Pdf(<span style="color: #0000ff">this</span> ControllerBase @<span style="color: #0000ff">this</span>, <span style="color: #0000ff">object</span> model)<br />    {<br />        <span style="color: #0000ff">byte</span>[] buf = <span style="color: #0000ff">null</span>;<br />        MemoryStream pdfTemp = <span style="color: #0000ff">new</span> MemoryStream();<br />        ViewEngineResult ver = ViewEngines.Engines.FindView(@<span style="color: #0000ff">this</span>.ControllerContext,  <br />            @<span style="color: #0000ff">this</span>.ControllerContext.RouteData.Values[<span style="color: #006080">"action"</span>].ToString(), <span style="color: #0000ff">null</span>);<br />        <span style="color: #0000ff">if</span> (ver.View != <span style="color: #0000ff">null</span>)<br />        {<br />            <span style="color: #0000ff">if</span> (model != <span style="color: #0000ff">null</span>)<br />            {<br />                @<span style="color: #0000ff">this</span>.ViewData.Model = model;<br />            }<br />            <span style="color: #0000ff">string</span> htmlTextView = GetViewToString(@<span style="color: #0000ff">this</span>.ControllerContext, ver);<br />            iTextSharp.text.Document doc = <span style="color: #0000ff">new</span> iTextSharp.text.Document();<br />            iTextSharp.text.pdf.PdfWriter writer = iTextSharp.text.pdf.PdfWriter.GetInstance(doc, pdfTemp);<br />            writer.CloseStream = <span style="color: #0000ff">false</span>;<br />            doc.Open();<br />            AddHTMLText(doc, htmlTextView);<br />            doc.Close();<br />            buf = <span style="color: #0000ff">new</span> <span style="color: #0000ff">byte</span>[pdfTemp.Position];<br />            pdfTemp.Position = 0;<br />            pdfTemp.Read(buf, 0, buf.Length);<br />        }<br />        <span style="color: #0000ff">return</span> <span style="color: #0000ff">new</span> System.Web.Mvc.FileContentResult(buf, <span style="color: #006080">"application/pdf"</span>);<br />    }<br /><br />    <span style="color: #0000ff">private</span> <span style="color: #0000ff">static</span> <span style="color: #0000ff">void</span> AddHTMLText(iTextSharp.text.Document doc, <span style="color: #0000ff">string</span> html)<br />    {<br /><br />        List&lt;iTextSharp.text.IElement&gt; htmlarraylist = HTMLWorker.ParseToList(<span style="color: #0000ff">new</span> StringReader(html), <span style="color: #0000ff">null</span>);<br />        <span style="color: #0000ff">foreach</span> (var item <span style="color: #0000ff">in</span> htmlarraylist)<br />        {<br />            doc.Add(item);<br />        }<br /><br />    }<br /><br />    <span style="color: #0000ff">private</span> <span style="color: #0000ff">static</span> <span style="color: #0000ff">string</span> GetViewToString(ControllerContext context, ViewEngineResult result)<br />    {<br />        <span style="color: #0000ff">string</span> viewResult = <span style="color: #006080">""</span>;<br />        TempDataDictionary tempData = <span style="color: #0000ff">new</span> TempDataDictionary();<br />        StringBuilder sb = <span style="color: #0000ff">new</span> StringBuilder();<br />        <span style="color: #0000ff">using</span> (StringWriter sw = <span style="color: #0000ff">new</span> StringWriter(sb))<br />        {<br />            <span style="color: #0000ff">using</span> (HtmlTextWriter output = <span style="color: #0000ff">new</span> HtmlTextWriter(sw))<br />            {<br />                ViewContext viewContext = <span style="color: #0000ff">new</span> ViewContext(context, result.View, context.Controller.ViewData, context.Controller.TempData, output);<br />                result.View.Render(viewContext, output);<br />            }<br />            viewResult = sb.ToString();<br />        }<br />        <span style="color: #0000ff">return</span> viewResult;<br />    }<br />}</pre>
              
              <p>
                </div> 
                
                <p>
                  Las únicas diferencias respecto el código que puso Marc son:
                </p>
                
                <ol>
                  <li>
                    El uso de vistas completas (en lugar de una vista parcial)
                  </li>
                  <li>
                    El soporte para pasar a la vista un modelo
                  </li>
                  <li>
                    El uso del ViewData y del TempData que tenga el controlador (en lugar de crear un ViewData y un TempData vacío)
                  </li>
                  <li>
                    Que la vista se selecciona a través del nombre de acción en lugar de ser una vista concreta.
                  </li>
                </ol>
                
                <p>
                  El resto es tal y como estaba el código de Marc (porque ya os digo que yo iTextSharp no es que lo domine mucho :P).
                </p>
                
                <p>
                  Y el resultado? Pues si llamamos a /Geeks/List o /Geeks/List(html) tenemos:
                </p>
                
                <p>
                  <a href="http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_5392922A.png"><img style="border-bottom: 0px; border-left: 0px; display: inline; border-top: 0px; border-right: 0px" title="image" border="0" alt="image" src="http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_4356A464.png" width="244" height="136" /></a>
                </p>
                
                <p>
                  Mientras que si llamamos a /Geeks/List(pdf) el resultado es:
                </p>
                
                <p>
                  <a href="http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_7B951B7C.png"><img style="border-bottom: 0px; border-left: 0px; display: inline; border-top: 0px; border-right: 0px" title="image" border="0" alt="image" src="http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_6C7E3CA2.png" width="244" height="141" /></a>
                </p>
                
                <p>
                  <strong>3. Conclusiones</strong>
                </p>
                
                <p>
                  Lo interesante del post no es ver como generar PDFs (de eso ya se encarga el post de Marc), lo interesante es ver como gracias a la tabla de rutas podemos crearnos nuestras propias URLs de forma muy sencilla!
                </p>
                
                <p>
                  Un saludo a todos y gracias a Marc por el post que me ha dado la excusa para escribir este! 😉
                </p>

 [1]: http://geeks.ms/members/mrubino/default.aspx
 [2]: http://geeks.ms/blogs/mrubino/archive/2010/09/08/tip-trick-asp-net-mvc-amp-pdf.aspx
 [3]: http://msdn.microsoft.com/library/system.string(v=VS.90).aspx "http://msdn.microsoft.com/library/system.string(v=VS.90).aspx"
 [4]: http://msdn.microsoft.com/library/system.string(v=VS.80).aspx "http://msdn.microsoft.com/library/system.string(v=VS.80).aspx"