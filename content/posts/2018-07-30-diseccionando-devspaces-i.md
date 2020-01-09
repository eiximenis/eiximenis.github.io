---
title: Diseccionando DevSpaces (i)
author: eiximenis

date: 2018-07-30T06:44:22+00:00
geeks_url: /?p=2042
geeks_ms_views:
  - 553
categories:
  - kubernetes

---
DevSpaces es una de las grandes novedades para desarrolladores que nos trae Microsoft (ehm sí... es _otro_ producto más que está en _preview_ :p). Básicamente se trata **de la posibilidad de desplegar parcialmente y depurar nuestros contenedores ejecutándose en un clúster de Kubernetes**. En lo que Hanselman llama un &#8220;entorno que _huele_ como producción&#8221;.
  
¿Pero... cómo funciona esa magia? Vamos a ver, dentro de lo posible, qué ha hecho Microsoft para proporcionarnos esta experiencia, dentro de un Kubernetes completamente normal...
  
Este post **no pretender ser un tutorial de DevSpaces, **ya que la [propia documentación][1] lo explica bastante bien. La idea es intentar ver &#8220;como&#8221; funciona DevSpaces, más que como lo podemos usar.
  
<!--more-->


  
Lo primero es que crees un AKS. Los dos requisitos es que **tenga &#8220;HTTP Application routing habilitado** (si lo creas desde el portal esta opción está marcada por defecto) y que lo crees en una zona con soporte DevSpaces (como p. ej. &#8220;East US&#8221;). Una vez tengas tu cluster creado usa _az aks_ para obtener una configuración para poder usarlo con _kubectl_:

<pre class="EnlighterJSRAW" data-enlighter-language="null">az aks get-credentials -n &lt;nombre-aks&gt; -g &lt;resource-group&gt;</pre>

**Que nos instala Devspaces en el clúster**
  
Vale, ha llegado el momento de usar _devspaces_. Lo primero, asegúrate de que tienes al menos la versión 2.0.38 de la cli de Azure y ya puedes lanzar los comandos:

<pre class="EnlighterJSRAW" data-enlighter-language="null">az extension add --name dev-spaces-preview
az aks use-dev-spaces -n &lt;nombre-aks&gt; -g &lt;resource-group&gt;</pre>

Eso te instalará también la CLI propia de DevSpaces (azds) y configurará el clúster de Kubernetes para DevSpaces. **A nivel de clúster se nos instalaran varias cosas, todas ellas en el espacio de nombres azds:**

  1. Un _deployment_ y su correspondiente servicio ejecutando Tiller. Los despliegues contra Devspaces los haremos usando Helm.
  2. Un _deployment _azds-initializer que crea un _pod_ que ejecuta la imagen [azds/azds-initializer-service][2].
  3. Un _daemon set_ azds-image-prepull que crea un pod en cada nodo que ejecuta la imagen [azds/azds-image-prepull][3]. Estos pods parece ser que hacen un pull de varias imágenes (node, aspnet core, el depurador de netcore, etc) y luego se ponen a dormir &#8220;para siempre&#8221;.

De hecho estos elementos (menos Tiller claro) se instalan vía Helm (aunque para nosotros es transparente). Pero si haces &#8220;_helm ls &#8211;tiller-namespace azds_&#8221; verás una _release_ de Helm llamada azds-<guid>.
  
**Probando Devspaces desde la CLI**
  
Para probar DevSpaces y ver lo que ocurre vamos a partir de una situación inicial:

  1. Un servicio escrito en NetCore llamado Consumer
  2. Un servicio escrito en NetCore llamado Producer

El servicio _Consumer_ hace una llamada a _Producer_ para que este le devuelva un item y poder consumirlo. Estamos hablando de una llamada en HTTP.
  
Para ello, **si quieres seguir el paso a paso, descárgate el código Zip de ambos servicios.  **Si descomprimes el zip, verás dos carpetas (producer y consumer). Levanta ambos y establece (en el consumer) la variable de entorno &#8220;URLS_PRODUCER&#8221; con el valor de la URL raiz del producer. Con esto si navegas a http://<consumer>/api/demo deberías ver un json con dos productos:
  
[<img class="alignnone wp-image-2168 size-full" src="https://geeks.ms/etomas/wp-content/uploads/sites/154/2018/07/demo-local.png" alt="Servicios corriendo en local" width="878" height="412" />][4]
  
