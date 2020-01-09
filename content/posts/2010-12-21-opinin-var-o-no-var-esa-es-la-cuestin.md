---
title: 'Opinión: Var o no var… esa es la cuestión.'
author: eiximenis

date: 2010-12-21T14:57:00+00:00
geeks_url: /?p=1547
geeks_visits:
  - 4509
geeks_ms_views:
  - 3105
categories:
  - Uncategorized

---
Hola a todos! Desde hace algunos días estoy usando <a target="_blank" href="http://www.jetbrains.com/resharper/" rel="noopener noreferrer">Resharper</a>. La verdad no era, como decirlo, muy proclive para instalármelo, ya que había tenido no muy buenas experiencas con <a target="_blank" href="http://www.devexpress.com/Products/Visual_Studio_Add-in/Coding_Assistance/" rel="noopener noreferrer"><em>CodeRush</em></a>_._ Seguramente no eran culpa de CodeRush sinó mías, pero bueno... Al final me lo instalé y debo decir que estoy gratamente sorprendido: Es una auténtica maravilla.

Una cosa interesante de Resharper es que te hace _sugerencias_ (que puedes desactivar si quieres, por supuesto) sobre como _codificar mejor_. Y una de las sugerencias es usar var **siempre que se pueda**:

[<img height="60" width="244" src="/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_1857A309.png" alt="image" border="0" title="image" style="background-image: none; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" />][1]

Fijaos que incluso en un caso **trivial** como int i=0; nos recomienda que usemos var.

> **Nota: Primero una pequeña aclaración sobre var:** Por si acaso... Recordad que var **no es tipado dinámico**, ni late-binding ni nada parecido a esto. Var simplemente le indica al _compilador_ que infiera él el tipo de la variable. Pero la variable _tiene_ un tipo concreto _y lo tiene en tiempo de compilación_. Por lo tanto olvidad todos vuestros prejuicios (si los tenéis) sobre tipos dinámicos.

Hay tres corrientes de opinión al respecto de cuando usar var: Hay gente que opina que debe usarse _sólo_ cuando es necesario (cuando se trabaja con objetos anónimos). 

Otros opinan que cuando el tipo ya aparece en la lína de código puede usarse var. Es decir, admiten esto:

<div id="codeSnippetWrapper" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; width: 97.5%; font-family: 'Courier New', courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; cursor: text; border: silver 1px solid; padding: 4px;">
  <pre id="codeSnippet" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 0em; width: 100%; font-family: 'Courier New', courier, monospace; direction: ltr; color: black; font-size: 8pt; overflow: visible; border-style: none; padding: 0px;">var dict = <span style="color: #0000ff;">new</span> Dictionary&lt;<span style="color: #0000ff;">int</span>, <span style="color: #0000ff;">string</span>&gt;();</pre>
</div>

Porque el tipo de la variable ya aparece en el new. Pero no admiten lo siguiente:

<div id="codeSnippetWrapper" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; width: 97.5%; font-family: 'Courier New', courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; cursor: text; border: silver 1px solid; padding: 4px;">
  <pre id="codeSnippet" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 0em; width: 100%; font-family: 'Courier New', courier, monospace; direction: ltr; color: black; font-size: 8pt; overflow: visible; border-style: none; padding: 0px;">var result = stockManager.GetStocks();</pre>
</div>

Porque, viendo el código: como se puede saber el tipo de _result_? (Debes irte a ver que devuelve el método GetStocks).

Por último el tercer grupo de opinión está a favor de usar var siempre. Incluso en los casos más triviales.

