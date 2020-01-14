---
title: Análisis de clouding.io
author: eiximenis
description: Desde clouding.io me han contactado para que pruebe su servicio y cuente mis impresiones... El resultado? Este post :)
date: 2020-01-14
categories:
  - general
  - cloud
tags:
  - patrocinado
---

Esos días he tenido el placer de poder probar la plataforma de la gente de [Clouding.io](https://clouding.io), que ofrece servicios de IaaS en el _cloud_. Te cuento ahora, brevemente, mis impresiones :)

## Creación de un servidor
_Clouding.io_ es un servicio que ofrece, básicamente, servidores VPS cloud. Eso es, un servicio enfocado al enfoque _IaaS_, en el cual alquilamos servidores virtuales y tenemos el control total sobre ellos. En el caso concretol de _Clouding.io_ se prima la sencillez sobre cualquier otro aspecto: crear un servidor es, literalmente, cuestión de un par de _clicks_:

![Pantalla de creación de un servidor](/images/posts/2020-01-14-server-create.png)

En este caso, voy a crear un servidor a partir de la plantilla de Centos 8.0, con 8Gib RAM y 4 vCores junto con un disco SSD de 250 GiB.


El sistema me informa que el **precio mensual** de este servidor será de unos 78€/mes aproximadamente (IVA incluído). ¿Es un buen precio? Bueno, tirando de la calculadora de Azure, descubro que en Azure una máquina similar (una D2v3 con 2 vCores, 8Gib RAM) junto con un disco SSD de 250 Gib se me va a los 100$ mensuales (impuestos aparte), aunque el precio baja si se reservan uno o más años de forma anticipada. En definitiva, el precio no parece fuera de mercado.

Una vez creas el servidor, el sistema tarda unos minutos en aprovisionarlo y luego ya podemos acceder a él.

![Listado de mis servidores](/images/posts/2020-01-14-servers.png)

Pulsando sobre el servidor entro en la página de detalles donde puedo administrarlo. También se puede acceder a la consola via web, pero es bastante incómodo (lo que suele ser habitual en esos casos). Es mucho mejor usar SSH junto con el usuario y contraseña que nos proporcionan:

![SSH - Login con usuario](/images/posts/2020-01-14-ssh-login-user.png)

O mejor, usar la clave SSH asociada al servidor. Cuando se crea un servidor se le asigna una clave SSH (puedes subir una que ya tengas). Te puedes descargar el fichero `.pem` con la clave privada y luego usar `ssh -i` para entrar sin contraseña:

![SSH - Login con clave privada](/images/posts/2020-01-14-ssh-login-key.png)

Una vez estoy dentro del servidor puedo **hacer lo que quiera con él**, es totalmente mío (aprovisionamos servidores, es un modelo IaaS).

## Seguridad

Es imprescindible tener una forma sencilla de configurar la seguridad de mi servidor. Para ello, desde la pestaña "Red" de mi servidor puedo configurar el cortafuegos. La forma en como funciona es que se crean "grupo de reglas de seguridad" y esas reglas se aplican a los servidores. Así la pestaña "Mis Firewalls" te muestra dichas reglas. Cada grupo de reglas (lo que ellos llaman un "firewall") es un grupo de entradas donde podemos especificar:

* Rango de puertos
* Protocolo
* IP Origen
* Tráfico aceptado o no

Solo se aceptará el tráfico que se incluya en alguna de esas reglas. No necesitas reiniciar nada ni esperar nada para que eso tenga efecto.

P. ej. tengo el grupo de reglas "default" asignado a mi servidor:

![Grupo de reglas "default"](/images/posts/2020-01-14-firewall-rules.png)

Observa como hay una regla que permite el puerto TCP/22 para todas las IPs. Eso es para que funcione SSH. Podría limitar las IPs y así poder acceder vía SSH solo desde mi casa u oficina. O puedo eliminar la regla y ya no puedo acceder via SSH.

Esos grupos de reglas los aplicas a los servidores y de este modo tienes la seguridad centralizada. Cuando añadimos una regla el sistema nos permite añadir "reglas por defecto" para habilitar los servicios más típicos.

Lo que no he encontrado es como hacer que en un servidor se sobreescriba una regla del _firewall_ asociado. Eso puede ser útil en escenarios en qué tengo un servidor que usa las mismas reglas que otro, pero con una sola diferencia. Por ejemplo varios servidores con SSH habilitado y otro con SSH habilitado pero solo desde una determinada IP. Añadir dos "firewalls" no funciona, ya que se toma la regla más general (en este caso la regla que permite SSH desde cualquier IP). Así que para esos casos no queda otra que crear un grupo de reglas propio para este servidor.

## Backups

Por supuesto, puedes habilitar _backups_ para tus servidores y estos los tienes disponibles. Puedes definir la frecuencia de Backup (diaria, semanal, mensual,...) y el número de backups a guardar. Esos backups son siempre programados y pueden servir de fuente para crear un servidor nuevo si hay necesidad.
Es importante destacar que la gente de _Clouding.io_ guardan tres copias de cada servidor (que se sincroinzan automáticamente) de forma que el servidor se cae, pueden arrancar otro casi al instante. Además vienen equipados con otras herramientas de seguridad (como evitar ataques DDoS, usando filtros de IPs) y otras utilidades.

## Servidores Windows

No solo de Linux vive el hombre, y _Cloduing.io_ soporta también servidores Windows, ya sea Windows Server (2003-2019) o Windows 10. No he visto la opción de crear servidores de otras versiones de Windows como 8.1, 8 ó 7.

En los servidores windows (al menos en los Windows 10) no hay SSH, pero puedes acceder a ellos vía RDP o bien desde la consola de emergencia que se proporciona desde el portal... que, ¡ojo! es gráfica:



![Consola Windows](/images/posts/2020-01-14-windows-console.png)

Además esta consola sigue conectada incluso cuando el ordenador se está iniciando y/o actualizando (lo que una conexión RDP no). Pero, para el día a día, por supuesto RDP va mucho mejor.

![Consola Windows mostrando actualización](/images/posts/2020-01-14-windows-console2.png)

En mi caso he creado un Windows 10, pero realmente parece que es un Windows Server 2016. Así se indica en la propia web:

![Detalles servidor windows](/images/posts/2020-01-14-windows-server.png)

Efectivamente, el ordenador se reporta a si mismo como un Windows Server 2016:

![Servidor Windows system settings](/images/posts/2020-01-14-windows-details.png)

No soy un experto en IT, así que desconozco que configuración es esa o si hay algún modo que Windows 2016 emule a Windows 10, pero la verdad es que en todo momento se tiene la sensación de estar delante de un Windows Server, no de un Windows 10. Vamos, si hasta aparece "Internet Explorer" (no Edge) y el Administrador del Servidor propio de Windows Server y que no existe en Windows 10.

No creo que sea algo especialmente relevante ya que claramente el uso de _Clouding.io_ es para tener entornos productivos (no de desarrollo) por lo que usar un Windows Server es lo más natural, pero me parece curioso que ofrezcan la opción de Windows 10 y luego sea un Server 2016. Cabe mencionar que este aspecto se menciona en la propia web de _Clouding.io_.

Lo interesante es **que la virtualización está activa** en estos servidores virtuales (quiero recalcar eso, porque no siempre es así). Es decir, puedes instalar máquinas virtuales en ellos. En mi caso he podido instalar un Centos, usando una MV Hyper-V en la máquina "Windows 10" que había creado y aunque he tenido que habilitar Hyper-V para ello, no ha habido mayor incoveniente.

## Otras plantillas

_Clouding.io_ ofrece otras plantillas, no solo partir de un SO Linux o Windows, tales como Docker (sobre Ubuntu), o todo un ecosistema LAMP o un Wordpress o Prestashop entre otros. Son las aplicaciones más usadas y supongo que irán agregando más si ven que hay demanda o necesidad.

Puedes crear _snapshots_ de tus servidores, que te sirven luego para crear otros servidores basados en este _snapshot_. Así los _snaphsots_ juegan un doble rol como una "copia de seguridad de emergencia" antes de hacer algo potencialmente peligroso con el servidor, o bien para crear plantillas base. 

![Crear servidor desde snapshot](/images/posts/2020-01-14-create-fromsnapshot.png)

Como se puede observar en la imagen, puedes usar el _snapshot_ para crear otro servidor a partir de él. El nuevo servidor no debe tener la misma memoria/cpu que el servidor del cual se creó el _snapshot_ pero sí que debe tener un disco del, al menos, el mismo tamaño.

## Conclusión

_Clouding.io_ es un servicio 100% _IaaS_ que proporciona servidores virtuales, ni más ni menos. Su principal virtud es una experiencia de usuario muy bien conseguida: es muy fácil para cualquiera crear un servidor y en todo caso se tiene claro el precio que costará. Consta con las funcionalidad básicas que se esperan en este tipo de servicios:

* Plantillas de servidores
* Copias de seguridad
* Reglas de seguridad
* Redes virtuales (para comunicar internamente dos servidores)

Su público objetivo es un público hispano (p. ej. al seleccionar Windows este se instala en castellano, no he visto donde se puede cambiar el idioma del SO) y en especial creo que PYMES que quieran empezar a poner su infraestructura o parte de ella en la nube (y que no quieran gestionar toda la complejidad de los _clouds_ más "enterprise" como AWS o Azure). Ofrecen una experiencia de usuario muy agradable, que te permite en un par de minutos y clicks tener un servidor listo en la nube, todo para tí. Tienen además una base de conocimiento bastante extensa, con numerosos artículos en castellano y se nota que están haciendo un esfuerzo para crear una comunidad a su alrededor y ofrecer un soporte técnico cercano y accesible, que ofrezca un valor añadido.

## Nota

**Nota**: Este es un **post patrocinado**. La gente de [Clouding.io](https://clouding.io/) se han puesto en contacto conmigo para que haga un pequeño análisis de su producto. He estado unos días evaluándolo y escribiendo mis impresiones. El _post_ ha sido visto por ellos antes de ser publicado, aunque debo destacar que no he recibido indicación alguna sobre el contenido (en caso contrario no hubiese aceptado).





