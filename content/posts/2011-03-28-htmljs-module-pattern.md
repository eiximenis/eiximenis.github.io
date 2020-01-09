---
title: '[HTML/JS] Module pattern'
author: eiximenis

date: 2011-03-28T17:31:53+00:00
geeks_url: /?p=1562
geeks_visits:
  - 1780
geeks_ms_views:
  - 677
categories:
  - Uncategorized

---
Muy buenas! Cuando creas un sitio web, es normal que vayas añadiendo cada vez más código javascript en él. Al final, seguramente terminaréis desarollando una mini-api, propia que muchas veces reside en un archivo .js, lleno de funciones. Algo como:

<div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
  <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; background-color: #f4f4f4; margin: 0em; border-left-style: none; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; border-right-style: none; font-size: 8pt; overflow: visible; padding-top: 0px" id="codeSnippet"><span style="color: #0000ff">function</span> HazAlgo(id, <span style="color: #0000ff">params</span>, callback)<br />{<br />}<br /><br /><span style="color: #0000ff">function</span> _HazAlgoHelper(domObj, <span style="color: #0000ff">params</span>)<br />{<br />}</pre>
  
  <p>
    </div> 
    
    <p>
      En este caso tenemos dos funciones, HazAlgo y _HazAlgoHelper. La función _HazAlgoHelper es realmente una función <em>privada</em>, es decir está pensada para ser llamada únicamente dentro de HazAlgo. Pero en Javascript no existe directamente el concepto de <em>función privada</em> así que le ponemos un idicador (que empieze por _) y asumimos que todas las funciones que empiecen por _, son privadas.
    </p>
    
    <p>
      Como podemos ver, el código javascript no está muy “organizado”: tenemos todas las funciones juntas y además podemos ver aquellas que no deberíamos. El patrón de módulo existe para solventar esos problemas.
    </p>
    
    <p>
      <strong>El patrón de módulo</strong>
    </p>
    
    <p>
      La idea del patrón de módulo (module pattern) en Javascript es simular el concepto de una clase estática con métodos públicos y métodos privados. Lo de clase estática viene porque no quiero ir creando objetos de mi módulo, simplemente quiero <em>tener ahí</em>, las funciones públicas. Pero sin ver las privadas. Para los que no lo sepáis: en javascript no existe el concepto de <em>variable estática</em>.
    </p>
    
    <p>
      La idea que hay detrás del patrón de módulo es muy simple, y como casi siempre pasa simple significa también brillante: Se trata de crear un <em>objeto anónimo</em>, cuyos campos públicos sean las funciones públicas. Finalmente se crea una instancia de ese objeto y se asigna al namespace <em>global</em> (eso es, al objeto window).
    </p>
    
    <p>
      El código de base sería algo como:
    </p>
    
    <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
      <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; background-color: #f4f4f4; margin: 0em; border-left-style: none; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; border-right-style: none; font-size: 8pt; overflow: visible; padding-top: 0px" id="codeSnippet">(<span style="color: #0000ff">function</span>(wnd) {<br />    <span style="color: #0000ff">var</span> miModulo = <span style="color: #0000ff">function</span>() {<br />        <span style="color: #0000ff">var</span> _HazAlgoHelper = <span style="color: #0000ff">function</span>(domObj, <span style="color: #0000ff">params</span>) {alert(<span style="color: #006080">"método privado"</span>);};<br />        <span style="color: #0000ff">var</span> _HazAlgo = <span style="color: #0000ff">function</span>(id, <span style="color: #0000ff">params</span>, callback) { _HazAlgoHelper(); alert(<span style="color: #006080">"método público"</span>}; };<br /><br />        <span style="color: #0000ff">return</span> {<br />           HazAlgo : _HazAlgo<br />        };<br />    }<br /><br />    wnd.m$ = miModulo();<br />})(window);</pre>
      
      <p>
        </div> 
        
        <p>
          Bien… se que el código puede parecer lioso, la primera vez, pero vamos a analizarlo.
        </p>
        
        <p>
          Podemos ver que estamos definiendo una función <strong>anónima</strong> que acepta un parámetro (llamado <em>wnd</em>).
        </p>
        
        <p>
          ¿Y que hace esa función anónima)? Pues para empezar define una variable miModulo que resulta ser… una función (<em>var miModulo = funcion() { …}</em>).
        </p>
        
        <p>
          Y que hará esa función cuando se invoque? Pues tres cosas:
        </p>
        
        <ol>
          <li>
            Definir una variable llamada _HazAlgoHelper… que es <em>otra función</em>.
          </li>
          <li>
            Definir una variable llamada _HazAlgo… <em>que es otra función</em>.
          </li>
          <li>
            Devolver un objeto anónimo con un campo, llamado HazAlgo. El valor de HazAlgo es el mismo que _HazAlgo (es decir una función).
          </li>
        </ol>
        
        <p>
          Con eso tenemos una función (miModulo) que cuando la invoquemos nos devolverá un objeto con un solo campo (llamado HazAlgo). Así, teoricamente:
        </p>
        
        <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
          <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; background-color: #f4f4f4; margin: 0em; border-left-style: none; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; border-right-style: none; font-size: 8pt; overflow: visible; padding-top: 0px" id="codeSnippet">miModulo.HazAlgo();    <span style="color: #008000">// Valido</span><br />miModulo._HazAlgo();   <span style="color: #008000">// No válido</span><br />miModulo._HazAlgoHelper(); // NO válido</pre>
          
          <p>
            </div> 
            
            <p>
              Fijaos que la gracia está en que HazAlgo() realmente es lo mismo que _HazAlgo()… y desde _HazAlgo() podemos llamar sin ningún problema a _HazAlgoHelper() que es nuestra función privada. Así la clave es mapear las funciones públicas como campos del objeto anónimo devuelto.
            </p>
            
            <p>
              ¡Con eso hemos simulado el concepto de funciones privadas!
            </p>
            
            <p>
              Pero, miModulo es una variable local, es local a la función anónima que estamos definiendo. Así que todavía nos queda un paso más: Guardar miModulo en alguna propiedad del parámetro wnd. Eso es lo que hacemos con la línea:
            </p>
            
            <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
              <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; background-color: #f4f4f4; margin: 0em; border-left-style: none; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; border-right-style: none; font-size: 8pt; overflow: visible; padding-top: 0px" id="codeSnippet">wnd.m$ = miModulo();</pre>
              
              <p>
                </div> 
                
                <p>
                  Invocamos miModulo (con parentesis, pues es una función) y guardamos el resultado (el objeto anónimo con el campo HazAlgo) en la propiedad m$ del parámetro wnd.
                </p>
                
                <p>
                  Ya casi estamos… Hasta este punto hemos definido simplemente una función anónima. Ahora toca invocarla. Fijaos en un detalle importante: la primera línea empieza con un paréntesis abierto. Con eso, de hecho, estamos <strong>invocando</strong> nuestra función anónima. Pero nuestra función anónima espera un parámetro (wnd) así que debemos pasárselo. Eso es lo que hacemos en la última línea: le pasamos window. Y porque window? Pues porque window es un namespace global: todo lo que esté en window es global.
                </p>
                
                <p>
                  Por lo tanto ahora, desde cualquier parte de mi página puedo hacer:
                </p>
                
                <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
                  <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; background-color: #f4f4f4; margin: 0em; border-left-style: none; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; border-right-style: none; font-size: 8pt; overflow: visible; padding-top: 0px" id="codeSnippet">&lt;script&gt;m$.HazAlgo(1,2,3);&lt;/script&gt;</pre>
                  
                  <p>
                    </div> 
                    
                    <p>
                      Y eso es correcto, mientras que hacer.
                    </p>
                    
                    <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
                      <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; background-color: #f4f4f4; margin: 0em; border-left-style: none; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; border-right-style: none; font-size: 8pt; overflow: visible; padding-top: 0px" id="codeSnippet">&lt;script&gt;m$._HazAlgoHelper(1,2);&lt;/script&gt;</pre>
                      
                      <p>
                        </div> 
                        
                        <p>
                          Nos dará un error.
                        </p>
                        
                        <p>
                          Para tener nuestro módulo listo para ser usado, basta con tenerlo en un .js propio e incluirlo en aquellas páginas donde lo necesitemos. ¡Y listos!
                        </p>
                        
                        <p>
                          Espero que esto os parezca interesante y os ayude a organizar vuestro código javascript!
                        </p>
                        
                        <p>
                          Un saludo!
                        </p>