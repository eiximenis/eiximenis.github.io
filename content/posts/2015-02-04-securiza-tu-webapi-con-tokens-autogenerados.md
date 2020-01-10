---
title: Securiza tu WebApi con tokens autogenerados

author: eiximenis

date: 2015-02-04T11:20:46+00:00
geeks_url: /?p=1693
geeks_visits:
  - 3298
geeks_ms_views:
  - 7643
categories:
  - Uncategorized

---
El escenario que vamos a abordar en este post es el siguiente: tienes una API creada con ASP.NET WebApi y quieres que sea accesible a través de un token. Pero en este caso quieres ser tu quien proporcione el token y no un tercero como facebook, twitter o Azure Mobile Services (como p. ej. [en el escenario que cubrimos en este post][1]). Para ello nuestra API expondrá un endpoint en el cual se le pasarán unas credenciales de usuario (login y password) para obtener a cambio un token. A partir de ese momento todo el resto de llamadas de la API se realizarán usando este token y las credenciales del usuario no seran necesarias más.

Para empezar crea un proyecto ASP.NET con el template “Empty” pero asegúrate de marcar la checkbox de “Web API” para que nos la incluya por defecto. Luego agregamos los paquetes para hospedar OWIN, ya que vamos a usar componentes OWIN tanto para la creación de los tokens oAuth como su posterior validación. Así pues debes incluir los paquetes “Microsoft.AspNet.WebApi.Owin” y “Microsoft.Owin.Host.SystemWeb”.

El siguiente paso será crear una clase de inicialización de Owin (la famosa Startup). Para ello puedes hacer click con el botón derecho sobre el proyecto en el _solution explorer_ y seleccionar “Add –> OWIN Startup class” o bien crear una clase normal y corriente llamada Startup. El código inicial es el siguiente:

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:fd9ccf9e-0c0f-40f6-98d9-5e6d53f585b9" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
    <div style="background: #ddd; max-height: 300px; overflow: auto">
      <ol start="1" style="background: #000000; margin: 0 0 0 2.5em; padding: 0 0 0 5px;">
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">[</span><span style="background:#1e1e1e;color:#569cd6">assembly</span><span style="background:#1e1e1e;color:#dcdcdc">: </span><span style="background:#1e1e1e;color:#4ec9b0">OwinStartup</span><span style="background:#1e1e1e;color:#dcdcdc">(</span><span style="background:#1e1e1e;color:#569cd6">typeof</span><span style="background:#1e1e1e;color:#dcdcdc">(OauthProviderTest</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#4ec9b0">Startup</span><span style="background:#1e1e1e;color:#dcdcdc">))]</span>
        </li>
        <li>
          &nbsp;
        </li>
        <li>
          <span style="background:#1e1e1e;color:#569cd6">namespace</span><span style="background:#1e1e1e;color:#dcdcdc"> OauthProviderTest</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">{</span>
        </li>
        <li>
              <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">public</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">class</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#4ec9b0">Startup</span>
        </li>
        <li>
              <span style="background:#1e1e1e;color:#dcdcdc">{</span>
        </li>
        <li>
                  <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">public</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">void</span><span style="background:#1e1e1e;color:#dcdcdc"> Configuration(</span><span style="background:#1e1e1e;color:#b8d7a3">IAppBuilder</span><span style="background:#1e1e1e;color:#dcdcdc"> app)</span>
        </li>
        <li>
                  <span style="background:#1e1e1e;color:#dcdcdc">{</span>
        </li>
        <li>
                      <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">var</span><span style="background:#1e1e1e;color:#dcdcdc"> config </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">new</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#4ec9b0">HttpConfiguration</span><span style="background:#1e1e1e;color:#dcdcdc">();</span>
        </li>
        <li>
                      <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#4ec9b0">WebApiConfig</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">Register(config);</span>
        </li>
        <li>
                      <span style="background:#1e1e1e;color:#dcdcdc">ConfigureOAuth(app);</span>
        </li>
        <li>
                      <span style="background:#1e1e1e;color:#dcdcdc">app</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">UseWebApi(config);</span>
        </li>
        <li>
                  <span style="background:#1e1e1e;color:#dcdcdc">}</span>
        </li>
        <li>
              <span style="background:#1e1e1e;color:#dcdcdc">}</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">}</span>
        </li>
      </ol>
    </div></p>
  </div></p>
</div>

