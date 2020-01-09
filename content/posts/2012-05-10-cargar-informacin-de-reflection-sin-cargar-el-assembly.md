---
title: Cargar información de reflection sin cargar el assembly
author: eiximenis

date: 2012-05-10T17:27:46+00:00
geeks_url: /?p=1596
geeks_visits:
  - 1964
geeks_ms_views:
  - 1538
categories:
  - Uncategorized

---
Bueno… veamos un post rapidito. En un proyecto en el que he participado hemos estado personalizando Visual Studio a través de varios custom editors, plugins, packages y demás fauna que pulula por la selva de extensibilidad de Visual Studio.

Estos editores, addines y demás necesitaban acceder a información de Reflection de la propia DLL que se estaba compilando. Teóricamente obtener la información es muy sencillo. Basta con obtener la ruta a la DLL que se está compilando:

<div style="font-family: courier new; background: white; color: black; font-size: 10pt">
  <p style="margin: 0px">
    <span style="color: blue">private</span> <span style="color: blue">static</span> EnvDTE.<span style="color: #2b91af">DTE</span> DTE
  </p>
  
  <p style="margin: 0px">
    {
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: blue">get</span> { <span style="color: blue">return</span> (EnvDTE.<span style="color: #2b91af">DTE</span>)<span style="color: #2b91af">Package</span>.GetGlobalService(<span style="color: blue">typeof</span>(EnvDTE.<span style="color: #2b91af">DTE</span>)); }
  </p>
  
  <p style="margin: 0px">
    }
  </p>
  
  <p style="margin: 0px">
    &#160;
  </p>
  
  <p style="margin: 0px">
    <span style="color: blue">public</span> <span style="color: blue">static</span> <span style="color: blue">string</span> ObtenerRutaEnsamblado()
  </p>
  
  <p style="margin: 0px">
    {
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: blue">var</span> project = DTE.ActiveDocument.ProjectItem.ContainingProject;
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: blue">return</span> project.Properties.Item(<span style="color: #a31515">"LocalPath"</span>).Value.ToString() +
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; project.ConfigurationManager.ActiveConfiguration.Properties.Item(<span style="color: #a31515">"OutputPath"</span>).Value.ToString();
  </p>
  
  <p style="margin: 0px">
    }
  </p>
  
  <p style="margin: 0px">
    &#160;
  </p>
  
  <p style="margin: 0px">
    <span style="color: blue">public</span> <span style="color: blue">static</span> <span style="color: blue">string</span> ObtenerNombreEnsamblado()
  </p>
  
  <p style="margin: 0px">
    {
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: blue">var</span> project = DTE.ActiveDocument.ProjectItem.ContainingProject;
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: blue">return</span> <span style="color: blue">string</span>.Concat(ObtenerRutaEnsamblado(), project.Properties.Item(<span style="color: #a31515">"OutputFileName"</span>).Value.ToString());
  </p>
  
  <p style="margin: 0px">
    }
  </p></p>
</div>

El método ObtenerNombreEnsamblado da la ruta física de la DLL que se está compilando. A partir de aquí, debería bastar con usar LoadAssembly, para cargar la DLL y listos. Pero por supuesto, si esto fuese así, esta entrada del blog no existiría 🙂

El tema está en que cuando accedemos a un Assembly via Reflection, este assembly se carga en el CLR.&#160; Y una vez un Assembly está cargado no puede ni cargarse de nuevo (para ver las modificaciones, por ejemplo, recordad que estamos cargando la propia DLL que el usuario está creando en VS) ni tampoco descargarse. Además el archivo físico se puede crear bloqueado (lo que en nuestro caso impedía que pudieses compilar el proyecto, ya que estaba bloqueado por el addin). Si alguno de vosotros está pensando en cargar el proyecto [“solo para Reflection”][1], que se olvide. Cargar un assembly “solo para Reflection” lo carga igual y tampoco se puede ni cargar de nuevo ni descargar.

¿La solución? Bueno, pues utilizar un AppDomain nuevo. Para los que no lo sepáis los AppDomains son como “procesos” dentro del CLR. Un programa se ejecuta dentro de un AppDomain pero puede crear más AppDomains, del mismo modo que un proceso puede crear procesos hijos. Por supuesto la comunicación entre dos AppDomains se trata como comunicación interproceso: o a través de proxies (objetos MarshalByRef) o pasando objetos serializables. ¡Viva la vida!

Al final, terminé con una clase AppDomainUtils, con métodos estáticos parecidos a los siguientes:

