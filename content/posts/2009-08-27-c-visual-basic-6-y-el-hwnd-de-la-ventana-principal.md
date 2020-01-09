---
title: 'C#, Visual Basic 6 y el HWND de la ventana principal…'
author: eiximenis

date: 2009-08-27T13:25:00+00:00
geeks_url: /?p=1462
geeks_visits:
  - 6651
geeks_ms_views:
  - 2214
categories:
  - Uncategorized

---
Hola! ¿Como ha ido el verano? A todos los que hayais disfrutado de unas buenas vacaciones, espero que os hayan sido provechosas... 

Pero como dicen, todo lo bueno se acaba, y toca volver al tajo. En el proyecto en el que estoy, nos hemos visto en la necesidad de comunicarnos con la ventana principal de _otro_ proceso, realizado en Visual Basic 6.

**Sobre ventanas, HWNDs y demás...**

Los que conozcais un poco como funciona internamente Windows, podéis saltaros este apartado y pasar directamente al siguiente 😉 Para el resto una pequeña introducción.

Todos tenemos claro lo que es la _ventana_ de un programa, pero en Windows el concepto de ventana es mucho mas amplio: todo, absolutamente todo lo que se ve en pantalla es, de alguna manera u otra una ventana. Un botón? Es un tipo de ventana específico. Una check-box? Es una ventana. Una label? Es otra ventana, y así con todo lo que os podáis imaginar. E incluso más: los procesos crean muchas ventanas ocultas y que _nunca jamás_ se llegan a mostrar.

¿Para que le puede interesar a un&nbsp; programa crear una ventana que no se muestre nunca? Pues porque las ventanas son ni más ni menos que el método de comunicación de Windows. Cuando el sistema operativo debe informar de un evento, lo que hace es _enviar_ un mensaje a los procesos que deben enterarse de dicho evento. Cada proceso recoje el mensaje y lo envía a la ventana que corresponda. Por suerte .NET se encarga de todo esto (y mucho más) de forma automática, pero si os pica la curiosidad como funciona el proceso echadle un vistazo a métodos del Api de Win32 como [PeekMessage][1], [GetMessage][2] o [DispatchMessage][3]. Las dos primeras son usadas para recojer mensajes de la cola de mensajes del proceso, y la tercera para enrutarlos a la ventana correspondiente.

Cada ventana que se cree (recordad: cada botón, cada label, todo) tiene su _procedimiento de ventana_, que define como actua esta ventana. Si el botón informa de los clicks que se hacen en él, es porque hay código específico en el su procedimiento de ventana para que esto ocurra.

Así una aplicación básica Windows consta de dos elementos fundamentales: El llamado _[bucle de mensajes][4]_, que se encarga de recoger todos los mensajes que Windows envía al proceso) y varios procedimientos de ventana que se encargan de procesar dichos mensajes. La clásica función main() de una aplicación Windows tradicional contiene el bucle de mensaje, y el resto de código está repartido en los distintos procedimientos de ventana.

Para comunicar partes de nuestro programa usamos siempre mensajes: desde el procedimiento de ventana de una ventana, podemos enviar mensajes a cualquier otra ventana usando funciones del API como [SendMessage][5] o [PostMessage][6]. Por eso usamos muchas veces ventanas ocultas: simplemente para poder recibir mensajes y poderlos procesar con código propio.

Cada ventana que se crea tiene un identificador único que se conoce como el HWND de la ventana. Las ventanas también tienen una _clase_. Aunque también define otras cosas podemos asumir que la clase de una ventana determina que procedimiento de ventana se usará: es una manera de que vayas ventanas distintas compartan un mismo procedimiento de ventana. Así p.ej. existe una clase llamada &ldquo;Button&rdquo; que define el procedimiento de ventana de todos los botones de Windows. Si quereis profundizar un poco sobre el tema, os recomiendo este artículo: [Window Classes in Win32][7]. Evidentemente nosotros podemos definir _clases nuevas_ que es lo que debemos hacer cuando queremos tener un procedimiento de ventana propio.

Así que, para resumir: En Windows casi todo son ventanas, que se comunican enviando y recibiendo mensajes (que son gestionados por el procedimiento de ventana) y cada ventana se referencia por su único HWND.

