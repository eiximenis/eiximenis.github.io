---
title: '[Depuración] Generación programática de mini dumps'

author: eiximenis

date: 2008-11-11T11:37:58+00:00
geeks_url: /?p=1426
geeks_visits:
  - 1491
geeks_ms_views:
  - 1046
categories:
  - development

---
Bueno… No voy a hablar sobre como fue el DevCamp de este pasado fin de semana en puesto que ya lo han hecho [Jordi Nuñez][1] y [José Manuel Alarcon][2]. Compartimos buenos momentos, buenos cubatas y buenas charlas. 

De todas las charlas que se dieron, me interesa comentar especialmente la que dio [Pablo Alvarez][3], una charla **impresionante** bajo el título de [depurando hasta la saciedad][4]. Aunque contó con apenas 45 minutos destripó a base de bien WinDbg y comentó los distintos escenarios de depuración, especialmente la depuración post-mortem (esto es, cuando el proceso ya ha reventado). Leeros su post y el ppt porque es imprescindible. 

<!--more-->

¿Y qué quiero comentar yo de la charla de Pablo? Pues bien, en la charla quedó claro que antes de meterse a depurar uno debe tener unos buenos… símbolos. Y no sólo eso sino que evidentemente es necesario tener el dump de la aplicación que ha reventado. Comentó distintas herramientas para generar este dump, pero a mí me interesa comentar un escenario que suele ser muy habitual cuando ponemos aplicaciones en producción: que sea la propia aplicación quien genere este dump cuando se produce un error. En un proyecto donde estoy trabajando tenemos nuestra aplicación desplegada en distintos clientes, y aunque simplemente con el log de excepciones managed cubrimos la mayoría de errores, en algunos casos necesitamos más información. Es ahí donde tener un dump de la aplicación nos es realmente útil. 

Bueno… vamos a ver dos maneras de generar un dump para depuración post-mortem. La fácil y la no tan fácil. 

## La fácil

Os vais a la página de [ClrDump][5] y os lo descargáis. ClrDump es básicamente una DLL que podéis llamar desde vuestras aplicaciones .NET para generar dumps. En la propia página encontrareis la información de los métodos y sus firmas para P/Invoke. 

P.ej. el siguiente código genera un dump completo cuando se produzca cualquier excepción (managed o nativa) no capturada: 

```cs
SetFilterOptions(CLRDMP_OPT_CALLDEFAULTHANDLER);
string minidumpFile = string.Format(@”c:temppostmortem-{0}.dmp”, DateTime.Now.ToString(“dd-MM-yyyy”));
RegisterFilter(minidumpFile, MiniDumpWithFullMemory);
```

Las constantes y las firmas P/Invoke serian: 

```cs
[DllImport(“clrdump.dll”, CharSet = CharSet.Unicode, SetLastError = true)]
private static extern int RegisterFilter(string FileName, int DumpType);
[DllImport(“clrdump.dll”)]
static extern Int32 SetFilterOptions(int Options);
private const int CLRDMP_OPT_CALLDEFAULTHANDLER = 0x1;
private const int MiniDumpWithFullMemory = 0x2;
```

Cuando vuestra aplicación reviente tendréis vuestro minidump (en este caso será bastante grande puesto que volcará toda la memoria del proceso lo que puede dar lugar con facilidad a dumps de más de 100 ó 200 MB). 

## La no tan fácil 

Bueno… ClrDump funciona realmente bien y no se me ocurre ninguna razón para no usarlo, excepto que os guste investigar un poco como funciona la generación de dumps… Si queréis podéis implementaros vuestra propia librería para generar dumps. Para ello nada mejor que C++, la librería dbghelp.dll y para adelante! 

Dbghelp.dll es la librería que contiene, entre otras, las funciones de generación de dumps. Aunque Windows tiene una, lo mejor es usar la que viene con las [Debugging Tools for Windows][6]. Dado que es una librería unmanaged os recomiendo el uso de C++ (supongo que puede invocarse via p/invoke pero ni me lo he planteado, algunas funciones tienen firmas un poco complejas). 

La función clave se llama [MiniDumpWriteDump][7] y si veis su información en la msdn podréis comprobar que tiene bastantes parámetros. Esta función es la que hace todo el trabajo de generar un minidump, ahora sólo nos queda saber cuándo llamarla: cuando se produzca cualquier excepción (managed o nativa) no controlada. Para ello debemos usar el método [SetUnhandledExceptionFilter][8]. Este método espera un puntero a una función que será la que deba ejecutarse cuando se produzca cualquier excepción no controlada. Es en este método cuando llamaremos a MiniDumpWriteDump. 

El método SetUnhandledExceptionFilter espera un puntero a una función que reciba un LPEXCEPTION_POINTERS, eso es un puntero con los datos de la excepción que ha ocurrido. Esos datos se los debermos pasar a MiniDumpWriteDump para que pueda procesar la información. 

A modo de ejemplo, he creado una clase DumpHelper, en C++ CLI que permite generar minidumps cuando se produzca una excepción no controlada. La cabecera sería así: 

