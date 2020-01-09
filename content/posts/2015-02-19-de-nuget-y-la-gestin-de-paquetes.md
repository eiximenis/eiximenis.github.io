---
title: De NuGet y la gestión de paquetes
description: De NuGet y la gestión de paquetes
author: eiximenis

date: 2015-02-19T09:11:40+00:00
geeks_url: /?p=1695
geeks_visits:
  - 1777
geeks_ms_views:
  - 5356
categories:
  - Uncategorized

---
Ya hace bastante tiempo que NuGet salió y desde entonces se ha convertido en un compañero inseparable de todos nosotros. Y más que va a serlo cuando vNext salga de forma definitiva. En este post doy por supuesto que conoces NuGet y que lo has usado alguna vez (si no… ¡debes aprender a usarlo ya!). En este post quiero comentar los **tres** modos de funcionamiento que tiene NuGet y algunas cosillas más con las que me he encontrado.

Funcione NuGet en el modo en que funcione, cuando agregamos un paquete **siempre** ocurre lo mismo:

  * Se crea (si no existe) un directorio **packages** a nivel de la solución (localizado en el mismo directorio que el .sln). 
  * Se descarga el paquete en dicho directorio 
  * Se crea (si no existe) un fichero packages.config en el proyecto al cual se haya agregado el paquete y se añade una línea indicando el paquete agregado. 
  * Se añade una referencia en el proyecto que apunta al ensamblado del paquete que se encuentra en el directorio packages. Y si el paquete tiene scripts de instalación adicionales, pues se ejecutan. 

Recuerda que NuGet es básicamente un automatizador de agregar referencias en VS. Hace todo aquello que harías tu manualmente (descargar el ensamblado, guardarlo en algún sitio, agregar la referencia y cosas extra como editar web.config) y nada más (o nada menos, depende de como se mire).

Vamos a configurar NuGet para que funcione en el primero de los modos. Para ello abre VS2013 y en Tools –> Options –> NuGet Package Manager **desmarca** las dos checkboxes que están bajo el título de “Package Restore”. De esta manera NuGet funciona de la forma en que funcionaba originalmente. Y dicha forma consiste en que NuGet **no hará nada más que lo que hemos descrito hasta ahora**. Eso significa que cuando subas a tu repositorio de control de código fuente el proyecto **debes** incluir el directorio packages que contiene los binarios de los paquetes instalados por NuGet. Si no lo haces, cuando otra persona quiera descargarse el código fuente el proyecto no le compilará porque no encontrará las referencias a los paquetes NuGet:

[<img title="image" style="border-left-width: 0px; border-right-width: 0px; background-image: none; border-bottom-width: 0px; padding-top: 0px; padding-left: 0px; display: inline; padding-right: 0px; border-top-width: 0px" border="0" alt="image" src="http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_5AB3A0D9.png" width="327" height="126" />][1]

¿Está todo perdido? Pues no, porque NuGet detectará que hay paquetes referenciados (eso lo sabe mirando el packages.config) que no existen en el directorio packages. Así mostrará el siguiente mensaje en la “Package Manager Console”:

[<img title="image" style="border-left-width: 0px; border-right-width: 0px; background-image: none; border-bottom-width: 0px; padding-top: 0px; padding-left: 0px; display: inline; padding-right: 0px; border-top-width: 0px" border="0" alt="image" src="http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_4EB1B0A5.png" width="1080" height="147" />][2]

Este mensaje aparece porque estamos en una versión nueva de NuGet. Cuando apareció NuGet este mensaje no aparecía y honestamente no sé cual era la solución entonces porque ese era un escenario no soportado: en la primera versión de NuGet **los paquetes descargados debían subirse al control de código fuente**.

Una versión de NuGet posterior habilitó el segundo modo de funcionamiento de NuGet. Para activarlo se debe pulsar sobre la solución en el “solution explorer” y seleccionar la opción “Enable NuGet Package Restore”:

