---
title: 'C#–Buenas prácticas en constructores'

author: eiximenis

date: 2016-07-21T07:14:31+00:00
geeks_url: /?p=1781
geeks_ms_views:
  - 3489
categories:
  - 'C#'

---
Escribir el constructor de una clase es algo que parece trivial… A fin de cuentas, el constructor se encarga de construir un objeto, ¿no? Pero la realidad es que escribir constructores no es tan sencillo como parece. ¿Qué significa “construir” un objeto? Por supuesto cada clase tendrá sus propias necesidades, pero hay una serie de guías y buenas prácticas que nos pueden ayudar a tomar ciertas decisiones. A esto va dedicado este post.

<!--more-->

**Guia 1: Haz lo mínimo posible en el constructor**

Sean cuales sean las necesidades de tu clase, haz que el constructor haga lo mínimo posible. Técnicamente un constructor _no debería poder fallar casi nunca_. De hecho el constructor debería limitarse a guardarse los parámetros necesarios y poca cosa más. **Evita constructores que hagan demasiadas cosas**. Veamos dos ejemplos contrapuestos. El primero la clase **FileStream**. El siguiente código:

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:e67962c3-9810-4637-8ced-7c5f189469e3" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
    <div style="background: #ddd; max-height: 300px; overflow: auto">
      <ol start="1" style="background: #ffffff; margin: 0 0 0 2em; padding: 0 0 0 5px; white-space: nowrap">
        <li>
          <span style="color:#000000"></span><span style="color:#0000ff">using</span><span style="color:#000000"> (</span><span style="color:#0000ff">var</span><span style="color:#000000"> fs = </span><span style="color:#0000ff">new</span><span style="color:#000000"> </span><span style="color:#2b91af">FileStream</span><span style="color:#000000">(</span><span style="color:#800000">@"C:&#92;&#92;foo.txt"</span><span style="color:#000000">, </span><span style="color:#2b91af">FileMode</span><span style="color:#000000">.Open, </span><span style="color:#2b91af">FileAccess</span><span style="color:#000000">.Read))</span>
        </li>
        <li>
          <span style="color:#000000">{</span>
        </li>
        <li>
          <span style="color:#000000">}</span>
        </li>
      </ol>
    </div></p>
  </div></p>
</div>

Falla si no existe el fichero C:\foo.txt o si el usuario no tiene permisos, o si dicho fichero existe pero está bloqueado. El constructor intenta abrir el fichero, una operación que por un lado es potencialmente larga (el fichero puede estar en una carpeta remota) y por otra tiene muchas posibilidades de fallo. **El constructor de FileStream hace demasiadas cosas**. Hubiese sido mejor si hubiesen seguido otra aproximación. P. ej. la de SqlConnection. El siguiente código no falla nunca:

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:0157e1c3-d318-4ecb-89ff-3a4aa132e8a2" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
    <div style="background: #ddd; max-height: 300px; overflow: auto">
      <ol start="1" style="background: #ffffff; margin: 0 0 0 2em; padding: 0 0 0 5px; white-space: nowrap">
        <li>
          <span style="color:#000000"></span><span style="color:#0000ff">using</span><span style="color:#000000"> (</span><span style="color:#0000ff">var</span><span style="color:#000000"> con = </span><span style="color:#0000ff">new</span><span style="color:#000000"> </span><span style="color:#2b91af">SqlConnection</span><span style="color:#000000">(</span><span style="color:#a31515">"server=100.0.0.1;database=myDb;uid=myUser;password=myPass;"</span><span style="color:#000000">))</span>
        </li>
        <li>
          <span style="color:#000000">{</span>
        </li>
        <li>
          <span style="color:#000000">}</span>
        </li>
      </ol>
    </div></p>
  </div></p>
</div>

