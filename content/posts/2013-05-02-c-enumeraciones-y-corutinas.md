---
title: '[C#]–Enumeraciones y corutinas'
description: '[C#]–Enumeraciones y corutinas'
author: eiximenis

date: 2013-05-02T17:10:07+00:00
geeks_url: /?p=1638
geeks_visits:
  - 2584
geeks_ms_views:
  - 1740
categories:
  - Uncategorized

---
¡Buenas! 

**Empezamos con una pregunta:**

¿Cual es el resultado de este programa?

<div style="font-size: 10pt; font-family: consolas; background: #1e1e1e; color: #dcdcdc">
  <p style="margin: 0px">
    <span style="color: #569cd6">class</span> <span style="color: #4ec9b0">Program</span>
  </p>
  
  <p style="margin: 0px">
    {
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: #569cd6">static</span> <span style="color: #569cd6">void</span> <span style="color: white">Main</span>(<span style="color: #569cd6">string</span>[] <span style="color: white">args</span>)
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; {
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #569cd6">var</span> <span style="color: white">data</span> <span style="color: #b4b4b4">=</span> <span style="color: white">Foos</span>;
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #569cd6">foreach</span> (<span style="color: #569cd6">var</span> <span style="color: white">foo</span> <span style="color: #569cd6">in</span> <span style="color: white">data</span>)
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; {
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: white">ChangeFooValue</span>(<span style="color: white">foo</span>);
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; }
  </p>
  
  <p style="margin: 0px">
    &#160;
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #569cd6">var</span> <span style="color: white">firstFoo</span> <span style="color: #b4b4b4">=</span> <span style="color: white">data</span><span style="color: #b4b4b4">.</span><span style="color: white">First</span>();
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #4ec9b0">Console</span><span style="color: #b4b4b4">.</span><span style="color: white">WriteLine</span>(<span style="color: white">firstFoo</span><span style="color: #b4b4b4">.</span><span style="color: white">Value</span>);
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #4ec9b0">Console</span><span style="color: #b4b4b4">.</span><span style="color: white">ReadLine</span>();
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; }
  </p>
  
  <p style="margin: 0px">
    &#160;
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: #569cd6">private</span> <span style="color: #569cd6">static</span> <span style="color: #b8d7a3">IEnumerable</span><span style="color: #b4b4b4"><</span><span style="color: #4ec9b0">Foo</span><span style="color: #b4b4b4">></span> <span style="color: white">Foos</span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; {
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #569cd6">get</span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; {
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #569cd6">for</span> (<span style="color: #569cd6">var</span> <span style="color: white">idx</span> <span style="color: #b4b4b4">=</span> <span style="color: #b5cea8">1</span>; <span style="color: white">idx</span> <span style="color: #b4b4b4"><=</span> <span style="color: #b5cea8">10</span>; <span style="color: white">idx</span><span style="color: #b4b4b4">++</span>)
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; {
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #569cd6">yield</span> <span style="color: #569cd6">return</span> <span style="color: #569cd6">new</span> <span style="color: #4ec9b0">Foo</span>(<span style="color: white">idx</span>);
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; }
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; }
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; }
  </p>
  
  <p style="margin: 0px">
    &#160;
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: #569cd6">private</span> <span style="color: #569cd6">static</span> <span style="color: #569cd6">void</span> <span style="color: white">ChangeFooValue</span>(<span style="color: #4ec9b0">Foo</span> <span style="color: white">foo</span>)
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; {
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: white">foo</span><span style="color: #b4b4b4">.</span><span style="color: white">Value</span> <span style="color: #b4b4b4">=</span> <span style="color: white">foo</span><span style="color: #b4b4b4">.</span><span style="color: white">Value</span> <span style="color: #b4b4b4">+</span> <span style="color: #b5cea8">10</span>;
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; }
  </p>
  
  <p style="margin: 0px">
    }
  </p>
  
  <p style="margin: 0px">
    &#160;
  </p>
  
  <p style="margin: 0px">
    <span style="color: #569cd6">internal</span> <span style="color: #569cd6">class</span> <span style="color: #4ec9b0">Foo</span>
  </p>
  
  <p style="margin: 0px">
    {
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: #569cd6">public</span> <span style="color: #569cd6">int</span> <span style="color: white">Value</span> { <span style="color: #569cd6">get</span>; <span style="color: #569cd6">set</span>; }
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: #569cd6">public</span> <span style="color: white">Foo</span>(<span style="color: #569cd6">int</span> <span style="color: white">i</span>)
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; {
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: white">Value</span> <span style="color: #b4b4b4">=</span> <span style="color: white">i</span>;
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; }
  </p>
  
  <p style="margin: 0px">
    }
  </p></p>
</div>

¿Ya lo has meditado? 

**Pues ahora la solución…**

Aunque el sentido común te pueda decir que el valor que se imprimirá es 11, el valor que realmente se imprime es 1.

¿Y eso? A priori todo parece claro: Tenemos una propiedad llamada Foos que nos devuelve 10 objetos Foo (con valores del 1 al 10). Nos guardamos dichos valores en _data_, iteramos sobre ellos y añadimos 10 al valor de cada objeto Foo. Luego imprimimos el valor del primero de esos objetos. Todo está perfecto, _salvo_ que el resultado es 1 y no 11 como debería ser.

La clave es en como está definida la propiedad Foo (usando yield return) lo que impacta directamente en lo que es la variable _data_. Por qué… ¿Qué es data? ¿Es una lista? ¿Es un array? ¿Alguien lo sabe?

