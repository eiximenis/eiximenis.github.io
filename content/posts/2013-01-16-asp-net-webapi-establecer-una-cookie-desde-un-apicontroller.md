---
title: ASP.NET WebApi – Establecer una cookie desde un ApiController

author: eiximenis

date: 2013-01-16T16:47:06+00:00
geeks_url: /?p=1625
geeks_visits:
  - 1281
geeks_ms_views:
  - 803
categories:
  - Uncategorized

---
Hoy, para una prueba de concepto que estoy realizando, me he encontrado con la necesidad de establecer una cookie desde un controlador de ASP.NET WebApi (un ApiController). Por supuesto podríamos discutir largo y tendido sobre la conveniencia de hacer esto o no (si suponemos que dichos controladores van a ser accedidos por clientes que no sean navegadores y por lo tanto pueden no entender las cookies). Pero obviando esta discusión el problema está en que desde el controlador no podemos acceder al objeto Response (a diferencia de un controlador MVC tradicional). Por supuesto siempre existe la opción de usar HttpContext.Current pero eso no es deseable ya que ata mi controlador a un pipeline de ASP.NET por lo que pierdo la facilidad de pruebas.

La solución que he encontrado pasa por devolver un <a href="http://msdn.microsoft.com/es-es/library/system.net.http.httpresponsemessage.aspx" target="_blank" rel="noopener noreferrer">HttpResponseMessage</a> desde el controlador. En mi caso la firma original del método era:

<div style="font-size: 10pt; font-family: consolas; background: #1e1e1e; color: #dcdcdc">
  <p style="margin: 0px">
    <span style="color: #569cd6">public</span> <span style="color: #4ec9b0">AuthResultDTO</span> <span style="color: white">PostLoginByUserPass</span>(<span style="color: #4ec9b0">LoginDTO</span> <span style="color: white">login</span>)
  </p>
  
  <p style="margin: 0px">
    {
  </p>
  
  <p style="margin: 0px">
    }
  </p></p>
</div>

El primer paso es modificarlo para que devuelva un HttpResponseMessage y usar el HttpResponseMessage para establecer las cookies.

Primero me he creado un método para que me crease la cookie (nota para curiosos: lo que quería hacer era usar un ticket de autenticación forms personalizado):

<div style="font-size: 10pt; font-family: consolas; background: #1e1e1e; color: #dcdcdc">
  <p style="margin: 0px">
    <span style="color: #569cd6">private</span> <span style="color: #569cd6">static</span> <span style="color: #4ec9b0">CookieHeaderValue</span> <span style="color: white">CreateCustomCookie</span>(<span style="color: #569cd6">string</span> <span style="color: white">name</span>, <span style="color: #569cd6">int</span> <span style="color: white">id</span>)
  </p>
  
  <p style="margin: 0px">
    {
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: #569cd6">var</span> <span style="color: white">userData</span> <span style="color: #b4b4b4">=</span> <span style="color: white">id</span><span style="color: #b4b4b4">.</span><span style="color: white">ToString</span>();
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: #569cd6">var</span> <span style="color: white">ticket</span> <span style="color: #b4b4b4">=</span> <span style="color: #569cd6">new</span> <span style="color: #4ec9b0">FormsAuthenticationTicket</span>(<span style="color: #b5cea8">1</span>, <span style="color: white">name</span>, <span style="color: #4ec9b0">DateTime</span><span style="color: #b4b4b4">.</span><span style="color: white">Now</span>, <span style="color: #4ec9b0">DateTime</span><span style="color: #b4b4b4">.</span><span style="color: white">Now</span><span style="color: #b4b4b4">.</span><span style="color: white">AddMinutes</span>(<span style="color: #b5cea8">30</span>), <span style="color: #569cd6">false</span>, <span style="color: white">userData</span>);
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: #569cd6">var</span> <span style="color: white">encTicket</span> <span style="color: #b4b4b4">=</span> <span style="color: #4ec9b0">FormsAuthentication</span><span style="color: #b4b4b4">.</span><span style="color: white">Encrypt</span>(<span style="color: white">ticket</span>);
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: #569cd6">var</span> <span style="color: white">cookieValue</span> <span style="color: #b4b4b4">=</span> <span style="color: #569cd6">new</span> <span style="color: #4ec9b0">CookieHeaderValue</span>(<span style="color: #4ec9b0">FormsAuthentication</span><span style="color: #b4b4b4">.</span><span style="color: white">FormsCookieName</span>, <span style="color: white">encTicket</span>)
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; {
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: white">Path</span> <span style="color: #b4b4b4">=</span> <span style="color: #d69d85">"/"</span>,
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: white">HttpOnly</span> <span style="color: #b4b4b4">=</span>&#160; <span style="color: #569cd6">true</span>,
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; };
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: #569cd6">return</span> <span style="color: white">cookieValue</span>;
  </p>
  
  <p style="margin: 0px">
    }
  </p></p>
