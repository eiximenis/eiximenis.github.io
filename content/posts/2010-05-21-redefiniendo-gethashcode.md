---
title: Redefiniendo GetHashCode

author: eiximenis

date: 2010-05-21T11:45:55+00:00
geeks_url: /?p=1511
geeks_visits:
  - 2997
geeks_ms_views:
  - 1959
categories:
  - Uncategorized

---
Hola a todos! Un post para comentar paranoias varias sobre algo que parece tan simple como redefinir `GetHashCode()`…

<!--more-->

Primero las dos normas básicas que supongo que la mayoría ya conoceréis: 

  1. Si se redefine el método Equals() de una clase debería redefinirse también el método GetHashCode(), para que pueda cumplirse la segunda norma que es…
  2. Si la llamada a Equals para dos objetos devuelve _true_, entonces GetHashCode() debe devolver el mismo valor para ambos objetos.

Una forma fácil y rápida de implementar GetHashCode() y que cumpla ambas normas es algo así:

<div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
  <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px" id="codeSnippet"><span style="color: #0000ff">public</span> <span style="color: #0000ff">class</span> Foo<br />{<br />    <span style="color: #0000ff">public</span> <span style="color: #0000ff">int</span> Bar { get; set;}<br />    <span style="color: #0000ff">public</span> <span style="color: #0000ff">int</span> Baz { get; set;}<br />    <br />    <span style="color: #0000ff">public</span> <span style="color: #0000ff">override</span> <span style="color: #0000ff">bool</span> Equals(<span style="color: #0000ff">object</span> obj) <br />    {<br />        <span style="color: #0000ff">return</span> obj <span style="color: #0000ff">is</span> Foo && ((Foo)obj).Bar == Bar && ((Foo)obj).Baz == Baz;<br />    }<br /><br />    <span style="color: #0000ff">public</span> <span style="color: #0000ff">override</span> <span style="color: #0000ff">int</span> GetHashCode()<br />    {<br />        <span style="color: #0000ff">return</span> <span style="color: #0000ff">string</span>.Format(<span style="color: #006080">"{0},{1}"</span>, Bar, Baz).GetHashCode();<br />    }<br />}</pre>
  
  <p>
    </div> 
    
    <p>
      Simplemente creamos una representación en cadena del objeto y llamamos a GetHashCode de dicha cadena. ¿Algún problema? Bueno… pues la <em>tercera</em> norma de GetHashCode que no siempre es conocida:
    </p>
    
    <ul>
      <li>
        La función de hash (GetHashCode) debe devolver <em>siempre el mismo valor con independencia de los cambios</em> que se realicen sobre dicho objeto (lo podéis leer en <a title="http://msdn.microsoft.com/library/system.object.gethashcode(VS.80).aspx" href="http://msdn.microsoft.com/library/system.object.gethashcode(VS.80).aspx">http://msdn.microsoft.com/library/system.object.gethashcode(VS.80).aspx</a> en el tercer punto de las “notas para implementadores”).
      </li>
    </ul>
    
    <p>
      Si a alguien le parece que esta tercera norma entra en contradicción con la segunda, en el caso de objetos mutables… bienvenido al club! 😉
    </p>
    
    <p>
      Si no cumplimos esta tercera norma… <strong>no podemos objetos de nuestra clase como claves de un diccionario: </strong>P.ej. el siguiente test unitario realizado sobre la clase Foo, falla:
    </p>
    
    <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
      <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px" id="codeSnippet">[TestClass]<br /><span style="color: #0000ff">public</span> <span style="color: #0000ff">class</span> FooTests<br />{<br />    [TestMethod()]<br />    <span style="color: #0000ff">public</span> <span style="color: #0000ff">void</span> FooUsedAsKey()<br />    {<br />        var dict = <span style="color: #0000ff">new</span> Dictionary&lt;Foo, <span style="color: #0000ff">int</span>&gt;();<br />        Foo foo1 = <span style="color: #0000ff">new</span> Foo() { Bar = 10, Baz = 20 };<br />        Foo foo2 = <span style="color: #0000ff">new</span> Foo() { Bar = 10, Baz = 30 };<br />        dict.Add(foo1, 1);<br />        dict.Add(foo2, 2);<br />        foo2.Baz = 20;<br />        <span style="color: #0000ff">int</span> <span style="color: #0000ff">value</span> = dict[foo2];<br />        Assert.AreEqual(2, <span style="color: #0000ff">value</span>);   <span style="color: #008000">// Assert.AreEqual failed. Expected:&lt;2&gt;. Actual:&lt;1&gt;.</span><br />    }<br />}</pre>
      
      <p>
        </div> 
        
        <p>
          Esperaríamos que la llamada a dict[foo2] nos devolviese 2, ya que este es el valor asociado con foo2… <em>pero</em> como foo2 ha <em>mutado</em> y ahora devuelve el mismo hashcode que foo1, esa es la entrada que nos devuelve el diccionario… y por eso el Assert falla.
        </p>
        
        <blockquote>
          <p>
            <strong>Nota: </strong>Si alguien piensa que usando structs en lugar de clases se soluciona el problema… falso: Usando structs ocurre exactamente lo mismo.
          </p>
        </blockquote>
        
        <p>
          Ahora… varias dudas filosóficas:
        </p>
        
        <ol>
          <li>
            Alguien entiende que el test unitario está mal? Es decir que el assert debería ser <em>AreEqual(1, value)</em> puesto que si foo2 <strong>es igual</strong> a foo1, debemos encontrar el valor asociado con foo1, <em>aunque usemos otra referencia</em> (en este caso foo2).
          </li>
          <li>
            Planteando lo mismo de otro modo: Debemos entender que el diccionario <em>indexa</em> por valor (basándose en equals) o por referencia (basándose en ==)? El caso es entendamos lo que entendamos, la clase Dictionary <strong>usa Equals</strong> y no ==.
          </li>
          <li>
            El meollo de todo el asunto <strong>¿Tiene sentido usar objetos <em>mutables</em> como claves</strong> en un diccionario?
          </li>
        </ol>
        
        <p>
          Yo entiendo que no tiene sentido usar objetos mutables como claves, ya que entonces nos encontramos con todas esas paranoias… y no se vosotros pero yo soy <strong>incapaz</strong> de escribir un método GetHashCode() para la clase Foo que he expuesto y que se cumpla la tercera condición.
        </p>
        
        <p>
          Si aceptamos que usar objetos mutables como claves de un diccionario no tiene sentido, ahora me viene otra pregunta: Dado que es muy normal querer redefinir Equals para objetos mutables, porque <em>se supone que siempre debemos redefinir también GetHashCode</em>? No hubiese sido mucho mejor definir una interfaz IHashable y que el diccionario sólo aceptase claves que implementasen IHashable?
        </p>
        
        <p>
          No se… intuyo que la respuesta puede tener que ver con el hecho de que genéricos no aparecieron hasta la versión 2 del lenguaje (en lo que a mi me parece uno de los errores más grandes que Microsoft ha cometido al respecto de .NET), pero quien sabe…
        </p>
        
        <p>
          … las mentes de Redmond son inescrutables.
        </p>
        
        <p>
          Un saludo!
        </p>