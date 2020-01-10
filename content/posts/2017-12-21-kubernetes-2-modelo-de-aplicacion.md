---
title: Kubernetes (2) – Modelo de aplicación

author: eiximenis

date: 2017-12-21T11:18:03+00:00
geeks_url: /?p=1948
geeks_ms_views:
  - 1305
categories:
  - docker
  - kubernetes
  - Sin categoría

---
Si conoces compose conocerás su &#8220;modelo de aplicación&#8221;. Es un modelo sencillo, contiene básicamente _servicios._ Un servicio en compose no es nada más que una imagen de Docker y su configuración asociada.  Luego cuando levantamos una aplicación compose con `docker-compose up` se crea uno (o varios) contenedor por cada servicio y listos.
  
Pero Kubernetes tiene su propio modelo de aplicación radicalmente distinto. En este post vamos a ver (de forma simplificada) cual es el modelo de aplicación que tiene Kubernetes y ¡desplegaremos nuestra primera aplicación!
  
<!--more-->


  
**Pods**
  
En k8s no despliegas contenedores, despliegas _pod_. Un _pod_ es la unidad mínima de despliegue (y escalado) de Kubernetes. Podríamos definir un pod como un conjunto de contenedores que se despliegan, escalan juntos y comparten un _port-space_. **Dos contenedores ejecutándose en el mismo pod se comunicaran entre ellos a través de ****_localhost_** y, dado que comparten el espacio de puertos, **no pueden abrir ambos el mismo puerto**.
  
**Servicios**
  
Los pods son como las cucarachas: tarde o temprano se mueren. Y cuando eso pasa no son &#8220;resucitados&#8221;, en todo caso si es necesario Kubernetes creará pods nuevos para reemplazarlos. Aunque cada pod tiene su propia IP interna esta no es &#8220;fiable&#8221; porque los pods pueden ser movidos en caso necesario.
  
Pero tarde o temprano un contenedor ejecutándose en un pod requerirá a otro contenedor ejecutándose en otro pod distinto, así que necesitamos tener un mecanismo que permita esta comunicación. Y ese mecanismo son los servicios. En Kubernetes un servicio engloba uno o varios pods **bajo una IP estable****. **Básicamente cuando un contenedor quiere hablar con otro contenedor (en otro pod) lo hace a través del servicio. Las peticiones al servicio son enrutadas automáticamente por Kubernetes al pod.
  
Imagina que tienes un contenedor con una API y lo quieres meter en Kubernetes.  Colocas el contenedor en un pod. Y añades un servicio por encima. Luego tienes otro contenedor con el cliente y lo metes en otro pod distinto. Ahora cuando el cliente se quiera comunicar con la API llamará a la IP del servicio. Si quieres escalar la API a más instancias, Kubernetes creará pods adicionales para la API, pero estaran dentro del mismo servicio. El cliente sigue hablando con el servicio y su petición es enrutada a uno de los pods. El cliente no tiene porque saber que la API está escalada ni nada. Todo eso corre del lado de Kubernetes.
  
**Volúmenes**
  
No hay mucho que añadir aquí: si un pod termina y Kubernetes debe reiniciarlo, este pod se reiniciará a partir de un estado limpio, por lo que todo lo que el contenedor haya escrito en su &#8220;disco&#8221; se perderá. Por otro lado a veces es necesario transferir ficheros entre contenedores del mismo pod. Un volumen en Kubernetes es lo mismo que un volumen en Docker: un espacio de almacenamiento persistente que se monta en el sistema de ficheros del contenedor.
  
**Despliegues (Deployments)**
  
Al contrario que los servicios, **no es normal crear pods directamente en Kubernetes**. En su lugar lo que se crean son &#8220;despliegues&#8221;. En un despliegue básicamente lo que haces es definir qué contenedores quieres y como los quieres escalar y dejas que Kubernetes cree los pods necesarios. En definitiva, un deployment me define como va a ser un pod (qué contenedores contendrá) y cuantas instancias de este tipo de pod quiero. Y Kubernetes creará los pods reales. Además Kubernetes garantiza que el deployment siempre se cumple, es decir si he especificado que quería tres instancias de un determinado pod y uno de los pods reales muere, Kubernetes automáticamente creará otro.
  
Cuando se crea un depoyment en Kubernetes, se crea automáticamente otro objeto, llamado _Replica Set_ que es, precisamente, el responsable de garantizar que el deployment siempre se cumple.
  
