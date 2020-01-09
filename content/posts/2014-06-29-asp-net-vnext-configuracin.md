---
title: ASP.NET vNext–Configuración
author: eiximenis

date: 2014-06-29T23:29:18+00:00
geeks_url: /?p=1673
geeks_visits:
  - 1244
geeks_ms_views:
  - 1514
categories:
  - Uncategorized

---
Otra de las cosas que cambia, radicalmente, en ASP.NET vNext es el tema de la configuración. Hasta ahora teníamos, generalmente, mezcladas en un mismo archivo (web.config) tanto la configuración propia de nuestra aplicación (app settings, connection strings y módulos de configuración propios) como la del framework (p. ej. la configuración de forms authentication o de los HttpModules).

En vNext eso cambia radicalmente. Para empezar **desaparece web.config**. La configuración de los distintos módulos del framework se realiza por código, en el método Configure de la clase Startup. La configuración de nuestra aplicación se mantiene en ficheros de configuración pero **desaparecen las nociones de appsettings o cadenas de conexión**. Ýa no hay una API más o menos “tipada” como la del ConfigurationManager (donde tenemos propiedades como ConnectionStrings o AppSettings). La configuración de vNext es básicamente un diccionario de claves, valores. Así pues los ficheros de configuración que usemos **ya no tienen una estructura predefinida**. Desaparece toda noción de esquema en los ficheros de configuración. De hecho se soportan varios formatos: XML, JSON y INI (si, puede ser sorprendente el soporte para INI pero debemos tener presente la aspiración “multiplataforma” de vNext y ficheros INI son muy usados en entornos Unix).

Vamos a ver un ejemplo sencillo del uso del sistema de configuración nuevo de vNext. Para ello partiremos de una aplicación vNext vacía (ASP.NET vNext Empty Web Application).

Una diferencia importante respecto a ASP.NET clásico es que la configuración **ahora es un objeto.** Ya no hay clases estáticas (como ConfigurationManager). Así que el primer punto es crear dicho objeto en el método Configure de la clase Startup:

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:ec535624-6c8a-4db1-83ea-3dd9427e8fa0" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
    <div style="background: #ddd; max-height: 100px; overflow: auto">
      <ol start="1" style="background: #1d1d1d; margin: 0 0 0 2em; padding: 0 0 0 5px;">
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">public</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">void</span><span style="background:#1e1e1e;color:#dcdcdc"> Configure(</span><span style="background:#1e1e1e;color:#b8d7a3">IBuilder</span><span style="background:#1e1e1e;color:#dcdcdc"> app)</span>
        </li>
        <li style="background: #111111">
          <span style="background:#1e1e1e;color:#dcdcdc">{</span>
        </li>
        <li>
              <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">var</span><span style="background:#1e1e1e;color:#dcdcdc"> config </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">new</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#4ec9b0">Configuration</span><span style="background:#1e1e1e;color:#dcdcdc">();    </span>
        </li>
        <li style="background: #111111">
          <span style="background:#1e1e1e;color:#dcdcdc">}</span>
        </li>
      </ol>
    </div></p>
  </div></p>
</div>

