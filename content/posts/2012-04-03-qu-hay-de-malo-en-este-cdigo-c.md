---
title: '¿Qué hay de malo en este código? (C#)'

author: eiximenis

date: 2012-04-03T10:11:20+00:00
geeks_url: /?p=1590
geeks_visits:
  - 3664
geeks_ms_views:
  - 1272
categories:
  - Uncategorized

---
Buenas 🙂 

Al estilo de muchos blogs que visito habitualmente y que proponen pequeños acertijos en base a un código que tiene un error (muchas veces no aparente, otras más evidente), os propongo hoy uno, que me he encontrado revisando código.

<!--more-->

Así que, amigos ¿qué hay de malo en este código?

<div style="font-family: courier new; background: white; color: black; font-size: 10pt">
  <p style="margin: 0px">
    <span style="color: blue">if</span> (!<span style="color: #2b91af">File</span>.Exists(fname))
  </p>
  
  <p style="margin: 0px">
    {
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: #2b91af">File</span>.Create(fname);
  </p>
  
  <p style="margin: 0px">
    }
  </p>
  
  <p style="margin: 0px">
    <span style="color: green">// Hacemos lo que sea...</span>
  </p></p>
</div>

Sencillo, ¿no? Si el fichero no existe lo creamos y luego hacemos lo que se supone que tengamos que hacer. Parece sencillo, pero hay algo que se nos escapa… ¿Qué puede ser?

Un saludo!