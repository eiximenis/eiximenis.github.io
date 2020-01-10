---
title: ASP.NET MVC – Patrón PRG sin sesión

author: eiximenis

date: 2013-05-25T16:50:00+00:00
geeks_url: /?p=1641
geeks_visits:
  - 2315
geeks_ms_views:
  - 1088
categories:
  - Uncategorized

---
Buenas! [El patrón PRG (Post &ndash; Redirect &ndash; Get)][1] es un patrón muy usado en el desarrollo web. Consiste en que la respuesta de una petición POST es siempre una redirección, lo que genera un GET del navegador y de ahí el nombre.

La idea que subyace tras el patrón PRG es, que dado que dado que las peticiones GET son (&iexcl;deberían ser!) idempotentes esas son las únicas que el usuario debe poder refrescar. De hecho los navegadores nos avisan si refrescamos una petición POST:

[<img height="269" width="484" src="/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_106234AD.png" alt="image" border="0" style="border-left-width: 0px; border-right-width: 0px; border-bottom-width: 0px; display: inline; border-top-width: 0px" title="image" />][2] 

La razón de este aviso no es tanto notificar al usuario que se reenviarán esos datos, la razón es que el hecho de que se envíen via POST hace que se asuma que dicha petición no es idempotente, o dicho de otro modo _modifica datos_ en el sistema (da de alta un usuario, o borra un producto o realiza una compra). Es pues un mecanismo de protección.

Para evitar esto en el patrón PRG cada método de acción que gestiona un POST, no devuelve la vista con el resultado de dicha petición si no que devuelve una redirección a otra acción que es la que muestra el resultado. Así si tenemos una vista que tiene un formulario como el siguiente:

<div class="csharpcode">
  <pre class="alt">@using (Html.BeginForm())</pre>
  
  <pre>{</pre>
  
  <pre class="alt">    @Html.TextBox("somevalue")</pre>
  
  <pre>    <span class="kwrd">&lt;</span><span class="html">input</span> <span class="attr">type</span><span class="kwrd">="submit"</span> <span class="attr">value</span><span class="kwrd">="send post"</span> <span class="kwrd">/&gt;</span></pre>
  
  <pre class="alt">}</pre>
</div>



El método de acción podría ser algo como:

<div class="csharpcode">
  <pre class="alt">[HttpPost]</pre>
  
  <pre><span class="kwrd">public</span> ActionResult Index(<span class="kwrd">string</span> somevalue)</pre>
  
  <pre class="alt">{</pre>
  
  <pre>    <span class="rem">// Procesar resultados.</span></pre>
  
  <pre class="alt">    var id = <span class="kwrd">new</span> Random().Next();</pre>
  
  <pre>    <span class="rem">// ...</span></pre>
  
  <pre class="alt">    <span class="kwrd">return</span> RedirectToAction(<span class="str">"View"</span>, <span class="kwrd">new</span> { id = id });</pre>
  
  <pre>}</pre>
</div>



El método que procesa el POST realiza las tareas que sean necesarias y luego redirecciona a otra acción, por lo que lo después de enviar el formulario el usuario refresca la página, refrescará la última petición web que es la petición GET (en lugar de intentar refrescar el POST).

Usar el patrón PRG es una muy buena práctica, pero conlleva un pequeño problema: como pasar información desde la acción POST hacia la acción GET.

P. ej. en el POST podemos crear o modificar datos de alguna entidad y luego en el GET podemos mostrar esa entidad creada o modificada. Una solución es pasarle el ID de la entidad (tal y como se hace en el código anterior). Eso es perfectamente válido pero implica un round-trip a la BBDD. En el POST teníamos los datos de toda la entidad, pero en el GET debemos recuperarlos de nuevo ya que solo tenemos el ID.

Si hubiese alguna manera de pasar todos los datos necesarios (p. ej. toda la entidad) des del POST hacía el GET entonces, en algunos casos, nos podríamos ahorrar tener que ir a buscar datos que ya teníamos.

En ASP.NET MVC existe un mecanismo pensado para transferir datos entre redirecciones, que es justo lo que necesitamos y es TempData:

