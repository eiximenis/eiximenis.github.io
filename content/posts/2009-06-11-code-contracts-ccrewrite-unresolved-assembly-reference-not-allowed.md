---
title: '[Code Contracts] ccrewrite – “Unresolved assembly reference not allowed”'
author: eiximenis

date: 2009-06-11T17:22:43+00:00
geeks_url: /?p=1455
geeks_visits:
  - 1070
geeks_ms_views:
  - 517
categories:
  - Uncategorized

---
Imaginaos que teneis una solución con varios proyectos, y que estos compilan en un directorio concreto, llamésmole Q:bin.

En otra solución teneis varios proyectos más, con referencias a los assemblies que estan en Q:bin (no son referencias de proyecto porque estan en distintas soluciones).

Y ya puestos, imaginad también que estáis usando Code Contracts. Y cuando compilais los proyectos de la **segunda** solución visual studio se descuelga con un bonito error:

_Unresolved assembly reference not allowed: assembly\_que\_esta\_en\_Q_bin.dll_ y el “fichero” que genera el error es ccrewrite.

La solución es ir al proyecto que no compila, abrir sus propiedades y en la pestaña Code Contracts añadir el directorio Q:bin a la entrada “Lib Paths”:

[<img style="border-bottom: 0px; border-left: 0px; display: inline; border-top: 0px; border-right: 0px" title="image" border="0" alt="image" src="http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_047965A4.png" width="244" height="160" />][1] 

Con esto el proyecto debería funcionar correctamente!

La solución gracias a [http://social.msdn.microsoft.com/Forums/en-US/codecontracts/thread/7225dc8d-7005-4da7-8a39-688e5b766434][2]

Saludos!

pd1: Que hace la entrada “Lib Paths” de la pestaña Code Contracts? Pues añade los directorios especificados al parámetro /libpaths: de ccrewrite

pd2: Que es ccrewrite? Es el MSIL rewriter de code contracts, el encargado de “mover” todas las precondiciones _al principio de todo_ de nuestro método y de “mover” las postcondiciones _justo al final_ de nuestro método.

 [1]: http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_3F3BED18.png
 [2]: http://social.msdn.microsoft.com/Forums/en-US/codecontracts/thread/7225dc8d-7005-4da7-8a39-688e5b766434 "http://social.msdn.microsoft.com/Forums/en-US/codecontracts/thread/7225dc8d-7005-4da7-8a39-688e5b766434"