[<img title="image" style="border-left-width: 0px; border-right-width: 0px; background-image: none; border-bottom-width: 0px; padding-top: 0px; padding-left: 0px; margin: 10px 10px 0px 0px; display: inline; padding-right: 0px; border-top-width: 0px" border="0" alt="image" src="http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_30D32CA4.png" width="463" height="248" />][3]

Cuando se pulsa esta opción **se crea una carpeta .nuget** en la raíz de la solución que contiene tres ficheros (NuGet.config, NuGet.exe y NuGet.targets). Y además se modificarán los ficheros de la solución para agregarles por un lado una entrada nueva dentro de <PropertyGroup>:

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:97aab908-4c3c-4cd9-a0df-fda45f0d1ac3" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
    <div style="background: #ddd; max-height: 300px; overflow: auto">
      <ol start="1" style="background: #000000; margin: 0 0 0 2em; padding: 0 0 0 5px;">
        <li>
          <span style="background:#1e1e1e;color:#808080"><</span><span style="background:#1e1e1e;color:#569cd6">RestorePackages</span><span style="background:#1e1e1e;color:#808080">></span><span style="background:#1e1e1e;color:#c8c8c8">true</span><span style="background:#1e1e1e;color:#808080"></</span><span style="background:#1e1e1e;color:#569cd6">RestorePackages</span><span style="background:#1e1e1e;color:#808080">></span>
        </li>
      </ol>
    </div></p>
  </div></p>
</div>

Y la ejecución de la tarea para que NuGet descargue los paquetes al compilar la solución:

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:2c201af6-2d91-4b83-a758-bd1a88d7d106" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
    <div style="background: #ddd; max-height: 300px; overflow: auto">
      <ol start="1" style="background: #000000; margin: 0 0 0 2em; padding: 0 0 0 5px;">
        <li>
          <span style="background:#1e1e1e;color:#808080"><</span><span style="background:#1e1e1e;color:#569cd6">Import</span><span style="background:#1e1e1e;color:#808080"> </span><span style="background:#1e1e1e;color:#92caf4">Project</span><span style="background:#1e1e1e;color:#808080">="</span><span style="background:#1e1e1e;color:#c8c8c8">$(SolutionDir)&#92;.nuget&#92;NuGet.targets</span><span style="background:#1e1e1e;color:#808080">" </span><span style="background:#1e1e1e;color:#92caf4">Condition</span><span style="background:#1e1e1e;color:#808080">="</span><span style="background:#1e1e1e;color:#c8c8c8">Exists('$(SolutionDir)&#92;.nuget&#92;NuGet.targets')</span><span style="background:#1e1e1e;color:#808080">" /></span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#808080"><</span><span style="background:#1e1e1e;color:#569cd6">Target</span><span style="background:#1e1e1e;color:#808080"> </span><span style="background:#1e1e1e;color:#92caf4">Name</span><span style="background:#1e1e1e;color:#808080">="</span><span style="background:#1e1e1e;color:#c8c8c8">EnsureNuGetPackageBuildImports</span><span style="background:#1e1e1e;color:#808080">" </span><span style="background:#1e1e1e;color:#92caf4">BeforeTargets</span><span style="background:#1e1e1e;color:#808080">="</span><span style="background:#1e1e1e;color:#c8c8c8">PrepareForBuild</span><span style="background:#1e1e1e;color:#808080">"></span>
        </li>
        <li>
            <span
 style="background:#1e1e1e;color:#808080"><</span><span style="background:#1e1e1e;color:#569cd6">PropertyGroup</span><span style="background:#1e1e1e;color:#808080">></span>
        </li>
        <li>
              <span style="background:#1e1e1e;color:#808080"><</span><span style="background:#1e1e1e;color:#569cd6">ErrorText</span><span style="background:#1e1e1e;color:#808080">></span><span style="background:#1e1e1e;color:#c8c8c8">This project references NuGet package(s) that are missing on this computer. Enable NuGet Package Restore to download them.  For more information, see http://go.microsoft.com/fwlink/?LinkID=322105. The missing file is {0}.</span><span style="background:#1e1e1e;color:#808080"></</span><span style="background:#1e1e1e;color:#569cd6">ErrorText</span><span style="background:#1e1e1e;color:#808080">></span>
        </li>
        <li>
            <span style="background:#1e1e1e;color:#808080"></</span><span style="background:#1e1e1e;color:#569cd6">PropertyGroup</span><span style="background:#1e1e1e;color:#808080">></span>
        </li>
        <li>
            <span style="background:#1e1e1e;color:#808080"><</span><span style="background:#1e1e1e;color:#569cd6">Error</span><span style="background:#1e1e1e;color:#808080"> </span><span style="background:#1e1e1e;color:#92caf4">Condition</span><span style="background:#1e1e1e;color:#808080">="</span><span style="background:#1e1e1e;color:#c8c8c8">!Exists('$(SolutionDir)&#92;.nuget&#92;NuGet.targets')</span><span style="background:#1e1e1e;color:#808080">" </span><span style="background:#1e1e1e;color:#92caf4">Text</span><span style="background:#1e1e1e;color:#808080">="</span><span style="background:#1e1e1e;color:#c8c8c8">$([System.String]::Format('$(ErrorText)', '$(SolutionDir)&#92;.nuget&#92;NuGet.targets'))</span><span style="background:#1e1e1e;color:#808080">" /></span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#808080"></</span><span style="background:#1e1e1e;color:#569cd6">Target</span><span style="background:#1e1e1e;color:#808080">></span>
        </li>
      </ol>
    </div></p>
  </div></p>