El siguiente paso será agregar un archivo json (p. ej. data.json) con el siguiente formato:

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:6553c645-456c-436b-a3b6-76a665d3f418" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
    <div style="background: #ddd; max-height: 300px; overflow: auto">
      <ol start="1" style="background: #1d1d1d; margin: 0 0 0 2.5em; padding: 0 0 0 5px;">
        <li>
          {
        </li>
        <li style="background: #111111">
              <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#d7ba7d">"environments"</span><span style="background:#1e1e1e;color:#b4b4b4">:</span><span style="background:#1e1e1e;color:#dcdcdc"> {</span>
        </li>
        <li>
                  <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#d7ba7d">"dev"</span><span style="background:#1e1e1e;color:#b4b4b4">:</span><span style="background:#1e1e1e;color:#dcdcdc"> {</span><span style="background:#1e1e1e;color:#dcdcdc"> </span>
        </li>
        <li style="background: #111111">
                      <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#d7ba7d">"background"</span><span style="background:#1e1e1e;color:#b4b4b4">:</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#d69d85">"blue"</span>
        </li>
        <li>
                  <span style="background:#1e1e1e;color:#dcdcdc">}</span><span style="background:#1e1e1e;color:#b4b4b4">,</span>
        </li>
        <li style="background: #111111">
            <span style="background:#1e1e1e;color:#dcdcdc">        </span><span style="background:#1e1e1e;color:#d7ba7d">"pre"</span><span style="background:#1e1e1e;color:#b4b4b4">:</span><span style="background:#1e1e1e;color:#dcdcdc">{</span>
        </li>
        <li>
            <span style="background:#1e1e1e;color:#dcdcdc">            </span><span style="background:#1e1e1e;color:#d7ba7d">"background"</span><span style="background:#1e1e1e;color:#b4b4b4">:</span><span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#d69d85">"yellow"</span>
        </li>
        <li style="background: #111111">
                  <span style="background:#1e1e1e;color:#dcdcdc">}</span><span style="background:#1e1e1e;color:#b4b4b4">,</span>
        </li>
        <li>
            <span style="background:#1e1e1e;color:#dcdcdc">        </span><span style="background:#1e1e1e;color:#d7ba7d">"prod"</span><span style="background:#1e1e1e;color:#b4b4b4">:</span><span style="background:#1e1e1e;color:#dcdcdc">{</span>
        </li>
        <li style="background: #111111">
            <span style="background:#1e1e1e;color:#dcdcdc">            </span><span style="background:#1e1e1e;color:#d7ba7d">"background"</span><span style="background:#1e1e1e;color:#b4b4b4">:</span><span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#d69d85">"red"</span>
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

El “esquema” de los datos JSON es totalmente inventado.

Bien. La clase Configuration tiene un método, llamado Add, al cual debe pasársele un IConfigurationSource. No tiene soporte para cargar distintos tipos de archivo, pero dicho soporte se obtiene a través de métodos de extensión. Uno de ellos es AddJsonConfig que está definido en el paquete Microsoft.Framework.ConfigurationModel.Json, así que debemos agregar una referencia a dicho paquete en project.json (en la sección “dependencies”):

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:8fbd46b0-211c-495e-9d35-0be03f288648" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
    <div style="background: #ddd; max-height: 300px; overflow: auto">
      <ol start="1" style="background: #1d1d1d; margin: 0 0 0 2em; padding: 0 0 0 5px;">
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc"> </span><span s
tyle="background:#1e1e1e;color:#d7ba7d">"Microsoft.Framework.ConfigurationModel.Json"</span><span style="background:#1e1e1e;color:#b4b4b4">:</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#d69d85">"0.1-alpha-build-0233"</span>
        </li>
      </ol>
    </div></p>
  </div></p>
</div>

Ahora ya podemos usar el método “AddJsonFile”:

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:4df2335f-c1fd-46d9-98c4-f629d33276b2" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
    <div style="background: #ddd; max-height: 300px; overflow: auto">
      <ol start="1" style="background: #1d1d1d; margin: 0 0 0 2em; padding: 0 0 0 5px;">
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">config</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">AddJsonFile(</span><span style="background:#1e1e1e;color:#d69d85">"data.json"</span><span style="background:#1e1e1e;color:#dcdcdc">);</span>
        </li>
      </ol>
    </div></p>
  </div></p>
</div>

Con eso cargamos los valores de dicho fichero de configuración dentro del objeto _Configuration_. El objeto Configuration es un diccionario plano, pero nuestro JSON tiene profundidad (p. ej. el objeto environments tiene tres propiedades (dev, pre, prod), cada una de las cuales tiene otra propiedad llamada background. Para convertir esa estructura jerárquica a una plana, se añaden las claves usando el carácter dos puntos (:) como separador. Así, después de procesar este JSON el objeto Configuration tendrá tres claves:

  1. environments:dev:background (con el valor blue)
  2. environments:pre:background (con el valor yellow)
  3. environments:prod:background (con el valor red)

Se usa el método Get para acceder a un valor de la configuración. El método Get acepta un solo parámetro: la clave y devuelve el valor (otra cadena). Al ejecutar el siguiente código value valdría “yellow”:

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:b3ba88e3-fab7-4858-8774-ecbe4a2122de" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
    <div style="background: #ddd; max-height: 300px; overflow: auto">
      <ol start="1" style="background: #1d1d1d; margin: 0 0 0 2em; padding: 0 0 0 5px;">
        <li>
          <span style="background:#1e1e1e;color:#569cd6">var</span><span style="background:#1e1e1e;color:#dcdcdc"> value </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> config</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">Get(</span><span style="background:#1e1e1e;color:#d69d85">"environments:pre:background"</span><span style="background:#1e1e1e;color:#dcdcdc">);</span>
        </li>
      </ol>
    </div></p>
  </div></p>
</div>

Recordad: ¡no hay una sección predeterminada para guardar valores de cadenas de conexión ni appsettings! Nosotros decidimos donde guardar cada cosa que necesitemos.

Vamos a ver ahora como hacer uso de dicha configuración desde un módulo de vNext. P. ej. agreguemos ASP.NET MVC a nuestro proyecto, agregando la referencia a Microsoft.AspNet.Mvc a project.json:

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:5010030c-7914-4f54-9c94-0dd8985b5b2f" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
    <div style="background: #ddd; max-height: 300px; overflow: auto">
      <ol start="1" style="background: #1d1d1d; margin: 0 0 0 2em; padding: 0 0 0 5px;">
        <li>
          <span style="background:#1e1e1e;color:#d7ba7d">"Microsoft.AspNet.Mvc"</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#b4b4b4">:</span><span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#d69d85">"0.1-alpha-build-1268"</span>
        </li>
      </ol>
    </div></p>
  </div></p>
</div>

Ahora añadamos el código al método Configure de la clase Startup para inicializar correctamente ASP.NET MVC. Además añadimos el objeto _config_ dentro del sistema de inyección de dependencias de vNext:

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:11bfdc5f-0779-4def-8c0c-9761659fdf39" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
    <div style="background: #ddd; max-height: 300px; overflow: auto">
      <ol start="1" style="background: #1d1d1d; margin: 0 0 0 2.5em; padding: 0 0 0 5px;">
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">public</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">void</span><span style="background:#1e1e1e;color:#dcdcdc"> Configure(</span><span style="background:#1e1e1e;color:#b8d7a3">IBuilder</span><span style="background:#1e1e1e;color:#dcdcdc"> app)</span>
        </li>
        <li style="background: #111111">
          <span style="background:#1e1e1e;color:#dcdcdc">{</span>
        </li>
        <li>
              <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">var</span><span style="background:#1e1e1e;color:#dcdcdc"> config </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">new</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#4ec9b0">Configuration</span><span style="background:#1e1e1e;color:#dcdcdc">();</span>
        </li>
        <li style="background: #111111">
              <span style="background:#1e1e1e;color:#dcdcdc">config</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">AddJsonFile(</span><span style="background:#1e1e1e;color:#d69d85">"data.json"</span><span style="background:#1e1e1e;color:#dcdcdc">);</span>
        </li>
        <li>
              <span style="background:#1e1e1e;color:#dcdcdc">app</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">UseServices(</span>
        </li>
        <li style="background: #111111">
                  <span style="background:#1e1e1e;color:#dcdcdc">s </span><span style="background:#1e1e1e;color:#b4b4b4">=></span>
        </li>
        <li>
                  <span style="background:#1e1e1e;color:#dcdcdc">{</span>
        </li>
        <li style="background: #111111">
                      <span style="background:#1e1e1e;color:#dcdcdc">s</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">AddInstance<</span><span style="background:#1e1e1e;color:#b8d7a3">IConfiguration</span><span style="background:#1e1e1e;color:#dcdcdc">>(config);</span>
        </li>
        <li>
                      <span style="background:#1e1e1e;color:#dcdcdc">s</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">AddMvc();</span>
        </li>
        <li style="background: #111111">
                  <span style="background:#1e1e1e;color:#dcdcdc">});</span>
        </li>
        <li>
              <span style="background:#1e1e1e;color:#dcdcdc">app</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">UseMvc();</span>
        </li>
        <li style="background: #111111">
          <span style="background:#1e1e1e;color:#dcdcdc">}</span>
        </li>
      </ol>
    </div></p>
  </div></p>
</div>

Ahora agrega un c
  
ontrolador Home y **haz que tenga en su constructor un parámetro de tipo IConfiguration_:_**

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:66064df7-b8d1-48a4-a099-d4d8fa9f0a7a" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
    <div style="background: #ddd; max-height: 300px; overflow: auto">
      <ol start="1" style="background: #1d1d1d; margin: 0 0 0 2.5em; padding: 0 0 0 5px;">
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">public</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">class</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#4ec9b0">HomeController</span><span style="background:#1e1e1e;color:#dcdcdc"> : </span><span style="background:#1e1e1e;color:#4ec9b0">Controller</span>
        </li>
        <li style="background: #111111">
          <span style="background:#1e1e1e;color:#dcdcdc">{</span>
        </li>
        <li>
              <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">private</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">readonly</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">string</span><span style="background:#1e1e1e;color:#dcdcdc"> _bg;</span>
        </li>
        <li style="background: #111111">
          &nbsp;
        </li>
        <li>
              <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">public</span><span style="background:#1e1e1e;color:#dcdcdc"> HomeController(</span><span style="background:#1e1e1e;color:#b8d7a3">IConfiguration</span><span style="background:#1e1e1e;color:#dcdcdc"> cfg)</span>
        </li>
        <li style="background: #111111">
              <span style="background:#1e1e1e;color:#dcdcdc">{</span>
        </li>
        <li>
                  <span style="background:#1e1e1e;color:#dcdcdc">_bg </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> cfg</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">Get(</span><span style="background:#1e1e1e;color:#d69d85">"environments:dev:background"</span><span style="background:#1e1e1e;color:#dcdcdc">);</span>
        </li>
        <li style="background: #111111">
              <span style="background:#1e1e1e;color:#dcdcdc">}</span>
        </li>
        <li>
          &nbsp;
        </li>
        <li style="background: #111111">
              <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">public</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#b8d7a3">IActionResult</span><span style="background:#1e1e1e;color:#dcdcdc"> Index()</span>
        </li>
        <li>
              <span style="background:#1e1e1e;color:#dcdcdc">{</span>
        </li>
        <li style="background: #111111">
                  <span style="background:#1e1e1e;color:#dcdcdc">ViewBag</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">bg </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> _bg;</span>
        </li>
        <li>
                  <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">return</span><span style="background:#1e1e1e;color:#dcdcdc"> View();</span>
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

Dado que hemos agregado la configuración dentro del mecanismo de inyección de dependencias de vNext, ahora podemos inyectar dicha configuración por el constructor. Fíjate que en el controlador no nos guardamos toda la configuración si no solo lo que necestiamos. Finalmente si te creas una vista Index.cshtml de prueba (en Views/Home):

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:043be4ae-3c07-436f-aac1-eaf1c41a919d" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
    <div style="background: #ddd; max-height: 300px; overflow: auto">
      <ol start="1" style="background: #1d1d1d; margin: 0 0 0 2em; padding: 0 0 0 5px;">
        <li>
          <span style="background:#1e1e1e;color:#808080"><</span><span style="background:#1e1e1e;color:#569cd6">html</span><span style="background:#1e1e1e;color:#808080">></span>
        </li>
        <li style="background: #111111">
          <span style="background:#1e1e1e;color:#808080"><</span><span style="background:#1e1e1e;color:#569cd6">head</span><span style="background:#1e1e1e;color:#808080">><</span><span style="background:#1e1e1e;color:#569cd6">title</span><span style="background:#1e1e1e;color:#808080">></span><span style="background:#1e1e1e;color:#dcdcdc">Demo Config</span><span style="background:#1e1e1e;color:#808080"></</span><span style="background:#1e1e1e;color:#569cd6">title</span><span style="background:#1e1e1e;color:#808080">></</span><span style="background:#1e1e1e;color:#569cd6">head</span><span style="background:#1e1e1e;color:#808080">></span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#808080"><</span><span style="background:#1e1e1e;color:#569cd6">body</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#9cdcfe">style</span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#c8c8c8">"background: </span><span style="background:#ffffb3;color:#000000">@</span><span style="background:#1e1e1e;color:#dcdcdc">ViewBag.bg</span><span style="background:#1e1e1e;color:#c8c8c8">"</span><span style="background:#1e1e1e;color:#808080">></span>
        </li>
        <li style="background: #111111">
              <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#808080"><</span><span style="background:#1e1e1e;color:#569cd6">h1</span><span style="background:#1e1e1e;color:#808080">></span><span style="background:#1e1e1e;color:#dcdcdc">Demo config</span><span style="background:#1e1e1e;color:#808080"></</span><span style="background:#1e1e1e;color:#569cd6">h1</span><span style="background:#1e1e1e;color:#808080">></span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#808080"></</span><span style="background:#1e1e1e;color:#569cd6">body</span><span style="background:#1e1e1e;color:#808080">></span>
        </li>
        <li style="background: #111111">
          <span style="background:#1e1e1e;color:#808080"></</span><span style="background:#1e1e1e;color:#569cd6">html</span><span style="background:#1e1e1e;color:#808080">></span>
        </li>
      </ol>
    </div></p>
  </div></p>
</div>

Ahora al ejecutar la demo deberías ver la página con el fondo azul, pues este es el valor de la entrada environments:dev:background de la configuración:

[<img title="image" style="border-top: 0px; border-right: 0px; background-image: none; border-bottom: 0px; padding-top: 0px; padding-left: 0px; border-left: 0px; display: inline; padding-right: 0px" border="0" alt="image" src="http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_2A795018.png" width="504" height="181" />][1]

Y eso viene a ser todo… ¡Espero que os haya resultado interesante!

 [1]: http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_38B53857.png