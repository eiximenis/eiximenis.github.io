---
title: Desplegar una aplicación ASP.NET Core 2.1 con https en Docker

author: eiximenis

date: 2018-07-02T15:49:12+00:00
geeks_url: /?p=2136
geeks_ms_views:
  - 2461
categories:
  - asp.net core
  - docker

---
Una de las novedades de ASP.NET Core 2.1 es que redirige automáticamente todo el tráfico de http a https y, además, fuerza el uso de HSTS. Sobre https nada que decir, seguro que todos lo conocéis.
  
Sobre HSTS simplemente comentar que es un protocolo mediante el cual el servidor informa a los _user-agents_ de que solo acepta conexiones seguras, es decir que deben usar HTTPS para acceder a sus recursos. A pesar de que HSTS tiene sus limitaciones (y que para evitarlas sería necesaria un protocolo a nivel de DNS y no de servidores) su uso es una práctica imprescindible en seguridad web. De hecho **hoy en día NO HAY EXCUSA PARA NO USAR HTTPS NI HSTS **en cualquier sitio web. Da absolutamente igual lo que haga. No usar https ni hsts es una negligencia que debería avergonzar a cualquiera.
  
<!--more-->


  
Hasta ahora en ASP.NET Core 2.0 no teníamos una manera &#8220;incorporada&#8221; de forzar el usto de HTTPS ni de HSTS, aunque tampoco era algo complejo de conseguir. Pero está bien que lo hayan incorporado en el framework: ahora tenemos una forma canónica de hacerlo. La plantilla por defecto de VS (y de _dotnet create_) nos crea el siguiente código en el método _Configure_:

<pre class="EnlighterJSRAW" data-enlighter-language="csharp">if (env.IsDevelopment())
{
    app.UseDeveloperExceptionPage();
}
else
{
    app.UseHsts();
}
app.UseHttpsRedirection();</pre>

Tenemos dos _middlewares _nuevos:

  * _UseHsts_: Habilita el uso de HSTS, que consiste básicamente en mandar una cabecera
  * _UseHttpsRedirection_: Este es el middleware que convierte todas las direcciones HTTP en 301 con HTTPS

Claro que cuando ponemos en marcha una aplicación que requiere https, entramos en el fascinante mundo de los certificados TLS: necesitamos un certificado para que el navegador acepte cargar la página y este certificado debe estar firmado por una autoridad certificadora (CA) reconocida. La primera vez que vayas a ejecutar desde VS recibirás el siguiente mensaje:
  
[<img class="alignnone size-medium wp-image-2137" src="https://geeks.ms/etomas/wp-content/uploads/sites/154/2018/07/iis-cert-vs-300x122.png" alt="Mensaje de VS pidiendo instalar el certificado" width="300" height="122" />][1]
  
Si le das a &#8220;Yes&#8221; pues eso: te instalará un certificado de desarrollo para que el navegador acepte cargar los datos. Si le das a &#8220;No&#8221; pues no lo instala y tu navegador te devolverá un  ERR\_CERT\_AUTHORITY_INVALID y te saldrá la típica página de error de &#8220;sitio no seguro&#8221;.
  
Si en lugar de VS usas la línea de comandos al hacer un &#8220;dotnet run&#8221; no ocurrirá nada especial, pero si accedes a la aplicación, recibirás el mismo error de certificado **a pesar de haberlo instalado antes.** La razón es que el certificado que has instalado al usar VS es el de IIS Express, no el de Kestrel. Recuerda que una aplicación ASP.NET Core puedes ejecutarla desde IIS Express o bien desde Kestrel. Para instalar el certificado de Kestrel debes usar el siguiente comando:

<pre class="EnlighterJSRAW" data-enlighter-language="shell">dotnet dev-certs https --trust</pre>

A partir de ahora puedes navegar usando https sin problemas.
  
**Instalando el certificado de desarrollo en Docker**
  
Si quieres usar Docker para desarrollo, vas a tener un problema al usar https y es que debes instalar el certificado en la imagen. Usar un _RUN dotnet dev-certs https &#8211;trust_ en el Dockerfile no es una opción, porque ese comando pertenece al SDK de netcore y por lo tanto no está disponible en una imagen con solo el runtime (además del pequeño detalle de que, de momento, no está disponible para Linux).
  
Para ver lo que necesitamos, vamos a partir del Dockerfile típico de _multistage_ que genera el propio Visual Studio:

<pre class="EnlighterJSRAW" data-enlighter-language="json">FROM microsoft/dotnet:2.1-aspnetcore-runtime AS base
WORKDIR /app
EXPOSE 80
EXPOSE 443
FROM microsoft/dotnet:2.1-sdk AS build
WORKDIR /src
COPY Webapi8/Webapi8.csproj Webapi8/
RUN dotnet restore Webapi8/Webapi8.csproj
COPY . .
WORKDIR /src/Webapi8
RUN dotnet build Webapi8.csproj -c Release -o /app
FROM build AS publish
RUN dotnet publish Webapi8.csproj -c Release -o /app
FROM base AS final
WORKDIR /app
COPY --from=publish /app .
ENTRYPOINT ["dotnet", "Webapi8.dll"]
</pre>

Si lo ponemos en marcha, lo primero que veremos es que **nos da un error al redirigir a https:**
  