</div>

De este modo al compilar la solución NuGet se descargará los paquetes que no existan de forma automática. Habilitar esta opción te marcará automáticamente la primera de las checkboxes que habíamos desmarcado antes en Tools –> Options –> NuGet Package Manager.

Así, ahora, una de las preguntas más recurrentes sobre NuGet (¿Tengo que subir los paquetes a mi sistema de control de código fuente?) se respondía ahora diciendo que si los querías subir podías hacerlo sin problemas pero que si no, no era necesario siempre y cuando la solución tuviese habilitada la opción de “Package Restore”. En este último caso la carpeta .nuget si que debías subirla.

En principio con esos dos modos cubrimos la totalidad de los escenarios pero con la versión 2.7 de NuGet agregaron un tercer modo de funcionamiento. Dicho tercer modo es básicamente el modo anterior que acabamos de describir **pero automatizado**. Ya **no** tenemos que hacer nada, excepto marcar la segunda de las checkboxes que hemos desmarcado al principio (y es que este es, a partir de la versión 2.7, el modo por defecto de NuGet).

Para verla en acción marca la segunda checkbox, y luego borra la carpeta .nuget. Luego recompila la solución y recibirás un error: “_This project references NuGet package(s) that are missing on this computer. Enable NuGet Package Restore to download them.&#160; For more information, see_ [_http://go.microsoft.com/fwlink/?LinkID=322105_][4]_. The missing file is XXX.nugetNuget.targets_”. Este error se da porque a pesar de que hemos borrado la carpeta .nuget, tenemos los proyectos todavía configurados para que la usen. Así, que no toca otra: abrir en modo texto los ficheros de proyecto y eliminar las líneas que se nos añadieron antes.

Una vez hecho esto, recargas los proyectos y al recompilar automáticamente NuGet descargará los paquetes. La ventaja de este modo de funcionamiento respecto al anterior es que no es intrusivo: no requiere modificar los ficheros de proyecto.

**Si actualmente usas NuGet 2.7 o superior (que es de esperar que sí) y tienes la carpeta .nuget en tu repositorio de código fuente lo mejor que puedes hacer es eliminarla**. Y luego modificar los proyectos para quitar las líneas indicadas anteriormente. Y con la segunda checkbox marcada (que es como está por defecto) ya tienes la descarga de paquetes automatizada. Por si tienes alguna duda sobre lo que tienes que eliminar en los ficheros de proyecto (aunque son las líneas mencionadas antes), <a href="http://docs.nuget.org/consume/package-restore/migrating-to-automatic-package-restore" target="_blank" rel="noopener noreferrer">todo el proceso está documentado en la propia web de NuGet</a>. Si usas team build también son necesarias pequeñas modificaciones en TFS2012 o anterior (en TFS2013 así como Visual Studio Online o bien deploys en Azure web sites el proceso está ya integrado). De nuevo tienes toda la <a href="http://docs.nuget.org/consume/package-restore/team-build" target="_blank" rel="noopener noreferrer">documentación en la web de NuGet sobre como configurar el team build</a>.

