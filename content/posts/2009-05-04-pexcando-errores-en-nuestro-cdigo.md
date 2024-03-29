---
title: Pexcando errores en nuestro código…
author: eiximenis
type: post
date: 2009-05-04T11:03:00+00:00
geeks_url: /?p=1449
geeks_visits:
  - 1122
geeks_ms_views:
  - 945
categories:
  - Uncategorized
---
Buenas&hellip; &iquest;Conoceis [Pex][1]? Es una herramienta que genera tests unitarios a partir de nuestro c&oacute;digo. Su caracter&iacute;stica principal es que _analiza_ el c&oacute;digo e intenta generar tests unitarios que cubran todas las posibilidades de nuestro c&oacute;digo.

<!--more-->

Vamos a ver un peque&ntilde;o ejemplo de como usarlo y como se integra con Microsoft Code Contracts. Antes que nada os recomiendo echar un vistazo al genial Post de Jorge Serrano [Precondiciones y Microsoft Code Contracts v1.0][2].

Vamos a hacer un m&eacute;todo muy simple (y conocido por aquellos que desarrollen en VB.NET):

<pre class="code"><span style="color: blue">public static class </span><span style="color: #2b91af">StringExtensions
</span>{
     <span style="color: blue">public static string </span>Right(<span style="color: blue">this string </span>value, <span style="color: blue">int </span>chars)
     {
         <span style="color: blue">return </span>value;
     }
}</pre>

[][3][][3][][3]

El m&eacute;todo Right debe devolver los &uacute;ltimos chars car&aacute;cteres de la cadena value.

Simplemente con este c&oacute;digo nos descargamos [Pex][1] y lo instalamos. Una vez hecho vereis que nos aparece una nueva opci&oacute;n el men&uacute; contextual del &ldquo;Solution Explorer&rdquo; llamada &ldquo;Run Pex Explorations&rdquo;. Con esta opci&oacute;n lo que hacemos es que Pex analice **todo** nuestro c&oacute;digo buscando generar Tests unitarios para nuestros m&eacute;todos p&uacute;blicos:

[<img height="81" width="224" src="/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_3700F674.png" alt="image" border="0" title="image" style="border-right-width: 0px; display: inline; border-top-width: 0px; border-bottom-width: 0px; border-left-width: 0px" />][4] 

Si ejecutamos esta opci&oacute;n Pex analiza el c&oacute;digo y nos genera tests unitarios para el m&eacute;todo Right:

[<img height="43" width="499" src="/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_61DBEB37.png" alt="image" border="0" title="image" style="border-right-width: 0px; display: inline; border-top-width: 0px; border-bottom-width: 0px; border-left-width: 0px" />][5] 

&nbsp;

En este caso nos ha generado un solo test. Es importante saber como Pex genera los tests: no lo hace al azar, sin&oacute; que intenta cubrir el m&aacute;ximo de nuestro c&oacute;digo. En nuestro caso como no nuestro c&oacute;digo apenas hace nada, con un solo test, Pex obtiene una cobertura del 100% y se queda ah&iacute;. Evidentemente Pex nada sabe de _cual debe ser el resultado_ de nuestro m&eacute;todo, por lo que no puede generar tests unitarios funcionales.

Ahora **antes** de empezar a analizar c&oacute;digo vamos a ver el uso de Code Contracts. Una vez instalada, a&ntilde;ad&iacute;s la referencia a &ldquo;Microsoft Contracts Library&rdquo; (Microsoft.Contracts.dll). En el framework 4.0 parece ser que se incluir&aacute; el c&oacute;digo dentro de mscorlib, por lo que no ser&aacute; necesario usar referencia alguna.

Ya estamos listos para definir los contratos en nuestra clase. Para ello debemos especificar las _precondiciones_ de nuestro m&eacute;todo. Una _precondici&oacute;n_ es algo que debe cumplirse s&iacute; o s&iacute; para asegurar que la llamada&nbsp; anuestro m&eacute;todo es &ldquo;v&aacute;lida&rdquo;.

Una precondici&oacute;n de nuestro m&eacute;todo es que NO vamos a aceptar que la cadena de entrada sea nula. El m&eacute;todo Contract.Requires sirve para especificar las precondiciones:

<pre class="code"><span style="color: blue">public static string </span>Right(<span style="color: blue">this string </span>value, <span style="color: blue">int </span>chars)
{
    <span style="color: #2b91af">Contract</span>.Requires(value !=<span style="color: blue">null</span>);
    <span style="color: blue">return </span>value;
}</pre>

[][3][][3]

