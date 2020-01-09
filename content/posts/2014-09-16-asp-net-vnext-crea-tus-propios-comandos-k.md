---
title: ASP.NET vNext–Crea tus propios “comandos K”
description: ASP.NET vNext–Crea tus propios “comandos K”
author: eiximenis

date: 2014-09-16T23:33:42+00:00
geeks_url: /?p=1679
geeks_visits:
  - 1074
geeks_ms_views:
  - 1117
categories:
  - Uncategorized

---
Una de las características de ASP.NET vNext son los “comandos K”, es decir aquellos comandos que se invocan desde línea de comandos a través del fichero K.cmd.

Dichos comandos están definidos en el project.json y la idea es que ofrezcan tareas necesarias durante el ciclo de vida de compilación y pruebas. Así podemos tener un comando (p. ej. K run) que nos ejecute el proyecto y otro (K test) que nos lance los tests unitarios. Recordad siempre que ASP.NET vNext se crea con el objetivo de que sea multiplataforma total: no solo que sea ejecutable en Linux o MacOSX a través de Mono, si no que sea posible desarrollar en esos sistemas operativos. Y eso implica “desligarse” de Visual Studio. Por supuesto, eso no quita que VS añada e implemente su propio soporte para el ciclo de vida de compilación y pruebas.

De hecho, incluso actualmente en VS no es nuevo usar comandos para gestionar algunas de las tareas necesarias para el ciclo de vida. Así muchos de vosotros conoceréis el comando Install-Package para instalar un paquete NuGet. Así, a dia de hoy, usamos la “Package Manager Console” que no es más que un wrapper sobre PowerShell. Para algunos comandos (como el citado Install-Package) el propio VS ofrece una alternativa gráfica pero hay otros comandos que solo se pueden lanzar desde dicha consola. El ejemplo más claro son todos los comandos de EF Migrations (p. ej. Update-Database).

La razón de que ASP.NET vNext se “olvide” de los comandos Powershell es que Powershell solo funciona en Windows y recuerda… ASP.NET vNext es multiplataforma de verdad.

Por supuesto **nosotros podemos crear nuestros propios “comandos K”** para añadir tareas que nos sean necesarias o útiles para el ciclo de vida de compilación y pruebas de la aplicación. **Porque básicamente un comando definido en el project.json lo único que hace es invocar a un ensamblado**.

Vamos a ver como podemos hacerlo. 🙂

Los comandos residen en un assembly que debe ser compilado con vNext. En nuestro caso vamos a crear una “ASP.NET vNext Console Application” usando VS14 CTP3. En mi caso la he llamado TestCommands.

[<img title="image" style="border-top: 0px; border-right: 0px; background-image: none; border-bottom: 0px; padding-top: 0px; padding-left: 0px; border-left: 0px; display: inline; padding-right: 0px" border="0" alt="image" src="http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_187A54CF.png" width="504" height="192" />][1]

La verdad es que el template que viene con VS14 CTP3 para dicho tipo de aplicaciones es casi inútil, pero bueno… menos da una piedra. Dicho template genera la clásica clase “Program” con su método Main, útil para ejecutables, pero en nuestro caso nuestra aplicación estará lanzada a través de un comando K, así que será el propio framework de ASP.NET vNext quien invocará nuestra aplicación. 

Lo primero es agregar una dependencia a Microsoft.Framework.Runtime.Common en nuestro project.json:

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:312d8e8f-d64b-4b2d-8726-0548d11e67c9" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
    <div style="background: #ddd; max-height: 300px; overflow: auto">
      <ol start="1" style="background: #1d1d1d; margin: 0 0 0 2em; padding: 0 0 0 5px;">
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#d7ba7d">"dependencies"</span><span style="background:#1e1e1e;color:#b4b4b4">:</span><span style="background:#1e1e1e;color:#dcdcdc"> {</span>
        </li>
        <li style="background: #111111">
              <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#d7ba7d">"Microsoft.Framework.Runtime.Common"</span><span style="background:#1e1e1e;color:#b4b4b4">:</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#d69d85">"1.0.0-alpha3"</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">}</span><span style="background:#1e1e1e;color:#b4b4b4">,</span>
        </li>
      </ol>
    </div></p>
  </div></p>
