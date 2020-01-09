---
title: VS bloquea assemblies cargados desde la web
description: VS bloquea assemblies cargados desde la web
author: eiximenis

date: 2010-05-28T15:03:00+00:00
geeks_url: /?p=1513
geeks_visits:
  - 1162
geeks_ms_views:
  - 637
categories:
  - Uncategorized

---
&iexcl;Hola!

Situación explicada rápidament: Si te descargas un assembly directamente desde internet, y añades una referencia a dicho assembly, la referencia te aparecerá como añadida, pero VS no hará caso de ella. Cuando compiles te aparecerá un error parecido a:

_Unable to load the metadata for assembly &#8216;AvalonDock&#8217;. This assembly may have been downloaded from the web.&nbsp; See_ [_http://go.microsoft.com/fwlink/?LinkId=179545_][1]_.&nbsp; The following error was encountered during load: Could not load file or assembly &#8216;AvalonDock, Version=1.2.2691.0, Culture=neutral, PublicKeyToken=85a1e0ada7ec13e4&#8217; or one of its dependencies. Operation is not supported. (Exception from HRESULT: 0x80131515)_

En mi caso, efectivamente me descargué la última versión de <a target="_blank" href="http://avalondock.codeplex.com/" rel="noopener noreferrer">AvalonDock</a> (<a target="_blank" href="http://avalondock.codeplex.com/releases/view/35297" rel="noopener noreferrer">desde su página de descargas</a> puedes descargarte un msi que lo instala o simplemente el assembly que contiene la librería y que es lo que yo hice).

Por suerte el propio mensaje de error de VS es más que expícito y si te vas a la página que menciona ([_http://go.microsoft.com/fwlink/?LinkId=179545_][1]) verás la explicación de lo que hay que hacer: Básicamente cerrar VS y con el explorador de archivos abrir las propiedades del archivo y darle a desbloquear:

[<img height="244" width="180" src="/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_030013C4.png" alt="image" border="0" title="image" style="border-bottom: 0px; border-left: 0px; display: inline; border-top: 0px; border-right: 0px" />][2] 

Y listos! Ya puedes usar tu assembly... ten presente que esto da permisos full trust al assembly, así que... andate con cuidado! 😉

**¿Y como sabe VS que el archivo ha sido descargado?**

Esta pregunta se aplica no sólo a VS sinó a Windows en general. Como sabe Windows que dicho archivo ha sido descargado de la web? Si lo mueves de carpeta, lo renombras o incluso lo copias a otro ordenador windows seguirá sabiendo que el archivo ha sido descargado de la web.

La respuesta es realmente simple: Windows guarda esta información en el **propio archivo** y lo hace gracias a una capacidad no muy conocida de NTFS llamada _alternate data streams (ADS)_. Dicha capacidad (insisto de NTFS) permite asociar _metadatos_ a un archivo. Cuando descargamos un archivo de la red, windows crea un ADS en dicho archivo y lo marca como descargado de la red. Los ADS se _crean_ junto con el archivo (no se guardan en ningún sitio separado).

Para poder abrir un ADS basta con saber el nombre de este y añadirlo junto con dos puntos (:) al nombre del archivo. P.ej. el ADS que windows asocia a los archivos descargados de internet se llama &ldquo;Zone.Identifier&rdquo;. Así, en mi caso si yo hago:

<span style="font-family: Courier New; font-size: x-small;"><strong>notepad AvalonDock.dll:Zone.Identifier</strong></span>

obtengo lo siguiente:

[<img height="65" width="244" src="/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_7691F09A.png" alt="image" border="0" title="image" style="border-bottom: 0px; border-left: 0px; display: inline; border-top: 0px; border-right: 0px" />][3] 

Cuando pulso el botón de &ldquo;Unblock&rdquo; desde el explorador de archivos lo que hace es borrar dicho ADS del archivo.

Usando explorer no podeis saber fácilmente si un archivo contiene ADS o no: aunque los ADS _modifican_ el tamaño _real_ del archivo (puesto que se guardan junto a este), el explorador de archivos sólo os muestra el tamaño del flujo principal (o sea el del archivo sin los ADS) y lo mismo hace el comando &ldquo;dir&rdquo; de MS-DOS. Si quereis saber los ADS que tienen vuestros archivos, hay por _ahí afuera varios visores de ADS_ (aunque yo no he probado ninguno).

Por cierto, que os he dicho que los ADS, dado que se guardan junto con el archivo, se mantienen cuando se copia, se modifica o se mueve el archivo y esto es cierto **siempre que se use NTFS**. Si se copia un archivo que contiene ADS a un sistema que no los soporta (p.ej. FAT32 usado en las llaves USB), el ADS se pierde.

Un saludote!! 😉

 [1]: http://go.microsoft.com/fwlink/?LinkId=179545
 [2]: /cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_0D80E7E4.png
 [3]: /cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_2242BA97.png