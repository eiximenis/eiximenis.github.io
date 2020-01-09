---
title: IoC o el poder de ceder el control
author: eiximenis

date: 2008-10-28T11:09:00+00:00
geeks_url: /?p=1425
geeks_visits:
  - 8728
geeks_ms_views:
  - 4976
categories:
  - patrones

---
Hablando con colegas de profesión, me he dado cuenta de que muchos de ellos no terminan de comprender el patrón IoC o las ventajas que su uso comporta… Así que sin ánimo de sentar cátedra he decidido escribir este post, por si le sirve a alguien… <span style="font-family: Wingdings;">J</span> 

IoC, que corresponde a las siglas de Inversion Of Control, agrupa a varios patrones que tienen en común que el flujo de ejecución del programa se invierte respecto a los métodos de programación tradicionales (<a href="http://es.wikipedia.org/wiki/Inversi%C3%B3n\_de\_Control" mce\_href="http://es.wikipedia.org/wiki/Inversi%C3%B3n\_de_Control">wikipedia dixit</a>). Generalmente el programador especifica las acciones (métodos) que se van llamando, pero en IoC lo que se especifican son las respuestas a determinados eventos o sucesos, dejando en manos de un elemento externo todas las acciones de control necesarias cuando lleguen estos sucesos. 

Un claro ejemplo de IoC se encuentra en el modelo de eventos de Windows Forms. Cuando vinculamos una función gestora a un evento (p.ej. el Click de un Button), estamos definiendo la respuesta a un determinado evento y nos despreocupamos cuando se llamará a nuestra función gestora. Es decir, cedemos el control del flujo al framework para que sea él que llame cuando sea necesario a nuestro método. 

