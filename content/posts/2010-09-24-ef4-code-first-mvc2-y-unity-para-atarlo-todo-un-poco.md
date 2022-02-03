---
title: EF4 Code First, MVC2 y Unity para atarlo todo un poco…

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

<!--more-->

Resumiendo rápidamente EF Code First nos permite desarrollar nuestra capa de acceso a datos codificando _primero_ las clases, clases que son [POCO][4]. Eso nos permite conseguir lo que se conoce como _persistance ignorance_ (o que las clases que nos representan los datos sean agnósticas sobre cualquier tecnología de acceso a datos).

Que quede claro que Code First **no** es la única manera de usar objetos POCO en EF: Unai y Alberto hablan del tema [aquí][5] y [aquí][6]. Y dad por seguro que ellos conocen EF mucho más que yo 🙂

Lo que os quiero comentar es como se puede montar EF Code First en una aplicación MVC2 para poder usar el patrón Repository y Unit Of Work, de forma que los controladores no deban saber nada del mecanismo subyacente de acceso a datos (es decir, los controladores ignoran completamente que están usando EF).

## Rápida, rapidísima introducción a EF Code First

Usar EF Code First es muy sencillo. En el post de Scott Guthrie está todo explicado paso a paso, pero resumiendo podríamos decir que me puedo crear una clase tal que:

```csharp
public class Persona
{
   public int PersonaId {get; set;}
   public string Nombre {get; set;}
}
```
 
Esta clase nos permitiría acceder a datos almacenados en una tabla con los campos PersonaId y Nombre (EF Code First es capaz de crear la BBDD a partir del modelo de objetos). Para acceder a la bbdd creamos un objeto derivado de DbContext:

```csharp
public class MyDatabase : DbContext
{
   public DbSet<Persona> Personas {get; set;}
}
```
Y a partir de aquí instanciamos un objeto MyDatabase y lanzamos consultas linq contra la propiedad Personas… 🙂
## Patrón repositorio y Unit of work</strong>

Esos patrones están explicados por activa y por pasiva en muchos sitios. P.ej. aquí está el <a href="http://martinfowler.com/eaaCatalog/repository.html">patrón repositorio</a> y aquí el <a href="http://martinfowler.com/eaaCatalog/unitOfWork.html">Unit of Work</a> (UoW). Básicamente entendemos que el repositorio es una colección de objetos en memoria y que el patrón UoW sincroniza todos los cambios hechos en memoria hacia la base de datos.

Supongamos estos dos interfaces para definir ambos patrones:        

```csharp
public interface IRepository<TEntity> where TEntity : class
{
    IQueryable<TEntity> AsQueryable();    
    IEnumerable<TEntity> GetAll();
    IEnumerable<TEntity> Find(Expression<Func<TEntity, bool>> where);
}

public interface IUnitOfWork
{
    void Commit();
}
```            
              
Mi repositorio es de sólo lectura (no hay métodos Add ni Remove ni nada, pero todo es añadirlos) y el Unit of Work lo único que tiene es el método Commit() para sincronizar los cambios hechos en los repositorios y mandarlos todos a la base de datos.
                
EF Code First, implementa de <em>per se</em> el patrón Unit Of Work (a través de DbContext) y también el patrón repositorio a través de DbSet<>, pero recordad que yo quiero “agnostizarme” al respecto de EF, por eso creo las interfaces y ahora debo crear implementaciones de ellas <em>para EF</em>:

```csharp
public class Repository<TEntity> : IRepository<TEntity> where TEntity : class
{
    private IDbSet<TEntity> _dbSet;

    public Repository(IDbContext objectContext)
    {
        _dbSet = objectContext.Set<TEntity>();
    }
    public IQueryable<TEntity> AsQueryable()
    {
        return _dbSet;
    }
    public IEnumerable<TEntity> GetAll()
    {
        return _dbSet.ToList();
    }
    public IEnumerable<TEntity> Find(Expression<Func<TEntity, bool>> where)
    {
        return _dbSet.Where(where);
    }
}

public class UnitOfWork : IUnitOfWork, IDisposable
{
    private readonly IDbContext _objectContext;
    
    public UnitOfWork(IDbContext objectContext)
    {
        _objectContext = objectContext;
    }

    public void Dispose()
    {
        if (_objectContext != null)
        {
            _objectContext.Dispose();
        }
        GC.SuppressFinalize(this);
    }

    public void Commit()
    {
        _objectContext.SaveChanges();
    }
}
```         
La única “complicación” es en la implementación de <em>UnitOfWork</em>, que en el constructor uso un objeto de tipo <em>IDbContext</em>, pero esta interfaz <strong>no existe</strong> en EF: me la he inventado yo, para poder realizar DI luego cuando use Unity. La interfaz IDbContext es muy simple:

```csharp
public interface IDbContext : IDisposable
{
    IDbSet<T> Set<T>() where T : class;
    int SaveChanges();
}
```

