---
title: 'ASP.NET 5: Configuración'
author: eiximenis

date: 2015-09-03T18:26:40+00:00
geeks_url: /?p=1699
geeks_visits:
  - 632
geeks_ms_views:
  - 1856
categories:
  - Uncategorized

---
Una de las novedades de ASP.NET5 es su sistema de configuración. En versiones anteriores el sistema de configuración estaba muy atado al fichero web.config. En este fichero se guardaba tanto la configuración propia del programa (cadenas de conexión, _appsettings_ o información adicional que suele estar en secciones de configuración propias) como información de configuración del propio runtime: tipo de seguridad, módulos a cargar, bindings de assemblies y un sinfin más de configuraciones.

En ASP.NET5 eso se ha simplificado mucho. Un tema importante es que ahora está claramente separada la configuración del framework que se realiza mayoritamente por código, la configuración del _runtime_ que se delega en el fichero project.json y la configuración propia del programa que está en cualquier otro sitio. En este post nos centraremos solamente en la configuración propia del programa.

**Configuración propia del programa (configuración de negocio)**

La configuración propia toma la forma básica de un diccionario con claves que son cadenas y valores que son… cadenas. No tenemos una interfaz propia para cadenas de conexión o para las secciones propias de configuración. Para entendernos, sería como si todo fuesen _appsettings_. Es un mecanismo sencillo (si necesitas algo por encima de eso es fácil construirlo). El sistema por si mismo no nos ata a ningún formato de fichero (esa configuración puede estar en un json, un xml o lo que sea).

Aunque, es cierto, que el sistema _de por si_ no nos ata a ningún esquema en concreto de nuestros datos, si queremos utilizar los métodos que son capaces de leer información automáticamente de un fichero, entonces si que debemos adaptarnos al esquema previsto, aunque es bastante ligero.

P. ej. esos tres ficheros contienen todos ellos la misma configuración:

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:b57b74d7-834a-4203-b530-84de66ffa0bd" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
    <div style="background: #ddd; max-height: 300px; overflow: auto">
      <ol start="1" style="background: #ffffff; margin: 0 0 0 2.5em; padding: 0 0 0 5px;">
        <li>
          &nbsp;
        </li>
        <li>
            <span style="background:#ffffff;color:#000000"></span><span style="background:#ffffff;color:#2e75b6">"smtp"</span><span style="background:#ffffff;color:#000000">: {</span>
        </li>
        <li>
              <span style="background:#ffffff;color:#000000"></span><span style="background:#ffffff;color:#2e75b6">"server"</span><span style="background:#ffffff;color:#000000">: {</span>
        </li>
        <li>
                <span style="background:#ffffff;color:#000000"></span><span style="background:#ffffff;color:#2e75b6">"user"</span><span style="background:#ffffff;color:#000000">: </span><span style="background:#ffffff;color:#a31515">"eiximenis"</span><span style="background:#ffffff;color:#000000">,</span>
        </li>
        <li>
                <span style="background:#ffffff;color:#000000"></span><span style="background:#ffffff;color:#2e75b6">"mail"</span><span style="background:#ffffff;color:#000000">: </span><span style="background:#ffffff;color:#a31515">"mail@domain.com"</span>
        </li>
        <li>
              <span style="background:#ffffff;color:#000000">},</span>
        </li>
        <li>
              <span style="background:#ffffff;color:#000000"></span><span style="background:#ffffff;color:#2e75b6">"auth"</span><span style="background:#ffffff;color:#000000">: {</span>
        </li>
        <li>
                <span style="background:#ffffff;color:#000000"></span><span style="background:#ffffff;color:#2e75b6">"anonymous"</span><span style="background:#ffffff;color:#000000">: </span><span style="background:#ffffff;color:#a31515">"true"</span>
        </li>
        <li>
              <span style="background:#ffffff;color:#000000">}</span>
        </li>
        <li>
            <span style="background:#ffffff;color:#000000">}</span>
        </li>
        <li>
          <span style="background:#ffffff;color:#000000">}</span>
        </li>
      </ol>
    </div></p>
  </div></p>
