---
title: "Project tye"
author: eiximenis
description: "Tye es un proyecto (experimental) del equipo de netcore para aydarnos al desarrollo de Microservicios. ¿Lo consigue? (TL;DR) ¡Sí!"
date: 2020-10-05
draft: false
categories:
  - netcore
tags:
  - microservices
  - kubernetes
  - k8s
---

[Project Tye](https://github.com/dotnet/tye) (simplemente Tye de ahora en adelante) es un **proyecto experimental** del equipo de NetCore pensado para **ayudarnos en el desarrollo de apliaciones de (micro)servicios** basadas en NetCore. Aunque podríamos llegar a usar Tye sin netcore, no es para lo que está concebida: NO es una herramienta de propósito general. Es una herramienta pensada para desarrolladores en netcore.

## Funcionalidades de Tye

Es complicado definir qué es exactamente Tye, ya que anda a medio camino entre un orquestador básico, un ejecutor de aplicaciones, un _control plane_ básico y un gestor de despliegues. Quizá sea más sencillo poner una lista de funcionalidades de Tye:

* Ejecuta con un solo comando (`tye run`) tu aplicación entera de microservicios, incluyendo las dependencias.
* Ofrece un pequeño portal des de el cual se puede ver los servicios ejecutándose y sus logs
* Reinicia automáticamente los servicios que den error
* Ofrece un sistema de _service discovery_
* Mantiene 1 o más réplicas de cada servicio
* Colecciona métricas de los servicios
* Es capaz de generar los manifiestos de Kubernetes necesarios y desplegarlos en cualquier clúster que tengas.
* Tiene algunas integraciones con otros productos (p. ej. con [Dapr](https://dapr.io/))

Analicemos todos esos puntos, empezando por el primero...

## Ejecutar tu aplicación con un solo comando (o tye vs compose)

No es ningún secreto que con _compose_ también puedo levantar toda mi aplicación y sus dependencias simplemente usando (`docker-compose up`). Qué ofrece, entonces, Tye que no me de compose?

La respuesta es que compose es de uso genérico, que ejecuta cargas de trabajo de contenedores y las orquesta levemente. Tye, por otro lado, está pensado para ejecutar aplicaciones hechas en netcore. La principal diferencia entre un `docker-compose up` y un `tye run` es que en el segundo **los proyectos netcore se ejecutan directamente con `dotnet run`**. A cada proceso de netcore, tye le asigna puertos aleatorios (en lugar de los tradicionales http/5000 y https/5001).

Al igual que Compose, Tye usa un fichero YAML, pero con su propio formato:

```yaml
name: testtye
services:
- name: netcoresvc
  project: NetcoreServer/NetcoreServer.csproj
- name: nodeclient
  dockerFile: NodeClient/Dockerfile
  dockerFileContext: NodeClient/
  bindings:
    - port: 3000
  env:
    - name: PORT
      value: 3000
```

Es parecido a Compose, pero con la salvedad de que en Tye los servicios no tienen porque ser contenedores. En este caso tenemos dos servicios, uno de los cuales es un proyecto de netcore (ese se ejecutará en local). El otro es un contenedor de node. Cuando uso contenedores a Tye le puedo pasar o bien una imagen (aunque en este caso siempre hace `docker pull` por lo que si es una imagen solo local no funcionará), o bien un Dockerfile y entonces Tye construye la imagen automáticamente. Cuando son procesos no netcore, debo indicarle a Tye un binding, que es **un puerto de mi máquina host que se enlazará al (mismo) puerto del contenedor**. Esos bindings además, se incluyen en el sistema de service discovery.

Al igual que compose, Tye puede levantar infraestructura adicional. Esa infraestructura por lo general son contenedores docker, pero (a diferencia de compose) pueden ser también directamente ejecutables. Eso plantea una duda: si mi código son procesos que corren directamente en mi máquina, pero la infraestructura adicional pueden ser contenedores Docker... ¿cómo se comunican?

Por un lado los contenedores Docker de infraestructura (pongamos un SQL Server p. ej.), exponen los puertos en `localhost`. Ahí es Docker quien se encarga de todo, tye no hace nada. Si tienes un SQL Server ejecutándose en tu aplicación tye, este estará en `localhost:1433` y tu código accederá a él usando esa dirección. Pero, ¿qué ocurre si es un contenedo el que debe comunicarse con tu proceso dotnet?

Imagina que tienes un microservicio hecho en nodejs, y el resto son de netcore. Cuando levantes con Tye la aplicación, Tye no sabe ejecutar nodejs, así que ese microservicio lo debes tener como imagen Docker. En este momento, tienes un contenedor (que ejecuta el microservicio de nodejs) y varios procesos locales que ejecutan tus servicios netcore. El contenedor se expone en `localhost:xxxx`, por lo que la comunicación desde netcore hacia el contenedor está, como ya hemos visto, asegurada. El problema es si el contenedor nodejs debe abrir una conexión contra nuestros microservicios netcore. Nuestros procesos netcore exponen puertos en `localhost` (Tye se encarga de eso), pero ese `localhost` no es el mismo `localhost` del contenedor. **¿Así pues, cómo puede comunicarse el microservicio de nodejs que se ejecuta bajo un contenedor con el servicio netcore que se ejecuta en local?**

Bien, tengo un cliente de nodejs y un servidor de netocore para ver ese caso. Lanzo la aplicación con `tye run` **y accedo al dashboard de tye** que se encuentra habitualmente en (`http://localhost:8000`) y veo que efectivamente la aplicación consta de un proyecto (el de netcore) y un contenedor (el de nodejs):

![Dashboard de Tye donde se ven los procesos](/images/posts/2020-10-05-tye1.png)

## Comuniación entre servicios

A diferencia de Compose (donde todo son contenedores), al usar Tye, tenemos una mezcla de procesos locales y de contenedores. En nuestro ejemplo `netcoresvc` es un proceso local, mientras que `nodeclient` es un contenedor. Acceder desde el proceso local (net core) al contenedor (nodejs) es sencillo, ya que este último expone los puertos en `localhost` a través de Docker, por lo que, el proceso local solo debe acceder a `localhost:xxxx` y allí estará el contenedor de nodejs escuchando. Ahí Tye no hace realmente nada, es Docker quien se encarga de todo.

Más divertido es el escenario inverso: acceder desde un contenedor a un proceso local. En este caso el proceso local expone también los puertos en `localhost`, pero ¡ojo! ese _localhost_ es la propia máquina, el host de Docker (recuerda que ahora se trata de un proceso local), por lo tanto si el contenedor accede a `localhost:yyyy`, no encontrará nadie, ya que para un contenedor `localhost` es sí mismo (y no el host de Docker). ¿Entonces, como habilita Tye esa comunicación?

La realidad ahí, me resulta un poco confusa, porque me da la sensación que Tye usa dos mecanismos distintos, o dicho de otro modo: habilita un mecanismo pero termina usando otro. Deja que te lo cuente y quedará más claro. El mecanismo que Tye habilita para permitir acceder desde un contenedor a un proceso local es que **por cada proceso local, crea un contenedor asociado que hace de _proxy_**. Esos _proxies_ los monta en la misma red de Docker donde están el resto de contenedores, de forma que se ven entre ellos. Así, cuando un contenedor debe comunicarse con un proceso local, se comunica realmente con el contenedor _proxy_ (controlado por Tye) y este simplemente reenvía la petición al proceso local que tenga asociado.

En nuestro ejemplo si ejecutas `tye run` verás en los logs que salen por pantalla, las siguientes líneas:

```
Running container netcoresvc-proxy_6bff71f4-e with ID 175cf67562d0
Running docker command network connect tye_network_6b903a53-e netcoresvc-proxy_6bff71f4-e --alias netcoresvc
Running container nodeclient_83750d08-3 with ID 9aea73dbd9c0
Running docker command network connect tye_network_6b903a53-e nodeclient_83750d08-3 --alias nodeclient
```

Las primeras dos líneas muestran como Tye pone en marcha el proxy y lo enlaza a la red (en mi caso `tye_network_6b903a53-e`) con el alias `netcoresvc`. Las dos úlimas muestran como Tye ejecuta el contenedor de node y lo enlaza a la misma red bajo el alias `nodeclient`. Si lanzo un `docker ps` tengo lo siguiente:

```
CONTAINER ID        IMAGE                                   COMMAND                  CREATED             STATUS                  PORTS                              NAMES
9aea73dbd9c0        nodeclient                              "docker-entrypoint.s…"   4 minutes ago       Up 4 minutes            0.0.0.0:3000->3000/tcp             nodeclient_83750d08-3
175cf67562d0        mcr.microsoft.com/dotnet/core/sdk:3.1   "dotnet Microsoft.Ty…"   4 minutes ago       Up 4 minutes                                               netcoresvc-proxy_6bff71f4-e
```

Por un lado el contenedor de node, por otro lado el proxy. Voy ahora a ejecutar una sesión interactiva mediante un contenedor de [busybox](https://hub.docker.com/_/busybox) enlazado a esa misma red (`docker run -it --network tye_network_6b903a53-e busybox /bin/sh`). Si ahora lanzas un `ping netcoresvc` verás como nos responde una IP, que es, precisamente la IP del proxy. Y puedes verificar con un `wget -qO- http://netcoresvc/` como obtienes la respuesta del proceso local. **Por lo tanto, el contenedor de proxy reenvía la petición al proceso local**. Pero, la realidad es que Tye parece usar otra estrategia para comunicar los contenedores con los procesos locales.

### Service Discovery

Uno de los servicios que ofrece Tye es un _service discovery_, es decir le podemos preguntar a Tye las URLs donde residen los distintos servicios (sean procesos locales o contenedores). Eso es necesario porque Tye elige puertos aleatorios para los procesos locales, por lo que, a priori, no sabemos en que URL se ejecutarán. Ese service discovery funciona "al estilo Kubernetes", es decir, **Tye inyecta variables de entorno (con un determinado patrón) que contienen las URLs de los distintos servicios**. De este modo, si quiero saber la URL por donde escucha por HTTP un servicio en concreto, solo debo consultar la variable de entorno correspondiente. El nombre de la variable de entorno se calcula a partir del nombre del servicio (el definido en `tye.yaml`) y el _binding_ que pidamos (el _binding_ significa si queremos la URL de HTTP, la de HTTPS o cualquier otro protocolo como gRPC). Dado que, por lo general, en .NET Core las variables de entorno se leen a través de [IConfiguration](https://docs.microsoft.com/en-us/dotnet/api/microsoft.extensions.configuration.iconfiguration?view=dotnet-plat-ext-3.1&viewFallbackFrom=net-5.0&WT.mc_id=AZ-MVP-4039791) existe un [paquete NuGet](https://www.nuget.org/packages/Microsoft.Tye.Extensions.Configuration) que ofrece un método de extensión sobre `IConfiguration` para pedir la URL de un servicio.

Pues, si analizamos las variables de entorno que nos ha inyectado Tye en el contenedor `nodeclient` podremos hacernos una idea de como espera Tye que accedamos al servicio de .NET (proceso local). Para verlo he abierto una sesión interactiva con el contenedor de node y he lanzado un `env | grep NETCORESVC` y ese ha sido el resultado:

```
SERVICE__NETCORESVC__HOST=host.docker.internal
SERVICE__NETCORESVC__PROTOCOL=http
SERVICE__NETCORESVC__PORT=59587
SERVICE__NETCORESVC__HTTPS__HOST=host.docker.internal
SERVICE__NETCORESVC__HTTPS__PROTOCOL=https
SERVICE__NETCORESVC__HTTPS__PORT=59588
NETCORESVC_SERVICE_HOST=host.docker.internal
NETCORESVC_SERVICE_PROTOCOL=http
NETCORESVC_SERVICE_PORT=59587
NETCORESVC_HTTPS_SERVICE_HOST=host.docker.internal
NETCORESVC_HTTPS_SERVICE_PROTOCOL=https
NETCORESVC_HTTPS_SERVICE_PORT=59588
``` 

Lo primero que puedes observar es que los valores están como repetidos (hay el doble de variables de entorno de las que se necesitarían). Eso es porque Tye crea variables de entorno en el formato Kubernetes (las que empiezan por `NETCORESVC_` y luego crea las variables de entorno en su propio formato (las que empiezan por `SERVICE__`). En todo caso lo que me interesa es que veas el valor del host, que es `host.docker.internal`. Recuerda que esas variables de entorno son las que ha inyectado Tye al contenedor de node, por lo que Tye "espera" que el contenedor de node acceda al servicio de netcore (proceso local) a través de `http://host.docker.internal:59587`. Este host `host.docker.internal` es un host que existe solo dentro de las redes de Docker y que representan a la propia máquina Host. Así, de ese modo el contenedor **accede al puerto 59587 del host directamente, sin pasar por proxy alguno**. 

### Investigando el contenedor proxy

Tye monta ese contenedor de proxy (aunque luego parece que no es necesario para permitir la comunicación entre contenedores y procesos locales), pero... ¿lo usa para algo más? Pues la verdad es que el código de este contenedor [es bastante sencillo](https://github.com/dotnet/tye/tree/master/src/Microsoft.Tye.Proxy). Parece que consta solo de un `Program.cs` y realmente parece que solo hace de proxy. He hecho algunas pruebas y efectivamente cuando, desde un contenedor accedo al servicio local usando el DNS de `netcoresvc` el proxy muestra logs de que redirige la conexión:

```
dbug: Microsoft.Tye.Proxy.Program[0]
      Attempting to connect to netcoresvc-proxy_acbe28d0-2 listening on 80:62577
dbug: Microsoft.Tye.Proxy.Program[0]
      Successfully connected to netcoresvc-proxy_acbe28d0-2 listening on 80:62577
dbug: Microsoft.Tye.Proxy.Program[0]
      Proxying traffic to netcoresvc-proxy_acbe28d0-2 80:62577
dbug: Microsoft.AspNetCore.Server.Kestrel.Transport.Sockets[6]
      Connection id "0HM38GQ6IUJ8O" received FIN.
dbug: Microsoft.AspNetCore.Server.Kestrel.Transport.Sockets[7]
      Connection id "0HM38GQ6IUJ8O" sending FIN because: "The Socket transport's send loop completed gracefully."
```

Observa que el proxy usa los puertos 80 y también el 443 y enruta tanto HTTP como HTTPS. Para configurarse este contendor usa determinadas variables de entorno (que le suministra Tye):

* `CONTAINER_HOST`: DNS del host de Docker (o sea `host.docker.internal` al menos en Docker Destkop bajo Windows)
* `PROXY_PORT`: Puertos locales (del contenedor) y remotos (del host) a mapear. Así, si el proceso local de netcore escucha en el `59587` (HTTP) y `59588` (HTTPS) esta variable valdrá: `80:59587;443:59588`.
* `APP_INSTANCE`: Parece contener el nombre del propio proxy, usado en los logs.

En resumen, el proxy está ahí y funciona perfectamente, por eso no entiendo porque Tye no lo usa en su sistema de service discovery. [He puesto una issue en el repo, preguntando precisamente eso](https://github.com/dotnet/tye/issues/687).

## Añadiendo réplicas

Una de las características de Tye es que hace súper sencillo el añadir réplicas de tus servicios y así probar en desarrollo escenarios concurrentes. He modificado el fichero `tye.yaml` añadiendo `replicas: 3` al servicio `netcoresvc` para ver que es lo que ocurre. Por un lado, efectivamente ahora tengo 3 réplicas del proceso (cada una de ellas escuchando en sus par de puertos de mi máquina (un puerto para HTTP, otro para HTTPS)). Ahora bien, en el dashboard Tye solo me muestra un par de puertos:

![Dashboard de Tye donde se ven los procesos. Se puede ver como el proceso de netcore solo muestra dos puertos](/images/posts/2020-10-05-tye2.png)

En la imagen he puesto el dashboard de Tye y parte de los logs. Observa como **en el dashboard solo vemos dos puertos, pero es que encima ninguno de esos dos puertos se corresponde a los puertos reales de los 3 procesos**. Eso es porque en este caso, el propio Tye hace de proxy y me ofrece un endpoint único para las 3 réplicas:

```
C:\>curl http://localhost:62838
Hello World from UFOBOT-V4 (2dc8cd39-a99e-4f4d-9cc7-f6df0bdf8792)
C:\>curl http://localhost:62838
Hello World from UFOBOT-V4 (da9e1ac7-183e-47bc-8f5e-22d9c7c463f8)
C:\>curl http://localhost:62838
Hello World from UFOBOT-V4 (2daab067-1bc3-4d14-833f-a241afd88c53)
C:\>curl http://localhost:62838
Hello World from UFOBOT-V4 (2dc8cd39-a99e-4f4d-9cc7-f6df0bdf8792)
```

El proceso `netcoresvc` simplemente imprime el nombre de la máquina y un Guid único. Se puede ver como parece que Tye usa un round robin y va repartiendo las peticiones entre las 3 réplicas. Por supuesto yo puedo acceder directamente a una réplica si sé su puerto (que sale en los logs). Teniendo réplicas, Tye configura el contenedor de proxy de la siguiente manera:

* `PROXY_PORT`: `80:62838;443:62842`
* `CONTAINER_HOST`: `host.docker.internal`

Nada inesperado: el contenedor de proxy queda enlazado al par de puertos que controla Tye, por lo que las llamadas a través del contenedor de proxy también serán balanceadas entre las instancias.

Y, como ya te debes imaginar, al contenedor de node, Tye le pasa la siguiente configuración:

```
SERVICE__NETCORESVC__PROTOCOL=http
SERVICE__NETCORESVC__HOST=host.docker.internal
SERVICE__NETCORESVC__PORT=62838
SERVICE__NETCORESVC__HTTPS__PROTOCOL=https
SERVICE__NETCORESVC__HTTPS__HOST=host.docker.internal
SERVICE__NETCORESVC__HTTPS__PORT=62842
```

Lo mismo que en el caso anterior. Si usamos esas variables, el contenedor de node accede directamente a la máquina Host, a los puertos controlados por Tye y por lo tanto las llamadas serán balanceadas.

## Replicas de contenedores

El escenario anterior era cuando las réplicas eran de los procesos locales (netcore). Pero también podemos usar réplicas en los contenedores (en este ejemplo replicar el contenedor de nodejs). En este caso Tye levanta un contenedor por cada réplica **y los puertos se mapean a puertos elegidos por Tye**. En mi ejemplo, el binding del contenedor de node es sobre el puerto `3000` y este es el puerto que se abría en mi máquina. Ahora bien, cuando tenemos réplicas eso cambia:

```
C:\>docker ps
CONTAINER ID        IMAGE                                   COMMAND                  CREATED              STATUS                PORTS                                NAMES
569f6ad05339        nodeclient                              "docker-entrypoint.s…"   35 seconds ago      Up 34 seconds         3000/tcp, 0.0.0.0:51478->51478/tcp   nodeclient_036f6f79-2
4bf332f84a23        nodeclient                              "docker-entrypoint.s…"   35 seconds ago      Up 34 seconds         3000/tcp, 0.0.0.0:51477->51477/tcp   nodeclient_fea63638-d
1d911b66007e        nodeclient                              "docker-entrypoint.s…"   35 seconds ago      Up 35 seconds         3000/tcp, 0.0.0.0:51476->51476/tcp   nodeclient_92aba817-4
```

En el log de `tye run` verás un mensaje parecido al siguiente:

```
Mapping external port 3000 to internal port(s) 51476, 51477, 51478 for nodeclient binding null
```

El problema ahora es el siguiente: Tye sabe que hay un _binding_ del contenedor de node, sobre el puerto 3000. Cuando solo había una réplica, Tye se limita a enlazar el puerto 3000 de mi máquina (el valor especificado en el binding), al puerto 3000 del contenedor, y por lo tanto accediendo a `localhost:3000`, ahí estaba el contenedor de node. Ahora con réplicas Tye ya no puede hacer eso, ya que no puede abrir N veces el puerto 3000. Por lo tanto lo que Tye hace es mapear un puerto al azar por cada réplica (en mi caso son esos `51478`, `51477`y `51476`) del contenedor al mismo puerto local de la máquina. Además de eso, Tye **usa el propio proxy que él contiene para enlazar el puerto 3000 de mi máquina a esos 3 puertos**. Es decir, llamando a `localhost:3000` desde mi máquina, la petición será interceptada por Tye y redirigida a cualquiera de esos 3 puertos y por lo tanto a cualquiera de las 3 réplicas del contenedor de node.

**Por lo tanto, Tye hace su trabajo magistralmente**, el problema está **en mi código de nodejs**. Dicho código abre siempre el puerto del contenedor especificado en la variable de entorno `PORT`. En el fichero `tye.yaml` siempre le especifico `3000`, por lo que cada réplica abre su puerto 3000, pero Tye no hace nada con ese puerto del contenedor. En el caso de proyectos netcore eso no ocurría, porque Kestrel usa por defecto la variable de entorno `ASPNETCORE_URLS` para abrir sus puertos y Tye, como entiende de netcore, modifica esa variable de entorno para cada réplica.

¿Puedo, de algún modo desde el contenedor de node saber qué puerto me ha sido asignado? Pues sí, porque Tye me inyecta esa configuración en la variable `PROXY_PORT` que tiene la forma `<container-port>:<host-port>`. Así me basta el siguiente código en mi contenedor node:

```js
const  port = process.env.PROXY_PORT 
  ? process.env.PROXY_PORT.substr(0, process.env.PROXY_PORT.indexOf(':'))
  : (process.env.PORT || 3000);
```

Con esa salida, ahora cada réplica de mi contenedor abre el puerto correspondiente (si existe la variable `PROXY_PORT` que nos pasa Tye usa dicha variable y en caso de no existir hace _fallback_ a la variable `PORT`). Ahora sí que la magia es completa:

* Llamando a `localhost:3000` me responde una de las réplicas (la que Tye elija, que parece que hace round robin)
* Llamando a `localhost:51478` me responde la primera réplica (ya que es su propio puerto). Si uso `localhost:51477` me responde siempre la segunda y si uso `localhost:51476` siempre la tercera. Pero **recuerda que esos puertos propios de cada réplica cambian a cada ejecución, el puerto importante es el definido en el _binding_, eso es el `3000`**. Ese es el puerto que Tye inyecta en las variable de entorno de los otros procesos, por si esos deben llamar al contenedor de node:

```
SERVICE__NODECLIENT__HOST=host.docker.internal
SERVICE__NODECLIENT__PORT=3000
```

## Bindings y cadenas de conexión

Un escenario típico en Tye es levantar la infrastructura usando contenedores. Ya levantes un Redis, un Mongo, un SQL Server o un RabbitMQ al final necesitarás algún tipo de cadena de conexión para conectarte con él. Tye es agnóstico de la cadena de conexión que definas, pero tiene algunas pequeñas ayudas. Por ejemplo, si quisieras levantar un SQL Server, tendrías lo siguiente en tu `tye.yaml`:

```yaml
- name: sqlserver
  image: mcr.microsoft.com/mssql/server:2019-latest
  bindings:
    - port: 1433
      connectionString: "Server=${host},${port};User Id=sa;Password=Pass@word1"
  env:
    - name: ACCEPT_EULA
      value: "Y"
    - name: SA_PASSWORD
      value: "Pass@word1" 
```

Observa la definición del _binding_. Por un lado indicamos que el contenedor escucha en el `1433`, por lo que al levantar eso, en `localhost:1433` estará escuchando nuestro servidor. Pero el _binding_ define también una cadena de conexión. Esta cadena de conexión me la inyectará Tye en los demás servicios (como variable de entorno):

```
CONNECTIONSTRINGS__SQLSERVER=Server=host.docker.internal,1433;User Id=sa;Password=Pass@word1
```

Que la variable de entorno tenga ese nombre (`CONNECTIONSTRINGS__SQLSERVER`) no es casual, eso nos permite leerla fácilmente desde netcore usando [GetConnectionString](https://docs.microsoft.com/en-us/dotnet/api/microsoft.extensions.configuration.configurationextensions.getconnectionstring?view=dotnet-plat-ext-3.1&viewFallbackFrom=net-5.0&WT.mc_id=AZ-MVP-4039791). Desde cualquier otro servicio puedo leerla sabiendo que el nombre es ese (`CONNECTIONSTRINGS__<NOMBRE-SERVICIO>`). Observa además como en la variable de entorno inyectada **se ha sustituído `${host}` por el host del servicio y `${port}` por el puerto**.

## Integración con Dapr

¿Conoces [Dapr](https://dapr.io/)? Tengo pendiente ir hablando de él en el blog, pero se trata de un _runtime_ de ejecución de aplicaciones distribuídas que permite desarrollar de forma portable e independiente de la tecnología ciertos aspectos de aplicaciones distribuídas. Dapr usa el [patrón sidecar](https://docs.microsoft.com/en-us/azure/architecture/patterns/sidecar?WT.mc_id=AZ-MVP-4039791), por lo que hay un proceso de Dapr ejecutándose y asignado a cada servicio. Este proceso de Dapr se comunica mediante el servicio usando HTTP o gRPC y es el que le da acceso a las funcionalidades ofrecidas por Dapr. En Kubernetes eso se traduce en que cada _pod_ termina ejecutando dos contenedores (el del servicio y el de Dapr). En local se traduce en que para iniciar tu aplicación debes hacerlo a través del ejecutable de dapr (pones en marcha dapr y dapr pone en marcha tu proceso). Eso entra en contradicción con Tye, ya que para empezar, en el caso de netcore en Tye no le indicamos procesos si no simples proyectos (Tye los compila y los lanza). Por suerte Tye se integra con Dapr, por lo que sin tener que hacer apenas nada, podemos indicar a Tye que lance nuestros proyectos netcore usando Dapr.

A grandes rasgos basta con añadir lo siguiente al fichero `tye.yaml`:

```yaml
extensions:
- name: dapr
  components-path: ./dapr-components
```

Con eso añadimos la extensión de dapr y le indicamos en qué directorio están los YAMLs que definen los componentes que Dapr debe usar. Los componentes en Dapr son la infraestructura subyacente que Dapr usa. Por ejemplo, si queremos pub/sub bajo Dapr, entonces necesitamos alguna infraestructura (p. ej. un Redis, o un SQS o un Service Bus). Qué infraestructura usar y su configuración se define en esos YAML. Luego Dapr nos abstrae completamente (ya que pasamos a usar la API de Dapr, no la de la infraestructura). Un tema a tener importante es que Tye solo lanza nuestros procesos, **no se encarga de que la infraestructura necesaria se esté ejecutando en mi máquina**. Si es necesario debo añadir dicha infraestructura en mi fichero `tye.yaml` o bien tenerla en marcha ya en mi máquina.

¡Y poco más! Lo dejo por aquí por el momento. Espero que os haya sido interesante y que os animéis a probar Tye, porque yo pienso que es una herramienta con un futuro prometedor. Y si, en un futuro, Visual Studio entiende de Tye y nos ofrece una experiencia de _F5_ similar a la que ofrece con Compose, ¡entonces ya sería la bomba!

En otro post veremos **como desplegar a Kubernetes desde Tye** :)
 
Saludos!





