---
title: Como hacer seguros tus servicios WebApi
description: Como hacer seguros tus servicios WebApi
author: eiximenis

date: 2013-02-20T12:46:28+00:00
geeks_url: /?p=1632
geeks_visits:
  - 8930
geeks_ms_views:
  - 8922
categories:
  - Uncategorized

---
Buenas! Este post surge a raíz del comentario de Felix Manuel en mi post anterior <a href="http://geeks.ms/blogs/etomas/archive/2013/01/31/inyecci-243-n-de-dependencias-per-request-en-mvc4-y-webapi.aspx" target="_blank" rel="noopener noreferrer">Inyección de dependencias per-Request</a>. En él Felix comentaba que le gustaría ver algo sobre autenticación y autorización de WebApi… así que vamos a ello.

Todo lo que comentaré en este post va destinado a servicios WebApi que queramos tener en internet. No hablaré nada de otros escenarios como intranets que pueden tener otras soluciones.

Vamos a partir del template de proyecto ASP.NET MVC4 – WebApi

**AuthorizeAttribute**

En ASP.NET MVC si queremos añadir seguridad a un controlador, basta con que usemos [Authorize] para decorar todas aquellas acciones que requieran seguridad.

¿Qué ocurre si aplicamos lo mismo a WebApi?

<div style="font-size: 10pt; font-family: consolas; background: #1e1e1e; color: #dcdcdc">
  <p style="margin: 0px">
    <span style="color: #569cd6">public</span> <span style="color: #569cd6">class</span> <span style="color: #4ec9b0">SecureController</span> : <span style="color: #4ec9b0">ApiController</span>
  </p>
  
  <p style="margin: 0px">
    {
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; [<span style="color: #4ec9b0">Authorize</span>]
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: #569cd6">public</span> <span style="color: #569cd6">string</span> <span style="color: white">GetSecure</span>()
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; {
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #569cd6">return</span> <span style="color: #d69d85">"Acces granted!"</span>;
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; }
  </p>
  
  <p style="margin: 0px">
    }
  </p></p>
</div>

Asegúrate de usar el <a href="http://msdn.microsoft.com/es-es/library/system.web.http.authorizeattribute(v=vs.108).aspx" target="_blank" rel="noopener noreferrer">AuthorizeAttribute</a> que está en System.Web.Http, no el que está en System.Web.Mvc!

Con esto, si realizo una llamada GET a /api/secure obtengo un mensaje de error (recuerda que puedes recibirlo en json o xml dependiendo de la cabecera accept que mandes):

<font size="2" face="Courier New"><Error><Message>Authorization has been denied for this request.</Message></Error></font>

Bien… parece que el filtro realiza su trabajo, pero ¿qué hace exactamente? Para ello echemos un vistazo a <a href="http://aspnetwebstack.codeplex.com/SourceControl/changeset/view/b2ed65aea577#src/System.Web.Http/AuthorizeAttribute.cs" target="_blank" rel="noopener noreferrer">su código fuente</a>. El método que realiza el trabajo es IsAuthorized que tiene un código como el siguiente:

<div style="font-size: 10pt; font-family: consolas; background: #1e1e1e; color: #dcdcdc">
  <p style="margin: 0px">
    <span style="color: #b8d7a3">IPrincipal</span> <span style="color: white">user</span> <span style="color: #b4b4b4">=</span> <span style="color: #4ec9b0">Thread</span><span style="color: #b4b4b4">.</span><span style="color: white">CurrentPrincipal</span>;
  </p>
  
  <p style="margin: 0px">
    <span style="color: #569cd6">if</span> (<span style="color: white">user</span> <span style="color: #b4b4b4">==</span> <span style="color: #569cd6">null</span> <span style="color: #b4b4b4">||</span> <span style="color: #b4b4b4">!</span><span style="color: white">user</span><span style="color: #b4b4b4">.</span><span style="color: white">Identity</span><span style="color: #b4b4b4">.</span><span style="color: white">IsAuthenticated</span>)
  </p>
  
  <p style="margin: 0px">
    {
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: #569cd6">return</span> <span style="color: #569cd6">false</span>;
  </p>
  
  <p style="margin: 0px">
    }
  </p></p>
</div>

Bueno… pues parece que está claro: [Authorize] se limita a validar que haya un CurrentPrincipal que esté autorizado.

¿Así que la pregunta nos rebota: quien crea y coloca el IPrincipal autorizado?

Para verlo, realicemos una prueba: Vamos a crear un controlador nuevo con un método GET (para llamarlo fácilmente desde el navegador) para autenticarnos, usando <a href="http://msdn.microsoft.com/es-es/library/system.web.security.formsauthentication(v=vs.100).aspx" target="_blank" rel="noopener noreferrer">FormsAuthentication</a> (el mecanismo clásico de autenticación de ASP.NET).