Perfecto, ya tenemos dos servicios que se comunican entre ellos 🙂 Ha llegado el momento de desplegarlos en nuestro AKS usando DevSpaces.
  
Para ello vete a la carpeta del **producer** y teclea _**azds prep**_. Ese comando básicamente crea dos Dockerfile (uno estándard multi-stage y otro llamado Dockerfile.develop), así como el chart de Helm para instalar. Si ahora tecleas _**azds up**_, se construirá la imagen de Docker a partir del Dockerfile.develop y se instalará en k8s el chart de Helm. El chart generado te instala un servicio y un deployment que ejecuta tu servicio.
  
La diferencia entre el Dockerfile y el Dockerfile.develop es que el primero es el tradicional multi-stage de netcore, mientras que el segundo usa la imagen del SDK para terminar haciendo un &#8220;dotnet run&#8221;. Si ahora miras las releases de helm deberías ver la release del _producer_ instalada. Esta release nos ha instalado un servicio (ClusterIP) , un deployment y el pod. ¡Genial!. Pero... indaguemos un poco más.
  
Si inspeccionas el _deployment_ verás que consta de un contenedor (nuestro producer, claro) pero fíjate la imagen del contenedor (puedes usar _kubectl get deployment producer -o yaml_ para ver la descripción del _deployment_). En mi caso tengo:

<pre class="EnlighterJSRAW" data-enlighter-language="json">spec:
  containers:
  - image: producer:mindaro-012f89e6
    imagePullPolicy: Never</pre>

**¿De donde sale esa imagen &#8220;_producer:mindaro-012f89e6_&#8220;? **La clave nos la da el _imagePullPolicy_, que está a Never. Eso significa que Kubernetes nunca hará un pull de la imagen, es decir **esta debe estar instalada en el clúster**. ¿Pero como llega al clúster? En mi máquina no está, y obviamente no está en ningún registro. Pues, la imagen está en el clúster **porque se construye allí**, es decir el proceso de build ocurre en el clúster, por lo que, la imagen ya está allí _antes_ de que se incie el _pod_ (de ahí claro que el _imagePullPolicy_ esté a _Never_, no queremos que Kubernetes se descargue esa imagen de ningún lugar, puesto que en ningún lugar existe).
  
Vale... ¿pero _quien_ dentro del clúster construye esa imagen? Bueno... Pues lo hace un _[init container][5]. _Un día escribiré sobre ellos en el blog, pero por ahora quédate en que es un contenedor que se ejecuta _antes_ de iniciar un _pod_. En este caso, este contenedor es quien ejecuta el _docker build_ y crea la imagen en el cluster. Si tecleas _kubectl describe pod <nombre-pod-producer>_ verás que parte de la salida es algo parecido a:
  
[<img class="alignnone size-large wp-image-2169" src="https://geeks.ms/etomas/wp-content/uploads/sites/154/2018/07/mindaro-build-1024x268.png" alt="salida de kubetl describe" width="660" height="173" />][6]
  
En el recuadro rojo **he señalado el nombre del contenedor de inicialización (mindaro-build)** y en la elipse amarilla **uno de los parámetros que se le pasan, que puedes ver que se corresponde con el nombre de la imagen del _producer_**. Observa que **otro parámetro es el Dockerfile.Develop**. Este es pues, quien nos construye la imagen.
  
Vale, ahora toca irnos a la carpeta del **consumer** y repetir los pasos,  aunque ahora usa _**azds prep &#8211;public**_. Con eso le indicas a DevSpaces que el chart que se genere exponga un endpoint público para el servicio. Al igual que antes, usa **_azds up_**_ _para lanzar el servicio. Esto, al igual que antes, te creará un servicio, el deployment ¡**y un ingress con un DNS propio!** (debido a que hemos usado &#8211;public):
  
[<img class="alignnone size-full wp-image-2172" src="https://geeks.ms/etomas/wp-content/uploads/sites/154/2018/07/azds-kubectl.png" alt="Vista de deployments, servicios y ing" width="764" height="287" />][7]
  
Quien proporciona el DNS es el &#8220;Http Application Routing&#8221; que en el fondo es una instalación preconfigurada de [external-dns][8], un producto que puedes instalar en cualquier Kubernetes para configurar DNS externos y vincularlos a servicios expuestos. En AKS se trabaja, obviamente, con Azure DNS y gracias al addon de &#8220;Http application routing&#8221; ya lo tenemos todo configurado. Eso permite tener distintos servicios, expuestos bajo el mismo ingress controller (por lo tanto misma IP pública) pero enlazados a distintas direcciones DNS.
  
