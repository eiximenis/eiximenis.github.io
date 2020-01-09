---
title: Middlewares de autenticación en asp.net core
author: eiximenis

date: 2016-04-19T10:25:32+00:00
geeks_url: /?p=1766
geeks_ms_views:
  - 4443
categories:
  - asp.net 5
  - asp.net vNext

---
La autenticación y autorización de peticiones es una de las funcionalidades que más quebraderos da en el desarrollo de aplicaciones en ASP.NET. Además es que ha ido cambiando con el tiempo… En un escenario de internet, en ASP.NET clásico, ya fuese Webforms o MVC usábamos FormsAuthentication. Por otra parte cuando apareció WebApi, incorporó sus propios mecanismos de autenticación y autorización, generalmente basados en la implementación de MessageHandlers.
  
<!--more-->


  
Con la aparición OWIN y Katana, FormsAuthentication fue reemplazado por el módulo de autenticación por cookies de Katana. A pesar de que MVC no era un módulo OWIN, era posible usar Katana para autenticar las peticiones ya fuesen para MVC o para WebApi (que sí que era un módulo OWIN). Posteriormente fueron surgiendo nuevos módulos de autenticación implementados como módulos OWIN para otros escenarios, como el uso de tokens. Esos módulos fueron reemplazando el uso de MessageHandlers ya que a diferencia de estos, podían securizar cualquier componente del pipeline OWIN (p. ej. una api realizada con NancyFx) y no solo WebApi.
  
Finalmente tenemos asp.net core que sigue el modelo de OWIN basado en middlewares de autenticación que securizan todo el pipeline. Da igual lo que tengas “por detrás”, ya sea MVC6 o cualquier otro middleware, la autenticación y autorización es común.
  
**Responsabilidades de un middleware de autenticación**
  
¿Qué debe hacer un middleware de autenticación? La respuesta rápida, por supuesto, es “autenticar la petición”. Pero esto, exactamente… ¿qué significa?
  
Desde un punto de vista técnico, autenticar una petición es dejar en el contexto un IPrincipal que tenga una identidad con la propiedad IsAuthenticated a true:
  
[<img style="background-image: none; padding-top: 0px; padding-left: 0px; display: inline; padding-right: 0px; border-width: 0px;" title="image" src="http://geeks.ms/etomas/wp-content/uploads/sites/154/2016/04/image_thumb-3.png" alt="image" width="639" height="210" border="0" />][1]
  
Para obtener esta identidad (en general un objeto ClaimsIdentity) el middleware debe tener determinados datos en la petición que mande el cliente. Esos datos dependen del middleware: una cookie, un token o lo que sea. Si existe este dato y es correcto el middleware puede crear la identidad y autenticarla.
  
Pero autenticar va un paso más allá. Imagina que te autenticas via Azure Active Directory (AAD). Es evidente que es necesario un paso adicional para poder autenticar al usuario: mostrar la página de login de AAD. Y no solo eso, si no gestionar la respuesta que AAD nos da (a través de una URL de _callback_ definida en nuestra aplicación). ¿Quien se debe encargar de mostrar la página de login de AAD? Pues de nuevo, es el middleware de autenticación el que lo hace. Y el que gestiona la URL de _callback_ y procesa la respuesta de AAD. Lo mismo, por supuesto, ocurre si en lugar de AAD, usas facebook, google o cualquier otro proveedor oAuth o OpenID Connect.
  
Por lo tanto hemos inferido al menos dos acciones que el middleware de autenticación debe hacer:

  1. Leer algún elemento de la petición (cookie, token, etc) y establecer una identidad autenticada en el contexto en base a la existencia y validez de dicho elemento.
  2. Iniciar un proceso de autenticación externo si es necesario (mostrar la página de login de AAD, de facebook, …)

Pero todavía queda una pregunta… **¿Debe realizar el middleware esas dos acciones automáticamente o no?** Para entender la pregunta imagina que un cliente hace una petición a una acción de un controlador que está protegida mediante [Authorize]. Authorize, básicamente, lo que hace es mirar si existe una identidad autenticada en el contexto (la da igual como se haya autenticado). Si no la encuentra Authorize cortocircuita la petición y envía una respuesta HTTP 401.
  
