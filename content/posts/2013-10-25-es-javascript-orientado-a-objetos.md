---
title: ¿Es Javascript orientado a objetos?
author: eiximenis

date: 2013-10-25T12:40:41+00:00
geeks_url: /?p=1655
geeks_visits:
  - 5814
geeks_ms_views:
  - 2219
categories:
  - Uncategorized

---
Ayer tuve el placer de participar en un hangout de [#JsIO][1] junto a dos bestias pardas como [Erick Ruiz][2] y [Tomás Corral][3] discutiendo si JavaScript es o no es un lenguaje orientado a objetos.

Así que aprovecho para escribir este post y hacer algunas reflexiones más al respecto sin las prisas ni la improvisación del “directo”.

**¿Qué es la orientación a objetos?**

Es complicado definir que significa exactamente “orientación a objetos”. Seguro que si le preguntas a varia gente distinta obtendrás diferentes respuestas. Con puntos coincidentes, pero también puntos divergentes. Esto se debe a que el concepto es lo suficientemente vago como para admitir varias interpretaciones y además de que la gran mayoría de lenguajes no son 100% orientados a objetos.

Si yo hubiese de definir orientación a objetos diría básicamente que se trata de modelar el problema, y por lo tanto de diseñar un programa, basándose en la interacción entre distintos objetos que tienen determinadas propiedades y con los cuales se pueden realizar determinadas acciones. Los objetos tienen tres propiedades fundamentales:

  1. Comportamiento: Se define como “lo que es posible hacer con un objeto”. En terminología más purista, el conjunto de mensajes a los que este objeto responde (en OOP purista se usa la terminología _mensaje_ para referirse al paso de información entre objetos). 
  2. Estado: Los objetos tienen un estado en todo momento, definido por el conjunto de las variables o campos que contienen. El estado es pues la _información_ contenida en un objeto. 
  3. Identidad: Cada objeto existe _independientemente_ del resto. Pueden haber dos objetos _iguales_ pero no tienen porque ser el mismo (de igual forma que dos mellizos pueden ser idénticos, pero no por ello dejan de ser dos personas). 

No iría mucho más allá, porque para mi esta definición contiene la clave que diferencia de la POO de la programación procedural: Los datos (estado) y funciones relacionadas (comportamiento) están agrupados en una sola entidad (objeto), en lugar de estar dispersos en el código. Es decir, el objeto _encapsula_ los datos y el comportamiento. Como desarrollador debemos pensar en modelar nuestro sistema como un conjunto de objetos, en lugar de como un conjunto de funciones (procedimientos) invocadas una tras otra y pasándose datos más o menos arbitrarios para solucionar el problema.

El resto son detalles: herencia, polimorfismo, visibilidad… son detalles. No son menores, ciertamente, pero desde un punto de visto teórico, son meros detalles.

<p align="left">
  Y fíjate: he sido capaz de definir POO sin usar la palabra <em>clase</em> una sola vez. Y es que las clases son un mecanismo para implementar la POO, pero nada más. Y por supuesto, no son el único.
</p>

<p align="left">
  JavaScript tiene objetos, y estos objetos tienen comportamiento, estado e identidad. Así que desde punto de vista… Es orientado a objetos.
</p>

<p align="left">
  Pero como entiendo que esta respuesta puede no colmar a todos, especialmente a aquellos que vengan de Java, C# o algún otro lenguaje orientado a objetos “tradicional” (debería decir<em> basado en clases</em>) vayamos a ver como en JavaScript también tenemos esos <em>detalles</em> que mencionaba antes disponibles… 😉
</p>

<p align="left">
  <strong>¿Herencia en JavaScript?</strong>
</p>

<p align="left">
  Vale… Antes que nada: Si has aprendido POO con Java, C#, C++ o algún otro lenguaje basado en clases recuerda el concepto fundamental: <strong>No hay clases en JavaScript</strong>. Por lo tanto, hazte un favor y deja de pensar en clases cuando desarrolles en JavaScript. Te ahorrarás muchos dolores de cabeza.
</p>

<p align="left">
  La herencia en JavaScript es <em>herencia entre objetos</em>.
</p>

  * <div align="left">
      En un lenguaje basado en clases, la herencia es una relación definida entre clases. Si una clase Rectángulo deriva de la clase Figura, todos los objetos Rectángulo <em>heredarán</em> todo el comportamiento y estado definidos en la clase Figura.
    </div>

  * <div align="left">
      En JavaScript la herencia es una relación definida entre objetos. Si un objeto deriva de otro, heredará todo el comportamiento y estado definido en el objeto base.
    </div>

<p align="left">
  El mecanismo de herencia que se usa en JavaScript se conoce como herencia por prototipo y para resumirlo diré básicamente que <em>cada objeto</em> tiene asociado un prototipo. Si el desarrollador invoca un método sobre un objeto y este método NO está definido en el propio objeto, el motor de JavaScript buscará el método en el prototipo de este objeto. Por supuesto el prototipo es un objeto y puede tener otro prototipo y así sucesivamente, creando lo que se conoce como <em>cadena de prototipado</em>. La pregunta es pues como crear y asignar el prototipo de un objeto. Hay varias maneras de hacerlo, pero veamos una rápida y sencilla:
</p>

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:a197d291-8470-4586-930f-15512fcd3454" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
    <div style="background: #000080; color: #fff; font-family: Verdana, Tahoma, Arial, sans-serif; font-weight: bold; padding: 2px 5px">
      Herencia por prototipo (1)
    </div>
    
    <div style="background: #ddd; max-height: 300px; overflow: auto">
      <ol start="1" style="background: #1e1e1e; margin: 0 0 0 2em; padding: 0 0 0 5px; white-space: nowrap">
        <li>
          <span style="background:#1e1e1e;color:#569cd6">var</span><span style="background:#1e1e1e;color:#dcdcdc"> figura </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> {</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">    </span><span style="background:#1e1e1e;color:#dcdcdc">draw</span><span style="background:#1e1e1e;color:#b4b4b4">:</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">function</span><span style="background:#1e1e1e;color:#b4b4b4">()</span><span style="background:#1e1e1e;color:#dcdcdc"> {</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">        </span><span style="background:#1e1e1e;color:#dcdcdc">console</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">log</span><span style="background:#1e1e1e;color:#b4b4b4">(</span><span style="background:#1e1e1e;color:#d69d85">"figura::draw"</span><span style="background:#1e1e1e;color:#b4b4b4">);</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">    }</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">}</span><span style="background:#1e1e1e;color:#b4b4b4">;</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#569cd6">var</span><span style="background:#1e1e1e;color:#dcdcdc"> rect </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> Object</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">create</span><span style="background:#1e1e1e;color:#b4b4b4">(</span><span style="background:#1e1e1e;color:#dcdcdc">figura</span><span style="background:#1e1e1e;color:#b4b4b4">);</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">rect</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">draw</span><span style="background:#1e1e1e;color:#b4b4b4">();</span>
        </li>
      </ol>
    </div></p>
  </div></p>
</div>

<p align="left">
  Si ejecutas este código, la salida por la consola es “figura::draw”. La clave es el método Object.create. Est<br /> e método crea un objeto vacío <em>cuyo prototipo es el objeto pasado por parámetro</em>. Esta es una forma sencilla de tener herencia en JavaScript.
</p>

<p align="left">
  Igual te preguntas si el objeto rect puede sobreescribir la implementación de draw (si vienes de Java sabrás que una clase hija puede redefinir los métodos de su clase base; si vienes de C# sabrás eso mismo siempre que los métodos sean virtuales). La respuesta es claro que sí:
</p>

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:7129eebd-dbf9-4497-9e59-9b9b4fb50d77" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
    <div style="background: #000080; color: #fff; font-family: Verdana, Tahoma, Arial, sans-serif; font-weight: bold; padding: 2px 5px">
      Redefinicion de metodos
    </div>
    
    <div style="background: #ddd; max-height: 300px; overflow: auto">
      <ol start="1" style="background: #1e1e1e; margin: 0 0 0 2.5em; padding: 0 0 0 5px; white-space: nowrap">
        <li>
          <span style="background:#1e1e1e;color:#569cd6">var</span><span style="background:#1e1e1e;color:#dcdcdc"> figura </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> {</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">    </span><span style="background:#1e1e1e;color:#dcdcdc">draw</span><span style="background:#1e1e1e;color:#b4b4b4">:</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">function</span><span style="background:#1e1e1e;color:#b4b4b4">()</span><span style="background:#1e1e1e;color:#dcdcdc"> {</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">        </span><span style="background:#1e1e1e;color:#dcdcdc">console</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">log</span><span style="background:#1e1e1e;color:#b4b4b4">(</span><span style="background:#1e1e1e;color:#d69d85">"figura::draw"</span><span style="background:#1e1e1e;color:#b4b4b4">);</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">    }</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">}</span><span style="background:#1e1e1e;color:#b4b4b4">;</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#569cd6">var</span><span style="background:#1e1e1e;color:#dcdcdc"> rect </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> Object</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">create</span><span style="background:#1e1e1e;color:#b4b4b4">(</span><span style="background:#1e1e1e;color:#dcdcdc">figura</span><span style="background:#1e1e1e;color:#b4b4b4">);</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">rect</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">draw </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">function</span><span style="background:#1e1e1e;color:#b4b4b4">()</span><span style="background:#1e1e1e;color:#dcdcdc"> {</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">    console</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">log</span><span style="background:#1e1e1e;color:#b4b4b4">(</span><span style="background:#1e1e1e;color:#d69d85">"rect::draw"</span><span style="background:#1e1e1e;color:#b4b4b4">);</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">}</span><span style="background:#1e1e1e;color:#b4b4b4">;</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">rect</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">draw</span><span style="background:#1e1e1e;color:#b4b4b4">();</span>
        </li>
      </ol>
    </div></p>
  </div></p>
</div>

<p align="left">
  Ahora la salida por pantalla es “rect::draw”. ¿Qué ha sucedido? Pues como hemos llamado al método <em>draw</em> del objeto rect y este ya lo tiene definido, JavaScript ya lo invoca y no lo busca por la cadena de prototipos.
</p>

<p align="left">
  ¿Lo ves? Herencia sin clases. ¡Es posible! 😉
</p>

<p align="left">
  <strong>¿Polimorfismo en JavaScript?</strong>
</p>

<p align="left">
  Una de las preguntas que salieron en el hangout fue si había polimorfismo en JavaScript. Respondí diciendo que esta pregunta implica una forma de pensar “basada en clases” pero que sí que era posible simular el polimorfismo en JavaScript.
</p>

<p align="left">
  Pero es que una de las claves es que <strong>no es necesario el concepto de polimorfismo en JavaScript</strong>. Antes que nada aclaremos que es el polimorfismo: La posibilidad de enviar un mismo mensaje (es decir, de invocar a un método) a varios objetos de naturaleza homogénea (<a href="http://es.wikipedia.org/wiki/Polimorfismo_(programaci%C3%B3n_orientada_a_objetos)">wikipedia</a>). Me gusta especialmente esta definición porque define polimorfismo sin usar la palabra “clase”. Si habéis aprendido POO con un lenguaje basado en clases quizá tenéis una definición de polimorfismo redactada de forma ligeramente distinta, algo como: Que los objetos de una clase derivada pueden tratarse como objetos de la clase base, pero manteniendo su comportamiento distintivo.
</p>

<p align="left">
  Supón que estás en un lenguaje basado en clases, como C#, y tienes la clase Figura y su derivada la clase Rect. La clase Figura define un método virtual llamado Draw() que es redefinido en la clase Rect. El polimorfismo implica que puedo pasar un objeto de la clase Rect a un método que acepte como parámetro un objeto de la clase Figura. Y que si este método llama al método Draw del parámetro Figura que ha recibido… se ejecutará el método Draw de la clase Rect, porque <em>aunque el parámetro es de tipo Figura, el objeto real es de tipo Rect</em>.
</p>

<p align="left">
  ¿Y en JavaScript? Veamos:
</p>

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:1aae93e5-d0cf-4163-9e44-9149c82d1d80" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
    <div style="background: #000080; color: #fff; font-family: Verdana, Tahoma, Arial, sans-serif; font-weight: bold; padding: 2px 5px">
      Polimorfismo
    </div>
    
    <div style="background: #ddd; max-height: 300px; overflow: auto">
      <ol start="1" style="background: #1e1e1e; margin: 0 0 0 2.5em; padding: 0 0 0 5px; white-space: nowrap">
        <li>
          <span style="background:#1e1e1e;color:#569cd6">var</span><span style="background:#1e1e1e;color:#dcdcdc"> figura </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> {</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">    </span><span style="background:#1e1e1e;color:#dcdcdc">draw</span><span style="background:#1e1e1e;color:#b4b4b4">:</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">function</span><span style="background:#1e1e1e;color:#b4b4b4">()</span><span style="background:#1e1e1e;color:#dcdcdc"> {</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">        </span><span style="background:#1e1e1e;color:#dcdcdc">console</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">log</span><span style="background:#1e1e1e;color:#b4b4b4">(</span><span style="background:#1e1e1e;color:#d69d85">"figura::draw"</span><span style="background:#1e1e1e;color:#b4b4b4">);</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">    }</span>
        </li>
        <li>
          <span style="ba
ckground:#1e1e1e;color:#dcdcdc">}</span><span style="background:#1e1e1e;color:#b4b4b4">;</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#569cd6">var</span><span style="background:#1e1e1e;color:#dcdcdc"> rect </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> Object</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">create</span><span style="background:#1e1e1e;color:#b4b4b4">(</span><span style="background:#1e1e1e;color:#dcdcdc">figura</span><span style="background:#1e1e1e;color:#b4b4b4">);</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">rect</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">draw </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">function</span><span style="background:#1e1e1e;color:#b4b4b4">()</span><span style="background:#1e1e1e;color:#dcdcdc"> {</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">    console</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">log</span><span style="background:#1e1e1e;color:#b4b4b4">(</span><span style="background:#1e1e1e;color:#d69d85">"rect::draw"</span><span style="background:#1e1e1e;color:#b4b4b4">);</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">}</span><span style="background:#1e1e1e;color:#b4b4b4">;</span>
        </li>
        <li>
          &nbsp;
        </li>
        <li>
          <span style="background:#1e1e1e;color:#569cd6">function</span><span style="background:#1e1e1e;color:#dcdcdc"> Polimorfismo</span><span style="background:#1e1e1e;color:#b4b4b4">(</span><span style="background:#1e1e1e;color:#dcdcdc">figura</span><span style="background:#1e1e1e;color:#b4b4b4">)</span><span style="background:#1e1e1e;color:#dcdcdc"> {</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">    figura</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">draw</span><span style="background:#1e1e1e;color:#b4b4b4">();</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">}</span>
        </li>
        <li>
          &nbsp;
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">Polimorfismo</span><span style="background:#1e1e1e;color:#b4b4b4">(</span><span style="background:#1e1e1e;color:#dcdcdc">rect</span><span style="background:#1e1e1e;color:#b4b4b4">);</span>
        </li>
      </ol>
    </div></p>
  </div></p>
</div>

<p align="left">
  La función <em>Polimorfismo</em> llama al método draw() del objeto que recibe. Como lo paso el objeto <em>rect</em> lo que se ejecuta es el método <em>draw</em> del objeto <em>rect</em>. Por lo tanto… <strong>tenemos polimorfismo automático en JavaScript. </strong>De hecho JavaScript tiene una característica conocida como <em>Duck Typing</em>, que bueno… es dificil de definir, pero una manera de hacerlo es: “Si camina como un pato, grazna como un pato y luce como un pato… es un pato”. O dicho de otro modo: Para el método Polimorfismo lo único que importa del parámetro <em>figura</em> es que tenga un método draw(). Nada más. Por lo tanto cualquier objeto que tenga un método <em>draw</em> es válido para ser usado como parámetro del método Polimorfismo.
</p>

<p align="left">
  El Duck Typing se suele asociar a los lenguajes dinámicos… pero no es exclusivo de ellos. Java y C# (sin usar dynamic) no tienen Duck Typing, pero p. ej. C++ lo ofrece a través de los <em>templates</em>.
</p>

<p align="left">
  <strong>¿Clases en JavaScript?</strong>
</p>

<p align="left">
  A esta altura del post ya debes tener claro que no. JavaScript no tiene clases, solo objetos y relaciones entre objetos. Pero existe un método de crear objetos a través de una función <em>constructora y la palabra clave new</em> que se parece mucho a como se hace en un lenguaje basado en clases. Eso que a priori es buena idea, ha ayudado mucho a la confusión de JavaScript… ¿Uso new para crear objetos pero no hay clases? ¿Y eso?
</p>

<p align="left">
  Déjame mostrarte un trozo de código de como crear un objeto basado en función constructora y el uso de new:
</p>

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:5fd0b0ca-4b5d-4c2c-a752-a25931f2d1ca" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
    <div style="background: #000080; color: #fff; font-family: Verdana, Tahoma, Arial, sans-serif; font-weight: bold; padding: 2px 5px">
      Funciones constructoras y new
    </div>
    
    <div style="background: #ddd; max-height: 300px; overflow: auto">
      <ol start="1" style="background: #1e1e1e; margin: 0 0 0 2.5em; padding: 0 0 0 5px; white-space: nowrap">
        <li>
          <span style="background:#1e1e1e;color:#569cd6">var</span><span style="background:#1e1e1e;color:#dcdcdc"> Figura </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">function</span><span style="background:#1e1e1e;color:#b4b4b4">()</span><span style="background:#1e1e1e;color:#dcdcdc"> {</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">    </span><span style="background:#1e1e1e;color:#569cd6">this</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">draw </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">function</span><span style="background:#1e1e1e;color:#b4b4b4">()</span><span style="background:#1e1e1e;color:#dcdcdc"> {</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">        console</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">log</span><span style="background:#1e1e1e;color:#b4b4b4">(</span><span style="background:#1e1e1e;color:#d69d85">"Figura::draw"</span><span style="background:#1e1e1e;color:#b4b4b4">);</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">    }</span><span style="background:#1e1e1e;color:#b4b4b4">;</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">}</span><span style="background:#1e1e1e;color:#b4b4b4">;</span>
        </li>
        <li>
          &nbsp;
        </li>
        <li>
          <span style="background:#1e1e1e;color:#569cd6">var</span><span style="background:#1e1e1e;color:#dcdcdc"> figura </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">new</span><span style="background:#1e1e1e;color:#dcdcdc"> Figura</span><span style="background:#1e1e1e;color:#b4b4b4">();</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#569cd6">var</span><span style="background:#1e1e1e;color:#dcdcdc"> rect </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> Object</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">create</span><span style="background:#1e1e1e;color:#b4b4b4">(</span><span style="background:#1e1e1e;color:#dcdcdc">figura</span><span style="background:#1e1e1e;color:#b4b4b4">);</span>
        </li>
        <li>
          &nbsp;
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">rect</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">draw</span><span style="background:#1e1e1e;color:#b4b4b4">();</span>
        </li>
      </ol>
    </div></p>
  </div></p>
</div>

<p align="left">
  Ahora la variable Figura no es nada más que una función. Pero un tipo especial de función que llamamos <em>función constructora. </em>Pero ¡ojo! lo que convierte Figura en una función constructora no es nada que defina la propia función. Es <em>como se invoca</em>. <strong>Es el hecho de usar new lo que convierte Figura en una función constructora</strong>. El uso de<br /> new implica varias cosillas, cuyo ámbito se escapa de este post, pero la norma básica es que es la forma para invocar funciones cuya funcionalidad es crear objetos. En este ejemplo pues tengo:
</p>

  * <div align="left">
      Figura: Función que sirve para construir objetos que tienen un método draw.
    </div>

  * <div align="left">
      figura (con la f minúscula): Objeto creado a partir de Figura
    </div>

  * <div align="left">
      rect: Objeto cuyo prototipo es el objeto figura.
    </div>

<p align="left">
  Lo interesante de usar funciones constructoras es que todos los objetos creados a través de ellas comparten el mismo prototipo. Si tenemos:
</p>

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:e85705dc-0aaa-471f-9b1a-90b46acee1c0" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
    <div style="background: #ddd; max-height: 300px; overflow: auto">
      <ol start="1" style="background: #1e1e1e; margin: 0 0 0 2em; padding: 0 0 0 5px; white-space: nowrap">
        <li>
          <span style="background:#1e1e1e;color:#569cd6">var</span><span style="background:#1e1e1e;color:#dcdcdc"> figura </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">new</span><span style="background:#1e1e1e;color:#dcdcdc"> Figura</span><span style="background:#1e1e1e;color:#b4b4b4">();</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#569cd6">var</span><span style="background:#1e1e1e;color:#dcdcdc"> figura2 </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">new</span><span style="background:#1e1e1e;color:#dcdcdc"> Figura</span><span style="background:#1e1e1e;color:#b4b4b4">();</span>
        </li>
      </ol>
    </div></p>
  </div></p>
</div>

<p align="left">
  Ambos objetos comparten el mismo prototipo. Si ahora quiero añadir un método, p.ej. clear() que esté disponible para todos los objetos creados a partir de la función constructora Figura puedo añadirlo al prototipo. Y como hago esto? Pues así:
</p>

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:a89a8fa8-4f4a-495c-abee-ff96d703a3b1" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
    <div style="background: #ddd; max-height: 300px; overflow: auto">
      <ol start="1" style="background: #1e1e1e; margin: 0 0 0 2.5em; padding: 0 0 0 5px; white-space: nowrap">
        <li>
          <span style="background:#1e1e1e;color:#569cd6">var</span><span style="background:#1e1e1e;color:#dcdcdc"> Figura </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">function</span><span style="background:#1e1e1e;color:#b4b4b4">()</span><span style="background:#1e1e1e;color:#dcdcdc"> {</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">    </span><span style="background:#1e1e1e;color:#569cd6">this</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">draw </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">function</span><span style="background:#1e1e1e;color:#b4b4b4">()</span><span style="background:#1e1e1e;color:#dcdcdc"> {</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">        console</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">log</span><span style="background:#1e1e1e;color:#b4b4b4">(</span><span style="background:#1e1e1e;color:#d69d85">"Figura::draw"</span><span style="background:#1e1e1e;color:#b4b4b4">);</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">    }</span><span style="background:#1e1e1e;color:#b4b4b4">;</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">}</span><span style="background:#1e1e1e;color:#b4b4b4">;</span>
        </li>
        <li>
          &nbsp;
        </li>
        <li>
          <span style="background:#1e1e1e;color:#569cd6">var</span><span style="background:#1e1e1e;color:#dcdcdc"> figura </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">new</span><span style="background:#1e1e1e;color:#dcdcdc"> Figura</span><span style="background:#1e1e1e;color:#b4b4b4">();</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#569cd6">var</span><span style="background:#1e1e1e;color:#dcdcdc"> figura2 </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">new</span><span style="background:#1e1e1e;color:#dcdcdc"> Figura</span><span style="background:#1e1e1e;color:#b4b4b4">();</span>
        </li>
        <li>
          &nbsp;
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">Figura</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">prototype</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">clear </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">function</span><span style="background:#1e1e1e;color:#b4b4b4">()</span><span style="background:#1e1e1e;color:#dcdcdc"> {</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">    console</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">log</span><span style="background:#1e1e1e;color:#b4b4b4">(</span><span style="background:#1e1e1e;color:#d69d85">"Figura::clear"</span><span style="background:#1e1e1e;color:#b4b4b4">);</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">}</span><span style="background:#1e1e1e;color:#b4b4b4">;</span>
        </li>
        <li>
          &nbsp;
        </li>
        <li>
          <span style="background:#1e1e1e;color:#569cd6">var</span><span style="background:#1e1e1e;color:#dcdcdc"> rect </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> Object</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">create</span><span style="background:#1e1e1e;color:#b4b4b4">(</span><span style="background:#1e1e1e;color:#dcdcdc">figura</span><span style="background:#1e1e1e;color:#b4b4b4">);</span>
        </li>
        <li>
          &nbsp;
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">figura</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">clear</span><span style="background:#1e1e1e;color:#b4b4b4">();</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">figura2</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">clear</span><span style="background:#1e1e1e;color:#b4b4b4">();</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">rect</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">clear</span><span style="background:#1e1e1e;color:#b4b4b4">();</span>
        </li>
      </ol>
    </div></p>
  </div></p>
</div>

<p align="left">
  Figura.prototype es el nombre del prototipo de todos los objetos creados a través de <em>new Figura</em>. Este codigo imprime tres veces “Figura::clear”. Fíjate que funciona incluso cuando el método <em>clear</em> ha sido añadido al prototipo <em>después</em> de crear figura y figura2. Y es interesante el caso del objeto rect. Que ocurre cuando hacemos rect.clear()?
</p>

  * <div align="left">
      JavaScript bu<br /> sca el método clear() en el objeto rect. No lo encuentra y
    </div>

  * <div align="left">
      Busca el método en el prototipo de rect que es figura. No lo encuentra y
    </div>

  * <div align="left">
      Busca el método en el prototipo de figura que es Figura.prototype. Lo encuentra y lo ejecuta.
    </div>

<p align="left">
  Aquí tienes a la cadena de prototipado en acción.
</p>

<p align="left">
  Fíjate que seguimos teniendo tan solo objetos. Figura <strong>no es una clase</strong>. Figura <strong>es una función</strong>. Una función para crear objetos que comparten un prototipo. Nada más.
</p>

<p align="left">
  <strong>¿Variables privadas en JavaScript?</strong>
</p>

<p align="left">
  JavaScript no incorpora de serie ningún mecanismo de visibilidad para los miembros de un objeto. Todo es público por defecto. Pero por supuesto no hay nada que no podamos conseguir… 🙂
</p>

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:a75e86db-ef4f-447f-b9e3-8427f4a6913d" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
    <div style="background: #000080; color: #fff; font-family: Verdana, Tahoma, Arial, sans-serif; font-weight: bold; padding: 2px 5px">
      Visibilidades en JavaScript
    </div>
    
    <div style="background: #ddd; max-height: 300px; overflow: auto">
      <ol start="1" style="background: #1e1e1e; margin: 0 0 0 2.5em; padding: 0 0 0 5px; white-space: nowrap">
        <li>
          <span style="background:#1e1e1e;color:#569cd6">var</span><span style="background:#1e1e1e;color:#dcdcdc"> Figura </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">function</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#b4b4b4">(</span><span style="background:#1e1e1e;color:#dcdcdc">c</span><span style="background:#1e1e1e;color:#b4b4b4">,</span><span style="background:#1e1e1e;color:#dcdcdc"> stroke</span><span style="background:#1e1e1e;color:#b4b4b4">)</span><span style="background:#1e1e1e;color:#dcdcdc"> {</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">    </span><span style="background:#1e1e1e;color:#57a64a">// Variable privada</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">    </span><span style="background:#1e1e1e;color:#569cd6">var</span><span style="background:#1e1e1e;color:#dcdcdc"> _color </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> c</span><span style="background:#1e1e1e;color:#b4b4b4">;</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">    </span><span style="background:#1e1e1e;color:#57a64a">// Variable pblica</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">    </span><span style="background:#1e1e1e;color:#569cd6">this</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">stroke </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">null</span><span style="background:#1e1e1e;color:#b4b4b4">;</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">    </span><span style="background:#1e1e1e;color:#57a64a">// Necesario para poder acceder</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">    </span><span style="background:#1e1e1e;color:#57a64a">// a this desde los mtodos privados</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">    </span><span style="background:#1e1e1e;color:#569cd6">var</span><span style="background:#1e1e1e;color:#dcdcdc"> self </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">this</span><span style="background:#1e1e1e;color:#b4b4b4">;</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">    </span><span style="background:#1e1e1e;color:#57a64a">// Mtodo privado</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">    </span><span style="background:#1e1e1e;color:#569cd6">var</span><span style="background:#1e1e1e;color:#dcdcdc"> _setup </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">function</span><span style="background:#1e1e1e;color:#b4b4b4">(</span><span style="background:#1e1e1e;color:#dcdcdc">s</span><span style="background:#1e1e1e;color:#b4b4b4">)</span><span style="background:#1e1e1e;color:#dcdcdc"> {</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">        </span><span style="background:#1e1e1e;color:#57a64a">// Ah podemos acceder a mtodos pblicos</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">        </span><span style="background:#1e1e1e;color:#57a64a">// a travs de self. Y a los privados directamente</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">        self</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">stroke </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> s</span><span style="background:#1e1e1e;color:#b4b4b4">;</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">    }</span><span style="background:#1e1e1e;color:#b4b4b4">;</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">    _setup</span><span style="background:#1e1e1e;color:#b4b4b4">(</span><span style="background:#1e1e1e;color:#dcdcdc">stroke</span><span style="background:#1e1e1e;color:#b4b4b4">);</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">    </span><span style="background:#1e1e1e;color:#57a64a">// Mtodos pblicos</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">    </span><span style="background:#1e1e1e;color:#569cd6">this</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">draw </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">function</span><span style="background:#1e1e1e;color:#b4b4b4">()</span><span style="background:#1e1e1e;color:#dcdcdc"> {</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">        console</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">log</span><span style="background:#1e1e1e;color:#b4b4b4">(</span><span style="background:#1e1e1e;color:#d69d85">"Figura::draw in color "</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#b4b4b4">+</span><span style="background:#1e1e1e;color:#dcdcdc"> _color  </span><span style="background:#1e1e1e;color:#b4b4b4">+</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#d69d85">" and stroke "</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#b4b4b4">+</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">this</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">stroke </span><span style="background:#1e1e1e;color:#b4b4b4">);</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">    }</span><span style="background:#1e1e1e;color:#b4b4b4">;</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">    </span><span style="background:#1e1e1e;color:#569cd6">this</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">getColor </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span
style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">function</span><span style="background:#1e1e1e;color:#b4b4b4">()</span><span style="background:#1e1e1e;color:#dcdcdc"> {</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">        </span><span style="background:#1e1e1e;color:#569cd6">return</span><span style="background:#1e1e1e;color:#dcdcdc"> _color</span><span style="background:#1e1e1e;color:#b4b4b4">;</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">    }</span><span style="background:#1e1e1e;color:#b4b4b4">;</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">}</span><span style="background:#1e1e1e;color:#b4b4b4">;</span>
        </li>
        <li>
          &nbsp;
        </li>
        <li>
          &nbsp;
        </li>
        <li>
          <span style="background:#1e1e1e;color:#569cd6">var</span><span style="background:#1e1e1e;color:#dcdcdc"> f </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">new</span><span style="background:#1e1e1e;color:#dcdcdc"> Figura</span><span style="background:#1e1e1e;color:#b4b4b4">(</span><span style="background:#1e1e1e;color:#d69d85">"red"</span><span style="background:#1e1e1e;color:#b4b4b4">,</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#d69d85">"thin"</span><span style="background:#1e1e1e;color:#b4b4b4">);</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">console</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">log</span><span style="background:#1e1e1e;color:#b4b4b4">(</span><span style="background:#1e1e1e;color:#dcdcdc">f</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">_color</span><span style="background:#1e1e1e;color:#b4b4b4">);</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#57a64a">// undefined</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">console</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">log</span><span style="background:#1e1e1e;color:#b4b4b4">(</span><span style="background:#1e1e1e;color:#dcdcdc">f</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">getColor</span><span style="background:#1e1e1e;color:#b4b4b4">());</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#57a64a">// red</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">console</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">log</span><span style="background:#1e1e1e;color:#b4b4b4">(</span><span style="background:#1e1e1e;color:#dcdcdc">f</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">stroke</span><span style="background:#1e1e1e;color:#b4b4b4">);</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#57a64a">// thin</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#57a64a">//f._setup("thick");  // Error: Object has no method _setup</span>
        </li>
        <li>
          &nbsp;
        </li>
        <li>
          <span style="background:#1e1e1e;color:#569cd6">var</span><span style="background:#1e1e1e;color:#dcdcdc"> f2 </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">new</span><span style="background:#1e1e1e;color:#dcdcdc"> Figura</span><span style="background:#1e1e1e;color:#b4b4b4">(</span><span style="background:#1e1e1e;color:#d69d85">"blue"</span><span style="background:#1e1e1e;color:#b4b4b4">,</span><span style="background:#1e1e1e;color:#d69d85">"thick"</span><span style="background:#1e1e1e;color:#b4b4b4">);</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">console</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">log</span><span style="background:#1e1e1e;color:#b4b4b4">(</span><span style="background:#1e1e1e;color:#dcdcdc">f2</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">_color</span><span style="background:#1e1e1e;color:#b4b4b4">);</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#57a64a">// undefined</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">console</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">log</span><span style="background:#1e1e1e;color:#b4b4b4">(</span><span style="background:#1e1e1e;color:#dcdcdc">f2</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">getColor</span><span style="background:#1e1e1e;color:#b4b4b4">());</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#57a64a">// blue</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">console</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">log</span><span style="background:#1e1e1e;color:#b4b4b4">(</span><span style="background:#1e1e1e;color:#dcdcdc">f2</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">stroke</span><span style="background:#1e1e1e;color:#b4b4b4">);</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#57a64a">// thick</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#57a64a">//f._setup("thick");  // Error: Object has no method _setup</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">f</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">draw</span><span style="background:#1e1e1e;color:#b4b4b4">();</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">f2</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">draw</span><span style="background:#1e1e1e;color:#b4b4b4">();</span>
        </li>
      </ol>
    </div></p>
  </div></p>
</div>

<p align="left">
  Este ejemplo muestra como definir métodos y variables privadas. ¡Hay otras técnicas y alternativas!
</p>

<p align="left">
  Básicamente la regla es:
</p>

  * <div align="left">
      Usar la técnica de función constructora
    </div>

  * <div align="left">
      Los métodos/variables privados son funciones o variables declaradas dentro de la función constructora.
    </div>

  * <div align="left">
      Los métodos/variables públicas se asignan a this.
    </div>

  * <div align="left">
      Guardar el valor de this en una variable privada (usualmente se usa that o self).
    </div>

  * <div align="left">
      Desde las funciones privadas debes usar self para acceder a las variables públicas.
    </div>

  * <div align="left">
      Desde las funciones públicas puedes usar self o this para acceder a las variables públicas.
    </div>

  * <div align="left">
      En ambos casos puedes acceder a las variables privadas directamente con su nombre.
    </div>

<p align="left">
  <strong>¿Herencia múltiple en JavaScript?</strong>
</p>

<p align="left">
  Si vienes de C# o Java igual alzas una ceja ahora… ¿No era peligrosa la herencia múltiple? C# y Java no incorporan este concepto debido a su posible peligrosidad y porque muchas veces da más problemas de los que puede solucionar. El peligro de la herencia múltiple es conocido como el <a href="http://es.wikipedia.org/wiki/Problema_del_diamante">problema de la herencia en diamante</a>. Todos los lenguajes que soportan herencia múltiple se deben enfrentar a este problema, así que para evitarlo algunos lenguajes como Java o C# han decidido prescindir de ella.
</p>

<p align="left">
  JavaScript no soporta por defecto herencia múltiple p<br /> ero si que soporta un tipo especial de herencia múltiple llamada <a href="http://es.wikipedia.org/wiki/Mixin">mixin</a>. Con los mixins ya entramos en un terreno pantanoso si vienes de C# o Java puesto que estos dos lenguajes no soportan este concepto.
</p>

<p align="left">
  Resumiendo, un Mixin es una clase que está pensada para ser <em>incorporada dentro de otra clase </em>ofreciendo funcionalidad adicional. Es como si tuvieras 3 clases A, B y C y las “combinases” todas ellas en una clase D (que además añadiría su propia funcionalidad). Algunos lenguajes como Lisp o Python soportan Mixins nativamente. Otros como C++ no, pero pueden imitarlos fácilmente debido al soporte de herencia múltiple (del que el uso de mixins es un tipo específico). Usar mixins en Java o C# es realmente complicado (aunque en Java8 el uso de default methods en interfaces lo hace posible). Si estás interesado en Mixins y C# echa un vistazo a <a href="https://code.google.com/p/heredar/">heredar</a> de <a href="https://twitter.com/jfroma">José F. Romaniello</a>.
</p>

<p align="left">
  ¿Y como usar Mixins en C#? Aquí tienes un ejemplo:
</p>

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:a362c44f-f1b4-48a5-91f8-a0d35c381ce7" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
    <div style="background: #000080; color: #fff; font-family: Verdana, Tahoma, Arial, sans-serif; font-weight: bold; padding: 2px 5px">
      Mixins
    </div>
    
    <div style="background: #ddd; max-height: 300px; overflow: auto">
      <ol start="1" style="background: #1e1e1e; margin: 0 0 0 2.5em; padding: 0 0 0 5px; white-space: nowrap">
        <li>
          <span style="background:#1e1e1e;color:#569cd6">var</span><span style="background:#1e1e1e;color:#dcdcdc"> asCircle </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">function</span><span style="background:#1e1e1e;color:#b4b4b4">()</span><span style="background:#1e1e1e;color:#dcdcdc"> {</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">    </span><span style="background:#1e1e1e;color:#569cd6">this</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">area </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">function</span><span style="background:#1e1e1e;color:#b4b4b4">()</span><span style="background:#1e1e1e;color:#dcdcdc"> {</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">        </span><span style="background:#1e1e1e;color:#569cd6">return</span><span style="background:#1e1e1e;color:#dcdcdc"> Math</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">PI </span><span style="background:#1e1e1e;color:#b4b4b4">*</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">this</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">radius </span><span style="background:#1e1e1e;color:#b4b4b4">*</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">this</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">radius</span><span style="background:#1e1e1e;color:#b4b4b4">;</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">    }</span><span style="background:#1e1e1e;color:#b4b4b4">;</span>
        </li>
        <li>
          &nbsp;
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">    </span><span style="background:#1e1e1e;color:#569cd6">return</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">this</span><span style="background:#1e1e1e;color:#b4b4b4">;</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">}</span><span style="background:#1e1e1e;color:#b4b4b4">;</span>
        </li>
        <li>
          &nbsp;
        </li>
        <li>
          <span style="background:#1e1e1e;color:#569cd6">var</span><span style="background:#1e1e1e;color:#dcdcdc"> asButton </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">function</span><span style="background:#1e1e1e;color:#b4b4b4">()</span><span style="background:#1e1e1e;color:#dcdcdc"> {</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">    </span><span style="background:#1e1e1e;color:#569cd6">this</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">click </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">function</span><span style="background:#1e1e1e;color:#b4b4b4">()</span><span style="background:#1e1e1e;color:#dcdcdc"> {</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">        console</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">log</span><span style="background:#1e1e1e;color:#b4b4b4">(</span><span style="background:#1e1e1e;color:#d69d85">"Button has been clicked"</span><span style="background:#1e1e1e;color:#b4b4b4">);</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">    }</span><span style="background:#1e1e1e;color:#b4b4b4">;</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">    </span><span style="background:#1e1e1e;color:#569cd6">return</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">this</span><span style="background:#1e1e1e;color:#b4b4b4">;</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">}</span><span style="background:#1e1e1e;color:#b4b4b4">;</span>
        </li>
        <li>
          &nbsp;
        </li>
        <li>
          <span style="background:#1e1e1e;color:#569cd6">var</span><span style="background:#1e1e1e;color:#dcdcdc"> a </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> {</span><span style="background:#1e1e1e;color:#dcdcdc"> radius</span><span style="background:#1e1e1e;color:#b4b4b4">:</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#b5cea8">10</span><span style="background:#1e1e1e;color:#dcdcdc"> }</span><span style="background:#1e1e1e;color:#b4b4b4">;</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">asCircle</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">call</span><span style="background:#1e1e1e;color:#b4b4b4">(</span><span style="background:#1e1e1e;color:#dcdcdc">a</span><span style="background:#1e1e1e;color:#b4b4b4">);</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">asButton</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">call</span><span style="background:#1e1e1e;color:#b4b4b4">(</span><span style="background:#1e1e1e;color:#dcdcdc">a</span><span style="background:#1e1e1e;color:#b4b4b4">);</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">console</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">log</span><span style="background:#1e1e1e;color:#b4b4b4">(</span><span style="background:#1e1e1e;color:#dcdcdc">a</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">area</span><span style="background:#1e1e1e;color:#b4b4b4">());</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">console</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">log</span><span style="background:#1e1e1e;color:#b4b4b4">(</span><span style="background:#1e
1e1e;color:#dcdcdc">a</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">click</span><span style="background:#1e1e1e;color:#b4b4b4">());</span>
        </li>
      </ol>
    </div></p>
  </div></p>
</div>

<p align="left">
  En este cas asCircle y asButton son los dos Mixins. El primero añade una funcionalidad area a todos los objetos que tengan una propiedad llamada radius. El segundo añade un método click.
</p>

<p align="left">
  Para aplicar los Mixins sobre un objeto usamos la función call. No entraré en detalles de call ahora porque excede el objetivo de este post. Pero la clave está en que después de aplicar los Mixins, el objeto <em>a</em> tiene los métodos area y click.
</p>

<p align="left">
  Y más o menos… ¿con esto podemos dar por terminado el post no? Espero, que esto os haya ayudado a entender un poco mejor la POO bajo JavaScript y que bueno… os hayáis convencido de que JavaScript no tendrá clases pero orientado a objetos es 🙂
</p>

<p align="left">
  Saludos!
</p>

<p align="left">
  PD: Os paso el enlace del video de youtube donde podeis ver el hangout: <a href="http://www.desarrolloweb.com/en-directo/particularidades-programacion-orientada-objetos-poo-devio-8452.html">http://www.desarrolloweb.com/en-directo/particularidades-programacion-orientada-objetos-poo-devio-8452.html</a>
</p>

 [1]: https://twitter.com/javascriptio
 [2]: https://twitter.com/erickrdch
 [3]: https://twitter.com/amischol