---
title: 'Novedades de IE11: SPDY'
description: 'Novedades de IE11: SPDY'
author: eiximenis

date: 2013-06-27T14:16:50+00:00
geeks_url: /?p=1646
geeks_visits:
  - 1781
geeks_ms_views:
  - 890
categories:
  - Uncategorized

---
Como ya es habitual una nueva versión de Windows (en este caso la 8.1) viene acompañado de un nuevo IE11. Y como siempre IE11 viene con varias novedades siendo quizá las dos más destacadas el soporte para WebGL (gráficos 3D) y el tema de este post, el soporte del protocolo SPDY.

Este será un post ligeramente distinto a los habituales del blog porque no hablaré de nada de desarrollo 🙂

**Un protocolo de Nivel 7**

[<img title="Niveles OSI (sacado de Wikipedia)" style="margin-left: 0px; display: inline; margin-right: 0px" alt="Niveles OSI (sacado de Wikipedia)" align="right" src="http://upload.wikimedia.org/wikipedia/commons/thumb/7/7d/Pila-osi-es.svg/300px-Pila-osi-es.svg.png" width="170" height="240" />][1]

El nivel OSI de un protocolo de comunicaciones determina “cuan alejado del medio físico” está este protocolo. Apareció a mediados de los 80 y determina 7 capas por las cuales los datos deben pasar para llegar desde el medio físico (p. ej. un cable) hacia el usuario final.

Para una descripción detallada de cada nivel OSI [podéis consultar este artículo de la microsoft][2]. Por curiosidad os puedo comentar que el protocolo IP (tanto v4 como v6) está situado en el nivel 3, TCP está situado en el nivel 4, SSL (o su variante más moderna TLS) está situado en el nivel 5 y que HTTP está situado en el nivel 7. Por cierto, aprovecho para comentar que TCP e IP son dos protocolos, así que cuando decimos que internet “usa el protocolo TCP/IP” realmente queremos decir que usa “el protocolo TCP funcionando bajo el protocolo IP”. Pero bajo IP pueden funcionar otros protocolos (como UDP p. ej.) o al revés TCP puede funcionar bajo otros protocolos de nivel 3 como IPX.

Pues bien, SPDY (pronunciado _speedy_) es un nuevo protocolo, situado en la capa 7 de OSI **que viene a reemplazar HTTP**.

La frase en negrita del párrafo anterior es la frase sensacionalista, porque como ahora verás el hecho de que SPDY sustituya a HTTP **no te afecta para nada como desarrollador de aplicaciones web**. Dejemos esto bien claro a partir de ya, ¿ok? SPDY tiene impacto cero para los desarrolladores de aplicaciones web pues **toda la semántica de HTTP se mantiene**. De hecho familiarmente hay quien se refiere a SPDY como HTTP 2.0 pero eso no es ni mucho menos una denominación oficial y además se presta a confusión puesto que hay una especificación (en borrador) de HTTP 2.0 (que además se basa en SPDY pero eso es otra historia).

**32x, 3.1Ghz, 12 Mpx…**

Los seres humanos somos simplistas por naturaleza, generalmente nos agobia trabajar con demasiados datos y deseamos reducirlo todo a un solo indicador. Por supuesto los que se encargan de venderte cosas ya escogerán el indicador que les vaya mejor _a ellos para venderte el producto_ con independencia de que dicho indicador sea el que mejor refleje la calidad global del producto. En los tiempos de las unidades de CD los fabricantes se esmeraban en tener un valor de “equis” lo más alto posible. Y así se asoció que una unidad de 50x era mucho mejor que una de 32x aunque esto era la velocidad punta y no la sostenida que sería mucho más importante (pero menos impactante) para dar cuenta del rendimiento de la unidad. Pero es que incluso la velocidad sostenida por si sola tampoco da toda la información. Algo parecido ocurre con los ordenadores que se suelen vender publicitando más los Ghz del procesador aunque este dato por si solo es totalmente irrelevante. O con las cámaras de fotos y los megapíxeles (solo hay que ver el revuelo que se está armando con este futuro nokia de 41 de megapíxeles). Los megapíxeles importan sí, pero es mucho más importante la calidad de las lentes, el estabilizador y, a partir de cierto número de Mpxs, el tamaño físico del CCD. Pero analizar todo esto son demasiados datos y como digo somos simplistas… 