La clase WebApiConfig es la que configura WebApi y la generó VS al crear el proyecto (está en la carpeta App_Start). **Nos falta ver el método ConfigureOAuth que veremos ahora mismo**. Observa que el método ConfigureOAuth se llama antes del app.UseWebApi, ya que vamos a añadir middleware OWIN en el pipeline http y debemos hacerlo _antes_ de que se ejecute WebApi. Y por cierto, dado que ahora inicializamos nuestra aplicación usando OWIN **puedes eliminar el fichero Global.asax**, ya que no lo necesitamos para nada.

Veamos ahora el método ConfigureOAuth. En este método debemos añadir el middleware OWIN para la creación de tokens OAuth. Para ello podemos usar el siguiente código:

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:797f2a9f-ea55-4d79-aa74-caaa73509e20" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
    <div style="background: #ddd; max-height: 300px; overflow: auto">
      <ol start="1" style="background: #000000; margin: 0 0 0 2.5em; padding: 0 0 0 5px;">
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">public</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">void</span><span style="background:#1e1e1e;color:#dcdcdc"> ConfigureOAuth(</span><span style="background:#1e1e1e;color:#b8d7a3">IAppBuilder</span><span style="background:#1e1e1e;color:#dcdcdc"> app)</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">{</span>
        </li>
        <li>
              <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">var</span><span style="background:#1e1e1e;color:#dcdcdc"> oAuthServerOptions </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">new</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#4ec9b0">OAuthAuthorizationServerOptions</span><span style="background:#1e1e1e;color:#dcdcdc">()</span>
        </li>
        <li>
              <span style="background:#1e1e1e;color:#dcdcdc">{</span>
        </li>
        <li>
                  <span style="background:#1e1e1e;color:#dcdcdc">AllowInsecureHttp </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">true</span><span style="background:#1e1e1e;color:#dcdcdc">,</span>
        </li>
        <li>
                  <span style="background:#1e1e1e;color:#dcdcdc">TokenEndpointPath </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="backgroun
d:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">new</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#4ec9b0">PathString</span><span style="background:#1e1e1e;color:#dcdcdc">(</span><span style="background:#1e1e1e;color:#d69d85">"/token"</span><span style="background:#1e1e1e;color:#dcdcdc">),</span>
        </li>
        <li>
                  <span style="background:#1e1e1e;color:#dcdcdc">AccessTokenExpireTimeSpan </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#4ec9b0">TimeSpan</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">FromDays(</span><span style="background:#1e1e1e;color:#b5cea8">1</span><span style="background:#1e1e1e;color:#dcdcdc">),</span>
        </li>
        <li>
                  <span style="background:#1e1e1e;color:#dcdcdc">Provider </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">new</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#4ec9b0">CredentialsAuthorizationServerProvider</span><span style="background:#1e1e1e;color:#dcdcdc">(),</span>
        </li>
        <li>
              <span style="background:#1e1e1e;color:#dcdcdc">};</span>
        </li>
        <li>
              <span style="background:#1e1e1e;color:#dcdcdc">app</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">UseOAuthAuthorizationServer(oAuthServerOptions);</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">}</span>
        </li>
      </ol>
    </div></p>
  </div></p>
</div>

Con ello habilitamos un endpoint (/token) para generar los tokens oAuth. El proveedor de dichos tokens es la clase CredentialsAuthorizationServerProvider (que veremos a continuación). Esta clase será **la que recibirá las credenciales (login y password), las validará y generará un token oAuth**.

