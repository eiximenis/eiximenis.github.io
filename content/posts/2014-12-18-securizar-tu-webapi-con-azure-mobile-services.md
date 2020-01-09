---
title: Securizar tu WebApi con Azure Mobile Services
description: Securizar tu WebApi con Azure Mobile Services
author: eiximenis

date: 2014-12-18T15:08:57+00:00
geeks_url: /?p=1688
geeks_visits:
  - 1139
geeks_ms_views:
  - 1504
categories:
  - Uncategorized

---
El escenario es el siguiente: tenemos un conjunto de servicios WebApi **que no tienen porque estar desplegados en Azure** pero queremos que esos servicios solo sean accesibles para usuarios que se hayan autenticado previamente a través de un proveedor externo (p. ej. Twitter) usando la infrastructura de Azure Mobile Services.

**Creación y configuración de Mobile Services**

Esa es la parte fácil… Una vez hemos creado nuestro mobile service debemos irnos a la pestaña Identity y colocar los valores que nos pide según el proveedor de autenticación externo que queramos usar. En este ejemplo usaremos twitter así que nos pide el Api Key y el Api Secret:

[<img title="image" style="border-top: 0px; border-right: 0px; background-image: none; border-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; border-left: 0px; display: inline; padding-right: 0px" border="0" alt="image" src="http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_60A7F377.png" width="244" height="83" />][1]

Estos valores nos lo da twitter cuando creamos una aplicación en <a href="http://apps.twitter.com" target="_blank" rel="noopener noreferrer">apps.twitter.com</a>:

[<img title="image" style="border-top: 0px; border-right: 0px; background-image: none; border-bottom: 0px; padding-top: 0px; padding-left: 0px; border-left: 0px; display: inline; padding-right: 0px" border="0" alt="image" src="http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_301475BF.png" width="244" height="125" />][2]

Otro aspecto a tener en cuenta es que twitter nos pide _la URL de callback,_ esa debe ser la URL de nuestro mobile service **añadiendo el sufijo /signin-twitter**.

Pero bueno… <a href="http://azure.microsoft.com/en-us/documentation/articles/mobile-services-windows-store-dotnet-get-started-users/" target="_blank" rel="noopener noreferrer">todos esos detalles están explicados fenomenalmente en la propia ayuda de Azure</a>.

**Autenticarse usando twitter _a través_ de Mobile Services (WAMS)**

El cliente de Azure Mobile Services realiza un trabajo genial a la hora de gestionar todo el flujo oAuth para autenticar a usuarios usando cualquiera de los proveedores que soporta (Facebook, twitter, Google, una cuenta de Live ID). Si creas una aplicación Windows 8.1 o bien Windows Phone, autenticarse usando un proveedor externo (twitter) a través de Mobile Services es trivial:

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:57324fba-db6c-44b0-95b9-a2304d3e7db9" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
    <div style="background: #ddd; max-height: 300px; overflow: auto">
      <ol start="1" style="background: #1d1d1d; margin: 0 0 0 2em; padding: 0 0 0 5px;">
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">public</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">async</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">void</span><span style="background:#1e1e1e;color:#dcdcdc"> PerformAuth()</span>
        </li>
        <li style="background: #111111">
          <span style="background:#1e1e1e;color:#dcdcdc">{</span>
        </li>
        <li>
              <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#4ec9b0">MobileServiceClient</span><span style="background:#1e1e1e;color:#dcdcdc"> client </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">new</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#4ec9b0">MobileServiceClient</span><span style="background:#1e1e1e;color:#dcdcdc">(</span><span style="background:#1e1e1e;color:#d69d85">"https://beerlover.azure-mobile.net/"</span><span style="background:#1e1e1e;color:#dcdcdc">);</span>
        </li>
        <li style="background: #111111">
              <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">var</span><span style="background:#1e1e1e;color:#dcdcdc"> user </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">await</span><span style="background:#1e1e1e;color:#dcdcdc"> client</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">LoginAsync(</span><span style="background:#1e1e1e;color:#b8d7a3">MobileServiceAuthenticationProvider</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">Twitter);</span>
        </li>
        <li>
              <span style="background:#1e1e1e;color:#dcdcdc">JwtToken </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> user</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">MobileServiceAuthenticationToken;</span>
        </li>
        <li style="background: #111111">
          <span style="background:#1e1e1e;color:#dcdcdc">}</span>
        </li>
      </ol>
    </div></p>
  </div></p>
</div>

Este código se encarga de gestionar todo el flujo OAuth y mostrar la UI correspondiente para que el usuario pueda introducir su login y password de twitter. Al final guarda en una propiedad JwtToken **el token de autenticación retornado por WAMS**.

En este punto finaliza nuestra interacción con mobile services: lo hemos usado para que el usuario se pudiese autenticar de forma fácil con un proveedor externo. Y al final obtenemos un token. **Ese token no es de twitter, ese token es de WAMS** y es un token JWT.

**Json Web Token – JWT**

