---
title: Git para dummies (pero dummies, dummies eh?)
description: Git para dummies (pero dummies, dummies eh?)
author: eiximenis

date: 2011-01-18T11:19:55+00:00
geeks_url: /?p=1552
geeks_visits:
  - 18320
geeks_ms_views:
  - 5987
categories:
  - Uncategorized

---
**Disclaimer:** Ese post ni es, ni lo pretende, ser un tutorial de Git. Es simplemente las impresiones de alguien (yo) que ayer empezó a usar, por primera vez, Git. Seguro que hay gente que lee ese blog y que sabe **mucho, pero mucho** más de Git que yo… Así que sus comentarios y correciones serán bienvenidas! 🙂

Esos días (ayer, vamos :p) he empezado a usar Git. Para quien no lo conozca Git es un sistema de control de código fuente distribuído. A diferencia de TFS, VSS o SVN que son centralizados, en Git cada desarrollador tiene su propia copia entera del repositorio _en local_. Los cambios son propagados entre repositorios locales y pueden (o no) sincronizarse con un repositorio central.

La diferencia fundamental con TFS ó VSS es que en esos está claro _quien es el repositorio_ (el servidor). Eso **no** existe en Git. En Git _cada usuario es el repositorio_ y se sincronizan cambios _entre repositorios (usuarios)_. Opcionalmente puede usarse _un repositorio central_ pero parece que no es obligatorio.

Los cambios en Git _se propagan_ a través de esas operaciones:

  1. **commit:** Envia los datos del _working directory_ al repositorio _local_ 
  2. **push:** Sincroniza los cambios del repositorio _local_ a otro repositorio remoto. 
  3. **fetch:** Sincroniza los cambios de un repositorio remoto al repositorio _local_. 
  4. **pull:** Sincroniza los cambios de un repositorio remoto al working directory. 
  5. **checkout:** Actualiza el _workind directory_ con los datos del repositorio _local_. 

La siguiente imagen lo clarifica bastante:<img style="padding-left: ; padding-right: ; display: block; float: none; margin-left: auto; margin-right: auto; padding-top: " title="title" alt="Comandos de transporte de Git" src="http://osteele.com/images/2008/git-transport.png" />

