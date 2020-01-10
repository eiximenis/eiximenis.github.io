---
title: Objetos anónimos como diccionarios
description: Usar objetos anónimos como diccionarios
author: eiximenis

date: 2009-05-26T17:27:48+00:00
geeks_url: /?p=1452
geeks_visits:
  - 1511
geeks_ms_views:
  - 950
categories:
  - Uncategorized

---
¡Hola!

Cada vez más existen frameworks y librerías que permiten usar objetos anónimos como si de diccionarios se tratase. Esto es muy cómodo porque permite realizar llamadas tal y como:

<pre class="code">Foo(<span style="color: blue">new </span>{ x = 10, y = 20, Data = <span style="color: #a31515">"Data" </span>});</pre>

[][1]

Por ejemplo, en ASP.NET MVC no se paran de hacer llamadas parecidas a esta…

Internamente el método Foo utilizará reflection para iterar sobre las propiedades del objeto anónimo que recibe y obtener los datos.

Si tenéis varios métodos que trabajan con IDictionary<string, T> y queréis que se puedan llamar fácilmente usando esta técnica podéis crear un método extensor de IDictionary<string, T> que rellene un diccionario a partir de las propiedades de un objeto… De acuerdo, vuestros métodos no serán invocables directamente pasándoles un objeto anónimo pero al menos los diccionarios se crearán mucho más fácilmente.

Si estáis vagos aquí tenéis un código que uso yo para esto:

<pre class="code"><span style="color: blue">public static class </span><span style="color: #2b91af">DictionaryStringStringExtensions
</span>{
    <span style="color: blue">public static </span><span style="color: #2b91af">IDictionary</span>&lt;<span style="color: blue">string</span>, T&gt;  FillFromObject&lt;T&gt;(
        <span style="color: blue">this </span><span style="color: #2b91af">IDictionary</span>&lt;<span style="color: blue">string</span>, T&gt; dictionary,
        <span style="color: blue">object </span>values)
    {
        <span style="color: blue">if </span>(values == <span style="color: blue">null</span>) <span style="color: blue">return </span>dictionary;
        <span style="color: #2b91af">Type </span>type = values.GetType();
        <span style="color: blue">foreach </span>(<span style="color: blue">var </span>property <span style="color: blue">in </span>type.GetProperties())
        {
            <span style="color: blue">if </span>(!dictionary.ContainsKey(property.Name))
            {
                <span style="color: blue">object </span>propValue = property.GetValue(values, <span style="color: blue">null</span>);
                T finalValue = <span style="color: blue">default</span>(T);
                <span style="color: blue">if </span>(property.PropertyType.Equals(<span style="color: blue">typeof</span>(T)))
                {
                    finalValue = (T)property.GetValue(values, <span style="color: blue">null</span>);
                }
                <span style="color: blue">else
                </span>{
                    finalValue = (T)<span style="color: #2b91af">Convert</span>.ChangeType (propValue,
                        <span style="color: blue">typeof</span>(T),  <span style="color: #2b91af">CultureInfo</span>.CurrentCulture);
                }
                dictionary.Add(property.Name, finalValue);
            }
        }
        <span style="color: blue">return </span>dictionary;
    }
}</pre>


El método realiza conversiones básicas entre tipos de propiedades… Así podeis inicializar fácilmente vuestros diccionarios:

<pre class="code"><span style="color: blue">var </span>d = <span style="color: blue">new </span><span style="color: #2b91af">Dictionary</span>&lt;<span style="color: blue">string</span>, <span style="color: blue">int</span>&gt;();
d.FillFromObject(<span style="color: blue">new </span>{ col = <span style="color: #a31515">"10"</span>, row = 20.2 });</pre>

Saludos!

