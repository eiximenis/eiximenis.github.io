---
title: Usando las herramientas para Git de VS2012
description: Usando las herramientas para Git de VS2012
author: eiximenis

date: 2013-07-19T09:21:00+00:00
geeks_url: /?p=1648
geeks_visits:
  - 2847
geeks_ms_views:
  - 1885
categories:
  - Uncategorized

---
Muy buenas! En este post voy a contar (o al menos intentarlo) como usar las herramientas de Git para VS2012 y trabajar con un repositorio Git instalado en TFS Services.

> **Nota:** Este post está muy orientado a gente que viene de TFS, está acostumbrada a TFS y se siente un poco &ldquo;perdida&rdquo; con esto de Git. No pretende ser, ni mucho menos, un tutorial de Git.

**Requerimientos previos**

Debes tener instalado VS2012 y al menos el Update 2. Ve a la <a target="_blank" href="http://www.microsoft.com/visualstudio/esn/visual-studio-update" rel="noopener noreferrer">página de Updates de VS2012</a> para descargarte el último update (en la actualidad es el 3).

Una vez tengas el VS2012 actualizado, debes instalarte las herramientas para Git de VS2012, que te puedes descargar desde el gestor de extensiones de VS2012 o bien desde la propia <a target="_blank" href="http://visualstudiogallery.msdn.microsoft.com/abafc7d6-dcaa-40f4-8a5e-d6724bdb980c" rel="noopener noreferrer">página de las Visual Studio Tools for Git</a>.

Finalmente debes tener una cuenta de TFS Services, que te puedes abrir en <http://tfs.visualstudio.com>. Abrir una cuenta es totalmente gratuito y te da acceso a un TFS listo para 5 usuarios de forma totalmente gratuita.

**Antes que nada: diferencias entre Git y TFS clásico**

El control de código fuente clásico de TFS es lo que se conoce como un CVS (Concurrent Version System), mientras que Git es un DCVS (Distributed Concurrent Version System). En TFS está claro quien es el servidor: te connectas a un servidor TFS, te bajas código de él y subes código en él.

Con Git, cada máquina es también un repositorio de código fuente. Vas a tener _tu propio_ repositorio de código fuente en local. No tendrás solo &ldquo;la última versión&rdquo; si no todo el control de código fuente. Esto hace que operaciones como Branch o Merge sean mega rápidas (y puedas hacerlas offline).

De hecho, no hay en Git, un flujo que obligue a que haya un &ldquo;servidor de Git centralizado&rdquo;: puedes tener varios repositorios remotos con los cuales te sincronizas.

De todos modos, dado que es bastante habitual un flujo donde haya un servidor Git &ldquo;central&rdquo;, este es el que veremos en este post. En nuestro caso este &ldquo;servidor git central&rdquo; será TFS Services con el control de código fuente Git habilitado.

En Git no hay check-in o check-out, ni tampoco lock. En su lugar las operaciones básicas que tenemos son:

  1. Commit: Pasa cambios de tu working folder a tu repositorio git local pero NO al remoto.
  2. Push: Pasa cambios de tu repositorio git local al remoto
  3. Pull: Pasa cambios del repositorio git remoto al local y a la working folder.

Ahora sí.... empecemos.

**Primer paso: Crear el team project**

Para tener un repositorio Git en TFS debemos tener un team project. Así que dale al botón &ldquo;New Team Project&rdquo; desde la página principal de TFS Services y en las propiedades del proyecto asegúrate de seleccionar Git como gestor de código fuente:

[<img height="413" width="484" src="/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_33F8AB70.png" alt="image" border="0" style="border-left-width: 0px; border-right-width: 0px; border-bottom-width: 0px; display: inline; border-top-width: 0px" title="image" />][1] 

Una vez tengas el team project creado estás listo para empezar a trabajar con Git.

**Añadir la solución al control de código duente.**

Para ello desde el solution explorer seleccionamos la opción &ldquo;Add solution to Source control&rdquo;. VS2012 nos preguntará si deseamos usar Git o bien el control de código fuente de TFS. Hasta ahora simplemente habíamos creado un repositorio de Git, pero no le habíamos indicado a VS2012 que íbamos a usarlo en nuestra solución:

[<img height="210" width="484" src="/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_5D101DE1.png" alt="image" border="0" style="border-left-width: 0px; border-right-width: 0px; border-bottom-width: 0px; display: inline; border-top-width: 0px" title="image" />][2] 

Asegúrate de marcar la opción de Git y dale a Ok. Ahora si que estamos listos para hacer commits.

Al marcar la opción de Git, VS2012 nos crea un repositorio de Git local que está en el mismo directorio que nuestra solución. De hecho si abres con el explorador de archivos la carpeta de la solución, verás una carpeta oculta, llamada .git que es la que contiene el repositorio local. 