De todos modos que tengas habilitada la descarga de paquetes automatizada no te impide colocar los paquetes (la carpeta packages) en el repositorio de control de código fuente: es una opción personal. Colocarlos en el sistema de control de código fuente te evita una dependencia con el propio NuGet (que aunque se cae pocas veces, a veces lo hace). Hace tiempo Juanma escribió en su blog <a href="http://blog.koalite.com/2014/07/el-lado-oscuro-de-los-gestores-de-paquetes/" target="_blank" rel="noopener noreferrer">un post sobre los peligros de depender del gestor de paquetes</a>. Hay soluciones más elaboradas como no tener los paquetes en el control de código fuente pero usar un servidor de NuGet corporativo. Aquí ya, cada caso es un mundo.

**Proyectos en varias soluciones**

Vale, eso es un poco más frustrante y es un aviso más que otra cosa: si tienes un proyecto con paquetes gestionados por NuGet y este proyecto lo tienes en varias soluciones, asegúrate de que todas las soluciones (los ficheros .sln) están en el mismo directorio. En caso contrario puedes tener problemas. Es lógico una vez se entiende que hace NuGet y realmente es difícil que pueda hacer otra cosa que la que hace, así que bueno… es algo a tener en cuenta.

Vamos a reproducirlo paso a paso, para entender que ocurre. Para ello crea un directorio, yo lo he llamado nuroot. Luego crea otra carpeta (yo la he llamado folder1) dentro de nuroot:

[<img title="image" style="border-top: 0px; border-right: 0px; background-image: none; border-bottom: 0px; padding-top: 0px; padding-left: 0px; border-left: 0px; display: inline; padding-right: 0px" border="0" alt="image" src="http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_052262A8.png" width="188" height="71" />][5]

Ahora crea una solución de VS (una aplicación de consola) dentro de folder1 (yo la he llamado DemoProject). Una vez hecho esto agrega un paquete de NuGet a la solución (p. ej. DotNetZip). Ahora la estructura de paquete debe ser como sigue:

[<img title="image" style="border-top: 0px; border-right: 0px; background-image: none; border-bottom: 0px; padding-top: 0px; padding-left: 0px; border-left: 0px; display: inline; padding-right: 0px" border="0" alt="image" src="http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_798CA568.png" width="397" height="224" />][6]

El directorio packages **está al nivel de la solución**, y si miras en el proyecto verás que la referencia al en
  
samblado (en el caso de DotNetZip el ensamblado se llama Ionic.Zip) apunta al directorio packages. De hecho la referencia se guarda relativa al fichero de proyecto (si abres el .csproj en modo texto lo verás):

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:77e43670-8fbf-49a8-a348-c2400a29f961" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
    <div style="background: #ddd; max-height: 300px; overflow: auto">
      <ol start="1" style="background: #000000; margin: 0 0 0 2em; padding: 0 0 0 5px;">
        <li>
          <span style="background:#1e1e1e;color:#808080"><</span><span style="background:#1e1e1e;color:#569cd6">Reference</span><span style="background:#1e1e1e;color:#808080"> </span><span style="background:#1e1e1e;color:#92caf4">Include</span><span style="background:#1e1e1e;color:#808080">="</span><span style="background:#1e1e1e;color:#c8c8c8">Ionic.Zip</span><span style="background:#1e1e1e;color:#808080">"></span>
        </li>
        <li>
            <span style="background:#1e1e1e;color:#808080"><</span><span style="background:#1e1e1e;color:#569cd6">HintPath</span><span style="background:#1e1e1e;color:#808080">></span><span style="background:#1e1e1e;color:#c8c8c8">..&#92;packages&#92;DotNetZip.1.9.3&#92;lib&#92;net20&#92;Ionic.Zip.dll</span><span style="background:#1e1e1e;color:#808080"></</span><span style="background:#1e1e1e;color:#569cd6">HintPath</span><span style="background:#1e1e1e;color:#808080">></span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#808080"></</span><span style="background:#1e1e1e;color:#569cd6">Reference</span><span style="background:#1e1e1e;color:#808080">></span>
        </li>
      </ol>
    </div></p>
  </div></p>
