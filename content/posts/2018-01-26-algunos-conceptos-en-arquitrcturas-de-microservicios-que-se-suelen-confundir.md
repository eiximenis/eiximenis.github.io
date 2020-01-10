---
title: Algunos conceptos en arquitrcturas de (¿micro?)servicios que se suelen confundir

author: eiximenis

date: 2018-01-26T09:01:21+00:00
geeks_url: /?p=1996
geeks_ms_views:
  - 1184
categories:
  - patrones

---
Bueno, dejémoslo en **arquitecturas distribuídas**, que a veces hablamos muy (demasiado) alegremente de _microservicios_...
  
La idea es hablar sobre algunos conceptos, patrones que se prestan a confusión porque muchas veces &#8220;se tocan&#8221; a nivel funcional y no queda claro si estamos uno u otro. Vamos a hablar de _proxies_, _API Gateway_, _Service Mesh_ y _Backend For Frontend_.
  
<!--more-->


  
**Transparent (o Forward) Proxy, Reverse Proxy**
  
Empecemos por el más sencillo: El _transparent (o forward) proxy_ se limita a reenviar las peticiones que realiza un cliente hacia un servidor determinado. Se suele usar para &#8220;agrupar&#8221; las conexiones de varios clientes, es decir, para el servidor que sirve el recurso es como si la petición la realizara el _transparent proxy _y desconoce cuantos (y cuales) clientes hay al &#8220;otro lado&#8221;. Uso típico: un proxy que agrupa la salida a Internet de toda una oficina.
  
El _reverse proxy_ por otro lado lo que hace es &#8220;aislar&#8221; los servidores del cliente. Si con un proxy normal son los servidores que desconocen qué clientes hay, con un _reverse proxy_ son los clientes que desconocen a los servidores.
  
Si expones varios servicios al exterior, tener que ofrecer una IP y/o puerto distinto para cada uno de ellos no suele ser algo deseable. Precisamente esta es la función del _reverse proxy_: enrutar las peticiones **que vienen del exterior** permitiendo ofrecer así un único punto de entrada (IP+puerto). De este modo si tienes dos servicios, uno es localizable usando http://my-server/service1 y el otro usando http://my-server/service2. Sin _reverse proxy,_ o bien ambos servicios tienen su propia IP o bien (si están ambos en _my-server_) tienen puertos distintos.
  
Para tener _proxies_ se suele confiar en productos de terceros que están especializados en ello como [NGINX][1], [Linkerd][2] o [Envoy][3] por citar tres ejemplos.
  
**Load balancer**
  
Un balanceador de carga es un producto que se dedica, como su nombre indica, a balancear la carga entre distintos servidores que ofrecen el mismo servicio. De este modo para el cliente hay un único punto de entrada y redistribuyen las peticiones basándose en determinadas reglas.
  
Hay que distinguir entre balanceadores de nivel 7 (L7) o de nivel 4 (L4). Esto define a que nivel del [modelo OSI][4] trabajan. Así un balanceador L4 se limita a reenviar paquetes de transporte (p. ej. TCP) a un servidor en concreto basándose en las características de esos paquetes en el nivel 4. Es decir poco más que la IP y el puerto de destino. Por otro lado un balanceador L7 funciona a nivel de aplicación (p. ej. HTTP) lo que indica que pueden analizar paquetes a este nivel y tomar decisiones en base a elementos del protocolo de aplicación. P. ej. una decisión que podrían tomar es decidir a que subconjunto de servidores reenviar basándose en la URL de la petición.  Observa que eso último hace que nuestro balanceador de carga pueda actuar como _reverse proxy_.
  
De todos modos conviene aclarar que por lo general **un_ __reverse proxy_ va un poco más allá que simplemente balancear la carga, ya que puede interaccionar con el protocolo de aplicación y así ofrecer servicios adicionales** tales como compresión, modificación de cabeceras, etc...
  
Sobre si es mejor usar un balanceador de carga L4 o L7 hay opiniones para todos los gustos y yo creo que, como todo en este mundo, depende de tus casos concretos. Una opinión muy generalizada en contra de usar L7 es que se suele usar para tener _sticky sessions_ (eso es, garantizar que un mismo cliente es enrutado siempre al mismo servidor). En efecto _sticky sessions_ es una práctica a evitar porque limita el escalado de tu aplicación y la hace más sensible a fallos. Pero _sticky sessions_ no es el único uso posible para un balanceador L7.
  
Por si os pica la curiosidad, Envoy es L7 y NGINX puede funcionar como L4 o L7.
  
**API Gateway**
  
Aquí abrimos ya la caja de pandora. Porque si definimos &#8220;API Gateway&#8221; a _grosso modo_ podríamos decir que es &#8220;un servicio que habla con servicios internos y ofrece funcionalidades adicionales&#8221;. Pero es que esta misma definición nos sirve exactamente así para definir un _reverse proxy_.
  
