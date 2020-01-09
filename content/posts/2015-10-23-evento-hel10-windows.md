---
title: '[Evento]–Hel10 Windows'
author: eiximenis

date: 2015-10-23T10:20:55+00:00
geeks_url: /?p=1708
geeks_visits:
  - 326
geeks_ms_views:
  - 994
categories:
  - eventos

---
Buenas! Ayer (22 octubre 2015) se celebró el “Hel10 Windows” un evento organizado por Microsoft en Madrid para hablar de las muchas novedades que incorpora Windows 10 y todo el ecosistema nuevo de la plataforma universal.

Antes que nada mi opinión general del evento: Estuvo basante bien. Una _keynote_ un poco sosilla para mi gusto, a pesar de que algunas de las novedades presentadas eran bastante interesantes. Pero, por un lado vivimos en la época de la información globalizada y quien más quien menos ya había oído hablar de _continuum_ y el resto de novedades, y por otro no tenemos el sentido del espectáculo de los americanos (por suerte o por desgracia según cada cual).

Después de la _keynote_ empezaron cuatro tracks, los nombres oficiales de los cuales están el la <a href="https://www.desarrollaconmicrosoft.com/windows10hel10world/" target="_blank" rel="noopener noreferrer">página del evento</a>, pero que básicamente eran:

  1. Desarrollo de aplicaciones universales y la UWP. Este track es el que se ha retransmitido via channel 9.
  2. Cosas raras de desarrollo. En este track estuve yo hablando sobre WinObjC.
  3. IoT y cacharritos
  4. Sistemas

Como digo yo estuve hablando de **WinObjC** o lo que es lo mismo “project Islandwood” o “Bridge for iOS” que es el nombre oficial. Se trata de un conjunto de herramientas que permitirá portar nuestras aplicaciones iOS en Objective-C a Windows10 como aplicación universal. Manteniendo la funcionalidad pero adaptando un poco el aspecto al aspecto de las aplicaciones universales.

Durante la charla tuve malditos problemas técnicos con _Parallels_ que básicamente me impidió ejecutar las demos. Mucha rabia porque había usado _Parallels_ (de hecho lo uso constantemente) sin problemas, pero no se si fue que el proyector estaba en una resolución no soportada por mi Mac o qué, pero el resultado es que el ratón en _Parallels_ iba donde le daba la gana y era imposible darle click a nada… En fin, que le vamos a hacer. Me limité a mostrar el código en XCode y como usar compilación condicional para reemplazar aquellas partes que el bridge no soporta con llamadas a la API nativa de Windows 10.

En la charla intenté explicar un poco lo que hay hecho, pero sin engañar a nadie: el proyecto está muy verde, y es más la promesa de lo que va a ser que lo que es en la actualidad. El soporte para ficheros.xib, storyboards y layout constraints es casi obligatorio, porque sin esas tres características casi no se va a poder migrar aplicación alguna (excepto juegos). Lo que hay hecho, a pesar de ser una _preview_ que, en mi opinión, no llega ni a alfa, pinta interesante, pero se tiene que ver como evoluciona.

He dejado la presentación de la charla en slideshare: [http://www.slideshare.net/eduardtomas/winobjc-windows-bridge-for-ios][1]

Y si te animas a probar las demos, las he subido todas a un repo de github para que puedas jugar con ellas: [https://github.com/eiximenis/WinObjC-Samples][2]

Saludos!

 [1]: http://www.slideshare.net/eduardtomas/winobjc-windows-bridge-for-ios "http://www.slideshare.net/eduardtomas/winobjc-windows-bridge-for-ios"
 [2]: https://github.com/eiximenis/WinObjC-Samples "https://github.com/eiximenis/WinObjC-Samples"