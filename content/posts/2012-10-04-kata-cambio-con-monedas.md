---
title: '[Kata]- Cambio con monedas'
author: eiximenis

date: 2012-10-04T17:56:10+00:00
geeks_url: /?p=1612
geeks_visits:
  - 1190
geeks_ms_views:
  - 970
categories:
  - Uncategorized

---
Muy buenas! Puro _divertimento_ 🙂

Os propongo un kata por si quereis entrenar las neuronas. Por supuesto no es un kata original mío (soy muy malo para eso), de hecho es un kata creo que bastante famosillo, pero es uno con el que me he enfrentado no hace mucho…

El enunciado es muy simple… Crear una función que reciba dos parámetros: Un entero que represente una cantidad de dinero, y una colección que represente los tipos de monedas de los que disponemos. Dicha función debe devolver _de cuantas maneras diferentes podemos repartir las mondedas para conseguir el valor total de dinero (asumiendo que tenemos infinitas monedas de cada tipo)_.

P.ej. si el dinero es 4 y los tipos de monedas son 1 y 2, la función debe devolver “3”, ya que hay 3&#160; maneras de sumar 4 usando 1 y 2:

  * 1+1+1+1 
  * 2+2 
  * 1+1+2 

Si los tipos de monedas son 5,10,20,50,100,200 y 500 y la cantidad de dinero es 300 el valor devuelto debe ser 1022. Con las mismas monedas y dinero 500 el resultado asciende a 6149. Y finalmente con las mismas monedas y la cantidad de dinero 301 el valor es 0, ya que no hay combinación alguna.

El kata es sencillo, simplemente rellenar la función:

<div style="font-size: 10pt; font-family: consolas; background: white; color: black">
  <p style="margin: 0px">
    <span style="color: blue">int</span> TiposCambio(<span style="color: blue">int</span> money, <span style="color: #2b91af">IEnumerable</span><<span style="color: blue">int</span>> coins)
  </p>
  
  <p style="margin: 0px">
    {
  </p>
  
  <p style="margin: 0px">
    }
  </p></p>
</div>

Como “pista” diré que usando C# 3.0 puede conseguirse una respuesta de 3 líneas de código y que no requiere ninguna variable local.

Podéis ir proponiendo soluciones, sugerencias, pistas… en los comentarios! 😉

Saludos!

PD: Si ya conoces la respuesta (como digo es un kata muy conocido) deja que los demás la busquen (aunque nada te impide dar sútiles pistas en los comentarios :p)

PD 2: No, no hay premio alguno… tan solo el placer de haber solucionado el acertijo… os parece poco? 😛 😛 😛