Una de las principales ventajas de usar patrones IoC es que reducimos el acople entre una clase y las clases de las cuales depende, y eso cuando queremos utilizar por ejemplo tests unitarios es muy importante (no seré yo quien hable ahora de las ventajas de TDD cuando ya lo ha hecho Rodrigo (<a href="/blogs/rcorral/archive/2006/12/02/beneficios-y-carater-iacute-sticas-de-un-buen-test-unitario.aspx" mce_href="/blogs/rcorral/archive/2006/12/02/beneficios-y-carater-iacute-sticas-de-un-buen-test-unitario.aspx">http://geeks.ms/blogs/rcorral/archive/2006/12/02/beneficios-y-carater-iacute-sticas-de-un-buen-test-unitario.aspx</a>)). De los distintos patrones IoC me interesa comentar específicamente dos: Service Locator y cómo podemos usarlo en .NET. Para más adelante dejo Dependency Injection, otra forma de IoC que también es extremadamente útil. 

Id a la nevera, coged una <a href="http://www.volldamm.es/" mce_href="http://www.volldamm.es/">Voll-Damm</a> o cualquier otra cervecilla y sentaos que este post es un poco largo… <span style="font-family: Wingdings;">J</span> 

## Service Locator

El objetivo de Service Locator es reducir las dependencias que tenga una clase. Imaginemos una clase, que depende de otras dos clases (p.ej. ServicioSeguridad y ServicioLogin). Podríamos tener un código tal como: 

```cs
class Client
  {
  static void Main(string[] args)
  {
    new Client().Run();
  }
  private void Run()
  {
    // En este punto necesitamos objetos de las clases ServicioSeguridad y ServicioLogger
    ServicioSeguridad ss = new ServicioSeguridad();
    ServicioLogger sl = new ServicioLogger()
    // ...
  }
}
```

En este punto tenemos un fuerte acople entre la clase Client y las clases ServicioSeguridad y ServicioLoggger. Seguiríamos teniendo este mismo acople incluso aunque utilizáramos interfaces porque deberíamos hacer el &#8220;new&#8221;: 

```cs
IServicioSeguridad ss = new ServicioSeguridad();
IServicioLogger sl = new ServicioLogger();
```

Las principales desventajas de esta situación son: 

  1. Las clases que implementan las dependencias (en nuestro caso ServicioSeguridad y ServicioLogger) deben estar disponibles en tiempo de compilación (no nos basta con tener solo las interfaces). 
  2. El fuerte acople de la clase con sus dependencias dificulta de sobremanera su testing. Si queremos utilizar Mocks para de ServicioSeguridad o ServicioLogger vamos a tener dificultades 

El patrón de ServiceLocator soluciona estos dos puntos, sustituyendo las dependencias de la clase Client por dependencias a los interfaces y a un elemento externo, que llamaremos _Contenedor_ encargado de devolver las referencias que se le piden. 

Cuando la clase necesita un objeto en concreto, lo pide al contenedor: 

```cs
IServicioSeguridad ss = container.Resolve<IServicioSeguridad>(“servicioseguridad”);
IServicioLogger sl = container.Resolve<IServicioLogger>(“serviciologger”);
```

El método Resolve devolvería una referencia del tipo especificado en el parámetro genérico de acuerdo con un identificador. Evidentemente falta alguien que cree el contenedor y que agregue los servicios a él. Es decir, en algún sitio habrá algo como: 

```cs
container = new IoCContainer();
container.Add<IServicioSeguridad, ServicioSeguridad>(new ServicioSeguridad(), “servicioseguridad”);
container.Add<IServicioLogger, ServicioLogger>(new ServicioLogger(), “serviciologger”);
```

La gran diferencia es que esto no tiene porque estar en la clase _Client_. Simplemente pasándole a la clase Client una referencia al contenedor, eliminamos todas las dependencias de la clase Client con las clases que implementan los servicios. Y donde creamos el contendor? Pues depende… si estamos en nuestra aplicación, lo podemos crear en el método que inicialice la aplicación, pero si queremos probar la clase Client con tests unitarios podemos crear el contenedor en la inicialización del test…. Y lo que es mejor: rellenarlo con Mocks de los servicios! 

Así, nuestro programa podría tener una clase `Bootstrapper` que crea el contenedor de IoC y lo inicializa con los objetos necesarios: 

```cs
class Bootstrapper
{
  static void Main(string[] args)
  {
    IIoCContainer container = new IoCContainer();
    container.Add<IServicioSeguridad, ServicioSeguridad>(new ServicioSeguridad(), “servicioseguridad”);
    container.Add<IServicioLogger, ServicioLogger>(new ServicioLogger(), “serviciologger”);
    new Client(container).Run();
  }
}
```

Pero si queremos usar tests unitarios de la clase `Client`, podemos cambiar los objetos por Mocks fácilmente: 

```cs
[ClassInitialize]
public static void Init(TestContext context)
{
  container = new IoCContainer();
  container.Add<IServicioSeguridad, ServicioSeguridadMock>(new
  ServicioSeguridadMock(), “servicioseguridad”);
  container.Add<IServicioLogger, ServicioLoggerMock>(new
  ServicioLoggerMock(), “serviciologger”);
}

[TestMethod]
public void TestMethod1()
{
  Client c = new Client(container);
  c.Run(); // Este método Run usará los Mocks!
}
```

Fijaos que podemos lanzar tests unitarios sobre la clase Client, sin necesidad alguna de cambiar su código y utilizando Mocks. Además, la clase Client no tiene ninguna dependencia con las implementaciones de los servicios que utiliza, así que no es necesario ni que existan para poder crear la clase Client (sólo necesitamos las interfaces). 

## Unity

Aunque existen varios contenedores IoC open source para .NET, Microsoft tiene el suyo, también open source, llamado Unity (<a href="http://www.codeplex.com/unity/" mce_href="http://www.codeplex.com/unity/">http://www.codeplex.com/unity/</a>). Unity proporciona soporte para los patrones Service Locator y Dependency Injection. 

Para que veais un poco su uso he dejado un proyecto de herramienta de línea de comandos para listar los ficheros de un directorio (vamos un dir, jejejee…. <span style="font-family: Wingdings;">J</span>). 

La solución está dividida en varios proyectos: 

  1. DemoIoC: Contiene el Bootstrapper, la clase que inicializa el contenedor de Unity, agrega la clase llamada ServicioFicheros y utiliza un objeto Cliente para realizar las acciones de la línea de comandos. 
  2. Cliente: Contiene la implementación de la clase Cliente. 
  3. Interfaces: Contiene las interfaces del servicio (en este caso sólo una) 
  4. Implementations: Contiene las implementaciones del servicio (en este caso sólo una) 
  5. Mocks: Contiene las implementaciones de los Mocks (en este caso sólo una) 
  6. UnitTests: Contiene tests sobre la clase Cliente, usando un Mock del servicio ServicioFicheros. 

El ejecutable (proyecto DemoIoC, assembly DemoIoC.exe) es un archivo de línea de comandos que acepta dos parámetros: 

* `DemoIoC –l` para listar todos los ficheros (del directorio actual) 
* `DemoIoC –e:fichero.ext` Indica si el fichero `fichero.ext` existe (en el directorio actual). 

Si lo ejecutáis veréis que NO funciona bien: lista los ficheros correctamente, pero devuelve que un fichero existe cuando no es cierto y viceversa. Si miráis el código de la clase Cliente, encontrareis el error, ya que es obvio, pero lo bueno es que el proyecto UnitTest, testea este método de la clase Cliente, usando un Mock del ServicioFicheros y el UnitTest detecta el error. Observad lo fácil que ha sido sustituir el ServicioFicheros por un Mock sin alterar la clase Cliente, gracias al uso del patrón Service Locator! 

PD: Estooooo... que me he dejado de adjuntar el código de ejemplo... Lo teneis <a href="//geeks.ms/blogs/etomas/IoC/DemoIoC.zip" mce_href="/blogs/etomas/IoC/DemoIoC.zip">aquí</a>!!! 

Saludos!!! 

## Referencias

Os dejo algunas referencias para su lectura por si os interesa profundizar un poco en el tema tratado: 

  * Unity Container (<a href="http://www.codeplex.com/unity/" mce_href="http://www.codeplex.com/unity/">http://www.codeplex.com/unity/</a>) 
  * Patrón Service Locator (<a href="http://msdn.microsoft.com/en-us/library/cc707905.aspx" mce_href="http://msdn.microsoft.com/en-us/library/cc707905.aspx">http://msdn.microsoft.com/en-us/library/cc707905.aspx</a>) 
  * Inversion of Control (<a href="http://msdn.microsoft.com/en-us/magazine/cc947917.aspx" mce_href="http://msdn.microsoft.com/en-us/magazine/cc947917.aspx">http://msdn.microsoft.com/en-us/magazine/cc947917.aspx</a>) 
  * Mocks (<a href="http://martinfowler.com/articles/mocksArentStubs.html" mce_href="http://martinfowler.com/articles/mocksArentStubs.html">http://martinfowler.com/articles/mocksArentStubs.html</a>)