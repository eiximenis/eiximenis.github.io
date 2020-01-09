---
title: Trabajando con AKS Devspaces
author: eiximenis

date: 2019-02-13T17:34:10+00:00
geeks_url: /?p=2282
geeks_ms_views:
  - 738
categories:
  - asp.net core
  - kubernetes

---
AKS Devspaces es una característica que Microsoft está promocionando bastante en sus charlas: casi en cualquier evento donde se hable de AKS se demuestra, ni que sea brevemente, Devspaces.
  
Pero... qué es y lo más importante: ¿Para qué sirve y como se usa?
  
<!--more-->


  
Devspaces es una tecnología diseñada **para facilitar las pruebas de desarrollo en entornos multi-contenedor**. Es decir, desarrollamos una solución compuesta de varios contenedores y queremos poder depurar o probar nuevas versiones de un contenedor sin afectar al resto.
  
Sin Devspaces hay varias opciones que podríamos usar:

  1. Ejecutar el &#8220;docker-compose.dcproj&#8221; desde Visual Studio: con esto obtenemos depuración multi-contenedor a costa de tener todo en una solución (sln) de Visual Studio. Esta opción **es la única que permite escenarios de depuración multi-contenedor**.
  2. Ejecutar todos los contenedores en local. Ya sea con _compose_ o si queremos un escenario más &#8220;realista&#8221; usar [Minikube][1] o el Kubernetes de Docker Desktop. El problema principal es que debo instalar y ejecutar todo el sistema en mi máquina de desarrollo. [**Es posible depurar un contenedor**, aunque hay que recurrir a técnicas un poco complejas][2].
  3. Tener un _namespace_ de Kubernetes para mi solo y en este _namespace_ tener instalado todo el sistema (o parte de él). Esta opción funciona, pero complica el proceso de pruebas: cada vez que desarrolles debes generar la imagen, subirla al registro y desplegarla en el clúster. Además, **la depuración es casi imposible**.

Devspaces intenta ofrecer una solución. **Actualmente (y desde hace bastante tiempo) está en _preview_. **Eso significa que en algunos escenarios la experiencia flaquea un poco, pero voy a intentar contaros qué podemos hacer a día de hoy.
  
Para probar _devspaces_ necesitais **un clúster de AKS creado en una región compatible con Devspaces (p. ej. east us) y con Http Application Routing habilitado**.
  
**Configurar el clúster para Devspaces**
  
Para poder usar devspaces lo primero es **configurar el clúster**. Esto es porque para que Devspaces funcione deben agregarse varios elementos al clúster. Para ello debemos teclear:

<pre class="EnlighterJSRAW" data-enlighter-language="null">az aks use-dev-spaces -g &lt;grupo-recursos&gt; -n &lt;nombre-aks&gt;</pre>

Esto (después de un rato) nos habilitará Devspaces en el clúster y además **si es la primera vez que lo usamos nos instalará la CLI de Devspaces (azds)**. Durante este proceso nos preguntará por un Devspace a usar de forma predeterminada. Vamos a decirle _default_.
  
A nivel de Kubernetes eso nos creará un _namespace_ nuevo llamado azds con toda la infraestructura propia de Devspaces.
  
[<img class="alignnone size-full wp-image-2284" src="https://geeks.ms/etomas/wp-content/uploads/sites/154/2019/02/aks-use-devspaces.png" alt="Ejecución del comando az aks use-dev-spaces" width="987" height="425" />][3]
  
**Preparando los proyectos para Devspaces**
  
Para poder desplegar un proyecto en Devspaces dicho proyecto debe estar preparado y para ello necesitamos básicamente tres cosas:

  * El fichero de configuración de Devspaces (azds.yaml)
  * Un Dockerfile especial, usado por Devspaces (Dockerfile.develop)
  * El chart de Helm para desplegar el proyecto

