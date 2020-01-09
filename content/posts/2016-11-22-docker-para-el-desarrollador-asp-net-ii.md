---
title: Docker para el desarrollador asp.net (ii)
author: eiximenis

date: 2016-11-22T12:30:46+00:00
geeks_url: /?p=1841
geeks_ms_views:
  - 2273
categories:
  - asp.net 5
  - asp.net vNext
  - docker

---
Seguimos con esta sobre el uso de Docker desde el punto de vista de un desarrollador asp.net (core). En este caso **vamos a construir nuestra primera imagen Docker**.

**Nota:** Visual Studio 2017 incorpora de serie las _Docker Tools_ que automatizan todo lo que veremos en estos artículos. Tiempo tendremos, más adelante en esta serie, de hablar de las _Docker Tools_. La razón de hacerlo primero todo “manual” es porque el objetivo de esta serie es ayudarte a que entiendas Docker, no presentarte el “botón mágico” que se encarga de todo. Yo, es que soy de la vieja escuela: me gusta entender las cosas (al menos hasta donde puedo).

<!--more-->

**Creando el proyecto**

Doy por supuesto que tienes instalado <a href="https://docs.docker.com/docker-for-windows/" target="_blank" rel="noopener noreferrer">Docker for Windows</a>, así que podemos ir directamente al grano. En este artículo **vamos a usar netcore 1.0 con project.json** o lo que es lo mismo las _tools_&nbsp; en preview2. Ya tendremos tiempo más adelante de pasar a las _tools_ en preview3 (es decir las que vienen con VS2017 y usan csproj).

Para ello vamos a crear un **directorio vacío** y dentro de él vamos a crear un _global.json_. El objetivo de este fichero es fijar el SDK en su versión de _preview2_ y así usar project.json. Si no se usa global.json se usa por defecto la última versión de las herramientas y si te has instalado Visual Studio 2017 tendrás la _preview3_ que usa&nbsp; csproj. De esta manera podemos forzar a seguir usando project.json, que es lo que queremos por ahora.

Coloca el siguiente contenido en dicho fichero:

<div class="highlight" style="background: #f8f8f8">
  <pre style="line-height: 125%">{
    <span style="font-weight: bold; color: #008000">"sdk"</span>: {
        <span style="font-weight: bold; color: #008000">"version"</span>: <span style="color: #ba2121">"1.0.0-preview2-003121"</span>
    }
}
</pre>
</div>

Genial, ¡ahora ejecutamos “dotnet new” desde la herramienta de línea de comandos y ya tenemos nuestro proyecto! Podemos probar que funciona ejecutando “dotnet restore” primero y “dotnet run” después y deberíamos ver “Hello world!”

**Creando la imagen**

Bueno, el siguiente paso es crear la imagen. Para ello debemos crear un _dockerfile_, que es el fichero que indica a Docker como debe crear una imagen. Este fichero debe llamarse _Dockerfile_ (sin extensión ni nada).

Las imágenes en Docker tienen una relación de herencia. Piensa que la imagen describe los contenedores creados a partir de ella. En estos momentos la palabra “contenedor” es ideal: piensa en los contenedores como eso… contenedores, en los cuales debemos meter “cosas” (vamos, ficheros, librerías y demás). Si queremos crear una imagen para contenedores que ejecuten netcore deberemos colocar el runtime de netcore en el contenedor… y todas las dependencias del runtime y las dependencias de las dependencias y las dependencias de las dependencias de las dependencias y… ¿lo pillas no?

Nosotros no queremos lidiar con todas esas dependencias, así que lo habitual es que nuestra imagen herede de otra. Es decir, que nuestra imagen parta de otra imagen y agregue ficheros a los ficheros especificada por la otra imagen. Podríamos partir de una imagen que tuviese un Ubuntu base y entonces deberíamos agregarle el runtime de netcore y todas sus dependencias (tal y como instalaríamos dotnet core en un Ubuntu salido de fábrica) o incluso mejor partir de una imagen que ya tenga esas dependencias instaladas y solo meterle dotnetcore… o mejor incluso partir de una que tenga dotnet core instalado y así ya solo meterle nuestro código.

