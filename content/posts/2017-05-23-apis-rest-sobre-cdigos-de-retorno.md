---
title: 'APIs REST: Sobre códigos de retorno'
description: 'APIs REST: Sobre códigos de retorno'
author: eiximenis

date: 2017-05-23T08:10:31+00:00
geeks_url: /?p=1879
geeks_ms_views:
  - 2187
categories:
  - webapi

---
Un [buen amigo][1] escribía [lo siguiente][2] el otro día en twitter:

_Me ha estallado un ojo hoy   
1) Llamo a un Endpoint de una API   
2) No me devuelve datos porque no encuentra lo que pido   
3) Me devuelve un 404_
  
Él estaba en contra del uso de 404 para indicar que no se encuentra un determinado recurso. A partir de aquí se sucedieron varios tweets y eso me ha motivado a escribir este post sobre **códigos de retorno en una API REST**, con **MIS** opiniones al respecto, por supuesto! Y como digo siempre, todo debate será bienvenido!
  
<!--more-->


  
**Devolver 404 vs null vs 204**
  
Mucha gente piensa que el 404 significa “la URL no existe”. Pero eso es falso: la URL existe ya que alguien te ha respondido precisamente con un 404. De hecho según la especificación oficial de HTTP (a la que todos deberíamos adherirnos si queremos cumplir REST) significa literalmente “_The server has not found anything matching the Request-URI_”.
  
De hecho, como curiosidad, lo más parecido que hay en HTTP a un código que sea “El servicio (NO el recurso) al cual hace referencia la URL no existe” es el 503, pero no significa que no exista si no que está caído (por cualquier razón).
  
Es decir: el servidor **no ha encontrado ningún recurso en esta URL. No dice que la URL sea inválida**. También la especificación deja claro que nada en un 404 indica que este sea temporal o permanente. ¿Qué significa eso en el contexto de una API REST? Recuerda: en HTTP (y por lo tanto en REST) las URLs **representan recursos. Si un recurso no existe lo suyo es devolver un 404**.
  
¿Crees que a tu cliente le importa si hay un controlador, un _middleware,_ un _servlet_ o lo que sea respondiendo a tu petición? **Tu cliente ha pedido un recurso. Si el recurso no existe, no le engañes: devuelvele un 404**.
  
**¿Qué no debes hacer nunca de los jamases?** Devolver un 200 con un JSON que diga algo como “Recurso no encontrado”. Un 200 significa que la petición es correcta y que en el _payload_ está el resultado. Preguntar por algo que NO existe no es una petición correcta.
  
**¿Qué no debes hacer tampoco nunca?** Devolver un “null” serializado. Eso ocurre p. ej. en WebApi. Si devuelves un _null_, el cliente recibe un 200 con el null serializado (en JSON&nbsp; es _null,_ pero en XML es un XML feote). A ver… si el cliente te ha pedido la cerveza con id 123, y no existe… ¿por qué le devuelves null? Déjale bien claro que la cerveza con id 123 no existe: 404 al canto.
  
**¿Que otra cosa deberías evitar hacer?** Devolver un 204 (No-Content). Un 204 significa que el recurso existe pero _está vacío_ (y por lo tanto no hay nada que mandar al cliente). Es muy distinto que algo exista pero esté vacío a que no exista. Para lo primero 204, para lo segundo 404.
  
**Devolver colecciones vacías ¿sí o no?** Imagina que invoca la url /api/beers para pedir todas las cervezas, solo que en este momento no hay ninguna. ¿Qué devuelves? ¿Un 404? ¿O un 200 con un json de array vacío? Para mi **esta situación es distinta de la anterior.** La pregunta es la misma: ¿existe el recurso /api/beers (la colección de cervezas)? Fíjate que el recurso existe: tenemos una colección de cervezas, solo que ahora está vacía. Así que devolver un 200 con la colección vacía es totalmente correcto, pero si has leído el punto anterior ya habrás deducido que hay otra opción: devolver un 204**.**
  
Entre un 204 o un 200 con un array vacío (no contemplo el 404 puesto que no lo considero correcto en este caso) me inclino personalmente con el 200 con el array vacío. Personalmente dejo el 204 para respuestas a peticiones de actualización (síncronas, que si son asíncronas entonces debe ser un 202), en especial los DELETE en los que poca cosa hay que devolver (para creaciones de recursos nuevos es mejor usar el 201). Aunque bueno… [los 204 pueden ser peligrosos si quieres usar hypermedia][3].
  
