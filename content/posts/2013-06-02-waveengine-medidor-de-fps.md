---
title: '[WaveEngine] Medidor de fps'
description: '[WaveEngine] Medidor de fps'
author: eiximenis

date: 2013-06-02T09:42:14+00:00
geeks_url: /?p=1643
geeks_visits:
  - 1830
geeks_ms_views:
  - 1192
categories:
  - Uncategorized

---
Bueno… sigo mi serie de posts sobre [WaveEngine][1]. En los dos primeros posts vimos como poner un sprite en pantalla y luego como animarlo. Ambos pasos (y algunos más sobre los que todavía no he comentado nada) están descritos en uno de los hand-on-labs de Wave: el [platform game sample][2].

Antes que nada el _disclaimer_ obligatorio: En todos esos posts sobre Wave, explico la manera que he encontrado _yo_ para hacer las cosas. Eso no significa que sea la mejor, la más óptima o incluso la correcta. Por el momento la documentación sobre Wave es bastante escueta. Algunos tutoriales, código fuente de algunos ejemplos y una referencia de todas las clases (que no sirve apenas para nada).

Bueno, aclarado esto, en este post voy a mostrar como crear un indicador que nos muestre, en todo momento, a cuantos frames por segundo se está ejecutando nuestro juego.

**Jerarquía de elementos de Wave**

En el primer post vimos que un juego en Wave tiene una jerarquía de elementos realmente sencilla:

  * Escena 
      * Entidades 
          * Componentes 

Básicamente una escena es un conjunto de entidades y cada entidad contiene uno o varios (generalmente varios) componentes. Los componentes son cualquier cosa que es susceptible de formar parte de una entidad. Algunos componentes, como p.ej. Sprite son meros contenedores de datos, pero hay dos tipos de componentes _especiales_:

  1. Behaviors: Son componentes de “lógica”. Disponen de un método Update() que Wave llama automáticamente en cada iteración del game-loop. Para implementarlos debe derivarse de Behavior. 
  2. Drawables: Son componentes de “dibujo”. Disponen de un método Draw() que Wave llama automáticamente en cada iteración del game-loop. Para implementarlos debe derivarse de Drawable2D o Drawable3D según sea el caso. 

Por supuesto es posible que un componente _dependa_ de otro componente. P. ej. el componente Sprite depende del componente Transform2D, es decir siempre que a una entidad se le agregue una instancia del primero, deberá agregársele también una instancia del segundo. Para indicar que un componente depende de otro, se usa el atributo RequiredComponentAttribute:

<div style="font-size: 10pt; font-family: consolas; background: #1e1e1e; color: #dcdcdc">
  <p style="margin: 0px">
    [<span style="color: white">RequiredComponent</span>]
  </p>
  
  <p style="margin: 0px">
    <span style="color: #569cd6">public</span> <span style="color: white">Transform2D</span> <span style="color: white">Transform2D</span>;
  </p></p>
</div>

Automáticamente Wave inyectará en la variable decorada con [RequiredComponent] el componente del tipo indicado (en este caso Transform2D) que esté añadido a la misma entidad.

**Opción 1: Usar un Drawable**

Para crear el monitor de fps vamos a crear un componente nuevo. Dado que debe dibujar en la pantalla (el valor de fps) el componente derivará de Drawable2D.

Por el mero hecho de derivar de Drawable2D obtenemos un método Draw que Wave nos llama en cada iteración del game-loop. Además nos pasa un parámetro que es un TimeSpan con el tiempo que ha pasado _desde la llamada anterior_. Eso es porque Wave **no se sincroniza** a ninguna velocidad en concreto. Es decir el game-loop de Wave básicamente:

  1. Llama al método Update() de todos los Behaviors 
  2. Llama al método Draw() de todos los Drawables 

Y eso lo hace continuamente y sin parar. Si queremos que nuestro videojuego vaya, al menos, a 30 fps, eso significa que la ejecución de todos los métodos Update y Draw de todos los componentes de todas las entidades de la escena debe tardar menos de 1/30 segundos. Cuanto menos tarde Wave en ejecutar todos componentes de la escena, nuestro juego irá a más fps. Por lo tanto, el fps no es un valor fijo si no que depende de la complejidad de la escena y del hardware que ejecuta el sistema. De ahí que Wave nos pase el TimeSpan para que podamos saber cuanto tiempo (cuantos milisegundos) han pasado desde la llamada anterior (este valor tampoco es constante ya que pueden aparecer o desaparecer entidades de la escena en cualquier momento).

