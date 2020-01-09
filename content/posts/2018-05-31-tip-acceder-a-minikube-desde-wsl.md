---
title: 'Tip: Acceder a Minikube desde WSL'
description: 'Tip: Acceder a Minikube desde WSL'
author: eiximenis

date: 2018-05-31T14:48:36+00:00
geeks_url: /?p=2059
geeks_ms_views:
  - 504
categories:
  - docker
  - kubernetes

---
Hoy me han preguntado eso, así que mira, aprovecho para apuntarlo aquí, por si alguien más tiene esta duda.
  
La situación es la siguiente: tienes minikube instalado y funcionando en Windows, pero quieres usarlo desde un _kubectl_ ejecutándose en un terminal WSL. ¿Es posible?
  
<!--more-->


  
La respuesta es que sí, no solo es posible, si no que es muy fácil. Los pasos que debes realizar son los siguientes.
  
Primero **copiar el fichero de configuración de kubectl al sistema de ficheros de WSL**. Este fichero es _C:\Users\<usuario>&#46;kube\config_. Así, pues, desde un terminal WSL teclea:

<pre class="EnlighterJSRAW" data-enlighter-language="raw">cp /mnt/c/Users/&lt;usuario&gt;/.kube/config ~/.kube/config</pre>

Con esto copias este fichero en tu directorio de usuario de WSL. Ten presente que esto **te sobreescribirá la configuración de _kubectl_ que tengas en WSL** (estoy asumiendo que no tenías).
  
El fichero _config_ es un fichero YAML que contiene la definición de los clústers así como de la información necesaria para acceder a él. P. ej. en mi caso la definición del clúster de Minikube es:

<pre class="EnlighterJSRAW" data-enlighter-language="json">- cluster:
    certificate-authority: C:\Users\etoma\.minikube\ca.crt
    server: https://172.25.117.234:8443
  name: minikube</pre>

Minikube guarda el certificado del clúster en el disco (por defecto en el fichero C:\Users\<usuario>&#46;minikube\ca.crt). Lo que debes hacer es **editar este fichero en WSL para que use las rutas &#8220;a lo WSL&#8221;**. Es decir, modificar &#8220;C:\Users\<usuario>&#46;minikube\ca.crt&#8221; por &#8220;/mnt/c/<usuario>/.minikube/ca.crt&#8221;.
  
Con esto solo no basta, también debes buscar en el mismo fichero, dentro de &#8220;Users&#8221; el usuariol &#8220;minikube&#8221;, ya que allí hay la referencia al certificado de cliente:

<pre class="EnlighterJSRAW" data-enlighter-language="json">- name: minikube
  user:
    as-user-extra: {}
    client-certificate: C:\Users\etoma\.minikube\client.crt
    client-key: C:\Users\etoma\.minikube\client.key</pre>

Sustitutye esas dos rutas para que sean rutas &#8220;a lo WSL&#8221; ¡listos!. Ahora desde WSL ya puedes usar kubectl conectado contra minikube:
  
[<img class="alignnone wp-image-2060 size-full" src="https://geeks.ms/etomas/wp-content/uploads/sites/154/2018/05/minikube-wsl.png" alt="minikube en wsl" width="979" height="201" />][1]
  
¡Ya os dije que era fácil! 🙂

 [1]: https://geeks.ms/etomas/wp-content/uploads/sites/154/2018/05/minikube-wsl.png