<div style="font-size: 10pt; font-family: consolas; background: #1e1e1e; color: #dcdcdc">
  <p style="margin: 0px">
    <span style="color: #569cd6">public</span> <span style="color: #569cd6">class</span> <span style="color: #4ec9b0">AuthController</span> : <span style="color: #4ec9b0">ApiController</span>
  </p>
  
  <p style="margin: 0px">
    {
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: #569cd6">public</span> <span style="color: #569cd6">string</span> <span style="color: white">Get</span>(<span style="color: #569cd6">string</span> <span style="color: white">id</span>)
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; {
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #4ec9b0">FormsAuthentication</span><span style="color: #b4b4b4">.</span><span style="color: white">SetAuthCookie</span>(<span style="color: white">id</span> <span style="color: #b4b4b4">??</span> <span style="color: #d69d85">"FooUser"</span>, <span style="color: #569cd6">false</span>);
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #569cd6">return</span> <span style="color: #d69d85">"You are autenthicated now"</span>;
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; }
  </p>
  
  <p style="margin: 0px">
    }
  </p></p>
</div>

Vale, ahora puedes probar de realizar desde el navegador:

  1. Una llamada a /api/Auth/XXX (XXX nombre de usuario que quieras) 
  2. Otra llamada a api/Secure y ver… como sigues **sin estar** autenticado. 

La llamada&#160; a FormsAuthentication lo que hace es colocar una cookie (llamada comúnmente ASPXAUTH) que es la cookie de autenticación de asp.net. Pero por alguna razón la estamos obviando. Bien, esta razón es que NO hemos definido modelo de seguridad en web.config. Si lo abres verás que hay la línea:

<div style="font-size: 10pt; font-family: consolas; background: #1e1e1e; color: #dcdcdc">
  <p style="margin: 0px">
    <span style="color: gray"><</span><span style="color: #569cd6">authentication</span><span style="color: gray"> </span><span style="color: #92caf4">mode</span><span style="color: gray">="</span><span style="color: #c8c8c8">None</span><span style="color: gray">" /></span>
  </p></p>
</div>

Modificala para que sea:

<div style="font-size: 10pt; font-family: consolas; background: #1e1e1e; color: #dcdcdc">
  <p style="margin: 0px">
    <span style="color: gray"><</span><span style="color: #569cd6">authentication</span><span style="color: gray"> </span><span style="color: #92caf4">mode</span><span style="color: gray">="</span><span style="color: #c8c8c8">Forms</span><span style="color: gray">" /></span>
  </p></p>
</div>

¡Y listos! Si repites el par de llamadas anterior, ahora verás como la llamada a /api/Secure si que te funciona, ya que ahora el proveedor de autenticación Forms utiliza la cookie y en base a ella crea un IPrincipal y lo autentica.

¿Qué, sencillo no? Pues no tanto…

**Cuando usar y cuando no usar autenticación por Forms**

Usar la autenticación Forms NO es mala idea si tus servicios WebApi van a ser utilizados **tan solo** desde el contexto de una aplicación web que antes haya autenticado al usuario. ¿Y por qué? Bueno… pues porque las cookies son algo muy del mundo web.

Pero si creas una API para ser usada en otros dispositivos o aplicaciones (p. ej. una aplicación nativa de móvil) usar cookies para autenticarnos es una mala idea, ya que entonces obligas a quien usa tus servicios a que ponga código para leer las cookies recibidas y enviarlas de nuevo. Eso, si usas un navega
  
dor, lo hace el navegador automáticamente.

Así que lo dicho: si tus servicios WebApi van a ser utilizados tan solo dentro de una aplicación web (p. ej. para darte soporte a llamadas Ajax) puedes usar el mecanismo que acabamos de ver. En este caso, habitualmente, los controladores WebApi se despliegan en el mismo proyecto que los de ASP.NET MVC (si no recuerda que deberás lidiar con cross-domain). Y quien autentica al usuario (o sea quien tiene la llamada a FormsAuthentication.SetAuthCookie) es un controlador de MVC (el que responde al formulario de Login, claro). Y los controladores WebApi los tienes decorados tan solo con el [Authorize].

Pero vamos a olvidarnos de este escenario y planteemos otro: estás creando simplemente una API. El consumidor de esta API va a ser una aplicación de móvil (p. ej.) o dicho de otro modo, no va a ser un browser.

Entonces NO debemos usar autenticación por Forms y debemos buscar otros mecanismos. La pregunta es… ¿Cuales?

**Autenticación básica**

Bueno… exploremos esta opción. La autenticación básica de HTTP es muy sencilla. Aunque tiene un flujo que empieza por mandar un 401 con una cabecera específica todo eso lo obviaremos (eso es porque los navegadores sepan que hay un proceso de autenticación pero en nuestro caso el cliente no es un navegador).

Asi, si nos centramos en como es una petición autenticada que use autenticación básica, es muy sencillo: Tiene una cabecera _Authorization_ con este formato:

<font size="2" face="Courier New">Authorization: Basic username:password</font>

La única salvedad es que username:password se envía codificado en BASE64.

Vale… ¿y como implementamos eso en WebApi? Pues hay varias maneras, p. ej. nos podríamos crear un filtro de autorización propio:

