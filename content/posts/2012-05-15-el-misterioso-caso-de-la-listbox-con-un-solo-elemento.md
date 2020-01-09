---
title: El misterioso caso de la ListBox con un solo elemento
author: eiximenis

date: 2012-05-15T15:26:31+00:00
geeks_url: /?p=1597
geeks_visits:
  - 1603
geeks_ms_views:
  - 1463
categories:
  - Uncategorized

---
Un post rapidito, para comentar algo que sucedió ayer…

Ayer por la tarde puse el siguiente tweet: [http://twitter.com/#!/eiximenis/status/202060274260389888][1]. Básicamente mostraba una ListBox en la cual tras añadirle un único elemento soltaba una OutOfMemoryException indicando que había demasiados elementos en la dicha lista:

[<img style="border-right-width: 0px; display: inline; border-top-width: 0px; border-bottom-width: 0px; border-left-width: 0px" title="As3cv7bCMAEOizK" border="0" alt="As3cv7bCMAEOizK" src="http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/As3cv7bCMAEOizK_5F00_thumb_5F00_4BA83636.png" width="506" height="184" />][2]&#160;

Vale que winforms tiene sus limitaciones, pero eso parece un poco excesivo, ¿no?

Mirando el valor de lstComandos.Count puedo ver que el elemento se ha añadido (antes de hacer el Add la lista estaba vacía) pero que después me lanza la excepción.

La propiedad InnerException está vacía:

[<img style="border-right-width: 0px; display: inline; border-top-width: 0px; border-bottom-width: 0px; border-left-width: 0px" title="image" border="0" alt="image" src="http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_6C51908C.png" width="461" height="149" />][3] 

Bueno… tras una rápida investigación (basada en F9 y F5) pude elaborar una suposición de lo que ocurría. El objeto que añadía a la lista era de una clase tal como la siguiente:

<div style="font-family: courier new; background: white; color: black; font-size: 10pt">
  <p style="margin: 0px">
    <span style="color: blue">class</span> <span style="color: #2b91af">Comando</span>
  </p>
  
  <p style="margin: 0px">
    {
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: blue">private</span> <span style="color: blue">readonly</span> <span style="color: #2b91af">Guid</span> _id;
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: blue">public</span> Comando()
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; {
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; _id = <span style="color: #2b91af">Guid</span>.NewGuid();
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; }
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: blue">public</span> <span style="color: #2b91af">Guid</span> Id { <span style="color: blue">get</span> { <span style="color: blue">return</span> _id; } }
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: blue">public</span> <span style="color: blue">string</span> Name { <span style="color: blue">get</span>; <span style="color: blue">set</span>; }
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: blue">public</span> <span style="color: blue">override</span> <span style="color: blue">string</span> ToString()
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; {
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: blue">return</span> Name;
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; }
  </p>
  
  <p style="margin: 0px">
    }
  </p></p>
</div>

Y como lo añadia a la lista:

<div style="font-family: courier new; background: white; color: black; font-size: 10pt">
  <p style="margin: 0px">
    lstComandos.Items.Add(<span style="color: blue">new</span> <span style="color: #2b91af">Comando</span>());
  </p></p>
</div>

Este Add ya daba la excepción antes mencionada.

¿Cuál es el verdadero problema? Pues simple y llanamente que el método ToString() (que es el que llama la ListBox para convertir los objetos de la clase Comando en una cadena para mostrar) devuelve null.

Basta con modificar el código del ToString:

<div style="font-family: courier new; background: white; color: black; font-size: 10pt">
  <p style="margin: 0px">
    <span style="color: blue">public</span> <span style="color: blue">override</span> <span style="color: blue">string</span> ToString()
  </p>
  
  <p style="margin: 0px">
    {
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: blue">return</span> Name ?? <span style="color: #a31515">"Unnamed comamand"</span>;
  </p>
  
  <p style="margin: 0px">
    }
  </p></p>
</div>

Y todo pasa a funcionar a la perfección. 🙂

¿Moraleja final? Pues básicamente que si lanzas una excepción asegúrate de que es **el tipo correcto de excepción**. Porque de “demasiados elementos en la lista” y OutOfMemoryException nada de nada… 😉

Saludos!

 [1]: http://twitter.com/#!/eiximenis/status/202060274260389888 "http://twitter.com/#!/eiximenis/status/202060274260389888"
 [2]: http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/As3cv7bCMAEOizK_5F00_533114F2.png
 [3]: http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_56681E3C.png