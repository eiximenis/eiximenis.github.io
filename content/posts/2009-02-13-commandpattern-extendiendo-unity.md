---
title: CommandPattern extendiendo Unity

author: eiximenis

date: 2009-02-13T13:59:00+00:00
geeks_url: /?p=1437
geeks_visits:
  - 1594
geeks_ms_views:
  - 781
categories:
  - Uncategorized

---
Hola a todos! Hoy voy a hablar del poder que nos da el mecanismo de extensiones de Unity. Doy por supuesto que todos conoceis lo que es un contenedor IoC en general y Unity en particular. Si no, echad un vistazo a los posts "[IoC o el poder de ceder el control](https://geeks.ms/etomas/archive/2008/10/28/ioc-o-el-poder-de-ceder-el-control.aspx)" (para una explicación general de IoC) y "[Microsoft Unity: Inyección de dependencias .NET](https://geeks.ms/jdieguez/archive/2009/01/25/microsoft-unity-inyecci-243-n-de-dependencias-net.aspx)" (para una explicación general sobre Unity en concreto).

<!--more-->

Para ilustrar el poder que nos da extender Unity voy a poner una _posible_ implementación del patron [Command Pattern](https://en.wikipedia.org/wiki/Command_pattern). Este patrón es un clásico para la construcción de interfaces desacopladas, donde el elemento de la UI que genera una acción y el código que implementa esta acción no tienen porque estar relacionados. Esto aumenta la mantenibilidad y la reutilización del código.

Lo que voy a exponer aquí, es una implementación de dicho patrón, usando Unity y que funciona con cualquier &ldquo;tecnología&rdquo; (Winforms, WPF, cónsola). Dado que esto va a ser un poco largo, coged una buena cervecita que empezamos! 😉

También comentaros que el código que mostraré, aunque funcional, no está 100% completo, pero tiene las bases para que sea fácilmente completable. 

**1. Extensiones de Unity... que son?**

Las extensiones de Unity son el mecanismo que nos permite personalizar el comportamiento del contenedor cuando deba crear o destruir un objeto, o bien cuando se registre algún mapping entre tipos. En este post vamos a hablar de dos mecanismos para extender Unity:

  * Extensiones: Una extensión es _básicamente_ una clase que deriva de <span style="color: #2b91af">UnityContainerExtension</span>. Cuando se añade una extensión a Unity (cosa que puede hacerse programáticamente o por configuración), se llama al método Initialize() de la extensión. Las extensiones se usan para registrar en el contenedor nuevas Estrategias o Políticas y para suscribirnos a eventos de cuando se registra un mapping entre tipos.
  * Estrategias: Una estrategia es _algo_ que debe hacer Unity antes o después de crear o destruir un objeto. P.ej. si colocamos un atributo [Dependency] en cualquier propiedad pública de un objeto, Unity nos _rellenerá_ automáticamente dicha propiedad. Esto se hace a través de una estrategia (built-in dentro de Unity).

**2. Unity y ObjectBuilder2**

Unity en si mismo, en el fondo es _simplemente_, un wrapper sore ObjectBuilder2 (Una evolución del ObjectBuilder que venia con CAB y EntLib). Aunque por norma general podemos usar Unity sin preocuparnos del ObjectBuilder2 subyacente, cuando nos ponemos a extender el contenedor, entonces si que debemos interactuar con ObjectBuilder2. En concreto las _estrategias_ se definen siempre a nivel de ObjectBuilder2, mientras que las _extensiones_ son un mecanismo propio de Unity.

**3. Nuestro modelo de Command Pattern**

Para este ejemplo he pensado en un modelo de command pattern, extremadamente sencillo y declarativo (es decir, basado en atributos). Para ello vamos a usar dos atributos propios:

  * CommandSource: Para indicar que un determinado evento debe vincularse a un command.
  * CommandTarget: Para indicar que un método es la implementación de un command.

El primer atributo se define a nivel de clase (tantas veces como sea necesario), mientras que el segundo se define a nivel de método (una sola vez).

Un ejemplo de su uso:

```cs
[CommandSource("Command1", "button1", "Click")]
public partial class View1 : UserControl
{
    public View1()
    {
        InitializeComponent();
    }
}
```

Cuando se lance el evento &ldquo;Click&rdquo; del objeto &ldquo;button1&rdquo; se debe invocar el command &ldquo;Command1&rdquo;. En algún sitio habrá una clase (que no debe porque tener ninguna relación con View1) con el siguiente código para gestionar el command:

```cs
[CommandTarget("Command1")]
private void Foo()
{
    MessageBox.Show("Command1 Invocado");
}
```

**4. Qué vamos a hacer...**

Lo que queremos hacer es lo siguiente:

  * Cuando se registre un mapping en Unity, vamos a mirar la clase que se registra para inspeccionar si tiene atributos CommandSource y/o CommandTarget y vamos a guardar esta información en una clase (el CommandBroker).
  * Cuando se resuelva un tipo (es decir se cree una instancia) vamos a inspeccionar su tipo para ver si tiene atributos CommandSource y/o CommandTarget (sólo lo haremos si es necesario, que será si no se había definido un mapping para este objeto previamente), y luego nos suscribiremos a los eventos necesarios.

Así el CommandBroker es una clase que:

  * Tiene toda la información de que atributos CommandSource y/o CommandTarget tiene cada tipo creado por Unity.
  * Se suscribe a todos los eventos que vengan de un CommandSource y ejecuta el CommandTarget asociado.

El primer punto lo llevaremos a cabo mediante una extensión, y el segundo mediante una estrategia.

**5. La extensión: CommandExtension**

Vamos a definir nuestra extensión de Unity, para que haga tres cosas principales:

  * Cree una instancia del CommandBroker y la coloque como singleton dentro de Unity.
  * Cree la estrategia que necesitaremos y la &ldquo;instale&rdquo; en Unity.
  * Se registre al evento de creación de mapping para poder inspeccionar los tipos (evento Registering).

El código es realmente simple:

```cs
public class CommandExtension : UnityContainerExtension
{
    protected override void Initialize()
    {
        this.Container.RegisterInstance<ICommandBroker>(new CommandBroker());
        this.Context.Strategies.Add(new CommandStrategy(this.Container), 
            UnityBuildStage.Creation);
        this.Context.Registering += (o, e) =>
            this.Container.Resolve<ICommandBroker>().
            ProcessTypeInfo(e.TypeTo ?? e.TypeFrom);

        }
}
```

El método &ldquo;ProcessTypeInfo&rdquo; de la clase CommandBroker es la que inspecciona un tipo para ver si tiene atributos CommandSource y CommandTarget.

**6. La estrategia: CommandStrategy**

En la extensión creada previamente instalamos la estrategia CommandStrategy. Una estrategia se llama cada vez que Unity debe crear (BuildUp) o destruir (TearDown) un objeto.

En mi caso el código de la estrategia también es realmente simple:

```cs
class CommandStrategy : IBuilderStrategy
{
    private IUnityContainer container;
    public CommandStrategy(IUnityContainer container)
    {
        this.container = container;
    }
    public void PostBuildUp(IBuilderContext context)
    {
        ICommandBroker cb = this.container.Resolve<ICommandBroker>();
        cb.ProcessObjectInfo(context.Existing);
    }
    public void PostTearDown(IBuilderContext context) { }
    public void PreBuildUp(IBuilderContext context) { }
    public void PreTearDown(IBuilderContext context) { }
}
```

En el método PostBuildUp (después de que ObjectBuilder2 haya construido el objeto) se llama a ProcessObjectInfo del CommandBroker. Este método hace dos cosas básicas:

  * Se suscribe a todos los eventos declarados en los atributos CommandSource del tipo del objeto
  * Guarda delegados a todos los métodos decorados con un CommandTarget (guardando el nombre del command asociado a cada delegado).

Así el CommandBroker se suscribirá a _cualquier_ evento que se lance desde _cualquier_ objeto creado por Unity, que esté referenciado por un CommandSource. Y en la función gestora de dicho evento mirará en su tabla interna de delegados&nbsp; si hay alguno que pueda responder a dicho evento (basándose en el nombre del comando).

Si observais el constructor de la CommandStrategy, vereis que recibe una instancia del propio Unity. Esto es para poder obtener el CommandBroker, puesto que en la extensión lo registrábamos como singleton en Unity. Esto puede parecer sorprendente (que desde una estrategia de Unity no se tenga acceso al propio Unity)... es lo que decía antes que las estrategias se definen a nivel de ObjectBuilder2.

**7. El código...**

No comento más el resto del código, puesto que sólo conseguiria liar el post

Como he dicho el código NO está del todo completo, p.ej. si se registra un singleton con RegisterInstance, dicho singleton no participa del sistema de comandos, y luego en ningún caso el CommandBroker se _desuscribe_ a ningún evento. También para que fuese una implementación completa del patrón CommandPattern, se debería poder (de alguna manera) activar o desactivar comandos. Al desactivar un comando todos los elementos de la UI vinculados a él (o sea todos sus `[CommandSource]`) deberían _deshabilitarse_...

... pero bueno, todo es meterse! 😉

Espero que este post os haya dado una idea del potencial de extender Unity (y también del patron Command Pattern&#8221;).

Saludos!