Da absolutamente igual que el servidor exista o no, o que la base de datos exista, o que el usuario tenga permisos. El constructor no intenta establecer conexión alguna y delega operación en un método de instancia (Open). Eso tiene varias ventajas:

  1. Crear objetos SqlConnection es una tarea sencilla y asegura al desarrollador que el tiempo en hacerlo es corto y que los fallos son inexistentes. 
      * El desarrollador puede retardar la llamada a la operación que puede fallar y/o tardar tiempo todo lo necesario. 
          * **Se puede proporcionar una versión asíncrona** (la propia clase SqlConnection define OpenAsync), lo que no es posible si el constructor realizara esas operaciones (no hay constructores asíncronos).</ol> 
        En resúmen **FileStream** debería proporcionar un método Open en lugar de intentar abrir el fichero desde el constructor. Al no hacerlo, implica que el desarrollador debe diferir la creación entera del objeto lo más tarde posible (en lugar de diferir solo la operación potencialmente costosa), lo que obliga a lidiar con posibles referencias null y/o usar otras técnicas como Lazy<T>.
        
        **Guía 2 –No llames métodos virtuales desde el constructor**
        
        O ándate con ojo si lo haces. Eso es de hecho un _warning_ del compilador, así que deberíamos prestarle atención. La razón es que, dada una clase A, que define un método virtual m() y una clase B que hereda de A y redefine el método virtual, cuando el constructor de A llame al método m(), el método llamado no tiene porque ser el método de la clase A, si no que puede ser el método de la clase derivada (B), que se ejecutará antes que el propio constructor de la clase derivada.
        
        Reconozco que explicado así puede costar de entender, así que mejor verlo con un ejemplo. Empecemos por la clase A:
        
        <div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:6f628e59-1f69-43ba-8c73-ae086d00f23e" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
          <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
            <div style="background: #ddd; max-height: 300px; overflow: auto">
              <ol start="1" style="background: #ffffff; margin: 0 0 0 2.5em; padding: 0 0 0 5px; white-space: nowrap">
                <li>
                  <span style="color:#000000"></span><span style="color:#0000ff">class</span><span style="color:#000000"> </span><span style="color:#2b91af">A</span>
                </li>
                <li>
                  <span style="color:#000000">{</span>
                </li>
                <li>
                      <span style="color:#000000"></span><span style="color:#0000ff">public</span><span style="color:#000000"> A()</span>
                </li>
                <li>
                      <span style="color:#000000">{</span>
                </li>
                <li>
                          <span style="color:#000000">m();</span>
                </li>
                <li>
                      <span style="color:#000000">}</span>
                </li>
                <li>
                      <span style="color:#000000"></span><span style="color:#0000ff">protected</span><span style="color:#000000"> </span><span style="color:#0000ff">virtual</span><span style="color:#000000"> </span><span style="color:#0000ff">void</span><span style="color:#000000"> m()</span>
                </li>
                <li>
                      <span style="color:#000000">{</span>
                </li>
                <li>
                      <span style="color:#000000">}</span>
                </li>
                <li>
                  <span style="color:#000000">}</span>
                </li>
              </ol>
            </div></p>
          </div></p>
        </div>
        
        El constructor de A, llama al método virtual m(). En este caso no hace nada más, en un caso real, probablemente el constructor de A haría algo antes y algo después y se quiere dar la opción a las clases derivadas de personalizar parte del comportamiento. Da igual, no es relevante. Veamos ahora la clase B:
        
        <div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:aca6b234-1ff8-437c-a5e1-f51b913f27a4" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
          <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
            <div style="background: #ddd; max-height: 300px; overflow: auto">
              <ol start="1" style="backgro
