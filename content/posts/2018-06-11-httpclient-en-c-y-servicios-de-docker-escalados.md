---
title: 'HttpClient en C# y servicios de Docker escalados.'
description: 'HttpClient en C# y servicios de Docker escalados.'
author: eiximenis

date: 2018-06-11T11:09:25+00:00
geeks_url: /?p=2080
geeks_ms_views:
  - 679
categories:
  - asp.net core
  - docker
  - kubernetes

---
En este post vamos a ver como escalar servicios, tanto en Compose, como en Swarm como en Kubernetes y luego veremos algunas consideraciones cuando usemos HttpClient desde el cliente al acceder a un servidor escalado.
  
Nos centramos en el escenario de escalado básico, es decir, sin demasiada lógica.
  
**[Autobombo]:** Si estás interesado en temas de Docker y Kubernetes, échale un vistazo a [mi curso de Docker y Kubernetes en CampusMVP][1].
  
<!--more-->


  
Para ello vamos a verlo en un ejemplo: Vamos a empezar creando un servidor nodejs que será el que escalaremos. Su código es muy, muy simple:

<pre class="EnlighterJSRAW" data-enlighter-language="js">const express = require('express');
const os = require("os");
const app = express();
let counter = 0;
app.get('/', function (req, res) {
  const machine = os.hostname();
  counter++;
  res.send(`Hello from ${machine}. You called me ${counter} times.`);
});
app.listen(3000, function () {
  console.log('Example app listening on port 3000!');
});</pre>

Este código levanta un servidor que escucha por el puerto especificado y que simplemente muestra el nombre de la máquina y un contador que se va incrementando.
  
Ahora creamos un _Dockerfile_ para poder construir la imagen:

<pre class="EnlighterJSRAW" data-enlighter-language="raw">FROM node:8.9-alpine
ENV NODE_ENV production
WORKDIR /usr/src/app
COPY ["package.json", "package-lock.json*", "npm-shrinkwrap.json*", "./"]
RUN npm install --production --silent && mv node_modules ../
COPY . .
EXPOSE 3000
ENTRYPOINT [ "node", "index.js" ]</pre>

Ahora vamos a crear un cliente en .NET que será un servicio que llamará a este servidor y nos devolverá su resultado. Para ello creamos una web en ASP.NET Core muy sencillita. Partiendo de la plantilla de proyecto de ASP.NET Core Empty, simplemente modificamos la clase Startup:

<pre class="EnlighterJSRAW" data-enlighter-language="csharp">public class Startup
{
    public IConfiguration Configuration { get; }
    public Startup(IConfiguration configuration) =&gt; Configuration = configuration;
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
            var client = new HttpClient();
            var response = await client.GetAsync(Configuration["helloserver"]);
            var serverData = await response.Content.ReadAsStringAsync();
            var toSend = new
            {
                Client = new { Name = Environment.MachineName },
                ServerData = serverData
            };
            context.Response.ContentType = "text/json";
            await context.Response.WriteAsync(JsonConvert.SerializeObject(toSend));
        });
    }
}</pre>

Puedes usar tanto NetCore 2.0 como 2.1, el código es el mismo.
  
No tiene más: simplemente a cada petición responde con un json que contiene dos campos: Client con el nombre de la máquina del cliente y ServerData con la respuesta del servidor. Finalmente creamos un Dockerfile para el proyecto:

<pre class="EnlighterJSRAW" data-enlighter-language="raw">FROM microsoft/dotnet:2.1-aspnetcore-runtime AS base
WORKDIR /app
EXPOSE 80
FROM microsoft/dotnet:2.1-sdk AS build
WORKDIR /src
COPY netclient/netclient.csproj netclient/
RUN dotnet restore netclient/netclient.csproj
COPY . .
WORKDIR /src/netclient
RUN dotnet build netclient.csproj -c Release -o /app
FROM build AS publish
RUN dotnet publish netclient.csproj -c Release -o /app
FROM base AS final
WORKDIR /app
COPY --from=publish /app .
ENTRYPOINT ["dotnet", "netclient.dll"]
</pre>

Para ponerlo todo en marcha podemos usar un fichero compose como el siguiente:

<pre class="EnlighterJSRAW" data-enlighter-language="json">version: '3.0'
services:
  server:
    image: helloserver
    build:
      context: ./server
      dockerfile: Dockerfile
    environment:
      NODE_ENV: production
    ports:
      - "3000"
  netclient:
    image: netclient
    build:
      context: ./netclient
      dockerfile: netclient/Dockerfile
    environment:
      helloserver: http://server:3000
    ports:
      - 8000:80</pre>

