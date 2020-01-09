---
title: Configurar SSL en un cluster de k8s
description: Configurar SSL en un cluster de k8s
author: eiximenis

date: 2017-10-16T14:38:27+00:00
geeks_url: /?p=1920
geeks_ms_views:
  - 792
categories:
  - Sin categoría
tags:
  - acs
  - k8s
  - nginx
  - ssl

---
¡Buenas! En esta entrada voy a resumir los pasos seguidos para añadir soporte SSL a un cluster Kubernetes. En mi caso lo tengo desplgado en ACS pero eso es irrelevante.
  
Lo único que si usas ACS y quieres usar una IP determinada, recuerda que la IP pública que vayas a usar debe estar creada anteriormente. Si no, por más que la especifiques dentro de la configuración del servicio (usando _loadBalancerIP_), Kubernetes no va a poder levantar el servicio.
  
<!--more-->


  
En este caso se usa un solo servicio de tipo _LoadBalancer_ de Kubernetes, es decir hay **una única IP de acceso desde el exterior**. Mi clúster contiene varios servicios (varias APIs REST) y se seleccionan mediante el path:

  * http://ip-cluster/api1/
  * http://ip-cluster/api2/
  * ...

El servicio de tipo _LoadBalancer_ que expone la &#8220;ip-cluster&#8221; es un contenedor de nginx. Así la mayor parte del trabajo se reduce en &#8220;dar soporte SSL a nginx&#8221;.
  
**Parte 1: Preparar la configuración del nginx**
  
Lo primero que hice fue seguir los pasos de [este post][1] (<https://www.digitalocean.com/community/tutorials/how-to-create-a-self-signed-ssl-certificate-for-nginx-in-ubuntu-16-04>). En concreto:

  1. Creé el certificado SSL y el fichero del grupo Diffie-Hellman, según las instrucciones del &#8220;Step 1&#8221; 
      1. La diferencia con  la sentencia del post enlazado es que en el post están configurando un nginx existente y generan los ficheros directamente en el directorio donde está nginx. Yo tengo que configurar un contenedor en k8s, así que ya llegaremos a eso. Por eso genero los ficheros en mi directorio local.
      2. **Nota:** Por experiencias anteriores evité usar cygwin para generar esos ficheros. Usa una distro de Linux **o bien usa bash for Windows** que funciona perfectamente.
  2. El siguiente paso fue crearme los ficheros ssl-params.conf y self-signed.conf con el mismo contenido que se menciona en el post en el &#8220;Step 2&#8221;. Esos ficheros contienen la configuración para que nginx soporte SSL.
  3. Finalmente edité mi fichero de configuración de nginx y agregué las siguientes líneas dentro de la sección _server_ (para que nginx escuche por el 443 para SSL):

<pre class="EnlighterJSRAW" data-enlighter-language="raw">listen 443 ssl;
include ./self-signed.conf;
include ./ssl-params.conf;
</pre>

**Parte 2: Configurar Kubernetes**
  
En este punto tengo los ficheros _dhparam.pem_, _nginx-selfsigned.key_ y _nginx-selfsigned.crt_ (creados a partir del &#8220;Step 1&#8221; del post anterior) y los ficheros  _ssl-params.conf_ y _self-signed.conf (_creados a partir del &#8220;Step 2&#8221; del post anterior).
  
El primer paso es crear config-maps de Kubernetes para poder montar esos ficheros en el contenedor de nginx. Para ello podemos usar los siguientes comandos:

<pre class="EnlighterJSRAW" data-enlighter-language="null">kubectl create configmap ssl-files --from-file=nginx-selfsigned.crt --from-file=dhparam.pem
kubectl create configmap self-cert-key --from-file=nginx-selfsigned.key
kubectl create configmap config-files --from-file=nginx.conf --from-file=self-signed.conf --from-file=ssl-params.conf</pre>

<div>
  Con eso tenemos tres config-maps (<em>ssl-files</em>, <em>self-cert-key</em> y <em>config-files</em>). Si ya estás usando un nginx desplegado ya tendrás al menos un config-map, que contiene el fichero nginx.conf. En mi caso a este config-map le agrego los otros dos ficheros .conf (uso un criterio de usar config-maps distintos para contenido que irá a distintos directorios del contenedor).
</div>

El siguiente paso es **configurar el servicio** de nginx. En mi caso este servicio se llama _frontend_. Y su yaml es como sigue:

<pre class="EnlighterJSRAW" data-enlighter-language="ini">apiVersion: v1
kind: Service
metadata:
  labels:
    app: myapp
    component: frontend
  name: frontend
spec:
  ports:
  - port: 80
    name: tcp-80
    targetPort: 8080
  - port: 443
    name: tcp-443
    targetPort: 443
  selector:
    app: myapp
    component: frontend
  type: LoadBalancer
  loadBalancerIP: x.x.x.x</pre>

Lo importante ahí es ver que tengo definidos tanto los puertos 80 como el 443. Por defecto mi nginx escuchaba por el 8080 (de ahí el _port forwarding_ de 80 a 8080 definido). Para el puerto 443 no hay _port forwarding_ porque en el fichero nginx.conf, he configurado directamente el 443 como el puerto de SSL.
  
Vayamos ahora a la configuración del _deployment_ en kubernetes. Es en este punto donde vamos a usar volúmenes para desplegar los distintos ficheros de los config-maps en el directorio destino del contenedor (pongo solo la parte referente a los volúmenes):

<pre class="EnlighterJSRAW" data-enlighter-language="ini">volumeMounts:
        - name: config
          mountPath: /etc/nginx
        - name: certs
          mountPath: /etc/ssl/certs/
        - name: ssl-private
          mountPath: /etc/ssl/private/
      volumes:
      - name: config
        configMap:
          name: config-files
          items:
          - key: nginx.conf
            path: nginx.conf
          - key: self-signed.conf
            path: self-signed.conf
          - key: ssl-params.conf
            path: ssl-params.conf
      - name: certs
        configMap:
          name: ssl-files
          items:
          - key: dhparam.pem
            path: dhparam.pem
          - key: nginx-selfsigned.crt
            path: nginx-selfsigned.crt
      - name: ssl-private
        configMap:
          name: self-cert-key
          items:
          - key: nginx-selfsigned.key
            path: nginx-selfsigned.key</pre>

Desplegamos cada uno de los ficheros a su _path_ indicado_._
  
¡Y listos! Con esto nuestro contenedor nginx ya estará preparado para aceptar conexiones SSL. 🙂

 [1]: https://www.digitalocean.com/community/tutorials/how-to-create-a-self-signed-ssl-certificate-for-nginx-in-ubuntu-16-04