</div>

Perfecto. Ahora crea otra solución vacía (New Project –> Other Project Types –> Visual Studio Solution –> Empty Solution) y dale el nombre que quieras. Yo la he llamado SecondSolution. Lo importante **es que la crees en nuroot, no en folder1**. Por defecto VS crea un directorio para la solución, pero vamos a eliminarlo. Ve a nurootSecondSolution y mueve el fichero SecondSolution.sln a nuroot. Luego borra el directorio SecondSoution. En este punto la estructura de directorios es pues la misma de antes, con la salvedad de que en la carpeta nuroot hay el fichero SecondSolution.sln.

Finalmente agrega el proyecto existente (DemoProject) a la solución SecondSolution. ¡Una vez cargues el proyecto verás el mensaje de que faltan paquetes de NuGet! Dale a Restore para que NuGet se descargue los paquetes faltantes y la estructura de directorios será la siguiente:

[<img title="image" style="border-top: 0px; border-right: 0px; background-image: none; border-bottom: 0px; padding-top: 0px; padding-left: 0px; border-left: 0px; display: inline; padding-right: 0px" border="0" alt="image" src="http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_1BE43AE2.png" width="409" height="288" />][7]

Ten presente que NuGet funciona **a nivel de solución**. Cuando hemos agregado el proyecto DemoProject a la segunda solución, NuGet ha examinado el fichero packages.config del proyecto y ha visto una referencia a DotNetZip. Luego ha examinado el directorio packages **de la solución**. Y al estar esta otra solución en otro directorio que la anterior, NuGet no encuentra el directorio packages, y por lo tanto asume que debe descargarse los paquetes. Y al descargarlos es cuando nos aparece el otro directorio packages ahora colgando de nuroot (el directorio donde tenemos SecondSolution.sln).

Si compilas el proyecto todo funcionará pero hay un tema importante ahí. El proyecto DemoProject **ya contenía una referencia al ensamblado de DotNetZip** y al existir dicha referencia NuGet no la modificará. Es decir, la referencia sigue apuntando donde apuntaba inicialmente (nurootfolder1DemoProjectPackages).

Cierra VS y **borra los dos directorios packages**. Con esto simulas lo que le ocurriría a alguien que se descargase el código fuente (suponiendo que los paquetes no están subidos en él). Ahora carga SecondSolution.sln otra vez y de nuevo verás que NuGet dice que faltan paquetes (obvio, pues los hemos borrado todos). Restaura de nuevo y verás como NuGet crea otra vez el directorio nurootpackages. Pero el fichero csproj sigue teniendo la referencia a nurootfolder1DemoProject **y por lo tanto no encontrará la referencia.** Es decir, el código no compilará. Para que te compile debes abirlo con la solución DemoProject.sln, restaurar paquetes (o compilar simplemente, recuerda que al compilar se restauran automáticamente) y entonces ya te compilará (desde ambas soluciones).

En este escenario quizá no te parezca tan grave porque total, haces un readme.txt y que diga “Abrir primero DemoProject.sln” y listos. Hombre, es feo y un poco chapuza pero bueno…

