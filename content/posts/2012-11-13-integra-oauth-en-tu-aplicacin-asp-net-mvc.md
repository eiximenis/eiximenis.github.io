---
title: Integra oAuth en tu aplicación ASP.NET MVC

author: eiximenis

date: 2012-11-13T16:39:44+00:00
geeks_url: /?p=1614
geeks_visits:
  - 3402
geeks_ms_views:
  - 1456
categories:
  - Uncategorized

---
¡Buenas! Una de las novedades de ASP.NET MVC4 es la facilidad para incorporar pseudo-autenticación basada en oAuth. Si seleccionamos la plantilla de proyecto de “internet application” esta ya viene configurada por defecto y usarla es de lo más sencillo.

Veamos ahora como añadirla en el caso de que estemos usando otra plantilla de proyecto. En este caso usaremos la plantilla de proyecto “Empty” que en el caso de MVC4 significa vacío de verdad, ya que no tendremos apenas nada en nuestro proyecto.

El módulo que se encarga de proporcionarnos pseudo-autenticación oAuth en MVC4 es un paquete de NuGet llamado <a href="http://nuget.org/packages/Microsoft.AspNet.WebPages.OAuth" target="_blank" rel="noopener noreferrer">Microsoft WebPages OAuth Library</a>, así que el primer paso será añadirlo a nuestro proyecto. Para ello abrimos la Package Manager Console y tecleamos _**Install-Package Microsoft.AspNet.WebPages.OAuth**_. Esto nos instalará el paquete y todas sus dependencias (entre las que están DotNetOpenAuth y algunos ensamblados propios de WebPages).

Como parte de instalación del paquete se nos modificará el web.config,&#160; básicamente con la configuración de DotNetOpenAuth.

Una&#160; vez tenemos el paquete instalado, ya podemos registrar proveedores de oAuth, es decir que sitios de oAuth vamos a poder usar para registrarnos en nuestra aplicación web. Para ello usaremos la clase <a href="http://msdn.microsoft.com/es-es/library/microsoft.web.webpages.oauth.oauthwebsecurity(v=vs.111).aspx" target="_blank" rel="noopener noreferrer">OAuthWebSecurity</a>. Esta es una clase estática que es la que usaremos para todo el flujo de pseudo-autenticación de oAuth. Lo bueno es que es extremadamente simple de usar 🙂

Para registrar un proveedor de oAuth usamos el método RegisterXXXClient donde XXX es el nombre del proveedor usado. Están soportados los más importantes como Facebook, Twitter y Linkedin p.ej. Para ello añadimos el siguiente código en el Application_Start:

<div style="font-size: 10pt; font-family: consolas; background: white; color: black">
  <p style="margin: 0px">
    <span style="color: #2b91af">OAuthWebSecurity</span>.RegisterTwitterClient(
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; consumerKey: <span style="color: #a31515">"dhkdshdhasjhdaskdhsk"</span>,
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; consumerSecret: <span style="color: #a31515">"eyuweywuieywuieywi"</span>);
  </p></p>
</div>

Por supuesto debeís usar los valores correctos de consumerKey y consumerSecret (esos valores son proporcionados por el proveedor de oAuth, en este caso twitter).

Ahora debemos crear el flujo de autenticación de nuestra aplicación. Va a ser muy sencillo:

  1. Le mostraremos una lista de los proveedores válidos que puede usar para autenticarse (en este caso tan solo twitter). 
  2. Cuando seleccione uno, empezará el flujo de oAuth: el usuario será redirigido hacia el proveedor (twitter) para que introduzca allí sus credenciales. 
  3. El proveedor nos hará hará una petición a nuestra dirección de callback y en esa dirección deberemos autenticar al usuario en nuestra aplicación web. 

Vamos a encapsularlo todo dentro de un controlador al que llamaremos AccountController. El primer paso es crear la vista que contenga la lista con los proveedores válidos a usar. Llamaremos Login a esta vista:

