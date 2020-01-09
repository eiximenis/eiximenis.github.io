---
title: Crear imágenes Docker de una app SPA pura y… como configurarla
description: Crear imágenes Docker de una app SPA pura y… como configurarla
author: eiximenis

date: 2018-10-26T16:44:13+00:00
geeks_url: /?p=2199
geeks_ms_views:
  - 782
categories:
  - docker

---
El otro día habé de [como crear imágenes Docker para las aplicaciones SPA de .NET Core][1]. Hoy quiero comentaros como crear imágenes Docker para aplicaciones SPA puras y un tema importante al respecto: **como configurarlas**.
  
<!--more-->


  
Xavi me preguntó por Twitter cual era la utilidad de usar aplicaciones SPA servidas por .NET Core (o para el caso que nos ocupa cualquier otro Backend como uno Node):

<blockquote class="twitter-tweet" data-width="550" data-dnt="true">
  <p lang="es" dir="ltr">
    No tiene más sentido si se va a hacer un proyecto en react o angular el no utilizar .Net sino una máquina sólo con el framework y nos o lo q sea???
  </p>
  
  <p>
    &mdash; Xavier Jorge Cerdá (@XaviPaper) <a href="https://twitter.com/XaviPaper/status/1055540142587416576?ref_src=twsrc%5Etfw">October 25, 2018</a>
  </p>
</blockquote>


  
Yo le respondí a Xavi que una utilida es si vas a servir una API de consumo propio de la SPA. En este caso, el mismo Backend que te sirve la API te sirve también la SPA. Es una solución simple y efectiva y también te olvidas de [CORS][2] p. ej. porque el origen de la SPA y de la API es el mismo.
  
Pero, en general, Xavi tiene razón: lo interesante de una app SPA pura es servirla directamente: a fin de cuentas se trata de ficheros estáticos (html, css, js). Se podría servir perfectamente desde un CDN. Pero y si necesitamos usar Docker?
  
Pues en este caso nada más sencillo: lo habitual es usar una imagen de node para el stage de build y una de nginx (o similar) para la imagen final. Realmente tu imagen final es un nginx, su configuración y todos los ficheros estáticos de tu app:

<pre class="EnlighterJSRAW" data-enlighter-language="null">FROM node:10-alpine as build
WORKDIR /app
COPY package.json .
COPY package-lock.json .
RUN npm install
COPY . .
RUN npm run build
FROM nginx:stable as final
WORKDIR /web
COPY ./nginx.conf /etc/nginx/conf.d/default.conf
COPY --from=build /app/build .
</pre>

Como puedes ver no tiene ningún secreto: En el stage de _build_ lanzo los comandos necesarios (en este ejemplo básicamente _npm run build_) y copio el resultado al stage _final_ que es una imagen de NGINX. Claro, debo copiar el fichero de configuración de nginx. Uno minimalista sería:

<pre class="EnlighterJSRAW" data-enlighter-language="null">server {
    listen 80;
    server_name _;
    root /web;
    index index.html;
    location / {
        try_files $uri /index.html;
    }
}</pre>

Este fichero de nginx o bien carga un fichero que exista o en caso contrario sirve &#8220;index.html&#8221;. Y listos.
  
Hasta ahí todo bien, esa era la parte fácil. Ahora la pregunta del millón: **¿como configuramos nuestra SPA?**
  
**Configuración de una SPA pura ejecutándose en Docker**
  
Nuestra SPA necesita algunos parámetros de configuración: p. ej. las URLs de las APIs a las que debamos acceder y quizá alguna otra cosilla más. Es importante recalcar **que hablo de configuración pública, no secretos**. Es decir, si mi SPA requiere algún secreto, tipo un token de Google Maps p. ej. debe obtenerlo siempre llamando a una API autenticada. Ahora bien, la URL de esta API si que forma parte de la configuración de la SPA, ya que es un dato público (no secreto).
  
Si eres muy frontender igual te sorprendas que discutamos como configurar una aplicación SPA. En muchos casos es habitual hacer esa configuración al construir la aplicación. Es decir, lanzo un comando tipo &#8220;npm run build:dev&#8221; y eso me genera una versión de la app configurada contra el entorno de Dev y si lanzo &#8220;npm run build:prod&#8221; tengo una versión configurada para el entorno de producción. Si me preguntas qué hay de malo en esa aproximación, te responderé que en general nada y que funciona bien. Un sistema de _builds_ puede construir las distintas versiones y desplegarlas en distintos entornos. Todo OK, ¿verdad?. Pues la verdad es **que esta visión no encaja para nada con Docker**. ¿Por qué? Pues porque una de las reglas de oro que debes intentar seguir siempre es que **la misma imagen que despliegas en un entorno es la que despliegas en otro**. Eso significa que no debes construir una imagen para desarrollo (generada a partir de &#8220;npm run build:dev&#8221; en el stage de _build_) y otra para otro entorno, como producción. **La imagen Docker debe ser única, debe ser generada una sola vez y desplegada en distintos entornos. La configuración la debe proveer el entorno**.
  
