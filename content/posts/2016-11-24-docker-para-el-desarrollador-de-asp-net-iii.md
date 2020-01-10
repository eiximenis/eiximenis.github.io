---
title: Docker para el desarrollador de asp.net (iii)

author: eiximenis

date: 2016-11-24T09:27:48+00:00
geeks_url: /?p=1855
geeks_ms_views:
  - 2750
categories:
  - asp.net 5
  - asp.net vNext
  - docker

---
En el <a href="http://geeks.ms/etomas/2016/11/22/docker-para-el-desarrollador-asp-net-ii/" target="_blank" rel="noopener noreferrer">post anterior</a> vimos como empaquetar y desplegar en Docker una sencilla aplicación (un _hello world_) en asp.net core. En este post vamos a ver como desplegar en Docker una aplicación asp.net core (con sus controladores y vistas) y también ver como lo podemos usar usando una imagen base que no tenga el SDK, solo el runtime.

¡Vamos allá!

<!--more-->

**Creando el proyecto**

Vamos a usar ya **Visual Studio 2015** para la creación del proyecto. De momento seguimos con la preview2 de las herramientas (es decir con project.json). Para crear el proyecto simplemente abrimos VS2015 y creamos un nuevo proyecto de tipo “ASP.NET Core Web Application (.NET Core)” y luego seleccionamos la plantilla de “Web Application”. Con eso VS2015 nos creará una aplicación con un _HomeController_ que muestra la clásica vista inicial de ASP.NET. No haremos más cambios al proyecto, con esto nos basta 🙂

**Publicando el proyecto**

Vamos a crear una imagen de Docker para servir esta aplicación web desde un contenedor. A diferencia del post anterior donde partíamos de una imagen base que tenía el SDK de NetCore, en este caso vamos a partir de una imagen que tenga solo el _runtime_. Si lo piensas tiene sentido: qué necesidad tenemos de compilar (con _dotnet run_) la aplicación cada vez que levantamos un contenedor? Podemos compilarla **una sola vez** y desplegar una imagen con el resultado de la compilación. De hecho, **es lo que haríamos si lo desplegaramos en una WebApp de Azure o en un IIS ¿no?** Pues no hay motivo alguno para tratar Docker de forma distinta.

Eso nos implica que **debemos publicar** nuestra aplicación **y copiar el resultado de la publicación** a la imagen. Vamos a ver como.

Para ello empecemos por añadir un Dockerfile a la solución y establezcamos la imagen base:

<pre style="max-width: 700px; font-family: consolas; background: #1e1e1e; white-space: nowrap; overflow-x: scroll; color: gainsboro"><span style="color: #569cd6">FROM</span> microsoft/aspnetcore:1.0.1</pre>

Esa es la que vamos a usar que contiene el runtime de asp.net core (¡pero no el SDK!)

El siguiente paso es **publicar la aplicación en local**, esto se puede hacer desde la lína de comandos de forma trivial. Abre una línea de comandos y sitúate en el directorio donde hay el project.json. Luego teclea “_dotnet publish -o app_” para publicar la aplicación.

**Es posible que recibas un error: _No executable found matching command &#8220;bower&#8221;_**. Eso es debido a que la plantilla por defecto de VS2015 utiliza bower (un gestor de paquetes JavaScript) y no lo tienes instalado en tu máquina. Personalmente no soy muy amigo de usar bower, pero bueno… Dado que la plantilla por defecto, lo requiere, vamos a ver como podemos hacerlo. Una solución seria installar bower a nivel global (_npm install –g bower_) pero es recomendable instalarlo de forma local (es decir, solo para tu proyecto). Con eso te aseguras de que la versión de _bower_ que requiere tu proyecto es la que tiene instalada. **Visual Studio 2015 debería gestionar eso, pero no lo hace** (asume que _bower_ está instalado de forma global). Veamos como podemos arreglarlo de forma rápida. Para ello desde el propio Visual Studio, añadimos un elemento al proyecto (Add –> New Item) y seleccionamos “npm Configuration file” (si no te aparece entra el texto “package.json” en el cuadro de búsqueda):

[<img title="SNAGHTML313e81" style="border-left-width: 0px; border-right-width: 0px; background-image: none; border-bottom-width: 0px; padding-top: 0px; padding-left: 0px; display: inline; padding-right: 0px; border-top-width: 0px" border="0" alt="SNAGHTML313e81" src="http://geeks.ms/etomas/wp-content/uploads/sites/154/2016/11/SNAGHTML313e81_thumb.png" width="644" height="360" />][1]

(Por supuesto, puedes usar _npm init_ desde la línea de comandos, si así lo prefieres).

Ahora que ya tenemos un _package.json_ ya podemos añadir dependencias npm. Eso lo necesitamos porque _bower_ es un paquete npm. Ahora abre el package.json con VS2015 y añade _bower_ como una dependencia de desarrollo:

<pre style="max-width: 700px; font-family: consolas; background: #1e1e1e; white-space: nowrap; overflow-x: scroll; color: gainsboro"><span style="color: #d7ba7d">"devDependencies"</span><span style="color: #b4b4b4">:</span>&nbsp;<span style="color: #b4b4b4">{</span><br />&nbsp; <span style="color: #d7ba7d">"bower"</span><span style="color: #b4b4b4">:</span>&nbsp;<span style="color: #d69d85">"^1.8.0"</span><br /><span style="color: #b4b4b4">}</span></pre>

(Si usas la línea de comandos puedes conseguir el mismo efecto con _npm install bower –save-dev_)

Eso nos instala _bower_ a nivel local, pero la publicación directamente seguirá dando un error, porque _bower_ no está en el PATH (está instalado en .\node_modules\.bin, siendo “.” el directorio inicial del proyecto). No es necesario que lo añadas, al path, vamos a modificar el project.json y colocar la ruta. Para ello abre el project.json y edita la sección “scripts” para añadir el path entero de bower:

<pre style="max-width: 700px; font-family: consolas; background: #1e1e1e; white-space: nowrap; overflow-x: scroll; color: gainsboro"><span style="color: #d7ba7d">"scripts"</span><span style="color: #b4b4b4">:</span>&nbsp;<span style="color: #b4b4b4">{</span><br />&nbsp; <span style="color: #d7ba7d">"prepublish"</span><span style="color: #b4b4b4">:</span>&nbsp;<span style="color: #b4b4b4">[</span>&nbsp;<span style="color: #d69d85">"./node_modules/.bin/bower.cmd install"</span><span style="color: #b4b4b4">,</span>&nbsp;<span style="color: #d69d85">"dotnet bundle"</span>&nbsp;<span style="color: #b4b4b4">],</span><br />&nbsp; <span style="color: #d7ba7d">"postpublish"</span><span style="color: #b4b4b4">:</span>&nbsp;<span style="color: #b4b4b4">[</span>&nbsp;<span style="color: #d69d85">"dotnet publish-iis --publish-folder %publish:OutputPath% --framework %publish:FullTargetFramework%"</span>&nbsp;<span style="color: #b4b4b4">]</span><br /><span style="color: #b4b4b4">}</span></pre>

¡Ahora sí, que sí! Abre una consola de línea de comandos y publica la aplicación “_dotnet publish –o app_”. Ahora no debería darte errores:

[<img title="image" style="border-left-width: 0px; border-right-width: 0px; background-image: none; border-bottom-width: 0px; padding-top: 0px; padding-left: 0px; display: inline; padding-right: 0px; border-top-width: 0px" border="0" alt="image" src="http://geeks.ms/etomas/wp-content/uploads/sites/154/2016/11/image_thumb-7.png" width="644" height="300" />][2]

Eso ha creado un directorio app con nuestra aplicación publicada. Esta publicación contiene todos los paquetes de NuGet necesarios para ejecutar la app, junto con la aplicación compilada y la carpeta wwwroot. Solo falta… meterlo en el contenedor.

**Creando la imagen**

Volvamos a nuestro Dockerfile. Tenemos que copiar **el directorio app** de nuestra app al contenedor y coloca el siguiente código:

<pre style="max-width: 700px; font-family: consolas; background: #1e1e1e; white-space: nowrap; overflow-x: scroll; color: gainsboro"><span style="color: #569cd6">FROM</span> microsoft/aspnetcore:1.0.1<br />ENTRYPOINT [<span style="color: #d69d85">"dotnet"</span>, <span style="color: #d69d85">"DemoWebApp.dll"</span>]<br />WORKDIR /app<br />EXPOSE 80<br />COPY ./DemoWebApp/app .<br /></pre>