```cpp
namespace DumpHelper {
    delegate LONG UnhandledExceptionFilterHandler(LPEXCEPTION_POINTERS pException);
    public ref class DumpHelper
    {
        private:
            UnhandledExceptionFilterHandler^ pManagedHandler;
            LPTOP_LEVEL_EXCEPTION_FILTER pFilter;
            LONG UnhandledExceptionFilter(LPEXCEPTION_POINTERS pException);
            void CreateMiniDump( EXCEPTION_POINTERS* pep );
            String^ name;
        public:
            BOOL SetExceptionHandler ();
            DumpHelper(String^ name);
    };
}
```
 

Defino un delegate (que contendrá el puntero a función) y simplemente dos funciones públicas: el constructor que espera una cadena (con el nombre de fichero a generar) y el método SetExceptionHandler para activar la generación de minidumps cuando el proceso reviente. 

En la parte privada, tenemos la instancia del delegate (pManagedHandler), el puntero a función (pFilter), y dos funciones adicionales: UnhandledExceptionFilter y CreateMiniDump. 

La implementación del método SetExceptionHandler es tal como sigue: 

```cpp
BOOL DumpHelper::SetExceptionHandler()
{
    pManagedHandler = gcnew UnhandledExceptionFilterHandler(this, &DumpHelper::UnhandledExceptionFilter);
    pFilter = reinterpret_cast(Marshal::GetFunctionPointerForDelegate (pManagedHandler).ToPointer());
    SetUnhandledExceptionFilter(pFilter);
    return TRUE;
}
```

Creamos el puntero pFilter para que apunte a la función UnhandledExceptionFilter del propio objeto DumpHelper. Para ello tenemos que hacerlo usando un delegate y conviertiendo luego el delegate a un puntero nativo usando la clase Marshal. Una vez tenemos el puntero, lo pasamos como parámetro al método de Windows SetUnhandledExceptionFilter. En este punto cuando se produzca una excepción no controlada en nuestro programa se nos llamará al método UnhandledExceptionFilter de nuestra clase DumpHelper. Este método es muy simplemente y simplemente llama a CreateMiniDump: 

```cpp
LONG DumpHelper::UnhandledExceptionFilter(LPEXCEPTION_POINTERS pException)
{
    CreateMiniDump(pException);            
    // 0: Que se llame al manejador por defecto de Windows al finalizar
    return 0;
}
```

Finalmente el método CreateMiniDump es el que realiza todo el trabajo. Para ello llama a MiniDumpWriteDump: 

```cpp
void DumpHelper::CreateMiniDump(EXCEPTION_POINTERS* pep)
{
pin_ptr<const wchar_t> fname = PtrToStringChars(name);
    HANDLE hFile = CreateFile(fname, GENERIC_READ | GENERIC_WRITE,
        FILE_SHARE_READ, NULL, CREATE_ALWAYS, FILE_ATTRIBUTE_NORMAL, NULL);        
    if( ( hFile != NULL ) && ( hFile != INVALID_HANDLE_VALUE ) )
    {    
        MINIDUMP_EXCEPTION_INFORMATION mdei;
        mdei.ThreadId = GetCurrentThreadId();
        mdei.ExceptionPointers = pep;
        mdei.ClientPointers = FALSE;
        MINIDUMP_TYPE mdt = (MINIDUMP_TYPE)(MiniDumpWithFullMemory |
                                                 MiniDumpWithFullMemoryInfo |
                                                 MiniDumpWithHandleData |
                                                 MiniDumpWithThreadInfo |
                                                 MiniDumpWithUnloadedModules );
        BOOL rv = MiniDumpWriteDump(GetCurrentProcess(), GetCurrentProcessId(),
            hFile, mdt, (pep != NULL) ? &mdei : NULL , NULL, NULL );
        CloseHandle( hFile );
    }
}
```

La función simplemente abre un fichero (usando [CreateFile][9]) y luego rellena los distintos parámetros de MiniDumpWriteDump para generar el minidump. 

El uso de esta librería es muy simple. En vuestra aplicación simplemente añadís una referencia a ella, creáis un objeto DumpHelper y llamáis acto seguido a SetExceptionHandler. El mejor sitio para hacer esto es en vuestro método Main: 

```cs
class Program
{
  static void Main(string[] args)
  {
    DumpHelper h = new DumpHelper(“C:\temp\test.dmp”);
    h.SetExceptionHandler();
    // Resto de código…
  }
}
```

Listos! Con esto vuestras aplicaciones ya generarán dumps cuando revienten! 

Un saludo… y nos vamos viendo!

 [1]: https://mx1.raona.com/owa/http:/geeks.ms/blogs/jnunez/archive/2008/11/10/devcamp-volviendo-al-pasado-y-viendo-el-futuro.aspx
 [2]: http://geeks.ms/blogs/jalarcon/archive/2008/11/10/guadalajara-charlas-t-233-cnicas-y-gymcana-este-fin-de-semana.aspx
 [3]: http://geeks.ms/blogs/palvarez/default.aspx
 [4]: http://geeks.ms/blogs/palvarez/archive/2008/11/09/weekend-warrior-depurando-hasta-la-saciedad-en-el-devcamp.aspx
 [5]: http://www.debuginfo.com/tools/clrdump.html
 [6]: http://www.microsoft.com/whdc/devtools/debugging/default.mspx
 [7]: http://msdn.microsoft.com/en-us/library/ms680360(VS.85).aspx
 [8]: http://msdn.microsoft.com/en-us/library/ms680634.aspx
 [9]: http://msdn.microsoft.com/en-us/library/aa363858(VS.85).aspx