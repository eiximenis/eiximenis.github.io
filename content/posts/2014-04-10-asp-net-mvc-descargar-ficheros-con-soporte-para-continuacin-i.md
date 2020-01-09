---
title: ASP.NET MVC–Descargar ficheros con soporte para continuación – i
author: eiximenis

date: 2014-04-10T09:39:41+00:00
geeks_url: /?p=1664
geeks_visits:
  - 964
geeks_ms_views:
  - 1222
categories:
  - Uncategorized

---
Muy buenas! El objetivo de esta serie posts es ver como podemos implementar en ASP.NET MVC descargas de ficheros con soporte para “pausa y continuación”.

> _En este primer post veremos (por encima) que cabeceras HTTP están involucradas en las peticiones y las respuestas para permitir continuar una descarga._

Dicho soporte debe estar implementado en el cliente, pero también en el servidor. En el cliente porque ese tiene que efectuar lo que se llama una _range request_, es decir pasar en la petición HTTP que “parte” del archivo quiere. Y el servidor debe tener soporte para mandar solo esta parte.

En ASP.NET MVC usamos un FileActionResult para soportar descargas de archivos. Es muy cómodo y sencillo pero **no tiene soporte para range requests**. En esta serie posts veremos como podemos crearnos un ActionResult propio para que soporte este tipo de peticiones!

En el <a href="http://www.w3.org/Protocols/rfc2616/rfc2616-sec14.html#sec14.35" target="_blank" rel="noopener noreferrer">apartado range de la definición de HTTP1.1</a>. se encuentra la definición de la cabecera _Range_ que es la madre del cordero. 

De hecho, básicamente hay **dos cabeceras** involucradas en una _range request_ (que envía el cliente y que el servidor debe entender):

  1. Range: Especifica el rango que desea obtener el cliente 
  2. If-Range: Especifica que se haga caso de “Range” solo si los datos no se han modificado desde xx (xx se especifica en la cabecera If-Range). Si los datos se han modificado, olvida Range y envía todos los datos. 

**Formato de Range**

El formato de Range es muy sencillo. Oh sí, leyendo la especificación parece que es super complejo, pero para eso son las especificaciones… 😛

A la práctica Range tiene el siguiente formato:

  * Range: bytes=x-y: El cliente quiere los bytes desde x hasta y (ambos inclusivos). P. ej. Range 0-499: Devuelve los 500 primeros bytes. 

Si y no aparece entonces significa “dame desde el byte x hasta el final”:

  * Range: bytes=9000: El cliente quiere desde el byte 9000 (inclusive) hasta el finak 

Si x no aparece entonces significa “dame los últimos y bytes”. P. ej:

  * Range: bytes=-500: El cliente quiere los últimos 500 bytes. 

La cabecera admite distintos rangos separados por comas:

  * Range: bytes=100-200,400-500, –800: Dame los bytes del 100 al 200. Y del 400 al 500 y los últimos 800. 

Si los rangos se solapan esto no debe generar un error:

  * Range: bytes=500-700, 601-999: El cliente quiere los bytes del 500 al 700 y del 601 al 999. 

> **Nota:** Que la cabecera Range empiece por bytes= no es superfluo. El estándard es extensible y permite que se puedan definir otras unidades además de bytes para especificar los rangos. De ahí que deba especificarse la unidad usada en la cabecera Range. De hecho el servidor puede usar la cabecera Accept-Ranges para especificar que unidades de rangos soporta (p. ej. Accept-Ranges: bytes). Nosotros nos centraremos única y exclusivamente en rangos de bytes.

**Respuesta del servidor**

Si el cliente solo pide un rango, la respuesta del servidor es una respuesta normal, **salvo que en lugar de usar el código 200, devuelve un 206 (Partial content) y con la cabecera _Content-Range_ añadida**.

