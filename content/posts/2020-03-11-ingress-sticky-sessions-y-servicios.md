---
title: "Ingress, sticky sessions y servicios"
author: eiximenis
description: O también se podría llamar "No trates a ingress como un recurso compartido", pero bueno... os cuento lo que nos ocurrió un día con un proyecto.
date: 2020-03-11T12:00:00
categories:
  - k8s
tags:
  - ingress
---

El otro día estuve revisando un proyecto, desplegado en un Kubernetes (un AKS, aunque eso no es relevante en este caso). El tema es que parecía que "las _sticky sessions_ no iban". Por motivos del proyecto, era necesario tener _sticky sessions_ y además estrictas, es decir, que en caso de que se escalara el número de _pods_ los usuarios NO fuesen redirigidos a esos nuevos _pods_ para repartir la carga.

Ya, ese tipo de proyectos escalan bastante mal, pero eso dará para otros posts. De momento centrémonos en lo que ocurría.

## Problema 1: Cookie de sticky  sessions mal configurada

El proyecto en cuestión usaba el [controlador ingress de NGINX](https://github.com/kubernetes/ingress-nginx) y este habilita el soporta para _sticky sessions_ a través de una _cookie_. Revisé el recurso ingress y efectivamente todas las anotaciones necesarias estaban:

```yaml
metadata:
  annotations:
    kubernetes.io/ingress.class: nginx
    nginx.ingress.kubernetes.io/affinity-mode: persistent
    nginx.ingress.kubernetes.io/session-cookie-name: lbsticky
    nginx.ingress.kubernetes.io/session-cookie-expires: "172800"
    nginx.ingress.kubernetes.io/session-cookie-max-age: "172800"    
```

Pero una simple navegación con las _developer tools_ del navegador activadas dejaba claro que el navegador no mandaba la _cookie_. Parece que la recibía, pero no la mandaba de vuelta. El error ahí es que **faltaba la anotación `nginx.ingress.kubernetes.io/session-cookie-path`.**. Esta anotación es **requerida cuando ingress usa expresiones regulares en los paths** de las reglas, como en este ejemplo (y nuestro caso):

```yaml
paths:
- backend:
    serviceName: mysvc
    servicePort: http
  path: /mysite(/|$)(.*)
```

Si no ponemos la anotación `nginx.ingress.kubernetes.io/session-cookie-path`, entonces NGINX manda la cookie pero con el valor de `Path` idéntico a la expresión regular (p. ej. `/mysite(/|$)(.*)`), por lo que el navegador no mandará de vuelta la cookie, ya que no estamos realmente en este path.

Una vez añadida a la anotación, y viendo que ahora la cookie se mandaba, ejecutamos los tests para verificar las _sticky sessions_... y de nuevo fallaron... Y eso nos trae a la parte interesante del post.

## No trates a _ingress_ como un recurso compartido

Sin entrar demasiado en detalles, diré que se trataba de un proyecto con varios servicios, cada uno desplegado en su espacio de nombres. Pero ingress se desplegaba como un recurso compartido, **en un espacio de nombres propio**. Y ahí vienen los problemas: [**no hay manera en ingress de enrutar hacia un servicio de otro espacio de nombres**](https://github.com/kubernetes/kubernetes/issues/17088). Al menos de momento, aunque [se está valorando para una futura versión de ingress](https://docs.google.com/document/d/1BxYbDovMwnEqe8lj8JwHo8YxHAt3oC7ezhlFsG_tyag/edit#heading=h.gs76b1pp1evi).

Así intentar enrutar desde ingress a un servicio `mysvc` que estuviese en un espacio de nombres `otherns` **NO FUNCIONA**:

```yaml
http:
paths:
- backend:
    serviceName: mysvc.otherns.svc.cluster.local
    servicePort: http
```

En general, da igual, si pones un punto en el `serviceName` vas a recibir un error al desplegar el recurso ingress al cluster. Cuando la gente se encuentra con este problema, usa un truco del almendruco, que en algunos casos funciona:

Este truco consiste **en declarar un servicio `ExternalName` que apunte al DNS del servicio real** al que quieres llamar:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: mysvc-proxy
  namespace: "mismo-namespace-que-el-ingress"
spec:
  type: ExternalName
  externalName: mysvc.otherns.svc.cluster.local
  ports:
  - port: 80  
```

Y luego configura el _ingress_ para que use el servicio `mysvc-proxy` en lugar de mysvc:

```yaml
http:
paths:
- backend:
    serviceName: msvc-proxy
    servicePort: 80
```

Esto funciona: ingress enruta a `msvc-proxy` que a su vez enruta via DNS interno a `mysvc.otherns.svc.cluster.local` que es el servicio real que redirigirá la llamada a un _pod_. ¡Todo perfecto!...

... Hasta que quieres usar _sticky sessions_. Y es que esta aproximación **rompe cualquier gestión de _sticky sessions_ que el controlador ingress (en nuestro caso NGINX) pueda hacer**. La razón es que, a pesar de que nosotros en el _ingress_ declaramos un servicio, NGINX **no usa el servicio para enrutar las llamadas**. NGINX se salta al servicio, y **enruta las llamadas directamente a los _pods_**.

>**Pregunta**: Todos los controladores _ingress_ hacen eso de saltarse el servicio y llamar a los _pods_ directamente? No tiene por qué (la definición del estándard no dice nada al respecto), pero si el controlador _ingress_ quiere ofrecer servicios avanzados (como es el caso de NGINX) no tiene otra opción que saltarse el servicio. 

Para saber qué _pods_ están "bajo el paraguas" del servicio se usa una API de Kubernetes, que es la endpoint API:

![Salida de "kubectl get endpoints un-servicio donde se ve tres IPs](/images/posts/2020-03-07-endpoints.png)

El comando `kubectl get endpoints mysvc` devuelve los endpoints asociados al servicio `mysvc`. Estos endpoints son, las IPs de los _pods_ que están bajo este servicio. Esta API es la que usa NGINX: Para cada _ingress_ NGINX mantiene una lista con sus endpoints (los _pods_ subyacentes). Esta lista de endpoints, devuelta por Kubernetes, ya tiene en cuenta p. ej. que un _pod_ puede no estar listo para recibir peticiones. Así NGINX puede enrutar directamente a uno de esos _pods_ saltándose el servicio. Hacer esto le permite **tener sus propias políticas de _load balancing_ y implementar, entre otras, _sticky sessions_**. Por supuesto NGINX actualiza esa lista de endpoints cuando un endpoint es creado o eliminado.

Así que lo que tenemos es un escenario en el que:

1. Llega una petición al controlador ingress (NGINX)
2. El controlador ingress, a partir de la ruta y el _host_ de la petición, mira a que servicio debe pasar la petición
3. De este servicio, elige uno de sus endpoints y le manda la petición directamente (sin pasar por el servicio en sí).

Es decir, solo usa el servicio para saber a qué endpoints (_pods_) debe mandar la petición.

¿Y qué ocurre cuando usamos el truco de usar un servicio `ExternalName` para llamar a un servicio que está en otro espacio de nombres que el del recurso ingress? Pues que entonces, **el servicio del cual NGINX mantiene sus endpoints, es el servicio `ExternalName`** (el que está en el ingress). Y los servicios `ExternalName` tienen siempre un solo endpoint: el DNS al que apuntan.

Así, lo que ocurre es que, a pesar de tener las _sticky sessions_ habilitadas en NGINX, al usar un servicio `ExternalName` el escenario es el siguiente:

1. Llega una petición al controlador ingress (NGINX)
2. El controlador ingress, a partir de la ruta y el _host_ de la petición, mira a que servicio debe pasar la petición
3. De este servicio, elige el endpoint basándose en la cookie de sticky sessions (si existe, si no, elige uno y manda la cookie).
4. Pero, como el servicio es `ExternalName`, y NO tiene endpoints, entonces NGINX le manda la peticion al servicio `ExternalName` directamente
5. El servicio `ExternalName` es una mera redirección DNS al servicio real.
6. La petición llega al servicio real (el que está en otro espacio de nombres) quien la manda a uno de sus _pods_.
7. Resultado: Las sticky sessions se pierden

En resumen: **tener los recursos ingress en otro espacio de nombres a los servicios a los que apuntan es una mala idea**. Y eso es porque (al menos en su concepción actual), **el recurso ingress NO es un recurso compartido**. Forma parte de tu aplicación.

![Diagrama arquitectónico con varios ingress en distintos namespaces](/images/posts/2020-03-11-diagram.png)

El controlador ingress SÍ es compartido, pero los recursos ingress no: ten presente que, el controlador ingress "recoge" todo los recursos ingress y los "mezcla". Así que no tienes por qué desplegar todos los recursos ingress juntos, ni desplegarlos en un espacio de nombres común, ni desplegarlos en el espacio de nombres donde esté el controlador ingress instalado.

En función de tus necesidades puedes, por supuesto, tener más de un recurso ingress en el mismo namespace. Al final, todos ellos se combinan en el controlador ingress (aunque también es posible tener varios controladores ingress, pero esto daría para otro post). Así el usuario realiza una petición, esa es recibida por el controlador ingress, y luego en base al _host_ y el _path_ de la petición es enrutada directamente, al _pod_ que pueda atender dicha petición.

Así que ya sabes: trata a ingress como un recurso de tu aplicación, que se despliegue junto a esta. ¡Te evitarás problemas!