Claro, **si tienes una SPA servida por .NET Core u otro Backend puedes hacer un truco muy sencillo**: desde el Backend lees las variables de entorno que tienen la configuración, montas un endpoint (p. ej. en /api/config) y en dicho endpoint sirves la configuración. Luego, solo debes llamar a ese endpoint al inicializar tu SPA y listos. Es sencillo y poco doloroso. Pero **en una imagen pura SPA no puedes usar esa técnica**, básicamente porque no tienes nada con qué servir una API. ¿Entonces? ¿Qué puedes hacer?
  
Bueno, pues aunque no puedes usar esta técnica directamente, si que puedes &#8220;simularla&#8221;. ¿Como? En nuestro caso **podemos configurar Nginx para que, dada una petición concreta (p. ej. /api/config) nos sirva un fichero estático**. Observa que eso es posible porque usamos un servidor tipo Nginx, con un CDN no podríamos (aunque como comenté antes, si despliegas en un CDN lo habitual es generar la configuración al construir la aplicación).
  
Así, podríamos configurar Nginx para que si llega una petición a **/api/conf** sirva el fichero estático /app/conf.js:

<pre class="EnlighterJSRAW" data-enlighter-language="null">location /api/conf.json {
   alias /app/config/conf.json
}
</pre>

NGinx servirá el fichero &#8220;/app/config/conf.json&#8221; cada vez que la aplicación web llame a &#8220;/api/conf.json&#8221;. Fantástico, ahora **solo tenemos que ver como podemos &#8220;inyectar&#8221; un fichero conf.json configurado según el entorno**.
  
Pues se me ocurren dos opciones:
  
**Opción 1: Usar fichero con tokens y sustituirlos al iniciar la aplicación**
  
Esta aproximación es muy sencilla, pero oye, funciona. Despliegas un fichero en la imagen (p. ej. en /app/config/template.json) que tenga ciertos tokens y **al iniciar el contenedor usas las variables de entorno** para sustituir esos tokens con los valores de dichas variables y generas el fichero final.
  
Y eso es extremadamente simple. Si tu fichero tiene el código:

<pre class="EnlighterJSRAW" data-enlighter-language="json">{
   "some-key": "$XXX"
}</pre>

Puedes usar [envsubst][3] para sustituir los tokens $XXX por el valor de la variable de entorno XXX. Así el comando que debes usar es &#8220;cat template.json | envsubst > conf.js&#8221;. Observa la imagen para ver como funciona envsubst:
  
[<img class="alignnone size-full wp-image-2207" src="https://geeks.ms/etomas/wp-content/uploads/sites/154/2018/10/envsubst.png" alt="Salida de envsubst" width="551" height="217" />][4]
  
Por lo tanto **te puedes hacer un shell script(p. ej. llamado tokenize.sh) que invoque esos comandos** y que el punto de entrada de tu imagen sea algo como:

<pre class="EnlighterJSRAW" data-enlighter-language="null">CMD tokenize.sh & nginx -g daemon off;</pre>

De este modo se ejecuta tu shell script que usando envsubst te genera la configuración y luego lanza Nginx.
  
Este mcanismo **es muy eficiente y tiene cero overhead: **se genera la configuración al iniciar el contenedor (basándose en las variables de entorno) y listos. Por supuesto debes asegurarte que envsubst está disponible en tu imagen.
  
**Opción 2: Usar bind mounts, volúmenes, ...**
  
Esa es otra opción para configurar nuestra aplicación. En este caso, en lugar de sustituír tokens en un fichero, vamos a usar _bind mounts_, para que el **contenedor acceda a un fichero proveído por el host_._ **Si estás en local, usarías _bind mounts_ de Docker. Así, en tu máquina, podrías tener un directorio cfg con varios subdirectorios y su fichero de configuración (p. ej. cfg/dev/conf.json y cfg/qa/conf.json). Cuando quieras ejecutar contra un determinado entorno te basta con lanzar el contenedor usando el parámetro &#8220;-v&#8221; para mapear el fichero que quieras al fichero &#8220;/app/config/conf.json&#8221; (o el que sea, claro).
  
Para más comodidad puedes usar compose y una sección _volumes_ de tu contenedor:

<pre class="EnlighterJSRAW" data-enlighter-language="null">volumes:
  - ./cfg/dev:/app/config/</pre>

En este caso mapeamos el directorio local _cfg/dev_ al directorio _/app/config_ del contenedor.
  
En entornos productivos, ya depende del entorno, pero si usas Kubernetes, pues puedes crear un config map con el contenido del fichero y luego desplegar el config map como un fichero local en el contenedor. Es una solución sencilla y elegante. Echa un vistazo a la [documentación de ConfigMaps][5] para ver como funciona esa técnica.
  
Espero que te sea útil, y como siempre: si tienes cualquier comentario ¡no te cortes!
  
&nbsp;

 [1]: http://geeks.ms/etomas/2018/10/25/generar-imagenes-docker-de-proyectos-spa-de-netcore/
 [2]: https://geeks.ms/etomas/2013/01/22/estn-tus-servicios-rest-en-otro-servidor/
 [3]: https://www.gnu.org/software/gettext/manual/html_node/envsubst-Invocation.html
 [4]: https://geeks.ms/etomas/wp-content/uploads/sites/154/2018/10/envsubst.png
 [5]: https://kubernetes.io/docs/tasks/configure-pod-container/configure-pod-configmap/#add-configmap-data-to-a-volume