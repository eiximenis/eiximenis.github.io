---
title: Autenticación por AAD en ASP.NET Core

author: eiximenis

date: 2016-04-07T16:33:41+00:00
geeks_url: /?p=1745
geeks_ms_views:
  - 2339
categories:
  - asp.net 5
  - asp.net vNext

---
Este es un **post introductorio, de una serie de posts**, donde veremos como podemos integrar ASP.NET Core y _Azure Active Directory_ (AAD). En este primer escenario el objetivo es tener una aplicación web, donde se requiera hacer login contra AAD para autenticarse.
  
**Nota:** El post está basado en la RC1 de ASP.NET Core… Lo digo porque bueno, a saber que <span style="text-decoration: line-through;">romperán</span> mejorarán en futuras versiones, pues igual algo cambia 🙂
  
<!--more-->


  
**Paso 1 – Configurar Azure Active Directory**
  
Antes de poder usar AAD como proveedor de autenticación necesitas **configurar una aplicación en AAD**. Para ello, debes entrar en el portal de azure. Actualmente parece que la configuración de AAD está solo en el portal viejo ([https://manage.windowsazure.com][1]), de hecho si desde el nuevo te redirige al viejo si vas a la sección de “Active Directory”. Supongo que algún día lo meterán en el nuevo. O no, a saber, la verdad es que el ritmo de novedades de Azure es tal que los desarrolladores del portal nuevo deben ir de culo :p\`
  
Lo dicho, una vez estés en la configuración del AAD debes ir a la pestaña “APPLICATIONS” y dar de alta la nueva aplicación, pulsando el botón de “ADD” de la parte inferior y seleccionar la opción “_Add an application my organization is developing”_. Te aparecerá una pantalla como la siguiente:
  
[<img style="background-image: none; padding-top: 0px; padding-left: 0px; display: inline; padding-right: 0px; border-width: 0px;" title="image" src="http://geeks.ms/etomas/wp-content/uploads/sites/154/2016/04/image_thumb.png" alt="image" width="629" height="446" border="0" />][2]
  
Seleccionamos la opción de “Web Application” y vamos al siguiente paso, donde solo nos pregunta dos cosas:

  1. SIGN-ON URL: Esa es la URL de inicio de sesión. Sirve que coloques directamente la URL inicial de la aplicación. Puedes usar localhost si estás desarrollando (luego se puede modificar, por supuesto). Así, si tienes la aplicación corriendo en localhost:5000 puedes entrar <http://localhost:5000> y listos. **Si entras cualquier otro valor verás que la autenticación sigue funcionando, pero puedes tener problemas en escenarios más avanzados**, así que entra la URL de tu aplicación.
  2. APP ID URI: Esa es una URL que debe ser única. Lo que se suele hacer es usar una URL con tu _tenant_ de Azure (*.onmicrosoft.com) junto con el nombre de la aplicación (p.ej. <http://eiximenis.onmicrosoft.com/DemoGeeks>).

Una vez has entrado esos dos valores se creará la aplicación, pero todavía no hemos terminado. Debes ir a la pestaña CONFIGURATION a entrar un valor adicional, la “REPLY URL”. **Esa sí que es importante**, es donde AAD se redirigirá después del proceso de login. En esta sí que debe estar ejecutándose nuestra aplicación. Se puede cambiar cuando quieras, y no hay problema en usar localhost, así que si tienes la aplicación ejecutandose en localhost:5000, entra algo como <http://localhost:5000/aad>. **No debes entrar la raíz**, así que debes usar una ruta (en este caso he usado /aad), cual es esa ruta, da igual (siempre y cuando no la quieras para nada más, puesto que el middleware de autenticación la va a gestionar).
  
Ahora debes copiar el “CLIENT ID” que aparece en esta misma pantalla de configuración (es un guid) puesto que lo vas a necesitar para configurar la aplicación:
  
[<img style="background-image: none; padding-top: 0px; padding-left: 0px; display: inline; padding-right: 0px; border-width: 0px;" title="image" src="http://geeks.ms/etomas/wp-content/uploads/sites/154/2016/04/image_thumb-1.png" alt="image" width="375" height="246" border="0" />][3]
  
Ahora sí, que hemos terminado con AAD. El siguiente paso es usar configurar la aplicación ASP.NET Core para que use AAD como proveedor de autenticación.
  
**Paso 2: Configurar la aplicación ASP.NET Core**
  
Lo primero es agregar la configuración necesaria. Para ello, lo más sencillo es crear un fichero json nuevo con el siguiente contenido:

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:96e1773e-a694-48b8-a584-f310a3a89e1c" class="wlWriterEditableSmartContent" style="float: none; margin: 0px; display: inline; padding: 0px;">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt;">
    <div style="background: #ddd; max-height: 300px; overflow: auto;">
      <ol style="background: #ffffff; margin: 0 0 0 2.5em; padding: 0 0 0 5px;" start="1">
        <li>
        </li>
        <li>
            <span style="background: #ffffff; color: #2e75b6;">&#8220;AzureAd&#8221;</span><span style="background: #ffffff; color: #000000;">: {</span>
        </li>
        <li>
              <span style="background: #ffffff; color: #2e75b6;">&#8220;AadInstance&#8221;</span><span style="background: #ffffff; color: #000000;">: </span><span style="background: #ffffff; color: #a31515;">&#8220;https://login.microsoftonline.com/{0}&#8221;</span><span style="background: #ffffff; color: #000000;">,</span>
        </li>
        <li>
              <span style="background: #ffffff; color: #2e75b6;">&#8220;CallbackPath&#8221;</span><span style="background: #ffffff; color: #000000;">: </span><span style="background: #ffffff; color: #a31515;">&#8220;/aad&#8221;</span><span style="background: #ffffff; color: #000000;">,     </span><span style="background: #ffffff; color: #008000;">// Este path DEBE ser el mismo que indicamos en el portal</span>
        </li>
        <li>
              <span style="background: #ffffff; color: #2e75b6;">&#8220;ClientId&#8221;</span><span style="background: #ffffff; color: #000000;">: </span><span style="background: #ffffff; color: #a31515;">&#8220;xxxx-xxxxx-xxxx&#8221;</span><span style="background: #ffffff; color: #000000;">,    </span><span style="background: #ffffff; color: #008000;">// El client ID de Azure</span>
        </li>
        <li>
              <span style="background: #ffffff; color: #2e75b6;">&#8220;PostLogoutRedirectUri&#8221;</span><span style="background: #ffffff; color: #000000;">: </span><span style="background: #ffffff; color: #a31515;">&#8220;https://localhost:5000/&#8221;</span><span style="background: #ffffff; color: #000000;">,      </span><span style="background: #ffffff; color: #008000;">// Coloca aqu la URL de la app</span>
        </li>
        <li>
              <span style="background: #ffffff; color: #2e75b6;">&#8220;Tenant&#8221;</span><span style="background: #ffffff; color: #000000;">: </span><span style="background: #ffffff; color: #a31515;">&#8220;eiximenis.onmicrosoft.com&#8221;</span> <span style="background: #ffffff; color: #008000;">// Tu tenant de azure</span>
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

**Nota:** Recuerdas que en la configuración de AAD comenté que el valor de “REPLY URL”, no podía ser la raíz? Pues igualmente el valor de **CallbackPath** no puede ser “/”. Si es así recibirás un 500 sin más información.
  
Guardalo con cualquier nombre (p. ej. AzureAd.json) y agrégalo a la configuración en el Startup.cs:

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:dbcd4726-b1e6-4605-b0c6-784b060e3d5c" class="wlWriterEditableSmartContent" style="float: none; margin: 0px; display: inline; padding: 0px;">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt;">
    <div style="background: #ddd; max-height: 300px; overflow: auto;">
      <ol style="background: #ffffff; margin: 0 0 0 2em; padding: 0 0 0 5px;" start="1">
        <li>
          <span style="background: #ffffff; color: #0000ff;">var</span><span style="background: #ffffff; color: #000000;"> builder = </span><span style="background: #ffffff; color: #0000ff;">new</span> <span style="background: #ffffff; color: #2b91af;">ConfigurationBuilder</span><span style="background: #ffffff; color: #000000;">()</span>
        </li>
        <li>
              <span style="background: #ffffff; color: #000000;">.SetBasePath(appEnv.ApplicationBasePath)</span>
        </li>
        <li>
              <span style="background: #ffffff; color: #000000;">.AddJsonFile(</span><span style="background: #ffffff; color: #a31515;">&#8220;appsettings.json&#8221;</span><span style="background: #ffffff; color: #000000;">)</span>
        </li>
        <li>
              <span style="background: #ffffff; color: #000000;">.AddJsonFile(</span><span style="background: #ffffff; color: #a31515;">&#8220;AzureAd.json&#8221;</span><span style="background: #ffffff; color: #000000;">);</span>
        </li>
      </ol>
    </div>
  </div>
</div>

Bueno, ya tenemos la configuración cargada, vamos a configurar el middleware. Lo primero, claro, va a ser agregarlo al project.json. Agrega esas dos entradas a la sección “dependences”:

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:80cfc40d-870e-4089-b079-2774bc9fe6d3" class="wlWriterEditableSmartContent" style="float: none; margin: 0px; display: inline; padding: 0px;">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt;">
    <div style="background: #ddd; max-height: 300px; overflow: auto;">
      <ol style="background: #ffffff; margin: 0 0 0 2em; padding: 0 0 0 5px;" start="1">
        <li>
          <span style="background: #ffffff; color: #2e75b6;">&#8220;Microsoft.AspNet.Authentication.OpenIdConnect&#8221;</span><span style="background: #ffffff; color: #000000;">: </span><span style="background: #ffffff; color: #a31515;">&#8220;1.0.0-rc1-final&#8221;</span><span style="background: #ffffff; color: #000000;">,</span>
        </li>
        <li>
          <span style="background: #ffffff; color: #2e75b6;">&#8220;Microsoft.AspNet.Authentication.Cookies&#8221;</span><span style="background: #ffffff; color: #000000;">: </span><span style="background: #ffffff; color: #a31515;">&#8220;1.0.0-rc1-final&#8221;</span><span style="background: #ffffff; color: #000000;">,</span>
        </li>
      </ol>
    </div>
  </div>
</div>

Necesitamos el middleware de autenticación de Cookies y el de OpenId (el primero es un requisito del segundo).
  
El siguiente paso es agregar ambos middlewares en el pipeline de la aplicación (en el método Configure de Startup.cs):

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:35615aaf-5749-42e4-b086-7d1b9800ca65" class="wlWriterEditableSmartContent" style="float: none; margin: 0px; display: inline; padding: 0px;">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt;">
    <div style="background: #ddd; max-height: 300px; overflow: auto;">
      <ol style="background: #ffffff; margin: 0 0 0 2.5em; padding: 0 0 0 5px; white-space: nowrap;" start="1">
        <li>
          <span style="background: #ffffff; color: #000000;">app.UseCookieAuthentication(</span><span style="background: #ffffff; color: #0000ff;">new</span> <span style="background: #ffffff; color: #2b91af;">CookieAuthenticationOptions</span><span style="background: #ffffff; color: #000000;">()</span>
        </li>
        <li>
          <span style="background: #ffffff; color: #000000;">{</span>
        </li>
        <li>
              <span style="background: #ffffff; color: #000000;">AutomaticAuthenticate = </span><span style="background: #ffffff; color: #0000ff;">true</span><span style="background: #ffffff; color: #000000;">,</span>
        </li>
        <li>
              <span style="background: #ffffff; color: #000000;">AutomaticChallenge = </span><span style="background: #ffffff; color: #0000ff;">true</span><span style="background: #ffffff; color: #000000;">,</span>
        </li>
        <li>
              <span style="background: #ffffff; color: #000000;">AuthenticationScheme = </span><span style="background: #ffffff; color: #2b91af;">CookieAuthenticationDefaults</span><span style="background: #ffffff; color: #000000;">.AuthenticationScheme</span>
        </li>
        <li>
          <span style="background: #ffffff; color: #000000;">});</span>
        </li>
        <li>
          <span style="background: #ffffff; color: #000000;">app.UseOpenIdConnectAuthentication(options => {</span>
        </li>
        <li>
              <span style="background: #ffffff; color: #000000;">options.AutomaticChallenge = </span><span style="background: #ffffff; color: #0000ff;">true</span><span style="background: #ffffff; color: #000000;">;</span>
        </li>
        <li>
              <span style="background: #ffffff; color: #000000;">options.ClientId = Configuration.Get(</span><span style="background: #ffffff; color: #a31515;">&#8220;AzureAd:ClientId&#8221;</span><span style="background: #ffffff; color: #000000;">, </span><span style="background: #ffffff; color: #a31515;">&#8220;&#8221;</span><span style="background: #ffffff; color: #000000;">);</span>
        </li>
        <li>
              <span style="background: #ffffff; color: #000000;">options.CallbackPath = Configuration.Get(</span><span style="background: #ffffff; color: #a31515;">&#8220;AzureAd:CallbackPath&#8221;</span><span style="background: #ffffff; color: #000000;">, </span><span style="background: #ffffff; color: #a31515;">&#8220;&#8221;</span><span style="background: #ffffff; color: #000000;">);</span>
        </li>
        <li>
              <span style="background: #ffffff; color: #000000;">options.Authority = </span><span style="background: #ffffff; color: #2b91af;">String</span><span style="background: #ffffff; color: #000000;">.Format(Configuration.Get(</span><span style="background: #ffffff; color: #a31515;">&#8220;AzureAd:AadInstance&#8221;</span><span style="background: #ffffff; color: #000000;">, </span><span style="background: #ffffff; color: #a31515;">&#8220;&#8221;</span><span style="background: #ffffff; color: #000000;">), Configuration.Get(</span><span style="background: #ffffff; color: #a31515;">&#8220;AzureAd:Tenant&#8221;</span><span style="background: #ffffff; color: #000000;">, </span><span style="background: #ffffff; color: #a31515;">&#8220;&#8221;</span><span style="background: #ffffff; color: #000000;">));</span>
        </li>
        <li>
              <span style="background: #ffffff; color: #000000;">options.PostLogoutRedirectUri = Configuration.Get(</span><span style="background: #ffffff; color: #a31515;">&#8220;AzureAd:PostLogoutRedirectUri&#8221;</span><span style="background: #ffffff; color: #000000;">, </span><span style="background: #ffffff; color: #a31515;">&#8220;&#8221;</span><span style="background: #ffffff; color: #000000;">);</span>
        </li>
        <li>
              <span style="background: #ffffff; color: #000000;">options.SignInScheme = </span><span style="background: #ffffff; color: #2b91af;">CookieAuthenticationDefaults</span><span style="background: #ffffff; color: #000000;">.AuthenticationScheme;</span>
        </li>
        <li>
          <span style="background: #ffffff; color: #000000;">});</span>
        </li>
      </ol>
    </div>
  </div>
</div>

Importante: **El valor de SignInScheme de las opciones de OpenId debe corresponderse con el valor de AuthenticationScheme de Cookies**. Si no lo haces obtendrás un error después del login:
  
_<span style="font-family: Consolas;">NotSupportedException: Specified method is not supported.<br /> Microsoft.AspNet.Authentication.RemoteAuthenticationHandler`1.HandleSignInAsync(SignInContext context)</span>_ 
  
Observa que colocamos la opción **AutomaticChallenge** a true, eso es porque si la autenticación es necesaria (p. ej. un [Authorize]) en lugar de devolver directamente un 401, se lance el proceso de autenticació contra AAD. Si esa propiedad vale _false_ entonces debemos lanzar este proceso manualmente.
  
El último paso es colocar un [Authorize] en algún controlador y navegar a este controlador. En este momento te debería saltar a la página de autenticación de AAD donde debes entrar tus credenciales. Si esas son correctas, AAD enviará el token a la dirección que pusimos en “REPLY URL” en Azure… y como hemos configurado el valor del **CallbackPath** para que sea esa misma dirección, el middleware de autenticación recojerá ese token y dejará las cookies necesarias. Luego el middleware de autenticación por cookies autenticará nuestra petición y accederemos a la acción (de ahí que uno sea requisito del otro). Una vez tenemos la cookie, por supuesto, ya no necesitamos posteriores accesos a Azure AD para autenticarnos.
  
Y evidentemente en la propiedad User, tendremos un ClaimsPrincipal con una ClaimsIdentity conteniendo los claims enviados por Azure AD:
  
[<img style="background-image: none; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px; border: 0px;" title="image" src="http://geeks.ms/etomas/wp-content/uploads/sites/154/2016/04/image_thumb-2.png" alt="image" width="497" height="246" border="0" />][4]
  
Bueno, lo dejamos aquí en este post. En posts sucesivos iremos viendo otros escenarios de autenticación y operaciones más “avanzadas”
  
Saludos!

 [1]: https://manage.windowsazure.com "https://manage.windowsazure.com"
 [2]: http://geeks.ms/etomas/wp-content/uploads/sites/154/2016/04/image.png
 [3]: http://geeks.ms/etomas/wp-content/uploads/sites/154/2016/04/image-1.png
 [4]: http://geeks.ms/etomas/wp-content/uploads/sites/154/2016/04/image-2.png