---
title: Mi (pequeño) homenaje al gran, genial e irrepetible Terry Pratchett
author: eiximenis

date: 2015-03-20T11:55:21+00:00
geeks_url: /?p=1697
geeks_visits:
  - 763
geeks_ms_views:
  - 1014
categories:
  - Uncategorized

---
<a href="http://es.wikipedia.org/wiki/Terry_Pratchett" target="_blank" rel="noopener noreferrer">Terry Pratchett</a> ha sido uno de los grandes escritores de novelas de fantasía. Su serie más conocida <a href="http://es.wikipedia.org/wiki/Mundodisco" target="_blank" rel="noopener noreferrer">Mundodisco</a>, cuenta con 41 libros escritos en un estilo desenfadado y humorístico que parodian el género fantástico pero que a la vez encierran durísimas y mordaces críticas contra muchos aspectos de nuestra sociedad. El humor de Pratchett es reconocido como uno de los más ácidos e inteligentes a la vez que absurdos y esta mezcla es explosiva: sus libros te hacen estallar en carcajadas a la vez que reflexionar. Para mi ha sido uno de los escritores que más me ha impactado.

Son muy conocidas sus frases, sacadas tanto de sus libros como de sus charlas y hay webs que se dedican a recopilarlas. Algunas hacen gracia por si mismas, otras hacen mucha más gracia dentro del contexto del libro. En inglés, la página “L-Space Web” contiene probablemente la <a href="http://www.lspace.org/books/pqf/" target="_blank" rel="noopener noreferrer">mayor recopilación de frases de Pratchett</a>. En castellano es “la concha de gran A’Tuin” quien contiene <a href="http://mundodisco.dreamers.com/citas.htm" target="_blank" rel="noopener noreferrer">otra buena colección</a>.

Ahora que Terry Pratchett ha muerto he pensado que es el momento de hacerle un pequeño (pequeñísimo) homenaje, similar a otros que se están haciendo al estilo de “<a href="http://www.gnuterrypratchett.com/" target="_blank" rel="noopener noreferrer">GNU Terry Pratchett</a>”.

En mi caso **he dedicido crear un middlewarwe OWIN que añade una cabecera HTTP con una cita de Terry Pratchett al azar**. Las citas están extraídas de la lista contenida en L-Space Web, aunque el middleware es configurable para que puedas usar tus propias citas si lo prefieres.

