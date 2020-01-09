---
title: La odisea de conectar mi portátil a mi televisor…
description: La odisea de conectar mi portátil a mi televisor…
author: eiximenis

date: 2009-09-09T09:17:01+00:00
geeks_url: /?p=1465
geeks_visits:
  - 34237
geeks_ms_views:
  - 3932
categories:
  - Uncategorized

---
Esta es una historia de como el desconocimiento de unos (el mío) y de otros (los de la tienda) convierte en una odisea algo que debería ser mucho más sencillo… y que sin embargo tampoco es lo simple que debiera.

Me explico: Llevo ya 3 viajes a la tienda de _cables_ (vamos a llamarla así) para encontrar un modo de conectar mi portátil (un latitude e6400) a mi televisor Philips de pantalla plana. Es decir&#160; ni el portátil ni el televisor son del año de la maría castaña (aunque el televisor no sea full hd, sólo hd ready). Soy (o mejor dicho _era_) un auténtico ignorante en materia de conexiones, cables, señales y demás. Jamás me había importado este tema hasta ahora… pero esto no debería ser un problema, uno supone que cuando va a comprar algo quien le vende el producto debe ser capaz de informarle… iluso.

**Viaje 1: Intento de conexión a HDMI**

Mi televisor tiene entrada HDMI, y esa es la entrada que yo me planteé utilizar para la conexión… total se supone que HDMI es la repera, así que vamos a usarla, no? Bien, voy a la tienda y digo que quiero conectar mi portátil, que **sólo tiene salida VGA**, a la entrada HDMI de mi televisor. Me dicen que no hay adaptador VGA a HDMI (es cierto) pero que si que lo hay de VGA a DVI y luego de DVI a HDMI. Así que compro el cable VGA –> DVI y luego el adaptador DVI a HDMI (mi televisor no tiene entrada DVI). Llego a casa, lo conecto… y no va.

_Luego_ (como siempre sólo busco información cuando las cosas fallan :p), me entero de que **es imposible enchufar una señal VGA a una entrada HDMI sin mediar por medio un conversor de señal**. La razón? VGA es analógica y HDMI es digital… así que o bien se digitaliza la señal proveniente de la VGA o nada de nada.

Entonces… por que existen cables VGA –> DVI y adaptadores DVI a HDMI? Google me dio la respuesta: Resulta ser que existen **tres** tipos de cables DVI: DVI-A, que transporta señal analógica, DVI-D, que transporta señal digial y DVI-I que puede transportar ambas señales pero _no_ puede realizar una conversión. Es decir, con un cable DVI-I puedo conectar un DVI-A con una entrada analógica o bien un DVI-D con una entrada digital, pero no puedo conectar un DVI-D con un DVI-A (por ejemplo). HDMI por su parte es sólamente digital, los conversores HDMI estan pensados para convertir a HDMI una entrada DVI-D, o DVI-I siempre que transporte señal digital.

**Viaje 2: De VGA a RGB (RCA)**

Finalmente miré el manual de la tele, y mira por donde hay un apartado que dice cómo conectar una salida VGA a la tele… según ellos debe usarse un cable VGA –> RGB H/V. Total que me fui a la tienda y terminé llevándome un cable VGA –> RGB. Reconozco que no estaba muy convencido, porque según el manual el cable VGA –> RGB H/V tiene cinco pins para conectar a la tele, pero el cable que me dieron solo tenía tres pins: uno verde, uno azul y el otro rojo. Y los conecté a la entrada verde, azul y roja del televisor… y no se veía nada 🙁

Todavía hoy no se cual es la diferencia entre un cable VGA –> RGB y un cable VGA –> RGB H/V (bueno, se supone que los dos pins addicionales son para el sincronismo horizontal y vertical). Pero bueno… un cable VGA –> RGB no funciona, aunque ha sido con el que más me he acercado… he llegado a _intuir_ la imagen, aunqué toda rosada y moviendose continuamente… por más que he cambiado la resolución y/o la frecuencia de refresco no he conseguido nada.

**Viaje 3: De VGA a Euroconector (SCART)**

De nuevo a la tienda, dispuesto a hablar con el dependiente que me había atendido las otras dos veces para decirle a ver si tenía el cable RGB H/V de las narices… pero no estaba. Así que de nuevo conté que tenía un portátil con salida VGA y una tele que _no_ tenia entrada VGA y que el cable VGA –> RGB no me había funcionado. Le pregunté por el cable VGA –> RGB H/V pero su cara fue un poema… Me dijo que nunca había visto ese cable y me preguntó si mi tele tenía entrada por euroconector, a lo que yo le respondí que sí y el me dijo que entonces ningún problema y me dio un cable VGA a euroconector. Y yo me lo llevé, aunque en mi descargo debo decir que no muy convencido.

Y evidentemente, no funcionó… Luego, leyendo por ahí, he visto que este cable sólo funciona si el emisor (o sea mi portátil) emite en la misma frecuencia y resolución que espera la tele, o sea PAL. Me peleé un rato con el portátil, a ver si podia hacer que emitiese en PAL, pero nada de nada… Probé desde 640&#215;480 con 50/60 Hz hasta 1024&#215;768 y nada de nada… 🙁

**En fin…**

Hoy, si puedo, iré a _otra_ tienda que aunque me cae bastante más lejos, entienden un poco de lo que venden, a ver como narices puedo conectar mi portátil sin morir en el intento. Hay gente que comenta que con un cable de VGA a video por componentes funciona… El cable VGA a video por componentes es de aspecto idéntico al cable VGA –> RGB que ya probé: también tiene tres conectores de colores, salvo que en lugar de llamarse R, G y B se llaman Y, Cb y Cr (o Y, Pb, Pr si estamos en analógico). Lo que me “escama” un poco es que el manual de la tele sí que menciona que puede conectarse un cable de video por componentes, pero cuando hablan de un portátil con salida VGA, se _olvidan_ de ese cable y entonces sólo hablan del famoso RGB H/V (el video por componentes lo mencionan cuando hablan de conectar DVDs o videos).

A ver que comentan en la tienda… 🙂

En fin… lo que sí me ha quedado claro de todo esto es que el próximo portátil que me pille, a poco que pueda, va a tener salida DVI-D o HDMI…

Saludos!