---
title: Integra oAuth en tus “viejas” aplicaciones ASP.NET MVC
description: Integra oAuth en tus “viejas” aplicaciones ASP.NET MVC
author: eiximenis

date: 2012-11-14T09:20:05+00:00
geeks_url: /?p=1615
geeks_visits:
  - 1852
geeks_ms_views:
  - 1021
categories:
  - Uncategorized

---
En el <a href="http://geeks.ms/blogs/etomas/archive/2012/11/13/integra-oauth-en-tu-aplicaci-243-n-asp-net-mvc.aspx" target="_blank" rel="noopener noreferrer">post anterior</a> vimos como integrar pseudo-autenticación basada en oAuth en aplicaciones MVC4, usando el paquete Microsoft WebPages OAuth Library. Pero… qué ocurre con versiones anteriores? O bien si no queréis usar este paquete? Es muy difícil integrar pseudo-autenticación basada en oAuth entonces?

Hace algún tiempo tuve precisamente esta necesidad y estuve buscando librerías que me permitiesen integrar oAuth en mi aplicación de una forma cómoda. Como todo el mundo terminé por encontrar <a href="http://www.dotnetopenauth.net/" target="_blank" rel="noopener noreferrer">DotNetOpenAuth</a> pero al final no la usé. Al final me decidí por investigar como funcionaba el protocolo de oAuth e implementarme una librería propia, que se adaptase un poco más a mis necesidades.

