---
title: Usar vscode para depurar tus contenedores netcore
description: Usar vscode para depurar tus contenedores netcore
author: eiximenis

date: 2018-06-15T11:14:24+00:00
geeks_url: /?p=2114
geeks_ms_views:
  - 902
categories:
  - asp.net core
  - depuracion
  - docker

---
Una de las ventajas que tiene **Visual Studio 2017 es el [soporte de depuración para contenedores netcore][1]. **A partir de la versión 15.7 el soporte está relativamente maduro soportando algunos escenarios que daban errores en versiones anteriores (p. ej. dos servicios compose usando la misma imagen).
  
Pero... ¿y si no podemos/queremos usar Visual Studio? ¿Tenemos alguna alternativa? Pues sí: **usar visual studio code**, y aunque el _workflow_ no es tan sencillo como en Visual Studio, al final se puede conseguir algo similar: ejecutar y depurar un contenedor. ¡Vamos allá!
  
<!--more-->


  
Lo primero es, por supuesto, partir de un proyecto netcore. **Vamos a hacerlo todo desde cero**. En mi post uso .net core 2.1, pero en .net core 2.0 es casi lo mismo, salvo que en el _Dockerfile_ cambiarían las imágenes base por supuesto. Es interesante (aunque no imprescindible) tener instalada la [extensión de Docker para Visual Studio Code][2].
  
Lo primero es crear el proyecto netcore que vamos a usar, en este caso una API REST usando netcore. Vamos a ver el ejemplo con una API REST aunque es extrapolable al resto de proyectos. Así lo primero es situarnos en una carpeta vacía y usar &#8220;dotnet new webapi -n MyApi&#8221; para crear el API REST llamada &#8220;MyApi&#8221;. Ahora ya podemos usar code y abrir la carpeta raíz. Si tienes [la extensión de C# instalada][3] (¿la tienes, verdad?) code te pedirá de añadir los assets necesarios para compilar y ejecutar el proyecto y te creará una carpeta .vscode con los ficheros necesarios para depurar el proyecto en local.
  
**Importante:** Si usas netcore2.1 **comenta la siguiente línea de Startup.cs**:

<pre class="EnlighterJSRAW" data-enlighter-language="csharp">app.UseHttpsRedirection();</pre>

Esta línea fuerza siempre la redirección de http a https (además del uso de hsts en entornos productivos), pero eso nos dará más quebraderos de cabeza. En un futuro post veremos como podemos depurar nuestros contenedores que usen https, pero por simplicidad, en este primero no vamos a verlo.
  
Bueno, perfecto, podemos ejecutar y depurar la aplicación en vscode, eso no es nada nuevo. Ahora el siguiente paso es crear un Dockerfile y un fichero compose. Lo primero es crear el Dockerfile, que tenemos que hacerlo a mano (la extensión de Docker de vscode permite crear Dockerfiles, pero no soporta ni [multi-staging][4] ni netcore 2).
  
Así que nada, lo creamos a mano. Empezamos por un multi-stage tradicional de netcore:

<pre class="EnlighterJSRAW" data-enlighter-language="null">ARG config=Debug
FROM microsoft/dotnet:2.1-aspnetcore-runtime AS base
WORKDIR /app
EXPOSE 80
FROM microsoft/dotnet:2.1-sdk AS restore
WORKDIR /src
COPY MyApi.csproj .
RUN dotnet restore -nowarn:msb3202,nu1503
FROM restore as build
ARG config
COPY . .
RUN dotnet restore
RUN dotnet build --no-restore -c $config -o /app
FROM build AS publish
ARG config
RUN dotnet publish --no-restore -c $config -o /app
FROM base AS final
WORKDIR /app
COPY --from=publish /app .
ENTRYPOINT ["dotnet", "MyApi.dll"]
</pre>

Y el correspondiente fichero compose:

<pre class="EnlighterJSRAW" data-enlighter-language="json">version: '3.4'
services:
  api:
    image: myapi
    build:
      context: MyApi
      dockerfile: Dockerfile
    ports:
      - 5000:80</pre>

