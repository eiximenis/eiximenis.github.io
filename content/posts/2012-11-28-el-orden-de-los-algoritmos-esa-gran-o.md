---
title: El orden de los algoritmos… esa gran O.
description: El orden de los algoritmos… esa gran O.
author: eiximenis

date: 2012-11-28T17:30:58+00:00
geeks_url: /?p=1619
geeks_visits:
  - 3916
geeks_ms_views:
  - 3870
categories:
  - Uncategorized

---
¡Muy buenas! Este va a ser un post teórico que nada tiene que ver con el desarrollo concreto en .NET. Lo que vamos a decir aquí es aplicable a cualquier algoritmo, esté implementado en .NET, Java, Scala, Javascript, Dart, Smalltalk o cualquier otro lenguaje existente. No, no voy a hablar de buenas prácticas, ni nada parecido. Voy a hablar de un concepto que se usa en la ingenería del softwara para _clasificar_ algoritmos en función de bueno… eso lo veremos con más detalle.

> **Disclaimer**: Este post pretende ser introductorio, así que voy a realizar ciertas simplificaciones. La idea es que se entienda el concepto y lo que significa… 🙂

A dicho concepto lo conocemos como _orden_ (realmente orden de complejidad) del algoritmo, y la notación empleada se conoce como _Big-O Notation_ (aahh… que bien que suenan las cosas en inglés :P) debido a que se emplea la letra O (mayúscula) como símbolo.

De hecho seguro que más de uno ha visto, cosas como p.ej. que un algoritmo es O(n) mientras que otro es O(n ^2). Pero… exactamente ¿qué significa eso?

Se puede argumentar que el orden de un algoritmo nos define _cuan rápido es_. Así un algoritmo con O(n) será más rápido que otro con O(n^2), por la sencilla razon de que n^2 >= n.

Pero esa visión **no es cierta**. El orden de un algoritmo _no mide su rapidez,_ aunque nos permite ordenar los algoritmos del más eficiente al menos eficiente. Alguna vez en literatura informática he visto cosas como “ese algoritmo es de O(2n)”. Bien, esta sentencia es totalmente incorrecta: **No existe O(2n)**. O dicho de otro modo O(2n) = O(n)

Quizá esto te sorprenda, pero es lo que hay 😉 Si el orden pretendiese medir cuan rápido es un algoritmo entonces si que tendría sentido algo como O(2n) que indicaría que un algoritmo determinado es el doble de lento que otro con O(n).

Pero, como digo, el orden **no pretende** medir cuan rápido es un algoritmo respecto a otro. El orden mide otra cosa. Mide _cuan rápidamente aumenta el tiempo de ejecución de un algoritmo cuando aumenten los datos de entrada_. Tomemos como ejemplo un algoritmo cualquiera, p.ej. uno para ordenar listas.

Si tiene O(n) significa que el tiempo aumenta **linealmente** al aumentar los datos de entrada. Es decir, que si para una lista de 100 elementos el algoritmo tarda x segundos, para una lista de 1000 elementos (10 veces más grande) tardará 10 veces más. Es, en definitiva, lo que nos dice el sentido común: si te doblo el tamaño de entrada, debes tardar el doble. Lógico, ¿no? De hecho, el orden lineal es una característica deseable de cualquier algoritmo.

Pero ¡ojo! por supuesto, podemos tener dos algoritmos distintos para ordenar listas, ambos de O(n) y uno mucho más rápido que otro. Pero ambos aumentarán linealmente su tiempo de ejecución al aumentar el tamaño de los datos de entrada.

Por otro lado si el algoritmo tiene O(n^2) significa que el tiempo aumenta **cuadráticamente**. No quieras tener algoritmos como esos: en este caso si para ordenar una lista de 100 elementos el algoritmo tarda x segundos, para una lista de 1000 elementos ¡tardara 100 veces más! Es decir, hemos aumentado los datos de entrada en un ratio de 10 y el algoritmo **no** tarda 10 veces más si no 100 (10^2). Pero bueno… si debes lidiar con uno de esos, consuelate pensando que, aunque no lo parezca, los hay de peores, como los que tienen orden polinomial (O(n^a) donde a es mayor que 2), los todavía peores de orden exponencial (O(a^n) donde a es mayor que 1) y quizá los peores de todos, totamente intratables, los de orden factorial (O(n!)).

Pero existen unos algoritmos que desafían al sentido común y son una maravilla: en estos el tiempo aumenta **logarítmicamente**. Es decir, si para ordenar una lista de 100 elementos el algoritmo tarda x segundos, para ordenar una lista 10 veces más larga tardará… ¡tan solo el doble! ¡Una maravilla! Esos son los que tienen un orden O(log n) y los que, por supuesto, se intentan encontrar en cualquier caso.

Por norma general, siempre que sea posible debemos usar un algoritmo del menor orden posible. Es decir si tenemos dos algoritmos que hacen lo mismo y uno es de O(n) y el otro es O(n^2) mejor usamos el primero, ya que nos protege mejor contra volúmenes de datos grandes, aunque todo depende… porque si sabemos que nuestros datos nunca van a crecer por encima de un determinado umbral, quizá nos interese usar uno de orden mayor pero que sea más rápido (aunque el orden nos dice que se volvería más lento a partir de cierto tamaño de los datos de entrada).

Por último mencionar que el orden se suele aplicar al tiempo de ejecución, pero puede aplicarse a otros aspectos (p.ej. la memoria que consume un determinado algoritmo).

¡Un saludo!