En este post os voy a mostrar como usar esta librería que he realizado y que está en codeplex: [http://epnukeoauth.codeplex.com/][1] Debo decir que aunque hay el código fuente en codeplex, no hay release binario porque no la considero “terminada”, aunque sí usable y no sé cuando la tendré “terminada” del todo. La intención es que termine siendo un paquete de NuGet.

Así que por el momento tan solo os queda descargaros el código fuente (solución VS2010) y compilarla.

A día de hoy la solución consta de cuatro proyectos:

  1. Epnuke.OAuth: Librería base de oAuth 
  2. Epnuke.OAuth.Mvc: Extensiones a la librería pensadas para MVC 
  3. Epnuke.OAuth.Tests: Tests unitarios 
  4. Epnuke.OAuth.Demo.SignWithTwitter: Demostración de pseudo-autenticación basada en oAuth. 

Es este útimo proyecto el que discutiremos en este post 😉

A diferencia de la clase OAuthWebSecurity que vimos ayer y que está atada a los diversos proveesdores oAuth (a través de los métodos RegisterXXXClient), esta librería es totalmente genérica (de hecho no solo permite pseudo-autenticación, también permite llamadas arbitrarias de oAuth, pero esto queda fuera de este post).

Eso significa que usarla no es tan sencillo como la solución de Microsoft, pero ya veréis que tampoco es mucho más complicado.

El primer paso (una vez bajada la librería) es referenciar el ensamblado _Epnuke.OAuth.dll_ y crear una aplicación MVC (puede ser MVC3).

Para iniciar el flujo de pseudo-autenticación de oAuth nos creamos una acción en un controlador (yo he usado para variar AccountController) y he llamado a dicha acción LogOnTwitter. Esta acción es la que iniciará todo el proceso de pseudo-autenticación. Para hacerlo:

  1. Debe crearse una nueva sesión de oAuth. Esta sesión es la que nos encapsula todo el proceso de oAuth. 
  2. Llamar al método RequestTemporaryCredentials. Este método hace una llamada para obtener un primer token oAuth. Este token oAuth es temporal y tan solo se usa para poder redirigir al usuario a la página de login del proveedor 
  3. Redirigir al usuario a dicha página (en este caso la página de login de twitter). 

El código sería tal y como sigue:

<div style="font-size: 10pt; font-family: consolas; background: white; color: black">
  <p style="margin: 0px">
    <span style="color: blue">public</span> <span style="color: #2b91af">ActionResult</span> LogOnTwitter()
  </p>
  
  <p style="margin: 0px">
    {
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: blue">var</span> oauthSession = <span style="color: blue">new</span> <span style="color: #2b91af">OAuthClientSession</span>(<span style="color: #2b91af">ConfigurationManager</span>.AppSettings[<span style="color: #a31515">"consumer-key"</span>],
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #2b91af">ConfigurationManager</span>.AppSettings[<span style="color: #a31515">"consumer-secret"</span>],
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: blue">new</span> <span style="color: #2b91af">NonceGenerator32Bytes</span>());
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: blue">var</span> uri = <span style="color: #2b91af">ConfigurationManager</span>.AppSettings[<span style="color: #a31515">"request_token"</span>];
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; oauthSession.RequestTemporaryCredentials(uri, <span style="color: #a31515">"POST"</span>, <span style="color: #a31515">"http://127.0.0.1:64983/Account/Callback"</span>);
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; Session[<span style="color: #a31515">"oauth"</span>] = oauthSession;
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: blue">return</span> Redirect(oauthSession.GetAuthorizationUri(<span style="color: #2b91af">ConfigurationManager</span>.AppSettings[<span style="color: #a31515">"authorize"</span>]));
  </p>
  
  <p style="margin: 0px">
    }
  </p></p>
</div>

La primera línea crea la sesión de OAuth. Los tres parámetros necesarios son el consumer key, el consumer secret (esos mismos son&#160; los dos que usábamos en el método RegisterTwitterClient del post anterior) y el generador de _nonces_. Un nonce es un valor “único” que se genera de forma aleatória para cada petición oAuth. El proveedor _puede_ (puede, no está obligado a) validar que no haya dos peticiones distintas con&#160; el mismo nonce durante un cierto periodo de tiempo. Es un mecanismo de seguridad. La especificación de oAuth es totalmente ambigua en cuanto al formato de _nonce_ usado. Se supone que cada proveedor puede usar el formato de nonce que prefiera. Aquí uso el generador de nonces de 32 bytes que es el formato que usa twitter para el nonce (aunque desconozco si acepta otros).

Con esto tenemos una sesión _lista_ para empezar a hacer peticiones oAuth. El siguiente método que llamamos es RequestTemporaryCredentials. A este método se le pasan los siguientes parámetros:

  1. La url del proveedor de oAuth que expone el endpoint de request_token (en el caso de twitter se llama <a href="https://dev.twitter.com/docs/api/1/post/oauth/request_token" target="_blank" rel="noopener noreferrer">oauth/request_token</a>). 
  2. El método http a usar (en el caso de twitter es POST). 
  3. Y la dirección de callback a la que twitter nos deberá redireccionar una vez el proceso de logon esté completado. 

El método _RequestTemporaryCredentials_ realiza la petición y obtiene las credenciales temporales de oAuth pero **no hace nada más**. A diferencia del método RequestAuthentication de OAuthWebSecurity que vimos ayer, aquí la redirección a la página de login del proveedor la debemos hacer nosotros.

Luego nos guardamos la sesión oAuth en la sesión de la aplicación web porque la necesitaremos luego (la librería ofrece alternativas a este punto en caso de no querer/poder usar la sesión). Y finalmente redirigimos el usuario a la página de login del proveedor (endpoint authorize de oAuth). En el caso de twitter la página de login que se usa para oAuth es <a href="https://dev.twitter.com/docs/api/1/get/oauth/authenticate" target="_blank" rel="noopener noreferrer">oauth/authenticate</a>. Pero NO basta con redirigir a dicha página: le hemos de pasar via querystring el token temporal que hemos obtenido previamente. Y eso es lo que hace el método GetAuthorizationUri de la clase oauthSession.

En este punto redirigimos el usuario a la página de login de twitter, donde el usuario podrá entrar sus credenciales. Y una vez entradas dichas credenciales el usuario será redirigido a la URL que pasamos como parámetro al método RequestTemporaryCredentials. En nuestro caso era /Account/Callback.

Dicho método recibe por querystring el código de verificación de twitter (parámetro oauth_verifier). Dicho código significa algo
  
como “hey! el usuario se ha autenticado correctamente”. Pero este código de verificación tan solo sirve para una cosa: para obtener los tokens oAuth definitivos y que nos permitirán hacer llamadas oAuth en nombre de este usuario.

Veamos pues el código de la acción Callback:

<div style="font-size: 10pt; font-family: consolas; background: white; color: black">
  <p style="margin: 0px">
    <span style="color: blue">public</span> <span style="color: #2b91af">ActionResult</span> Callback(<span style="color: blue">string</span> oauth_verifier)
  </p>
  
  <p style="margin: 0px">
    {
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: blue">var</span> oauthSession = Session[<span style="color: #a31515">"oauth"</span>] <span style="color: blue">as</span> <span style="color: #2b91af">OAuthClientSession</span>;
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; oauthSession.Authorize(oauth_verifier, <span style="color: #2b91af">ConfigurationManager</span>.AppSettings[<span style="color: #a31515">"access_token"</span>]);
  </p>
  
  <p style="margin: 0px">
    &#160;
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: blue">if</span> (oauthSession.IsAuthorized)
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; {
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: blue">var</span> name = oauthSession.GetAdditionalData(<span style="color: #a31515">"screen_name"</span>);
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #2b91af">FormsAuthentication</span>.SetAuthCookie(name, <span style="color: blue">false</span>);
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: blue">return</span> RedirectToAction(<span style="color: #a31515">"Index"</span>, <span style="color: #a31515">"Private"</span>);
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; }
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: blue">else</span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; {
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: blue">return</span> RedirectToAction(<span style="color: #a31515">"Index"</span>, <span style="color: #a31515">"Home"</span>);
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; }
  </p>
  
  <p style="margin: 0px">
    &#160;
  </p>
  
  <p style="margin: 0px">
    }
  </p></p>
</div>

En la primera línea recuperamos la sesión oAuth que teníamos guardada y luego llamamos al método Authorize. Dicho método espera los siguientes parámetros:

  1. El códgo de verificación enviado por el proveedor
  2. La URL que expone el endpoint access_token de oAuth. En el caso de twitter dicha URL es <a href="https://dev.twitter.com/docs/api/1/post/oauth/access_token" target="_blank" rel="noopener noreferrer">oauth/access_token</a>.

Una vez se haya llamado al método Authorize, se puede usar la propiedad IsAuthorized del objeto OAuthClientSession para saber si el usuario está autorizado o ha habido algún error.

De nuevo, a diferencia de la clase OAuthWebSecurity que vimos en el post anterior, la libería se limita a decirnos si el usuario está autenticado o no. Pero no nos dice su ID o su login. Para obtenerlo ya dependemos del proveedor de oAuth que usemos. Hay dos opciones:

  1. Si el proveedor de oAuth nos ha mandado en la respuesta de acces_token el login del usuario o el ID podremos recogerlo si sabemos el nombre del campo que usa.
  2. Si el proveedor de oAuth no nos la ha mandado deberemos hacer una llamada oAuth a la API del proveedor.

Twitter nos manda el login del usuario en el parámetro screen\_name, así que usamos el método GetAdditionalData de la clase OAuthClientSession para obtenerlo (si preferís el ID lo teneis en el parámetro user\_id). Y una vez tenemos el login del usuario lo autenticamos en nuestro sistema con FormsAuthentication.

En este punto ya tenemos el usuario autenticado en ASP.NET. No solo esto, también tenemos en el objeto OAuthClientSession los tokens oAuth que podemos usar para hacer llamadas a la API de twitter _como si fuesemos este usuario_… pero esto lo dejamos para otro post 😉

Espero que os haya resultado interesante! 

Un saludo!

Os dejo como referencia el enlace al <a href="https://dev.twitter.com/docs/auth/implementing-sign-twitter" target="_blank" rel="noopener noreferrer">proceso entero de pseudo-autenticación basada en oAuth usando twitter</a>.

 [1]: http://epnukeoauth.codeplex.com/ "http://epnukeoauth.codeplex.com/"