Como todo en informática el truco está en esperar un tiempo prudencial para que otro se coma el marrón de generar esas imágenes. En nuestro caso ese tiempo ya ha pasado y Microsoft ya ha generado imágenes base con el runtime de netcore instalado. Así podemos empezar a editar nuestro _Dockerfile_ colocando el siguiente código en él:

<div class="highlight" style="background: #f8f8f8">
  <pre style="line-height: 125%"><span style="font-weight: bold; color: #008000">FROM</span> microsoft/dotnet:1.0.1-runtime
</pre>
</div>

¡Bienvenido a mi tutorial rápido de _Dockerfile_! Con FROM XXX indicamos que nuestra imagen hereda de la imagen XXX (calenturientos de mente apartaos XD). Los nombres de imágen tienen un formato usuario/imagen:tag. En este caso el usuario es “microsoft”, la imagen es “dotnet” y el tag es “1.0.1-runtime”.

Vale, esa imagen por si sola no hace nada, sirve solo de base. Ahora debemos meter nuestro código en él. Para ello usamos el comando COPY del _Dockerfile_. Ese comando copia ficheros del filesystem local al filesystem del contenedor. Dejemos nuestro _Dockerfile_ tal y como sigue:

<div class="highlight" style="background: #f8f8f8">
  <pre style="line-height: 125%"><span style="font-weight: bold; color: #008000">FROM</span> microsoft/dotnet:1.0.1-runtime
COPY *.cs /app/
COPY project.json /app/
<span style="font-weight: bold; color: #008000">CMD</span> ls app
</pre>
</div>

Hemos añadido dos líneas para copiar nuestros ficheros. La sintaxis de **COPY** es muy sencilla: _COPY origen destino_, donde origen está en el filesystem local y destino en el del contenedor (fíjate en las barras a lo Unix, que los contenedores son Linux).

Finalmente la última línea necesita una explicación. ¿Recuerdas que un contenedor ejecuta un solo proceso? Por defecto este proceso es “/bin/sh –c”. Eso lo que hace es abrir un shell y ejecutar lo que se pase despues de –c. Pues bien, **CMD** nos permite especificar lo que se pasa a –c. Esta imagen todavía no ejecuta nuestro código, pero ya llegaremos a ello. De momento veamos como ejecutarla. ¡Ha llegado el momento de ejecutar la CLI de Docker!

El primer paso es **construir la imágen**. Para ello lo más sencillo es ejecutar el comando _docker build ._ (observa el punto final). Este comando construye la imagen indicada por el _Dockerfile_ que esté en el directorio indicado (le pasamos el punto que es el directorio actual):

[<img title="SNAGHTML5e28fde" style="border-left-width: 0px; border-right-width: 0px; background-image: none; border-bottom-width: 0px; padding-top: 0px; padding-left: 0px; display: inline; padding-right: 0px; border-top-width: 0px" border="0" alt="SNAGHTML5e28fde" src="http://geeks.ms/etomas/wp-content/uploads/sites/154/2016/11/SNAGHTML5e28fde_thumb.png" width="644" height="354" />][1]

Con ese comando Docker se baja si necesita la imagen base (si no la tienes ya en local) y luego construye nuestra imagen. Nos interesa el ID de esta imagen que es lo que nos da con el mensaje “_Succesfully built xxxxxxxx_” (lo he marcado en rojo). Por supuesto tu ID será distinto, los genera Docker cada vez.

Ya tenemos una imagen… ahora ya podemos crear y ejecutar contenedores basados en ella. Para ello basta con ejecutar _docker run <id>_ donde <id> es el id de la imagen:

[<img title="image" style="border-left-width: 0px; border-right-width: 0px; background-image: none; border-bottom-width: 0px; padding-top: 0px; padding-left: 0px; display: inline; padding-right: 0px; border-top-width: 0px" border="0" alt="image" src="http://geeks.ms/etomas/wp-content/uploads/sites/154/2016/11/image_thumb-2.png" width="323" height="141" />][2]

Podemos ver que la salida es el resultado de listar los ficheros en el directorio /app, es decir el resultado de ejecutar la sentencia especificada en el CMD del Dockerfile. Una vez hecho esto el contenedor termina.

Vamos a borrar la imagen, para ello ejecuta _docker rmi <id> &#8211;force_ donde el <id> es el id de la imagen. El parámetro “&#8211;force” es necesario porque hay contenedores parados basados en esa imagen (el que acabamos de ejecutar):

[<img title="image" style="border-left-width: 0px; border-right-width: 0px; background-image: none; border-bottom-width: 0px; padding-top: 0px; padding-left: 0px; display: inline; padding-right: 0px; border-top-width: 0px" border="0" alt="image" src="http://geeks.ms/etomas/wp-content/uploads/sites/154/2016/11/image_thumb-3.png" width="644" height="143" />][3]

Vale, vamos a modificar nuestro _Dockerfile_ para lanzar nuestro programa. Para ello una suposición sería tener un _Dockerfile_ como el siguiente:

<div class="highlight" style="background: #f8f8f8">
  <pre style="line-height: 125%"><span style="font-weight: bold; color: #008000">FROM</span> microsoft/dotnet:1.0.1-runtime
COPY *.cs /app/
COPY project.json /app/
<span style="font-weight: bold; color: #008000">CMD</span> dotnet restore <span style="color: #666666">&&</span> dotnet run
</pre>
</div>

Pero… ¡**pataplaf!** Ese _Dockerfile_ no funciona. Si construyes la imagen y la ejecutas verás un error como el siguiente:

[<img title="image" style="border-left-width: 0px; border-right-width: 0px; background-image: none; border-bottom-width: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px; border-top-width: 0px" border="0" alt="image" src="http://geeks.ms/etomas/wp-content/uploads/sites/154/2016/11/image_thumb-4.png" width="644" height="208" />][4]

Este error se da porque la imagen base tiene el runtime de netcore, pero NO el SDK instalado. Podríamos instalarlo nosotros, pero por suerte Microsoft lo ha hecho por nosotros. Hay otra imagen que viene con el SDK instalado: _microsoft/dotnet:1.0-sdk-projectjson_.

De todos modos, tema de la imagen base aparte este _Dockerfile_ no es correcto. No tiene sentido **ejecutar un dotnet restore como parte del comando del contenedor**. ¿Por que no? Pues porque entonces cada vez que levantemos un contenedor (con _docker run_) se ejecutará el restore. ¿Realmente queremos eso? No, porque todos los contenedores tienen los mismos ficheros. Basta con ejecutar el _restore_ una vez y luego “guardarnos” el resultado del restore y cuando levantemos un contenedor ejecutar solo el dotnet run.

Recuerda que una imagen es una descripción a partir de la cual creamos contenedores. La imagen contiene los ficheros que va a tener el contenedor. El resultado de ejecutar “dotnet restore” añade ficheros (como mínimo el project.lock.json). Lo que queremos es que estos ficheros añadidos como parte de “dotnet restore” formen parte de la imagen. 

Si estás pensando en ejecutar “dotnet restore” de forma manual y luego copiar el resultado a la imagen… ¡eso no es forma de hacerlo! (entre otras razones porque no tienes la garantía de que “dotnet restore” en tu máquina local genere lo mismo que “dotnet restore” en la máquina que ejecuta los contenedores (recuerda una VM de Linux)). No, la solución es ejecutar este comando **y generar una nueva imágen** a partir de él.