JWT es un formato que define datos que puede incorporar un token que realmente es un objeto JSON. Es un formato que se está usando bastante ahora y <a href="http://self-issued.info/docs/draft-ietf-oauth-json-web-token.html" target="_blank" rel="noopener noreferrer">tienes su especificación aquí</a>.

Si ejecuto una aplicación que tenga el código anterior y me autentico usando twitter puedo ver como es el token JWT que me devuelve Mobile Services:

[<img title="image" style="border-top: 0px; border-right: 0px; background-image: none; border-bottom: 0px; padding-top: 0px; padding-left: 0px; border-left: 0px; display: inline; padding-right: 0px" border="0" alt="image" src="http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_08708A46.png" width="504" height="154" />][3]

Bueno… es una ristra de carácteres bastante larga, pero básicamente se trata del objeto JSON codificado en Base64.

Para ver el contenido, puedes copiar el token y usar <a href="http://jwt.io" target="_blank" rel="noopener noreferrer">jwt.io</a> para descodificar el token:

[<img title="image" style="border-top: 0px; border-right: 0px; background-image: none; border-bottom: 0px; padding-top: 0px; padding-left: 0px; border-left: 0px; display: inline; padding-right: 0px" border="0" alt="image" src="http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_48CB4F15.png" width="504" height="264" />][4]

Podemos ve
  
r que hay tres colores en la imagen anterior y cada uno se corresponde a una parte. El primer color (verde) es un JSON con el siguiente formato:

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:7b2a3e7e-52d3-410f-b61a-edb0cae9df39" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
    <div style="background: #ddd; max-height: 300px; overflow: auto">
      <ol start="1" style="background: #1d1d1d; margin: 0 0 0 2em; padding: 0 0 0 5px;">
        <li>
          {
        </li>
        <li style="background: #111111">
            <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#d69d85">"typ"</span><span style="background:#1e1e1e;color:#dcdcdc">: </span><span style="background:#1e1e1e;color:#d69d85">"JWT"</span><span style="background:#1e1e1e;color:#dcdcdc">,</span>
        </li>
        <li>
            <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#d69d85">"alg"</span><span style="background:#1e1e1e;color:#dcdcdc">: </span><span style="background:#1e1e1e;color:#d69d85">"HS256"</span>
        </li>
        <li style="background: #111111">
          <span style="background:#1e1e1e;color:#dcdcdc">}</span>
        </li>
      </ol>
    </div></p>
  </div></p>
</div>

Esto se conoce como JWT Envelope y nos indica el formato del token que viene a continuación (JWT) y el algoritmo usado para la firma digital (HS256).

