---
title: 'Webforms: Forzar postbacks'
author: eiximenis

date: 2012-04-16T10:08:00+00:00
geeks_url: /?p=1592
geeks_visits:
  - 2725
geeks_ms_views:
  - 2186
categories:
  - Uncategorized

---
Jejejee... Sí, aunque no lo parezca _a veces_ hago temillas con Webforms, y es que uno tiene que conocer al enemigo! 😛

Lo que voy a comentar hoy, es como forzar un postback desde un control propio. Una búsqueda en google da varios resultados, pongo un par de ejemplo:

  1. [http://tratadooscuro.blogspot.com.es/2009/02/dopostback-ese-gran-desconocido.html][1] 
  2. [http://programacion.porexpertos.es/provocar-un-postback-desde-javascript-con-aspnet/][2] 

Ambos ejemplos dicen lo mismo pero lo cierto es que, en mi opinión, hay una manera ligeramente mejor que hacerlo, pero parece que se desconoce bastante porque buscando en google aparecen menos resultados.

Bueno, si miráis los dos enlaces que he puesto arriba, forzar un postback desde un control propio es tan simple como llamar a __doPostback. Este método lo añade automáticamente Webforms cuando lo necesita.

Si seguimos las instrucciones de cualquiera de los dos enlaces anteriores, si queremos generar un control que sea p.ej. un enlace que al pulsarlo genere un postback vemos que debemos usar un código como este:

<div style="font-family: courier new; background: white; color: black; font-size: 10pt">
  <p style="margin: 0px">
    [<span style="color: #2b91af">DefaultProperty</span>(<span style="color: #a31515">&#8220;Text&#8221;</span>)]
  </p>
  
  <p style="margin: 0px">
    [<span style="color: #2b91af">ToolboxData</span>(<span style="color: #a31515">&#8220;<{0}:MyControl runat=server></{0}:MyControl>&#8221;</span>)]
  </p>
  
  <p style="margin: 0px">
    <span style="color: blue">public</span> <span style="color: blue">class</span> <span style="color: #2b91af">MyControl</span> : <span style="color: #2b91af">WebControl</span>
  </p>
  
  <p style="margin: 0px">
    {
  </p>
  
  <p style="margin: 0px">
    &nbsp;&nbsp;&nbsp; [<span style="color: #2b91af">Bindable</span>(<span style="color: blue">true</span>)]
  </p>
  
  <p style="margin: 0px">
    &nbsp;&nbsp;&nbsp; [<span style="color: #2b91af">Category</span>(<span style="color: #a31515">&#8220;Appearance&#8221;</span>)]
  </p>
  
  <p style="margin: 0px">
    &nbsp;&nbsp;&nbsp; [<span style="color: #2b91af">DefaultValue</span>(<span style="color: #a31515">&#8220;&#8221;</span>)]
  </p>
  
  <p style="margin: 0px">
    &nbsp;&nbsp;&nbsp; [<span style="color: #2b91af">Localizable</span>(<span style="color: blue">true</span>)]
  </p>
  
  <p style="margin: 0px">
    &nbsp;&nbsp;&nbsp; <span style="color: blue">public</span> <span style="color: blue">string</span> Text
  </p>
  
  <p style="margin: 0px">
    &nbsp;&nbsp;&nbsp; {
  </p>
  
  <p style="margin: 0px">
    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span style="color: blue">get</span>
  </p>
  
  <p style="margin: 0px">
    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; {
  </p>
  
  <p style="margin: 0px">
    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span style="color: #2b91af">String</span> s = (<span style="color: #2b91af">String</span>)ViewState[<span style="color: #a31515">&#8220;Text&#8221;</span>];
  </p>
  
  <p style="margin: 0px">
    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span style="color: blue">return</span> s ?? <span style="color: blue">string</span>.Empty;
  </p>
  
  <p style="margin: 0px">
    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }
  </p>
  
  <p style="margin: 0px">
    &nbsp;
  </p>
  
  <p style="margin: 0px">
    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span style="color: blue">set</span>
  </p>
  
  <p style="margin: 0px">
    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; {
  </p>
  
  <p style="margin: 0px">
    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; ViewState[<span style="color: #a31515">&#8220;Text&#8221;</span>] = <span style="color: blue">value</span>;
  </p>
  
  <p style="margin: 0px">
    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }
  </p>
  
  <p style="margin: 0px">
    &nbsp;&nbsp;&nbsp; }
  </p>
  
  <p style="margin: 0px">
    &nbsp;
  </p>
  
  <p style="margin: 0px">
    &nbsp;&nbsp;&nbsp; <span style="color: blue">protected</span> <span style="color: blue">override</span> <span style="color: blue">void</span> RenderContents(<span style="color: #2b91af">HtmlTextWriter</span> output)
  </p>
  
  <p style="margin: 0px">
    &nbsp;&nbsp;&nbsp; {
  </p>
  
  <p style="margin: 0px">
    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; output.AddAttribute(<span style="color: #2b91af">HtmlTextWriterAttribute</span>.Href,
  </p>
  
  <p style="margin: 0px">
    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span style="color: #a31515">&#8220;javascript:__doPostBack(&#8221;,&#8221;);&#8221;</span>);
  </p>
  
  <p style="margin: 0px">
    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; output.RenderBeginTag(<span style="color: #2b91af">HtmlTextWriterTag</span>.A);
  </p>
  
  <p style="margin: 0px">
    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; output.Write(Text);
  </p>
  
  <p style="margin: 0px">
    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; output.RenderEndTag();
  </p>
  
  <p style="margin: 0px">
    &nbsp;&nbsp;&nbsp; }
  </p>
  
  <p style="margin: 0px">
    }
  </p>
</div>

Si creamos una Webform vacío y añadimos el control y probamos la página, efectivamente se renderiza el tag <a> que incluye la llamada a __doPostBack. Pero fijaos que este es el código HTML generado por el Webform:

<div style="font-family: courier new; background: white; color: black; font-size: 10pt">
  <p style="margin: 0px">
    <span style="color: blue"><!</span><span style="color: #a31515">DOCTYPE</span> <span style="color: red">html</span> <span style="color: red">PUBLIC</span> <span style="color: blue">&#8220;-//W3C//DTD XHTML 1.0 Transitional//EN&#8221;</span> <span style="color: blue">&#8220;http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd&#8221;></span>
  </p>
  
  <p style="margin: 0px">
    &nbsp;
  </p>
  
  <p style="margin: 0px">
    <span style="color: blue"><</span><span style="color: #a31515">html</span> <span style="color: red">xmlns</span><span style="color: blue">=&#8221;http://www.w3.org/1999/xhtml&#8221;</span> <span style="color: blue">></span>
  </p>
  
  <p style="margin: 0px">
    <span style="color: blue"><</span><span style="color: #a31515">head</span><span style="color: blue">><</span><span style="color: #a31515">title</span><span style="color: blue">></span>
  </p>
  
  <p style="margin: 0px">
    &nbsp;
  </p>
  
  <p style="margin: 0px">
    <span style="color: blue"></</span><span style="color: #a31515">title</span><span style="color: blue">></</span><span style="color: #a31515">head</span><span style="color: blue">></span>
  </p>
  
  <p style="margin: 0px">
    <span style="color: blue"><</span><span style="color: #a31515">body</span><span style="color: blue">></span>
  </p>
  
  <p style="margin: 0px">
    &nbsp;&nbsp;&nbsp; <span style="color: blue"><</span><span style="color: #a31515">form</span> <span style="color: red">name</span><span style="color: blue">=&#8221;form1&#8243;</span> <span style="color: red">method</span><span style="color: blue">=&#8221;post&#8221;</span> <span style="color: red">action</span><span style="color: blue">=&#8221;default.aspx&#8221;</span> <span style="color: red">id</span><span style="color: blue">=&#8221;form1&#8243;></span>
  </p>
  
  <p style="margin: 0px">
    <span style="color: blue"><</span><span style="color: #a31515">div</span><span style="color: blue">></span>
  </p>
  
  <p style="margin: 0px">
    <span style="color: blue"><</span><span style="color: #a31515">input</span> <span style="color: red">type</span><span style="color: blue">=&#8221;hidden&#8221;</span> <span style="color: red">name</span><span style="color: blue">=&#8221;__VIEWSTATE&#8221;</span> <span style="color: red">id</span><span style="color: blue">=&#8221;__VIEWSTATE&#8221;</span> <span style="color: red">value</span><span style="color: blue">=&#8221;/wEPDwUKLTk1OTYxMjkxMmRkGui/AV2Pv2fTTQJWmL8w2grZiZY=&#8221;</span> <span style="color: blue">/></span>
  </p>
  
  <p style="margin: 0px">
    <span style="color: blue"></</span><span style="color: #a31515">div</span><span style="color: blue">></span>
  </p>
  
  <p style="margin: 0px">
    &nbsp;
  </p>
  
  <p style="margin: 0px">
    &nbsp;&nbsp;&nbsp; <span style="color: blue"><</span><span style="color: #a31515">div</span><span style="color: blue">></span>
  </p>
  
  <p style="margin: 0px">
    &nbsp;
  </p>
  
  <p style="margin: 0px">
    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span style="color: blue"><</span><span style="color: #a31515">span</span> <span style="color: red">id</span><span style="color: blue">=&#8221;MyControl1&#8243;><</span><span style="color: #a31515">a</span> <span style="color: red">href</span><span style="color: blue">=&#8221;javascript:__doPostBack(&#8221;,&#8221;);&#8221;></span>Demo<span style="color: blue"></</span><span style="color: #a31515">a</span><span style="color: blue">></</span><span style="color: #a31515">span</span><span style="color: blue">></span>
  </p>
  
  <p style="margin: 0px">
    &nbsp;
  </p>
  
  <p style="margin: 0px">
    &nbsp;&nbsp;&nbsp; <span style="color: blue"></</span><span style="color: #a31515">div</span><span style="color: blue">></span>
  </p>
  
  <p style="margin: 0px">
    &nbsp;&nbsp;&nbsp; <span style="color: blue"></</span><span style="color: #a31515">form</span><span style="color: blue">></span>
  </p>
  
  <p style="margin: 0px">
    <span style="color: blue"></</span><span style="color: #a31515">body</span><span style="color: blue">></span>
  </p>
  
  <p style="margin: 0px">
    <span style="color: blue"></</span><span style="color: #a31515">html</span><span style="color: blue">></span>
  </p>
</div>

¿Alguien ve el método __doPostBack? No, ¿verdad? Eso es porque **Webforms no lo ha generado**, y no lo ha generado porque no sabe que alguien va a usarlo. La verdad es que Webforms solo genera este método cuando sabe que algún control lo requiere. En páginas medio complejas la probabilidad de que _algún_ control requiera postback es tan grande que por eso mucha gente cree que _siempre_ se genera. Pero no es así.

Poner código &ldquo;a saco&rdquo; para llamar a __doPostBack funciona en la mayoría de casos pero no es una buena práctica porque estamos usando **una característica interna de Webforms**. Si en la siguiente release de Webforms Microsoft decide renombrar esta función todo nuestro código se viene abajo.

Vale... entonces si no podemos llamara a __doPostBack a saco, que debemos hacer? Pues usar el método GetPostBackClientHyperlink de la clase ClientScriptManager. Para obtener una instancia de ClientScriptManager se puede usar la propiedad ClientScript del objeto Page:

<div style="font-family: courier new; background: white; color: black; font-size: 10pt">
  <p style="margin: 0px">
    <span style="color: blue">protected</span> <span style="color: blue">override</span> <span style="color: blue">void</span> RenderContents(<span style="color: #2b91af">HtmlTextWriter</span> output)
  </p>
  
  <p style="margin: 0px">
    {
  </p>
  
  <p style="margin: 0px">
    &nbsp;&nbsp;&nbsp; output.AddAttribute(<span style="color: #2b91af">HtmlTextWriterAttribute</span>.Href,
  </p>
  
  <p style="margin: 0px">
    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span style="color: blue">this</span>.Page.ClientScript.GetPostBackClientHyperlink(<span style="color: blue">this</span>, <span style="color: #a31515">&#8220;&#8221;</span>));
  </p>
  
  <p style="margin: 0px">
    &nbsp;&nbsp;&nbsp; output.RenderBeginTag(<span style="color: #2b91af">HtmlTextWriterTag</span>.A);
  </p>
  
  <p style="margin: 0px">
    &nbsp;&nbsp;&nbsp; output.Write(Text);
  </p>
  
  <p style="margin: 0px">
    &nbsp;&nbsp;&nbsp; output.RenderEndTag();
  </p>
  
  <p style="margin: 0px">
    }
  </p>
</div>

&iexcl;Listos! Ahora si ejecutamos de nuevo el Webform, vemos que el código generado es:

<div style="font-family: courier new; background: white; color: black; font-size: 10pt">
  <p style="margin: 0px">
    <span style="color: blue"><!</span><span style="color: #a31515">DOCTYPE</span> <span style="color: red">html</span> <span style="color: red">PUBLIC</span> <span style="color: blue">&#8220;-//W3C//DTD XHTML 1.0 Transitional//EN&#8221;</span> <span style="color: blue">&#8220;http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd&#8221;></span>
  </p>
  
  <p style="margin: 0px">
    &nbsp;
  </p>
  
  <p style="margin: 0px">
    <span style="color: blue"><</span><span style="color: #a31515">html</span> <span style="color: red">xmlns</span><span style="color: blue">=&#8221;http://www.w3.org/1999/xhtml&#8221;</span> <span style="color: blue">></span>
  </p>
  
  <p style="margin: 0px">
    <span style="color: blue"><</span><span style="color: #a31515">head</span><span style="color: blue">><</span><span style="color: #a31515">title</span><span style="color: blue">></span>
  </p>
  
  <p style="margin: 0px">
    &nbsp;
  </p>
  
  <p style="margin: 0px">
    <span style="color: blue"></</span><span style="color: #a31515">title</span><span style="color: blue">></</span><span style="color: #a31515">head</span><span style="color: blue">></span>
  </p>
  
  <p style="margin: 0px">
    <span style="color: blue"><</span><span style="color: #a31515">body</span><span style="color: blue">></span>
  </p>
  
  <p style="margin: 0px">
    &nbsp;&nbsp;&nbsp; <span style="color: blue"><</span><span style="color: #a31515">form</span> <span style="color: red">name</span><span style="color: blue">=&#8221;form1&#8243;</span> <span style="color: red">method</span><span style="color: blue">=&#8221;post&#8221;</span> <span style="color: red">action</span><span style="color: blue">=&#8221;default.aspx&#8221;</span> <span style="color: red">id</span><span style="color: blue">=&#8221;form1&#8243;></span>
  </p>
  
  <p style="margin: 0px">
    <span style="color: blue"><</span><span style="color: #a31515">div</span><span style="color: blue">></span>
  </p>
  
  <p style="margin: 0px">
    <span style="color: blue"><</span><span style="color: #a31515">input</span> <span style="color: red">type</span><span style="color: blue">=&#8221;hidden&#8221;</span> <span style="color: red">name</span><span style="color: blue">=&#8221;__VIEWSTATE&#8221;</span> <span style="color: red">id</span><span style="color: blue">=&#8221;__VIEWSTATE&#8221;</span> <span style="color: red">value</span><span style="color: blue">=&#8221;/wEPDwUKLTk1OTYxMjkxMmRkGui/AV2Pv2fTTQJWmL8w2grZiZY=&#8221;</span> <span style="color: blue">/></span>
  </p>
  
  <p style="margin: 0px">
    <span style="color: blue"></</span><span style="color: #a31515">div</span><span style="color: blue">></span>
  </p>
  
  <p style="margin: 0px">
    &nbsp;
  </p>
  
  <p style="margin: 0px">
    <span style="color: blue"><</span><span style="color: #a31515">div</span><span style="color: blue">></span>
  </p>
  
  <p style="margin: 0px">
    &nbsp;
  </p>
  
  <p style="margin: 0px">
    &nbsp;&nbsp;&nbsp; <span style="color: blue"><</span><span style="color: #a31515">input</span> <span style="color: red">type</span><span style="color: blue">=&#8221;hidden&#8221;</span> <span style="color: red">name</span><span style="color: blue">=&#8221;__EVENTTARGET&#8221;</span> <span style="color: red">id</span><span style="color: blue">=&#8221;__EVENTTARGET&#8221;</span> <span style="color: red">value</span><span style="color: blue">=&#8221;&#8221;</span> <span style="color: blue">/></span>
  </p>
  
  <p style="margin: 0px">
    &nbsp;&nbsp;&nbsp; <span style="color: blue"><</span><span style="color: #a31515">input</span> <span style="color: red">type</span><span style="color: blue">=&#8221;hidden&#8221;</span> <span style="color: red">name</span><span style="color: blue">=&#8221;__EVENTARGUMENT&#8221;</span> <span style="color: red">id</span><span style="color: blue">=&#8221;__EVENTARGUMENT&#8221;</span> <span style="color: red">value</span><span style="color: blue">=&#8221;&#8221;</span> <span style="color: blue">/></span>
  </p>
  
  <p style="margin: 0px">
    <span style="color: blue"></</span><span style="color: #a31515">div</span><span style="color: blue">></span>
  </p>
  
  <p style="margin: 0px">
    &nbsp;&nbsp;&nbsp; <span style="color: blue"><</span><span style="color: #a31515">div</span><span style="color: blue">></span>
  </p>
  
  <p style="margin: 0px">
    &nbsp;
  </p>
  
  <p style="margin: 0px">
    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span style="color: blue"><</span><span style="color: #a31515">span</span> <span style="color: red">id</span><span style="color: blue">=&#8221;MyControl1&#8243;><</span><span style="color: #a31515">a</span> <span style="color: red">href</span><span style="color: blue">=&#8221;javascript:__doPostBack(&#8216;MyControl1&#8217;,&#8221;)&#8221;></span>Demo<span style="color: blue"></</span><span style="color: #a31515">a</span><span style="color: blue">></</span><span style="color: #a31515">span</span><span style="color: blue">></span>
  </p>
  
  <p style="margin: 0px">
    &nbsp;
  </p>
  
  <p style="margin: 0px">
    &nbsp;&nbsp;&nbsp; <span style="color: blue"></</span><span style="color: #a31515">div</span><span style="color: blue">></span>
  </p>
  
  <p style="margin: 0px">
    &nbsp;
  </p>
  
  <p style="margin: 0px">
    <span style="color: blue"><</span><span style="color: #a31515">script</span> <span style="color: red">type</span><span style="color: blue">=&#8221;text/javascript&#8221;></span>
  </p>
  
  <p style="margin: 0px">
    <span style="color: green">//<![CDATA[</span>
  </p>
  
  <p style="margin: 0px">
    <span style="color: blue">var</span> theForm = document.forms[<span style="color: #a31515">&#8216;form1&#8217;</span>];
  </p>
  
  <p style="margin: 0px">
    <span style="color: blue">if</span> (!theForm) {
  </p>
  
  <p style="margin: 0px">
    &nbsp;&nbsp;&nbsp; theForm = document.form1;
  </p>
  
  <p style="margin: 0px">
    }
  </p>
  
  <p style="margin: 0px">
    <span style="color: blue">function</span> __doPostBack(eventTarget, eventArgument) {
  </p>
  
  <p style="margin: 0px">
    &nbsp;&nbsp;&nbsp; <span style="color: blue">if</span> (!theForm.onsubmit || (theForm.onsubmit() != <span style="color: blue">false</span>)) {
  </p>
  
  <p style="margin: 0px">
    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; theForm.__EVENTTARGET.value = eventTarget;
  </p>
  
  <p style="margin: 0px">
    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; theForm.__EVENTARGUMENT.value = eventArgument;
  </p>
  
  <p style="margin: 0px">
    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; theForm.submit();
  </p>
  
  <p style="margin: 0px">
    &nbsp;&nbsp;&nbsp; }
  </p>
  
  <p style="margin: 0px">
    }
  </p>
  
  <p style="margin: 0px">
    <span style="color: green">//]]></span>
  </p>
  
  <p style="margin: 0px">
    <span style="color: blue"></</span><span style="color: #a31515">script</span><span style="color: blue">></span>
  </p>
  
  <p style="margin: 0px">
    &nbsp;
  </p>
  
  <p style="margin: 0px">
    <span style="color: blue"></</span><span style="color: #a31515">form</span><span style="color: blue">></span>
  </p>
  
  <p style="margin: 0px">
    <span style="color: blue"></</span><span style="color: #a31515">body</span><span style="color: blue">></span>
  </p>
  
  <p style="margin: 0px">
    <span style="color: blue"></</span><span style="color: #a31515">html</span><span style="color: blue">></span>
  </p>
</div>

Por un lado **ahora Webforms nos ha generado el __doPostBack** y por otro como podemos ver el el atributo href de nuestro enlace tenemos la llamada a doPostBack correcta (por norma general el primer parámetro es el ID del control que realiza la llamada).

**Y ya que sabemos generar PostBacks... ¿como recibirlos?**

Hemos visto como generar un PostBack desde nuestro control, ahora... como podemos decirle a Webforms que _nuestro control_ quiere enterarse de los postbacks que _él_ haya efecutado?

Pues para esto basta con implementar la interfaz IPostBackEventHandler que tiene un solo método:

<div style="font-family: courier new; background: white; color: black; font-size: 10pt">
  <p style="margin: 0px">
    <span style="color: blue">public</span> <span style="color: blue">interface</span> <span style="color: #2b91af">IPostBackEventHandler</span>
  </p>
  
  <p style="margin: 0px">
    {
  </p>
  
  <p style="margin: 0px">
    &nbsp;&nbsp;&nbsp; <span style="color: blue">void</span> RaisePostBackEvent(<span style="color: blue">string</span> eventArgument);
  </p>
  
  <p style="margin: 0px">
    }
  </p>
</div>

Esta interfaz nos permite hacer algo cuando hay un postback generado por el propio control. Por norma general lo que se hace es lanzar un evento de .NET (p.ej. Click). Vamos a ver un ejemplo muy rápido. Para ello añadimos un evento Click a nuestro control:

<div style="font-family: courier new; background: white; color: black; font-size: 10pt">
  <p style="margin: 0px">
    <span style="color: blue">public</span> <span style="color: blue">event</span> <span style="color: #2b91af">EventHandler</span> Click;
  </p>
  
  <p style="margin: 0px">
    <span style="color: blue">protected</span> <span style="color: blue">virtual</span> <span style="color: blue">void</span> OnClicked()
  </p>
  
  <p style="margin: 0px">
    {
  </p>
  
  <p style="margin: 0px">
    &nbsp;&nbsp;&nbsp; <span style="color: blue">var</span> handler = Click;
  </p>
  
  <p style="margin: 0px">
    &nbsp;&nbsp;&nbsp; <span style="color: blue">if</span> (handler != <span style="color: blue">null</span>) handler(<span style="color: blue">this</span>, <span style="color: #2b91af">EventArgs</span>.Empty);
  </p>
  
  <p style="margin: 0px">
    }
  </p>
</div>

Ahora implementamos la interfaz IPostBackEventHandler y el método RaisePostBackEvent:

<div style="font-family: courier new; background: white; color: black; font-size: 10pt">
  <p style="margin: 0px">
    <span style="color: blue">public</span> <span style="color: blue">void</span> RaisePostBackEvent(<span style="color: blue">string</span> eventArgument)
  </p>
  
  <p style="margin: 0px">
    {
  </p>
  
  <p style="margin: 0px">
    &nbsp;&nbsp;&nbsp; OnClicked();
  </p>
  
  <p style="margin: 0px">
    }
  </p>
</div>

Cuando ahora se pulse nuestro enlace se generará un evento Click que podemos capturar desde el Webform, como cualquier otro evento.

Además fijaos que RaisePostBackEvent recibe una cadena (eventArgument). El valor de esta cadena **es el valor del segundo parámetro de GetClientPostBackHyperlink** que hemos usado para generar la llamada a __doPostBack. De esta manera podemos crear controles que lancen varios eventos en función del valor de eventArgument.

Un saludo!

 [1]: http://tratadooscuro.blogspot.com.es/2009/02/dopostback-ese-gran-desconocido.html "http://tratadooscuro.blogspot.com.es/2009/02/dopostback-ese-gran-desconocido.html"
 [2]: http://programacion.porexpertos.es/provocar-un-postback-desde-javascript-con-aspnet/ "http://programacion.porexpertos.es/provocar-un-postback-desde-javascript-con-aspnet/"