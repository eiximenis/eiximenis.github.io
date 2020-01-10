---
title: WebApi–Recibir en un controlador un IEnumerable desde URL

author: eiximenis

date: 2013-03-05T18:04:48+00:00
geeks_url: /?p=1634
geeks_visits:
  - 1768
geeks_ms_views:
  - 1601
categories:
  - Uncategorized

---
Muy buenas!

En un proyecto en el que estoy trabajando ha surgido la necesidad de pasarle via GET una lista de ids con los que hacer algo. La acción del controlador _FinishersController_ está declarada de la siguiente manera:

<div style="font-size: 10pt; font-family: consolas; background: #1e1e1e; color: #dcdcdc">
  <p style="margin: 0px">
    <span style="color: #569cd6">public</span> <span style="color: #b8d7a3">IEnumerable</span><span style="color: #b4b4b4"><</span><span style="color: #4ec9b0">TrackingAndCompetitorDTO</span><span style="color: #b4b4b4">></span> <span style="color: white">GetByRaces</span>(<span style="color: #b8d7a3">IEnumerable</span><span style="color: #b4b4b4"><</span><span style="color: #569cd6">int</span><span style="color: #b4b4b4">></span> <span style="color: white">id</span>)
  </p>
  
  <p style="margin: 0px">
    {
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: #569cd6">return</span> <span style="color: #569cd6">null</span>;
  </p>
  
  <p style="margin: 0px">
    }
  </p></p>
</div>

Ahora viene el momento de llamar al controlador:

  * /api/Finishers/10,20,30 –> Devuelve un 404 
  * /api/Finishers/10 –> Enlaza a al acción pero id es null 
  * /api/Finishers/?id=10,20,30 –> Enlaza a la acción pero id es null 
  * /api/Finishers/?id=10&id=20&id=30 –> Enlaza a la acción pero id es null 
  * /api/Finishers/?id[0]=10&id[1]=20&id[2]=30 –> Enlaza a la acción pero id es null 

Si tienes experiencia en ASP.NET MVC entenderás que las tres primeras fallen es _comprensible_ (también fallarían en MVC). Pero las dos últimas en MVC funcionarían correctamente… ¿entonces por qué fallan en WebApi?

La forma en como se realiza el enlace de parámetros de la request hacia los controladores **es totalmente diferente en WebApi que en MVC**. Insisto: WebApi es _otra cosa completamente distinta_ aunque luzca muy parecida a ASP.NET MVC y aunque vengan juntos. No des por supuesto nada de lo que sabes en ASP.NET MVC para WebApi. Algunas cosas funcionan igual, otras, completamente distinto.

En ASP.NET MVC el enlace desde la request hacia los controladores se realiza mediante los ValueProviders y los Model Binders. Los primeros son los encargados de inspeccionar la request (formdata, querystring, pathinfo, headers, …) y dejar los valores en un sitio común. Luego los model binders leen de “ese sitio común” y construyen los valores de los parámetros que el controlador recibe.

WebApi **añade** a estos dos conceptos, un tercero: los media type formatters. Bien, recuerda siempre la gran diferencia entre WebApi y MVC: En MVC se usa un buffer para guardar la petición (y la respuesta). Eso significa que los value providers pueden leer n veces el cuerpo de la petición sin que de error. En WebApi NO. WebApi es, por decirlo de algún modo, _stream-based_. Nuestros media type formatters reciben un stream y pueden leer de él una sola vez. Por lo tanto tan solo un media type formatter puede leer el cuerpo de la petición.

Pero ojo… he dicho que WebApi **añade** el concepto de media type formatters, porque en WebApi **también** se usan value providers y model binders. ¿Cuando? Pues para enlazar parámetros que no provienen del cuerpo de la petición (o sea, usualmente de la URL). Esa es la norma báscia:

  1. El parámetro no está en el cuerpo? Se enlaza vía un model binder 
  2. El parámetro está en el cuerpo de la petición? Se enlaza via un media type formatter 

Webapi hace ciertas asunciones sobre si un parámetro debe enlazarse via model binder o media type formatter. Básicamente: si es tipo simple se usará un model binder. Si es un tipo _complejo_ se usará un media type formatter. Por lo tanto, por defecto nuestro parámetro IEnumerable<int> al NO ser un tipo simple se intenta enlazar mediante un media type formatter. De ahi que no se encuentren datos porque los media type formatters miran tan solo el cuerpo y en nuestro caso está vacío. 

¿Y como podemos modificar este comportamiento? Pues bien:

  1. Si decoras un parámetro con [FromUri] indicas a WebApi que este parámetro vendrá en la URL 
  2. Si decoras un parámetro con [FromBody] indicas a WebApi que este parámetro vendrá en el cuerpo de la petición 
  3. Si decoras un parámetro con [ModelBinder] puedes especificar un Model Binder específico para tu parámetro. (De hecho [FromUri] deriva de [ModelBinder]). 
  4. Y recuerda: El cuerpo de la petición tan solo puede leerse una vez. Por lo tanto si tu controlador recibe dos parámetros complejos tan solo uno puede ser procesado mediante un media type formatter (y venir en el cuerpo). El otro debe ser procesado mediante un model binder y venir en alguna otra parte que NO sea el cuerpo de la petición&#160; (y estar decorado con [ModelBinder] o [FromUri]). 

Bueno… ahora ya ves la solución a nuestro problema no? Basta con decorar el parámetro con [FromUri]:

<div style="font-size: 10pt; font-family: consolas; background: #1e1e1e; color: #dcdcdc">
  <p style="margin: 0px">
    <span style="color: #569cd6">public</span> <span style="color: #b8d7a3">IEnumerable</span><span style="color: #b4b4b4"><</span><span style="color: #4ec9b0">TrackingAndCompetitorDTO</span><span style="color: #b4b4b4">></span> <span style="color: white">GetByRaces</span>([<span style="color: #4ec9b0">FromUri</span>] <span style="color: #b8d7a3">IEnumerable</span><span style="color: #b4b4b4"><</span><span style="color: #569cd6">int</span><span style="color: #b4b4b4">></span> <span style="color: white">id</span>)
  </p>
  
  <p style="margin: 0px">
    {
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: #569cd6">return</span> <span style="color: #569cd6">null</span>;
  </p>
  
  <p style="margin: 0px">
    }
  </p></p>
</div>

Con esto estamos forzando a WebApi a que enlace este parámetro usando los value providers y model binders. Y ahora la URL

  * /api/Finishers/?id=10&id=20&id=30 –> Funciona correctamente. 

Curiosamente la URL

  * /api/Finishers?id[0]=10&id[1]=20&id[2]=30 NO funciona, pero eso ya se debe a diferencias de como está implementado el Model binder de WebApi y el de MVC cuando tratan con colecciones… 

Saludos!