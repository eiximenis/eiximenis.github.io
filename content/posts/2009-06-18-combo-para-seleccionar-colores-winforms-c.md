---
title: 'Combo para seleccionar colores (Winforms + C#)'

author: eiximenis

date: 2009-06-18T18:24:00+00:00
geeks_url: /?p=1457
geeks_visits:
  - 9189
geeks_ms_views:
  - 2470
categories:
  - Uncategorized

---
Hola... qué tal?

Imagina que en algún proyecto que estés haciendo, quieres ofrecer una combo para seleccionar colores. De acuerdo, ya se que hay otros métodos para hacer que el usuario seleccione un color, como usar el [ColorDialog][1], pero a lo mejor te interesa que el usuario sólo pueda escoger colores de una lista predeterminada...

<!--more-->

Por suerte en .NET hacer que una combo dibuje sus elementos como nosotros queremos, es realmente simple... ¿quieres tener una combo como esta?

[<img style="border-right: 0px; border-top: 0px; border-left: 0px; border-bottom: 0px" alt="image" src="/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_2.png" border="0" width="163" height="119" />][2] 

Pues, ya verás que fácil... apenas cinco líneas de código, y será tuya...

**Paso 1: Añadir la combo al formulario**

Añade una ComboBox normal a tu proyecto, y establece las propiedades _DropDownStyle_ a DropDownList (para que el usuario no pueda teclear valores nuevos) y _[DrawMode][3]_ a OwnerDrawFixed.

La propiedad _DrawMode_ es la clave: Puede tener tres valores, que son &#8220;Normal&#8221; (el sistema operativo se encarga de dibujar la combo), &#8220;OwnerDrawFixed&#8221; (nosotros nos encargamos de dibujar cada elemento) y &#8220;OwnerDrawVariable&#8221; (nosotros nos encargamos de dibujar cada elemento y además cada elemento puede tener un tamaño distinto).

**Paso 2: Código para dibujar cada elemento**

Crea una función gestora para el evento _DrawItem_: la combo lanza este evento cada vez que debe dibujar un elemento. La función gestora recibe un objeto _DrawItemEventArgs_ que contiene toda la información que necesitamos para poder &#8220;dibujar&#8221; el elemento: de que elemento se trata, el contexto gráfico a usar y las coordenadas que ocupa el rectángulo del elemento dentro del contexto gráfico de la combo...

Las primeras líneas de la función gestora serían las siguientes:

<pre class="code"><span style="color: #2b91af">ComboBox </span>cmb = sender <span style="color: blue">as </span><span style="color: #2b91af">ComboBox</span>;<br /><span style="color: blue">if </span>(cmb == <span style="color: blue">null</span>) <span style="color: blue">return</span>;<br /><span style="color: blue">if </span>(e.Index &lt; 0) <span style="color: blue">return</span>;<br /><span style="color: blue">if </span>(!(cmb.Items[e.Index] <span style="color: blue">is </span><span style="color: #2b91af">Color</span>)) <span style="color: blue">return</span>;<br /><span style="color: #2b91af">Color </span>color = (<span style="color: #2b91af">Color</span>)cmb.Items[e.Index];</pre>

[][4]

Aquí simplemente estamos asegurándonos de que el elemento que vamos a dibujar sea un objeto Color. Para ello accedemos a la propiedad _Index_ del objeto _DrawItemEventArgs_ que recibimos que nos indica el índice del elemento que estamos dibujando.

Ahora solo nos queda dibujar el elemento: un rectángulo con los bordes negros y rellenado del color indicado, junto al nombre del color:

<pre class="code"><span style="color: #2b91af">Brush </span>brush = <span style="color: blue">new </span><span style="color: #2b91af">SolidBrush</span>(color);<br />e.Graphics.DrawRectangle(<br />    <span style="color: #2b91af">Pens</span>.Black, <br />    <span style="color: blue">new </span><span style="color: #2b91af">Rectangle</span>(e.Bounds.Left + 2, e.Bounds.Top + 2, 19, <br />        e.Bounds.Size.Height - 4));<br />e.Graphics.FillRectangle(brush, <br />    <span style="color: blue">new </span><span style="color: #2b91af">Rectangle</span>(e.Bounds.Left + 3, e.Bounds.Top + 3, 18, <br />        e.Bounds.Size.Height - 5));<br />e.Graphics.DrawString(color.Name, cmb.Font, <br />    <span style="color: #2b91af">Brushes</span>.Black, e.Bounds.Left + 25, e.Bounds.Top + 2);<br />brush.Dispose();</pre>

[][4]

El primer DrawRectangle dibuja el borde del rectángulo, el FillRectangle pinta su interior y el DrawString escribe el nombre del color. Finalmente recordad que debemos hacer dispose de cualquier objeto GDI+ que usemos.

Como se puede observar, apenas cinco líneas de codigo GDI+.

**Paso 3: Perfeccionando el tema...**