Si derivamos de la clase Drawable2D veremos que existen DOS métodos Draw. Uno llamado Draw y otro llamado DrawBasicUnit que es abstracto y estamos obligados a implementar. Tras buscar en vano información sobre que diferencia había entre ambos métodos, al final me sumergí (con la ayuda de Resharper) en el código de Wave Engine y he llegado a la siguiente conclusión:

  * **El código de renderizado debe ir en DrawBasicUnit**. El parámetro que recibimos (un int llamado con el somero nombre de “parameter”) es el orden de nuestro Drawable. Es decir valdrá 0 si se ha invocado el primero, 1 si es el segundo, etc. 
  * En el método Draw() debe ir todo aquel código que dependa de saber el valor del TimeStamp que mencionaba antes. Pero NO pongas código de renderizado en él. 

Si tu Drawable no requiere sincronizarse con el tiempo (la gran mayoría no lo necesitan, ya que simplemente se dibujan cada vez) entonces no debes redefinir Draw(). Sólo debes implementar DrawBasicUnit.

Armado con esta suposición he codificado mi componente que muestra los fps tal y como sigue:

<div style="font-size: 10pt; font-family: consolas; background: #1e1e1e; color: #dcdcdc">
  <p style="margin: 0px">
    <span style="color: #569cd6">public</span> <span style="color: #569cd6">class</span> <span style="color: #4ec9b0">FpsMeasure</span> : <span style="color: #4ec9b0">Drawable2D</span>
  </p>
  
  <p style="margin: 0px">
    {
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: #569cd6">private</span> <span style="color: #569cd6">int</span> <span style="color: white">_fps</span>;
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: #569cd6">private</span> <span style="color: #569cd6">int</span> <span style="color: white">_currentSecondFps</span>;
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: #569cd6">private</span> <span style="color: #569cd6">int</span> <span style="color: white">_ms</span>;
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: #569cd6">private</span> <span style="color: #4ec9b0">SpriteFont</span> <span style="color: white">_font</span>;
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: #569cd6">private</span> <span style="color: #569cd6">static</span> <span style="color: #569cd6">int</span> <span style="color: white">_instances</span>;
  </p>
  
  <p style="margin: 0px">
    &#160;
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: #569cd6">public</span> <span style="color: white">FpsMeasure</span>()
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; : <span style="color: #569cd6">base</span>(<span style="color: #d69d85">"fps"</span> <span style="color: #b4b4b4">+</span> (<span style="color: white">_instances</span><span style="color: #b4b4b4">++</span>), <span style="color: #4ec9b0">DefaultLayers</span><span style="color: #b4b4b4">.</span><span style="color: white">Opaque</span>)
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; {
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; }
  </p>
  
  <p style="margin: 0px">
    &#160;
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: #569cd6">protected</span> <span style="color: #569cd6">override</span> <span style="color: #569cd6">void</span> <span style="color: white">Initialize</span>()
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; {
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #569cd6">base</span><span style="color: #b4b4b4">.</span><span style="color: white">Initialize</span>();
  </p>
  
  <p style="mar
gin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: white">_font</span> <span style="color: #b4b4b4">=</span> <span style="color: white">Assets</span><span style="color: #b4b4b4">.</span><span style="color: white">LoadAsset</span><span style="color: #b4b4b4"><</span><span style="color: #4ec9b0">SpriteFont</span><span style="color: #b4b4b4">></span>(<span style="color: #d69d85">"Content/Arial Rounded MT.wpk"</span>);
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; }
  </p>
  
  <p style="margin: 0px">
    &#160;
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: #569cd6">protected</span> <span style="color: #569cd6">override</span> <span style="color: #569cd6">void</span> <span style="color: white">Dispose</span>(<span style="color: #569cd6">bool</span> <span style="color: white">disposing</span>)
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; {
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; }
  </p>
  
  <p style="margin: 0px">
    &#160;
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: #569cd6">public</span> <span style="color: #569cd6">override</span> <span style="color: #569cd6">void</span> <span style="color: white">Draw</span>(<span style="color: #4ec9b0">TimeSpan</span> <span style="color: white">gameTime</span>)
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; {
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: white">_ms</span> <span style="color: #b4b4b4">+=</span> <span style="color: white">gameTime</span><span style="color: #b4b4b4">.</span><span style="color: white">Milliseconds</span>;
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: white">_currentSecondFps</span><span style="color: #b4b4b4">++</span>;
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #569cd6">if</span> (<span style="color: white">_ms</span> <span style="color: #b4b4b4">>=</span> <span style="color: #b5cea8">1000</span>)
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; {
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: white">_fps</span> <span style="color: #b4b4b4">=</span> <span style="color: white">_currentSecondFps</span>;
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: white">_currentSecondFps</span> <span style="color: #b4b4b4">=</span> <span style="color: #b5cea8"></span>;
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: white">_ms</span> <span style="color: #b4b4b4">=</span> <span style="color: #b5cea8"></span>;
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; }
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #569cd6">base</span><span style="color: #b4b4b4">.</span><span style="color: white">Draw</span>(<span style="color: white">gameTime</span>);
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; }
  </p>
  
  <p style="margin: 0px">
    &#160;
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: #569cd6">protected</span> <span style="color: #569cd6">override</span> <span style="color: #569cd6">void</span> <span style="color: white">DrawBasicUnit</span>(<span style="color: #569cd6">int</span> <span style="color: white">parameter</span>)
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; {
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #569cd6">if</span> (<span style="color: white">_fps</span> <span style="color: #b4b4b4">!=</span> <span style="color: #b5cea8"></span>)
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; {
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: white">spriteBatch</span><span style="color: #b4b4b4">.</span><span style="color: white">DrawStringVM</span>(<span style="color: white">_font</span>, <span style="color: white">_fps</span><span style="color: #b4b4b4">.</span><span style="color: white">ToString</span>(), <span style="color: #569cd6">new</span> <span style="color: #4ec9b0">Vector2</span>(<span style="color: #b5cea8">10</span>, <span style="color: #b5cea8">10</span>), <span style="color: #4ec9b0">Color</span><span style="color: #b4b4b4">.</span><span style="color: white">White</span>);
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; }
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; }
  </p>
  
  <p style="margin: 0px">
    }
  </p></p>
