---
title: Ejecutar pruebas de tus contenedores en VSTS
author: eiximenis

date: 2018-03-07T12:33:37+00:00
geeks_url: /?p=2013
geeks_ms_views:
  - 1601
categories:
  - asp.net core
  - docker
  - Sin categoría

---
Si desarrollas con Docker es probable que uses [_multi-stage builds_][1] para crear tus contenedores, en este caso unificas bajo un mismo Dockerfile la creación del binario (usando una imagen de compilación) y la creación de la imagen final (basandote en una imagen de _runtime_).
  
Ahora bien, si usas un pipeline de CI/CD con VSTS... ¿como gestionar los tests de esos contenedores? Eso es lo que vamos a discutir en este post.
  
<!--more-->


  
Si tu sistema se compone **de un único contenedor** en este caso lo más normal puede ser **ejecutar las pruebas como parte del proceso de construcción** de la imagen final. Es decir, ejecutar las pruebas en el Dockerfile.
  
Para ello en el proceso de _multi-stage_ simplemente cuando usas la imagen de _build_ ejecutas los tests como un paso más de construcción de la imagen final. Veamos un ejemplo básico de ello. Partamos de dos proyectos: una aplicación web y uno de tests unitarios:
  
[<img class="alignnone size-full wp-image-2014" src="https://geeks.ms/etomas/wp-content/uploads/sites/154/2018/03/solution.png" alt="Esquema de la solución" width="220" height="287" />][2]
  
No nos preocupemos por ahora de lo que hace la aplicación web. Por su parte el test unitario prueba el comportamiento del servicio &#8220;GuidProvider&#8221; y es como sigue:

<pre class="EnlighterJSRAW" data-enlighter-language="csharp">public class guid_provider_should
{
    [Fact]
    public void never_return_a_empty_guid()
    {
        var provider = new GuidProvider();
        var id = provider.Id;
        id.Should().NotBe(Guid.Empty, "Empty guid can't be returned");
    }
}</pre>

Bien, vamos ahora a crear un Dockerfile con _multi-stage_ que me construya la imagen final de WebApplication1 y a la vez que me ejecute los tests:

<pre class="EnlighterJSRAW" data-enlighter-language="raw">FROM microsoft/aspnetcore:2.0 AS base
WORKDIR /app
EXPOSE 80
FROM microsoft/aspnetcore-build:2.0 AS build
WORKDIR /src
COPY CiCd.sln ./
COPY WebApplication1/WebApplication1.csproj WebApplication1/
COPY WebApplication1.Tests/WebApplication1.Tests.csproj WebApplication1.Tests/
RUN dotnet restore -nowarn:msb3202,nu1503
COPY . .
WORKDIR /src/WebApplication1
RUN dotnet build -c Release -o /app
FROM build as test
WORKDIR /src/WebApplication1.Tests
RUN dotnet test
FROM build AS publish
WORKDIR /src/WebApplication1
RUN dotnet publish -c Release -o /app
FROM base AS final
WORKDIR /app
COPY --from=publish /app .
ENTRYPOINT ["dotnet", "WebApplication1.dll"]</pre>

Este Dockerfile necesita como contexto de build de Docker el directorio donde está la solución, así pues colocado en este directorio puedes construir la imagen  ejecutando:

<pre class="EnlighterJSRAW" data-enlighter-language="shell">docker build -t webapplication1 . -f WebApplication1\Dockerfile</pre>

En este caso el test unitario falla (hay un error en GuidProvider y siempre devuelve Guid.Empty) así que _docker build_ fallará con una salida parecida a:

<pre class="EnlighterJSRAW" data-enlighter-language="raw">Step 15/22 : RUN dotnet test
 ---&gt; Running in 776cb8501892
