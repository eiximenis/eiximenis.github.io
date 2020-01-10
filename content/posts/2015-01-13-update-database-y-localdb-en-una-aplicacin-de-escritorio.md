---
title: update-database y LocalDb en una aplicación de escritorio

author: eiximenis

date: 2015-01-13T12:13:10+00:00
geeks_url: /?p=1690
geeks_visits:
  - 722
geeks_ms_views:
  - 733
categories:
  - Uncategorized

---
Estos días he estado desarrollando un aplicación de escritorio (wpf aunque eso es lo de menos) que va a hacer uso de LocalDb para guardar datos. Ciertamente no es un escenario muy habitual, ya que al instalar la aplicación en un ordenador cliente se requiere instalar LocalDb pero en este caso eso era asumible. Otras opciones para escritorio podrían pasar por usar algúna BBDD de proceso (como VistaDb o similares).

“Teoricamente” eso no debería diferir del workflow usado en aplicaciones web. Supongamos que tenemos nuestro proyecto con el contexto de EF creado y hemos habilitado migrations (enable-migrations) para controlar las modificaciones del esquema.

Supongamos una cadena de conexión que use AttachDbFilename:

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:87085c80-8ae3-469c-b67b-d95478351a3f" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
    <div style="background: #ddd; max-height: 300px; overflow: auto">
      <ol start="1" style="background: #1d1d1d; margin: 0 0 0 2em; padding: 0 0 0 5px;">
        <li>
          <span style="background:#1e1e1e;color:#808080"><</span><span style="background:#1e1e1e;color:#569cd6">add</span><span style="background:#1e1e1e;color:#808080"> </span><span style="background:#1e1e1e;color:#92caf4">name</span><span style="background:#1e1e1e;color:#808080">="</span><span style="background:#1e1e1e;color:#c8c8c8">DbClient</span><span style="background:#1e1e1e;color:#808080">" </span><span style="background:#1e1e1e;color:#92caf4">connectionString</span><span style="background:#1e1e1e;color:#808080">="</span><span style="background:#1e1e1e;color:#c8c8c8">Data Source=(LocalDb)&#92;v11.0;AttachDbFilename=|DataDirectory|&#92;App_Data&#92;clientdb_v1.mdf;Initial Catalog=clientdb-v1;Integrated Security=True</span><span style="background:#1e1e1e;color:#808080">" </span><span style="background:#1e1e1e;color:#92caf4">providerName</span><span style="background:#1e1e1e;color:#808080">="</span><span style="background:#1e1e1e;color:#c8c8c8">System.Data.SqlClient</span><span style="background:#1e1e1e;color:#808080">" /></span>
        </li>
      </ol>
    </div></p>
  </div></p>
</div></p> 

Se puede observar que se usa |DataDirectory| al igual que en una aplicación web. Esta es una variable de “entorno” que entiende el .NET Framework (a partir de la 4.0.2) y <a href="http://blogs.msdn.com/b/smartclientdata/archive/2005/08/26/456886.aspx" target="_blank" rel="noopener noreferrer">cuyos valores están definidos de la siguiente manera, según se cuenta en este post</a>.

_By default, the |DataDirectory| variable will be expanded as follow:_

 _&#8211; For applications placed in a directory on the user machine, this will be the app&#8217;s (.exe) folder.   
&#8211; For apps running under ClickOnce, this will be a special data folder created by ClickOnce   
&#8211; For Web apps, this will be the App_Data folder_

_Under the hood, the value for |DataDirectory| simply comes from a property on the app domain. It is possible to change that value and override the default behavior by doing this:_

_&#160;&#160;&#160;&#160;&#160; AppDomain.CurrentDomain.SetData("DataDirectory", newpath)_

Por lo tanto tenemos que para aplicaciones de escritorio |DataDirectory| se mapea al directorio donde está la BBDD según este post. He de decir que **en mi experiencia eso NO es cierto. Se mapea a la subcarpeta App_Data de la carpeta donde está el ejecutable** (así, si tenemos el ejecutable en c:xxxbinDebug se mapeará a c:xxxbinDebugApp_Data). Quizá es un cambio posterior a la publicación de este post.

