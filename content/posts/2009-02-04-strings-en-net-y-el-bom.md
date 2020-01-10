---
title: Strings en .NET y el BOM

author: eiximenis

date: 2009-02-04T20:12:00+00:00
geeks_url: /?p=1436
geeks_visits:
  - 1594
geeks_ms_views:
  - 1121
categories:
  - Uncategorized

---
¿Conoceis el BOM? Los que no, teneis suerte... los que sí, seguro que lo habeis sufrido... 🙂 Para los que no, contaros que el BOM, o Byte Order Mask que es lo que significan sus siglas, no es nada más que una marca (de entre 2 y 3 bytes) al principio de un archivo Unicode que indica el formato de los datos... si están en little endian o big endian p.ej.

<!--more-->

Quereis verlo en acción? Abrid el bloc de notas y teclead cualquier palabra, como p.ej. [Ag&uuml;ero][http://es.wikipedia.org/wiki/Sergio_Ag%C3%BCero] (algún fan del atleti por aqui???). Ahora haced un &ldquo;guardar como&rdquo; y marcad la opción &ldquo;Unicode big endian&rdquo; en codificación.

Ahora si miramos el tamaño del archivo, vereis que ocupa 14 bytes... Las cuentas no salen: Ag&uuml;ero tiene 6 letras, a 2 bytes la letra Unicode... sobran 2 bytes. El BOM. ¿Queréis más pruebas? Haced un type del archivo desde una consola. Vereis algo como:

```
■&nbsp; A g &sup3; e r o
```

Este &ldquo;cuadradito negro&rdquo; que aparece al principio es el BOM. Que pasa si os lo cargais??? Si abris el fichero con un editor hexadecimal (como el mismo Visual Studio) vereis algo como:

```
FE FF 00 41 00 67 00 FC 00 65 00 72 00 6F&nbsp;&nbsp; ...A.g...e.r.o
```

Los dos primeros bytes (FE FF) son el BOM... borradlos para que vuestro archivo quede tal como:

```
00 41 00 67 00 FC 00 65 00 72 00 6F&nbsp;&nbsp; .A.g...e.r.o
``` 

Lo guardais de nuevo y lo abrís con el bloc de notas... y esto es lo que vereis:

```
A g &uuml; e r o
```

Sin BOM el bloc de notas identifica este archivo de texto como ANSI en lugar de Unicode, e interpreta el byte 00 de cada carácter Unicode como un carácter ANSI adicional.

¿Divertido, eh? Pues no os digo nada cuando uno se encuentra que según el protocolo o producto que use el BOM puede ser opcional, obligatorio o hasta prohibido...

¿Y porque os cuento todo esto? Pues porque me he encontrado con un comportamiento curioso (no digo ni que esté mal ni que esté bien) con las strings de .NET y el BOM. Tengo el siguiente código:

```cs
class Program
{
    static void Main(string[] args)
    {
        string foo = (char)0xfeff + "Foo";
    }
}
```

Fácil, eh? Creo una string y le añado el BOM al principio... Y ahora viene lo curioso:

  1. foo.Length devuelve 4 porque cuenta el BOM como un carácter más 
  2. foo[0] es un carácter con valor 0xfeff 
  3. foo[1] es un carácter con valor 0x0046 (&lsquo;F&#8217;) 
  4. foo.StartsWith(foo[0] + &#8220;&#8221;) devuelve true, indicando que la cadena empieza con el BOM 
  5. foo.StartsWith(foo[1] + &#8220;&#8221;) también devuelve true, indicando que la cadena empieza por &ldquo;F&rdquo; 
  6. foo.Equals(foo.Substring(1)) devuelve false, indicando que ambas cadenas son distintas 
  7. foo.CompareTo(foo.Substring(1)) devuelve 0, indicando que ambas cadenas son iguales 
  8. foo.Trim se carga el BOM (o sea foo.Trim().Length vale 3) 

En fin... parece ser que algunos métodos _conocen_ el BOM y lo ignoran y otros no y lo tratan como un caracter más...

¿Curioso, no?
