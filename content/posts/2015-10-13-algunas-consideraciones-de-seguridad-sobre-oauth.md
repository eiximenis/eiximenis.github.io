---
title: Algunas consideraciones de seguridad sobre OAuth
description: Algunas consideraciones de seguridad sobre OAuth
author: eiximenis

date: 2015-10-13T16:30:42+00:00
geeks_url: /?p=1706
geeks_visits:
  - 838
geeks_ms_views:
  - 2012
categories:
  - oauth

---
Cuando hablo de OAuth y del acceso con _tokens_, ya sea en formaciones o en charlas más o menos coloquiales, muchas veces surgen dudas sobre _cuan seguros son los tokens_. En este post, sin pretender dar un repaso exhaustivo a todos los flujos OAuth existentes, si que me gustaría comentar algunas consideraciones básicas de seguridad.

**En el principio estaba OAuth 1.0**

La <a href="http://tools.ietf.org/html/rfc5849" target="_blank" rel="noopener noreferrer">versión 1.0 de OAuth</a> especifica, básicamente, un solo token, conocido como _oauth_token_. Dicho token es el que, enviado junto a una petición, permite validar que la petición es enviada un cliente en concreto. Dejando de lado como obtenemos dicho token (los distintos flujos OAuth) el resultado final es que el cliente debe mandar lo siguiente en cada petición:

  1. El _oauth\_consumer\_key_: El ID (público) de cliente. Dicho ID identifica a la aplicación cliente, aunque no a un usuario determinado. Es decir, identifica a la _aplicación cliente_, no al usuario. 
  2. El _oauth_token_ que identifica a un usuario en concreto. El _oauth_token_ conjuntamente al _oauth\_consumer\_key_ identifican un usuario concreto de una aplicación. 

Visto así, parece que si alguien interceptase una petición concreta, podría ver tanto el _oauth\_consumer\_key_ como el _oauth_token_ y de esa manera impersonar al usuario en concreto. Para protegernos de eso, lo más sencillo es usar un canal seguro (es decir https) pero **OAuth 1.0 incorpora un mecanismo para securizar nuestros tokens** incluso en canales no seguros: **la firma de peticiones**.

Cada petición OAuth debe ir firmada digitalmente, y esa firma digital se realiza utilizando _otro_ token (_oauth\_token\_secret_), dicho token es conocido es conocido solo por el cliente y el servidor y no es enviado nunca por el cliente (el cliente lo recibe _junto_ su token oauth cuando realiza el flujo correspondiente). De esta manera si alguien intercepta una petición y obtiene tanto el _oauth\_consumer\_key_ como el _oauth_token_ no podrá generar nuevas peticiones válidas, ya que no podrá firmarlas. Por supuesto podría reenviar la petición tantas veces como quisiera (lo que podría tener consecuencias fatales), pero nunca podría enviar una petición modificada o con otros datos.

Para evitar que alguien pueda reenviar una petición se establece un nuevo mecanismo de seguridad en OAuth: Cada petición incluye un _nonce_ (acrónimo de “number once”) que es un identificador “único” de petición. El formato del _nonce_ queda a elección del servidor (puede ser un _guid,_ puede ser un timestamp en formato Unix, o cualquier otra cosa). La idea es que el servidor no aceptará dos peticiones del mismo cliente con el mismo _nonce_ (en cierto intervalo de tiempo a definir por el servidor). Además, las peticiones OAuth llevan un timestamp que indica cuando el cliente las emite (en un tiempo que debe estar sincronizado con el del servidor, por lo que se suele usar UTC). El servidor valida dicho timestamp y si no coincide con el de la hora del servidor (dentro de cierto margen de tolerancia) la petición es rechazada. De este modo basta con que los _nonce_ no se puedan repetir en todo el periodo de tolerancia del timestamp. Debe notarse que tanto el _nonce_ como el __timestamp forman parte de la petición, por lo que son firmados digitalmente (y por lo tanto no pueden alterarse).

Creedme si os digo por experiencia que la parte más dura de crear un cliente (o un servidor) OAuth es la creación y/o validación de las firmas digitales. Eso incluye crear lo que la especificación llama una “forma canónica de la petición” que no es trivial. La gran mayoría de errores de implementación de clientes o servidores OAuth vienen por ahí.

A cambio tenemos una cierta garantía de que incluso en canales no seguros, si alguien intercepta una petición no podrá usar los tokens para generar nuevas peticiones y impersonar al usuario. Las únicas peticiones en OAuth 1 que debemos mandar bajo https sí o sí son las que nos permiten obtener los tokens iniciales… Aunque mi recomendación sería usar siempre https por supuesto.

