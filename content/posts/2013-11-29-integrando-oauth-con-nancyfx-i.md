---
title: Integrando oAuth con NancyFx–(i)

author: eiximenis

date: 2013-11-29T16:01:18+00:00
geeks_url: /?p=1658
geeks_visits:
  - 1323
geeks_ms_views:
  - 1226
categories:
  - Uncategorized

---
Buenas! El objetivo de este post es explicar la solución a la que he llegado para integrar autenticación con oAuth en un sitio web desarrollado con NancyFx. En este primer post veremos como hacerlo “de la forma clásica” pero en otro siguiente nos aprovecharemos de los componentes de autenticación de Katana.

Iré hablando en este blog más sobre OWIN, Katana y también sobre NancyFx, pero por el momento algunas definiciones rápidas:

  * OWIN: Especificación que indica como deben comunicarse los servidores web, el middleware web y las aplicaciones web en .NET.
  * Katana: Implementación de varios componentes (hosts, servidores, middlewares varios) OWIN por parte de Microsoft.
  * <a href="http://nancyfx.org" target="_blank" rel="noopener noreferrer">NancyFx</a>: Un framework basado en el patrón MVC para desarrollar aplicaciones web y APIs REST. Por decirlo de algún modo NancyFx “compite” con ASP.NET MVC y con WebApi a la vez.

Para saber más de OWIN hay [un par de posts en mi blog][1] pero también una [serie fenomenal del Maestro][2]. Échales un ojo.

**Creando una aplicación Nancy ejecutándose en IIS** 

Vamos a crear una aplicación web, y vamos a usar NancyFx en lugar de ASP.NET, pero usando IIS (IISExpress).

NancyFx viene en dos “sabores”: puede ser un HttpHandler de ASP.NET (así instalas y configuras NancyFx usando el web.config), pero también existe como componente OWIN, lo que permite su uso dentro de cualquier entorno OWIN.

Para usar NancyFx como HttpHandler de ASP.NET debes instalar el paquete <a href="http://www.nuget.org/packages/Nancy.Hosting.Aspnet/" target="_blank" rel="noopener noreferrer">Nancy.Hosting.Aspnet</a> que es el que contiene todas las referencias a ASP.NET (NancyFx es **totalmente independiente de ASP.NET**). Ahora vamos a crear una aplicación web ASP.NET, así que lo lógico parece ser usar NancyFx como HttpHandler y listos. En muchos casos puede ser lo más lógico, pero **no es** lo que vamos a hacer. No vamos a usar NancyFx como HttpHandler de ASP.NET. En su lugar lo vamos a utilizar como componente OWIN y, para ello, nos aprovecharemos de un componente de Katana que permite usar componentes OWIN dentro del pipeline de ASP.NET (en terminología OWIN diríamos que este componente de Katana implementa un Host y un servidor Web OWIN). Esto parece dar muchas vueltas, y en muchos casos así puede ser, pero tiene las siguientes ventajas:

  1. Dado que nuestra aplicación será OWIN en cualquier momento podremos abandonar el cómodo regazo de IIS e irnos a cualquier otro host o servidor OWIN.
  2. Incluso aunque no tengamos pensado divorciarnos de IIS, podremos utilizar cualquier componente OWIN que haya por ahí 😉

Como digo, para poder utilizar NancyFx como componente OWIN pero ejecutándose en un pipeline de ASP.NET necesitamos la ayuda de Katana, en concreto del componente <a href="https://www.nuget.org/packages/Microsoft.Owin.Host.SystemWeb/" target="_blank" rel="noopener noreferrer">Microsoft.Owin.Host.SystemWeb</a>.

Así, los pasos para usar NancyFx como componente OWIN dentro del pipeline de ASP.NET son los siguientes. Antes que nada crea un proyecto de tipo ASP.NET y selecciona el template “Empty”:

[<img title="image" style="border-top: 0px; border-right: 0px; background-image: none; border-bottom: 0px; float: left; padding-top: 0px; padding-left: 0px; margin: 2px 2px 5px; border-left: 0px; display: inline; padding-right: 0px" border="0" alt="image" src="http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_158EC279.png" width="504" align="left" height="355" />][3]

Luego instala los siguientes paquetes de NuGet:

  * Nancy (el core de NancyFx)
  * Nancy.Owin (Para usar Nancy como módulo OWIN)
  * Microsoft.Owin.Host.SystemWeb (Para usar módulos OWIN en el pipeline de ASP.NET)

Con esto ya tenemos el esqueleto necesario.