Build started, please wait...
Build completed.
Test run for /src/WebApplication1.Tests/bin/Debug/netcoreapp2.0/WebApplication1.Tests.dll(.NETCoreApp,Version=v2.0)
Microsoft (R) Test Execution Command Line Tool Version 15.5.0
Copyright (c) Microsoft Corporation.  All rights reserved.
Starting test execution, please wait...
[xUnit.net 00:00:01.3504231]   Discovering: WebApplication1.Tests
[xUnit.net 00:00:01.4152529]   Discovered:  WebApplication1.Tests
[xUnit.net 00:00:01.4223592]   Starting:    WebApplication1.Tests
[xUnit.net 00:00:01.7516805]     WebApplication1.Tests.guid_provider_should.never_return_a_empty_guid [FAIL]
[xUnit.net 00:00:01.7545026]       Did not expect id to be {00000000-0000-0000-0000-000000000000} because Empty guid can't be returned.
[xUnit.net 00:00:01.7553789]       Stack Trace:
[xUnit.net 00:00:01.7561760]         C:\projects\fluentassertions-vf06b\Src\FluentAssertions\Execution\XUnit2TestFramework.cs(32,0): at FluentAssertions.Execution.XUnit2TestFramework.Throw(String message)
[xUnit.net 00:00:01.7562630]         C:\projects\fluentassertions-vf06b\Src\FluentAssertions\Execution\AssertionScope.cs(224,0): at FluentAssertions.Execution.AssertionScope.FailWith(String message, Object[] args)
[xUnit.net 00:00:01.7563084]         C:\projects\fluentassertions-vf06b\Src\FluentAssertions\Primitives\GuidAssertions.cs(125,0): at FluentAssertions.Primitives.GuidAssertions.NotBe(Guid unexpected, String because, Object[] becauseArgs)
[xUnit.net 00:00:01.7563496]         /src/WebApplication1.Tests/UnitTests.cs(15,0): at WebApplication1.Tests.guid_provider_should.never_return_a_empty_guid()
[xUnit.net 00:00:01.7751963]   Finished:    WebApplication1.Tests
Failed   WebApplication1.Tests.guid_provider_should.never_return_a_empty_guid
Error Message:
 Did not expect id to be {00000000-0000-0000-0000-000000000000} because Empty guid can't be returned.
Stack Trace:
   at FluentAssertions.Execution.XUnit2TestFramework.Throw(String message) in C:\projects\fluentassertions-vf06b\Src\FluentAssertions\Execution\XUnit2TestFramework.cs:line 32
   at FluentAssertions.Execution.AssertionScope.FailWith(String message, Object[] args) in C:\projects\fluentassertions-vf06b\Src\FluentAssertions\Execution\AssertionScope.cs:line 224
   at FluentAssertions.Primitives.GuidAssertions.NotBe(Guid unexpected, String because, Object[] becauseArgs) in C:\projects\fluentassertions-vf06b\Src\FluentAssertions\Primitives\GuidAssertions.cs:line 125
   at WebApplication1.Tests.guid_provider_should.never_return_a_empty_guid() in /src/WebApplication1.Tests/UnitTests.cs:line 15
Total tests: 1. Passed: 0. Failed: 1. Skipped: 0.
Test Run Failed.
Test execution time: 3.1584 Seconds
The command '/bin/sh -c dotnet test' returned a non-zero code: 1</pre>

Ahora veamos como podemos integrar esto en un pipeline de VSTS.
  
Lo primero es crear nuestra _build_ que tenga, por ahora, una sola tarea de tipo &#8220;Docker&#8221;:
  
[<img class="alignnone size-medium wp-image-2015" src="https://geeks.ms/etomas/wp-content/uploads/sites/154/2018/03/build-1-300x122.png" alt="" width="300" height="122" />][3]
  
Es importante que desmarques la casilla de &#8220;Use default build context&#8221;, y dejes el texto &#8220;Build Context&#8221; vacío para que así apunte a la raíz del repo (donde tenemos el fichero _sln_).
  
Una vez encolada esta falla, ya que falla el test unitario. Hasta ahí todo bien **pero no tenemos el resultado de los tests en VSTS**. La pestaña _tests_ está vacía ya que por lo que a VSTS respecta no ha habido ejecución alguna de tests:
  
[<img class="alignnone size-medium wp-image-2016" src="https://geeks.ms/etomas/wp-content/uploads/sites/154/2018/03/build-1-tests-300x101.png" alt="" width="300" height="101" />][4]
  
