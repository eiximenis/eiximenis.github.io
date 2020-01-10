---
title: Kubernetes (3) – Controladores Ingress

author: eiximenis

date: 2018-01-03T16:48:58+00:00
geeks_url: /?p=1973
geeks_ms_views:
  - 1307
categories:
  - docker
  - kubernetes
  - Sin categoría

---
Seguimos con esta serie de posts sobre Kubernetes. Los posts anteriores:

  1. [Componentes de Kubernetes][1] (donde vimos los distintos componentes de Kubernetes y como usar [Minikube][2] para ejecutarlo en local).
  2. [Modelo de aplicación][3] (donde vimos como crear nuestra primera aplicación en k8s).

En este tercer post veremos que son los recursos _ingress_ y los controladores _ingress_ y que ventajas nos aportan.
  
<!--more-->


  
En el post anterior vimos como se usaban los servicios en Kubernetes y vimos como exponer un servicio para que fuese accesible desde el exterior. Pero **exponer un servicio directamente no es la única manera de que este sea accesible desde el exterior**. Usar un controlador _ingress_ es la otra opción.
  
Vamos a redefinir el ejemplo anterior para que use _ingress _en lugar de exponer directamente el servicio. Cuando se usa _ingress_ hay que diferenciar entre dos conceptos:

  * El **recurso _ingress_** que no es más que una definición de reglas que indica como se enrutan los servicios hacia el exterior.
  * El **controlador _ingress_** que es un contenedor que enruta las peticiones hacia los servicios correspondientes en base a la definición del recurso _ingress_.

Lo importante es notar que **Kubernetes no incorpora ningún controlador _ingress_, **todo lo que proporciona es una API propia para leer la definición del recurso _ingress_. Eso puede parecer poca cosa, pero al final lo que se ha conseguido es que **haya una gran cantidad de controladores _ingress_**. En general cualquier contenedor que ejecute un software capaz de actuar como un proxy inverso (p. ej. nginx) puede actuar de controlador _ingress_. ¿La ventaja? La configuración del recurso _ingress_ forma parte de la definición de tu aplicación Kubernetes.
  
En el [post anterior][3] terminamos exponiendo un servicio Go al exterior. Veamos como podemos hacer esto mismo usando _ingress_. Lo primero a destacar es que los YAML que definían el servicio y el deployment son los mismos, con la excepción de que vamos a quitar el tipo _NodePort_ al servicio. Al margen de eso, no hay ningún otro cambio.
  
**Nota:** Si vas a realizar lo descritom en este post, asegúrate de empezar con un cluster vacío (puedes borrar con _minikube delete_ el cluster y crearlo otra vez). Una vez tengas el clúster vacío asegúrate de seguir todos los pasos del post anterior: despliega el _deployment_, luego el servicio **pero quitando la línea _type: NodePort_ del fichero YAML**.  Haciendo un _kubectl get services_ te debería aparecer el servicio _hello-world_ pero ahora con el valor de EXTERNAL-IP a <none> (en lugar de <nodes>). **Este es el estado inicial del clúster que se presupone en este post**.
  
Veamos como es el recurso _ingress_, como ya habrás deducido se define mediante otro fichero YAML (llámalo _ingress.yaml_).

<pre class="EnlighterJSRAW" data-enlighter-language="ini">apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: hello-world-ingress
spec:
  backend:
    serviceName: hello-world
    servicePort: 80</pre>

Una vez lo tienes puedes usar _kubectl create -f ingress.yaml_ para crear el recurso _ingress_. Luego con _kubectl get ing_ deberías ver el recurso:

<pre class="EnlighterJSRAW" data-enlighter-language="no-highlight">NAME                  HOSTS     ADDRESS   PORTS     AGE
hello-world-ingress   *                   80        3m</pre>

Ahora nos falta **que nuestro clúster ejecute un controlador _ingress_. **Será ese controlador el que se encargará de ejecutar las reglas definidas en este controlador _ingress_, que en nuestro caso enviará **cualquier petición al cluster al puerto 80 del servicio hello-world_._**
  
Para ello, dado que el controlador _ingress_ es un contenedor, debemos definir y crear un _pod_ que lo ejecute (como vimos en el post anterior a través de un deployment). Esto es propio para cada controlador _ingress_ que queramos ejecutar. En nuestro caso usaremos [ingress-nginx][4] que implementa un controlador _ingress_ usando nginx. Por si quieres verlo, [el fichero de configuración está aquí][5].  No obstante este fichero hace referencia a otros elementos que deben estar instalados en el clúster. Por supuesto en el repositorio de ingress-nginx están todos los ficheros YAML de configuración para instalar todas las dependencias, así que puedes instalarlas:

<pre class="EnlighterJSRAW" data-enlighter-language="shell">kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/master/deploy/namespace.yaml
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/master/deploy/default-backend.yaml
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/master/deploy/configmap.yaml
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/master/deploy/tcp-services-configmap.yaml
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/master/deploy/udp-services-configmap.yaml
kubectl apply -f kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/master/deploy/without-rbac.yaml</pre>

Por si tienes curiosidad de que instala cada fichero te lo enumero a continuación (aunque hay aspectos que todavía no hemos visto):

  * El _namespace_ donde se crean los elementos de nginx-controller
  * El servicio usado como _default backend_ que se encargará de gestionar las peticiones que no se puedan enrutar por ninguna regla del controlador _ingress_
  * Tres mapas de configuración
  * El controlador _ingress_

Usando _kubectl get pods -n ingress-nginx_ puedes verificar que tienes corriendo los _pods_ de ingress-nginx:

<pre class="EnlighterJSRAW" data-enlighter-language="no-highlight">NAME                                        READY     STATUS    RESTARTS   AGE
default-http-backend-6c59748b9b-pxclx       1/1       Running   2          9m
nginx-ingress-controller-864d449cc6-45sj2   1/1       Running   2          7m</pre>

Y tecleando _kubectl get services -n ingress-nginx_ deberías ver el servicio:

<pre class="EnlighterJSRAW" data-enlighter-language="no-highlight">NAME                   CLUSTER-IP       EXTERNAL-IP   PORT(S)   AGE
default-http-backend   10.108.175.195   &lt;none&gt;        80/TCP    10m</pre>

Recapitulemos para ver que hemos instalado:

  1. Un servicio _hello-world_ y un pod _hello-world_ (en el post anterior)
  2. Un recurso _ingress_
  3. Un servicio _default-http-backend_&#8211;
  4. Dos pods (_default-http-backend_ y _nginx-ingress-controller_)
  5. Algunos elementos más (configmaps y un namespace, pero que no son especialmente relevantes)

Ahora obtén la IP del nodo de MiniKube:

<pre class="EnlighterJSRAW" data-enlighter-language="shell">kubectl describe node minikube  | findstr InternalIP</pre>

Esto te debe dar una IP, que es la de nodo que ejecuta MiniKube. Bien, ahora **abre un navegador y accede a esa IP via HTTP por el puerto 80**. Y te debería responder el servicio _hello-world_.
  
Observa las diferencias respecto a lo realizado en el post anterior.
  
Si haces _kubectl get services _verás algo como:

<pre class="EnlighterJSRAW" data-enlighter-language="null">NAME          CLUSTER-IP    EXTERNAL-IP   PORT(S)   AGE
hello-world   10.97.21.36   &lt;none&gt;        80/TCP    23h
kubernetes    10.96.0.1     &lt;none&gt;        443/TCP   15d</pre>

Observa que el servicio _hello-world_ **no tiene ninguna IP externa asociada** (a diferencia del post anterior). Además ahora cuando navegamos a la IP del nodo lo hacemos usando el puerto 80 **porque así lo hemos definido en el recurso _ingress _**(en el post anterior teníamos que encontrar el puerto que se nos asignaba).
  
**Resumiendo**
  
Hemos visto como podemos exponer un servicio al exterior usando un recurso _ingress_ en lugar de exponerlo directamente usando _NodePort_. Para que ingress funcione debemos instalar un controlador ingress y en este caso hemos usado el [ingress-nginx][4] con su configuración por defecto. Esto instala un controlador ingress usando nginx.
  
Observa como el controlador ingress automáticamente se asocia con el recurso ingress que hemos creado (es posible asociar distintos recursos a distintos controladores si es necesario).
  
Visto así puede parecer que usar ingress no nos aporta mucho respecto a exponer un servicio usando _NodePort_, pero eso es porque estamos exponiendo un único servicio. **En el siguiente post veremos como exponer más de un servicio y como aquí sí que usar ingress nos ofrece una gran ventaja**.

 [1]: https://geeks.ms/etomas/2017/12/19/kubernetes-1-componentes-de-kubernetes/
 [2]: https://github.com/kubernetes/minikube
 [3]: https://geeks.ms/etomas/2017/12/21/kubernetes-2-modelo-de-aplicacion/
 [4]: https://github.com/kubernetes/ingress-nginx
 [5]: https://raw.githubusercontent.com/kubernetes/ingress-nginx/master/deploy/without-rbac.yaml