<div style="font-size: 10pt; font-family: consolas; background: white; color: black">
  <p style="margin: 0px">
    <span style="background: yellow">@</span><span style="color: blue">using</span> Microsoft.Web.WebPages.OAuth
  </p>
  
  <p style="margin: 0px">
    <span style="background: yellow">@{</span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; ViewBag.Title = <span style="color: #a31515">"Login"</span>;
  </p>
  
  <p style="margin: 0px">
    <span style="background: yellow">}</span>
  </p>
  
  <p style="margin: 0px">
    &#160;
  </p>
  
  <p style="margin: 0px">
    <span style="color: blue"><</span><span style="color: maroon">h2</span><span style="color: blue">></span>Login to BeerHunters<span style="color: blue"></</span><span style="color: maroon">h2</span><span style="color: blue">></span>
  </p>
  
  <p style="margin: 0px">
    &#160;
  </p>
  
  <p style="margin: 0px">
    &#160;
  </p>
  
  <p style="margin: 0px">
    <span style="background: yellow">@</span><span style="color: blue">using</span> (Html.BeginForm())
  </p>
  
  <p style="margin: 0px">
    {
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: blue">foreach</span> (<span style="color: blue">var</span> registerData <span style="color: blue">in</span> OAuthWebSecurity.RegisteredClientData)
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; {
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: blue"><</span><span style="color: maroon">input</span> <span style="color: red">name</span><span style="color: blue">="provider"</span> <span style="color: red">type</span><span style="color: blue">="submit"</span> <span style="color: red">value</span><span style="color: blue">="</span><span style="background: yellow">@</span>registerData.DisplayName<span style="color: blue">"</span> <span style="color: blue">/></span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; }
  </p>
  
  <p style="margin: 0px">
    }
  </p></p>
</div>

Esta vista utiliza la propiedad RegisteredClientData de OAuthWebSecurity para mostrar tantos botones como clientes hayamos registrado. En este caso es un poco inútil ya que tan solo hemos registrado un proveedor (twitter) pero bueno… nos sirve para ver como tratar varios proveedores. Al final termina haciendo un post con el nombre del cliente registrado (según el botón que se pulse). Es en el método de acción que trata dicho post donde debemos lanzar el flujo de oAuth.

Para lanzar el flujo de oAuth es suficiente con llamar al método _RequestAuthentication_ de OAuthWebSecurity. A este método le tenemos que pasar el nombre del proveedor usado:

<div style="font-size: 10pt; font-family: consolas; background: white; color: black">
  <p style="margin: 0px">
    [<span style="color: #2b91af">HttpPost</span>]
  </p>
  
  <p style="margin: 0px">
    <span style="color: blue">public</span> <span style="color: #2b91af">ActionResult</span> Login(<span style="color: blue">string</span> provider)
  </p>
  
  <p style="margin: 0px">
    {
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: #2b91af">OAuthWebSecurity</span>.RequestAuthentication(provider);
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: blue">return</span> <span style="color: blue">null</span>;
  </p>
  
  <p style="margin: 0px">
    }
  </p></p>
</div>

Podemos devolver null como ActionResult porque el resultado “escapa” a ASP.NET MVC: seremos redirigidos a twitter y allí entraremos las credenciales. Cuando las haya entrado twitter me mandará de vuelta (hacia la dirección que se le especifique, que por defecto es la misma de la cual venimos), pero en la querystring me dejará datos relativos a la autenticación:

[<img title="image" style="border-left-width: 0px; border-right-width: 0px; background-image: none; border-bottom-width: 0px; padding-top: 0px; padding-left: 0px; display: inline; padding-right: 0px; border-top-width: 0px" border="0" alt="image" src="http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_0AD0E8AC.png" width="484" height="150" />][1]

Si no queremos ser redirigidos a la URL de donde venimos (/Account/Login) podemos usar el método RequestAuthentication con dos parámetros, el segundo del cual es la URL a la cual seremos redirigidos:

<div style="font-size: 10pt; font-family: consolas; background: white; color: black">
  <p style="margin: 0px">
    [<span style="color: #2b9
1af">HttpPost</span>]
  </p>
  
  <p style="margin: 0px">
    <span style="color: blue">public</span> <span style="color: #2b91af">ActionResult</span> Login(<span style="color: blue">string</span> provider)
  </p>
  
  <p style="margin: 0px">
    {
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: #2b91af">OAuthWebSecurity</span>.RequestAuthentication(provider, Url.Action(<span style="color: #a31515">"ExternalLogin"</span>));
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: blue">return</span> <span style="color: blue">null</span>;
  </p>
  
  <p style="margin: 0px">
    }
  </p></p>
</div>

Ahora seremos redirigidos a la acción ExternalLogin una vez hayamos entrado las credenciales en twitter.

Finalmente tan sólo nos queda validar que todo ha ido bien. Para ello en la acción ExternalLogin miramos si la autenticación ha sido correcta, y si lo es autorizar al usuario en nuestra aplicación:

<div style="font-size: 10pt; font-family: consolas; background: white; color: black">
  <p style="margin: 0px">
    <span style="color: blue">public</span> <span style="color: #2b91af">ActionResult</span> ExternalLogin()
  </p>
  
  <p style="margin: 0px">
    {
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: blue">var</span> result = <span style="color: #2b91af">OAuthWebSecurity</span>.VerifyAuthentication();
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: blue">if</span> (result.IsSuccessful)
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; {
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #2b91af">FormsAuthentication</span>.SetAuthCookie(result.UserName, <span style="color: blue">false</span>);
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: blue">return</span> RedirectToAction(<span style="color: #a31515">"Private"</span>);
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; }
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: blue">return</span> View();
  </p>
  
  <p style="margin: 0px">
    }
  </p></p>
</div>

El método VerifyAuthentication hace su trabajo basándose en los datos que nos ha enviado el proveedor via la querystring. El objeto result contiene los datos relativos al intento de autenticación. Si en lugar del nombre del usuario queremos su ID (su id dentro de twitter en este caso) debemos usar result.ProviderUserID.

La acción “Private” es una acción privada, tan solo accesible a usuarios autenticados:

<div style="font-size: 10pt; font-family: consolas; background: white; color: black">
  <p style="margin: 0px">
    [<span style="color: #2b91af">Authorize</span>]
  </p>
  
  <p style="margin: 0px">
    <span style="color: blue">public</span> <span style="color: #2b91af">ActionResult</span> Private()
  </p>
  
  <p style="margin: 0px">
    {
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; ViewBag.UserName = User.Identity.Name;
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: blue">return</span> View();
  </p>
  
  <p style="margin: 0px">
    }
  </p></p>
</div>

Y listos! Con esto ya tenemos el usuario identificado en nuestra web a través de su cuenta de twitter. La verdad es que más simple y sencillo no podía ser 🙂

 [1]: http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_05627808.png