Vamos a corregir este punto. Para ello debemos cambiar nuestra aproximación **y ejecutar los tests en otro contenedor**.
  
Ejecutar los tests **como parte de construcción de la imagen no es que esté mal pero eso nos va a impedir que VSTS se entere del resultado**. Esto es debido a una &#8220;limitación&#8221; de Docker que no permite tener volúmenes durante &#8220;docker build&#8221; por lo que no podemos compartir el fichero de resultados de test (que podemos generar con _dotnet test_) con VSTS: este fichero se queda en el contenedor intermedio y no podemos sacarlo fácilmente de allí.
  
Por lo tanto vamos a seguir otra aproximación, ligeramente distinta y es **ejecutar los tests en otro contenedor**. Por lo tanto vamos a levantar primero un contenedor de _tests_ y ejecutar los tests en él. Lo bueno es que **vamos a poder usar el mismo Dockerfile** para ambos contenedores. Para ello **lo primero es eliminar del Dockerfile la lína que ejecuta el &#8220;dotnet test&#8221;**, ya que ahora lo ejecutaremos aparte.
  
Con eso se nos guardará el fichero &#8220;test-results.xml&#8221; con el resultado de los tests en el directorio (/tests/) del contenedor. Vale, ahora vamos a aprovecharnos de una característica de _Docker run_ que es poder ejecutar un Dockerfile no hasta el final si no solo hasta un _stage_ determinado. En nuestro caso este _stage_ es el stage de tests:

<pre class="EnlighterJSRAW" data-enlighter-language="shell">docker build -t webapplication1-tests . -f WebApplication1\Dockerfile --target test</pre>

Observa el _&#8211;target test_ que indica que solo debemos construir hasta este _stage_. Observa que la imagen generada se llamara &#8220;webapplication1-tests&#8221;. Esto generará la imagen, y ahora ya podemos ejecutar los tests en ella:

<pre class="EnlighterJSRAW" data-enlighter-language="null">docker run -v/c/tests:/tests  webapplication1-tests --entrypoint "dotnet test --logger trx;LogFileName=/tests/test-results.xml"</pre>

Con este _docker run_ ejecutamos la imagen creada en el paso anterior **y mediante un volúmen mapeamos el directorio /tests/ del contenedor a un directorio del host** (en mi caso _c:\tests\_). Por lo tanto ahora tengo en c:\tests\ los resultados de los tests.
  
Si ahora quiero construir la imagen final puedo hacer otro _docker build_:

<pre class="EnlighterJSRAW" data-enlighter-language="null">docker build -t webapplication1 . -f WebApplication1\Dockerfile</pre>

La **ventaja es que gracias al modelo de capas de Docker no es necesario re-ejecutar todos los otros stages** (es decir, no es necesario recompilar la aplicación).
  
Bien, vamos ahora a aplicar todo esto a VSTS. Para simplificar la build y evitar meter tantos parámetros nos apoyaremos en compose. Para ello crea el fichero _docker-compose.yml_ que tenga el siguiente contenido:

<pre class="EnlighterJSRAW" data-enlighter-language="null">version: '3.4'
services:
  webapplication1:
    image: webapplication1
    build:
      context: .
      dockerfile: WebApplication1/Dockerfile
  webapplication1-tests:
    image: webapplication1-tests
    build:
      context: .
      dockerfile: WebApplication1/Dockerfile
      target: test
    volumes:
      - ${BUILD_ARTIFACTSTAGINGDIRECTORY:-./tests-results/}:/tests</pre>

Aquí definimos las dos imágenes (webapplication1 y webapplication1-tests). Nos falta el fichero _docker-compose.override.yml_:

<pre class="EnlighterJSRAW" data-enlighter-language="null">version: '3.4'
services:
  webapplication1:
    environment:
      - ASPNETCORE_ENVIRONMENT=Development
    ports:
      - "80"
  webapplication1-tests:
    environment:
      - ASPNETCORE_ENVIRONMENT=Development
    ports:
      - "80"
    entrypoint:
      - dotnet
      - test
      - --logger
      - trx;LogFileName=/tests/test-results.xml</pre>