<div style="font-size: 10pt; font-family: consolas; background: #1e1e1e; color: #dcdcdc">
  <p style="margin: 0px">
    <span style="color: #569cd6">public</span> <span style="color: #569cd6">class</span> <span style="color: #4ec9b0">BasicAuthorizeAttribute</span> : <span style="color: #4ec9b0">AuthorizationFilterAttribute</span>
  </p>
  
  <p style="margin: 0px">
    {
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: #569cd6">public</span> <span style="color: #569cd6">override</span> <span style="color: #569cd6">void</span> <span style="color: white">OnAuthorization</span>(<span style="color: #4ec9b0">HttpActionContext</span> <span style="color: white">actionContext</span>)
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; {
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #569cd6">var</span> <span style="color: white">headers</span> <span style="color: #b4b4b4">=</span> <span style="color: white">actionContext</span><span style="color: #b4b4b4">.</span><span style="color: white">Request</span><span style="color: #b4b4b4">.</span><span style="color: white">Headers</span>;
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #569cd6">if</span> (<span style="color: white">headers</span><span style="color: #b4b4b4">.</span><span style="color: white">Authorization</span> <span style="color: #b4b4b4">!=</span> <span style="color: #569cd6">null</span> <span style="color: #b4b4b4">&&</span> <span style="color: white">headers</span><span style="color: #b4b4b4">.</span><span style="color: white">Authorization</span><span style="color: #b4b4b4">.</span><span style="color: white">Scheme</span> <span style="color: #b4b4b4">==</span> <span style="color: #d69d85">"Basic"</span>)
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; {
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #569cd6">try</span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; {
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #569cd6">var</span> <span style="color: white">userPwd</span> <span style="color: #b4b4b4">=</span> <span style="color: #4ec9b0">Encoding</span><span style="color: #b4b4b4">.</span><span style="color: white">UTF8</span><span style="color: #b4b4b4">.</span><span style="color: white">GetString</span>(<span style="color: #4ec9b0">Convert</span><span style="color: #b4b4b4">.</span><span style="color: white">FromBase64String</span>(<span style="color: white">headers</span><span style="color: #b4b4b4">.</span><span style="color: white">Authorization</span><span style="color: #b4b4b4">.</span><span style="color: white">Parameter</span>));
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #569cd6">var</span> <span style="color: white">user</span> <span style="color: #b4b4b4">=</span> <span style="color: white">userPwd</span><span style="color: #b4b4b4">.</span><span style="color: white">Substring</span>(<span style="color: #b5cea8"></span>, <span style="color: white">userPwd</span><span style="color: #b4b4b4">.</span><span style="color: white">IndexOf</span>(<span style="color: #d69d85">&#8216;:&#8217;</span>));
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #569cd6">var</span> <span style="color: white">password</span> <span style="color: #b4b4b4">=</span> <span style="color: white">userPwd</span><span style="color: #b4b4b4">.</span><span style="color: white">Substring</span>(<span style="color: white">userPwd</span><span style="color: #b4b4b4">.</span><span style="color: white">IndexOf</span>(<span style="color: #d69d85">&#8216;:&#8217;</span>) <span style="color: #b4b4b4">+</span> <span style="color: #b5cea8">1</span>);
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #608b4e">// Validamos user y password (aquí asumimos que siempre son ok)</span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; }
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #569cd6">catch</span> (<span style="color: #4ec9b0">Exception</span>)
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; {
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: white">PutUnauthorizedResult</span>(<span style="color: white">actionContext</span>, <span style="color: #d69d85">"Invalid Authorization header"</span>);
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; }
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; }
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #569cd6">else</span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; {
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #608b4e">// No hay el header Authorization</span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: white">PutUnauthorizedResult</span>(<span style="color: white">actionContext</span>, <span style="color: #d69d85">"Auhtorization needed"</span>);
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; }
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; }
  </p>
  
  <p style="margin: 0px">
    &#160;
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: #569cd6">private</span> <span style="color: #569cd6">void</span> <span style="color: white">PutUnauthorizedResult</span>(<span style="color: #4ec9b0">HttpActionContext</span> <span style="color: white">actionContext</span>, <span style="color: #569cd6">string</span> <span style="color: white">msg</span>)
  </p>
  
  <p s
tyle="margin: 0px">
    &#160;&#160;&#160; {
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: white">actionContext</span><span style="color: #b4b4b4">.</span><span style="color: white">Response</span> <span style="color: #b4b4b4">=</span> <span style="color: #569cd6">new</span> <span style="color: #4ec9b0">HttpResponseMessage</span>(<span style="color: #b8d7a3">HttpStatusCode</span><span style="color: #b4b4b4">.</span><span style="color: white">Unauthorized</span>)
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; {
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: white">Content</span> <span style="color: #b4b4b4">=</span> <span style="color: #569cd6">new</span> <span style="color: #4ec9b0">StringContent</span>(<span style="color: white">msg</span>)
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; };
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; }
  </p>
  
  <p style="margin: 0px">
    }
  </p></p>
</div>