La combo ya es 100% funcional, pero si la usais vereis que &#8220;no resalta&#8221; en azul el elemento seleccionado, tal y como hace por defecto la combobox... Evidentemente no lo hace porque no hemos puesto código para ello, pero tranquilos: son apenas tres líneas de código.

El propio objeto _DrawItemEventArgs_ tiene todo lo que necesitamos para poder hacerlo:

  1. El método DrawBackground() que dibuja el fondo del elemento actual: en blanco en azul en función de si tiene el cursor del ratón encima o no.
  2. La propiedad ForeColor que nos devuelve el color con el que dibujar el elemento actual (por defecto es negro si no tiene el cursor del ratón encima o blanco en caso contrario).
  3. La propiedad BackColor que nos devuelve el color que se usa en el método DrawBackground.

El código **final** para dibujar nuestra combo es el siguiente:

<pre class="code"><span style="color: blue">private void </span>cmbColor_DrawItem(<span style="color: blue">object </span>sender, <br />    <span style="color: #2b91af">DrawItemEventArgs </span>e)<br />{<br />    <span style="color: #2b91af">ComboBox </span>cmb = sender <span style="color: blue">as </span><span style="color: #2b91af">ComboBox</span>;<br />    <span style="color: blue">if </span>(cmb == <span style="color: blue">null</span>) <span style="color: blue">return</span>;<br />    <span style="color: blue">if </span>(e.Index &lt; 0) <span style="color: blue">return</span>;<br />    <span style="color: blue">if </span>(!(cmb.Items[e.Index] <span style="color: blue">is </span><span style="color: #2b91af">Color</span>)) <span style="color: blue">return</span>;<br />    <span style="color: #2b91af">Color </span>color = (<span style="color: #2b91af">Color</span>)cmb.Items[e.Index];<br />    <span style="color: green">// Dibujamos el fondo<br />    </span>e.DrawBackground();<br />    <span style="color: green">// Creamos los objetos GDI+<br />    </span><span style="color: #2b91af">Brush </span>brush = <span style="color: blue">new </span><span style="color: #2b91af">SolidBrush</span>(color);<br />    <span style="color: #2b91af">Pen </span>forePen = <span style="color: blue">new </span><span style="color: #2b91af">Pen</span>(e.ForeColor);<br />    <span style="color: #2b91af">Brush </span>foreBrush = <span style="color: blue">new </span><span style="color: #2b91af">SolidBrush</span>(e.ForeColor);<br />    <span style="color: green">// Dibujamos el borde del rectángulo<br />    </span>e.Graphics.DrawRectangle(<br />        forePen, <br />        <span style="color: blue">new </span><span style="color: #2b91af">Rectangle</span>(e.Bounds.Left + 2, e.Bounds.Top + 2, 19, <br />            e.Bounds.Size.Height - 4));<br />    <span style="color: green">// Rellenamos el rectángulo con el Color seleccionado<br />    // en la combo<br />    </span>e.Graphics.FillRectangle(brush, <br />        <span style="color: blue">new </span><span style="color: #2b91af">Rectangle</span>(e.Bounds.Left + 3, e.Bounds.Top + 3, 18, <br />            e.Bounds.Size.Height - 5));            <br />    <span style="color: green">// Dibujamos el nombre del color<br />    </span>e.Graphics.DrawString(color.Name, cmb.Font,<br />        foreBrush, e.Bounds.Left + 25, e.Bounds.Top + 2);<br />    <span style="color: green">// Eliminamos objetos GDI+<br />    </span>brush.Dispose();<br />    forePen.Dispose();<br />    foreBrush.Dispose();<br />}</pre>

[][4]

Y listos!! Nuestra combo ya está lista para usar. Podeis meterle algunos colores para probar:

<pre class="code">cmbColor1.Items.Add(<span style="color: #2b91af">Color</span>.Black);<br />cmbColor1.Items.Add(<span style="color: #2b91af">Color</span>.Blue);<br />cmbColor1.Items.Add(<span style="color: #2b91af">Color</span>.Red);<br />cmbColor1.Items.Add(<span style="color: #2b91af">Color</span>.White);<br />cmbColor1.Items.Add(<span style="color: #2b91af">Color</span>.Pink);<br />cmbColor1.Items.Add(<span style="color: #2b91af">Color</span>.Green);</pre>

[][4]

Y este es el resultado final:

[<img style="border-right: 0px; border-top: 0px; border-left: 0px; border-bottom: 0px" alt="image" src="/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_3.png" border="0" width="164" height="118" />][5] 

¿No está nada mal, eh?

Saludos a todos!

 [1]: http://msdn.microsoft.com/es-es/library/system.windows.forms.colordialog.aspx
 [2]: /cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_6.png
 [3]: http://msdn.microsoft.com/en-us/library/system.windows.forms.combobox.drawmode.aspx
 [4]: http://11011.net/software/vspaste
 [5]: /cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_8.png