</div></p> 

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:90fb9c52-ebda-4332-8434-fe3836bb2a9f" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
    <div style="background: #ddd; max-height: 400px; overflow: auto">
      <ol start="1" style="background: #ffffff; margin: 0 0 0 2.5em; padding: 0 0 0 5px;">
        <li>
          <span style="background:#ffffff;color:#0000ff"><</span><span style="background:#ffffff;color:#a31515">smtp</span><span style="background:#ffffff;color:#0000ff">></span>
        </li>
        <li>
              <span style="background:#ffffff;color:#0000ff"><</span><span style="background:#ffffff;color:#a31515">server</span><span style="background:#ffffff;color:#0000ff">></span>
        </li>
        <li>
                  <span style="background:#ffffff;color:#0000ff"><</span><span style="background:#ffffff;color:#a31515">user</span><span style="background:#ffffff;color:#0000ff">></span><span style="background:#ffffff;color:#000000">eiximenis</span><span style="background:#ffffff;color:#0000ff"></</span><span style="background:#ffffff;color:#a31515">user</span><span style="background:#ffffff;color:#0000ff">></span>
        </li>
        <li>
                  <span style="background:#ffffff;color:#0000ff"><</span><span style="background:#ffffff;color:#a31515">mail</span><span style="background:#ffffff;color:#0000ff">></span><span style="background:#ffffff;color:#000000">mail@domain.com</span><span style="background:#ffffff;color:#0000ff"></</span><span style="background:#ffffff;color:#a31515">mail</span><span style="background:#ffffff;color:#0000ff">></span>
        </li>
        <li>
              <span style="background:#ffffff;color:#0000ff"></</span><span style="background:#ffffff;color:#a31515">server</span><span style="background:#ffffff;color:#0000ff">></span>
        </li>
        <li>
              <span style="background:#ffffff;color:#0000ff"><</span><span style="background:#ffffff;color:#a31515">auth</span><span style="background:#ffffff;color:#0000ff">></span>
        </li>
        <li>
                  <span style="background:#ffffff;color:#0000ff"><</span><span style="background:#ffffff;color:#a31515">anonymous</span><span style="background:#ffffff;color:#0000ff">></span><span style="background:#ffffff;color:#000000">true</span><span style="background:#ffffff;color:#0000ff"></</span><span style="background:#ffffff;color:#a31515">anonymous</span><span style="background:#ffffff;color:#0000ff">></span>
        </li>
        <li>
              <span style="background:#ffffff;color:#0000ff"></</span><span style="background:#ffffff;color:#a31515">auth</span><span style="background:#ffffff;color:#0000ff">></span>
        </li>
        <li>
          <span style="background:#ffffff;color:#0000ff"></</span><span style="background:#ffffff;color:#a31515">smtp</span><span style="background:#ffffff;color:#0000ff">></span>
        </li>
      </ol>
    </div></p>
  </div></p>
</div></p> 

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:ab4376cb-9f54-4032-a71a-c9881d3d13b1" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
    <div style="background: #ddd; max-height: 300px; overflow: auto">
      <ol start="1" style="background: #ffffff; margin: 0 0 0 2em; padding: 0 0 0 5px;">
        <li>
          <span style="background:#ffffff;color:#000000">[smtp:server]</span>
        </li>
        <li>
          <span style="background:#ffffff;color:#000000">user=true</span>
        </li>
        <li>
          <span style="background:#ffffff;color:#000000">mail=mail@domain.com</span>
        </li>
        <li>
          <span style="background:#ffffff;color:#000000">[smtp:auth]</span>
        </li>
        <li>
          <span style="b
ackground:#ffffff;color:#000000">anonymous=true</span>
        </li>
      </ol>
    </div></p>
  </div></p>
</div></p> 

Los 3 ficheros definen las siguientes claves:

  * smtp:server:user –> Con valor “eiximenis”
  * stmp:server:mail –> Con valor [“mail@domain.com][1]”
  * smtp:auth:anonymous –> Con valor “true”

En versiones anteriores esos 3 valores hubiesen ido, seguramente, en la sección<appSettings /> del web.config. La verdad es que usar el separador dos puntos (:) en las claves de appsettings es algo que ya era como un “estándard de facto”.

