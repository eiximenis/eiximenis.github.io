---
title: Swashbuckle con ASP.NET Core y AAD B2C
description: Swashbuckle con ASP.NET Core y AAD B2C
author: eiximenis

date: 2017-10-06T09:11:43+00:00
geeks_url: /?p=1909
geeks_ms_views:
  - 1392
categories:
  - asp.net core
  - oauth

---
[Swashbuckle][1] es una gran herramienta para crear documentaciones de tus APIs desarrolladas con ASP.NET Core. Por debajo usa Swagger y [Swagger UI][2] pero nos abstrae de instalar y configurar esos dos productos. Tan solo tenemos que instalar el paquete NuGet [Swashbuckle.AspNetCore][3] y ya tenemos todo lo que necesitamos.
  
<!--more-->


  
El propio paquete lleva incorporada una versión de Swagger UI y lo único que tenemos que hacer es introducir un par de líneas en nuestra clase _Startup_. La primera de ellas en el _ConfigureServices_:

<pre class="EnlighterJSRAW" data-enlighter-language="csharp">services.AddSwaggerGen(c =&gt;
{
    c.SwaggerDoc("v1", new Info { Title = "Bookings Api", Version = "v1" });
});</pre>

Con esa línea habilitamos la generación de la documentación. El segundo paso activar el _endpoint_ de Swagger UI para poder ver esta documentación. Para ello debemos añadir las siguientes líneas en el método _Configure_ de la clase _Startup_:

<pre class="EnlighterJSRAW" data-enlighter-language="csharp">app.UseSwagger();
app.UseSwaggerUI(c =&gt;
{
    c.SwaggerEndpoint("/swagger/v1/swagger.json", "Bookings Api");
});</pre>

Y con esto tenemos ya la documentación automática para nuestra API. Basta con navegar a /Swagger/UI para poder verla (¡y probar nuestra API!).
  
**Añadiendo soporte para Azure Active Directory B2C**
  
Por defecto Swagger UI no usa ningún tipo de autenticación, por lo que si nuestra API está autenticada, las llamadas de prueba que Swagger UI realiza recibiran un 401. Swagger UI (y Swashbuckle) soporta integración on OAuth2, aunque **en la versión que usa Swashbuckle no hay un soporte directo para OIDC**, pero dado que OIDC es una capa por encima de OAuth2, más o menos lo podremos montar. Vayamos por partes.
  
La primera modificación **debemos realizarla en el propio B2C**. Una característica de B2C es que las URLs de _callbacks_ de los clientes deben ser completas, es decir no es solo el orígen. Eso significa que si indicas que la URL de callback de una aplicación es http://localhost:12345/, B2C solo acepará esa URL de callback. No aceptará otra como http://localhost:12345/callback-result, p. ej.
  
Cuando configuramos Swagger UI (a través de los métodos ofrecidos por Swashbuckle, porque no tenemos acceso al Swagger UI interno) para usar OAuth2, es el propio Swagger UI quien gestiona la URL de callback y esa URL es /swagger-ui/o2c.html. Así que debemos añadir dicha URL de callback a las URL aceptadas por B2C:
  
[<img class="alignnone wp-image-1912 size-full" src="https://geeks.ms/etomas/wp-content/uploads/sites/154/2017/10/b2c-callback-e1507277169599.png" width="618" height="182" />][4]
  
Con esto ya tenemos la aplicación configurada para que B2C nos acepte la URL de callback que usará. Ahora el siguiente punto es configurar Swagger UI. Para ello debemos usar el método _ConfigureSwaggerGen_ en el método _ConfigureServices_ y añadir un _Security Descriptor_, que es lo que le dice a Swagger UI que nuestra API está securizada. Con esto, Swagger UI mostrará un botón de &#8220;Authorize&#8221; que lanzará el flujo de seguridad definido:

<pre class="EnlighterJSRAW" data-enlighter-language="csharp">services.ConfigureSwaggerGen(swaggerGen =&gt;
{
    var authority = string.Format("https://login.microsoftonline.com/{0}/oauth2/v2.0", tenant);
    swaggerGen.AddSecurityDefinition("Swagger", new OAuth2Scheme
    {
        AuthorizationUrl = authority + "/authorize",
        Flow = "implicit",
        TokenUrl = authority + "/connect/token",
        Scopes = new Dictionary&lt;string, string&gt;
        {
            { "openid","User offline" },
        }
    });
});</pre>

Vayamos por partes. La variable _tenant _contiene el tenant de B2C (generalmente algo como _eiximenis.onmicrosoft.com_. Por último en _Scopes_ definimos la lista de scopes oAuth que vamos a pedir. En este caso nos basta con _openid_. Si nuestra API tuviese scopes adicionales definidos en B2C los deberíamos pedir. Swagger UI nos mostrará esta lista de scopes cuando pulsemos el botón de Authorize y podremos elegir los que queremos obtener.
  
El resto de parámetros son los _endpoints_ de autorización y el flujo a usar.
  
Dado que queremos usar OAuth 2, debemos configurar Swagger UI para que sepa los parámetros  a usar y así generar las URLs finales con todos los parámetros. Bueno, de hecho solo necesita uno que es el _client_id_ (el identificador de cliente de la propia API en el B2C). Para ello vamos a añadir una llamada a _ConfigureOAuth2_ dentro del código en el método _UseSwaggerUI__:_

<pre class="EnlighterJSRAW" data-enlighter-language="csharp">app.UseSwaggerUI(c =&gt;
    {
        var b2cConfig = Configuration.GetSection("b2c");
        c.SwaggerEndpoint("/swagger/v1/swagger.json", "Bookings Api");
        c.ConfigureOAuth2(clientId,"y", "z", "z","",
            new
            {
                p = "B2C_1_SignUpInPolicy",
                prompt = "login",
                nonce = "defaultNonce"
            });
    });
    app.UseMvc();
}</pre>

