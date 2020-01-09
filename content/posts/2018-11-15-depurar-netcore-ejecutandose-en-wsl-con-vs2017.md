---
title: Depurar netcore ejecutándose en WSL con VS2017
description: Depurar netcore ejecutándose en WSL con VS2017
author: eiximenis

date: 2018-11-15T09:27:24+00:00
geeks_url: /?p=2211
geeks_ms_views:
  - 690
categories:
  - depuracion
  - visual studio

---
Hoy he tenido que afrontar la dolorosa situación de un código en netcore que funcionaba correctamente en mi máquina (Windows 10) pero que al ejecutarse en Docker fallaba miserablemente.
  
Dado que el error que imprimía por consola era que no encontraba un fichero, lo primero fue comprobar que nuestro código era _cross-platform_ (p. ej. que usábamos siempre Path.Combine y no concatenábamos el caráter `\` para separar directorios) ya que la imagen Docker era de Linux. Luego hicimos todas las verificaciones habidas y por haber (permisos, etc) y nada. Todo parecía correcto. Al final, nos preguntamos si era un problema de Docker o de cualquier Linux en general y ahí acudimos a WSL.
  
Ejecutamos el programa bajo WSL y obtuvimos el mismo error. Eso era bueno por dos motivos: No era Docker la fuente del error y además nos permitía depurar el proceso.
  
<!--more-->


  
Había pero un problema:  **teníamos clarísismo que el error NO estaba directamente en nuestro código**, si no que era lanzado por una librería externa. Afortunadamente la librería era de Microsoft (se trataba de ML.NET 0.7) y eso significa que podíamos usar la potencia de &#8220;Source link&#8221; de VS2017 para depurar el código fuente... sin necesidad de tenerlo.
  
**1. Habilitar Source Link**
  
Así, que esto fue lo primero: habilitar Source Link. Resumiendo Source Link permite a VS2017 descargarse el código fuente (de las librerías que lo soportan, claro) automáticamente. No es nada &#8220;nuevo&#8221;, versiones anteriores de VS ya lo permitían (bajo una opción llamada &#8220;Source Server&#8221;), pero Source Link solventa muchas de las limitaciones que Source Server tenia. Una de las ventajas de Source Link es que una de sus fuentes puede ser Github.
  
Para habilitar Source Link hay que modificar **tres settings** de VS2017. El primero es permitir depurar código que no sea propio. Para ello en Options->Debugging->General debes deshabilitar la opción &#8220;Enable Just My Code&#8221;:
  
[<img class="alignnone size-full wp-image-2212" src="https://geeks.ms/etomas/wp-content/uploads/sites/154/2018/11/mycode-disable.png" alt="Pantalla de opciones con MyCode deshabilitada" width="744" height="434" />][1]
  
Con esta opción habilitada, VS2017 no intenta depurar código que no sea el de nuestro proyecto (que es lo habitual, no deseas generalmente depurar un framework o librería externa).
  
La segunda opción a modificar se trata de habilitar &#8220;source link&#8221;. De nuevo está en Options->Debugging->General:
  
[<img class="alignnone size-full wp-image-2213" src="https://geeks.ms/etomas/wp-content/uploads/sites/154/2018/11/enable-source-link.png" alt="Opción source link habilitada" width="744" height="434" />][2]
  
Habilitar Source Link puede ser suficiente en el caso de nugets que tengan los PDBs incrustados, pero en muchos casos no es así. Para estos otros casos necesitamos dar de alta un servidor de símbolos, del cual VS2017 se pueda descargar los símbolos (PDBs) de las librerías que se van cargando. Microsoft ofrece un servidor de símbolos. Para habilitarlo solo debes ir a &#8220;Options->Debugging->Symbols&#8221; y marcar la casilla &#8220;Microsoft Symbol Servers&#8221;:
  
[<img class="alignnone size-full wp-image-2214" src="https://geeks.ms/etomas/wp-content/uploads/sites/154/2018/11/synbol-server.png" alt="Añadiendo el servidor de símbolos de MS" width="744" height="434" />][3]
  
Con esto tenemos VS2017 configurado para depurar código externo, descargándose los PDBs del servidor de símbolos y el código fuente. Estamos listos.
  
**2. Depurar contra WSL**
  
La depuración contra WSL se realiza a través de SSH. Lo único que debes tener presente es que no tengas un posible conflicto de puertos. Dado que WSL comparte espacio de puertos con el propio Windows 10, si tienes un servidor de SSH escuchando en Windows, el puerto 22 ya lo tendrás ocupado, así que entonces debes modificar el puerto del servidor SSH del WSL. **Yo para que me funcionase he tenido que hacer lo siguiente:**

  1. Desinstalar el servidor de SSH de WSL (sudo apt-get remove openssh-server)
  2. Reinstalarlo (sudo apt-get install openssh-server).
  3. Editar el fichero /etc/ssh/sshd_config y poner las entradas (si alguna no existe, la creas): 
      1. Port a 2200 (o el valor que prefieras)
      2. UsePrivilegeSeperation a &#8220;no&#8221; (sin las comillas)
      3. PasswordAuthentication a &#8220;yes&#8221; (sin las comillas)
  4. Reiniciar el servidor de ssh (sudo service ssh &#8211;full-restart)

Esos han sido mis pasos hasta que me ha funcionado (sin ellos VS2017 me daba error al conectarme via SSH).
  
El siguiente punto fue ejecutar el proyecto en WSL (con dotnet run). Mi proyecto era una Web Api y el error lo daba cuando recibía un POST, así que podía ponerlo en marcha y luego adjuntarme al proceso con VS2017. Para ello (con la sln cargada en VS), en Debug->Attach To Process, seleccionas SSH y en &#8220;Connection Target&#8221; entras un nombre descriptivo tipo &#8220;WSL&#8221;. Eso te lanzará el diálogo de conexión SSH:
  
[<img class="alignnone size-full wp-image-2215" src="https://geeks.ms/etomas/wp-content/uploads/sites/154/2018/11/connect-remote.png" alt="Diálogo &quot;connect to remote system&quot;" width="856" height="608" />][4]
  
Entras los datos correctos (en Host name es el nombre de tu máquina, Port el puerto SSH de WSL y por supuesto &#8220;User name&#8221; y &#8220;Password&#8221; es tu usuario y password de WSL).
  
Hecho esto, VS2017 se connectará con WSL (via SSH) y te mostrará los procesos. En mi caso he tenido que marcar &#8220;show process from all users&#8221; , ya que los procesos aparecían bajo un usuario llamado &#8220;eiximen+&#8221; (mi usuario de WSL es eiximenis). No sé a que se debe eso, pero no es problema alguno. Te adjuntas a un proceso y ya podrás empezar a depurar.
  
La sesión de depuración puede ser muy lenta, ya que VS2017 se debe descargar los símbolos de todas las librerías (así que ten paciencia). En la ventana de Output irás viendo mensajes del tipo:

<pre>Loaded '/home/eiximenis/dotnet/shared/Microsoft.AspNetCore.App/2.2.0-preview3-35497/Microsoft.AspNetCore.Diagnostics.Abstractions.dll'. Symbols loaded.
* Searching for 'System.ComponentModel.Annotations.pdb' on the configured symbol servers.
.</pre>

Ahora debes tener un breakpoint colocado en tu código fuente, y una vez se para, puedes ir usando F11 para entrar en los métodos. Si entras en un método que no es tuyo (si no del framework o de una librería), VS2017 se descargará el código fuente y podrás seguir depurando.
  
En mi caso, al final encontré la causa del error (observa que el código fuente cargado es [ImageLoaderTransform.cs][5] que forma parte de ML.NET (y que VS2017 se ha descargado):
  
[<img class="alignnone wp-image-2218 size-large" src="https://geeks.ms/etomas/wp-content/uploads/sites/154/2018/11/debug-session-mlnet-1024x474.png" alt="Sesión de depuración remota con el error" width="660" height="306" />][6]
  
Vaya, se trata de que no se encuentra una librería &#8220;libgdiplus&#8221;. ¡Hemos encontrado el error!
  
Por cierto que con esa info, encontré una [issue ya entrada][7] referente a este problema, pero no la hubiera encontrado si no hubiese sabido qué problema era 🙂
  
Eso me confirma dos cosas:

  1. A pesar de que cada vez uso menos VS2017, en algunos casos se convierte en un amigo irremplazable.
  2. **Tragarse todas las excepciones para terminar lanzando otra excepción sin información es una <span style="text-decoration: underline;">muy mala práctica</span>**.

Saludos!

 [1]: https://geeks.ms/etomas/wp-content/uploads/sites/154/2018/11/mycode-disable.png
 [2]: https://geeks.ms/etomas/wp-content/uploads/sites/154/2018/11/enable-source-link.png
 [3]: https://geeks.ms/etomas/wp-content/uploads/sites/154/2018/11/synbol-server.png
 [4]: https://geeks.ms/etomas/wp-content/uploads/sites/154/2018/11/connect-remote.png
 [5]: https://github.com/dotnet/machinelearning/blob/master/src/Microsoft.ML.ImageAnalytics/ImageLoaderTransform.cs
 [6]: https://geeks.ms/etomas/wp-content/uploads/sites/154/2018/11/debug-session-mlnet.png
 [7]: https://github.com/dotnet/machinelearning/issues/489