Para cargar esos ficheros usamos los métodos AddJsonFile(), AddXmlFile() o AddIniFile() del objeto ConfigurationBuilder, que se suele crear en el constructor de la clase Startup:

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:f75edf3d-2812-461e-b054-2a9ee5c3229b" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
    <div style="background: #ddd; max-height: 300px; overflow: auto">
      <ol start="1" style="background: #ffffff; margin: 0 0 0 2.5em; padding: 0 0 0 5px;">
        <li>
          <span style="background:#ffffff;color:#000000"></span><span style="background:#ffffff;color:#0000ff">public</span><span style="background:#ffffff;color:#000000"> Startup(</span><span style="background:#ffffff;color:#2b91af">IHostingEnvironment</span><span style="background:#ffffff;color:#000000"> env, </span><span style="background:#ffffff;color:#2b91af">IApplicationEnvironment</span><span style="background:#ffffff;color:#000000"> appEnv)</span>
        </li>
        <li>
          <span style="background:#ffffff;color:#000000">{</span>
        </li>
        <li>
              <span style="background:#ffffff;color:#000000"></span><span style="background:#ffffff;color:#008000">// Setup configuration sources.</span>
        </li>
        <li>
              <span style="background:#ffffff;color:#000000"></span><span style="background:#ffffff;color:#0000ff">var</span><span style="background:#ffffff;color:#000000"> builder = </span><span style="background:#ffffff;color:#0000ff">new</span><span style="background:#ffffff;color:#000000"> </span><span style="background:#ffffff;color:#2b91af">ConfigurationBuilder</span><span style="background:#ffffff;color:#000000">(appEnv.ApplicationBasePath)</span>
        </li>
        <li>
                  <span style="background:#ffffff;color:#000000"></span><span style="background:#ffffff;color:#008000">//.AddJsonFile("config.json")</span>
        </li>
        <li>
                  <span style="background:#ffffff;color:#000000"></span><span style="background:#ffffff;color:#008000">//.AddXmlFile("config.xml")</span>
        </li>
        <li>
                  <span style="background:#ffffff;color:#000000">.AddIniFile(</span><span style="background:#ffffff;color:#a31515">"config.ini"</span><span style="background:#ffffff;color:#000000">)</span>
        </li>
        <li>
                  <span style="background:#ffffff;color:#000000">.AddEnvironmentVariables();</span>
        </li>
        <li>
              <span style="background:#ffffff;color:#000000">Configuration = builder.Build();</span>
        </li>
        <li>
          <span style="background:#ffffff;color:#000000">}</span>
        </li>
        <li>
          &nbsp;
        </li>
        <li>
          <span style="background:#ffffff;color:#000000"></span><span style="background:#ffffff;color:#0000ff">public</span><span style="background:#ffffff;color:#000000"> </span><span style="background:#ffffff;color:#2b91af">IConfiguration</span><span style="background:#ffffff;color:#000000"> Configuration { </span><span style="background:#ffffff;color:#0000ff">get</span><span style="background:#ffffff;color:#000000">; </span><span style="background:#ffffff;color:#0000ff">set</span><span style="background:#ffffff;color:#000000">; }</span>
        </li>
      </ol>
    </div></p>
  </div></p>
</div>

El objeto Configuration básicamente expone un indexer para acceder a los valores de la configuración:

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:29dbfc18-ac23-4a92-be8c-5add3fd82efa" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
    <div style="background: #ddd; max-height: 300px; overflow: auto">
      <ol start="1" style="background: #ffffff; margin: 0 0 0 2em; padding: 0 0 0 5px;">
        <li>
          <span style="background:#ffffff;color:#0000ff">var</span><span style="background:#ffffff;color:#000000"> usermail = Configuration[</span><span style="background:#ffffff;color:#a31515">"smtp:server:user"</span><span style="background:#ffffff;color:#000000">];</span>
        </li>
      </ol>
    </div></p>
  </div></p>
</div>

