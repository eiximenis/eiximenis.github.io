---
title: Listas "seguras" en JavaScript
author: eiximenis

date: 2016-06-08T10:00:59+00:00
categories:
  - js
---

Que las clases en ECMASCript 2015 son algo muy diferente a las clases en otros lenguajes como C# o Java, es algo que ya deberíamos tener muy claro. El propio lenguaje funciona de forma muy distinta (dinámico, basado en prototipos, no existen las clases en _runtime_,...), pero podemos hacer cosas igualmente interesantes.

Una necesidad que podríamos tener es la de tener una clase que nos permita solo tener objetos de una determinada clase. Algo que en Java o C# podríamos conseguir fácilmente con genéricos, pero que en JavaScript parece más complicado, ¿verdad?
La realidad es que **no es muy difícil conseguir en JavaScript limitar los elementos de una colección a un tipo (clase) determinado**. Vamos a ver como.

Primero nos creamos dos clases distintas, para poder probarlo:

```js
class Dog {
    constructor(n) {
        this.name = 'dog ' + n;
    }
}
class Cat {
    constructor(n) {
        this.name = 'cat ' + n;
    }
}
```

Ahora creamos la clase `List` que usaremos para tener elementos de un solo tipo. Podríamos tenerla declarada en un módulo de ES6:

```js
const _privates = new WeakMap();
export default class List {
    constructor(c) {
        let pr = {c, items:[]};
        _privates.set(this, pr);
    }
    
    add (item) {
        let pr = _privates.get(this);
        if (!(item instanceof pr.c)) {
            throw new Error ("Bad type item");
        }
        pr.items.push(item);
    }
    
    *[Symbol.iterator] () {
        return yield* _privates.get(this).items;
    }
}
```

¡Listos! Ya podemos probar nuestra clase:

```js
let l = new List(Dog);
l.add(new Dog("bobby"));
l.add(new Dog("milu"));
// iteramos
for (var d of l) {
    console.log(d.name);
}
l.add(new Cat("azrael"));   // Error: Bad item
```

El código de la classe `List` es muy sencillo. Básicamente, **en el constructor aceptamos una clase**. Recuerda que, realmente, las clases son funciones en tiempo de ejecución. Por lo tanto, podemos pasarlas (y devolverlas) como parámetro. Luego en el método `add` simplemente verificamos que el objeto pasado es un objeto de la clase que nos hemos guardado (a través del operador `instanceof`) y si lo es, lo añadimos al array de elementos.
Y ya está, ya hemos creado una colección que es segura en cuanto al tipo de sus elementos. 
Tampoco era tan difícil, ¿verdad? ;-)