Si ahora tecleas &#8220;**azds list-up&#8221;** verás una lista con los dos servicios que estás ejecutando y con &#8220;**azds list-uris**&#8221; verás una lista con los puntos de acceso a los distintos servicios:
  
[<img class="alignnone size-full wp-image-2171" src="https://geeks.ms/etomas/wp-content/uploads/sites/154/2018/07/azds-list.png" alt="Salida de azds list-up y list-uris" width="564" height="339" />][9]
  
De hecho si navegas a la URL indicada con el path /api/demo debería aparecerte una página de error. Esto es porque **no hemos establecido la configuración del _consumer_ (no le hemos indicado la url del _producer_)_._**
  
Vamos a establecer esta configuración. **En la carpeta raíz del consumer** (es la misma carpeta donde hay el values.yaml) **crea un fichero llamado values.dev.yaml** con este contenido:

<pre class="EnlighterJSRAW" data-enlighter-language="json">secrets:
  urls:
    producer: http://producer</pre>

**Nota importante:** En la [documentación][10] pone que el fichero _values.dev.yaml_ debe crearse _en el directorio charts/consumer_ (donde está el values.yaml). A mi, eso no me ha funcionado. No sé si es un error de la doc o qué, pero el fichero values.dev.yaml debe estar en el mismo directorio donde hay el fichero azds.yaml.
  
**Ojo: **Es un poco curioso como se mapean estos ficheros de valores a las variables de entorno que termina recibiendo el contenedor. Así este fichero termina generando la variable de entorno &#8220;urls\_producer&#8221;. Observa que para este mismo fichero tratado como si fuese un appsettings en netcore usaríamos la variable de entorno &#8220;urls\__producer&#8221; (doble subrayado) o &#8220;urls:producer&#8221; en Windows. Es algo pues, a tener presente.
  
El siguiente punto es **modificar el fichero azds.yaml dentro de la carpeta raíz del _consumer_** para que use este fichero de secretos que hemos creado. Para ello agregamos la entrada &#8220;values&#8221; **dentro de container (que ya debe existir)_:_**

<pre class="EnlighterJSRAW" data-enlighter-language="json">container:
  values:
    - "charts/consumer/values.dev.yaml"</pre>

Ahora puedes eliminar el consumer (azds down) y volverlo a instalar con azds up.
  
Si ahora navegas a la URL (DNS) asignado al _consumer_ (recuerda que la puedes ver con azds list-uris) ya te debería funcionar. **Nota:** Los DNS pueden tardar un cierto tiempo a refrescarse (bueno... ojo que lo de &#8220;cierto tiempo&#8221; en pruebas que he hecho yo, ha excedido una hora. La URL aparece en estado de &#8220;Pending&#8221; durante este tiempo.
  
Así que es posible que quizá el principio no te funcione. Lo que sí te deberían funcionar son las urls _tunneled_ (que te permiten acceder a los servicios del clúster mediante direcciones http://localhost:puerto). Estas urls también aparecen con &#8220;azds list-uris&#8221;.
  
Bueno: Lo dejaremos aquí en este primer post. Como te dije, la idea era ver un poco como funcionaba DevSpaces, y en post sucesivos intentaré ir diseccionando más partes!
  
Saludos!

 [1]: https://docs.microsoft.com/es-es/azure/dev-spaces/azure-dev-spaces
 [2]: https://hub.docker.com/r/azds/azds-initializer-service/
 [3]: https://hub.docker.com/r/azds/azds-image-prepull/
 [4]: https://geeks.ms/etomas/wp-content/uploads/sites/154/2018/07/demo-local.png
 [5]: https://kubernetes.io/docs/concepts/workloads/pods/init-containers/
 [6]: https://geeks.ms/etomas/wp-content/uploads/sites/154/2018/07/mindaro-build.png
 [7]: https://geeks.ms/etomas/wp-content/uploads/sites/154/2018/07/azds-kubectl.png
 [8]: https://github.com/kubernetes-incubator/external-dns
 [9]: https://geeks.ms/etomas/wp-content/uploads/sites/154/2018/07/azds-list.png
 [10]: https://docs.microsoft.com/es-es/azure/dev-spaces/how-to/manage-secrets