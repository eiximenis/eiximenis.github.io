---
title: 'STM.NET: Software Transactional Memory'
description: 'STM.NET: Software Transactional Memory'
author: eiximenis

date: 2009-09-02T14:50:00+00:00
geeks_url: /?p=1463
geeks_visits:
  - 1257
geeks_ms_views:
  - 1236
categories:
  - Uncategorized

---
Uno de los retos más importantes a los que se enfrenta en breve el desarrollo de aplicaciones tiene que ver con la programación paralela. Ahora que se empieza a vislumbrar el acercamiento del fin de la [Ley de Moore][1], si queremos seguir el espectacular aumento de potencia debemos irnos a entornos multi-procesador o multi-core. Hace unos años eran coto reservado a entornos de investigación, y ahora ya están encima de nuestra mesa...

Hay varias visiones sobre como se debe atacar este problema, cual debe ser el rol del desarrollador, de los frameworks y del sistema operativo para asegurar una buena productividad y a la vez ser capaz de asegurar un óptimo consumo de los distintos recursos. Como dijo Rodrigo una vez... La [Broma ha terminado!][2]

La versión 4.0 del framework trae novedades al respecto, en concreto las famosas [Parallel Extensions][3] (que también [pueden descargarse para funcionar con el framework 3.5][4]), que permiten un nivel de abstracción más alto y facilitan el uso de la programación concurrente. Se ha hablado largo y tendido en geeks sobre las parallel.

Por otro lado, hace ya bastante tiempo que Microsoft tiene el [DevLabs][5], una especie de laboratorio de investigación de donde salen proyectos de investigación que en un futuro pueden ver la luz (como productos independientes o bien integrándose en otros como puede ser .NET Framework). Del DevLabs han salido cosas tan interesantes como [Code Contracts][6] (de las que ya he hablado en este mismo blog), [Pex][7], [Small Basic][8] (una versión de Basic para aprender a programar) y lo que os quiero comentar en este post: [STM.NET][9].

**¿Que es STM.NET?**

STM.NET son unas extensiones sobre el framework 4.0 que implementa el concepto de Software Transactional Memory, un mecanismo que nos permite aislar zonas de código para que se ejecuten de forma segura en un entorno multiprocesador. STM.NET no ofrece nuevas abstracciones sobre tareas concurrentes (como hacen las Parallel) sinó que se centra en _como proteger las zonas de código compartido_. Evidentemente esto no es nuevo, .NET ya ofrece un mecanismo para la protección de código compartido: los [locks][10] y toda una pléyade de clases para la sincronización de threads.

¿Entonces para que STM?&nbsp; STM ataca el problema desde otro punto de vista: usando el concepto de transacciones. En concreto de las propiedades ACID (atomicidad _&ndash; atomicy_, coherencia &ndash; _consistency_, aislamiento &ndash; _isolation_ y permanencia &ndash; _durability_), STM usa dos: La atomicidad y el aislamiento.

Así, del mismo modo que se definen transacciones de bases de datos, podemos definir _transacciones de código_ de tal manera que sean atomicas (o se ejecuta **todo** el código o no se ejecuta **nada**) y aisladas (si se ejecutan varias transacciones de forma concurrente sus efectos serán como si se ejecutaran una tras otra).

**Instalación de STM.NET**

Para instalar STM.NET se requiere Visual Studio 2008 y **no**&nbsp; se soporta la beta de Visual Studio 2010: si instalas STM.NET en una máquina con Visual Studio 2010 [vas a tener problemas][11]. Desde la página web de STM.NET puede instalarse una versión modificada del .NET Framework 4.0 (que se instala con la versión 4.0.20506) y el adaptador para Visual Studio 2008, que nos permite crear aplicaciones de consola usando STM.NET. Una vez instalados ambos estamos listos para empezar 🙂

**El primer ejemplo...**

Vamos a empezar con un trozo simple de código donde dos threads van a acceder a la misma variable compartida:

<pre class="code"><span style="color: blue">static int </span>sharedValue = 0;
<span style="color: blue">static void </span>Main(<span style="color: blue">string</span>[] args)
{
<span style="color: green">    </span><span style="color: #2b91af">Thread </span>t1 = <span style="color: blue">new </span><span style="color: #2b91af">Thread</span>(ThreadProc);
    <span style="color: #2b91af">Thread </span>t2 = <span style="color: blue">new </span><span style="color: #2b91af">Thread</span>(ThreadProc);
    t1.Start();
    t2.Start();
    t1.Join();
    t2.Join();
    <span style="color: #2b91af">Console</span>.WriteLine(<span style="color: #a31515">"sharedValue es {0}"</span>, sharedValue);
    <span style="color: #2b91af">Console</span>.ReadLine();
}
<span style="color: blue">static void </span>ThreadProc()
{
    <span style="color: blue">for </span>(<span style="color: blue">int </span>i = 0; i &lt; 100; i++)
    {
        sharedValue++;
<span style="color: green">        </span><span style="color: #2b91af">Thread</span>.Sleep(10);
    }
}</pre>

