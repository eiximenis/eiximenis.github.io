---
title: Unity, Proxies, AOP y un poco de todo eso…
description: Unity, Proxies, AOP y un poco de todo eso…
author: eiximenis

date: 2009-09-24T11:34:00+00:00
geeks_url: /?p=1468
geeks_visits:
  - 2257
geeks_ms_views:
  - 1289
categories:
  - Uncategorized

---
En mi opinión, usar un contenedor de IoC hoy en día, no es una opción sinó una _obligación_. Las ventajas que nos ofrecen son incotestables. Los patrones [Service Locator][1] y [Dependency Injection][2] nos permiten _desacoplar_ nuestro código, y son la base para poder trabajar de forma modular y poder generar unos tests unitarios de forma más sencilla. Pero hoy no quiero hablaros de ninguno de estos patrones, sinó de otra de las capacidades de los contenedores de IoC: la generación de proxies.

Con esta técnica lo que podemos hacer es _inyectar_ nuestro propio código para que se ejecute antes o después del código que contenga la clase en particular. Esto, si lo combinamos con los atributos nos proporciona unas capacidades potentísimas para poder tener [programación orientada a aspectos][3].

Vamos a ver como realizar esta técnica usando [Unity][4], pero no es exclusiva de este contenedor de IoC, otros contenedores como [Windsor][5] también tienen esta capacidad.

El mecanismo en Unity que nos permite generar proxies a partir de clases del usuario, se llama intercepción. Cuando creamos una intercepción en Unity debemos definir básicamente dos cosas:

  1. _Qué_ ocurre cuando un método es interceptado (la política de intercepción).
  2. _Cómo_ se intercepta un método (el interceptor).

Vamos a ver paso a paso como funciona el mecanimso.

**1. Preparación del entorno**

Vamos a crear una aplicación de consola, y añadimos las referencias a todos los ensamblados de Unity.

Luego vamos a crear una interfaz, y la clase que vamos a interceptar:

<pre class="code"><span style="background: black; color: #3e60fd">public interface </span><span style="background: black; color: #2b91af">IMyInterface
</span><span style="background: black; color: silver">{
</span><span style="background: black; color: #3e60fd"><span style="color: #c0c0c0;">    </span>string </span><span style="background: black; color: white">SomeProperty </span><span style="background: black; color: silver">{ </span><span style="background: black; color: #3e60fd">get</span><span style="background: black; color: silver">; </span><span style="background: black; color: #3e60fd">set</span><span style="background: black; color: silver">; }
}
</span><span style="background: black; color: #3e60fd">public class </span><span style="background: black; color: #2b91af">MyClass </span><span style="background: black; color: silver">: </span><span style="background: black; color: #2b91af">IMyInterface
</span><span style="background: black; color: silver">{
</span><span style="background: black; color: #3e60fd"><span style="color: #c0c0c0;">    </span>public string </span><span style="background: black; color: white">SomeProperty </span><span style="background: black; color: silver">{ </span><span style="background: black; color: #3e60fd">get</span><span style="background: black; color: silver">; </span><span style="background: black; color: #3e60fd">set</span><span style="background: black; color: silver">; }
}</span></pre>

[][6]

Finalmente, en el método Main() creamos un contenedor de Unity y registramos el mapping entre la interfaz y el tipo:

<pre class="code"><span style="background: black; color: #3e60fd">static void </span><span style="background: black; color: white">Main</span><span style="background: black; color: silver">(</span><span style="background: black; color: #3e60fd">string</span><span style="background: black; color: silver">[] </span><span style="background: black; color: white">args</span><span style="background: black; color: silver">)
{
</span><span style="background: black; color: #2b91af"><span style="color: #c0c0c0;">    </span>UnityContainer </span><span style="background: black; color: white">uc </span><span style="background: black; color: cyan">= </span><span style="background: black; color: #3e60fd">new </span><span style="background: black; color: #2b91af">UnityContainer</span><span style="background: black; color: silver">();
    </span><span style="background: black; color: white">uc</span><span style="background: black; color: cyan">.</span><span style="background: black; color: white">RegisterType</span><span style="background: black; color: cyan">&lt;</span><span style="background: black; color: #2b91af">IMyInterface</span><span style="background: black; color: silver">, </span><span style="background: black; color: #2b91af">MyClass</span><span style="background: black; color: cyan">&gt;</span><span style="background: black; color: silver">();
}</span></pre>

[][6]

Ahora estamos listos para empezar!!!

