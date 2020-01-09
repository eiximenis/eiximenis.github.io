---
title: Explorando ASP.NET MVC 2.0… áreas
author: eiximenis

date: 2009-09-22T18:49:00+00:00
geeks_url: /?p=1467
geeks_visits:
  - 3085
geeks_ms_views:
  - 1253
categories:
  - Uncategorized

---
No hace mucho Jorge Dieguez comentaba [la salida de ASP.NET MVC 2.0 Preview 1][1]. He estado investigando un poco las novedades del framework, y hoy quiero hablaros de lo que se conoce como áreas.

Cuando hablamos de áreas no nos referimos a zonas de la pantalla (tipo webparts) sinó que las áreas en ASP.NET MVC permiten construir una aplicación en base a _módulos_. Actualmente en la preview 1, el sistema consiste en:

  * Definir la área principal: Un proyecto ASP.NET MVC completo, con sus vistas, sus controladores, sus hojas de estilo, ...
  * Definir el resto de áreas (_módulos_). Cada área es un proyecto ASP.NET MVC que añade sus propias vistas y controladores. Cada área confía en el área principal para el resto de temas (p.ej. scripts y css están en el área principal). De igual modo toda la inicialización (global.asax) se realiza únicamente en el área principal: el resto de áreas no tienen global.asax.

De esta manera se pretende poder gestionar mejor grandes aplicaciones: al estar formadas por áreas independientes, que a su vez son proyectos independientes de ASP.NET MVC, equipos distintos podrían realizar cada una de las áreas.

En el archivo global.asax, del área principal, cuando registramos las rutas, mapeamos determinadas URLs a cada uno de las áreas. Imaginad que tenemos una área que contiene toda la parte de administración del sitio, podríamos añadir a global.asax, una línea tal como:

<pre class="code"><span style="background: black; color: white">routes</span><span style="background: black; color: cyan">.</span><span style="background: black; color: white">MapAreaRoute</span><span style="background: black; color: silver">(</span><span style="background: black; color: gray">"Admin"</span><span style="background: black; color: silver">, </span><span style="background: black; color: gray">"Admin_Default"</span><span style="background: black; color: silver">,
    </span><span style="background: black; color: gray">"admin/{controller}/{action}/{id}"</span><span style="background: black; color: silver">,
    </span><span style="background: black; color: #3e60fd">new string</span><span style="background: black; color: silver">[] { </span><span style="background: black; color: gray">" Admin.Controllers" </span><span style="background: black; color: silver">});</span></pre>

[][2][][2]

El primer parámetro es el nombre del área (luego veremos que _el nombre tiene implicaciones_), el segundo el nombre de la ruta (cada área puede tener tantas rutas como se desee), el tercer parámetro es la URL a enrutar y el cuarto parámetro es un array con todos los namespaces donde buscar los controladores.

De esta manera todos las URLs que empiecen por /Admin/ serán enrutadas a la área _Admin_. P.ej. la URL /Admin/Users/View/20 sería enrutada al controlador &ldquo;UsersController&rdquo;, llamando a su acción View con el Id de 20. El último parámetro (un array de strings) es importante puesto que contiene todos los namespaces de los posibles controladores de la área de administración. Dado que son proyectos distintos pueden tener controladores con el mismo nombre, así que el motor de ASP.NET MVC debe saber el namespace a utilizar... En fin, que _simplemente_ le estamos diciendo al motor de ASP.NET MVC que use el controlador que sea que esté en el namespace _Admin.Controllers_.

P.ej. en mi caso estoy trabajando con 3 áreas (la principal y dos más) y tengo definida la siguiente tabla de enrutamiento:

<pre class="code"><span style="background: black; color: white">routes</span><span style="background: black; color: cyan">.</span><span style="background: black; color: white">MapAreaRoute</span><span style="background: black; color: silver">(</span><span style="background: black; color: gray">"Game"</span><span style="background: black; color: silver">, </span><span style="background: black; color: gray">"Game_Default"</span><span style="background: black; color: silver">, <br />    </span><span style="background: black; color: gray">"game/{controller}/{action}/{id}"</span><span style="background: black; color: silver">,
    </span><span style="background: black; color: #3e60fd">new </span><span style="background: black; color: silver">{ </span><span style="background: black; color: white">controller </span><span style="background: black; color: cyan">= </span><span style="background: black; color: gray">"Home"</span><span style="background: black; color: silver">, </span><span style="background: black; color: white">action </span><span style="background: black; color: cyan">= </span><span style="background: black; color: gray">"Index"</span><span style="background: black; color: silver">, </span><span style="background: black; color: white">id </span><span style="background: black; color: cyan">= </span><span style="background: black; color: gray">"" </span><span style="background: black; color: silver">},
    </span><span style="background: black; color: #3e60fd">new string</span><span style="background: black; color: silver">[] { </span><span style="background: black; color: gray">"Game.Controllers" </span><span style="background: black; color: silver">});
