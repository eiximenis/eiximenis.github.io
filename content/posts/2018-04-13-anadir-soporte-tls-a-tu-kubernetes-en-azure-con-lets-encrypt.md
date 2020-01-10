---
title: "Añadir soporte TLS a tu Kubernetes en Azure con Let's Encrypt"

author: eiximenis

date: 2018-04-13T16:05:06+00:00
geeks_url: /?p=2034
geeks_ms_views:
  - 1012
categories:
  - docker
  - kubernetes
  - Sin categoría

---
¡Buenas! En este post vamos a ver como añadir soporte TLS a tu clúster de Kubernetes desplegado en ACS o AKS. Hace tiempo escribí un post sobre como añadir certificados de desarrollo a un servicio NGINX que tuvieses en Kubernetes. Aunque lo dicho en [aquel post][1] sigue siendo válido, hay una manera mucho más sencilla con la condición de que [usemos _ingress_ para exponer nuestros servicios al exterior][2].
  
En este post parto de la suposición de que:

  1. Tienes un clúster de Kubernetes en ACS/AKS y _kubectl_ configurado para atacar a él
  2. Tienes el controlador _ingress_ de NGINX instalado en el clúster exponiendo cualquier servicio

Es decir, haciendo http://IP-CLUSTER/servicio obtienes alguna respuesta. Lo que queremos es poder usar https en su lugar.
  
<!--more-->


  
El controlador _ingress_ de NGINX **viene con un certificado SSL preinstalado**. De hecho para la gente que lo usa por primera vez le molesta más que otra cosa, porque cualquier petición vía http se redirige a https de forma automática y obtenemos este certificado incorrecto:
  
[<img class="alignnone size-medium wp-image-2035" src="https://geeks.ms/etomas/wp-content/uploads/sites/154/2018/04/fake-cert-error-282x300.jpg" alt="Error de certificado" width="282" height="300" />][3]
  
Si observas el _issuer_ es &#8220;_Kubernetes Ingress Controller Fake Certificate_&#8220;: este es el certificado que incorpora el controlador _ingress_ de NGINX.
  
Si has usado el controlador _ingress_ de NGINX seguramente ya lo sabrás, pero por si acaso: **Puedes deshabilitar la redirección automática de http a https** añadiendo anotaciones al recurso _ingress_:

<pre class="EnlighterJSRAW" data-enlighter-language="generic">annotations:
  ingress.kubernetes.io/ssl-redirect: "false"
  nginx.ingress.kubernetes.io/ssl-redirect: "false"</pre>

De este modo, el controlador NGINX ya no te redireccionará de http a https.
  
Vale, ahora vayamos a lo que nos interesa, que es **añadir soporte https con un certificado válido de Let&#8217;s Encrypt**.
  
Para ello vamos a usar [cert-manager][4] que es un paquete de Kubernetes que **auto-aprovisiona certificados TLS**. Es decir, todo el proceso pasará de forma automática, sin que nosotros tengamos que hacer nada especial. Lo primero es **instalar [helm][5]** para desplegar cert-manager en nuestro clúster. Helm se compone de dos partes:

  1. Una CLI en cliente (helm)
  2. Un componente servidor en el clúster (tiller)

