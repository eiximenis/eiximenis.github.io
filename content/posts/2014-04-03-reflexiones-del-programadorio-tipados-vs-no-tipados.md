---
title: 'Reflexiones del #programadorIO: Tipados vs no tipados'
description: 'Reflexiones del #programadorIO: Tipados vs no tipados'
author: eiximenis

date: 2014-04-03T23:39:39+00:00
geeks_url: /?p=1663
geeks_visits:
  - 1476
geeks_ms_views:
  - 1614
categories:
  - Uncategorized

---
Esta noche he tenido el placer de participar en el marco de un #programadorIO en un debate sobre los lenguajes tipados vs los no tipados. Puedes ver el debate en youtube: [https://www.youtube.com/watch?v=sxOM6sYgn5U][1]

> **Nota**: En el contexto de este post “no tipado” significa débilmente tipado o de tipado dinámico. Y tipado significa fuertemente tipado o de tipado estático.

Mi opinión es que los lenguajes no tipados son muy adecuados para prototipados, por que las herramientas suelen ser más ágiles y porque te permiten “saltarte” en primera instancia una fase mucho más formal de diseño (fase que luego tarde o temprano tiene que venir, pero en un lenguaje tipado tiene que realizarse al principio para, precisamente, poder diseñar los tipos). De todos modos mi experiencia profesional versa mayoritariamente en los lenguajes tipados (C++, Java y C#). También he mencionado que creo que el auge de JavaScript no es tanto por el lenguaje en sí, si no que viene de la mano del auge del desarrollo web. Si desarrollas para la web, debes hacerlo casi si o si, en JavaScript. Hubiese estado bien la opinión de alguien que hubiese desarrollado tan solo en un lenguaje dinámico que no sea JavaScript (p. ej. Ruby) porque dentro de pequeñas diferencias creo que todos compartíamos mucho en común y que estábamos más del lado de los tipados que de los no tipados.

Bien, aclarado esto, yo he hecho bastante incapié, o lo he intentado al menos, en que a veces un sistema estático de tipos es un “corsé” no deseado y que para ciertas tareas un lenguaje no tipado es mejor o te permite realizarlas de forma mucho más natural o productiva. Y voy a poner algunos ejemplos concretos usando C# (casi todo lo que diré es aplicable a Java también).

**Ejemplo 1: Deserialización de datos dinámicos**

Este es el ejemplo que apuntaba Pedro. En el fondo es muy simple, puesto que si los datos son dinámicos ¿qué mejor que un lenguaje dinámico para deserializarlos?

Imagina que tienes que consumir una api REST que te puede devolver un objeto (da igual el formato, JSON, XML o lo que sea) que tiene 150 campos posibles, todos ellos opcionales. Pueden aparecer o no pueden aparecer. Si tienes que deserializarlo en un lenguaje tipado, que haces: crear una clase con 150 miembros? Y si alguno de los miembros es un int y vale 0… este 0 es porque no ha aparecido o bien porque _realmente_ he llegado un 0. Si claro, puedes usar Nullable<int> pero… bonito y divertido no es.

¿Y si en lugar de ser campos simples son compuestos? ¡Terminas teniendo una jerarquía enorme de clases tan solo para deserializar las respuestas!

El mismo problema te lo encuentras si eres el que crea la API REST por supuesto. Pero incluso peor… porque igual no puedes usar la clase con 150 miembros porque a lo mejor los miembros vacíos o con el valor por defecto se serializarían también y no quieres eso. Vas a terminar igual: con un numero enorme de clases tan solo para serializar los datos.

**Si los datos con los que trabajas tienen una naturaleza dinámica, un lenguaje dinámico es lo mejor para tratarlos.**

Por supuesto podrías trabajar con algo parecido a un Dictionary<string, object> y serializar el diccionario con el formato de datos esperado. Si, pero haciendo esto **estás haciendo un workaround, te estás enfrentando al sistema de tipos**. Estás simulando un tipo dinámico en un lenguaje estático. Todas las ventajas del tipado estático desaparecen (el compilador no te ayudará si te equivocas en el nombre de una clase), las herramientas de refactoring no pueden ayudarte en nada (incluso menos que en el caso de un lenguaje dinámico), tu código queda “sucio” (lleno de dictionarios, llamadas a métodos .Add) y además… tardas más.

**Ejemplo 2: Jerarquías de clases distintas autogeneradas**

Imagina que tienes dos servicios WCF distintos que te devuelven datos muy parecidos. En ambos casos son datos de productos. En ambos casos siempre hay un nombre, un precio y un id. Los nombres de los campos y los tipos SOAP asociados son los mismos (imagina que eso lo puedes definir o imponer).

Si generas los proxies para acceder a los servicios vas a terminar con dos clases diferentes (una por cada servicio). Pero incluso aunque los miembros para el nombre, precio e id se llamasen igual **no podrías intercambiar esos proxies en código**. Vas a tener dos clases iguales (ambas tendrán un string nombre, un decimal precio y un int id) pero para el compilador son dos clases distintas. **Cualquier función que opere sobre uno de los proxies no puede operar con el otro**. A pesar de que el aspecto de ambas clases es “idéntico”, a pesar de que representan el “mismo” concepto, para el compilador tienen la mismo parecido que el de un perro con una manzana.

Sí: el problema principal está en que tienes dos clases distintas para lo mismo, pero eso **ocurre en la vida real cuando hay herramientas que autogeneran código**. ¿Qué soluciones tienes? Pues crear una tercera clase que sea tu “producto” y “transformar” cada uno de los objetos proxy a un objeto de tu clase “producto” que será con la que trabaje tu código. Sí, es posible que los lenguajes tipados tengan un rendimiento superior a los no tipados, pero si empiezas a tener que copiar objetos muchas veces…

¿No estaría bien que tu código que trabaja con un producto pudiese trabajar directamente con cualquiera de los dos proxies? Aunque sean de clases “distintas”. Aunque no haya ninguna interfaz en común. A fin de cuenta tu código tan solo necesita un nombre, un id y un precio. Estaría bien que pudiese funcionar con _cualquier_ objeto que tiene esos tres campos no? Eso se llama _duck typing_ y viene “de serie” con los lenguajes no tipados.

¡Ojo! Que el hecho de que un lenguaje sea tipado no le impide tener algo muy parecido (a efectos prácticos idéntico) al _duck typing:_ P. ej. este problema de los proxies se podría solucionar en C++ con el uso de templates. El uso de templates en C++ es un ejemplo de lo que conoce como _structural typing_ (que es, básicamente, _duck typing_ en tiempo de compilación). Pero no, ni Java ni C# tienen soporte para _structural typing_.

**Ejemplo 3: Generics**

Podría poner muchos ejemplos parecidos al de los proxies, incluso cuando no hay código generado. Un ejemplo rápido. Tengo cuatro clases mías, independientes entre ellas. 

Ahora quiero crear una colección propia, que implemente IEnumerable<T> pero que internamente use un diccionario, ya que continuamente se estarán buscando elementos por nombre. 

Por supuesto las 4 clases tienen una propiedad string Name para guardar el nombre.

Pues bien, para crear esas cuatro colecciones, tienes dos opciones:

  1. Crearte cuatro clases colección idénticas que solo cambian el tipo de datos que aceptan / devuelven. Si, eso suena muy .NET 1.0
  2. Usar generics… **Salvo que no puedes.**

Y no puedes usar generics porque dentro del código de la clase genérica no puedes acceder a la propiedad Name del tipo genérico. Porque el tipo genérico es “object” por defecto. Por supuesto si las 4 clases implementasen una interfaz común, que se llamase INamedItem (p. ej.) y que definiese la propiedad Name, podrías poner una restricción de generics para que el tipo genérico implementase INamedItem y entonces podrías usar generics para crear tu colección propia. **Pero realmente la interfaz INamedItem no representa ningún concepto real. Está tan solo para permitirte usar generics en este caso**. Este es otro caso donde duck typing vendría
  
bien: tu colección propia debería funcionar con cualquier objeto que tenga la propiedad Name. Pero el sistema de tipos de C# (con el de Java pasa lo mismo) es incapaz de dar soporte a esta situación.

**Ejemplo 4: delegados**

Tengo una función que devuelve un bool y acepta un int. Tengo un delegado de tipo Func<int, bool> que “apunta” a dicha función.

Quiero pasar este delegado a otra función… que espera un Predicate<int>. 

**Conceptualmente Func<T, bool> es lo mismo que Predicate<T> pero para el compilador son totalmente distintos**. Por suerte en este caso la solución es muy sencilla, convertir un Func a un Predicate es muy sencillo, pero tienes que hacerlo igualmente.

> **Nota:** Por cierto, aprovechando, no uses nunca Predicate<T> en tu código. Está obsoleto. Func<T,bool> es lo que se debe usar.

**Ejemplo 5: Instanciación de tipos dinámica**

Este es muy sencillo: quieres instanciar un tipo **cuyo nombre no conoces en tiempo de compilación**. Da igual la razón: el método puede venirte de un fichero, BBDD o lo que sea.

Cierto, en C# y en Java puedes usar reflection (p. ej. Activator.CreateInstance en C#) para crear la instancia. El problema es **que usar reflection elimina todas las ventajas del tipado estático y además el código queda muy “sucio”**. Pasar de reflection a tipado estático otra vez no siempre es posible (si sabes que cualquiera de las posibles clases implementa el mismo interfaz puedes convertir el resultado al interfaz y a partir de allí recuperar el tipado estático). Y si tienes que hacer varias cosas usando reflection el código queda “sucio”, dificil de entender y ninguna herramienta de refactorización puede ayudarte.

En un lenguaje no tipado en cambio, la creación del objeto puede requerir una sintaxis distinta pero una vez creado el objeto **invocar los métodos será con la misma sintaxis de siempre**.

**En resumen**

Todos los ejemplos presentados (y hay más de posibles), se resumen en dos grandes tipos: comportamiento dinámico (ejemplos 1 y 5) y “objetos semánticamente compatibles pero incompatibles a la práctica” (el resto de ejemplos).

¿Justifican esos casos usar un lenguaje dinámico para todos tus proyectos? No. Pero si que justifican **que los lenguajes estáticos añadan opciones para facilitar la programación dinámica**. Por ejemplo el dynamic de C# es un paso en esa dirección. Los templates de C++ son otro (los genérics de .NET o de Java ni de lejos).

Entonces… ¿tienes que usar un lenguaje estático para todos tus proyectos? Pues no. **Puedes hacer grandes proyectos tanto en lenguajes tipados como en no tipados**. Y puedes hacer aberraciones en ambos.

Aunque yo personalmente prefiero un lenguaje estático a uno dinámico, **me siento cómodo en estos y a veces cuando estoy en C# si que me gustaría tener toda la flexibilidad que estos me ofrecen.** De hecho, cuanto más me he acostrumbrado a JavaScript más echo en falta ciertas cosas en C#. Pero no siempre, no continuamente. Solo “cuando yo quiero”.

Ahora sí, lo que tengo claro es que **desarrollar bien en un lenguaje no tipado requiere mayor disciplina que en un lenguaje tipado y que cualquier desarrollador que se precie debería conocer los conceptos de orientación a objetos clásicos (**de hecho yo creo que cualquier desarrollador debería aprender C++, pero esa es otra batalla :P).

Bueno… si has llegado hasta aquí… gracias por leer este tostón! Y por supuesto, siéntete libre de dejar un comentario con tu opinión!

Un saludo!

 [1]: https://www.youtube.com/watch?v=sxOM6sYgn5U "https://www.youtube.com/watch?v=sxOM6sYgn5U"