</span><span style="background: black; color: white">routes</span><span style="background: black; color: cyan">.</span><span style="background: black; color: white">MapAreaRoute</span><span style="background: black; color: silver">(<br />    </span><span style="background: black; color: gray">"Admin"</span><span style="background: black; color: silver">, </span><span style="background: black; color: gray">"Admin_Default"</span><span style="background: black; color: silver">, </span><span style="background: black; color: gray">"admin/{controller}/{action}/{id}"</span><span style="background: black; color: silver">,
    </span><span style="background: black; color: #3e60fd">new </span><span style="background: black; color: silver">{ </span><span style="background: black; color: white">controller </span><span style="background: black; color: cyan">= </span><span style="background: black; color: gray">"Home"</span><span style="background: black; color: silver">, </span><span style="background: black; color: white">action </span><span style="background: black; color: cyan">= </span><span style="background: black; color: gray">"Index"</span><span style="background: black; color: silver">, </span><span style="background: black; color: white">id </span><span style="background: black; color: cyan">= </span><span style="background: black; color: gray">"" </span><span style="background: black; color: silver">},
    </span><span style="background: black; color: #3e60fd">new string</span><span style="background: black; color: silver">[] { </span><span style="background: black; color: gray">"Admin.Controllers" </span><span style="background: black; color: silver">});
</span><span style="background: black; color: white">routes</span><span style="background: black; color: cyan">.</span><span style="background: black; color: white">MapAreaRoute</span><span style="background: black; color: silver">(</span><span style="background: black; color: gray">"Main"</span><span style="background: black; color: silver">, <br />    </span><span style="background: black; color: gray">"Main_Default"</span><span style="background: black; color: silver">, </span><span style="background: black; color: gray">"{controller}/{action}/{id}"</span><span style="background: black; color: silver">,
    </span><span style="background: black; color: #3e60fd">new </span><span style="background: black; color: silver">{ </span><span style="background: black; color: white">controller </span><span style="background: black; color: cyan">= </span><span style="background: black; color: gray">"Home"</span><span style="background: black; color: silver">, </span><span style="background: black; color: white">action </span><span style="background: black; color: cyan">= </span><span style="background: black; color: gray">"Index"</span><span style="background: black; color: silver">, </span><span style="background: black; color: white">id </span><span style="background: black; color: cyan">= </span><span style="background: black; color: gray">"" </span><span style="background: black; color: silver">},
    </span><span style="background: black; color: #3e60fd">new string</span><span style="background: black; color: silver">[] { </span><span style="background: black; color: gray">"Web.Controllers" </span><span style="background: black; color: silver">});</span></pre>

[][2][][2]