</div>

Este método crea un ticket con un campo addicional (el id del usuario) y crea el objeto CookieHeaderValue que es el que contiene los datos de las cookies a añadir.

Ahora toca modificar el método del controlador. Lo primero es hacer que devuelva un HttpResponseMessage en lugar del AuthResultDTO que devolvía antes:

<div style="font-size: 10pt; font-family: consolas; background: #1e1e1e; color: #dcdcdc">
  <p style="margin: 0px">
    <span style="color: #569cd6">public</span> <span style="color: #4ec9b0">HttpResponseMessage</span> <span style="color: white">PostLoginByUserPass</span>(<span style="color: #4ec9b0">LoginDTO</span> <span style="color: white">login</span>)
  </p>
  
  <p style="margin: 0px">
    {
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: #569cd6">var</span> <span style="color: white">response</span> <span style="color: #b4b4b4">=</span> <span style="color: #569cd6">new</span> <span style="color: #4ec9b0">HttpResponseMessage</span>();
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: #569cd6">return</span> <span style="color: white">response</span>;
  </p>
  
  <p style="margin: 0px">
    }
  </p></p>
</div>

El primer paso es añadir la cookie generada a nuestro mensaje:

<div style="font-size: 10pt; font-family: consolas; background: #1e1e1e; color: #dcdcdc">
  <p style="margin: 0px">
    <span style="color: #569cd6">var</span> <span style="color: white">cookieValue</span> <span style="color: #b4b4b4">=</span> <span style="color: white">CreateCustomCookie</span>(<span style="color: white">login</span><span style="color: #b4b4b4">.</span><span style="color: white">Name</span>, <span style="color: white">id</span>);
  </p>
  
  <p style="margin: 0px">
    <span style="color: white">response</span><span style="color: #b4b4b4">.</span><span style="color: white">Headers</span><span style="color: #b4b4b4">.</span><span style="color: white">AddCookies</span>(<span style="color: #569cd6">new</span> <span style="color: #4ec9b0">CookieHeaderValue</span>[] {<span style="color: white">cookieValue</span>});
  </p></p>
</div>

Pero ahora nos falta incrustar el AuthResultDTO dentro de la respuesta. Para ello debemos usar la propiedad Content del HttpResponseMessage (authResult es una variable de tipo AuthResultDTO):

<div style="font-size: 10pt; font-family: consolas; background: #1e1e1e; color: #dcdcdc">
  <p style="margin: 0px">
    <span style="color: white">response</span><span style="color: #b4b4b4">.</span><span style="color: white">Content</span> <span style="color: #b4b4b4">=</span> <span style="color: #569cd6">new</span> <span style="color: #4ec9b0">ObjectContent</span>(<span style="color: #569cd6">typeof</span>(<span style="color: #4ec9b0">AuthResultDTO</span>), <span style="color: white">authResult</span>, <span style="color: #569cd6">new</span> <span style="color: #4ec9b0">JsonMediaTypeFormatter</span>());
  </p></p>
</div>

