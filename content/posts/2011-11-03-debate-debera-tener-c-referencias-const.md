---
title: '[Debate] ¿Debería tener C# referencias const?'
author: eiximenis

date: 2011-11-03T12:46:29+00:00
geeks_url: /?p=1582
geeks_visits:
  - 1690
geeks_ms_views:
  - 1059
categories:
  - Uncategorized

---
Muy buenas! Para ser sinceros esta es una pregunta que me he hecho siempre y, creo yo, que se han hecho muchas personas que vienen de C++. ¿Debería tener C# referencias const? El hecho es que hasta ayer no había encontrado una explicación _razonada_ y de alguien de peso (quien mejor que [Eric Lippert][1], cuyo blog es lectura obligada) del porque C# no las incluye. Al final del post hay el enlace al post de stackoverflow en el que Eric explica _sus_ razones por las que C# no tiene referencias const.

En este post voy a intentar explicar que son las referencias const (en C++) porque a Eric Lippert no le convencen (ojo que no queda claro con una sola lectura de lo que él dice, tuve que leérmelo un par o tres de veces junto con los comentarios, porque Eric es de los que cuando dice algo _cualquier_ palabra cuenta), además de algunas opciones que hay actualmente en C# para simularlas.

El objetivo del post es captar vuestra opinión: es decir, creéis que estarían bien? O que sobran totalmente? O que tal y como están en C++ no, pero de otra forma podrían ser interesantes?

Dado que voy a exponer varias opiniones al respecto este post será largo… Pero espero que os resulte de interés 😉

**Referencias const en C++**

Antes que nada aclaremos a que nos referimos por referencias const. Porque hay dos posibilidades:

  1. La _referencia es constante_, es decir, una vez inicializada su valor NO puede ser modificado (no puede apuntar a _otro_ objeto). En C# eso sería una variable _readonly_. 
  2. La _referencia NO es constante pero a través de esa referencia NO puede modificarse el objeto apuntado_. 

En C++ ambas posibilidades existen, así que NO es lo mismo:

<div style="font-family: courier new; background: white; color: black; font-size: 10pt">
  <p style="margin: 0px">
    Foo& <span style="color: blue">const</span> foo
  </p></p>
</div>

que

<div style="font-family: courier new; background: white; color: black; font-size: 10pt">
  <p style="margin: 0px">
    <span style="color: blue">const</span> Foo& foo
  </p></p>
</div>

