---
title: Lidiando con oAuth (1/n) – Pseudoautenticacion oAuth
author: eiximenis

date: 2012-06-01T11:04:46+00:00
geeks_url: /?p=1598
geeks_visits:
  - 3614
geeks_ms_views:
  - 838
categories:
  - Uncategorized

---
Muy buenas! Este post es el primero de una serie de “n” donde veremos como podemos lidiar un poco con oAuth 1.0a. Vamos a ver como implementar un cliente y lo más interesante un proveedor.

Para seguir esta serie de posts recomiendo la lectura del documento “Entendiendo oAuth” que he dejado en Slideshare ([http://www.slideshare.net/eduardtomas/entendiendo-o-auth][1]) donde se describe brevemente el procolo oAuth y los distintos flujos asociados a él.

Comentaros también que he dejado en codeplex una librería que os va a permtir crear de forma extremadamente fácil un proveedor de oAuth para ASP.NET y también un cliente. Podéis encontrar la librería en <https://epnukeoauth.codeplex.com/> (de momento está tan solo el código fuente, debéis compilarla vosotros mismos).

**Breve, brevísima, introducción a oAuth**

oAuth es un protocolo de autorización pensado especialmente para aplicaciones web, aunque su uso se ha extendido en aplicaciones móviles y de escritorio. Lo que permite es que una aplicación (cliente) pueda acceder a los recursos de un servidor (usualmente una API estilo REST) en nombre de un determinado usuario, sin que en ningún momento la aplicación cliente tenga acceso a las credenciales de dicho usuario. Para conseguir esto hay un juego a 3 bandas entre la aplicación cliente, el servidor y quien posee les credenciales del usuario (que sí, suele ser el servidor).

Pongamos twitter como ejemplo. Cuando instalas un cliente de twitter, este cliente debe conectarse a tu cuenta para poder leer tweets y enviar tweets en tu nombre. Para ello el cliente no te solicitará tus credenciales si no que en su lugar:

  1. La aplicación cliente hará una petición al proveedor de servicios pasándole su token de aplicación (que identifica a la aplicación cliente). Este es el inicio de todo, signfica que la aplicación “quiere empezar una negociación oAuth”. 
  2. Si el proveedor de servicios acepta la aplicación le mandará un token temporal. 
  3. La aplicación cliente abrirá una ventana de navegador hacia una URL de quien pueda autenticar el usuario. En este caso twitter. Y le pasará a twitter este token temporal, diciéndole “Hey! El proveedor de servicios me acepta como cliente (tengo este token que lo demuestra) pero necesito que el usuario se autentique).” 
  4. El usuario hará login en twitter con tus credenciales. Insisto: en twitter. En ningún momento la aplicación cliente recibe las credenciales. 
  5. Twitter validará las credenciales y si son correctas pedirá que confirmes que das acceso a la aplicación”XXX” a tu cuenta. Si confirmas, se genera un token, el cual es enviado (el mecanismo exacto no importa ahora) a la aplicación cliente. 
  6. La aplicación cliente, usa este token para hablar con el proveedor de servicios diciéndole: “Hey! Tengo este token que me han dado con el cual se supone que tengo acceso a los servicios”. En el caso de twitter el proveedor de servicios es el propio twitter. 
  7. El proveedor de servicios valida este token y si es correcto, le manda _otro_ token al cliente. Este otro token es el que sirve (durante un tiempo determinado) para hacer llamados a los servicios _en nombre del_ usuario que se autenticó. El resto de tokens pasan a ser inváldos. 
  8. El cliente usa este segundo token para hacer llamadas al proveedor de servicios. 

Este es el escenario más complejo de oAuth (el llamado 3-legged). En el fondo cada vez que hemos hablado de “token” hay realmente dos tokens (uno público que se manda junto con la petición y otro secreto que se usa para firmar digitalmente la petición).

El principal problema a la hora de crear un proveedor de servicios de oAuth es todo el tema de la firma digital, ya que se debe ser extremadamente cuidadoso en este punto. Probablemente el 90% de implementaciones fallidas de oAuth, fallan en este punto.

**Pseudo Autenticación oAuth**

Este sea probablemente uno de los usos más conocidos de oAuth, el de usarlo para “autenticar” un usuario (el famoso botón de sign-in with twitter p.ej.). Vamos a ver como podemos crear una web que tenga una sección privada a la que se pueda acceder a través de un botón de “sign with twitter”. Vamos a usar la librería que os comentaba antes para hacerlo.

La idea es acceder a una acción de un controlador que sea privada:

<div style="border-bottom: #000080 1px solid; border-left: #000080 1px solid; font-family: &#39;Courier New&#39;, courier, monospace; color: #000; font-size: 10pt; border-top: #000080 1px solid; border-right: #000080 1px solid">
  <div style="background: #fff; max-height: 300px; overflow: auto">
    <ol style="padding-bottom: 0px; margin: 0px; padding-left: 5px; padding-right: 0px; white-space: nowrap; background: #ffffff; padding-top: 0px">
      <li>
        <span style="color: #0000ff">public</span> <span style="color: #0000ff">class</span> <span style="color: #2b91af">PrivateController</span> : <span style="color: #2b91af">Controller</span>
      </li>
      <li style="background: #f3f3f3">
        {
      </li>
      <li>
        &#160;&#160;&#160; [<span style="color: #2b91af">Authorize</span>]
      </li>
      <li style="background: #f3f3f3">
        &#160;&#160;&#160; <span style="color: #0000ff">public</span> <span style="color: #2b91af">ActionResult</span> Index()
      </li>
      <li>
        &#160;&#160;&#160; {
      </li>
      <li style="background: #f3f3f3">
        &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #0000ff">return</span> View();
      </li>
      <li>
        &#160;&#160;&#160; }
      </li>
      <li style="background: #f3f3f3">
        }
      </li>
    </ol>
  </div></p>
</div>

Fijaos que la acción está marcada como privada de la forma estándard en ASP.NET MVC (usando [Authorize]). Si intentamos navegar a /Private/Index al no estar autenticados en ASP.NET MVC seremos redirigidos a la vista de login:

[<img style="background-image: none; border-right-width: 0px; padding-left: 0px; padding-right: 0px; display: inline; border-top-width: 0px; border-bottom-width: 0px; border-left-width: 0px; padding-top: 0px" title="image" border="0" alt="image" src="http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_382A5C49.png" width="244" height="162" />][2]

La vista muestra un formulario de login estándard (eso seria para entrar sin usar twitter, vamos a obviar este punto) y un enlace para entrar usando twitter. Si miramos el código fuente de la vista /Views/Account/LogOn.cshtml el enlace para usar twitter llama a la acción LogOnTwitter del controlador Account. Esta es la acción que empieza todo el flujo de oAuth. Veamos primero su código:

<div style="border-bottom: #000080 1px solid; border-left: #000080 1px solid; font-family: &#39;Courier New&#39;, courier, monospace; color: #000; font-size: 10pt; border-top: #000080 1px solid; border-right: #000080 1px solid">
  <div style="background: #fff; max-height: 300px; overflow: auto">
    <ol style="padding-bottom: 0px; margin: 0px; padding-left: 5px; padding-right: 0px; white-space: nowrap; background: #ffffff; padding-top: 0px">
      <li>
        <span style="color: #0000ff">public</span> <span style="color: #2b91af">ActionResult</span> LogOnTwitter()
      </li>
      <li style="background: #f3f3f3">
        {
      </li>
      <li>
        &#160;&#160;&#160; <span style="color: #0000ff">var</span> oauthSession = <span style="color: #0000ff">new</span> <span style="color: #2b91af">OAuthClientSession</span>(<span style="color: #2b91af">ConfigurationManager</span>.AppSettings[<span style="color: #a31515">"consumer-key"</span>],
      </li>
      <li style="background: #f3f3f3">
        &#160;&#160;&#160;&#160;&#160;&#160;&#1<br /> 60; <span style="color: #2b91af">ConfigurationManager</span>.AppSettings[<span style="color: #a31515">"consumer-secret"</span>],
      </li>
      <li>
        &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #0000ff">new</span> <span style="color: #2b91af">NonceGenerator32Bytes</span>());
      </li>
      <li style="background: #f3f3f3">
        &#160;&#160;&#160; <span style="color: #0000ff">var</span> uri = <span style="color: #2b91af">ConfigurationManager</span>.AppSettings[<span style="color: #a31515">"request_token"</span>];
      </li>
      <li>
        &#160;&#160;&#160; oauthSession.RequestTemporaryCredentials(uri, <span style="color: #a31515">"POST"</span>, <span style="color: #a31515">"http://127.0.0.1:64983/Account/Callback"</span>);
      </li>
      <li style="background: #f3f3f3">
        &#160;&#160;&#160; Session[<span style="color: #a31515">"oauth"</span>] = oauthSession;
      </li>
      <li>
        &#160;&#160;&#160; <span style="color: #0000ff">return</span> Redirect(oauthSession.GetAuthorizationUri(<span style="color: #2b91af">ConfigurationManager</span>.AppSettings[<span style="color: #a31515">"authorize"</span>]));
      </li>
      <li style="background: #f3f3f3">
        }
      </li>
    </ol>
  </div></p>
</div>

Creamos un objeto de la clase OAuthClientSession. Esta clase está pensada para encapsular toda una sesión oAuth, desde el inicio hasta la obtención de los tokens finales.

El constructor de OAuthClienteSession (que se encuentra en Epnuke.OAuth) es:

<div style="border-bottom: #000080 1px solid; border-left: #000080 1px solid; font-family: &#39;Courier New&#39;, courier, monospace; color: #000; font-size: 10pt; border-top: #000080 1px solid; border-right: #000080 1px solid">
  <div style="background: #fff; max-height: 300px; overflow: auto">
    <ol style="padding-bottom: 0px; margin: 0px; padding-left: 5px; padding-right: 0px; white-space: nowrap; background: #ffffff; padding-top: 0px">
      <li>
        <span style="color: #0000ff">public</span> OAuthClientSession(<span style="color: #0000ff">string</span> ckey, <span style="color: #0000ff">string</span> csecret, <span style="color: #2b91af">INonceGenerator</span> nonceGenerator)
      </li>
      <li style="background: #f3f3f3">
        {
      </li>
      <li>
        &#160;&#160;&#160; _consumerKey = ckey;
      </li>
      <li style="background: #f3f3f3">
        &#160;&#160;&#160; _consumerSecret = csecret;
      </li>
      <li>
        &#160;&#160;&#160; _nonceGenerator = nonceGenerator;
      </li>
      <li style="background: #f3f3f3">
        }
      </li>
    </ol>
  </div></p>
</div>

Listemos brevemento los parámetros:

  * ckey: Es el consumer key. Este es la clave que te da el proveedor (en este caso twitter) cuando registras tu aplicación en twitter. Todas las aplicaciones que quieran usar oAuth deben ser registradas en twitter. 
  * csecret: Es el consumer secret. Es la _otra_ clave que te da el proveedor cuando registras tu aplicación. 
  * nonceGenerator: Es el generador de nonces. El nonce es un parámetro que debe ser introducido en cada llamada oAuth y que básicamente “debe ser único”. La librería incluye un generador de nonces llamado NonceGenerator32Bytes. La especificación oAuth no dice NADA al respecto del formato que debe tener el nonce. 

Una vez tenemos el objeto oauthSession creado llamamos al método RequestTemporaryCredentials y empezamos la danza de oAuth. Llamara este método hará lo siguiente:

  1. Creará una petición http oAuth. Una petición http es oAuth si tiene una cabecera “oAuth” con un formato determinado. 
  2. Envia esta petición alproveedor de servicios. En el caso de twitter esto es la URL [https://api.twitter.com/oauth/request_token][3]. Esta petición servirá para indicar al proveedor de servicios que la “aplicación XXX quiere iniciar una sesión oAuth”. 

Si miramos con fiddler, ahí podremos ver nuestra petición:

<img style="background-image: none; border-right-width: 0px; padding-left: 0px; padding-right: 0px; display: inline; border-top-width: 0px; border-bottom-width: 0px; border-left-width: 0px; padding-top: 0px" title="image" border="0" alt="image" src="http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_5DB3D9AA.png" width="528" height="70" />

Los datos enviados son:

<font face="Courier New">POST </font>[<font face="Courier New">https://api.twitter.com/oauth/request_token</font>][4] <font face="Courier New">HTTP/1.1 <br />Authorization:<font color="#0000ff"> OAuth oauth_consumer_key="dbMEy71w8pGLPpbmxlwg", oauth_signature_method="HMAC-SHA1", oauth_version="1.0", oauth_timestamp="1338538452", oauth_nonce="v1TsKDiFyVZNjQ9dfffWS3AMRsWsvM3iycTJilGMY", oauth_callback="http%3A%2F%2F127.0.0.1%3A64983%2FAccount%2FCallback", oauth_signature="%2BsRXDw7GrMcZZ4Xp%2Bvyf%2FVNnhaE%3D" <br /></font>Host: api.twitter.com</font> 

Y los datos recibidos (elimino algunos no relevantes):

<font face="Courier New">HTTP/1.1 200 OK <br />Date: Fri, 01 Jun 2012 08:14:17 GMT <br />Status: 200 OK <br />Content-Length: 147 <br />Content-Type: text/html; charset=utf-8 <br />Pragma: no-cache <br />ETag: "6f0566f3a5d5e338f2cefbc9bd1a7c1f" <br />X-Runtime: 0.01432 <br />X-Transaction: 91371892a8469732 <br />X-MID: 48cd92e044b19b57d1a6b5e471ccd03d2915ce48 <br />Expires: Tue, 31 Mar 1981 05:00:00 GMT <br />Vary: Accept-Encoding <br />Server: tfe</font>

<font face="Courier New">oauth_token=P6xUReTuLm7cDxbSAbDZ8jUriKBIO3zvFfTemXc6VM& <br />oauth_token_secret=yJi3IP7CRts2o68Rl2CQOgNsjRKFvZ8pFxMPJ7TKozY& <br />oauth_callback_confirmed=true</font>

Fijaos en los datos de la cabecera de la petición (en azul). Esos datos conforman la cabecera oAuth y se van enviando a cada petición. Fijaos la firma digital que se computa usando el consumer secret y en como hemos enviado el consumer key (para identificar la aplicación que hace la petición).

Bueno, ya tenemos la respuesta de twitter: nos ha mandado el primer token (como dije antes siempre hay uno público y el secreto que usaremos para firmar la petición): oauth\_token y oauth\_token\_secret. Este token (oauth\_token y oauth\_token\_secret) es el token que dice que “twitter ha aceptado la aplicación cliente XXX como un cliente oAuth válido)”.

Ahora debemos recojer el token (oauth_token) y mandarlo a quien posee las credenciales, en este caso el mismo twitter (la URL exacta es [https://api.twitter.com/oauth/authenticate][5]). Eso es lo que hace el Redirect() de la última línea de la acción LogOnTwitter. Y el resultado:

[<img style="background-image: none; border-right-width: 0px; padding-left: 0px; padding-right: 0px; display: inline; border-top-width: 0px; border-bottom-width: 0px; border-left-width: 0px; padding-top: 0px" title="image" border="0" alt="image" src="http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_13D551FA.png" width="506" height="276" />][6]

Fijaos en la URL: [https://api.twitter.com/oauth/authenticate?**oauth_token=P6xUReTuLm7cDxbSAbDZ8jUriKBIO3zvFfTemXc6VM**][7]

Hemos pasado el oauth_token que hemos obtenido en el punto anterior. De esta manera el autenticador (twitter) sabe que el proveedor de servicios (la api de twitter) sabe quien es esa aplicación que se autodenomina “Epnuke OAuth Library Test”).

Ahora entro las credenciales de twitter… y c
  
ontinúa el baile:

[<img style="background-image: none; border-right-width: 0px; padding-left: 0px; padding-right: 0px; display: inline; border-top-width: 0px; border-bottom-width: 0px; border-left-width: 0px; padding-top: 0px" title="image" border="0" alt="image" src="http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_33B5C377.png" width="521" height="49" />][8]

Las dos peticiones marcadas de amarillo son la petición que acabamos de realizar a api.twitter.com/oauth/authenticate (el Redirect() que teníamos) y otra petición a 127.0.0.1/Account/Callback

¿Quien hace esta petición? Pues bien, mi navegador claro pero ¿por que? Pues bien porque **una vez me he autenticado twitter debe comunicar a mi aplicación que el usuario está autenticado**. Y esto lo hace mandando un código HTTP de redirección hacia esta URL. Y porque esta URL, como la sabe twitter? Pues básicamente porque esta URL (la llamada URL de callback) se introduce cuando se registra la aplicación en twitter. Fijaos en un tema importante: esto tan solo es posible si la aplicación que se quiere conectar con twitter es capaz de exponer un endpoint http. Es decir es, básicamente, una aplicación web. En el caso de que no sea así, existe otro mecanismo (en otro post lo trataremos).

Sigamos… Twitter nos ha mandado via querystring un código, llamado oauth_verifier que sirve para indicarnos que el autenticador ha aceptado las credenciales del usuario. Veamos el código de la acción Callback:

<div style="border-bottom: #000080 1px solid; border-left: #000080 1px solid; font-family: &#39;Courier New&#39;, courier, monospace; color: #000; font-size: 10pt; border-top: #000080 1px solid; border-right: #000080 1px solid">
  <div style="background: #fff; max-height: 300px; overflow: auto">
    <ol style="padding-bottom: 0px; margin: 0px; padding-left: 5px; padding-right: 0px; white-space: nowrap; background: #ffffff; padding-top: 0px">
      <li>
        <span style="color: #0000ff">public</span> <span style="color: #2b91af">ActionResult</span> Callback(<span style="color: #0000ff">string</span> oauth_verifier)
      </li>
      <li style="background: #f3f3f3">
        {
      </li>
      <li>
        &#160;&#160;&#160; <span style="color: #0000ff">var</span> oauthSession = Session[<span style="color: #a31515">"oauth"</span>] <span style="color: #0000ff">as</span> <span style="color: #2b91af">OAuthClientSession</span>;
      </li>
      <li style="background: #f3f3f3">
        &#160;&#160;&#160; oauthSession.Authorize(oauth_verifier, <span style="color: #2b91af">ConfigurationManager</span>.AppSettings[<span style="color: #a31515">"access_token"</span>]);
      </li>
      <li>
        &#160;
      </li>
      <li style="background: #f3f3f3">
        &#160;&#160;&#160; <span style="color: #0000ff">if</span> (oauthSession.IsAuthorized)
      </li>
      <li>
        &#160;&#160;&#160; {
      </li>
      <li style="background: #f3f3f3">
        &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #0000ff">var</span> name = oauthSession.GetAdditionalData(<span style="color: #a31515">"screen_name"</span>);
      </li>
      <li>
        &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #2b91af">FormsAuthentication</span>.SetAuthCookie(name, <span style="color: #0000ff">false</span>);
      </li>
      <li style="background: #f3f3f3">
        &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #0000ff">return</span> RedirectToAction(<span style="color: #a31515">"Index"</span>, <span style="color: #a31515">"Private"</span>);
      </li>
      <li>
        &#160;&#160;&#160; }
      </li>
      <li style="background: #f3f3f3">
        &#160;&#160;&#160; <span style="color: #0000ff">else</span>
      </li>
      <li>
        &#160;&#160;&#160; {
      </li>
      <li style="background: #f3f3f3">
        &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #0000ff">return</span> RedirectToAction(<span style="color: #a31515">"Index"</span>, <span style="color: #a31515">"Home"</span>);
      </li>
      <li>
        &#160;&#160;&#160; }
      </li>
      <li style="background: #f3f3f3">
        &#160;
      </li>
      <li>
        }
      </li>
    </ol>
  </div></p>
</div>

Recuperamos el objeto oAuthSession (que habíamos guardado a la sesión), recogemos el oauth_verifier y llamamos al método “Authorize”. Dicho método hace lo siguiente:

  1. Creará una petición oAuth (con el header oAuth y la firma digital) 
  2. Enviará dicha petición al proveedor de servicios. Para decirle: “Hey! El autenticador ha aceptado las credenciales del usuario y me ha devuelto este token. Puedes darme el token final? Gracias”. 

Si seguimos mirando fiddler:

[<img style="background-image: none; border-right-width: 0px; padding-left: 0px; padding-right: 0px; display: inline; border-top-width: 0px; border-bottom-width: 0px; border-left-width: 0px; padding-top: 0px" title="image" border="0" alt="image" src="http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_4A9494F3.png" width="536" height="51" />][9]

Fijaos como esta última petición NO ha hecho Chrome sino el servidor web (es la petición del método Authorize). De nuevo si miramos los datos de la petición:

<font face="Courier New">POST </font>[<font face="Courier New">https://api.twitter.com/oauth/access_token?oauth_verifier=heqcwkOPuopuMdsoqdMYP8My6kycqOuLYFSeIFK33w</font>][10] <font face="Courier New">HTTP/1.1 <br />Authorization: <font color="#0000ff">OAuth</font> <font color="#0000ff">oauth_consumer_key="dbMEy71w8pGLPpbmxlwg", oauth_signature_method="HMAC-SHA1", oauth_version="1.0", oauth_timestamp="1338539804", oauth_nonce="NQZmiFzvGpaBlY2JW4DAhdL1RBJrGq7qmlygByoIc", oauth_token="P6xUReTuLm7cDxbSAbDZ8jUriKBIO3zvFfTemXc6VM", oauth_signature="kiAPOn%2F8Ybb5oDfd6aPqsG6M750%3D" <br /></font>Host: api.twitter.com <br />Connection: Keep-Alive</font>

Y los de la respuesta (quito datos no relevantes)

<font face="Courier New">HTTP/1.1 200 OK <br />Date: Fri, 01 Jun 2012 08:37:03 GMT <br />Status: 200 OK <br />X-MID: ee5c8c5470911804d71f005a28c69a5be0b72cf4 <br />ETag: "4521b768dcce87386275e193df917953" <br />Expires: Tue, 31 Mar 1981 05:00:00 GMT <br />Content-Length: 162 <br />X-Frame-Options: SAMEORIGIN <br />Pragma: no-cache <br />Last-Modified: Fri, 01 Jun 2012 08:37:03 GMT <br />X-Runtime: 0.04706 <br />Content-Type: text/html; charset=utf-8 <br />Vary: Accept-Encoding <br />Server: tfe</font>

<font color="#0000ff" face="Courier New">oauth_token=84274067-2TMYBrU2cH4x6B3zsYeKYOdCyOpCrxoxKeYZCuDQ& <br />oauth_token_secret=TYRlbDfgb6gVGLxcoVWBTuJyR60pwC3w6fvaq2aw9g& <br />user_id=xxxxxxxx& <br />screen_name=eiximenis</font>

Bueno, hemos recibido otro par de tokens. Ese par de tokens son el par de tokens “final” y que nos van a permitir hacer llamadas oAuth a twitter en nombre del usuario. Además recibimos el ID interno del usuario y su nombre.

En este punto hemos terminado el baile de tokens oAuth. Ya tenemos los tokens finales para “hacer cosas” en nombre del usuario.

Pero nosotros no queríamos hacer nada en nombre del usuario, tan solo usar oAuth para autenticar el usuario. ¡Bueno, ningún problema! El resto de código de la Accion Callback es puro ASP.NET:

  1. Nos aseguramos que la respuesta de oAuth ha sido correcta y que el usuario está autorizado según oAuth (oauthSession.IsAuthorized) 
  2. Recogemos el valor del parámetro screen_name (ese parámetro NO es estándard de oAuth, lo envía twitter) 
  3. Llamam
  
    os a FormsAuthorization.SetAuthCookie para autenticar el usuario dentro de ASP.NET. 

¡Y listos! ¡Hemos terminado!

[<img style="background-image: none; border-right-width: 0px; padding-left: 0px; padding-right: 0px; display: inline; border-top-width: 0px; border-bottom-width: 0px; border-left-width: 0px; padding-top: 0px" title="image" border="0" alt="image" src="http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_68266FF2.png" width="244" height="111" />][11]

¡Hemos conseguido autenticarnos a nuesta web usando las credenciales de twitter!

En posteriores posts iremos explorando como generar estas peticiones oAuth, siempre apoyándonos en el código fuente de la librería que he dejado en CodePlex.

Un saludo!

PD: Si os descargáis el código fuente de Codeplex está este ejemplo.

 [1]: http://www.slideshare.net/eduardtomas/entendiendo-o-auth "http://www.slideshare.net/eduardtomas/entendiendo-o-auth"
 [2]: http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_2EB19DD3.png
 [3]: https://api.twitter.com/oauth/request_token "https://api.twitter.com/oauth/request_token"
 [4]: https://api.twitter.com/oauth/request_token
 [5]: https://api.twitter.com/oauth/authenticate "https://api.twitter.com/oauth/authenticate"
 [6]: http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_2E81EE13.png
 [7]: https://api.twitter.com/oauth/authenticate?oauth_token=P6xUReTuLm7cDxbSAbDZ8jUriKBIO3zvFfTemXc6VM
 [8]: http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_26BBE366.png
 [9]: http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_6B88079A.png
 [10]: https://api.twitter.com/oauth/access_token?oauth_verifier=heqcwkOPuopuMdsoqdMYP8My6kycqOuLYFSeIFK33w
 [11]: http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_0266D917.png