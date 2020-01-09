---
title: 'Opinión: De si Silverlight está muerto, agonizante o mejor que nunca'
author: eiximenis

date: 2010-11-03T10:50:00+00:00
geeks_url: /?p=1538
geeks_visits:
  - 11258
geeks_ms_views:
  - 2560
categories:
  - Uncategorized

---
Andamos todos revolucionados estos dias, a raíz de unas declaraciones de [Bob Muglia][1] donde decía &ldquo;[Our strategy with Silverlight has shifted][2]&rdquo;. Eso unido al énfasis que se dio a HTML5 en el keynote del PDC y la no mención en absoluto de nada referente a Silverlight han disparado los rumores.

Así pues... ¿está Silverlight muerto, agonizante o por el contrario está mejor que nunca? Primero un _disclaimer_: este es un post de opinión, todo lo que yo afirmo tajantemente en este post son cosas que _yo_ creo. No tengo ni el conocimiento ni la razón absoluta, y además en MS nunca se sabe, así que como suele decirse: al final el tiempo dará y quitará razones 🙂

La estrategia _inicial_ de Microsoft para posicionar Silverlight fue darle el nicho de aplicaciones web: Con Silverlight tus aplicaciones web tendrán una experiencia de usuario mejor, decían. En estos tiempos, comparar Silverlight con Flash era habitual, aunque Microsoft insistía en que no eran comparables: que Silverlight era para _aplicaciones completas_ y no para pequeños adornos. Cuando hacía tiempo que Flash había asumido que no sustituiría a HTML, parecía ser que Silverlight quería asumir el relevo.

Pero con Silverlight 3 todo cambió: la posibilidad de ejecutar aplicaciones out-of-browser, la interoperabilidad COM y mejoras en el acceso a dispositivos locales, convertían a SL3 en algo distinto. Estaba claro que Silverlight llegaba a sitios donde HTML no sueña con llegar nunca. La batalla que inicialmente se libraba en el navegador ahora se trasladaba también al escritorio, y el rival era Adobe AIR. Habéis visto [Seesmic Desktop 2][3]? Es un excelente cliente de Twitter, para Windows y Mac, es una aplicación de escritorio... y está hecha con Silverlight.

**¿Y donde estamos ahora?**

El otro día en twitter _conversaba_ con [Alejandro Mezcua][4] ([@byteabyte][5]) y Dani Mora ([@LoDani][6]) sobre el futuro de Silverlight. Yo argumentaba que **no creo ni mucho menos que Silverlight** esté muerto. De hecho le veo un gran futuro a Silverlight en aplicaciones de escritorio y en aplicaciones para Windows Phone 7 (bueno, aquí depende de como le vaya a WP7, claro). Pero **no le veo futuro para aplicaciones web**. Quiere decir esto que Silverlight va a desaparecer del browser? No, tiene su nicho en sites específicos donde HTML no llega: p. ej. nada mejor que Silverlight para streaming de video. Sí, sí... HTML5 va a soportar vídeo, pero no soportará DRM ni bitrates variables. Si necesitas esto, necesitarás _algo externo_ que te lo de, y ahí entra Silverlight. Pero para la creación _de aplicaciones web_, no lo veo, y diría que Microsoft tampoco 🙂

¿Y RIA? La clave de RIA es lo que significa la I. Dani me comentaba que para él, la I de RIA era &ldquo;aquel entorno que el administrador de sistemas podía controlar&rdquo;. Entonces _no_ estamos hablando de internet... llamésmole intranet o alguna otra cosa, pero no internet. Si quieres desarrollar una aplicación _para internet_ no hay mejor opción que HTML. Y los tiempos han cambiado mucho: de acuerdo que hace tiempo desarrollar con javascript era poco menos que un infierno, pero ahora tenemos a [jQuery][7] que convierte eso en un juego de niños. Antes Ajax era doloroso y entraba con vaselina: ahora con un par de líneas puedes usar Ajax sin preocuparte de que navegadores se usan ni nada parecido. Y sí: hay [jQuery para dispositivos][8] móviles. Aunque de todos modos la experiencia parece demostrar que en los móviles la gente prefiere aplicaciones nativas antes que webs.

Así que si quieres estar presente en aplicaciones web dentro de poco tiempo, _aprende ya HTML5_. No hagas caso de los que dicen que no está terminado, que no saldrá hasta dentro de no se cuantos años: son verdades a medias. Es cierto que la especificación no está lista, pero muchas partes ya estan terminadas, y ya están siendo implementadas en los navegadores. No ha salido hace poco la noticia de que [IE9 era el navegador más compatible con HTML5][9]? Como podría ser si no hubiese salido nada de HTML5? Dentro de poco ya comenzaremos a ver sites desarrollados en HTML5. El momento de lanzarse no es dentro de un, dos o tres años: es ahora.

Pero seguiremos viendo Silverlight en el navegador? Pues creo que sí: en la gran internet en aquellos sitios _junto_ a HTML, donde se requieran aspectos que HTML5 no va asumir y en intranets (o entornos controlados) también supongo que podrán verse... Aunque en estos entornos, porque obligar al usuario a usar un navegador? Las capacidades out of browser de Silverlight hacen que el usuario tenga la sensación de usar una aplicación nativa, con toda la seguridad del sandbox de Silverlight y la misma facilidad de actualización que si una aplicación web se tratara.

Y fuera del navegador? Pues sin duda. Veremos cada vez más aplicaciones de escritorio desarrolladas en Silverlight, y tener todas sus ventajas: actualizaciones automáticas, seguridad y multiplataforma (Windows, Mac). Y ojalá Microsoft se lance en serio a Linux y haga su implementación de Silverlight (y no lo deje medio abandonado con [Moonlight][10]).

De hecho, para resumir este post, si hay alguien que deba sentirse amenazado por Silverlight no es ni mucho menos HTML5... en todo caso, si hemos de buscar a alguien, se trataría de WPF. Cada versión de Silverlight acorta distancias con _su hermano mayor_ y no es descabellado pensar si algún dia llegarán a _unirse_.

Y a vosotros que os parece?

Un saludo!

 [1]: http://www.microsoft.com/presspass/exec/bobmuglia/
 [2]: http://www.zdnet.com/blog/microsoft/microsoft-our-strategy-with-silverlight-has-shifted/7834
 [3]: http://seesmic.com/seesmic_desktop/sd2/
 [4]: /blogs/amezcua
 [5]: http://twitter.com/byteabyte
 [6]: http://twitter.com/lodani
 [7]: http://jquery.com/
 [8]: http://jquerymobile.com/
 [9]: http://test.w3.org/html/tests/reporting/report.htm
 [10]: http://www.mono-project.com/Moonlight