Me permite obtener un <em>IDbSet</em> de entidades de tipo T y el método <em>SaveChanges</em> para persistir los cambios realizados en este contexto hacia la BBDD.
                            
## Uso de todo esto…

Para usar directamente un repositorio me basta con instanciarlo, pasándole como parámetro el IDbContext. Pero recordad que la clase DbContext de EF Code First <strong>no</strong> implementa IDbContext (que me lo he inventado yo), así que uso el patrón<em> <a href="http://en.wikipedia.org/wiki/Adapter_pattern">Adapter</a></em> para <em>traducir </em>los objetos DbContext a IDbContext:

```csharp
public class DbContextAdapter : IDbContext
{
    private readonly DbContext _context;
    public DbContextAdapter(DbContext context)
    {
        _context = context;
    }
    public void Dispose()
    {
        _context.Dispose();
    }
    public IDbSet<T> Set<T>() where T : class
    {
        return _context.Set<T>();
    }
    public int SaveChanges()
    {
        return _context.SaveChanges();
    }
}
```

Ahora sí que ya puedo crear un repositorio:

```csharp
IDbContext ctx = new DbContextAdapter(MyDatabase);  // MyDatabase deriva de DbContext
var rep = new Repository<Persona>(ctx);
var personas = rep.FindAll();
```

Para usar el Unit of Work vinculado a un contexto haríamos lo mismo.
## Añadiendo IoC a todo esto…</strong>
                                    
Ahora que ya vemos como podemos usar nuestro repositorio, vamos a ver como Unity nos ayuda a tener IoC y independizarnos realmente de EF.

Primero podríamos registrar las implementaciones concretas de `IRepository` y `IUnitOfWork`:
                                    
```csharp
// Container es el IUnityContainer
Container.RegisterType(typeof(IRepository<>), typeof(Repository<>));
Container.RegisterType<IUnitOfWork, UnitOfWork>();
```