und: #ffffff; margin: 0 0 0 2.5em; padding: 0 0 0 5px; white-space: nowrap">
                </p> 
                
                <li>
                  <span style="color:#000000"></span><span style="color:#0000ff">class</span><span style="color:#000000"> </span><span style="color:#2b91af">B</span><span style="color:#000000"> : </span><span style="color:#2b91af">A</span>
                </li>
                <li>
                  <span style="color:#000000">{</span>
                </li>
                <li>
                      <span style="color:#000000"></span><span style="color:#0000ff">private</span><span style="color:#000000"> </span><span style="color:#0000ff">string</span><span style="color:#000000"> _name;</span>
                </li>
                <li>
                      <span style="color:#000000"></span><span style="color:#0000ff">private</span><span style="color:#000000"> </span><span style="color:#0000ff">int</span><span style="color:#000000"> _len;</span>
                </li>
                <li>
                      <span style="color:#000000"></span><span style="color:#0000ff">public</span><span style="color:#000000"> B(</span><span style="color:#0000ff">string</span><span style="color:#000000"> name) </span>
                </li>
                <li>
                      <span style="color:#000000">{</span>
                </li>
                <li>
                          <span style="color:#000000">_name = name ?? </span><span style="color:#a31515">""</span><span style="color:#000000">;</span>
                </li>
                <li>
                      <span style="color:#000000">}</span>
                </li>
                <li>
                      <span style="color:#000000"></span><span style="color:#0000ff">protected</span><span style="color:#000000"> </span><span style="color:#0000ff">override</span><span style="color:#000000"> </span><span style="color:#0000ff">void</span><span style="color:#000000"> m()</span>
                </li>
                <li>
                      <span style="color:#000000">{</span>
                </li>
                <li>
                          <span style="color:#000000"></span><span style="color:#0000ff">base</span><span style="color:#000000">.m();</span>
                </li>
                <li>
                          <span style="color:#000000">_len = _name.Length;</span>
                </li>
                <li>
                      <span style="color:#000000">}</span>
                </li>
                <li>
                  <span style="color:#000000">}</span>
                </li>
              </ol>
            </div></p>
          </div></p>
        </div>
        
        La clase B hereda de A y redefine m. A priori no parece que haya nada incorrecto en el código, ¿verdad? Hasta que hacemos:
        
        <div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:a95afb19-a600-46dd-9418-07a07dce3b21" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
          <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
            <div style="background: #ddd; max-height: 300px; overflow: auto">
              <ol start="1" style="background: #ffffff; margin: 0 0 0 2em; padding: 0 0 0 5px; white-space: nowrap">
                <li>
                  <span style="color:#0000ff">var</span><span style="color:#000000"> b = </span><span style="color:#0000ff">new</span><span style="color:#000000"> </span><span style="color:#2b91af">B</span><span style="color:#000000">(</span><span style="color:#a31515">"hola"</span><span style="color:#000000">);</span>
                </li>
              </ol>
            </div></p>
          </div></p>
        </div>
        
        Y obtenemos una _NullReferenceException_ en el método m() de B. ¿Y eso? Pues muy sencillo:
        
          * Estamos creando un objeto de tipo B 
              * La clase B hereda de A, por lo que primero se ejecuta el constructor de la clase base (A) antes que el constructor propio (B) 
                  * El constructor de la clase A llama al método m que es virtual y por lo tanto se ejecuta el método basándose en el tipo de objeto. El objeto es de tipo B por lo que se ejecuta el método m() de la clase B a pesar de estar llamado desde el constructor de la clase A. 
                      * El método m() de la clase B accede a una propiedad \_name, que se inicializa en el constructor de la clase B, todavía no ejecutado. Así que \_name vale null y obtenemos nuestra excepción.</ul> 
                    De hecho, de nuevo, llamar a métodos virtuales desde un constructor suele ser indicación de que, quizá, el constructor hace demasiadas cosas.
                    
                    **¿Constructores o métodos estáticos?**
                    
                    Esa es una muy buena pregunta. No tengo una respuesta contundente, solo aspectos que podemos considerar:
                    
                      1. Un método estático puede devolver null en lugar de tener que lanzar una excepción. 
                          * Un método estático puede devolver instancias previamente creadas (aplicar memoización, especialmente en el caso de objetos inmutables). 
                              * Un método estático puede devolver un objeto de un subtipo si es necesario. 
                                  * Un método estático puede tener cualquier nombre, lo que puede hacer el código más legible. 
                                      * Al ver “new” queda claro que se crea un objeto. Llamando a un método estático puede ser más confuso (¿se está creando realmente un objeto?).</ol> 
                                    Hay gente que cuando el constructor va más allá de hacer algo sencillo prefieren usar un método estático. P. ej. el método File.OpenRead, devuelve un FileStream configurado para leer un fichero. Vale, en el fondo se limita a llamar al constructor de FileStream con unos determinados parámetros (aunque no es el caso File.OpenRead podría hacer otras cosas como devolver null si el fichero no existe, en lugar de propagar la excepción lanzada por el constructor de FileStream). La clave ahí está en que al ser File.OpenRead un método, uno puede esperar una semántica más compleja que la que pueda esperar de un constructor. Es decir, usar un método estático es una manera de decir “hey, eso crea un objeto, pero lo hace de una forma que es más compleja que la habitual”. También permite agrupar la creación y la inicialización (p. ej. un método estático SqlConnection.OpenNew() podría crear y abrir una conexión, todo a la vez).
                                    
                                    Veamos un ejemplo (**está forzado**, luego cuento el por qué, pero como ejemplo nos servirá): System.Guid. Uno puede esperar que para crear un Guid baste con hacer:
                                    
                                    <div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:052b34e4-0a0b-4f37-bc66-07f83b422139" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
                                      <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
                                        <div style="background: #ddd; max-height: 300px; overflow: auto">
                                          <ol start="1" style="background: #ffffff; margin: 0 0 0 2em; padding: 0 0 0 5px; white-space: nowrap">
                                            <li>
                                              <span style="color:#0000ff">var</span><span style="color:#000000"> guid = </span><span style="color:#0000ff">new</span><span style="color:#000000"> </span><span style="color:#2b91af">Guid</span><span style="color:#000000">();</span>
                                            </li>
                                          </ol>
                                        </div></p>
                                      </div></p>
                                    </div>
                                    
                                    Pero la realidad es que con esto obtenemos el Guid vacío, con valor igual a cero, que no es muy útil (generalmente queremos que los Guids sean identificadores únicos). Para conseguir un Guid único debemos usar un método estático:
                                    
                                    <div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:129c04c7-2d98-4410-b220-c5cb7776806a" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
                                      <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
                                        <div style="background: #ddd; max-height: 300px; overflow: auto">
                                          <ol start="1" style="background: #ffffff; margin: 0 0 0 2em; padding: 0 0 0 5px; white-space: nowrap">
                                            <li>
                                              <span style="color:#0000ff">var</span><span style="color:#000000"> guid = </span><span style="color:#2b91af">Guid</span><span style="color:#000000">.NewGuid();</span>
                                            </li>
                                          </ol>
                                        </div></p>
                                      </div></p>
                                    </div>
                                    
                                    No se me ocurre ninguna razón por la cual este comportamiento no podría ser el del constructor por defecto (y más cuando tenemos Guid.Empy para obtener el Guid vacío). Cuando creamos un Guid raramente queremos el Guid vacío, queremos eso, un Guid único. Crear un Guid único no es una operación costosa ni que a priori deba fallar: basta con inicializar con cierto algoritmo los valores del Guid. **Antes he dicho que este ejemplo estaba forzado y es que realmente hay una razón de peso por la que System.Guid se comporte así y es que es una struct, no una clase**. La
  
                                    s structs tienen siempre un constructor por defecto que no se puede redefinir, de ahí que **realmente los creadores de System.Guid no tenían otra opción**. Es una **limitación del lenguaje** lo que les ha obligado a esa aproximación, pero bueno… me ha servido como ejemplo. Es uno de esos casos en que una limitación del lenguaje afecta al diseño de un tipo 😉
                                    
                                    Personalmente, si hay muchas maneras de crear un objeto, a partir de distintos tipos de parámetros prefiero tener varios métodos estáticos antes que muchas sobrecargas del constructor. Un ejemplo de esto en el framework lo tenemos con la clase (realmente struct, pero ahora sí que no importa) DateTime. DateTime tiene 12 constructores, pero realmente esos 12 constructores son “dos” constructores que tienen todos los parámetros opcionales (de ahí las sobrecargas). Podemos agrupar los 12 constructores en:
                                    
                                      * 10 constructores que nos permiten crear un DateTime a partir de un áño, mes, día, hora, minuto, segundo y calendario. 
                                          * 2 constructores que nos permiten crear un DateTime a partir de unos _ticks_ y un calendario</ul> 
                                        Además de estos “dos” constructores tenemos varios métodos estáticos en DateTime tales como FromBinary o FromFileTime para obtener un DateTime a partir de otros elementos. Nos podemos preguntar por qué para crear un DateTime a partir de _ticks_ se usa el constructor y para hacerlo a partir del tiempo de un fichero lo hacemos usando un método estático. Hay, de hecho, una razón técnica: tanto los _ticks_ y el tiempo de un fichero es un long. Obviamente no podemos tener dos constructores que ambos acepten solo un long, así que los diseñadores de la clase han optado por el constructor en un caso (el que, probablemente, consideran más “normal”) y en un método estático en el otro. ¿Mi opinión? He dicho antes que los 12 constructores de DateTime son realmente dos, que definen dos maneras de crear un DateTime (a partir de año, mes, día y demás y a partir de _ticks_). Yo, quizá, hubiese eliminado los dos constructores que usan _ticks_ y hubiese creado un método estático FromTicks. ¿Por qué? Pues porque intento que **el constructor defina la forma canónica (normal, habitual) de crear un tipo**. Por supuesto alguien puede considerar que tan habitual es crear un DateTime a partir de _ticks_ como usando años, meses y demás. Bajo este punto de vista, no me parece mal que ambos mecanismos sean los constructores. Pero sí intento esto: que el constructor defina la forma habitual de crear un objeto. Si hay otros mecanismos, adicionales, prefiero que estén en métodos estáticos.
                                        
                                        Por supuesto, esto es solo una opinión 🙂
                                        
                                        **¿Cuantos parámetros debe tener el constructor?**
                                        
                                        Para responder a esta pregunta creo que debemos distinguir si los parámetros recibidos son dependencias del objeto o meramente lo describen. P. ej. podríamos suponer una clase SolidRectangle con un constructor que aceptase 9 parámetros: x, y, altura, anchura, color de relleno, color de línea, estilo de relleno, estilo de línea y transparencia. ¿Esos 9 parámetros son demasiados? En este caso, realmente, esos 9 parámetros se limitan a describir el rectángulo como tal. Si 9 parámetros en el constructor nos parecen demasiados, tenemos otras aproximaciones:
                                        
                                          1. Sustituir algunos (o todos) de esos parámetros por propiedades. Pero con esto perdemos la posibilidad de que los objetos SolidRectangle sean inmutables. Muchas veces la inmutabilidad es una característica deseable. 
                                              * Agrupar esos parámetros en un objeto tipo “SolidRectangleProps” que, básicamente, contiene esas mismas propiedades (o casi todas ellas). Esta aproximación reduce en efecto los parámetros del constructor de 9 a quizá 1, pero realmente no ha cambiado nada sustancial. Si “SolidRectangleProps” solo se usa para crear objetos SolidRectangle realmente no hemos ganada nada. Otra cosa es si a partir de un SolidRectangle puedo extraer sus “SolidRectangleProps” y usar este objeto para crear otros SolidRectangle o incluso otro tipo de figuras.</ol> 
                                            Lo importante es tener claro que un número elevado de parámetros en el constructor no es malo “per se” y agruparlos en un objeto tampoco tiene por qué aportar nada concreto. Otra cosa es si esos parámetros son _dependencias_ del objeto. En este caso, si tenemos muchas dependencias es un síntoma de que nuestra clase puede estar rompiendo el SRP. Así, p. ej. si un controlador de ASP.NET MVC recibe 9 parámetros, es para revisarlo (los controladores no se “describen” así que esos 9 parámetros seguro que son dependencias). Este es un error en el que se cae muy frecuentemente cuando se usa inyección de dependencias.
                                            
                                            Y nada, eso es todo… ¡Espero que esas ideas y reflexiones te hayan resultado útiles!
                                            
                                            Saludos! 😉