Y qué ocurre con los filtros? Como indicamos&nbsp; que un filtro aplicado produce “ningún resultado”: P. ej. ¿una url tipo “/api/beers?name=e*” que debe devolver si no hay ninguna cerveza cuyo nombre empiece por “e”?
  
Pues aquí hay cierta confusión. Inicialmente en la [RFC2396][4] la query string **no identificaba al recurso.** Literalmente pone esto:

<pre>The query component is a string of information to be interpreted by the resource</pre>

Eso significa que la query string es un dato _que debe ser procesado por el recurso_**.** Siguiendo esta definición, entonces el recurso sería /api/beers por lo que devolver un 404 al aplicar query strings sería incorrecto (por el mismo motivo por el cual es incorrecto aplicarlo a la URL sin query string: el recurso sigue existiendo). **Pero en la** [**RFC3986**][5] **se cambió el significado de la query string**:

<pre>The query component contains non-hierarchical data that, along with data in the path component (Section 3.3), serves to identify a resource</pre>

Eso da a entender que puede usarse la query string para definir el recurso. Bajo este punto de vista, se podría devolver un 200 para /api/beers y un 404 para /api/beers?name=e* ya que serían recursos distintos. Yo personalmente, me inclino por devolver 200 (con el array vacío si no hay resultados) cuando se aplican filtros, pero bueno, ahí queda 🙂
  
**¿401 vs 403?**
  
Esa es otra de las clásicas. Parece que todos tenemos claro cuando devolver un 401: cuando un usuario no autenticado intenta acceder a una URL que es solo para usuarios autenticados. Pero… ¿y si un usuario autenticado intenta acceder a un recurso para el cual él no tiene permisos? ¿Qué le devolvemos en este caso? ¿Un 401? ¿O un 403? Veamos que dice [la especifación HTTP al respecto][6].
  
Sobre el 401 indica eso (el énfasis es mío):

<pre>The request requires user authentication. The response MUST include a WWW-Authenticate header field (section 14.47) containing a challenge applicable to the requested resource. The client MAY repeat the request with a suitable Authorization header field (section <a href="https://www.w3.org/Protocols/rfc2616/rfc2616-sec14.html#sec14.8">14.8</a>). <strong>If the request already included Authorization credentials, then the 401 response indicates that authorization has been refused for those credentials</strong></pre>

O sea que según esto es correcto devolver un 401 incluso para usuarios autenticados. ¿Y entonces el 403? Pues la especificación nos dice esto (de nuevo, el énfasis es mío):

<pre>The server understood the request, but is refusing to fulfill it. <strong>Authorization will not help and the request SHOULD NOT be repeated</strong></pre>

La frase destacada es importante: La autorización no va a funcionar. Es decir, esta petición (por la razón que sea, pero NO tiene que ver con permisos del cliente) está prohibido**.**
  
Bajo este punto de vista **403 no debe usarse para indicar que el usuario no tiene permisos y debemos usar un 401… pero resulta que esto NO ES así**.
  
¿Por qué? Pues por el [RFC7231][7]. Este RFC define la semántica de la autorización HTTP y, entre otras cosas, da la respuesta a esta cuestión. En su definición del estado 403 pone literalmente:

<pre>If authentication credentials were provided in the request, the server considers them insufficient to grant access</pre>

O sea **que sí. Que se puede devolver un 403 si un usuario autenticado no tiene permisos para acceder a un recurso**. Fin de la discusión. Y no solo eso… dado que **está permitido usar un 404 en lugar de un 403** si se quiere esconder un recurso no accesible, esto significa que:

  1. Si el usuario no está autenticado o la autenticación es incorrecta debemos devolver un 401 
      * Si el usuario está autenticado pero no tiene permisos podemos devolver o bien un 403 o bien un 404 si queremos “ocultar” la existencia de recursos no accesibles.</ol> 
    Bueno… eso es todo. ¿Y por supuesto… qué opinión tenéis vosotros? ¿Qué otras dudas tenéis en los códigos de retorno?
  
    Saludos!

 [1]: https://twitter.com/J0rgeSerran0
 [2]: https://twitter.com/J0rgeSerran0/status/866666093347209217
 [3]: http://blog.ploeh.dk/2013/04/30/rest-lesson-learned-avoid-204-responses/
 [4]: http://www.ietf.org/rfc/rfc2396.txt
 [5]: http://www.ietf.org/rfc/rfc3986.txt
 [6]: https://www.w3.org/Protocols/rfc2616/rfc2616-sec10.html
 [7]: https://tools.ietf.org/html/rfc7231#section-6.5.3