Por supuesto nos falta ver el código para validar las credenciales y aquí es donde entra la clase CredentialsAuthorizationServerProvider. Esa clase es la que recibe el login y el password del usuario, los valida y crea el token oAuth:

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:97070a4b-2f22-4063-acf4-5d14cf3d87f4" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
    <div style="background: #ddd; max-height: 300px; overflow: auto">
      <ol start="1" style="background: #000000; margin: 0 0 0 2.5em; padding: 0 0 0 5px;">
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">public</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">class</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#4ec9b0">CredentialsAuthorizationServerProvider</span><span style="background:#1e1e1e;color:#dcdcdc"> : </span><span style="background:#1e1e1e;color:#4ec9b0">OAuthAuthorizationServerProvider</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">{</span>
        </li>
        <li>
              <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">public</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">override</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">async</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#4ec9b0">Task</span><span style="background:#1e1e1e;color:#dcdcdc"> ValidateClientAuthentication(</span><span style="background:#1e1e1e;color:#4ec9b0">OAuthValidateClientAuthenticationContext</span><span style="background:#1e1e1e;color:#dcdcdc"> context)</span>
        </li>
        <li>
              <span style="background:#1e1e1e;color:#dcdcdc">{</span>
        </li>
        <li>
                  <span style="background:#1e1e1e;color:#dcdcdc">context</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">Validated();</span>
        </li>
        <li>
              <span style="background:#1e1e1e;color:#dcdcdc">}</span>
        </li>
        <li>
          &nbsp;
        </li>
        <li>
              <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">public</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">override</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">async</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#4ec9b0">Task</span><span style="background:#1e1e1e;color:#dcdcdc"> GrantResourceOwnerCredentials(</span><span style="background:#1e1e1e;color:#4ec9b0">OAuthGrantResourceOwnerCredentialsContext</span><span style="background:#1e1e1e;color:#dcdcdc"> context)</span>
        </li>
        <li>
              <span style="background:#1e1e1e;color:#dcdcdc">{</span>
        </li>
        <li>
          &nbsp;
        </li>
        <li>
                  <span style="background:#1e1e1e;color:#dcdcdc">context</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">OwinContext</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">Response</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">Headers</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">Add(</span><span style="background:#1e1e1e;color:#d69d85">"Access-Control-Allow-Origin"</span><span style="background:#1e1e1e;color:#dcdcdc">, </span><span style="background:#1e1e1e;color:#569cd6">new</span><span style="background:#1e1e1e;color:#dcdcdc">[] { </span><span style="background:#1e1e1e;color:#d69d85">"*"</span><span style="background:#1e1e1e;color:#dcdcdc"> });</span>
        </li>
        <li>
          &nbsp;
        </li>
        <li>
                  <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">using</span><span style="background:#1e1e1e;color:#dcdcdc"> (</span><span style="background:#1e1e1e;color:#4ec9b0">TestContext</span><span style="background:#1e1e1e;color:#dcdcdc"> db </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">new</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#4ec9b0">TestContext</span><span style="background:#1e1e1e;color:#dcdcdc">())</span>
        </li>
        <li>
                  <span style="background:#1e1e1e;color:#dcdcdc">{</span>
        </li>
        <li>
                      <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">var</span><span style="background:#1e1e1e;color:#dcdcdc"> user </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> db</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">Users</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">FirstOrDefault(u </span><span style="background:#1e1e1e;color:#b4b4b4">=></span><span style="background:#1e1e1e;color:#dcdcdc"> u</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">Login </span><span style="background:#1e1e1e;color:#b4b4b4">==</span><span style="background:#1e1e1e;color:#dcdcdc"> context</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">UserName </span><span style="background:#1e1e1e;color:#b4b4b4">&&</span><span style="background:#1e1e1e;color:#dcdcdc"> u</span><span st
yle="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">Password </span><span style="background:#1e1e1e;color:#b4b4b4">==</span><span style="background:#1e1e1e;color:#dcdcdc"> context</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">Password);</span>
        </li>
        <li>
                      <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">if</span><span style="background:#1e1e1e;color:#dcdcdc"> (user </span><span style="background:#1e1e1e;color:#b4b4b4">==</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">null</span><span style="background:#1e1e1e;color:#dcdcdc">)</span>
        </li>
        <li>
                      <span style="background:#1e1e1e;color:#dcdcdc">{</span>
        </li>
        <li>
                          <span style="background:#1e1e1e;color:#dcdcdc">context</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">SetError(</span><span style="background:#1e1e1e;color:#d69d85">"invalid_grant"</span><span style="background:#1e1e1e;color:#dcdcdc">, </span><span style="background:#1e1e1e;color:#d69d85">"The user name or password is incorrect."</span><span style="background:#1e1e1e;color:#dcdcdc">);</span>
        </li>
        <li>
                          <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">return</span><span style="background:#1e1e1e;color:#dcdcdc">;</span>
        </li>
        <li>
                      <span style="background:#1e1e1e;color:#dcdcdc">}</span>
        </li>
        <li>
                  <span style="background:#1e1e1e;color:#dcdcdc">}</span>
        </li>
        <li>
          &nbsp;
        </li>
        <li>
                  <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">var</span><span style="background:#1e1e1e;color:#dcdcdc"> identity </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">new</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#4ec9b0">ClaimsIdentity</span><span style="background:#1e1e1e;color:#dcdcdc">(context</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">Options</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">AuthenticationType);</span>
        </li>
        <li>
                  <span style="background:#1e1e1e;color:#dcdcdc">identity</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">AddClaim(</span><span style="background:#1e1e1e;color:#569cd6">new</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#4ec9b0">Claim</span><span style="background:#1e1e1e;color:#dcdcdc">(</span><span style="background:#1e1e1e;color:#4ec9b0">ClaimTypes</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">Name, context</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">UserName));</span>
        </li>
        <li>
                  <span style="background:#1e1e1e;color:#dcdcdc">identity</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">AddClaim(</span><span style="background:#1e1e1e;color:#569cd6">new</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#4ec9b0">Claim</span><span style="background:#1e1e1e;color:#dcdcdc">(</span><span style="background:#1e1e1e;color:#4ec9b0">ClaimTypes</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">Role, </span><span style="background:#1e1e1e;color:#d69d85">"user"</span><span style="background:#1e1e1e;color:#dcdcdc">));</span>
        </li>
        <li>
                  <span style="background:#1e1e1e;color:#dcdcdc">context</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">Validated(identity);</span>
        </li>
        <li>
              <span style="background:#1e1e1e;color:#dcdcdc">}</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">}</span>
        </li>
      </ol>
    </div></p>
  </div></p>