Ahora debemos mapear IDbContext a DbContextAdapter (porque para instanciar tanto IRepository como IUnitOfWork les debemos pasar un IDbContext. Podríamos usar lo siguiente:
                                        
```csharp
Container.RegisterType<IDbContext, DbContextAdapter>();
```                                          
                                          
Este RegisterType está bien, pero el problema nos viene cuando vemos que el constructor de DbContextAdapter tiene un objeto `DbContext`. Eso significaría que Unity nos crearía un objeto de la clase `DbContext`y no de la clase derivada MyDatabase, para instanciar los objetos DbContextAdapter. **Nota:** Una solución sería que la clase DbContextAdapter tuviese en su constructor un objeto que no fuera DbContext sinó MyDatabase pero eso entonces limita la reutilización de todo lo que estamos haciendo!

Por suerte no estamos perdidos! Unity incorpora un mecanismo mediante el cual le podemos decir <em>como crear</em> un objeto de una determinada clase. <a href="http://geeks.ms/blogs/etomas/archive/2010/06/02/novedades-de-unity-2-0.aspx">En mi post sobre Unity 2.0</a>, hablé de las Injection Factories. La idea es decirle a Unity que cuando necesite un objeto de la clase indicada, en lugar de crearlo tal cual use una factoría nuestra.
                                            
Es decir podemos hacer una factoría para crear DbContext que en lugar de un DbContext devuelva un MyDatabase, y Unity usará dicha factoría cada vez que deba crear un DbContext:

```csharp
Container.RegisterType<DbContext>(new InjectionFactory(x => new MyDatabase()));
```
                                            
Listos! Con esto cuando Unity deba pasar a DbContextAdapter un objeto DbContext (como indica el constructor) creará realmente un objeto MyDatabase.

Ahora si que ya puedo hacer:

```csharp
var rep = Container.Resolve<IRepository<Persona>>();
```

## Y finalmente… MVC2</strong>

Para integrar esto dentro de MVC2 es muy sencillo: como siempre que queramos inyectar IoC en los controladores lo que debemos tocar es… la factoría de controladores:

```csharp
public class UnityControllerFactory : DefaultControllerFactory
{
    private readonly IUnityContainer _sl;

    public UnityControllerFactory(IUnityContainer sl) 
    {
        _sl = sl;
    }

    protected override IController GetControllerInstance(RequestContext requestContext, Type controllerType)
    {
        IController controller = _sl.Resolve(controllerType);
        return controller;
    }
}
```

Y como siempre establecerla en el `Global.asax`:

```csharp
UnityControllerFactory slcf = new UnityControllerFactory(container);
ControllerBuilder.Current.SetControllerFactory(slcf);
```

Y listos! Ahora si que hemos llegado al punto <strong>final</strong>. Por fin podemos declarar un controlador tal que:

```csharp
public class PersonasController : Controller
{
    private IRepository<Persona> _rep;
 
    public HandController(IRepository<Persona> rep)
    {
        _rep = rep;
    }
}
```

Y empezar a trabajar usando el repositorio! 🙂

Fijaos que en este punto (el controlador) no tenemos nada que nos ate a EF: el repositorio es una interfaz y las clases que usa el repositorio son objetos POCO.

## Y para terminar (pero NO lo menos importante)...
                                                              
Cuando trabajamos con aplicaciones web, es recomendable que los contextos de base de datos tengan una duración (lifetime) de request (se les llama objetos per-request): es decir si durante una misma petición se necesitan dos repostorios, estos dos repositorios deben de compartir del contexto de base de datos. Con lo que tenemos ahora <em>no</em> sucede esto, ya que cada vez que Unity deba crear un Repository<> creará su DbContextAdapter asociado que a su vez creará un DbContext (MyDatabase) <strong>nuevo</strong>. Ese comportamiento no es el desado.
                                                              
Por suerte Unity tiene un mecanismo muy bueno para poder establecer <em>cada cuando</em> debe el contenedor crear un objeto de un tipo determinado: <a href="http://geeks.ms/blogs/etomas/archive/2009/10/26/lifetime-managers-en-unity-o-191-como-s-233-que-eso-que-me-das-es-un-singleton.aspx">los lifetime managers</a>. Una solución es crearse un HttpRequestLifetimeManager, de forma que los objetos sólo persistirán durante la misma petición y usar este lifetime manager cuando hagamos el RegisterType de DbContext.

En <a title="http://unity.codeplex.com/Thread/View.aspx?ThreadId=38588" href="http://unity.codeplex.com/Thread/View.aspx?ThreadId=38588">http://unity.codeplex.com/Thread/View.aspx?ThreadId=38588</a> tenéis una implementación de un <strong>HttpRequestLifetimeManager</strong>, junto con una interesante aportación sobre el uso (en su lugar) de contenedores hijos que mueran a cada request, que tengan los objetos registrados como singletons y eliminar el contenedor hijo (con Dispose()) al final de cada request. Esto tiene la ventaja de que se llamaría a Dispose() de los objetos contenidos (en esta segunda aproximación es el contendor hijo el que tiene un ciclo de vida per-request).
                                                              
## Referencias

Los siguientes posts contienen más información al respecto:

* <a href="http://elegantcode.com/2009/12/15/entity-framework-ef4-generic-repository-and-unit-of-work-prototype/">Entity Framework POCO (EF4): Generic Repository and Unit of Work Prototype</a>. Este es el post que me ha servido de inspiración y de ayuda. Tiene una continuación (<a href="http://elegantcode.com/2009/12/15/building-a-unity-extension-for-entity-framework-poco-configuration-repository-and-unit-of-work/">Unity Extension for Entity Framework POCO Configuration, Repository and Unit of Work</a>) donde comenta el uso de Unity. Estos dos posts son <strong>de lectura imprescindible, </strong>aunque por un lado haya tenido que modificar algunas cosillas para que funcione con la CTP4 de EF Code first y por otro en el tercer post utilize una StaticFactoryExtension en lugar de una Injection Factory (supongo que usa Unity 1.2 que no soporta Injection Factories). Este tercer post utiliza una aproximación <strong>genial</strong> que es el uso de una extensión de Unity para configurar todos los mappings para los objetos de EF. Repito: lectura imprescindible.
* Los posts de Scott Guthrie sobre EF Code First:
  * <a title="http://weblogs.asp.net/scottgu/archive/2010/07/16/code-first-development-with-entity-framework-4.aspx" href="http://weblogs.asp.net/scottgu/archive/2010/07/16/code-first-development-with-entity-framework-4.aspx">http://weblogs.asp.net/scottgu/archive/2010/07/16/code-first-development-with-entity-framework-4.aspx</a>
  * <a title="http://weblogs.asp.net/scottgu/archive/2010/07/23/entity-framework-4-code-first-custom-database-schema-mapping.aspx" href="http://weblogs.asp.net/scottgu/archive/2010/07/23/entity-framework-4-code-first-custom-database-schema-mapping.aspx">http://weblogs.asp.net/scottgu/archive/2010/07/23/entity-framework-4-code-first-custom-database-schema-mapping.aspx</a>
  * <a title="http://weblogs.asp.net/scottgu/archive/2010/08/03/using-ef-code-first-with-an-existing-database.aspx" href="http://weblogs.asp.net/scottgu/archive/2010/08/03/using-ef-code-first-with-an-existing-database.aspx">http://weblogs.asp.net/scottgu/archive/2010/08/03/using-ef-code-first-with-an-existing-database.aspx</a>

 [1]: http://geeks.ms/blogs/unai/
 [2]: http://weblogs.asp.net/scottgu/
 [3]: http://weblogs.asp.net/scottgu/archive/2010/07/16/code-first-development-with-entity-framework-4.aspx
 [4]: http://en.wikipedia.org/wiki/Plain_Old_CLR_Object
 [5]: http://geeks.ms/blogs/unai/archive/2010/01/19/ef-4-0-poco-y-proxies-din-225-micos.aspx
 [6]: http://geeks.ms/blogs/adiazmartin/archive/2010/01/17/poco-en-entity-framework-4-0.aspx