Esos tres elementos **son por proyecto** (en nuestro caso pues los tendremos tanto para la web como para la API). Por suerte para crearlos podemos usar la herramienta de línea de comandos &#8220;azds&#8221;. Solo debes situarte en cada uno de los csprojs y teclear &#8220;azds prep &#8211;public&#8221;
  
**Nota:** El parámetro &#8220;&#8211;public&#8221; es para indicar que queremos acceso desde el exterior (eso nos creará un recurso _ingress_ asociado).
  
Este comando **no instala nada en el clúster**, solo nos crea el Dockerfile.Develop, el chart de Helm y el fichero de configuración azds.yaml:
  
[<img class="alignnone size-full wp-image-2285" src="https://geeks.ms/etomas/wp-content/uploads/sites/154/2019/02/ficheros-azds.png" alt="Ficheros creados por azds" width="261" height="507" />][4]
  
Por supuesto esos ficheros son una plantilla que podemos retocar (y deberemos hacerlo especialmente el fichero Dockerfile.develop y el chart de Helm). Luego veremos porqué necesitamos este Dockerfile.develop.
  
**Instalando la versión &#8220;base&#8221;**
  
Vale, el escenario que vamos a cubrir va a ser el de que tenemos dos contenedores (una web y una api) y nosotros somos desarrolladores de la API. Queremos poder probar (y depurar) los cambios de la API, pero en un escenario end-to-end (es decir usando la web).
  
Para ello, lo primero es tener en un _devspace_ &#8220;la versión base&#8221;: eso es, la versión que va a ser compartida por todos los desarrolladores. En este _devspace_ va a estar desplegado **todo el proyecto** (en nuestro caso la web y la api).
  
Ahora que tenemos los proyectos preparados ya los podemos desplegar en el &#8220;devspaces&#8221; base (que es default). Para ello simplemente nos vamos a ambos proyectos y tecleamos:

<pre class="EnlighterJSRAW" data-enlighter-language="null">azds up -d</pre>

Eso hace muchas cosas:

  1. Sincroniza los ficheros locales con los del clúster
  2. Instala el chart de Helm para crear los objetos de Kubernetes
  3. Crea la imagen (usando Dockerfile.develop)
  4. Nos da una URL pública (si teníamos acceso desde el exterior)

[<img class="alignnone size-full wp-image-2286" src="https://geeks.ms/etomas/wp-content/uploads/sites/154/2019/02/azds-up.png" alt="Salida del comando azds up" width="979" height="452" />][5]
  
Una vez hemos ejecutado &#8220;azds up -d&#8221; en todos los proyectos, ya tenemos la versión base de la aplicación instalada en el _devspace_ default:
  
[<img class="alignnone size-full wp-image-2287" src="https://geeks.ms/etomas/wp-content/uploads/sites/154/2019/02/azds-default.png" alt="Objetos k8s (servicios, deployments, ingresses) creados" width="653" height="325" />][6]
  
También puedes usar el comando &#8220;azds list-up&#8221; que te dirá lo que se está ejecutando y que está controlado por devspaces:
  
[<img class="alignnone size-full wp-image-2290" src="https://geeks.ms/etomas/wp-content/uploads/sites/154/2019/02/azds-list-up.png" alt="Salida del comando azds list-up" width="524" height="205" />][7]
  
**Arreglando la API**
  
Eres un desarrollador nuevo y te han asignado un error en la API. Y es que parece que la web siempre muestra el mismo valor (42) recibido de la API:
  
[<img class="alignnone size-full wp-image-2288" src="https://geeks.ms/etomas/wp-content/uploads/sites/154/2019/02/web-error.png" alt="Error en la web (siempre muestra 42)" width="751" height="261" />][8]
  
Teóricamente este valor debería ser distinto cada vez. Vale, vale, vale... este es un escenario tan simple que no requeriría el uso de _devspaces_ (vamos, bastaría con depurar la API en local y depurarla con llamdas usando curl o similar), pero bueno...
  
Lo que vamos a ver es como:

  1. Como nuevo desarrollador puedes desplegar **solo la API** de una forma privada para tí
  2. Como puedes usar _tu versión_ de la API y a la vez ejectuar la versión &#8220;base&#8221; de la web