Perfecto, ahora para ejecutar los tests nos basta con:

<pre class="EnlighterJSRAW" data-enlighter-language="null">docker-compose run webapplication1-tests</pre>

Eso ejecuta los tests (y nos deja el XML de salida en el directorio indicado por la variable de entorno BUILD_ARTIFACTSTAGINGDIRECTORY). Y para construir la imagen final nos basta con:

<pre class="EnlighterJSRAW" data-enlighter-language="null">docker-compose build webapplication1</pre>

Ahora ya podemos integrarnos con VSTS. Para ello vamos a crear una build con los siguientes pasos:

  1. Ejecutar _docker-compose run webapplication1-tests_ para ejecutar los tests
  2. Publicar los resultados de los tests
  3. Acciones adicionales (p. ej. construir _webapplication1_ y hacer el push).

Lo interesante es que si **los tests fallan, docker-compose run falla también**, lo que hará que la build sea errónea.
  
Empecemos por la primera tarea, que es una tarea de Docker compose en VSTS configurada como sigue:
  
[<img class="alignnone size-medium wp-image-2019" src="https://geeks.ms/etomas/wp-content/uploads/sites/154/2018/03/build-2-task1-300x162.png" alt="" width="300" height="162" />][5]
  
Las opciones son a configurar son:

  * Docker compose file: docker-compose.yml
  * Additional docker compose files: docker-compose.override.yml
  * Action: &#8220;Run a specific service image&#8221;
  * Service Name: webapplication1-tests

La segunda tarea es del tipo &#8220;Publish Test Results&#8221;:
  
[<img class="alignnone size-medium wp-image-2021" src="https://geeks.ms/etomas/wp-content/uploads/sites/154/2018/03/build-2-task2-300x152.png" alt="" width="300" height="152" />][6]
  
Las opciones a configurar son:

  * Test Result Format: VSTest
  * Test result files: **/test-results.xml
  * Search folder: $(Build.ArtifactStagingDirectory)
  * **Importante: **En Control Options -> Run this task -> Seleccionar  &#8220;Even if a previous task has failed, unless the build was cancelled&#8221;. Esta opción es importante ya que si no nunca se subirían los resultados si los tests fallaran.

A partir de aquí, ya crearías el resto de tareas  (p. ej. construir el resto de servicios y publicarlos en un repositorio) de la forma tradicional.
  
Ahora una vez ejecutada la build, si los tests fallan la build falla pero ahora vemos los resultados en VSTS:
  
[<img class="alignnone size-medium wp-image-2022" src="https://geeks.ms/etomas/wp-content/uploads/sites/154/2018/03/build-2-tests-300x64.png" alt="Tests integrados en VSTS" width="300" height="64" />][7]
  
¡Espero que os haya sido útil!
  
PD: He dejado un .zip con el código en [https://1drv.ms/u/s!Asa-selZwiFlg\_Ag\_p9batvLwRS5zw][8] (por si quieres echarle un vistazo y probarlo de forma rápida)

 [1]: https://geeks.ms/etomas/2017/11/28/docker-multi-stage-builds-o-como-compilar-casi-cualquier-cosa-sin-tener-que-instalar-nada/
 [2]: https://geeks.ms/etomas/wp-content/uploads/sites/154/2018/03/solution.png
 [3]: https://geeks.ms/etomas/wp-content/uploads/sites/154/2018/03/build-1.png
 [4]: https://geeks.ms/etomas/wp-content/uploads/sites/154/2018/03/build-1-tests.png
 [5]: https://geeks.ms/etomas/wp-content/uploads/sites/154/2018/03/build-2-task1.png
 [6]: https://geeks.ms/etomas/wp-content/uploads/sites/154/2018/03/build-2-task2.png
 [7]: https://geeks.ms/etomas/wp-content/uploads/sites/154/2018/03/build-2-tests.png
 [8]: https://1drv.ms/u/s!Asa-selZwiFlg_Ag_p9batvLwRS5zw