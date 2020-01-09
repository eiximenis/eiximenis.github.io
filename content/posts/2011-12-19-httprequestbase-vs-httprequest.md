---
title: HttpRequestBase vs HttpRequest
author: eiximenis

date: 2011-12-19T15:03:50+00:00
geeks_url: /?p=1584
geeks_visits:
  - 2664
geeks_ms_views:
  - 1810
categories:
  - Uncategorized

---
Muy buenas!

Coged a alguien que no conozca mucho ASP.NET y preguntadle que relación tienen las siguientes clases entre ellas:

  1. HttpRequest 
  2. HttpRequestBase 
  3. HttpRequestWrapper 

La respuesta más probable será que HttpRequestBase es la clase base, de la cual deriva HttpRequest y que HttpRequestWrapper es… bueno, por el nombre no queda muy claro: es un wrapper de algo pero de qué?

Pues no. Nada más lejos de la realidad. Aunque el nombre _sugiera_ lo contrario HttpRequestBase no es la clase base de HttpRequest, de hecho ambas clases **no tienen relación alguna** entre ellas, pero en Redmond no tuvieron un buen día al escoger el nombre… Aunque al menos debe reconocerse que HttpRequestBase sí es clase base de alguien… ¡concretamente de HttpRequestWrapper!

Dado que no hay mucha gente que conozca esas dos últimas clases, hagamos pues una pequeña presentación.

HttpRequestBase es una clase cuyo interfaz público (es decir sus métodos y propiedades) son **idénticos** a los de HttpRequest. ¿Que HttpRequest tiene una propiedad llamada Cookies? HttpRequestBase la tiene también… y del mismo tipo. Y así con todas las propiedades y todos los métodos.

¿Y por qué han decidido crear semejante… _cosa_? 

Pues para corregir una carencia que tenía el Framework: permitir _abstraernos_ de HttpRequest de forma fácil: HttpRequestBase tiene la misma interfaz pública que HttpRequest pero _no_ está vinculada a ASP.NET. No requiere un pipeline web ejecutándose, ni nada de nada: es un simple contenedor de datos.

Por supuesto ASP.NET está construido alrededor de HttpRequest, lo cual sigue dificultando mucho los tests unitarios… ¿así que entonces? Bueno ASP.NET arrastra toda una historia y no es fácil (ni sensato) romper con todo, así que lo máximo que podemos exigir es que lo más nuevo sí se haga bien. Y lo más nuevo es ASP.NET MVC. En efecto, ASP.NET MVC está construido alrededor de HttpRequestBase y no de HttpRequest. 

P.ej. se puede acceder a la request desde un controlador:

<div style="font-family: courier new; background: white; color: black; font-size: 10pt">
  <p style="margin: 0px">
    <span style="color: blue">var</span> queryString = ControllerContext.HttpContext.Request.QueryString;
  </p></p>
</div>

Pero si observáis con detalle veréis que la propiedad Request **no** es de tipo HttpRequest si no de tipo HttpRequestBase:

[<img style="border-right-width: 0px; display: inline; border-top-width: 0px; border-bottom-width: 0px; border-left-width: 0px" title="image" border="0" alt="image" src="http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_2DFA14E8.png" width="244" height="37" />][1] 

¿Y como podemos usar esto en nuestros tests unitarios?

Imaginad un controlador que tenga el siguiente código:

<div style="font-family: courier new; background: white; color: black; font-size: 10pt">
  <p style="margin: 0px">
    <span style="color: blue">public</span> <span style="color: #2b91af">ActionResult</span> About()
  </p>
  
  <p style="margin: 0px">
    {
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: blue">var</span> queryString = ControllerContext.HttpContext.Request.QueryString;
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: blue">var</span> modelo = <span style="color: blue">new</span> <span style="color: #2b91af">FooModel</span>();
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; modelo.Nombre = ControllerContext.HttpContext.Request.QueryString[<span style="color: #a31515">"p1"</span>];
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: blue">return</span> View(modelo);
  </p>
  
  <p style="margin: 0px">
    }
  </p></p>
</div>

El controlador recoge el valor del parámetro “p1” de la querystring y establece la propiedad _Nombre_ del viewmodel con este valor (_Nota: ¡Este código es solo para demostrar lo que se comenta en este post, en ASP.NET MVC hay maneras mejores de hacer esto!)_

Ahora vamos a ver como sería un test unitario que probase este método. Para ello… y aquí es donde entra en juego HttpRequestBase: nos permite crear un mock o un fake de ella:

<div style="font-family: courier new; background: white; color: black; font-size: 10pt">
  <p style="margin: 0px">
    <span style="color: blue">public</span> <span style="color: blue">class</span> <span style="color: #2b91af">HttpRequestFake</span> : <span style="color: #2b91af">HttpRequestBase</span>
  </p>
  
  <p style="margin: 0px">
    {
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: blue">public</span> <span style="color: blue">override</span> <span style="color: #2b91af">NameValueCollection</span> QueryString
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; {
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: blue">get</span>&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; {
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: blue">var</span> values = <span style="color: blue">new</span> <span style="color: #2b91af">NameValueCollection</span>();
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; values.Add(<span style="color: #a31515">"p1"</span>, <span style="color: #a31515">"v1"</span>);
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: blue">return</span> values;
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; }
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; }
  </p>
  
  <p style="margin: 0px">
    }
  </p></p>
</div>

Ya tenemos nuestro fake de HttpRequestBase para que devuelva una querystring fijada por nosotros (donde el parámetro p1 valga v1).

Ahora nos toca hacer un par de fakes más: de HttpContextBase (para que devuelva nuestro objeto Request) y de ControllerContext para que nos devuelva el fake de HttpContextBase:

<div style="font-family: courier new; background: white; color: black; font-size: 10pt">
  <p style="margin: 0px">
    <span style="color: blue">public</span> <span style="color: blue">class</span> <span style="color: #2b91af">HttpContextFake</span> : <span style="color: #2b91af">HttpContextBase</span>
  </p>
  
  <p style="margin: 0px">
    {
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: blue">public</span> <span style="color: blue">override</span> <span style="color: #2b91af">HttpRequestBase</span> Request
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; {
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: blue">get</span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; {
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: blue">return</span> <span style="color: blue">new</span> <span style="color: #2b91af">HttpRequestFake</span>();
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; }
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; }
  </p>
  
  <p style="margin: 0px">
    }
  </p></p>
</div>

<div style="font-family: courier new; background: white; color: black; font-size: 10pt">
  <p style="margin: 0px">
    <span style="color: blue">public</span> <span style="color: blue">class</span> <span style="color: #2b91af">ControllerContextFake</span> : <span style="color: #2b91af">ControllerContext</span>
  </p>
  
  <p style="margin: 0px">
    {
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: blue">public</span> ControllerContextFake()
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; {
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;<br /> &#160;&#160;&#160;&#160; HttpContext = <span style="color: blue">new</span> <span style="color: #2b91af">HttpContextFake</span>();
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; }
  </p>
  
  <p style="margin: 0px">
    &#160;
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: blue">public</span> <span style="color: blue">override</span> <span style="color: #2b91af">HttpContextBase</span> HttpContext { <span style="color: blue">get</span>; <span style="color: blue">set</span>; }
  </p>
  
  <p style="margin: 0px">
    }
  </p></p>
</div>

Y finalmente ya podemos tener nuestro test unitario:

<div style="font-family: courier new; background: white; color: black; font-size: 10pt">
  <p style="margin: 0px">
    [<span style="color: #2b91af">TestMethod</span>]
  </p>
  
  <p style="margin: 0px">
    <span style="color: blue">public</span> <span style="color: blue">void</span> TestMethod1()
  </p>
  
  <p style="margin: 0px">
    {
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: blue">var</span> controller = <span style="color: blue">new</span> <span style="color: #2b91af">HomeController</span>();
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; controller.ControllerContext = <span style="color: blue">new</span> <span style="color: #2b91af">ControllerContextFake</span>();
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: blue">var</span> result = controller.About() <span style="color: blue">as</span> <span style="color: #2b91af">ViewResult</span>;
  </p>
  
  <p style="margin: 0px">
    &#160;
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: #2b91af">Assert</span>.AreEqual(<span style="color: #a31515">"v1"</span>, (result.ViewData.Model <span style="color: blue">as</span> <span style="color: #2b91af">FooModel</span>).Nombre);
  </p>
  
  <p style="margin: 0px">
    }
  </p></p>
</div>

Y listos! Con esto comprobamos que el controlador hace lo que se supone que debe hacer (asignar el valor del parámetro p1 de la querystring a la propiedad Nombre del viewmodel).

Gracias al hecho de que HttpRequestBase y HttpContextBase NO están ligadas a ningún pipeline de ASP.NET y de que ASP.NET MVC ha sido construído en torno a ellas y no a las _reales_ HttpRequest y HttpContext nos es mucho más fácil la realización de tests unitarios.

Espero que os haya sido de interés! 😉   
Un saludo!

Ah sí… y HttpRequestWrapper? Pues HttpRequestWrapper no es nada más que una clase que deriva de HttpRequestBase y que sirve para… _convertir_ un objeto HttpRequest en un HttpRequestBase… ya, probablemente un nombre mejor para ella hubiese sido _HttpRequestAdapter_. 😉

 [1]: http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_4355399F.png