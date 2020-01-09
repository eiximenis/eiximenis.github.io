---
title: 'STM.NET: Software Transactional Memory (ii)'
description: 'STM.NET: Software Transactional Memory (ii)'
author: eiximenis

date: 2009-09-07T11:26:00+00:00
geeks_url: /?p=1464
geeks_visits:
  - 783
geeks_ms_views:
  - 701
categories:
  - Uncategorized

---
Hola a todos! En el [post anterior][1] os comenté algunas cosillas sobre STM.NET, un &ldquo;experimento&rdquo; de los DevLabs de Microsoft para introducir conceptos transaccionales dentro de .NET. En este segundo post quiero extenderme un poco más con algunos ejemplos un pelín más elaborados.

**Ejemplo 3**

En el ejemplo 2 (del post anterior) vimos como lanzar una excepción dentro de una transacción definida por Atomic.Do() hacía un rollback de todos los cambios producidos dentro de la transacción. Vamos a verlo con más detalle.

Empezamos por definir una clase &ldquo;Cuenta&rdquo; para permitir ingresar o quitar determinadas cantidades de dinero:

<pre class="code"><span style="color: blue">class </span><span style="color: #2b91af">Account
</span>{
    <span style="color: blue">public </span>Account(<span style="color: blue">int </span>saldoInicial)
    {
        Saldo = saldoInicial;
    }
    <span style="color: blue">public int </span>Saldo { <span style="color: blue">get</span>; <span style="color: blue">private set</span>; }
    <span style="color: blue">public void </span>Ingreso(<span style="color: blue">int </span>i)
    {
        Saldo += i;
    }
    <span style="color: blue">public void </span>Dispensacion(<span style="color: blue">int </span>i)
    {
        Saldo -= i;
        <span style="color: blue">if </span>(Saldo &lt; 0)
            <span style="color: blue">throw new </span><span style="color: #2b91af">Exception</span>(
                <span style="color: blue">string</span>.Format(<span style="color: #a31515">"Error al sacar {0} eur"</span>, i));
<span style="color: green">    </span>}
}</pre>

[][2]

Es una clase normal de .NET, sin ninguna otra particularidad. Ahora vamos a hacer un programilla para permitir ir haciendo transferencias de una cuenta a otra:

<pre class="code"><span style="color: blue">class </span><span style="color: #2b91af">Program
</span>{
    [<span style="color: #2b91af">AtomicNotSupported</span>]
    <span style="color: blue">static void </span>Main(<span style="color: blue">string</span>[] args)
    {
        <span style="color: #2b91af">Account </span>cuentaDestino = <span style="color: blue">new </span><span style="color: #2b91af">Account</span>(0);
        <span style="color: #2b91af">Account </span>cuentaOrigen = <span style="color: blue">new </span><span style="color: #2b91af">Account</span>(100);
        <span style="color: #2b91af">Console</span>.WriteLine(<span style="color: #a31515">"=== INICIO ==="</span>);
        <span style="color: #2b91af">Console</span>.WriteLine(<span style="color: #a31515">"Destino {0} eur"</span>, cuentaDestino.Saldo);
        <span style="color: #2b91af">Console</span>.WriteLine(<span style="color: #a31515">"Origen {0} eur"</span>, cuentaOrigen.Saldo);
        <span style="color: blue">int </span>transferir = 0;
        <span style="color: blue">do
        </span>{
            transferir = 0;
            <span style="color: #2b91af">Console</span>.WriteLine(<span style="color: #a31515">"Entrar fondos a transferir..."</span>);
            <span style="color: blue">string </span>s = <span style="color: #2b91af">Console</span>.ReadLine();
            <span style="color: blue">int </span>valor;
            <span style="color: blue">if </span>(<span style="color: blue">int</span>.TryParse(s, <span style="color: blue">out </span>valor))
            {
                transferir = valor;
                <span style="color: blue">try
                </span>{
                    <span style="color: #2b91af">Console</span>.WriteLine
                        (<span style="color: #a31515">"=== Transfiriendo {0} eur ==="</span>,
                        transferir);
                    Transferencia(cuentaOrigen, cuentaDestino,
                        transferir);
                }
                <span style="color: blue">catch </span>(<span style="color: #2b91af">Exception </span>ex)
                {
                    <span style="color: #2b91af">Console</span>.WriteLine(<span style="color: #a31515">"ERROR en transferencia: {0} "</span>,
                        ex.Message);
                }
            }
            <span style="color: #2b91af">Console</span>.WriteLine(<span style="color: #a31515">"=== SALDOS ==="</span>);
            <span style="color: #2b91af">Console</span>.WriteLine(<span style="color: #a31515">"Destino {0} eur"</span>, cuentaDestino.Saldo);
            <span style="color: #2b91af">Console</span>.WriteLine(<span style="color: #a31515">"Origen {0} eur"</span>, cuentaOrigen.Saldo);
        } <span style="color: blue">while </span>(transferir != 0);
    }
    <span style="color: blue">static void </span>Transferencia(<span style="color: #2b91af">Account </span>origen, <span style="color: #2b91af">Account </span>destino,
        <span style="color: blue">int </span>qty)
    {
        destino.Ingreso(qty);
        origen.Dispensacion(qty);
<span style="color: green">    </span>}
}</pre>