En el primer caso tenemos una referencia que es constante (básicamente lo que en C# conocemos como readonly, con la salvedad que pueden ser inicializadas en cualquier momento), mientras que en el segundo tenemos una referencia a través de la cual **no podemos modificar** el objeto. **A esas referencias son a las que nos referimos con el nombre de “Referencias const”.**

En C++ una clase debe declarar que métodos NO modifican los datos de la clase:

<div style="font-family: courier new; background: white; color: black; font-size: 10pt">
  <p style="margin: 0px">
    <span style="color: blue">class</span> Foo
  </p>
  
  <p style="margin: 0px">
    {
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: blue">int</span> value;
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: blue">public</span>:
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: blue">int</span> GetValue (<span style="color: blue">void</span>) <span style="color: blue">const</span>;
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: blue">void</span> SetValue(<span style="color: blue">int</span> nv);
  </p>
  
  <p style="margin: 0px">
    };
  </p></p>
</div>

La clase Foo tiene dos métodos, GetValue y SetValue. Y el método GetValue está declarado como “const” para indicar que **no** modifica ningún miembro de la clase.

Así, el siguiente código **NO** compilará:

<div style="font-family: courier new; background: white; color: black; font-size: 10pt">
  <p style="margin: 0px">
    <span style="color: blue">void</span> FooConst1 (<span style="color: blue">const</span> Foo& foo)
  </p>
  
  <p style="margin: 0px">
    {
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; foo.SetValue(10);
  </p>
  
  <p style="margin: 0px">
    }
  </p></p>
</div>

El compilador nos avisará que estamos intentando modificar un objeto Foo a través de una referencia const (el error real es un mensaje raro de que no puede convertir _this_ pero quiere decir eso :p).

**Usar una referencia const convierte el objeto en inmutable?** No. Usar una referencia const evita que pueda ser modificado dicho objeto _a través de esa referencia_. Única y exclusivamente. Mira ese código y piensa cual es el valor de _i_ después de ejecutar la función _Bar_:

<div style="font-family: courier new; background: white; color: black; font-size: 10pt">
  <p style="margin: 0px">
    <span style="color: blue">void</span> Bar(<span style="color: blue">const</span> <span style="color: blue">int</span>& v1, <span style="color: blue">int</span>& v2)
  </p>
  
  <p style="margin: 0px">
    {
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; v2 = v1+1;
  </p>
  
  <p style="margin: 0px">
    }
  </p>
  
  <p style="margin: 0px">
    <span style="color: blue">int</span> _tmain(<span style="color: blue">int</span> argc, _TCHAR* argv[])
  </p>
  
  <p style="margin: 0px">
    {
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: blue">int</span> i=10;
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; Bar(i, i);
  </p>
  
  <p style="margin: 0px">
    }
  </p></p>
</div>

El valor de _i_ después de ejecutar Bar es de 11. Porque el valor de _i_ se _puede modificar a través de v2 que es una referencia tradicional (_por supuesto intentar v1=v2+1 dentro de _Bar_ sí que da error).

**Ventajas de tener referencias const en el lenguaje**

Como desarrollador el tener referencias const te permite:

  1. Tener la seguridad de que un método _no va a modificar_ nada de tu clase. Si un método espera una referencia _const_ sabes que el método no va a poder modificar el objeto que reciba. 
  2. Devolver desde un método un objeto que no va a poder ser modificado por quien ha llamado el método. P.ej. en lugar de devolver una _ReadOnlyCollection<T>_ (una clase que es una auténtica aberración desde el punto de vista de OOP) se podría devolver _const List<T>_. 

**La razón (de Eric Lippert) por la cual no existen en C#**

En resumidas cuentas porque tal y como están implementadas en C++ _no sirven_. Él tiene dos objeciones al respecto, una muy pragmática y la otra mucho más profunda y filosófica.

  1. El casting, bien al estilo C, bien const_cast de C++, elimina la seguridad que podrían ofrecer las referencias const. Dicho de otro modo, la posibilidad de obtener una referencia _tradicional_ a partir de una referencia const, convierten a éstas totalmente en inútiles (en lo que refiere a _asegurar que a través de dicha referencia nadie podrá modificar el objeto)._ Imaginad que yo devuelvo una _const List<T>_ con datos de mi objeto. Si quien obtiene la referencia _puede_ a partir de esa referencia const, obtener una referencia tradicional (y por lo tanto añadir o eliminar elementos de la lista), yo pierdo la seguridad que tenía al devolver la referencia _const_. No puedo asumir que los contenidos del objeto no van a ser modificados. A eso se refiere Eric cuando dice que _const is broken._ 
  2. La segunda razón es mucho más fuerte… Para Eric una referencia const sería realmente útil si **convirtiese el objeto apuntado en inmutable**. Lo deja muy claro con el siguiente comentario: <font color="#0000ff">If I have a reliably constant queue then I should be able to say "if (!q.Empty()) { M(); x = q.First(); }" and <i>regardless of what M() does, the queue is still empty when it returns</i>. The same way that if I say "const int y = 123; ... if (y > 0) { M(); " and after M() returns, y is still greater than zero <i>because it is constant</i>.</font> 

Es
  
decir, del mismo modo que const int i=10; significa que la variable i **nunca podrá ser modificada** bajo ningún concepto, del mismo modo _const Queue&#160; q_ debería significar que el objeto q es inmutable. No puede modificarse. Eso plantea problemas muy serios en cuanto a como inicializamos estos objetos, o como _convertimos_ un objeto mutable en inmutable, etc, etc. Pero quedaros con la clave: el objeto pasa a ser inmutable. No hay manera de echar esto atrás. No es una referencia const a un objeto, es una referencia a un objeto constante. Esa segunda razón es muy fuerte y va mucho más allá de lo que una referencia _const_ de C++ pretende (que el objeto no sea modificable a través de esa referencia).

Olvidemos pues esa segunda razón, a pesar de que para Eric tiene un peso fundamental, y volvamos a la primera: la posibilidad de hacer casting y de obtener una referencia normal a partir de una referencia const. La solución parece rápida y trivial: se prohíben esos castings. Y punto.

Y como pasa casi siempre, nada es tan trivial:

  1. Que hacemos con _reflection?_ Si alguien usa _reflection_ teoricamente podría llamara a un método que modificase el objeto apuntado por la referencia (aquí sale a colación de nuevo el segundo argumento de Eric: dado que el objeto no es inmutable, sino que es la referencia la que está marcada como _const_). Es eso importante? Bueno, depende… actualmente con _reflection_ podemos _llamar a métodos privados_ de una clase, lo que (sin negar su utilidad) puede llegar a representar una violación todavía mayor. 
  2. Que hacemos con _object_? O dicho de otro modo… una referencia const Foo puede ser pasada a un método que acepte Object? Seguramente sí, responderéis todos, ya que los Foo son Objects con independencia de que sean mutables o no. Pero ahora imaginad una jerarquía de clases: 

<div style="font-family: courier new; background: white; color: black; font-size: 10pt">
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: blue">class</span> <span style="color: #2b91af">Foo</span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; {
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: blue">public</span> <span style="color: blue">int</span> PropFoo { <span style="color: blue">get</span>; <span style="color: blue">set</span>; }
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; }
  </p>
  
  <p style="margin: 0px">
    &#160;
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: blue">class</span> <span style="color: #2b91af">Bar</span> : <span style="color: #2b91af">Foo</span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; {
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: blue">public</span> <span style="color: blue">string</span> PropBar { <span style="color: blue">get</span>; <span style="color: blue">set</span>; }
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; }
  </p></p>
