---
title: '[C# Básico] ¿Que es la herencia?'
description: '[C# Básico] ¿Que es la herencia?'
author: eiximenis

date: 2010-07-14T08:58:31+00:00
geeks_url: /?p=1526
geeks_visits:
  - 7513
geeks_ms_views:
  - 5455
categories:
  - Uncategorized

---
Hola a todos! Despues de la buena acogida que tuvo la [primera entrega de C# Básico (dedicada a las interfaces)][1], me gustaría abordar hoy una de las cuestiones que se pusieron en los comentarios: _¿Qué es la herencia?_ De nuevo os recuerdo que esta serie _es vuestra_: no tengáis reparos en pedir posts de algún tema en concreto o aclaraciones e intentaré contestaros dentro de mis conocimientos 🙂

En la [wikipedia se define herencia][2] como: _<font color="#0000a0">En orientación a objetos la herencia es el mecanismo fundamental para implementar la reutilización y extensibilidad del software. A través de ella los diseñadores pueden construir nuevas clases partiendo de una jerarquía de clases ya existente (comprobadas y verificadas) evitando con ello el rediseño, la remodificación y verificación de la parte ya implementada. La herencia facilita la creación de objetos a partir de otros ya existentes, obteniendo características (métodos y atributos) similares a los ya existentes.</font>_

MMMmm… no se que queréis que os diga: a mi no me queda muy claro que significa exactamente esta definición de herencia, así que vamos a buscar una que, aunque quizá no sea tan _correcta_ sea, al menos, más entenedora. Pero antes aclaremos…

**¿Que es una clase?**

En muchos escritos sobre [POO][3] se define clase como una abstracción de un objeto, una plantilla para crear objetos, una definición en base a la que crear objetos, etc… Esto nos lleva a buscar la definición de objeto, para encontrarnos, muchas veces, que se nos define como _la instancia de una clase_. Pues vaya.

Una clase representa **un concepto**. Cuando desarrollamos un programa lo hacemos para solucionar una cierta necesidad de negocio, y por lo tanto debemos modelar los conceptos de negocio necesarios. Cada **concepto** que modelemos es una clase. Si p.ej. estamos creando un software para la creación de mesas, el concepto de mesa seguramente tendrá sentido y nos aparecerá la clase _Mesa_. Si estamos creando un software de dibujo, serán otros conceptos los que necesitaremos y así nos aparecerán la clase _Círculo_ o _Rectángulo_. Las clases no sólo representan conceptos de negocio, también conceptos “más técnicos” como pueden ser un fichero de log, o un gestor de transacciones. Entonces, si las clases representan conceptos…

**Relaciones entre clases son relaciones entre conceptos**

Lógico: si las clases representan conceptos (ideas) las relaciones entre clases son relaciones entre conceptos. Y que relación puede haber entre dos conceptos? Pues **que un concepto represente una idea _más específica_ de otro concepto** (o al revés que un concepto represente una idea más _general_ que otro).

P. ej. todos estaremos de acuerdo que el concepto de _Mamífero_ es más específico que el concepto de _Animal_ o que el concepto de _Coche_ es más específico que el concepto de _Vehículo_. Pues bien, esta relación, en a la programación orientada a objetos se traduce en la relación de **generalización** (en este caso Vehículo es la clase general, clase base o superclase mientras que Coche es la clase derivada, clase hija o subclase)… sí, en POO nos gusta poner nombres raros a las cosas 🙂

Una relación de generalización trae varias consecuencias y una de ellas es precisamente **la herencia:** la clase derivada **hereda** todos los miembros (campos, propiedades y métodos) de la clase base.

Eso tiene su lógica, imaginad una clase Vehículo tal que:

<div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
  <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px" id="codeSnippet"><span style="color: #0000ff">class</span> Vehiculo<br />{<br />   <span style="color: #0000ff">public</span> <span style="color: #0000ff">int</span> Velocidad { get { ... } }<br />   <span style="color: #0000ff">public</span> <span style="color: #0000ff">void</span> Acelerar (<span style="color: #0000ff">int</span> delta) { ...}<br />   <span style="color: #0000ff">public</span> <span style="color: #0000ff">void</span> Frenar (<span style="color: #0000ff">int</span> vf) { ... }<br />   <span style="color: #0000ff">public</span> <span style="color: #0000ff">void</span> Girar (<span style="color: #0000ff">int</span> angulos) { ... }<br />}</pre>
  
  <p>
    </div> 
    
    <p>
      La clase Vehiculo representa un concepto de algo que tiene una determinada <em>Velocidad</em> y que puede Acelerar, Frenar y Girar. Ahora bien, si nos aparece el concepto de Coche y lo modelamos como una nueva clase, es de esperar que un Coche también tenga una Velocidad, pueda Acelerar, pueda Frenar y pueda Girar… Para que vamos a codificar de nuevo todos esos métodos si ya los hemos definido (codificado, recordad que una clase <em>implementa</em> un concepto) para Vehiculo?
    </p>
    
    <p>
      Este es un punto importante: la herencia <strong>no es herencia sólo de interfaz sinó también de implementación</strong>. Es decir, si tengo mi clase:
    </p>
    
    <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
      <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px" id="codeSnippet"><span style="color: #0000ff">class</span> Coche : Vechiculo<br />{<br />   <span style="color: #0000ff">public</span> <span style="color: #0000ff">string</span> Marca { get { ... } }<br />   <span style="color: #0000ff">public</span> <span style="color: #0000ff">string</span> Modelo { get { ... } }<br />}</pre>
      
      <p>
        </div> 
        
        <p>
          Un Coche es un Vehículo que además tiene una Marca y un Modelo. Así yo puedo hacer:
        </p>
        
        <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
          <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px" id="codeSnippet">Coche c = <span style="color: #0000ff">new</span> Coche();<br />c.Acelerar(100);<br /><span style="color: #008000">// A que velocidad va mi coche?</span><br /><span style="color: #0000ff">int</span> velocidad  = c.Velocidad;<br /><span style="color: #008000">// Es un SEAT?</span><br /><span style="color: #0000ff">if</span> (c.Marca == <span style="color: #006080">"SEAT"</span>) { ... }<br /></pre>
          
          <p>
            </div> 
            
            <p>
              Fijaos que puedo llamar a la propiedad <em>Marca</em> (definida en la propia clase Coche) para evaluar de que marca es mi coche, pero también puedo llamar al método <em>Acelerar</em> y a la propiedad <em>Velocidad</em>…
            </p>
            
            <p>
              Así pues la clase <em>Coche</em> obtiene “una copia” de <strong>todos los métodos y propiedades de la clase de la cual deriva (su clase base) y además puede añadir métodos o propiedades nuevos.</strong> Eso es, ni más ni menos lo que entendemos por herencia.
            </p>
            
            <p>
              Es importante que os quedéis con la idea de que la clase derivada obtiene <em>una copia</em> de los métodos y propiedades heredados porque así podréis entender el…
            </p>
            
            <p>
              <strong>Problema de la herencia múltiple</strong>
            </p>
            
            <p>
              C# (al igual que Java) es un lenguaje con <strong>herencia simple</strong>. Eso significa que una clase <strong>sólo puede derivar de una clase base a la vez</strong>. P.ej. esto en C# <strong>no es válido</strong>:
            </p>
            
            <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
              <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px" id="codeSnippet"><span style="color: #0000ff">class</span> VehiculoAereo { ... }<br /><span style="color: #0000ff">class</span> VehiculoMaritimo { ... }<br /><span style="color: #008000">// Error: Herencia múltiple no soportada en C# (no lo está en .NET en general)</span><br /><span style="color: #0000ff">class</span> Hidroavion : VehiculoAereo, VehiculoMaritimo { ... }</pre>
              
              <p>
                </div> 
                
                <p>
                  Existen otros lenguajes (como C++) que soportan herencia múltiple y donde lo anterior seria válido. La <em>teoría de programación orientada a objetos</em> no pone restricciones al respecto… En muchos libros se menciona esa limitación aduciendo simplemente “que la herencia múltiple es peligrosa”… pero, por que?
                </p>
                
                <p>
                  Es difícil contar exactamente el problema en el que <em>puede</em> (y subrayo el <em>puede</em>) incurrir la herencia múltiple sin entrar en aspectos demasiado técnicos sobre como los compiladores implementan la herencia realmente, pero el motivo por el cual <em>la herencia múltiple es peligrosa</em> se llama “<a href="http://es.wikipedia.org/wiki/Problema_del_diamante">Herencia en diamante</a>”. Básicamente el problema de la herencia en diamante se produce cuando una clase D, hereda de dos clases B y C, las cuales ambas heredan de A. Imaginad ahora que:
                </p>
                
                <ul>
                  <li>
                    La clase A define un método FooA()
                  </li>
                  <li>
                    La clase B (que hereda de A) obtiene una copia del método FooA()
                  </li>
                  <li>
                    La clase C (que hereda de A) obtiene una copia del método FooA()
                  </li>
                  <li>
                    La clase D que hereda de B y de C… obtiene <strong>dos</strong> copias del método FooA (el de B y el de C). Pero una clase <strong>no puede tener dos métodos FooA()</strong> con el mismo nombre y parámetros… así que eso da error.
                  </li>
                </ul>
                
                <p>
                  Los lenguajes que soportan herencia múltiple (como C++) abordan ese problema mediante la llamada <em><a href="http://en.wikipedia.org/wiki/Virtual_inheritance">herencia virtual</a></em> pero otros lenguajes optan por eliminar el problema de raiz impidiendo la herencia múltiple. Entre estos lenguajes están <strong>todos los de .NET</strong> (C#, VB.NET, …) o Java entre ellos.
                </p>
                
                <p>
                  No tener herencia múltiple puede suponer una limitación y en algún momento puede serlo, pero nunca es muy grave: muchos diseños que requieren herencia múltiple pueden repensarse para usar sólo herencia simple. La clave está en que la mayoría de veces realmente no queremos herencia múltiple sinó <a href="http://es.wikipedia.org/wiki/Polimorfismo_(inform%C3%A1tica)">polimorfismo</a> múltiple. El polimorfismo múltiple está soportado en C# (y en .NET en general) mediante el uso de interfaces: Una clase puede implementar uno o más interfaces.
                </p>
                
                <blockquote>
                  <p>
                    <strong>Nota:</strong> En el contexto de este post polimorfismo significa poder usar un objeto de una clase CD que derive de CB, o que implemente una interfaz IX en cualquier sitio donde se espera un objeto de la clase base CB o de la interfaz IX.
                  </p>
                </blockquote>
                
                <p>
                  <strong>Redefinición de métodos</strong>
                </p>
                
                <p>
                  Imaginad ahora la siguiente jerarquía de clases:
                </p>
                
                <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
                  <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px" id="codeSnippet"><span style="color: #0000ff">class</span> Rectangulo<br />{<br />    <span style="color: #0000ff">public</span> Color Color { get; set; }<br />    <span style="color: #0000ff">public</span> <span style="color: #0000ff">void</span> Dibujar() { ... }<br />}<br /><br /><span style="color: #0000ff">class</span> RectanguloPintado : Rectangulo<br />{<br />    <span style="color: #0000ff">public</span> Color ColorFondo { get; set;}<br />}<br /></pre>
                  
                  <p>
                    </div> 
                    
                    <p>
                      Tenemos una clase <em>Rectangulo</em> que define una propiedad llamada Color (cuyo tipo es un objeto de una clase Color) y un método Dibujar. Luego tenemos una clase derivada <em>RectanguloPintado</em> que añade otra propiedad ColorFondo y obtiene por herencia la propiedad Color y el método Dibujar().
                    </p>
                    
                    <p>
                      Pero claro… un <em>RectanguloPintado</em> no se dibuja como un <em>Rectangulo</em> verdad? Cuando nos encontramos en este caso es cuando debemos usar la <a href="http://en.wikipedia.org/wiki/Method_overriding"><strong>redefinición de métodos</strong></a>: Es decir modificar el método heredado en la clase derivada con el código necesario para que funcione correctamente (en nuestro caso dibujar el rectángulo pintado).
                    </p>
                    
                    <p>
                      Pero ojo, y eso es importante, la clase base debe estar bien pensada y debe preveer que va a ser extendida y que dicho método <em>puede</em> ser redifinido: la clase base (Rectangulo) debe declarar el método Dibujar como <a href="http://en.wikipedia.org/wiki/Virtual_function">virtual</a>. Eso es algo que la gente que viene de Java le cuesta entender, puesto que en Java todos los métodos son virtuales por defecto. En C# (y en VB.NET) debemos explicitar que un método es virtual. Recordad el significado de virtual, es: Si alguien deriva de esta clase <em>puede</em> redefinir este método si lo necesita.
                    </p>
                    
                    <p>
                      Una vez la clase base define el método como virtual, la clase derivada puede redefinir el método:
                    </p>
                    
                    <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
                      <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px" id="codeSnippet"><span style="color: #0000ff">class</span> Rectangulo<br />{<br />   <span style="color: #0000ff">public</span> Color Color { get; set;}<br />   <span style="color: #0000ff">public</span> <span style="color: #0000ff">virtual</span> <span style="color: #0000ff">void</span> Dibujar() { ... }<br />}<br /><br /><span style="color: #0000ff">class</span> RectanguloPintado : Rectangulo<br />{<br />   <span style="color: #0000ff">public</span> ColorFondo {get; set;}<br />   <span style="color: #0000ff">public</span> <span style="color: #0000ff">override</span> <span style="color: #0000ff">void</span> Dibujar()<br />   {<br />       <span style="color: #0000ff">base</span>.Dibujar();<br />       <span style="color: #008000">// Pintamos...</span><br />   }<br />}</pre>
                      
                      <p>
                        </div> 
                        
                        <p>
                          Fijaos en cuatro aspectos clave:
                        </p>
                        
                        <ol>
                          <li>
                            El método Dibujar está declarado como virtual en la clase base
                          </li>
                          <li>
                            El método Dibujar está declarado como override en la clase derivada
                          </li>
                          <li>
                            El método de la clase derivada debe llamarse exactamente igual y tener los mismos parámetros, valor de retorno y visibilidad que el método de la clase base.
                          </li>
                          <li>
                            La llamada a base.Dibujar() lo que hace es llamar al método Dibujar() de la clase base. En este caso lo usamos porque para dibujar un rectángulo pintado lo que hacemos es dibujar el rectángulo primero y luego pintarlo. Para dibujar el rectángulo, en lugar de duplicar el código del método Dibujar() de la clase Rectangulo es mucho mejor llamara&#160; a dicho código. Y eso es lo que se puede hacer con base. Obviamente el uso de base <strong>no es obligatorio</strong> (en según que casos el método redefinido tiene una implementación totalmente distinta y no se puede aprovechar el método de la clase base).
                          </li>
                        </ol>
                        
                        <p>
                          En fin… espero que este post os ayude a comprender un poco más que es la herencia y como funciona… Como siempre comentarios, dudas son bienvenidos! 😉
                        </p>
                        
                        <p>
                          Un saludo!
                        </p>

 [1]: http://geeks.ms/blogs/etomas/archive/2010/07/07/c-b-225-sico-interfaces.aspx
 [2]: http://es.wikipedia.org/wiki/Herencia_(programaci%C3%B3n_orientada_a_objetos)
 [3]: http://es.wikipedia.org/wiki/Programaci%C3%B3n_orientada_a_objetos