[][2]

Más simple el agua... El programa va haciendo transferencias de una cuenta a otra. Aquí teneis la salida del programa después de varias ejecuciones:

<table border="1">
  <tr>
    <td>
      <pre>=== INICIO ===
Destino 0 eur
Origen 100 eur
Entrar fondos a transferir...
120
=== Transfiriendo 120 eur ===
ERROR en transferencia: Error al sacar 120 eur
=== SALDOS ===
Destino 120 eur
Origen -20 eur
Entrar fondos a transferir...</pre>
    </td>
  </tr>
</table>

Al hacer la transferencia de 120, nos salta la excepción (capturada correctamente por la función Main), pero hay dos problemas:

  1. Los 100 euros se han ingresado ya en la cuenta de destino 
  2. La cuenta de origen se ha quedado en &ndash;20 

El punto (1) es debido a como se ha codificado la función Transferencia (que primero hace el ingreso y luego el cargo) y el punto (2) es debido a que la función Account.Dispensacion quita el dinero primero y lanza la excepción después. Vale, con un par de ifs esto se arregla, pero es un buen punto de partida para ver como funciona STM.NET.

Aquí está claro que una transferencia es una operación atómica: O bien se hace el cargo y luego el abono, o bien no se hace nada. Si es una operación atómica, lo ideal es ejecutarla dentro de un Atomic.Do(). Podemos modificar la función Transferencia, para que quede con el siguiente código:

<pre class="code"><span style="color: blue">static void </span>Transferencia(<span style="color: #2b91af">Account </span>origen, <span style="color: #2b91af">Account </span>destino,
    <span style="color: blue">int </span>qty)
{
    <span style="color: #2b91af">Atomic</span>.Do(() =&gt;
    {
        destino.Ingreso(qty);
        origen.Dispensacion(qty);
    });
}</pre>

[][2]

Si ahora ejecutamos el programa la salida que tenemos es:

<table border="1">
  <tr>
    <td>
      <pre>=== INICIO ===
Destino 0 eur
Origen 100 eur
Entrar fondos a transferir...
120
=== Transfiriendo 120 eur ===
ERROR en transferencia: Error al sacar 120 eur
=== SALDOS ===
Destino 0 eur
Origen 100 eur
Entrar fondos a transferir...</pre>
    </td>
  </tr>
</table>

La llamada a origen.Dispensacion genera una excepción que provoca que se haga un rollback. El rollback afecta tanto a la cuenta de destino (a la cual ya se le habían ingresado los 120 euros) como a la cuenta de origen (a la cual ya se le habían quitado los 120 euros, quedando en negativo).

Simple, sencillo y elegante.

**Ejemplo 4: Supresión parcial de una transacción**

Dentro de una transacción el código que se ejecuta debe estar preparado para ejecutarse en una transacción. Hay métodos que por su naturaleza no pueden ser transaccionales (porque realizan efectos visibles al usuario). P.ej el siguiente código falla:

<pre class="code"><span style="color: #2b91af">Atomic</span>.Do(() =&gt;  <span style="color: #2b91af">Console</span>.WriteLine(<span style="color: #a31515">"Hola Mundo"</span>));</pre>

[][2]

Lanza una excepción _AtomicContractViolationException_ indicando que Console.WriteLine se accede dentro de un bloque transaccional pero está marcado con el atributo [AtomicNotSupported] indicando que **no** se soportan transacciones.

