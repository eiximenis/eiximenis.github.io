---
title: '[VS2008] Visual 2008 casca al adjuntarse a un proceso'

author: eiximenis

date: 2010-03-30T12:22:54+00:00
geeks_url: /?p=1503
geeks_visits:
  - 639
geeks_ms_views:
  - 731
categories:
  - Uncategorized

---
Post rapidito… Me encontré que VS2008 me rebentaba al adjuntarme a un proceso para depurar. Aunque salía el mensaje de “Visual Studio ha causado un error y debe cerrarse”, dándole a OK, Visual Studio no se cerraba y realmente se adjuntaba al proceso… eso sí, todas las ventanas se iban a la posición que les daba la gana… lo cual da bastante por el saco si tienes que ir poniéndolas cada una a su sitio de nuevo…

<!--more-->

En fin, si te ocurre esto o bien VS2008 te da errores cuando mueves y/o desacoplas ventanas, descárgate el patch que se encuentra en [http://code.msdn.microsoft.com/KB960075][1] y listos.

Ah, por cierto, y si no te ha pasado nunca no te confíes… yo hasta hoy no lo había necesitado… En fin…

Saludos!

 [1]: http://code.msdn.microsoft.com/KB960075 "http://code.msdn.microsoft.com/KB960075"