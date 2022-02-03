---
title: 'ASP.NET MVC3 Beta: Mis impresiones'

author: eiximenis

date: 2010-10-07T14:19:00+00:00
geeks_url: /?p=1537
geeks_visits:
  - 1986
geeks_ms_views:
  - 1086
categories:
  - Uncategorized

---
Buenooo… ayer fue un día movidito en Microsoft: [anunciaron de golpe][1] la beta 2 de WebMatrix, la beta de MVC3 y un gestor de paquetes OSS para Visual Studio llamado NuPack. También he visto a través del [Web PI][2] que está la CTP2 de Compact SQL 4.

<!--more-->

MVC1 salió con 5 (creo) previews antes de la beta, con MVC2 juraria que hicieron un par o tres, y con MVC3 sólo un preview1 y luego ya la beta… a ese ritmo MVC4 cuando salga lo hará ya con el SP1 incorporado. 😛 Esos de Microsoft van cada vez más rápido.

Bueno, aquí van un poco mis impresiones sobre la Beta 3 de MVC 🙂

Me centraré básicamente en lo que ha cambiado desde la preview1 hasta la Beta.

## Creación de aplicaciones

Cuando le damos a nuevo proyecto en el Visual Studio, nos sale _una sola_ opción de MVC3:&#160; ASP.NET MVC 3 Web Application y una vez la seleccionamos es cuando nos deja elegir si queremos una aplicación vacía o la estándard (con los controladores Account y Home) y que View Engine queremos usar. Son las mismas opciones que teníamos en MVC2 (excepto lo del view engine) pero mejor organizadas.

Pero vamos… nada nuevo bajo el sol 😀

## Donde está IServiceLocator

En [mi post sobre la preview1][3] comentaba que el soporte para IoC se basaba en la interfaz [IServiceLocator][4], un proyecto de codeplex para proporcionar una interfaz común a distintos contenedores IoC y así permitir independizarnos de ellos. Decía también que la Preview1 contenía una implementación propia de IServiceLocator en lugar de usar la del assembly que está en codeplex, pero que se esperaba que en la versión final eso no fuera así. 

Pues bien… al final eso ha cambiado un poco: Ahora, a través de la clase _DependencyResolver_ podemos indicar a MVC3 que debe usar para resolver sus dependencias. Y tenemos tres posibilidades:

* Pasarle un objeto que implemente una interfaz nueva llamada _IDependencyResolver_ (propia de MVC3). Dicha interfaz está definida:

```csharp
  public interface IDependencyResolver
{
    object GetService(Type serviceType);
    IEnumerable<object> GetServices(Type serviceType);
}
```

* Pasarle dos delegates, el primero del cual se corresponde al método GetService y el segundo al método GetServices
* Pasarle un object. En este caso el object <em>debe</em> implementar la interfaz IServiceLocator. Pero (imagino que) ASP.NET MVC3 invocará los métodos via reflection, por lo que <em>cualquier</em> IServiceLocator vale, no es necesario usar el assembly de codeplex. Supongo que es para evitar tener una dependencia de MVC3 hacia un assembly “externo”.


Veamos un ejemplo:

P.ej. si usásemos Unity podríamos crear un `IDependencyResolver` propio:

```cshap
public class UnityDependencyResolver : IDependencyResolver
{
    private IUnityContainer _ctr;

    public UnityDependencyResolver(IUnityContainer ctr)
    {
        _ctr = ctr;
    }

    public object GetService(Type serviceType)
    {
        return _ctr.Resolve(serviceType);
    }

    public IEnumerable<object> GetServices(Type serviceType)
    {
        return _ctr.ResolveAll(serviceType);
    }
}
```   

Y registrarlo en el global.asax:

```csharp
UnityContainer ctr = new UnityContainer();
DependencyResolver.SetResolver(new UnityDependencyResolver(ctr));
```  
        
Guay! Pero también podríamos usar la versión que admite un object y pasarle una implementación de IServiceLocator. La versió 2.0 de Unity ya viene con el assembly del service locator incorporado y un adaptador creado (si usais versiones anteriores de Unity os podéis <a href="http://commonservicelocator.codeplex.com/">descargar tanto la implementación de IServiceLocator como el adaptador para Unity desde codeplex</a>).

En nuestro caso nos basta con agregar una referencia al ensamblado <em>Microsoft.Practices.ServiceLocation.dll</em> (que viene con Unity o os habéis descargado de codeplex) y usar:

```csharp
UnityContainer ctr = new UnityContainer();
DependencyResolver.SetResolver(new UnityServiceLocator(ctr));
```

(Si usáis una versión de Unity anterior a la 2.0 necesitaréis también una referencia a <em>Microsoft.Practices.Unity.ServiceLocatorAdapter.dll</em> que también os habréis descargado de codeplex).

Listos! Con esto MVC3 ya usará nuestro DependencyResolver para crear los controladores, así que ya podremos inyectarles dependencias (fijaos que no ha sido necesario crear una factoría propia como hacíamos en MVC2).

## IControllerActivator
                
IControllerActivator es una interfaz nueva de MVC3 cuya responsabilidad es devolver una instancia de un controlador de un determinado tipo. Antes esa responsabilidad era <em>una más</em> de las responsabilidades de la factoría de controladores: cuando en MVC2 queríamos usar DI generalmente implementábamos una factoría propia <em>derivada</em> de <em>DefaultControllerFactory </em>y refefiniamos el método GetControllerInstance:

P.ej. esa sería una factoría de controladores para usar IServiceLocator en MVC2:

```csharp
public class ServiceLocatorControllerFactory : DefaultControllerFactory
 {
     private readonly IServiceLocator _sl;
     public ServiceLocatorControllerFactory(IServiceLocator sl) 
     {
         _sl = sl;
     }
     protected override IController GetControllerInstance(RequestContext requestContext, Type controllerType)
     {
         IController controller = null;
         controller = _sl != null ?  _sl.GetInstance(controllerType) as IController : 
             base.GetControllerInstance(requestContext, controllerType);
         return controller;
     }
 }
 ```
                
En MVC3 han separado los dos comportamientos: Por un lado el decidir qué tipo de controlador instanciar se queda en la factoría de controladores y por otro lado instanciar el controlador es responsabilidad de `IControllerActivator`.

El uso de IControllerActivator nos permite definir como se crean nuestros controladores. Si no hay ningún `IControllerActivator` MVC3 usará el `DependencyResolver` (así que **no** es necesario implementar `IControllerActivator` sólo para DI). Si no hay `DependencyResolver` pues supongo que usará Reflection.

Como MVC3 busca las cosas en varios sitios podemos pasarle el `IControllerActivator` a la DefaultControllerFactory (tiene un parámetro que acepta un `IControllerActivator`) **o bien**, registrarlo en el `DependencyResolver`. O sea podemos usar esto:

```csharp
// Registramos un singleton con nuestro IControllerActivator
ctr.RegisterInstance<IControllerActivator>(new CustomControllerActivator());
DependencyResolver.SetResolver(new UnityServiceLocator(ctr));
// No registramos IControllerFactory por lo que se usará la DefaultControllerFactory
```

O bien esto:

```csharp
// Registramos la DefaultControllerFactory como la factoría a usar
// pasándole el IControllerActivator deseado
ctr.RegisterInstance<IControllerFactory>(new DefaultControllerFactory(new CustomControllerActivator()));
DependencyResolver.SetResolver(new UnityServiceLocator(ctr));
```

Pero **ambas** cosas a la vez no (os dará error).

## Nuevos Helpers

Bueno, ahora tenemos un conjunto de _Helpers_ nuevos (bueno, nuevos para MVC, porque ya estaban en WebMatrix). Para usar esos helpers basta con agregar una referencia a `System.Web.Helpers` (viene ya por defecto):

```html
<div>
    El Hash de eiximenis es: @Crypto.Hash("eiximenis")
</div>
```

Diria que **no** están disponibles todos los helpers que hay en WebMatrix… Espero que en 
la versión final sí estén todos, porque hay algunos que son una pasada!

## Ajax no “obtrusivo” (unobtrusive)

El helper Ajax de MVC3 puede generar código Ajax que **no** está mezclado con el código HTML, aprovechando jQuery y una de las nuevas capacidades de HTML5… P.ej. el siguiente código en una vista:

```html
@Ajax.ActionLink("Pulsa aquí para modif el DIV", "AjaxAction", new AjaxOptions() { UpdateTargetId="mydiv"})
<div id="mydiv">Div a modificar</div>
```

Nos genera el código HTML:

```html
<a data-ajax="true" data-ajax-mode="replace" data-ajax-update="#mydiv" href="/Home/AjaxAction">Pulsa aquí para modif el DIV</a>
<div id="mydiv">Div a modificar</div>
```

Fijaos que hay atributos `data-ajax-*` que son interpretados por jQuery para realizar las acciones: de esa manera no hay ninguna llamada a ningún script mezclado con el código. Los atributos que empiezan por “data-“ son considerados especiales en HTML5 (<a href="http://www.javascriptkit.com/dhtmltutors/customattributes.shtml">custom HTML5 attributes</a>): HTML5 no admite que pongamos nuestros propios atributos a no ser que empiecen por data-. Esto permite separar el comportamiento (definiendolo en estos atributos especiales) del código.

## Soporte para Razor

Pues… sigue bajo mínimos… Es decir, Razor está totalmente soportado en run-time (y ahora con sintaxis VB.NET también!) pero para VS2010 Razor sigue sin existir. Las vistas .cshtml (o .vbhtml) no tienen ni sintaxis coloreada ni tampoco intellisense. Pero es que ni tan siquiera las reconoce como HTML (o sea ni tenemos intellisense de HTML tampoco).

Al margen de esto hay algunas novedades menores para Razor como el soporte de @model para definir el modelo, pero vamos… nada excepcionalmente nuevo.

## Conclusiones

Las dos grandes novedades de esta Beta, respecto a la preview1, son algunas mejoras más en la inyección de dependencias (DependencyResolver, IControllerActivator) y el soporte para ajax no “obtrusivo”. El resto son mejoras menores (aunque interesantes).

Sigo pensando que, del mismo modo que se llama MVC3 se podría llamar MVC2.5, porque no hay cambios espectacularmente grandes (excepto Razor, pero es _simplemente_ otro ViewEngine, como muchos más que ya existían). Eso no es necesariamente malo: el framework ya está alcanzando cierto grado de madurez.

Para finalizar, leeros las <a href="http://www.asp.net/learn/whitepapers/mvc3-release-notes">release notes de la beta de MVC3</a> donde se enumeran las novedades!

Un saludo!

 [1]: http://weblogs.asp.net/scottgu/archive/2010/10/06/announcing-nupack-asp-net-mvc-3-beta-and-webmatrix-beta-2.aspx
 [2]: http://www.microsoft.com/web/downloads/platform.aspx
 [3]: http://geeks.ms/blogs/etomas/archive/2010/07/28/asp-net-mvc-3-preview-1.aspx
 [4]: http://commonservicelocator.codeplex.com/