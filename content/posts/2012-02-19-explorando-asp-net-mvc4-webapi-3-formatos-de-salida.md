---
title: 'Explorando ASP.NET MVC4 WebAPI–3: Formatos de salida'
author: eiximenis

date: 2012-02-19T23:19:19+00:00
geeks_url: /?p=1588
geeks_visits:
  - 2527
geeks_ms_views:
  - 1231
categories:
  - Uncategorized

---
Bueno… seguimos esta serie explorando ASP.NET WebAPI. En este post vamos a hablar de los formatos de salida. Como ya hemos dicho, de serie ASP.NET WebAPI tiene soporte para XML y para JSON. Pero… como decide el framework si enviar la salida en XML o en JSON?

**La cabecera accept**

Una de las cabeceras que define HTTP es la cabecera accept. Esta cabecera se usa para que el cliente informe al servidor de los _tipos de datos_ (content type) que acepta. De nuevo un par de pruebas con fiddler nos permiten verlo fácilmente. Este va a ser nuestro controlador:

<div style="border-bottom: #000080 1px solid; border-left: #000080 1px solid; font-family: &#39;Courier New&#39;, courier, monospace; color: #000; font-size: 10pt; border-top: #000080 1px solid; border-right: #000080 1px solid">
  <div style="background: #ddd; max-height: 300px; overflow: auto">
    <ol style="padding-bottom: 0px; margin: 0px 0px 0px 2em; padding-left: 5px; padding-right: 0px; white-space: nowrap; background: #ffffff; padding-top: 0px">
      <li>
        <span style="color: #0000ff">public</span> <span style="color: #0000ff">class</span> <span style="color: #2b91af">ValuesController</span> : <span style="color: #2b91af">ApiController</span>
      </li>
      <li style="background: #f3f3f3">
        {
      </li>
      <li>
        &#160;&#160;&#160; <span style="color: #0000ff">public</span> <span style="color: #2b91af">IEnumerable</span><<span style="color: #0000ff">int</span>> GetAll()
      </li>
      <li style="background: #f3f3f3">
        &#160;&#160;&#160; {
      </li>
      <li>
        &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #0000ff">return</span> <span style="color: #2b91af">Enumerable</span>.Range(1, 30);
      </li>
      <li style="background: #f3f3f3">
        &#160;&#160;&#160; }
      </li>
      <li>
        }
      </li>
    </ol>
  </div></p>
</div>

Y ahora de nuevo usamos fiddler para crear y ver una petición GET a /api/values, como la siguiente:

<font face="Courier New">GET </font>[<font face="Courier New">http://worldoffighters.epnuke2.com:55603/api/values</font>][1] <font face="Courier New">HTTP/1.1 <br />User-Agent: Fiddler <br />Host: worldoffighters.epnuke2.com:55603</font>

La respuesta recibida es:

<font face="Courier New">HTTP/1.1 200 OK <br />Server: ASP.NET Development Server/10.0.0.0 <br />Date: Sun, 19 Feb 2012 20:08:20 GMT <br />X-AspNet-Version: 4.0.30319 <br />Cache-Control: no-cache <br />Pragma: no-cache <br />Expires: -1 <br />Content-Type: application/json; charset=utf-8 <br />Connection: Close <br />Content-Length: 82</font>

<font face="Courier New">[1,2,3,4,5,6,7,8,9,10,11,12,13,14,15,16,17,18,19,20,21,22,23,24,25,26,27,28,29,30]</font>

Bueno, parece pues claro que ante cualquier ausencia de accept, la salida se envía en JSON (content-type: application/json). Añadimos ahora una cabecera accept:

[<img style="background-image: none; border-right-width: 0px; padding-left: 0px; padding-right: 0px; display: inline; border-top-width: 0px; border-bottom-width: 0px; border-left-width: 0px; padding-top: 0px" title="image" border="0" alt="image" src="http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_533E248C.png" width="644" height="167" />][2]

Y generar una petición como la que sigue:

<font face="Courier New">GET </font>[<font face="Courier New">http://worldoffighters.epnuke2.com:55603/api/values</font>][1] <font face="Courier New">HTTP/1.1 <br />User-Agent: Fiddler <br />Host: worldoffighters.epnuke2.com:55603 <br /><strong><font color="#ff0000">accept: text/xml</font></strong></font>

Y esta es la respuesta que recibimos ahora:

<font face="Courier New">HTTP/1.1 200 OK <br />Server: ASP.NET Development Server/10.0.0.0 <br />Date: Sun, 19 Feb 2012 20:11:55 GMT <br />X-AspNet-Version: 4.0.30319 <br />Cache-Control: no-cache <br />Pragma: no-cache <br />Expires: -1 <br /><strong><font color="#ff0000">Content-Type: text/xml; charset=utf-8</font></strong> <br />Connection: Close <br />Content-Length: 543</font>

<p style="word-wrap: break-word">
  <font face="Courier New"><?xml version="1.0" encoding="utf-8"?><ArrayOfInt xmlns:xsi="</font><a href="http://www.w3.org/2001/XMLSchema-instance&quot;"><font face="Courier New">http://www.w3.org/2001/XMLSchema-instance"</font></a><font face="Courier New"> xmlns:xsd="http://www.w3.org/2001/XMLSchema"><int>1</int><int>2</int><int>3</int><int>4</int><int>5</int><int>6</int><int>7</int><int>8</int><int>9</int><int>10</int><int>11</int><int>12</int><int>13</int><int>14</int><int>15</int><int>16</int><int>17</int><int>18</int><int>19</int><int>20</int><int>21</int><int>22</int><int>23</int><int>24</int><int>25</int><int>26</int><int>27</int><int>28</int><int>29</int><int>30</int></ArrayOfInt></font>
</p>

Bueno, hemos visto el mecanismo que usa el framework para determinar el formato de salida: la [cabecera accept][3] (no es nada nuevo, es el standard de HTTP y de hecho ya hablé hace tiempo de como aplicarlo en ASP.NET MVC: <http://geeks.ms/blogs/etomas/archive/2010/09/10/asp-net-mvc-formato-de-salida-seg-250-n-content-type.aspx>).

Bueno, vamos a ver ahora como crear un tipo de salida nuevo y tenerlo vinculado a un content-type determinado.

**Usando MediaTypeFormatter**

Para añadir un formato de salida nuevo debemos crear una clase que derive de MediaTypeFormatter:

<div style="border-bottom: #000080 1px solid; border-left: #000080 1px solid; font-family: &#39;Courier New&#39;, courier, monospace; color: #000; font-size: 10pt; border-top: #000080 1px solid; border-right: #000080 1px solid">
  <div style="background: #ddd; max-height: 300px; overflow: auto">
    <ol style="padding-bottom: 0px; margin: 0px 0px 0px 2.5em; padding-left: 5px; padding-right: 0px; white-space: nowrap; background: #ffffff; padding-top: 0px">
      <li>
        <span style="color: #0000ff">public</span> <span style="color: #0000ff">class</span> <span style="color: #2b91af">MediaBinaryFormatter</span> : <span style="color: #2b91af">MediaTypeFormatter</span>
      </li>
      <li style="background: #f3f3f3">
        {
      </li>
      <li>
        &#160;&#160;&#160; <span style="color: #0000ff">public</span> MediaBinaryFormatter()
      </li>
      <li style="background: #f3f3f3">
        &#160;&#160;&#160; {
      </li>
      <li>
        &#160;&#160;&#160;&#160;&#160;&#160;&#160; SupportedMediaTypes.Add(<span style="color: #0000ff">new</span> <span style="color: #2b91af">MediaTypeHeaderValue</span>(<span style="color: #a31515">"application/octet-stream"</span>));
      </li>
      <li style="background: #f3f3f3">
        &#160;&#160;&#160; }
      </li>
      <li>
        &#160;
      </li>
      <li style="background: #f3f3f3">
        &#160;&#160;&#160; <span style="color: #0000ff">protected</span> <span style="color: #0000ff">override</span> <span style="color: #0000ff">bool</span> CanWriteType(<span style="color: #2b91af">Type</span> type)
      </li>
      <li>
        &#160;&#160;&#160; {
      </li>
      <li style="background: #f3f3f3">
        &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #0000ff">return</span> <span style="color: #0000ff">true</span>;
      </li>
      <li>
        &#160;<br /> &#160;&#160; }
      </li>
      <li style="background: #f3f3f3">
        &#160;
      </li>
      <li>
        &#160;&#160;&#160; <span style="color: #0000ff">protected</span> <span style="color: #0000ff">override</span> <span style="color: #2b91af">Task</span> OnWriteToStreamAsync(<span style="color: #2b91af">Type</span> type, <span style="color: #0000ff">object</span> value, System.IO.<span style="color: #2b91af">Stream</span> stream, <span style="color: #2b91af">HttpContentHeaders</span> contentHeaders, <span style="color: #2b91af">FormatterContext</span> formatterContext, System.Net.<span style="color: #2b91af">TransportContext</span> transportContext)
      </li>
      <li style="background: #f3f3f3">
        &#160;&#160;&#160; {
      </li>
      <li>
        &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #0000ff">return</span> <span style="color: #2b91af">Task</span>.Factory.StartNew(() =>
      </li>
      <li style="background: #f3f3f3">
        &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; {
      </li>
      <li>
        &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #0000ff">var</span> formatter = <span style="color: #0000ff">new</span> <span style="color: #2b91af">BinaryFormatter</span>();
      </li>
      <li style="background: #f3f3f3">
        &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; formatter.Serialize(stream, value);
      </li>
      <li>
        &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; });
      </li>
      <li style="background: #f3f3f3">
        &#160;&#160;&#160; }&#160;&#160;&#160;
      </li>
      <li>
        }
      </li>
    </ol>
  </div></p>
</div>

Hay tres aspectos interesantes a recalcar:

  1. En el constructor de la clase es donde vinculamos este serializador a un content-type específico, en este caso a _application/octet-stream_ 
  2. Indicamos que podemos serializar cualquier tipo .NET. Esto es porque siempre devolvemos _true_ en el método CanWriteType. Pero podríamos devolver _true_ solo para determinados tipos, lo que permite tener serializados personalizados para ciertos tipos de datos 😉 
  3. Finalmente en el método _OnWriteToStreamAsync_ es donde realizamos la serialización y escritura en el stream de salida. Fijaos que debemos devolver un objeto de la clase _Task_ con el código a ejecutar, ya que ese método será llamado de forma asíncrona por el framework (aunque a nosotros no nos preocupe demasiado). En este caso el código lo que hace es usar el BinaryFormatter de .NET para enviar la serialización en bnario del objeto que reciba. Por supuesto esto es a modo de demostración, ya que es lo más _anti-internet_ que existe: este formato de deserialización es propio de .NET por lo que tan solo un cliente .NET puede entenderlo. 

Y listos! Con esto **casi** hemos terminado… Nos falta simplemente registrar este MediaTypeFormatter en el framework. Y siguiendo la filosofía clásica de ASP.NET MVC la configuración está en una clase estática que podemos inicializar fácilmente desde gloabal.asax. En nuestro caso basta con añadir (p.ej. en el Application_Start) la línea:

<div style="border-bottom: #000080 1px solid; border-left: #000080 1px solid; font-family: &#39;Courier New&#39;, courier, monospace; color: #000; font-size: 10pt; border-top: #000080 1px solid; border-right: #000080 1px solid">
  <div style="background: #ddd; max-height: 300px; overflow: auto">
    <ol style="padding-bottom: 0px; margin: 0px 0px 0px 2em; padding-left: 5px; padding-right: 0px; background: #ffffff; padding-top: 0px">
      <li>
        <span style="color: #2b91af">GlobalConfiguration</span>.Configuration.Formatters.Add(<span style="color: #0000ff">new</span> <span style="color: #2b91af">MediaBinaryFormatter</span>());
      </li>
    </ol>
  </div></p>
</div>

Y ahora sí que hemos terminado. Si ejecutamos una petición con fiddler poniendo _application/octet-stream_ en la cabecera accept, esta es la respuesta:

[<img style="background-image: none; border-bottom: 0px; border-left: 0px; padding-left: 0px; padding-right: 0px; display: inline; border-top: 0px; border-right: 0px; padding-top: 0px" title="image" border="0" alt="image" src="http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_14A8B119.png" width="644" height="166" />][4]

Como se puede observar es una serialización binaria de .NET!

En resumen hemos visto como a través del header HTTP accept, el cliente puede especificar que formato de respuesta desea y como podemos añadir MediaTypeFormatters propios para dar soporte a otros tipos de datos que no sean JSON o XML.

Un saludo a todos!

 [1]: http://worldoffighters.epnuke2.com:55603/api/values
 [2]: http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_60187AAA.png
 [3]: http://www.w3.org/Protocols/rfc2616/rfc2616-sec14.html
 [4]: http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_6F8B66AC.png