Por curiosidad:

  1. La msdn se situa en el primer grupo de opinión (literalmente dice &ldquo;_the use of var does have at least the potential to make your code more difficult to understand for other developers. For that reason, the C# documentation generally uses var only when it is required&rdquo; &#8211; <http://msdn.microsoft.com/en-us/library/bb384061.aspx>_). 
  2. Resharper se sitúa en el tercer grupo, como hemos visto 
  3. Yo me situaba en el segundo grupo de opinión.

**¿Que aporta usar siempre var?**

Que Resharper me estuviese continuamente insistiendo en usar _var_ me hizo pensar en el _por que_ de esa razón. Así que lo que hice fue probar y hacerle caso. Y empecé a usar _var_ en todos los sitios. A ver que ocurría.

Y ocurrió una cosa interesante...

Al usar _var_ continuamente pierdes el _tipo_ de la variable de forma visual (es decir no sabes de que tipo es sólo viendo esa línea de código) y entonces te das cuenta de una cosa: que muchos nombres de variables **aportan muy poca información** sobre que hace realmente la variable. Los detractores de var dicen que puede complicar la lectura de código... pero que es más dificil de entender, esto:

<div id="codeSnippetWrapper" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; width: 97.5%; font-family: 'Courier New', courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; cursor: text; border: silver 1px solid; padding: 4px;">
  <pre id="codeSnippet" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 0em; width: 100%; font-family: 'Courier New', courier, monospace; direction: ltr; color: black; font-size: 8pt; overflow: visible; border-style: none; padding: 0px;">Player plr = GetCurrentPlayer();<br />Location l = plr.GetCustomLocation();<br />// Varias líneas de código hablando de <span style="color: #006080;">"l"</span> y <span style="color: #006080;">"plr"</span></pre>
</div>

o esto:

<div id="codeSnippetWrapper" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; width: 97.5%; font-family: 'Courier New', courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; cursor: text; border: silver 1px solid; padding: 4px;">
  <pre id="codeSnippet" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 0em; width: 100%; font-family: 'Courier New', courier, monospace; direction: ltr; color: black; font-size: 8pt; overflow: visible; border-style: none; padding: 0px;">var player = GetCurrentPlayer();<br />var location = player.GetCustomLocation();<br /><span style="color: #008000;">// Varias líneas de código hablando de "location" y "player"</span><br /></pre>
</div>

El tema está en que la variable la declaras en una línea, y la usas en varias más. Evidentemente, si no usas var, la línea que declara la variable te da más información (exactamente el tipo de la variable), información que pierdes si usas var. Pero, como te das cuenta que estás _perdiendo_ esta información, lo que haces es usar lo que resta de la línea para aumentar la claridad. Y que es lo que queda? El nombre de la variable.

Al usar var en todos los sitios _te fuerzas_ a poner nombres más declarativos a tus variables, que expresen lo que la variable hace. Y todos sabemos que esa **es una muy buena práctica**. Y a veces no la seguimos, y una de las razones es que al tener una línea tipo:

<div id="codeSnippetWrapper" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; width: 97.5%; font-family: 'Courier New', courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; cursor: text; border: silver 1px solid; padding: 4px;">
  <pre id="codeSnippet" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 0em; width: 100%; font-family: 'Courier New', courier, monospace; direction: ltr; color: black; font-size: 8pt; overflow: visible; border-style: none; padding: 0px;">Location l = <span style="color: #0000ff;">new</span> Location();</pre>
</div>

Mentalmente piensas: _<<Ya se ve que &ldquo;l&rdquo; es una Location. Ya queda claro.>>_

Pero no es cierto, porque al cabo de unas cuantas líneas te aparece una &ldquo;l&rdquo; y tienes que recordad que era una _Location_.

Mientras que si usas var, cuando escribes la línea tiendes a usar nombres más descriptivos, porque escribir:

<div id="codeSnippetWrapper" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; width: 97.5%; font-family: 'Courier New', courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; cursor: text; border: silver 1px solid; padding: 4px;">
  <pre id="codeSnippet" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 0em; width: 100%; font-family: 'Courier New', courier, monospace; direction: ltr; color: black; font-size: 8pt; overflow: visible; border-style: none; padding: 0px;">var l = <span style="color: #0000ff;">new</span> Location();</pre>
</div>

Hace como daño a la vista 🙂

Así que la sugerencia de usar _var_ siempre, personalmente no me parece muy desacertada, y creo que voy a tomar esa opción de ahora en adelante.

Pero... Y vosotros? ¿Que opináis al respecto?

Un saludo!

**PD:**&nbsp; También creo que hay otra razón para que Resharper nos guie a usar siempre var y es que el código con var es más fácil de refactorizar (menos cambios) que el que usa declaraciones explícitas.

 [1]: /cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_082BDB10.png