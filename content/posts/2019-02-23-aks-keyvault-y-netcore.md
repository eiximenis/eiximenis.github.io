---
title: AKS, KeyVault y netcore
author: eiximenis

date: 2019-02-23T16:16:31+00:00
geeks_url: /?p=2315
geeks_ms_views:
  - 677
categories:
  - asp.net core
  - kubernetes

---
¡Buenas! Vamos a ver en este artículo **como podemos leer secretos almacenados en un [Azure Key Vault][1]** desde nuestro código netcore ejecutándose en un AKS.
  
A diferencia de ACR donde contamos con una integración nativa en la cual nos basta con usar un _service account _de AKS vinculado a un _service principal _de Azure que tenga permisos de lectura contra el ACR (escribiéndolo veo que esto da para un pequeño futuro post), para Key Vault no tenemos integración nativa.
  
<!--more-->


  
Por supuesto siempre puedo usar la API de Key Vault y acceder al Key Vault por código. Así, si estoy usando .Net Core puedo usar el [proveedor de configuración para Azure Key Vault][2] y santas pascuas. Para otros lenguajes pues me tocará pelearme con la [API de Key Vault][3] o buscar a alguien que lo haya hecho.
  
No es lo que veremos en este post. Aquí quiero mostrar **como usar una librería que nos permite mapear los secretos de un Key Vault a un volúmen de nuestros _pods_.** De este modo, acceder a Key Vault se traduce en &#8220;leer de un directorio&#8221;. No puede ser más sencillo.
  
[Esta librería][4] usa una característica de Kubernetes llamada _FlexVolume_. Básicamente FlexVolume es un mecanismo que nos permite crear adaptadores de volúmenes para distintas fuentes de información. No quiero entrar en demasiados detalles sobre como funciona FlexVolume (daría para otro post), para lo que nos ocupa basta saber eso, que nos permite, de forma sencilla, crear adaptadores para crear volúmenes para determinadas fuentes de información.
  
**Usando el FlexVolume para Azure Key Vault**
  
Puedes instalar la librería ya sea **como add-on de AKS Engine **o bien &#8220;a mano&#8221;. Como nunca he hablado de AKS Engine en este blog (todo llegará), vamos a ver la manera de instalarlo a mano que además no puede ser más sencillo:

<pre class="EnlighterJSRAW" data-enlighter-language="null">kubectl create -f https://raw.githubusercontent.com/Azure/kubernetes-keyvault-flexvol/master/deployment/kv-flexvol-installer.yaml</pre>

Esto te creará un _namespace_ nuevo (kv) que contiene un _daemonset_ (_keyvault-flexvolume_) que ejecuta un pod por cada nodo:
  
[<img class="alignnone size-full wp-image-2316" src="https://geeks.ms/etomas/wp-content/uploads/sites/154/2019/02/flexvol-keyvault-ds.png" alt="kubectl mostrando el daemonset y los pods" width="887" height="183" />][5]
  
Para configurar keyvault-flexvol y darle acceso al Key Vault, podemos usar dos mecanismos: un _service principal_ o bien _pod identity_. No hemos hablado de [_pod identity_,][6] así que voy a usar un service principal. Cuando en un futuro post introduzca pod identity, ya veremos el correspondiente ejemplo (ufff, cuantos futuros posts están saliendo, ¿no? xD).
  
Usar un service principal implica básicamente pasarle a Kubernetes (usando un secreto) las credenciales de un service principal que tenga acceso al Key Vault. Así que lo primero es tener a mano un service principal. Si no lo tienes puedes crear uno usando:

<pre class="EnlighterJSRAW" data-enlighter-language="null">az ad sp create-for-rbac</pre>

Y te devolverá un JSON con el &#8220;appId&#8221; y el &#8220;Password&#8221; (login y password).
  
Ahora debes crear el secreto en el clúster:

<pre class="EnlighterJSRAW" data-enlighter-language="null">kubectl create secret generic keyvaultreader --from-literal clientid=&lt;appId&gt; --from-literal clientsecret=&lt;password&gt; --type=azure/kv</pre>

Esto crea el secreto llamado _keyvaultreader_. Si haces &#8220;_kubectl get secret_&#8221; te debería salir un secreto llamado _keyvaultreader_ de tipo _azure/kv_.
  
Vale, el siguiente paso claro, es crear un keyvault, ya que si no poca cosa vamos a probar. Con el siguiente código creas el keyvault y le damos permisos de lectura al _service principal:_