El código principal del middleware es muy sencillo:

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:d63dd64d-de6a-40ed-8fa0-46bb890f0eef" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
    <div style="background: #ddd; max-height: 300px; overflow: auto">
      <ol start="1" style="background: #000000; margin: 0 0 0 2.5em; padding: 0 0 0 5px;">
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">public</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">class</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#4ec9b0">PratchettOwinMiddleware</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">{</span>
        </li>
        <li>
              <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">private</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">readonly</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#4ec9b0">AppFunc</span><span style="background:#1e1e1e;color:#dcdcdc"> _next;</span>
        </li>
        <li>
              <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">private</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">readonly</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#4ec9b0">PratchettQuotesFactory</span><span style="background:#1e1e1e;color:#dcdcdc"> _quotesFactory;</span>
        </li>
        <li>
              <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">private</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">readonly</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#4ec9b0">Random</span><span style="background:#1e1e1e;color:#dcdcdc"> _random;</span>
        </li>
        <li>
              <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">public</span><span style="background:#1e1e1e;color:#dcdcdc"> PratchettOwinMiddleware(</span><span style="background:#1e1e1e;color:#4ec9b0">AppFunc</span><span style="background:#1e1e1e;color:#dcdcdc"> next, </span><span style="background:#1e1e1e;color:#4ec9b0">PratchettQuotesFactory</span><span style="background:#1e1e1e;color:#dcdcdc"> quotesFactory)</span>
        </li>
        <li>
              <span style="background:#1e1e1e;color:#dcdcdc">{</span>
        </li>
        <li>
                  <span style="background:#1e1e1e;color:#dcdcdc">_next </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> next;</span>
        </li>
        <li>
                  <span style="background:#1e1e1e;color:#dcdcdc">_quotesFactory </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> quotesFactory;</span>
        </li>
        <li>
                  <span style="background:#1e1e1e;color:#dcdcdc">_random </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">new</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#4ec9b0">Random</span><span style="background:#1e1e1e;color:#dcdcdc">();</span>
        </li>
        <li>
              <span style="background:#1e1e1e;color:#dcdcdc">}</span>
        </li>
        <li>
          &nbsp;
        </li>
        <li>
              <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">public</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">async</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#4ec9b0">Task</span><span style="background:#1e1e1e;color:#dcdcdc"> Invoke(</span><span style="background:#1e1e1e;color:#b8d7a3">IDictionary</span><span style="background:#1e1e1e;color:#b4b4b4"><</span><span style="background:#1e1e1e;color:#569cd6">string</span><span style="background:#1e1e1e;color:#dcdcdc">, </span><span style="background:#1e1e1e;color:#569cd6">object</span><span style="background:#1e1e1e;color:#b4b4b4">></span><span style="background:#1e1e1e;color:#dcdcdc"> environment)</span>
        </li>
        <li>
              <span style="background:#1e1e1e;color:#dcdcdc">{</span>
        </li>
        <li>
                  <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">await</span><span style="background:#1e1e1e;color:#dcdcdc"> _next</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">Invoke(environment);</span>
        </li>
        <li>
                  <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">var</span><span style="background:#1e1e1e;color:#dcdcdc"> headers </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> environment[</span><span style="background:#1e1e1e;color:#d69d85">"owin.ResponseHeaders"</span><span style="background:#1e1e1e;color:#dcdcdc">] </span><span style="background:#1e1e1e;color:#569cd6">as</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#b8d7a3">IDictionary</span><span style="background:#1e1e1e;color:#b4b4b4"><</span><span style="background:#1e1e1e;color:#569cd6">string</span><span style="background:#1e1e1e;color:#dcdcdc">, </span><span style="background:#1e1e1e;color:#569cd6">string</span><span style="background:#1e1e1e;color:#dcdcdc">[]</span><span style="background:#1e1e1e;color:#b4b4b4">></span><span style="background:#1e1e1e;color:#dcdcdc">;</span>
        </li>
        <li>
                  <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">var</span><span style="background:#1e1e1e;color:#dcdcdc"> quotes </s pan><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> _quotesFactory</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">GetQuotes();</span></li> 
          
          <li>
                    <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">var</span><span style="background:#1e1e1e;color:#dcdcdc"> quote </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> quotes[_random</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">Next(</span><span style="background:#1e1e1e;color:#b5cea8"></span><span style="background:#1e1e1e;color:#dcdcdc">, quotes</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">Length)];</span>
          </li>
          <li>
                    <span style="background:#1e1e1e;color:#dcdcdc">headers</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">Add(</span><span style="background:#1e1e1e;color:#d69d85">"X-Pratchett-Quote"</span><span style="background:#1e1e1e;color:#dcdcdc">, </span><span style="background:#1e1e1e;color:#569cd6">new</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">string</span><span style="background:#1e1e1e;color:#dcdcdc">[] { quote});</span>
          </li>
          <li>
                <span style="background:#1e1e1e;color:#dcdcdc">}</span>
          </li>
          <li>
            <span style="background:#1e1e1e;color:#dcdcdc">}</span>
          </li></ol></div> </p></div> </p></div> </p> 
          
          <p>
            Básicamente se dedica a añadir la cabecera X-Pratchett-Quote a cualquier petición que sea procesada. Para registrarlo en el pipeline de OWIN se usa un método de extensión sobre IAppBuilder definido en la clase PratchettAppBuilderExtensions:
          </p>
          
          <div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:c58af828-ad45-4ea0-a188-8e3fa29fff95" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
            <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
              <div style="background: #ddd; max-height: 300px; overflow: auto">
                <ol start="1" style="background: #000000; margin: 0 0 0 2.5em; padding: 0 0 0 5px;">
                  <li>
                      <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">public</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">static</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">class</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#4ec9b0">PratchettAppBuilderExtensions</span>
                  </li>
                  <li>
                      <span style="background:#1e1e1e;color:#dcdcdc">{</span>
                  </li>
                  <li>
                          <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">public</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">static</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">void</span><span style="background:#1e1e1e;color:#dcdcdc"> UseTerryPratchett(</span><span style="background:#1e1e1e;color:#569cd6">this</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#b8d7a3">IAppBuilder</span><span style="background:#1e1e1e;color:#dcdcdc"> app)</span>
                  </li>
                  <li>
                          <span style="background:#1e1e1e;color:#dcdcdc">{</span>
                  </li>
                  <li>
                              <span style="background:#1e1e1e;color:#dcdcdc">app</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">Use(</span><span style="background:#1e1e1e;color:#569cd6">typeof</span><span style="background:#1e1e1e;color:#dcdcdc">(</span><span style="background:#1e1e1e;color:#4ec9b0">PratchettOwinMiddleware</span><span style="background:#1e1e1e;color:#dcdcdc">), </span>
                  </li>
                  <li>
                                  <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">new</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#4ec9b0">PratchettQuotesFactory</span><span style="background:#1e1e1e;color:#dcdcdc">(</span>
                  </li>
                  <li>
                                      <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">new</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#4ec9b0">InternalFileQuoteParser</span><span style="background:#1e1e1e;color:#dcdcdc">(),</span>
                  </li>
                  <li>
                    <span style="background:#1e1e1e;color:#dcdcdc">                    () </span><span style="background:#1e1e1e;color:#b4b4b4">=></span><span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#4ec9b0">Assembly</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">GetExecutingAssembly()</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">GetManifestResourceStream(</span><span style="background:#1e1e1e;color:#d69d85">"PratchettQuotes.terry_quotes.txt"</span><span style="background:#1e1e1e;color:#dcdcdc">)));</span>
                  </li>
                  <li>
                          <span style="background:#1e1e1e;color:#dcdcdc">}</span>
                  </li>
                  <li>
                    &nbsp;
                  </li>
                  <li>
                          <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">public</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">static</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">void</span><span style="background:#1e1e1e;color:#dcdcdc"> UseTerryPratchett(</span><span style="background:#1e1e1e;color:#569cd6">this</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#b8d7a3">IAppBuilder</span><span style="background:#1e1e1e;color:#dcdcdc"> app, </span><span style="background:#1e1e1e;color:#b8d7a3">IQuoteParser</span><span style="background:#1e1e1e;color:#dcdcdc"> quoteParser, </span><span style="background:#1e1e1e;color:#569cd6">string</span><span style="background:#1e1e1e;color:#dcdcdc"> filename)</span>
                  </li>
                  <li>
                          <span style="background:#1e1e1e;color:#dcdcdc">{</span>
                  </li>
                  <li>
                              <span style="background:#1e1e1e;color:#dcdcdc">app</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">Use(</span><span style="background:#1e1e1e;color:#569cd6">typeof</span><span style="background:#1e1e1e;color:#dcdcdc">(</span><span style="background:#1e1e1e;color:#4ec9b0">PratchettOwinMiddleware</span><span style="background:#1e1e1e;color:#dcdcdc">),</span>
                  </li>
                  <li>
                                  <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">new</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#4ec9b0">PratchettQuotesFactory</span><span style="background:#1e1e1e;color:#dcdcdc">(quoteParser, () </span><span style="background:#1e1e1e;color:#b4b4b4">=></span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">new</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#4ec9b0">FileStream</span><span style="background:#1e1e1e;color:#dcdcdc">(filename, </span><span style="background:#1e1e1e;color:#b8d7a3">FileMode</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">Open, </span><span style="background:#1e1e1e;color:#b8d7a3">FileAccess</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcd
cdc">Read)));</span>
                  </li>
                  <li>
                          <span style="background:#1e1e1e;color:#dcdcdc">}</span>
                  </li>
                  <li>
                    &nbsp;
                  </li>
                  <li>
                          <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">public</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">static</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">void</span><span style="background:#1e1e1e;color:#dcdcdc"> UseTerryPratchett(</span><span style="background:#1e1e1e;color:#569cd6">this</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#b8d7a3">IAppBuilder</span><span style="background:#1e1e1e;color:#dcdcdc"> app, </span><span style="background:#1e1e1e;color:#b8d7a3">IQuoteParser</span><span style="background:#1e1e1e;color:#dcdcdc"> quoteParser, </span><span style="background:#1e1e1e;color:#4ec9b0">Func</span><span style="background:#1e1e1e;color:#b4b4b4"><</span><span style="background:#1e1e1e;color:#4ec9b0">Stream</span><span style="background:#1e1e1e;color:#b4b4b4">></span><span style="background:#1e1e1e;color:#dcdcdc"> quotesProvider)</span>
                  </li>
                  <li>
                          <span style="background:#1e1e1e;color:#dcdcdc">{</span>
                  </li>
                  <li>
                              <span style="background:#1e1e1e;color:#dcdcdc">app</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">Use(</span><span style="background:#1e1e1e;color:#569cd6">typeof</span><span style="background:#1e1e1e;color:#dcdcdc">(</span><span style="background:#1e1e1e;color:#4ec9b0">PratchettOwinMiddleware</span><span style="background:#1e1e1e;color:#dcdcdc">), </span><span style="background:#1e1e1e;color:#569cd6">new</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#4ec9b0">PratchettQuotesFactory</span><span style="background:#1e1e1e;color:#dcdcdc">(quoteParser, quotesProvider));</span>
                  </li>
                  <li>
                          <span style="background:#1e1e1e;color:#dcdcdc">}</span>
                  </li>
                  <li>
                      <span style="background:#1e1e1e;color:#dcdcdc">}</span>
                  </li>
                </ol>
              </div></p>
            </div></p>
          </div>
          
          <p>
            El método está sobrecargado para admitir tus propias citas. En este caso debes pasar también una clase que implemente la interfaz IQuoteParser que indique como leer los datos del stream que contiene las citas. No se incluye ninguna implementación de dicha interfaz (bueno, se incluye una pero es interna ya que se usa para parsear los datos del fichero de citas, que está sacado literalmente de L-Space Web).
          </p>
          
          <p>
            Si llamas simplemente a UseTerryPratchett() se usará el fichero de citas que está incrustado dentro del ensamblado.
          </p>
          
          <p>
            El resultado lo puedes ver en esta captura de la pestaña red de Chrome:
          </p>
          
          <p>
            <a href="http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_39F7890F.png"><img title="image" style="border-top: 0px; border-right: 0px; background-image: none; border-bottom: 0px; padding-top: 0px; padding-left: 0px; border-left: 0px; display: inline; padding-right: 0px" border="0" alt="image" src="http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_12539D96.png" width="1136" height="77" /></a>
          </p>
          
          <p>
            El código fuente del proyecto lo teneis en github: <a title="https://github.com/eiximenis/PratchettQuotes" href="https://github.com/eiximenis/PratchettQuotes">https://github.com/eiximenis/PratchettQuotes</a>
          </p>
          
          <p>
            Saludos!:D
          </p>