Cuando usamos yield return **no** se crea un espacio de almacenamiento para guardar los valores que vamos devolviendo. Se van creando uno tras otro tantas veces como se necesita. Realmente _data_ no contiene ningún objeto (la frase que he usado antes de “Nos guardamos dichos valores en data” es totalmente inexacta), es como un _apuntador_ a “una colección virtual (de 10 elementos)”. Por eso cuando iteramos con el foreach pasamos 10 veces por el yield return y cuando luego usamos el .First() pasamos _otra_ vez más por el yield return. Aunque _antes_ en el foreach se han recuperado los 10 elementos de Foos, como no están guardados en ningún sitio, al hacer .First() se vuelve a recuperar el primero. Lo que crea un Foo nuevo cuyo valor es 1. De ahí el resultado final.

Si usas Resharper, que sepas que te va avisar:

[<img title="image" style="border-left-width: 0px; border-right-width: 0px; background-image: none; border-bottom-width: 0px; padding-top: 0px; padding-left: 0px; display: inline; padding-right: 0px; border-top-width: 0px
" border="0" alt="image" src="http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_55A29B8F.png" width="504" height="73" />][1]

Este aviso, simplemente te indica que se está recorriendo más de una vez el mismo IEnumerable (en este ejemplo se recorre una vez con el foreach y otra vez al usar .First()). Recorrer dos veces un IEnumerable _puede_ tener consecuencias no esperadas o puede ser perfectamente correcto. Como Resharper no tiene manera de saberlo, te avisa, para que le eches un vistazo.

Esto es así tan solo si en algún momento NO se guardan en ningún sitio los elementos recuperados. Basta con hacer:

<div style="font-size: 10pt; font-family: consolas; background: #1e1e1e; color: #dcdcdc">
  <p style="margin: 0px">
    <span style="color: #569cd6">var</span> <span style="color: white">data</span> <span style="color: #b4b4b4">=</span> <span style="color: white">Foos</span><span style="color: #b4b4b4">.</span><span style="color: white">ToList</span>();
  </p></p>
</div>

Ahora el resultado final es 11, ya que el método .ToList() _copia_ los valores a una List<T> por lo que ahora data si que tiene almacenados los 10 valores Foo. Y cuando hacemos data.First() recuperamos simplemente el primer valor almacenado en _data_.

**Un caso real…**

Bien, probablemente pienses que esto tampoco tiene mucha importancia porque tu no vas por ahí haciendo “colecciones virtuales” con yield, pero que sepas que hay alguien que si que lo hace: alguien llamado EF. En un proyecto en el que estuve nos encontramos con este comportamiento precisamente con el uso de EF: Usábamos EF para recuperar ciertos registros de la BBDD que luego convertíamos a DTOs con un método extensor que dado un IEnumerable<T> devolvía un IEnumerable<TDto>. Dicho método extensor iteraba sobre el IEnumerable<T> original y devolvia otro IEnumerable de Dtos:

<div style="font-size: 10pt; font-family: consolas; background: #1e1e1e; color: #dcdcdc">
  <p style="margin: 0px">
    <span style="color: #569cd6">public</span> <span style="color: #569cd6">static</span> <span style="color: #b8d7a3">IEnumerable</span><span style="color: #b4b4b4"><</span>R<span style="color: #b4b4b4">></span> <span style="color: white">Map</span><span style="color: #b4b4b4"><</span><span style="color: white">T</span>, <span style="color: white">R</span><span style="color: #b4b4b4">></span>(<span style="color: #569cd6">this</span> <span style="color: #b8d7a3">IEnumerable</span><span style="color: #b4b4b4"><</span>T<span style="color: #b4b4b4">></span> <span style="color: white">source</span>)
  </p>
  
  <p style="margin: 0px">
    {
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: #569cd6">foreach</span> (<span style="color: #569cd6">var</span> <span style="color: white">t</span> <span style="color: #569cd6">in</span> <span style="color: white">source</span>)
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; {
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #569cd6">yield</span> <span style="color: #569cd6">return</span> <span style="color: #4ec9b0">Mapper</span><span style="color: #b4b4b4">.</span><span style="color: white">Map</span><span style="color: #b4b4b4"><</span>T, R<span style="color: #b4b4b4">></span>(<span style="color: white">t</span>);
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; }
  </p>
  
  <p style="margin: 0px">
    }
  </p></p>
</div>

(En nuestro caso la clase Mapper era <a href="https://github.com/AutoMapper/AutoMapper" target="_blank" rel="noopener noreferrer">AutoMapper</a>).

A priori parece todo correcto: Obtenemos datos de EF y los mapeamos a DTOs. Si nos limitamos a devolver el IEnumerable obtenido por Map<T,R> todo va perfecto. El problema es si _modificamos_ alguno de los DTOs que hemos obtenido. Cuando volvemos a iterar sobre la variable que contiene los resultados volvemos a obtener los resultados _originales_. Eso es debido a que EF crea una colección virtual (es decir usa yield return) para devolvernos los resultados y nuestro Map<T,R> hace lo mismo, por lo que en ningún momento hemos “guardado” esos datos en ningún sitio. Estamos en la misma situación que el ejemplo inicial de este post.

La solución pasa por _materializar_ (es decir llamar a .ToList() o .ToArray()) el resultado devuelto por EF.

Para finalizar solo comentar que el uso de yield return en C# es un ejemplo de lo que se conoce como “<a href="http://en.wikipedia.org/wiki/Coroutine" target="_blank" rel="noopener noreferrer">corutina</a>”. Una corutina no deja de ser una subrutina (es decir una función o pedazo de código) pero que es reentrante en distintos puntos. Otro ejemplo de corutinas en C# lo puedes encontrar en el uso de la palabra clave <a href="http://msdn.microsoft.com/en-us/library/vstudio/hh156528.aspx" target="_blank" rel="noopener noreferrer">await</a> (en C# 5).

¡Un saludo!

 [1]: http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_1DD0576C.png