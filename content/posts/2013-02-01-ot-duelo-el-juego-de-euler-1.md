---
title: '[OT] Duelo: El juego de Euler (1)'
description: '[OT] Duelo: El juego de Euler (1)'
author: eiximenis

date: 2013-02-01T13:35:00+00:00
geeks_url: /?p=1630
geeks_visits:
  - 1211
geeks_ms_views:
  - 1046
categories:
  - Uncategorized

---
Bueno… Está por ahí <a href="https://twitter.com/quiqu3" target="_blank" rel="noopener noreferrer">Quique</a>, que como se aburre se ha decidido a picarme un poco (pobre mortal ^_^).

Se ve que junto con <a href="https://twitter.com/acasquete" target="_blank" rel="noopener noreferrer">Álex</a> se han embarcado en resolver los problemas de Euler en distintos lenguajes… Álex se ha quedado con F# (el patito desconocido de .NET), mientras que Quique se ha armado con todo el poder de (mi amado) C#.

Dado que <a href="https://twitter.com/tonirecio" target="_blank" rel="noopener noreferrer">Toni Recio</a> se ha sumado y ha pillado Javascript (quien lo ha visto y quien lo ve) yo he decidido participar, pero esta vez armado con todo el potencial de un lenguaje de programación funcional de verdad. Y el elegido ha sido Scala.

Cada post que haga con la solución aprovecharé para intentar comentar algo de Scala. Ya, Scala no es .NET, pero lo pongo en este blog porque no tengo otro y me da muuuucha pereza abrirlo. Así que… :p

Bueno, al tajo. El primer reto es sencillo (si todos son así, vamos bien): Sumar todos los números del 1 al 1000 que sean divisibles entre 3 ó entre 5. Pues al tajo:

<div style="font-size: 10pt; font-family: consolas; background: #1e1e1e; color: #dcdcdc">
  <p style="margin: 0px">
    <span style="color: white">package</span> <span style="color: white">duel</span><span style="color: #b4b4b4">.</span><span style="color: white">Euler</span>
  </p>
  
  <p style="margin: 0px">
    <span style="color: #569cd6">object</span> <span style="color: white">Euler1</span> <span style="color: white">extends</span> <span style="color: white">App</span> {
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160; <span style="color: white">println</span>(
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; (<span style="color: #b5cea8">1</span> <span style="color: #ff3333">until</span> <span style="color: #b5cea8">1000</span>)<span style="color: #b4b4b4">.</span><span style="color: white">filter</span>(<span style="color: #ff3333">n</span><span style="color: #b4b4b4">=></span> (<span style="color: #ff3333">n</span> <span style="color: #b4b4b4">%</span> <span style="color: #b5cea8">3</span> <span style="color: #b4b4b4">==</span> <span style="color: #b5cea8"></span>) <span style="color: #b4b4b4">||</span> (<span style="color: #ff3333">n</span> <span style="color: #b4b4b4">%</span> <span style="color: #b5cea8">5</span> <span style="color: #b4b4b4">==</span><span style="color: #b5cea8"></span> ))<span style="color: #b4b4b4">.</span><span style="color: white">foldLeft</span>(<span style="color: #b5cea8"></span>)(<span style="color: #ff3333">_</span><span style="color: #b4b4b4">+</span><span style="color: #ff3333">_</span>)
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160; );
  </p>
  
  <p style="margin: 0px">
    }
  </p></p>
</div>

¡Listos!

Si conocéis LINQ no hay mucho que añadir:

  1. La construcción (1 until 1000) es equivalente a Enumerable.Range
  2. filter es nuestro amado Where
  3. foldLeft equivale al Aggregate de LINQ. Usado con la sintaxis (\_+\_) equivale al Sum() de LINQ.

**Y que narices es esto de foldLeft**

Pues bien, foldLeft es una función existente en todas las colecciones de Scala que básicamente permite ejecutar una función que acepte dos parámetros (que serán sacados iterando por la colección). Además foldLeft proporciona un valor inicial.

La sintaxis básica de foldLeft es:

_coleccion.foldLeft(valor\_inicial) función\_a_ejecutar_

En nuestro caso está claro:

  1. La colección son los números de 1 a 1000 que son divisibles entre 3 ó 5
  2. El valor inicial es 0
  3. La función a ejecutar es (\_+\_). Esta es una sintaxis especial de Scala que equivale a una función anónima que suma los dos operandos que se le pasan.

De hecho un código totalmente equivalente hubiese sido usar _foldLeft(0)((a,b) => a+b)_. En este caso estamos pasando la función a usar (sumar los dos operandos) de forma más explícita.

Listos para el siguiente reto 😉