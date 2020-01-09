---
title: Gestionar la configuración en asp.net core y Docker
author: eiximenis

date: 2018-01-31T12:40:32+00:00
geeks_url: /?p=2001
geeks_ms_views:
  - 1128
categories:
  - asp.net core
  - docker

---
Cuando desarrollamos una aplicación en asp.net core que queremos terminar ejecutando con Docker, el como gestionar la configuración puede causarnos más de un quebradero de cabeza.
  
En este post voy a comentar, brevemente, como podemos gestionar varios escenarios de configuración.
  
<!--more-->


  
Un tema que me gusta conseguir **es poder ejecutar mi aplicación en localhost sin usar Docker**. Es decir directamente en Kestrel/IIS Express. Es cierto que con VS2017 puedo lanzar y depurar vía F5 el contenedor Docker pero, por un lado no siempre tengo VS2017 a mano y por otra VS ejecuta siempre todo el fichero compose, lo que en soluciones multi-proyecto es muy lento. Si p. ej. tienes una solución con 5 APIs, VS2017 te levantará las 5 cuando levante el fichero compose. A veces me interesa levantar solo una. **Eso es impedimento para ejecutar la infraestructura vía compose, claro.** Es decir, ¿para qué voy a usar LocalDb si puedo tener un contenedor de Docker con Sql Server? Cambiad LocalDb por cualquier otra infrastructura tipo Redis, MongoDb, RabbitMq y similares.
  
Ejecutar mi aplicación sin Docker es un requisito. El otro es, por supuesto, poder ejecutarla en Docker vía VS y también claro está, en Docker vía la CLI. El resultado no es el mismo, ya que las imágenes que se montan vía VS y via la CLI son radicalmente distintas.
  
**1. Gestión de secretos**
  
Vale, a lo que iba, en ASP.NET Core tenemos [**los developer secrets**][1], que funcionan editando un fichero json que está fuera del directorio de código fuente (por lo que nunca se incluye en el repo). Luego el método _AddUserSecrets_ de _ConfigurationBuilder _puede leer estos secretos. **Por supuesto eso solo funciona en TU máquina de desarrollo**. Total, el fichero de secretos está en un directorio local (en windows cuelga de %APPDATA%).
  
**Problema:** Si ejecutas sin Docker todo te funcionará, pero con Docker no te funcionará, con independencia de como generes las imágenes. El motivo es que el fichero de secretos no forma parte de la imagen. Por lo tanto, cuando ejecutes via Docker debes proporcionar el valor de estos secretos en otra parte. Antes de las _Docker Tools_ eso no tenía mucha importancia, porque &#8220;no desarrollabas ni depurabas contra un contenedor Docker&#8221;. Es decir, desarrollabas siempre sin Docker y ya luego, creabas las imágenes de Docker y hacías los tests necesarias.
  
