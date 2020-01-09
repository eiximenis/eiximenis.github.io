---
title: ASP.NET MVC–Recibir contenido en mayúsculas
author: eiximenis

date: 2014-04-25T12:57:47+00:00
geeks_url: /?p=1666
geeks_visits:
  - 1001
geeks_ms_views:
  - 1277
categories:
  - Uncategorized

---
Imagina que estás desarrollando un proyecto, con ASP.NET MVC y cuando llevas digamos unas, no sé, cincuenta pantallas, todas llenas con sus formularios, algunos que hacen peticiones AJAX, otros que no… bueno, imagina que cuando llevas ya bastantes vistas hechas, aparece un nuevo requisito, de aquellos que están agazapados, esperando el momento propicio para saltate a la yugular: Todos los datos que entre el usuario, deben ser guardados y mostrados en mayúsculas.

Hay varias técnicas que puedes empezar a usar: Podríamos usar text-transform de CSS para transformar lo que ve el usuario en mayúsculas. Pero eso no sirve porque los datos que el servidor recibiría serían tal y como los ha entrado el usuario (text-transform cambia solo la representación pero no el propio texto).

Otra opción, claro está, es modificar todos los viewmodels porque al enlazar las propiedades las conviertan en mayúsculas. Así, si el usuario entra “edu” en la propiedad Name de un viewmodel, el viewmodel lo convierte a EDU y listos. Pero claro… si tienes muchos viewmodels ya hechos, eso puede ser muy tedioso. Y si no los tienes, pero vas a tenerlos, pues también…

En este dilema estaba un compañero de curro, cuando me planteó exactamente dicho problema. La cuestión era si había una manera que no implicase tener que tocar todos los viewmodels, para asegurar que recibíamos los datos siempre en mayúsculas. Y por suerte, la hay.

Si conoces un poco como funciona ASP.NET MVC internamente, sabrás que hay dos grupos de objetos que cooperan entre ellos para traducir los datos de la petición HTTP a los parámetros de la acción del controlador (que suele ser un viewmodel en el caso de una acción invocada via POST). Esos dos grupos de objetos son los value providers y los model binders.

La idea es muy sencilla: Los value providers recogen los datos de la petición HTTP y los dejan en un “saco común”. Los model binders recogen los datos de ese “saco común” y con esos datos recrean los valores de los parámetros de los controladores. De esa manera los value providers tan solo deben entender de la petición HTTP y los model binders solo deben entender como recrear los parámetros del controlador. Separación de responsabilidades.

Hay varios value providers porque cada uno de ellos se encarga de una parte de la petición HTTP. Así uno se encarga de los datos en la query string, otro de los form values (datos en el body usando application/x-www-form-urlencoded), otro de los datos en json… Y hay varios model binders, porque clases específicas pueden tener necesidades específicas de conversión. Aunque el DefaultModelBinder que viene de serie puede con casi todo, a veces es necesario crearse un model binder propio para suportar algunos escenarios.

En este caso la solución pasa **por usar un value provider nuevo**. Un value provider que recogerá los datos de la petición HTTP y los convertirá a maýsuculas antes de enviarlos a los model binders. Con eso los model binders recibirán los datos ya en mayúsculas, como si siempre hubiesen estado así. Solucionado el problema.

Veamos el código brevemente.