Una vez tengo un depoyment en k8s puedo modificarlo (p. ej. modificar el número de instancias de un tipo de pod) y el Replica Set automáticamente adaptará el estado actual del clúster al nuevo deseado (creando o destruyendo los pods necesarios). Así pues el deployment es un mecanismo declarativo para configurar mi aplicación.
  
**Replication Controllers**
  
Menciono los _Replication Controllers_ solo para completitud: son la manera &#8220;antigua&#8221; de gestionar el escalado en Kubernetes. Cubren la misma función que los Replica Sets, pero funcionan imperativamente. Es decir, en lugar de usar una forma declarativa (como es el deployment) para definir cual es el estado de los pods, uso comandos con la herramienta _kubectl_ para modificar el estado. El consejo actual es usar deployments (y por lo tanto, Replica Sets) en lugar de replication controllers.
  
**Hello k8s: Desplegando nuestra primera aplicación en k8s**
  
Vale, ha llegado el momento de poner todo eso en marcha. Así que vamos a desplegar una aplicación muy sencilla en Kubernetes: un solo contenedor. **Vamos a desplegarlo en MiniKube, así que lo primero es hacer _minikube start_ para tener un contexto de trabajo contra el clúster local**.
  
Voy a usar la imagen _dockercampusmvp/go-hello-world_ que es una imagen muy sencilla que levanta un servidor web con un contador que se incrementa cada vez que recibe una petición.
  
**El primer paso es definir el deployment** donde indicaremos que queremos un pod con un contenedor de nuestra imagen. Un deployment se define en un fichero yaml:

<pre class="EnlighterJSRAW" data-enlighter-language="md">apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: hello-world
spec:
  paused: true
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
      - name: hello-world-ctr
        image: dockercampusmvp/go-hello-world
        imagePullPolicy: Always
        ports:
        - containerPort: 80</pre>

Vale, analicemos este fichero yaml. Las dos primeras líneas identifican que se trata de un fichero que describe un deployment. Luego viene la sección _metadata_ donde simplemente se añaden metadatos al objeto deployment  (en este caso el nombre).
  
La sección _spec_ es la interesante porque especifica como es el deployment: En nuestro caso inicialmente estará parado (_paused_ vale true) y luego **dentro de _template _especificamos la plantilla de pod que queremos**. Por un lado hay metadatos de la plantilla del pod. Ahí es importante la sección _labels_ que nos permite etiquetar esta plantilla de pod para poder referenciarla luego. En este caso añadimos una sola etiqueta llamada _app_ y cuyo valor es _myapp_. Y luego **dentro de _spec_ viene la especificación de la plantilla del pod**, es decir los contenedores. Observa que _containers _es una lista de los contenedores que debe contener el pod. En nuestro caso tenemos una sola entrada. Ahí especificamos:

  * _name_: Nombre del contenedor
  * _image_: Imagen a partir de la cual se creará el contenedor
  * _imagePullPolicy_: Eso indica a k8s cuando debe hacer un _pull_ de esa imagen.
  * _ports_: Definimos los puertos del contenedor

Bien, una vez tenemos ese fichero debemos crear el deployment en Kubernetes. Para ello usamos la herramienta kubectl:

<pre class="EnlighterJSRAW" data-enlighter-language="generic">kubectl create -f deployment.yaml</pre>

El parámetro _-f_ especifica el fichero yaml a desplegar (en mi caso, soy así de original, se llama _deployment.yaml_). Cuando haya finalizado kubectl nos responderá con un mensaje similar a _&#8220;deployment &#8220;hello-world&#8221; created&#8221;._
  
Vamos a usar _kubectl_ un poco más para inspeccionar que tenemos en el clúster. Lo primero es mirar si tenemos algún pod usando _kubectl get pods. _Kubectl te dirá que no hay ningún pod... Igual te extraña ya que hemos creado un deployment pero recuerda que estaba pausado. De hecho, si tecleas _kubectl get deployments_ verás nuestro deployment. Vale, vamos a ponerlo en marcha:

<pre class="EnlighterJSRAW" data-enlighter-language="null">kubectl rollout resume deployment/hello-world</pre>

Con este comando indicamos a Kubernetes que ponga en marcha el deployment &#8220;hello-world&#8221;, y ahora sí que sí: si tecleas _kubectl get pods_ deberás ver tu pod en marcha:

<pre class="EnlighterJSRAW" data-enlighter-language="null">NAME                           READY     STATUS    RESTARTS   AGE
hello-world-6cc89c7457-5sl4v   1/1       Running   0          42s</pre>