(En mi caso, el fichero &#8220;docker-compose.yml&#8221; está en la carpeta raíz):
  
Bien, ahora podemos poner en marcha y parar nuestro contenedor, usando compose. Veamos ahora como enchufarlo a **vscode**.
  
**Descargar el depurador remoto para Linux**
  
Claro, necesitamos el depurador remoto de netcore, para poder depurar nuestros contenedores. **Necesitamos la versión de Linux ya que nuestros contenedores son Linux**.
  
Si tienes Visual Studio 2017 quizás lo tengas en tu carpeta de usuario. En mi caso lo tengo en **c:\users\etoma\vsdbg\vs2017u5** ya que visual studio lo ha descargado allí.
  
Pero bueno, ningún problema, vamos a descargarlo en una ubicación conocida. Hay dos versiones del depurador remoto, la &#8220;antigua&#8221; llamada clrdbg y la nueva, llamada vsdbg. Vamos a descargarnos esa última:

<pre class="EnlighterJSRAW" data-enlighter-language="null">curl -sSL https://aka.ms/getvsdbgsh | bash /dev/stdin -v vs2017u5 -l ~/vsdbg</pre>

Efectivamente, las instrucciones son para linux, así que necesitas uno (https://aka.ms/getvsdbgsh se descarga un fichero .sh). Por supuesto podemos usar el WSL en Windows 10, así que nada. Abre un terminal WSL, vete a ~ y ejecutalo:
  
[<img class="alignnone size-medium wp-image-2115" src="https://geeks.ms/etomas/wp-content/uploads/sites/154/2018/06/instalar-vsdbg-300x109.png" alt="Usar WSL para instalar vsdbg" width="300" height="109" />][5]
  
Bien, ahora solo debes copiar el contenido de ~/vsdbg a tu perfil de usuario de windows, usando (usa tu perfil de usuario, claro):

<pre class="EnlighterJSRAW" data-enlighter-language="null">cp vsdbg/ /mnt/c/Users/etoma/vsdbg-core -r</pre>

Y ahora en tu perfil de usuario de Windows tendrás la carpeta vsdbg-core con el depurador remoto.
  
**&#8220;Simular&#8221; lo que hace VS2017**
  
Para depurar contenedores VS2017 lo que hace es _tunear_ los contenedores (a partir de un docker-compose que gestiona él) y añadir algunos volúmenes (el código fuente, el depurador remoto y la cache de nuget). Nosotros podemos simular lo mismo &#8220;a mano&#8221;. Así, create otro fichero compose y llámalo _docker-compode.debug.yml_ con el siguiente contenido:

<pre class="EnlighterJSRAW" data-enlighter-language="json">version: '3.4'
services:
  api:
    image: myapi:dev
    build:
      target: base
    labels:
      - "com.microsoft.visualstudio.targetoperatingsystem=linux"
    environment:
      - ASPNETCORE_ENVIRONMENT=Development
      - DOTNET_USE_POLLING_FILE_WATCHER=1
    volumes:
      - .:/app
      - ~/.nuget/packages:/root/.nuget/packages:ro
      - ~/vsdbg-core:/vsdbg:ro
    entrypoint: tail -f /dev/null
</pre>

Este fichero compose intenta simular lo que hace VS2017:

  * Forzamos el tag _dev_ en la imagen
  * **Importante:** Indicamos que el _stage_ a usar es base. **Eso hace que NO SE ejecute ninguno de los otros stages del Dockerfile.** Observa que el stage &#8220;base&#8221; básicamente parte de la imagen de runtime de netcore
  * Añadimos variables de entorno para forzar el entorno de desarrollo y usar el file watcher
  * Dado que partimos del stage &#8220;base&#8221; que es, básicamente, la imagen del runtime de netcore usamos volúmenes para 
      * Mapear el directorio local con el código al directorio /app del contenedor
      * Mapear el directorio de cache de paquetes de nuget local al directorio de la cache de paquetes de nuget del contenedor
      * Y por último, mapeamos el directorio donde tenemos el depurador remoto de netcore a un directorio del contenedor.
  * Finalmente establecemos _tail -f /dev/null_ como entrypoint del cotnenedor. Esto lo que hace es que &#8220;el contenedor se espera eternamente&#8221;.

Recapitulemos: si usamos este nuevo compose, tendremos un contenedor que es básicamente la imagen del runtime de netcore, con los ficheros de código fuente mapeados, con el depurador remoto instalado y ejecutándose _ad-eternum_.
  
Puedes verificar que todo funciona usando:

<pre class="EnlighterJSRAW" data-enlighter-language="null">docker-compose -f docker-compose.yml -f docker-compose.debug.yml up -d</pre>

Y luego con un &#8220;docker ps&#8221; deberías ver el contenedor _myapi:dev. _Si abres una sesión interactiva contra él (_docker exec -it <id-contenedor> /bin/bash_) verás los ficheros en él:
  
[<img class="alignnone size-medium wp-image-2116" src="https://geeks.ms/etomas/wp-content/uploads/sites/154/2018/06/listado-ficheros-contenedor-300x70.png" alt="" width="300" height="70" />][6]
  
Genial. Nuestro contenedor de depuración está listo. Vamos a configurar code para usarlo y enchufarse a él 🙂
  
Para ello debemos editar el fichero .vscode/launch.json y añadir una nueva entrada:

<pre class="EnlighterJSRAW" data-enlighter-language="json">{
    "name": ".NET Core Attach (docker)",
    "type": "coreclr",
    "request": "launch",
    "preLaunchTask": "build",
    "program": "/app/MyApi/bin/Debug/netcoreapp2.1/MyApi.dll",
    "args": [],
    "cwd": "/app/MyApi",
    "stopAtEntry": false,
    "console": "internalConsole",
    "pipeTransport": {
        "pipeCwd": "${workspaceRoot}",
        "pipeProgram": "docker",
        "pipeArgs": [
            "exec", "-i", "dockercode_api_1" ,"${debuggerCommand}"
         ],
         "quoteArgs": false,
        "debuggerPath": "/vsdbg/vsdbg"
    }
},</pre>

Comento las partes más importantes:

  * _program_: Programa a ejecutar **dentro del contenedor**
  * _cwd_: Directorio de ejecución **dentro del contenedor**
  * _pipeTransport_: Las opciones para usar docker 
      * _pipeProgram:_ Indicamos que usaremos docker
      * _pipeArgs_: Array de argumentos que le pasamos a docker. Básicamente es ejecutar el depurador remoto, en el contenedor. **Observa que tenemos el nombre del contenedor** (dockercode\_api\_1). La variable $debuggerCommand la pasa Visual Studio Code y contiene el comando para lanzar el depurador remoto.
      * _quoteArgs_: Ponlo a false, si no lo pones recibirás errores de sintaxis y/o directorios no encontrados
      * _debuggerPath_: Ubicación del ejecutable_ __vsdbg_ en el contenedor

Ahora ya podemos probarlo. Para ello debemos:

  1. **Ejecutar el docker-compose up_ _**desde una terminal. Esto crea el contenedor que, recuerda, se queda levantado _ad-eternum_ o sea que te puedes olvidar de él.
  2. Para depurar selecciona la opción &#8220;.NET Core Attach (docker)&#8221; y listos. Visual Studio code compilará tu proyecto y lo depurará en el contenedor.

Hay dos problemas con esta aproximación, uno poco importante y otro más chungo que no he podido resolver.
  
El primero es que necesitas una definición en launch.json para cada contenedor, ya que el nombre de este está a fuego. No hay manera (que yo sepa) de que vscode te &#8220;pregunte&#8221; por qué contenedor quieres depurar (recuerda que, básicamente, son todos ellos idénticos).
  
Vale y ahora... **el segundo problema**
  
**No he podido conseguir  hacer funcionar el soporte multi-contenedor 🙁**
  
Pues eso... que no he sido capaz de depurar una solución multi-contenedor (entiéndase eso por depurar N contenedores a la vez). **Os cuento la aproximación que he seguido, el error que me da y el porque creo que es debido**.
  
Mi intención era aprovecharme de las [_compound launch actions_][7] de visual studio code. Para la prueba necesitamos crear otro proyecto y lo llamamos _ApiServer__. _Modificamos el código de _ValuesController_ para que sea:

<pre class="EnlighterJSRAW" data-enlighter-language="csharp">[Route("api/[controller]")]
[ApiController]
public class ValuesController : ControllerBase
{
    // GET api/values
    [HttpGet]
    public IActionResult Get()
    {
        return Ok(new {
            Server = Environment.MachineName,
            Date = DateTime.UtcNow
        });
    }
}</pre>

Ahora modificamos el proyecto api original para que en ValuesController llame a ApiServer usando HttpClient:

<pre class="EnlighterJSRAW" data-enlighter-language="csharp">[Route("api/[controller]")]
[ApiController]
public class ValuesController : ControllerBase
{
    private readonly string _serverUrl;
    public ValuesController(IConfiguration cfg) =&gt; _serverUrl = cfg["server"];
    // GET api/values
    [HttpGet]
    public async Task&lt;IActionResult&gt; Get()
    {
        var client = new HttpClient();
        var response = await client.GetAsync(_serverUrl + "/api/values");
        var json = await response.Content.ReadAsStringAsync();
        dynamic serverData = JObject.Parse(json);
        return Ok(new {
            serverData = serverData,
            client =  Environment.MachineName
            });
    }
}</pre>

Ahora debemos crear un _Dockerfile_ para este segundo proyecto. De hecho es idéntico al _Dockerfile_ anterior, solo cambiando el entrypoint y el COPY del csproj.
  
Luego modificamos el fichero compose para agregar el nuevo servicio y añadir la entrada _server_ que requiere la api:

<pre class="EnlighterJSRAW" data-enlighter-language="csharp">version: '3.4'
services:
  api:
    image: myapi
    build:
      context: MyApi
      dockerfile: Dockerfile
    ports:
      - 5000:80
    environment:
      - server=http://server
  server:
    image: myserver
    build:
      context: ApiServer
      dockerfile: Dockerfile
    ports:
      - 5001:80</pre>

Y, por supuesto, añadimos el servicio _server_ en el _docker-compose.debug.yml_:

<pre class="EnlighterJSRAW" data-enlighter-language="json">version: '3.4'
services:
  api:
    image: myapi:dev
    build:
      target: base
    labels:
      - "com.microsoft.visualstudio.targetoperatingsystem=linux"
    environment:
      - ASPNETCORE_ENVIRONMENT=Development
      - DOTNET_USE_POLLING_FILE_WATCHER=1
    volumes:
      - .:/app
      - ~/.nuget/packages:/root/.nuget/packages:ro
      - ~/vsdbg-core:/vsdbg:ro
    entrypoint: tail -f /dev/null
  server:
    image: myserver:dev
    build:
      target: base
    labels:
      - "com.microsoft.visualstudio.targetoperatingsystem=linux"
    environment:
      - ASPNETCORE_ENVIRONMENT=Development
      - DOTNET_USE_POLLING_FILE_WATCHER=1
    volumes:
      - .:/app
      - ~/.nuget/packages:/root/.nuget/packages:ro
      - ~/vsdbg-core:/vsdbg:ro
    entrypoint: tail -f /dev/null
</pre>

(Efectivamente, ambas entradas son idénticas, solo cambia el nombre de la imagen)
  
Vale, ahora ya podemos configurar vscode. Por un lado necesitamos dos tareas de build, para construir de forma separada ambos proyectos, así que modificamos tasks.json para que quede:

<pre class="EnlighterJSRAW" data-enlighter-language="json">"tasks": [
    {
        "label": "build-api",
        "command": "dotnet",
        "type": "process",
        "args": [
            "build",
            "${workspaceFolder}/MyApi/MyApi.csproj"
        ],
        "problemMatcher": "$msCompile"
    },
    {
        "label": "build-server",
        "command": "dotnet",
        "type": "process",
        "args": [
            "build",
            "${workspaceFolder}/ApiServer/ApiServer.csproj"
        ],
        "problemMatcher": "$msCompile"
    }
]
</pre>

Y ahora necesitamos definir otra tarea de configuración en launch.json para usar el nuevo contenedor. Además necesitamos modificar la primera para que use _build-api_ en lugar de _build_ que ya no tenemos:

<pre class="EnlighterJSRAW" data-enlighter-language="json">"name": "MyApi (docker) ",
    "type": "coreclr",
    "request": "launch",
    "preLaunchTask": "build-api",
    "program": "/app/MyApi/bin/Debug/netcoreapp2.1/MyApi.dll",
    "args": [],
    "cwd": "/app/MyApi",
    "stopAtEntry": false,
    "console": "internalConsole",
    "pipeTransport": {
        "pipeCwd": "${workspaceRoot}",
        "pipeProgram": "docker",
        "pipeArgs": [
            "exec", "-i", "dockercode_api_1" ,"${debuggerCommand}"
         ],
         "quoteArgs": false,
        "debuggerPath": "/vsdbg/vsdbg"
    }
},
{
     "name": "ApiServer (docker) ",
     "type": "coreclr",
     "request": "launch",
     "preLaunchTask": "build-server",
     "program": "/app/ApiServer/bin/Debug/netcoreapp2.1/ApiServer.dll",
     "args": [],
     "cwd": "/app/ApiServer",
     "stopAtEntry": false,
     "console": "internalConsole",
     "pipeTransport": {
         "pipeCwd": "${workspaceRoot}",
         "pipeProgram": "docker",
         "pipeArgs": [
             "exec", "-i", "dockercode_server_1" ,"${debuggerCommand}"
         ],
         "quoteArgs": false,
         "debuggerPath": "/vsdbg/vsdbg"
     }
 },</pre>

Y ahora debemos agregar, en el mismo _launch.json_ la tarea compuesta. Para ello (después del array de _configurations_) añadimos la sección _compounds_:

<pre class="EnlighterJSRAW" data-enlighter-language="null">"compounds": [
    {
        "name": "Both containers",
        "configurations": ["MyApi (docker)", "ApiServer (docker)"]
    }
]</pre>

Si ahora lanzamos la tarea _&#8220;Both Containers&#8221; _eso **nos debería permitir** depurar ambos contenedores a la vez, pero no: **recibiremos un error**:
  
[<img class="alignnone size-medium wp-image-2119" src="https://geeks.ms/etomas/wp-content/uploads/sites/154/2018/06/error-debug-300x39.png" alt="Error: OCI runtime exec failed: exec failed: container_linux.go:348: starting container process caused &quot;no such file or directory&quot;: unknown" width="300" height="39" />][8]
  
Se puede ver como **intenta ambos docker exec** pero recibe un error. Por supuesto, si lanzas cualquiera de las sesiones de forma independiente funciona. Pero si las lanzas a la vez, falla.
  
**Lo que (<span style="text-decoration: underline;">creo</span>) que pasa es que la primera sesión captura stdin/stdout** del terminal de code, y cuando la segunda sesión intenta usar stdin/stdout (para comunicarse via code con el _pipe_) le da un error.
  
Lo que se me ha ocurrido (muy listo yo xD) ha sido decirle a code que **lance un terminal externo para cada sesión, **de este modo cada sesión tendrá un stdin/stdout propio y no debería haber problemas, ¿verdad? Así que añadí la opción:

<pre class="EnlighterJSRAW" data-enlighter-language="json">"console": "externalTerminal"</pre>

A ambas tareas para que así cada una tenga su propio terminal. Pero **no funciona: la opción _console _es ignorada si se usa pipeTransport**. Mi gozo en un pozo. He visto algunas issues al respecto:

  * https://github.com/Microsoft/vscode/issues/49941 (donde le dicen que eso es de una extensión y que lo diga allí)
  * https://github.com/OmniSharp/omnisharp-vscode/issues/2313: La issue correspondiente en la extensión de Omnisharp (la que gestiona la depuración de netcore)

No he dado con ninguna solución a este problema **y no creo que sea abordable sin modificar la extensión de OmniSharp,** cosa que desconozco si es posible...
  
Si alguien encuentra o sabe una solución a este problema: ¡que ponga un comentario! Yo, al menos, le estaré muy agradecido 🙂
  
&nbsp;

 [1]: https://geeks.ms/etomas/2017/07/27/las-herramientas-de-docker-de-vs2017/
 [2]: https://marketplace.visualstudio.com/items?itemName=PeterJausovec.vscode-docker
 [3]: https://marketplace.visualstudio.com/items?itemName=ms-vscode.csharp
 [4]: https://geeks.ms/etomas/2017/11/28/docker-multi-stage-builds-o-como-compilar-casi-cualquier-cosa-sin-tener-que-instalar-nada/
 [5]: https://geeks.ms/etomas/wp-content/uploads/sites/154/2018/06/instalar-vsdbg.png
 [6]: https://geeks.ms/etomas/wp-content/uploads/sites/154/2018/06/listado-ficheros-contenedor.png
 [7]: https://code.visualstudio.com/Docs/editor/debugging#_multitarget-debugging
 [8]: https://geeks.ms/etomas/wp-content/uploads/sites/154/2018/06/error-debug.png