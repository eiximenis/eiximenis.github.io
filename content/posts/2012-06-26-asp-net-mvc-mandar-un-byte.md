---
title: 'ASP.NET MVC: Mandar un byte[]'

author: eiximenis

date: 2012-06-26T23:12:02+00:00
geeks_url: /?p=1599
geeks_visits:
  - 2820
geeks_ms_views:
  - 1880
categories:
  - Uncategorized

---
Este post surge a raíz de la siguiente pregunta en los foros de ASP.NET MVC de MSDN: <http://social.msdn.microsoft.com/Forums/es-ES/aspnetmvces/thread/20a6935c-5903-4efd-8ca1-f5a70a047a15>. El usuario se pregunta como mandar un byte[] de la vista al controlador. Y comenta que lo hace de la siguiente manera:

<pre>&lt;iframe src="&lt;%: Url.Action("GenerarPdf", "Consulta", <br />new { documento = Model.Documento})%&gt;" width="725" height="725"&gt;&lt;/iframe&gt;</pre>

En el controlador tiene definida la acción correspondiente con un parámetro llamado documento de tipo byte[]. Y comenta que siempre recibe el parámetro con el valor null.

Veamos porque ocurre esto y cual es la solución.

**El porqué**

Primero veamos que URL nos genera una llamada a Url.Action como la siguiente:

<pre style="overflow: auto; border-top: #cecece 1px solid; border-right: #cecece 1px solid; border-bottom: #cecece 1px solid; padding-bottom: 5px; padding-top: 5px; padding-left: 5px; min-height: 40px; border-left: #cecece 1px solid; padding-right: 5px; width: 500px; background-color: #fbfbfb"><pre style="font-size: 12px; font-family: consolas,&#39;Courier New&#39;,courier,monospace; margin: 0em; width: 100%; background-color: #fbfbfb">@{
</pre>


<pre style="font-size: 12px; font-family: consolas,&#39;Courier New&#39;,courier,monospace; margin: 0em; width: 100%; background-color: #ffffff">    var data = new byte[] {0x10, 0x11, 0x12};
</pre>


<pre style="font-size: 12px; font-family: consolas,&#39;Courier New&#39;,courier,monospace; margin: 0em; width: 100%; background-color: #fbfbfb">}
</pre>


<pre style="font-size: 12px; font-family: consolas,&#39;Courier New&#39;,courier,monospace; margin: 0em; width: 100%; background-color: #ffffff"><span style="color: #0000ff">&lt;</span><span style="color: #800000">script</span> <span style="color: #ff0000">type</span>=<span style="color: #0000ff">"text/javascript"</span><span style="color: #0000ff">&gt;</span>
</pre>


<pre style="font-size: 12px; font-family: consolas,&#39;Courier New&#39;,courier,monospace; margin: 0em; width: 100%; background-color: #fbfbfb">    alert('@Url.Action("Index", "Home", new {documento=data})')
</pre>


<pre style="font-size: 12px; font-family: consolas,&#39;Courier New&#39;,courier,monospace; margin: 0em; width: 100%; background-color: #ffffff"><span style="color: #0000ff">&lt;/</span><span style="color: #800000">script</span><span style="color: #0000ff">&gt;</span></pre>


<p>
  El código HTML generado es el siguiente:
</p>


<pre style="overflow: auto; border-top: #cecece 1px solid; border-right: #cecece 1px solid; border-bottom: #cecece 1px solid; padding-bottom: 5px; padding-top: 5px; padding-left: 5px; min-height: 40px; border-left: #cecece 1px solid; padding-right: 5px; width: 500px; background-color: #fbfbfb"><pre style="font-size: 12px; font-family: consolas,&#39;Courier New&#39;,courier,monospace; margin: 0em; width: 100%; background-color: #fbfbfb"><span style="color: #0000ff">&lt;</span><span style="color: #800000">script</span> <span style="color: #ff0000">type</span>=<span style="color: #0000ff">"text/javascript"</span><span style="color: #0000ff">&gt;</span>
</pre>


<pre style="font-size: 12px; font-family: consolas,&#39;Courier New&#39;,courier,monospace; margin: 0em; width: 100%; background-color: #ffffff">    alert('/?documento=System.Byte%5B%5D')
</pre>


<pre style="font-size: 12px; font-family: consolas,&#39;Courier New&#39;,courier,monospace; margin: 0em; width: 100%; background-color: #fbfbfb"><span style="color: #0000ff">&lt;/</span><span style="color: #800000">script</span><span style="color: #0000ff">&gt;</span></pre>


