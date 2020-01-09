---
title: Desacopla tus datos XML del formato…
description: Desacopla tus datos XML del formato…
author: eiximenis

date: 2010-05-26T12:56:55+00:00
geeks_url: /?p=1512
geeks_visits:
  - 6749
geeks_ms_views:
  - 4260
categories:
  - Uncategorized

---
Leyendo <a href="http://geeks.ms/blogs/gtorres/archive/2010/05/24/xmlserializer-y-xml-attributes.aspx" target="_blank" rel="noopener noreferrer">este post de Gisela sobre la serialización XML</a> me he decidido escribir este… es lo que tiene la realimentación en los blogs 🙂

El uso de atributos que menciona Gis en su post es realmente genial. A mi me encanta: me permite definir mis clases en un momento y es muy útil cuando leemos datos xml de una fuente externa. Pero hay un detalle que _puede_ ser un problema: El esquema XML está totalmente acoplado de la clase que tiene los datos. Si estamos leyendo de dos fuentes externas que tienen esquemas XML distintos pero tienen los mismos datos, debemos duplicar las clases que serializan esos datos, ya que los atributos a usar serán distintos.

P.ej. supon que tenemos dos fuentes externas, que nos devuelven los mismos datos, pero de forma diferente:

<div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
  <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px" id="codeSnippet"><span style="color: #0000ff">&lt;</span><span style="color: #800000">info</span><span style="color: #0000ff">&gt;</span><br />  <span style="color: #0000ff">&lt;</span><span style="color: #800000">usuarios</span><span style="color: #0000ff">&gt;</span><br />    <span style="color: #0000ff">&lt;</span><span style="color: #800000">usuario</span><span style="color: #0000ff">&gt;</span><br />      <span style="color: #0000ff">&lt;</span><span style="color: #800000">nombre</span><span style="color: #0000ff">&gt;</span>eiximenis<span style="color: #0000ff">&lt;/</span><span style="color: #800000">nombre</span><span style="color: #0000ff">&gt;</span><br />      <span style="color: #0000ff">&lt;</span><span style="color: #800000">nombrereal</span><span style="color: #0000ff">&gt;</span>Edu<span style="color: #0000ff">&lt;/</span><span style="color: #800000">nombrereal</span><span style="color: #0000ff">&gt;</span><br />    <span style="color: #0000ff">&lt;/</span><span style="color: #800000">usuario</span><span style="color: #0000ff">&gt;</span><br />  <span style="color: #0000ff">&lt;/</span><span style="color: #800000">usuarios</span><span style="color: #0000ff">&gt;</span><br /><span style="color: #0000ff">&lt;/</span><span style="color: #800000">info</span><span style="color: #0000ff">&gt;</span></pre>
  
  <p>
    </div> 
    
    <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
      <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px" id="codeSnippet"><span style="color: #0000ff">&lt;</span><span style="color: #800000">information</span><span style="color: #0000ff">&gt;</span><br />  <span style="color: #0000ff">&lt;</span><span style="color: #800000">twitter.users</span><span style="color: #0000ff">&gt;</span><br />    <span style="color: #0000ff">&lt;</span><span style="color: #800000">twitter.user</span> name="eiximenis" realname="Edu" <span style="color: #0000ff">/&gt;</span><br />  <span style="color: #0000ff">&lt;/</span><span style="color: #800000">twitter.users</span><span style="color: #0000ff">&gt;</span><br /><span style="color: #0000ff">&lt;/</span><span style="color: #800000">information</span><span style="color: #0000ff">&gt;</span></pre>
      
      <p>
        </div> 
        
        <p>
          La información es exactamente la misma, pero el esquema es distinto. <u>Si queremos usar atributos</u> para leer estos dos archivos xml debemos implementar dos conjuntos de clases:
        </p>
        
        <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
          <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px" id="codeSnippet"><span style="color: #008000">// Clases para leer el 1er formato</span><br />[XmlRoot(<span style="color: #006080">"info"</span>)]<br /><span style="color: #0000ff">public</span> <span style="color: #0000ff">class</span> Info<br />{<br />    <span style="color: #0000ff">private</span> <span style="color: #0000ff">readonly</span> List&lt;UserInfo&gt; _users;<br /><br />    <span style="color: #0000ff">public</span> Info()<br />    {<br />        _users = <span style="color: #0000ff">new</span> List&lt;UserInfo&gt;();<br />    }<br /><br />    [XmlArray(<span style="color: #006080">"usuarios"</span>)]<br />    [XmlArrayItem(<span style="color: #006080">"usuario"</span>, <span style="color: #0000ff">typeof</span>(UserInfo))]<br />    <span style="color: #0000ff">public</span> List&lt;UserInfo&gt; Usuarios { get { <span style="color: #0000ff">return</span> _users; } }<br />}<br /><br /><span style="color: #0000ff">public</span> <span style="color: #0000ff">class</span> UserInfo<br />{<br />    [XmlElement(<span style="color: #006080">"nombre"</span>)]<br />    <span style="color: #0000ff">public</span> <span style="color: #0000ff">string</span> Nombre { get; set; }<br /><br />    [XmlElement(<span style="color: #006080">"nombrereal"</span>)]<br />    <span style="color: #0000ff">public</span> <span style="color: #0000ff">string</span> NombreReal { get; set; }<br />}</pre>
          
          <p>
            </div> 
            
            <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
              <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px" id="codeSnippet"><span style="color: #008000">// Clases para leer el segundo formato</span><br />[XmlRoot(<span style="color: #006080">"information"</span>)]<br /><span style="color: #0000ff">public</span> <span style="color: #0000ff">class</span> TweeterInfo<br />{<br />    <span style="color: #0000ff">private</span> <span style="color: #0000ff">readonly</span> List&lt;TweeterUserInfo&gt; _users;<br /><br />    <span style="color: #0000ff">public</span> TweeterInfo()<br />    {<br />        _users = <span style="color: #0000ff">new</span> List&lt;TweeterUserInfo&gt;();<br />    }<br /><br />    [XmlArray(<span style="color: #006080">"twitter.users"</span>)]<br />    [XmlArrayItem(<span style="color: #006080">"twitter.user"</span>, <span style="color: #0000ff">typeof</span>(TweeterUserInfo))]<br />    <span style="color: #0000ff">public</span> List&lt;TweeterUserInfo&gt; Users { get { <span style="color: #0000ff">return</span> _users; } }<br />}<br /><br /><span style="color: #0000ff">public</span> <span style="color: #0000ff">class</span> TweeterUserInfo<br />{<br />    [XmlAttribute(<span style="color: #006080">"name"</span>)]<br />    <span style="color: #0000ff">public</span> <span style="color: #0000ff">string</span> Name { get; set; }<br /><br />    [XmlAttribute(<span style="color: #006080">"realname"</span>)]<br />    <span style="color: #0000ff">public</span> <span style="color: #0000ff">string</span> RealName { get; set; }<br />}</pre>
              
              <p>
                </div> 
                
                <p>
                  El problema no es que tengamos que realizar el doble de trabajo… es que tenemos dos conjuntos de clases <strong>totalmente distintos</strong> que no tienen ninguna relación entre ellos. Si desarrollamos un método que trabaje con objetos de la clase Info, dicho método no trabajará con objetos de la clase TweeterInfo <em>aún cuando ambas clases representan la misma información</em>.
                </p>
                
                <p>
                  La solución pasa, obviamente, por usar interfaces: Ambas clases deberían implementar una misma inferfaz que nos permitiese acceder a los datos:
                </p>
                
                <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
                  <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px" id="codeSnippet"><span style="color: #008000">// Interfaces</span><br /><br /><span style="color: #0000ff">public</span> <span style="color: #0000ff">interface</span> ITwitterInfo<br />{<br />    IEnumerable&lt;ITwitterUser&gt; Users { get; }<br />}<br /><span style="color: #0000ff">public</span> <span style="color: #0000ff">interface</span> ITwitterUser<br />{<br />    <span style="color: #0000ff">string</span> Name { get; }<br />    <span style="color: #0000ff">string</span> RealName { get; }<br />}</pre>
                  
                  <p>
                    </div> 
                    
                    <p>
                      Las interfaces se limitan a definir los datos que vamos a consultar desde nuestra aplicación. En este caso sólo hay getters porque se supone que dichos datos son de lectura sólamente.
                    </p>
                    
                    <p>
                      El siguiente paso es implementar dichas interfaces en nuestras clases. Lo mejor es usar una implementación <strong>explícita</strong>. La razón es evitar tipos de retorno distintos entre propiedades que pueden tener el mismo nombre (p.ej. la propiedad Users de TweeterInfo devuelve una List<TweeterUserInfo> mientras que la propiedad de la interfaz ITwitterInfo devuelve un IEnumerable<ITwitterUser> y eso no compilaría:
                    </p>
                    
                    <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
                      <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px" id="codeSnippet">[XmlRoot(<span style="color: #006080">"information"</span>)]<br /><span style="color: #0000ff">public</span> <span style="color: #0000ff">class</span> TweeterInfo : ITwitterInfo<br />{<br />    <span style="color: #0000ff">private</span> <span style="color: #0000ff">readonly</span> List&lt;TweeterUserInfo&gt; _users;<br /><br />    <span style="color: #0000ff">public</span> TweeterInfo()<br />    {<br />        _users = <span style="color: #0000ff">new</span> List&lt;TweeterUserInfo&gt;();<br />    }<br /><br />    [XmlArray(<span style="color: #006080">"twitter.users"</span>)]<br />    [XmlArrayItem(<span style="color: #006080">"twitter.user"</span>, <span style="color: #0000ff">typeof</span>(TweeterUserInfo))]<br />    <span style="color: #0000ff">public</span> List&lt;TweeterUserInfo&gt; Users { get { <span style="color: #0000ff">return</span> _users; } }<br />}<br /><br /><span style="color: #008000">// error CS0738: 'XmlDesacoplado.Formato2.TweeterInfo' does not implement interface member </span><br /><span style="color: #008000">// 'XmlDesacoplado.Interfaz.ITwitterInfo.Users'. 'XmlDesacoplado.Formato2.TweeterInfo.Users' cannot </span><br /><span style="color: #008000">// implement 'XmlDesacoplado.Interfaz.ITwitterInfo.Users' because it does not have the matching return type </span><br /><span style="color: #008000">// of 'System.Collections.Generic.IEnumerable&lt;XmlDesacoplado.Interfaz.ITwitterUser&gt;'</span><br /></pre>
                      
                      <p>
                        </div> 
                        
                        <p>
                          Como os digo, para que funcione basta con una implementación explícita de la interfaz. Es decir basta con añadir:
                        </p>
                        
                        <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
                          <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px" id="codeSnippet"><span style="color: #008000">// Implementación explícita</span><br />IEnumerable&lt;ITwitterUser&gt; ITwitterInfo.Users { get { <span style="color: #0000ff">return</span> Users; } }<br /></pre>
                          
                          <p>
                            </div> 
                            
                            <p>
                              En el resto de clases debemos hacer lo mismo (implementar las interfaces).
                            </p>
                            
                            <p>
                              Ahora todo nuestro código puede trabajar simplemente con objetos ITwitterInfo.
                            </p>
                            
                            <p>
                              Vale… esto está muy bien, pero como puedo cambiar dinámicamente el “formato” del archivo a leer?
                            </p>
                            
                            <p>
                              Una posible solución es realizar una clase “lectora” de XMLs, algo tal que así:
                            </p>
                            
                            <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
                              <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px" id="codeSnippet"><span style="color: #0000ff">class</span> LectorXmls<br />{<br />    <span style="color: #0000ff">public</span> ITwitterInfo LeerFormato&lt;TSer&gt;(<span style="color: #0000ff">string</span> file) <span style="color: #0000ff">where</span> TSer : <span style="color: #0000ff">class</span>, ITwitterInfo<br />    {<br />        TSer data = <span style="color: #0000ff">default</span>(TSer);<br />        <span style="color: #0000ff">using</span> (FileStream fs = <span style="color: #0000ff">new</span> FileStream(file, FileMode.Open))<br />        {<br />            XmlSerializer ser = <span style="color: #0000ff">new</span> XmlSerializer(<span style="color: #0000ff">typeof</span>(TSer));<br />            data = ser.Deserialize(fs) <span style="color: #0000ff">as</span> TSer;<br />        }<br />        <span style="color: #0000ff">return</span> data;<br />    }<br />}</pre>
                              
                              <p>
                                </div> 
                                
                                <p>
                                  Y luego en vuestro código podéis escoger que clase serializadora usar:
                                </p>
                                
                                <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
                                  <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px" id="codeSnippet">ITwitterInfo f1 = <span style="color: #0000ff">new</span> LectorXmls().LeerFormato&lt;Info&gt;(<span style="color: #006080">"Formato1.xml"</span>);</pre>
                                  
                                  <p>
                                    </div> 
                                    
                                    <p>
                                      Por supuesto, si quieres que el <em>tipo</em> de la clase serializadora esté en un archivo .config y así soportar “futuros formatos” de serialización no podrás usar un método genérico, pero en este caso te basta con pasar un Type como parámetro:
                                    </p>
                                    
                                    <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
                                      <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px" id="codeSnippet"><span style="color: #0000ff">public</span> ITwitterInfo LeerFormato(<span style="color: #0000ff">string</span> file, Type serType)<br />{<br />    ITwitterInfo data = <span style="color: #0000ff">null</span>;<br />    <span style="color: #0000ff">using</span> (FileStream fs = <span style="color: #0000ff">new</span> FileStream(file, FileMode.Open))<br />    {<br />        XmlSerializer ser = <span style="color: #0000ff">new</span> XmlSerializer(serType);<br />        data = ser.Deserialize(fs) <span style="color: #0000ff">as</span> ITwitterInfo;<br />    }<br />    <span style="color: #0000ff">return</span> data;<br />}</pre>
                                      
                                      <p>
                                        </div> 
                                        
                                        <p>
                                          Y el valor del Type lo puedes obtener a partir de un fichero de .config.
                                        </p>
                                        
                                        <p>
                                          Un saludo!!!
                                        </p>