Todas las URLs que empiecen por /Game/ se enrutarán al área que implementa la aplicación web en sí (efectivamente es un juego :p), mientras que todas las URLs que empiecen por /Admin/ se enrutarán al área que implementa la zona de administración. Finalmente el resto de URLs (que no empiecen ni por /Game/ ni por /Admin/ serán enrutadas al área principal. Fijaos que las 3 áreas definen un controlador &ldquo;Home&rdquo; pero no es problema, puesto que el namespace es distinto 🙂

Obviamente se puede generar un enlace desde una área a otra (sino maldita la gracia). Así de esta manera se genera un enlace a la acción _About_ del controlador Home _de la área actual_:

<pre class="code"><span style="background: black; color: yellow">&lt;%</span><span style="background: black; color: teal">= </span><span style="background: black; color: white">Html</span><span style="background: black; color: cyan">.</span><span style="background: black; color: white">ActionLink</span><span style="background: black; color: silver">(</span><span style="background: black; color: gray">"Pulse aquí"</span><span style="background: black; color: silver">, </span><span style="background: black; color: gray">"About"</span><span style="background: black; color: silver">, </span><span style="background: black; color: gray">"Home"</span><span style="background: black; color: silver">) </span><span style="background: black; color: yellow">%&gt;</span></pre>

[][2]

En fin, como siempre... Para cambiar de área simplemente debemos pasar el nombre del área (debe usarse el mismo nombre con el cual se ha registrado el área en _MapAreaRoute_):

<pre class="code"><span style="background: black; color: yellow">&lt;%</span><span style="background: black; color: teal">= </span><span style="background: black; color: white">Html</span><span style="background: black; color: cyan">.</span><span style="background: black; color: white">ActionLink</span><span style="background: black; color: silver">(</span><span style="background: black; color: gray">"Pulse aquí"</span><span style="background: black; color: silver">, </span><span style="background: black; color: gray">"About"</span><span style="background: black; color: silver">, </span><span style="background: black; color: gray">"Home"</span><span style="background: black; color: silver">, <br /></span><span style="background: black; color: #3e60fd">new </span><span style="background: black; color: silver">{ </span><span style="background: black; color: white">area </span><span style="background: black; color: cyan">= </span><span style="background: black; color: gray">"Game" </span><span style="background: black; color: silver">} , </span><span style="background: black; color: #3e60fd">null</span><span style="background: black; color: silver">)</span><span style="background: black; color: yellow">%&gt;</span></pre>

[][2]

Esto genera un enlace a la acción _About_ del controlador _Home_ del área _Game_ (en mi caso una URL tipo /Game/Home/About).

**Todo muy bonito pero...**

... estamos en una preview 1, así que no todo es tan directo.

Imaginad el caso del enlace que he puesto que genera una URL a la acción About del controlador Home del área Game. Imaginad que el código de la acción About de dicho controlador es:

<pre class="code"><span style="background: black; color: #3e60fd">public </span><span style="background: black; color: #2b91af">ActionResult </span><span style="background: black; color: white">About</span><span style="background: black; color: silver">() { </span><span style="background: black; color: #3e60fd">return </span><span style="background: black; color: white">View</span><span style="background: black; color: silver">(); }</span></pre>

[][2]

Esto debería retornar la vista que est&agrave; en Views/Home/About.aspx **del proyecto que es el área Game**... Pues no 🙁 Esto utiliza la vista que está en Views/Home/About.aspx **del proyecto que es el área principal**. Si el área principal **no tiene** esta vista todo rebienta. Obviamente este comportamiento ni es correcto (cada área debe proporcionar sus propias vistas) ni útil ya que si de todos modos el proyecto del área principal debe contener todas las áreas... donde está la capacidad de poder modularizar el desarrollo?

Bueno, todo esto viene debido a que estamos en preview 1, y simplemente debemos cambiar la configuración de los proyectos. Si editamos los ficheros .csproj de cada aplicación web, veremos primero unas líneas como estas (que como nos indican debemos descomentar):

<pre class="code"><span style="background: black; color: gray">&lt;!--</span><span style="background: #151515; color: green"> To enable MVC area subproject support, uncomment the following two lines:
&lt;UsingTask TaskName="Microsoft.Web.Mvc.Build.CreateAreaManifest" <br />AssemblyName="Microsoft.Web.Mvc.Build, Version=2.0.0.0, Culture=neutral, <br /></span><span style="background: #151515; color: green">PublicKeyToken=31bf3856ad364e35" /&gt;
&lt;UsingTask TaskName="Microsoft.Web.Mvc.Build.CopyAreaManifests" <br />AssemblyName="Microsoft.Web.Mvc.Build, Version=2.0.0.0, Culture=neutral, <br />PublicKeyToken=31bf3856ad364e35" /&gt;
</span><span style="background: black; color: gray">--&gt;</span></pre>

[][2]

y luego otras como las siguientes (que debemos descomentar según el proyecto sea el área principal o no):

<pre class="code"><span style="background: black; color: gray">&lt;!--</span><span style="background: #151515; color: green"> If this is an area child project, uncomment the following line:
&lt;CreateAreaManifest AreaName="$(AssemblyName)" AreaType="Child" <br />AreaPath="$(ProjectDir)" ManifestPath="$(AreasManifestDir)" <br />ContentFiles="@(Content)" /&gt;
</span><span style="background: black; color: gray">--&gt;
&lt;!--</span><span style="background: #151515; color: green"> If this is an area parent project, uncomment the following lines:
&lt;CreateAreaManifest AreaName="$(AssemblyName)" AreaType="Parent" <br />AreaPath="$(ProjectDir)" ManifestPath="$(AreasManifestDir)" <br />ContentFiles="@(Content)" /&gt;
</span><span style="background: #151515; color: green">&lt;CopyAreaManifests ManifestPath="$(AreasManifestDir)" <br />CrossCopy="false" RenameViews="true" /&gt;
</span><span style="background: black; color: gray">--&gt;</span></pre>

[][2]

Así pues ya vemos lo que se debe hacer, no? 😉 Descomentamos **en cada proyecto** las líneas que pertoquen y listos!

Esto básicamente lo que hace es establecer una _custom action build_ que copia las vistas de las áreas hijas dentro de la carpeta Views del área principal. De esta manera dentro de la carpeta Views del área principal tendremos:

Views/Home/About.aspx &ndash;> Vista para la acción Home.About del área principal

Views/Areas/Game/Home/About.aspx &ndash;> Vista para la acción Home.About del área &ldquo;Game&rdquo; (copia de la vista Views/Home/About.aspx) del proyecto que es el área Game.

**Nota Importante:** Para que todo esto funcione el nombre del área **debe** corresponderse con el nombre del ensamblado del proyecto que genera dicha área. En mi caso, el proyecto del área &ldquo;Game&rdquo; debe generar el ensamblado Game.dll. Si el nombre del área y el nombre del ensamblado del proyecto es distinto, no funcionará porque...

... el motor de ASP.NET MVC usa el nombre del área (definido en la llamada a MapAreaRoute) 

...y en cambio la vista estará copiada en Views/Areas/<NombreAssembly>

Si no quereis que el assembly se llame igual que nombre del área podeis modificar la línea:

<pre class="code"><span style="background: black; color: gray">&lt;</span><span style="background: black; color: #3e60fd">CreateAreaManifest </span><span style="background: black; color: silver">AreaName</span><span style="background: black; color: gray">=</span><span style="background: black; color: white">"</span><span style="background: black; color: gray">$(AssemblyName)</span><span style="background: black; color: white">" </span><span style="background: black; color: silver">AreaType</span><span style="background: black; color: gray">=</span><span style="background: black; color: white">"</span><span style="background: black; color: gray">Child</span><span style="background: black; color: white">" <br /></span><span style="background: black; color: silver">AreaPath</span><span style="background: black; color: gray">=</span><span style="background: black; color: white">"</span><span style="background: black; color: gray">$(ProjectDir)</span><span style="background: black; color: white">" </span><span style="background: black; color: silver">ManifestPath</span><span style="background: black; color: gray">=</span><span style="background: black; color: white">"</span><span style="background: black; color: gray">$(AreasManifestDir)</span><span style="background: black; color: white">" <br /></span><span style="background: black; color: silver">ContentFiles</span><span style="background: black; color: gray">=</span><span style="background: black; color: white">"</span><span style="background: black; color: gray">@(Content)</span><span style="background: black; color: white">" </span><span style="background: black; color: gray">/&gt;</span></pre>

[][2]

que hemos descomentado antes y poner en _AreaName_ el nombre lógico del área (el que habéis usado en RegisterMapRoute). De todos modos aunque funciona no os lo recomiendo ya que entonces las vistas de este ensamblado se duplican dentro de Views/Areas... es decir se siguen copiando como Views/Areas/<NombreAssembly> y además como View/Areas/<NuestoNombreDeArea>... o sea que lo más práctico es que los assemblies se llamen como las áreas y listos 🙂

Toda esta paranoia de editar el fichero de proyecto desaparecerá en futuras versiones donde tendremos un template de proyecto para aplicación ASP.NET MVC &ldquo;área principal&rdquo; y otro para &ldquo;área hija&rdquo;... recordad: preview 1 😉

En fin... espero que esto os haya servido para ver un poco que son las áreas en ASP.NET MVC 2.0 Preview 1. Resumiendo:

  * Una área es un proyecto de ASP.NET MVC 2.0 que implementa _parte_ de una aplicación entera (p.ej. el módulo de administración)
  * Una aplicación ASP.NET MVC 2.0 se compone de una (y sólo una) área padre y varias áreas hijas (una por módulo)
  * El área padre proporciona los estilos (css) y ficheros de javascript, así como la inicialización/finalización de la aplicación (global.asax). Puede también proporcionar vistas y controladores
  * Las áreas hijas proporcionan vistas y controladores adicionales
  * El método MapAreaRoute sirve para enrutar determinadas URLs a los controladores de una área determinada
  * Todo está bastante verde todavía pero bueno... 🙂

Saludos!!!

 [1]: /blogs/jdieguez/archive/2009/07/31/asp-net-mvc-2-preview-1.aspx
 [2]: http://11011.net/software/vspaste