**2. Configuración de Unity para que use un interceptor**

Vamos a configurar Unity para que use un interceptor cuando se resuelva la interfaz IMyInterface. Para ello, primero debemos añadir la extensión de intercepción a Untiy y luego configurarla:

<pre class="code"><span style="background: black; color: #3e60fd">static void </span><span style="background: black; color: white">Main</span><span style="background: black; color: silver">(</span><span style="background: black; color: #3e60fd">string</span><span style="background: black; color: silver">[] </span><span style="background: black; color: white">args</span><span style="background: black; color: silver">)
{
</span><span style="background: black; color: #2b91af"><span style="color: #c0c0c0;">    </span>UnityContainer </span><span style="background: black; color: white">uc </span><span style="background: black; color: cyan">= </span><span style="background: black; color: #3e60fd">new </span><span style="background: black; color: #2b91af">UnityContainer</span><span style="background: black; color: silver">();
    </span><span style="background: black; color: white">uc</span><span style="background: black; color: cyan">.</span><span style="background: black; color: white">RegisterType</span><span style="background: black; color: cyan">&lt;</span><span style="background: black; color: #2b91af">IMyInterface</span><span style="background: black; color: silver">, </span><span style="background: black; color: #2b91af">MyClass</span><span style="background: black; color: cyan">&gt;</span><span style="background: black; color: silver">();
    </span><span style="background: #151515; color: green">// Añadimos la extensión de intercepción
</span><span style="background: black; color: silver">    </span><span style="background: black; color: white">uc</span><span style="background: black; color: cyan">.</span><span style="background: black; color: white">AddNewExtension</span><span style="background: black; color: cyan">&lt;</span><span style="background: black; color: #2b91af">Interception</span><span style="background: black; color: cyan">&gt;</span><span style="background: black; color: silver">();
    </span><span style="background: #151515; color: green">// La configuramos para que nos devuelva
</span><span style="background: black; color: silver">    </span><span style="background: #151515; color: green">// un TransparentProxy cuando resolvamos </span><span style="background: #151515; color: green">IMyInterface
</span><span style="background: black; color: white"><span style="color: #c0c0c0;">    </span>uc</span><span style="background: black; color: cyan">.</span><span style="background: black; color: white">Configure</span><span style="background: black; color: cyan">&lt;</span><span style="background: black; color: #2b91af">Interception</span><span style="background: black; color: cyan">&gt;</span><span style="background: black; color: silver">()</span><span style="background: black; color: cyan">.
    </span><span style="background: black; color: white">SetInterceptorFor</span><span style="background: black; color: cyan">&lt;</span><span style="background: black; color: #2b91af">IMyInterface</span><span style="background: black; color: cyan">&gt;
        </span><span style="background: black; color: silver">(</span><span style="background: black; color: #3e60fd">new </span><span style="background: black; color: #2b91af">TransparentProxyInterceptor</span><span style="background: black; color: silver">());
    </span><span style="background: black; color: #3e60fd">var </span><span style="background: black; color: white">u </span><span style="background: black; color: cyan">= </span><span style="background: black; color: white">uc</span><span style="background: black; color: cyan">.</span><span style="background: black; color: white">Resolve</span><span style="background: black; color: cyan">&lt;</span><span style="background: black; color: #2b91af">IMyInterface</span><span style="background: black; color: cyan">&gt;</span><span style="background: black; color: silver">();
    </span><span style="background: black; color: white">u</span><span style="background: black; color: cyan">.</span><span style="background: black; color: white">SomeProperty </span><span style="background: black; color: cyan">= </span><span style="background: black; color: gray">"test"</span><span style="background: black; color: silver">;
}</span></pre>

[][6][][6][][6]

Vamos a meter un breakpoint en la última línea y a ejecutar el código, para ver si Unity ha echo algo:

[<img height="90" width="454" src="/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_42A5351F.png" alt="image" border="0" title="image" style="border-bottom: 0px; border-left: 0px; display: inline; border-top: 0px; border-right: 0px" />][7] 

Vaya... pues no parece que haya hecho nada, la verdad. La variable u es de tipo MyClass, no parece haber ningún proxy por ahí...

Es normal, ya que hemos configurado Unity para que use un TransparentProxy al resolver la interfaz IMyInterface, pero no lo hemos dicho _que_ debe hacer Unity con este proxy, así que simplemente para no hacer nada, no crea ni el proxy...

**3. Crear el interceptor**