Esto mismo aplica a tu conexión de internet. Y es que… es mejor una conexión de 20 Megas que una de 15 no? Pues no, necesariamente. No voy a escribir el porque una conexión de 20 Megas puede ser peor que una de 15 (asumindo que ambas funcionen al 100% de su velocidad) porque [ya lo hizo el fenómeno de José Manuel Alarcón en su blog][3]. Léete el post porque es un must read.

Volvamos a SPDY. Seguramente la pregunta que nos podríamos hacer sería… ¿**por qué un nuevo protocolo para reemplazar HTTP?** HTTP funciona y es la base de Internet que es probablemente una de las revoluciones más importantes de la (corta) historia de la humanidad. ¿Entonces?

Hoy en día las páginas web se han echo muy complejas, para tener números actualizados acabo de probar varias páginas para contar cuantas peticiones http se realizan para cargarlas todas (el javascript, las imágenes, css, videos, etc). No quiero fijarme en el tamaño de los elementos transferidos, solo en cuantas peticiones http debe enviar el navegador. Pues bien, esos son algunos números:

  * facebook.com (con usuario logado): 153 peticiones http 
  * marca.com: 455 peticiones http 
  * twitter.com (con usuaruo logado): 44 peticiones http 
  * google.com: 16 peticiones que se disparan a las 29 al empezar a teclear una búsqueda. 

(Todas estas mediciones son usando las webs de producción y sin cache).

Como desarrollador web probablemente ya sabes lo importante que es minimizar el número de peticiones HTTP (usando CSS sprites, imágenes en data-uris, compactación de js, etc). Los navegadores también ayudan y suelen usar hasta 6 (antes eran 2) conexiones simultáneas.

Pero tenemos dos problemas ahí… veamos ambos.

**Problema 1: Crear una conexión TCP es “lento”.**

La “lentitud” de crear una conexión TCP viene dada porque usa un sistema conocido como 3-way-handshake para establecer dicha conexión:

<img style="margin: 0px 10px 0px 0px; display: inline" align="left" src="http://upload.wikimedia.org/wikipedia/commons/thumb/9/98/Tcp-handshake.svg/500px-Tcp-handshake.svg.png" width="263" height="235" />

El cliente primero debe enviar un paquete SYN, el servidor debe responder con un SYN ACK y finalmente el cliente debe responder con un ACK.

En este punto la conexión TCP está establecida y se pueden empezar a mandar datos.

&#160;

Es decir para poder empezar a mandar datos hay una comunicación del cliente al servidor, una del servidor al cliente y otra del cliente al servidor. ¿Y cuando digo lento cuan lento quiero decir? Pues con un buen ADSL (y no me refiero a las megas) pues establecer una conexión TCP te puede costar de 20 a 30ms. Pero ojo… en un móvil estos tiempos se pueden disparar hasta los 300ms por cada conexión TCP.

En resumen: **El ancho de banda NO es importante. Lo verdaderamente importante es la latencia** (¿te ha dicho tu proveedor de adsl la latencia de tu conexión? ¿No, verdad?).

En HTTP1.0 cada petición HTTP abre y cierra una conexión TCP.

**Solución: HTTP1.1 al rescate**

En HTTP1.1 es posible usar conexiones _keep-alive_ lo que básicamente significa que bajo una misma conexión TCP se pueden enviar varias peticiones HTTP solucionando así el problema anterior.

Actualmente (casi) todos los servidores y todos los navegadores soportan HTTP1.1 así que… problema arreglado 🙂

**Problema #2: Todo el proceso es síncrono**

Vale… gracias a las conexiones _keep-alive_ de HTTP1.1 hemos “eliminado” el problema de la “lentitud” de abrir conexiones TCP. Pero nos queda el problema fundamental: Todo el proceso es síncrono:

 <img title="HTTP sin pipeline" style="border-t
op: 0px; border-right: 0px; border-bottom: 0px; margin: 0px 5px 0px 0px; border-left: 0px; display: inline" border="0" alt="HTTP sin pipeline" align="left" src="http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_147D1103.png" width="162" height="244" />

