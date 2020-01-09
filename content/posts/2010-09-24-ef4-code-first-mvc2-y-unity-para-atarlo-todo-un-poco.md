---
title: EF4 Code First, MVC2 y Unity para atarlo todo un poco…
description: EF4 Code First, MVC2 y Unity para atarlo todo un poco…
author: eiximenis

date: 2010-09-24T12:57:00+00:00
geeks_url: /?p=1535
geeks_visits:
  - 6018
geeks_ms_views:
  - 2360
categories:
  - Uncategorized

---
Buenas! No soy ni mucho menos un experto en EF (es más, me acabo de poner), como pueda serlo p.ej. [Unai][1], pero desde que [Scott Guthrie][2] publicó un [post sobre EF Code First][3] he empezado a mirar algunas cosillas.

Resumiendo rápidamente EF Code First nos permite desarrollar nuestra capa de acceso a datos codificando _primero_ las clases, clases que son [POCO][4]. Eso nos permite conseguir lo que se conoce como _persistance ignorance_ (o que las clases que nos representan los datos sean agnósticas sobre cualquier tecnología de acceso a datos).

Que quede claro que Code First **no** es la única manera de usar objetos POCO en EF: Unai y Alberto hablan del tema [aquí][5] y [aquí][6]. Y dad por seguro que ellos conocen EF mucho más que yo 🙂

Lo que os quiero comentar es como se puede montar EF Code First en una aplicación MVC2 para poder usar el patrón Repository y Unit Of Work, de forma que los controladores no deban saber nada del mecanismo subyacente de acceso a datos (es decir, los controladores ignoran completamente que están usando EF).

**1. Rápida, rapidísima introducción a EF Code First**

Usar EF Code First es muy sencillo. En el post de Scott Guthrie está todo explicado paso a paso, pero resumiendo podríamos decir que me puedo crear una clase tal que:

<div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: 'Courier New', courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
  <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; background-color: #f4f4f4; margin: 0em; border-left-style: none; padding-left: 0px; width: 100%; padding-right: 0px; font-family: 'Courier New', courier, monospace; direction: ltr; border-top-style: none; color: black; border-right-style: none; font-size: 8pt; overflow: visible; padding-top: 0px" id="codeSnippet"><span style="color: #0000ff">public</span> <span style="color: #0000ff">class</span> Persona<br />{<br />   <span style="color: #0000ff">public</span> <span style="color: #0000ff">int</span> PersonaId {get; set;}<br />   <span style="color: #0000ff">public</span> <span style="color: #0000ff">string</span> Nombre {get; set;}<br />}</pre>
  
  <p>
    </div> 
    
    <p>
      Esta clase nos permitiría acceder a datos almacenados en una tabla con los campos PersonaId y Nombre (EF Code First es capaz de crear la BBDD a partir del modelo de objetos). Para acceder a la bbdd creamos un objeto derivado de DbContext:
    </p>
    
    <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: 'Courier New', courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
      <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; background-color: #f4f4f4; margin: 0em; border-left-style: none; padding-left: 0px; width: 100%; padding-right: 0px; font-family: 'Courier New', courier, monospace; direction: ltr; border-top-style: none; color: black; border-right-style: none; font-size: 8pt; overflow: visible; padding-top: 0px" id="codeSnippet"><span style="color: #0000ff">public</span> <span style="color: #0000ff">class</span> MyDatabase : DbContext<br />{<br />   <span style="color: #0000ff">public</span> DbSet&lt;Persona&gt; Personas {get; set;}<br />}</pre>
      
      <p>
        </div> 
        
        <p>
          Y a partir de aquí instanciamos un objeto MyDatabase y lanzamos consultas linq contra la propiedad Personas… 🙂
        </p>
        
        <p>
          <strong>2. Patrón repositorio y Unit of work</strong>
        </p>
        
        <p>
          Esos patrones están explicados por activa y por pasiva en muchos sitios. P.ej. aquí está el <a href="http://martinfowler.com/eaaCatalog/repository.html">patrón repositorio</a> y aquí el <a href="http://martinfowler.com/eaaCatalog/unitOfWork.html">Unit of Work</a> (UoW). Básicamente entendemos que el repositorio es una colección de objetos en memoria y que el patrón UoW sincroniza todos los cambios hechos en memoria hacia la base de datos.
        </p>
        
        <p>
          Supongamos estos dos interfaces para definir ambos patrones:
        </p>
        
        <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: 'Courier New', courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
          <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; background-color: #f4f4f4; margin: 0em; border-left-style: none; padding-left: 0px; width: 100%; padding-right: 0px; font-family: 'Courier New', courier, monospace; direction: ltr; border-top-style: none; color: black; border-right-style: none; font-size: 8pt; overflow: visible; padding-top: 0px" id="codeSnippet"><span style="color: #0000ff">public</span> <span style="color: #0000ff">interface</span> IRepository&lt;TEntity&gt; <span style="color: #0000ff">where</span> TEntity : <span style="color: #0000ff">class</span><br />{<br />    IQueryable&lt;TEntity&gt; AsQueryable();    <br />    IEnumerable&lt;TEntity&gt; GetAll();<br />    IEnumerable&lt;TEntity&gt; Find(Expression&lt;Func&lt;TEntity, <span style="color: #0000ff">bool</span>&gt;&gt; <span style="color: #0000ff">where</span>);<br />}</pre>
          
          <p>
            </div> 
            
            <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: 'Courier New', courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
              <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; background-color: #f4f4f4; margin: 0em; border-left-style: none; padding-left: 0px; width: 100%; padding-right: 0px; font-family: 'Courier New', courier, monospace; direction: ltr; border-top-style: none; color: black; border-right-style: none; font-size: 8pt; overflow: visible; padding-top: 0px" id="codeSnippet"><span style="color: #0000ff">public</span> <span style="color: #0000ff">interface</span> IUnitOfWork<br />{<br />    <span style="color: #0000ff">void</span> Commit();<br />}</pre>
              
              <p>
                </div> 
                
                <p>
                  Mi repositorio es de sólo lectura (no hay métodos Add ni Remove ni nada, pero todo es añadirlos) y el Unit of Work lo único que tiene es el método Commit() para sincronizar los cambios hechos en los repositorios y mandarlos todos a la base de datos.
                </p>
                
                <p>
                  EF Code First, implementa de <em>per se</em> el patrón Unit Of Work (a través de DbContext) y también el patrón repositorio a través de DbSet<>, pero recordad que yo quiero “agnostizarme” al respecto de EF, por eso creo las interfaces y ahora debo crear implementaciones de ellas <em>para EF</em>:
                </p>
                
                <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: 'Courier New', courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
                  <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; background-color: #f4f4f4; margin: 0em; border-left-style: none; padding-left: 0px; width: 100%; padding-right: 0px; font-family: 'Courier New', courier, monospace; direction: ltr; border-top-style: none; color: black; border-right-style: none; font-size: 8pt; overflow: visible; padding-top: 0px" id="codeSnippet"><span style="color: #0000ff">public</span> <span style="color: #0000ff">class</span> Repository&lt;TEntity&gt; : IRepository&lt;TEntity&gt; <span style="color: #0000ff">where</span> TEntity : <span style="color: #0000ff">class</span><br />{<br />    <span style="color: #0000ff">private</span> IDbSet&lt;TEntity&gt; _dbSet;<br /><br />    <span style="color: #0000ff">public</span> Repository(IDbContext objectContext)<br />    {<br />        _dbSet = objectContext.Set&lt;TEntity&gt;();<br />    }<br />    <span style="color: #0000ff">public</span> IQueryable&lt;TEntity&gt; AsQueryable()<br />    {<br />        <span style="color: #0000ff">return</span> _dbSet;<br />    }<br />    <span style="color: #0000ff">public</span> IEnumerable&lt;TEntity&gt; GetAll()<br />    {<br />        <span style="color: #0000ff">return</span> _dbSet.ToList();<br />    }<br />    <span style="color: #0000ff">public</span> IEnumerable&lt;TEntity&gt; Find(Expression&lt;Func&lt;TEntity, <span style="color: #0000ff">bool</span>&gt;&gt; <span style="color: #0000ff">where</span>)<br />    {<br />        <span style="color: #0000ff">return</span> _dbSet.Where(<span style="color: #0000ff">where</span>);<br />    }<br />}</pre>
                  
                  <p>
                    </div> 
                    
                    <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: 'Courier New', courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
                      <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; background-color: #f4f4f4; margin: 0em; border-left-style: none; padding-left: 0px; width: 100%; padding-right: 0px; font-family: 'Courier New', courier, monospace; direction: ltr; border-top-style: none; color: black; border-right-style: none; font-size: 8pt; overflow: visible; padding-top: 0px" id="codeSnippet"><span style="color: #0000ff">public</span> <span style="color: #0000ff">class</span> UnitOfWork : IUnitOfWork, IDisposable<br />{<br />    <span style="color: #0000ff">private</span> <span style="color: #0000ff">readonly</span> IDbContext _objectContext;<br />    <br />    <span style="color: #0000ff">public</span> UnitOfWork(IDbContext objectContext)<br />    {<br />        _objectContext = objectContext;<br />    }<br /><br />    <span style="color: #0000ff">public</span> <span style="color: #0000ff">void</span> Dispose()<br />    {<br />        <span style="color: #0000ff">if</span> (_objectContext != <span style="color: #0000ff">null</span>)<br />        {<br />            _objectContext.Dispose();<br />        }<br />        GC.SuppressFinalize(<span style="color: #0000ff">this</span>);<br />    }<br /><br />    <span style="color: #0000ff">public</span> <span style="color: #0000ff">void</span> Commit()<br />    {<br />        _objectContext.SaveChanges();<br />    }<br />}</pre>
                      
                      <p>
                        </div> 
                        
                        <p>
                          La única “complicación” es en la implementación de <em>UnitOfWork</em>, que en el constructor uso un objeto de tipo <em>IDbContext</em>, pero esta interfaz <strong>no existe</strong> en EF: me la he inventado yo, para poder realizar DI luego cuando use Unity. La interfaz IDbContext es muy simple:
                        </p>
                        
                        <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: 'Courier New', courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
                          <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; background-color: #f4f4f4; margin: 0em; border-left-style: none; padding-left: 0px; width: 100%; padding-right: 0px; font-family: 'Courier New', courier, monospace; direction: ltr; border-top-style: none; color: black; border-right-style: none; font-size: 8pt; overflow: visible; padding-top: 0px" id="codeSnippet"><span style="color: #0000ff">public</span> <span style="color: #0000ff">interface</span> IDbContext : IDisposable<br />{<br />    IDbSet&lt;T&gt; Set&lt;T&gt;() <span style="color: #0000ff">where</span> T : <span style="color: #0000ff">class</span>;<br />    <span style="color: #0000ff">int</span> SaveChanges();<br />}</pre>
                          
                          <p>
                            </div> 
                            
                            <p>
                              Me permite obtener un <em>IDbSet</em> de entidades de tipo T y el método <em>SaveChanges</em> para persistir los cambios realizados en este contexto hacia la BBDD.
                            </p>
                            
                            <p>
                              <strong>3. Uso de todo esto…</strong>
                            </p>
                            
                            <p>
                              Para usar directamente un repositorio me basta con instanciarlo, pasándole como parámetro el IDbContext. Pero recordad que la clase DbContext de EF Code First <strong>no</strong> implementa IDbContext (que me lo he inventado yo), así que uso el patrón<em> <a href="http://en.wikipedia.org/wiki/Adapter_pattern">Adapter</a></em> para <em>traducir </em>los objetos DbContext a IDbContext:
                            </p>
                            
                            <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: 'Courier New', courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
                              <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; background-color: #f4f4f4; margin: 0em; border-left-style: none; padding-left: 0px; width: 100%; padding-right: 0px; font-family: 'Courier New', courier, monospace; direction: ltr; border-top-style: none; color: black; border-right-style: none; font-size: 8pt; overflow: visible; padding-top: 0px" id="codeSnippet"><span style="color: #0000ff">public</span> <span style="color: #0000ff">class</span> DbContextAdapter : IDbContext<br />{<br />    <span style="color: #0000ff">private</span> <span style="color: #0000ff">readonly</span> DbContext _context;<br />    <span style="color: #0000ff">public</span> DbContextAdapter(DbContext context)<br />    {<br />        _context = context;<br />    }<br />    <span style="color: #0000ff">public</span> <span style="color: #0000ff">void</span> Dispose()<br />    {<br />        _context.Dispose();<br />    }<br />    <span style="color: #0000ff">public</span> IDbSet&lt;T&gt; Set&lt;T&gt;() <span style="color: #0000ff">where</span> T : <span style="color: #0000ff">class</span><br />    {<br />        <span style="color: #0000ff">return</span> _context.Set&lt;T&gt;();<br />    }<br />    <span style="color: #0000ff">public</span> <span style="color: #0000ff">int</span> SaveChanges()<br />    {<br />        <span style="color: #0000ff">return</span> _context.SaveChanges();<br />    }<br />}</pre>
                              
                              <p>
                                </div> 
                                
                                <p>
                                  Ahora sí que ya puedo crear un repositorio:
                                </p>
                                
                                <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: 'Courier New', courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
                                  <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; background-color: #f4f4f4; margin: 0em; border-left-style: none; padding-left: 0px; width: 100%; padding-right: 0px; font-family: 'Courier New', courier, monospace; direction: ltr; border-top-style: none; color: black; border-right-style: none; font-size: 8pt; overflow: visible; padding-top: 0px" id="codeSnippet">IDbContext ctx = <span style="color: #0000ff">new</span> DbContextAdapter(MyDatabase);  <span style="color: #008000">// MyDatabase deriva de DbContext</span><br />var rep = <span style="color: #0000ff">new</span> Repository&lt;Persona&gt;(ctx);<br />var personas = rep.FindAll();</pre>
                                  
                                  <p>
                                    </div> 
                                    
                                    <p>
                                      Para usar el Unit of Work vinculado a un contexto haríamos lo mismo.
                                    </p>
                                    
                                    <p>
                                      <strong>4. Añadiendo IoC a todo esto…</strong>
                                    </p>
                                    
                                    <p>
                                      Ahora que ya vemos como podemos usar nuestro repositorio, vamos a ver como Unity nos ayuda a tener IoC y independizarnos realmente de EF.
                                    </p>
                                    
                                    <p>
                                      Primero podríamos registrar las implementaciones concretas de IRepository y IUnitOfWork:
                                    </p>
                                    
                                    <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: 'Courier New', courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
                                      <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; background-color: #f4f4f4; margin: 0em; border-left-style: none; padding-left: 0px; width: 100%; padding-right: 0px; font-family: 'Courier New', courier, monospace; direction: ltr; border-top-style: none; color: black; border-right-style: none; font-size: 8pt; overflow: visible; padding-top: 0px" id="codeSnippet"><span style="color: #008000">// Container es el IUnityContainer</span><br />Container.RegisterType(<span style="color: #0000ff">typeof</span>(IRepository&lt;&gt;), <span style="color: #0000ff">typeof</span>(Repository&lt;&gt;));<br />Container.RegisterType&lt;IUnitOfWork, UnitOfWork&gt;();</pre>
                                      
                                      <p>
                                        </div> 
                                        
                                        <p>
                                          Ahora debemos mapear IDbContext a DbContextAdapter (porque para instanciar tanto IRepository como IUnitOfWork les debemos pasar un IDbContext. Podríamos usar lo siguiente:
                                        </p>
                                        
                                        <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: 'Courier New', courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
                                          <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; background-color: #f4f4f4; margin: 0em; border-left-style: none; padding-left: 0px; width: 100%; padding-right: 0px; font-family: 'Courier New', courier, monospace; direction: ltr; border-top-style: none; color: black; border-right-style: none; font-size: 8pt; overflow: visible; padding-top: 0px" id="codeSnippet">Container.RegisterType&lt;IDbContext, DbContextAdapter&gt;();</pre>
                                          
                                          <p>
                                            </div> 
                                            
                                            <p>
                                              Este RegisterType está bien, pero el problema nos viene cuando vemos que el constructor de DbContextAdapter tiene un objeto <em>DbContext</em>. Eso significaría que Unity nos crearía un objeto de la clase <strong>DbContext</strong> y no de la clase derivada MyDatabase, para instanciar los objetos DbContextAdapter. <strong>Nota:</strong> Una solución sería que la clase DbContextAdapter tuviese en su constructor un objeto que no fuera DbContext sinó MyDatabase pero eso entonces limita la reutilización de todo lo que estamos haciendo!
                                            </p>
                                            
                                            <p>
                                              Por suerte no estamos perdidos! Unity incorpora un mecanismo mediante el cual le podemos decir <em>como crear</em> un objeto de una determinada clase. <a href="http://geeks.ms/blogs/etomas/archive/2010/06/02/novedades-de-unity-2-0.aspx">En mi post sobre Unity 2.0</a>, hablé de las Injection Factories. La idea es decirle a Unity que cuando necesite un objeto de la clase indicada, en lugar de crearlo tal cual use una factoría nuestra.
                                            </p>
                                            
                                            <p>
                                              Es decir podemos hacer una factoría para crear DbContext que en lugar de un DbContext devuelva un MyDatabase, y Unity usará dicha factoría cada vez que deba crear un DbContext:
                                            </p>
                                            
                                            <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: 'Courier New', courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
                                              <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; background-color: #f4f4f4; margin: 0em; border-left-style: none; padding-left: 0px; width: 100%; padding-right: 0px; font-family: 'Courier New', courier, monospace; direction: ltr; border-top-style: none; color: black; border-right-style: none; font-size: 8pt; overflow: visible; padding-top: 0px" id="codeSnippet">Container.RegisterType&lt;DbContext&gt;(<span style="color: #0000ff">new</span> InjectionFactory(x =&gt; <span style="color: #0000ff">new</span> MyDatabase()));</pre>
                                              
                                              <p>
                                                </div> 
                                                
                                                <p>
                                                  Listos! Con esto cuando Unity deba pasar a DbContextAdapter un objeto DbContext (como indica el constructor) creará realmente un objeto MyDatabase.
                                                </p>
                                                
                                                <p>
                                                  Ahora si que ya puedo hacer:
                                                </p>
                                                
                                                <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: 'Courier New', courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
                                                  <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; background-color: #f4f4f4; margin: 0em; border-left-style: none; padding-left: 0px; width: 100%; padding-right: 0px; font-family: 'Courier New', courier, monospace; direction: ltr; border-top-style: none; color: black; border-right-style: none; font-size: 8pt; overflow: visible; padding-top: 0px" id="codeSnippet">var rep = Container.Resolve&lt;IRepository&lt;Persona&gt;&gt;();</pre>
                                                  
                                                  <p>
                                                    </div>
                                                  </p>
                                                  
                                                  <p>
                                                    <strong>5. Y finalmente… MVC2</strong>
                                                  </p>
                                                  
                                                  <p>
                                                    Para integrar esto dentro de MVC2 es muy sencillo: como siempre que queramos inyectar IoC en los controladores lo que debemos tocar es… la factoría de controladores:
                                                  </p>
                                                  
                                                  <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: 'Courier New', courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
                                                    <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; background-color: #f4f4f4; margin: 0em; border-left-style: none; padding-left: 0px; width: 100%; padding-right: 0px; font-family: 'Courier New', courier, monospace; direction: ltr; border-top-style: none; color: black; border-right-style: none; font-size: 8pt; overflow: visible; padding-top: 0px" id="codeSnippet"><span style="color: #0000ff">public</span> <span style="color: #0000ff">class</span> UnityControllerFactory : DefaultControllerFactory<br />{<br />    <span style="color: #0000ff">private</span> <span style="color: #0000ff">readonly</span> IUnityContainer _sl;<br /><br />    <span style="color: #0000ff">public</span> UnityControllerFactory(IUnityContainer sl) <br />    {<br />        _sl = sl;<br />    }<br /><br />    <span style="color: #0000ff">protected</span> <span style="color: #0000ff">override</span> IController GetControllerInstance(RequestContext requestContext, Type controllerType)<br />    {<br />        IController controller = _sl.Resolve(controllerType);<br />        <span style="color: #0000ff">return</span> controller;<br />    }<br />}</pre>
                                                    
                                                    <p>
                                                      </div> 
                                                      
                                                      <p>
                                                        Y como siempre establecerla en el Global.asax:
                                                      </p>
                                                      
                                                      <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: 'Courier New', courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
                                                        <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; background-color: #f4f4f4; margin: 0em; border-left-style: none; padding-left: 0px; width: 100%; padding-right: 0px; font-family: 'Courier New', courier, monospace; direction: ltr; border-top-style: none; color: black; border-right-style: none; font-size: 8pt; overflow: visible; padding-top: 0px" id="codeSnippet">UnityControllerFactory slcf = <span style="color: #0000ff">new</span> UnityControllerFactory(container);<br />ControllerBuilder.Current.SetControllerFactory(slcf);</pre>
                                                        
                                                        <p>
                                                          </div> 
                                                          
                                                          <p>
                                                            Y listos! Ahora si que hemos llegado al punto <strong>final</strong>. Por fin podemos declarar un controlador tal que:
                                                          </p>
                                                          
                                                          <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: 'Courier New', courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
                                                            <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; background-color: #f4f4f4; margin: 0em; border-left-style: none; padding-left: 0px; width: 100%; padding-right: 0px; font-family: 'Courier New', courier, monospace; direction: ltr; border-top-style: none; color: black; border-right-style: none; font-size: 8pt; overflow: visible; padding-top: 0px" id="codeSnippet"><span style="color: #0000ff">public</span> <span style="color: #0000ff">class</span> PersonasController : Controller<br />{<br />    <span style="color: #0000ff">private</span> IRepository&lt;Persona&gt; _rep;<br /> <br />    <span style="color: #0000ff">public</span> HandController(IRepository&lt;Persona&gt; rep)<br />    {<br />        _rep = rep;<br />    }<br />}</pre>
                                                            
                                                            <p>
                                                              </div> 
                                                              
                                                              <p>
                                                                Y empezar a trabajar usando el repositorio! 🙂
                                                              </p>
                                                              
                                                              <p>
                                                                Fijaos que en este punto (el controlador) no tenemos nada que nos ate a EF: el repositorio es una interfaz y las clases que usa el repositorio son objetos POCO.
                                                              </p>
                                                              
                                                              <p>
                                                                <strong>6. Y para terminar (pero NO lo menos importante)…</strong>
                                                              </p>
                                                              
                                                              <p>
                                                                Cuando trabajamos con aplicaciones web, es recomendable que los contextos de base de datos tengan una duración (lifetime) de request (se les llama objetos per-request): es decir si durante una misma petición se necesitan dos repostorios, estos dos repositorios deben de compartir del contexto de base de datos. Con lo que tenemos ahora <em>no</em> sucede esto, ya que cada vez que Unity deba crear un Repository<> creará su DbContextAdapter asociado que a su vez creará un DbContext (MyDatabase) <strong>nuevo</strong>. Ese comportamiento no es el desado.
                                                              </p>
                                                              
                                                              <p>
                                                                Por suerte Unity tiene un mecanismo muy bueno para poder establecer <em>cada cuando</em> debe el contenedor crear un objeto de un tipo determinado: <a href="http://geeks.ms/blogs/etomas/archive/2009/10/26/lifetime-managers-en-unity-o-191-como-s-233-que-eso-que-me-das-es-un-singleton.aspx">los lifetime managers</a>. Una solución es crearse un HttpRequestLifetimeManager, de forma que los objetos sólo persistirán durante la misma petición y usar este lifetime manager cuando hagamos el RegisterType de DbContext.
                                                              </p>
                                                              
                                                              <p>
                                                                En <a title="http://unity.codeplex.com/Thread/View.aspx?ThreadId=38588" href="http://unity.codeplex.com/Thread/View.aspx?ThreadId=38588">http://unity.codeplex.com/Thread/View.aspx?ThreadId=38588</a> tenéis una implementación de un <strong>HttpRequestLifetimeManager</strong>, junto con una interesante aportación sobre el uso (en su lugar) de contenedores hijos que mueran a cada request, que tengan los objetos registrados como singletons y eliminar el contenedor hijo (con Dispose()) al final de cada request. Esto tiene la ventaja de que se llamaría a Dispose() de los objetos contenidos (en esta segunda aproximación es el contendor hijo el que tiene un ciclo de vida per-request).
                                                              </p>
                                                              
                                                              <p>
                                                                <strong>7. Referencias</strong>
                                                              </p>
                                                              
                                                              <p>
                                                                Los siguientes posts contienen más información al respecto:
                                                              </p>
                                                              
                                                              <ol>
                                                                <li>
                                                                  <a href="http://elegantcode.com/2009/12/15/entity-framework-ef4-generic-repository-and-unit-of-work-prototype/">Entity Framework POCO (EF4): Generic Repository and Unit of Work Prototype</a>. Este es el post que me ha servido de inspiración y de ayuda. Tiene una continuación (<a href="http://elegantcode.com/2009/12/15/building-a-unity-extension-for-entity-framework-poco-configuration-repository-and-unit-of-work/">Unity Extension for Entity Framework POCO Configuration, Repository and Unit of Work</a>) donde comenta el uso de Unity. Estos dos posts son <strong>de lectura imprescindible, </strong>aunque por un lado haya tenido que modificar algunas cosillas para que funcione con la CTP4 de EF Code first y por otro en el tercer post utilize una StaticFactoryExtension en lugar de una Injection Factory (supongo que usa Unity 1.2 que no soporta Injection Factories). Este tercer post utiliza una aproximación <strong>genial</strong> que es el uso de una extensión de Unity para configurar todos los mappings para los objetos de EF. Repito: lectura imprescindible.
                                                                </li>
                                                                <li>
                                                                  Los posts de Scott Guthrie sobre EF Code First:
                                                                </li>
                                                                <ol>
                                                                  <li>
                                                                    <a title="http://weblogs.asp.net/scottgu/archive/2010/07/16/code-first-development-with-entity-framework-4.aspx" href="http://weblogs.asp.net/scottgu/archive/2010/07/16/code-first-development-with-entity-framework-4.aspx">http://weblogs.asp.net/scottgu/archive/2010/07/16/code-first-development-with-entity-framework-4.aspx</a>
                                                                  </li>
                                                                  <li>
                                                                    <a title="http://weblogs.asp.net/scottgu/archive/2010/07/23/entity-framework-4-code-first-custom-database-schema-mapping.aspx" href="http://weblogs.asp.net/scottgu/archive/2010/07/23/entity-framework-4-code-first-custom-database-schema-mapping.aspx">http://weblogs.asp.net/scottgu/archive/2010/07/23/entity-framework-4-code-first-custom-database-schema-mapping.aspx</a>
                                                                  </li>
                                                                  <li>
                                                                    <a title="http://weblogs.asp.net/scottgu/archive/2010/08/03/using-ef-code-first-with-an-existing-database.aspx" href="http://weblogs.asp.net/scottgu/archive/2010/08/03/using-ef-code-first-with-an-existing-database.aspx">http://weblogs.asp.net/scottgu/archive/2010/08/03/using-ef-code-first-with-an-existing-database.aspx</a>
                                                                  </li>
                                                                </ol>
                                                              </ol>
                                                              
                                                              <p>
                                                                Un saludo!!! 🙂
                                                              </p>

 [1]: http://geeks.ms/blogs/unai/
 [2]: http://weblogs.asp.net/scottgu/
 [3]: http://weblogs.asp.net/scottgu/archive/2010/07/16/code-first-development-with-entity-framework-4.aspx
 [4]: http://en.wikipedia.org/wiki/Plain_Old_CLR_Object
 [5]: http://geeks.ms/blogs/unai/archive/2010/01/19/ef-4-0-poco-y-proxies-din-225-micos.aspx
 [6]: http://geeks.ms/blogs/adiazmartin/archive/2010/01/17/poco-en-entity-framework-4-0.aspx