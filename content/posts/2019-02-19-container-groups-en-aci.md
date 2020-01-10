---
title: Container Groups en ACI

author: eiximenis

date: 2019-02-19T22:18:45+00:00
geeks_url: /?p=2299
geeks_ms_views:
  - 610
categories:
  - azure
  - docker
  - serverless
tags:
  - aci
  - aks

---
Este es el segundo post sobre Azure Container instances. En el [post anterior][1] vimos lo fácil que era publicar un contenedor y ejecutarlo usando ACI y discutimos algunas de sus limitaciones.
  
En este post veremos que ACI nos permite **ejecutar grupos de contenedores**, de forma igualmente sencilla y así tener algunos escenarios más complejos.
  
<!--more-->


  
**Grupos de contenedores en ACI**
  
ACI permite **ejecutar más de un contenedor** que se ejecutarán en un mismo _host_ y compartirán espacio de red y memoria. Viene a ser el mismo concepto de un _pod_ en Kubernetes. Por lo tanto si dos contenedores se ejecutan en el mismo grupo de contenedores **no podrán ambos abrir el mismo puerto y se comunicarán entre ellos usando _localhost_.**
  
No sé (creo que no) si existe una manera de crear un grupo de contenedores en ACI usando el portal, así que bueno, en este caso vamos a tirar de la CLI y de ficheros YAML. Si (cada uno tiene sus gustos) en lugar de YAML prefieres usar plantillas ARM, que sepas que también se puede. Usar ARM te permite usar las herramientas tradicionales de ARM y crear grupos de contenedores a la vez que otros recursos todo de golpe.
  
En este ejemplo vamos a ver YAML, porque, la verdad, la sintaxis del fichero es mucho más sencilla y compacta que la sintaxis de la plantilla ARM y usaremos la CLI de ACI (_az container_) para crear el grupo de contenedores.
  
Vamos a ver dos ejemplos: en el primero vamos a desplegar una web y su API en un mismo grupo. Por supuesto, eso implica que la web no puede ser SPA, ya que todas las llamadas de la web a la API deben realizarse desde código de servidor (usando _HttpClient_) contra _localhost_. En el **segundo ejemplo veremos** como, gracias a los grupos de recursos, **podemos añadir soporte TLS/SSL a un contenedor en ACI** (y así saltarnos una de sus limitaciones).
  
**Ejemplo 1: Web y API**
  
Vamos a usar los mismos ejemplos (sí, lo sé, soy muy vago xD) del [post pasado de devspaces][2] ya que precisamente allí teníamos una web y una API. Parto **de la situación en qué tengo ambas imágenes (apiserver y webclient) en un registro de Docker** (un ACR en mi caso).
  
Este sería el fichero YAML en nuestro caso (yo lo he llamado deployment-group.yaml):

```json
apiVersion: 2018-06-01
location: eastus
name: test-web-api
properties:
  containers:
  - name: webclient
    properties:
      image: testaciedu.azurecr.io/webclient:latest
      environmentVariables:
        - name: "ApiUrl"
          value: "http://localhost:8080"
      resources:
        requests:
          cpu: 1
          memoryInGb: 1.5
      ports:
      - port: 80
  - name: apiserver
    properties:
      image: testaciedu.azurecr.io/apiserver:latest
      environmentVariables:
        - name: "ASPNETCORE_URLS"
          value: "http://0.0.0.0:8080"
      resources:
        requests:
          cpu: 1
          memoryInGb: 1.5
      ports:
      - port: 8080
  imageRegistryCredentials:
  - server: testaciedu.azurecr.io
    username: testaciedu
    password: "password-del-acr"
  osType: Linux
  ipAddress:
    type: Public
    ports:
    - protocol: tcp
      port: '80'
    - protocol: tcp
      port: '8080'
tags: null
type: Microsoft.ContainerInstance/containerGroups
```

Observa como definimos dos contenedores (webclient y apiserver) donde el primero escucha por el puerto 80 y el segundo por el 8080 (recuerda: comparten espacio de puertos, por lo tanto no pueden usar el mismo). La web usa la variable de entorno &#8220;ApiUrl&#8221; con la url de la API (el otro contenedor) que como puedes ver tiene el valor de _localhost:8080_.
  
Ahora solo nos queda usar la CLI de ACI para desplegar el grupo:

<pre class="EnlighterJSRAW" data-enlighter-language="null">az container create -g test-aci --file deployment-group.yaml</pre>

Al cabo de un rato ya deberíamos ver en el portal el grupo de contenedores:
  