El siguiente paso es crear la clase de inicialización de OWIN. Para ello agrega una clase normal y llámala Startup:

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:03e7a601-b411-41bd-858b-b66a12b77691" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
    <div style="background: #000080; color: #fff; font-family: Verdana, Tahoma, Arial, sans-serif; font-weight: bold; padding: 2px 5px">
      Clase de inicializacion OWIN
    </div>
    
    <div style="background: #ddd; max-height: 300px; overflow: auto">
      <ol start="1" style="background: #1e1e1e; margin: 0 0 0 2em; padding: 0 0 0 5px; white-space: nowrap">
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">    </span><span style="background:#1e1e1e;color:#569cd6">public</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">class</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#4ec9b0">Startup</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">    {</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">        </span><span style="background:#1e1e1e;color:#569cd6">public</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">void</span><span style="background:#1e1e1e;color:#dcdcdc"> Configuration(</span><span style="background:#1e1e1e;color:#b8d7a3">IAppBuilder</span><span style="background:#1e1e1e;color:#dcdcdc"> app)</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">        {</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">            app</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">UseNancy();</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">        }</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">    }</span>
        </li>
      </ol>
    </div></p>
  </div></p>
</div>

Es importante que la clase se llame Startup. Esto es una convención que sigue Katana (pues quien inicializa los módulos OWIN es Katana a través del paquete Microsoft.Owin.Host.SystemWeb). Si nuestra clase tiene otro nombre recibirás un error como el siguiente:

[<img title="image" style="border-top: 0px; border-right: 0px; background-image: none; border-bottom: 0px; padding-top: 0px; padding-left: 0px; border-left: 0px; display: inline; padding-right: 0px" border="0" alt="image" src="http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_204EAA7F.png" width="504" height="87" />][4]

Una solución es o bien renombrar la clase para que se llame Startup o en el caso de que no quieras hacerlo usar el atributo OwinStartupAttribute:

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:ef5361c2-6dd6-4e49-90b8-a78ac9256c49" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
    <div style="background: #ddd; overflow: auto">
      <ol start="1" style="background: #1e1e1e; margin: 0 0 0 2em; padding: 0 0 0 5px; white-space: nowrap">
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">[</span><span style="background:#1e1e1e;color:#569cd6">assembly</span><span style="background:#1e1e1e;color:#dcdcdc">: </span><span style="background:
#1e1e1e;color:#4ec9b0">OwinStartup</span><span style="background:#1e1e1e;color:#dcdcdc">(</span><span style="background:#1e1e1e;color:#569cd6">typeof</span><span style="background:#1e1e1e;color:#dcdcdc">(</span><span style="background:#1e1e1e;color:#4ec9b0">MyCustomStartup</span><span style="background:#1e1e1e;color:#dcdcdc">))]</span>
        </li>
      </ol>
    </div></p>
  </div></p>
</div>

Si ahora ejecutas la aplicación deberías ver algo parecido a lo siguiente:

[<img title="image" style="border-top: 0px; border-right: 0px; background-image: none; border-bottom: 0px; padding-top: 0px; padding-left: 0px; border-left: 0px; display: inline; padding-right: 0px" border="0" alt="image" src="http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_245B8902.png" width="244" height="131" />][5]

Eso significa que Nancy está funcionando correctamente (este 404 ha sido servido por Nancy). ¡Felicidades! Ya tienes a Nancy corriendo bajo IIS.

Vamos ahora a crear un módulo de Nancy (más o menos equivalente a un controlador de ASP.NET MVC), para gestionar la llamada a la URL / y mostrar una vista con solo el enlace “Login with twitter”. Añade una clase a tu proyecto con el siguiente código:

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:10301434-e23d-4b16-88ed-8e7ded4866c1" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
    <div style="background: #000080; color: #fff; font-family: Verdana, Tahoma, Arial, sans-serif; font-weight: bold; padding: 2px 5px">
      Nancy MainModule
    </div>
    
    <div style="background: #ddd; max-height: 300px; overflow: auto">
      <ol start="1" style="background: #1e1e1e; margin: 0 0 0 2em; padding: 0 0 0 5px; white-space: nowrap">
        <li>
          <span style="background:#1e1e1e;color:#569cd6">public</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">class</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#4ec9b0">MainModule</span><span style="background:#1e1e1e;color:#dcdcdc"> : </span><span style="background:#1e1e1e;color:#4ec9b0">NancyModule</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">{</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">    </span><span style="background:#1e1e1e;color:#569cd6">public</span><span style="background:#1e1e1e;color:#dcdcdc"> MainModule()</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">    {</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">        Get[</span><span style="background:#1e1e1e;color:#d69d85">"/"</span><span style="background:#1e1e1e;color:#dcdcdc">] </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> _ </span><span style="background:#1e1e1e;color:#b4b4b4">=></span><span style="background:#1e1e1e;color:#dcdcdc"> View[</span><span style="background:#1e1e1e;color:#d69d85">"main.html"</span><span style="background:#1e1e1e;color:#dcdcdc">];</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">    }</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">}</span>
        </li>
      </ol>
    </div></p>
  </div></p>
</div>

Con esto has creado un módulo de NancyFx y has enrutado todas las peticiones que vayan a la URL “/” para que muestren la vista main.html. Añade pues un archivo main.html (no es necesario que esté en ninguna carpeta _Views_ ni nada) que muestre un enlace a la URL “/authentication/redirect/twitter”.

Si ahora ejecutas el proyecto deberías se tendría que ver el contenido del fichero main.html. Y si pulsas sobre el enlace tienes que recibir el 404 de Nancy.

