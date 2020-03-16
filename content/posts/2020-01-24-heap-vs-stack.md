---
title: Heap vs Stack
author: eiximenis
description: En este post vamos a ver las diferencias entre la pila (stack) y el heap (lo siento, pero "montículo" no me gusta nada xD) y como diferentes lenguajes gestionan estos dos ámbitos de almacenamiento de valores. Compararemos Rust, Go, C++ y por supuesto nuestro amado C#
date: 2020-01-20T18:20:00
categories:
  - programacion
---

Es curioso como hoy en día mucha gente no tiene nada clara la diferencia entre la pila y el _heap_, que son los dos "lugares" donde se pueden almacenar valores (tales como objetos). Por norma general (salvo que uses _interop_ para gestionar memoria directamente con funciones provistas por el sistema operativo) tus objetos terminarán o bien en la pila, o bien en el _heap_.

* La **pila es rapidísima** pero su alcance se limita al de la función: es decir, cada función tiene su propia pila y esa es destruída una vez se sale de la función. Almacenar y recuperar datos de la pila es más rápido que hacerlo en el _heap_. La pila **es ideal para guardar valores locales a una función**.
* El _heap_ es más lento, **pero su alcance es el de la aplicación**: un valor almacenado en el _heap_ persiste hasta que es destruído explícitamente o bien hasta que el programa termina. El _heap_ pues debe usarse para guardar todos aquellos que deben vivir más allá de una función.

## Implicaciones de que un objeto esté en la pila

La implicación principal es **no podemos devolver ningún valor que esté en la pila** (recuerda que la pila se destruye al salir de una función), **a no ser que hagamos una copia de él**: en este caso la función llamante recibe una copia del valor de retorno y por lo tanto cuando el valor original es destruído (al destruirse la pila) no hay problema.

Debemos distinguir también lo que es semántica de referencia y semántica de valor:

* Una variable de tipo con semántica de referencia no contiene el objeto si no "una referencia (apuntador, puntero)" al objeto
* Una variable de tipo con semántica de valor es en sí misma el valor
* Copiar una variable de tipo con semántica de referencia, copia la referencia no el valor.
* Copiar una variable de tipo con semántica de valor copia el valor en sí mismo.

## C#

Veamos como gestiona C# este tema. En C# los tipos se separan en dos grandes grupos: tipos por referencia (clases) y tipos por valor (estructuras). Todo tipo, incluyendo los tipos "simples" o es una clase (como p. ej. `string` que realmente es `System.String`) o una estructura (como p. ej. `int` que realmente es `System.Int32`).

De forma general podemos asumir que:
* Los tipos por referencia se almacenan siempre en el _heap_
* Los tipos por valor se almacenan siempre en la pila, a no ser que:
  * Sean campos de un objeto que termina almacenado en el _heap_.

Un objeto almacenado en la pila puede referenciar a uno almacenado en el _heap_, pero nunca al revés. Es por ello que si tienes una clase, y uno de sus miembros es una estructura, cuando crees un objeto de dicha clase, éste terminará en el _heap_ incluyendo la estructura que forma parte de él:

```cs
struct S {}
class A{
  private S s;
}

var s = new S();    // s está en la pila
var a = new A();    // a está en el heap, incluyendo a.s
```

Las clases tienen semántica de referencia (de ahí su nombre de "tipos por referencia") mientras que las estructuras tienen semántica de valor:

```csharp
struct S{
  public int i;
}

var s = new S();      // Tenemos un objeto S en la pila
var s2 = s;           // Semántica de valor: ahora hay DOS objetos S en la pila
s2.i = 10;            // El valor de s.i no se ve afectado, ya que s2 es otro objeto

class C {
  public int i;
}

var c = new C();      // Tenemos un objeto C en el heap i una referencia c que apunta a él.
var c2 = c;           // Semántica de referencia: Se copia la referencia, no el valor. Tenemos una referencia c2 que apunta al MISMO objeto que c
c2.i = 10;            // Modificar c2.i también modifica c.i, ya que c y c2 son dos referencias al MISMO objeto.
```

Dado que las estructuras tienen semántica de valor, podemos tranquilamente devolver una estructura porque el llamante recibirá una copia:

```csharp
S Foo() {             // TIpo S es struct
  var s = new S(); 
  return s;
}

var s = Foo();
```

En el código anterior, la función `Foo` crea una estructura que se almacena en la pila. Luego cuando devuelve la estructura se hace una copia, por lo que el llamante recibe la copia (que a su vez se almacena en la pila de la función llamante).

Es interesante pensar como funcionaría el mismo código si `S` fuese una clase y no una estructura. En este caso, el objeto `s` creado por `Foo` se almacenaría en el _heap_ pero... ¿qué ocurre con la referencia `s`? ¿Donde está almacenada?

La respuesta es sencilla si entiendes que **las referencias tienen semántica de valor**. Es decir, cuando devuelves (o pasas como parámetro) un tipo por referencia pasas realmente una referencia al objeto del _heap_, pero esa referencia la pasas por valor, es decir **pasas una copia de la referencia original**. La referencia en si misma (la variable `s` creada en `Foo`) se almacena en la pila de `Foo`. Solo la variable, no el objeto de tipo `S` que este si que está en el _heap_. Por lo tanto cuando `Foo` devuelve la variable `s`, se crea una copia **de la variable, es decir, de la referencia** y el llamante obtiene dicha copia. Como es una copia apunta al mismo objeto que sigue existiendo tranquilamente en el _heap_.

### Tipos por valor y referencias

Pasar y devolver un tipo por valor (estructura) realiza una copia como hemos visto. Esto puede ser lento si el tipo es grande (p. ej. una estructura con 40 campos). ¿Podemos forzar a C# para poder pasar un tipo por valor, pero usando una referencia en su lugar? Sí, podemos.

Tenemos hasta **tres** palabras clave que nos permiten declarar que una función que recibe una estructura la quiere recibir por referencia: `out` y `ref` (existentes desde los inicios de los tiempos) e `in` (existente desde C# 7.2).

```csharp
void Foo(ref S s) {
  s.i=10;
}

var s = new S();
s.i = 100;
Foo(ref s);
// s.i es 10 aquí (no 100)
```

`ref` y `out` son muy parecidas y básicamente permiten a la función receptora modificar el objeto pasado. La diferencia entre `ref` y `out` es que la segunda **obliga** (la primera solo lo permite) a asignar el parámetro con un valor:

```csharp
void Foo (out S s) {
  s = new S() {...}   // La función DEBE crear un valor y asignarlo al parámetro out
}
```

La idea es que `out` son parámetros de salida (creados por la función llamada) y `ref` son parámetros por referencia (creados por la función llamante y modificados por la función llamada). Tanto `out` como `ref` pueden usarse tanto con tipos con valor como con tipos con referencia. Si te preguntas qué necesidad hay de usar `ref` o `out` en tipos por referencia (que ya se pasan por referencia sin necesidad de usar esos modificadores) es que recuerda que con `ref` y `out` puedo modificar la referencia y que apunte a otro objeto distinto.

Por otro lado `in` declara un parámetro de entrada que se pasa por referencia, pero **no es modificable**. Bueno... ojo. Porque `in` se puede usar en tipos por valor y tipos por referencia:

* En tipos por valor, usar `in` hace que dicho tipo se pase por referencia, pero el compilador no nos dejará modificar el objeto
* En tipos por referencia, usar `in` hace que dicho tipo se pase por referencia, pero el compilado NOS dejará modificar el objeto, pero no asignar la referencia a otro valor.

Personalmente, no veo que aporta `in` a tipos por referencia (diría que nada).






