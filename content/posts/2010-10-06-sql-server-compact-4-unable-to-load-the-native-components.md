---
title: 'SQL Server Compact 4: Unable to load the native components'
author: eiximenis

date: 2010-10-06T12:03:57+00:00
geeks_url: /?p=1536
geeks_visits:
  - 2627
geeks_ms_views:
  - 1556
categories:
  - Uncategorized

---
Buenas! Un post ligerito, ligerito 🙂

Ando esos días probando cosillas con SQL Server Compact 4 (que os podéis descargar desde [su página de descargas][1] o bien usando [Web Platofrm Installer][2]).

Habiendo probado código con [EF 4 Code First][3] que estaba funcionando bien, empecé otras pruebas usando el proveedor propio de ADO.NET. Así que agregué una referencia a la _System.Data.SqlServerCe.dll_ que viene con SQL Server Compact 4, y cree una conexión:

<div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
  <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; background-color: #f4f4f4; margin: 0em; border-left-style: none; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; border-right-style: none; font-size: 8pt; overflow: visible; padding-top: 0px" id="codeSnippet">SqlCeConnection con = <span style="color: #0000ff">new</span> SqlCeConnection()</pre>
  
  <p>
    </div> 
    
    <p>
      Pues bien, al crear la conexión (fijaos que aquí no hay cadena de conexión, todavía no me conecto a ningún .sdf) ya me salta una <em>SqlCeException</em>:
    </p>
    
    <p>
      <em>Unable to load the native components of SQL Server Compact corresponding to the ADO.NET provider of version 8402. Install the correct version of SQL Server Compact. Refer to KB article 974247 for more details.</em>
    </p>
    
    <p>
      Hombre, es de agradecer que la propia excepción te diga un KB que consultar, así que ni corto ni perezoso me dirigo al <a href="http://support.microsoft.com/kb/974247">KB974247</a>, pero nada de lo que dice es aplicable en mi caso.
    </p>
    
    <p>
      Descartando el hecho de que sea una mala instalación (puesto que con EF 4 Code First era capaz de conectarme) me puse a mirar que podría estar pasando… y la respuesta final es que <strong>se deben copiar los binarios de SQL Server Compact 4 a la carpeta bin</strong> de la aplicación.
    </p>
    
    <p>
      En mi caso SQL Server Compact 4 se ha instalado en <em>C:Program FilesMicrosoft SQL Server Compact Editionv4.0</em> y los binarios que he copiado son las carpetas <em>x86</em> y <em>amd64</em> que están en la carpeta <em>Private</em>. De hecho en mi caso me basta con la carpeta x86 puesto que mi máquina es de 32 bits.
    </p>
    
    <p>
      Así pues en el bin/Debug de mi aplicación tengo mi aplicación, la <em>System.Data.SqlServerCe.dll</em> y la carpeta x86 con los binarios de SQL Server Compact 4.
    </p>
    
    <p>
      Y entonces me funciona todo correctamente.
    </p>
    
    <p>
      Un saludo!
    </p>
    
    <p>
      PD: Y de regalo, os dejo (por si no lo conocíais) el enlace de <a href="http://sqlcetoolbox.codeplex.com/">una extensión, llamada SqlCeToolbox, para vs2010 para poder tratar con ficheros .sdf de la versión 4</a> (sinó la <em>otra</em> manera que hay es usar WebMatrix que tiene un diseñador de tablas, pero me parece más potente esta exensión).
    </p>

 [1]: http://www.microsoft.com/downloads/en/details.aspx?FamilyID=0d2357ea-324f-46fd-88fc-7364c80e4fdb&displaylang=en
 [2]: http://www.microsoft.com/web/downloads/platform.aspx
 [3]: http://www.microsoft.com/downloads/en/details.aspx?FamilyID=4e094902-aeff-4ee2-a12d-5881d4b0dd3e&displaylang=en