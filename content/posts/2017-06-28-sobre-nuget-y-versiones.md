---
title: Sobre NuGet y versiones…
description: Sobre NuGet y versiones…
author: eiximenis

date: 2017-06-28T11:51:21+00:00
geeks_url: /?p=1902
geeks_ms_views:
  - 1776
categories:
  - .net

---
Que NuGet ha supuesto una revolución en .NET es más que evidente. Lejos han quedado aquellos tiempos en que gestionábamos las dependencias como podíamos. Poco a poco el modelo de desarrollo está migrando de estar basado en “dependencias a ensamblados” a “dependencias a paquetes”, y a medida que netcore vaya teniendo una mayor relevancia esto irá a más.
  
Pero esta gestión semi-automatizada de las dependencias también trae sus propios quebraderos de cabeza…
  
En nodejs es muy común hablar del “_npm hell_” o el infierno que puede suponer la gestión de paquetes usando npm. Que al cabo de un tiempo alguien se baje el código de tu repositorio y que no le funcione o bien que actualices un paquete y se terminen rompiendo 400 más, es algo muy (demasiado) habitual. ¿Tenemos en .NET un _nuget hell_?
  
<!--more-->


  
**TL;DR**
  
Este es un post bastante largo, así que aquí hay un resúmen de los puntos elementales tratados:

  1. Este post trata sobre NuGet y los nuevos csproj. Algunas cosas pueden ser válidas en versiones anteriores de NuGet y csproj clásicos pero no asumas nada al respecto.
  2. La gestión de paquetes es complicada. No hay una estrategia válida en todos los casos.
  3. Los paquetes NuGet no existen en tiempo de ejecución. Solo tenemos ensamblados.
  4. El csproj referencia a NuGets, pero los ensamblados solo referencian a otros ensamblados
  5. Las versiones de NuGet y de ensamblados pueden ser distintas
  6. NuGet trabaja sólo con versiones de NuGet
  7. El CLR trabaja sólo con versiones de ensamblados
  8. Un paquete NuGet se restaura en una única versión (por cada proyecto). NuGet debe encontrar “cual es la mejor versión” de un paquete teniendo en cuenta todos los paquetes que le referencian.
  9. NuGet intenta proporcionar siempre las versiones más bajas posibles de un paquete
 10. Se puede recibir una versión (de una dependencia) más baja que la que se haya pedido
 11. Se puede recibir una versión (de una dependencia) más alta que la que se haya pedido
 12. Se puede recibir una versión (de una dependencia) fuera del rango de versiones que se hayan pedido
 13. A veces NuGet nos avisa, pero a veces no
 14. Los warnings de NuGet en general son peligrosos

Y ahora si tienes ganas de leer… el post 😉
  
**Preparando el entorno: _UltraConsoleLib_**
  
Para empezar create un directorio vacío que vamos a usar como servidor de NuGet. En mi caso he creado D:\TestNuget, pero puedes usar el que quieras.
  
Vamos a crearnos una librería de clases que sea netstandard1.4. Ya sabes que ahora la buena práctica es intentar que las librerías cumplan netstandard para que así sean utilizables por cualquier plataforma (netfx, netcore, …) que implemente la versión de netstandard que nuestra librería cumple.
  
Nuestra librería, llamada UltraConsoleLib, tendrá una sola clase con un solo método:

<pre style="background: #1e1e1e; color: gainsboro; font-family: consolas; white-space: nowrap; -ms-overflow-x: scroll; -ms-overflow-y: scroll;"><span style="color: #569cd6;">public</span> <span style="color: #569cd6;">class</span> <span style="color: #4ec9b0;">UltraConsole</span>
{
    <span style="color: #569cd6;">public</span> <span style="color: #569cd6;">static</span> <span style="color: #569cd6;">void</span> WriteLine (<span style="color: #569cd6;">string</span> message, <span style="color: #569cd6;">params</span> <span style="color: #569cd6;">object</span>[] args)
    {
        <span style="color: #4ec9b0;">Console</span><span style="color: #b4b4b4;">.</span>WriteLine(<span style="color: #d69d85;">$"UC 1.0 </span>{message}<span style="color: #d69d85;">"</span>, args);
    }
}</pre>

Esa es la versión de UltraConsole 1.0.
  
Ahora vamos a publicar UltraConsole a NuGet. Para ello, desde VS2017 hacemos click con el botón derecho en el proyecto (en el _solution explorer_) y pulsamos “_Pack_”. Pack es el comando para crear un paquete NuGet y eso nos dejará el paquete _UltraConsoleLib.1.0.0.nupkg_ en el directorio bin/Debug. Copia este paquete al directorio vacío que has creado antes.
  
**Creando un cliente de UltraConsoleLib**
  
Esa es fácil: Abre otro VS2017 y antes que nada añade el directorio que hemos usado como servidor de NuGet local. Para ello ve a _Tools –> NuGet Package Manager –> Package Manager Settings –> Package Sources_ y allí añade el directorio que has creado antes. Con eso VS2017 usará también ese directorio para resolver dependencias NuGet.
  
[<img style="display: inline; background-image: none;" title="image" src="https://geeks.ms/etomas/wp-content/uploads/sites/154/2017/06/image_thumb.png" alt="image" width="644" height="437" border="0" />][1]
  
Ahora ya puedes crear un proyecto, que sea también una librería de clases netstandard1.4. Llamemos UltraLogger a esa librería. Lo primero es instalar el paquete “UltraConsoleLib” mediante Install-Package. Como has añadido el “package source” en VS2017 esto te tiene que funcionar:
  
[<img style="display: inline; background-image: none;" title="image" src="https://geeks.ms/etomas/wp-content/uploads/sites/154/2017/06/image_thumb-1.png" alt="image" width="644" height="271" border="0" />][2]
  
Si ahora abres el csproj, verás que se ha añadido un elemento <PackageReference>:

<pre style="background: #1e1e1e; color: gainsboro; font-family: consolas; white-space: nowrap; -ms-overflow-x: scroll; -ms-overflow-y: scroll;"><span style="color: gray;">&lt;</span><span style="color: #569cd6;">PackageReference</span><span style="color: gray;"> </span><span style="color: #92caf4;">Include</span><span style="color: gray;">=</span><span style="color: gray;">"</span><span style="color: #c8c8c8;">UltraConsoleLib</span><span style="color: gray;">"</span><span style="color: gray;"> </span><span style="color: #92caf4;">Version</span><span style="color: gray;">=</span><span style="color: gray;">"</span><span style="color: #c8c8c8;">1.0.0</span><span style="color: gray;">"</span><span style="color: gray;"> /&gt;</span></pre>

**Esta es una nueva capacidad del nuevo csproj**: las referencias a NuGet van directamente dentro del csproj, de forma análoga a como las teníamos en difunto _project.json_.
  
**El fichero NuGet.config**
  
Veamos ahora como configurar el entorno si no usas VS. Lo primer es modificar el fichero _UltraLogger.csproj_ para añadir un elemento <PackageReference> que contiene la referencia. **Solo ten presente que <PackageReference> debe ir dentro de un elemento <ItemGroup>**. Si tienes cualquier <ItemGroup> en el csproj puedes añadir el <PackageReference> en él, y si no lo creas y listos.
  
Una vez tienes la referencia en el csproj debes **añadir un fichero NuGet.config en el mismo directorio del proyecto, con el siguiente contenido**:

<pre style="background: #1e1e1e; color: gainsboro; font-family: consolas; white-space: nowrap; -ms-overflow-x: scroll; -ms-overflow-y: scroll;"><span style="color: gray;">&lt;?</span><span style="color: #569cd6;">xml</span><span style="color: gray;"> </span><span style="color: #92caf4;">version</span><span style="color: gray;">=</span><span style="color: gray;">"</span><span style="color: #c8c8c8;">1.0</span><span style="color: gray;">"</span><span style="color: gray;"> </span><span style="color: #92caf4;">encoding</span><span style="color: gray;">=</span><span style="color: gray;">"</span><span style="color: #c8c8c8;">utf-8</span><span style="color: gray;">"</span><span style="color: gray;">?&gt;</span>
<span style="color: gray;">&lt;</span><span style="color: #569cd6;">configuration</span><span style="color: gray;">&gt;</span>
<span style="color: gray;">  &lt;</span><span style="color: #569cd6;">packageSources</span><span style="color: gray;">&gt;</span>
<span style="color: gray;">    &lt;</span><span style="color: #569cd6;">add</span><span style="color: gray;"> </span><span style="color: #92caf4;">key</span><span style="color: gray;">=</span><span style="color: gray;">"</span><span style="color: #c8c8c8;">repositoryPath</span><span style="color: gray;">"</span><span style="color: gray;"> </span><span style="color: #92caf4;">value</span><span style="color: gray;">=</span><span style="color: gray;">"</span><span style="color: #c8c8c8;">D:\TestNuGet</span><span style="color: gray;">"</span><span style="color: gray;"> /&gt;</span>
<span style="color: gray;">  &lt;/</span><span style="color: #569cd6;">packageSources</span><span style="color: gray;">&gt;</span>
<span style="color: gray;">&lt;/</span><span style="color: #569cd6;">configuration</span><span style="color: gray;">&gt;</span></pre>

(Por supuesto usa el valor de tu directorio en el atributo _value_ del tag _packageSources/add_).
  
Si ahora ejecutas un “dotnet restore” desde la línea de comandos (en el mismo directorio del proyecto) todo debería funcionar:
  
[<img style="display: inline; background-image: none;" title="image" src="https://geeks.ms/etomas/wp-content/uploads/sites/154/2017/06/image_thumb-2.png" alt="image" width="644" height="363" border="0" />][3]
  
Ahora ya puedes editar el código de _UltraLogger_ para crear una clase _UltraLogger_ que use _UltraConsoleLib_:

<pre style="background: #1e1e1e; color: gainsboro; font-family: consolas; white-space: nowrap; -ms-overflow-x: scroll; -ms-overflow-y: scroll;"><span style="color: #569cd6;">public</span> <span style="color: #569cd6;">class</span> <span style="color: #4ec9b0;">UltraLogger</span>
{
    <span style="color: #569cd6;">public</span> <span style="color: #569cd6;">void</span> LogError(<span style="color: #569cd6;">string</span> message, <span style="color: #569cd6;">params</span> <span style="color: #569cd6;">object</span> [] args)
    {
        UltraConsoleLib<span style="color: #b4b4b4;">.</span><span style="color: #4ec9b0;">UltraConsole</span><span style="color: #b4b4b4;">.</span>WriteLine(<span style="color: #d69d85;">"[ERR] "</span> <span style="color: #b4b4b4;">+</span> message, args);
    }
}</pre>

Perfecto. Ahora publica UltraLogger en NuGet. Si usas VS simplemente usa la opción “Pack” desde el _solution explorer_ y si usas la CLI simplemente teclea “_dotnet pack_” desde el directorio del proyecto. Finalmente copia el fichero nupkg al directorio que uses como servidor de NuGet.
  
**Creando el cliente**
  
Vamos a crear un proyecto ahora, que use esas dos referencias. Para ello creamos una aplicación de linea de comandos vacía. Procura que sea una aplicación _netcore._ Eso es porque, lamentablemente, una aplicación netfx todavía usa el csproj antiguo y para nuestras pruebas nos va mejor tener el _csproj_ nuevo. Si no, sería irrelevante ya que nuestras librerías netstandard se pueden usar en netfx y en netcore 🙂
  
Ahora añadimos una referencia a _UltraLogger_  (editando el csproj a mano o bien usando VS e Install-Package). En el método _Main_ de nuestro cliente instanciamos un Logger con un mensaje cualquiera:

<pre style="background: #1e1e1e; color: gainsboro; font-family: consolas; white-space: nowrap; -ms-overflow-x: scroll; -ms-overflow-y: scroll;"><span style="color: #569cd6;">static</span> <span style="color: #569cd6;">void</span> Main(<span style="color: #569cd6;">string</span>[] args)
{
    <span style="color: #569cd6;">new</span> UltraLoggerLib<span style="color: #b4b4b4;">.</span><span style="color: #4ec9b0;">UltraLogger</span>()
        <span style="color: #b4b4b4;">.</span>LogError(<span style="color: #d69d85;">"Test Error"</span>);
}</pre>

Por supuesto esto funciona y se muestra por pantalla “UC 1.0 [ERR] Test Error”. La situación de las dependencias que tenemos ahora es la siguiente:

<pre>Client ---&gt; UltraLogger (1.0.0) ---&gt; UltraConsoleLib (1.0.0)</pre>

**Actualizando UltraConsoleLib**
  
Vamos a crear una versión 1.1 de _UltraConsoleLib_. Para ello abre el VS que tenía ese proyecto y simplemente cambia el método _WriteLine_ para que en lugar de poner “UC 1.0” ponga “UC 1.1”:

<pre style="background: #1e1e1e; color: gainsboro; font-family: consolas; white-space: nowrap; -ms-overflow-x: scroll; -ms-overflow-y: scroll;"><span style="color: #569cd6;">public</span> <span style="color: #569cd6;">class</span> <span style="color: #4ec9b0;">UltraConsole</span>
{
    <span style="color: #569cd6;">public</span> <span style="color: #569cd6;">static</span> <span style="color: #569cd6;">void</span> WriteLine (<span style="color: #569cd6;">string</span> message, <span style="color: #569cd6;">params</span> <span style="color: #569cd6;">object</span>[] args)
    {
        <span style="color: #4ec9b0;">Console</span><span style="color: #b4b4b4;">.</span>WriteLine(<span style="color: #d69d85;">$"UC 1.1 </span>{message}<span style="color: #d69d85;">"</span>, args);
    }
}</pre>

Ahora desde VS vete a las propiedades del proyecto y en la pestaña _package_ coloca la versión 1.1.0:
  
[<img style="display: inline; background-image: none;" title="SNAGHTML68dcaa" src="https://geeks.ms/etomas/wp-content/uploads/sites/154/2017/06/SNAGHTML68dcaa_thumb.png" alt="SNAGHTML68dcaa" width="545" height="467" border="0" />][4]
  
Si no usas VS simplemente añade una etiqueta _<Version>,_  justo después de la etiqueta <_TargetFramework>,_ con el valor de la versión:

<pre style="background: #1e1e1e; color: gainsboro; font-family: consolas; white-space: nowrap; -ms-overflow-x: scroll; -ms-overflow-y: scroll;"><span style="color: gray;">  &lt;</span><span style="color: #569cd6;">PropertyGroup</span><span style="color: gray;">&gt;</span>
<span style="color: gray;">    &lt;</span><span style="color: #569cd6;">TargetFramework</span><span style="color: gray;">&gt;</span><span style="color: #c8c8c8;">netstandard1.4</span><span style="color: gray;">&lt;/</span><span style="color: #569cd6;">TargetFramework</span><span style="color: gray;">&gt;</span>
<span style="color: gray;">    &lt;</span><span style="color: #569cd6;">Version</span><span style="color: gray;">&gt;</span><span style="color: #c8c8c8;">1.1.0</span><span style="color: gray;">&lt;/</span><span style="color: #569cd6;">Version</span><span style="color: gray;">&gt;</span>
<span style="color: gray;">  &lt;/</span><span style="color: #569cd6;">PropertyGroup</span><span style="color: gray;">&gt;</span></pre>

Ahora publica de nuevo el paquete NuGet y copialo a tu directorio que actúa como servidor de NuGet (observa que ahora se llama _UltraConsoleLib.**1.1.0**.nupkg_).
  
**Regla #0 de NuGet**
  
Esa regla nos limitaremos a anunciarla, ya que es muy simple: **Un paquete NuGet se resuelve siempre a una única version**. Es decir, nunca tenemos a la vez un paquete en dos o más versiones distintas.
  
El resto de reglas nos permitirán saber cual es esa versión 🙂
  
**Regla #1 de NuGet**
  
Vayamos a nuestro Cliente, a ver qué ocurre ahora. Para ello vamos a borrar la información de los paquetes restaurados y luego los restauramos de nuevo. Para ello **simplemente borra el fichero obj/project.assets.json** del proyecto y los restauramos de nuevo (“dotnet restore” desde la CLI o bien un Build desde VS). Y así queda la situación:
  
[<img style="display: inline; background-image: none;" title="image" src="https://geeks.ms/etomas/wp-content/uploads/sites/154/2017/06/image_thumb-3.png" alt="image" width="338" height="138" border="0" />][5]
  
Observa que _Client_ depende de _UltraLogger 1.0.0_ que a su vez depende de _UltraConsoleLib_ 1.0.0**.** **Observa que se sigue usando la versión 1.0.0 de _UltraConsoleLib_, no la 1.1.0.** 
  
**_>>> Inicio paréntesis a la regla #1_** 
  
¿Debería usar NuGet la 1.1.0? Bueno, el proyecto _UltraLogger_ depende de _UltraConsoleLib_ en su versión 1.0.0, no en su versión 1.1.0, ¿verdad? Pues **no.** Déjame que te cuente **la regla #1 del PackageReference: Una referencia con versión X.Y.Z no significa que dependamos de la versión X.Y.Z. Significa que dependemos de la versión >= X.Y.Z.**

<pre style="background: #1e1e1e; color: gainsboro; font-family: consolas; white-space: nowrap; -ms-overflow-x: scroll; -ms-overflow-y: scroll;"><span style="color: gray;">&lt;</span><span style="color: #569cd6;">PackageReference</span><span style="color: gray;"> </span><span style="color: #92caf4;">Include</span><span style="color: gray;">=</span><span style="color: gray;">"</span><span style="color: #c8c8c8;">UltraConsoleLib</span><span style="color: gray;">"</span><span style="color: gray;"> </span><span style="color: #92caf4;">Version</span><span style="color: gray;">=</span><span style="color: gray;">"</span><span style="color: #c8c8c8;">1.0.0</span><span style="color: gray;">"</span><span style="color: gray;"> /&gt;</span></pre>

Eso **no significa que dependamos de UltraConsoleLib en su versión 1.0.0. Eso significa que dependemos de UltraConsoleLib con versión >= 1.0.0**. Este >= es importante: para NuGet cualquier versión igual o superior 1.0.0 le sirve. Para comprobarlo haz una prueba rápida:

  1. Mueve la versión 1.0.0 de UltraConsoleLib fuera de NuGet (es decir, ponla en otro directorio que no sea el que usamos como servidor de NuGet)
  2. Borra el fichero obj/project.assets.json para forzar a NuGet a resolver todos los paquetes otra vez
  3. Vacía la cache de nuget mediante el comando “dotnet nuget locals global-packages –c”. Es importante vaciar la cache de NuGet porque si no, a pesar de que la versión 1.0.0 de _UltraConsoleLib_ la has quitado de NuGet, NuGet la seguiría encontrando en la cache.

Ahora ya puedes resolver los paquetes otra vez y verás como _UltraLogger_ recibe la versión 1.1.0 de _UltraConsoleLib_:
  
[<img style="display: inline; background-image: none;" title="image" src="https://geeks.ms/etomas/wp-content/uploads/sites/154/2017/06/image_thumb-4.png" alt="image" width="317" height="119" border="0" />][6]
  
Por lo tanto **eso nos demuestra que podemos recibir una referencia más alta que la que hemos pedido, pero nunca recibiremos una más baja que la que hemos pedido** (vale, a veces sí, ya lo verás luego).
  
**<<< _Fin del paréntesis a la regla #1_**
  
Antes del paréntesis hemos visto como a pesar de haber la versión 1.1.0 de _UltraConsoleLib_, _UltraLogger_ recibía la 1.0.0. El paréntesis nos ha enseñado como podemos recibir una versión “más alta” de la que hemos pedido. Combinando esas dos observaciones podemos obtener la regla #1:

> **Regla #1: NuGet siempre intenta resolver la versión MÁS BAJA POSIBLE de un paquete**.

**Regla #2 de NuGet**
  
Antes de nada acuerdate de volver a colocar la versión 1.0.0 de _UltraConsoleLib_ en NuGet. 🙂
  
El siguiente punto es tener la versión 1.1.0 de _UltraLogger_ que dependa de la versión 1.1.0 de _UltraConsoleLib_: Simplemente cambia la <PackageReference> del csproj de UltraLogger y añade una etiqueta <Version> con 1.1.0. Lanza el comando _Pack_ otra vez y añade el fichero UltraLogger.**1.1.0**.nupkg al directorio que usas como servidor de NuGet.
  
Ahora tenemos dos versiones de UltraLogger: La 1.0.0 que depende de UltraConsoleLib en la 1.0.0 y la 1.1.0 que depende de UltraConsoleLib en la 1.1.0.
  
Finalmente en el Cliente modifica la referencia a UltraLogger para que sea a la 1.1.0 y añade una referencia a UltraConsoleLib en su versión 1.0.0:

<pre style="background: #1e1e1e; color: gainsboro; font-family: consolas; white-space: nowrap; -ms-overflow-x: scroll; -ms-overflow-y: scroll;"><span style="color: gray;">    &lt;</span><span style="color: #569cd6;">PackageReference</span><span style="color: gray;"> </span><span style="color: #92caf4;">Include</span><span style="color: gray;">=</span><span style="color: gray;">"</span><span style="color: #c8c8c8;">UltraLogger</span><span style="color: gray;">"</span><span style="color: gray;"> </span><span style="color: #92caf4;">Version</span><span style="color: gray;">=</span><span style="color: gray;">"</span><span style="color: #c8c8c8;">1.1.0</span><span style="color: gray;">"</span><span style="color: gray;"> /&gt;</span>
<span style="color: gray;">    &lt;</span><span style="color: #569cd6;">PackageReference</span><span style="color: gray;"> </span><span style="color: #92caf4;">Include</span><span style="color: gray;">=</span><span style="color: gray;">"</span><span style="color: #c8c8c8;">UltraConsoleLib</span><span style="color: gray;">"</span><span style="color: gray;"> </span><span style="color: #92caf4;">Version</span><span style="color: gray;">=</span><span style="color: gray;">"</span><span style="color: #c8c8c8;">1.0.0</span><span style="color: gray;">"</span><span style="color: gray;"> /&gt;</span></pre>

Las dependencias que tenemos ahora son las siguientes:

<pre>Client ---&gt; UltraLogger (1.1.0) ---&gt; UltraConsoleLib ---&gt; (1.1.0)
Client ---&gt; UltraConsoleLib (1.0.0)</pre>

Ahora restaura los paquetes otra vez y obtendrás el warning de _Detected package downgrade: UltraConsoleLib from 1.1.0 to 1.0.0_. Si miras las dependencias ahora verás lo siguiente:
  
[<img style="display: inline; background-image: none;" title="image" src="https://geeks.ms/etomas/wp-content/uploads/sites/154/2017/06/image_thumb-5.png" alt="image" width="313" height="159" border="0" />][7]
  
¿Lo ves? **UltraLogger recibe una referencia a la versión 1.0.0 de UltraConsoleLib (a pesar de pedir una versión >= 1.0.0**. Esto, precisamente, es lo que nos indica el _warning_. ¿Por qué ocurre esto? Por la regla #2.

> **Regla #2: Cuando “más arriba” (más cerca del cliente) en al árbol de dependencias está una referencia, más prioridad tiene.**

¿Qué referencias (directas e indirectas) a UltraConsoleLib tenemos en el proyecto? Pues dos:

  1. Client->_UltraConsoleLib (1.0.0)_
  2. Client->_UltraLogger_ (1.1.0) ->_UltraConsoleLib (1.1.0)_

De esas dos, la que está “más arriba” (o más cerca de Client) es la primera. Por tanto esa referencia toma prioridad, respecto al resto de referencias a _UltraConsoleLib_ que aparecen “más abajo” en el árbol de dependencias. Esto está hecho para optimizar el proceso de decidir las dependencias de paquetes (especialmente en grafos grandes como la BCL) pero el precio que se paga es que **podemos recibir una versión inferior a la que hemos pedido**. Cuando eso ocurre, recibiremos el warning de “Detected package downgrade”.
  
**Errores debidos a la Regla #2**
  
Como digo la Regla #2 es potencialmente muy peligrosa. Veamos un ejemplo de ella.
  
Si restauras los paquetes, recibirás el warning de _Detected package downgrade_. Ahora compila y ejecuta el proyecto y… ¡kaboom!

<pre>Unhandled Exception: System.IO.FileLoadException: Could not load file or assembly 'UltraConsoleLib, <strong>Version=1.1.0.0</strong>, Culture=neutral, PublicKeyToken=null'. The located assembly's manifest definition does not match the assembly reference.</pre>

Nos da un error de que no se encuentra _UltraConsoleLib (1.1.0)_. Lógico, porque dicha versión no existe, ya que se ha instalado la versión 1.0.0. Aunque…
  
**¡Ojo con el concepto de versión!**
  
Hasta ahora hemos estado hablando de versiones de paquetes NuGet, pero **en tiempo de ejecución NuGet no existe**. Así que esta “versión 1.1.0.0” no es la versión 1.1.0.0 del paquete NuGet **si no la versión del ensamblado**. Es decir, _la versión de UltraConsoleLib.dll_.
  
Esto te puede resultar confuso, pero recuerda: **una cosa es la versión del paquete NuGet y otra la versión del ensamblado**. NuGet se basa en la versión del paquete al restaurar dependencias, pero el CLR se basa en la versión del ensamblado para cargarlas. Y la clave es que **la versión NuGet y la versión del ensmablado no tienen por qué coincidir**. De hecho en el fichero csproj tenemos dos etiquetas distintas:

  * <Version> que indica la versión
  * <PackageVersion> que indica la versión **de NuGet**. Si no existe, se usa el valor de <Version>
  * <AssemblyVersion> que indica la versión del ensamblado. Si no existe se usa el valor de <Version>

Lo que nos importa a nosotros es que NuGet se fija en PackageVersion y el CLR en AssemblyVersion. Esto nos da la herramienta que necesitamos para intentar seguir…
  
**Semantic versioning (semver)**
  
No voy a entrar a discutir que es [semver][8], pero he aquí **las reglas para (intentar) seguir semver correctamente en NuGet**. Cuando crees un paquete NuGet:

  * **Si modificas** la versión **PATCH** la **versión del ensamblado no debe verse modificada**. Es decir, paquetes NuGet con versiones 3.1.0, 3.1.1 y 3.1.2 deben tener la misma versión del ensamblado.
  * **Si modificas** la versión **MINOR puedes modificar la versión del ensamblado**, pero entonces ten presente que si se produce un _downgrade package_ la aplicación afectada dejará de funcionar. Si al modificar la versión MINOR no modificas la versión del ensamblado, entonces la aplicación afectada solo dejará de funcionar si llama a cualquier método que hayas agregado nuevo en dicha versión. P. ej. en nuestro caso, si _UltraConsoleLib (1.1.0)_ no hubiese cambiado la versión de _UltraConsoleLib.dll_ el programa seguiría funcionando.
  * **Si modificas** la versión **MAJOR deberías modificar la versión del ensamblado**. Una versión MAJOR incorpora _breaking changes_, así que ya está bien que las aplicaciones afectadas por un _downgrade package_ se vean afectadas.

Como desarrollador, si usas un paquete NuGet y recibes un _downgrade package_ no deberías preocuparte si las versiones afectadas son de PATCH (p. ej. un _downgrade_ de la 9.1.2 a la 9.1.0). Por otro lado si son de MINOR o MAJOR entonces ten por seguro que, probablemente, tendrás problemas.
  
**Versiones de ensamblado… un poco más**
  
Hasta ahora hemos visto como en el csproj establecemos una dependencia a una versión de NuGet. Ten presente que la traducción entre versiones NuGet y versiones de DLL se realiza cuando se compila el proyecto. P. ej., volvamos al proyecto UltraLogger (1.0.0) que tiene una <PackageReference> a UltraConsoleLib en su versión 1.0.0. Supongamos que dicha versión **no existe en NuGet.** Supongamos que en NuGet solo existe la 1.1.0
  
En este caso cuando restaures los paquetes recibirás un _warning_ de NuGet:

<pre>Dependency specified was UltraConsoleLib (&gt;= 1.0.0) but ended up with UltraConsoleLib 1.1.0.</pre>

Este _warning_ es relativamente peligroso porque nos está diciendo que UltraLogger esperaba la versión 1.0.0 de UltraConsoleLib pero ha terminado con la 1.1.0. Si ahora compilamos el proyecto de UltraLogger es cuando se crea _UltraLogger.dll_ y se crea la referencia al ensamblado de _UltraConsoleLib_. En mi caso la versión del ensamblado coincide con la de NuGet. Es decir:

  1. UltraConsoleLib (1.0.0) –> Contiene _UltraConsoleLib v1.0,0_
  2. UltraConsoleLib (1.1.0) –> Contiene _UltraConsoleLib v1.1.0_

La situación es que _UltraLogger_ declara en el csproj una referencia a _UltraConsoleLib (1.0.0)_ pero como no existía ha terminado con _UltraConsoleLib (1.1.0)_. Al compilar eso significa que el ensamblado _UltraLogger.dll_ tendrá una referencia a _UltraConsoleLib.dll_ en su versión 1.1.0. De hecho si compilamos y abrimos _UltraLogger.dll_ con ildasm podremos ver las siguientes líneas en el MANIFEST del ensamblado:

<pre>.assembly extern UltraConsoleLib
{
   .ver 1:1:0:0
}</pre>

Eso es la referencia de _UltraLogger.dll_ hacia _UltraConsoleLib.dll_ en su versión 1.1.0.0. Eso es lo que ve el CLR y eso es lo que busca. Nada de versiones de NuGets.
  
Ahora imagina un Cliente que tiene una <PackageReference> hacia este UltraLogger (1.0.0) que está en NuGet y compilamos este Cliente **en un entorno cuyo NuGet sí que tiene UltraConsoleLib (1.0.0).** Cuando restaures los paquetes:

  * NuGet restaurará UltraLogger (1.0.0). Y NuGet verá que UltraLogger (1.0.0) tiene una dependencia de UltrConsoleLib (1.0.0) (eso está en los metadatos del nupkg), así que…
  * NuGet restaurará UltraConsoleLib (1.0.0) (que ahora sí que existe).

Así ahora terminamos con UltraLogger.dll v1.0.0 y con UltraConsoleLib.dll v1.0.0. Pero si recuerdas UltraLogger.dll v1.0.0 tenía una referencia a UltraConsoleLib.dll v.1.1.0 que **no** está. ¿El resultado? El siguiente:

<pre>System.IO.FileLoadException: Could not load file or assembly 'UltraConsoleLib, Version=1.1.0.0, Culture=neutral, PublicKeyToken=null'. The located assembly's manifest definition does not match the assembly reference.</pre>

Por lo tanto: Es importante que entiendas la diferencia entre versión de NuGet, versión de ensamblado y que la referencia entre ensamblados se crea al momento en que se construye el proyecto. Y si ves el warning de _Dependency specified was xxx but ended up with yyy_, vigila porque eso puede dar problemas en un futuro: básicamente puedes asumir que los dará si ese warning te aparece debido a que te falta un _feed_ de NuGet configurado.
  
**Rangos de versiones en csproj**
  
Recuerda que una <PackageReference> a una versión X.Y.Z significa que nos conformamos con cualquier versión igual o superior a X.Y.Z. Hemos visto que NuGet nos intentará dar la versión más baja posible que cumpla la condición (regla #1) y hemos visto como a veces podemos terminar con una versión más baja (por la regla #2) lo que nos puede causar problemas.
  
Pero podemos especificar rangos de versiones en el csproj. Simplemente en lugar de una versión, especifica un rango abierto (con paréntesis) o cerrado (con corchetes) entre versiones:

<pre style="background: #1e1e1e; color: gainsboro; font-family: consolas; white-space: nowrap; -ms-overflow-x: scroll; -ms-overflow-y: scroll;"><span style="color: gray;">&lt;</span><span style="color: #569cd6;">PackageReference</span><span style="color: gray;"> </span><span style="color: #92caf4;">Include</span><span style="color: gray;">=</span><span style="color: gray;">"</span><span style="color: #c8c8c8;">UltraConsoleLib</span><span style="color: gray;">"</span><span style="color: gray;"> </span><span style="color: #92caf4;">Version</span><span style="color: gray;">=</span><span style="color: gray;">"</span><span style="color: #c8c8c8;">[1.0.0, 2.0.0)</span><span style="color: gray;">"</span><span style="color: gray;"> /&gt;</span></pre>

Eso significa que aceptamos cualquier versión **mayor o igual a 1.0.0 e inferior a 2.0.0.** 
  
Imagina que tenemos esto en UltraLogger (1.0.0). Ahora **vamos a crear otro paquete de NuGet** que use UltraConsoleLib. En mi caso he llamado _UltraDebugger (1.0.0)_ a este paquete de NuGet y tiene las siguientes <PackageReferences>:

<pre style="background: #1e1e1e; color: gainsboro; font-family: consolas; white-space: nowrap; -ms-overflow-x: scroll; -ms-overflow-y: scroll;"><span style="color: gray;">&lt;</span><span style="color: #569cd6;">PackageReference</span><span style="color: gray;"> </span><span style="color: #92caf4;">Include</span><span style="color: gray;">=</span><span style="color: gray;">"</span><span style="color: #c8c8c8;">UltraConsoleLib</span><span style="color: gray;">"</span><span style="color: gray;"> </span><span style="color: #92caf4;">Version</span><span style="color: gray;">=</span><span style="color: gray;">"</span><span style="color: #c8c8c8;">2.0.0</span><span style="color: gray;">"</span><span style="color: gray;"> /&gt;</span></pre>

Ahora vamos al Cliente, que como podrás suponer tiene las referencias:

<pre style="background: #1e1e1e; color: gainsboro; font-family: consolas; white-space: nowrap; -ms-overflow-x: scroll; -ms-overflow-y: scroll;"><span style="color: gray;">    &lt;</span><span style="color: #569cd6;">PackageReference</span><span style="color: gray;"> </span><span style="color: #92caf4;">Include</span><span style="color: gray;">=</span><span style="color: gray;">"</span><span style="color: #c8c8c8;">UltraLogger</span><span style="color: gray;">"</span><span style="color: gray;"> </span><span style="color: #92caf4;">Version</span><span style="color: gray;">=</span><span style="color: gray;">"</span><span style="color: #c8c8c8;">1.0.0</span><span style="color: gray;">"</span><span style="color: gray;"> /&gt;</span>
<span style="color: gray;">    &lt;</span><span style="color: #569cd6;">PackageReference</span><span style="color: gray;"> </span><span style="color: #92caf4;">Include</span><span style="color: gray;">=</span><span style="color: gray;">"</span><span style="color: #c8c8c8;">UltraDebugger</span><span style="color: gray;">"</span><span style="color: gray;"> </span><span style="color: #92caf4;">Version</span><span style="color: gray;">=</span><span style="color: gray;">"</span><span style="color: #c8c8c8;">1.0.0</span><span style="color: gray;">"</span><span style="color: gray;"> /&gt;</span></pre>

Si ahora restauramos los paquetes de NuGet… obtendremos un error:

<pre>Version conflict detected for UltraConsoleLib.
  Client (&gt;= 1.0.0) -&gt; UltraDebugger (&gt;= 1.0.0) -&gt; UltraConsoleLib (&gt;= 2.0.0)
  Client (&gt;= 1.0.0) -&gt; UltraLogger (&gt;= 1.0.0) -&gt; UltraConsoleLib (&gt;= 1.0.0).
</pre>

El error se da porque tenemos el siguiente árbol de dendencias:

<pre>Client ---&gt; UltraDebugger &gt;= (1.0.0) ---&gt; UltraConsoleLib &gt;= (2.0.0)
Client ---&gt; UltraLogger &gt;= (1.0.0) ---&gt; UltraConsoleLib &gt;= (1.0.0) && &lt; (2.0.0)</pre>

O sera, UltraDebugger pide como mínimo la versión 2.0.0 de UltraConsoleLib y UltraLogger pide como mínimo la 1.0.0 y como máximo cualquier inferior a la 2.0.0 (recuerda el rango que metimos en el <PackageReference>. Así pues.. NuGet no puede resolver los paquetes.
  
**¿Como solucionar este conflicto?** Pues aplicando la regla #2. En este caso las dos dependencias a _UltraConsoleLib_ estan “a la misma distancia” de Client (en ambos casos hay una dependencia por en medio). Pero si p. ej. añadimos a Client una referencia directa a una versión concreta de _UltraConsoleLib_ esa dependencia (por la regla #2) mandará sobre las otras dos (es la que esá más arriba en el árbol). Así, si añadimos a Client la siguiente <PackageReference>:

<pre style="background: #1e1e1e; color: gainsboro; font-family: consolas; white-space: nowrap; -ms-overflow-x: scroll; -ms-overflow-y: scroll;"><span style="color: gray;">&lt;</span><span style="color: #569cd6;">PackageReference</span><span style="color: gray;"> </span><span style="color: #92caf4;">Include</span><span style="color: gray;">=</span><span style="color: gray;">"</span><span style="color: #c8c8c8;">UltraConsoleLib</span><span style="color: gray;">"</span><span style="color: gray;"> </span><span style="color: #92caf4;">Version</span><span style="color: gray;">=</span><span style="color: gray;">"</span><span style="color: #c8c8c8;">2.0.0</span><span style="color: gray;">"</span><span style="color: gray;"> /&gt;</span></pre>

Entonces ya podrás resolver los paquetes y todo el mundo recibirá _UltraConsoleLib (2.0.0):_
  
[<img style="display: inline; background-image: none;" title="image" src="https://geeks.ms/etomas/wp-content/uploads/sites/154/2017/06/image_thumb-6.png" alt="image" width="311" height="256" border="0" />][9]
  
Observa como UltraLogger recibe (por la regla #2) la versión (2.0.0) de UltraConsoleLib a pesar de estar fuera del rango. Lamentablemente **no recibimos ningún warning en este caso**, lo que (en mi opinión) es bastante peligroso, ya que claramente un paquete recibe una versión “incorrecta” de una dependencia.
  
Por supuesto si la <PackageReference> del Client fuese:

<pre style="background: #1e1e1e; color: gainsboro; font-family: consolas; white-space: nowrap; -ms-overflow-x: scroll; -ms-overflow-y: scroll;"><span style="color: gray;">&lt;</span><span style="color: #569cd6;">PackageReference</span><span style="color: gray;"> </span><span style="color: #92caf4;">Include</span><span style="color: gray;">=</span><span style="color: gray;">"</span><span style="color: #c8c8c8;">UltraConsoleLib</span><span style="color: gray;">"</span><span style="color: gray;"> </span><span style="color: #92caf4;">Version</span><span style="color: gray;">=</span><span style="color: gray;">"</span><span style="color: #c8c8c8;">1.0.0</span><span style="color: gray;">"</span><span style="color: gray;"> /&gt;</span></pre>

Entonces tanto UltraLogger (1.0.0) como UltraDebugger (1.0.0) recibirían la versión (1.0.0) de UltraConsoleLib, y entonces el paquete que recibe una versión “incorrecta” es UltraDebugger. Pero en este caso NuGet sí que nos avisa con un _downgrade package version detected_.
  
O sea, **NuGet nos avisa cuando un paquete recibe una versión inferior a la pedida, pero no nos avisa cuando un paquete recibe una versión superior a la máxima admitida**.
  
Bueno… y creo que más o menos eso es todo lo que tenía pendiente de contar sobre NuGet y versiones. Como podéis ver… ¡hay más tela que la que parece!

 [1]: https://geeks.ms/etomas/wp-content/uploads/sites/154/2017/06/image.png
 [2]: https://geeks.ms/etomas/wp-content/uploads/sites/154/2017/06/image-1.png
 [3]: https://geeks.ms/etomas/wp-content/uploads/sites/154/2017/06/image-2.png
 [4]: https://geeks.ms/etomas/wp-content/uploads/sites/154/2017/06/SNAGHTML68dcaa.png
 [5]: https://geeks.ms/etomas/wp-content/uploads/sites/154/2017/06/image-3.png
 [6]: https://geeks.ms/etomas/wp-content/uploads/sites/154/2017/06/image-4.png
 [7]: https://geeks.ms/etomas/wp-content/uploads/sites/154/2017/06/image-5.png
 [8]: http://semver.org/
 [9]: https://geeks.ms/etomas/wp-content/uploads/sites/154/2017/06/image-6.png