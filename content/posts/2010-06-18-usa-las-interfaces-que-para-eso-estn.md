---
title: Usa las interfaces… que para eso están!
description: Usa las interfaces… que para eso están!
author: eiximenis

date: 2010-06-18T10:50:42+00:00
geeks_url: /?p=1519
geeks_visits:
  - 1357
geeks_ms_views:
  - 846
categories:
  - Uncategorized

---
Ole! Vaya título tan imperativo me ha salido, eh??? Un post cortito para comentar un problemilla que hemos tenido en casa de un cliente, y que al final era debido por no usar interfaces (en nuestro caso interfaces COM).

**El problemilla…**

En todos los ordenadores de desarrollo el sistema funcionaba perfectamente (como siempre… _en mi máquina funciona!_).

En los ordenadores de prueba el sistema daba error, concretamente se quejaba que no encontraba el ensamblado mshtml.dll. Nosotros usábamos una referencia contra el ensamblado PIA que está en “_%PROGRAMFILES%Microsoft.NETPrimary Interop Assemblies_” y que se llama _Microsoft.mshtml.dll_.

Vimos que efectivamente dicho ensamblado **no** se distribuía junto con nuestro proyecto y que no estaba en la GAC de los ordenadores, así que cambiamos la referencia para que fuese con _Copy Local=true_. Ahora sí que aparecia el ensamblado: Microsoft.mshtml.dll.

Desplegando esta versión de Microsoft.mshtml.dll el sistema… seguía sin funcionar. Pero ahora ya no daba errores de carga de assemblies ni de validaciones falladas de strong name, sino que daba una excepción:

_Unable to cast COM object of type &#8216;System.\_\_ComObject&#8217; to class type &#8216;mshtml.HTMLDocumentClass&#8217;. COM components that enter the CLR and do not support IProvideClassInfo or that do not have any interop assembly registered will be wrapped in the \_\_ComObject type. Instances of this type cannot be cast to any other class; however they can be cast to interfaces as long as the underlying COM component supports QueryInterface calls for the IID of the interface._

El código que daba el error era el siguiente:

<div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
  <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px" id="codeSnippet"><span style="color: #0000ff">var</span> mainDocument = (mshtml.HTMLDocumentClass)webBrowserControl.Document.DomDocument;</pre>
  
  <p>
    </div> 
    
    <p>
      <strong>La solución</strong>
    </p>
    
    <p>
      Bueno… Lo primero que pensamos era que alguna diferencia de versión de internet explorer podía causar eso, pero descartamos el tema: aunque es cierto que había distintas versiones (aunque todos los ordenadores tenían la 6(*), no todos tenían las mismas actualizaciones) había <strong>un </strong>ordenador de desarrollo que tenía la 8… Si compilábamos en dicho ordenador podíamos ejecutar el programa en <em>otros</em> ordenadores de desarrollo que tuvieran la 6. Estaba claro que interet explorer no era el problema.
    </p>
    
    <p>
      Entonces? Bueno… si habéis leído el título del post ya sabréis la causa: estábamos utilizando directamente las clases COM (P.ej. HTMLDocumentClass) en lugar de usar la interfaz (p.ej. IHTMLDocument2). Cambiando el código por el siguiente:
    </p>
    
    <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
      <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px" id="codeSnippet"><span style="color: #0000ff">var</span> mainDocument = (mshtml.IHTMLDocument2)webBrowserControl.Document.DomDocument;</pre>
      
      <p>
        </div> 
        
        <p>
          Todo funcionó a la perfección!
        </p>
        
        <p>
          ¿Conclusion? Pues eso… <strong>usad las interfaces (COM) que para eso están!</strong> 🙂
        </p>
        
        <p>
          Un saludo!!!!
        </p>
        
        <p>
          (*) Sí, sí, sí… Internet Explorer 6 ya no está suportado por MS, es más viejo que matusalén, tienes más agujeros de seguridad que un queso gruyere pero… en muchos sitios sigue formando parte de la plataforma corporativa.
        </p>