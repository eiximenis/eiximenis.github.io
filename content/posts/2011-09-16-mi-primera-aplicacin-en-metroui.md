---
title: Mi primera aplicación en MetroUI
author: eiximenis

date: 2011-09-16T19:38:00+00:00
geeks_url: /?p=1575
geeks_visits:
  - 1965
geeks_ms_views:
  - 1132
categories:
  - Uncategorized

---
Muy buenas! Como muchos otros he descargado el Windows 8 Developers Preview, y he empezado a jugar con la nueva API de WinRT para la creación de aplicaciones basadas en MetroUI.

Vamos a ver como realizar una aplicación MetroUI usando C# que simplemente nos muestre las imágenes que tenemos en la carpeta de &ldquo;Mis Imágenes&rdquo;.

Para ello, abrimos el Visual Studio 2011 Express que viene con el Windows 8 Developers Preview y seleccionamos el tipo de proyecto de tipo &ldquo;Windows Metro Style &ndash;> Application&rdquo;:

[<img height="119" width="244" src="/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_4DC8F3D7.png" alt="image" border="0" style="border-left-width: 0px; border-right-width: 0px; background-image: none; border-bottom-width: 0px; padding-top: 0px; padding-left: 0px; display: inline; padding-right: 0px; border-top-width: 0px" title="image" />][1]

Con eso VS2010 nos genera el esqueleto del proyecto inicial.

**&iexcl;Hey eso es WPF!**

Pues no. Aunque sin duda se le parece mucho. Veamos, por un lado tenemos la definición de la interfaz en XAML. Si tecleamos un poco, vemos que nuestros controles de WPF (o Silverlight) están aquí:

[<img height="39" width="502" src="/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_2B24B55C.png" alt="image" border="0" style="border-left-width: 0px; border-right-width: 0px; background-image: none; border-bottom-width: 0px; padding-top: 0px; padding-left: 0px; display: inline; padding-right: 0px; border-top-width: 0px" title="image" />][2]

Tenemos TextBlock, Button, Grid, StackPanel... con las mismas propiedades y las mismas extensiones de XAML que nos podemos encontrar en WPF. Si no son las mismas son muy, muy parecidas. Y esa es la primera lección que extraemos: **el conocimiento que hemos adquirido desarrollando con WPF o Silverlight no está perdido**. Así pues, tranquilos por este lado: no empezamos de cero!

En mi caso he diseñado una página muy cutre que se compone básicamente de un botón, una etiqueta y una lista. Cuando se pulse el botón la lista debe mostrar los nombres y un thumbnail de las imágenes que tenemos en &ldquo;Mis Imagenes&rdquo;. Esa es la idea. La definición de la vista es muy simple:

[<img height="375" width="497" src="/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_6DD3DAC7.png" alt="image" border="0" style="border-left-width: 0px; border-right-width: 0px; background-image: none; border-bottom-width: 0px; padding-top: 0px; padding-left: 0px; display: inline; padding-right: 0px; border-top-width: 0px" title="image" />][3]

Los que hayáis desarrollado en WPF o Silverlight no veréis nada nuevo. Insisto, todos los conceptos que conocemos están aquí. Estilos, recursos, templates, bindings... Todo funciona exactamente igual.

**WinRT la nueva API que está detrás...**

Aunque este XAML parezca de WPF o Silverlight, realmente _no_ estamos usando WPF ni Silverlight. Eso significa que _por detrás_, es decir en el código C#, tenemos una nueva API que si que nos va a tocar aprender a utilizar... Pero es que si no... donde estaría la diversión?

Bueno, veamos lo primero que quiero es recorrer las imágenes de la carpeta de &ldquo;Mis Imagenes&rdquo; cuando se pulse el botón. Eso, hasta ahora, lo conseguiría usando el método [GetFiles][4] de la clase System.IO.Directory, p.ej. Ahora, no: olvidaos de todo esto. De hecho, si en el código C# tecleáis System.IO:

[<img height="176" width="244" src="/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_3F6A2F4D.png" alt="image" border="0" style="border-left-width: 0px; border-right-width: 0px; background-image: none; border-bottom-width: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px; border-top-width: 0px" title="image" />][5]