Y el _Dockerfile_ tiene un comando específico para ello. El comando RUN. Dicho comando tiene la sintaxis _RUN <cmd>_ donde <cmd> es un comando que se ejecuta y se genera una nueva imagen que hereda de la que se está construyendo y contiene los cambios generados por el comando. Y esa nueva imagen es la que se utiliza de ahora en adelante (si no quieres verlo de esa manera, puedes verlo como que la imagen ejecuta dotnet run. Si lo ves así, recuerda que la imagen se ejecuta solo una vez, cuando se ejecuta docker build).

Así nuestro _Dockerfile_ queda como sigue:

<div class="highlight" style="background: #f8f8f8">
  <pre style="line-height: 125%"><span style="font-weight: bold; color: #008000">FROM</span> microsoft/dotnet:1.0-sdk-projectjson
COPY *.cs /app/
COPY project.json /app/
<span style="font-weight: bold; color: #008000">WORKDIR</span> /app
<span style="font-weight: bold; color: #008000">RUN</span> dotnet restore
<span style="font-weight: bold; color: #008000">CMD</span> dotnet run
</pre>
</div>

Descrito línea por línea:

  1. Heredamos de la imagen base que contiene el SDK de netcore 
      * Copiamos los ficheros .cs en /app 
          * Copiamos el project.json en /app 
              * Establecemos /app como el nuevo directorio de trabajo 
                  * Se ejecuta dotnet restore. Esto añade ficheros en la imagen. 
                      * Definimos que el comando a ejecutar al iniciar el contenedor es dotnet run (que se ejecutará en /app debido a que es el directorio de trabajo).</ol> 
                    Si ahora ejecutas “docker build .” verás como el proceso tarda un poco más. Eso es porque el “dotnet restore” se ejecuta durante el proceso de construcción de la imagen:
                    
                    [<img title="image" style="border-left-width: 0px; border-right-width: 0px; background-image: none; border-bottom-width: 0px; padding-top: 0px; padding-left: 0px; display: inline; padding-right: 0px; border-top-width: 0px" border="0" alt="image" src="http://geeks.ms/etomas/wp-content/uploads/sites/154/2016/11/image_thumb-5.png" width="644" height="415" />][5]
                    
                    Observa (marcado en verde) la salida del _dotnet restore_ como parte del _docker build._ Ten presente que a pesar de que veas eso en la consola de windows, la construcción de la imagen la ejecuta la máquina virtual que tiene Docker for Windows.
                    
                    Ahora ya podemos ejecutar la imagen y probar que realmente funciona:
                    
                    [<img title="image" style="border-left-width: 0px; border-right-width: 0px; background-image: none; border-bottom-width: 0px; padding-top: 0px; padding-left: 0px; display: inline; padding-right: 0px; border-top-width: 0px" border="0" alt="image" src="http://geeks.ms/etomas/wp-content/uploads/sites/154/2016/11/image_thumb-6.png" width="644" height="230" />][6]
                    
                    ¡Funciona! Podemos ver el resultado del _dotnet run_ en nuestra consola. Una vez se ha ejecutado el contenedor termina, siempre podemos volver a lanzar otro contenedor ejecutando _docker run_ de nuevo.
                    
                    Bueno… dejémoslo aquí por hoy. Hemos visto el proceso de creación de una imagen y como crear un _Dockerfile_ sencillo para ejecutar una app netcore. También hemos explorado un poco la CLI de docker.
                    
                    Espero que os haya gustado… en el siguiente artículo, un poco más 😉

 [1]: http://geeks.ms/etomas/wp-content/uploads/sites/154/2016/11/SNAGHTML5e28fde.png
 [2]: http://geeks.ms/etomas/wp-content/uploads/sites/154/2016/11/image-2.png
 [3]: http://geeks.ms/etomas/wp-content/uploads/sites/154/2016/11/image-3.png
 [4]: http://geeks.ms/etomas/wp-content/uploads/sites/154/2016/11/image-4.png
 [5]: http://geeks.ms/etomas/wp-content/uploads/sites/154/2016/11/image-5.png
 [6]: http://geeks.ms/etomas/wp-content/uploads/sites/154/2016/11/image-6.png