<p>
  Qué ha ocurrido aquí? Pues que al intentar pasar un byte[] a través de la URL, el helper Url.Action ha llamado simplemente al método .ToString() de dicho byte[] (que siempre devuelve System.Byte[]). ¡Es evidente con esto que el Model Binder no va a poder recuperar los valores de dicho byte[]!
</p>


<p>
  <strong>La solución</strong>
</p>


<p>
  Lo que necesitamos pasar es el <em>contenido</em> del byte[], pero claro… en qué formato podemos hacerlo? Porque recordad que estamos pasando bytes, que es contenido binario, pero en las URLs (y en los cuerpos de las peticiones http) no hay contenido binario, hay texto (un subconjunto de ANSI). La mejor solución es usar Base64. Base64 es un mecanismo para codificar bytes (no carácteres, bytes)&#160; a un subconjunto de carácteres ANSI.
</p>


<p>
  En C# pasar un byte[] a su cadena Base64 equivalente es tan simple como llamar al método Convert.ToBase64String:
</p>


<pre style="overflow: auto; border-top: #cecece 1px solid; border-right: #cecece 1px solid; border-bottom: #cecece 1px solid; padding-bottom: 5px; padding-top: 5px; padding-left: 5px; min-height: 40px; border-left: #cecece 1px solid; padding-right: 5px; width: 500px; background-color: #fbfbfb"><pre style="font-size: 12px; font-family: consolas,&#39;Courier New&#39;,courier,monospace; margin: 0em; width: 100%; background-color: #fbfbfb"><span style="color: #0000ff">&lt;</span><span style="color: #800000">script</span> <span style="color: #ff0000">type</span>=<span style="color: #0000ff">"text/javascript"</span><span style="color: #0000ff">&gt;</span>
</pre>


<pre style="font-size: 12px; font-family: consolas,&#39;Courier New&#39;,courier,monospace; margin: 0em; width: 100%; background-color: #ffffff">    alert('@Url.Action("Index", "Home", new {documento=Convert.ToBase64String(data)})')
</pre>


<pre style="font-size: 12px; font-family: consolas,&#39;Courier New&#39;,courier,monospace; margin: 0em; width: 100%; background-color: #fbfbfb"><span style="color: #0000ff">&lt;/</span><span style="color: #800000">script</span><span style="color: #0000ff">&gt;</span></pre>


<p>
  Ahora el código HTML generado es:
</p>


<pre style="overflow: auto; border-top: #cecece 1px solid; border-right: #cecece 1px solid; border-bottom: #cecece 1px solid; padding-bottom: 5px; padding-top: 5px; padding-left: 5px; min-height: 40px; border-left: #cecece 1px solid; padding-right: 5px; width: 500px; background-color: #fbfbfb"><pre style="font-size: 12px; font-family: consolas,&#39;Courier New&#39;,courier,monospace; margin: 0em; width: 100%; background-color: #fbfbfb"><span style="color: #0000ff">&lt;</span><span style="color: #800000">script</span> <span style="color: #ff0000">type</span>=<span style="color: #0000ff">"text/javascript"</span><span style="color: #0000ff">&gt;</span>
</pre>


<pre style="font-size: 12px; font-family: consolas,&#39;Courier New&#39;,courier,monospace; margin: 0em; width: 100%; background-color: #ffffff">    alert('/?documento=EBES')
</pre>


<pre style="font-size: 12px; font-family: consolas,&#39;Courier New&#39;,courier,monospace; margin: 0em; width: 100%; background-color: #fbfbfb"><span style="color: #0000ff">&lt;/</span><span style="color: #800000">script</span><span style="color: #0000ff">&gt;</span></pre>


<p>
  La cadena EBES es la codificación en Base64 del array de bytes. Ahora nos toca recibir esto en el controlador. Primero voy a modificar la vista para que genere un enlace:
</p>