<pre class="EnlighterJSRAW" data-enlighter-language="null">az keyvault create -g &lt;grupo-recursos&gt; -n &lt;nombre-keyvault&gt;
az role assignment create --role Reader --assignee &lt;service-principal-appid&gt; --scope /subscriptions/&lt;id-suscripcion&gt;/resourcegroups/&lt;grupo-recursos&gt;/providers/Microsoft.KeyVault/vaults/&lt;nombre-keyvault&gt;
az keyvault set-policy -n &lt;nombre-keyvault&gt; --key-permissions get --spn &lt;service-principal-appid&gt;
az keyvault set-policy -n &lt;nombre-keyvault&gt; --secret-permissions get --spn &lt;service-principal-appid&gt;
az keyvault set-policy -n &lt;nombre-keyvault&gt; --certificate-permissions get --spn &lt;service-principal-appid&gt;
</pre>

Vamos a crear dos secretos (secret1 y secret2) para poder hacer la prueba:

<pre class="EnlighterJSRAW" data-enlighter-language="null">az keyvault secret set --vault-name &lt;nombre-keyvault&gt; -n secret1 --value "42 is the answer"
az keyvault secret set --vault-name &lt;nombre-keyvault&gt; -n secret2 --value "But the question remains unknown"</pre>

¡Perfecto! Ya tenemos las credenciales del _service principal _en el clúster (_keuvaultreader)_, y el _service principal_ tiene permisos de lectura al Key Vault donde tenemos dos secretos.
  
**Configurar el volúmen en el deployment**
  
Para montar el volúmen _flexvolume_ y así tener acceso a los secretos del KeyVault debemos definir el volumen. Nuestra definición del deployment queda así:

<pre class="EnlighterJSRAW" data-enlighter-language="json">apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: test-kv
spec:
  template:
    metadata:
      labels:
        app: test-kv
        component: test-kv
    spec:
      containers:
      - name: test-kv
        image: testedu.azurecr.io/demokv
        imagePullPolicy: Always
        ports:
        - containerPort: 80
        volumeMounts:
        - name: my-kv
          mountPath: /kv-data
          readOnly: true
      imagePullSecrets:
      - name: acrsecret
      volumes:
      - name: my-kv
        flexVolume:
          driver: "azure/kv"
          secretRef:
            name: keyvaultreader
          options:
            keyvaultname: &lt;nombre-keyvault&gt;
            keyvaultobjectnames: secret1;secret2
            keyvaultobjecttypes: secret;secret
            resourcegroup: &lt;grupo-recursos-keyvault&gt;
            subscriptionid: &lt;id-subscripción-azure&gt;
            tenantid: &lt;id-tenant-keyvault&gt;</pre>

En el volumen debemos pasarle:

  1. El nombre, grupo de recursos, id suscripción y id del tenant del KeyVault
  2. El nombre de los secretos a recuperar separados por punto y coma (en este caso _secret1 _y _secret2_).
  3. El tipo de los recursos (si son certificados o secretos)

Luego usando _volumeMounts_ montamos este volumen en el path _/kv-data_ del pod.
  
**No es necesario nada más**. Ahora si creamos este deployment y ponemos en marcha el pod, veremos **que en el directorio /kv-data aparece un fichero por cada secreto del Key Vault**. El nombre del fichero es el nombre del secreto y su contenido es el secreto en sí:
  
[<img class="alignnone size-full wp-image-2318" src="https://geeks.ms/etomas/wp-content/uploads/sites/154/2019/02/secrets-kv.png" alt="Se muestra como el pod tiene el directorio con los secretos" width="844" height="282" />][7]
  
**Leer los secretos desde ASP.NET Core**
  
Vale, hemos visto como podemos obtener los secretos como un volúmen de nuestro pod. Veamos ahora como leerlos desde ASP.NET Core. Por supuesto podríamos leer &#8220;a mano&#8221; los ficheros del directorio cuando necesitemos el valor de un secreto, pero así no es como hacemos las cosas en ASP.NET Core. Lo que nos interesa es que estos valores estén en la configuración de la aplicación y que se integren con el resto de valores que provengan de otras fuentes (variables de entorno, fichero _appsettings.json_, etc).
  
Por suerte eso es muy sencillo, **basta con crear un proveedor de configuración nuevo y añadirlo a nuestra aplicación**. Vamos a **partir de un proyecto ASP.NET Core con la plantilla Empty**.
  
El primer paso es crear el IConfigurationSource que nos indica **desde donde leemos los valores de configuración y devuelve el objeto que debe leer desde esa fuente****. **En nuestro caso se trata de un directorio:

<pre class="EnlighterJSRAW" data-enlighter-language="csharp">public class FolderConfigurationSource : IConfigurationSource
{
    public string Folder { get; }
    public FolderConfigurationSource(string folder)
    {
        Folder = folder;
    }
    public IConfigurationProvider Build(IConfigurationBuilder builder)
    {
        return new FolderConfigurationProvider(this);
    }
}</pre>

Bien, ahora nos toca crear el _FolderConfigurationProvider_ que es el encargado de leer los valores de configuración a partir de un _FolderConfigurationSource_:

