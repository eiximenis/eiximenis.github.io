---
title: Cambiar el nombre de un cliente de un Workspace de TFS

author: eiximenis

date: 2009-12-18T09:02:00+00:00
geeks_url: /?p=1485
geeks_visits:
  - 1415
geeks_ms_views:
  - 1082
categories:
  - Uncategorized

---
Saludos! Un post cortito, cortito, cortito 🙂

Si renombramos una máquina cliente de TFS, vemos que perdemos los mappings ya que el workspace está asociado a un usuario + nombre de máquina.

<!--more-->

Aunque podemos crearnos un workspace nuevo y borrar el antiguo también podemos _modificar_ el workspace antiguo y cambiar el nombre de máquina, aunque para ello deberemos usar la herramienta de línea de comandos tf.exe:

<div id="codeSnippetWrapper" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; width: 97.5%; font-family: 'Courier New', courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; cursor: text; border: silver 1px solid; padding: 4px;">
  <pre id="codeSnippet" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 0em; width: 100%; font-family: 'Courier New', courier, monospace; direction: ltr; color: black; font-size: 8pt; overflow: visible; border-style: none; padding: 0px;">tf workspaces /owner:TuUsuarioDeTFS /updateComputerName:NombreAntiguoDeLaMaquina /s:URLDelTFS</pre>
  
  <p>
    </div> 
    
    <p>
      (El parámetro /owner: no se si es estrictamente necesario, yo lo he metido por si acaso).
    </p>
    
    <p>
      Sí, sí... cambiar el nombre de máquina no es muy habitual, pero si se os ocurre hacerlo como yo, con n check-ins pendientes... Pues se agradece la opción 🙂
    </p>
    
    <p>
      Sacado de <a href="http://blogs.msdn.com/buckh/archive/2006/03/03/update-workspace.aspx" title="http://blogs.msdn.com/buckh/archive/2006/03/03/update-workspace.aspx">http://blogs.msdn.com/buckh/archive/2006/03/03/update-workspace.aspx</a> donde además dicen como modificar el workspace si lo que cambia es tu nombre de usuario.
    </p>
    
    <p>
      Saludos!
    </p>
    
    <p>
      PD: Esta vez ha sido cortito de veras, eh??? 😉
    </p>