veréis que NO sale la clase Directory. No existe! Podéis pensar que a lo mejor falta alguna referencia, pero no... Simplemente no tenemos disponible esta clase. Así que debemos usar la nueva API WinRT que se encuentra colgando del namespace Windows.

Y esa nueva API tiene una característica muy especial: está diseñada de un modo muy asíncrono. Muchas de sus clases tienen métodos cuyo nombre termina en Async() y que son asíncronas. Por suerte en C#5 vamos a disponer de las palabras clave async y await que hacen que llamar a métodos asíncronamente sea lo más fácil del mundo. Ya veréis vais a usar await y async constantemente...

Bien, a lo que íbamos, cuando se pulse el botón quiero que:

  1. El TextBlock de estado diga &ldquo;Obteniendo&rdquo;.
  2. Se obtengan las imágenes que hay en mis imágenes.
  3. Y se enlacen con la ListBox para que se muestren.

Y el código es el siguiente:

[<img height="116" width="529" src="/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_230CC760.png" alt="image" border="0" style="border-top: 0px; border-right: 0px; background-image: none; border-bottom: 0px; padding-top: 0px; padding-left: 0px; border-left: 0px; display: inline; padding-right: 0px" title="image" />][6]

Veis lo que os decía de async y await? Bueno... a ver, este código es muy simple, lo que hace es lo siguiente:

  1. Pone el texto &ldquo;Obteniendo&rdquo; en nuestro TextBlock de estado
  2. Llama al método DisplayImagesAsync(). Método que se ejecutará asíncronamente.
  3. Cuando el método termine, se ejecutará el código que está a continuación del await, es decir pondrá el TextBlock a blanco, para indicar que ya ha terminado.

En este caso, realmente no hay asincronidad. Me explico: el método DisplayImagesAsync() está pensado _para que pueda ser usado asíncronamente_, pero nosotros no hacemos nada entre que llamamos el método y nos esperamos (await) a que termine. Si entre la línea _var displayImagesTask = DisplayImagesAsync()_ y la siguiente (_await displayImagesTask_) yo hubiese colocado código, este código se ejecutaría en paralelo al código del método DisplayImagesAsync(). Esta es la potencia brutal de async/await: facilitan hasta el absurdo la creación de métodos _que pueden ser invocados asíncronamente_ y la _espera para que terminen esos métodos_.

Bueno... veamos el método DisplayImagesAsync(). Para empezar dicho método debe obtener acceso a la carpeta de &ldquo;Mis Imagenes&rdquo;. Para ello usamos la clase _KnownFolders_ de Windows.Storage:

[<img height="24" width="453" src="/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_2F2E4187.png" alt="image" border="0" style="border-top: 0px; border-right: 0px; background-image: none; border-bottom: 0px; padding-top: 0px; padding-left: 0px; border-left: 0px; display: inline; padding-right: 0px" title="image" />][7]

Con eso en picsFolder tenemos un objeto que nos permitirá recorrer la carpeta de &ldquo;Mis Imágenes&rdquo;. Para ello tiene un método GetItemsAsync() que nos devuelve una lista con todos los elementos. Así que, lo invocamos:

[<img height="19" width="314" src="/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_3FC63C75.png" alt="image" border="0" style="border-top: 0px; border-right: 0px; background-image: none; border-bottom: 0px; padding-top: 0px; padding-left: 0px; border-left: 0px; display: inline; padding-right: 0px" title="image" />][8]

Esta forma de usar await es muy común y en el fondo es lo mismo que hemos hecho antes (cuando hemos llamado a DisplayImagesAsync) pero en una sóla linea: invocar el método asíncrono, esperarnos a que termine y en pics tenemos el resultado 🙂

Este resultado es un enumerable con todos los _items_ de la carpeta. Eso incluye ficheros y directorios. Lo que vamos a hacer ahora es iterar por todos los ficheros y por cada fichero obtener un thumbnail junto con su nombre y su ruta:

[<img height="214" width="515" src="/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_1D8E30EF.png" alt="image" border="0" style="border-top: 0px; border-right: 0px; background-image: none; border-bottom: 0px; padding-top: 0px; padding-left: 0px; border-left: 0px; display: inline; padding-right: 0px" title="image" />][9]

Listos! Con esto rellenamos la lista _images_ (una lista dinámica) con los datos (nombre, ruta y thumbnail) de las imágenes que tengamos. Nos queda una pieza para verlo todo y es el método GetBitmapData.

