---
title: 'C#5 Tomando una cerveza con Feynman (o await 2a parte)'
author: eiximenis

date: 2011-09-19T16:40:59+00:00
geeks_url: /?p=1577
geeks_visits:
  - 2281
geeks_ms_views:
  - 1667
categories:
  - Uncategorized

---
En el [post anterior][1] vimos como gracias a C# 5 y las nuevas palabras clave async y await el uso de métodos asíncronos era tan sencillo como ir al bar y tomarnos una cerveza. Como resumen del post vimos que async nos permitía indicar que un método quería realizar llamadas asíncronas y await nos permitía esperarnos al retorno de una llamada asíncrona. Si no has leído el post, antes de leer este échale un vistazo.

En el post de ayer hice una simplificación, porque no quería liar las cosas más de la cuenta y total sólo iba a tomarme una cerveza. Hoy para mi desgracia, y para la vuestra, he quedado con [Richard Feynman][2] para tomarme esa misma cerveza… No interpretéis mal lo de “desgracia”, honestamente si pudiese elegir alguna personalidad de todos los tiempos con los que poder sentarme en una terraza y hablar de temas varios él estaría en uno de los primeros lugares de la lista. 

Supongamos el mismo ejemplo de ayer. Es decir, yo me voy al bar,vistiendo mi **jersey rojo** y allí me encuentro al bueno de Richard, tomando una cervecilla. Me uno a él, pido una cerveza (o sea, le encargo al camarero la tarea de que vaya al almacén a buscar una cerveza y me la traiga)&#160; y mientras esperamos hablamos de cosas de la vida. En esto, Richard que es un poco despistado y no se ha dado cuenta de que todavía _no tengo_ mi cerveza, propone un brindis. Pero como todavía no tengo mi cerveza nos esperamos a que el camarero me la traiga. Cuando el camarero nos trae la cerveza, Richard sonríe y me dice que el nuevo **jersey azul** que me he comprado es muy bonito. A eso yo me miro y efectivamente veo que llevo un jersey azul nuevo. Pero… _no estaba yo esperando a que el camarero me trajera la cerveza?? No recuerdo haber ido a comprar ningún jersei azul… o sí?_

¿Que ha ocurrido? La verdad es que Richard me lo explicó, a su manera claro, usando su [teoría de las múltiples historias][3], y resumiendo eso es más o menos lo que ocurrió…

La clave del meollo es que cuando nos esperamos a que el camarero nos trajera la cerveza **no** _nos quedamos sin hacer nada_. Al contrario, cada uno de nosotros _continuamos haciendo las tareas que teníamos pendientes hacer_. En mi caso, comprarme un jersei rojo (en el de Richard probablemente revolucionar la física moderna con algún nuevo descubrimiento). Luego, en el momento en que el camarero llega con mi cerveza, tanto Richard como yo _continuamos haciendo lo que teníamos que hacer cuando yo tuviese mi cerveza: el brindis_. Ya… eso es lo que ocurre cuando hablas con genios de la talla de Feynman: te dejan el cerebro hecho fosfatina.

Y ahora, por salud mental, permitidme que abandone el ejemplo de la cerveza y hable en términos de programadores, a ver si nos entendemos mejor. La clave es que _await_ no realiza una espera bloqueante (si lo hiciera convertiríamos la tarea por la que esperamos en una llamada síncrona). Lo que realmente hace await es que el método _async_ devuelve a su llamador _justo en este momento_. I el _resto del método async (el código que sigue al await) se ejecutará cuando la tarea por la que está esperando await ha terminado_.

Imaginad el siguiente código que representa un día en mi vida:

[<img title="image" style="border-top: 0px; border-right: 0px; background-image: none; border-bottom: 0px; padding-top: 0px; padding-left: 0px; border-left: 0px; display: inline; padding-right: 0px" border="0" alt="image" src="http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_3E618E88.png" width="324" height="105" />][4]

Simplemente IrAlBareto() y luego IrAComprarRopa().

El código de IrAlBareto(), que es un método async, es tal y como sigue:

[<img title="image" style="border-top: 0px; border-right: 0px; background-image: none; border-bottom: 0px; padding-top: 0px; padding-left: 0px; border-left: 0px; display: inline; padding-right: 0px" border="0" alt="image" src="http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_4E8D5681.png" width="381" height="152" />][5]

[<img title="image" style="border-top: 0px; border-right: 0px; background-image: none; border-bottom: 0px; padding-top: 0px; padding-left: 0px; border-left: 0px; display: inline; padding-right: 0px" border="0" alt="image" src="http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_7AC9DD65.png" width="374" height="73" />][6]

Me voy al bar, hablo de cosas de la vida con Richard, pido mi cerveza al camarero, me espero por ella con _await_ y luego hacemos un brindis. Ahora viene la clave: **lo que ocurre al llamar a await es que el método IrAlBareto retorna a su llamador (el método UnDiaEnMiVida).** Con lo que la ejecución del sistema continúa con la llamada a IrAComprarRopa():

[<img title="image" style="border-top: 0px; border-right: 0px; background-image: none; border-bottom: 0px; padding-top: 0px; padding-left: 0px; border-left: 0px; display: inline; padding-right: 0px" border="0" alt="image" src="http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_1214E1D7.png" width="321" height="199" />][7]

Finalmente, cuando el método _PedirCerveza,_ que es por el cual estábamos haciendo _await_ termina, se **continua ejecutando el código que sigue al await, es decir la llamada al método Brindis**

Esa es la salida que genera el programa:

[<img title="image" style="border-top: 0px; border-right: 0px; background-image: none; border-bottom: 0px; padding-top: 0px; padding-left: 0px; border-left: 0px; display: inline; padding-right: 0px" border="0" alt="image" src="http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_64471911.png" width="425" height="208" />][8]

He dejado el código del proyecto en mi [carpeta de Skydrive – AsyncAwait2][9]. Es un proyecto console application de VS2011 Developers Preview.

Así pues recordad:

  1. Await realiza una espera no bloqueante sobre un objeto awaitable (una Task). 
  2. Mientras dura la espera de await, el método async (que contiene la llamada a await) retorna a su llamador
  3. Cuando la espera de await ha terminado, se ejecuta el resto del código del método async que contenía el await (exacto… es como si el resto del código que sigue al await fuese el callback!).

Un saludo a todos… y felices cervezas! xD

 [1]: http://geeks.ms/blogs/etomas/archive/2011/09/17/c-5-async-await.aspx
 [2]: http://en.wikipedia.org/wiki/Richard_Feynman
 [3]: http://en.wikipedia.org/wiki/Multiple_histories
 [4]: http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_33187A4B.png
 [5]: http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_6B767B56.png
 [6]: http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_22705990.png
 [7]: http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_6C1F3180.png
 [8]: http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_17EF8570.png
 [9]: https://skydrive.live.com/?cid=6521c259e9b1bec6&sc=documents&uc=1&id=6521C259E9B1BEC6%21167# "AsyncAwait2"