Ha llegado el momento de crear un interceptor, que defina _que_ ocurre cuando se intercepta un método o propiedad de la clase. Para ello vamos a crear una clase nueva que implementa la interfaz ICallHandler:

<pre class="code"><span style="background: black; color: #3e60fd">public class </span><span style="background: black; color: #2b91af">MyHandler </span><span style="background: black; color: silver">: </span><span style="background: black; color: #2b91af">ICallHandler
</span><span style="background: black; color: silver">{
</span><span style="background: black; color: #3e60fd"><span style="color: #c0c0c0;">    </span>public </span><span style="background: black; color: #2b91af">IMethodReturn </span><span style="background: black; color: white">Invoke</span><span style="background: black; color: silver">(</span><span style="background: black; color: #2b91af">IMethodInvocation </span><span style="background: black; color: white">input</span><span style="background: black; color: silver">, <br />        </span><span style="background: black; color: #2b91af">GetNextHandlerDelegate </span><span style="background: black; color: white">getNext</span><span style="background: black; color: silver">)
    {
        </span><span style="background: black; color: #2b91af">IMethodReturn </span><span style="background: black; color: white">msg </span><span style="background: black; color: cyan">= </span><span style="background: black; color: white">getNext</span><span style="background: black; color: silver">()(</span><span style="background: black; color: white">input</span><span style="background: black; color: silver">, </span><span style="background: black; color: white">getNext</span><span style="background: black; color: silver">);
        </span><span style="background: black; color: #3e60fd">return </span><span style="background: black; color: white">msg</span><span style="background: black; color: silver">;
     }
        </span><span style="background: black; color: #3e60fd">public int </span><span style="background: black; color: white">Order </span><span style="background: black; color: silver">{ </span><span style="background: black; color: #3e60fd">get</span><span style="background: black; color: silver">; </span><span style="background: black; color: #3e60fd">set</span><span style="background: black; color: silver">; }
    }</span></pre>

[][6]

Esta es la implementación _básica_ por defecto de ICallHandler: no estamos haciendo nada, salvo pasar la llamada a la propiedad _real_ de la clase. Es decir, nuestro interceptor no está haciendo realmente nada.

Podemos añadir aquí el código que queramos, p.ej:

<pre class="code"><span style="background: black; color: #3e60fd">public </span><span style="background: black; color: #2b91af">IMethodReturn </span><span style="background: black; color: white">Invoke</span><span style="background: black; color: silver">(</span><span style="background: black; color: #2b91af">IMethodInvocation </span><span style="background: black; color: white">input</span><span style="background: black; color: silver">, <br />     </span><span style="background: black; color: #2b91af">GetNextHandlerDelegate </span><span style="background: black; color: white">getNext</span><span style="background: black; color: silver">)
{
</span><span style="background: black; color: #2b91af"><span style="color: #c0c0c0;">    </span>Console</span><span style="background: black; color: cyan">.</span><span style="background: black; color: white">WriteLine</span><span style="background: black; color: silver">(</span><span style="background: black; color: gray">"Se ha llamado {0} con valor {1}"</span><span style="background: black; color: silver">, <br />     </span><span style="background: black; color: white">input</span><span style="background: black; color: cyan">.</span><span style="background: black; color: white">MethodBase</span><span style="background: black; color: cyan">.</span><span style="background: black; color: white">Name</span><span style="background: black; color: silver">, </span><span style="background: black; color: white">input</span><span style="background: black; color: cyan">.</span><span style="background: black; color: white">Inputs</span><span style="background: black; color: silver">[</span><span style="background: black; color: yellow"></span><span style="background: black; color: silver">]);
    </span><span style="background: black; color: #2b91af">IMethodReturn </span><span style="background: black; color: white">msg </span><span style="background: black; color: cyan">= </span><span style="background: black; color: white">getNext</span><span style="background: black; color: silver">()(</span><span style="background: black; color: white">input</span><span style="background: black; color: silver">, </span><span style="background: black; color: white">getNext</span><span style="background: black; color: silver">);
</span><span style="background: black; color: #3e60fd"><span style="color: #c0c0c0;">    </span>return </span><span style="background: black; color: white">msg</span><span style="background: black; color: silver">;
}</span></pre>

[][6]

**4. Indicar que métodos / propiedades queremos interceptar**

Tenemos a Unity configurado para usar intercepción, y un interceptor creado... ahora nos queda finalmente vincular este interceptor con las propiedades o métodos que deseemos.

