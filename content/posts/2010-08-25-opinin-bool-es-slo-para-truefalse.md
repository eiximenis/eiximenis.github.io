---
title: 'Opinión: bool es sólo para true/false'
author: eiximenis

date: 2010-08-25T13:37:29+00:00
geeks_url: /?p=1531
geeks_visits:
  - 2033
geeks_ms_views:
  - 1666
categories:
  - Uncategorized

---
Saludos a todos! Tanto a los que estéis trabajando, cómo aquellos que estando de vacaciones seais tan frikis que leais geeks.ms! 🙂

Hoy quiero hablar un poco sobre _bool_. Puede parecer un tipo de datos aburridote: a fin de cuentas sólo puede tener dos valores, pero precisamente ahí radica su gracia y de eso os quería contar. La idea del post es muy simple: _bool es sólo para true/false_.

Por ejemplo, en los arcanos tiempos en que un servidor usaba Visual C++ 6 para el desarrollo de aplicaciones windows, en las MFCs había un método muy divertido llamado [UpdateData][1] (que por lo que veo aún está). MFC tenía una cosa muy buena que era la posibilidad de realizar _bindings_ entre variables de la clase que representaba la ventana (usualmente una clase derivada de CWnd) y los controles que contenía dicha ventana. Eso, ahora, puede parecer una chorrada pero por aquel entonces era una auténtica pasada.

El método UpdateData era el que se encargaba de realizar dicho binding. Se llamaba con un parámetro BOOL que si valía TRUE significaba que se pasaban los datos de los controles a las variables y si valía FALSE pues era al revés: se pasaban los datos de las variables a los controles. Eso lo acabo de leer ahora en la MSDN, pero cuanda usaba MFC lo tenía que leer a menudo: nunca me acordaba cual era el significado de TRUE y FALSE en este contexto… En el fondo el parámetro de dicha función no es true/false es “BindingDeControlesAVariables” o “BindingDeVariablesAControles”, es decir un enum con dos opciones.

Y a eso me refiero en este post: un enum con dos opciones no es un booleano, aunque en ambos casos tengamos sólo dos valores posibles. Establecer arbitrariamente un valor a _true_ y otro a _false_ sólo hace que la gente se confunda.

Que código creéis que es más legible?

<div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
  <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px" id="codeSnippet">Fichero.Abrir(<span style="color: #006080">"foo.txt"</span>,<span style="color: #0000ff">true</span>);</pre>
  
  <p>
    </div> 
    
    <p>
      O bien:
    </p>
    
    <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
      <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px" id="codeSnippet">Fichero.Abrir(<span style="color: #006080">"foo.txt"</span>,ModoApertura.Lectura);</pre>
      
      <p>
        </div> 
        
        <p>
          En el primer caso, el segundo parámetro es un bool que si vale true se abre el fichero para lectura y si es false, pues se abre para escritura. Pues vale, pero es una decisión arbitraria y cuando alguien deba usar Abrir probablemente deba consultar que hace exactamente este parámetro bool (y lo mismo si alguien revisa código).
        </p>
        
        <p>
          Pero no sólo es por legibilidad de código… Podéis asegurar que dos opciones serán siempre dos opciones? Me explico, y esta vez, como nadie (ni mucho menos yo) está libre de pecado, con un ejemplo de cosecha propia.
        </p>
        
        <p>
          En un framework que estamos desarrollando para permitir desarrollos de aplicaciones con ciertas características hay una propiedad de una clase (llamésmola Aplicacion) que indica si cuando se va a la pantalla anterior (las aplicaciones tienen navegación parecida a la de un browser) se debe destruir la vista de la cual se proviene o bien dicha vista se mantiene en memoria.
        </p>
        
        <p>
          En su momento implementamos dicha propiedad con un bool (true = destruir la vista, false = no destruirla). Y estuvo bien… hasta cierto día.
        </p>
        
        <p>
          Cierto día, nos vimos en la necesidad de que en ciertas aplicaciones la vista se debía destruir sólo si <strong>no</strong> contenía datos (en caso contrario se podía reaprovechar). A priori esto dependía de cada aplicación. Y ahí tuvimos el problema: como mapeamos esto en nuestra propiedad booleana? Porque ahora teníamos tres valores:
        </p>
        
        <ol>
          <li>
            No destruir la vista
          </li>
          <li>
            Destruir la vista siempre
          </li>
          <li>
            Destruir la vista sólo si no tiene datos
          </li>
        </ol>
        
        <p>
          Nosotros solucionamos el problema añadiendo una segunda propiedad que fuese “que tipo de destrucción de vistas se quería”, y lo hicimos así, simplemente porque todas estas propiedades estaban serializadas en XML (en archivos que algunos ya estaban en producción), por lo que la propiedad original debía ser siendo bool. Total, ahora tenemos dos propiedades a establecer para un mismo comportamiento lo que es susceptible de errores y de confusión. Y todo por no haber usado en su momento un enum 🙂
        </p>
        
        <p>
          Así pues, un consejo: cuando creeis propiedades bool, aseguraos de que <em>realmente</em> un bool es lo que necesitais y no un enum (aunque sea con dos opciones!).
        </p>
        
        <p>
          Un saludo!!!!
        </p>

 [1]: http://msdn.microsoft.com/en-us/library/t9fb9hww(VS.80).aspx