La siguiente parte del token (azul) es realmente el JSON con los datos:

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:4ab8b317-a9a9-4716-a9aa-6840d3712bb3" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
    <div style="background: #ddd; max-height: 300px; overflow: auto">
      <ol start="1" style="background: #1d1d1d; margin: 0 0 0 2.5em; padding: 0 0 0 5px;">
        <li>
          {
        </li>
        <li style="background: #111111">
            <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#d69d85">"iss"</span><span style="background:#1e1e1e;color:#dcdcdc">: </span><span style="background:#1e1e1e;color:#d69d85">"urn:microsoft:windows-azure:zumo"</span><span style="background:#1e1e1e;color:#dcdcdc">,</span>
        </li>
        <li>
            <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#d69d85">"aud"</span><span style="background:#1e1e1e;color:#dcdcdc">: </span><span style="background:#1e1e1e;color:#d69d85">"urn:microsoft:windows-azure:zumo"</span><span style="background:#1e1e1e;color:#dcdcdc">,</span>
        </li>
        <li style="background: #111111">
            <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#d69d85">"nbf"</span><span style="background:#1e1e1e;color:#dcdcdc">: </span><span style="background:#1e1e1e;color:#b5cea8">1418892674</span><span style="background:#1e1e1e;color:#dcdcdc">,</span>
        </li>
        <li>
            <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#d69d85">"exp"</span><span style="background:#1e1e1e;color:#dcdcdc">: </span><span style="background:#1e1e1e;color:#b5cea8">1421484674</span><span style="background:#1e1e1e;color:#dcdcdc">,</span>
        </li>
        <li style="background: #111111">
            <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#d69d85">"urn:microsoft:credentials"</span><span style="background:#1e1e1e;color:#dcdcdc">: </span><span style="background:#1e1e1e;color:#d69d85">"{&#92;"accessToken&#92;":&#92;"84274067-6C7zM6rnbL5VAIF8ARIZXWg6XTZ49x67klxRTAyIU&#92;",&#92;"accessTokenSecret&#92;":&#92;"fGDwXuJg7i8rJqwJFTki5EVbEiLvgx3MlCvunOaNOm81X&#92;"}"</span><span style="background:#1e1e1e;color:#dcdcdc">,</span>
        </li>
        <li>
            <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#d69d85">"uid"</span><span style="background:#1e1e1e;color:#dcdcdc">: </span><span style="background:#1e1e1e;color:#d69d85">"Twitter:84274067"</span><span style="background:#1e1e1e;color:#dcdcdc">,</span>
        </li>
        <li style="background: #111111">
            <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#d69d85">"ver"</span><span style="background:#1e1e1e;color:#dcdcdc">: </span><span style="background:#1e1e1e;color:#d69d85">"2"</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">}</span>
        </li>
      </ol>
    </div></p>
  </div></p>
</div>

Realmente cada uno de esos campos **es un claim** (JWT está basado en claims).

De esos claims nos interesan realmente dos. El claim iss que contiene el _issuer_ o quien ha generado el token y el claim aud que contiene la _audience_ que indica _para quien va dirigido_ el token. En este caso iss nos indica que el token ha sido generado por Mobile Services y el token aud nos indica que el token es para consumo de Mobile Services. Podemos ver más claims como uid donde hay un ID de usuario.

La última parte (en rojo en la imágen) es **la firma digital** del token. En este caso el Envelope nos indica que la firma digital es HS256 (HMAC usando SHA256) y eso nos sirve para comprobar que el token es válido.

**Securizando nuestros servicios WebApi**

La idea para securizar nuestros servicios WebApi es muy simple: A cada petición de nuestros servicios miraremos si se nos pasa el token JWT en la cabcera HTTP Authentication. Si recibimos un token JWT lo parsearemos y comprobaremos la firma digital (podríamos comprobar además todos los claims que queramos). Si la firma digital es válida, el token es válido y la petición se considera que proviene de un usuario de confianza.

Para validar el token JWT **vamos a usar un DelegatingHandler**. Ya <a href="http://geeks.ms/blogs/etomas/archive/2013/02/20/como-hacer-seguros-tus-servicios-webapi.aspx" target="_blank" rel="noopener noreferrer">expliqué en un post anterior sobre como securizar servicios WebApi lo que era un DelegatingHandler</a>.

Para la validación del token JWT **nos vamos a ayudar del paquete System.IdentityModel.Tokens.Jwt** así que lo primero es agregarlo a la solución. **Eso sí agrega la versión 3.0.0.0** mediante el comando:

Install-Package System.IdentityModel.Tokens.Jwt -Version 3.0.0.0

Esa versión es la misma que usa internamente Mobile Services para generar el token JWT.

Así lo primero es crearte una clase (yo la he llamado JwtDelegatingHandler) que derive de DelegatingHandler y redefinir el método SendAsync:

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:2e4ca6fe-7ad4-47a6-a9d5-27f4fddab414" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
    <div style="background: #ddd; max-height: 300px; overflow: auto">
      <ol start="1" style="background: #1d1d1d; margin: 0 0 0 2.5em; padding: 0 0 0 5px;">
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">protected</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">override</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#4ec9b0">Task</span><span style="background:#1e1e1e;color:#b4b4b4"><</span><span style="background:#1e1e1e;color:#4ec9b0">HttpResponseMessage</span><span style="background:#1e1e1e;color:#b4b4b4">></span><span style="background:#1e1e1e;color:#dcdcdc"> SendAsync(</span><span style="background:#1e1e1e;color:#4ec9b0"><br /> HttpRequestMessage</span><span style="background:#1e1e1e;color:#dcdcdc"> request, </span><span style="background:#1e1e1e;color:#4ec9b0">CancellationToken</span><span style="background:#1e1e1e;color:#dcdcdc"> cancellationToken)</span>
        </li>
        <li style="background: #111111">
          <span style="background:#1e1e1e;color:#dcdcdc">{</span>
        </li>
        <li>
              <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">if</span><span style="background:#1e1e1e;color:#dcdcdc"> (request</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">Method </span><span style="background:#1e1e1e;color:#b4b4b4">!=</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#4ec9b0">HttpMethod</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">Options)</span>
        </li>
        <li style="background: #111111">
              <span style="background:#1e1e1e;color:#dcdcdc">{</span>
        </li>
        <li>
                  <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">var</span><span style="background:#1e1e1e;color:#dcdcdc"> tokenstr </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> RetrieveToken(request);</span>
        </li>
        <li style="background: #111111">
                  <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">if</span><span style="background:#1e1e1e;color:#dcdcdc"> (tokenstr </span><span style="background:#1e1e1e;color:#b4b4b4">!=</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">null</span><span style="background:#1e1e1e;color:#dcdcdc">)</span>
        </li>
        <li>
                  <span style="background:#1e1e1e;color:#dcdcdc">{</span>
        </li>
        <li style="background: #111111">
                      <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">var</span><span style="background:#1e1e1e;color:#dcdcdc"> handler </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">new</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#4ec9b0">JwtSecurityTokenHandler</span><span style="background:#1e1e1e;color:#dcdcdc">();</span>
        </li>
        <li>
                      <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">if</span><span style="background:#1e1e1e;color:#dcdcdc"> (handler</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">CanReadToken(tokenstr))</span>
        </li>
        <li style="background: #111111">
                      <span style="background:#1e1e1e;color:#dcdcdc">{</span>
        </li>
        <li>
                          <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">var</span><span style="background:#1e1e1e;color:#dcdcdc"> token </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> handler</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">ReadToken(tokenstr);</span>
        </li>
        <li style="background: #111111">
                          <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">var</span><span style="background:#1e1e1e;color:#dcdcdc"> secret </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> GetSigningKey(MasterKey);</span>
        </li>
        <li>
                          <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">var</span><span style="background:#1e1e1e;color:#dcdcdc"> validationParams </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">new</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#4ec9b0">TokenValidationParameters</span><span style="background:#1e1e1e;color:#dcdcdc">()</span>
        </li>
        <li style="background: #111111">
                          <span style="background:#1e1e1e;color:#dcdcdc">{</span>
        </li>
        <li>
                              <span style="background:#1e1e1e;color:#dcdcdc">SigningToken </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">new</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#4ec9b0">BinarySecretSecurityToken</span><span style="background:#1e1e1e;color:#dcdcdc">(secret),</span>
        </li>
        <li style="background: #111111">
                              <span style="background:#1e1e1e;color:#dcdcdc">AllowedAudience </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#d69d85">"urn:microsoft:windows-azure:zumo"</span><span style="background:#1e1e1e;color:#dcdcdc">,</span>
        </li>
        <li>
                              <span style="background:#1e1e1e;color:#dcdcdc">ValidIssuer </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#d69d85">"urn:microsoft:windows-azure:zumo"</span>
        </li>
        <li style="background: #111111">
                          <span style="background:#1e1e1e;color:#dcdcdc">};</span>
        </li>
        <li>
                          <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">var</span><span style="background:#1e1e1e;color:#dcdcdc"> principal </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> handler</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">ValidateToken(tokenstr, validationParams);</span>
        </li>
        <li style="background: #111111">
                          <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#4ec9b0">Thread</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">CurrentPrincipal </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> principal;</span>
        </li>
        <li>
                          <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">if</span><span style="background:#1e1e1e;color:#dcdcdc"> (</span><span style="background:#1e1e1e;color:#4ec9b0">HttpContext</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">Current </span><span style="background:#1e1e1e;color:#b4b4b4">!=</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">null</span><span style="background:#1e1e1e;color:#dcdcdc">)</span>
        </li>
        <li style="background: #111111">
                          <span style="background:#1e1e1e;color:#dcdcdc">{</span>
        </li>
        <li>
                              <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#4ec9b0">HttpContext</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">Current</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">User </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> principal;</span>
        </li>
        <li style="background: #111111">
                          <span style="background:#1e1e1e;color:#dcdcdc">}</span>
        </li>
        <li>
                      <span style="background:#1e1e
1e;color:#dcdcdc">}</span>
        </li>
        <li style="background: #111111">
                  <span style="background:#1e1e1e;color:#dcdcdc">}</span>
        </li>
        <li>
              <span style="background:#1e1e1e;color:#dcdcdc">}</span>
        </li>
        <li style="background: #111111">
              <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">return</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">base</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">SendAsync(request, cancellationToken);</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">}</span>
        </li>
      </ol>
    </div></p>
  </div></p>
</div>

El método usa la clase JwtSecurityTokenHandler para mirar si el token JWT es válido sintácticamente y luego llama al método ValidateToken que devuelve un ClaimsPrincipal si el token es correcto. Dicho método valida la firma digital y también comprueba que el _issuer_ y el _audence_ del token sean correctos. Es por ello que los establecemos a los valores que coloca WAMS.

Finalmente si no salta ninguna excepción es que el token es correcto por lo que establecemos el ClaimsPrincipal devuelto como Principal del Thread actual de forma que el atributo [Authorize] entenderá que la petición está autenticada.

El método RetrieveToken obtiene el token JWT de la cabecera:

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:ef223cf1-75c0-4f68-badc-0b0ae7faa3aa" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
    <div style="background: #ddd; max-height: 300px; overflow: auto">
      <ol start="1" style="background: #1d1d1d; margin: 0 0 0 2.5em; padding: 0 0 0 5px;">
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">private</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">static</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">string</span><span style="background:#1e1e1e;color:#dcdcdc"> RetrieveToken(</span><span style="background:#1e1e1e;color:#4ec9b0">HttpRequestMessage</span><span style="background:#1e1e1e;color:#dcdcdc"> request)</span>
        </li>
        <li style="background: #111111">
           <span style="background:#1e1e1e;color:#dcdcdc">{</span>
        </li>
        <li>
               <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">string</span><span style="background:#1e1e1e;color:#dcdcdc"> token </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">null</span><span style="background:#1e1e1e;color:#dcdcdc">;</span>
        </li>
        <li style="background: #111111">
               <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#b8d7a3">IEnumerable</span><span style="background:#1e1e1e;color:#b4b4b4"><</span><span style="background:#1e1e1e;color:#569cd6">string</span><span style="background:#1e1e1e;color:#b4b4b4">></span><span style="background:#1e1e1e;color:#dcdcdc"> authzHeadersEnum;</span>
        </li>
        <li>
               <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">bool</span><span style="background:#1e1e1e;color:#dcdcdc"> hasHeader </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> request</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">Headers</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">TryGetValues(</span><span style="background:#1e1e1e;color:#d69d85">"Authorization"</span><span style="background:#1e1e1e;color:#dcdcdc">, </span><span style="background:#1e1e1e;color:#569cd6">out</span><span style="background:#1e1e1e;color:#dcdcdc"> authzHeadersEnum);</span>
        </li>
        <li style="background: #111111">
               <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">if</span><span style="background:#1e1e1e;color:#dcdcdc"> (</span><span style="background:#1e1e1e;color:#b4b4b4">!</span><span style="background:#1e1e1e;color:#dcdcdc">hasHeader)</span>
        </li>
        <li>
               <span style="background:#1e1e1e;color:#dcdcdc">{</span>
        </li>
        <li style="background: #111111">
                   <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">return</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">null</span><span style="background:#1e1e1e;color:#dcdcdc">;</span>
        </li>
        <li>
               <span style="background:#1e1e1e;color:#dcdcdc">}</span>
        </li>
        <li style="background: #111111">
          &nbsp;
        </li>
        <li>
               <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">var</span><span style="background:#1e1e1e;color:#dcdcdc"> authzHeaders </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> authzHeadersEnum</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">ToList();</span>
        </li>
        <li style="background: #111111">
               <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">if</span><span style="background:#1e1e1e;color:#dcdcdc"> (authzHeaders</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">Count </span><span style="background:#1e1e1e;color:#b4b4b4">></span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#b5cea8">1</span><span style="background:#1e1e1e;color:#dcdcdc">)</span>
        </li>
        <li>
               <span style="background:#1e1e1e;color:#dcdcdc">{</span>
        </li>
        <li style="background: #111111">
                   <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">return</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">null</span><span style="background:#1e1e1e;color:#dcdcdc">;</span>
        </li>
        <li>
               <span style="background:#1e1e1e;color:#dcdcdc">}</span>
        </li>
        <li style="background: #111111">
          &nbsp;
        </li>
        <li>
               <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">var</span><span style="background:#1e1e1e;color:#dcdcdc"> bearerToken </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> authzHeaders[</span><span style="background:#1e1e1e;color:#b5cea8"></span><span style="background:#1e1e1e;color:#dcdcdc">];</span>
        </li>
        <li style="background: #111111">
               <span style="background:#1e1e1e;color:#dcdcdc">token </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> bearerToken</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">StartsWith(</span><span style="background:#1e1e1e;color:#d69d85">"Bearer "</span><span style="background:#1e1e1e;color:#dcdcdc">) </span><span style="background:#1e1e1e;color:#b4b4b4">?</span><span style="background:#1e1e1e;color:#dcdcdc"> bearerToken</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">Substring(</span><span style="background:#1e1e1e;color:#b5cea8">7</span><span style="background:#1e1e1e;color:#dcdcdc">) : bearerToken;</span>
        </li>
        <li>
               <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">return</span><span style="background:#1e1e1e;color:#dcdcdc"> token;</span>
        </li>
        <li style="background: #111111">
           <span st
yle="background:#1e1e1e;color:#dcdcdc">}</span>
        </li>
      </ol>
    </div></p>
  </div></p>
</div>

Y nos queda el método más importante el método GetSigninKey. Dicho método obtiene la clave que usó WAMS para firmar el mensaje y es la misma que debemos usar para comprobar la firma:

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:53e0d891-a3df-451a-b2ea-d2005372bf04" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
    <div style="background: #ddd; max-height: 300px; overflow: auto">
      <ol start="1" style="background: #1d1d1d; margin: 0 0 0 2em; padding: 0 0 0 5px;">
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">internal</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">static</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">byte</span><span style="background:#1e1e1e;color:#dcdcdc">[] GetSigningKey(</span><span style="background:#1e1e1e;color:#569cd6">string</span><span style="background:#1e1e1e;color:#dcdcdc"> secretKey)</span>
        </li>
        <li style="background: #111111">
          <span style="background:#1e1e1e;color:#dcdcdc">{</span>
        </li>
        <li>
              <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">var</span><span style="background:#1e1e1e;color:#dcdcdc"> bytes </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">new</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#4ec9b0">UTF8Encoding</span><span style="background:#1e1e1e;color:#dcdcdc">(</span><span style="background:#1e1e1e;color:#569cd6">true</span><span style="background:#1e1e1e;color:#dcdcdc">, </span><span style="background:#1e1e1e;color:#569cd6">true</span><span style="background:#1e1e1e;color:#dcdcdc">)</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">GetBytes(secretKey);</span>
        </li>
        <li style="background: #111111">
              <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">using</span><span style="background:#1e1e1e;color:#dcdcdc"> (</span><span style="background:#1e1e1e;color:#4ec9b0">SHA256Managed</span><span style="background:#1e1e1e;color:#dcdcdc"> managed </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">new</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#4ec9b0">SHA256Managed</span><span style="background:#1e1e1e;color:#dcdcdc">())</span>
        </li>
        <li>
              <span style="background:#1e1e1e;color:#dcdcdc">{</span>
        </li>
        <li style="background: #111111">
                  <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">return</span><span style="background:#1e1e1e;color:#dcdcdc"> managed</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">ComputeHash(bytes);</span>
        </li>
        <li>
              <span style="background:#1e1e1e;color:#dcdcdc">}</span>
        </li>
        <li style="background: #111111">
          <span style="background:#1e1e1e;color:#dcdcdc">}</span>
        </li>
      </ol>
    </div></p>
  </div></p>
</div>

El parámetro secretKey es la “Master Key” de WAMS que puedes ver en el portal de Azure en la opción de “Manage Keys”:

[<img title="image" style="border-top: 0px; border-right: 0px; background-image: none; border-bottom: 0px; padding-top: 0px; padding-left: 0px; border-left: 0px; display: inline; padding-right: 0px" border="0" alt="image" src="http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_454F4EDC.png" width="244" height="172" />][5]

> **Nota:** Si buscas por Internet verás que en muchos sitios agregan la cadena JWTSig a la Master Key. No lo hagas. No sé si en versiones anteriores de WAMS era necesario, pero ahora no lo es.

Con esto ya puedes usar [Authorize] para proteger tus servicios WebApi y que solo sean válidos para aquellas peticiones que tengan un token JWT que proviene de Windows Azure Mobile Services.

**Usando System.IdentityModel.Tokens.Jwt 4.0.1**

Hemos visto el código necesario para validar el token JWT de WAMS usando la versión 3.0.0.0 de System.IdentityModel.Tokens.Jwt que es la que usa internamente WAMS. Pero la última versión de este paquete (y la que os instalará NuGet si no indicáis versión) es, a la hora de escribir este post, la 4.0.1.

El código **no es compatible** ya que hay cambios en la clase JwtSecurityTokenHandler. Si usáis la versión 4.0.1 debéis usar el siguiente código en el DelegatingHandler:

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:9d207708-3b21-4ba6-a23c-97ccc6164c2a" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
    <div style="background: #ddd; max-height: 300px; overflow: auto">
      <ol start="1" style="background: #1d1d1d; margin: 0 0 0 2em; padding: 0 0 0 5px;">
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">var</span><span style="background:#1e1e1e;color:#dcdcdc"> validationParams </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">new</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#4ec9b0">TokenValidationParameters</span><span style="background:#1e1e1e;color:#dcdcdc">()</span>
        </li>
        <li style="background: #111111">
          <span style="background:#1e1e1e;color:#dcdcdc">{</span>
        </li>
        <li>
              <span style="background:#1e1e1e;color:#dcdcdc">IssuerSigningToken </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">new</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#4ec9b0">BinarySecretSecurityToken</span><span style="background:#1e1e1e;color:#dcdcdc">(secret),</span>
        </li>
        <li style="background: #111111">
              <span style="background:#1e1e1e;color:#dcdcdc">ValidAudience </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#d69d85">"urn:microsoft:windows-azure:zumo"</span><span style="background:#1e1e1e;color:#dcdcdc">,</span>
        </li>
        <li>
              <span style="background:#1e1e1e;color:#dcdcdc">ValidIssuer </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#d69d85">"urn:microsoft:windows-azure:zumo"</span>
        </li>
        <li style="background: #111111">
          <span style="background:#1e1e1e;color:#dcdcdc">};</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#4ec9b0">SecurityToken</span><span style="background:#1e1e1e;color:#dcdcdc"> outToken;</span>
        </li>
        <li style="background: #111111">
          <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">var</span><span style="background:#1e1e1e;color:#dcdcdc"> principal </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e
1e1e;color:#dcdcdc"> handler</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">ValidateToken(tokenstr, validationParams, </span><span style="background:#1e1e1e;color:#569cd6">out</span><span style="background:#1e1e1e;color:#dcdcdc"> outToken);</span>
        </li>
      </ol>
    </div></p>
  </div></p>
</div>

Este código es equivalente al anterior, podéis ver que los cambios básicos son nombres de propiedades y la firma de ValidateToken.

**Usando Middleware OWIN**

Comprobar que System.IdentityModel.Tokens.Jwt en su versión 4.0.1 era compatible con las firmas generadas por WAMS para los tokens JWT supone que podemos usar el middleware OWIN para validar el token JWT y olvidarnos del DelegatingHandler. Si la versión 4.0.1 no hubiese sido compatible (y algo de eso he leído en Internet, al menos en la versión 4.0.0) eso no sería posible ya que el middleware de OWIN depende de dicha versión (realmente la 4.0.0) para la validación de los tokens JWT.

Vale, para agregar el middleware OWIN basta con instalar el paquete Microsoft.Owin.Security.Jwt:

install-package Microsoft.Owin.Security.Jwt

Eso nos instalará las dependencias OWIN que tenemos, pero si no teníamos nada de OWIN instalada deberemos agregar el hosting de OWIN manualmente. Si después de este install-package no tenemos el paquete Microsoft.Owin.Host.SystemWeb deberemos instalarlo. Este paquete es el responsable de integrar el pipeline de OWIN y que se ejecute la clase de Owin Startup. El siguiente paso será crear una clase Startup:

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:716ef695-23bc-4169-add3-dd6abecb0801" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
    <div style="background: #ddd; max-height: 300px; overflow: auto">
      <ol start="1" style="background: #1d1d1d; margin: 0 0 0 2.5em; padding: 0 0 0 5px;">
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">[</span><span style="background:#1e1e1e;color:#569cd6">assembly</span><span style="background:#1e1e1e;color:#dcdcdc">: </span><span style="background:#1e1e1e;color:#4ec9b0">OwinStartup</span><span style="background:#1e1e1e;color:#dcdcdc">(</span><span style="background:#1e1e1e;color:#569cd6">typeof</span><span style="background:#1e1e1e;color:#dcdcdc">(Beerlover</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">Server</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#4ec9b0">Startup</span><span style="background:#1e1e1e;color:#dcdcdc">))]</span>
        </li>
        <li style="background: #111111">
          &nbsp;
        </li>
        <li>
          <span style="background:#1e1e1e;color:#569cd6">namespace</span><span style="background:#1e1e1e;color:#dcdcdc"> Beerlover</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">Server</span>
        </li>
        <li style="background: #111111">
          <span style="background:#1e1e1e;color:#dcdcdc">{</span>
        </li>
        <li>
              <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">public</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">class</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#4ec9b0">Startup</span>
        </li>
        <li style="background: #111111">
              <span style="background:#1e1e1e;color:#dcdcdc">{</span>
        </li>
        <li>
                  <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">public</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">void</span><span style="background:#1e1e1e;color:#dcdcdc"> Configuration(</span><span style="background:#1e1e1e;color:#b8d7a3">IAppBuilder</span><span style="background:#1e1e1e;color:#dcdcdc"> app)</span>
        </li>
        <li style="background: #111111">
                  <span style="background:#1e1e1e;color:#dcdcdc">{</span>
        </li>
        <li>
          &nbsp;
        </li>
        <li style="background: #111111">
                      <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">var</span><span style="background:#1e1e1e;color:#dcdcdc"> issuer </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#d69d85">"urn:microsoft:windows-azure:zumo"</span><span style="background:#1e1e1e;color:#dcdcdc">;</span>
        </li>
        <li>
                      <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">var</span><span style="background:#1e1e1e;color:#dcdcdc"> audience </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#d69d85">"urn:microsoft:windows-azure:zumo"</span><span style="background:#1e1e1e;color:#dcdcdc">;</span>
        </li>
        <li style="background: #111111">
                      <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">var</span><span style="background:#1e1e1e;color:#dcdcdc"> secret </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#4ec9b0">WebConfigurationManager</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">AppSettings[</span><span style="background:#1e1e1e;color:#d69d85">"ClientSecret"</span><span style="background:#1e1e1e;color:#dcdcdc">];</span>
        </li>
        <li>
          &nbsp;
        </li>
        <li style="background: #111111">
                      <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">var</span><span style="background:#1e1e1e;color:#dcdcdc"> signkey </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> GetSigningKey(secret);</span>
        </li>
        <li>
                      <span style="background:#1e1e1e;color:#dcdcdc">app</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">UseJwtBearerAuthentication(</span>
        </li>
        <li style="background: #111111">
                          <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">new</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#4ec9b0">JwtBearerAuthenticationOptions</span>
        </li>
        <li>
                          <span style="background:#1e1e1e;color:#dcdcdc">{</span>
        </li>
        <li style="background: #111111">
                              <span style="background:#1e1e1e;color:#dcdcdc">AuthenticationMode </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#b8d7a3">AuthenticationMode</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">Active,</span>
        </li>
        <li>
                              <span style="background:#1e1e1e;color:#dcdcdc">AllowedAudiences </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">new</span><span style="background:#1e1e1e;color:#dcdcdc">[] { audience },</span>
        </li>
        <li style="background: #111111">
                              <span style="background:#1e1e1e;color:#dcdcdc">IssuerSecurityTokenProviders </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">new</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#b8d7a3">IIssuerSe<br /> curityTokenProvider</span><span style="background:#1e1e1e;color:#dcdcdc">[]</span>
        </li>
        <li>
                              <span style="background:#1e1e1e;color:#dcdcdc">{</span>
        </li>
        <li style="background: #111111">
                                  <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">new</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#4ec9b0">SymmetricKeyIssuerSecurityTokenProvider</span><span style="background:#1e1e1e;color:#dcdcdc">(issuer, signkey)</span>
        </li>
        <li>
                              <span style="background:#1e1e1e;color:#dcdcdc">},</span>
        </li>
        <li style="background: #111111">
          &nbsp;
        </li>
        <li>
                          <span style="background:#1e1e1e;color:#dcdcdc">});</span>
        </li>
        <li style="background: #111111">
                  <span style="background:#1e1e1e;color:#dcdcdc">}</span>
        </li>
        <li>
          &nbsp;
        </li>
        <li style="background: #111111">
                  <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">private</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">byte</span><span style="background:#1e1e1e;color:#dcdcdc">[] GetSigningKey(</span><span style="background:#1e1e1e;color:#569cd6">string</span><span style="background:#1e1e1e;color:#dcdcdc"> secret)</span>
        </li>
        <li>
                  <span style="background:#1e1e1e;color:#dcdcdc">{</span>
        </li>
        <li style="background: #111111">
                      <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">var</span><span style="background:#1e1e1e;color:#dcdcdc"> bytes </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">new</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#4ec9b0">UTF8Encoding</span><span style="background:#1e1e1e;color:#dcdcdc">(</span><span style="background:#1e1e1e;color:#569cd6">true</span><span style="background:#1e1e1e;color:#dcdcdc">, </span><span style="background:#1e1e1e;color:#569cd6">true</span><span style="background:#1e1e1e;color:#dcdcdc">)</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">GetBytes(secret);</span>
        </li>
        <li>
                      <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">using</span><span style="background:#1e1e1e;color:#dcdcdc"> (</span><span style="background:#1e1e1e;color:#4ec9b0">SHA256Managed</span><span style="background:#1e1e1e;color:#dcdcdc"> managed </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">new</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#4ec9b0">SHA256Managed</span><span style="background:#1e1e1e;color:#dcdcdc">())</span>
        </li>
        <li style="background: #111111">
                      <span style="background:#1e1e1e;color:#dcdcdc">{</span>
        </li>
        <li>
                          <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">return</span><span style="background:#1e1e1e;color:#dcdcdc"> managed</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">ComputeHash(bytes);</span>
        </li>
        <li style="background: #111111">
                      <span style="background:#1e1e1e;color:#dcdcdc">}</span>
        </li>
        <li>
                  <span style="background:#1e1e1e;color:#dcdcdc">}</span>
        </li>
        <li style="background: #111111">
              <span style="background:#1e1e1e;color:#dcdcdc">}</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">}</span>
        </li>
      </ol>
    </div></p>
  </div></p>
</div>

Dicha clase configura el middleware de OWIN para validar tokens JWT (a través del método UseJwtBearerAuthentication). De esta manera delegamos en OWIN la validación de los tokens JWT **y ya no es necesario hacer nada en WebApi,&#160; es decir ya no debemos usar el DelegatingHandler**. Por supuesto debemos seguir usando [Authorize] para indicar que servicios deben ser llamados de forma segura (con token JWT).

La solución usando OWIN **es la recomendada ya que la validación de los tokens JWT se hace _antes_ de que la petición llegue siquiera a WebApi** y por lo tanto te permite integrarlo en otros middlewares (p. ej. si usas Nancy con OWIN este mismo código te sirve, mientras que el DelegatingHandler es algo propio de WebApi).

**Enviando el token desde el cliente**

Y simplemente por completitud del post, el código para enviar el token JWT desde el cliente sería el siguiente:

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:47ba5c11-4559-4ea0-9005-ee26193d9e10" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
    <div style="background: #ddd; max-height: 300px; overflow: auto">
      <ol start="1" style="background: #1d1d1d; margin: 0 0 0 2em; padding: 0 0 0 5px;">
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">protected</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">void</span><span style="background:#1e1e1e;color:#dcdcdc"> AddJwtToken(</span><span style="background:#1e1e1e;color:#4ec9b0">HttpClient</span><span style="background:#1e1e1e;color:#dcdcdc"> client)</span>
        </li>
        <li style="background: #111111">
          <span style="background:#1e1e1e;color:#dcdcdc">{</span>
        </li>
        <li>
              <span style="background:#1e1e1e;color:#dcdcdc">client</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">DefaultRequestHeaders</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">Authorization </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">new</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#4ec9b0">AuthenticationHeaderValue</span><span style="background:#1e1e1e;color:#dcdcdc">(</span><span style="background:#1e1e1e;color:#d69d85">"Bearer"</span><span style="background:#1e1e1e;color:#dcdcdc">, </span><span style="background:#1e1e1e;color:#4ec9b0">TwitterAuthService</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">Instance</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">JwtToken);</span>
        </li>
        <li style="background: #111111">
          <span style="background:#1e1e1e;color:#dcdcdc">}</span>
        </li>
      </ol>
    </div></p>
  </div></p>
</div>

Donde TwitterAuthService.Instance.JwtToken contiene el token devuelto por WAMS.

La posibilidad de delegar en WAMS todo el flujo oauth es muy cómoda y ello no nos impide que nuestra propia WebApi que puede estar en Azure (en un Website) o on-premise esté protegida a través del token JWT que emite WAMS. De esa manera la aplicación cliente tiene un solo token que usa tanto para acceder a los servicios WAMS (si los usase) como al resto de la API WebAPi que ni tiene que estar en Azure.

Espero que este post os sea de utilidad.

 [1]: http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_31C02031.png
 [2]: http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_752D42F5.png
 [3]: http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_58B2E1C6.png
 [4]: http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_19E37BCF.png
 [5]: http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_4A320298.png