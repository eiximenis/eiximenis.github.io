---
title: 'Curiosidad: Esos nombres de contenedores en Docker…'
author: eiximenis

date: 2019-03-05T15:51:11+00:00
geeks_url: /?p=2333
geeks_ms_views:
  - 639
categories:
  - docker

---
Una cosa que causa cierta confusión en la gente que empieza con Docker es **el nombre de los contenedores. **La verdad es que cuando ejecutamos un contenedor usando _docker run_ este tiene un nombre aleatorio.
  
<!--more-->


  
Es cierto que _docker run_ tiene el parámetro _&#8211;name_ para indicar el nombre del contenedor, pero la verdad no es algo que mucha gente haga. Si no especificas tú el nombre, Docker le asigna este nombre &#8220;aleatorio&#8221; que comentaba:
  
[<img class="alignnone size-large wp-image-2331" src="https://geeks.ms/etomas/wp-content/uploads/sites/154/2019/03/docker-ps-1024x100.png" alt="La salida de &quot;docker ps&quot; que muestra 4 contenedores y sus nombres: angry_curie, pensive_gates, thirsty_sanderson, y focused_edison" width="660" height="64" />][1]
  
Puedes ver como tengo 4 contenedores que se llman &#8220;angry\_curie&#8221;, &#8220;pensive\_gates&#8221;, &#8220;thirsty\_sanderson&#8221; y &#8220;focused\_edison&#8221;. Si te fijas hay siempre un patrón: un adjetivo y un nombre (apellido de hecho).
  
Lo que quizá no sepas es **que los apellidos elegidos lo son de personas relevantes en el ámbito de la ciencia y el colectivo _hacker_.** Así en mi caso _curie_ viene, obviamente por [Maire Curie][2], _gates_ viene por (quien si no) [Bill Gates][3], _sanderson_ viene por la matemática [Mildred Sanderson][4] y _edison_ viene pues de... [Edison][5] xD
  
Si quieres ver todos los nombres qué hay (y por quien se han incluído) puedes [echarle un vistazo al generador de nombres de Docker][6]. Es una **curiosa manera de rendir homenaje** a personajes relevantes.
  
Y para terminar, una Pull Request curiosa: La [PR 159][7] se carga a [Clifford Christopher Cocks][8] de la lista de nombres porque en su caso, su apellido _cocks_ ha pesado más que sus logros matemáticos... y es que bueno, estar haciendo una demo de Docker y que aparezca un contenedor llamado _hardcore_cocks_ quizá no quedaba del todo bien 😀
  
Un saludo!

 [1]: https://geeks.ms/etomas/wp-content/uploads/sites/154/2019/03/docker-ps.png
 [2]: https://es.wikipedia.org/wiki/Marie_Curie
 [3]: https://es.wikipedia.org/wiki/Bill_Gates
 [4]: https://en.wikipedia.org/wiki/Mildred_Sanderson
 [5]: https://es.wikipedia.org/wiki/Thomas_Alva_Edison
 [6]: https://github.com/docker/engine/blob/master/pkg/namesgenerator/names-generator.go
 [7]: https://github.com/docker/engine/pull/159
 [8]: https://en.wikipedia.org/wiki/Clifford_Cocks