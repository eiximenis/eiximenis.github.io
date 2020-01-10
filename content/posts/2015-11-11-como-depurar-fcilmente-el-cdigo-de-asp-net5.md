---
title: Como depurar fácilmente el código de ASP.NET5

author: eiximenis

date: 2015-11-11T10:09:55+00:00
geeks_url: /?p=1710
geeks_visits:
  - 741
geeks_ms_views:
  - 1492
categories:
  - asp.net 5

---
¡Muy buenas! Para los que andamos trasteando con versiones alfas y betas con nula o poca documentación, poder depurar el código fuente de las librerías es una manera muy buena de ver _qué_ hace y _como_ lo hace. Cierto, solo leyendo el código fuente se puede aprender mucho, pero poder depurarlo paso a paso es todavía más útil.

El primer paso es, por supuesto, disponer del código fuente. En según que librerías eso no es posible, aunque siempre se podía tirar de los desensambladores (por más que eso pueda no ser legal). El problema de usar los desensambladores es que el código fuente obtenido no siempre es muy legible y que por otro lado no puedes depurarlo a no ser que tu desensemblador soporte la creación de PDBs.

Depurar ASP.NET MVC nunca ha sido un problema importante, en tanto el código fuente estaba disponible. Podías descargarte el código fuente de ASP.NET MVC, compilarlo en alguna carpeta local, pongamos c:mvc, junto con sus PDBs. Luego tenías que modificar tu proyecto web, quitar las referencias a ASP.NET MVC _reales_ y sustituirlas por las versiones locales con PDBs. Con eso, podías depurar ASP.NET MVC sin muchos problemas. Pero era un poco peñazo el tener que sustituir las referencias reales por las tuyas compiladas, lo que podía redundar en algún cambio adicional de configuración (por estar firmadas las reales y no estarlos, o estarlo con otra clave, las locales).

En ASP.NET5 el proceso se ha vuelto ridículamente simple. El proceso es como sigue:

  1. Clona los repositorios de ASP.NET5 que quieras. P. ej. yo he clonado algunos de ellos en la carpeta c:tfsaspnet5.
  2. Crea un proyecto web con VS2015 que use ASP.NET5. O abre un proyecto existente, eso es lo de menos.
  3. Abre el fichero global.json que está en _Solution Items_

Este fichero contiene en “projects”, la lista de carpetas en las que VS2015 sabe que hay código que debe incorporar como parte de tu proyecto (por defecto las carpetas _src_ y _test_). Añade la carpeta base en la cual has clonado los repos de aspnet5 que quieras depurar:

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:1fa13c26-7121-4a4a-bc3f-d58c7a1e30e5" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
    <div style="background: #ddd; max-height: 300px; overflow: auto">
      <ol start="1" style="background: #ffffff; margin: 0 0 0 2em; padding: 0 0 0 5px;">
        <li>
          {
        </li>
        <li>
            <span style="background:#ffffff;color:#000000"></span><span style="background:#ffffff;color:#2e75b6">"projects"</span><span style="background:#ffffff;color:#000000">: [ </span><span style="background:#ffffff;color:#a31515">"src"</span><span style="background:#ffffff;color:#000000">, </span><span style="background:#ffffff;color:#a31515">"test"</span><span style="background:#ffffff;color:#000000">, </span><span style="background:#ffffff;color:#a31515">"c:/tfs/aspnet5/mvc/src"</span><span style="background:#ffffff;color:#000000"> ],</span>
        </li>
        <li>
            <span style="background:#ffffff;color:#000000"></span><span style="background:#ffffff;color:#2e75b6">"sdk"</span><span style="background:#ffffff;color:#000000">: {</span>
        </li>
        <li>
              <span style="background:#ffffff;color:#000000"></span><span style="background:#ffffff;color:#2e75b6">"version"</span><span style="background:#ffffff;color:#000000">: </span><span style="background:#ffffff;color:#a31515">"1.0.0-beta8"</span>
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
</div>

En mi caso en c:tfsaspnet5mvc he clonado el repo de ASP.NET MVC6 así lo añado en projects (debo añadir el /src final que es donde hay los proyectos). Eso automáticamente me añade los proyectos de ASP.NET MVC6 a mi solución:

[<img title="image" style="border-top: 0px; border-right: 0px; background-image: none; border-bottom: 0px; padding-top: 0px; padding-left: 0px; border-left: 0px; margin: 10px 10px 0px 0px; display: inline; padding-right: 0px" border="0" alt="image" src="http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_61D25BA8.png" width="405" height="462" />][1]

Además en el _solution explorer_ VS2015 nos indica, con un icono distinto, qué referencias ha cargado en local:

[<img title="image" style="border-top: 0px; border-right: 0px; background-image: none; border-bottom: 0px; padding-top: 0px; padding-left: 0px; border-left: 0px; margin: 10px 10px 0px 0px; display: inline; padding-right: 0px" border="0" alt="image" src="http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_2A1FA178.png" width="598" height="418" />][2]

Ahora puedes navegar por el código fuente y colocar breakpoints sin ningún problema… ejecutar tu aplicación y ¡depurar!

[<img title="image" style="border-top: 0px; border-right: 0px; background-image: none; border-bottom: 0px; padding-top: 0px; padding-left: 0px; border-left: 0px; display: inline; padding-right: 0px" border="0" alt="image" src="http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_27797678.png" width="644" height="348" />][3]

Eso sí, la clave para que te funcione es que tengas los repos sincronizados con la versión que estás usando. P. ej. si en el project.json estás referenciando la versión 1.0.0-beta8 debes tener los repos con el código fuente que se corresponda a dicha versión (todas las versiones tienen un _tag_ de Git asociado, así que puedes obtener la que quieras). También que estés usando el entorno de ejecución correspondiente (no te funcionará si intentas depurar beta8 usando el dnx de beta5).

Observa que no es necesario cambio alguno en nuestro proyecto. Cuando hayas depurado, basta con que elimines del global.json el directorio con los fuentes y tu aplicación pasará a referenciar los paquetes NuGet oficiales.

Más fácil, ¡imposible!

Saludos!

PD: Antes de que me deis las gracias, se las dáis a **Hugo Biarge** (<a href="http://twitter.com/hbiarge" target="_blank" rel="noopener noreferrer">@hbiarge</a>) que fue quien me lo contó. En este post hay más info (un poco desactualizada, ojo): [http://blogs.msdn.com/b/webdev/archive/2015/02/06/debugging-asp-net-5-framework-code-using-visual-studio-2015.aspx][4]

 [1]: http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_3E1412AC.png
 [2]: http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_3D40CE24.png
 [3]: http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_3EA4F0F6.png
 [4]: http://blogs.msdn.com/b/webdev/archive/2015/02/06/debugging-asp-net-5-framework-code-using-visual-studio-2015.aspx "http://blogs.msdn.com/b/webdev/archive/2015/02/06/debugging-asp-net-5-framework-code-using-visual-studio-2015.aspx"