A pesar de que podemos utilizar una sola conexión TCP para enviar los datos, el navegador no puede enviar la segunda petición HTTP hasta haber recibido la primera y no puede enviar la tercera hasta haber recibio la segunda.

Si la segunda petición tarda mucho (por la razón que sea y en la dirección que sea) el navegador estará esperando y esperando hasta recibir la respuesta sin poder enviar la tercera petición.

&#160;

**Intento de solución: HTTP1.1 pipelining**

HTTP1.1 además de conexiones keep-alive ofrecía un modo _pipeline_ que básicamente viene a romper la sincronidad. Permite que el navegador envíe **todas** las peticiones de golpe y espere por las respuestas:

 <img title="Pipelining en HTTP1.1" style="border-top: 0px; border-right: 0px; border-bottom: 0px; margin: 0px 5px 0px 0px; border-left: 0px; display: inline" border="0" alt="Pipelining en HTTP1.1" align="left" src="http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_0C856EA1.png" width="161" height="244" />

Con esto el problema #2 parece solucionado ¿no? El navegador puede enviar todas las peticiones y luego esperar a que lleguen todas las respuestas de golpe.

Pues no.

Primero hay un motivo práctico: Para que el pipeline de HTTP1.1 funcione es necesario que esté soportado por todos los dispositivos intermedios que hay entre el servidor y el funcione (o sea, los proxies) y hay algunos no lo soportan. Por esta razón HTTP1.1 pipelining apenas se usa (de hecho está soportado pero desactivado en casi todos los navegadores de la actualidad).

Pero **incluso si HTTP1.1. pipelining se estuviese usando de forma masiva** no es la solución ideal al problema. ¿Por qué? Pues porque HTTP obliga a una semántica FIFO en las peticiones. **Es decir el navegador DEBE recibir la respuesta de la primera petición antes de recibir la respuesta de la segunda. Y DEBE recibir la respuesta de la segunda petición antes de la respuesta de la tercera. Y así sucesivamente.**

Por lo tanto si hay una petición lenta todas las posteriores se verán retrasadas también, porque la respuesta a esta petición lenta debe ser enviada al navegador _antes_ que las respuestas de las peticiones siguientes.

****

****

****

****

****

****

**Solución: SPDY al rescate**

Aquí es donde entra SPDY. Este protocolo, desarrollado inicialmente por Google, ofrece un pipelining real sobre una sola conexión TCP (además de otras mejoras) para de esta manera reducir los tiempos de latencia y espera.

Además SPDY añade más funcionalidades a HTTP (como _server push_) y toda la petición es comprimida (en HTTP se puede comprimir la respuesta pero no las cabeceras).

Y lo más importante de todo: **SPDY no requiere ningún cambio** en la infraestructura de red actual ni en las aplicaciones web desarrolladas. Insisto: a ti, como desarrollador web, que se use SPDY te es totalmente transparente. Recuerda: la semántica de HTTP (verbos, cabeceras, URLs) está totalmente mantenida. SPDY tan solo modifica en como se usa TCP por debajo (una sola conexión y pipelining real)

Para que se use SPDY tan solo es necesario que el servidor web y el navegador lo soporten. IE11 finalmente soporta SPDY y se une así a Firefox, Opera y obviamente Chrome. En Android la ultima versión de Chrome y Opera Mobile soportan SPDY. De Safari, la verdad no tengo ni idea. Y en el mundo de los servidores hay módulos SPDY para los principales servidores web. De IIS no tengo noticias, pero imagino que a partir de Windows 8.1 estará soportado (a nivel de http.sys supongo).

En fin, el soporte para SPDY de IE11 es una muy buena noticia que ayudará a que este protocolo se vaya extendiendo más y que todos tengamos una web (un poco) más rápida!

Como dije… un post diferente 😉

Saludos!

 [1]: http://es.wikipedia.org/wiki/Modelo_OSI
 [2]: http://support.microsoft.com/kb/103884/es
 [3]: http://www.jasoft.org/Blog/post/Un-error-comun-de-concepto-La-velocidad-de-conexion-a-Internet.aspx