Si quiero acceder a un valor de la configuración desde un controlador MVC puedo incorporar el objeto Configuration dentro del sistema de inyección de dependencias de asp.net 5:

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:cf2a7e6d-d951-4762-9e0a-3aedc9f32a8d" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
    <div style="background: #ddd; max-height: 300px; overflow: auto">
      <ol start="1" style="background: #ffffff; margin: 0 0 0 2em; padding: 0 0 0 5px;">
        <li>
          <span style="background:#ffffff;color:#000000"></span><span style="background:#ffffff;color:#0000ff">public</span><span style="background:#ffffff;color:#000000"> </span><span style="background:#ffffff;color:#0000ff">void</span><span style="background:#ffffff;color:#000000"> ConfigureServices(</span><span style="background:#ffffff;color:#2b91af">IServiceCollection</span><span style="background:#ffffff;color:#000000"> services)</span>
        </li>
        <li>
          <span style="background:#ffffff;color:#000000">{</span>
        </li>
        <li>
              <span style="background:#ffffff;color:#000000">services.AddInstance<</span><span style="background:#ffffff;color:#2b91af">IConfiguration</span><span style="background:#ffffff;color:#000000">>(Configuration);</span>
        </li>
        <li>
          <span style="background:#ffffff;color:#000000">}</span>
        </li>
      </ol>
    </div></p>
  </div></p>
</div>

Y ahora puedo añadir un parámetro IConfiguration a cada controlador que lo requiera.

El método GetConfigurationSection me devuelve otro IConfiguration pero cuya raíz es la clave que yo indique. Es decir dado el objeto Configuration que tenía si hago:

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:e56f2434-410b-47da-9382-b56de2b6ff1f" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
    <div style="background: #ddd; max-height: 300px; overflow: auto">
      <ol start="1" style="background: #ffffff; margin: 0 0 0 2em; padding: 0 0 0 5px;">
        <li>
          <span style="background:#ffffff;color:#000000"></span><span style="background:#ffffff;color:#0000ff">var</span><span style="background:#ffffff;color:#000000"> usermail = Configuration[</span><span style="background:#ffffff;color:#a31515">"smtp:server:user"</span><span style="background:#ffffff;color:#000000">];</span>
        </li>
        <li>
          <span style="background:#ffffff;color:#000000"></span><span style="background:#ffffff;color:#0000ff">var</span><span style="background:#ffffff;color:#000000"> smtpcfg = Configuration.GetConfigurationSection(</span><span style="background:#ffffff;color:#a31515">"smtp"</span><span style="background:#ffffff;color:#000000">);</span>
        </li>
        <li>
          <span style="background:#ffffff;color:#000000"></span><span style="background:#ffffff;color:#0000ff">var</span><span style="background:#ffffff;color:#000000"> usermail2 = smtpcfg[</span><span style="background:#ffffff;color:#a31515">"server:user"</span><s pan style="background:#ffffff;color:#000000">];</span>
        </li>
        <li>
          <span style="background:#ffffff;color:#000000"></span><span style="background:#ffffff;color:#0000ff">var</span><span style="background:#ffffff;color:#000000"> servercfg = Configuration.GetConfigurationSection(</span><span style="background:#ffffff;color:#a31515">"smtp:server"</span><span style="background:#ffffff;color:#000000">);</span>
        </li>
        <li>
          <span style="background:#ffffff;color:#000000"></span><span style="background:#ffffff;color:#0000ff">var</span><span style="background:#ffffff;color:#000000"> usermail3 = servercfg[</span><span style="background:#ffffff;color:#a31515">"user"</span><span style="background:#ffffff;color:#000000">];</span>
        </li>
      </ol>
    </div></p>
  </div></p>
</div>

Los valores de usermail, usermail2 y usermail3 son el mismo.

**Configuración tipada**