**Desplegar tu versión de la API**
  
Para ello lo que necesitas **es un devspace nuevo**: si en _default_ está la versión base, en el devspace nuevo que crees para tí, estará tu versión de la API.
  
Para ello, vete al directorio donde está la API y crea el nuevo _devspace_ mediante el comando:

<pre class="EnlighterJSRAW" data-enlighter-language="null">azds space select --name eiximenis</pre>

(Sustituye _eiximenis_ por el nombre que quieras, claro). Es **importante que cuando te pregunte por el Devspace padre selecciones &#8220;default&#8221;**. ¿Por qué? Porque de ese modo establecemos la relación de que para todos aquellos contenedores que no estén en _eiximenis_ se ejecute la versión de &#8220;default&#8221;.
  
A nivel de **Kubernetes** se ha creado un namespace nuevo (eiximenis) que actualmente está vacío. Pero no solo ha ocurrido eso: se han **creado nuevas entradas de DNS para los servicios que había en el devspace base:**
  
[<img class="alignnone size-full wp-image-2289" src="https://geeks.ms/etomas/wp-content/uploads/sites/154/2019/02/nuevos-ingress.png" alt="Nuevas entradas DNS (ingresses) creadas" width="842" height="145" />][9]
  
Observa como el nombre es <nombre-devspace>.s.<dns-base>.
  
Si ahora accedo a http://eiximenis.s.webclient.82gwcx5dsw.eus.azds.io/ veré igualmente la web, pero **ahora estoy accediendo a través de mi devspace nuevo (eiximenis).** Lo que pasa es que como **no hay nada en este devspace y su padre era default, se ejecuta todo lo que hay en default**.
  
**Probando la API**
  
No vamos a ver como **depurar el código** porque eso depende de la herramienta que uses (Visual Studio o Visual Studio Code), pero veamos como sería el flujo.
  