El método GetThumbnailAsync no devuelve directamente un bitmap con el thumbnail, sino que devuelve un objeto (de la clase StorageItemThumbnail) a partir del cual podemos obtener un _stream_ a los datos que representan el bitmap del thumbnail. Pero nosotros queremos algo que sea un ImageSource (para enlazarlo a la propiedad Source del DataTemplate de la ListBox). Y eso que necesitamos es un objeto de la clase BitmapImage. Por suerte es muy fácil obtener un BitmapImage a partir de un StorageItemThumbnail:

[<img height="125" width="459" src="/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_66F05FDD.png" alt="image" border="0" style="border-top: 0px; border-right: 0px; background-image: none; border-bottom: 0px; padding-top: 0px; padding-left: 0px; border-left: 0px; display: inline; padding-right: 0px" title="image" />][10]

Y listos! Con esto lo tenemos todo ya hecho... sólo nos queda enlazar la lista _images_ con nuestra ListBox, y eso lo conseguimos con este código (al final de DisplayImagesAsync), que resultará muy familiar a quien haya desarrollado con WPF o Silverlight:

[<img height="23" width="228" src="/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_5E205791.png" alt="image" border="0" style="border-top: 0px; border-right: 0px; background-image: none; border-bottom: 0px; padding-top: 0px; padding-left: 0px; border-left: 0px; display: inline; padding-right: 0px" title="image" />][11]

Y listos! Con esto conseguimos mostrar las imágenes que tenga el usuario en la carpeta &ldquo;Mis Imágenes&rdquo;:

[<img height="229" width="244" src="/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_23F13F18.png" alt="image" border="0" style="border-top: 0px; border-right: 0px; background-image: none; border-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; border-left: 0px; display: inline; padding-right: 0px" title="image" />][12]

Hemos realizado nuestra primera aplicación MetroUI! 🙂

**Ah sí! Permisos, permisos, permisos**

Que me olvido! Las aplicaciones MetroUI tienen asignado un sistema de seguridad, donde tienen que declarar exactamente que permisos necesitan. Esos permisos se mostraran al usuario cuando este quiera ejecutar la aplicación (o la compre desde el nuevo Windows Store). Para ello es necesario editar el archivo Package.appxmanifest y allí poner los permisos necesarios. Visual Studio 2011 viene con un editor para este archivo. En nuestro caso, debemos pedir específicamente permiso para acceder a la carpeta de imágenes del usuario:

[<img height="179" width="244" src="/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_5F81280B.png" alt="image" border="0" style="border-top: 0px; border-right: 0px; background-image: none; border-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; border-left: 0px; display: inline; padding-right: 0px" title="image" />][13]

Tenemos que ir a &ldquo;Capabilities&rdquo; y marcar la checkbox de _Picture Library Access_.

Ahora sí, que ya lo tenemos todo! 😉

Un saludo!

PD: El código en este post son imágenes porque estoy posteando desde el propio Win8 y no tengo ningún plugin de código instalado en el Live Writer

PD2: He dejado el proyecto (de Visual Studio 2011 Developers Preview) en mi carpeta de skydrive. Lo podeís descargar desde aquí: [MyFirstApp (código fuente)][14]

PD3: No toméis esa aplicación como una guía de buenas prácticas ni nada parecido! Este post es simplemente para compartir mi experiencia de &ldquo;desvirgamiento&rdquo; en WinRT 😛 😛 😛

 [1]: /cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_64B7EB20.png
 [2]: /cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_1E2AD54B.png
 [3]: /cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_705D0C85.png
 [4]: http://msdn.microsoft.com/en-us/library/system.io.directory.getfiles.aspx
 [5]: /cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_677CDE6C.png
 [6]: /cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_3D4D3084.png
 [7]: /cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_7AFA1840.png
 [8]: /cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_4E70E85A.png
 [9]: /cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_25F20646.png
 [10]: /cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_7A118C89.png
 [11]: /cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_3EDDB0BE.png
 [12]: /cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_165ECEAA.png
 [13]: /cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_4F556012.png
 [14]: https://skydrive.live.com/?cid=6521c259e9b1bec6&sc=documents&uc=1&id=6521C259E9B1BEC6%21167 "MyFirstApp"