<pre style="overflow: auto; border-top: #cecece 1px solid; border-right: #cecece 1px solid; border-bottom: #cecece 1px solid; padding-bottom: 5px; padding-top: 5px; padding-left: 5px; min-height: 40px; border-left: #cecece 1px solid; padding-right: 5px; width: 500px; background-color: #fbfbfb"><pre style="font-size: 12px; font-family: consolas,&#39;Courier New&#39;,courier,monospace; margin: 0em; width: 100%; background-color: #fbfbfb">@{
</pre>


<pre style="font-size: 12px; font-family: consolas,&#39;Courier New&#39;,courier,monospace; margin: 0em; width: 100%; background-color: #ffffff">    var data = new byte[] {0x10, 0x11, 0x12};
</pre>


<pre style="font-size: 12px; font-family: consolas,&#39;Courier New&#39;,courier,monospace; margin: 0em; width: 100%; background-color: #fbfbfb">}
</pre>


<pre style="font-size: 12px; font-family: consolas,&#39;Courier New&#39;,courier,monospace; margin: 0em; width: 100%; background-color: #ffffff"><span style="color: #0000ff">&lt;</span><span style="color: #800000">a</span> <span style="color: #ff0000">href</span>=<span style="color: #0000ff">"@Url.Action("</span><span style="color: #ff0000">Ver</span>", "<span style="color: #ff0000">Home</span>", <span style="color: #ff0000">new</span> {<span style="color: #ff0000">documento</span>=<span style="color: #0000ff">Convert.ToBase64String(data)})</span>"<span style="color: #0000ff">&gt;</span>Pulsar aquí<span style="color: #0000ff">&lt;/</span><span style="color: #800000">a</span><span style="color: #0000ff">&gt;</span></pre>


<p>
  Y ahora la acción Ver del controlador:
</p>


<pre style="overflow: auto; border-top: #cecece 1px solid; border-right: #cecece 1px solid; border-bottom: #cecece 1px solid; padding-bottom: 5px; padding-top: 5px; padding-left: 5px; min-height: 40px; border-left: #cecece 1px solid; padding-right: 5px; width: 500px; background-color: #fbfbfb"><pre style="font-size: 12px; font-family: consolas,&#39;Courier New&#39;,courier,monospace; margin: 0em; width: 100%; background-color: #fbfbfb"><span style="color: #0000ff">public</span> ActionResult Ver(<span style="color: #0000ff">byte</span>[] documento)
</pre>


<pre style="font-size: 12px; font-family: consolas,&#39;Courier New&#39;,courier,monospace; margin: 0em; width: 100%; background-color: #ffffff">{
</pre>


<pre style="font-size: 12px; font-family: consolas,&#39;Courier New&#39;,courier,monospace; margin: 0em; width: 100%; background-color: #fbfbfb">  <span style="color: #008000">// Código</span>
</pre>


<pre style="font-size: 12px; font-family: consolas,&#39;Courier New&#39;,courier,monospace; margin: 0em; width: 100%; background-color: #ffffff">}</pre>


<p>
  Y esto es lo que recibimos en el controlador:
</p>


<p>
  <img title="image" style="border-left-width: 0px; border-right-width: 0px; border-bottom-width: 0px; display: inline; border-top-width: 0px" border="0" alt="image" src="http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_15DEFFDC.png" width="449" height="89" /> 
</p>


<p>
  Como podemos ver ASP.NET MVC ha sido capaz de convertir <strong>automáticamente</strong>la cadena en formato BASE64 a un array de bytes!
</p>


<p>
  Por supuesto aquí he usado @Url.Action para mandar el array de bytes a través de la URL, pero si usáis un @Html.Hidden para mandarlo a través de un campo Hidden os funcionará igual. La clave es que el byte[] esté codificado usando Base64.
</p>


<p>
  <strong>Disclaimer</strong>
</p>


<p>
  En mi post he usado MVC3, pero en el foro el usuario usaba MVC2. No tengo ahora una versión de MVC2 a mano para probar si esta conversión automática de cadenas Base64 a byte[] ocurre como en MVC3. Creo que sí (aunque debería comprobarlo) <strong>pero supongamos que no</strong>. 
</p>


<p>
  Supongamos que si mandas una cadena Base64 recibes un null si el parámetro es de tipo byte[]. En este caso… ¿qué deberías hacer?
</p>


<p>
  Pues la solución pasa por crearte tu propio Model Binder. Este Model Binder es el que recibiría una cadena (en formato Base64) y la transformaría a un byte[]:
</p>


<pre style="overflow: auto; border-top: #cecece 1px solid; border-right: #cecece 1px solid; border-bottom: #cecece 1px solid; padding-bottom: 5px; padding-top: 5px; padding-left: 5px; min-height: 40px; border-left: #cecece 1px solid; padding-right: 5px; width: 500px; background-color: #fbfbfb"><pre style="font-size: 12px; font-family: consolas,&#39;Courier New&#39;,courier,monospace; margin: 0em; width: 100%; background-color: #fbfbfb"><span style="color: #0000ff">public</span> <span style="color: #0000ff">class</span> Base64ModelBinder : IModelBinder
</pre>