</div>

Si un método recibe un objeto Foo, le puedo pasar un _const Bar_? Los objetos Bar son todos ellos objetos Foo, pero los _const Bar_ lo son? Tiene sentido que lo sean? Tiene sentido que no lo sean? O los objetos _const Bar_ sólo pueden ser _const Foo?_ Dicho de otro modo: **tenemos UNA sola jerarquía de objetos que empieza por Object, o tenemos DOS (una que empieza por Object y la otra por const Object)?** Si admitimos un sola jerarquía, entonces estaremos de acuerdo en que yo puedo tener un _const Bar_ y ese ser modificado (al menos la propiedad PropFoo) a través de un método que espere un _Foo_. 

Por supuesto hay versiones mixtas, como tener una sola jerarquía, pero sólo admitir llamadas a métodos que no muten el objeto. De esa manera una referencia _const Bar_ podría ser pasada a un método _Foo_ y eso sólo compilaría si el método no hace nada que pueda mutar el objeto (p.ej. estaría bien llamar al getter de la propiedad pero no al setter). Pero eso complica mucho (pero mucho, eh?) el tema y es que de hecho con esta visión estamos volviendo al segundo punto que comentaba Eric (no es la referencia lo que está marcada como _const_, es el objeto que está marcado como inmutable).&#160; Y de todos modos para mi esa opción es inviable el hecho de que no hay nada que diga _a priori_ a quien llama el método si esa llamada es correcta o no: que el código compilase dependería de lo que hiciera internamente un método X (que puede estar en _otro assembly ya compilado_) y no de sus parámetros, ni nada que yo pueda ver _externamente_. Lo único que podría hacer es intentar pasar mi referencia _const Bar_ a un método que acepta un Foo y… ver si compila! Surrealista.

**¿Son necesarias las referencias const tal y como están en C++?**

Dicho de forma rápida y corta: No. Eso no significa que no sean útiles. Simplemente que hay otros métodos para hacer lo mismo. ¿Y en que consisten esos métodos? Pues básicamente en declararse una versión _solo lectura_ de mi clase. La mejor manera de hacer esto es a través de una interfaz:

<div style="font-family: courier new; background: white; color: black; font-size: 10pt">
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: blue">interface</span> <span style="color: #2b91af">IReadOnlyFoo</span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; {
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: blue">int</span> PropFoo { <span style="color: blue">get</span>; }
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; }
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: blue">class</span> <span style="color: #2b91af">Foo</span> : <span style="color: #2b91af">IReadOnlyFoo</span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; {
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: blue">public</span> <span style="color: blue">int</span> PropFoo { <span style="color: blue">get</span>; <span style="color: blue">set</span>; }
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; }
  </p></p>
</div>

Si quiero devolver una referencia a un objeto _Foo_ que no pueda modificarse devuelvo un objeto _Foo_ a través de una referencia de tipo _IReadOnlyFoo_. Y listos! Si un método quiere aceptar como parámetro un objeto de tipo _Foo_ pero NO quiere modificarlo puede aceptar un parámetro de tipo _IReadOnlyFoo_. Así el equivalente de _const Foo_ es _IReadOnlyFoo_.

Por lo tanto la misma semántica que obtenemos a través de las referencias const, las obtenemos con este mecanismo. Cierto: es más tedioso y nos obliga a hacerlo por cada clase de la cual querramos tener una versión de sólo lectura.