</div>

Lo único remarcable es la primera línea del método GrantResourceOwnerCredentials que se encarga de habilitar CORS. WebApi tiene soporte para CORS, pero el **endpoint /token no está gestionado por WebApi** si no por el middleware OWIN así que debemos asegurarnos de que mandamos las cabeceras para habilitar CORS. El resto del código es un acceso a la BBDD usando un contexto de EF para encontrar el usuario con el login y el password correcto. **Por supuesto en un entorno de producción eso no se haría así.** Este código no tiene en cuenta que las passwords deben guardarse como un hash en la BBDD. Si quieres acceder a BBDD directamente debes generar el hash de los passwords aunque si una mejor opción es usar Identity (y la clase UserManager) para acceder a los datos de los usuarios. Una vez validado que las credenciales son correctas creamos la ClaimsIdentity y generamos el token correspondiente.

Para probarlo podéis hacer un POST a /token con las siguientes características:

  * Content-type: application/x-www-form-urlencoded
  * En el cuerpo de la petición añadir los campos:
  * grant_type = “password”
  * username = “Login del usuario”
  * password = “Password del usuario”

Es decir como si tuvieses un formulario con esos campos y lo enviaras via POST al servidor. Os pongo una captura de la petición usando <a href="https://chrome.google.com/webstore/detail/postman-rest-client/fdmmgilgnpjigdojojpjoooidkmcomcm" target="_blank" rel="noopener noreferrer">postman</a>:

[<img title="image" style="border-top: 0px; border-right: 0px; background-image: none; border-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 10px 10px 0px 0px; border-left: 0px; display: inline; padding-right: 0px" border="0" alt="image" src="http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_168F9B21.png" width="644" height="271" />][2]

Se puede ver que la respuesta es el token, el tiempo de expiración (que se corresponde con el valor de la propiedad AccessTokenExpireTimeSpan) y el tipo de autenticación que es bearer (ese es el valor que deberemos colocar en la cabecera Authorization).

A partir de ese punto **tenemos un token válido y nos olvidamos de las credenciales del usuario**. Al menos hasta que el token no caduque. Cuando el token caduque, se deberá generar uno nuevo con otro POST al endpoint /token.

El siguiente punto es **habilitar WebApi para que tenga en cuenta esos tokens**. Hasta ahora nos hemos limitado a generar tokens, pero WebApi no hace nada con ellos. De hecho no habilitamos WebApi si no que añadimos otro módulo OWIN para autenticarnos en base a esos tokens. El proceso ocurre antes y es transparente a WebApi. Para ello debemos **añadir** las siguientes líneas a la clase Startup al final del método ConfigureOAuth:

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:fe888b50-bb5b-4f9d-9a22-533c043d974a" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
    <div style="background: #ddd; max-height: 300px; overflow: auto">
      <ol start="1" style="background: #000000; margin: 0 0 0 2em; padding: 0 0 0 5px;">
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:
#569cd6">var</span><span style="background:#1e1e1e;color:#dcdcdc"> authOptions </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">new</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#4ec9b0">OAuthBearerAuthenticationOptions</span><span style="background:#1e1e1e;color:#dcdcdc">()</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">{</span>
        </li>
        <li>
              <span style="background:#1e1e1e;color:#dcdcdc">AuthenticationMode </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> Microsoft</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">Owin</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">Security</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#b8d7a3">AuthenticationMode</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">Active</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">};</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">app</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">UseOAuthBearerAuthentication(authOptions);</span>
        </li>
      </ol>
    </div></p>
  </div></p>