El nombre del pod será distinto ya que el sufijo es aleatorio. ¡**Bien! Has desplegado tu contenedor en Kubernetes pero... ¿como acceder a él? **Vale, el pod tiene una IP (la puedes ver tecleando _kubectl describe pod nombre_pod | findstr IP_), pero esa IP es interna y además puede variar. La verdad es que **para acceder al pod necesitas un servicio**.
  
Pues bien, para desplegar un servicio necesitamos otro fichero yaml:

<pre class="EnlighterJSRAW" data-enlighter-language="null">apiVersion: v1
kind: Service
metadata:
  labels:
    app: myapp
  name: hello-world
spec:
  type: NodePort
  ports:
  - port: 80
  selector:
    app: myapp</pre>

Miremos un poco el fichero yaml: Las dos primeas líneas indican que es un servicio, y luego viene una sección de metadatos. En este caso asignamos el nombre del servicio (hello-world) y una etiqueta (app con el valor myapp). Como antes **la sección _spec_ es la que especifica el servicio**. Lo primero es el tipo de servicio, en este caso NodePort (ya veremos más tipos) y luego los puertos que queremos usar, en este caso el 80. La idea ahí es que queremos exponer un port que escucha por el puerto 80, así que ese es el puerto que queremos acceder a través del servicio.
  
Pero nos queda seleccionar qué pod queremos exponer a través del servicio y para ello usamos la sección _selector_. **En la sección _selector_ colocamos etiquetas que nos permiten seleccionar el pod (o los pods)** del servicio. En este caso expondremos todos los pods que tengan una etiqueta llamada _app_ y con valor _myapp_... que si repasas el fichero del deployment es lo que tenemos dentro de los metadatos de la plantilla. Con eso indicamos que este servicio expondrá este pod.
  
Ahora desplegamos el servicio con _kubectl -f service.yaml_ (otra vez en un alarde de originalidad he llamado service.yaml al fichero). Una vez haya terminado puedo mirar mis servicios con _kubectl get services_ (debería ver dos, el _hello-world_ y uno llamado _kubernetes_). Cada servicio tiene una IP (CLUSTER-IP) pero es interna del cluster.
  
Observa el valor de _nodes_ en la columna EXTERNAL-IP. Eso es debido a que nuestro servicio es de tipo _NodePort_. Eso hace que para acceder al servicio desde fuera del nodo podamos usar la ip <NodeIP>:<NodePort>. Pero para ello debemos saber cual es la IP del nodo.
  
**Vale, empecemos por el método fácil**: si tecleas _minikube service hello-world &#8211;url_ verás una URL. Esta es la URL externa (IP y puerto) que puedes llamar para acceder al servicio. Y ya está, no necesitas saber nada más. En este caso MiniKube nos da automáticamente la IP externa del nodo. Esto lo podemos hacer porque estamos bajo un contexto de minikube y por lo tanto minikube &#8220;nos puede ayudar&#8221;.
  
**Vale, ahora el &#8220;difícil&#8221; o sin usar minikube**. Lo primero es usar el comando _kubectl get nodes_ para obtener una lista de los nodos (deberías tener solo uno llamado _minikube_). Ahora lanza el comando _kubectl describe node minikube_. Esto te imprime toda la información del nodo por pantalla. Busca una sección llamada _Addresses_ que tendrá una información similar a:

<pre class="EnlighterJSRAW" data-enlighter-language="null">Addresses:
  InternalIP:   192.168.1.42
  Hostname:     minikube</pre>

El valor de _InternalIP_ es el que debes usar para la IP del nodo.  Ahora te falta el puerto. Para ello mira el valor de _kubectl get services_:

<pre class="EnlighterJSRAW" data-enlighter-language="null">NAME          CLUSTER-IP       EXTERNAL-IP   PORT(S)        AGE
hello-world   10.105.233.193   &lt;nodes&gt;       80:30325/TCP   20m
kubernetes    10.96.0.1        &lt;none&gt;        443/TCP        2d</pre>

Observa como hay un mapeo del puerto 80 al 30325. Pues bien **el puerto 30325 es el que debes usar**. Así la IP para acceder al servicio (y por lo tanto al pod) desde el exterior es http://192.168.1.42:30325 (que es el valor que te habrá devuelto _minikube service hello-world &#8211;url_).
  
**¡Y listos! Lo dejamos aquí. **Has desplegado tu primera aplicación con MiniKube. Si vas accediendo a la URL verás como el contador se va incrementando!
  
&nbsp;
  
&nbsp;