</div>

Aspectos a comentar de este código:

  1. En el constructor asignamos un nombre único a este componente (ya que todos los componentes deben tener uno). 
  2. En el Initialize() es el lugar donde podemos inicializar todo aquello que dependa de Wave. En este punto **los componentes requeridos por nuestro componente (y que hemos decorado con [RequiredComponent] ya tendrán valor**). En mi caso cargo la fuente de un asset que he creado 
  3. Para crear el asset con la fuente he usado el Assets Exporter, y he añadido una fuente (menú _Project –> Add Font_). Al exportar me genera el .wpk de la fuente que debo añadir al proyecto de VS2012.
  4. En el método Draw simplemente guardo en una variable (\_fps) los frames del segundo actual. Pero NO dibujo nada en la pantalla. Simplemente actualizo el valor de \_fps cada segundo. 
  5. En el método DrawBasicUnit es donde hay el código de dibujo: llamo al método DrawStringVM del objeto SpriteBatch que como Drawable tenemos. El método DrawStringVM básicamente dibuja un texto en la posición especificada. 

> **Nota importante**: Si redefines Draw ¡**no te olvides la llamada a base.Draw** o el método DrawBasicUnit no se llamará nunca!

Para usar nuestro componente debemos vincularlo a una entidad para poder añadirlo a la escena:

<div style="font-size: 10pt; font-family: consolas; background: #1e1e1e; color: #dcdcdc">
  <p style="margin: 0px">
    <span style="color: #569cd6">var</span> <span style="color: white">fps</span> <span style="color: #b4b4b4">=</span> <span style="color: #569cd6">new</span> <span style="color: #4ec9b0">Entity</span>(<span style="color: #d69d85">"fps"</span>)<span style="color: #b4b4b4">.</span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: white">AddComponent</span>(<span style="color: #569cd6">new</span> <span style="color: #4ec9b0">FpsMeasure</span>());
  </p>
  
  <p style="margin: 0px">
    <span style="color: white">EntityManager</span><span style="color: #b4b4b4">.</span><span style="color: white">Add</span>(<span style="color: white">fps</span>);
  </p></p>
</div>

¡Y listos! Con esto ya tenemos un contador que nos muestra los fps a los que se está ejecutando nuestro juego.

**Opción 2: Usar un Drawable y un Behavior**

El Drawable que hemos creado tiene cierta lógica en el método Draw: calcular los fps. Para mostrar la naturaleza modular de Wave y la interdependencia entre Componentes, vamos a mover esta lógica y colocarla en un Behavior. El Drawable quedará tan solo con el código de renderizado.

Empezamos quitando todo el código de cálculo de los frames por segundo del Drawable y dejando solo el de renderizado e inicialización de la fuente:

<div style="font-size: 10pt; font-family: consolas; background: #1e1e1e; color: #dcdcdc">
  <p style="margin: 0px">
    <span style="color: #569cd6">public</span> <span style="color: #569cd6">class</span> <span style="color: #4ec9b0">FpsMeasure</span> : <span style="color: #4ec9b0">Drawable2D</span>
  </p>
  
  <p style="margin: 0px">
    {
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: #569cd6">private</span> <span style="color: #4ec9b0">SpriteFont</span> <span style="color: white">_font</span>;
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: #569cd6">private</span> <span style="color: #569cd6">stati<br /> c</span> <span style="color: #569cd6">int</span> <span style="color: white">_instances</span>;
  </p>
  
  <p style="margin: 0px">
    &#160;
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: #569cd6">public</span> <span style="color: #569cd6">int</span> <span style="color: white">FpsValue</span> { <span style="color: #569cd6">get</span>; <span style="color: #569cd6">set</span>; }
  </p>
  
  <p style="margin: 0px">
    &#160;
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: #569cd6">public</span> <span style="color: white">FpsMeasure</span>()
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; : <span style="color: #569cd6">base</span>(<span style="color: #d69d85">"fps"</span> <span style="color: #b4b4b4">+</span> (<span style="color: white">_instances</span><span style="color: #b4b4b4">++</span>), <span style="color: #4ec9b0">DefaultLayers</span><span style="color: #b4b4b4">.</span><span style="color: white">Opaque</span>)
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; {
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; }
  </p>
  
  <p style="margin: 0px">
    &#160;
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: #569cd6">protected</span> <span style="color: #569cd6">override</span> <span style="color: #569cd6">void</span> <span style="color: white">Initialize</span>()
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; {
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #569cd6">base</span><span style="color: #b4b4b4">.</span><span style="color: white">Initialize</span>();
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: white">_font</span> <span style="color: #b4b4b4">=</span> <span style="color: white">Assets</span><span style="color: #b4b4b4">.</span><span style="color: white">LoadAsset</span><span style="color: #b4b4b4"><</span><span style="color: #4ec9b0">SpriteFont</span><span style="color: #b4b4b4">></span>(<span style="color: #d69d85">"Content/Arial Rounded MT.wpk"</span>);
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; }
  </p>
  
  <p style="margin: 0px">
    &#160;
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: #569cd6">protected</span> <span style="color: #569cd6">override</span> <span style="color: #569cd6">void</span> <span style="color: white">Dispose</span>(<span style="color: #569cd6">bool</span> <span style="color: white">disposing</span>)
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; {
  </p>
  
  <p style="margin: 0px">
    &#160;
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; }
  </p>
  
  <p style="margin: 0px">
    &#160;
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: #569cd6">protected</span> <span style="color: #569cd6">override</span> <span style="color: #569cd6">void</span> <span style="color: white">DrawBasicUnit</span>(<span style="color: #569cd6">int</span> <span style="color: white">parameter</span>)
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; {
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #569cd6">if</span> (<span style="color: white">FpsValue</span> <span style="color: #b4b4b4">!=</span> <span style="color: #b5cea8"></span>)
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; {
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: white">spriteBatch</span><span style="color: #b4b4b4">.</span><span style="color: white">DrawStringVM</span>(<span style="color: white">_font</span>, <span style="color: white">FpsValue</span><span style="color: #b4b4b4">.</span><span style="color: white">ToString</span>(), <span style="color: #569cd6">new</span> <span style="color: #4ec9b0">Vector2</span>(<span style="color: #b5cea8">10</span>, <span style="color: #b5cea8">10</span>), <span style="color: #4ec9b0">Color</span><span style="color: #b4b4b4">.</span><span style="color: white">White</span>);
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; }
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; }
  </p>
  
  <p style="margin: 0px">
    }
  </p></p>
</div>

Fíjate como ahora ya no hay necesidad de redefinir Draw, ya que tenemos un Drawable que básicamente lo único que hace es dibujar el valor de la propiedad FpsValue.

Ahora nos toca crear el Behavior. Así pues creamos una clase que derive de Behavior y en el método Update() ponemos el código que teníamos antes en el método Draw():