Perfecto! Ya tenemos el esqueleto base de la aplicación. Ahora vayamos a integrar la autenticación por twitter.

**Integrando oAuth con NancyFx a la manera clásica** 

> **Nota:** El contenido de este apartado presupone que tienes creada una aplicación en twitter y que por lo tanto tienes un consumer key y un consumer secret. También debes configurar la aplicación twitter para que la URL de callback sea [/authentication/authenticatecallback">http://localhost:<puerto>/authentication/authenticatecallback][6]
> 
> **Nota 2:** Twitter (y otros proveedores oAuth) no dejan dar de alta aplicaciones cuyo callback sea una dirección de localhost. En este caso lo más rápido es usar el dominio xxx.localtest.me (usa cualquier valor para xxx). El dominio localtest.me está especialmente pensado para estos casos: cualquier subdominio en localtest.me resuelve a 127.0.0.1 😉

NancyFx es un framework muy modular, y no tiene incorporado el concepto de autenticación o autorización. Eso significa que no hay por defecto ninguna API ni nada que nos diga si el usuario está autenticado o bien poder autenticarlo. Todo eso se deja a implementaciones “externas” al _core_ de NancyFx.

P. ej. si queremos autenticar nuestra aplicación basándonos en cookies (lo que en el mundo ASP.NET conocemos como autenticación por forms) debemos instalar el paquete <a href="https://www.nuget.org/packages/Nancy.Authentication.Forms" target="_blank" rel="noopener noreferrer">Nancy.Authentication.Forms</a>. Hay otros paquetes para otros tipos de autenticación (como puede ser <a href="https://www.nuget.org/packages/Nancy.Authentication.Basic" target="_blank" rel="noopener noreferrer">Nancy.Authentication.Basic</a> para autenticación básica de HTTP o bien <a href="https://www.nuget.org/packages/Nancy.Authentication.Stateless" target="_blank" rel="noopener noreferrer">Nancy.Authentication.Stateless</a> para basar la autenticación en alguna cabecera específica de la petición). Todos estos paquetes (y más que hay para otros tipos de autenticación) se basan en el modelo de extensibilidad de NancyFx.

Por supuesto hay un paquete para integrarnos con oAuth que antes respondía al interesante nombre de Nancy.Authentication.WorldDomination pero que ahora tiene el aburrido y anodino nombre de <a href="https://www.nuget.org/packages/Nancy.SimpleAuthentication/" target="_blank" rel="noopener noreferrer">Nancy.SimpleAuthentication</a>. Así que añade este paquete a tu proyecto y ya estarás listo para iniciar un flujo para autenticación.

Al añadir este paquete tu web.config se habrá modificado y se habrán añadido las siguientes líneas:

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:e3409aab-e52a-403e-a9df-a0a69c712618" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
    <div style="background: #000080; color: #fff; font-family: Verdana, Tahoma, Arial, sans-serif; font-weight: bold; padding: 2px 5px">
      Code Snippet
    </div>
    
    <div style="background: #ddd; max-height: 300px; overflow: auto">
      <ol start="1" style="background: #1e1e1e; margin: 0 0 0 2.5em; padding: 0 0 0 5px; white-space: nowrap">
        <li>
          <span style="background:#1e1e1e;color:#808080">  <</span><span style="background:#1e1e1e;color:#569cd6">configSections</span><span style="background:#1e1e1e;color:#808080">></span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#808080">    <</span><span style="background:#1e1e1e;color:#569cd6">section</span><span style="background:#1e1e1e;color:#808080"> </span><span style="background:#1e1e1e;color:#92caf4">name</span><span style="background:#1e1e1e;color:#808080">="</span><span style="background:#1e1e1e;color:#c8c8c8">authenticationProviders</span><span style="background:#1e1e1e;color:#808080">" </span><span s
tyle="background:#1e1e1e;color:#92caf4">type</span><span style="background:#1e1e1e;color:#808080">="</span><span style="background:#1e1e1e;color:#c8c8c8">SimpleAuthentication.Core.Config.ProviderConfiguration, SimpleAuthentication.Core</span><span style="background:#1e1e1e;color:#808080">" /></span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#808080">  </</span><span style="background:#1e1e1e;color:#569cd6">configSections</span><span style="background:#1e1e1e;color:#808080">></span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#808080">  <</span><span style="background:#1e1e1e;color:#569cd6">authenticationProviders</span><span style="background:#1e1e1e;color:#808080">></span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#808080">    <</span><span style="background:#1e1e1e;color:#569cd6">providers</span><span style="background:#1e1e1e;color:#808080">></span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#808080">      <</span><span style="background:#1e1e1e;color:#569cd6">add</span><span style="background:#1e1e1e;color:#808080"> </span><span style="background:#1e1e1e;color:#92caf4">name</span><span style="background:#1e1e1e;color:#808080">="</span><span style="background:#1e1e1e;color:#c8c8c8">Facebook</span><span style="background:#1e1e1e;color:#808080">" </span><span style="background:#1e1e1e;color:#92caf4">key</span><span style="background:#1e1e1e;color:#808080">="</span><span style="background:#1e1e1e;color:#c8c8c8">please-enter-your-real-value</span><span style="background:#1e1e1e;color:#808080">" </span><span style="background:#1e1e1e;color:#92caf4">secret</span><span style="background:#1e1e1e;color:#808080">="</span><span style="background:#1e1e1e;color:#c8c8c8">please-enter-your-real-value</span><span style="background:#1e1e1e;color:#808080">" /></span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#808080">      <</span><span style="background:#1e1e1e;color:#569cd6">add</span><span style="background:#1e1e1e;color:#808080"> </span><span style="background:#1e1e1e;color:#92caf4">name</span><span style="background:#1e1e1e;color:#808080">="</span><span style="background:#1e1e1e;color:#c8c8c8">Google</span><span style="background:#1e1e1e;color:#808080">" </span><span style="background:#1e1e1e;color:#92caf4">key</span><span style="background:#1e1e1e;color:#808080">="</span><span style="background:#1e1e1e;color:#c8c8c8">please-enter-your-real-value</span><span style="background:#1e1e1e;color:#808080">" </span><span style="background:#1e1e1e;color:#92caf4">secret</span><span style="background:#1e1e1e;color:#808080">="</span><span style="background:#1e1e1e;color:#c8c8c8">please-enter-your-real-value</span><span style="background:#1e1e1e;color:#808080">" /></span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#808080">      <</span><span style="background:#1e1e1e;color:#569cd6">add</span><span style="background:#1e1e1e;color:#808080"> </span><span style="background:#1e1e1e;color:#92caf4">name</span><span style="background:#1e1e1e;color:#808080">="</span><span style="background:#1e1e1e;color:#c8c8c8">Twitter</span><span style="background:#1e1e1e;color:#808080">" </span><span style="background:#1e1e1e;color:#92caf4">key</span><span style="background:#1e1e1e;color:#808080">="</span><span style="background:#1e1e1e;color:#c8c8c8">please-enter-your-real-value</span><span style="background:#1e1e1e;color:#808080">" </span><span style="background:#1e1e1e;color:#92caf4">secret</span><span style="background:#1e1e1e;color:#808080">="</span><span style="background:#1e1e1e;color:#c8c8c8">please-enter-your-real-value</span><span style="background:#1e1e1e;color:#808080">" /></span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#808080">      <</span><span style="background:#1e1e1e;color:#569cd6">add</span><span style="background:#1e1e1e;color:#808080"> </span><span style="background:#1e1e1e;color:#92caf4">name</span><span style="background:#1e1e1e;color:#808080">="</span><span style="background:#1e1e1e;color:#c8c8c8">WindowsLive</span><span style="background:#1e1e1e;color:#808080">" </span><span style="background:#1e1e1e;color:#92caf4">key</span><span style="background:#1e1e1e;color:#808080">="</span><span style="background:#1e1e1e;color:#c8c8c8">please-enter-your-real-value</span><span style="background:#1e1e1e;color:#808080">" </span><span style="background:#1e1e1e;color:#92caf4">secret</span><span style="background:#1e1e1e;color:#808080">="</span><span style="background:#1e1e1e;color:#c8c8c8">please-enter-your-real-value</span><span style="background:#1e1e1e;color:#808080">" /></span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#808080">    </</span><span style="background:#1e1e1e;color:#569cd6">providers</span><span style="background:#1e1e1e;color:#808080">></span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#808080">  </</span><span style="background:#1e1e1e;color:#569cd6">authenticationProviders</span><span style="background:#1e1e1e;color:#808080">></span>
        </li>
      </ol>
    </div></p>
  </div></p>
</div>

En nuestro caso podemos dejar solo la información del provider de Twitter y debeis rellenar el valor de consumer key y el consumer sectret que os provee Twitter. Acuérdate tambien de mover el tag <configSections> para que sea el primero dentro del <configuration> en el web.config, si no IIS os dará un error!

Ahora el siguiente paso es crear una clase que implemente IAuthenticationCallbackProvider:

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:928b69b2-6866-4e24-adba-8d1a36e669e6" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
    <div style="background: #000080; color: #fff; font-family: Verdana, Tahoma, Arial, sans-serif; font-weight: bold; padding: 2px 5px">
      Code Snippet
    </div>
    
    <div style="background: #ddd; max-height: 300px; overflow: auto">
      <ol start="1" style="background: #1e1e1e; margin: 0 0 0 2.5em; padding: 0 0 0 5px; white-space: nowrap">
        <li>
          <span style="background:#1e1e1e;color:#569cd6">public</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">class</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#4ec9b0">MyCustomAuthCallbackProvider</span><span style="background:#1e1e1e;color:#dcdcdc"> : </span><span style="background:#1e1e1e;color:#b8d7a3">IAuthenticationCallbackProvider</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">{</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">    </span><span style="background:#1e1e1e;color:#569cd6">public</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">dynamic</span><span style="background:#1e1e1e;color:#dcdcdc"> Process(</span><span style="background:#1e1e1e;color:#4ec9b0">NancyModule</span><span style="background:#1e1e1e;color:#dcdcdc"> nancyModule, </span><span style="background:#1e1e1e;color:#4ec9b0">AuthenticateCallbackData</span><span style="background:#1e1e1e;color:#dcdcdc"> model)</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">    {</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">        </span><span style="background:#1e1e1e;color:#57a64a">// En caso de autenticacin OK</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">    }</span>
        </li>
        <li>
          &nbsp;
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">    </span><span style="background:#1e1e1e;color:#569cd6">public</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">dynamic</span><span style="background:#1e1e1e;color:#dcdcdc"> OnRedirectToAuthenticationProviderError(</span><span style="background:#1e1e1e;color:#4ec9b0">NancyModule</span><span style="b
ackground:#1e1e1e;color:#dcdcdc"> nancyModule, </span><span style="background:#1e1e1e;color:#569cd6">string</span><span style="background:#1e1e1e;color:#dcdcdc"> errorMessage)</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">    {</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">        </span><span style="background:#1e1e1e;color:#57a64a">// En caso de error</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">    }</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">}</span>
        </li>
      </ol>
    </div></p>
  </div></p>
</div>

Ejecuta de nuevo tu aplicación. Ahora si pulsas sobre el enlace de login with twitter… ¡deberías ver la página de Twitter!:

[<img title="image" style="border-top: 0px; border-right: 0px; background-image: none; border-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; border-left: 0px; display: inline; padding-right: 0px" border="0" alt="image" src="http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_3391F1CF.png" width="207" height="244" />][7]

Una vez el usuario haya dado sus datos y haya autorizado a tu aplicación entonces se ejecuta el método Process de la clase que acabamos de crear. En el parámetro “model” tendrás toda la información necesaria:

[<img title="image" style="border-top: 0px; border-right: 0px; background-image: none; border-bottom: 0px; padding-top: 0px; padding-left: 0px; border-left: 0px; display: inline; padding-right: 0px" border="0" alt="image" src="http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_65F855FF.png" width="504" height="132" />][8]

Una vez sabes cual es el usuario autenticado debes autenticarlo en tu propia web. Es decir Twitter nos ha dado el OK, pero ahora lo que nos falta es utilizar algún mecanismo para autenticar todas las peticiones que realice este usuario en nuestra web.

> **Nota:** Para que todo funcione recuerda que la URL de callback de la aplicación en twitter debe apuntar a /authentication/authenticatecallback y que el enlace de “Login with twitter” debe apuntar a /authentication/redirect/twitter. Esas dos URLs son gestionadas automáticamente por el paquete WorldDomination.

Si queremos utilizar una cookie, tenemos que instalar el paquete <a href="https://www.nuget.org/packages/Nancy.Authentication.Forms" target="_blank" rel="noopener noreferrer">Nancy.Authentication.Forms</a>, así… que nada, a por él!

**Añadiendo la seguridad por Forms**

Una vez tenemos el paquete Nancy.Authentication.Forms añadido ya podemos configurarlo. Son necesarios 3 pasos para que todo funcione correctamente.

El primero es crear una clase que implemente la interfaz IUserIdentity. Esta interfaz es toda la información que el core de Nancy mantiene sobre un usuario autenticado (nombre y permisos):

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:11c4c954-a099-4e5c-b2d2-4ec8bf27343f" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
    <div style="background: #ddd; max-height: 300px; overflow: auto">
      <ol start="1" style="background: #1e1e1e; margin: 0 0 0 2em; padding: 0 0 0 5px; white-space: nowrap">
        <li>
          <span style="background:#1e1e1e;color:#569cd6">public</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">class</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#4ec9b0">AuthenticatedUser</span><span style="background:#1e1e1e;color:#dcdcdc"> : </span><span style="background:#1e1e1e;color:#b8d7a3">IUserIdentity</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">{</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">    </span><span style="background:#1e1e1e;color:#569cd6">public</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">string</span><span style="background:#1e1e1e;color:#dcdcdc"> UserName { </span><span style="background:#1e1e1e;color:#569cd6">get</span><span style="background:#1e1e1e;color:#dcdcdc">; </span><span style="background:#1e1e1e;color:#569cd6">set</span><span style="background:#1e1e1e;color:#dcdcdc">; }</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">    </span><span style="background:#1e1e1e;color:#569cd6">public</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#b8d7a3">IEnumerable</span><span style="background:#1e1e1e;color:#b4b4b4"><</span><span style="background:#1e1e1e;color:#569cd6">string</span><span style="background:#1e1e1e;color:#b4b4b4">></span><span style="background:#1e1e1e;color:#dcdcdc"> Claims { </span><span style="background:#1e1e1e;color:#569cd6">get</span><span style="background:#1e1e1e;color:#dcdcdc">; </span><span style="background:#1e1e1e;color:#569cd6">set</span><span style="background:#1e1e1e;color:#dcdcdc">; }</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">}</span>
        </li>
      </ol>
    </div></p>
  </div></p>
</div>

El segundo es crear un “User Mapper”. Esto vendría a ser el equivalente (solo lectura) del Membership Provider en ASP.NET. Es decir el encargado de obtener los datos del usuario del repositorio donde se guarden:

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:2d090522-6a16-4d9a-bfd4-bacfda140591" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
    <div style="background: #ddd; max-height: 300px; overflow: auto">
      <ol start="1" style="background: #1e1e1e; margin: 0 0 0 2.5em; padding: 0 0 0 5px; white-space: nowrap">
        <li>
          <span style="background:#1e1e1e;color:#569cd6">public</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">class</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#4ec9b0">MyUserMapper</span><span style="background:#1e1e1e;color:#dcdcdc"> : </span><span style="background:#1e1e1e;color:#b8d7a3">IUserMapper</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">{</span>
        </li>
        <li>
          &nbsp;
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">    </span><span style="background:#1e1e1e;color:#569cd6">public</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#b8d7a3">IUserIdentity</span><span style="background:#1e1e1e;color:#dcdcdc"> GetUserFromIdentifier(</span><span style="background:#1e1e1e;color:#4ec9b0">Guid</span><span style="background:#1e1e1e;color:#dcdcdc"> identifier, </span><span style="background:#1e1e1e;color:#4ec9b0">NancyContext</span><span style="background:#1e1e1e;color:#dcdcdc"> context)</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">    {</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">        </span><span style="background:#1e1e1e;color:#57a64a">// Obtendriamos el usuario de la BBDD</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">        </span><span style="background:#1e1e1e;color:#569cd6">return</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">new</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#4ec9b0">AuthenticatedUser</span><span style="background:#1e1e1e;color:#dcdcdc">()</span>
        </li>
        <li>
          <span style= "background:#1e1e1e;color:#dcdcdc">        {</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">            UserName </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#d69d85">"eiximenis"</span><span style="background:#1e1e1e;color:#dcdcdc">,</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">            Claims </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">new</span><span style="background:#1e1e1e;color:#dcdcdc">[] {</span><span style="background:#1e1e1e;color:#d69d85">"Read"</span><span style="background:#1e1e1e;color:#dcdcdc">, </span><span style="background:#1e1e1e;color:#d69d85">"Write"</span><span style="background:#1e1e1e;color:#dcdcdc">, </span><span style="background:#1e1e1e;color:#d69d85">"Admin"}</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">        };</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">    }</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">}</span>
        </li>
      </ol>
    </div></p>
  </div></p>
</div>

En Nancy todos los usuarios se identifican por un Guid (por supuesto esto no significa que en la BBDD este Guid tenga que ser la PK de la tabla de usuarios!).

El tercer y último paso es indicar al core de Nancy que queremos autenticación por formularios:

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:064360d7-6b15-4712-9ab9-37a818875303" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
    <div style="background: #ddd; max-height: 300px; overflow: auto">
      <ol start="1" style="background: #1e1e1e; margin: 0 0 0 2.5em; padding: 0 0 0 5px; white-space: nowrap">
        <li>
          <span style="background:#1e1e1e;color:#569cd6">public</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">class</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#4ec9b0">Bootstrapper</span><span style="background:#1e1e1e;color:#dcdcdc"> : </span><span style="background:#1e1e1e;color:#4ec9b0">DefaultNancyBootstrapper</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">{</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">    </span><span style="background:#1e1e1e;color:#569cd6">protected</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">override</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">void</span><span style="background:#1e1e1e;color:#dcdcdc"> RequestStartup(</span><span style="background:#1e1e1e;color:#4ec9b0">TinyIoCContainer</span><span style="background:#1e1e1e;color:#dcdcdc"> container, </span><span style="background:#1e1e1e;color:#b8d7a3">IPipelines</span><span style="background:#1e1e1e;color:#dcdcdc"> pipelines, </span><span style="background:#1e1e1e;color:#4ec9b0">NancyContext</span><span style="background:#1e1e1e;color:#dcdcdc"> context)</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">    {</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">        </span><span style="background:#1e1e1e;color:#569cd6">base</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">RequestStartup(container, pipelines, context);</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">        </span><span style="background:#1e1e1e;color:#569cd6">var</span><span style="background:#1e1e1e;color:#dcdcdc"> formsAuthConfiguration </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">new</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#4ec9b0">FormsAuthenticationConfiguration</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">        {</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">            DisableRedirect </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">true</span><span style="background:#1e1e1e;color:#dcdcdc">,</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">            UserMapper </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">new</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#4ec9b0">MyUserMapper</span><span style="background:#1e1e1e;color:#dcdcdc">()</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">        };</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">        </span><span style="background:#1e1e1e;color:#4ec9b0">FormsAuthentication</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">Enable(pipelines, formsAuthConfiguration);</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">    }</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">}</span>
        </li>
      </ol>
    </div></p>
  </div></p>
</div>

El Bootstrapper es la clase que inicializa todo el core de Nancy. Hasta ahora no teníamos, así que lo añadimos y listos (no hay que indicar en ningún sitio cual es nuestro Bootstrapper, se descubre automáticamente). Con esto ya tenemos la autenticación por forms habilitada en nuestro proyecto.

Volvamos ahora a nuestro CallbackProvider. Lo que haríamos en el método Process es:

  * Consultar la BBDD de usuarios para encontrar el usuario que se corresponda con los datos que nos ha devuelto twitter.
  * En caso de no existir crearlo y obtener su Guid.
  * En caso de existir obtener su Guid.
  * Una vez tenemos el Guid llamar al método LoginAndRedirect. Este método vendría a ser el equivalente al SetAuthCookie de ASP.NET:

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:628f1fe3-50dd-429e-8d1f-23b17fa5020e" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
    <div style="background: #ddd; max-height: 300px; overflow: auto">
      <ol start="1" style="background: #1e1e1e; margin: 0 0 0 2em; padding: 0 0 0 5px; white-space: nowrap">
        <li>
          <span style="background:#1e1e1e;color:#569cd6">public</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">dynamic</span><span style="background:#1e1e1e;color:#dcdcdc"> Process(</span><span style="background:#1e1e1e;color:#4ec9b0">NancyModule</span><span style="background:#1e1e1e;color:#dcdcdc"> nancyModule, </span><span style="background:#1e1e1e;color:#4ec9b0">AuthenticateCallbackData</span><span style="background:#1e1e1e;color:#dcdcdc"> model)</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">{</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">    </span><span style="background:#1e1e1e;color:#57a64a">// Accederiamos a BBDD para obtener el GUID del usuario</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">    </span><span style="background:#1e1e1e;color:#57a64a">// O si no lo crearamos</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">    </span><span style="background:#1e1e1e;color:#569cd6">var</span><span style="background:#1e1e1e;color:#dcdcdc"> userGuid </span><span styl
e="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#4ec9b0">Guid</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">NewGuid();</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">    </span><span style="background:#1e1e1e;color:#569cd6">return</span><span style="background:#1e1e1e;color:#dcdcdc"> nancyModule</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">LoginAndRedirect(userGuid);</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">}</span>
        </li>
      </ol>
    </div></p>
  </div></p>
