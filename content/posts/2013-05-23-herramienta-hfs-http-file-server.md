---
title: 'Herramienta: HFS – Http File Server'

author: eiximenis

date: 2013-05-23T08:53:45+00:00
geeks_url: /?p=1640
geeks_visits:
  - 1926
geeks_ms_views:
  - 1183
categories:
  - Uncategorized

---
Muy buenas! Cuando preparo demos de HTML5 y JS, si no hay involucrado un servidor de por medio, no suelo utilizar VS para generar el proyecto si no algún editor más liviano, como [Sublime Text][1] o [Notepad++][2] (personalmente prefiero el primero mil veces al segundo).

El único problema reside en que algunos navegadores, por seguridad, no ejecutan Javascript cuando el origen es file:// (es decir cuando estamos cargando un fichero del sistema de ficheros). P. ej. tengo una página que usa el API de geolocalización de HTML5 para mostrar mis coordenadas y cuando la cargo desde el sistema de ficheros, Chrome deniega la petición para geolocalización automáticamente, sin preguntar:

[<img title="image" style="border-top: 0px; border-right: 0px; border-bottom: 0px; border-left: 0px; display: inline" border="0" alt="image" src="http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_5204C78D.png" width="484" height="142" />][3]&#160;

Por otro lado IE no es tan restrictivo, pero me salta con un mensaje diciendo que los scripts (o ActiveX) se han bloqueado y un botón para permitir su ejecución:

[<img title="image" style="border-top: 0px; border-right: 0px; border-bottom: 0px; border-left: 0px; display: inline" border="0" alt="image" src="http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_4FE7C8C4.png" width="484" height="194" />][4] </p> 

Bien, aunque esto personalmente me gusta (es una buena medida de seguridad) a veces, cuando preparas demos, da un poco por el saco. La solución es, obviamente, servir los ficheros via http, desde un servidor web, así que busqué la manera más sencilla de hacerlo.

Una es, teniendo instalado IIS, copiar los ficheros al directorio Inetpubwwwroot de IIS, pero hacer esto cada vez (además con un directorio protegido con derechos de administrador) es un peñazo.

Otra es crear un proyecto ASP.NET en Visual Studio, meter allí los html y ejecutarlo. Pero claro, iniciar VS tan solo para ejecutar un par de htmls y javascripts me parece excesivo. Pero vaya, eso es más o menos lo que iba haciendo, hasta que un día me dije “_tiene que haber una manera más sencilla”_.

Y nada, así di con [HFS (Http File Server)][5]: un pequeño programa que al ejecutarlo crea un servidor http y empieza a servir los ficheros que tu le digas. Una vez lo descargas y ejecutas (no se instala ni nada), aparece la ventana principal:

[<img title="image" style="border-top: 0px; border-right: 0px; border-bottom: 0px; border-left: 0px; display: inline" border="0" alt="image" src="http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_57AFBB66.png" width="484" height="257" />][6] 

Y luego tan solo arrastras los ficheros que quieres servir via http. P. ej. si arrastro el fichero c:personalgeolocalizacion.html automáticamente aparece en la lista de la izquierda, indicando que ya se puede acceder a él, via http:

[<img title="image" style="border-top: 0px; border-right: 0px; border-bottom: 0px; border-left: 0px; display: inline" border="0" alt="image" src="http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_239B1C13.png" width="484" height="257" />][7] 

Lo bueno: **El fichero NO se copia en ningún otro directorio**, ni nada parecido. No hay nada más a configurar. Ahora ya puedo abrir un navegador y ver mi fichero servido via http:

[<img title="image" style="border-top: 0px; border-right: 0px; border-bottom: 0px; border-left: 0px; display: inline" border="0" alt="image" src="http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_5A94FA4C.png" width="484" height="167" />][8]&#160;

Fíjate como ahora, la página está servida via http y Chrome si que me pregunta si quiero compartir mi ubicación con localhost:8080 (tal y como manda la especificación de HTML5).

Personalmente me parece una herramienta muy sencilla y útil y la quería compartir con todos vosotros 🙂

Un saludo a todos!

 [1]: http://www.sublimetext.com/
 [2]: http://notepad-plus-plus.org/
 [3]: http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_1E3CD13C.png
 [4]: http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_29F2186E.png
 [5]: http://www.rejetto.com/hfs/
 [6]: http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_4064B6F5.png
 [7]: http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_2B92BE75.png
 [8]: http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_1BA379B1.png