Para entender rápidamente que es helm, podríamos decir que es &#8220;el nuget de Kubernetes&#8221;. Nos permite desplegar paquetes de Kubernetes, donde un paquete es un conjunto de elementos de Kubernetes (controladores, servicios, etc). La instalación de helm es muy sencilla y está muy bien [detallada en su página web][6]. Pero básicamente son dos pasos:

  1. [Descárgate][7] la CLI
  2. Ejecuta _helm init_: Eso instalará _tiller_ en el cluster (Si usas una RC de helm añade el parámetro _&#8211;canary-image). _Ojo, que la última RC de _helm_ puede dar errores en AKS, te recomiendo la última estable.

Una vez tengas helm instalado, el siguiente paso es usarlo para instalar cert-manager, lo que tambien es trivial. Basta con teclear:

<pre class="EnlighterJSRAW" data-enlighter-language="shell">helm install --name cert-manager --namespace kube-system stable/cert-manager --set rbac.create=false</pre>

(El parámetro _rbac.create _es para que no cree roles RBAC, que AKS no los soporta de momento).
  
Ahora que ya tenemos cert-manager estamos listos para empezar.
  
**Definiendo el issuer**
  
Cert-manager añade dos tipos de objetos nuevos a la API de Kubernetes: _issuer_ y _certificate_. El primero define una entidad certificadora (tal como Let&#8217;s Encrypt) y el segundo define un certificado concreto. Vamos a empezar por el primero. Como todos los objetos de k8s se define mediante un YAML:

<pre class="EnlighterJSRAW" data-enlighter-language="generic">apiVersion: certmanager.k8s.io/v1alpha1
kind: Issuer
metadata:
  name: letsencrypt-staging
  namespace: default
spec:
  acme:
    server: https://acme-staging.api.letsencrypt.org/directory
    email: user@example.com
    privateKeySecretRef:
      name: letsencrypt-staging
    http01: {}</pre>

Con eso definimos un _issuer_ llamado _letsencrypt-staging. _Las entradas importantes ahí son:

  * server: Cual es el servidor a usar. Este servidor debe soportar el protocolo [ACME][8] (ya, el nombre suena a broma :P), que es un protocolo que se usa para la gestión automatizada de certificados. El valor apunta al **servidor de pruebas de Let&#8217;s Encrypt**
  * El valor de _http01_ (un objeto vacío) indica que usaremos el mecanismo de validación HTTP-01. Mediante ese mecanismo garantizamos que nosotros somos los dueños del dominio colocando un archivo concreto en una ruta concreta que se nos pide.

Usa _kubectl apply_ para añadir este fichero en el clúster. Luego puedes verificar que el _issuer_ está instalado con _kubect get issuers._
  
**Crear el certificado**
  
Ahora vamos a definir el objeto certificado. Para ello **tenemos un pre-requisito importante: debes localizar la IP pública de tu cluster en Azure y asignarle un DNS**.
  
Bien, ahora usa el siguiente YAML como plantilla para tu certificado:

<pre class="EnlighterJSRAW" data-enlighter-language="null">apiVersion: certmanager.k8s.io/v1alpha1
kind: Certificate
metadata:
  name: my-cert
  namespace: default
spec:
  secretName: my-cert-tls
  issuerRef:
    name: letsencrypt-staging
  commonName: example.com
  dnsNames:
  - example.com
  acme:
    config:
    - http01:
        ingress: my-ingress
      domains:
      - example.com</pre>

Aquí definimos un certificado para el DNS _example.com_. Por supuesto debes usar el que hayas asignado a la IP pública de Azure (a no ser que uses dominios propios el DNS será del estilo _mycluster.eastus.cloudapp.azure.com_).
  
Otras entradas a tener en cuenta son:

  * secretName: Nombre del secreto en k8s que contendrá el certificado descargado. Deberemos usarlo en el recurso _ingress_
  * issuerRef: El nombre del recurso _issuer_ que emite el certificado
  * http01: La configuración de http01. Eso requiere algunas palabras más 🙂

Si recuerdas en HTTP-01 validamos que somos los propietarios del dominio porque se nos pedirá (como parte del protocolo ACME) que coloquemos un fichero en una determinada ruta. Bien, la idea **es que de forma automática cert-manager creará un servicio que servirá este fichero y usará un recurso _ingress_ para exponer este servicio en la ruta que le pida Let&#8217;s Encrypt**. Todo eso ocurre automáticamente, pero debemos indicarle que recurso _ingress_ usar. Lo habitual es tener un solo recurso _ingress_ y ese es el que usarías, pero en el caso (tampoco tan raro) de que tengas más de uno, debes usar uno que esté asignado al controlador NGINX cuya IP pública estás validando (en el caso, este sí bastante raro, de que tengas más de un controlador _ingress_ instalado).
  
En este YAML de ejemplo definimos que para el dominio (example.com) use el recurso _ingress_ llamado _my-ingress_ (podríamos usar recursos _ingress_ distintos por dominios distintos).
  
Como antes usa _kubectl apply_ para crear el certificado. Ahora usando _kubectl get certificates_ deberías verlo. **Si todo ha ido bien, ya tienes tu certificado instalado en el clúster**. De hecho si usas _kubectl get secrets_ deberías ver un secreto llamado _my-cert-tls_ (o el nombre usado en _secretName_ del YAML del objeto certificado) de tipo  kubernetes.io/tls:

<pre class="EnlighterJSRAW" data-enlighter-language="null">NAME                  TYPE                                  DATA      AGE
letsencrypt-staging   Opaque                                1         2h
my-cert-tls           kubernetes.io/tls                     2         1h</pre>

(Puedes ver otros secretos adicionales, pero esos dos deberías verlos)
  
**Asociar el recurso _ingress_ con el certificado**
  
Ahora solo falta asociar el recurso _ingress_ con el certificado.  Para ello debes usar una entrada _tls_ para que _ingress_ sepa cual es el secreto que contiene el certificado:

<pre class="EnlighterJSRAW" data-enlighter-language="null">spec:
  rules:
  - host: YOUR_DNS_NAME
    http:
      paths:
      - path: /foo
        backend:
          serviceName: foosvc
          servicePort: 80
  tls:
  - secretName: my-cert-tls
    hosts:
      - YOUR_DNS_NAME</pre>

Observa la sección _tls_ donde se indica el secreto que contiene el certificado (y la lista de hosts asociados a este certificado).
  
Ahora solo te queda redesplegar el recurso _ingress_ con _kubectl apply_ y ¡listos! No tienes que hacer nada más: **ya tienes un certificado de Let&#8217;s Encrypt en tu clúster**.
  
**NOTA IMPORTANTE:** Si has usado el servidor de staging de Let&#8217;s Encrypt el certificado NO ES VÁLIDO. Pero sabrás que has hecho todos los pasos bien porque **la entidad es Fake LE Intermediate X1.** Si te aparece esta entidad, es que todo lo has hecho bien. Para obtener un certificado de producción **realiza los siguientes pasos:**

  1. Actualiza el _issuer_ para usar el servidor de producción de Let&#8217;s Encrypt (https://acme-v01.api.letsencrypt.org/directory) y redespliegalo con kubectl apply
  2. Elimina el secreto que contiene el certificado (el my-cert-tls)
  3. Elimina el objeto certificado (p. ej. si el YAML se llamaba certificate.yaml puedes usar _kubectl delete -f certificate.yaml_)
  4. Vuelve a crear el certificado (_kubectl apply -f certificate.yaml_)

Al cabo de unos segundos te debe volver a aparecer el secreto _my-cert-tls_ pero **ahora contiene un certificado firmado y ya deberías poder acceder vía https a tu clúster (usando el DNS asignado).**
  
¡Espero que te haya resultado intersante! **Eso sí, una cosilla: cert-manager está en &#8220;alfa&#8221; por lo que puede haber _breaking changes_**. Funcionar funciona, pero los YAML que definen los objetos _issuer_ y _certificate_ pueden no funcionar bien en futuras versiones. Si buscas algo más estable puedes usar [kube-lego][9], pero está deprecado (cert-manager es su sucesor) y no soporta versiones de Kubernetes más allá de 1.8

 [1]: https://geeks.ms/etomas/2017/10/16/configurar-ssl-en-un-cluster-de-k8s/
 [2]: https://geeks.ms/etomas/2018/01/03/kubernetes-3-controladores-ingress/
 [3]: https://geeks.ms/etomas/wp-content/uploads/sites/154/2018/04/fake-cert-error.jpg
 [4]: https://github.com/jetstack/cert-manager
 [5]: https://github.com/kubernetes/helm
 [6]: https://docs.helm.sh/using_helm/#installing-helm
 [7]: https://github.com/kubernetes/helm/releases
 [8]: https://en.wikipedia.org/wiki/Automated_Certificate_Management_Environment
 [9]: https://github.com/jetstack/kube-lego