Pero el problema lo tienes si luego añades otro paquete de NuGet al proyecto cuando lo tienes abierto con la solución SecondSolution.sln. P. ej. yo he instalado el CommandLineParser. Por supuesto este paquete está instalado en nurootpackages y la referencia del proyecto apunta a este directorio. Observa como han quedado las referencias del proyecto:

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:d94d5f3a-ff0b-4819-aef1-688296efecf5" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
    <div style="background: #ddd; max-height: 300px; overflow: auto">
      <ol start="1" style="background: #000000; margin: 0 0 0 2em; padding: 0 0 0 5px;">
        <li>
          <span style="background:#1e1e1e;color:#808080"><</span><span style="background:#1e1e1e;color:#569cd6">Reference</span><span style="background:#1e1e1e;color:#808080"> </span><span style="background:#1e1e1e;color:#92caf4">Include</span><span style="background:#1e1e1e;color:#808080">="</span><span style="background:#1e1e1e;color:#c8c8c8">CommandLine</span><span style="background:#1e1e1e;color:#808080">"></span>
        </li>
        <li>
            <span style="background:#1e1e1e;color:#808080"><</span><span style="background:#1e1e1e;color:#569cd6">HintPath</span><span style="background:#1e1e1e;color:#808080">></span><span style="background:#1e1e1e;color:#c8c8c8">..&#92;..&#92;..&#92;packages&#92;CommandLineParser.1.9.71&#92;lib&#92;net45&#92;CommandLine.dll</span><span style="background:#1e1e1e;color:#808080"></</span><span style="background:#1e1e1e;color:#569cd6">HintPath</span><span style="background:#1e1e1e;color:#808080">></span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#808080"></</span><span style="background:#1e1e1e;color:#569cd6">Reference</span><span style="background:#1e1e1e;color:#808080">></span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#808080"><</span><span style="background:#1e1e1e;color:#569cd6">Reference</span><span style="background:#1e1e1e;color:#808080"> </span><span style="background:#1e1e1e;color:#92caf4">Include</span><span style="background:#1e1e1e;color:#808080">="</span><span style="background:#1e1e1e;color:#c8c8c8">Ionic.Zip</span><span style="background:#1e1e1e;color:#808080">"></span>
        </li>
        <li>
            <span style="background:#1e1e1e;color:#808080"><</span><span style="background:#1e1e1e;color:#569cd6">HintPath</span><span style="background:#1e1e1e;color:#808080">></span><span style="background:#1e1e1e;color:#c8c8c8">..&#92;packages&#92;DotNetZip.1.9.3&#92;lib&#92;net20&#92;Ionic.Zip.dll</span><span style="background:#1e1e1e;color:#808080"></</span><span style="background:#1e1e1e;color:#569cd6">H<br /> intPath</span><span style="background:#1e1e1e;color:#808080">></span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#808080"></</span><span style="background:#1e1e1e;color:#569cd6">Reference</span><span style="background:#1e1e1e;color:#808080">></span>
        </li>
      </ol>
    </div></p>
  </div></p>
</div>

Por lo tanto ahora si borras los directorios packages, debes abrir el proyecto en ambas soluciones y compilarlas en ambas (para forzar la restauración de paquetes). La primera solución que compiles te dará un error (le faltará el paquete que se añadió a través de la otra solución), da igual el orden en que lo hagas. La segunda que compiles si que compilará bien.

Al final terminarás con ambos paquetes instalados en ambos directorios packages pero solo se usará uno de cada (el referenciado por el proyecto).

Igual no te parece muy grave pero si tienes una build que compila una de esas soluciones dala por perdida: cada vez que se ejecuta la build se parte d un entorno nuevo, por lo que la build no compilará, está rota.

La mejor solución para ello es simplemente tenerlo presente: evita que un mismo proyecto esté en varias soluciones localizadas en directorios distintos (si las soluciones, los ficheros .sln, están todos en la misma carpeta no hay problema porque el directorio packages de todas ellas es el mismo).

Espero que el post os haya resultado interesante!

Saludos!

 [1]: http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_58E34B12.png
 [2]: http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_6166AA5C.png
 [3]: http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_4A3B2FDE.png
 [4]: http://go.microsoft.com/fwlink/?LinkID=322105
 [5]: http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_45587C22.png
 [6]: http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_72D99BE5.png
 [7]: http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_1531315F.png