Vayamos por partes: La variable _clientId_ contiene el ID de cliente. Es lo único que necesitamos. El resto de parámetros de _ConfigureOAuth2_ (excepto el último del que ahora vamos a hablar) podrían ser _null_ (o cadena vacía). Pero si  en los dos primeros pones _null_ o cadena vacía, vas a recibir un error de JavaScript en el JS generado por Swagger UI. Por eso yo uso &#8220;y&#8221;, &#8220;z&#8221; y &#8220;z&#8221; como parámetros.
  
El último parámetro **es un objeto anónimo que contiene la queryString** de la URL que se va a generar. En este caso añadimos parámetros propios de B2C. El más importante es el parámetro _p_ que define la política a usar de B2C. En mi caso uso &#8220;_B2C\_1\_SignUpInPolicy_&#8221; porque así se llama mi política, ahí debes usar tu la política de &#8220;Sign-in&#8221; que tengas definida en B2C y quieras usar. El valor de los dos otros parámetros (_nonce_ y _prompt_) los he sacado de la URL de login que usa B2C (puedes verla si, en el portal de configuración del B2C, te vas a los detalles de la política).
  
Eso te **debería funcionar, pero no lo hará**. La razón es que Swagger UI usa siempre el parámetro de _query string _ &#8220;response\_type=token&#8221;, mientras que B2C requiere &#8220;response\_type=id\_token&#8221;. La razón de que Swagger UI use &#8220;response\_type=token&#8221; es porque ese es el valor que OAuth2 especifica para el flujo implícito. Pero B2C usa OIDC, y en OIDC el flujo implícito usa &#8220;response\_type=id\_token&#8221;.
  
La versión de Swagger UI que viene incorporada en Swashbuckle no soporta OIDC, así que no tenemos ninguna manera de cambiar el &#8220;response\_type=token&#8221; por &#8220;response\_type=id_token&#8221; en la URL que nos genera Swagger UI. Por suerte, donde no llegamos des del servidor con C# llegamos desde el cliente con JavaScript.
  
Recuerda que Swagger UI al final se ejecuta en tu navegador. Cuando pulsas el botón para autenticarte en Swagger UI, este genera la URL y luego usa window.open para abrir una nueva ventana con esa URL. Esa nueva ventana es la que gestiona el flujo de autenticación. Así una solución (a lo bruto pero que funciona) es **reemplazar la función window.open por otra que haga lo mismo pero sustituya en la URL el token por id_token**:

<pre class="EnlighterJSRAW" data-enlighter-language="js">window.swaggerUiAuth = window.swaggerUiAuth || {};
window.swaggerUiAuth.tokenName = 'id_token';
if (!window.isOpenReplaced) {
    window.open = function (open) {
        return function (url) {
            url = url.replace('response_type=token', 'response_type=id_token');
            return open.call(window, url);
        };
    }(window.open);
    window.isOpenReplaced = true;
}</pre>

Este código hace el truco. Claro que ahora nos queda saber como podemos &#8220;inyectarlo&#8221; a Swagger UI. Si tuviesemos instalado Swagger UI sería fácil, pero no es nuestro caso: con Swashbuckle, Swagger UI está incrustado en su interior, no vemos los ficheros JavaScript de Swagger UI. Por suerte Swagger UI ofrece un mecanismo para &#8220;inyectar&#8221; JS externo y Swashbuckle soporta ese mecanismo. Basta con añadir la siguiente línea dentro del método _UseSwaggerUI_:

<pre class="EnlighterJSRAW" data-enlighter-language="csharp">c.InjectOnCompleteJavaScript("/swagger-ui-b2c.js");</pre>

El fichero _swagger-ui-b2c.js_ contiene el código anterior. Por supuesto asegúrate que dicho fichero está en _wwwroot_ en la ruta que desees (yo lo he puesto en la raíz, pero eso como lo prefieras).
  
Y ahora si! Ahora ya puedes autenticar tu API contra AAD B2C usando Swagger UI.
  
**Nota:** Swagger actualmente ya soporta OIDC, incluído OIDC discovery (del que un día ya hablaremos), se trata de la versión &#8220;incrustada&#8221; en Swashbuckle la que no lo soporta.
  
&nbsp;

 [1]: https://github.com/domaindrivendev/Swashbuckle
 [2]: https://swagger.io/swagger-ui/
 [3]: https://www.nuget.org/packages/Swashbuckle.AspNetCore/
 [4]: https://geeks.ms/etomas/wp-content/uploads/sites/154/2017/10/b2c-callback.png