En ejecución se espera que la BBDD (el fichero .mdf) esté en este directorio. Perfecto, pero ahora tenemos el problema de las herramientas de VS. Antes de nada, agregamos un archivo .mdf a nuestra solución (en el **startup** project):

[<img title="image" style="border-top: 0px; border-right: 0px; background-image: none; border-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; border-left: 0px; display: inline; padding-right: 0px" border="0" alt="image" src="http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_0EDBE801.png" width="188" height="59" />][1]

Con esto ya podemos ejecutar update-database y transferir todas las migraciones a este fichero .mdf.

Pero ahora tenemos un problema, y es que los ficheros .mdf son tratados como un fichero “content” tradicional, es decir que lo máximo que VS puede hacer es copiarlos al output folder:

[<img title="image" style="border-top: 0px; border-right: 0px; background-image: none; border-bottom: 0px; padding-top: 0px; padding-left: 0px; border-left: 0px; display: inline; padding-right: 0px" border="0" alt="image" src="http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_7980C349.png" width="478" height="275" />][2]

Si marcamos:

  1. Copy always: Cada vez que compilemos el proyecto se copiará el .mdf hacia el directorio de ejecución. Resultado: perderemos todos los datos, dado que el .mdf de la solución está vacío (solo tiene el esquema generado por update-database).
  2. Copy if newer: Solo se copiará en caso que el .mdf de la solución sea más nuevo, lo que solo ocurrirá en el caso de cambios de esquema. Entonces en cada cambio de esquema perdemos los datos.
  3. Do not copy: El fichero .mdf no se copia en el directorio de salida, lo que implica que la aplicación… no lo encontrará.

Ninguna de las 3 opciones es deseable. Esto en aplicaciones web no ocurre, debido a la forma en como DataDirectory es gestionado por ASP.NET, pero ahora estamos en escritorio 🙁

Una posible solución es olvidarnos de la copia (Do not copy) y hacer que DataDirectory apunte al fichero .mdf de la solución. Para ello se puede usar AppDomain.CurrentDomain.SetData para conseguir el efecto deseado:

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:c664cd5e-91bb-4406-8d76-7c030b5bbb0d" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
    <div style="background: #ddd; max-height: 300px; overflow: auto">
      <ol start="1" style="background: #1d1d1d; margin: 0 0 0 2em; padding: 0 0 0 5px;">
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">var</span><span style="background:#1e1e1e;color:#dcdcdc"> path </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#4ec9b0">Path</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">GetDirectoryName(</span><span style="background:#1e1e1e;color:#4ec9b0">Assembly</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">GetExecutingAssembly()</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">Location) </span><span style="background:#1e1e1e;color:#b4b4b4">+</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#d69d85">@"&#92;..&#92;..&#92;MyClient&#92;App_Data"</span><span style="background:#1e1e1e;color:#dcdcdc">;</span>
        </li>
        <li style="background: #111111">
          <span style="backgro
und:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#4ec9b0">AppDomain</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">CurrentDomain</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">SetData(</span><span style="background:#1e1e1e;color:#d69d85">"DataDirectory"</span><span style="background:#1e1e1e;color:#dcdcdc">, path);</span>
        </li>
      </ol>
    </div></p>
  </div></p>
</div>

Básicamente obtenemos el directorio de ejecución del assembly y nos movemos hacia el directorio donde hay realmente el fichero en la solución.

Eso hace que ahora tanto VS al usar update-database como nuestra aplicación usen el mismo fichero .mdf. **Por supuesto en cuanto despleguemos de verdad la app deberemos utilizar otra técnica**. Porque así **dependemos de un fichero (el .mdf) que NO está desplegado en el directorio de salida**.

Una solución para ello es volver a poner el “Copy aways” (por lo que el .mdf se copiará cada vez, lo que ahora&#160; no es problema porque tiene datos y esquema) y ejecutar o no la llamada a AppDomain.CurrentDomain.SetData según sea de menester (si se ejecuta la llamada se usa el .mdf de la solución, en caso contrario se usa el .mdf localizado en el directorio de salida).

Un saludo!

 [1]: http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_788642BB.png
 [2]: http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_50E25742.png