<div style="font-family: courier new; background: white; color: black; font-size: 10pt">
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: gray">///</span><span style="color: green"> </span><span style="color: gray"><summary></span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: gray">///</span><span style="color: green"> Carga el tipo TObj en un AppDomain nuevo.</span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: gray">///</span><span style="color: green"> TObj DEBE ser MarshalByRef</span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: gray">///</span><span style="color: green"> </span><span style="color: gray"></summary></span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: blue">private</span> <span style="color: blue">static</span> TObj LoadFromType<TObj>(<span style="color: #2b91af">AppDomain</span> appDomain)
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; {
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: blue">var</span> tokens = <span style="color: blue">typeof</span>(TObj).AssemblyQualifiedName.Split(<span style="color: #a31515">&#8216;,&#8217;</span>);
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: blue">var</span> assName = tokens[1];
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: blue">var</span> typeName = tokens[0];
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: blue">var</span> obj = appDomain.CreateInstanceAndUnwrap(assName, typeName);
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: blue">return</span> (TObj)obj;
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; }
  </p>
  
  <p style="margin: 0px">
    &#160;
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: gray">///</span><span style="color: green"> </span><span style="color: gray"><summary></span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: gray">///</span><span style="color: green"> Obtiene información (de tipo TR) de un System.Type.</span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: gray">///</span><span style="color: green"> </span><span style="color: gray"></summary></span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: gray">///</span><span style="color: green"> </span><span style="color: gray"><typeparam name="TR"></span><span style="color: green">Tipo de información que se devuelve. Debe ser Serializable</span><span style="color: gray"></typeparam></span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: gray">///</span><span style="color: green"> </span><span style="color: gray"><typeparam name="TU"></span><span style="color: green">Tipo de la clase que extrae la información a partir del System.Type</span><span style="color: gray"></typeparam></span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: gray">///</span><span style="color: green"> </span><span style="color: gray"><param name="fullName"></span><span style="color: green">Nombre del System.Type a cargar (con assembly incorporado)</span><span style="color: gray"></param></span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: gray">///</span><span style="color: green"> </span><span style="color: gray"><param name="locationPath"></span><span style="color: green">Ruta fisica real del assembly</span><span style="color: gray"></param></span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: gray">///</span><span style="color: green"> </span><span style="color: gray"><returns></span
><span style="color: green">La información extraída del System.Type</span><span style="color: gray"></returns></span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: blue">public</span> <span style="color: blue">static</span> TR GetTypeInfo<TR, TU>(<span style="color: blue">string</span> fullName, <span style="color: blue">string</span> locationPath)
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: blue">where</span> TU : <span style="color: #2b91af">TypeLoader</span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; {
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: blue">var</span> appDomain = <span style="color: #2b91af">AppDomain</span>.CreateDomain(<span style="color: #2b91af">Guid</span>.NewGuid().ToString());
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: blue">var</span> tloader = LoadFromType<TU>(appDomain);
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: blue">var</span> result = tloader.LoadTypeInfo<TR>(fullName, locationPath);
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #2b91af">AppDomain</span>.Unload(appDomain);
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: blue">return</span> result;
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; }
  </p>
  
  <p style="margin: 0px">
    }
  </p></p>
</div>

La clase TypeLoader es como sigue:

<div style="font-family: courier new; background: white; color: black; font-size: 10pt">
  <p style="margin: 0px">
    <span style="color: gray">///</span><span style="color: green"> </span><span style="color: gray"><summary></span>
  </p>
  
  <p style="margin: 0px">
    <span style="color: gray">///</span><span style="color: green"> Carga información de un tipo.</span>
  </p>
  
  <p style="margin: 0px">
    <span style="color: gray">///</span><span style="color: green"> </span><span style="color: gray"></summary></span>
  </p>
  
  <p style="margin: 0px">
    <span style="color: blue">public</span> <span style="color: blue">class</span> <span style="color: #2b91af">TypeLoader</span> : <span style="color: #2b91af">MarshalByRefObject</span>
  </p>
  
  <p style="margin: 0px">
    {
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: gray">///</span><span style="color: green"> </span><span style="color: gray"><summary></span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: gray">///</span><span style="color: green"> Carga el tipo y extrae la información</span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: gray">///</span><span style="color: green"> </span><span style="color: gray"></summary></span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: blue">public</span> TR LoadTypeInfo<TR>(<span style="color: blue">string</span> fullName, <span style="color: blue">string</span> locationPath)
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; {
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: blue">var</span> type = <span style="color: #2b91af">Type</span>.GetType(fullName);
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: blue">if</span> (type == <span style="color: blue">null</span>)
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; {
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: blue">var</span> tokens = fullName.Split(<span style="color: #a31515">&#8216;,&#8217;</span>).Select(x => x.Trim()).ToArray();
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: blue">var</span> assFileName = tokens[1];
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: blue">var</span> assFileNameWithExtension = <span style="color: blue">string</span>.Concat(assFileName.Trim(), <span style="color: #a31515">".dll"</span>);
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: blue">var</span> assembly = <span style="color: #2b91af">AssemblyLoader</span>.CargarAssemblyDesdeByteArray(<span style="color: #2b91af">Path</span>.Combine(locationPath, assFileNameWithExtension));
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: blue">var</span> typeName = tokens[0];
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; type = assembly.GetTypes().FirstOrDefault(x => x.FullName == typeName);
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; }
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: blue">return</span> type != <span style="color: blue">null</span> ? (TR)Select(type) : <span style="color: blue">default</span>(TR);
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; }
  </p>
  
  <p style="margin: 0px">
    &#160;
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: gray">///</span><span style="color: green"> </span><span style="color: gray"><summary></span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: gray">///</span><span style="color: green"> Este método recibe un Type y debe devolver la info que se necesita de dicho Type.</span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: gray">///</span><span style="color: green"> Este objeto DEBE ser serializable y debe ser una instancia (o casteable) de TR</span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: gray">///</span><span style="color: green"> </span><span style="color: gray"></summary></span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: blue">protected</span> <span style="color: blue">virtual</span> <span style="color: blue">object</span> Select(<span style="color: #2b91af">Type</span> type) { <span style="color: blue">return</span> <span style="color: blue">null</span>; }
  </p></p>
</div>

La idea es cargar un System.Type, extraer información de él y devolverla. Evidentemente esto debe hacerse en un AppDomain nuevo. El método GetTypeInfo lo que hace es crear este AppDomain nuevo y luego, dentro de este AppDomain crear una instancia de un objeto propio, de un tipo cualquiera TU, pero que TU derive de TypeLoader. Y llama al método LoadTypeInfo de este objeto propio. El método LoadTypeInfo (definido en la clase TypeLoader) es el método que:

  1. Carga el assembly (usando un método propio que lo carga desde un array de bytes para asegurar que el fichero no se queda bloqueado. Simplemente lee todo el contenido del fichero en un byte[] y luego usa Assembly.Load pasándole este byte[]). 
  2. Obtiene el tipo (System.Type) especificado. 
  3. Llama al método Select que recibe un System.Type y debe devolver un objeto serializable con la información. Este objeto es el que se transmitirá al AppDomain principal (de ahí que deba ser serializable). Y no, System.Type no lo es. 

El uso al final es bastante sencillo:

<div style="font-family: courier new; background: white; color: black; font-size: 10pt">
  <p style="margin: 0px">
    <span style="color: blue">var</span> data = <span style="color: #2b91af">AppDomainUtils</span>.GetTypeInfo<<span style="color: #2b91af">TypeIdInfo</span>, <span style="color: #2b91af">TypeIdInfoLoader</span>>(tag.TypeName, <span style="color: #2b91af">OperativaReader</span>.ObtenerRutaEnsamblado());
  </p></p>
</div>

En la variable tag.TypeName está el nombre del tipo (full-qualified) a cargar. Al ejecutar esta línea en data tenemos un objeto de tipo TypeIdInfo que contiene la información que nos interesaba extraer del objeto System.Type. La clase TypeIdInfoLoader es la que transforma un System.Type en un TypeIdInfo:

<div style="font-family: courier new; background: white; color:
black; font-size: 10pt">
  </p> 
  
  <p style="margin: 0px">
    <span style="color: blue">class</span> <span style="color: #2b91af">TypeIdInfoLoader</span> : <span style="color: #2b91af">TypeLoader</span>
  </p>
  
  <p style="margin: 0px">
    {
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: blue">protected</span> <span style="color: blue">override</span> <span style="color: blue">object</span> Select(<span style="color: #2b91af">Type</span> type)
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; {&#160;&#160;&#160;&#160;&#160;&#160;&#160;
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: blue">var</span> data = <span style="color: blue">new</span> <span style="color: #2b91af">TypeIdInfo</span>() { FullName = type.FullName };
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: blue">return</span> data;
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; }
  </p>
  
  <p style="margin: 0px">
    }
  </p></p>
</div>

El código del méotdo Select de la clase TypeIdInfoLoader se ejecuta en el otro AppDomain, de ahí que deba devolver un objeto serializable (la clase TypeIdInfo debe estar marcada como tal).

En fin… comentar tan solo que todo este peñazo de usar AppDomains es porque los señores de Microsoft no han tenido a bien proporcionar una API que permite usar Reflection sin cargar la DLL dentro del CLR. Y no, lo siento, pero <a href="http://msdn.microsoft.com/en-us/library/fk4hw0yf.aspx" target="_blank" rel="noopener noreferrer">esta API no me sirve</a>. Quiero algo que para usarlo no deba morir mil veces.

Saludos! 😉

 [1]: http://msdn.microsoft.com/en-us/library/ms172331.aspx