Imagina ahora un pipeline compuesto de un middleware de autenticación y MVC6 con un controlador con una acción decorada con [Authorize]. Imaginemos los siguientes escenarios:

  1. **La petición no está autenticada** (no hay cookie, ni token, ni nada). El middleware de autenticación no hace nada y la petición llega al Authorize. Este comprueba que no hay identidad autenticada en el contexto y devuelve un 401. Este 401 pasa de vuelta por el middleware de autenticación, **quien no hace nada. El usuario recibe un 401.**
  2. **La petición no está autenticada** (no hay cookie, ni token, ni nada). El middleware de autenticación no hace nada y la petición llega al Authorize. Este comprueba que no hay identidad autenticada en el contexto y devuelve un 401. Este 401 pasa de vuelta por el middleware de autenticación, quien **inicia un proceso de autenticación**. Este proceso puede ser redirigir el usuario a una página de login interno, o bien a la página de facebook o de AAD o cualquier otra cosa.
  3. La petición **está autenticada** (hay cookie, o token o lo que sea necesario). **El middleware de autenticación lee la información de la petición, crea la identidad y la coloca en el contexto**. La petición llega al Authorize quien comprueba la existencia de la identidad autenticada y deja proseguir la petición hacia la acción del controlador. **La acción se invoca y el resultado se devuelve**.
  4. La petición **está autenticada** (hay cookie, o token o lo que sea necesario). **El middleware de autenticación NO lee la información (a pesar de que existe) y por lo tanto no crea la identidad** en el contexto. La petición llega a Authorize quien, al no encontrar identidad autenticada, devuelve un 401. El middleware de autenticación recibe el 401 y no hace nunca nada (no tiene sentido que inicie un proceso de autenticación porque la petición está ya autenticada).

Esos cuatro escenarios surgen de la posibilidad de que el middleware haga o no de forma automática las dos acciones que habíamos comentado antes:

  * Leer la información de la request y crear la identidad en el contexto
  * Iniciar un proceso de autenticación al recibir un 401

A la primera de las dos acciones la llamamos “Autenticar la petición” y **si el middleware debe hacerlo de forma automática o no, se controla mediante la propiedad AutomaticAuthenticate**.
  
A la segunda de las dos acciones la llamamos “Iniciar el _challenge_ de autenticación” y **si el middleware debe hacerlo de forma automática o no, se controla mediante la propiedad AutomaticChallenge**.
  
Esas dos propiedades las establecemos cuando registramos el middleware, en el método Configure de la clase Startup, como p. ej. en el siguiente código:

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:a0c3ad61-5a4a-4153-bd7b-736873f12e60" class="wlWriterEditableSmartContent" style="float: none; margin: 0px; display: inline; padding: 0px;">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt;">
    <div style="background: #ddd; max-height: 300px; overflow: auto;">
      <ol style="background: #ffffff; margin: 0 0 0 2em; padding: 0 0 0 5px; white-space: nowrap;" start="1">
        <li>
          <span style="background: #ffffff; color: #000000;">app.UseCookieAuthentication(opt =></span>
        </li>
        <li>
          <span style="background: #ffffff; color: #000000;">{</span>
        </li>
        <li>
              <span style="background: #ffffff; color: #000000;">opt.AutomaticAuthenticate = </span><span style="background: #ffffff; color: #0000ff;">true</span><span style="background: #ffffff; color: #000000;">;</span>
        </li>
        <li>
              <span style="background: #ffffff; color: #000000;">opt.AutomaticChallenge = </span><span style="background: #ffffff; color: #0000ff;">false</span><span style="background: #ffffff; color: #000000;">;</span>
        </li>
        <li>
          <span style="background: #ffffff; color: #000000;">});</span>
        </li>
      </ol>
    </div>
  </div>
</div>

En este caso registramos el middleware de cookies. Si la cookie existe la petición se autenticará automáticamente (AutomaticAuthenticate vale true). Si la cookie NO existe, el usuario recibirá un 401 (asumiendo que accede a una acción protegida por [Authorize]). Si AutomaticChallenge fuese true, en lugar de recibir un 401, el usuario sería redirigido a una página de Login.
  
**Autenticar peticiones de forma manual**
  
De los cuatros escenarios mencionados, igual piensas que el último no tiene sentido: Si ponemos AutomaticAuthenticate a false, la petición no se autenticará a pesar de tener los datos. ¿Qué sentido tiene eso? Pues eso nos permite **autenticar la petición cuando lo deseemos**. Por supuesto, el escenario no tiene sentido si lo combinamos con Authorize (porque Authorize siempre espera una petición ya autenticada), pero si que lo tiene en otros escenarios. Veamos un ejemplo.
  