Este esquema de configuración es funcional y sencillo, pero a veces queremos usar objetos POCO para guardar nuestra propia configuración. Supongamos que tenemos la siguiente clase para guardar los datos del servidor:

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:31eeb83a-db32-44ea-be2d-8bd6dc64c258" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
    <div style="background: #ddd; max-height: 300px; overflow: auto">
      <ol start="1" style="background: #ffffff; margin: 0 0 0 2em; padding: 0 0 0 5px;">
        <li>
          <span style="background:#ffffff;color:#000000"></span><span style="background:#ffffff;color:#0000ff">public</span><span style="background:#ffffff;color:#000000"> </span><span style="background:#ffffff;color:#0000ff">class</span><span style="background:#ffffff;color:#000000"> </span><span style="background:#ffffff;color:#2b91af">ServerConfig</span>
        </li>
        <li>
          <span style="background:#ffffff;color:#000000">{</span>
        </li>
        <li>
              <span style="background:#ffffff;color:#000000"></span><span style="background:#ffffff;color:#0000ff">public</span><span style="background:#ffffff;color:#000000"> </span><span style="background:#ffffff;color:#0000ff">string</span><span style="background:#ffffff;color:#000000"> User { </span><span style="background:#ffffff;color:#0000ff">get</span><span style="background:#ffffff;color:#000000">; </span><span style="background:#ffffff;color:#0000ff">set</span><span style="background:#ffffff;color:#000000">; }</span>
        </li>
        <li>
              <span style="background:#ffffff;color:#000000"></span><span style="background:#ffffff;color:#0000ff">public</span><span style="background:#ffffff;color:#000000"> </span><span style="background:#ffffff;color:#0000ff">string</span><span style="background:#ffffff;color:#000000"> Mail { </span><span style="background:#ffffff;color:#0000ff">get</span><span style="background:#ffffff;color:#000000">; </span><span style="background:#ffffff;color:#0000ff">set</span><span style="background:#ffffff;color:#000000">; }</span>
        </li>
        <li>
          <span style="background:#ffffff;color:#000000">}</span>
        </li>
      </ol>
    </div></p>
  </div></p>
</div>

Podemos mapear los datos que están en “smtp:server” con el siguiente código:

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:0941d2b7-3f8e-4adf-a3d1-9a4041204463" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
    <div style="background: #ddd; max-height: 300px; overflow: auto">
      <ol start="1" style="background: #ffffff; margin: 0 0 0 2em; padding: 0 0 0 5px;">
        <li>
          <span style="background:#ffffff;color:#000000"></span><span style="background:#ffffff;color:#0000ff">public</span><span style="background:#ffffff;color:#000000"> </span><span style="background:#ffffff;color:#0000ff">void</span><span style="background:#ffffff;color:#000000"> ConfigureServices(</span><span style="background:#ffffff;color:#2b91af">IServiceCollection</span><span style="background:#ffffff;color:#000000"> services)</span>
        </li>
        <li>
          <span style="background:#ffffff;color:#000000">{</span>
        </li>
        <li>
              <span style="background:#ffffff;color:#000000">services.Configure<</span><span style="background:#ffffff;color:#2b91af">ServerConfig</span><span style="background:#ffffff;color:#000000">>(Configuration.GetConfigurationSection(</span><span style="background:#ffffff;color:#a31515">"smtp:server"</span><span style="background:#ffffff;color:#000000">));</span>
        </li>
        <li>
          <span style="background:#ffffff;color:#000000">}</span>
        </li>
      </ol>
    </div></p>
  </div></p>
</div>

Recuerda que GetConfigurationSection(“stmp:server”) me devuelve un IConfiguration que apunta directamente a esta clave y que por lo tanto tiene las dos claves “user” y “mail” que se corresponen con los nombres de las propiedades de la clase ServerConfig. Esta línea además incluye dentro del sistema de inyección de dependencias la clase ServerConfig así que ahora puedo inyectarla a cualquier controlador MVC que lo requiera:

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:4c98a549-5e37-416f-9daa-294a06cc52ee" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
    <div style="background: #ddd; max-height: 300px; overflow: auto">
      <ol start="1" style="background: #ffffff; margin: 0 0 0 2em; padding: 0 0 0 5px;">
        <li>
          <span style="background:#ffffff;color:#000000"></span><span style="background:#ffffff;color:#0000ff">public</span><span style="background:#ffffff;color:#000000"> HomeController(</span><span style="background:#ffffff;color:#2b91af">IOptions</span><span style="background:#ffffff;color:#000000"><</span><span style="background:#ffffff;color:#2b91af">ServerConfig</span><span style="background:#ffffff;color:#000000">> sc)</span>
        </li>
        <li>
          <span style="background:#ffffff;color:#000000">{</span>
        </li>
        <li>
              <span style="background:#ffffff;color:#000000"></span><span style="background:#ffffff;color:#0000ff">var</span><span style="background:#ffffff;color:#000000"> cfg = sc.Options;</span>
        </li>
        <li>
              <span style="background:#ffffff;color:#000000"></span><span style="background:#ffffff;color:#0000ff">var</span><span style="background:#ffffff;color:#000000"> user = cfg.User;</span>
        </li>
        <li>
          <span style="background:#ffffff;color:#000000">}</span>
        </li>
      </ol>
    </div></p>
  </div></p>
