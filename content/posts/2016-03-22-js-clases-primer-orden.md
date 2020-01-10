---
title: Clases como ciudadanos de primer orden en JS

author: eiximenis

date: 2016-03-22T10:00:59+00:00
categories:
  - js
---

Una de las novedades de ES2015 es el concepto de clases. Las clases en JavaScript nada tienen que ver con el concepto de "clase" de otros lenguajes como C++, Java o C#. En JavaScript **las clases son una sintaxis alternativa para definir funciones constructoras**. Pero, lo que seguimos teniendo por debajo es la herencia entre objetos, basada en prototipos.
Un punto importante, y que se suele pasar por alto, es que dado que una clase al final termina definiendo una función, y las funciones son ciudadanos de primer orden, también deben serlo las clases. O dicho de otro modo: es posible pasar una clase como parámetro y devolver una clase como valor de retorno.

**Nota:** Dado que los navegadores todavía no soportan el concepto de clase, si quieres experimentar con él, vas a necesitar un transpilador. Para ir rápido puedes usar el [Babel online](https://babeljs.io/repl/), que traduce código ES21015 a ES5 (compatible con los navegadores actuales) sin necesidad de instalarte nada.

Empecemos por el siguiente código, para comprobar que efectivamente las clases son funciones. Ese código imprime "function" por la consola. Nada de "class" o "Animal". Esos conceptos no existen en tiempo de ejecución.

```javascript
class Animal {
  constructor(name) {
    this.name = name;
  }
}

console.log(typeof(Animal))
```

Ahora, como podemos devolver una clase desde una función? Pues nada más fácil:

```javascript
class Animal {
  constructor(name) {
    this.name = name;
  }
}

class Perro extends Animal {
  ladra() {console.log(`${this.name} guaaaaau`)}
}

class Gato extends Animal {
  maulla() {console.log(`${this.name} miaaaaaau`)}
}

let animalFactory = function(type) {
  if (type == "perro") {
    return Perro;
  }
  return Gato;
}

const perroClass = animalFactory("perro");
let milu = new perroClass("milú");
milu.ladra();
```

Observa como `perroClass` no es un objeto, si no que es la propia clase `Perro` (de ahí que podamos hacer un new luego).

Pasar clases a clases
---------------------
Del mismo modo que podemos pasar o devolver una clase a una función, también podemos pasar una clase... a una clase. Eso puede parecer raro, pero da una especie de contrapartida a los genéricos y a los templates de lenguajes como C#, Java o C++.
Observa el siguiente código:

```javascript
let withArea = (base) => class  extends base {
  get area() {
    return this.radius * 2 * 3.14159;
  }
}

class Circle {
  constructor (radius) {
    this.radius = radius;
  }
}

var cwa = new (withArea(Circle))(10.0);
console.log(cwa.area);
```

La función `withArea` recibe una clase como parámetro (`base`) y devuelve una clase (anónima). Esa clase devuelta por `withArea` es una clase que **extiende la clase pasada como parámetro y añade una propiedad de solo lectura `area`**.
Posteriormente creamos una clase `Circle` que tan solo tiene una propiedad `radius` y finalmente. Finalmente llamamos a `withArea(Circle)` lo que nos devuelve una clase que estiende `Circle` con la propiedad `area` añadida. Luego con `new` creamos un objeto de esa clase y consultamos el valor de la propiedad `area`.
Observa como todo funciona correctamente y al final obtenemos el valor esperado. Es importante tener presente que en este ejemplo hemos:

1. Creado una función que recibe una clase y
2.    Crea una clase anónima que extiende la clase pasada y
3.    Devuelve esta clase anónima

Como puedes ver las clases en ES6 encierran más de lo que parece a simple vista. Pero la clave es tener presente que, dado que una clase es realmente una función, la podemos pasar como parámetro o devolverla desde otras funciones :)

**Nota:** Y sí... Efectivamente la función `withArea` actúa como un mixin (es decir agrega comportamiento a una clase).

Un saludo!!