Lo primero es crear la factoría que cree el nuevo value provider. Los value providers se crean y se eliminan a cada petición, y el responsable de hacerlo es una factoría, que es el único objeto que existe durante todo el ciclo de vida de la aplicación:

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:5d4e1588-cb87-4f16-8ca5-6c08899fbd46" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
    <div style="background: #ddd; max-height: 300px; overflow: auto">
      <ol start="1" style="background: #1d1d1d; margin: 0 0 0 2.5em; padding: 0 0 0 5px; white-space: nowrap">
        <li>
          <span style="background:#1e1e1e;color:#569cd6">public</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">class</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#4ec9b0">ToUpperValueProviderFactory</span><span style="background:#1e1e1e;color:#dcdcdc"> : </span><span style="background:#1e1e1e;color:#4ec9b0">ValueProviderFactory</span>
        </li>
        <li style="background: #111111">
          <span style="background:#1e1e1e;color:#dcdcdc">{</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">    </span><span style="background:#1e1e1e;color:#569cd6">private</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">readonly</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#4ec9b0">ValueProviderFactory</span><span style="background:#1e1e1e;color:#dcdcdc"> _originalFactory;</span>
        </li>
        <li style="background: #111111">
          <span style="background:#1e1e1e;color:#dcdcdc">    </span><span style="background:#1e1e1e;color:#569cd6">public</span><span style="background:#1e1e1e;color:#dcdcdc"> ToUpperValueProviderFactory(</span><span style="background:#1e1e1e;color:#4ec9b0">ValueProviderFactory</span><span style="background:#1e1e1e;color:#dcdcdc"> originalFactory)</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">    {</span>
        </li>
        <li style="background: #111111">
          <span style="background:#1e1e1e;color:#dcdcdc">        _originalFactory </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> originalFactory;</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">    }</span>
        </li>
        <li style="background: #111111">
          <span style="background:#1e1e1e;color:#dcdcdc">    </span><span style="background:#1e1e1e;color:#569cd6">public</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">override</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#b8d7a3">IValueProvider</span><span style="background:#1e1e1e;color:#dcdcdc"> GetValueProvider(</span><span style="background:#1e1e1e;color:#4ec9b0">ControllerContext</span><span style="background:#1e1e1e;color:#dcdcdc"> controllerContext)</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">    {</span>
        </li>
        <li style="background: #111111">
          <span style="background:#1e1e1e;color:#dcdcdc">        </span><span style="background:#1e1e1e;color:#569cd6">var</span><span style="background:#1e1e1e;color:#dcdcdc"> provider </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> _originalFactory</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">GetValueProvider(controllerContext);</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">        </span><span style="background:#1e1e1e;color:#569cd6">return</span><span style="background:#1e1e1e;color:#dcdcdc"> provider </span><span style="background:#1e1e1e;color:#b4b4b4">!=</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">null</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#b4b4b4">?</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">new</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#4ec9b0">ToUpperProvider</span><span style="background:#1e1e1e;color:#dcdcdc">(provider) : </span><span style="background:#1e1e1e;color:#569cd6">null</span><span style="background:#1e1e1e;color:#dcdcdc">;</span>
        </li>
        <li style="background: #111111">
          <span style="background:#1e1e1e;color:#dcdcdc">    }</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">}</span>
        </li>
      </ol>
    </div></p>
  </div></p>
</div>

La idea es muy sencilla: dicha factoría deleg
  