Si lanzamos Pex ahora vemos que&hellip; no ocurre nada distinto. Esto es porque por defecto los contratos **no se eval&uacute;an**. Si queremos que se eval&uacute;en debemos irnos a la pesta&ntilde;a &ldquo;Code Contracts&rdquo; de las propiedades del proyecto y habilitar &ldquo;Perform runtime contract checking&rdquo;. Una vez hecho esto, volvemos a ejecutar Pex y obtenemos lo siguiente:

[<img height="52" width="506" src="/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_213C2F79.png" alt="image" border="0" title="image" style="border-right-width: 0px; display: inline; border-top-width: 0px; border-bottom-width: 0px; border-left-width: 0px" />][6] 

Vemos que ahora Pex nos ha generado dos tests: uno que pase el contrato (le manda una cadena con un &lsquo; &rsquo; y otro que no (le manda null).

Otra pecondici&oacute;n que vamos a usar es que el segundo par&aacute;metro debe ser mayor que cero:

<pre class="code"><span style="color: blue">public static string </span>Right(<span style="color: blue">this string </span>value, <span style="color: blue">int </span>chars)
{
    <span style="color: #2b91af">Contract</span>.Requires(value !=<span style="color: blue">null</span>);
    <span style="color: #2b91af">Contract</span>.Requires(chars &gt; 0);
    <span style="color: blue">return </span>value;
}</pre>

[][3][][3]

&nbsp;

Una vez establecidas las precondiciones nos interesa establecer las _postcondiciones,_ es decir todo aquello que aseguramos que se cumple al finalizar el m&eacute;todo. Para ello usamos Contract.Ensures. P.ej. para a&ntilde;adir una postcondici&oacute;n que asegure que nunca devolveremos null:

<pre class="code"><span style="color: blue">public static string </span>Right(<span style="color: blue">this string </span>value, <span style="color: blue">int </span>chars)
{
    <span style="color: #2b91af">Contract</span>.Requires(value !=<span style="color: blue">null</span>);
    <span style="color: #2b91af">Contract</span>.Requires(chars &gt; 0);
    <span style="color: #2b91af">Contract</span>.Ensures(<span style="color: #2b91af">Contract</span>.Result&lt;<span style="color: blue">string</span>&gt;() != <span style="color: blue">null</span>);
    <span style="color: blue">return </span>value;
}</pre>

[][3][][3][][3]

Una cosa realmente **importante**: Contract.Ensures se eval&uacute;a **SIEMPRE** al final del m&eacute;todo&hellip; aunque como en este caso lo hayamos colocada _antes_ del return, se evaluar&aacute; _despu&eacute;s_ del return. El secreto est&aacute; en que Code Contracts viene con un MSIL rewriter que modifica el c&oacute;digo MSIL desplazando las llamadas a Contract.Requires al principio del m&eacute;todo y de Contract.Ensures al final. El m&eacute;todo `Contract.Result<T>` sirve para acceder al valor retornado por el m&eacute;todo.

Contract.Requires **no** deja obsoleto a Debug.Assert. Ambos son necesarios: Con Contract.Requires comprobaremos las precondiciones l&oacute;gicas de nuestro m&eacute;todo, mientras que Debug.Assert lo seguiremos usando para comprobaciones de depuraci&oacute;n (comprobaciones internas). 

Tenemos el contrato de nuestro m&eacute;todo especificado, tenemos a Pex para que nos genere tests unitarios&hellip; podemos empezar a meter c&oacute;digo!

<pre class="code"><span style="color: blue">public static class </span><span style="color: #2b91af">StringExtensions
</span>{
     <span style="color: blue">public static string </span>Right(<span style="color: blue">this string </span>value, <span style="color: blue">int </span>chars)
     {
         <span style="color: #2b91af">Contract</span>.Requires(value !=<span style="color: blue">null</span>);
         <span style="color: #2b91af">Contract</span>.Requires(chars &gt; 0);
         <span style="color: #2b91af">Contract</span>.Ensures(<span style="color: #2b91af">Contract</span>.Result&lt;<span style="color: blue">string</span>&gt;() != <span style="color: blue">null</span>);
         <span style="color: blue">if </span>(chars &gt;= value.Length) <span style="color: blue">return </span>value;
         <span style="color: blue">else
         </span>{
             <span style="color: blue">int </span>inicial = value.Length - chars;
             <span style="color: blue">if </span>(inicial &lt; 0) inicial = 0;
             <span style="color: blue">return </span>value.Substring(inicial, chars);
         }
     }
}</pre>

[][3][][3]

Una vez tenemos el c&oacute;digo ya listo (o eso creemos jejejeeee&hellip;) relanzamos Pex para que nos cree un nuevo conjunto de tests unitarios para nuestro m&eacute;todo. Como ahora tenemos un c&oacute;digo un poco m&aacute;s complejo, Pex nos crear&aacute; varios tests unitarios:

[<img height="107" width="488" src="/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_2236B007.png" alt="image" border="0" title="image" style="border-right-width: 0px; display: inline; border-top-width: 0px; border-bottom-width: 0px; border-left-width: 0px" />][7] 

Pex ha creado 4 tests unitarios con el objetivo de cubrir al m&aacute;ximo nuestro c&oacute;digo. A partir de este punto si queremos podemos generar un proyecto de test con estos 4 tests unitarios. Los podemos seleccionar desde la ventana de Pex Exploration Results y le dais a &ldquo;Save&rdquo;:

[<img height="76" width="497" src="/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_63674A0F.png" alt="image" border="0" title="image" style="border-right-width: 0px; display: inline; border-top-width: 0px; border-bottom-width: 0px; border-left-width: 0px" />][8] 

Con ello Pex nos genera el proyecto de tests unitarios:

[<img height="115" width="261" src="/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_3BC35E96.png" alt="image" border="0" title="image" style="border-right-width: 0px; display: inline; border-top-width: 0px; border-bottom-width: 0px; border-left-width: 0px" />][9] 

El proyecto tiene dos archivos especiales: El fichero StringExtensionsTest.cs y el fichero StringExtensionsTest.Right.g.cs.

El segundo (.g.cs) se regenera cada vez que ejecutamos Pex, por lo que NO debemos poner c&oacute;digo en &eacute;l. El primero por su parte se mantiene y contiene lo que Pex llama PUTs, o Parametrized Unit Tests. Un PUT es un test unitario pero que acepta par&aacute;metros. Pex los usa para poder &ldquo;agrupar&rdquo; clausulas Assert: En lugar de colocar un Assert para cada test unitario, coloca uno de solo en el PUT y los tests unitarios que genera Pex, son llamadas al PUT con los par&aacute;metros correspondientes.

P.ej. el c&oacute;digo del PUT generado es:

<pre class="code">[<span style="color: #2b91af">PexMethod</span>]
<span style="color: blue">public string </span>Right(<span style="color: blue">string </span>value, <span style="color: blue">int </span>chars)
{
    <span style="color: green">// TODO: add assertions to method StringExtensionsTest.Right(String, Int32)
    </span><span style="color: blue">string </span>result = <span style="color: #2b91af">StringExtensions</span>.Right(value, chars);
    <span style="color: blue">return </span>result;
}</pre>

[][3]

Como se puede ver no hace nada salvo llamar al m&eacute;todo real que estamos testeando pas&aacute;ndole los par&aacute;metros. Pero aqu&iacute; nosotros podr&iacute;amos poner Asserts adicionales, que se comprobar&iacute;an para **todos** los unit tests de Pex. El c&oacute;digo de un unit test de Pex es similar a:

<pre class="code">[<span style="color: #2b91af">TestMethod</span>]
[<span style="color: #2b91af">PexGeneratedBy</span>(<span style="color: blue">typeof</span>(<span style="color: #2b91af">StringExtensionsTest</span>))]
<span style="color: blue">public void </span>Right04()
{
    <span style="color: blue">string </span>s;
    s = <span style="color: blue">this</span>.Right(<span style="color: #a31515">"   "</span>, 1);
    <span style="color: #2b91af">Assert</span>.AreEqual&lt;<span style="color: blue">string</span>&gt;(<span style="color: #a31515">" "</span>, s);
}</pre>

[][3]

La llamada a this.Right es la llamada al PUT que ha generado antes.

Os dejo un par de videos con m&aacute;s informaci&oacute;n sobre Code Contracts y Pex para que les echeis un vistazo, dado que es un tema que vale realmente la pena!

  * [http://channel9.msdn.com/posts/Peli/The-Synergy-of-Code-Contracts-and-Pex/][10] 
  * [Research: Contract Checking and Automated Test Generation with Pex][11] 

Saludos!

 [1]: http://research.microsoft.com/en-us/projects/Pex/
 [2]: /blogs/jorge/archive/2009/04/26/precondiciones-y-microsoft-code-contracts.aspx
 [3]: http://11011.net/software/vspaste
 [4]: /cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_0E628A6D.png
 [5]: /cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_3D47CD02.png
 [6]: /cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_31B4A074.png
 [7]: /cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_49DA9B80.png
 [8]: /cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_1871488F.png
 [9]: /cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_00DC2BCD.png
 [10]: http://channel9.msdn.com/posts/Peli/The-Synergy-of-Code-Contracts-and-Pex/
 [11]: http://channel9.msdn.com/pdc2008/TL51/