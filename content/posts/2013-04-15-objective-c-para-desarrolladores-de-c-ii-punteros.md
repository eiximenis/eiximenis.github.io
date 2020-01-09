---
title: 'Objective-C para desarrolladores de C# (ii)–Punteros'
description: 'Objective-C para desarrolladores de C# (ii)–Punteros'
author: eiximenis

date: 2013-04-15T17:54:21+00:00
geeks_url: /?p=1635
geeks_visits:
  - 2638
geeks_ms_views:
  - 1161
categories:
  - Uncategorized

---
Aaahhh… los punteros son una de las bestias negras del desarrollo. Desterrados de los dominios de los lenguajes orientados a objetos “modernos” como Java por ser demasiado “próclives a errores” los punteros se han convertido en una especie de ser mitológico, temido por muchos desarrolladores que tiemblan cuando ven un asterisco dando vueltas por ahí… Incluso C# los tiene medio apartados por ahí, rodeados de unsafes por todas partes.

**¿Qué es un puntero?**

Un puntero no es nada más que una variable normal y corriente pero que su valor en lugar de ser un entero, o un carácter, o un número decimal es una dirección de memoria:

<div style="font-size: 10pt; font-family: consolas; background: #1e1e1e; color: #dcdcdc">
  <p style="margin: 0px">
    <span style="color: #569cd6">int</span> <span style="color: #b4b4b4">*</span><span style="color: #c8c8c8">a</span><span style="color: #b4b4b4">;</span>
  </p>
  
  <p style="margin: 0px">
    <span style="color: #569cd6">int</span> <span style="color: #c8c8c8">i</span><span style="color: #b4b4b4">=</span><span style="color: #b5cea8">10</span><span style="color: #b4b4b4">;</span>
  </p>
  
  <p style="margin: 0px">
    <span style="color: #c8c8c8">a</span><span style="color: #b4b4b4">=&</span><span style="color: #c8c8c8">i</span><span style="color: #b4b4b4">;</span>
  </p></p>
</div>

La primera línea declara un puntero a int. No hay un tipo de datos específico para puntero, en su lugar se pone un asterisco que sigue al tipo de datos.

> **Nota:** El tipo de datos de la variable a (el puntero) es int\*. Que en mi código haya un espacio entre el int y el \* y ninguno entre el * y la a es porque el parser es así de majo y me deja hacerlo. Verás otra gente que teclea <font face="Consolas"><em>int* a;</em></font> y es equivalente (y estrictamente hablando más cierto).

La segunda línea es obvia (declara un int) y la tercera línea utiliza el operador de referencia (&) para obtener la dirección de la variable i y asignarla al puntero a.

En este punto el valor de a es la dirección de memoria de la variable i. Si tengo un puntero puedo modificar el contenido de dicho puntero utilizando el operador de dereferencia (*):

<div style="font-size: 10pt; font-family: consolas; background: #1e1e1e; color: #dcdcdc">
  <p style="margin: 0px">
    <span style="color: #b4b4b4">*</span><span style="color: #c8c8c8">a</span><span style="color: #b4b4b4">=</span><span style="color: #b5cea8">11</span><span style="color: #b4b4b4">;</span>
  </p></p>
</div>

En este punto el contenido del puntero a vale 11. O dicho de otra manera: el valor de posición de memoria a la que apuntaba el puntero ha pasado a valer 11. ¿Y qué había en esta posición de memoria? Pues la variable i. Así si ahora imprimo la variable i veré que vale 11.

O de forma más clara, \*a es equivalente a i. Modificar el valor de \*a implica modificar el valor de i (y a la inversa, modificar el valor de i implica modificar el valor de *a).

**¿Por qué son peligrosos los punteros?**

_Per se_, los punteros no son peligrosos. Son simplemente un _alias_ para acceder a una posición de memoria. Pues entonces, ¿qué es lo que los hace peligrosos?

Pues su peligro reside en lo que se llama aritmética de punteros y que permite modificar no el contenido de un puntero si no el propio puntero en sí. Eso significa que un puntero tiene la capacidad de apuntar a direcciones arbitrarias de memoria:

<div style="font-size: 10pt; font-family: consolas; background: #1e1e1e; color: #dcdcdc">
  <p style="margin: 0px">
    <span style="color: #569cd6">int</span> <span style="color: #b4b4b4">*</span><span style="color: #c8c8c8">a</span><span style="color: #b4b4b4">;</span>
  </p>
  
  <p style="margin: 0px">
    <span style="color: #569cd6">int</span> <span style="color: #c8c8c8">i</span><span style="color: #b4b4b4">=</span><span style="color: #b5cea8">10</span><span style="color: #b4b4b4">;</span>
  </p>
  
  <p style="margin: 0px">
    <span style="color: #c8c8c8">a</span><span style="color: #b4b4b4">=&</span><span style="color: #c8c8c8">i</span><span style="color: #b4b4b4">;</span>
  </p>
  
  <p style="margin: 0px">
    <span style="color: #c8c8c8">a</span> <span style="color: #b4b4b4">=</span> <span style="color: #c8c8c8">a</span> <span style="color: #b4b4b4">+</span> <span style="color: #b5cea8">0x300</span><span style="color: #b4b4b4">;</span>
  </p>
  
  <p style="margin: 0px">
    <span style="color: #b4b4b4">*</span><span style="color: #c8c8c8">a</span><span style="color: #b4b4b4">=</span><span style="color: #b5cea8">11</span><span style="color: #b4b4b4">;</span>
  </p></p>
