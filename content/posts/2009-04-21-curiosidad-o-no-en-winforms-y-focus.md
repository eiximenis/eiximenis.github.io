---
title: Curiosidad (o no) en WinForms y focus…
author: eiximenis

date: 2009-04-21T15:01:06+00:00
geeks_url: /?p=1448
geeks_visits:
  - 2673
geeks_ms_views:
  - 879
categories:
  - Uncategorized

---
Os cuento una curiosidad de la que me acabo de dar cuenta ahora mismo… Un funcionamiento, como mínimo _curioso_ en Winforms… siempre entendiendo como curioso que “yo no lo sabía y mi no encaja en mi (poco) sentido común”.

El titular sensacionalista sería: **Control desactivado recibe el focus**.

La realidad es que si en el evento “Leave” de un control, desactivamos el _siguiente_ control que debe recibir el foco, este control NO recibe el foco (como es de esperar) pero su evento “Enter” se ejecuta…

… y no sólo eso! La siguiente vez que pulsemos TAB (o que con el mouse pongamos el foco en cualquier otro control _incluyendo_ el propio control que tiene el foco), el evento “Leave” del control deshabilitado se ejecutará!

A partir de aquí, el focus ya no “entrará” ninguna otra vez en el control deshabilitado (no lanzará más eventos Enter/Leave).

Si “quereis” jugar con esta curiosidad, aquí os dejo un pequeño programilla, con el que comprobar in-situ esta curiosidad…

[Link al programa para testear la curiosidad][1]. Compilado con VS2008 y .NET 3.5 SP1, bajo Windows XP.

Y una capturilla para que lo veais (al tener la checkbox activa, se deshabilita el textbox que está a la derecha del TextBox con fondo naranja-suave (que he llamado en un alarde de pereza textBox3)):

[<img style="border-bottom: 0px; border-left: 0px; display: inline; border-top: 0px; border-right: 0px" title="image" border="0" alt="image" src="http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_24FF8D98.png" width="244" height="119" />][2] 

Evidentemente, si alguien tiene más información al respecto y sabe el porqué de esta causa… pues le estaré muy agradecido!!! 😉

Saludos!

 [1]: http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas.21042009/Jopetas.zip
 [2]: http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_4EE001CD.png