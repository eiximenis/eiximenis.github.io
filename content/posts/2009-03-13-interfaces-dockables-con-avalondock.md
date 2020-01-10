---
title: Interfaces “Dockables” con AvalonDock

author: eiximenis

date: 2009-03-13T13:39:00+00:00
geeks_url: /?p=1443
geeks_visits:
  - 2894
geeks_ms_views:
  - 1155
categories:
  - Uncategorized

---
Hace algún tiempo [escribí como integrar AvalonDock con PRISM][1]. En el post daba por asumidos algunos conceptos de AvalonDock, pero algunos comentarios recibidos me han pedido si puedo profundizar un poco, así que voy a ello. Vamos a ver como crear paso a paso una aplicación AvalonDock y luego, en otro post ya veremos como podemos PRISMearla... 🙂

<!--more-->

[AvalonDock][2] es una librería para la creación de interfaces con ventanas flotantes (al estilo del propio Visual Studio). Según su creador soporta también winforms, aunque yo siempre la he usado con WPF, así que nada puedo deciros de su integración con winforms. 

Hay un tutorial de AvalonDock en el blog de su creador ([http://www.youdev.net/post/2008/09/25/AvalonDock-Tutorial.aspx][3]) que aunque muy básico explica los conceptos clave... echadle una ojeada si os apetece 🙂

Supongo que teneis instalada AvalonDock... si usais el instalador, os creará una toolbar en Visual Studio con los controles de AvalonDock, no és imprescindible pero ayuda 🙂

El primer paso es crear una aplicación WPF, añadir una referencia a AvalonDock.dll y en la ventana principal, debemos añadir el componente _padre_ de AvalonDock, el _DockingManager_:

<divre class="code"></divre><span style="color: blue"><</span><span style="color: #a31515">Window </span><span style="color: red">x</span><span style="color: blue">:</span><span style="color: red">Class</span><span style="color: blue">=&#8221;DockReader.Window1&#8243; </span><span style="color: red">xmlns</span><span style="color: blue">=&#8221;http://schemas.microsoft.com/winfx/2006/xaml/presentation&#8221; </span><span style="color: red">xmlns</span><span style="color: blue">:</span><span style="color: red">x</span><span style="color: blue">=&#8221;http://schemas.microsoft.com/winfx/2006/xaml&#8221; </span><span style="color: red">Title</span><span style="color: blue">=&#8221;Window1&#8243; </span><span style="color: red">Height</span><span style="color: blue">=&#8221;300&#8243; </span><span style="color: red">Width</span><span style="color: blue">=&#8221;300&#8243; </span><span style="color: red">xmlns</span><span style="color: blue">:</span><span style="color: red">ad</span><span style="color: blue">=&#8221;clr-namespace:AvalonDock;assembly=AvalonDock&#8221;> <</span><span style="color: #a31515">Grid</span><span style="color: blue">> <</span><span style="color: #a31515">ad</span><span style="color: blue">:</span><span style="color: #a31515">DockingManager </span><span style="color: red">Name</span><span style="color: blue">=&#8221;dockManager&#8221;/> </</span><span style="color: #a31515">Grid</span><span style="color: blue">> </</span><span style="color: #a31515">Window</span><span style="color: blue">> </span>

<pre></pre>

[][4][][4]

En este caso DockManager ocupa todo el espacio disponible en la Grid del control. Si nos interesa tener algún control en la ventana principal que no participe del sistema de ventanas flotantes (p.ej. una statusbar o una ribbon siempre visibles y fijas), podemos colocarla en alguna otra fila de la grid:

<divre class="code"></divre><span style="color: blue"><</span><span style="color: #a31515">Window </span><span style="color: red">x</span><span style="color: blue">:</span><span style="color: red">Class</span><span style="color: blue">=&#8221;DockReader.Window1&#8243; </span><span style="color: red">xmlns</span><span style="color: blue">=&#8221;http://schemas.microsoft.com/winfx/2006/xaml/presentation&#8221; </span><span style="color: red">xmlns</span><span style="color: blue">:</span><span style="color: red">x</span><span style="color: blue">=&#8221;http://schemas.microsoft.com/winfx/2006/xaml&#8221; </span><span style="color: red">Title</span><span style="color: blue">=&#8221;Window1&#8243; </span><span style="color: red">Height</span><span style="color: blue">=&#8221;300&#8243; </span><span style="color: red">Width</span><span style="color: blue">=&#8221;300&#8243; </span><span style="color: red">xmlns</span><span style="color: blue">:</span><span style="color: red">ad</span><span style="color: blue">=&#8221;clr-namespace:AvalonDock;assembly=AvalonDock&#8221;> <</span><span style="color: #a31515">Grid</span><span style="color: blue">> <</span><span style="color: #a31515">Grid.RowDefinitions</span><span style="color: blue">> <</span><span style="color: #a31515">RowDefinition </span><span style="color: red">Height</span><span style="color: blue">=&#8221;50&#8243;/> <</span><span style="color: #a31515">RowDefinition</span><span style="color: blue">/> </</span><span style="color: #a31515">Grid.RowDefinitions</span><span style="color: blue">> </span><span style="color: green"><!&#8211; ToolBar fija &#8211;> </span><span style="color: blue"><</span><span style="color: #a31515">ToolBar </span><span style="color: red">Grid.Row</span><span style="color: blue">=&#8221;0&#8243;> <</span><span style="color: #a31515">Button</span><span style="color: blue">> <</span><span style="color: #a31515">Image </span><span style="color: red">Source</span><span style="color: blue">=&#8221;/DockReader;component/open.png&#8221; </span><span style="color: red">Height</span><span style="color: blue">=&#8221;32&#8243; </span><span style="color: red">Width</span><span style="color: blue">=&#8221;32&#8243;></</span><span style="color: #a31515">Image</span><span style="color: blue">> </</span><span style="color: #a31515">Button</span><span style="color: blue">> </</span><span style="color: #a31515">ToolBar</span><span style="color: blue">> </span><span style="color: green"><!&#8211; Docking Manager &#8211;> </span><span style="color: blue"><</span><span style="color: #a31515">ad</span><span style="color: blue">:</span><span style="color: #a31515">DockingManager </span><span style="color: red">Grid.Row</span><span style="color: blue">=&#8221;1&#8243; </span><span style="color: red">Name</span><span style="color: blue">=&#8221;dockManager&#8221;/> </</span><span style="color: #a31515">Grid</span><span style="color: blue">> </</span><span style="color: #a31515">Window</span><span style="color: blue">> </span>

<pre></pre>

[][4]

El DockingManager por sí solo no hace nada... tenemos que rellenarlo y para ello podemos usar los dos contenidos que tiene AvalonDock:

  * DockableContent: Un DockableContent se puede &ldquo;dockar&rdquo; en cualquier parte del DockingManager y también puede aparecer en forma de ventana flotante. 
  * DocumentContent: Los DocumentContent aparecen &ldquo;fijos&rdquo; en una zona, y pueden &ldquo;dockarse&rdquo; en cualquier parte de esta zona (y generalmente no aparecen en forma de ventanas flotantes).

P.ej. en Visual Studio las distintas ventanas con documentos serian DocumentContents, y el restro de ventanas flotantes (p.ej. la toolbox) serian DockableContents.

Para que AvalonDock funcione correctamente debemos incrustar los DockableContent y los DocumentContent en sus propios paneles que son DocumentPane (para contener DocumentContents) y DockablePane para contener DockableContents.

P.ej. si colocamos un DocumentPane con dos DocumentContent dentro del DockingManager, obtenemos la siguiente interfaz:

<table border="0" width="400" cellpadding="2" cellspacing="0">
  <tr>
    <td width="200" valign="top">
      <a href="/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_57DE03DF.png"><img border="0" width="244" src="/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_02B8F8A3.png" alt="image" height="244" style="border-right: 0px; border-top: 0px; display: inline; border-left: 0px; border-bottom: 0px" title="image" /></a>
    </td>
    
    <td width="200" valign="top">
      <a href="/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_24382832.png"><img border="0" width="244" src="/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_6AB7A8EB.png" alt="image" height="211" style="border-right: 0px; border-top: 0px; display: inline; border-left: 0px; border-bottom: 0px" title="image" /></a>
    </td>
  </tr>
</table>

Como podeis observar sólo con un DocumentPane ya tenemos una interfaz totalmente dockable.

Si queremos combinar varios DocumentPane (cada uno con sus ContentPane) y/o varios DockablePane (cada uno con sus DockableContents) debemos usar un panel dentro del DockingManager. P.ej, la siguiente interfaz és el mismo ContentPane de antes y un DockableContent, todo ello dentro de un StackPanel:

[<img border="0" width="244" src="/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_00A3ABED.png" alt="image" height="244" style="border-right: 0px; border-top: 0px; display: inline; border-left: 0px; border-bottom: 0px" title="image" />][5] 

El código XAML sería tal y como sigue:

<divre class="code"></divre><span style="color: green"><!&#8211; Docking Manager &#8211;> </span><span style="color: blue"><</span><span style="color: #a31515">ad</span><span style="color: blue">:</span><span style="color: #a31515">DockingManager </span><span style="color: red">Grid.Row</span><span style="color: blue">=&#8221;1&#8243; </span><span style="color: red">Name</span><span style="color: blue">=&#8221;dockManager&#8221;> <</span><span style="color: #a31515">StackPanel </span><span style="color: red">Orientation</span><span style="color: blue">=&#8221;Horizontal&#8221;> <</span><span style="color: #a31515">ad</span><span style="color: blue">:</span><span style="color: #a31515">DocumentPane </span><span style="color: red">Width</span><span style="color: blue">=&#8221;200&#8243;> <</span><span style="color: #a31515">ad</span><span style="color: blue">:</span><span style="color: #a31515">DocumentContent</span><span style="color: blue">> <</span><span style="color: #a31515">Label</span><span style="color: blue">></span><span style="color: #a31515">Documento 1</span><span style="color: blue"></</span><span style="color: #a31515">Label</span><span style="color: blue">> </</span><span style="color: #a31515">ad</span><span style="color: blue">:</span><span style="color: #a31515">DocumentContent</span><span style="color: blue">> <</span><span style="color: #a31515">ad</span><span style="color: blue">:</span><span style="color: #a31515">DockableContent</span><span style="color: blue">> <</span><span style="color: #a31515">Label</span><span style="color: blue">></span><span style="color: #a31515">Documento 2</span><span style="color: blue"></</span><span style="color: #a31515">Label</span><span style="color: blue">> </</span><span style="color: #a31515">ad</span><span style="color: blue">:</span><span style="color: #a31515">DockableContent</span><span style="color: blue">> </</span><span style="color: #a31515">ad</span><span style="color: blue">:</span><span style="color: #a31515">DocumentPane</span><span style="color: blue">> <</span><span style="color: #a31515">ad</span><span style="color: blue">:</span><span style="color: #a31515">DockablePane </span><span style="color: red">Width</span><span style="color: blue">=&#8221;80&#8243;> <</span><span style="color: #a31515">ad</span><span style="color: blue">:</span><span style="color: #a31515">DockableContent</span><span style="color: blue">> <</span><span style="color: #a31515">Button</span><span style="color: blue">></span><span style="color: #a31515">Botón Dockable</span><span style="color: blue"></</span><span style="color: #a31515">Button</span><span style="color: blue">> </</span><span style="color: #a31515">ad</span><span style="color: blue">:</span><span style="color: #a31515">DockableContent</span><span style="color: blue">> </</span><span style="color: #a31515">ad</span><span style="color: blue">:</span><span style="color: #a31515">DockablePane</span><span style="color: blue">> </</span><span style="color: #a31515">StackPanel</span><span style="color: blue">> </</span><span style="color: #a31515">ad</span><span style="color: blue">:</span><span style="color: #a31515">DockingManager</span><span style="color: blue">></span>

<pre></pre>

[][4]

&nbsp;

&nbsp;

Esta ventana tiene el problema de que el tamaño del DocumentPane y del DockablePane está fijo... AvalonDock nos ofrece un panel (ResizingPanel) que integra un splitter... simplemente cambiando el StackPanel por un ResizingPanel nuestra interfaz ya es totalmente funcional!

Vamos a hacer una aplicación completa: un lector de ficheros XPS. En la parte izquierda tendremos un DocumentPane con los distintos ficheros abiertos, y en la parte derecha una lista con los nombres de los ficheros abiertos.

El XAML de la ventana principal quedaria:<divre class="code"></divre>

<span style="color: blue"><</span><span style="color: #a31515">Window </span><span style="color: red">x</span><span style="color: blue">:</span><span style="color: red">Class</span><span style="color: blue">=&#8221;DockReader.Window1&#8243; </span><span style="color: red">xmlns</span><span style="color: blue">=&#8221;http://schemas.microsoft.com/winfx/2006/xaml/presentation&#8221; </span><span style="color: red">xmlns</span><span style="color: blue">:</span><span style="color: red">x</span><span style="color: blue">=&#8221;http://schemas.microsoft.com/winfx/2006/xaml&#8221; </span><span style="color: red">Title</span><span style="color: blue">=&#8221;Window1&#8243; </span><span style="color: red">Height</span><span style="color: blue">=&#8221;300&#8243; </span><span style="color: red">Width</span><span style="color: blue">=&#8221;300&#8243; </span><span style="color: red">xmlns</span><span style="color: blue">:</span><span style="color: red">ad</span><span style="color: blue">=&#8221;clr-namespace:AvalonDock;assembly=AvalonDock&#8221;> <</span><span style="color: #a31515">Grid</span><span style="color: blue">> <</span><span style="color: #a31515">Grid.RowDefinitions</span><span style="color: blue">> <</span><span style="color: #a31515">RowDefinition </span><span style="color: red">Height</span><span style="color: blue">=&#8221;50&#8243;/> <</span><span style="color: #a31515">RowDefinition</span><span style="color: blue">/> </</span><span style="color: #a31515">Grid.RowDefinitions</span><span style="color: blue">> </span><span style="color: green"><!&#8211; ToolBar fija &#8211;> </span><span style="color: blue"><</span><span style="color: #a31515">ToolBar </span><span style="color: red">Grid.Row</span><span style="color: blue">=&#8221;0&#8243;> <</span><span style="color: #a31515">Button </span><span style="color: red">x</span><span style="color: blue">:</span><span style="color: red">Name</span><span style="color: blue">=&#8221;cmdAbrir&#8221; </span><span style="color: red">Click</span><span style="color: blue">=&#8221;cmdAbrir_Click&#8221;> <</span><span style="color: #a31515">Image </span><span style="color: red">Source</span><span style="color: blue">=&#8221;/DockReader;component/open.png&#8221; </span><span style="color: red">Height</span><span style="color: blue">=&#8221;32&#8243; </span><span style="color: red">Width</span><span style="color: blue">=&#8221;32&#8243;></</span><span style="color: #a31515">Image</span><span style="color: blue">> </</span><span style="color: #a31515">Button</span><span style="color: blue">> </</span><span style="color: #a31515">ToolBar</span><span style="color: blue">> </span><span style="color: green"><!&#8211; Docking Manager &#8211;> </span><span style="color: blue"><</span><span style="color: #a31515">ad</span><span style="color: blue">:</span><span style="color: #a31515">DockingManager </span><span style="color: red">Grid.Row</span><span style="color: blue">=&#8221;1&#8243; </span><span style="color: red">Name</span><span style="color: blue">=&#8221;dockManager&#8221;> <</span><span style="color: #a31515">ad</span><span style="color: blue">:</span><span style="color: #a31515">ResizingPanel </span><span style="color: red">Orientation</span><span style="color: blue">=&#8221;Horizontal&#8221;> <</span><span style="color: #a31515">ad</span><span style="color: blue">:</span><span style="color: #a31515">DocumentPane </span><span style="color: red">x</span><span style="color: blue">:</span><span style="color: red">Name</span><span style="color: blue">=&#8221;docsPane&#8221;> </</span><span style="color: #a31515">ad</span><span style="color: blue">:</span><span style="color: #a31515">DocumentPane</span><span style="color: blue">> <</span><span style="color: #a31515">ad</span><span style="color: blue">:</span><span style="color: #a31515">DockablePane</span><span style="color: blue">> <</span><span style="color: #a31515">ad</span><span style="color: blue">:</span><span style="color: #a31515">DockableContent</span><span style="color: blue">> <</span><span style="color: #a31515">DockPanel</span><span style="color: blue">> <</span><span style="color: #a31515">Label </span><span style="color: red">DockPanel.Dock</span><span style="color: blue">=&#8221;Top&#8221;><br /></span><span style="color: #a31515">Ficheros Abiertos:</span><span style="color: blue"></</span><span style="color: #a31515">Label</span><span style="color: blue">> <</span><span style="color: #a31515">ListBox </span><span style="color: red">x</span><span style="color: blue">:</span><span style="color: red">Name</span><span style="color: blue">=&#8221;openFiles&#8221;/> </</span><span style="color: #a31515">DockPanel</span><span style="color: blue">> </</span><span style="color: #a31515">ad</span><span style="color: blue">:</span><span style="color: #a31515">DockableContent</span><span style="color: blue">> </</span><span style="color: #a31515">ad</span><span style="color: blue">:</span><span style="color: #a31515">DockablePane</span><span style="color: blue">> </</span><span style="color: #a31515">ad</span><span style="color: blue">:</span><span style="color: #a31515">ResizingPanel</span><span style="color: blue">> </</span><span style="color: #a31515">ad</span><span style="color: blue">:</span><span style="color: #a31515">DockingManager</span><span style="color: blue">> </</span><span style="color: #a31515">Grid</span><span style="color: blue">> </</span><span style="color: #a31515">Window</span><span style="color: blue">></span>

<pre></pre>

Vemos la toolbar fija (fuera del DockingManager). Luego tenemos un DocumentPane vacío (inicialmente no tenemos cargado ningún documento) y también vemos un DockablePane que tiene un DockableContent con una label y la listbox...

En la función gestora del evento Click del botón &ldquo;cmdAbrir&rdquo;, mostramos un [OpenFileDialog][6] para seleccionar un fichero xps:

<divre class="code"></divre><span style="color: blue">private void </span>cmdAbrir_Click(<span style="color: blue">object </span>sender, <span style="color: #2b91af">RoutedEventArgs </span>e) { <span style="color: #2b91af">OpenFileDialog </span>ofd = <span style="color: blue">new </span><span style="color: #2b91af">OpenFileDialog</span>(); ofd.AddExtension = <span style="color: blue">true</span>; ofd.DefaultExt = <span style="color: #a31515">&#8220;.xps&#8221;</span>; ofd.Multiselect = <span style="color: blue">false</span>; ofd.CheckFileExists = <span style="color: blue">true</span>; ofd.Filter = <span style="color: #a31515">&#8220;Documentos XPS | *.xps&#8221;</span>; <span style="color: blue">if </span>(ofd.ShowDialog() == <span style="color: blue">true</span>) { <span style="color: blue">string </span>s = ofd.FileName; CrearNuevoVisor(s); } }

<pre></pre>

[][4]

Finalmente la función CrearNuevoVisor es la que creará un DocumentContent con el contenido del fichero XPS:

<divre class="code"></divre><span style="color: blue">private void </span>CrearNuevoVisor(<span style="color: blue">string </span>fname) { <span style="color: green">// Creamos el DocumentContent </span><span style="color: #2b91af">DocumentContent </span>dc = <span style="color: blue">new </span><span style="color: #2b91af">DocumentContent</span>(); dc.Closed += <span style="color: blue">new </span><span style="color: #2b91af">EventHandler</span>(Document_Closed); dc.Title = System.IO.<span style="color: #2b91af">Path</span>.GetFileNameWithoutExtension(fname); <span style="color: blue">this</span>.files.Add(dc.Title); <span style="color: green">// Carga el XPS y crea un DocumentViewer </span><span style="color: #2b91af">XpsDocument </span>doc = <span style="color: blue">new </span><span style="color: #2b91af">XpsDocument</span>(fname, <span style="color: #2b91af">FileAccess</span>.Read, <span style="color: #2b91af">CompressionOption</span>.NotCompressed); <span style="color: #2b91af">DocumentViewer </span>dv = <span style="color: blue">new </span><span style="color: #2b91af">DocumentViewer</span>(); <span style="color: green">// Muestra el XPS en el DocumentViewer </span>dv.Document = doc.GetFixedDocumentSequence(); <span style="color: green">// Incrusta el DocumentViewer en el DocumentContent </span>dc.Content = dv; <span style="color: green">// Incrusta el DocumentContent en el ContentPane </span><span style="color: blue">this</span>.docsPane.Items.Add(dc); }

<pre></pre>

[][4]

El código es realmente simple, no? Si os preguntais que es this.files, os diré que es una ObservableCollection<string> que está enlazada a la listbox:

<divre class="code"></divre><span style="color: blue">private </span><span style="color: #2b91af">ObservableCollection</span><<span style="color: blue">string</span>> files; <span style="color: blue">public </span>Window1() { InitializeComponent(); <span style="color: blue">this</span>.files = <span style="color: blue">new </span><span style="color: #2b91af">ObservableCollection</span><<span style="color: blue">string</span>>(); <span style="color: blue">this</span>.openFiles.ItemsSource = <span style="color: blue">this</span>.files; }

<pre></pre>

[][4]

Finalmente, en el método Document_Closed lo único que hacemos es eliminar de this.files la cadena que hemos añadido antes (no pongo el código, ya que es trivial).

Con esto ya tenemos un lector de XPS completamente funcional y con una interfaz totalmente &ldquo;dockable&rdquo;...

<table border="0" width="400" cellpadding="2" cellspacing="0">
  <tr>
    <td width="200" valign="top">
      <a href="/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_12169D76.png"><img border="0" width="244" src="/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_0A241C76.png" alt="image" height="184" style="border-right: 0px; border-top: 0px; display: inline; border-left: 0px; border-bottom: 0px" title="image" /></a>
    </td>
    
    <td width="200" valign="top">
      <a href="/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_6762E4B8.png"><img border="0" width="244" src="/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_66B1BAD4.png" alt="image" height="185" style="border-right: 0px; border-top: 0px; display: inline; border-left: 0px; border-bottom: 0px" title="image" /></a>
    </td>
  </tr>
</table>

En un próximo post veremos como convertir este DockReader en una aplicación compuesta usando PRISM!

Saludos!

 [1]: /blogs/etomas/archive/2009/01/20/prism-y-avalondock.aspx
 [2]: http://www.codeplex.com/AvalonDock/
 [3]: http://www.youdev.net/post/2008/09/25/AvalonDock-Tutorial.aspx "http://www.youdev.net/post/2008/09/25/AvalonDock-Tutorial.aspx"
 [4]: http://11011.net/software/vspaste
 [5]: /cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_2E8E6DF4.png
 [6]: http://msdn.microsoft.com/es-es/library/microsoft.win32.openfiledialog.aspx