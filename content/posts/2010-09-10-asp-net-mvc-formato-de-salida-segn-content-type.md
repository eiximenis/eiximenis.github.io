---
title: ASP.NET MVC – Formato de salida según Content-Type
description: ASP.NET MVC – Formato de salida según Content-Type
author: eiximenis

date: 2010-09-10T13:03:36+00:00
geeks_url: /?p=1534
geeks_visits:
  - 6089
geeks_ms_views:
  - 1707
categories:
  - Uncategorized

---
El otro día escribí un post donde vimos como [mostrar una vista en PDF o HTML en función de una URL del tipo /controlador/accion(formato)/parámetros][1]. El post estaba centrado básicamente en la tabla de rutas y cómo la URL clásica de ASP.NET MVC /Controlador/Accion/Parámetros no es una obligación sinó básicamente una convención.

[Hadi Hariri][2] realizó un comentario, muy interesante a mi jucio. Venía a decir que antes que añadir en la ruta el parámetro formato es mejor usar el campo _Accept_ de la request. Copio literalmente: “_La tercera opcion, que lo hace más transparente al usuario y además está en acorde a ReST, es la de usar las el ContentType en la petición, que es lo que yo normalmente hago._”

Si quieres exponer una API lo más ReST posible en ASP.NET MVC y que tenga salidas en distintos formatos, sin duda deberías tener en cuenta la sugerencia de Hadi.

**1. La cabecera de la petición http**

Cuando un cliente envía una [petición http][3] a un servidor, que contiene [una cabecera con varios parámetros][4]. Dicha cabecera tiene varios campos que permiten especificar determinadas opciones que el cliente desea. Uno de esos campos es el campo _Accept_ que permite indicar que formatos de respuesta acepta el cliente y en que orden.

P.ej. si hago una petición con Firefox a _http://www.google.es_, el contenido del campo _Accept_ de la cabecera que firefox envia es (lo acabo de mirar con Firebug):

`text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8`

Que podríamos interpretar (más o menos) como: _Mis formatos preferidos son text/html y application/xhtml+xml, si no puedes en ninguno de esos dos, envíamelo en application/xml y si no puedes, pues me tragaré lo que me mandes._

El valor exacto de dicha cabecera depende del browser… P.ej. IE8 para la misma peticion envia el siguiente valor en Accept (lo acabo de mirar con Fiddler):