</div>

¡Y listos! ¡Has terminado!

**Añadir contenido securizado**

Vamos a añadir un módulo a Nancy que solo se ejecute si el usuario está autenticado. En ASP.NET MVC meterías un [Authorize] en la acción del controlador. En Nancy lo equivalente es usar el _module hook_ Before. Cada modulo de Nancy tiene una propiedad Before en la que puedes poner código que se ejecuta _antes_ de que se ejecute cualquier otro código del módulo (es decir, el código correspondiente a la petición). Desde este código puedes verificar si el usuario está autenticado:

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:61d45879-15b4-4a53-8f3d-e42c081c12dc" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
    <div style="background: #ddd; max-height: 300px; overflow: auto">
      <ol start="1" style="background: #1e1e1e; margin: 0 0 0 2.5em; padding: 0 0 0 5px; white-space: nowrap">
        <li>
          <span style="background:#1e1e1e;color:#569cd6">public</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">class</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#4ec9b0">SecureModule</span><span style="background:#1e1e1e;color:#dcdcdc"> : </span><span style="background:#1e1e1e;color:#4ec9b0">NancyModule</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">{</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">    </span><span style="background:#1e1e1e;color:#569cd6">public</span><span style="background:#1e1e1e;color:#dcdcdc"> SecureModule()</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">    {</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">        Before </span><span style="background:#1e1e1e;color:#b4b4b4">+=</span><span style="background:#1e1e1e;color:#dcdcdc"> ctx </span><span style="background:#1e1e1e;color:#b4b4b4">=></span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">        {</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">            </span><span style="background:#1e1e1e;color:#569cd6">if</span><span style="background:#1e1e1e;color:#dcdcdc"> (ctx</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">CurrentUser </span><span style="background:#1e1e1e;color:#b4b4b4">==</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">null</span><span style="background:#1e1e1e;color:#dcdcdc">)</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">                </span><span style="background:#1e1e1e;color:#569cd6">return</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">new</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#4ec9b0">Response</span><span style="background:#1e1e1e;color:#dcdcdc">()</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">                {</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">                    StatusCode </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#b8d7a3">HttpStatusCode</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">Unauthorized</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">                };</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">            </span><span style="background:#1e1e1e;color:#569cd6">return</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">null</span><span style="background:#1e1e1e;color:#dcdcdc">;</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">        };</span>
        </li>
        <li>
          &nbsp;
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">        Get[</span><span style="background:#1e1e1e;color:#d69d85">"/Secure"</span><span style="background:#1e1e1e;color:#dcdcdc">] </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> _ </span><span style="background:#1e1e1e;color:#b4b4b4">=></span><span style="background:#1e1e1e;color:#dcdcdc"> View[</span><span style="background:#1e1e1e;color:#d69d85">"secure.html"</span><span style="background:#1e1e1e;color:#dcdcdc">, </span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">            </span><span style="background:#1e1e1e;color:#569cd6">new</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">            {</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">                Name </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> Context</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">CurrentUser</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">UserName</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">            }];</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">    }</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">}</span>
        </li>
      </ol>
    </div></p>
  </div></p>
</div>

Si la propiedad del CurrentUser del contexto es igual a null devolvemos un 401. En caso contrario no hacemos nada y dejamos que siga el pipeline de Nancy (por lo que se ejecutará el código que toque según la URL).

Para terminar añade el archivo secure.html:

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:c05ee687-2a65-43dd-b180-42b1fa235d2c" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
    <div style="background: #ddd; max-height: 300px; overflow: auto">
      <ol start="1" style="background: #1e1e1e; margin: 0 0 0 2.5em; padding: 0 0 0 5px; white-space: nowrap">
        <li>
          <span style="background:#1e1e1e;color:#808080"><</span><span style="background:#1e1e1e;color:#569cd6">!DOCTYPE</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#9cdcfe">html</span><span style="background:#1e1e1e;color:#808080">></span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#808080"><</span><span style="background:#1e1e1e;color:#569cd6">html</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#9cdcfe">xmlns</span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#c8c8c8">"http://www.w3.org/1999/xhtml"</span><span style="background:#1e1e1e;color:#808080">></span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#808080"><</span><span style="background:#1e1e1e;color:
#569cd6">head</span><span style="background:#1e1e1e;color:#808080">></span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">    </span><span style="background:#1e1e1e;color:#808080"><</span><span style="background:#1e1e1e;color:#569cd6">title</span><span style="background:#1e1e1e;color:#808080">></</span><span style="background:#1e1e1e;color:#569cd6">title</span><span style="background:#1e1e1e;color:#808080">></span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#808080"></</span><span style="background:#1e1e1e;color:#569cd6">head</span><span style="background:#1e1e1e;color:#808080">></span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#808080"><</span><span style="background:#1e1e1e;color:#569cd6">body</span><span style="background:#1e1e1e;color:#808080">></span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">    Hello @Model.Name</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#808080"></</span><span style="background:#1e1e1e;color:#569cd6">body</span><span style="background:#1e1e1e;color:#808080">></span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#808080"></</span><span style="background:#1e1e1e;color:#569cd6">html</span><span style="background:#1e1e1e;color:#808080">></span>
        </li>
      </ol>
    </div></p>
  </div></p>
</div>

(Aunque lo parezca esto NO es Razor. Es el view engine por defecto de Nancy que se llama SSVE (Super Simple View Engine)).

Ya estamos listos para probar! Ejecuta el proyecto y navega a /secure. Deberías ver una página sin nada, pero un vistazo a la pestaña Network de las developer tools nos informa que estamos teniendo un 401:

[<img title="image" style="border-top: 0px; border-right: 0px; background-image: none; border-bottom: 0px; padding-top: 0px; padding-left: 0px; border-left: 0px; display: inline; padding-right: 0px" border="0" alt="image" src="http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_3BABAED5.png" width="504" height="132" />][9]

Pefecto. Vuelve a la raíz, y autentícate por twitter. Después del proceso de autenticación volverás a la raiz. Navega a /secure de nuevo y ahora deberías ver la vista secure.html:

[<img title="image" style="border-top: 0px; border-right: 0px; background-image: none; border-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; border-left: 0px; display: inline; padding-right: 0px" border="0" alt="image" src="http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_32527A52.png" width="244" height="94" />][10]

(Recuerda que todos los usuarios se llamarán eiximenis ya que nuestro IUserMapper siempre devuelve lo mismo ;))