**Visual Basic 6 y la &ldquo;ventana principal&rdquo;**

Generalmente para encontrar la ventana principal de un proceso, podemos usar directamente la propiedad [MainWindowHandle][8] de la clase [Process][9], que nos devuelve un IntPtr con el HWND de la ventana principal. Una vez tenemos el HWND podemos hacer, literalmente, lo que nos de la gana con esta ventana.

Peeeeeeeeeeeeero, resulta que cuando se ejecuta un programa en VB6 se crean _siempre_ al menos dos ventanas:

  1. Una ventana visible (aunque con tamaño 0x0) cuya clase es ThunderRT6Main (nota freak: Thunder es el nombre en clave de VB).
  2. Otra ventana visible, cuya clase es ThunderRT6FormDC y que es en efecto el formulario principal.

Esto es así incluso para el más simple programa en VB6 que muestre una ventana: un formulario vacío que sea el &ldquo;Startup Object&rdquo; del proyecto VB6.

[<img height="102" width="244" src="/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_6CF6E207.png" alt="image" border="0" title="image" style="border-bottom: 0px; border-left: 0px; display: inline; border-top: 0px; border-right: 0px" />][10] 

La captura de pantalla está sacada Spy++, una herramienta que viene con Visual Studio desde tiempos inmemoriales y que sirve para ver todas las ventanas que hay creadas en un momento dado en el sistema. He marcado en rojo la ventana ThunderRT6Main y en azul la ventna ThunderRT6FormDC. El dialogo &ldquo;Property Inspector&rdquo; se corresponde a la ventana &ldquo;Formulari sense Main&rdquo; que es el formulario principal de la aplicación. Podemos ver dos cosas:

  1. La ventana ThunderRT6Main es la **propietaria** de la ventana ThunderRT6FormDC...
  2. ... pero esta no es su hija: ambas son hijas del Desktop (o ambas son &ldquo;ventanas principales&rdquo; por lo que respecta a Windows). Si observais el &ldquo;Property Inspector&rdquo; el handle de la &ldquo;Owner Window&rdquo; es el mismo handle que la ventana ThunderRT6Main.

¿Y cual es el handle que nos devuelve Process.MainWindowHandle? Bueno... pues el nombre de las clases de ambas ventanas ya lo aclara un poco, no? Es el handle de la ventana ThunderRT6Main, una ventana cuyo tamaño es 0x0 y que realmente no vemos (esto se puede observar también con Spy++).

Entonces como obtener el handle a la ventana principal &ldquo;real&rdquo;? Esa es la necesidad que tenía yo en mi proyecto. Una solución es usar [EnumThreadWindows][11]: esta función itera por todas las ventanas principales creadas por un thread determinado, llamando a una función de callback por cada función. Y que podemos hacer en la función callback? Pues encontrar su clase (llamando a [GetClassName][12]) y mirar si es igual a ThunderRT6FormDC.

El código a grandes rasgos sería como sigue:

<pre class="code"><span style="color: green">// Delegate para EnumThreadWindows
</span><span style="color: blue">delegate bool </span><span style="color: #2b91af">EnumThreadWndProc</span>(System.<span style="color: #2b91af">IntPtr </span>hwnd, <span style="color: #2b91af">IntPtr </span>lParam);
<span style="color: blue">class </span><span style="color: #2b91af">Program
</span>{
    [<span style="color: #2b91af">DllImport</span>(<span style="color: #a31515">"user32.dll"</span>, CharSet = <span style="color: #2b91af">CharSet</span>.Auto)]
    <span style="color: blue">static extern int </span>GetClassName(<span style="color: #2b91af">IntPtr </span>hWnd,
        <span style="color: #2b91af">StringBuilder </span>lpClassName, <span style="color: blue">int </span>nMaxCount);
    [<span style="color: #2b91af">DllImport</span>(<span style="color: #a31515">"user32.dll"</span>)]
    [<span style="color: blue">return</span>: <span style="color: #2b91af">MarshalAs</span>(<span style="color: #2b91af">UnmanagedType</span>.Bool)]
    <span style="color: blue">static extern bool </span>EnumThreadWindows
        (<span style="color: blue">uint </span>dwThreadId, <span style="color: #2b91af">EnumThreadWndProc </span>lpfn, <span style="color: #2b91af">IntPtr </span>lParam);
    <span style="color: blue">static void </span>Main(<span style="color: blue">string</span>[] args)
    {
        <span style="color: #2b91af">Process </span>process = <span style="color: blue">new </span><span style="color: #2b91af">Process</span>();
        <span style="color: green">// NoMainProject es un proyecto simple de
        // VB6 que muestra un form
        </span>process.StartInfo = <span style="color: blue">new </span><span style="color: #2b91af">ProcessStartInfo</span>(<span style="color: #a31515">@"C:NoMainProject.exe"</span>);
        process.Start();
        <span style="color: green">// Damos tiempo al proceso VB6 de ponerse en marcha
        </span><span style="color: #2b91af">Thread</span>.Sleep(1500);
        <span style="color: green">// Enumeramos las ventanas del thread principal
        </span>EnumThreadWindows((<span style="color: blue">uint</span>)process.Threads[0].Id,
            FindVB6Window, <span style="color: #2b91af">IntPtr</span>.Zero);
        <span style="color: #2b91af">Console</span>.ReadLine();
    }
    <span style="color: green">// Encuentra la venana cuya clase sea ThunderRT6FormDC
    </span><span style="color: blue">static bool </span>FindVB6Window(<span style="color: #2b91af">IntPtr </span>hwnd, <span style="color: #2b91af">IntPtr </span>lParam)
    {
        <span style="color: #2b91af">StringBuilder </span>sb = <span style="color: blue">new </span><span style="color: #2b91af">StringBuilder</span>(1024);
        <span style="color: green">// Obtenemos el nombre de la clase
        </span>GetClassName(hwnd, sb, sb.Capacity);
        <span style="color: blue">string </span>className = sb.ToString();
        <span style="color: blue">if </span>(className.Equals(<span style="color: #a31515">"ThunderRT6FormDC"</span>))
        {
            <span style="color: #2b91af">Console</span>.WriteLine(<span style="color: #a31515">"HWND ventana ppal 0x{0:X} "</span>,
                hwnd.ToInt32());
            <span style="color: blue">return false</span>;
        }
        <span style="color: blue">return true</span>;
    }
}</pre>

[][13]

Y una captura de la salida, junto con la verificación con Spy++:

[<img height="138" width="244" src="/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_0B970176.png" alt="image" border="0" title="image" style="border-bottom: 0px; border-left: 0px; display: inline; border-top: 0px; border-right: 0px" />][14] 

Evidentemente hay casos más compejos (p.ej. nuestra aplicación VB6 podría crear dos formularios y mostrar ambos) pero la idea de fondo sería la misma (aunque entonces debemos utilizar algún mecanismo adicional además del nombre de clase para discernir cual es la &ldquo;principal&rdquo;).

Saludos!!! 🙂

 [1]: http://msdn.microsoft.com/en-us/library/ms644943(VS.85).aspx
 [2]: http://msdn.microsoft.com/en-us/library/ms644936(VS.85).aspx
 [3]: http://msdn.microsoft.com/en-us/library/ms644934(VS.85).aspx
 [4]: http://msdn.microsoft.com/en-us/library/ms644928(VS.85).aspx
 [5]: http://msdn.microsoft.com/en-us/library/ms644950(VS.85).aspx
 [6]: http://msdn.microsoft.com/en-us/library/ms644944(VS.85).aspx
 [7]: http://msdn.microsoft.com/en-us/library/ms997511.aspx
 [8]: http://msdn.microsoft.com/en-us/library/system.diagnostics.process.mainwindowhandle.aspx
 [9]: http://msdn.microsoft.com/en-us/library/system.diagnostics.process.aspx
 [10]: /cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_2A7FE016.png
 [11]: http://msdn.microsoft.com/en-us/library/ms633495(VS.85).aspx
 [12]: http://msdn.microsoft.com/en-us/library/ms633582(VS.85).aspx
 [13]: http://11011.net/software/vspaste
 [14]: /cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_2FFA6288.png