Una solución, a lo bruto, es montar en el fichero compose el directorio donde está el fichero de secretos a un volúmen de tu contenedor. Si además en el contenedor este volúmen lo montas donde se supone que estaría el fichero de secretos, entonces todo te funcionará. Así si desarrollas en Windows pero usas contenedores Linux deberías montar %APPDATA%/usersecrets/<userSecretsId> al directorio ~/.microsoft/usersecrets/<userSecretsId> del contenedor (donde <userSecretsId> es el ID de secretos de tu proyecto (lo puedes ver abriendo el csproj y buscando la etiqueta  <span class="hljs-tag"><em><<span class="hljs-name">UserSecretsId</span>> </em>(más contexto en <a href="https://github.com/Microsoft/DockerTools/issues/24">esta issue de github</a>).</span>
  
Esto funciona pero hay un problema: **la ubicación del fichero de secretos puede canviar en futuras versiones** (tanto su ubicación como su formato). Por lo tanto eso, a pesar de que funciona, no es recomendable y va en contra de las prácticas recomendadas.
  
Si no quieres depender de este volúmen, entonces la otra opción **es tener un fichero compose que establezca esos secretos**. El problema: **si agregas este fichero al repositorio de código fuente estás agregando los secretos**. En algunos secretos eso no importa (p. ej. cadenas de conexión a una bbdd local, o a un redis local, etc) pero en muchos otros sí (cadenas de conexión a bbdd de desarrollo en el cloud, tokens de apis, etc). Así que esto es peligroso. Recuerda, **a pesar de que el repo sea privado deberíamos evitar siempre meter secretos en él** (al menos los secretos sensibles).
  
**También puedes editar el fichero $(SolutionDir)/obj/Docker/docker-compose.vs.debug.g.yml** (donde $(SolutionDir) es el directorio de la solución). Este fichero NO se incluye en el control de código fuente y VS lo usa cuando levanta los contenedores. **El problema es que entonces cuando uses compose desde la CLI no te va a funcionar** (a no ser que incluyas este fichero, cosa que no se suele hacer porque cambia como se generan las imágenes).
  
Otra opción disponible sería **usar la [opción _secrets_ del fichero compose][2]**, pero tiene un par de problemas: el principal es que solo funciona con servicios _swarm_ no con contenedores normales y el segundo es que no hay una manera rápida de leer esos secretos desde ASP.NET Core: había un paquete llamado _Microsoft.Extensions.Configuration.DockerSecrets_ pero ya no está, así que supongo que al final lo han abandonado. Si te apetece ver como era [hay disponible una versión &#8220;no oficial&#8221;][3].
  
En fin, que no hay una &#8220;opción ideal&#8221; ahora mismo (al menos que yo sepa), así que recuerda: si usas _developer secrets_ y las Docker tools, cuando ejecutes tus contenedores vía Docker no te funcionará directamente.
  
**2. Datos de configuración **
  
Vale, los datos de configuración que no son secretos no dan tantos problemas, porque esos sí que pueden estar en el repo. En general lo que se sigue ahí es tenerlos en un fichero _appsettings_ y redefinirlos en el fichero compose siempre que sea necesario. Hay una cosa que cambia cuando ejecutas con Docker o sin Docker y es las direcciones (DNS, IP) de otros servicios. P. ej. si tengo la Sql Server ejecutándose en Docker, con el puerto 1433 redirigido al 5433 del host, entonces:

  * Cuando ejecute desde Docker el servidor es **nombre-servicio_,1433_** (donde nombre-servicio es el nombre definido por el fichero compose).
  * Cuando ejecute sin Docker el servidor es **_.,5433_**

Observa que en este caso a pesar de ser una cadena de conexión no es realmente &#8220;un secreto&#8221; ya que es una cadena de conexión a una BBDD &#8220;local&#8221;. Pero vamos hay otros casos. Si tu servicio llama a otro servicio, cuando estés en Docker la URL del otro servicio será _http://localhost:<puerto>_, y cuando estrés en Docker será _http://<nombre-otro-servicio>_ (generalmente al usar Docker expones todos tus servicios por el puerto 80, ya que tienes un 80 por cada servicio).
  
En este caso, yo he adoptado la siguiente política (habría muchas otras):

  1. Creo un fichero &#8220;appsettings.localhost.json&#8221; con todos esos datos que canvian al ejecutar sin Docker a ejecutar con Docker. Este fichero contiene los valores &#8220;de ejecutar sin Docker&#8221;.
  2. En el fichero compose redefino esta configuración (a través de la sección _environment_ que añade variables de entorno).
  3. Añado el fichero &#8220;appsettings.localhost.json&#8221; al fichero .dockerignore **para asegurar que este fichero nunca se añade a ninguna imágen**

En el fichero _appsettings.json_ mantengo aquello que es igual tanto sin Docker como con Docker (y lo mismo en _appsettings.{environment}.json_ pero para el entorno, aunque ojo, no lo uses para distinguir Dev, de QA p. ej. ya que esas diferencias de configuración deberían ir en configuración de compose en el caso de Docker).
  
A pesar de (3) cuando ejecutas las imágenes vía VS con F5, el fichero estará en la imagen (porque VS monta un volúmen con todo el contenido del proyecto), así que **debes asegurarte de que lees este fichero ANTES de leer las variables de entorno**). De este modo las variables de entorno sobreescriben los valores de este fichero (cuando ejecutas con Docker).
  
Si usas _WebHost.CreateDefaultBuilder_ (lo habitual en aplicaciones netcore2) ten presente que este método añade las siguientes fuentes de configuración en este orden:

  1. Diccionario en memoria
  2. Fichero _appsettings.json_
  3. Fichero _appsettings.{environment}.json_
  4. Las _variables de entorno_
  5. Los parámetros de línea de comandos

**Por lo tanto el siguiente código no funciona:**

<pre class="EnlighterJSRAW" data-enlighter-language="null">WebHost.CreateDefaultBuilder(args)
    .ConfigureAppConfiguration(ic =&gt; ic.AddJsonFile("appsettings.localhost.json", true))
    .UseStartup&lt;Startup&gt;()
    .Build();</pre>

El problema es que **el fichero appsettings.localhost.json** es añadido **al final y por lo tanto después de las variables de entorno**, por lo que incluso al ejecutar con Docker usarás los valores de este fichero (lo que no quieres).
  
La solución **es insertar este fichero ANTES de las variables de entorno**. Por suerte eso es posible:

<pre class="EnlighterJSRAW" data-enlighter-language="null">WebHost
    .CreateDefaultBuilder(args)
    .ConfigureAppConfiguration(cb =&gt;
    {
        var sources = cb.Sources;
        sources.Insert(3, new Microsoft.Extensions.Configuration.Json.JsonConfigurationSource()
        {
            Optional = true,
            Path = "appsettings.localhost.json",
            ReloadOnChange = false
        });
    })
    .UseStartup&lt;Startup&gt;()
    .Build();</pre>

En este caso **se inserta en tercera posición** los datos del fichero _appsettings.localhost.json_, esto es **justo ANTES de las variables de entorno**. Por lo que esto te funcionará en todos los casos:

  * Sin Docker se usarán los valores de _appsettings.localhost.json_
  * Con Docker (via VS) se usarán los valores del fichero compose, a pesar de que el fichero _appsettings.localhost.json_ está en la imagen
  * Con Docker (via CLI) se usarán los valores del fichero compose. El fichero _appsettings.localhost.json_ ni aparecerá en la imagen (asumiendo que lo has añadido en .dockerignore)

Finalmente, un apunte: observa que este último código depende de que WebHost.CreateDefaultBuilder agregue las variables de entorno en (como mínimo) la tercera posición. Eso es lo que hace actualmente, pero podría llegar a cambiar en futuras versiones, así que ojo con eso. Para curarse en salud se debería hacer un código más robusto, p. ej. encontrar la posición del objeto cuyo tipo es _EnvironmentVariablesConfigurationSource_ e insertar en esta posición (esto te asegura insertar justo antes de las variables de entorno).
  
Saludos!

 [1]: https://blogs.msdn.microsoft.com/mihansen/2017/09/10/managing-secrets-in-net-core-2-0-apps/
 [2]: https://docs.docker.com/engine/swarm/secrets/
 [3]: https://github.com/RehanSaeed/Microsoft.Extensions.Configuration.DockerSecrets.Unofficial