<pre class="EnlighterJSRAW" data-enlighter-language="csharp">public class FolderConfigurationProvider : ConfigurationProvider
{
    private readonly FolderConfigurationSource _source;
    public FolderConfigurationProvider(FolderConfigurationSource source)
    {
        _source = source;
    }
    public override void Load()
    {
        var entries = Directory.EnumerateFiles(_source.Folder);
        foreach (var entry in entries)
        {
            var name = Path.GetFileName(entry);
            Data.Add(name, File.ReadAllText(entry));
        }
    }
}</pre>

Más fácil imposible, ¿no? Leemos todos los ficheros de la carpeta (representada por el _FolderConfigurationSource_) y los agregamos al diccionario Data que obtenemos de la clase base _ConfiurationProvider_.
  
Finalmente nos queda ver como agregar el objeto _FolderConfigurationSource_, la forma habitual es usando un método de extensión sobre _IConfigurationBuilder,_ aunque este método es opcional, pero facilita el uso de nuestro proveedor a los clientes:

<pre class="EnlighterJSRAW" data-enlighter-language="csharp">public static class FolderConfigurationExtensions
{
    public static IConfigurationBuilder AddFolder(this IConfigurationBuilder builder, string folderName) =&gt;
        builder.Add(new FolderConfigurationSource(folderName));
}</pre>

Ahora debemos añadir este proveedor de configuración a nuestra aplicación. Para ello editamos el método _CreateWebHostBuilder _de la clase _Program_:

<pre class="EnlighterJSRAW" data-enlighter-language="csharp">public static IWebHostBuilder CreateWebHostBuilder(string[] args) =&gt;
    WebHost.CreateDefaultBuilder(args)
        .ConfigureAppConfiguration(cb =&gt; cb.AddFolder("/kv-data"))
        .UseStartup&lt;Startup&gt;();</pre>

Observa como usamos el método de extensión &#8220;AddFolder&#8221; que hemos definido antes.
  
Ahora simplemente edita la clase _Startup_ para que quede como a continuación:

<pre class="EnlighterJSRAW" data-enlighter-language="csharp">public class Startup
{
    public IConfiguration Configuration { get; }
    public Startup(IConfiguration conf)
    {
        Configuration = conf;
    }
    public void ConfigureServices(IServiceCollection services)
    {
    }
    public void Configure(IApplicationBuilder app, IHostingEnvironment env)
    {
        if (env.IsDevelopment())
        {
            app.UseDeveloperExceptionPage();
        }
        app.Run(async (context) =&gt;
        {
            var entries = Configuration.AsEnumerable();
            foreach (var entry in entries)
            {
                await context.Response.WriteAsync($"{entry.Key} = {entry.Value}\n");
            }
        });
    }
}</pre>

No hay ningún secreto: **esto levanta un servidor que para cualquier petición que reciba hará lo mismo: mostrar todos los valores que hay en el objeto de configuración global**.
  
Bien, este código es el que tengo yo en la imagen _testedu.azurecr.io/demokv_ que hemos usado antes. En el clúster no he desplegado ningún servicio ni nada, así que no tengo acceso al pod desde el exterior. Pero para mostrar que funciona tampoco lo necesito: puedo abrir una sesión interactiva de terminal contra el _pod_ y ejecutar curl (que está presente en las imágenes de runtime de ASP.NET Core), hacer una petición a _localhost_ y ver el resultado:
  
[<img class="alignnone size-full wp-image-2319" src="https://geeks.ms/etomas/wp-content/uploads/sites/154/2019/02/secrets-kv-netcore.png" alt="Sesión interactiva contra el pod donde se ve que el objeto IConfiguration contiene los valores del keyvault" width="850" height="786" />][8]
  
Observad como se me muestra todo el contenido del objeto IConfiguration que contiene las variables de entornio, entradas que hubiese en _appsettings.json** **_**pero también los dos secretos del Key Vault**. De este modo, los valores del Key Vault participan del sistema de configuración estándard de ASP.NET Core.
  
Más fácil... imposible, ¿no?
  
¡Espero que os haya resultado interesante!
  
&nbsp;

 [1]: https://azure.microsoft.com/es-es/services/key-vault/
 [2]: https://docs.microsoft.com/en-us/aspnet/core/security/key-vault-configuration?view=aspnetcore-2.2
 [3]: https://docs.microsoft.com/en-us/rest/api/keyvault/
 [4]: https://github.com/Azure/kubernetes-keyvault-flexvol
 [5]: https://geeks.ms/etomas/wp-content/uploads/sites/154/2019/02/flexvol-keyvault-ds.png
 [6]: https://github.com/Azure/aad-pod-identity
 [7]: https://geeks.ms/etomas/wp-content/uploads/sites/154/2019/02/secrets-kv.png
 [8]: https://geeks.ms/etomas/wp-content/uploads/sites/154/2019/02/secrets-kv-netcore.png