**Commit**

Vamos a guardar nuestro código o en el repositorio de Git local. A diferencia del TFS clásico en Git no se protegen o desprotegen ficheros, así que no hay opciones de check-in o check-out. Básicamente Git mira todos los cambios que se producen entre la working folder y el repositorio Git local y son los ficheros que tengan cambios (de cualquier tipo) los que podemos &ldquo;commitear&rdquo; y enviar al repositorio Git local. Para hacerlo vete a la página inicial del team explorer pulsando el icono de home:

[<img height="154" width="484" src="/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_1E0E7779.png" alt="image" border="0" style="border-left-width: 0px; border-right-width: 0px; border-bottom-width: 0px; display: inline; border-top-width: 0px" title="image" />][3] 

&nbsp;

Ahora pulsa sobre de &ldquo;Changes&rdquo;. Con eso VS2012 te mostrará todos los archivos de la working folder que tengan algún cambio respecto al repositorio de Git local (es decir que se hayan añadido, borrado o modificado).

[<img height="484" width="469" src="/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_556462DA.png" alt="image" border="0" style="border-left-width: 0px; border-right-width: 0px; border-bottom-width: 0px; display: inline; border-top-width: 0px" title="image" />][4] 

&nbsp;

Los ficheros que aparecen en Included Changes son los ficheros que se incluirán en el commit. Los que aparecen en Excluded Changes son los que no se incluirán en el commit. Puedes pasar ficheros de un sitio a otro arrastrándolos. Una vez tengas el commit listo, debes añadir un mensaje de commit y ya podrás darle al botón &ldquo;Commit&rdquo;.

Una vez lo hayas hecho VS2012 te mostrará un mensaje diciendo que el commit ha sido correcto **y continuarás en la misma página de commits**, para seguir haciendo commits si quedan ficheros con cambios (es decir, ficheros que antes habías colocado en &ldquo;Excluded Changes&rdquo;). Eso es así porque con Git es costumbre hacer muchos commits y muy pequeños.

**Push**

Si ahora vuelves a la página inicial del Team Explorer (con el icono de la casita) y pulsas sobre el enlace &ldquo;Commits&rdquo; te aparecerá una ventana como la que sigue:

[<img height="261" width="484" src="/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_388AA22B.png" alt="image" border="0" style="border-left-width: 0px; border-right-width: 0px; border-bottom-width: 0px; display: inline; border-top-width: 0px" title="image" />][5] 

VS2012 nos pide que repositorio Git va a ser el remoto. Este debe estar vacío. Recuerda que antes hemos creado uno en TFS Services, pero todavía no habíamos dicho en ningún sitio que queríamos usarlo. Ahora ha llegado el momento. Copia la URL del repositorio Git remoto (la puedes encontrar en la página de TFS Services si vas al team project creado y al apartado &ldquo;Code&rdquo;) y dale al botón &ldquo;Publish&rdquo;. Con este proceso asociarás tu repositorio de Git local con el repositorio de Git remoto y además harás un push de los commits pendientes (es decir pasarás el código de tu repositorio Git local al remoto).

Cuando haya terminado te aparecerá una página parecida a:

[<img height="320" width="484" src="/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_3016A707.png" alt="image" border="0" style="border-left-width: 0px; border-right-width: 0px; border-bottom-width: 0px; display: inline; border-top-width: 0px" title="image" />][6] 

**&iexcl;Felicidades! Ya tienes un repositorio de Git local y remoto enlazado a tu solución**.

Veamos ahora como añadir un cambio.

Para ello modifica un par de archivos del proyecto. Puedes hacerlo directamente: recuerda, olvida el concepto de check-out (desproteger). Vete a la página de Changes del team explorer (ve a la página principal a través del icono de casa y luego pulsa en Changes) y te aparecerán los dos archivos modificados:

[<img height="336" width="484" src="/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_44C853ED.png" alt="image" border="0" style="border-left-width: 0px; border-right-width: 0px; border-bottom-width: 0px; display: inline; border-top-width: 0px" title="image" />][7]&nbsp;

Arrastra uno de los dos a Excluded Changes, añade un mensaje de commit y dale a commit. Te aparecerá una pantalla como la siguiente:

[<img height="349" width="484" src="/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_734E3702.png" alt="image" border="0" style="border-left-width: 0px; border-right-width: 0px; border-bottom-width: 0px; display: inline; border-top-width: 0px" title="image" />][8] 

Ahora arrastra el otro archivo a Included Changes, pon otro comentario y dale a commit de nuevo. Con esto has generado dos commits en el repositorio git local.

&nbsp;

Si ahora te vas a la página de Push (vete a la inicial con el icono de la casa) y dale Commits (debajo de Changes) verás algo parecido a:

[<img height="356" width="484" src="/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_49E6C937.png" alt="image" border="0" style="border-left-width: 0px; border-right-width: 0px; border-bottom-width: 0px; display: inline; border-top-width: 0px" title="image" />][9] 

En outgoing commits te aparecerán los commits que has hecho en el repositorio local y que no están en el repositorio remoto. Si le das a &ldquo;Push&rdquo; subirás los cambios (los commits) desde el repositorio local al remoto.

**Obtener los cambios de otro usuario**

Vale, a no ser que estés tu solo en un proyecto (y sí, si estás tu solo en un proyecto te recomiendo encarecidamente también usar control de código fuente) en algún momento deberás incorporar los cambios que hay en el repositorio remoto al tu repositorio local.

Vete a la página de commits del team explorer (ya sabes, desde la home le das al enlace de Commits):

[<img height="279" width="484" src="/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_5089ACED.png" alt="image" border="0" style="border-left-width: 0px; border-right-width: 0px; border-bottom-width: 0px; display: inline; border-top-width: 0px" title="image" />][10] 

Debajo de &ldquo;Incoming Commits&rdquo; hay dos opciones:

  1. Fetch: Te permite obtener el listado de cambios que tienes pendientes de integrar. Es decir, el listado de commits que están en el repositorio remoto pero NO en tu repositorio local.
  2. Pull: Se trae los commits desde el repositorio remoto al repositorio local y actualiza la working folder.

Si le das a Pull el proceso es automático (Git se trae los commits del repositorio remoto al local y desde el local hacia la working folder resolviendo los conflictos si hiciese falta).

Si le das a Fetch, te aparecerán los commits que existen en el repositorio remoto, pero están pendientes de ser integrados a tu repositorio local:

[<img height="334" width="484" src="/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_33FC9540.png" alt="image" border="0" style="border-top: 0px; border-right: 0px; border-bottom: 0px; border-left: 0px; display: inline" title="image" />][11] 

**Viendo los detalles de un commit**

A veces, antes de integrar el código del repositorio remoto (es decir, antes de hacer pull) te interesa ver como este commit puede afectar a tu trabajo. Para ello, una vez has obtenidos los commits pendientes de integrar (es decir, una vez has hecho fetch) puedes ver los detalles de un commit pulsando con el botón derecho sobre él y seleccionando la opción &ldquo;View Commit Details&rdquo;:

[<img height="395" width="484" src="/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_0B6D8D5F.png" alt="image" border="0" style="border-top: 0px; border-right: 0px; border-bottom: 0px; border-left: 0px; display: inline" title="image" />][12] 

Esto te permite ver los ficheros que conforman el commit, ver su contenido y compararlo con la versión anterior de este mismo fichero.

**Resolviendo conflictos.**

Imagina la situación en que tienes modificado un fichero en tu carpeta local pero NO has hecho commit de este fichero en tu repositorio git local. Imagina que haces pull y hay un commit que tiene este fichero modificado.

Cuando esto ocurrer recibirás el siguiente mensaje de error:

[<img height="382" width="484" src="/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_16A67BCF.png" alt="image" border="0" style="border-top: 0px; border-right: 0px; border-bottom: 0px; border-left: 0px; display: inline" title="image" />][13] 

Si te aparece este mensaje debes primero hacer commit de tus cambios pendientes en tu repositorio Git local. Una vez hayas hecho el commit (no es necesario que hagas el push hacia el repositorio remoto) ya puedes volver a hacer el pull.

Entonces, Git intentará hacer merge automático entre el contenido de tu repositorio local y los commits que vienen del repositorio remoto). Pero puede que el merge automático no sea posible:

[<img height="389" width="484" src="/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_7E433BE6.png" alt="image" border="0" style="border-top: 0px; border-right: 0px; border-bottom: 0px; border-left: 0px; display: inline" title="image" />][14] 

> **Nota:** En esta pantalla &ldquo;Incoming Commits&rdquo; son los dos commits que estoy intentando integrar y en &ldquo;Outgoing Commits&rdquo; hay el commit que he hecho para colocar los cambios de la carpeta local hacia mi repositorio remoto. Como solo he hecho commit pero no push, por eso me aparece aquí.

Ahora pulsas sobre el enlace &ldquo;Resolve the conflicts&rdquo; y te aparece la página de resolución de conflictos:

[<img height="236" width="484" src="/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_2ADBCFF3.png" alt="image" border="0" style="border-top: 0px; border-right: 0px; border-bottom: 0px; border-left: 0px; display: inline" title="image" />][15] 