Si ahora lo ejecutamos (_docker-compose up_) y accedemos al cliente, vemos como nos devuelve la salida del servidor. Todo correcto:
  
[<img class="alignnone size-medium wp-image-2082" src="https://geeks.ms/etomas/wp-content/uploads/sites/154/2018/06/helloserver-and-netclient-300x95.png" alt="Salida de curl donde se ve que el cliente habla con el servidor" width="300" height="95" />][2]
  
**Escalando en compose**
  
Escalar en compose es fácil, para ello puedes poner la aplicación en marcha usando:

<pre class="EnlighterJSRAW" data-enlighter-language="null">docker-compose up --scale server=10</pre>

Eso te levantará el cliente y 10 servidores:
  
[<img class="alignnone size-medium wp-image-2083" src="https://geeks.ms/etomas/wp-content/uploads/sites/154/2018/06/escalado-compose-300x123.png" alt="Escalado en compose" width="300" height="123" />][3]
  
(Si te da errores de puerto, asegúrate que en el _docker-compose_ no tienes el puerto 3000 del servidor mapeado a ningún puerto del host. Compose no puede escalar servicios si tienes los puertos mapeados, ya que todas las N instancias intentarían usar el mismo puerto del host).
  
Si ahora ejecutas varias veces el cliente verás como la salida es correcta: te van respondiendo distintos servidores, pero sin ningún orden en concreto (en la imagen uno ha respondido dos veces y otros ninguno):
  
[<img class="alignnone size-medium wp-image-2084" src="https://geeks.ms/etomas/wp-content/uploads/sites/154/2018/06/escalado-compose2-300x267.png" alt="Responden varios servidores" width="300" height="267" />][4]
  
**Escalando en swarm**
  
Dado que tenemos un fichero compose es fácil pasar a modo swarm y usar escalado de swarm en lugar compose. Para ello debes poner Docker &#8220;en modo swarm&#8221; tecleando &#8220;_docker swarm init_&#8220;. Con eso has creado un clúster de Swarm con una sola máquina (la tuya). Puedes obviar la salida del comando ya que no agregaremos más máquinas al clúster.
  
El siguiente paso es desplegar en Swarm. Swarm usa los ficheros de compose, así que desplegar es muy fácil:

<pre class="EnlighterJSRAW" data-enlighter-language="raw">docker stack deploy httpclient --compose-file docker-compose.yml</pre>

Es importante que tengas las imágenes construídas en tu máquina, ya que Swarm (a diferencia de compose) no puede construir imágenes. Ahora debemos escalar el servicio servidor a 10 instancias, con el comando &#8220;_docker service scale httpclient_server=10_&#8221; (httpclient_server es el nombre del servicio):
  
[<img class="alignnone size-medium wp-image-2085" src="https://geeks.ms/etomas/wp-content/uploads/sites/154/2018/06/escalado-swarm-300x117.png" alt="Escalado con Swarm" width="300" height="117" />][5]
  
Si ahora vas lanzando otra vez peticiones a &#8220;http://localhost:8000&#8221; verás como te van respondiendo los servicios uno tras otro. Hay una diferencia con el caso de compose: **cada vez responde un servicio distinto, y cuando han respondido todos, vuelta a empezar**. Es decir, es un round-robin puro: La primera petición se enruta al primer servicio, la segunda al segundo y así hasta que todos los servicios han ejecutado una petición, donde entonces la siguiente se vuelve a enrutar al primero de nuevo y vuelta a empezar.
  
Una vez hecho el experimento puedes abandonar (y destruír) el cluster de Swarm usando &#8220;_docker swarm leave &#8211;force_&#8220;.
  
**Escalando en Kubernetes**
  
Vamos ahora a repetir el experimento en Kubernetes. En este caso, necesitamos los ficheros de despliegue de Kubernetes **y además tener las imágenes en algún registro **(no basta con que estén en nuestra máquina).
  
Los ficheros de despliegue de Kubernetes son muy sencillos: dos deployments y dos servicios. En este caso los he agrupado todos juntos en un mismo fichero (k8s.yml):

<pre class="EnlighterJSRAW" data-enlighter-language="json">apiVersion: v1
kind: Service
metadata:
  labels:
    component: server
  name: server
spec:
  ports:
  - port: 80
  selector:
    component: server
---
apiVersion: v1
kind: Service
metadata:
  labels:
    component: client
  name: client
spec:
  type: NodePort
  ports:
  - port: 80
  selector:
    component: client
---
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: server
spec:
  template:
    metadata:
      labels:
        component: server
    spec:
      containers:
      - name: helloserver
        image: dockercampusmvp/helloserver
        env:
        - name: PORT
          value: "80"
