---
title: Diseñar clases para ser heredadas…
author: eiximenis

date: 2010-04-12T08:55:43+00:00
geeks_url: /?p=1504
geeks_visits:
  - 3471
geeks_ms_views:
  - 2527
categories:
  - Uncategorized

---
Una de las ventajas de la programación orientada a objetos, es la herencia de clases y el polimorfismo: eso es la capacidad para crear clases _derivadas_ a partir de otras clases y poder usar las clases derivadas en cualquier lugar donde se espere la clase base.

El comportamiento por _defecto_ de C# (y VB.NET) es que cuando creamos una clase, _esa se puede exteder_, es decir puede crearse una clase derivada. Debemos declarar explicitamente la clase como sellada (<a href="http://msdn.microsoft.com/es-es/library/88c54tsw(VS.80).aspx" target="_blank" rel="noopener noreferrer">sealed</a>) para impedir que alguien cree una clase derivada a partir de la nuestra. Es una buena práctica declarar tantas clases _sealed_ como sea posible (al menos las clases públicas, para las internas no son necesarias tantas precauciones ya que no serán visibles desde fuera de nuestro assembly). Si dejamos una clase sin sellar, debemos **ser conscientes** de que estamos dando la posibilidad a alguien de que derive de nuestra clase. Eso, obviamente, no tiene nada de malo… pero entonces debemos asegurarnos de que nuestra clase **está preparada** para que se derive de ella.

**Métodos virtuales**

Los métodos virtuales definen los puntos de extensión de nuestra clase: es decir la clase derivada sólo puede redefinir (override) los métodos que nosotros hayamos marcado como virtuales en nuestra clase. Ese es uno de los aspectos que más me gustan de C# respecto a Java: en Java no hay el concepto de método virtual (o dicho de otro modo, todos lo son). En C# (y en VB.NET) al tener que marcar **explícitamente** los métodos que vamos a permitir que las clases derivadas redefinan, nos obliga a tener que pensar en cómo puede extenderse nuestra clase. Si nuestra clase **no** tiene ningún método virtual… qué sentido tiene dejarla sin sellar? Si nuestra clase no tiene métodos virtuales es que o bien hemos pensado que no tiene sentido que se extienda de ella, o que ni hemos pensado en cómo puede extenderse, y en ambos casos, para evitar problemas, es mejor dejar la clase sellada.

**Miembros privados**

Los miembros privados sólo son accesibles _dentro de la propia clase que los declara_. Cuando creamos una clase pensada para que se pueda heredar de ella, debemos tener siempre presente el <a href="http://en.wikipedia.org/wiki/Liskov_substitution_principle" target="_blank" rel="noopener noreferrer">principio de sustitución de Liskov</a> (LSP). Este principio es, digamos, la base teórica del polimorfismo, y viene a decir que si S es una clase derivada de T, entonces los objetos de tipo T pueden ser reemplazados por objetos de tipo S <u>sin alterar el comportamiento de nuestro sistema</u>.

Si estáis habituados con el polimorfismo, diréis que viene a ser lo mismo… pero de hecho es posible tener polimorfismo sin respetar LSP. El polimorfismo viene dado por el lenguaje: es el lenguaje _quien nos deja usar_ objetos de tipo S donde se esperen objetos de tipo T, pero el lenguaje no nos garantiza que se respete el LSP… eso debemos hacerlo nosotros.

¿Y que tiene que ver ese rollo con los métodos privados? Pues bien… imaginad un método virtual (por lo tanto redefinible desde la clase base), que accede a un método **privado**, para comprobar por ejemplo **una precondición**:

<div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
  <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px" id="codeSnippet"><span style="color: #0000ff">public</span> <span style="color: #0000ff">class</span> T {<br />   <span style="color: #0000ff">private</span> <span style="color: #0000ff">int</span> count;<br />   <span style="color: #0000ff">public</span> <span style="color: #0000ff">virtual</span> <span style="color: #0000ff">void</span> Remove() {<br />     <span style="color: #0000ff">if</span> (count &lt;= 0) <span style="color: #0000ff">throw</span> <span style="color: #0000ff">new</span> InvalidOperationException();<br />   }<br />}<br /></pre>
  
  <p>
    </div> 
    
    <p>
      Si alguien deriva de nuestra clase T, y redefine el método <em>Remove </em>no tiene mecanismo para poder comprobar esa precondición. Es decir, <strong>incluso aunque nosotros documentemos la precondición</strong> quien redefine el método Remove() no tiene manera de poder reproducirla, puesto que no puede acceder a la variable <em>count</em>.
    </p>
    
    <p>
      Así, si quieres que quien herede de tus clases pueda respetar el LSP, recuerda de <strong>no acceder a miembros privados desde métodos virtuales</strong>.
    </p>
    
    <p>
      <strong>Excepciones</strong>
    </p>
    
    <p>
      El LSP implica que los métodos redefinidos en una clase derivada, no deben lanzar <em>nuevos tipos de excepciones</em> que los que lanza el mismo método en la clase base (a no ser que esos nuevos tipos de excepciones sean a la vez subtipos de las excepciones lanzadas en el método de la clase base). Es decir, si un método <em>foo()</em> de una clase base T, lanza la excepcion <em>IOException </em>y se deriva de dicha clase T, al redefinir el método <em>foo</em> puede lanzarse la excepción <em>IOException</em> o cualquier derivada de esta, pero no podría lanzar una excepción de tipo <em>ArgumentException</em> p.ej.
    </p>
    
    <p>
      Java define la clausula <em>throws</em> que indica que tipo de excepciones puede lanzar un método y no permite que los métodos redefinidos lancen excepciones de cualquier otro tipo al declarado en <em>throws</em>. C# no tiene ningún mecanismo que pueda obligar al cumplimiento de esta norma del LSP, así que sólo nos queda, al menos, documentar las excepciones que lanza cada método. Otra opción es definir métodos protegidos para lanzar todas las excepciones. De esa manera si quien deriva de nuestra clase detecta que debe lanzar la excepción X, puede llamar al método ThrowX. Eso garantiza que todas las excepciones se lanzan de forma coherente.
    </p>
    
    <p>
      <strong>Code Contracts</strong>
    </p>
    
    <p>
      Los que seguís mi blog sabréis que he hablado de <em><a href="http://msdn.microsoft.com/en-us/devlabs/dd491992.aspx" target="_blank" rel="noopener noreferrer">Code Contracts</a></em> un par de veces. Si eres de los que piensa que Code Contracts es un nuevo Debug.Assert, cuando quieras quedamos para tomar unas cervecillas y discutimos el asunto 🙂
    </p>
    
    <p>
      Para el tema que nos ocupa, Code Contracts es básicamente una bendición. LSP obliga a que una clase derivada:
    </p>
    
    <ul>
      <li>
        Debe <strong>mantener</strong> todas las precondiciones de la clase base, sin poder añadir más precondiciones.
      </li>
      <li>
        Debe <strong>garantizar</strong> todas las postcondiciones de la clase base, sin poder eliminar postcondiciones.
      </li>
      <li>
        Debe <strong>preservar </strong>todos los invariantes de la clase base.
      </li>
    </ul>
    
    <p>
      Si no usamos Code Contracts, el principal problema es que las precondiciones, postcondiciones y invariantes, son <em>codigo normal</em>. Si en mi método virtual Foo() tengo un código que comprueba una precondición determinada, cuando se redefina este método dicho código debe ser escrito otra vez, para volver a comprobar la precondición si queremos mantener el LSP. Code Contracts <strong>nos garantiza esto automáticamente</strong>:
    </p>
    
    <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
      <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px" id="codeSnippet"><span style="color: #0000ff">class</span> T<br />{<br />    <span style="color: #0000ff">protected</span> <span style="color: #0000ff">bool</span> empty;<br />    <span style="color: #0000ff">public</span> T()<br />    {<br />        empty = <span style="color: #0000ff">true</span>;<br />    }<br />    <span style="color: #0000ff">public</span> <span style="color: #0000ff">virtual</span> <span style="color: #0000ff">void</span> Add()<br />    {<br />        Contract.Requires(empty);<br />        Contract.Ensures(!empty);<br />        empty = <span style="color: #0000ff">false</span>;<br />    }<br />}<br /><br /><span style="color: #0000ff">class</span> S : T<br />{<br />    <span style="color: #0000ff">public</span> <span style="color: #0000ff">override</span> <span style="color: #0000ff">void</span> Add()<br />    {<br />    }<br />}<br /></pre>
      
      <p>
        </div> 
        
        <p>
          En este código cuando llamamos al método Add() de la clase S, se evalúan las precondiciones del método… <strong>que están definidas en la clase base</strong>.
        </p>
        
        <p>
          De esta manera el desarrollador de la clase S, no debe preocuparse de reimplementar todas las precondiciones y postcondiciones de nuevo y puede concentrarse en lo que interesa: la redefinición del método Add().
        </p>
        
        <blockquote>
          <p>
            <strong>Nota Técnica:</strong> Usar Code Contracts no nos exime de declara la variable <em>empty</em> con la suficiente visibilidad. Es decir, aunque sólo accedamos a <em>empty</em> dentro de la precondición contenida en T.Add(), debemos tener presente que desde el punto de vista de Code Contracts, esta precondición también se ejecutará dentro del método S.Add(). Y eso en Code Contracts es literal: Code Contracts <strong>no</strong> funciona creando un método “oculto” en la clase T que compruebe las precondiciones, sinó que modifica el IL generado por el compilador, para “copiar y pegar” las precondiciones y postcondiciones en cada método donde sean requeridas. Así, pues es “literalmente” como si las llamadas a Contract estuviesen también en S.Add(). Si declaramos la variable empty como private, el código compila (puesto que para el compilador todo es correcto), pero al ejecutarse se lanzará una excepción indicandonos que desde S.Add() estamos intentando acceder a un miembro sobre el cual no tenemos visibilidad.
          </p>
        </blockquote>
        
        <p>
          Code Contracts <strong>no</strong> obliga al cumplimiento estricto de LSP, dado que el desarrollador de la clase S puede <em>añadir</em> nuevas precondiciones al método Add:
        </p>
        
        <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
          <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px" id="codeSnippet"><span style="color: #0000ff">public</span> <span style="color: #0000ff">override</span> <span style="color: #0000ff">void</span> Add()<br />{<br />    Contract.Requires(otherPostcondition);<br />}<br /></pre>
          
          <p>
            </div> 
            
            <p>
              De todos modos si el desarrollador de la clase derivada hace esto, ya lo hace bajo su conocimiento y responsabilidad y <strong>además</strong> Code Contracts emite un warning para que quede claro: <em>Method &#8216;CC1.S.Add&#8217; overrides &#8216;CC1.T.Add&#8217;, thus cannot add Requires.</em>
            </p>
            
            <p>
              <strong>Conclusión</strong>
            </p>
            
            <p>
              Cuando creamos clases, espcialmente clases públicas que formen parte de una API que usen otras personas, debemos tener especial cuidado a la hora de diseñarlas. Debemos prestar especial atención al LSP y tener presente que aunqué cumplir el LSP (aunqué siempre es muy recomendable) no sea siempre obligatorio, sí que puede serlo en otros casos, y nosotros como creadores de la clase base, debemos asegurarnos de tener el cuidado necesario y facilitar la vida al máximo para que quien derive de nuestras clases pueda cumplir el LSP… Y a riesgo de hacerme pesado, insisto que Code Contracts es una bendición.
            </p>
            
            <p>
              Un saludo a todos!
            </p>