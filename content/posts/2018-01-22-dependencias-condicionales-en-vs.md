---
title: Dependencias condicionales en VS
description: Dependencias condicionales en VS
author: eiximenis

date: 2018-01-22T16:28:19+00:00
geeks_url: /?p=1987
geeks_ms_views:
  - 824
categories:
  - herramientas
  - Sin categoría
  - visual studio

---
Bueno, imagina que trabajas en un proyecto en NetCore que debe ser multiplataforma. En general el propio framework te provee de todo lo necesario, pero sigamos imaginando que algunas partes de tu proyecto dependen via P/Invoke de llamadas nativas.
  
En este caso puedes optar por tener todos los enlaces P/Invoke para cada plataforma en el mismo proyecto (no hay ningún problema) o bien tenerlos separados en proyectos por cada una de las plataformas.
  
<!--more-->


  
Si este último es tu caso, lo más probable es que quieras que cuando compiles para una plataforma (p. ej. Windows) **no se incluya la referencia al proyecto que contiene las referencias P/Invoke de las otras plataformas** (en nuestro caso Linux y OSX). Igual también tienes código específico para cada plataforma que no quieres que se incluya. La verdad es que &#8220;no pasa nada&#8221; si se incluye, pero eso hace que aumente el tamaño del paquete de tu aplicación.
  
Lo bueno es que **msbuild soporta referencias condicionales**, es decir puedes tener algo como lo siguiente:

<pre class="EnlighterJSRAW" data-enlighter-language="xml">&lt;ProjectReference Include="..\MyProject.Win32\MyProject.Win32.csproj" Condition="'$(TargetsWindows)'=='true'" /&gt;</pre>

**Nota: **En este ejemplo es una referencia a proyecto pero eso mismo funcionaría con otro tipo de referencia (tal como <PackageReference /> a un paquete NuGet).
  
En este caso concreto la referencia solo se incluye si la variable _TargetsWindow_ toma el valor de _true_.
  
¿Y como toma el valor dicha variable? Por un lado se le podría pasar directamente mediante el parametero /p:TargetsWindow=true al invocar msbuild.
  
Aunque déjame que veamos otra aproximación y así de paso aprendemos un par de cosillas sobre msbuild y csproj 😉
  
Personalmente me gusta ir un paso más allá y agrupar los sistemas operativos en &#8220;familias&#8221;. Así Linux y OSX comparten muchas similitudes ya que ambos son sabores de Unix. Podríamos hace algo así:

<pre class="EnlighterJSRAW" data-enlighter-language="xml">&lt;PropertyGroup&gt;
  &lt;TargetFramework&gt;netstandard2.0&lt;/TargetFramework&gt;
  &lt;TargetsLinux&gt;false&lt;/TargetsLinux&gt;
  &lt;TargetsUnix&gt;false&lt;/TargetsUnix&gt;
  &lt;TargetsWindows&gt;false&lt;/TargetsWindows&gt;
  &lt;TargetsOSX&gt;false&lt;/TargetsOSX&gt;
&lt;/PropertyGroup&gt;</pre>

Al principio del csproj declaro las variables con los distintos sabores de SOs que queremos soportar y las pongo todas a _false_. Luego en función del valor de** ****otra variable** (_OSGroup_) establecemos los valores de nuestras variables:

<pre class="EnlighterJSRAW" data-enlighter-language="xml">&lt;Choose&gt;
  &lt;When Condition="'$(OSGroup)'=='Windows_NT'"&gt;
    &lt;PropertyGroup&gt;
      &lt;TargetsWindows&gt;true&lt;/TargetsWindows&gt;
    &lt;/PropertyGroup&gt;
  &lt;/When&gt;
  &lt;When Condition="'$(OSGroup)'=='Linux'"&gt;
    &lt;PropertyGroup&gt;
      &lt;TargetsLinux&gt;true&lt;/TargetsLinux&gt;
      &lt;TargetsUnix&gt;true&lt;/TargetsUnix&gt;
    &lt;/PropertyGroup&gt;
  &lt;/When&gt;
  &lt;When Condition="'$(OSGroup)'=='OSX'"&gt;
    &lt;PropertyGroup&gt;
      &lt;TargetsOSX&gt;true&lt;/TargetsOSX&gt;
      &lt;TargetsUnix&gt;true&lt;/TargetsUnix&gt;
    &lt;/PropertyGroup&gt;
  &lt;/When&gt;
&lt;/Choose&gt;</pre>

Observad el uso de _<Choose/>_ que es básicamente una sucesión de if...else en msbuild: Cada _<When/>_ define la condición que se cumple. En este caso se establecen los valores de _TargetsXXXX_ a true o a false en función del valor de _OSGroup._
  
Por supuesto, tener referencias condicionales no basta: a veces necesitas **código condicional**, es decir usar #if...#endif. Para ello se necesitan tener definidas constantes distintas por cada SO. Esto también se puede hacer desde el csproj:

<pre class="EnlighterJSRAW" data-enlighter-language="xml">&lt;PropertyGroup Condition="'$(TargetsWindows)'=='true'"&gt;
  &lt;DefineConstants&gt;PLATFORM_WINDOWS;$(DefineConstants)&lt;/DefineConstants&gt;
&lt;/PropertyGroup&gt;</pre>

En este caso si el valor de _TargetsWindows_ es true se define la constante de compilación _PLATFORM_WINDOWS. _Constantes similares se definirían para Linux, OSX, el propio Unix como familia y demás.
  
Ahora ya podemos usar msbuild con el parámetro /p:OSGroup=Windows_NT para compilar para WIndows y /p:OSGroup=Linux para compilar para Linux p. ej.
  
**¿Y Visual Studio?**
  
Bueno, pues a Visual Studio no le vengas con milongas: **no hay manera de que VS pase parámetros msbuild**. Es frustrante que eso sea así en VS2017, pero así son las cosas. Por suerte [Hugo Biarge][1] ha acudido a mi rescate y [me ha hablado][2] de [Directory.build.props][3]. Este es un fichero que **sirve para establecer propiedades adicionales de msbuild**. Se busca este fichero en el directorio actual o cualquier directorio que lo contenga lo que es perfecto. En mi caso, p. ej. si quiero compilar la versión de Windows basta con que tenga un fichero _Directory.build.props_ en la raíz del repo con el contenido:

<pre class="EnlighterJSRAW" data-enlighter-language="xml">&lt;Project&gt;
 &lt;PropertyGroup&gt;
  &lt;OSGroup&gt;Windows_NT&lt;/OSGroup&gt;
 &lt;/PropertyGroup&gt;
&lt;/Project&gt;</pre>

Y... ¡voilá! Eso establece el valor de _OsGroup_ a &#8220;Windows_NT&#8221; lo que hace que a su vez el valor de _TargetsWindows_ sea true, lo que hace que se incluya mi dependencia y además se defina la constante de compilación _PLATFORM_WINDOWS_. Y **lo mejor... ¡Visual Studio lo respeta!**
  
No es una solución ideal, pero me permite usar VS sin necesidad de modificar los csprojs y seleccionar (editando dicho fichero) que versión quiero compilar.
  
En resúmen: msbuild y csproj son muy poderosos... bastante más que VS que no soporta todas las casuísticas. Es una pena, pero es así 😉

 [1]: https://twitter.com/hbiarge
 [2]: https://twitter.com/hbiarge/status/955386329822580736
 [3]: https://docs.microsoft.com/en-us/visualstudio/msbuild/customize-your-build