Para ello tenemos registrado el middleware de autenticación por cookies de la siguiente manera:

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:d1b08394-b7e6-4740-a30d-3f6b8e38f954" class="wlWriterEditableSmartContent" style="float: none; margin: 0px; display: inline; padding: 0px;">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt;">
    <div style="background: #ddd; max-height: 300px; overflow: auto;">
      <ol style="background: #ffffff; margin: 0 0 0 2em; padding: 0 0 0 5px; white-space: nowrap;" start="1">
        <li>
          <span style="background: #ffffff; color: #000000;">app.UseCookieAuthentication(opt =></span>
        </li>
        <li>
          <span style="background: #ffffff; color: #000000;">{</span>
        </li>
        <li>
              <span style="background: #ffffff; color: #000000;">opt.AutomaticAuthenticate = </span><span style="background: #ffffff; color: #0000ff;">false</span><span style="background: #ffffff; color: #000000;">;</span>
        </li>
        <li>
              <span style="background: #ffffff; color: #000000;">opt.AutomaticChallenge = </span><span style="background: #ffffff; color: #0000ff;">false</span><span style="background: #ffffff; color: #000000;">;</span>
        </li>
        <li>
              <span style="background: #ffffff; color: #000000;">opt.AuthenticationScheme = </span><span style="background: #ffffff; color: #2b91af;">CookieAuthenticationDefaults</span><span style="background: #ffffff; color: #000000;">.AuthenticationScheme;</span>
        </li>
        <li>
          <span style="background: #ffffff; color: #000000;">});</span>
        </li>
      </ol>
    </div>
  </div>
</div>