<p align="center">
  Operaciones de Git (imagen original de <a href="http://osteele.com/" target="_blank" rel="noopener noreferrer">Oliver Steele</a> en <a title="http://www.gitready.com/beginner/2009/01/21/pushing-and-pulling.html" href="http://www.gitready.com/beginner/2009/01/21/pushing-and-pulling.html">http://www.gitready.com/beginner/2009/01/21/pushing-and-pulling.html</a>).
</p>

**Usar Git desde Visual Studio**

Para usar Git desde VS2010 he encontrado las <a href="http://sourceforge.net/projects/gitextensions/" target="_blank" rel="noopener noreferrer">Git Extensions</a> que instalan, no sólo un plugin para VS2010, sinó también el propio Git y clientes adicionales que pueden necesitarse (SSH o PuTTY si queremos conexiones seguras con nuestros repositorios).

Una vez instaladas nos aparecerá un menú nuevo llamado “Git” en el VS2010.

**Crear un repositorio local y rellenarlo**

Para empezar a trabajar con Git, lo primero es **crear un repositorio**. Recordad que los repositorios son _locales_, por lo que un repositorio es simplemente un directorio de nuestro disco. En **mi caso, tenía ya una solución de VS2010 en local que es la que quería empezar a compartir con Git.** Por ello lo que hice fue _crear un nuevo repositorio local_. Hay otra opción, que es _crear un repositorio local a partir de los datos de un repositorio remoto_ (<a href="http://www.kernel.org/pub/software/scm/git/docs/git-clone.html" target="_blank" rel="noopener noreferrer">git-clone</a>) pero todavía no lo he hecho. Si todo va como está planeado, el viernes tocará hacerlo, y ya escribiré al respecto!

Usando las Git Extensions crearnos nuestro propio repositorio es tan sencillo como usar la opción “Initialize new repository” y nos saldrá una ventana como la siguiente:

[<img style="background-image: none; border-right-width: 0px; margin: ; padding-left: 0px; padding-right: 0px; display: inline; border-top-width: 0px; border-bottom-width: 0px; border-left-width: 0px; padding-top: 0px" title="image" border="0" alt="image" src="http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_33DFEB38.png" width="487" height="133" />][1]

La opción normal es “Personal repository”. Un _Central repository_ es un repositorio sin _Working directory_, que sólo sirve para sincronizar datos.

Una vez entrada la carpeta (en este caso D:Gittest) **esa pasa a ser el directorio de trabajo (working directory)** para ese repositorio. Si abrimos la carpeta con el explorador de windows veremos que hay una carpeta en su interior llamada _.git_: ese es el repositorio local.

> **Nota:** Dado que el directorio donde inicializamos el repositorio local pasa a ser el directorio de tabajo, lo suyo es inicializar el repositorio local **en el mismo directorio** donde tenemos la solución de VS. Es decir, si la solución de VS la tenemos en el directorio C:projectssourcemyproject, ese sería el directorio que usariamos (recordad que el repositorio se crea en una subcarpeta llamada .git, por lo que no modificarà ni borrará nada de la carpeta que le digáis).

En mi caso en D:Gittest ya tenía una solución, por lo que el árbol de directorios me ha quedado:

[<img style="background-image: none; border-right-width: 0px; margin: ; padding-left: 0px; padding-right: 0px; display: inline; border-top-width: 0px; border-bottom-width: 0px; border-left-width: 0px; padding-top: 0px" title="image" border="0" alt="image" src="http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_10CF79C8.png" width="244" height="150" />][2]

Ahora vamos a hacer _commit_ de la solución al repositorio _local._ Antes que nada, pero debemos tener presente de que **no** queremos que todos los archivos que cuelgan del working directory (D:gittest) se guarden en el repositorio local. Git no entiende de _tipos_ de archivo, no sabe que es código fuente y que no. Existe un fichero **en el working directory** llamado .gitignore que sirve para indicar que ficheros no deben guardarse en el repositorio local al hacer un commit.

Por suerte editar este fichero con las Git Exensions, es trivial. Nos vamos al menú Git y seleccionamos “Edit .gitignore”. Nos aparecerá una ventana parecida a:

[<img style="background-image: none; border-right-width: 0px; margin: ; padding-left: 0px; padding-right: 0px; display: inline; border-top-width: 0px; border-bottom-width: 0px; border-left-width: 0px; padding-top: 0px" title="image" border="0" alt="image" src="http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_3AEF01E3.png" width="244" height="177" />][3]

A la izquierda el contenido del .gitignore (en este caso vacío, normal ya que ni existe el fichero). A la derecha un ejemplo de .gitignore adaptado a VS. Si pulsais el botón “add default” os copiará las entradas de la derecha a la izquierda:

[<img style="background-image: none; border-right-width: 0px; margin: ; padding-left: 0px; padding-right: 0px; display: inline; border-top-width: 0px; border-bottom-width: 0px; border-left-width: 0px; padding-top: 0px" title="image" border="0" alt="image" src="http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_6FCBE153
.png" width="244" height="177" />][4]

Fijaos que el .gitignore se ha confiugrado para evitar archivos como \*.exe, \*.pdb, etc, pero también directorios como TestResult\* (que usa VS para guardar los resultados de las pruebas unitarias) o _Resharper\* (que usa resharper para guardar sus configuraciones). Nosotros podríamos añadir más entradas y finalmente darle a “Save”. Eso nos creará el archivo .gitignore en nuestro _working directory_ (D:gittest).

Ahora estamos listos para hacer el _commit_ y rellenar por primera vez el repositorio local. Para ello, de nuevo nos vamos al menú Git y seleccionamos “Commit”. Nos aparecerá una ventana parecida a:

[<img style="background-image: none; border-right-width: 0px; margin: ; padding-left: 0px; padding-right: 0px; display: inline; border-top-width: 0px; border-bottom-width: 0px; border-left-width: 0px; padding-top: 0px" title="image" border="0" alt="image" src="http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_7C29DEAF.png" width="244" height="149" />][5]

Se divide en cuatro secciones:

  1. Izquierda superior: listado de operaciones pendientes de realizar (cambios entre el working directory y el repositorio local) 
  2. Izquierda inferior: listado de operaciones que se van a realizar (es decir cuales de los cambios del working directory queremos propagar al repositorio local). 
  3. Derecha superior: preview del archivo seleccionado en la parte (1). 
  4. Derecha inferior: comandos de Git. 

En este caso vamos a propagar todos los cambios, para ello pulsamos el botón “Staged Files” y en el menú desplegable marcamos “Stage all”. Con eso todos los ficheros de la izquierda superior pasarán a la izquiera inferior. Ahora vamos a realizar el _commit_ (si quisieramos podríamos hacer también un _push_ a un repositorio remoto pero todavía no hemos configurado ninguno).&#160; Así que entramos un mensaje de commit en la parte derecha inferior (p.ej. commit inicial) y le damos al botón “Commit”. Git Extensions nos mostrará al final un diálogo con el resumen de lo hecho:

[<img style="background-image: none; border-right-width: 0px; margin: ; padding-left: 0px; padding-right: 0px; display: inline; border-top-width: 0px; border-bottom-width: 0px; border-left-width: 0px; padding-top: 0px" title="image" border="0" alt="image" src="http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_5F6043CD.png" width="244" height="116" />][6]

**Añadir cambios**

Bien, una vez hemos hecho el commit podemos seguir trabajando con VS2010 sin necesidad de hacer nada especial. Olvidaros de conceptos como “proteger” o “desproteger” de TFS o VSS. Simplemente trabajáis y modificáis el contenido del directorio de trabajo.

Cuando decidáis propagar los cambios del directorio de trabajo al repositorio local, repetís la operación de antes: Git –> Commit.

**Repositorio remoto**

Bien, ha llegado la hora de configurar un repositorio remoto. Para ello, lo primero que necesitamos es tener acceso a un repositorio remoto. **Hay varios proveedores** de repositorios Git en internet, entre los que destaca <a href="https://github.com/" target="_blank" rel="noopener noreferrer">GitHub</a>. GitHub pero está limitado a proyectos open source. Si neceistáis un hosting de Git para proyectos **no open source** hay varios de pago (con algunas versiones gratuitas). En esta consulta de StackOverflow hablan de ello: <a href="http://stackoverflow.com/questions/109440/best-git-repository-hosting-for-commercial-project" target="_blank" rel="noopener noreferrer">Best git repository hosting for commercial project</a>.

> **Nota:** A título informativo os diré que yo me dí de alta en **<a href="http://www.assembla.com/catalog/tag/Free" target="_blank" rel="noopener noreferrer">el plan gratuito de Assembla</a>**, que te da 2GB de espacio Git.

La configuración de un repositorio remoto dependerá de donde lo tengamos, pero básicamente son dos pasos muy simples:

  1. Generar el par clave pública-privada para ssh o PuTTY y subir la clave pública al repositorio (si queremos acceso seguro).
  2. Ponerle a Git Extensions la URL del repositorio remoto.

Vamos a ver una demostración. Para independizarnos de cualquier proveedor, vamos a crear _otro repositorio de Git_ en nuestra máquina (podríamos hacerlo en una carpeta compartida p.ej.) y lo usaremos como repositorio remoto.

Para ello de nuevo nos vamos al menú Git y le damos a “Initialize new repository” y ahora marcamos la opción de Central Repository. Y ponemos cualquier path nuevo que creemos (en mi caso he creado una carpeta D:Remote):

[<img style="background-image: none; border-bottom: 0px; border-left: 0px; margin: ; padding-left: 0px; padding-right: 0px; display: inline; border-top: 0px; border-right: 0px; padding-top: 0px" title="image" border="0" alt="image" src="http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_4E2C662A.png" width="470" height="128" />][7]

Si ahora vamos a D:Remote veremos que allí tenemos el repositorio. En este caso el repositorio **no** está en un subdirectorio .git, porque hemos elegido la opción de _Central repository_ que no tiene directorio de trabajo.

Bien, ahora vamos a hacer un _push_ de nuestro repositorio _local_ al repositorio remoto. Para ello, primero, debemos dar de alta este repositorio remoto en las Git Extensions. Para ello nos vamos al menú Git y seleccionamos la opción de “Manage Remotes”. Nos aparecerá una ventana y ponemos los datos:

[<img style="background-image: none; border-bottom: 0px; border-left: 0px; margin: ; padding-left: 0px; padding-right: 0px; display: inline; border-top: 0px; border-right: 0px; padding-top: 0px" title="image" border="0" alt="image" src="http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_62A18FDB.png" width="289" height="117" />][8]

El nombre es un nombre identificativo, mientras que la URL en este caso es la carpeta donde está el repositorio. Finalmente le damos a “Save” para guardarlo.

Ahora ya podemos hacer un _push_ para pasar los datos del repositorio _local_ al remoto. Para ello, de nuevo nos vamos al menú Git y marcamos la opción Push. En la ventana que nos aparece marcamos la opción “Remote” y de la combo seleccionamos el repositorio remoto que dimos antes de alta:

[<img style="background-image: none; border-bottom: 0px; border-left: 0px; margin: ; padding-left: 0px; padding-right: 0px; display: inline; border-top: 0px; border-right: 0px; padding-top: 0px" title="image" border="0" alt="image" src="http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_2550B547.png" width="244" height="134" />][9]

Luego pulsamos el botón Push. Como antes nos mostrará un diálogo cuando haya finalizado:

[<img style="background-image: none; border-bottom: 0px; border-left: 0px; margin: ; padding-left: 0px; padding-
right: 0px; display: inline; border-top: 0px; border-right: 0px; padding-top: 0px" title="image" border="0" alt="image" src="http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_05DE5EB4.png" width="244" height="83" />][10]

**Navegar por repositorios**

Podemos navegar por un repositorio, usando la opción “Browse” del menú Git. Seleccionamos el repositorio y podemos navegar, ver los commits/push que se han realizado y ver los archivos y cambios contenidos en cada commit/push. De todos modos sólo he visto como hacer esto en repositorios (locales o remotos) que estén en disco. No sé como navegar por mi repositorio en Assembla p.ej.

[<img style="background-image: none; border-bottom: 0px; border-left: 0px; margin: ; padding-left: 0px; padding-right: 0px; display: inline; border-top: 0px; border-right: 0px; padding-top: 0px" title="image" border="0" alt="image" src="http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_05CE38E7.png" width="244" height="139" />][11]

Y aquí lo dejamos por el momento… A medida que trabaje y que vaya aprendiendo como funciona Git iré poniendo más información al respecto!

Espero que este post os haya sido útil y que en todo caso haya servido para que veáis que es Git y las diferencias con otros sistemas de control de código fuente como TFS.

Un saludo!

 [1]: http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_64C2A2A3.png
 [2]: http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_3F95326A.png
 [3]: http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_567403E6.png
 [4]: http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_39AA6904.png
 [5]: http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_551F453A.png
 [6]: http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_28666594.png
 [7]: http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_04E9C12F.png
 [8]: http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_472CB3A5.png
 [9]: http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_2B7801E2.png
 [10]: http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_63D602ED.png
 [11]: http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_008F7803.png