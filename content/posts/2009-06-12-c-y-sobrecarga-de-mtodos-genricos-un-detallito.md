---
title: 'C# y sobrecarga de métodos genéricos… un detallito'

author: eiximenis

date: 2009-06-12T09:14:40+00:00
geeks_url: /?p=1456
geeks_visits:
  - 5457
geeks_ms_views:
  - 1371
categories:
  - Uncategorized

---
A veces hay aspectos de C# que no pensamos hasta que nos encontramos con ellos… A mi me pasó con un código parecido a este:

<!--more-->

<pre class="code"><span style="color: blue">class </span><span style="color: #2b91af">Program
</span>{
    <span style="color: blue">static void </span>Main(<span style="color: blue">string</span>[] args)
    {
        <span style="color: #2b91af">Baz </span>baz = <span style="color: blue">new </span><span style="color: #2b91af">BazDerived</span>();
        <span style="color: blue">new </span><span style="color: #2b91af">Foo</span>().Bar(baz);
        <span style="color: #2b91af">Console</span>.ReadLine();
    }
}
<span style="color: blue">class </span><span style="color: #2b91af">Foo
</span>{
    <span style="color: blue">public void </span>Bar&lt;T&gt;(T t)
    {
        <span style="color: #2b91af">Console</span>.WriteLine(<span style="color: #a31515">"Bar&lt;T&gt; typeof(T) = " </span>+ <span style="color: blue">typeof</span>(T).Name);
    }
    <span style="color: blue">public void </span>Bar(<span style="color: blue">object </span>o)
    {
        <span style="color: #2b91af">Console</span>.WriteLine(<span style="color: #a31515">"Bar o.GetType() = " </span>+ o.GetType().Name);
    }
}
<span style="color: blue">class </span><span style="color: #2b91af">Baz </span>{ }
<span style="color: blue">class </span><span style="color: #2b91af">BazDerived </span>: <span style="color: #2b91af">Baz </span>{ }</pre>

[][1]

La pregunta seria: ¿cual es la salida por pantalla de este programa?

Pues… esta es la respuesta (para los que querais pensar un poco la he puesto en negro sobre negro… seleccionad el texto para verlo):</p> 

<div style="background-color: black">
  <font color="#000000" size="2" face="Courier New">Bar<T> typeof(T) = Baz</font>
</div></p> 

La verdad es que tiene toda su lógica… lo que personalmente no me gusta nada es que este código compile _sin generar ningún warning_. ¿Que opinais vosotros?

Saludos!!!

 [1]: http://11011.net/software/vspaste