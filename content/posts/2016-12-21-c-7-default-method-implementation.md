---
title: 'C# 7–Default method implementation?'
author: eiximenis

date: 2016-12-21T11:28:52+00:00
geeks_url: /?p=1857
geeks_ms_views:
  - 3963
categories:
  - 'C#'
  - opinion

---
El otro día estuve hablando con <a href="https://twitter.com/jjane90" target="_blank" rel="noopener noreferrer">Joan Jané</a>, sobre la funcionalidad que se <a href="https://github.com/dotnet/roslyn/issues/258" target="_blank" rel="noopener noreferrer">está valorando</a> para C#7 o (probablemente, dado su estado) más adelante. A falta de un nombre mejor llamaré a esa funcionalidad “Default method implementation” porque así se conoce en Java, donde esa funcionalidad ya existe.

Joan y yo teníamos puntos de vista totalmente opuestos a dicha característica, mientras que para mi era un añadido **muy interesante** al lenguaje, Joan se alineaba más con las tesis que Fernando Escolar <a href="http://fernandoescolar.github.io/2016/11/16/csharp-7/" target="_blank" rel="noopener noreferrer">expone en un post en su blog</a>. **Para <a href="https://twitter.com/fernandoescolar" target="_blank" rel="noopener noreferrer">Fer</a>, esa feature es la peor idea que ha tenido Java en los últimos años**. A mi me da la sensación que **verlo así es no entender exactamente que añade esa característica** y analizarla desde una posición demasiado rígida. Joan argumentaba problemas relacionados con SOLID, generalmente con el SRP y con la segregación de interfaces. En este post voy a comentar lo que, desde mi punto de vista, permitiría esa característica de estar disponible. Y por qué, no solo no es una mala idea, si no, a priori, todo lo contrario (tanto en Java como en C# por más que se empecine Fer en decir lo contrario).

<!--more-->

**¿En qué consiste esa característica?**

Bueno, básicamente consiste en **que las interfaces pueden definir una implementación para sus métodos**. Dicho de golpe y porrazo esto parece una aberración. Plantean algo parecido a lo siguiente (copio y pego el código que pone Fer en su blog):

<div class="highlight" style="background: #f8f8f8">
  <pre style="line-height: 100%"><span style="font-weight: bold; color: #008000">interface</span> ISomeInterface
{
    <span style="color: #b00040">string</span> Property { <span style="font-weight: bold; color: #008000">get</span>; }
    <span style="font-weight: bold; color: #008000">default</span> <span style="color: #b00040">string</span> <span style="color: #0000ff">Format</span>()
    {
        <span style="font-weight: bold; color: #008000">return</span> <span style="color: #b00040">string</span>.Format(<span style="color: #ba2121">"{0} ({1})"</span>, GetType().Name, Property);
    }
}
<span style="font-weight: bold; color: #008000">class</span> <span style="font-weight: bold; color: #0000ff">SomeClass</span> : ISomeInterface
{
    <span style="font-weight: bold; color: #008000">public</span> <span style="color: #b00040">string</span> Property { <span style="font-weight: bold; color: #008000">get</span>; <span style="font-weight: bold; color: #008000">set</span>; }
}
</pre>
</div>

Visto así, parece realmente algo sin sentido. ¿Interfaces definiendo métodos? ¿**Nos hemos vuelto locos?**

El primer argumento que se esgrime contra esa característica es el mantra de que “las interfaces definen un contrato y por lo tanto no deben poder implementar métodos”. De esta afirmación se infiere que no se _puede añadir comportamiento a una interfaz_. Todo muy bonito, pero extremadamente limitante en un sistema de tipos tan estricto como el de C# (y el de Java). Igual, alguien se pregunta **qué sentido tiene agregarle comportamiento a una interfaz.** Bien, quien se pregunte eso, debería revisar por ejemplo Linq. Linq **no es nada más que agregar comportamiento a la interfaz IEnumerable<T>**. Si en C# no se hubieran inventado un mecanismo para poder hacer eso, ahora existirían N implementaciones de Linq (una por cada clase que implementa IEnumerable<T>). Y, por supuesto, cada clase futura que implementase IEnumerable<T> (ya fuese o no del framework) debería implementar su propia versión de Linq. ¿Qué… ya no suena tan bonito eh? Simplemente **a veces es interesante que las interfaces puedan tener un comportamiento por defecto**.

Por supuesto C# dio con un mecanismo para ello, que es los métodos de extensión sobre interfaces. Al final, si la idea es proponer un método Format() que sirva para todos los objetos ISomeInterface **podemos declarar un método de extensión**:

<div class="highlight" style="background: #f8f8f8">
  <pre style="line-height: 100%"><span style="font-weight: bold; color: #008000">interface</span> ISomeInterface
{
    <span style="color: #b00040">string</span> Property { <span style="font-weight: bold; color: #008000">get</span>; }
}
<span style="font-weight: bold; color: #008000">class</span> <span style="font-weight: bold; color: #0000ff">SomeClass</span> : ISomeInterface
{
    <span style="font-weight: bold; color: #008000">public</span> <span style="color: #b00040">string</span> Property { <span style="font-weight: bold; color: #008000">get</span>; <span style="font-weight: bold; color: #008000">set</span>; }
}
<span style="font-weight: bold; color: #008000">class</span> <span style="font-weight: bold; color: #0000ff">SomeInterfaceExtensions</span> {
    <span style="font-weight: bold; color: #008000">public</span> <span style="font-weight: bold; color: #008000">static</span> <span style="color: #b00040">string</span> <span style="color: #0000ff">Format</span>(<span style="font-weight: bold; color: #008000">this</span> ISomeInterface self) {
        <span style="font-weight: bold; color: #008000">return</span> <span style="color: #b00040">string</span>.Format(<span style="color: #ba2121">"{0} ({1})"</span>,
            self.GetType().Name, self.Property);
    }
}
</pre>
</div>

Con tal de que añadamos el _using_ correspondiente al namespace donde está declarada la clase _SomeInterfaceExtensions_ (si es que está en un namespace distinto del de la interfaz _ISomeInterface_) ya podemos llamar a Format() sobre cualquier objeto que implemente _ISomeInterface_. Exactamente igual que en el caso anterior. Efectivamente, Linq está implementado como un conjunto de métodos de extensión sobre IEnumerable<T>. Pero debemos entender que para **el tema que nos ocupa es irrelevante si dichos métodos se implementan en como métodos estáticos en una clase separada** y el compilador hace el “truco del almendruco” **o bien dichos métodos están implementados en la propia interfaz**. Es irrelevante porque en ambos casos la idea fundamental es la misma: añadir comportamiento a una interfaz.

En definitiva, **si te sientes cómodo con los métodos de extensión sobre interfaces y te parecen bien, y ves como una aberración la implementación de métodos en una interfaz… es que no entiendes el motivo de los métodos de extensión**. Porque ambas técnicas, en el fondo, persiguen lo mismo.

Por supuesto, dado que C# ya tiene métodos de extensión es lícito preguntarnos si la implementación de métodos en interfaces aporta algo al lenguaje (en Java la respuesta es mucho más clara, ya que no hay métodos de extensión). Yo pienso **que sí**, que la implementación de métodos en una interfaz aporta soluciones incluso teniendo ya los métodos de extensión, y voy a dedicar el resto del post en hablar de eso.

**Métodos de extensión vs default implementation methods**

**Nota:** Cuando analice el funcionamiento de los _default implementation methods_ voy a hacerlo en base a como funciona en Java. En C# se está discutiendo, así que las decisiones que se tomen en base a como dicha característica aterriza (si finalmente aterriza) a C# pueden afectar lo que diga a continuación.

Ambas características sirven a un propósito similar, pero lo interesante es que son complementarias en tanto que el punto “fuerte” de cada una de ellas es el punto “débil” de la otra:

  * Con métodos de extensión **podemos extender interfaces que no controlamos**. Eso es, interfaces que no son “nuestros”. Podemos extender IEnumerable<T> a pesar de que IEnumerable<T> es una interfaz del framework. Con los default implementation methods no podemos hacer eso, ya que para añadir un método default a IEnumerable<T> debemos modificar el código de IEnumerable<T> ya que el método se añade en la propia interfaz. 
      * Los default extension methods son **resueltos en tiempo de ejecución, no de compilación**, por lo que pueden ser redefinidos en una clase derivada, y la implementación redefinida será usada en lugar de la implementación propuesta por la interfaz, con independencia del tipo de la referencia. Vamos, que son métodos virtuales. Eso no ocurre con los métodos de extensión, que se resuelven en tiempo de compilación:</ul> 
    <div class="highlight" style="background: #f8f8f8">
      <pre style="line-height: 100%"><span style="font-weight: bold; color: #008000">class</span> <span style="font-weight: bold; color: #0000ff">Program</span>
{
    <span style="font-weight: bold; color: #008000">static</span> <span style="font-weight: bold; color: #008000">void</span> <span style="color: #0000ff">Main</span>(<span style="color: #b00040">string</span>[] args)
        {
            IA a = <span style="font-weight: bold; color: #008000">new</span> A();
            a.Test();
        }
    }
}
<span style="font-weight: bold; color: #008000">interface</span> IA { }
<span style="font-weight: bold; color: #008000">class</span> <span style="font-weight: bold; color: #0000ff">A</span> : IA {
    <span style="font-weight: bold; color: #008000">public</span> <span style="font-weight: bold; color: #008000">void</span> <span style="color: #0000ff">Test</span>()
    {
        Console.WriteLine(<span style="color: #ba2121">"A::Test"</span>);
    }
}
<span style="font-weight: bold; color: #008000">static</span> <span style="font-weight: bold; color: #008000">class</span> <span style="font-weight: bold; color: #0000ff">AExtensions</span>
{
    <span style="font-weight: bold; color: #008000">public</span> <span style="font-weight: bold; color: #008000">static</span> <span style="font-weight: bold; color: #008000">void</span> <span style="color: #0000ff">Test</span>(<span style="font-weight: bold; color: #008000">this</span> IA a)
    {
        Console.WriteLine(<span style="color: #ba2121">"AExtensions::Test"</span>);
    }
}
</pre>
    </div>
    
    Este código imprime “AExtensions::Test” en lugar de “A::Test” por la pantalla. A pesar de que el objeto es de tipo A y a pesar de que la clase A proporciona su propia implementación del método de extensión. Pero como la referencia es de tipo IA, se usa el método de extensión. Eso, a la práctica, impide que una clase redefina un método de extensión (con una implementación más eficiente o más adaptada) con la total seguridad de que ese método será usado en todos los casos posibles. Una pena, pero claro, cuando el compilador hace “trucos” estamos limitados a los trucos que el compilador puede hacer. Igual te parece que eso no es relevante, pero lo es. Imagina un método Last() declarado como método de extensión sobre IEnumerable<T>. Este método puede implementarse más o menos eficientemente en función del tipo real del IEnumerable. P. ej. si tenemos acceso directo y sabemos el número de elementos (como una List<T>) podemos devolver el último elemento de forma directa. Ahora bien, el método de extensión Last() no puede hacer eso, debe recorrer todo el IEnumerable, porque lo único que se puede hacer con un IEnumerable es recorrerlo. Por supuesto se podría usar reflection o una serie de “ifs con is” y preguntar si el IEnumerable es realmente una IList (y Linq hace eso en algunos casos) pero es una solución que ya se ve que no escala y que no es de uso general.
    
    Lo que realmente necesitaríamos en estos casos es que la clase List<T> definiese su propio método Last() y que se llamase a ese método siempre que llamara a Last() de cualquier objeto que fuese una List<T> a pesar de que la referencia fuese de tipo IEnumerable<T>. Precisamente, lo que conseguiríamos si Last() fuese un método “default” implementado en IEnumerable<T> y redefinido en List<T>.
    
    Por lo tanto: **sí. Hay lugar para los default implementation methods** a pesar de tener ya métodos de extension.
    
    **Anda… ¡y traits!**
    
    Otra ventaja de los default implementation methods es que abren de par en par las puertas de los traits a C#. Los traits son un tipo particular de herencia múltiple, que no experimenta los problemas de la herencia múltiple genérica. Un trait no es nada más que “un pedazo de funcionalidad” (es decir un conjunto de métodos) pensada para ser añadida a cualquier clase existente. No es la idea de un trait que se creen instancias de él.
    
    En efecto **una interfaz con todos sus métodos implementados como default methods es ni más ni menos que un trait**. Quizá te preguntes porque no podemos usar una clase abstracta con todos sus métodos implementados (es decir no abstractos) como trait. Pues muy fácil: en C# solo puedo heredar de una clase. Por lo tanto eso me limitaría a aplicar como máximo un trait a cada clase (y además impediría que dicha clase heredase de otra clase). No, las clases abstractas no nos sirven… Pero una interfaz con todos sus métodos implementados como métodos _default_ es casi idéntico a una clase abstracta y ¡una clase puede implementar los que desee! Así, aplicar un trait se convierte en implementar una interfaz. Pero, dado que dicha interfaz tiene todos sus métodos implementados como métodos _default_, a la clase “no le cuesta nada” implementar dicha interfaz (es decir, no debe añadirse código alguno a la clase).
    
    **Protocolos**
    
    El concepto de “interface” que nos parece tan natural y “tan OOP” no existe en muchos lenguajes orientados a objetos. Muchos de esos lenguajes trabajan con un concepto parecido al de interface, pero no igual, que es el de protocolo. Básicamente un protocolo es una “funcionalidad (conjunto de métodos) que está disponible para un tipo determinado”. Parece lo mismo que una interface, pero no lo es. Una diferencia entre protocolo y la interface clásica de C#/Java es que el primero puede definir comportamiento (recuerda que el protocolo se define como un comportamiento disponible para un tipo). Es decir, exactamente lo mismo que los _default implementation methods._ Lenguajes como Swift o Objective-C usan constantemente protocolos. Así, que la idea de “interfaces que definen métodos” no es tan aberrante como pueda parecer.
    
    Otros lenguajes, como Elixir, llevan el concepto más allá y permiten “aplicar un protocolo a un tipo”. Eso es, hacer que las instancias de un determinado tipo X sean también instancias del protocolo P, aplicando el protocolo P a X. Esa aplicación ocurre fuera de X y consiste en definir los métodos de P basados en una instancia de X. Es parecido a los métodos de extensión pero el efecto es radicalmente distinto: con un método de extensión conseguimos extender una interfaz I, pero si el tipo A no implementa I, seguirá sin hacerlo. Aplicando protocolos lo que conseguimos es añadir los métodos de I al tipo A, de forma que A ahora implementa I, pero lo hemos hecho sin modificar A.
    
    **En resúmen…**
    
    Por supuesto hay aspectos que deben analizarse sobre como añadir los _default implementation methods_ en C# y hay muchos aspectos que no he considerado en este post. Para solucionar algunos de ellos basta fijarse en Java y para solucionar otros más especificos de C# (p. ej. que ocurre con implementaciones explícitas) se debe analizar en más profundidad.
    
    Pero lo que está clarísimo **es que el concepto de que una interfaz implemente métodos** no es, ni de lejos, una aberración. Y que añadirlo al lenguaje, lo enriquece. Por supuesto, como característica puede usarse mal. Pero eso ya ocurre con cualquier característica actual, así que no debería ser motivo para descartarla. **Para mí**, incluso en su forma más sencilla (parecida a la actual de Java) **aporta lo suficiente como para valer la pena.** Personalmente me gustaría ver aplicación de protocolos en C#, pero me temo que eso está mucho más lejos (si los _default implementatio methods_ requieren ya ayuda del CLR, eso mucho más todavía).
    
    ¿Y vosotros? ¿Qué opináis? ¿Os parece interesante esa característica, os da igual u os parece una aberración? Cualquiera está invitado, como siempre, a comentarlo 🙂
    
    Saludos!