</div>

Luego podemos modificar la clase Program para que quede de la siguiente así:

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:ee4a56d6-1f69-4540-8995-34388b7a9476" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
    <div style="background: #ddd; max-height: 300px; overflow: auto">
      <ol start="1" style="background: #1d1d1d; margin: 0 0 0 2.5em; padding: 0 0 0 5px;">
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">public</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">class</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#4ec9b0">Program</span>
        </li>
        <li style="background: #111111">
          <span style="background:#1e1e1e;color:#dcdcdc">{</span>
        </li>
        <li>
              <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">public</span><span style="background:#1e1e1e;color:#dcdcdc"> Program(</span><span style="background:#1e1e1e;color:#b8d7a3">IApplicationEnvironment</span><span style="background:#1e1e1e;color:#dcdcdc"> env)</span>
        </li>
        <li style="background: #111111">
              <span style="background:#1e1e1e;color:#dcdcdc">{</span>
        </li>
        <li>
                  <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#4ec9b0">Console</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">WriteLine(</span><span style="background:#1e1e1e;color:#d69d85">"In Progrm.ctor "</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#b4b4b4">+</span><span style="background:#1e1e1e;color:#dcdcdc"> env</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">ApplicationBasePath);</span>
        </li>
        <li style="background: #111111">
              <span style="background:#1e1e1e;color:#dcdcdc">}</span>
        </li>
        <li>
              <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">public</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">void</span><span style="background:#1e1e1e;color:#dcdcdc"> Main(</span><span style="background:#1e1e1e;color:#569cd6">string</span><span style="background:#1e1e1e;color:#dcdcdc">[] args)</span>
        </li>
        <li style="background: #111111">
              <span style="background:#1e1e1e;color:#dcdcdc">{</span>
        </li>
        <li>
                  <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#4ec9b0">Console</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">WriteLine(</span><span style="background:#1e1e1e;color:#d69d85">"In main"</span><span style="background:#1e1e1e;color:#dcdcdc">);</span>
        </li>
        <li style="background: #111111">
                  <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#4ec9b0">Console</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">ReadLine();</span>
        </li>
        <li>
              <span style="background:#1e1e1e;color:#dcdcdc">}</span>
        </li>
        <li style="background: #111111">
          <span style="background:#1e1e1e;color:#dcdcdc">}</ span></li> </ol></div> </p></div> </p></div> 
          
          <p>
            Si lo ejecutas con VS directamente verás algo parecido:
          </p>
          
          <p>
            <a href="http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_60C79A9E.png"><img title="image" style="border-top: 0px; border-right: 0px; background-image: none; border-bottom: 0px; padding-top: 0px; padding-left: 0px; border-left: 0px; display: inline; padding-right: 0px" border="0" alt="image" src="http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_6304235A.png" width="504" height="85" /></a>
          </p>
          
          <p>
            Por supuesto puedes ejecutarlo también usando KRE. Para ello asegúrate de tener en el path la carpeta donde está KRE instalado. Puedes tener varios KREs instalados side by side y por defecto se instalan en %HOME%.krepackages. Así p. ej. en mi maquina tengo:
          </p>
          
          <p>
            <a href="http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_3E0662E1.png"><img title="image" style="border-top: 0px; border-right: 0px; background-image: none; border-bottom: 0px; padding-top: 0px; padding-left: 0px; border-left: 0px; display: inline; padding-right: 0px" border="0" alt="image" src="http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_74E347D8.png" width="504" height="134" /></a>
          </p>
          
          <p>
            Debes agregar al path la carpeta bin del KRE que quieras usar. Así p. ej. yo tengo agregado al path la carpeta <em>C:Usersetomas.krepackagesKRE-svr50-x86.1.0.0-alpha3bin</em>
          </p>
          
          <p>
            Así si navegamos a la carpeta donde está el fichero project.json y ejecutamos “k run”:
          </p>
          
          <p>
            <a href="http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_5DB7CD5A.png"><img title="image" style="border-top: 0px; border-right: 0px; background-image: none; border-bottom: 0px; padding-top: 0px; padding-left: 0px; border-left: 0px; display: inline; padding-right: 0px" border="0" alt="image" src="http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_3207035E.png" width="504" height="78" /></a>
          </p>
          
          <p>
            En el caso de que os de un error de que no puede encontrar algún assembly lanzando un mensaje de error parecido al siguiente:
          </p>
          
          <p>
            <em>System.InvalidOperationException: Failed to resolve the following dependencies: <br />&#160;&#160; Microsoft.Framework.Runtime.Common 1.0.0-alpha3</em>
          </p>
          
          <p>
            Debes ejecutar el comando “kpm restore”. Eso es necesario cuando la maquina en la que estás ejecutando no tiene alguno de los paquetes marcados como dependencias en el project.json. Si es la propia maquina en la que tienes VS14 eso no te ocurrirá (VS14 se descarga los paquetes) pero, p. ej. yo he copiado el proyecto a <em>otra</em> máquina donde no tenía VS14, pero sí el runtime de vNext y he necesitado ejecutar dicho comando.
          </p>
          
          <p>
            Fijate en cuatro detalles:
          </p>
          
          <ol>
            <li>
              Se pasa primero por el constructor de la clase Program
            </li>
            <li>
              El framework nos inyecta automáticamente el objeto IApplicationEnvironment
            </li>
            <li>
              Luego se llama automáticamente al método Main
            </li>
            <li>
              No es necesario que el método Main sea estático.
            </li>
          </ol>
          
          <p>
            Si te preguntas como pasar parámetros al método main, pues simplemente añadiéndolos al comando “k run”. Así, si p. ej. tecleas “k run remove /s” el método Main recibirá dos parámetros (“remove” y “/s”). Para pasar parámetros con VS debes usar las propiedades del proyecto (sección Debugging).
          </p>
          
          <p>
            Podríamos implementar un comando “list” que listase todos los ficheros del proyecto:
          </p>
          
          <div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:340509a3-2ca0-4ec6-848e-21cc6c55c7ca" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
            <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
              <div style="background: #ddd; max-height: 300px; overflow: auto">
                <ol start="1" style="background: #1d1d1d; margin: 0 0 0 2.5em; padding: 0 0 0 5px;">
                  <li>
                    <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">public</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">class</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#4ec9b0">Program</span>
                  </li>
                  <li style="background: #111111">
                    <span style="background:#1e1e1e;color:#dcdcdc">{</span>
                  </li>
                  <li>
                        <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">private</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">readonly</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">string</span><span style="background:#1e1e1e;color:#dcdcdc"> _folder;</span>
                  </li>
                  <li style="background: #111111">
                        <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">public</span><span style="background:#1e1e1e;color:#dcdcdc"> Program(</span><span style="background:#1e1e1e;color:#b8d7a3">IApplicationEnvironment</span><span style="background:#1e1e1e;color:#dcdcdc"> env)</span>
                  </li>
                  <li>
                        <span style="background:#1e1e1e;color:#dcdcdc">{</span>
                  </li>
                  <li style="background: #111111">
                            <span style="background:#1e1e1e;color:#dcdcdc">_folder </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> env</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">ApplicationBasePath;</span>
                  </li>
                  <li>
                        <span style="background:#1e1e1e;color:#dcdcdc">}</span>
                  </li>
                  <li style="background: #111111">
                        <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">public</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">void</span><span style="background:#1e1e1e;color:#dcdcdc"> Main(</span><span style="background:#1e1e1e;color:#569cd6">string</span><span style="background:#1e1e1e;color:#dcdcdc">[] args)</span>
                  </li>
                  <li>
                        <span style="background:#1e1e1e;color:#dcdcdc">{</span>
                  </li>
                  <li style="background: #111111">
                            <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">if</span><span style="background:#1e1e1e;color:#dcdcdc"> (args</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">Length </span><span style="background:#1e1e1e;color:#b4b4b4">==</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#b5cea8">1</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#b4b4b4">&&</span><span style="background:#1e1e1e;color:#dcdcdc"> args[</span><span style="background:#1e1e1e;color:#b5cea8"></span><span style="background:#1e1e1e;color:#dcdcdc">] </span><span style="background:#1e1e1e;color:#b4b4b4">==</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#d69d85">"list"</span><span style="background:#1e1e1e;color:#dcdcdc">)</span>
                  </li>
                  <li>
                            <span style="background:#1e1e1e;color:#dcdcdc">{</span>
                  </li>
                  <li style="background: #111111">
                                <span style="background:#1e1e1e;color:#dcdcdc">ListFiles();</span>
                  </li>
                  <li>
                            <span style="background:#1e1e1e;color:#dcdcdc">}</span>
                  </li>
                  <li style="background: #111111">
                            <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#4ec9b0">Console</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">ReadLine();</span>
                  </li>
                  <li>
                        <span style="background:#1e1e1e;color:#dcdcdc">}</span>
                  </li>
                  <li
style="background: #111111">
                    &nbsp;
                  </li>
                  <li>
                        <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">private</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">void</span><span style="background:#1e1e1e;color:#dcdcdc"> ListFiles()</span>
                  </li>
                  <li style="background: #111111">
                        <span style="background:#1e1e1e;color:#dcdcdc">{</span>
                  </li>
                  <li>
                            <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">var</span><span style="background:#1e1e1e;color:#dcdcdc"> files </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#4ec9b0">Directory</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">EnumerateFiles(_folder, </span><span style="background:#1e1e1e;color:#d69d85">"*.*"</span><span style="background:#1e1e1e;color:#dcdcdc">, </span><span style="background:#1e1e1e;color:#b8d7a3">SearchOption</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">AllDirectories);</span>
                  </li>
                  <li style="background: #111111">
                            <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">foreach</span><span style="background:#1e1e1e;color:#dcdcdc"> (</span><span style="background:#1e1e1e;color:#569cd6">var</span><span style="background:#1e1e1e;color:#dcdcdc"> file </span><span style="background:#1e1e1e;color:#569cd6">in</span><span style="background:#1e1e1e;color:#dcdcdc"> files)</span>
                  </li>
                  <li>
                            <span style="background:#1e1e1e;color:#dcdcdc">{</span>
                  </li>
                  <li style="background: #111111">
                                <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#4ec9b0">Console</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">WriteLine(file);</span>
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
          
          <p>
            Para poder usar la clase Directory debes añadir una dependencia en el project.json al paquete <em>System.IO.FileSystem</em> (recuerda que en vNext todo el framework está fragmentado y dividido en paquetes NuGet).
          </p>
          
          <p>
            Bien, ahora ya sabemos que podemos ejecutar este programa con “k run list”, pero a nosotros lo que nos interesa es tener un “comando k” adicional para usar con <em>otro</em> programa vNext.
          </p>
          
          <p>
            Para ver como lo podemos montar vamos a agregar <strong>otro proyecto de consola de ASP.NET vNext</strong> a la solución. Yo lo he llamado DemoLauncher. No es necesario que toques nada del código.
          </p>
          
          <p>
            Ahora debes agregar una dependencia desde DemoLauncher al otro proyecto (que en mi caso se llamaba TestCommands):
          </p>
          
          <div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:4fc4d8f4-c450-4814-9081-5f167141bd74" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
            <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
              <div style="background: #ddd; max-height: 300px; overflow: auto">
                <ol start="1" style="background: #1d1d1d; margin: 0 0 0 2em; padding: 0 0 0 5px;">
                  <li>
                    <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#d7ba7d">"dependencies"</span><span style="background:#1e1e1e;color:#b4b4b4">:</span><span style="background:#1e1e1e;color:#dcdcdc"> {</span>
                  </li>
                  <li style="background: #111111">
                        <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#d7ba7d">"TestCommands"</span><span style="background:#1e1e1e;color:#b4b4b4">:</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#d69d85">""</span>
                  </li>
                  <li>
                    <span style="background:#1e1e1e;color:#dcdcdc">}</span><span style="background:#1e1e1e;color:#b4b4b4">,</span>
                  </li>
                </ol>
              </div></p>
            </div></p>
          </div>
          
          <p>
            Puedes agregar esta dependencia porque, a pesar de que TestCommands no
          </p>
          
          <p>
            está en NuGet está en la misma solución.
          </p>
          
          <p>
            Ahora <strong>damos de alta el comando</strong>. Los comandos se dan de alta en el project.json. Así editamos el project.json de DemoLauncher para añadir una sección de commands:
          </p>
          
          <div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:e057f81b-a491-4ef1-b52d-7e6ea6e5bfc6" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
            <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
              <div style="background: #ddd; max-height: 300px; overflow: auto">
                <ol start="1" style="background: #1d1d1d; margin: 0 0 0 2em; padding: 0 0 0 5px;">
                  <li>
                    <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#d7ba7d">"commands"</span><span style="background:#1e1e1e;color:#b4b4b4">:</span><span style="background:#1e1e1e;color:#dcdcdc"> {</span><span style="background:#1e1e1e;color:#dcdcdc"> </span>
                  </li>
                  <li style="background: #111111">
                        <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#d7ba7d">"tc"</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#b4b4b4">:</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#d69d85">"TestCommands"</span>
                  </li>
                  <li>
                    <span style="background:#1e1e1e;color:#dcdcdc">}</span>
                  </li>
                </ol>
              </div></p>
            </div></p>
          </div>
          
          <p>
            Con esto le indicamos que cuando se lance el comando “tc” se invoque al ensamblado “TestCommands”.
          </p>
          
          <p>
            Ahora puedo ir a la carpeta donde está el project.json de DemoLauncher y si tecleo “k run” se ejecutará DemoLauncher (eso era de esperar):
          </p>
          
          <p>
            <a href="http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_7AC07C22.png"><img title="image" style="border-top: 0px; border-right: 0px; background-image: none; border-bottom: 0px; padding-top: 0px; padding-left: 0px; border-left: 0px; display: inline; padding-right: 0px" border="0" alt="image" src="http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_4F0FB226.png" width="504" height="72" /></a>
          </p>
          
          <p>
            Pero lo bueno viene ahora. Si desde esa misma carpeta tecleas “k tc list” se te listarán todos los ficheros que haya en la carpeta (y subcarpetas) de DemoLauncher:
          </p>
          
          <p>
            <a href="http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_4C698726.png"><img title="image" style="border-top: 0px; border-right: 0px; background-image: none; border-bottom: 0px; padding-top: 0px; padding-left: 0px; border-left: 0px; display: inline; padding-right: 0px" border="0" alt="image" src="http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_4EA60FE2.png" width="504" height="124" /></a>
          </p>
          
          <p>
            Al lanzar “k tc” como en el project.json hay definido el comando “tc” se invoca al ensamblado “TestCommands” y se le pasan los parámetros que se hayan pasado después de tc. Así pues TestCommands recibe el parámetro “list” y lista todos los ficheros. Lo interesante es que TestCommands se ejecuta bajo el contexto de ejecución de DemoLauncher (la propiedad ApplicationBasePath del IApplicationEnvironment apunta al directorio donde está DemoLauncher).
          </p>
          
          <p>
            En un escenario final real, tendríamos “TestCommands” publicado a NuGet, pero el resto vendría a ser lo mismo que hemos visto.
          </p>
          
          <p>
            La gente de EF ya ha<br /> empezado a usar esa táctica para los comandos de Migrations (y así posibilitar el uso de Migrations en entornos no windows al no depender más de Powershell). Y personalmente creo que vamos a ver bastantes de esos futuros comandos.
          </p>
          
          <p>
            Un saludo!
          </p>

 [1]: http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_7B71A606.png