A veces (p.ej. para depurar) nos puede interesar llamar a métodos que no soportan transacciones (y por lo tantos marcados mediante [AtomicNotSupported]). En este caso debemos encapsular las llamadas a este método desde otro método, decorado con [AtomicSupress]. El atributo [AtomicSupress] _suspende_ temporalmente la transacción. En este caso deberíamos tener el siguiente código:

<pre class="code">[<span style="color: #2b91af">AtomicSupported</span>, <span style="color: #2b91af">AtomicSuppress</span>]
<span style="color: blue">public void </span>Log([<span style="color: #2b91af">AtomicMarshalReadonly</span>] <span style="color: blue">string </span>msg)
{
    <span style="color: #2b91af">Console</span>.WriteLine(msg);
}</pre>

[][2]

El atributo [AtomicSupported] indica que el método Log _soporta_ ser llamado desde una transacción, mientras que el atributo [AtomicSuppress] lo que hace es suspender la transacción de forma temporal, permitiendo así que dentro del método Log se llamen a métodos decorados con [AtomicNotSupported] como Console.WriteLine.

Si el método recibe referencias a objetos **creados** dentro de la transacción, estas referencias deben ser serializadas (código fuera de transacción no puede acceder directamente a referencias creadas dentro de código transaccional). Por suerte para nosotros de este trabajo se puede encargar STM.NET: sólo debemos decorar el parámetro con el atributo [AtomicMarshalReadonly].

**Ejemplo 5: Integración con LTM i DTC**

Las transacciones iniciadas con STM se integrarán con LTM y DTC:

<pre class="code"><span style="color: #2b91af">Atomic</span>.Do(() =&gt;
{
    <span style="color: green">// Operaciones en memoria
    // Operaciones sobre BBDD
    // Otras operaciones (p.ej MSMQ)
</span>});</pre>

[][2]

La transacción se inicia como una transacción interna de STM (_relativamente_ poco costosa y cuando sea necesario (p.ej. llamada a BBDD o MSQM) se promocionará a una transacción LTM (que a su vez puede ser promocionada de nuevo a una transacción DTC).

De todos modos esa funcionalidad está todavia en desarrollo **y funciona sólo** para el caso de MSMQ: no es posible usar (de momento) ADO.NET dentro de Atomic.Do() aunque obviamente es algo que está previsto.

**Ejemplo 6: Compensaciones**

No siempre es posible hacer un rollback: hay veces que una acción tiene efectos visibles inmediatos y no es posible deshacerlos sin que el usuario se entere. En este caso lo que se hace en el rollback es hacer _otra_ acción que compense la acción que se quiere anular. Esto rompe el concepto de &ldquo;Isolación&rdquo; ya que el usuario percibe operaciones que no debería percibir (la priemra acción y la acción de compensación), pero en muchos casos es la única alternativa posible.

STM proporciona soporte para comensaciones, usando el método Atomic.DoWithCompensation(). Este método toma dos delegates, con la acción a realizar y la acción de compensación. La acción a realizar se ejecuta de inmediato y la acción de compensación se ejecutará siempre que sea necesario hacer un rollback de la acción. Ambas acciones se ejecutan dentro de un contexto decorado con [AtomicSupress], por lo que pueden llamar a métodos que no soporten transacciones.

Dicho esto... no he conseguido hacer funcionar Atomic.DoWithCompensation(). Intente lo que intente me da una excepción (System error) y ocasionalmente una NullReferenceException. Incluso el propio ejemplo que viene con STM.NET me da esos errores... igual se me está pasando algo por alto 🙁

**Resumen**

Bueno... espero que este post, [junto con anterior][3] os hayan servido para comprender un poco que es STM.NET. Recordad que es un proyecto de DevLabs y por lo tanto no es, ni mucho menos, un producto terminado (de hecho, su licencia prohibe usarlo para realizar s/w que deba ir en producción)... El tiempo dirá si algunas, o todas, de sus ideas las terminamos viendo en alguna versión del Framework 😉

[Página principal de STM.NET][4]

[STM.NET versus TransactionScope][5]

 [1]: /blogs/etomas/archive/2009/09/02/stm-net-software-transactional-memory.aspx
 [2]: http://11011.net/software/vspaste
 [3]: http://burbujasnet.blogspot.com/2009/09/stmnet-software-transactional-memory.html
 [4]: http://msdn.microsoft.com/en-us/devlabs/ee334183.aspx
 [5]: http://social.msdn.microsoft.com/Forums/en-US/stmdevlab/thread/c970e953-b833-42da-a9b7-886fde2ab372