[<img class="alignnone size-medium wp-image-2139" src="https://geeks.ms/etomas/wp-content/uploads/sites/154/2018/07/no-redirect-port-300x107.png" alt="Error: Failed to determine the https port for redirect." width="300" height="107" />][2]
  
Este error (_Failed to determine the https port for redirect_) es debido a que el middleware de redirección **no sabe cual es el puerto de HTTPs, **ya que dicho puerto se establece en una variable de entorno llamada _ASPNETCORE\_HTTPS\_PORT_, por lo tanto debemos establecer un valor a esa variable, usando p. ej.:

<pre class="EnlighterJSRAW" data-enlighter-language="shell">docker run -p5000:80 -p:5001:443 -e ASPNETCORE_HTTPS_PORT=5001 webapi8</pre>

Si ahora navegas a http://localhost:5000/api/values el navegador será redirigido a https://localhost:5001/api/values, pero nadie está escuchando por allí. Kestrel no ha levantado el puerto SSL. Para que lo levante debemos indicárselo mediante la variable de entorno ASPNETCORE_URLS:

<pre class="EnlighterJSRAW" data-enlighter-language="shell">docker run -p5000:80 -p:5001:443 -e ASPNETCORE_HTTPS_PORT=5001 -e ASPNETCORE_URLS=https://+;http://+ webapi8</pre>

Y ahora sí que... ¡patapam! Al poner en marcha la imagen, recibiremos un error de que Kestrel no puede levantar el puerto https porque no tiene el certificado:

<pre class="EnlighterJSRAW" data-enlighter-language="raw">crit: Microsoft.AspNetCore.Server.Kestrel[0]
      Unable to start Kestrel.
System.InvalidOperationException: Unable to configure HTTPS endpoint. No server certificate was specified, and the default developer certificate could not be found.</pre>

Ahora nos falta compartir el certificado de desarrollo que tenemos en nuestra máquina, con la imagen de Docker. Para ello lo exportamos a pfx, usando el siguiente comando desde nuestra máquina (usa la ruta que tu quieras):

<pre class="EnlighterJSRAW" data-enlighter-language="null">dotnet dev-certs https -v -ep d:\temp\cert-aspnetcore.pfx -p ufo</pre>

**Importante:** DEBES usar -p para establecer una contraseña para el fichero pfx, en caso contrario no te funcionará, ya que la clave privada no se incorporará en el fichero (y Kestrel te dará un error (_System.NotSupportedException: The server mode SSL must use a certificate with the associated private key_) al acceder vía https).
  
Con eso exportamos el certificado de desarrollo de nuestra máquina (en mi caso a d:\temp\cert-aspnetcore.pfx). El siguiente paso **es indicarle al Kestrel que se ejecuta en Docker donde está dicho certificado**, usando las variable de entorno Kestrel\_\_Certificates\_\_Default\_\_Path y Kestrel\_\_Certificates\_\_Default\_\_Password y por supuesto compartiendo el fichero pfx mediante un _bind mount_:

<pre class="EnlighterJSRAW" data-enlighter-language="shell">docker run -p5000:80 -p:5001:443 -e ASPNETCORE_HTTPS_PORT=5001 -e ASPNETCORE_URLS=https://+;http://+ -e Kestrel__Certificates__Default__Path=/root/.dotnet/https/cert-aspnetcore.pfx -e Kestrel__Certificates__Default__Password=ufo -v D:\temp\:/root/.dotnet/https webapi8</pre>

Con eso Kestrel busca el certificado en el directorio indicado (/root/.dotnet/https) donde hay el pfx que tenemos en nuestra máquina de desarrollo.
  
Ahora nuestra aplicación se levanta correctamente en Docker:
  
[<img class="alignnone size-medium wp-image-2140" src="https://geeks.ms/etomas/wp-content/uploads/sites/154/2018/07/kestrel-docker-https-300x70.png" alt="Kestrel levantado usando https" width="300" height="70" />][3]
  
Y si ahora accedes a http://localhost:5000/api/values, serás redirigido a **https**://localhost:5001/api/values y no deberías recibir ningún error.
  
Por supuesto, es **mucho mejor usar un fichero _compose_** antes que esa llamada a &#8220;docker run&#8221; con tantos parámetros -e y -v, pero bueno, eso ya son detalles menores 😉 Lo importante es que te quedes con la idea:

  1. Generamos un certificado en nuestra máquina usando _dotnet dev-certs_.
  2. Lo exportamos a pfx
  3. Compartimos mediante _bind mount_ dicho certificado con la imagen de Docker
  4. Usamos el bloque de configuración Kestrel:Certificates:Default para indicarle donde está el certificado y su password

En el punto (4) he usado variables de entorno, pero cualquier cosa que se mapee al IConfiguration será válido (p. ej. tenerlo en un appsettings.json, etc).
  
Y nada más, espero que te haya resultado interesante 🙂
  
&nbsp;

 [1]: https://geeks.ms/etomas/wp-content/uploads/sites/154/2018/07/iis-cert-vs.png
 [2]: https://geeks.ms/etomas/wp-content/uploads/sites/154/2018/07/no-redirect-port.png
 [3]: https://geeks.ms/etomas/wp-content/uploads/sites/154/2018/07/kestrel-docker-https.png