<div class="csharpcode">
  <pre class="alt">[HttpPost]</pre>
  
  <pre><span class="kwrd">public</span> ActionResult Index(<span class="kwrd">string</span> somevalue)</pre>
  
  <pre class="alt">{</pre>
  
  <pre>    <span class="rem">// Procesar resultados.</span></pre>
  
  <pre class="alt">    var id = <span class="kwrd">new</span> Random().Next();</pre>
  
  <pre>    var someData = <span class="kwrd">new</span> Person() { </pre>
  
  <pre class="alt">       Id = id, Name = somevalue </pre>
  
  <pre>     };</pre>
  
  <pre class="alt">    TempData[<span class="str">"someData"</span>] = someData;</pre>
  
  <pre>    <span class="kwrd">return</span> RedirectToAction(<span class="str">"View"</span>, </pre>
  
  <pre class="alt">          <span class="kwrd">new</span> { id = id });</pre>
  
  <pre>}</pre>
  
  <pre class="alt">&nbsp;</pre>
  
  <pre>[ActionName(<span class="str">"View"</span>)]</pre>
  
  <pre class="alt"><span class="kwrd">public</span> ActionResult ViewGet(<span class="kwrd">string</span> id)</pre>
  
  <pre>{</pre>
  
  <pre class="alt">    var data = TempData[<span class="str">"someData"</span>] <span class="kwrd">as</span> Person;</pre>
  
  <pre>    <span class="kwrd">if</span> (data == <span class="kwrd">null</span>)</pre>
  
  <pre class="alt">    {</pre>
  
  <pre>        <span class="rem">// Recargar data usando el id que </span></pre>
  
  <pre class="alt">        <span class="rem">//tenemos por parámetro</span></pre>
  
  <pre>    }</pre>
  
  <pre class="alt">    <span class="kwrd">return</span> View(data);</pre>
  
  <pre>}</pre>
</div>



Fíjate que usar TempData no exime de pasar el id igualmente a la acción GET ya que los datos que recogemos de TempData pueden no existir. Si el usuario refresca la página o bien teclea directamente la URL de la acción View los datos de TempData no existirán. Recuerda que TempData es un contenedor que permite una sola lectura (los datos desaparecen una vez leídos).

Bien, el punto a tener en cuenta (y motivo principal de este post) al usar TempData es que éste usa sesión. Y no siempre podemos o queremos usar la sesión. Si por cualquier razón no queremos usar la sesión, ¿podemos seguir usando TempData?

La respuesta es que si, pero debes implementarte un **custom TempData provider**. Claro que la otra pregunta es donde podemos guardar esos datos. Recuerda que TempData debe persistir entre peticiones (de ahí que por defecto se use la sesión). No hay muchos lugares más donde lo podamos guardar, así que si estás pensando en cookies has dado en el clavo. Si en el POST enviamos una cookie, cuando el navegador realice la petición GET posterior reenviará la cookie que contendrá los datos de TempData. 

Para crear un proveedor propio de TempData tenemos que seguir 2 pasos:

  1. Crear una clase que implemente [ITempDataProvider][3]. Esta interfaz define dos métodos (SaveTempData y LoadTempData). 
  2. Redefinir el método [CreateTempDataProvider][4] de la clase Controller y devolver una instancia de nuestra clase que implementa ITempDataProvider. Esto debemos hacerlo en cada controlador que queramos que use nuestro TempData provider o bien lo podemos poner en un controlador base. 

No voy a colgarme medallas que no me pertenecen poniendo la implementación de un proveedor de TempData que use cookies, ya que hay varios por internet [e incluso en el código fuente de MVC4 viene uno][5]. Está en el código fuente pero no en los binarios, ya que forma parte del paquete [Mvc4Futures][6]. Si quieres usarlo, debes instalar primero este paquete usando <span style="font-family: Consola;">Install-Package Mvc4Futures</span> desde la cónsola de NuGet o bien usando la GUI.

Y ya puedes usar TempData sin necesidad de usar la sesión! 😉

Saludos!

 [1]: /blogs/jmaguilar/archive/2009/11/23/el-patr-243-n-post-redirect-get.aspx
 [2]: /cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_53F9E5CF.png
 [3]: http://msdn.microsoft.com/es-es/library/system.web.mvc.itempdataprovider(v=vs.100).aspx
 [4]: http://msdn.microsoft.com/es-es/library/system.web.mvc.controller.createtempdataprovider(v=vs.108).aspx
 [5]: http://aspnetwebstack.codeplex.com/SourceControl/latest#src/Microsoft.Web.Mvc/CookieTempDataProvider.cs
 [6]: http://nuget.org/packages/Mvc4Futures/