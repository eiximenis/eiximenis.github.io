---
title: El operador ++ postfijo y el sentido común…

author: eiximenis

date: 2009-07-02T18:08:00+00:00
geeks_url: /?p=1461
geeks_visits:
  - 2108
geeks_ms_views:
  - 1531
categories:
  - Uncategorized

---
Post cortito… 😉

El otro día estaba revisando código y vi algo parecido a lo siguiente (C#):

```cs
int x = 1;
iny y = x++;
```

<!--more-->

Si os pregunto los valores de x e y al final de este código… ¿cual seria vuestra respuesta? No lo preguntéis a Visual Studio… pensadlo.

¿Ya lo teneis?

Exacto: x vale 2 (es de esperar puesto que su valor se incrementa con ++), mientras que por su parte y vale 1. Es decir, el valor de x se incrementa _después_ de la asignación.

Quizá os parezca ilógico, pero está así especificado explícitamente en la especificación de C#:

The `++` and `--` operators also support prefix notation, as described in [Section 7.6.5][2]. **The result of `x++` or `x--` is the value of `x` _before_ the operation**, whereas the result of `++x` or `--x` is the value of `x` _after_ the operation. In either case, `x` itself has the same value after the operation.

Sacado de: [http://msdn.microsoft.com/en-us/library/aa691363(VS.71).aspx][3]

Ya veis con que cosillas nos divertimos, eh??? 😉

Saludos

PD: De la misma manera (y por la misma razón) en el siguiente código:

<pre class="code"><span style="color: blue">int </span>x =1;
x=x++;</pre>

`x` tiene el valor 1 al finalizar.

En C# este código es correcto (otra cosa es que sea útil), a diferencia de p.ej. C++ donde este código es _incorrecto_: compila pero su resultado **no está definido** por el estándard… en algunos compiladores x puede terminar valiendo 1 y en otros compiladores puede terminar valiendo 2 (el caso de VC++ 9 en debug).

 [1]: http://11011.net/software/vspaste
 [2]: http://msdn.microsoft.com/en-us/library/aa691369(VS.71).aspx
 [3]: http://msdn.microsoft.com/en-us/library/aa691363(VS.71).aspx "http://msdn.microsoft.com/en-us/library/aa691363(VS.71).aspx"