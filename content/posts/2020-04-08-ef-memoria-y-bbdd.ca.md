---
title: "EF memòria i BBDD"
author: eiximenis
description: Anem a repassar conceptes bàsics, però que et poden fer ballar el cap com no vigilis. Concretament, com assegurar-te de que totes les teves consultes LINQ amb EF s'executen realment a la BBBDD. Som-hi!
date: 2020-04-15T18:00:00
language: ca
categories:
  - ef
---

Si tens experiència amb Entity Framework, probablament aquesta entrada tampoc t'aportarà massa, però després de veure els mateixos errors en varis projectes, mira tu... m'he decidit a escriure-la. I va de com **assegurar-te de que totes les teves consultes LINQ (usant EF) s'executin a la BBDD**.

## Avaluació en client

Les primeres versions d'EF (fins la 3.0, aquesta ja exclosa) tenien la possibilitat de realitzar el que s'anomenava "avaluació en client". Però recordem primer el què fa EF: ha de traduir un **arbre d'expressió** a una sentència SQL. Un arbre d'expressió en C# és una instància de la classe `Expression<T>` a on el tipus `T` és un delegat. P. ex. `Expression<Func<int, bool>>` sería un arbre d'expressió. Aquests objectes tenen la particularitat que es poden avaluar en temps d'execució i justament això és el que fa EF per generar el SQL. Com a programadors nosaltres mai creem directament objectes `Expression<T>` si no que deixem que el compilador ho faci per nosaltres, a partir d'un delegat, que habitualment expresem en forma d'una expressió lambda. P. ex. el compilador pot convertir `x => x+1` a una `Expression<T>` a on `T` sigui compatible amb el delegat proporcionat com p. ex. `Expression<Func<int, int>>`:

```csharp
Expression<Func<int, int>> expr = x => x + 1;   // OK
``` 

Com ja s'ha dit, EF fa servir aquestes expressions per generar el SQL final i aquestes expressions es construeixen a partir dels delegats que passem quan fem servir els diferents mètodes de LINQ. Això planteja un problema: què passa si EF no sap generar el SQL d'una determinada expressió?

Anem a veure un exemple. Partirem de la següentt aplicació de terminal (netcore 3):

```csharp
        static void Main(string[] args)
        {
            Console.WriteLine("Creating DbContext...");
            var lf = LoggerFactory.Create(c => c
                .AddFilter("*", LogLevel.Debug)
                .AddConsole());
            var builder = new DbContextOptionsBuilder<FooContext>()
                .UseSqlServer("Data Source=(LocalDb)\\MSSQLLocalDB;Initial Catalog=foo;Integrated Security=SSPI")
                .UseLoggerFactory(lf);
            var ctx = new FooContext(builder.Options);
            ctx.Database.EnsureCreated();
            if (!ctx.Persons.Any())
            {
                ctx.Persons.Add(new Person() { Name = "Baby Monster", Age = 2 });
                ctx.Persons.Add(new Person() { Name = "Young Monster", Age = 14 });
                ctx.Persons.Add(new Person() { Name = "Adult Monster", Age = 30 });
                ctx.Persons.Add(new Person() { Name = "Senior Monster", Age = 70 });
                ctx.SaveChanges();
            }
            Console.WriteLine("'Simple query'");
            var adults = ctx.Persons.Where(p => p.Age > 17).ToList();
        }
```

La classe `FooContext` és tot just un `DbSet<Person>`:

```csharp
    public class FooContext : DbContext
    {
        public FooContext(DbContextOptions<FooContext> options) : base(options)
        {
        }
        public DbSet<Person> Persons { get; set; }
    }
    public class Person
    {
        public int Id { get; set; }
        public string Name { get; set; }
        public int Age { get; set; }
    }    
```

L'aplicació mostra pel terminal les consultes generades per EF, i aquest es el _log_ que podem veure:

```
Executing DbCommand [Parameters=[], CommandType='Text', CommandTimeout='30']
SELECT [p].[Id], [p].[Age], [p].[Name]
FROM [Persons] AS [p]
WHERE [p].[Age] > 17 
```

D'acord. EF ha generat la sentència SQL esperada a partir de la nostra expressió LINQ. Ara bé, imagina que decidim refactoritzar això i eliminar aquest `17` d'aquí i encapsular-lo en una funció:

```csharp
private static bool IsAdult(Person p) => p.Age > 17
```

Després, òbviament, modifiques el teu codi LINQ:

```csharp
var adults = ctx.Persons.Where(p => IsAdult(p)).ToList();
```

Aquest codi compila (a fi de comptes `p => IsAdult(p)` es un delegat perfectament vàlid i per tant es pot crear un arbre d'expressió). Però... funciona? Doncs bé, **si fas servir EF 2.x o anterior, aparentment sí**. L'aplicació s'executa i el resultat és el mateix. Però si et fixes en el _log_ d'EF:

```
Executing DbCommand [Parameters=[], CommandType='Text', CommandTimeout='30']
SELECT [p].[Id], [p].[Age], [p].[Name]
FROM [Persons] AS [p]
```

Ho veus? El WHERE ja no hi és! Fixa't que **EF s'està traient tots els registres de la taula**. Val, el què ha passat és el següent: Quan EF ha de convertir el codi LINQ a SQL, es troba una crida a `IsAdult(p)` i **no sap com traduir-ho a SQL**. Per tant, en aquest punt, deixa de generar SQL i continuarà l'avaluació en memòria. Això és el que passarà:

* S'executarà el resultat de traduir `Persons` (que és recuperar tots els registres de la taula)
* El resultat anirà a memòria (en un `IEnumerable<Person>`)
* En memòria s'executarà la resta de la consulta LINQ (el `Where`)

Aquesta característica (de continuar consultes en memòria) és precisament el que anomenem "avaluació en client".

Si la BBDD té pocs registres, això no ho notaràs (o quasi), però imagina una taula amb milions de regitres...

Honestament, **l'avaluació en client és una pèssima característica**. Penso que estava en les primeres versions d'EF, perquè aquestes primeres versions (sobretot EF 1.x) no era capaç de generar SQL per a alguns casos quasi trivials. Ara bé, un consell: **si fas servir EF 2.x desactiva-la**. Per això nomès has de generar el següent codi al crear el `DbContextOptionsBuilder`:

```csharp
ConfigureWarnings(w => w.Throw(RelationalEventId.QueryClientEvaluationWarning))
```

Amb aquest codi, cada vegada que EF no pugui traduir LINQ a SQL, es llançarà una excepció en comptes d'avaluar en client. I això és molt millor, creu-me, perque així te n'adones que aquesta consulta no es pot executar a BBDD. I millor adonar-te'n en la fase de proves que no pas perquè cau producció, no?

**A EF 3.x l'avaluació en client està desactivada ja de sèrie (el codi anterior funciona, senzillament no fa res)**, per tant ja rebries l'excepció sense necessitat d'afegir el codi anterior. Aquesta decisió és un _breaking change_, però crec que és una gran decisió. EF 3.x ja està molt més madur i és capaç de generar SQL en una gran varietat d'escenaris. No hi ha necessitat de tenir una bomba de rellotgeria activa com és l'avaluació en client.

## Obligar a l'avaluació en client

A vegades ens pot interessar obligar a l'avaluació en client, més que res perquè no hi ha manera humana d'escriure part de la consulta d'una forma que es pugui traduir a SQL. Podem forçar l'avaluació en client de part d'una query cridant a `AsEnumerable()`:

```csharp
var adults = ctx.Persons.Where(p => p.Age > 17).
    AsEnumerable().Where(p => p.Name.StartsWith("Se")).ToList();
```

Si ara observes el _log_ d'EF el que veuràs serà:

```
Executed DbCommand (2ms) [Parameters=[], CommandType='Text', CommandTimeout='30']
SELECT [p].[Id], [p].[Age], [p].[Name]
FROM [Persons] AS [p]
WHERE [p].[Age] > 17
```

Pots veure com **un cop hem cridat a `AsEnumerable()` passem a avaluar en client** i la verificació de que el nom comenci per `Se` es fa en client, no en la BBDD.

## Avaluar en client sense voler

Forçar l'avaluació en client està bé, el problema és quan ho fem sense voler. I com que l'hem forçada EF no ens avisarà de cap manera.

```csharp
var adults = ctx.Persons.Adults().ToList();
// ...
static class MyExtensions
{
    public static IEnumerable<Person> Adults (this IEnumerable<Person> source)
    {
        return source.Where(p => p.Age > 17);
    }
}
```

Com veus aquest codi? Tot bé no? EF no es queixa, l'aplicació segueix retornant els resultats correctes... però observa el _log_:

```
Executed DbCommand (1ms) [Parameters=[], CommandType='Text', CommandTimeout='30']
SELECT [p].[Id], [p].[Age], [p].[Name]
FROM [Persons] AS [p]
```

Al passar la condició a un mètodo d'extensió sobre `IEnumerable` s'ha forçat l'avaluació en client. A tots els efectes és com si invoquessis a `AsEnumerable()`. Per tant EF no t'avisarà perquè pressuposa que saps el què estàs fent. **Aquest error l'he vist en molts casos**.

Si vols encapsular consultes en mètodes separats, has de declarar-los sobre `IQueryable`, no sobre `IEnumerable`.

```csharp
public static IQueryable<Person> Adults (this IQueryable<Person> source)
{
    return source.Where(p => p.Age > 17);
}
```

Declarar el mètode sobre `IEnumerable` funciona perquè `IQueryable` hereta de `IEnumerable`. Però quan fem servir `IEnumerable` estem fent servir sempre avaluació en client. El per què passa això té a veure amb la combinació de dos aspectes de C#:

1. El _dispatch_ dels mètodes d'extensió es sempre en temps de compilació.
2. Els mètodes de LINQ de `IEnumerable` treballan amb delegats, no amb arbres d'expressió.

Comencem amb el segon punt. El mètode `Where` de `IEnumerable` està definit així:

```csharp
public static IEnumerable<TSource> Where<TSource>(this IEnumerable<TSource> source, Func<TSource, bool> predicate);
```

Per la seva banda el mètode `Where` de `IQueryable` està definit de forma diferent:

```csharp
public static IQueryable<TSource> Where<TSource>(this IQueryable<TSource> source, Expression<Func<TSource, bool>> predicate);
```

Efectivament, en el segon cas el paràmetre és un arbre d'expressió (`Expression`), i en el primer és simplement un delegat tal qual. El `Where` de `IEnumerable` no permet que EF analitzi res. No es pot transformar un delegat a SQL, perquè un delegat no es pot analitzar. Per això ens cal un arbre d'expressió.

I ara entra en joc el primer dels dos punts d'abans: els mètodes d'extensió es seleccionen en temps de compilació. Això el què vol dir és que si tenim:

```csharp
source.Where(p => p.Age > 17);
```

Quan el compilador ha de generar el codi per invocar al `Where` ho fa en **funció del tipus de la variable `source`**. Ho repeteixo perquè és la clau: en funció del tipus de la variable, no pas en funció del tipus real de l'objecte referenciat per la variable. No importa que l'objecte "real" sigui un `IQueryable`: si `source` és del tipus `IEnumerable`, s'invocarà al mètode d'extensió de `IEnumerable`. Perque aquesta decisió la pren el compilador (no el CLR) i el compilador nomès pot saber el tipus de la variable.

Per tant, compte en definir mètodes sobre `IEnumerable` per que és una manera de forçar l'avaluació en client.

Ara bé, **vull deixar clar que el problema és que s'acava invocant al `Where` de `IEnumerable`**, no pas que el nostre mètode d'extensió estigui definit sobre `IEnumerable` (malgrat les dos coses van molt relacionades). Per exemple, el següent mètode d'extensió `Adults()` està definit sobre `IEnumerable` però no força l'avaluació en client:

```csharp
public static IEnumerable<Person> Adults(this IEnumerable<Person> source)
{
    return source.CWhere(p => p.Age > 17);
}
public static IEnumerable<T> CWhere<T>(this IEnumerable<T> source, Expression<Func<T, bool>> predicate)
{
    if (source is IQueryable<T>)
    {
        return ((IQueryable<T>)source).Where(predicate);
    }
    return source.Where(predicate.Compile());
}
```

Aquí la clau és que `CWhere` analitza (en temps d'execució) si l'objecte implementa o no `IQueryable` i en funció del resultat invoca directament el mètode `Where` que toca (observa el cast a `IQueryable<T>`). Obviament, tot el que encadenessim després de `Adults()` s'avaluaria en client (perquè `Adults()` retorna un `IEnumerable<T>`).

## Avaluació mandrosa (lazy evaluation)

No s'ha de confondre l'avaluació en client amb l'avaluació mandrosa (_lazy_). L'avaluació mandrosa vol dir que **fins que no es recorrin els objectes de un `IEnumerable` no s'avaluarà aquest `IEnumerable`**. Això passa també amb els `IQueryable`, per tant tenim avaluació mandrosa tant a BBDD com en client. Es quelcom inherent a .NET, no ho podem desactivar.

Això vol dir que si tinc aquesta consulta LINQ:

```chsarp
var adults = ctx.Persons.Where(p => p.Age > 17);
```

La variable `adults` conté el resultat, **però no es generarà fins que es recorri aquest resultat**. Aquest recorregut pot ser amb un `foreach` o bé "materialitzant" el resultatt (p. ex. cridant a `.ToList()` per a copiar el resultat en una llista).

En aquest punt hi ha una diferència super importnat entre l'avaluació mandrosa en client i l'avaluació mandrosa en BBDD:

* L'avaluació mandrosa en BBDD es **ejecutar el SQL**
* L'avaluació mandrosa en client és **generar els elements del `IEnumerable` a mesura que es necessiten**

Què vull dir amb això? Doncs que si tens una consulta com la següent (a on `Adults()` està definit sobre `IEnumerable` i per tant ens força l'avaluació en client):

```csharp
Console.WriteLine("'Simple query'");
var adults = ctx.Persons.Adults();
Console.WriteLine("Iterating results");
foreach (var a in adults.Take(1))
{
    Console.WriteLine(a.Name);
}
```

El _log_ que ara veuràs serà semblant a:

```
'Simple query'
Iterating results
Executed DbCommand (1ms) [Parameters=[], CommandType='Text', CommandTimeout='30']
SELECT [p].[Id], [p].[Age], [p].[Name]
FROM [Persons] AS [p]
Adult Monster
```

Fixa't que **no s'executa la consulta SQL fins que no comencem a iterar**, però clar, aquesta consulta s'emporta tots els registres de la BBDD. És indiferent que després facis un `Take(1)`, aquest `Take` és en client. En aquest cas la situació es:

1. Comencem a iterar
2. S'executa UNA SOLA VEGADA la consulta SQL (i s'emporta, en aquest cas, tots els registres)
3. Es generen un a un tots els elements del `IEnumerable` (en aquest cas nomès n'hi ha un pel `Take`)

El punt positiu de l'avaluació mandrosa és que et permet realitzar les consultes LINQ quan vulguis, però no pagaràs el preu fins que les recorris o les materialitzis (cridant a `ToList()` o similar, pensa que `AsEnumerable()` no materialitza res).

Espero que aquesta entrada t'hagi ajudat a entendre com funciona l'avaluació en client a EF.