**Y hemos finalizado!** Ya tienes tu web con Nancy autenticada mediante oAuth. Hemos elegido twitter pero para el resto de proveedores no hay muchas diferencias (el mérito es todo de WorldDomination).

**Unas palabras finales**

Hemos utilizado Katana para usar NancyFx en “modo OWIN” bajo el pipeline de ASP.NET en IIS. Esto nos permitiría integrarnos con otro middleware OWIN. Así pues, **y esta es precisamente la idea de OWIN**, podría usar un middleware OWIN para autenticar y autorizar las peticiones. Es decir, no delegar la autenticación y la autorización en el propio NancyFx si no tener un módulo OWIN encargado precisamente de esto…

… en el siguiente post veremos como 😉

Saludos!

**PD: Tienes el proyecto completo (VS2013) en** [**http://sdrv.ms/18brcO7**][11]**&#160;**(Archivo NancyKatanaIIS.zip)

 [1]: http://geeks.ms/blogs/etomas/archive/tags/katana/default.aspx
 [2]: http://www.variablenotfound.com/search/label/owin
 [3]: http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_0A62A77E.png
 [4]: http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_10AC0EBD.png
 [5]: http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_0D99B0C8.png
 [6]: http://localhost:<puerto>/authentication/authenticatecallback
 [7]: http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_185998CE.png
 [8]: http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_3CEDB703.png
 [9]: http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_752EBECC.png
 [10]: http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_3015F196.png
 [11]: http://sdrv.ms/18brcO7 "http://sdrv.ms/18brcO7"