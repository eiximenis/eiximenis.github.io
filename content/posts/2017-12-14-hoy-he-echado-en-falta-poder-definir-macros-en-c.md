---
title: 'Hoy he echado en falta poder definir macros en C#'
author: eiximenis

date: 2017-12-14T17:43:12+00:00
geeks_url: /?p=1933
geeks_ms_views:
  - 1508
categories:
  - 'C#'

---
Pues sí... y no me avergüenzo de decirlo, si hoy C# tuviese algo como el preprocesador de C/C++ me hubiese hecho feliz.
  
<!--more-->


  
Si lees el [por qué C# no soporta macros][1], tendrás unas cuantas razones por las cuales es **una buena idea no usar macros** (macros al estilo  #define del preprocesador):

  * Legibilidad: Es obvio que si el código fuente que se compila **es distinto** que el código fuente que se edita eso lo hace más ininteligible, porque debes &#8220;inferir&#8221; esas macros para poder entender el código fuente.
  * No respetan las reglas del lenguaje: Las macros tratan el código como texto, no como código y por lo tanto no respetan las reglas del lenguaje, tales como  ámbitos.
  * Son una guarrada: Pues sí, en general lo son. Pero bueno, tampoco nos vamos a poner quisquillosos... hay otras cosas en C# que son una guarrada también y nos callamos.

Pero **a pesar de todos esos inconvenientes, a pesar de que debemos evitar usarlas** a veces nos pueden sacar de un apuro... precisamente porque tratan el código como texto, no como código 😛
  
He aquí un ejemplo concreto. Tengo una función definida tal y como sigue:

<pre class="EnlighterJSRAW" data-enlighter-language="csharp">public static class Check
{
    public static void NotNull&lt;TParameter&gt;(TParameter parameter, string name=null)
        where TParameter : class
    {
        if (parameter == null)
        {
            throw new ArgumentNullException(name ?? nameof(parameter));
        }
    }
}</pre>

Vale, ahora quiero que la siguiente línea de código:

<pre class="EnlighterJSRAW" data-enlighter-language="csharp">Check.NotNull(foo);</pre>

Lance una ArgumentNullException, pero **quiero que el argumento que se pasa a la excepción sea la cadena &#8220;foo&#8221;** (que es el parámetro que estamos validando).
  
Yo no sé vosotros, pero a mi no se me ocurre manera de hacerlo. Fijaos que la función &#8220;NotNull&#8221; lo intenta todo lo bien que sabe: usa nameof(parameter), pero claro nameof(parameter) vale... &#8220;parameter&#8221;. Esta función siempre lanza una ArgumentNullException de &#8220;parameter&#8221;, ya que el nombre de la variable &#8220;foo&#8221; se pierde totalmente. Es algo **totalmente normal**.
  
Por supuesto, si hubiese tenido macros, podría haber hecho algo como lo siguiente (ejemplo en C++):

<pre class="EnlighterJSRAW" data-enlighter-language="cpp">#define NN(X) NotNull(X, "X")
template &lt;typename T&gt; void NotNull(T const& v, std::string const& arg)
{
  if (v == nullptr) {
    throw std::invalid_argument(arg);
  }
}
int main()
{
  int *a = nullptr;
  NN(a);   // catapum con una std::invalid_argument("a")
}</pre>

**PD: **No estoy pidiendo que se añadan macros en C#, solo que mira... hoy las he echado en falta y he decidido compartirlo con vosotros 😉

 [1]: https://blogs.msdn.microsoft.com/csharpfaq/2004/03/09/why-doesnt-c-support-define-macros/