Pulsando sobre cada uno de los archivos en &ldquo;Conflicts&rdquo; tendremos las opciones clásicas de tomar el archivo local (el de nuestro repositorio Git local), el remoto (el del repositorio Git remoto) o combinarlos:

[<img height="343" width="484" src="/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_0E4EB846.png" alt="image" border="0" style="border-top: 0px; border-right: 0px; border-bottom: 0px; border-left: 0px; display: inline" title="image" />][16] 

Si le damos a &ldquo;Merge&rdquo; nos aparecerá la clásica ventana de Merge:

[<img height="236" width="484" src="/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_01643C5B.png" alt="image" border="0" style="border-top: 0px; border-right: 0px; border-bottom: 0px; border-left: 0px; display: inline" title="image" />][17] 

&nbsp;

&nbsp;

&nbsp;

Una vez hayamos resuelto todos los conflictos, aceptamos el merge y repetimos la operación por todos los archivos. Al final llegaremos a una página parecida a:

[<img height="236" width="484" src="/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_6B1DFB3B.png" alt="image" border="0" style="border-top: 0px; border-right: 0px; border-bottom: 0px; border-left: 0px; display: inline" title="image" />][18] 

Donde no habrá nada en &ldquo;Conflicts&rdquo; y todo estará en &ldquo;Resolved&rdquo;. Ahora tenemos estos cambios en nuestra working folder, pero NO en nuestro repositorio local de Git (y mucho menos en el remoto obviamente). Así que le damos a &ldquo;Commit Merge&rdquo; para pasar estos cambios de nuestra working folder a nuestro repositorio Gi local. Al hacerlo iréis a la página de Changes con el commit listo para ser enviado al repositorio local:

[<img height="387" width="484" src="/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_428EF35A.png" alt="image" border="0" style="border-top: 0px; border-right: 0px; border-bottom: 0px; border-left: 0px; display: inline" title="image" />][19] 

Fíjate que el commit que se me ha creado incluye tanto el archivo Program.cs (que es el que tenía conflictos) como el archivo App.config que pertenecía al otro &ldquo;incoming commit&rdquo;.

Una vez le deis a commit ya lo tendréis en el repositorio git local.

Finalmente os vais a la página de &ldquo;Commits&rdquo; y vereis los commits pendientes de integrar hacia el repositorio remoto (es decir, pendientes de hacer push):

[<img height="338" width="484" src="/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_7114D66F.png" alt="image" border="0" style="border-top: 0px; border-right: 0px; border-bottom: 0px; border-left: 0px; display: inline" title="image" />][20] 

Ahora puedes hacer push sin ningún problema y tu código integrado ya está en el reposiorio Git remoto.

**Conclusiones finales**

Cuando se trabaja en Git, **no se trabaja a nivel de fichero,** como con el control de código fuente clásico de TFS, si no que trabajamos a nivel de commit.

Debido a esto, haz commits **pequeños.**

Recuerda siempre el concepto de que hay un repositorio Git local y otro de remoto. A diferencia del control de código fuente clásico de TFS donde solo hay el remoto.

Si intentas hacer pull y tienes algún fichero modificado en tu working folder (del cual no has hecho commit hacia tu repositorio local) que está incluído en los commits que vas a integrar desde el servidor remoto, te dará error y deberás hacer primero un commit de este fichero (es decir pasarlo a tu repositorio local).

Espero que con esto os quede un poco más claro como funciona Git usando esta extensión de VS2012.

**Finalmente**: esta extensión no es, ni de lejos, el mejor cliente Git para Windows. Para flujos sencillos funciona bien, ya que está integrada en Visual Studio, pero para flujos de trabajo más complejos se queda corta. En estos casos es mejor usar un cliente de Git específico como <a target="_blank" href="http://www.sourcetreeapp.com/" rel="noopener noreferrer">SoureTree</a>.

Saludos!

**[Editado 20/07/2013]** &#8211; Corregido un error (mirar comentario de Enrique Ortuño).

&nbsp;

 [1]: /cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_52731226.png
 [2]: /cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_6F1C616E.png
 [3]: /cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_39274687.png
 [4]: /cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_49DECB68.png
 [5]: /cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_7C8E8642.png
 [6]: /cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_3FE661D8.png
 [7]: /cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_355567EB.png
 [8]: /cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_069F136F.png
 [9]: /cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_75C742F3.png
 [10]: /cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_45ACCBA5.png
 [11]: /cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_2659FF05.png
 [12]: /cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_6BEE6356.png
 [13]: /cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_22D81BC3.png
 [14]: /cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_515DFED8.png
 [15]: /cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_74BA57A3.png
 [16]: /cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_3A2F3202.png
 [17]: /cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_0A14BAB4.png
 [18]: /cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_74C66971.png
 [19]: /cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_6F47D300.png
 [20]: /cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_31E6D29F.png