Aquí hay que distinguir entre &#8220;API Gateways genéricas&#8221; y &#8220;API Gateways personalizadas&#8221;. Las primeras son productos tipo [Kong][5] o [Azure API Management][6]. Esos productos ofrecen servicios genéricos tales como monitorización, logging, autenticación centralizada, modificación de _claims_, cuotas de llamadas, _cache_, etc
  
Algunos de esos servicios (p. ej. _cache_, _logging_) se pueden ofrecer también desde _reverse proxies_ tipo NGINX, de ahí que sea posible montar API Gateways genéricas usando un _reverse proxy_. Observa que conceptualmente tanto el reverse proxy como el API Gateway &#8220;hacen lo mismo&#8221;: reciben una petición del exterior y la mandan a un servicio interno y ofrecen servicios adicionales. **Hay pues, una fina línea entre que consideramos _reverse proxy_ y API Gateway genérica**. En general una API Gateway ofrece servicios &#8220;más avanzados&#8221; o &#8220;más de aplicación&#8221;. Por supuesto muchos productos que son API Gateways implementan también un _reverse proxy_, pero también puedes tenerlos explícitamente separados en dos productos distintos (p. ej. tener un Kong detrás de un NGINX). Eso ya dependerá de tus necesidades.
  
Otra cosa habitual en las API Gateways es que a partir de una petición de un cliente, se llame a más de un servicio distinto y se combine la respuesta. El objetivo es que tu puedes tener varios servicios internos, pero no quieres ofrecerlos todos a los clientes (por ejemplo no quieres que tu cliente dependa de tu topología lógica de servicios). Así puedes crear una (o varias) API Gateways que ofrezcan otra interfaz al cliente. De este modo el cliente interacciona con el API Gateway: hace una petición y el API Gateway realiza internamente N peticiones a distintos servicios y combina los resultados y los manda al cliente. Este otro tipo de API Gateways, que yo llamo &#8220;personalizadas&#8221;, si que se distinguen de un _reverse proxy_ ya que tienen inteligencia del negocio: deben saber qué servicios internos llamar y como combinar las respuestas.
  
**Backend for Frontend (BFF)**
  
[BFF][7] es un patrón que consiste en ofrecer API Gateways específicas por el tipo de cliente. La idea es que tenemos nuestros servicios internos que desconocen por completo sus posibles clientes. La topología de esos servicios internos depende de otros aspectos, pero no de qué clientes interaccionarán con ellos. Luego **por cada cliente (frontend) se crea una API Gateway (Backend)**. Este Backend está creado ad-hoc por el cliente, por lo que ofrecerá una API pública alineada con las necesidades de éste. Luego, esta API Gateway será la encargada de hablar con los servicios internos y combinar los resultados para mandárselos al cliente.
  
Así podemos decir que BFF es un patrón para organizar nuestras API Gateways específicas.
  
**Service Mesh**
  
Un _service mesh_ es infraestructura dedicada para realizar llamadas entre servicios. El objetivo es implementar protocolos que estas llamadas sean rápidas y fiables. Eso significa que el _service mesh_ implementa, entre otros, patrones tipo _retry_ y _circuit breaker_, desligando así a nuestro código de los servicios de hacerlo.
  
Cuando un servicio debe llamar a otro, esa llamada va a través del _service mesh_ que ofrece estos servicios adicionales. A nivel de red podríamos decir que el _service mesh_ funciona en los niveles 3 y 4 de OSI. Es como si tuviéramos un sustituto de TCP/IP que nos garantiza reintentos, circuit breakers y cache en determinados casos.
  
Un producto que puede funcionar como _service mesh_ es [Linkerd][2]. P. ej. cuando se despliega Linkerd en Kubernetes, se termina desplegando un _daemon_ en cada nodo, que intercepta las llamadas entre _pods_ y les añade esos servicios adicionales. Para nuestro código esto es transparente. Es importante notar que el _**service mesh **_**solo actúa entre las llamadas entre nuestros servicios**, no entre los servicios y los clientes externos.
  
Si empiezas a dibujar cajitas es fácil confundir el _service mesh_ con una API Gateway: al final ambos realizan llamadas entre servicios internos. Pero recuerda que el API Gateway es funcional, mientras que el _service mesh_ es pura infraestructura. Eso significa que, cuando una API Gateway llama a varios servicios para agregar sus resultados, esas llamadas pasan por el _service mesh_.
  
Bueno, lo dejamos aquí. Espero que esto os ayude a clarificar un poco los distintos conceptos. Y como siempre, si tenéis cualquier comentario o ideas distintas de lo que son cada uno de esos conceptos, ¡no dudéis en dejar un comentario!

 [1]: https://www.nginx.com/
 [2]: https://linkerd.io/
 [3]: https://www.envoyproxy.io/
 [4]: https://es.wikipedia.org/wiki/Modelo_OSI
 [5]: https://getkong.org/
 [6]: https://azure.microsoft.com/es-es/services/api-management/
 [7]: https://samnewman.io/patterns/architectural/bff/