---
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: client
spec:
  template:
    metadata:
      labels:
        component: client
    spec:
      containers:
      - name: helloclient
        image: dockercampusmvp/helloclient:2.1
        env:
        - name: helloserver
          value: http://server</pre>

(Los ficheros son para MiniKube. Si usas AKS o GCE entonces solo debes cambiar &#8220;NodePort&#8221; por &#8220;LoadBalancer&#8221; para que el servicio se exponga al exterior.
  
Una vez instalado este fichero en el servidor, puedes escalar el servicio _server_ usando &#8220;_kubectl scale &#8211;replicas=10 deployments/server&#8221;_:
  
[<img class="alignnone size-medium wp-image-2087" src="https://geeks.ms/etomas/wp-content/uploads/sites/154/2018/06/escalado-k8s-300x220.png" alt="Escalado en k8s" width="300" height="220" />][6]
  
Ahora puedes usar el comando _minikube service client_ que te abrirá un navegador a la IP de este servicio (o bien mirar la IP del nodo de _minikube_ y el puerto del servicio, lo que es equivalente. En el caso de usar _LoadBalancer_ simplemente mira en la columna &#8220;EXTERNAL-IP&#8221; del comando &#8220;_kubectl get svc_&#8220;). Al igual que antes te irán respondiendo distintos servidores:
  
[<img class="alignnone size-medium wp-image-2089" src="https://geeks.ms/etomas/wp-content/uploads/sites/154/2018/06/escalado-k8s2-300x259.png" alt="Respondiendo varios servidores en k8s" width="300" height="259" />][7]
  
**Usando HttpClient &#8220;de la forma correcta&#8221;**
  
Vale, si arrugaste la nariz cuando viste el código del cliente, entonces ya sabes que **[NO HE USANDO HTTPCLIENT DE LA FORMA RECOMENDADA][8].** Por si acaso, recordad: **HttpClient debe usarse como singleton**, ya que ir creando instancias de este puede llevarnos a problemas por exhaurir todos los sockets. De hecho la [documentación oficial][9] lo deja bastante claro:

<pre>HttpClient is intended to be instantiated once and re-used throughout the life of an application. Instantiating an HttpClient class for every request will exhaust the number of sockets available under heavy loads. This will result in SocketException errors.</pre>

Para más información consultad [ese artículo][8].
  
Bien, he modificado el código para usar mi cliente de la forma recomendada, simplemente creando un HttpClient en el constructor de Startup:

<pre class="EnlighterJSRAW" data-enlighter-language="csharp">private readonly HttpClient _client;
public Startup(IConfiguration configuration)
{
    _client = new HttpClient();
    Configuration = configuration;
}</pre>

Y usando siempre este mismo __client_. Y desplegando eso en k8s el resultado es:
  
[<img class="alignnone wp-image-2098 size-medium" src="https://geeks.ms/etomas/wp-content/uploads/sites/154/2018/06/escalado-k8s3-300x254.png" alt="escalado en k8s - responde siempre el mismo" width="300" height="254" />][10]
  
¡Siempre nos está respondiendo el mismo servidor! **¿Qué está ocurriendo? **(Si lo pruebas con compose ocurre lo mismo).
  
Ahora bien, **si usamos varios clientes a la vez**, el resultado no es tan dramático, aunque sigue siendo lejos de lo deseable:
  
[<img class="alignnone size-medium wp-image-2099" src="https://geeks.ms/etomas/wp-content/uploads/sites/154/2018/06/escalado-compose-multiples-clientes-300x92.png" alt="" width="300" height="92" />][11]
  
En esta imagen ha abierto seis terminales y he introducido el mismo comando en ellas, para hacer 100 peticiones y contar cuantas veces me responde el mismo servidor. Se puede ver que las 600 peticiones totales se las reparten entre 3 servidores. Por otro lado si solo abres un terminal y haces 600 peticiones de golpe irán todas al mismo servidor. Curioso.
  
Bueno.. vamos a ver como funciona el escalado en Compose/Swarm y luego en Kubernetes.
  
**Como funciona el escalado en Compose/Swarm**
  
Tanto Compose como Swarm hacen lo mismo: agregan varias entradas  a la tabla de DNS. Para verlo, con la aplicación de compose en marcha y el servicio _server_ escalado, abre una sesión ineractiva contra el cliente usando &#8220;_docker exec -it <id-contenedor-client> /bin/bash_&#8220;.
  
Ahora solo debes instalar nslookup:

<pre class="EnlighterJSRAW" data-enlighter-language="shell">apt-get update
apt-get install dnsutils -y</pre>

Y ya podemos usar _nslookup server_ para ver el resultado:
  
[<img class="alignnone size-medium wp-image-2095" src="https://geeks.ms/etomas/wp-content/uploads/sites/154/2018/06/nslookup-compose-116x300.png" alt="nslookup en compose" width="116" height="300" />][12]
  
Observa como hay diez entradas en la tabla de DNS del cliente, apuntando a las 10 ips internas de los 10 contenedores que ejecutan el servidor. Cada vez que hago _nslookup_ el orden cambia. La idea es que si elijes la primera IP de la lista, cada vez obtienes una IP distinta. Bien.
  
**Como funciona el escalado en Kubernetes**
  
Kubernetes funciona distinto que Compose/Swarm pero el efecto final es parecido (y además tiene varios modos de escalado). Al igual que antes necesitamos instalar _nslookup_ en el contenedor que ejecuta el cliente. Para ello miramos cual es el _pod_ que ejecuta el cliente usando el comando &#8220;_kubectl get pods | findstr -I client_&#8221; y luego ejecutando un &#8220;_kubectl exec -it <nombre-pod> /bin/bash_&#8221; para abrir una sesión interactiva con ese pod:
  
[<img class="alignnone size-medium wp-image-2092" src="https://geeks.ms/etomas/wp-content/uploads/sites/154/2018/06/k8s-sesion-interactiva1-300x103.png" alt="Abriendo sesión interactiva en k8s" width="300" height="103" />][13]
  
Ahora, al igual que antes, instalamos nslookup en este contenedor, tecleando:

<pre class="EnlighterJSRAW" data-enlighter-language="shell">apt-get update
apt-get install dnsutils -y</pre>

Y ya podemos usar _nslookup server_ para ver el resultado:
  
[<img class="alignnone size-medium wp-image-2093" src="https://geeks.ms/etomas/wp-content/uploads/sites/154/2018/06/nslookup-k8s-300x153.png" alt="nslookup en k8s" width="300" height="153" />][14]
  
Hay una sola entrada en el DNS que apunta a la IP (interna) del servicio. En efecto, en Kubernetes los servicios son los encargados de agrupar los distintos _pods_ en una sola IP.
  
Por supuesto si dentro del pod ejecutas varias veces &#8220;_curl http://server_&#8221; (la imagen base de Net Core tiene curl instalado, así que lo tenemos disponible), verás como, efectivamente, nos van respondiendo servidores distintos (como era de esperar)
  
Vamos a ver como escala el servicio. Para ello debemos ejecutar comandos no en ningún pod, si no en el nodo de Kubernetes. Si usas minikube puedes usar simplemente &#8220;_minikube ssh_&#8221; para abrir una sesión SSH contra el (único) nodo de Minikube. Si usas otro Kubernetes, debes mirar las instrucciones de tu proveedor, ya que cada caso es distinto.
  
Una vez en el nodo teclea &#8220;_iptables-save | grep server_&#8221; (quizá debas usar sudo) para ver el contenido:
  
[<img class="alignnone size-medium wp-image-2094" src="https://geeks.ms/etomas/wp-content/uploads/sites/154/2018/06/iptables-k8s-300x137.png" alt="iptables en k8s" width="300" height="137" />][15]
  
Por defecto los servicios en MiniKube funcionan en el modo **iptables**. En este modo, se instalan reglas de _iptables_ que redirigen las peticiones a la IP del servicio a uno de los _pods_ subyacentes (de forma aleatoria).
  
No soy un experto, ni de lejos, en _iptables_ y todo eso (lamentablemente) se me escapa un poco, pero esas reglas de _iptables_ son las que redirigen el tráfico desde la IP virtual del servicio a alguna de las IPs reales.
  
**Vale... ¿y como solucionar ese problema?**
  
Bueno, este problema se debe a **que el objeto HttpClient (el único que tenemos ahora)** reutiliza, en algunos casos, la conexión. Eso es para mejorar el rendimiento, a costa de rompernos este balanceo basado en iptables o múltiples entradas de DNS.
  
La solución pasa por usar la línea:

<pre class="EnlighterJSRAW" data-enlighter-language="csharp">_client.DefaultRequestHeaders.ConnectionClose = true;</pre>

Justo después de crear el objeto _HttpClient_ (en el constructor de _Startup_) para así forzar que HttpClient cierre la conexión. **Ojo, que eso penaliza el rendimiento**. Con esa línea agregada el resultado es el siguiente:
  
[<img class="alignnone wp-image-2101 size-medium" src="https://geeks.ms/etomas/wp-content/uploads/sites/154/2018/06/escalado-compose-multiples-clientes_2-300x158.png" alt="" width="300" height="158" />][16]
  
Ahora sí que vemos que van respondiendo todos los servidores, consiguiéndose el objetivo con el escalado. Si lo pruebas en Kubernetes el resultado es muy parecido:
  
[<img class="alignnone size-medium wp-image-2102" src="https://geeks.ms/etomas/wp-content/uploads/sites/154/2018/06/escalado-k8s-multiples-clientes-300x160.png" alt="Ahora en k8s también responden varios servicios" width="300" height="160" />][17]
  
**En resumen**, hemos visto lo siguiente:

  1. Como escalar en Compose
  2. Como escalar en Swarm
  3. Como escalar en Kubernetes
  4. Qué hace el escalado en Compose/Swarm
  5. Qué hace el escalado en Kubernetes
  6. El detalle a tener en cuenta si usamos esos mecanismos de escalado y usamos HttpClient como singleton (la forma recomendada)

Por supuesto este escenario de escalado que hemos visto es el más simple, para escenarios más avanzados podemos o bien desplegar contenedores que realicen tareas de _load balancer_ o bien, en el caso de Kubernetes, usar el modo de servicios **_ipvs_**.
  
Os dejo lo siguiente por si queréis experimentar:

  1. [El código fuente del servicio y cliente][18] (netcore 2.0 y 2.1). Con sus dockerfiles
  2. Los ficheros de compose/swarm y Kubernetes
  3. Para Kubernetes, he puesto las imágenes en el DockerHub: 
      1. dockercampusmvp/helloclient:2.0 -> Versión inicial del cliente en netcore 2.0
      2. dockercampusmvp/helloclient:2.1 -> Versión inicial del cliente en netcore 2.1 (el tag _latest_ apunta a esa versión)
      3. dockercampusmvp/helloclient:2.1-singleton -> Cliente en 2.1 y con HttpClient en singleton
      4. dockercampusmvp/helloclient:2.1-singleton-close -> Cliente en 2.1 y con HttpClient en singleton y cerrando la conexión
      5. dockercampusmvp/helloserver:latest -> Servidor nodejs

¡Y, por supuesto, si alguien ha experimentado otros comportamientos con el tema del escalado, que se sienta libre de poner un comentario!

 [1]: https://www.campusmvp.es/catalogo/Product-Docker-y-Kubernetes-desarrollo-y-despliegue-de-aplicaciones-basadas-en-contenedores_237.aspx
 [2]: https://geeks.ms/etomas/wp-content/uploads/sites/154/2018/06/helloserver-and-netclient.png
 [3]: https://geeks.ms/etomas/wp-content/uploads/sites/154/2018/06/escalado-compose.png
 [4]: https://geeks.ms/etomas/wp-content/uploads/sites/154/2018/06/escalado-compose2.png
 [5]: https://geeks.ms/etomas/wp-content/uploads/sites/154/2018/06/escalado-swarm.png
 [6]: https://geeks.ms/etomas/wp-content/uploads/sites/154/2018/06/escalado-k8s.png
 [7]: https://geeks.ms/etomas/wp-content/uploads/sites/154/2018/06/escalado-k8s2.png
 [8]: https://aspnetmonsters.com/2016/08/2016-08-27-httpclientwrong/
 [9]: https://docs.microsoft.com/en-gb/dotnet/api/system.net.http.httpclient?view=netcore-2.1
 [10]: https://geeks.ms/etomas/wp-content/uploads/sites/154/2018/06/escalado-k8s3.png
 [11]: https://geeks.ms/etomas/wp-content/uploads/sites/154/2018/06/escalado-compose-multiples-clientes.png
 [12]: https://geeks.ms/etomas/wp-content/uploads/sites/154/2018/06/nslookup-compose.png
 [13]: https://geeks.ms/etomas/wp-content/uploads/sites/154/2018/06/k8s-sesion-interactiva1.png
 [14]: https://geeks.ms/etomas/wp-content/uploads/sites/154/2018/06/nslookup-k8s.png
 [15]: https://geeks.ms/etomas/wp-content/uploads/sites/154/2018/06/iptables-k8s.png
 [16]: https://geeks.ms/etomas/wp-content/uploads/sites/154/2018/06/escalado-compose-multiples-clientes_2.png
 [17]: https://geeks.ms/etomas/wp-content/uploads/sites/154/2018/06/escalado-k8s-multiples-clientes.png
 [18]: https://1drv.ms/f/s!Asa-selZwiFlg_Af-xYxw8ugq0y7xQ