En esta configuración el middleware ni autenticará peticiones ni iniciará ningún _challenge_. Supongamos ahora que tenemos en un controlador (HomeController) una acción para crear la cookie de autenticación:

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:9602791f-a01e-4448-8280-fb70ddb49186" class="wlWriterEditableSmartContent" style="float: none; margin: 0px; display: inline; padding: 0px;">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt;">
    <div style="background: #ddd; max-height: 300px; overflow: auto;">
      <ol style="background: #ffffff; margin: 0 0 0 2.5em; padding: 0 0 0 5px; white-space: nowrap;" start="1">
        <li>
          <span style="background: #ffffff; color: #0000ff;">public</span> <span style="background: #ffffff; color: #0000ff;">async</span> <span style="background: #ffffff; color: #2b91af;">Task</span><span style="background: #ffffff; color: #000000;"><</span><span style="background: #ffffff; color: #2b91af;">IActionResult</span><span style="background: #ffffff; color: #000000;">> Signin()</span>
        </li>
        <li>
          <span style="background: #ffffff; color: #000000;">{</span>
        </li>
        <li>
              <span style="background: #ffffff; color: #0000ff;">var</span><span style="background: #ffffff; color: #000000;"> principal = </span><span style="background: #ffffff; color: #0000ff;">new</span> <span style="background: #ffffff; color: #2b91af;">ClaimsPrincipal</span><span style="background: #ffffff; color: #000000;">(</span>
        </li>
        <li>
                  <span style="background: #ffffff; color: #0000ff;">new</span> <span style="background: #ffffff; color: #2b91af;">ClaimsIdentity</span><span style="background: #ffffff; color: #000000;">(</span>
        </li>
        <li>
                      <span style="background: #ffffff; color: #0000ff;">new</span> <span style="background: #ffffff; color: #2b91af;">Claim</span><span style="background: #ffffff; color: #000000;">[] { </span><span style="background: #ffffff; color: #0000ff;">new</span> <span style="background: #ffffff; color: #2b91af;">Claim</span><span style="background: #ffffff; color: #000000;">(</span><span style="background: #ffffff; color: #2b91af;">ClaimTypes</span><span style="background: #ffffff; color: #000000;">.NameIdentifier, </span><span style="background: #ffffff; color: #a31515;">&#8220;Eiximenis&#8221;</span><span style="background: #ffffff; color: #000000;">) },</span>
        </li>
        <li>
                      <span style="background: #ffffff; color: #2b91af;">CookieAuthenticationDefaults</span><span style="background: #ffffff; color: #000000;">.AuthenticationScheme</span>
        </li>
        <li>
                   <span style="background: #ffffff; color: #000000;">)</span>
        </li>
        <li>
              <span style="background: #ffffff; color: #000000;">);</span>
        </li>
        <li>
              <span style="background: #ffffff; color: #0000ff;">var</span><span style="background: #ffffff; color: #000000;"> authManager = HttpContext.Authentication;</span>
        </li>
        <li>
              <span style="background: #ffffff; color: #0000ff;">await</span><span style="background: #ffffff; color: #000000;"> authManager.SignInAsync(</span><span style="background: #ffffff; color: #2b91af;">CookieAuthenticationDefaults</span><span style="background: #ffffff; color: #000000;">.AuthenticationScheme, principal);</span>
        </li>
        <li>
              <span style="background: #ffffff; color: #0000ff;">return</span><span style="background: #ffffff; color: #000000;"> Json(</span><span style="background: #ffffff; color: #0000ff;">new</span><span style="background: #ffffff; color: #000000;"> { Data = </span><span style="background: #ffffff; color: #a31515;">&#8220;Cookie set&#8221;</span><span style="background: #ffffff; color: #000000;"> });</span>
        </li>
        <li>
          <span style="background: #ffffff; color: #000000;">}</span>
        </li>
      </ol>
    </div>
  </div>
</div>

En este código usamos el **AuthorizationManager** para “hacer un signin”, es decir para crear la cookie que necesitamos para que la petición esté autenticada. Observa que le indicamos que middleware de autenticación debe “hacer el signin” (recuerda que hacer un signin puede ser crear una cookie u otra cosa). Para ello el parámetro que le pasamos a **SignInAsync** debe ser el mismo valor que la propiedad AuthenticationScheme de alguno de los middlewares de autenticación registrados.
  
Ahora imagina otra acción como la siguiente:

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:d85f9bac-0750-4b22-995c-d3058c000d25" class="wlWriterEditableSmartContent" style="float: none; margin: 0px; display: inline; padding: 0px;">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt;">
    <div style="background: #ddd; max-height: 300px; overflow: auto;">
      <ol style="background: #ffffff; margin: 0 0 0 2em; padding: 0 0 0 5px; white-space: nowrap;" start="1">
        <li>
          <span style="background: #ffffff; color: #000000;">[</span><span style="background: #ffffff; color: #2b91af;">Authorize</span><span style="background: #ffffff; color: #000000;">]</span>
        </li>
        <li>
          <span style="background: #ffffff; color: #0000ff;">public</span> <span style="background: #ffffff; color: #2b91af;">IActionResult</span><span style="background: #ffffff; color: #000000;"> Secure()</span>
        </li>
        <li>
          <span style="background: #ffffff; color: #000000;">{</span>
        </li>
        <li>
              <span style="background: #ffffff; color: #0000ff;">return</span><span style="background: #ffffff; color: #000000;"> Json(</span><span style="background: #ffffff; color: #0000ff;">new</span><span style="background: #ffffff; color: #000000;"> { Ok = </span><span style="background: #ffffff; color: #0000ff;">true</span><span style="background: #ffffff; color: #000000;"> });</span>
        </li>
        <li>
          <span style="background: #ffffff; color: #000000;">}</span>
        </li>
      </ol>
    </div>
  </div>
</div>

Si el usuario navega primero a /Home/Signin (que establece la cookie) vemos que efectivamente la cookie se establece:
  
[<img style="background-image: none; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px; border-width: 0px;" title="image" src="http://geeks.ms/etomas/wp-content/uploads/sites/154/2016/04/image_thumb-4.png" alt="image" width="640" height="133" border="0" />][2]
  
Y si ahora navegamos a /Home/Secure vemos **que recibimos un 401, a pesar de mandar la cookie**:
  
[<img style="background-image: none; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px; border-width: 0px;" title="image" src="http://geeks.ms/etomas/wp-content/uploads/sites/154/2016/04/image_thumb-5.png" alt="image" width="640" height="160" border="0" />][3]
  
Eso es **porque la propiedad AutomaticAuthenticate** la hemos puesto a false, por lo que el middleware no coloca la identidad en el contexto y por lo tanto Authenticate devuelve el 401.
  
Ahora bien, **podemos autenticar la petición manualmente**, es decir obtener la identidad autenticada si la hay:

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:2d67dd4e-9aff-4189-8c41-16f16c8a7287" class="wlWriterEditableSmartContent" style="float: none; margin: 0px; display: inline; padding: 0px;">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt;">
    <div style="background: #ddd; max-height: 300px; overflow: auto;">
      <ol style="background: #ffffff; margin: 0 0 0 2.5em; padding: 0 0 0 5px; white-space: nowrap;" start="1">
        <li>
          <span style="background: #ffffff; color: #0000ff;">public</span> <span style="background: #ffffff; color: #0000ff;">async</span> <span style="background: #ffffff; color: #2b91af;">Task</span><span style="background: #ffffff; color: #000000;"><</span><span style="background: #ffffff; color: #2b91af;">IActionResult</span><span style="background: #ffffff; color: #000000;">> SecureManual()</span>
        </li>
        <li>
          <span style="background: #ffffff; color: #000000;">{</span>
        </li>
        <li>
              <span style="background: #ffffff; color: #0000ff;">var</span><span style="background: #ffffff; color: #000000;"> authManager = HttpContext.Authentication;</span>
        </li>
        <li>
              <span style="background: #ffffff; color: #0000ff;">var</span><span style="background: #ffffff; color: #000000;"> principal = </span><span style="background: #ffffff; color: #0000ff;">await</span><span style="background: #ffffff; color: #000000;"> authManager.AuthenticateAsync(</span><span style="background: #ffffff; color: #2b91af;">CookieAuthenticationDefaults</span><span style="background: #ffffff; color: #000000;">.AuthenticationScheme);</span>
        </li>
        <li>
              <span style="background: #ffffff; color: #0000ff;">if</span><span style="background: #ffffff; color: #000000;"> (principal != </span><span style="background: #ffffff; color: #0000ff;">null</span><span style="background: #ffffff; color: #000000;"> &&  principal.Identity.IsAuthenticated)</span>
        </li>
        <li>
              <span style="background: #ffffff; color: #000000;">{</span>
        </li>
        <li>
                  <span style="background: #ffffff; color: #0000ff;">return</span><span style="background: #ffffff; color: #000000;"> Json(</span><span style="background: #ffffff; color: #0000ff;">new</span><span style="background: #ffffff; color: #000000;"> { Ok = </span><span style="background: #ffffff; color: #0000ff;">true</span><span style="background: #ffffff; color: #000000;"> });</span>
        </li>
        <li>
              <span style="background: #ffffff; color: #000000;">}</span>
        </li>
        <li>
              <span style="background: #ffffff; color: #0000ff;">else</span>
        </li>
        <li>
              <span style="background: #ffffff; color: #000000;">{</span>
        </li>
        <li>
                  <span style="background: #ffffff; color: #0000ff;">return</span><span style="background: #ffffff; color: #000000;"> HttpUnauthorized();</span>
        </li>
        <li>
              <span style="background: #ffffff; color: #000000;">}</span>
        </li>
        <li>
          <span style="background: #ffffff; color: #000000;">}</span>
        </li>
      </ol>
    </div>
  </div>
</div>

En este código usamos, de nuevo, el **AuthorizationManager** para autenticar la petición a través del método **AuthenticateAsync**. En este caso “autenticar la petición” significa, leer la cookie y obtener el principal con la identidad autenticada. Observa como la acción no está decorada con [Authorize] y **también que a AuthenticateAsync le indicamos qué middleware de autenticación debe autenticar la petición.** El valor pasado a AuthenticateAsync (una cadena) debe ser el mismo que el valor de la propiedad AuthenticationScheme de alguno de los middlewares de autenticación registrados.
  
**En resumen**: Los middlewares de autenticación gestionan todo el proceso de “autenticar una petición”, leyendo sus datos y iniciando el proceso de _challenge_  si es necesario. Debemos tener presente que tanto leer los datos de la petición como inciar el _challenge_ pueden hacerlo automáticamente o de forma manual.
  
Podemos tener varios mecanismos de autenticación (y en muchas aplicaciones es así), p. ej. uno via cookies que proteja la aplicación y otro via bearer token para accesos externos a la api REST. Y otro para acceso via OpenId Connect. Y otro para… en fin, los escenarios son muchísimos.
  
Espero que este post os haya ayudado a entender un poco más los middlewares de autenticación. Aunqué seguiré hablando de ellos, **no puedo menos que recomendaros <a href="https://channel9.msdn.com/Events/NET-Conference/2016/Autenticacin-y-autorizacin-en-ASPNET-Core-10" target="_blank" rel="noopener noreferrer">la charla que Hugo Biarge dio en la DotNet Conference donde desgranó este y otros muchos aspectos de la autenticación y autorización en .NET</a>**.
  
Saludos!

 [1]: http://geeks.ms/etomas/wp-content/uploads/sites/154/2016/04/image-3.png
 [2]: http://geeks.ms/etomas/wp-content/uploads/sites/154/2016/04/image-4.png
 [3]: http://geeks.ms/etomas/wp-content/uploads/sites/154/2016/04/image-5.png