**OAuth 2 y los bearer tokens**

OAuth 2 cambió las normas del juego: dado que la generación y validación de la firma digital son complejos, en OAuth 2 se elimina la firma digital, lo que deja los tokens en una situación bastante más comprometida.

De hecho si en OAuth 1 era bastante natural que un _oauth_token_ tuviese una validez muy larga (o incluso que no caducase) en OAuth 2 eso sería como pegarse un tiro: Una vez el cliente obtiene un token oauth (al igual que en OAuth 1.0 existen varios flujos para hacerlo) dicho token realmente impersona al usuario. Es decir, si alguien intercepta el token, es a todas luces, como si tuviese las credenciales del usuario… durante el tiempo de validez del token. A ese tipo de tokens los conocemos como _bearer tokens_.

Insisto para dejarlo claro: si una petición incluye el _bearer token_ dicha petición se considerada autenticada y además impersona a un usuario en concreto. Por lo tanto cualquiera que intercepte una petición que incluya el _bearer token_ tiene abiertas las puertas de par en par.

Para minimizar daños se hace que los _bearer tokens_ tengan una duración corta (lo de corta depende de cada uno), de forma que a pesar de tener un _bearer token_ uno puede, al cabo de cierto tiempo, recibir un 401 lo que indica que el token ha caducado. En este caso existe un flujo OAuth 2 que permite “renovar” el token. Renovar el token incluye enviar _otro_ token específico (el _refresh token_). Dicho token se obtiene junto al token de acceso y sirve solo para esto: para refrescar el token de acceso cuando ha caducado. Dado que el _refresh token_ no viaja en cada petición, alguien que haya obtenido un token oauth, no podrá renovarlo.

Por supuesto si este alguien que está escuchando intercepta una petición de “refresco de token” entonces si que la situación se vuelve complicada de verdad: a pesar de que los _refresh tokens_ pueden caducar la verdad es que su tiempo de vida suele ser muy largo y aunque se pueden revocar esto solo consigue “cerrar las puertas” una vez alguien ya ha podido hacer daño.

En resumen OAuth 2 con los _bearer tokens_ toma una filosofía muy clara: que el canal de comunicaciones es seguro. O dicho de otro modo: **no puede usarse OAuth 2 con _bearer tokens_ si no se usa un canal seguro (https)**. Y no me refiero solo a las peticiones que obtienen los tokens, si no a todas. **Todas las peticiones OAuth 2 con _bearer tokens_ deben enviarse bajo https**. Si no, la seguridad queda totalmente comprometida.

**MAC Tokens**

Para ser justos con OAuth 2, no obliga a usar _bearer tokens_ (a pesar de que esos sean los más usados). OAuth 2 soporta también otro tipo de tokens, conocidos como tokens MAC (Message Authentication Code). En este caso OAuth 2 se comporta de forma parecida a OAuth 1: cuando el cliente obtiene el token, obtiene también _otro_ token adicional que le sirve para firmar las peticiones. En este caso, al igual que en OAuth 1, se usan los parámetros _nonce_ y _timestamp_ de la petición para evitar que alguien pudiese reenviar peticiones. Al igual que en OAuth 1 se soportan varios algoritmos de firma digital (aunque <a href="https://en.wikipedia.org/wiki/Hash-based_message_authentication_code" target="_blank" rel="noopener noreferrer">HMAC SHA256</a> suele ser el más usado).

Si usamos tokens MAC el requisito de canal seguro desaparece, excepto, por supuesto, en las peticiones que permiten obtener los tokens.

Os dejo algunos enlaces con más
  
información al respecto:

  * <a href="http://tools.ietf.org/html/rfc6749" target="_blank" rel="noopener noreferrer">Especificación OAuth 2</a>
  * <a href="http://hueniverse.com/2010/09/29/oauth-bearer-tokens-are-a-terrible-idea/" target="_blank" rel="noopener noreferrer">OAuth Bearer tokens are a terrible idea</a>
  * <a href="http://blog.facilelogin.com/2013/01/oauth-20-bearer-token-profile-vs-mac.html" target="_blank" rel="noopener noreferrer">Bearer token vs MAC Token</a>
  * <a href="http://tools.ietf.org/html/rfc5849" target="_blank" rel="noopener noreferrer">Especificación OAuth 1</a>

Espero que si teníais alguna duda al respecto esto os ayude un poco 🙂