Como vemos hemos de usar la clase <a href="http://msdn.microsoft.com/en-us/library/system.net.http.objectcontent(v=vs.108).aspx" target="_blank" rel="noopener noreferrer">ObjectContent</a> ya que en mi caso quiero mandar el objeto serializado. El tercer parámetro indica que serializador se usa. En mi caso uso el serializador de JSON (<a href="http://ms
dn.microsoft.com/es-es/library/system.net.http.formatting.jsonmediatypeformatter(v=vs.108).aspx" target="_blank" rel="noopener noreferrer">JsonMediaTypeFormatter</a>). Notad que aquí estoy fijando que la respuesta será siempre en JSON, con independencia de la cabecera _Accept_ que me mande el cliente (contraviniendo lo que hace WebApi por defecto).

¡Y listos! Con eso ya hemos conseguido establecer nuestra cookie desde un controlador de WebApi.

**Bonus track: ¿Cómo seleccionar el MediaTypeFormatter correcto?**

Como he dicho antes el usar JsonMediaTypeFormatter directamente estoy “rompiendo” la capacidad de WebApi de seleccionar el tipo de respuesta según la cabecera Accept que envíe el cliente. Por suerte no es muy dificil restablecer dicha funcionalidad. Basta con mirar en la configuración global de WebApi todos los formatters que haya y encontrar el primero que pueda procesar el tipo especificado. Una posible implementación sería:

<div style="font-size: 10pt; font-family: consolas; background: #1e1e1e; color: #dcdcdc">
  <p style="margin: 0px">
    <span style="color: #569cd6">private</span> <span style="color: #4ec9b0">MediaTypeFormatter</span> <span style="color: white">ChooseMediaTypeFormatter</span>(<span style="color: #4ec9b0">HttpHeaderValueCollection</span><span style="color: #b4b4b4"><</span><span style="color: #4ec9b0">MediaTypeWithQualityHeaderValue</span><span style="color: #b4b4b4">></span> <span style="color: white">accept</span>, <span style="color: #4ec9b0">Type</span> <span style="color: white">typeToWrite</span>)
  </p>
  
  <p style="margin: 0px">
    {
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: #569cd6">var</span> <span style="color: white">cfg</span> <span style="color: #b4b4b4">=</span> <span style="color: #4ec9b0">GlobalConfiguration</span><span style="color: #b4b4b4">.</span><span style="color: white">Configuration</span>;
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: #569cd6">foreach</span> (<span style="color: #569cd6">var</span> <span style="color: white">formatter</span> <span style="color: #569cd6">in</span> <span style="color: white">cfg</span><span style="color: #b4b4b4">.</span><span style="color: white">Formatters</span>)
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; {
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #569cd6">if</span> (<span style="color: white">formatter</span><span style="color: #b4b4b4">.</span><span style="color: white">SupportedMediaTypes</span><span style="color: #b4b4b4">.</span><span style="color: white">Any</span>(<span style="color: white">x</span> <span style="color: #b4b4b4">=></span> <span style="color: white">accept</span><span style="color: #b4b4b4">.</span><span style="color: white">Select</span>(<span style="color: white">a</span> <span style="color: #b4b4b4">=></span> <span style="color: white">a</span><span style="color: #b4b4b4">.</span><span style="color: white">MediaType</span>)<span style="color: #b4b4b4">.</span><span style="color: white">Contains</span>(<span style="color: white">x</span><span style="color: #b4b4b4">.</span><span style="color: white">MediaType</span>)) <span style="color: #b4b4b4">&&</span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: white">formatter</span><span style="color: #b4b4b4">.</span><span style="color: white">CanWriteType</span>(<span style="color: white">typeToWrite</span>)) <span style="color: #569cd6">return</span> <span style="color: white">formatter</span>;
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; }
  </p>
  
  <p style="margin: 0px">
    &#160;
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: #569cd6">return</span> <span style="color: #569cd6">new</span> <span style="color: #4ec9b0">JsonMediaTypeFormatter</span>();
  </p>
  
  <p style="margin: 0px">
    }
  </p></p>
</div>

Y llamaríamos a dicho método de la forma:

<div style="font-size: 10pt; font-family: consolas; background: #1e1e1e; color: #dcdcdc">
  <p style="margin: 0px">
    <span style="color: #569cd6">var</span> <span style="color: white">mediaFormatter</span> <span style="color: #b4b4b4">=</span> <span style="color: white">ChooseMediaTypeFormatter</span>(<span style="color: white">Request</span><span style="color: #b4b4b4">.</span><span style="color: white">Headers</span><span style="color: #b4b4b4">.</span><span style="color: white">Accept</span>, <span style="color: #569cd6">typeof</span>(<span style="color: #4ec9b0">AuthResultDTO</span>));
  </p></p>
</div>

> **Nota:** Realmente esta implementación de ChooseMediaTypeFormatter no es del todo correcta ya que no tiene en cuenta la calidad especificada en la cabecera accept… Pero bueno, creo que la idea de como se podría hacer ya queda clara, no?

Saludos!

> **Actualización:** Nicolás Herrera (@<a href="https://twitter.com/nicolocodev" target="_blank" rel="noopener noreferrer">nicolocodev</a>) me ha hecho dar cuenta de algo en lo que no he caído cuando he escrito el post. En un tweet me <a href="https://twitter.com/nicolocodev/status/291585272779513856" target="_blank" rel="noopener noreferrer">propone usar Request.CreateResponse</a> en lugar de buscar manualmente el MediaTypeFormatter como estoy haciendo yo. Y tiene toda la razón del mundo 🙂 Anotado queda!
> 
> Información del <a href="http://msdn.microsoft.com/query/dev11.query?appId=Dev11IDEF1&l=EN-US&k=k(System.Net.Http.HttpRequestMessageExtensions.CreateResponse);k(TargetFrameworkMoniker-.NETFramework,Version%3Dv4.0);k(DevLang-csharp)&rd=true" target="_blank" rel="noopener noreferrer">Método (de extensión) CreateRequest</a>.
> 
> Gracias! 😉