a en la factoría original para obtener el value provider. Y luego devuelve un ToUpperProvider que va a ser nuestro value provider propio:

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:c724ebf7-4ad6-4b89-a96c-399fe6a7aed6" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
    <div style="background: #ddd; max-height: 300px; overflow: auto">
      <ol start="1" style="background: #1d1d1d; margin: 0 0 0 2.5em; padding: 0 0 0 5px; white-space: nowrap">
        <li>
          <span style="background:#1e1e1e;color:#569cd6">public</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">class</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#4ec9b0">ToUpperProvider</span><span style="background:#1e1e1e;color:#dcdcdc"> : </span><span style="background:#1e1e1e;color:#b8d7a3">IValueProvider</span>
        </li>
        <li style="background: #111111">
          <span style="background:#1e1e1e;color:#dcdcdc">{</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">    </span><span style="background:#1e1e1e;color:#569cd6">private</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">readonly</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#b8d7a3">IValueProvider</span><span style="background:#1e1e1e;color:#dcdcdc"> _realProvider;</span>
        </li>
        <li style="background: #111111">
          <span style="background:#1e1e1e;color:#dcdcdc">    </span><span style="background:#1e1e1e;color:#569cd6">public</span><span style="background:#1e1e1e;color:#dcdcdc"> ToUpperProvider(</span><span style="background:#1e1e1e;color:#b8d7a3">IValueProvider</span><span style="background:#1e1e1e;color:#dcdcdc"> realProvider)</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">    {</span>
        </li>
        <li style="background: #111111">
          <span style="background:#1e1e1e;color:#dcdcdc">        _realProvider </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> realProvider;</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">    }</span>
        </li>
        <li style="background: #111111">
          <span style="background:#1e1e1e;color:#dcdcdc">    </span><span style="background:#1e1e1e;color:#569cd6">public</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">bool</span><span style="background:#1e1e1e;color:#dcdcdc"> ContainsPrefix(</span><span style="background:#1e1e1e;color:#569cd6">string</span><span style="background:#1e1e1e;color:#dcdcdc"> prefix)</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">    {</span>
        </li>
        <li style="background: #111111">
          <span style="background:#1e1e1e;color:#dcdcdc">        </span><span style="background:#1e1e1e;color:#569cd6">return</span><span style="background:#1e1e1e;color:#dcdcdc"> _realProvider</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">ContainsPrefix(prefix);</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">    }</span>
        </li>
        <li style="background: #111111">
          <span style="background:#1e1e1e;color:#dcdcdc">    </span><span style="background:#1e1e1e;color:#569cd6">public</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#4ec9b0">ValueProviderResult</span><span style="background:#1e1e1e;color:#dcdcdc"> GetValue(</span><span style="background:#1e1e1e;color:#569cd6">string</span><span style="background:#1e1e1e;color:#dcdcdc"> key)</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">    {</span>
        </li>
        <li style="background: #111111">
          <span style="background:#1e1e1e;color:#dcdcdc">        </span><span style="background:#1e1e1e;color:#569cd6">var</span><span style="background:#1e1e1e;color:#dcdcdc"> result </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> _realProvider</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">GetValue(key);</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">        </span><span style="background:#1e1e1e;color:#569cd6">if</span><span style="background:#1e1e1e;color:#dcdcdc"> (result </span><span style="background:#1e1e1e;color:#b4b4b4">==</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">null</span><span style="background:#1e1e1e;color:#dcdcdc">)</span>
        </li>
        <li style="background: #111111">
          <span style="background:#1e1e1e;color:#dcdcdc">        {</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">            </span><span style="background:#1e1e1e;color:#569cd6">return</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">null</span><span style="background:#1e1e1e;color:#dcdcdc">;</span>
        </li>
        <li style="background: #111111">
          <span style="background:#1e1e1e;color:#dcdcdc">        }</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">        </span><span style="background:#1e1e1e;color:#569cd6">var</span><span style="background:#1e1e1e;color:#dcdcdc"> rawString </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> result</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">RawValue </span><span style="background:#1e1e1e;color:#569cd6">as</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">string</span><span style="background:#1e1e1e;color:#dcdcdc">;</span>
        </li>
        <li style="background: #111111">
          <span style="background:#1e1e1e;color:#dcdcdc">        </span><span style="background:#1e1e1e;color:#569cd6">if</span><span style="background:#1e1e1e;color:#dcdcdc"> (rawString </span><span style="background:#1e1e1e;color:#b4b4b4">!=</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">null</span><span style="background:#1e1e1e;color:#dcdcdc">)</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">        {</span>
        </li>
        <li style="background: #111111">
          <span style="background:#1e1e1e;color:#dcdcdc">            </span><span style="background:#1e1e1e;color:#569cd6">return</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">new</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#4ec9b0">ValueProviderResult</span><span style="background:#1e1e1e;color:#dcdcdc">(rawString</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">ToUpperInvariant(), </span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">                result</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">AttemptedValue</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">ToUpperInvariant(),</span>
        </li>
        <li style="background: #111111">
          <span style="background:#1e1e1e;color:#dcdcdc">                result</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">Culture);</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">        }</span>
        </li>
        <li style="background: #111111">
          <span style="background:#1e1e1e;color:#dcdcdc">        </span><span style="background:#1e1e1e;color:#569cd6">var</span><span style="background:#1e1e1e;color:#dcdcdc"> rawStrings </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> result</ span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">RawValue </span><span style="background:#1e1e1e;color:#569cd6">as</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">string</span><span style="background:#1e1e1e;color:#dcdcdc">[];</span></li> 
          
          <li>
            <span style="background:#1e1e1e;color:#dcdcdc">        </span><span style="background:#1e1e1e;color:#569cd6">if</span><span style="background:#1e1e1e;color:#dcdcdc"> (rawStrings </span><span style="background:#1e1e1e;color:#b4b4b4">!=</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">null</span><span style="background:#1e1e1e;color:#dcdcdc">)</span>
          </li>
          <li style="background: #111111">
            <span style="background:#1e1e1e;color:#dcdcdc">        {</span>
          </li>
          <li>
            <span style="background:#1e1e1e;color:#dcdcdc">            </span><span style="background:#1e1e1e;color:#569cd6">return</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">new</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#4ec9b0">ValueProviderResult</span><span style="background:#1e1e1e;color:#dcdcdc">(</span>
          </li>
          <li style="background: #111111">
            <span style="background:#1e1e1e;color:#dcdcdc">                rawStrings</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">Select(s </span><span style="background:#1e1e1e;color:#b4b4b4">=></span><span style="background:#1e1e1e;color:#dcdcdc"> s </span><span style="background:#1e1e1e;color:#b4b4b4">!=</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">null</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#b4b4b4">?</span><span style="background:#1e1e1e;color:#dcdcdc"> s</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">ToUpperInvariant() : </span><span style="background:#1e1e1e;color:#569cd6">null</span><span style="background:#1e1e1e;color:#dcdcdc">)</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">ToArray(),</span>
          </li>
          <li>
            <span style="background:#1e1e1e;color:#dcdcdc">                result</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">AttemptedValue</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">ToUpperInvariant(),</span>
          </li>
          <li style="background: #111111">
            <span style="background:#1e1e1e;color:#dcdcdc">                result</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">Culture);</span>
          </li>
          <li>
            <span style="background:#1e1e1e;color:#dcdcdc">        }</span>
          </li>
          <li style="background: #111111">
            <span style="background:#1e1e1e;color:#dcdcdc">        </span><span style="background:#1e1e1e;color:#569cd6">return</span><span style="background:#1e1e1e;color:#dcdcdc"> result;</span>
          </li>
          <li>
            <span style="background:#1e1e1e;color:#dcdcdc">    }</span>
          </li>
          <li style="background: #111111">
            <span style="background:#1e1e1e;color:#dcdcdc">}</span>
          </li></ol></div> </p></div> </p></div> 
          
          <p>
            La clase ToUpperProvider implementa la interfaz IValueProvider, pero delega en el value provider original.
          </p>
          
          <p>
            Lo único que hace es, en el método GetValue, una vez que tiene el valor original obtenido por el value provider original convertirlo a mayúsculas. Tratamos dos casuísticas: que el valor sea una cadena o un array de cadenas.
          </p>
          
          <p>
            ¡Y listos!
          </p>
          
          <p>
            El último paso es configurar ASP.NET MVC. P. ej. para hacer que todos los datos enviados via POST usando form data (es decir, un submit de form estándard) se reciban en mayúsculas basta con hacer (en el Application_Start de Global.asax.cs):
          </p>
          
          <div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:86cedda7-27b7-4dbc-993a-65dc895e5148" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
            <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
              <div style="background: #ddd; max-height: 300px; overflow: auto">
                <ol start="1" style="background: #1d1d1d; margin: 0 0 0 2em; padding: 0 0 0 5px; white-space: nowrap">
                  <li>
                    <span style="background:#1e1e1e;color:#569cd6">var</span><span style="background:#1e1e1e;color:#dcdcdc"> old </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#4ec9b0">ValueProviderFactories</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">Factories</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">OfType</span><span style="background:#1e1e1e;color:#b4b4b4"><</span><span style="background:#1e1e1e;color:#4ec9b0">FormValueProviderFactory</span><span style="background:#1e1e1e;color:#b4b4b4">></span><span style="background:#1e1e1e;color:#dcdcdc">()</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">FirstOrDefault();</span>
                  </li>
                  <li style="background: #111111">
                    <span style="background:#1e1e1e;color:#569cd6">if</span><span style="background:#1e1e1e;color:#dcdcdc"> (old </span><span style="background:#1e1e1e;color:#b4b4b4">!=</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">null</span><span style="background:#1e1e1e;color:#dcdcdc">)</span>
                  </li>
                  <li>
                    <span style="background:#1e1e1e;color:#dcdcdc">{</span>
                  </li>
                  <li style="background: #111111">
                    <span style="background:#1e1e1e;color:#dcdcdc">    </span><span style="background:#1e1e1e;color:#4ec9b0">ValueProviderFactories</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">Factories</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">Remove(old);</span>
                  </li>
                  <li>
                    <span style="background:#1e1e1e;color:#dcdcdc">    </span><span style="background:#1e1e1e;color:#4ec9b0">ValueProviderFactories</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">Factories</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">Add(</span><span style="background:#1e1e1e;color:#569cd6">new</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#4ec9b0">ToUpperValueProviderFactory</span><span style="background:#1e1e1e;color:#dcdcdc">(old));</span>
                  </li>
                  <li style="background: #111111">
                    <span style="background:#1e1e1e;color:#dcdcdc">}</span>
                  </li>
                </ol>
              </div></p>
            </div></p>
          </div>
          
          <p>
            Con eso sustituimos la factoría original que devuelve el value provider que se encarga de los datos en el form data por nuestra propia factoría.
          </p>
          
          <p>
            Para probarlo basta con crear una vista con un formulario y hacer un post normal y corriente… y verás que todos los datos que se entren se pasarán a mayúsculas automáticamente 🙂
          </p>
          
          <p>
            Saludos!
          </p>