<pre style="font-size: 12px; font-family: consolas,&#39;Courier New&#39;,courier,monospace; margin: 0em; width: 100%; background-color: #ffffff">{
</pre>


<pre style="font-size: 12px; font-family: consolas,&#39;Courier New&#39;,courier,monospace; margin: 0em; width: 100%; background-color: #fbfbfb">    <span style="color: #0000ff">public</span> <span style="color: #0000ff">object</span> BindModel(ControllerContext controllerContext, ModelBindingContext bindingContext)
</pre>


<pre style="font-size: 12px; font-family: consolas,&#39;Courier New&#39;,courier,monospace; margin: 0em; width: 100%; background-color: #ffffff">    {
</pre>


<pre style="font-size: 12px; font-family: consolas,&#39;Courier New&#39;,courier,monospace; margin: 0em; width: 100%; background-color: #fbfbfb">        var data = bindingContext.ValueProvider.GetValue(bindingContext.ModelName).AttemptedValue;
</pre>


<pre style="font-size: 12px; font-family: consolas,&#39;Courier New&#39;,courier,monospace; margin: 0em; width: 100%; background-color: #ffffff">        var array = Convert.FromBase64String(data);
</pre>


<pre style="font-size: 12px; font-family: consolas,&#39;Courier New&#39;,courier,monospace; margin: 0em; width: 100%; background-color: #fbfbfb">        <span style="color: #0000ff">return</span> array;
</pre>


<pre style="font-size: 12px; font-family: consolas,&#39;Courier New&#39;,courier,monospace; margin: 0em; width: 100%; background-color: #ffffff">    }
</pre>


<pre style="font-size: 12px; font-family: consolas,&#39;Courier New&#39;,courier,monospace; margin: 0em; width: 100%; background-color: #fbfbfb">}</pre>


<p>
  Este ModelBinder convierte el valor en Base64 a un array de bytes. Por supuesto faltaría añadir código de comprobación de errores.
</p>


<p>
  Ahora simplemente debemos decirle a ASP.NET MVC que use este ModelBinder cuando se encuentre con un byte[]:
</p>


<pre style="overflow: auto; border-top: #cecece 1px solid; border-right: #cecece 1px solid; border-bottom: #cecece 1px solid; padding-bottom: 5px; padding-top: 5px; padding-left: 5px; min-height: 40px; border-left: #cecece 1px solid; padding-right: 5px; width: 500px; background-color: #fbfbfb"><pre style="font-size: 12px; font-family: consolas,&#39;Courier New&#39;,courier,monospace; margin: 0em; width: 100%; background-color: #fbfbfb">ModelBinders.Binders.Remove(<span style="color: #0000ff">typeof</span> (<span style="color: #0000ff">byte</span>[]));
</pre>


<pre style="font-size: 12px; font-family: consolas,&#39;Courier New&#39;,courier,monospace; margin: 0em; width: 100%; background-color: #ffffff">ModelBinders.Binders.Add(<span style="color: #0000ff">typeof</span>(<span style="color: #0000ff">byte</span>[]), <span style="color: #0000ff">new</span> Base64ModelBinder());</pre>


<p>
  Fijaos que primero elimino el ModelBinder asociado a byte[] y luego asocio mi Model Binder al tipo byte[]. La primera línea es necesaria porque ASP.NET MVC3 <strong>ya tiene un Model Binder asociado a byte[] </strong>(lógico,simplemente MVC3 trae de serie lo que estamos comentando justo ahora)<strong>:</strong> Concretamente uno llamado ByteArrayModelBinder (<a href="http://msdn.microsoft.com/en-us/library/system.web.mvc.bytearraymodelbinder.aspx">http://msdn.microsoft.com/en-us/library/system.web.mvc.bytearraymodelbinder.aspx</a>).
</p>


<p>
  Insisto en que creo que MVC2 lo tiene también, pero bueno, estamos suponiendo que no (en este caso la llamada&#160; a Remove no sería necesaria claro).
</p>


<p>
  Y listos, con esto ya podemos enlazar cadenas en Base64 a parámetros byte[].
</p>


<p>
  Un saludo!
</p>