Este Dockerfile contiene algunas sentencias nuevas, en concreto ENTRYPOINT y EXPOSE, vamos a analizar lo que hace línea por línea:

  1. Usa la imágen base microsoft/aspnetcore:1.0.1 que contiene el _runtime_ de asp.net 
      * Con ENTRYPOINT le indicamos al contenedor cual es el proceso que debe lanzar al inicio. Quizá te preguntes cual es la diferencia con CMD que vimos en el post anterior. Es muy sencillo: recuerda que un contenedor lanza un solo proceso. Por defecto este proceso es /bin/sh –c. El valor de CMD es lo que pasa como parámetro al –c. Ahora bien, puede ser que **no queramos ejecutar /bin/sh, sino otro ejecutable** al iniciar el contenedor. En este caso es cuando usamos ENTRYPOINT. ENTRYPOINT indica el ejecutable a ejecutar al iniciar un contenedor, mientras que CMD indica el parámetro a pasar al ejecutable (que por defecto, es decir sin ENTRYPOINT es /bin/sh). El formato de ENTRYPOINT es ENTRYPOINT [“ejecutable”, “param1”,…,”paramN”] para ejecutar el ejecutable indicado con todos los parámetros pasados. Pero si quieres puedes usar también CMD para pasar parámetros adicionales al ENTRYPOINT. 
          * Establecemos /app como el directorio inicial del contenedor 
              * Le indicamos a _Docker_ que este contenedor debe exponer el puerto 80 
                  * Copiamos el contenido de DemoWebApp/app (es decir el directorio local donde hemos publicado la web, cambialo para que sea el tuyo. Observa que es relativo al Dockerfile) al directorio actual del contenedor (que es app debido al WORKDIR anterior).</ol> 
                Vale, ahora ya podemos construir la imagen. Para ello desde una línea de comandos situados en el directorio donde hay el Dockerfile teclea “_docker build –t eiximenis/demoweb ._”:
                
                [<img title="image" style="border-left-width: 0px; border-right-width: 0px; background-image: none; border-bottom-width: 0px; padding-top: 0px; padding-left: 0px; display: inline; padding-right: 0px; border-top-width: 0px" border="0" alt="image" src="http://geeks.ms/etomas/wp-content/uploads/sites/154/2016/11/image_thumb-8.png" width="644" height="300" />][3]
                
                El parámetro –t es **para poner un nombre a la imagen**. Genial ya tenemos la imagen construida. Para ejecutarla necesitamos saber su ID (no nos sirve el nombre). Recuerda que docker te dice el ID como resultado del _docker build_ pero siempre puedes obtenerlo mediante el comando _docker images_. Este comando lista todas las imagenes que tengas instaladas con su nombre&nbsp; (si lo tiene) y su ID:
                
                [<img title="image" style="border-left-width: 0px; border-right-width: 0px; background-image: none; border-bottom-width: 0px; padding-top: 0px; padding-left: 0px; display: inline; padding-right: 0px; border-top-width: 0px" border="0" alt="image" src="http://geeks.ms/etomas/wp-content/uploads/sites/154/2016/11/image_thumb-9.png" width="644" height="156" />][4]
                
                **Lanzando la imagen**
                
                Para lanzar la imagen nos basta como siempre con _**docker run <id>**_ pero antes de que te lances a ejecutarlo, tenemos que hablar del mapeo de puertos. La imagen indica que el contenedor expone el puerto 80. Este puerto **es el puerto local del contenedor** y nada tiene que ver con el puerto del host. Recuerda que podrías tener varios contenedores corriendo al mismo tiempo, todos ellos generados por la misma imagen y todos ellos exponiendo su puerto 80. Es obvio pues que el puerto 80 del contenedor no puede ser el 80 del host. Así, al ejecutar el contenedor **debemos decirle a docker a que puerto del host debe mapear el puerto 80 del contenedor.** Para ello usaremos el parámetro –p, que toma la forma de –p:<puerto\_local>:<puerto\_contenedor>, es decir si p. ej. queremos que nuestro contenedor use el puerto del host 8001 usaríamos el comando “_docker run –p 8001:80 <id>._ Esto crea un contenedor basado en la imagen <id> y mapea el puerto 80 del contenedor (que este expone via el EXPOSE) al 8001 del host:
                
                [<img title="image" style="border-left-width: 0px; border-right-width: 0px; background-image: none; border-bottom-width: 0px; padding-top: 0px; padding-left: 0px; display: inline; padding-right: 0px; border-top-width: 0px" border="0" alt="image" src="http://geeks.ms/etomas/wp-content/uploads/sites/154/2016/11/image_thumb-10.png" width="644" height="124" />][5]
                
                Se puede ver como el contenedor se queda escuchando. Por lo tanto si ahora abrimos un navegador y navegamos a <http://localhost:8001>…
                
                [<img title="image" style="border-left-width: 0px; border-right-width: 0px; background-image: none; border-bottom-width: 0px; padding-top: 0px; padding-left: 0px; display: inline; padding-right: 0px; border-top-width: 0px" border="0" alt="image" src="http://geeks.ms/etomas/wp-content/uploads/sites/154/2016/11/image_thumb-11.png" width="644" height="295" />][6]
                
                ¡Ya tenemos nuestra web funcionando en Docker!
                
                Podemos ver que nuestro contenedor está corriendo abriendo una consola de línea de comandos y tecleando _docker ps_. Eso nos muestra un listado de todos los contenedores actualmente en marcha. Podemos parar el contenedor con _docker stop <id_contenedor>_ y reiniciarlo cuando queramos con _docker start <id_contenedor>_. Finalmente podemos borrarlo con _docker rm <id_contenedor>_
                
                Bueno… lo dejamos aquí por hoy. En estos tres posts hemos avanzado bastante… Hemos cubierto lo básico de Docker y hemos visto paso a paso como publicar un proyecto en Docker de forma “manual”. Estamos listos ya para el siguiente paso, que se llama docker-compose… 😉

 [1]: http://geeks.ms/etomas/wp-content/uploads/sites/154/2016/11/SNAGHTML313e81.png
 [2]: http://geeks.ms/etomas/wp-content/uploads/sites/154/2016/11/image-7.png
 [3]: http://geeks.ms/etomas/wp-content/uploads/sites/154/2016/11/image-8.png
 [4]: http://geeks.ms/etomas/wp-content/uploads/sites/154/2016/11/image-9.png
 [5]: http://geeks.ms/etomas/wp-content/uploads/sites/154/2016/11/image-10.png
 [6]: http://geeks.ms/etomas/wp-content/uploads/sites/154/2016/11/image-11.png