Recuerda que estamos situados en el directorio de la API**, así que ahora usamos azds up -d** para instalar la API en nuestro devspace nuevo.  En este momento tengo la API instalada en mi devspace (aunque es la misma versión que la &#8220;base&#8221; que está en _default_).
  
Ahora uso Code o cualquier editor y abro el código de la API. Como soy un desarrollador Ninja Master Senior, intuyo que el error estará en /Controllers/RandomController.cs:

<pre class="EnlighterJSRAW" data-enlighter-language="csharp">[HttpGet("value")]
public ActionResult&lt;int&gt; Get()
{
    return 42;
}</pre>

Ajá... he encontrado el error! Ahora simplemente lo corrijo:

<pre class="EnlighterJSRAW" data-enlighter-language="null">[HttpGet("value")]
public ActionResult&lt;int&gt; Get()
{
    return new Random().Next(1,101);
}</pre>

Una vez corregido vuelvo a lanzar &#8220;azds up -d&#8221; y eso me subirá la nueva versión a AKS. Ahora **ya puedo probar la web **pero desde mi _devspace. _**Y el resultado es... (redoble de tambores)... EL MISMO QUE ANTES**. La web sigue mostrando siempre el valor de 42!
  
No obstante **si pruebas la API con curl verás que el resultado es distinto cada vez.** ¿Qué está ocurriendo?
  
[<img class="alignnone size-large wp-image-2291" src="https://geeks.ms/etomas/wp-content/uploads/sites/154/2019/02/web-error2-1024x179.png" alt="Error de la web aunque la API va bien" width="660" height="115" />][10]
  
**El header de Devspaces**
  
Lo que está ocurriendo es lo siguiente:

  1. Entras a la web **a través de la URL del devspace eiximenis **(la que empieza por eiximenis.s).
  2. La web no está en dicho devspace, pero como es hijo de &#8220;default&#8221; se ejecuta la web de default
  3. La web de default llama a la API, y se ejecuta la API... de default, no del devspace eiximenis.

**Y es que nos hemos olvidado de una cosa**: de una cabecera HTTP. Pero para entenderlo retrocedamos un poco. Si observas como son los recursos ingress de devspaces verás lo siguiente:
  
[<img class="alignnone size-full wp-image-2292" src="https://geeks.ms/etomas/wp-content/uploads/sites/154/2019/02/ingress-adzs.png" alt="Ingress de azds con la anotación de ingress.class" width="525" height="244" />][11]
  
Observa que los ingress tienen la anotación &#8220;_kubernetes.io/ingress.class: traefik-azds_&#8220;. ¿Y que narices es traefik-adzs? Pues nada más ni nada menos que el controlador ingress de Devspaces. Si miras en el _namespace_ de azds verás el servicio _LoadBalancer_ asociado.
  
[Traefik][12] es un proxy inverso que puede ser configurado como [controlador ingress][13]. Lo que nos interesa a nosotros es **que cuando usemos un devspace xyz, traefik añadirá una cabecera &#8220;azds-route-as&#8221; con el valor del devspace**. Nosotros **debemos propagar esta cabecera en las llamadas HTTP que hagamos**.
  
¡Y ese es el problema! Como la web no propaga esta cabecera, cuando desde la web invocamos a la API, se ejecuta la API del mismo devspace que la web (en este caso _default_).
  
Vamos **ahora a modificar la web**. Para ello, lo primero es **volver a situarnos en el devspace &#8220;base&#8221;** (default) usando:

<pre class="EnlighterJSRAW" data-enlighter-language="null">azds space select --name default</pre>

Ahora modificamos la web para propagar la cabecera. Por suerte **dado que la web usaba IHttpClientFactory** (como estoy seguro hacen todos tus nuevos desarrollos, ¿verdad?) este cambio es muy sencillo y en un solo punto. Básicamente en Startup, modifico:

<pre class="EnlighterJSRAW" data-enlighter-language="csharp">services.AddHttpClient("api").ConfigureHttpClient(c =&gt;
{
    c.BaseAddress = new Uri(Configuration["ApiUrl"], UriKind.Absolute);
});</pre>

por:

<pre class="EnlighterJSRAW" data-enlighter-language="null">services.AddHttpContextAccessor();
services.AddTransient&lt;DevspacesMessageHandler&gt;();
services.AddHttpClient("api").ConfigureHttpClient(c =&gt;
{
    c.BaseAddress = new Uri(Configuration["ApiUrl"], UriKind.Absolute);
}).AddHttpMessageHandler&lt;DevspacesMessageHandler&gt;();</pre>

Vale, vale, eso no es todo... tengo que implementar la clase _DevpsacesMessageHandler_:

<pre class="EnlighterJSRAW" data-enlighter-language="csharp">public class DevspacesMessageHandler : DelegatingHandler
{
    private readonly IHttpContextAccessor _httpContextAccessor;
    public DevspacesMessageHandler(IHttpContextAccessor httpContextAccessor)
    {
        _httpContextAccessor = httpContextAccessor;
    }
    protected override Task&lt;HttpResponseMessage&gt; SendAsync(HttpRequestMessage request, CancellationToken cancellationToken)
    {
        var req = _httpContextAccessor.HttpContext.Request;
        if (req.Headers.ContainsKey("azds-route-as"))
        {
            request.Headers.Add("azds-route-as", req.Headers["azds-route-as"] as IEnumerable&lt;string&gt;);
        }
        return base.SendAsync(request, cancellationToken);
    }
}</pre>

Una vez hechos esos cambios, publico la nueva versión &#8220;base&#8221; de la web usando &#8220;azds up -d&#8221;.
  
Ahora **ya podemos volver a situarnos en el devspace eiximenis**:

<pre class="EnlighterJSRAW" data-enlighter-language="null">azds space select --name eiximenis</pre>

Ok, si ahora accedes a la web **usando la URL del devspace eiximenis** verás como **la web llama a la versión de la API que está en el devspace eiximenis**, pero que si accedes mediante la URL del devspace default (sin eiximenis.s delante) verás como se usa la versión &#8220;base&#8221; de la API (y recibimos siempre 42):
  
[<img class="alignnone size-full wp-image-2293" src="https://geeks.ms/etomas/wp-content/uploads/sites/154/2019/02/web-ok.png" alt="Web llamando a la versión de la API correcta" width="668" height="773" />][14]
  
**¡Y listos!** Con eso hemos visto como usar devspaces. Nos ha faltado por ver **como depurar**, pero eso depende de la herramienta y haría el post más largo de lo que ya es.
  
Así, que en resumen:

  1. Tenemos una versión &#8220;base&#8221; de todos los servicios
  2. En mi devspace tengo **solo** el/los servicios con los que estoy trabajando
  3. Debo usar siempre la URL asociada con mi devspace (la que empieza por <nombre-devspace>.s
  4. Debo acordarme de pasar la cabecera azds-route-as
  5. En todo momento, mediante azds, trabajo en un devspace. Con &#8220;azds space select &#8211;name <nombre>&#8221; selecciono con que devspace trabajo
  6. Con &#8220;azds up&#8221; instalo el servicio en el devspace en el que esté trabajando
  7. Con &#8220;azds down&#8221; elimino el servicio de dicho devspace

**Ay sí, que me olvido: el fichero Dockerfile.develop**
  
Antes he comentado que veríamos el por qué del fichero &#8220;Dockerfile.develop&#8221;. Bien, este es el Dockerfile que se usa para construir el contenedor que se despliega en devspaces. En devspaces, dado que es un entorno de desarrollo y depuración  (por más AKS que sea), no desplegamos los contenedores finales si no unos de desarrollo.
  
Esos contenedores básicamente, parten de la imagen del SDK y **contienen el código fuente y ejecutan un &#8220;dotnet run&#8221;** como punto de entrada. Cada vez que usamos &#8220;adzs up&#8221;, se copia el código fuente al contenedor de desarrollo y se ejecuta el &#8220;dotnet run&#8221;. Esto, el tener el código fuente en el contenedor, es lo que habilita los escenarios de depuración.
  
Ahora sí... como introducción no ha estado mal, ¿verdad?
  
**Os dejo un .zip con el ejemplo final**: [https://1drv.ms/u/s!Asa-selZwiFlg\_t\_BFZaTiiDbmLByQ][15]
  
¡Espero que os haya sido interesante!

 [1]: https://kubernetes.io/docs/setup/minikube/
 [2]: https://geeks.ms/etomas/2018/06/15/usar-vscode-para-depurar-tus-contenedores-netcore/
 [3]: https://geeks.ms/etomas/wp-content/uploads/sites/154/2019/02/aks-use-devspaces.png
 [4]: https://geeks.ms/etomas/wp-content/uploads/sites/154/2019/02/ficheros-azds.png
 [5]: https://geeks.ms/etomas/wp-content/uploads/sites/154/2019/02/azds-up.png
 [6]: https://geeks.ms/etomas/wp-content/uploads/sites/154/2019/02/azds-default.png
 [7]: https://geeks.ms/etomas/wp-content/uploads/sites/154/2019/02/azds-list-up.png
 [8]: https://geeks.ms/etomas/wp-content/uploads/sites/154/2019/02/web-error.png
 [9]: https://geeks.ms/etomas/wp-content/uploads/sites/154/2019/02/nuevos-ingress.png
 [10]: https://geeks.ms/etomas/wp-content/uploads/sites/154/2019/02/web-error2.png
 [11]: https://geeks.ms/etomas/wp-content/uploads/sites/154/2019/02/ingress-adzs.png
 [12]: https://traefik.io/
 [13]: https://docs.traefik.io/configuration/backends/kubernetes/
 [14]: https://geeks.ms/etomas/wp-content/uploads/sites/154/2019/02/web-ok.png
 [15]: https://1drv.ms/u/s!Asa-selZwiFlg_t_BFZaTiiDbmLByQ