</div>

Fíjate en la penúltima línea: Le estamos sumando 0x300 (768) a la dirección de a. De forma que ahora a apunta 768 _elementos_ más allá de la dirección de memoria de i. Cuando modifico el contenido de a, estoy modificando una dirección de memoria arbitraria y lo más normal es que:

[<img title="image" style="border-left-width: 0px; border-right-width: 0px; background-image: none; border-bottom-width: 0px; padding-top: 0px; padding-left: 0px; display: inline; padding-right: 0px; border-top-width: 0px" border="0" alt="image" src="http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_76CFD0A9.png" width="504" height="221" />][1]

El otro problema típico de los punteros, no es tanto de los punteros en sí, si no del runtime del lenguaje (me da igual si es C/C++ o Objective-C). En runtimes dotados de garbage collector (como el CLR), el propio sistema se encarga de destruir aquellos objetos que _no se usan_. ¿Y como sabe el runtime que nadie usa un objeto? Pues, porque no hay referencias que apunten a él. El hecho de tener un GC nos garantiza de que **las referencias siempre apuntan a un objeto válido** (ya que el GC no destruirá un objeto mientras tenga referencias que lo apunten). Pero, que ocurre si no tenemos GC? Pues que puedo tener dos referencias que apunten a un objeto y alguien puede destruir este objeto. Cuando luego más tarde uso una de esas referencias para acceder al objeto, me puede dar un error ya que en la dirección de memoria apuntada por esta referencia ya no está el objeto, hay otros datos. Este error viene a ser el contrario del _NullReferenceException:_ En lugar de tener un error porque la referencia NO apunta a nada (vale null), tengo un error porque la dirección de memoria a la que apunta la referencia ya no contiene el objeto que contenía porque este ha sido destruído.

He estado usando en todo este párrafo la palabra referencia y no puntero, para poner de manifiesto que podríamos tener este error también si usáramos un lenguaje basado en referencias (y no en punteros) como C# si tuviesemos una gestión manual de memoria. **La principal causa de errores de C/C++ es la gestión manual de memoria, no los punteros en sí**. En los lenguajes que tienen gestión manual de memoria se denomina con el término _dangling pointer_ a un puntero que apunta a una dirección de memoria donde ya NO hay el objeto que había.

No voy a entrar en las diferencias entre punteros y referencias (porque son muy tenues y depende un poco de cada lenguaje). Quédate con que tanto un puntero como una referencia apuntan a una dirección de memoria. Si tienes GC puedes estar seguro de que en esta dirección habrá un objeto válido. Si no tienes GC entonces NO puedes estar seguro de esto y ahí empiezan realmente tus problemas. La única característica que tienen los punteros y no tienen las referencias es la aritmética de punteros.

**¿Cual es la posición de Objective-C?**

Primero, Objective-C es un superconjunto de C (eso significa que todo el código C es código Objective-C válido). C tiene punteros y aritmética de punteros así que Objectiv
  
e-C también.

Lo que diferencia Objective-C de C es la gestión de memoria. En C (y en C++) la gestión es totalmente manual: Cuando quieres crear un objeto debes reservar la memoria una sola vez y luego liberarla una sola vez cuando ya nadie más vaya a usar el objeto.

Òbjective-C usa un contador de referencias para ello: El runtime mantiene un contador que indica cuantos punteros apuntan a un objeto en un momento dado. Cuando dicho contador llega a 0 el runtime destruye el objeto. Hay dos modelos de gestión de memoria dentro del runtime:

  1. Manual: Nosotros somos los encargados de indicarle al runtime cuando queremos que incremente y decremente dicho contador. Este modelo de memoria adolece de los mismos problemas que tiene C/C++ para gestionar la memoria (memory leaks si no decrementamos suficientes veces el contador o _dangling pointers_ si lo decrementamos demasiadas veces). 
  2. Automática (ARC): El encargado de decrementar o incrementar el contador es el **compilador**. La verdad es que ARC es muy sencillo de usar y nos permite desarrollar casi, casi, casi como si tuviesemos un GC. 

> Nota: El runtime de Objective-C soporta también el uso de Garbage Collector. Pero como en iOS no se puede usar, no hablaré del GC de Objective-C. Además su uso está marcado como obsoleto en favor de ARC.

En esta serie de posts veremos tanto la gestión manual (MRC) como la automática (ARC) de memoria.

Desde el punto de vista de un desarrollador de C# debes quedarte con lo siguiente:

  1. Si usas ARC la forma de desarrollar será muy parecida a C#: vamos a crear objetos y nos despreocuparemos de liberar la memoria. ARC lo hará por nosotros. 
  2. A diferencia de C++ que admite el paso por valor de objetos, en Objective-C se admite tan solo paso por referencia a través de punteros. Eso lo hace más parecido a C# (cuando se pasa un objeto no se pasa por valor, si no que se pasa una referencia a dicho objeto). 

Bueno… lo dejamos aquí por el momento. En el siguiente post de la serie, más 😉

Saludos a todos!

 [1]: http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_6B86BC6C.png