Si ejecutais este programa la salida debería ser 200, ya que los dos threads incrementan cada uno 100 veces la variable. Pero como sabemos, threads accediendo a datos compartidos... es sinónimo de problemas. Actualmente para solucionar este problema podríamos usar un lock, pero STM.NET nos da otro mecanismo: crear una _transacción_ para acceder a la variable sharedValue. Actualmente para crear una transacción se usa el método Do, de la clase Atomic y se le pasa un delegate con el método que tiene el código de la transacción.

Así modificamos ThreadProc para que quede:

<pre class="code"><span style="color: blue">static void </span>ThreadProc()
{
    <span style="color: blue">for </span>(<span style="color: blue">int </span>i = 0; i &lt; 100; i++)
    {
        <strong><span style="color: #2b91af">Atomic</span>.Do(() =&gt; sharedValue++);</strong>
        <span style="color: #2b91af">Thread</span>.Sleep(10);
    }
}</pre>

Con esto todo el código que hemos metido dentro del Atomic.Do (en nuestro caso el incremento de la variable) se ejecutará dentro de una transacción, y por el principio de aislamiento, si dos (o más) transacciones se ejecutan a la vez, sus efectos serán los mismos que si se ejecutasen una tras otra. En efecto, ahora al finalizar el programa sharedValue siempre vale 200.

Este es el ejemplo más sencillo: como imitar un lock usando STM.

**El segundo ejemplo...**

Veamos ahora la propiedad de atomicidad, y como Atomic.Do() **no es** lo mismo que usar un lock de C#.

Si tenemos el siguiente código:

<pre class="code"><span style="color: blue">int </span>_x = 10;
<span style="color: blue">int </span>_y = 20;
<span style="color: blue">try
</span>{
    <span style="color: #2b91af">Atomic</span>.Do(() =&gt;
    {
        _x++;
        _y--;
        <span style="color: blue">throw new </span><span style="color: #2b91af">Exception</span>();
    });
}
<span style="color: blue">catch </span>(<span style="color: #2b91af">Exception</span>)
{
    <span style="color: #2b91af">Console</span>.WriteLine(<span style="color: #a31515">"_x = {0}, _y = {1}"</span>, _x, _y);
}</pre>

[][12]

Cual es el valor de las variables dentro del catch? Pues \_x seguirá teniendo el valor de 10 e \_y seguirá teniendo el valor de 20. Al lanzarse una excepción dentro de la transacción se realiza un _rollback_, de forma que las variables recuperan el valor original que tenian _antes_ de entrar en la transacción (principio de atomicidad).

**Resumiendo...**

Este post ha sido una muy breve introducción a [STM.NET][9]. Obviamente se han quedado cosas en el tintero, pero sólo queria mostraros un poco algunas de las ideas que se están cociendo en los DevLabs... El tiempo dirá si estas ideas se terminan consolidando en alguna versión futura del Framework (como Code Contracts) o se quedan en el olvido...

Saludos!

 [1]: http://es.wikipedia.org/wiki/Ley_de_Moore
 [2]: /blogs/jalarcon/archive/2008/10/10/procesadores-multicore-amenaza-para-la-industria.aspx
 [3]: http://blogs.msdn.com/pfxteam/
 [4]: http://www.microsoft.com/downloads/details.aspx?FamilyId=348F73FD-593D-4B3C-B055-694C50D2B0F3&displaylang=en
 [5]: http://msdn.microsoft.com/en-us/devlabs/default.aspx
 [6]: http://msdn.microsoft.com/en-us/devlabs/dd491992.aspx
 [7]: http://research.microsoft.com/en-us/projects/Pex/
 [8]: http://msdn.microsoft.com/en-us/devlabs/cc950524.aspx
 [9]: http://msdn.microsoft.com/en-us/devlabs/ee334183.aspx
 [10]: http://msdn.microsoft.com/en-us/library/system.threading.monitor(VS.71).aspx
 [11]: http://social.msdn.microsoft.com/Forums/en-US/stmdevlab/thread/8ece5a15-d02e-410d-ba23-30f778dad00d/
 [12]: http://11011.net/software/vspaste