</div>

Observa, eso sí, que el parámetro no es un ServerConfig, si no un IOptions<ServerConfig> (eso es porque services.Configure incluye la interfaz IOptions<T> en el sistema de inyección de dependencias). Para acceder al objeto con la configuración usamos la propiedad Options.

**Configuración anidada**

Por supuesto el sistema soporta configuración anidada. Es decir, lo siguiente es correcto:

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:96c2d432-05f5-4ccb-867b-af49898d2239" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
    <div style="background: #ddd; max-height: 300px; overflow: auto">
      <ol start="1" style="background: #ffffff; margin: 0 0 0 2.5em; padding: 0 0 0 5px;">
        <li>
          <span style="background:#ffffff;color:#000000"></span><span style="background:#ffffff;color:#0000ff">public</span><span style="background:#ffffff;color:#000000"> </span><span style="background:#ffffff;color:#0000ff">class</span><span style="background:#ffffff;color:#000000"> </span><span style="background:#ffffff;color:#2b91af">SmtpConfig</span>
        </li>
        <li>
           <span style="background:#ffffff;co
lor:#000000">{</span>
        </li>
        <li>
               <span style="background:#ffffff;color:#000000"></span><span style="background:#ffffff;color:#0000ff">public</span><span style="background:#ffffff;color:#000000"> </span><span style="background:#ffffff;color:#2b91af">ServerConfig</span><span style="background:#ffffff;color:#000000"> Server { </span><span style="background:#ffffff;color:#0000ff">get</span><span style="background:#ffffff;color:#000000">; </span><span style="background:#ffffff;color:#0000ff">set</span><span style="background:#ffffff;color:#000000">; }</span>
        </li>
        <li>
               <span style="background:#ffffff;color:#000000"></span><span style="background:#ffffff;color:#0000ff">public</span><span style="background:#ffffff;color:#000000"> </span><span style="background:#ffffff;color:#2b91af">AuthConfig</span><span style="background:#ffffff;color:#000000"> Auth { </span><span style="background:#ffffff;color:#0000ff">get</span><span style="background:#ffffff;color:#000000">; </span><span style="background:#ffffff;color:#0000ff">set</span><span style="background:#ffffff;color:#000000">; }</span>
        </li>
        <li>
           <span style="background:#ffffff;color:#000000">}</span>
        </li>
        <li>
          &nbsp;
        </li>
        <li>
           <span style="background:#ffffff;color:#000000"></span><span style="background:#ffffff;color:#0000ff">public</span><span style="background:#ffffff;color:#000000"> </span><span style="background:#ffffff;color:#0000ff">class</span><span style="background:#ffffff;color:#000000"> </span><span style="background:#ffffff;color:#2b91af">ServerConfig</span>
        </li>
        <li>
           <span style="background:#ffffff;color:#000000">{</span>
        </li>
        <li>
               <span style="background:#ffffff;color:#000000"></span><span style="background:#ffffff;color:#0000ff">public</span><span style="background:#ffffff;color:#000000"> </span><span style="background:#ffffff;color:#0000ff">string</span><span style="background:#ffffff;color:#000000"> User { </span><span style="background:#ffffff;color:#0000ff">get</span><span style="background:#ffffff;color:#000000">; </span><span style="background:#ffffff;color:#0000ff">set</span><span style="background:#ffffff;color:#000000">; }</span>
        </li>
        <li>
               <span style="background:#ffffff;color:#000000"></span><span style="background:#ffffff;color:#0000ff">public</span><span style="background:#ffffff;color:#000000"> </span><span style="background:#ffffff;color:#0000ff">string</span><span style="background:#ffffff;color:#000000"> Mail { </span><span style="background:#ffffff;color:#0000ff">get</span><span style="background:#ffffff;color:#000000">; </span><span style="background:#ffffff;color:#0000ff">set</span><span style="background:#ffffff;color:#000000">; }</span>
        </li>
        <li>
           <span style="background:#ffffff;color:#000000">}</span>
        </li>
        <li>
          &nbsp;
        </li>
        <li>
           <span style="background:#ffffff;color:#000000"></span><span style="background:#ffffff;color:#0000ff">public</span><span style="background:#ffffff;color:#000000"> </span><span style="background:#ffffff;color:#0000ff">class</span><span style="background:#ffffff;color:#000000"> </span><span style="background:#ffffff;color:#2b91af">AuthConfig</span>
        </li>
        <li>
           <span style="background:#ffffff;color:#000000">{</span>
        </li>
        <li>
               <span style="background:#ffffff;color:#000000"></span><span style="background:#ffffff;color:#0000ff">public</span><span style="background:#ffffff;color:#000000"> </span><span style="background:#ffffff;color:#0000ff">string</span><span style="background:#ffffff;color:#000000"> Anonymous { </span><span style="background:#ffffff;color:#0000ff">get</span><span style="background:#ffffff;color:#000000">; </span><span style="background:#ffffff;color:#0000ff">set</span><span style="background:#ffffff;color:#000000">; }</span>
        </li>
        <li>
           <span style="background:#ffffff;color:#000000">}</span>
        </li>
      </ol>
    </div></p>
  </div></p>