![Container group en el portal de Azure](https://geeks.ms/etomas/wp-content/uploads/sites/154/2019/02/container-group-1024x271.png)
  
Si accedemos a la IP dada (que la podemos ver en la pestaña _Overview_) veremos como, efectivamente, la web está llamando a la API:
  
![La web funcionando y llamando a la API](https://geeks.ms/etomas/wp-content/uploads/sites/154/2019/02/web-ok-1.png)
  
En este caso, la API también está expuesta al exterior via el puerto 8080, **pero eso no es obligatorio**. En nuestro caso lo está ya que en la sección _ipAddress_ del fichero YAML exponemos el puerto 8080. Pero, insisto, no tenemos por qué: **aunque el puerto 8080 no se exponga al exterior, la API sigue siendo accesible a través de dicho puerto por el resto de contenedores del grupo** (usando _localhost_).
  
**Ejemplo 2: Añadiendo TLS/SSL al grupo de contenedores**
  
Veamos un ejemplo de como añadir soporte para TLS al grupo de contenedores apoyándonos de un contenedor nginx. Básicamente, agregaremos un tercer contenedor nginx con el certificado, expondremos dicho contenedor al exterior y los otros dos (la web y la api) no los expondremos.
  
Vamos a usar la imagen estándard de NGINX lo que nos lleva a ver **como podemos pasarle a la imagen el fichero nginx.conf** que configura el servidor. Así que **tenemos que ver como montar un volumen para pasarle dicho fichero al contenedor**.
  
Los volúmenes se montan en un Azure Files, así que debes tener uno creado:

```bash
az storage account create -g test-aci -n testacistorage --sku Standard_LRS
az storage share create -n testacishare --account-name testacistorage
```


Nos falta el ficherol nginx.conf:

```
pid /tmp/nginx.pid;
worker_processes 1;
events {
    worker_connections 1024;
}
http {
    server_tokens off;
    add_header X-Frame-Options SAMEORIGIN;
    add_header X-Content-Type-Options nosniff;
    add_header X-XSS-Protection "1; mode=block";
    client_body_temp_path /tmp/client_body;
    fastcgi_temp_path /tmp/fastcgi_temp;
    proxy_temp_path /tmp/proxy_temp;
    scgi_temp_path /tmp/scgi_temp;
    uwsgi_temp_path /tmp/uwsgi_temp;
    gzip on;
    gzip_comp_level 6;
    gzip_min_length 1024;
    gzip_buffers 4 32k;
    gzip_types text/plain application/javascript text/css;
    gzip_vary on;
    keepalive_timeout 65;
    proxy_buffer_size 128k;
    proxy_buffers 4 256k;
    proxy_busy_buffers_size 256k;
    server {
        listen 80;
        location / {
            proxy_pass http://localhost:8088;
            proxy_redirect off;
            proxy_set_header Host $host;
        }
    }
}
```

Este mismo servirá. Básicamente NGINX escuchará por el puerto 80 y redirigirá las peticiones de &#8220;/&#8221; a &#8220;localhost:8088&#8221;.
  
**Debes subir este fichero nginx.conf al share de Azure Files**. Puedes hacer eso desde el portal, usando Azure Storage Explorer [o la CLI][5] (_az storage file upload_).
  
Dado que ahora es NGINX quien escucha por el puerto 80, la web ya no puede escuchar por este puerto (recuerda: se comparte el espacio de red), así que modificaremos la configuración de la web para que escuche por otro puerto como el 8088.
  
Veamos como queda el fichero YAML ahora:

<pre class="EnlighterJSRAW" data-enlighter-language="null">apiVersion: 2018-06-01
location: eastus
name: test-web-api
properties:
  volumes:
  - name: nginxvol
    azureFile:
      shareName: testacishare
      storageAccountName: testacistorage
      storageAccountKey: "Clave de acceso del Storage"
  containers:
  - name: nginx
    properties:
      image: nginx
      volumeMounts:
      - name: nginxvol
        mountPath: /etc/nginx
      resources:
        requests:
          cpu: 1
          memoryInGb: 1.5
      ports:
      - port: 80
  - name: webclient
    properties:
      image: testaciedu.azurecr.io/webclient:latest
      environmentVariables:
        - name: "ApiUrl"
          value: "http://localhost:8080"
        - name: "ASPNETCORE_URLS"
          value: "http://0.0.0.0:8088"
      resources:
        requests:
          cpu: 1
          memoryInGb: 1.5
      ports:
      - port: 8088
  - name: apiserver
    properties:
      image: testaciedu.azurecr.io/apiserver:latest
      environmentVariables:
        - name: "ASPNETCORE_URLS"
          value: "http://0.0.0.0:8080"
      resources:
        requests:
          cpu: 1
          memoryInGb: 1.5
      ports:
      - port: 8080
  imageRegistryCredentials:
  - server: testaciedu.azurecr.io
    username: testaciedu
    password: "password-ACR"
  osType: Linux
  ipAddress:
    type: Public
    ports:
    - protocol: tcp
      port: '80'
tags: null
type: Microsoft.ContainerInstance/containerGroups</pre>

  1. Definimos el volúmen llamado _nginxvol_ usando el share _testacishare_ (en el storage _testacistorage_).
  2. Aplicamos este volúmen al contenedor _nginx_ a través de la sección &#8220;_volumeMounts_&#8220;: indicamos el volúmen (el _nginxvol_ creado anteriormente) e indicamos una ruta dentro del contenedor (_/etc/nginx_ que es donde NGINX espera el fichero nginx.conf)

Al margen de esos dos puntos definimos el tercer contenedor (nginx) y hacemos que escuche por el puerto 80. Observa como hemos modificado el contenedor webclient para que escuche por el 8088 **y como solo exponemos el puerto 80 públicamente**.
  
Bien, si despliegas este grupo de contenedores y accedes a la IP que te dé, verás que el resultado es el mismo de antes (se muestra la web). Pero ahora nos la está sirviendo nginx, no kestrel:
  
[<img class="alignnone size-full wp-image-2308" src="https://geeks.ms/etomas/wp-content/uploads/sites/154/2019/02/nginx-header.png" alt="Header de nginx" width="701" height="249" />][6]
  
Bien, ahora ya **solo queda añadir el certificado TLS a nginx y exponer el puerto 443**. En este caso **usaré un certificado auto-firmado, pero, obviamente, en la realidad se usaría un certificado emidido por una CA autorizada.**
  
Para crear nuestro certificado auto-firmado hay varias maneras, yo usaré _OpenSSL_ en Linux. Si tienes Windows puedes usar el WSL o bien [alguna otra alternativa][7].
  
En Linux (o WSL) puedes usar el comando:

<pre class="EnlighterJSRAW" data-enlighter-language="shell">sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout localhost.key -out localhost.crt
</pre>

Esto te preguntará los datos del certificado (país, ciudad, provincia), entra lo que quieras ahí. Una vez ejecutado te generará dos ficheros (_localhost.key_ y _localhost.crt_). El primero es la clave privada y el segundo el certificado SSL.
  
Ahora **debes subir ambos ficheros al Azure File Share** y debemos editar el fichero nginx.conf para que la sección _server_ quede tal y como sigue:

<pre class="EnlighterJSRAW" data-enlighter-language="null">server {
    listen 80;
    listen 443 ssl http2;
    listen [::]:443 ssl http2;
    ssl_certificate /etc/nginx/localhost.crt;
    ssl_certificate_key /etc/nginx/localhost.key;
    location / {
        proxy_pass http://localhost:8088;
        proxy_redirect off;
        proxy_set_header Host $host;
    }
}</pre>

Hemos indicado a nginx que escuche por el puerto 443 y le indicamos donde está el certificado SSL y la clave privada.
  
Ahora debemos modificar el fichero YAML de ACI, ya que debemos:

  * Abrir el puerto 443 del contenedor nginx
  * Exponer el puerto 443 al exterior

Para ello basta modificar la sección _ports_ del contenedor _nginx_:

<pre class="EnlighterJSRAW" data-enlighter-language="json">ports:
- port: 80
- port: 443</pre>

Y también la sección _ports_ de _ipAddress_:

<pre class="EnlighterJSRAW" data-enlighter-language="json">ipAddress:
  type: Public
  ports:
  - protocol: tcp
    port: '80'
  - protocol: tcp
    port: '443'</pre>

Si ahora despliegas el grupo de contenedores **observarás que ya puedes acceder via https**. Por supuesto, en este caso **el navegador se te quejará de que el certificado es inválido** (claro es un autofirmado generado por nosotros), pero bueno... eso no es relevante en lo que al post se refiere 🙂 En un entorno real instalarías un certificado SSL real (p. ej. uno emitido por Let&#8217;s Encrypt) y... ¡listos!
  
[<img class="alignnone size-large wp-image-2310" src="https://geeks.ms/etomas/wp-content/uploads/sites/154/2019/02/invalid-cert-1024x562.png" alt="Acceso via https con el error de certificado" width="660" height="362" />][8]
  
En resúmen: hemos visto como podemos desplegar más de un contenedor en un grupo de contenedores ACI y como esos **comparten espacio de red y se comunican entre ellos via _localhost_.** Hemos visto un ejemplo (desplegando una web y su API) y luego hemos visto como aprovechar esta característica y usar un contenedor NGINX para dotar a ACI de soporte para TLS/SSL.
  
¡Espero que os haya resultado interesante!
  
**Posts de esta serie**
  
Iré añdiendo los enlaces a medida que los posts se publiquen 🙂

  1. [ACI: Serverless containers][9]
  2. Container groups en ACI (este)
  3. ACI y AKS: nodos virtuales
  4. ACI y AKS: virtual kubelet

¡Espero que os sea interesante!

 [1]: https://geeks.ms/etomas/2019/02/01/aci-azure-container-instances-serverless-containers/
 [2]: https://geeks.ms/etomas/2019/02/13/trabajando-con-aks-devspaces/
 [3]: https://geeks.ms/etomas/wp-content/uploads/sites/154/2019/02/container-group.png
 [4]: https://geeks.ms/etomas/wp-content/uploads/sites/154/2019/02/web-ok-1.png
 [5]: https://docs.microsoft.com/es-es/azure/storage/files/storage-how-to-use-files-cli
 [6]: https://geeks.ms/etomas/wp-content/uploads/sites/154/2019/02/nginx-header.png
 [7]: https://stackoverflow.com/questions/2355568/create-a-openssl-certificate-on-windows
 [8]: https://geeks.ms/etomas/wp-content/uploads/sites/154/2019/02/invalid-cert.png
 [9]: http://geeks.ms/etomas/2019/02/01/aci-azure-container-instances-serverless-containers/