<font size="2" face="Courier New">image/jpeg, application/x-ms-application, image/gif, application/xaml+xml, image/pjpeg, application/x-ms-xbap, application/x-shockwave-flash, application/vnd.ms-excel, application/vnd.ms-powerpoint, application/msword, */*</font>

Por lo tanto vemos como en el campo _Accept_ el cliente nos dice que formatos de respuesta entiende (y en que orden los prefiere).

**2. Acceso a la cabecera desde ASP.NET MVC**

Imaginad que tenemos un controlador que puede devolver datos en dos formatos: XML y JSON. Y queremos usar el campo _Accept_ de la cabecera http que envíe el cliente para devolver los datos en uno u otro formato.

Acceder a la cabecera http desde un controlador es extremadamente sencillo, usando **Request.AcceptTypes**, que es un array con todos los campos de la cabecera a_ccept_.

**3. Devolver datos en formato XML**

ASP.NET MVC no trae ningún mecanismo incluído para devolver datos en formato xml, lo que me va de coña para enseñaros como nos podemos crear un _ActionResult_ propio:

<div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
  <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px" id="codeSnippet"><span style="color: #0000ff">public</span> <span style="color: #0000ff">class</span> XmlActionResult : ActionResult<br />{<br />    <span style="color: #0000ff">private</span> <span style="color: #0000ff">readonly</span> <span style="color: #0000ff">object</span> _data;<br />    <span style="color: #0000ff">public</span> XmlActionResult(<span style="color: #0000ff">object</span> data)<br />    {<br />        _data = data;<br />    }<br />    <span style="color: #0000ff">public</span> <span style="color: #0000ff">override</span> <span style="color: #0000ff">void</span> ExecuteResult(ControllerContext context)<br />    {<br />        XmlSerializer ser = <span style="color: #0000ff">new</span> XmlSerializer(_data.GetType());<br />        context.HttpContext.Response.ContentType = <span style="color: #006080">"text/xml"</span>;<br />        ser.Serialize(context.HttpContext.Response.OutputStream, _data);<br />    }<br />}</pre>
  
  <p>
    </div> 
    
    <p>
      Crear un ActionResult propio es trivial: deriváis de ActionResult y implementáis el método abstracto <em>ExecuteResult</em> y en él hacéis lo que sea necesario (usualmente interaccionar con la Response). En este caso simplemente serializo el objeto que se le pasa con el serializador estándard de .NET. Ah si! Y pongo el content-type a <em>text/xml</em> que es el content-type usado para documentos en XML.
    </p>
    
    <p>
      Yo suelo acompañar los ActionResults propios con un método extensor para los controladores, para llamarlos de forma similar a los ActionResults que vienen en el framework. Mi método extensor (trivial) es:
    </p>
    
    <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
      <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px" id="codeSnippet"><span style="color: #0000ff">public</span> <span style="color: #0000ff">static</span> <span style="color: #0000ff">class</span> ControllerExtensions<br />{<br />    <span style="color: #0000ff">public</span> <span style="color: #0000ff">static</span> XmlActionResult Xml(<span style="color: #0000ff">this</span> ControllerBase @<span style="color: #0000ff">this</span>, <span style="color: #0000ff">object</span> data)<br />    {<br />        <span style="color: #0000ff">return</span> <span style="color: #0000ff">new</span> XmlActionResult(data);<br />    }<br />}</pre>
      
      <p>
        </div> 
        
        <p>
          Y ahora ya puedo realizar la acción de mi controlador:
        </p>
        
        <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
          <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px" id="codeSnippet"><span style="color: #0000ff">public</span> ActionResult List()<br />{<br />    var data = <span style="color: #0000ff">new</span> GeeksModel().GetAllGeeks();<br />    <span style="color: #0000ff">return</span> Request.AcceptTypes.Contains(<span style="color: #006080">"application/json"</span>) ?<br />        (ActionResult)Json(data, JsonRequestBehavior.AllowGet) :<br />        (ActionResult)<span style="color: #0000ff">this</span>.Xml(data);         <br />}</pre>
          
          <p>
            </div> 
            
            <p>
              Simplemente pregunto si está el accept <em>application/json(*)</em> <a href="http://www.ietf.org/rfc/rfc4627.txt">(que parece ser el content-type para JSON)</a>. Si lo está envío los datos en json y si no pues en xml! Si abrimos un navegador y vamos a /Geeks/List veremos los datos en XML porque ningún (bueno, ni FF ni IE que son los que he probado :p) envían <em>application/json</em> en el <em>accept</em> de la request.
            </p>
            
            <blockquote>
              <p>
                (*) Ok, acepto que esta pregunta no es del todo correcta: debería mirar si application/json está preferido <em>antes que</em> text/xml (por si me manda ambos). Igual que teoricamente, debería comprobar si no me manda ninguno de los dos, y si es el caso devolver un <a href="http://www.checkupdown.com/status/E406_es.html">error 406</a>.
              </p>
            </blockquote>
            
            <p>
              <strong>4. Un detallito final…</strong>
            </p>
            
            <p>
              Bueno, eso parece funcionar, pero lo que chirría un poco es tener que meter este if() para comprobar en <em>cada acción de cada controlador</em> si la request contiene application/json o no y serializar el resultado en JSON o en XML.
            </p>
            
            <p>
              Para evitar esto he encontrado dos alternativas en la red:
            </p>
            
            <ol>
              <li>
                Usar otro action result y que sea el action result quien decida si serializar los datos en XML o en JSON. Es decir, crearnos un JsonOrXmlActionResult, devolver siempre una instancia de este action result desde los controladores y en el <em>ExecuteResult</em>, preguntar por el campo <em>accept</em> y serializar en un formato en otro. Esta aproximación la he visto en el post “<a href="http://omaralzabir.com/create_rest_api_using_asp_net_mvc_that_speaks_both_json_and_plain_xml/">Create REST API using ASP.NET MVC that speaks both Json and plain Xml</a>” del blog de <strong>Omar Al Zabir</strong>.
              </li>
              <li>
                Otra aproximación totalmente distinta (pero muy interesante) que usa un <em>action filter</em> para ello. Está en el blog de <strong>Aleem Bawany</strong>, en el post “<a href="http://aleembawany.com/2009/03/27/aspnet-mvc-create-easy-rest-api-with-json-and-xml/">ASP.NET MVC – Create easy REST API with JSON and XML</a>”.
              </li>
            </ol>
            
            <p>
              Os recomiendo la lectura de estos dos posts.
            </p>
            
            <p>
              Un saludo y gracias a todos, especialmente a <strong>Hadi Hariri</strong> quien con su comentario anterior, ha motivado este post! 🙂
            </p>

 [1]: http://geeks.ms/blogs/etomas/archive/2010/09/09/asp-net-mvc-mostrar-datos-en-html-o-pdf-pero-en-el-fondo-vamos-a-hablar-de-la-tabla-de-rutas.aspx
 [2]: http://www.hadihariri.com/
 [3]: http://www.w3.org/Protocols/rfc2616/rfc2616-sec5.html
 [4]: http://www.w3.org/Protocols/rfc2616/rfc2616-sec14.html