</div>

Y registrarlas de la siguiente manera:

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:d2710411-015d-4b89-8f3a-6e925c82ad11" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
    <div style="background: #ddd; max-height: 300px; overflow: auto">
      <ol start="1" style="background: #ffffff; margin: 0 0 0 2em; padding: 0 0 0 5px;">
        <li>
          <span style="background:#ffffff;color:#000000">services.Configure<</span><span style="background:#ffffff;color:#2b91af">SmtpConfig</span><span style="background:#ffffff;color:#000000">>(Configuration.GetConfigurationSection(</span><span style="background:#ffffff;color:#a31515">"smtp"</span><span style="background:#ffffff;color:#000000">));</span>
        </li>
      </ol>
    </div></p>
  </div></p>
</div>

Y ahora podemos inyectar un&#160; IOptions<SmtpConfig> en cualquier controlador que lo requiera:

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:97ca206a-94b8-4322-a26e-393b88d12acb" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
    <div style="background: #ddd; max-height: 300px; overflow: auto">
      <ol start="1" style="background: #ffffff; margin: 0 0 0 2em; padding: 0 0 0 5px;">
        <li>
          <span style="background:#ffffff;color:#000000"></span><span style="background:#ffffff;color:#0000ff">public</span><span style="background:#ffffff;color:#000000"> HomeController(</span><span style="background:#ffffff;color:#2b91af">IOptions</span><span style="background:#ffffff;color:#000000"><</span><span style="background:#ffffff;color:#2b91af">SmtpConfig</span><span style="background:#ffffff;color:#000000">> sc)</span>
        </li>
        <li>
          <span style="background:#ffffff;color:#000000">{</span>
        </li>
        <li>
              <span style="background:#ffffff;color:#000000"></span><span style="background:#ffffff;color:#0000ff">var</span><span style="background:#ffffff;color:#000000"> cfg = sc.Options;</span>
        </li>
        <li>
              <span style="background:#ffffff;color:#000000"></span><span style="background:#ffffff;color:#0000ff">var</span><span style="background:#ffffff;color:#000000"> user = cfg.Server.User;</span>
        </li>
        <li>
          <span style="background:#ffffff;color:#000000">}</span>
        </li>
      </ol>
    </div></p>
  </div></p>
</div>

¡Sencillo y rápido!

Y hasta aquí este post sobre la configuración en ASP.NET5… espero que os haya resultado interesante!

Saludos!

 [1]: mailto:&ldquo;mail@domain.com