</div>

Así añadimos el módulo de OWIN que autentica en base a esos tokens. Para hacer la prueba vamos a crear un controlador de WebApi y vamos a indicar que es solo para usuarios autenticados:

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:f72bf34c-876d-46df-bf29-5446859f0bd8" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
    <div style="background: #ddd; max-height: 300px; overflow: auto">
      <ol start="1" style="background: #000000; margin: 0 0 0 2em; padding: 0 0 0 5px;">
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">[</span><span style="background:#1e1e1e;color:#4ec9b0">Authorize</span><span style="background:#1e1e1e;color:#dcdcdc">]</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">public</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">class</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#4ec9b0">SecureController</span><span style="background:#1e1e1e;color:#dcdcdc"> : </span><span style="background:#1e1e1e;color:#4ec9b0">ApiController</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">{</span>
        </li>
        <li>
              <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">public</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#b8d7a3">IHttpActionResult</span><span style="background:#1e1e1e;color:#dcdcdc"> Get()</span>
        </li>
        <li>
              <span style="background:#1e1e1e;color:#dcdcdc">{</span>
        </li>
        <li>
                  <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">return</span><span style="background:#1e1e1e;color:#dcdcdc"> Ok(</span><span style="background:#1e1e1e;color:#d69d85">"Welcome "</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#b4b4b4">+</span><span style="background:#1e1e1e;color:#dcdcdc"> User</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">Identity</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">Name);</span>
        </li>
        <li>
              <span style="background:#1e1e1e;color:#dcdcdc">}</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">}</span>
        </li>
      </ol>
    </div></p>
  </div></p>
</div>

Y ahora para probarlo hacemos un GET a la URL del controlador (/api/secure) **y tenemos que pasar la cabecera Authorization**. El valor de dicha cabecera es “Bearer <token>”:

[<img title="image" style="border-top: 0px; border-right: 0px; background-image: none; border-bottom: 0px; padding-top: 0px; padding-left: 0px; border-left: 0px; display: inline; padding-right: 0px" border="0" alt="image" src="http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_6A729E2F.png" width="644" height="55" />][3]

Y con esto deberíamos obtener la respuesta del controlador. En caso de que no pasar la cabecera o que el token fuese incorrecto el resultado sería un HTTP 401 (no autorizado).

**Unas palabras sobre los tokens**

Fíjate que **en ningún momento** guardamos en BBDD los tokens de los usuarios y esos tokens son válidos durante todo su tiempo de vida **incluso en caso de que el servidor se reinicie**. Eso es así porque el token realmente es un ticket de autenticación encriptado. El middleware de OWIN cuando recibe un token se limita a desencriptarlo y en caso de que sea válido, extrae los datos (los claims de la ClaimIdentity creada al generar el token) y coloca dicha ClaimIdentity como identidad de la petición. Es por eso que en el controlador podemos usar User.Identity.Name y recibimos el nombre del usuario que entramos.

Por lo tanto **cualquier persona que intercepte el token** podrá ejecutar llamadas “en nombre de” el usuario mientras este token sea válido. A todos los efectos poseer el token de autenticación equivale a poseer las credenciales del usuario, al menos mientras el token sea válido. Tenlo presente si optas por ese mecanismo de autenticación: si alguien roba el token, la seguridad se ve comprometida mientras el token sea válido. Por supuesto eso no es distinto al caso de usar una cookie, donde el hecho de robar la cookie compromete la seguridad de igual forma. Y los tokens son más manejables que las cookies y dan menos problemas, en especial en llamadas entre orígenes web distintos.

¡Espero que este post os resulte interesante!

 [1]: http://geeks.ms/blogs/etomas/archive/2014/12/18/securizar-tu-webapi-con-azure-mobile-services.aspx
 [2]: http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_7F618FF1.png
 [3]: http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_76085B6E.png