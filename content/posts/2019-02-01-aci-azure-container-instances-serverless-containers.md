---
title: 'ACI (Azure Container Instances): Serverless containers'
description: 'ACI (Azure Container Instances): Serverless containers'
author: eiximenis

date: 2019-02-01T11:56:15+00:00
geeks_url: /?p=2263
geeks_ms_views:
  - 675
categories:
  - azure
  - docker
  - serverless
tags:
  - aci
  - aks

---
Cuando hablamos de _serverless_ todo el mundo lo asociamos a las soluciones tipo FaaS como Azure Functions o Amazon Lambda, pero hay otros productos que se engloban dentro de ese paradigma y en Azure uno de los más interesantes es **Azure Container Instances**. Del mismo modo que con una Azure Function me limito a poner código en &#8220;ejecución&#8221; y esto se ejecuta en algún sitio, usando ACI lo que hago es &#8220;poner un contenedor&#8221; que se ejecutará en... bueno, donde sea. Eso es _serverless_ señores.
  
Con este post quiero iniciar una pequeña serie dedicada a ACI, como ACI se compara con su [primo Zumosol][1] (AKS) y como AKS y ACI habilitan interesantes escenarios (altísima escalabilidad y workloads mixtos de contenedores).
  
<!--more-->


  
Pero como digo, ese es solo el primer post, así que hablemos de ACI: el caso de uso más simple es tienes un contenedor y quieres ponerlo en Azure **sin preocuparte de nada más?** Bueno, pues usa ACI. Mira, vamos a usar la imagen _dockercampusmvp/go-hello-world_ (**publicidad:** esa es una imagen de uno de los labs de [mi curso de Docker y Kubernetes en CampusMVP][2]). Este contendor simplemente expone un servidor web implementaado en Golang, que responde siempre con &#8220;hello world&#8221;.
  
Pues bien vamos a desplegarlo en Azure, usando la Azure CLI con dos líneas (y una será para crear el resource group):

<pre class="EnlighterJSRAW" data-enlighter-language="null">az group create -n test-aci -l northeurope
az container create -g test-aci -n go-hello-world-ctr --image dockercampusmvp/go-hello-world --ports 80 --ip-address public --dns-name-label go-hello-world</pre>

El parametro &#8220;&#8211;ports&#8221; indica los puertos a abrir al exterior, con &#8220;&#8211;ip-address public&#8221; le indicamos que queremos una IP pública y usando &#8220;&#8211;dns-name-label&#8221; especificamos el prefijo DNS que queremos.
  
¡Con eso ya tenemos nuestro contenedor en Azure! Podemos usar &#8220;az container list&#8221; o &#8220;az container show&#8221; para verlo:
  
[<img class="alignnone size-large wp-image-2265" src="https://geeks.ms/etomas/wp-content/uploads/sites/154/2019/02/az-container-list-1024x69.png" alt="Salida de az cntainer list -g test-aci -o table" width="660" height="44" />][3]
  
¡Fantástico! Y ahora... como accedemos al contenedor? Bueno, pues simplemente usando la IP asociada o bien buscando su DNS, que puedes encontrar con &#8220;_az container show -g test-aci -n go-hello-world-ctr | findstr fqdn_&#8221; (usa &#8220;grep&#8221; en lugar de &#8220;findstr&#8221; si estás en Linux):
  
[<img class="alignnone size-large wp-image-2266" src="https://geeks.ms/etomas/wp-content/uploads/sites/154/2019/02/calling-aci-1024x121.png" alt="Llamando al contenedor con curl" width="660" height="78" />][4]
  
¿Fácil, verdad? Ya os lo dije: dos comandos de az y todo listo. Nuestro contenedor ya está creado.
  
**Mapeo de puertos en ACI**
  
Nada es perfecto y ACI tampoco: **actualmente el mapeo de puertos no está soportado. **Es decir ACI siempre enruta del puerto externo X al mismo puerto X del contenedor. Por lo tanto si tu contenedor escucha por el puerto 8080, a este puerto deberás llamar cuando accedas a tu contenedor desde el exterior.
  
Eso sí, al menos sabemos que [el mapeo de puertos está en el roadmap][5].
  
**ACI versus Webapps**
  
&nbsp;
  
En Azure también se puede desplegar un contenedor en una Webapp. Aunque es igualmente muy sencillo, no lo es tanto como usar ACI. Y es que, de lejos, **ACI es la manera más sencilla para desplegar un contenedor** en Azure. Lo interesante de ACI es que se crea y destruye muy rápidamente y por lo tanto solo pagarás por el tiempo que el contenedor esté levantado (haga o no haga nada, por supuesto).
  
Ahora bien **ACI tiene un gran, gran, gran problema al compararse con una webapp y es que <span style="text-decoration: underline;">NO TIENE SOPORTE BUILT-IN PARA TLS/SSL</span>**. Ojo, que quede claro: **hay maneras de usar TLS/SSL con ACI** pero complican el despliegue y perdemos esa inmediatez. Desde septiembre del 2018 está disponible en preview el poder crear ACIs en vnets propias de Azure, lo que nos da otra posibilidad de añadir TLS/SSL. Pero de nuevo, eso nos complica el despliegue.
  
**ACI versus AKS**
  
ACI y AKS se complementan más que compiten entre ellos. Si bien el uso de ACI permite levantar y parar contenedores y asumir así cargas muy variables de trabajo, para cargas más estables el uso de un clúster de máquinas virtuales (como AKS) es más barato. Todo es hacer los números, claro.
  
Por otra parte, como digo, se complementan porque **AKS puede usar ACI como nodos virtuales** (lo veremos en un futuro post), lo que permite usar todo el ecosistema de Kubernetes, ejecutar parte del workload en las propias VMs del clúster, pero tener otra parte del workload, la altamente variable, ejecutándose en ACI que se crean y destruyen bajo demanda.
  
Por supuesto AKS es mucho más complejo de manejar que ACI, y al mismo tiempo mucho más potente, siendo la solución que debería usarse para aquellos workloads que se componen de varios tipos de contenedores.
  
**Posts de esta serie**
  
Iré añdiendo los enlaces a medida que los posts se publiquen 🙂

  1. ACI: Serverless containers (este)
  2. [Container groups en ACI][6]
  3. ACI y AKS: nodos virtuales
  4. ACI y AKS: virtual kubelet

¡Espero que os sea interesante!

 [1]: https://www.youtube.com/watch?v=B90D3Eq7KjU
 [2]: https://www.campusmvp.es/catalogo/Product-Docker-y-Kubernetes-desarrollo-y-despliegue-de-aplicaciones-basadas-en-contenedores_237.aspx
 [3]: https://geeks.ms/etomas/wp-content/uploads/sites/154/2019/02/az-container-list.png
 [4]: https://geeks.ms/etomas/wp-content/uploads/sites/154/2019/02/calling-aci.png
 [5]: https://feedback.azure.com/forums/602224-azure-container-instances/suggestions/34082284-support-for-port-mapping
 [6]: https://geeks.ms/etomas/?p=2299&preview=true