**Métodos puros**

Una de las ventajas de la implementación de referencias const en C++ es que obliga al creador de la clase a _declarar que métodos son susceptibles de ser llamados a través de una referencia const_. A estos métodos se los conoce en C++ como métodos const. El compilador **comprueba** efectivamente que un método declarado como _const_ lo sea:

<div style="font-family: courier new; background: white; color: black; font-size: 10pt">
  <p style="margin: 0px">
    <span style="color: blue">class</span> Foo
  </p>
  
  <p style="margin: 0px">
    {
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: blue">int</span> value;
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: blue">public</span>:
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: blue">int</span> GetValue (<span style="color: blue">void</span>) <span style="color: blue">const</span>;
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: blue">void</span> SetValue(<span style="color: blue">int</span> nv);
  </p>
  
  <p st
yle="margin: 0px">
    };
  </p>
  
  <p style="margin: 0px">
    &#160;
  </p>
  
  <p style="margin: 0px">
    <span style="color: blue">int</span> Foo::GetValue(<span style="color: blue">void</span>) <span style="color: blue">const</span>
  </p>
  
  <p style="margin: 0px">
    {
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; value&#8211;;
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: blue">return</span> value;
  </p>
  
  <p style="margin: 0px">
    }
  </p></p>
</div>

Este código no compila, porque el método GetValue intenta modificar value a pesar de ser declarado como método const.

Debajo de los métodos const subyace un concepto realmente potente e interesante: los [métodos puros][2]. Dicho rápidamente, un método puro (no confundir con un [método virtual puro][3] de C++ que no tiene nada que ver) es aquel que siempre devuelve el mismo valor si se le pasan los mismos parámetros y además durante su ejecución no se observa ningún efecto colateral (en el caso de una clase significa que **no modifica el estado** de dicha clase). Ojo, que los métodos const y los métodos puros NO son exactamente lo mismo:

  1. Un método const puede ser impuro si devuelve datos aleatorios o dependientes de algo externo no controlable (p.ej. un fichero). Así p.ej. la propiedad Now de DateTime no es pura porque su valor no es predecible (cada día cambia!).
  2. Un método const NO puede modificar ningún miembro de la clase. Pero NO todos los miembros de la clase conforman su estado. Puede haber miembros privados que sean susceptibles de ser modificados pero que no formen parte del estado de la clase. Un método puro podría modificar esos miembros y un método const no (aunque este segundo punto puede solventarse con el uso de [mutable][4] en C++). 

¿Y porque son interesantes los métodos puros? Bueno… tenerlos identificados **permite ciertas optimizaciones** (se puede _[memoizar][5]_ (así sin erre) la llamada) y también **habilita estos métodos para ser usados en comprobaciones de precondiciones, postcondiciones e invariantes.** Por definición la comprobación de precondiciones, postcondiciones e invariantes deben ser métodos puros (dado que deben ser predecibles y no modificar el estado del objeto). Sin métodos puros el diseño por contratos no es posible (de forma segura y validada por el compilador)…

¿Hola? Si todavía estás aquí… muchas gracias por llegar hasta el final del post. Recuérdame que te pague una cervecita cuando nos veamos por ahí 😉

He creado una encuesta para que podáis expresar vuestra opinión al respecto de todo lo comentado y por supuesto, tenéis los comentarios para explayaros a gusto! 😉

Enlace a la encuesta: [http://www.easypolls.net/poll.html?p=4eb26acb011eb0e44d6335ab][6]

Enlace al post de stackoverflow: [http://stackoverflow.com/questions/3263001/why-const-parameters-are-not-allowed-in-c-sharp][7]

Un saludo! 🙂

 [1]: http://blogs.msdn.com/b/ericlippert/
 [2]: http://en.wikipedia.org/wiki/Pure_function
 [3]: http://en.wikipedia.org/wiki/Virtual_function#Abstract_classes_and_pure_virtual_functions
 [4]: http://msdn.microsoft.com/en-us/library/4h2h0ktk(v=VS.100).aspx
 [5]: http://en.wikipedia.org/wiki/Memoization
 [6]: http://www.easypolls.net/poll.html?p=4eb26acb011eb0e44d6335ab "http://www.easypolls.net/poll.html?p=4eb26acb011eb0e44d6335ab"
 [7]: http://stackoverflow.com/questions/3263001/why-const-parameters-are-not-allowed-in-c-sharp "http://stackoverflow.com/questions/3263001/why-const-parameters-are-not-allowed-in-c-sharp"