> **Nota:** Para realizar tareas de autorización (como este caso) debemos usar un filtro de autorización. **No uses para ello un filtro de acción** (es decir, no derives de ActionFilterAttribute y redefinas OnActionExecuting). La razón es que los filtros de autorización se ejecutan _antes_ que los filtros de acción, ya que precisamente su tarea es esa: autorizar la llamada a una acción. No está de más que te descargues el poster con el ciclo de vida de una petición WebApi de [http://www.microsoft.com/en-us/download/details.aspx?id=36476][1]

Ahora tan solo debemos decorar la acción Secure del controlador con [BasicAuthorize] en lugar de [Authorize] y ya tenemos la autenticación básica implementada (para pasar de cadena a base64 puedes usar cualquiera de las páginas que hay por ahí que lo hacen como [http://www.motobit.com/util/base64-decoder-encoder.asp][2]):

[<img title="image" style="border-left-width: 0px; border-right-width: 0px; background-image: none; border-bottom-width: 0px; padding-top: 0px; padding-left: 0px; display: inline; padding-right: 0px; border-top-width: 0px" border="0" alt="image" src="http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_488D02A1.png" width="504" height="113" />][3]

Si eliminar el tag Authorization verás como la respuesta pasa a ser un 401. 

Por supuesto, la autenticación básica de HTTP es tan insegura que tan solo debería usarse sobre HTTPS.

Vale… pero quizá te estás preguntando si tenemos más alternativas que usar un filtro de autorización para validar si una petición está autorizada. Y la respuesta es que sí. Tenemos al menos dos más:

  1. Usar un Message Handler 
  2. Usar un módulo de autenticación de ASP.NET 

¿Te parece si exploramos ambas?

**Usando un Message Handler**

Los Message Handlers se ejecutan incluso antes que los filtros de autorización. Es más, se ejecutan incluso antes de que el pipeline de WebApi seleccione un controlador así que son un lugar propicio para poner código de autorización.

Los Message Handlers tienen la potestad de generar ellos mismos un resultado y entonces todo el resto de la pipeline de WebApi es eliminada. Es decir es posible que un Message Handler intercepte una petición, genere un resultado y entonces esta petición no será transferida al controlador a la cual debería haber llegado.

Esto puede parecer que hace que los Message Handlers sean un buen sitio para autorizar peticiones y lo son, pero NO son un buen sitio para rechazar peticiones no autorizadas. Me explico: sigamos con nuestro ejemplo de autenticación básica.

Si usas un Message Handler y rechazamos todas las peticiones que NO tengan la cabecera Authorize entonces tendremos un problema **si necesitamos tener alguna parte de la API pública**. Un Message Handler se ejecuta _siempre_. Para _todas_ las peticiones.

Si quieres usar un Message Handler para autorización lo que debes hacer no es rechazar las peticiones que estén autorizadas (en nuestro caso que no tengan la cabecera Authorize). No. En su lugar debes autorizar aquellas peticiones válidas. ¿Como? Colocando un IPrincipal en el Thread. Y luego, en los controladores utilizas [Authorize].

Veamos un ejemplo rápido:

<div style="font-size: 10pt; font-family: consolas; background: #1e1e1e; color: #dcdcdc">
  <p style="margin: 0px">
    <span style="color: #569cd6">public</span> <span style="color: #569cd6">class</span> <span style="color: #4ec9b0">BasicAuthMessageHandler</span> : <span style="color: #4ec9b0">DelegatingHandler</span>
  </p>
  
  <p style="margin: 0px">
    {
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: #569cd6">protected</span> <span style="color: #569cd6">override</span> <span style="color: #4ec9b0">Task</span><span style="color: #b4b4b4"><</span><span style="color: #4ec9b0">HttpResponseMessage</span><span style="color: #b4b4b4">></span> <span style="color: white">SendAsync</span>(<span style="color: #4ec9b0">HttpRequestMessage</span> <span style="color: white">request</span>,
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #4ec9b0">CancellationToken</span> <span style="color: white">cancellationToken</span>)
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; {
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #569cd6">var</span> <span style="color: white">headers</span> <span style="color: #b4b4b4">=</span> <span style="color: white">request</span><span style="color: #b4b4b4">.</span><span style="color: white">Headers</span>;
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #569cd6">if</span> (<span style="color: white">headers</span><span style="color: #b4b4b4">.</span><span style="color: white">Authorization</span> <span style="color: #b4b4b4">!=</span> <span style="color: #569cd6">null</span> <span style="color: #b4b4b4">&&</span> <span style="color: white">headers</span><span style="color: #b4b4b4">.</span><span style="color: white">Authorization</span><span style="color: #b4b4b4">.</span><span style="color: white">Scheme</span> <span style="color: #b4b4b4">==</span> <span style="color: #d69d85">"Basic"</span>)
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; {
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #569cd6">var</span> <span style="color: white">userPwd</span> <span style="color: #b4b4b4">=</span> <span style="color: #4ec9b0">Encoding</span><span style="color: #b4b4b4">.</span><span style="color: white">UTF8</span><span style="color: #b4b4b4">.</span><span style="color: white">GetString</span>(<span style="color: #4ec9b0">Convert</span><span style="color: #b4b4b4">.</span><span style="color: white">FromBase64String</span>(<span style="color: white">headers</span><span style="color: #b4b4b4">.</span><span style="color: white">Authorization</span><span style="color: #b4b4b4">.</span><span style="color: white">Parameter</span>));
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #569cd6">var</span> <span style="color: white">user</span> <span style="color: #b4b4b4">=</span> <span style="color: white">userPwd</span><span style="color: #b4b4b4">.</span><span style="color: white">Substring</span>(<span style="color: #b5cea8"></span>, <span style="color: wh
ite">userPwd</span><span style="color: #b4b4b4">.</span><span style="color: white">IndexOf</span>(<span style="color: #d69d85">&#8216;:&#8217;</span>));
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #569cd6">var</span> <span style="color: white">password</span> <span style="color: #b4b4b4">=</span> <span style="color: white">userPwd</span><span style="color: #b4b4b4">.</span><span style="color: white">Substring</span>(<span style="color: white">userPwd</span><span style="color: #b4b4b4">.</span><span style="color: white">IndexOf</span>(<span style="color: #d69d85">&#8216;:&#8217;</span>) <span style="color: #b4b4b4">+</span> <span style="color: #b5cea8">1</span>);
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #608b4e">// Validamos user y password (aquí asumimos que siempre son ok)</span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #569cd6">var</span> <span style="color: white">principal</span> <span style="color: #b4b4b4">=</span> <span style="color: #569cd6">new</span> <span style="color: #4ec9b0">GenericPrincipal</span>(<span style="color: #569cd6">new</span> <span style="color: #4ec9b0">GenericIdentity</span>(<span style="color: white">user</span>), <span style="color: #569cd6">null</span>);
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: white">PutPrincipal</span>(<span style="color: white">principal</span>);
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; }
  </p>
  
  <p style="margin: 0px">
    &#160;
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #569cd6">return</span> <span style="color: #569cd6">base</span><span style="color: #b4b4b4">.</span><span style="color: white">SendAsync</span>(<span style="color: white">request</span>, <span style="color: white">cancellationToken</span>);
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; }
  </p>
  
  <p style="margin: 0px">
    &#160;
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: #569cd6">private</span> <span style="color: #569cd6">void</span> <span style="color: white">PutPrincipal</span>(<span style="color: #b8d7a3">IPrincipal</span> <span style="color: white">principal</span>)
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; {
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #4ec9b0">Thread</span><span style="color: #b4b4b4">.</span><span style="color: white">CurrentPrincipal</span> <span style="color: #b4b4b4">=</span> <span style="color: white">principal</span>;
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #569cd6">if</span> (<span style="color: #4ec9b0">HttpContext</span><span style="color: #b4b4b4">.</span><span style="color: white">Current</span> <span style="color: #b4b4b4">!=</span> <span style="color: #569cd6">null</span>)
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; {
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #4ec9b0">HttpContext</span><span style="color: #b4b4b4">.</span><span style="color: white">Current</span><span style="color: #b4b4b4">.</span><span style="color: white">User</span> <span style="color: #b4b4b4">=</span> <span style="color: white">principal</span>;
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; }
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; }
  </p>
  
  <p style="margin: 0px">
    }
  </p></p>
</div>

Este Message Handler crea y coloca un GenericPrincipal si la cabecera Authorize está presente. Luego en el controlador debemos usar [Authorize] para aquellas acciones que requieran de usuario autenticado.

Acuerdate de registrar el Message Handler desde Application_Start:

<div style="font-size: 10pt; font-family: consolas; background: #1e1e1e; color: #dcdcdc">
  <p style="margin: 0px">
    <span style="color: white">config</span><span style="color: #b4b4b4">.</span><span style="color: white">MessageHandlers</span><span style="color: #b4b4b4">.</span><span style="color: white">Add</span>(<span style="color: #569cd6">new</span> <span style="color: #4ec9b0">BasicAuthMessageHandler</span>());
  </p></p>
</div>

Por lo tanto estamos usando una combinación entre un Message Handler y un filtro de autorización (Authorize).

**Usando un módulo de autenticación**

Bien… exploremos esta otra alternativa. Ahora vamos a usar un módulo de autenticación (un HttpModule) para realizar las mismas tareas que realiza el Message Handler, es decir para crear un IPrincipal.

¿Y qué diferencias hay entre usar un HttpModule o un Message Handler? Pues buena pregunta, básicamente las siguientes:

  1. Un HttpModule es algo específico de IIS. No lo puedes usar en aplicaciones WebApi que no estén hospedadas en IIS. Por otro lado un Message Handler es algo propio de WebApi. 
  2. Un HttpModule ve todas las peticiones que estén dirigidas al pipeline de ASP.NET. Sean peticiones de WebApi o no. Un Message Handler tan solo ve las peticiones WebApi. 
  3. Un HttpModule se ejecuta antes en el pipeline que un Message Handler. De hecho se ejecuta antes que cualquier parte de WebApi. 

Por norma general, **es preferible usar un HttpModule si sabes que vas a hospedar tu WebApi siempre en un IIS**. Si no, si quieres tener la posibilidad de hospedar tu WebApi en otros procesos entonces usa un Message Handler.

¿Listo para nuestro módulo de autenticación? Aquí lo tienes:

<div style="font-size: 10pt; font-family: consolas; background: #1e1e1e; color: #dcdcdc">
  <p style="margin: 0px">
    <span style="color: #569cd6">public</span> <span style="color: #569cd6">class</span> <span style="color: #4ec9b0">BasicAuthModule</span> : <span style="color: #b8d7a3">IHttpModule</span>
  </p>
  
  <p style="margin: 0px">
    {
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160; <span style="color: #569cd6">public</span> <span style="color: #569cd6">void</span> <span style="color: white">Init</span>(<span style="color: #4ec9b0">HttpApplication</span> <span style="color: white">context</span>)
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160; {
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: white">context</span><span style="color: #b4b4b4">.</span><span style="color: white">AuthenticateRequest</span> <span style="color: #b4b4b4">+=</span> <span style="color: white">OnAuthenticateRequest</span>;
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160; }
  </p>
  
  <p style="margin: 0px">
    &#160;
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160; <span style="color: #569cd6">private</span> <span style="color: #569cd6">void</span> <span style="color: white">OnAuthenticateRequest</span>(<span style="color: #569cd6">object</span> <span style="color: white">sender</span>, <span style="color: #4ec9b0">EventArgs</span> <span style="color: white">e</span>)
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160; {
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #569cd6">var</span> <span style="color: white">application</span> <span style="color: #b4b4b4">=</span> (<span style="color: #4ec9b0">HttpApplication</span>)<span style="color: white">sender</span>;
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #569cd6">var</span> <span style="color: white">request</span> <span style="color: #b4b4b4">=</span> <span style="color: #569cd6">new</span> <span style="color: #4ec9b0">HttpRequestWrapper</span>(<span style="color: white">application</span><span style="color: #b4b4b4">.</span><span style="color: white">Request</span>);
  </p>
  
  <p style="margin: 0px">
    &#160;
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #569cd6">var</span> <span style="color: white">headers</span> <span style="color
: #b4b4b4">=</span> <span style="color: white">request</span><span style="color: #b4b4b4">.</span><span style="color: white">Headers</span>;
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #569cd6">var</span> <span style="color: white">authData</span> <span style="color: #b4b4b4">=</span> <span style="color: white">headers</span>[<span style="color: #d69d85">"Authorization"</span>];
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #569cd6">if</span> (<span style="color: #b4b4b4">!</span><span style="color: #569cd6">string</span><span style="color: #b4b4b4">.</span><span style="color: white">IsNullOrEmpty</span>(<span style="color: white">authData</span>))
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; {
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #569cd6">var</span> <span style="color: white">scheme</span> <span style="color: #b4b4b4">=</span> <span style="color: white">authData</span><span style="color: #b4b4b4">.</span><span style="color: white">Substring</span>(<span style="color: #b5cea8"></span>, <span style="color: white">authData</span><span style="color: #b4b4b4">.</span><span style="color: white">IndexOf</span>(<span style="color: #d69d85">&#8216; &#8216;</span>));
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #569cd6">var</span> <span style="color: white">parameter</span> <span style="color: #b4b4b4">=</span> <span style="color: white">authData</span><span style="color: #b4b4b4">.</span><span style="color: white">Substring</span>(<span style="color: white">scheme</span><span style="color: #b4b4b4">.</span><span style="color: white">Length</span>)<span style="color: #b4b4b4">.</span><span style="color: white">Trim</span>();
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #569cd6">var</span> <span style="color: white">userPwd</span> <span style="color: #b4b4b4">=</span> <span style="color: #4ec9b0">Encoding</span><span style="color: #b4b4b4">.</span><span style="color: white">UTF8</span><span style="color: #b4b4b4">.</span><span style="color: white">GetString</span>(<span style="color: #4ec9b0">Convert</span><span style="color: #b4b4b4">.</span><span style="color: white">FromBase64String</span>(<span style="color: white">parameter</span>));
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #569cd6">var</span> <span style="color: white">user</span> <span style="color: #b4b4b4">=</span> <span style="color: white">userPwd</span><span style="color: #b4b4b4">.</span><span style="color: white">Substring</span>(<span style="color: #b5cea8"></span>, <span style="color: white">userPwd</span><span style="color: #b4b4b4">.</span><span style="color: white">IndexOf</span>(<span style="color: #d69d85">&#8216;:&#8217;</span>));
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #569cd6">var</span> <span style="color: white">password</span> <span style="color: #b4b4b4">=</span> <span style="color: white">userPwd</span><span style="color: #b4b4b4">.</span><span style="color: white">Substring</span>(<span style="color: white">userPwd</span><span style="color: #b4b4b4">.</span><span style="color: white">IndexOf</span>(<span style="color: #d69d85">&#8216;:&#8217;</span>) <span style="color: #b4b4b4">+</span> <span style="color: #b5cea8">1</span>);
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #608b4e">// Validamos user y password (aquí asumimos que siempre son ok)</span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #569cd6">var</span> <span style="color: white">principal</span> <span style="color: #b4b4b4">=</span> <span style="color: #569cd6">new</span> <span style="color: #4ec9b0">GenericPrincipal</span>(<span style="color: #569cd6">new</span> <span style="color: #4ec9b0">GenericIdentity</span>(<span style="color: white">user</span>), <span style="color: #569cd6">null</span>);
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: white">PutPrincipal</span>(<span style="color: white">principal</span>);
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; }
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160; }
  </p>
  
  <p style="margin: 0px">
    &#160;
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160; <span style="color: #569cd6">public</span> <span style="color: #569cd6">void</span> <span style="color: white">Dispose</span>()
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160; {
  </p>
  
  <p style="margin: 0px">
    &#160;
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160; }
  </p>
  
  <p style="margin: 0px">
    &#160;
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160; <span style="color: #569cd6">private</span> <span style="color: #569cd6">void</span> <span style="color: white">PutPrincipal</span>(<span style="color: #b8d7a3">IPrincipal</span> <span style="color: white">principal</span>)
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160; {
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #4ec9b0">Thread</span><span style="color: #b4b4b4">.</span><span style="color: white">CurrentPrincipal</span> <span style="color: #b4b4b4">=</span> <span style="color: white">principal</span>;
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #569cd6">if</span> (<span style="color: #4ec9b0">HttpContext</span><span style="color: #b4b4b4">.</span><span style="color: white">Current</span> <span style="color: #b4b4b4">!=</span> <span style="color: #569cd6">null</span>)
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; {
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #4ec9b0">HttpContext</span><span style="color: #b4b4b4">.</span><span style="color: white">Current</span><span style="color: #b4b4b4">.</span><span style="color: white">User</span> <span style="color: #b4b4b4">=</span> <span style="color: white">principal</span>;
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; }
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160; }
  </p>
  
  <p style="margin: 0px">
    }
  </p></p>
</div>

Puedes ver como el código es muy parecido al del Message Handler. Ahora debemos registrarlo. Los HttpModules se registran en el web.config, dentro de la sección modules de system.webServer:

<div style="font-size: 10pt; font-family: consolas; background: #1e1e1e; color: #dcdcdc">
  <p style="margin: 0px">
    <span style="color: gray"><</span><span style="color: #569cd6">system.webServer</span><span style="color: gray">></span>
  </p>
  
  <p style="margin: 0px">
    <p style="margin: 0px">
      <span style="color: gray"></span>
    </p>
    
    <p>
      <span style="color: gray">&#160; <</span><span style="color: #569cd6">modules</span><span style="color: gray">></span>
    </p>
    
    <p style="margin: 0px">
      <span style="color: gray">&#160;&#160;&#160; <</span><span style="color: #569cd6">add</span><span style="color: gray"> </span><span style="color: #92caf4">name</span><span style="color: gray">="</span><span style="color: #c8c8c8">BasicAuthHttpModule</span><span style="color: gray">" </span>
    </p>
    
    <p style="margin: 0px">
      <span style="color: gray">&#160;&#160;&#160;&#160;&#160; </span><span style="color: #92caf4">type</span><span style="color: gray">="</span><span style="color: #c8c8c8">MvcSecureDemo.BasicAuthModule, MvcSecureDemo</span><span style="color: gray">"/></span>
    </p>
    
    <p style="margin: 0px">
      <span style="color: gray">&#160; </</span><span style="color: #569cd6">modules</span><span style="color: gray">></span>
    </p>
    
    <p s
tyle="margin: 0px">
      <span style="color: gray"></</span><span style="color: #569cd6">system.webServer</span><span style="color: gray">></span>
    </p>
  </p>
</div>

La situación es la misma que teníamos en el caso del Message Handler: en los controladores debemos usar [Authorize] para proteger las acciones que requieran usuarios autenticados.

**Más allá de autenticación básica**

Vale… hemos explorado como securizar nuestros servicios WebApi (a través de filtros de autorización, Message Handlers y HttpModules). Lo hemos hecho a partir de la autenticación básica de HTTP porque es muy sencilla y así el código no queda liado. 😉

Pero autenticación básica NO es un buen mecanismo. No lo es, a menos que uses HTTPS, ya que el login y el password viajan en texto plano (recuerda que Base64 no es cifrado).

Bien, ¿qué debéríamos hacer si no tenemos HTTPS? Una solución pasa por encriptar realmente el login y el password. Esto sigue teniendo un riesgo, tan pequeño como seguro sea nuestro algoritmo de encriptación y segura esté la clave que usemos).

Exploremos otra alternativa, otra que signifique que ni el login ni el password viajan por la red. Nadie con un sniffer será capaz de saber nuestra password escuchando los mensajes que nos intercambiamos con el servidor… Veamos que deberíamos hacer.

Partamos de la suposición de que tanto el cliente (una aplicación móvil p. ej.) y el servidor comparten un código, llamésmole código de acceso. Da igual como lo han compartido (p. ej. a través de un email). Lo importante es que lo tienen. 

Entonces básicamente si el cliente adjunta este código en una cabecera (sigamos suponiendo la misma cabecera Authorization) la llamada se considera autenticada y en caso contrario se considera válida. Este código (insisto: obtenido previamente) identifica al cliente (la aplicación móvil) y al usuario de dicho cliente, por lo que adjuntando este código una aplicación puede hacer peticiones _en nombre del usuario._

Vale, no parece que hayamos progresado mucho verdad: alguien con un sniffer pilla nuestra petición y ya tiene el código. Muy seguro el sistema no parece.

Bueno… sigamos suponiendo. Sigamos suponiendo que además de este código, tanto cliente como servidor comparten otro código. Un código secreto. Lo de secreto viene porque este segundo código jamás viajará por la red. Nunca. Lo comparten al inicio de los tiempos (p. ej. por mail) y luego el cliente se lo guarda y el servidor también.

Ahora lo que debe hacer el cliente es mandar la petición pero **usará el código secreto para generar un hash del código de acceso**. Y la cabecera Authorization contendrá este hash. Cuando el servidor recibe una petición del cliente debe:

  * Calcular el hash del código de acceso del cliente usando el código secreto (que ambos comparten)
  * Si el resultado es el mismo que ha enviado el cliente, la petición se considera autenticada.

Vale… hay un problemilla ahora. El código secreto debe ser distinto por cada cliente, pero si tan solo recibimos en la cabecera Authorization el hash del código de acceso… ¿como sabemos qué código secreto debemos usar para calcular el hash?. Es decir, como sabemos de qué cliente es la petición.

Pues nada. Hacemos que cliente y servidor comparten otro elemento: un identificador de cliente. Ahora el cliente en la cabecera Authorization mandará su código identificador y el hash de su código de acceso (calculado con su código secreto). Cuando el servidor recibe la petición, sabe de que cliente es (por el identificador de cliente de la cabecera Authorization) y podrá usar el código secreto de este cliente para calcular el hash del código de acceso de este cliente y validar que sea el mismo que el clienté envía.

¿Te parece seguro el sistema ahora? No lo es en absoluto. Alguien que pille una petición podrá hacer lo que quiera, porque tendrá el hash del código de acceso. Estamos igual que al principio… pero a un paso de la solución.

El problema de que si nos pillan una petición luego ya nos puedan suplantar siempre es porque estamos calculando un hash de algo que jamás se modifica: el código de acceso. Porque en lugar de calcular un hash de dicho código no calculamos un hash de una cadena que esté compuesta de:

  1. La URL de destino
  2. Los datos en query string/formdata
  3. El código de acceso

Los datos 1 y2 son variables (distintos por cada petición que hagamos). Así cada petición tendrá un hash distinto. Cuando el servidor reciba la petición calculará esta cadena con la URL, la query string/formdata y el código de acceso del cliente y usará el código secreto del cliente para calcular el hash. Si coinciden la petición queda autenticada.

Ahora si alguien nos pilla una petición con un sniffer, tan solo podrá hacer una cosa: enviarla otra vez al servidor. No podrá modificarla, porque si lo hace (p. ej. modifica la querystring para cambiar un parámetro) automáticamente el hash pasa a ser inválido. Y no puede construir un hash nuevo porque no tiene acceso al código secreto.

Así pues un hacker nos puede pillar peticiones y puede ver su contenido. Hasta puede guardarlas y reenviarlas al servidor más adelante. Pero NO puede crear peticiones falsas en nuestro nombre.

Vayamos un paso más allá. ¿Por qué no adjuntamos el tiempo en que se realiza la petición? Es decir el cliente añade en una cabecera la fecha y hora en que realiza la petición. Y usa este dato también para calcular el hash. Ahora el servidor puede validar que la fecha/hora que envía el cliente sea (aproximadamente) la actual. Si el servidor recibe una petición de un cliente con una fecha/hora de hace 4 minutos, la puede aceptar pero si es de hace 1 hora la puede rechazar, ya que seguramente es una petición interceptada, guardada y mandada luego. Dado que la fecha/hora se usa también para calcular el hash, el hacker que nos intercepte la petición no puede falsear este dato.

Incluso podríamos hacer algo más: añadir un número (llamésmole nonce) de secuencia. La idea es que dos peticiones del mismo cliente jamás pueden repetir el nonce. Por supuesto el nonce se manda en otra cabecera y también se usa para calcular el hash. Si ahora un hacker nos intercepta la petición y la intenta mandar al cabo de 3 minutos… será rechazada porque este nonce ya ha sido usado.

Ahora estamos protegidos de todos los ataques, excepto de un Man-in-the-middle. Ojo, no tenemos privacidad (tanto las peticiones como las respuestas van sobre HTTP en texto llano), pero nuestras credenciales están seguras y nadie puede crear peticiones en nuestro nombre ni guardarse peticiones válidas y enviarlas más adelante. Si requerimos privacidad y protección contra ataques tipo Man-in-the-middle entonces lo más rápido es usar HTTPS.

Por supuesto este sistema que he descrito, no me lo he inventado yo. Este sistema lo inventó <a href="http://en.wikipedia.org/wiki/Blaine_Cook_(programmer)" target="_blank" rel="noopener noreferrer">Blain Cook</a> y le dio el nombre de <a href="http://oauth.net/" target="_blank" rel="noopener noreferrer">oAuth</a>. Lo que he descrito aquí es (rápido y mal, por supuesto) el flujo básico de oAuth 1.0.

Las ventajas de oAuth son que en ningún momento las credenciales del usuario circulan por la red y que ofrece un buen balance de seguridad siempre y cuando el código secreto no se vea comprometido. Es pues una muy buena opción para proteger tus servicios WebApi.

Y por supuesto, usar otros mecanismos de autenticación no modifica en nada lo dicho en este post, tan solo “complica” el código que has de poner en tu filtro de autorización, message handler o HttpModule.

Un saludo!

 [1]: http://www.microsoft.com/en-us/download/details.aspx?id=36476 "http://www.microsoft.com/en-us/download/details.aspx?id=36476"
 [2]: http://www.motobit.com/util/base64-decoder-encoder.asp "http://www.motobit.com/util/base64-decoder-encoder.asp"
 [3]: http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_782B212D.png