El formato de la cabecera Content-Range es el rango servido, seguido por la longitud total del elemento separado por /. P. ej:

  * Content-Range: bytes 100-200/5000 –> Se están sirviendo los bytes 100 a 200 (ambos inclusives) de un recurso cuya longitud es de 5000 bytes. 
  * Content-Range: bytes 100-200/* –> Se están sirviendo los bytes 100 a 200 de un recurso cuya longitud es desconocida. 

> Sí, en Content-Range **no hay el símbolo =** entre bytes y el valor de rango. Mientras que en la cabecera Range si que existe dicho símbolo…

Por otra parte, si el cliente ha pedido más de un rango la respuesta del servidor pasa a ser una _multipart_, es decir, dado que el cliente nos envía varios rangos, en la respuesta debemos incluirlos todos por separado. Hay varias cosas a tener presente.

  1. El código de retorno **no es 200, es 206** (Como en el caso anterior) 
  2. El content-type debe establecerse a multipart/byteranges **y debe indicarse cual es la cadena separadora** (el _boundary_). 
  3. Cada _subrespuesta_ viene precedida del _boundary_ y tiene su propio content-type y content-range indicados. 

Un ejemplo sería como sigue (sacado de [http://greenbytes.de/tech/webdav/draft-ietf-httpbis-p5-range-latest.html#status.206][1]):

<pre>HTTP/1.1 206 Partial Content
Date: Wed, 15 Nov 1995 06:25:24 GMT
Last-Modified: Wed, 15 Nov 1995 04:58:08 GMT
Content-Length: 1741
Content-Type: multipart/byteranges; boundary=THIS_STRING_SEPARATES
--THIS_STRING_SEPARATES
Content-Type: application/pdf
Content-Range: bytes 500-999/8000
...the first range...
--THIS_STRING_SEPARATES
Content-Type: application/pdf
Content-Range: bytes 7000-7999/8000
...the second range
--THIS_STRING_SEPARATES--</pre>

Si el servidor **no puede satisfacer la petición del cliente debido a que los rangos pedidos son inválidos o hay demasiados que se solapan** o lo que sea, puede devolver un HTTP 416 (Range not Satisfiable). Si lo hace _deberia_ añadir una cabecera Content-Range inválida con el formato:

Content-Range: bytes */x (siendo x el número de bytes totales del recurso).

Otra opción si los rangos son inválidos es **devolver el recurso entero usando un HTTP200 tradicional** (muchos servidores hacen esto, ya que si un cliente usa rangos debe estar siempre preparado por si el servidor no los admite).

**Cabecera If-Range**

Nos falta mencionar la cabecera If-Range. La idea de dicha cabecera es que el cliente pueda decir algo como “_Oye, tengo una parte del recurso descargado pero es de hace 3 días. Si no ha cambiado, pues me mandas los rangos que te indico en Range, en caso contrario, pues que le vamos a hacer me lo mandas todo con un 200 tradicional_”.

El valor de If-Range puede ser, o bien un ETag que identifique el recurso o bien una fecha. Si no sabes lo que es un ETag, pues bueno es simplemente un identificador del contenido (o versión) del recurso. Puede ser un valor de hash, un valor de revisión, lo que sea. No hay un estándar definido, pero la idea es que si el cliente sabe el ETag de un recurso y lo manda, el servidor puede indicarle al cliente si dicho recurso ha cambiado o no. El cliente manda el ETag que tiene para dicho recurso (en la cabecera If-Range o bien en la If-None-Match si no hablamos de rangos).

Oh, por supuesto, uno _podría_ crear un servidor que devolviese un ETag único cada vez que el cliente no le pasa un ETag previo y devolver siempre ETag recibido por el cliente en caso contrario. En este caso, se podría asegurar que cada cliente tendría un ETag distinto. ¿Ves por donde vamos, no? Un mecanismo sencillo y barato para distinguir usuarios únicos. Mucho mejor que todas esas maléficas cookies y además para usar ETags no es necesario colocar ningún popup ni nada parecido. Además, a diferencia de las cookies que se eliminan borrando las cookies (lo que mucha gente hace), los ETags se borran generalmente vaciando la cache del navegador (lo que hace mucha menos gente). Por supuesto, he dicho que uno _podría_ hacer esto… no que se haya hecho 😛

Bueno… Hasta ahí el primer post. Rollo teórico, pero bueno, siempre es importante entender como funcionan las cosas ¿no?… <a href="http://geeks.ms/blogs/etomas/archive/2014/04/10/asp-net-mvc-descargar-ficheros-con-soporte-para-continuaci-243-n-ii.aspx" target="_blank" rel="noopener noreferrer">en el siguiente post pasaremos a la práctica</a>!!! 😀

 [1]: http://greenbytes.de/tech/webdav/draft-ietf-httpbis-p5-range-latest.html#status.206 "http://greenbytes.de/tech/webdav/draft-ietf-httpbis-p5-range-latest.html#status.206"