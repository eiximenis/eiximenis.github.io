---
title: TxF – NTFS Transaccional
description: TxF – NTFS Transaccional
author: eiximenis

date: 2009-11-25T17:14:25+00:00
geeks_url: /?p=1478
geeks_visits:
  - 1222
geeks_ms_views:
  - 964
categories:
  - Uncategorized

---
Una capacidad de la que no se habla mucho es de TxF, que apareció junto con Vista: es la capacidad de tener transacciones sobre ficheros NTFS. Esas transacciones pueden afectar a uno _o a varios_ ficheros… y no solo eso: gracias al poder de DTS podemos coordinar una transaccion TxF con otros tipos de transacciones como SQL Server o MSMQ!

Como pasa con muchas de las características avanzadas de windows, sólo se puede usar en .NET a través de p/invoke (si obviamos C++/CLI claro)… vamos a ver un ejemplo!

Primero creamos una clase que contenga todas las definiciones de los métodos Win32 que vamos a usar:

<div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
  <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px" id="codeSnippet"><span style="color: #0000ff">public</span> <span style="color: #0000ff">static</span> <span style="color: #0000ff">class</span> NativeMethods<br />{<br />    <span style="color: #0000ff">public</span> <span style="color: #0000ff">const</span> <span style="color: #0000ff">uint</span> GENERIC_READ = 0x80000000;<br />    <span style="color: #0000ff">public</span> <span style="color: #0000ff">const</span> <span style="color: #0000ff">uint</span> GENERIC_WRITE = 0x40000000;<br />    [DllImport(<span style="color: #006080">"kernel32.dll"</span>, SetLastError = <span style="color: #0000ff">true</span>)]<br />    <span style="color: #0000ff">public</span> <span style="color: #0000ff">static</span> <span style="color: #0000ff">extern</span> SafeFileHandle CreateFileTransacted(<span style="color: #0000ff">string</span> lpFileName,<br />           <span style="color: #0000ff">uint</span> dwDesiredAccess, <span style="color: #0000ff">uint</span> dwShareMode, IntPtr lpSecurityAttributes, <span style="color: #0000ff">uint</span> dwCreationDisposition,<br />           <span style="color: #0000ff">uint</span> dwFlagsAndAttributes, IntPtr hTemplateFile, IntPtr hTransaction, IntPtr pusMiniVersion,<br />           IntPtr pExtendedParameter);<br /><br />    [DllImport(<span style="color: #006080">"ktmw32.dll"</span>, SetLastError = <span style="color: #0000ff">true</span>)]<br />    <span style="color: #0000ff">public</span> <span style="color: #0000ff">static</span> <span style="color: #0000ff">extern</span> IntPtr CreateTransaction(IntPtr lpTransactionAttributes, IntPtr uow,<br />        <span style="color: #0000ff">uint</span> createOptions, <span style="color: #0000ff">uint</span> isolationLevel, <span style="color: #0000ff">uint</span> isolationFlags, <span style="color: #0000ff">uint</span> timeout, <span style="color: #0000ff">string</span> description);<br /><br />    [DllImport(<span style="color: #006080">"ktmw32.dll"</span>, SetLastError = <span style="color: #0000ff">true</span>)]<br />    <span style="color: #0000ff">public</span> <span style="color: #0000ff">static</span> <span style="color: #0000ff">extern</span> <span style="color: #0000ff">bool</span> CommitTransaction(IntPtr transaction);<br /><br />    [DllImport(<span style="color: #006080">"ktmw32.dll"</span>, SetLastError = <span style="color: #0000ff">true</span>)]<br />    <span style="color: #0000ff">public</span> <span style="color: #0000ff">static</span> <span style="color: #0000ff">extern</span> <span style="color: #0000ff">bool</span> RollbackTransaction(IntPtr transaction);<br /><br />    [DllImport(<span style="color: #006080">"Kernel32.dll"</span>)]<br />    <span style="color: #0000ff">public</span> <span style="color: #0000ff">static</span> <span style="color: #0000ff">extern</span> <span style="color: #0000ff">bool</span> CloseHandle(IntPtr handle);<br />}<br /></pre>
  
  <p>
    </div> 
    
    <p>
      Vamos a usar los siguientes métodos del api de Win32:
    </p>
    
    <ul>
      <li>
        <a href="http://msdn.microsoft.com/en-us/library/aa363859(VS.85).aspx">CreateFileTransacted</a>: Crea o abre un fichero para ser usado de forma transaccional. Una vez obtenido el handle, <strong>el resto de funciones win32</strong> que trabajan con handles funcionan transaccionalmente con el nuevo handle.
      </li>
      <li>
        <a href="http://msdn.microsoft.com/en-us/library/aa366011(VS.85).aspx">CreateTransaction</a>: Crea la transaccion
      </li>
      <li>
        <a href="http://msdn.microsoft.com/en-us/library/aa366001(VS.85).aspx">CommitTransaction</a>: Hace el commit de la transaccion&#160;
      </li>
      <li>
        <a href="http://msdn.microsoft.com/en-us/library/aa366366(VS.85).aspx">RollbackTransaction</a>: Hace el rollback de la transacción
      </li>
      <li>
        <a href="http://msdn.microsoft.com/en-us/library/ms724211(VS.85).aspx">CloseHandle</a>: Cierra un handle cualquiera (sea de fichero o no y sea transaccional o no).
      </li>
    </ul>
    
    <p>
      Vamos a hacer un programilla que abra dos ficheros en modo transaccional, escriba algo en ellos y luego haga commit y/o rollback de la transacción para ver sus efectos…
    </p>
    
    <p>
      Primero creamos la transacción y abrimos dos ficheros (suponemos que los nombres nos los pasarán por línea de comandos):
    </p>
    
    <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
      <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px" id="codeSnippet"><span style="color: #008000">// 1. Crear la transacción</span><br />IntPtr transaction = NativeMethods.CreateTransaction(IntPtr.Zero, IntPtr.Zero, 0, 0, 0, 0, <span style="color: #0000ff">null</span>);<br /><span style="color: #008000">// 2. Abrir dos ficheros de forma transaccional</span><br />SafeFileHandle sfh1 = NativeMethods.CreateFileTransacted(args[0], NativeMethods.GENERIC_READ | NativeMethods.GENERIC_WRITE,<br />    0, IntPtr.Zero, (<span style="color: #0000ff">uint</span>)FileMode.CreateNew, 0, IntPtr.Zero, transaction, IntPtr.Zero, IntPtr.Zero);<br />SafeFileHandle sfh2 = NativeMethods.CreateFileTransacted(args[1], NativeMethods.GENERIC_READ | NativeMethods.GENERIC_WRITE,<br />    0, IntPtr.Zero, (<span style="color: #0000ff">uint</span>)FileMode.CreateNew, 0, IntPtr.Zero, transaction, IntPtr.Zero, IntPtr.Zero);<br /></pre>
      
      <p>
        </div> 
        
        <p>
          Una vez tenemos los handles cualquier función Win32 estándard para escribir o leer en ellos nos serviría… por suerte los file streams de .NET <em>pueden</em> ser creados a partir de un handle de Win32, así que podremos utilizar la clase FileStream.
        </p>
        
        <p>
          Ahora ya sólo nos queda hacer un commit o un rollback… Para ello si se produce cualquier error durante la escritura haremos un rollback, y en caso contrario haremos un commit. Finalmente <strong>debemos cerrar la transacción</strong> y por ello usaremos CloseHandle.
        </p>
        
        <p>
          El código es tal y como sigue:
        </p>
        
        <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
          <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px" id="codeSnippet"><span style="color: #0000ff">try</span><br />{<br />    <span style="color: #008000">// Escribimos algo en ambos ficheros</span><br />    <span style="color: #0000ff">using</span> (FileStream f1 = <span style="color: #0000ff">new</span> FileStream(sfh1, FileAccess.ReadWrite))<br />    <span style="color: #0000ff">using</span> (FileStream f2 = <span style="color: #0000ff">new</span> FileStream(sfh2, FileAccess.ReadWrite))<br />    <span style="color: #0000ff">using</span> (StreamWriter sw1 = <span style="color: #0000ff">new</span> StreamWriter(f1))<br />    <span style="color: #0000ff">using</span> (StreamWriter sw2 = <span style="color: #0000ff">new</span> StreamWriter(f2))<br />    {<br />        sw1.Write(<span style="color: #006080">"Fichero 1"</span>);<br />        <span style="color: #0000ff">if</span> (args.Length &gt; 2 && args[2] == <span style="color: #006080">"/e"</span>)<br />            <span style="color: #0000ff">throw</span> <span style="color: #0000ff">new</span> IOException(<span style="color: #006080">"Error cualquiera"</span>);<br />        sw2.Write(<span style="color: #006080">"Fichero 2"</span>);<br />    }<br /><br />    <span style="color: #008000">// Lanzamos el commit. Si todo ha ido bien llegaremos aquí</span><br />    NativeMethods.CommitTransaction(transaction);<br />}<br /><span style="color: #0000ff">catch</span> (IOException ex)<br />{<br />    <span style="color: #008000">// Algun error: lanzamos el rollback</span><br />    Console.WriteLine(<span style="color: #006080">"Error en el acceso a ficheros:"</span> + ex.Message);<br />    NativeMethods.RollbackTransaction(transaction);<br />}<br /><span style="color: #0000ff">finally</span><br />{<br />    <span style="color: #008000">// Cerramos la transacción TxF</span><br />    NativeMethods.CloseHandle(transaction);<br />}</pre>
          
          <p>
            </div> 
            
            <p>
              Como se puede observar la escritura en los ficheros se hace a través de las clases estándard de .NET. Es solo la apertura del fichero que debe hacerse a través de la API de Win32, así como la creación de la transacción.
            </p>
            
            <p>
              Si el tercer parámetro que se le pasa al programa es /e el programa simulará un error.
            </p>
            
            <p>
              Si ejecutais el programa <font face="cour"><em>TxFDemo.exe f1.txt f2.txt</em></font> vereis que os aparecen los dos ficheros.
            </p>
            
            <p>
              Por otro lado si ejecutais <em>TxFDemo f1.txt f2.txt /e</em> vereis que NO os aparece ninguno de los dos ficheros, ya que se lanza la excepción y con ella se hace un rollback de la transacción NTFS.
            </p>
            
            <p>
              Ahora imaginad las posibilidades que este sistema ofrece!
            </p>
            
            <p>
              Eso sí recordad que es para Windows Vista o superior…
            </p>
            
            <p>
              Link: <a href="http://msdn.microsoft.com/es-es/magazine/cc163388.aspx">Un artículo sobre TxF bastante interesante de Jason Olson.</a>
            </p>
            
            <p>
              Saludos!
            </p>