Para ello podemos usar los atributos: la idea es _decorar_ cada propiedad o método con un atributo que indique _que_ interceptor se usa para dicha propiedad y además configure dicho interceptor. Así, generalmente, vamos a usar un atributo para cada interceptor. En nuestro caso tenemos un sólo interceptor (MyHandler), así que añadiremos un atributo MyHandlerAttribute.

Para ello vamos a crear una clase que derive de HandlerAttribute (la cual a su vez deriva de Attribute), y redefinir el método CreateHandler. En este método debemos devolver el Handler que deseemos:

<pre class="code"><span style="background: black; color: silver">[</span><span style="background: black; color: #2b91af">AttributeUsage</span><span style="background: black; color: silver">(</span><span style="background: black; color: #2b91af">AttributeTargets</span><span style="background: black; color: cyan">.</span><span style="background: black; color: white">Property</span><span style="background: black; color: silver">)]
</span><span style="background: black; color: #3e60fd">class </span><span style="background: black; color: #2b91af">MyHandlerAttribute </span><span style="background: black; color: silver">: </span><span style="background: black; color: #2b91af">HandlerAttribute
</span><span style="background: black; color: silver">{
</span><span style="background: black; color: #3e60fd"><span style="color: #c0c0c0;">    </span>public override </span><span style="background: black; color: #2b91af">ICallHandler </span><span style="background: black; color: white">CreateHandler<br />        </span><span style="background: black; color: silver">(</span><span style="background: black; color: #2b91af">IUnityContainer </span><span style="background: black; color: white">container</span><span style="background: black; color: silver">)
    {
        </span><span style="background: black; color: #3e60fd">return new </span><span style="background: black; color: #2b91af">MyHandler</span><span style="background: black; color: silver">();
    }
}</span></pre>

En este caso nuestro atributo es trivial, pero en otros casos el atributo puede tener parámetros (y pasárselos al constructor del interceptor, o incluso crear un interceptor u otro en función de dichos parámetros).

**5. Aplicar el atributo a las propiedades que deseemos**

Para ello simplemente decoramos las propiedades (o métodos) que deseemos con el atributo:

<pre class="code"><span style="background: black; color: #3e60fd">public class </span><span style="background: black; color: #2b91af">MyClass </span><span style="background: black; color: silver">: </span><span style="background: black; color: #2b91af">IMyInterface
</span><span style="background: black; color: silver">{
    [</span><span style="background: black; color: #2b91af">MyHandler</span><span style="background: black; color: silver">]
    </span><span style="background: black; color: #3e60fd">public string </span><span style="background: black; color: white">SomeProperty </span><span style="background: black; color: silver">{ </span><span style="background: black; color: #3e60fd">get</span><span style="background: black; color: silver">; </span><span style="background: black; color: #3e60fd">set</span><span style="background: black; color: silver">; }
}</span></pre>

[][6]

Y.... ya hemos terminado! Si colocamos un breakpoint en el mismo lugar de antes, veremos que ahora si que Unity nos ha creado un proxy:

[<img height="88" width="508" src="/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_70B7332C.png" alt="image" border="0" title="image" style="border-bottom: 0px; border-left: 0px; display: inline; border-top: 0px; border-right: 0px" />][8] 

Y si ejecutamos el programa, veréis como la salida por pantalla es la siguiente:

<table border="1">
  <tr>
    <td>
      <pre>Se ha llamado set_SomeProperty con valor test</pre>
    </td>
  </tr>
</table>

Nuestro interceptor ha sido llamado... hemos triunfado!!! 😉

Si añadís una propiedad extra a la interfaz (y a la clase) y NO la decoráis con el atributo veréis que, obviamente dicha propiedad NO es interceptada.

Esta técnica tiene unas posibilidades **brutales**... a mi se me ocurren a brote pronte, temas de logging, seguridad, validación de propiedades... vamos, todo aquello en lo que es aplicable la programación orientada a aspectos!

Un saludo a todos! 😉

 [1]: http://msdn.microsoft.com/en-us/library/cc707905.aspx
 [2]: http://msdn.microsoft.com/en-us/library/dd458879.aspx
 [3]: http://es.wikipedia.org/wiki/Programaci%C3%B3n_Orientada_a_Aspectos
 [4]: http://www.codeplex.com/unity/
 [5]: http://www.castleproject.org/container/index.html
 [6]: http://11011.net/software/vspaste
 [7]: /cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_0969ACC8.png
 [8]: /cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_41CF5FE6.png