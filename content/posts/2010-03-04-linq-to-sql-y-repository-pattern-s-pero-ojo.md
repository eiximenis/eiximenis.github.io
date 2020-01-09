---
title: Linq To SQL y Repository Pattern… sí, pero ojo!
description: Linq To SQL y Repository Pattern… sí, pero ojo!
author: eiximenis

date: 2010-03-04T10:13:00+00:00
geeks_url: /?p=1499
geeks_visits:
  - 3348
geeks_ms_views:
  - 714
categories:
  - Uncategorized

---
Hola a todos! Hoy, por temas que no vienen al caso, estaba mirando el <a target="_blank" href="http://asp.net/mvc" rel="noopener noreferrer">tutorial de MVC que hay en asp.net</a>. Hay dos apartados dedicados a explicar como se pueden realizar modelos usando Linq to Sql y EF. Hasta ahí, ningún problema.

El problema viene, cuando en el <a target="_blank" href="http://www.asp.net/learn/mvc/tutorial-10-cs.aspx" rel="noopener noreferrer">apartado dedicado a Linq to Sql</a>, una vez han dado un ejemplo de uso de las clases de Linq to Sql desde un controlador, dicen que esta solución, aunque correcta, implica que si en un futuro cambiamos el proveedor de acceso a datos vamos a tener que tocar todos nuestros controladores, ya que usamos las clases Linq to Sql desde ellos. También se menciona que el uso del patrón <a target="_blank" href="http://martinfowler.com/eaaCatalog/repository.html" rel="noopener noreferrer">Repositorio</a> (_repository pattern_) nos permite aislarnos de Linq to Sql de modo que si más adelante migramos, digamos a EF, no tengamos que modificar nuestros controladores.

Cuando usamos Linq to Sql se nos generan por un lado una clase que hereda de _DataContext_ y que representa _nuestra base de datos_, y por otro un conjunto de clases que representan a nuestras tablas (nuestros datos). Esas clases no son <a target="_blank" href="http://es.wikipedia.org/wiki/Plain_Old_CLR_Object" rel="noopener noreferrer">POCO</a> y están generadas y controladas por Linq to Sql.

Esta es el ejemplo que proporcionan en asp.net sobre el uso del patrón repositorio:

<div id="codeSnippetWrapper" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; width: 97.5%; font-family: 'Courier New', courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; cursor: text; border: silver 1px solid; padding: 4px;">
  <pre id="codeSnippet" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 0em; width: 100%; font-family: 'Courier New', courier, monospace; direction: ltr; color: black; font-size: 8pt; overflow: visible; border-style: none; padding: 0px;"><span style="color: #0000ff">namespace</span> MvcApplication1.Models<br />{<br />         <span style="color: #0000ff">public</span> <span style="color: #0000ff">class</span> MovieRepository : IMovieRepository<br />         {<br />              <span style="color: #0000ff">private</span> MovieDataContext _dataContext;<br /><br />              <span style="color: #0000ff">public</span> MovieRepository()<br />              {<br />                    _dataContext = <span style="color: #0000ff">new</span> MovieDataContext();<br />              }<br />              <span style="color: #0000ff">public</span> IList&lt;Movie&gt; ListAll()<br />              {<br />                   var movies = from m <span style="color: #0000ff">in</span> _dataContext.Movies<br />                        select m;<br />                   <span style="color: #0000ff">return</span> movies.ToList();<br />              }<br />         }<br />}</pre>
  
  <p>
    </div> 
    
    <p>
      El repositorio <em>MovieRepository </em>es quien nos da acceso a los datos contenidos en la base de datos y nos independiza de la clase Linq to Sql <em>MovieDataContext</em>. &iexcl;Bien!
    </p>
    
    <p>
      El ejemplo de uso que proporcionan desde un controlador es el siguiente:
    </p>
    
    <div id="codeSnippetWrapper" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; width: 97.5%; font-family: 'Courier New', courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; cursor: text; border: silver 1px solid; padding: 4px;">
      <pre id="codeSnippet" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 0em; width: 100%; font-family: 'Courier New', courier, monospace; direction: ltr; color: black; font-size: 8pt; overflow: visible; border-style: none; padding: 0px;"><span style="color: #0000ff">public</span> <span style="color: #0000ff">class</span> MoviesController : Controller<br />{<br />     <span style="color: #0000ff">private</span> IMovieRepository _repository;<br />     <span style="color: #0000ff">public</span> MoviesController(IMovieRepository repository)<br />     {<br />          _repository = repository;<br />     }<br />     <span style="color: #0000ff">public</span> ActionResult Index()<br />     {<br />          <span style="color: #0000ff">return</span> View(_repository.ListAll());<br />     }<br />}<br /></pre>
      
      <p>
        </div> 
        
        <p>
          Aprovechan además para comentarnos que el parámetro <em>repository</em> del constructor podría esta inyectado por un contenedor IoC (con lo que estoy totalmente de acuerdo). Luego nos enfatizan de que la dependencia de este controlador es solamente con <em>IMovieRepository</em>, con lo que por un lado podemos pasar un Mock de IMovieRepository cuando usamos tests unitarios y por otro si algún dia migramos a EF, nos basta con crear un <em>EFMovieRepository</em> y... listos.
        </p>
        
        <p>
          Pues no.
        </p>
        
        <p>
          Veis el problema? No? Y si cambio el código del método Index() por el siguiente código <strong>equivalente</strong>?
        </p>
        
        <div id="codeSnippetWrapper" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; width: 97.5%; font-family: 'Courier New', courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; cursor: text; border: silver 1px solid; padding: 4px;">
          <pre id="codeSnippet" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 0em; width: 100%; font-family: 'Courier New', courier, monospace; direction: ltr; color: black; font-size: 8pt; overflow: visible; border-style: none; padding: 0px;"> <span style="color: #0000ff">public</span> ActionResult Index()<br /> {<br />      IList&lt;Movie&gt; movies = _repository.ListAll();<br />      <span style="color: #0000ff">return</span> View(movies);<br /> }</pre>
          
          <p>
            </div> 
            
            <p>
              Detectais ahora el problema? El controlador MoviesController no depende de MovieDataContext, de acuerdo, <strong>pero sigue dependiendo</strong> de las clases generadas por Linq-to-Sql (en este caso de la clase <em>Movie</em>). Con el ejemplo que hay en asp.net tal y como está, de poco nos sirve implementar el patrón repositorio: cuando migremos a otro proveedor de datos (pongamos EF) vamos a tener que tocar igualmente todos los controladores.
            </p>
            
            <p>
              Hay solución para el problema? Como (casi) todo en esta vida tiene solución, pues si que la hay: podemos forzar a Linq to SQL a que utilice clases POCO aunque perdemos algunas de sus capacidades (y por lo que he visto tampoco es que sea trivial). Existe una buena explicación al respecto en <a href="http://blogs.msdn.com/digital_ruminations/archive/2007/08/28/linq-to-sql-poco-support.aspx" title="http://blogs.msdn.com/digital_ruminations/archive/2007/08/28/linq-to-sql-poco-support.aspx">http://blogs.msdn.com/digital_ruminations/archive/2007/08/28/linq-to-sql-poco-support.aspx</a>.
            </p>
            
            <p>
              Si buscais <em>&ldquo;Repository Pattern&rdquo; &ldquo;Linq to Sql&rdquo;</em> en google encontrareis interesantes discusiones y posibles implementaciones al respecto, pero el objetivo de mi post no era tanto ofrecer soluciones al problema, si no hacer notar el pequeño problema en el ejemplo de asp.net y que, como siempre, no debemos <em>creernos</em> directamente todo lo que vemos.
            </p>
            
            <p>
              Un saludo!
            </p>