<div style="font-size: 10pt; font-family: consolas; background: #1e1e1e; color: #dcdcdc">
  <p style="margin: 0px">
    <span style="color: #569cd6">public</span> <span style="color: #569cd6">class</span> <span style="color: #4ec9b0">FpsMeasureBehavior</span> : <span style="color: #4ec9b0">Behavior</span>
  </p>
  
  <p style="margin: 0px">
    {
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: #569cd6">private</span> <span style="color: #569cd6">int</span> <span style="color: white">_currentSecondFps</span>;
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: #569cd6">private</span> <span style="color: #569cd6">int</span> <span style="color: white">_ms</span>;
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: #569cd6">private</span> <span style="color: #569cd6">static</span> <span style="color: #569cd6">int</span> <span style="color: white">_instances</span>;
  </p>
  
  <p style="margin: 0px">
    &#160;
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: #569cd6">public</span> <span style="color: white">FpsMeasureBehavior</span>()
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; : <span style="color: #569cd6">base</span>(<span style="color: #d69d85">"fpsBehavior"</span> <span style="color: #b4b4b4">+</span> (<span style="color: white">_instances</span><span style="color: #b4b4b4">++</span>))
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; {
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; }
  </p>
  
  <p style="margin: 0px">
    &#160;
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; [<span style="color: #4ec9b0">RequiredComponent</span>]
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: #569cd6">public</span> <span style="color: #4ec9b0">FpsMeasure</span> <span style="color: white">fpsDrawable</span>;
  </p>
  
  <p style="margin: 0px">
    &#160;
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: #569cd6">protected</span> <span style="color: #569cd6">override</span> <span style="color: #569cd6">void</span> <span style="color: white">Update</span>(<span style="color: #4ec9b0">TimeSpan</span> <span style="color: white">gameTime</span>)
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; {
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: white">_ms</span> <span style="color: #b4b4b4">+=</span> <span style="color: white">gameTime</span><span style="color: #b4b4b4">.</span><span style="color: white">Milliseconds</span>;
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: white">_currentSecondFps</span><span style="color: #b4b4b4">++</span>;
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #569cd6">if</span> (<span style="color: white">_ms</span> <span style="color: #b4b4b4">>=</span> <span style="color: #b5cea8">1000</span>)
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; {
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: white">fpsDrawable</span><span style="color: #b4b4b4">.</span><span style="color: white">FpsValue</span> <span style="color: #b4b4b4">=</span> <span style="color: white">_currentSecondFps</span>;
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: white">_currentSecondFps</span> <span style="color: #b4b4b4">=</span> <span style="color: #b5cea8"></span>;
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: white">_ms</span> <span sty
le="color: #b4b4b4">=</span> <span style="color: #b5cea8"></span>;
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; }
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; }
  </p>
  
  <p style="margin: 0px">
    }
  </p></p>
</div>

Observa el uso de [RequiredComponent] para indicar que este Behavior requiere del componente FpsMeasure (el Drawable) para su funcionamiento. Si creásemos una entidad y le añadiésemos este Behavior, pero no le añadiésemos un FpsMeasure, Wave nos daría un error:

[<img title="SNAGHTMLa191587" style="border-left-width: 0px; border-right-width: 0px; border-bottom-width: 0px; display: inline; border-top-width: 0px" border="0" alt="SNAGHTMLa191587" src="http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/SNAGHTMLa191587_5F00_thumb_5F00_30923E2E.png" width="456" height="375" />][3]

¡Listos! Ahora ya tenemos nuestro contador de fps implementado a través de un Behavior (que proporciona la lógica) y un Drawable (que se limita a dibujar datos). Por supuesto, para que funcione hemos de crear una entidad con ambos componentes:

<div style="font-size: 10pt; font-family: consolas; background: #1e1e1e; color: #dcdcdc">
  <p style="margin: 0px">
    <span style="color: #569cd6">var</span> <span style="color: white">fps</span> <span style="color: #b4b4b4">=</span> <span style="color: #569cd6">new</span> <span style="color: #4ec9b0">Entity</span>(<span style="color: #d69d85">"fps"</span>)<span style="color: #b4b4b4">.</span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: white">AddComponent</span>(<span style="color: #569cd6">new</span> <span style="color: #4ec9b0">FpsMeasure</span>())<span style="color: #b4b4b4">.</span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: white">AddComponent</span>(<span style="color: #569cd6">new</span> <span style="color: #4ec9b0">FpsMeasureBehavior</span>());
  </p></p>
</div>

**¿Y cual es mi opción preferida?**

La segunda, sin duda alguna. Porque es mucho modular, porque separa mejor las responsabilidades y porque permite reaprovechar más los componentes. Fíjate que realmente el Drawable FpsMeasure, se comporta en realidad como una label (se limita a mostrar texto sin&#160; importarle de donde viene este texto), así que con poco trabajo sería convertible a un componente más genérico (p. ej. TextSprite o algo así) y ser reutilizado en otros sitios.

En fin, espero que os haya resultado interesante 😉

Saludos!

 [1]: http://www.waveengine.net/
 [2]: http://blog.waveengine.net/2013/04/12/platform-game-sample/
 [3]: http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/SNAGHTMLa191587_5F00_448BD0C4.png