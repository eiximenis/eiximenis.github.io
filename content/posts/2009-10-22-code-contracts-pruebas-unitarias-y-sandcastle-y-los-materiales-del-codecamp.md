---
title: Code Contracts, Pruebas Unitarias y SandCastle (y los materiales del CodeCamp)
description: Code Contracts, Pruebas Unitarias y SandCastle (y los materiales del CodeCamp)
author: eiximenis

date: 2009-10-22T09:44:00+00:00
geeks_url: /?p=1474
geeks_visits:
  - 1096
geeks_ms_views:
  - 549
categories:
  - Uncategorized

---
Buenas! Como prometí en el [post anterior sobre el CodeCamp][1], en mi charla sobre Code Contracts, quedaron por ver algunos temillas que aprovecho para comentar ahora.

**Pruebas Unitarias**

Primero remarcar que **no** realiceis pruebas unitarias para validar que vuestros contratos están bien, es decir, si teneis un método:

<div id="codeSnippetWrapper" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; width: 97.5%; font-family: 'Courier New', courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; cursor: text; border: silver 1px solid; padding: 4px;">
  <pre id="codeSnippet" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 0em; width: 100%; font-family: 'Courier New', courier, monospace; direction: ltr; color: black; font-size: 8pt; overflow: visible; border-style: none; padding: 0px;"><span style="color: #0000ff">public</span> <span style="color: #0000ff">void</span> Foo(<span style="color: #0000ff">int</span> arg)<br />{<br />    Contract.Requires(arg &gt; 0);<br />}</pre>
  
  <p>
    </div> 
    
    <p>
      No hagáis una prueba unitaria que compruebe que el contracto falla si se le pasa cero a arg.
    </p>
    
    <p>
      ¿Por qué no hacer esta prueba? Pues por dos razones.
    </p>
    
    <p>
      La primera es &ldquo;filosófica&rdquo;: Un contrato fallado significa código erróneo, y no tiene sentido que una prueba unitaria que es código erróneo funcione correctamente.
    </p>
    
    <p>
      La segunda es más práctica: Qué va a suceder cuando el contrato falle? Code Contracts dos modos de funcionamiento (se realiza un Asert, o bien se lanza una excepción), pero podemos añadir <em>nuestro propio comportamiento</em>. Además la prueba unitaria no debería asumir nada en cuanto al comportamiento de Code Contracts, ya que ese es variable por configuración (recordad que incluso podríamos tener todos los contratos deshabilitados en Release).
    </p>
    
    <p>
      Si un contrato se rompe durante la ejecución de la prueba unitaria, la prueba <strong>debería fallar</strong>, con independencia del comportamiento en que tengamos configurado Code Contracts. Para ello puede usarse el evento ContractFailed, de la clase Contract, que se lanza cada vez que un contrato se rompe. El siguiente código lo podéis usar a tal efecto:
    </p>
    
    <div id="codeSnippetWrapper" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; width: 97.5%; font-family: 'Courier New', courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; cursor: text; border: silver 1px solid; padding: 4px;">
      <pre id="codeSnippet" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 0em; width: 100%; font-family: 'Courier New', courier, monospace; direction: ltr; color: black; font-size: 8pt; overflow: visible; border-style: none; padding: 0px;"><span style="color: #008000">/* Convierte los fallos de contrato en fallos de Tests */</span><br />[AssemblyInitialize]<br /><span style="color: #0000ff">public</span> <span style="color: #0000ff">static</span> <span style="color: #0000ff">void</span> AssemblyInitialize(TestContext tc)<br />{<br />    Contract.ContractFailed += (sender, e) =&gt;<br />    {<br />        e.SetHandled();<br />        e.SetUnwind();<br />        Assert.Fail(e.FailureKind.ToString() + <span style="color: #006080">":"</span> + e.Message);<br />    };<br />}</pre>
      
      <p>
        </div> 
        
        <p>
          Nos suscribimos al evento ContractFailed y en él:
        </p>
        
        <ol>
          <li>
            Indicamos a Code Contracts que <em>no</em> realice el comportamiento por defecto. P.ej. si Code Contracts estaba configurada para lanzar una excepción evitamos que la lance.
          </li>
          <li>
            Fallamos la prueba unitaria (mostrando cual es el contrato fallado).
          </li>
        </ol>
        
        <p>
          De esta manera podéis &ldquo;olvidaros&rdquo; de Code Contracts y realizar pruebas unitarias que verifiquen vuestro código en lugar de los contratos, con la total seguridad de que si en cualquier momento un contrato falla, la prueba unitaria os lo verificará!
        </p>
        
        <p>
          <strong>Sandcastle</strong>
        </p>
        
        <p>
          <a href="http://www.codeplex.com/Sandcastle">Sandcastle</a> es la herramienta para generar archivos de ayuda a partir de los comentarios XML en código fuente. No es excesivamente agradable de utilizar, aunque por suerte existe <a href="http://www.codeplex.com/SHFB">SandCastle Help File Builder</a>, una GUI sobre Sandcastle que convierte su uso en casi trivial (si lo usáis sabed que puede integrarse con Team Build).
        </p>
        
        <p>
          Los contratos son un elemento fundamental de los métodos y de una clase, pero sandcastle no los soporta (la última versión de sandcastle es mayo del 2008). Code contracts viene con un parche para sandcastle que se instala encima de sandcastle y que añade soporte para contratos. Con este parche instalado, podemos utilizar nuevas etiquetas xml, en nuestra documentación:
        </p>
        
        <div id="codeSnippetWrapper" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; width: 97.5%; font-family: 'Courier New', courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; cursor: text; border: silver 1px solid; padding: 4px;">
          <pre id="codeSnippet" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 0em; width: 100%; font-family: 'Courier New', courier, monospace; direction: ltr; color: black; font-size: 8pt; overflow: visible; border-style: none; padding: 0px;"><span style="color: #008000">/// &lt;summary&gt;</span><br /><span style="color: #008000">/// Construye la pila de un tamaño máximo indicado.</span><br /><span style="color: #008000">/// &lt;/summary&gt;</span><br /><span style="color: #008000">/// &lt;param name="size"&gt;Tamaño máximo&lt;/param&gt;</span><br /><span style="color: #008000">/// &lt;requires /&gt;</span><br /><span style="color: #0000ff">public</span> Pila(<span style="color: #0000ff">int</span> size)<br />{<br />    Contract.Requires(size &gt; 0, <span style="color: #006080">"Tamaño inicial mayor que cero"</span>);<br />    <span style="color: #008000">// Código...</span><br />}</pre>
          
          <p>
            </div> 
            
            <p>
              P.ej. en este caso la etiqueta <requires /> indica que queremos <em>documentar</em> las precondiciones de este método. Luego en código utilizamos una sobrecarga de Contract.Requires que permite especificar un mensaje. Este mensaje es el que usa cuando falla el contrato, y también es el que usará sandcastle. Si no ponemos la etiqueta <requires /> las precondiciones no se documentarán aunque existan llamadas a Contract.Requires. Si alguien se pregunta porqué, es lo mismo que ocurre con el resto de documentación. P.ej. si no ponemos una etiqueta <param> no se documenta un parámetro de método, aunque éste exista.
            </p>
            
            <p>
              El resultado será algo parecido a lo siguiente:
            </p>
            
            <p>
              <a href="/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_094503BA.png"><img height="184" width="244" src="/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_46F1EB76.png" alt="image" border="0" title="image" style="border-right-width: 0px; display: inline; border-top-width: 0px; border-bottom-width: 0px; border-left-width: 0px" /></a>
            </p>
            
            <p>
              Como se puede observar se incluye un apartado &ldquo;Contracts&rdquo; con las precondiciones. Igual que existe la etiqueta <requires /> existen otras como <ensures /> para las postcondiciones,... Están todas ellas documentadas en el manual de Code Contracts.
            </p>
            
            <p>
              <strong>Y el material del CodeCamp...</strong>
            </p>
            
            <p>
              Aunque se lo mandaré a la organización para que lo cuelguen en la <a href="http://www.codecamp.es">web del codecamp</a> os dejo aquí el enlace al material de mi charla del codecamp. Son varias demos, que se auto-explican bastante y el ppt usado. Para cualquier duda... ya sabéis donde encontrarme!
            </p>
            
            <p>
              &nbsp;<iframe scrolling="no" marginwidth="0" frameborder="0" src="http://cid-6521c259e9b1bec6.skydrive.live.com/embedrowdetail.aspx/Public/CodeCamp2009" marginheight="0" style="background-color: #ffffff; margin: 3px; width: 240px; height: 66px; border: #dde5e9 1px solid; padding: 0px;"></iframe>
            </p>
            
            <p>
              Saludos!
            </p>

 [1]: /blogs/etomas/archive/2009/10/20/de-resaca-del-codecamp.aspx