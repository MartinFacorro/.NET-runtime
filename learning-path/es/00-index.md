# Ruta de Aprendizaje del Runtime de .NET — Índice General

> 🌐 [English version](../en/00-index.md)

---

## Cómo Usar Esta Ruta

Esta ruta de aprendizaje está diseñada para llevarte desde tu primer contacto con .NET hasta contribuir patches al repositorio `dotnet/runtime`. Es **source-code-first**: cada concepto está anclado a un archivo real que podés abrir, leer y depurar. El código fuente del runtime es tu libro de texto.

La ruta está organizada en **5 niveles progresivos**. Cada nivel asume que internalizaste el material del anterior. Sin embargo, no tenés que seguir el camino linealmente — si ya tenés experiencia con .NET, usá la autoevaluación de abajo para encontrar tu nivel de entrada, y después elegí los tracks temáticos que te importen.

Cada módulo es un documento Markdown autocontenido con objetivos de aprendizaje, mapa conceptual, lecciones secuenciales, guía curada de lectura de código fuente, ejercicios prácticos y preguntas de autoevaluación. Podés completarlos a tu ritmo. Los tiempos estimados asumen que estás leyendo código fuente activamente y haciendo los ejercicios — no solo ojeando el texto.

---

## Autoevaluación: Encontrá Tu Nivel

Respondé estas preguntas con honestidad para determinar dónde empezar:

### ¿Podés responder SÍ a todas estas?
- [ ] Sé qué son el CLR, la BCL y el SDK, y cómo se relacionan entre sí
- [ ] Puedo crear una app de consola con `dotnet new` y explicar qué hacen los archivos generados
- [ ] Entiendo value types vs reference types y puedo explicar cuándo cada uno se aloca en el stack vs el heap

**Si NO → Empezá en [Nivel 1 — Fundamentos](#nivel-1--fundamentos)**

### ¿Podés responder SÍ a todas estas?
- [ ] Uso dependency injection, middleware y async/await todos los días
- [ ] Entiendo generics, covarianza/contravarianza y el patrón async basado en `Task`
- [ ] Puedo explicar qué hace `IDisposable` y por qué existe el patrón dispose
- [ ] He usado el CLI de `dotnet` para compilar, testear y publicar

**Si NO → Empezá en [Nivel 2 — Practicante](#nivel-2--practicante)**

### ¿Podés responder SÍ a todas estas?
- [ ] He usado `Span<T>` y `Memory<T>` para optimizar hot paths
- [ ] Puedo perfilar una aplicación con `dotnet-trace` o `dotnet-counters` e interpretar los resultados
- [ ] Entiendo las generaciones del GC, el LOH, y puedo diagnosticar memory leaks
- [ ] He leído código fuente de la BCL en GitHub para entender un comportamiento del framework

**Si NO → Empezá en [Nivel 3 — Avanzado](#nivel-3--avanzado)**

### ¿Podés responder SÍ a todas estas?
- [ ] Puedo explicar tiered compilation del JIT y cómo afecta el startup vs rendimiento en estado estable
- [ ] Entiendo `MethodTable`, `EEClass` y cómo el runtime representa tipos
- [ ] He usado `DOTNET_JitDisasm` o `DOTNET_JitDump` para inspeccionar la salida del JIT
- [ ] Puedo navegar `src/coreclr/vm/` y `src/coreclr/jit/` con fluidez razonable

**Si NO → Empezá en [Nivel 4 — Internals](#nivel-4--internals)**

### Si respondiste SÍ a todo lo anterior:
**→ Estás listo para [Nivel 5 — Experto / Contribuidor](#nivel-5--experto--contribuidor)**

---

## Vista General de la Ruta

La ruta de aprendizaje cubre **8 tracks temáticos** que atraviesan los 5 niveles. Cada track se profundiza progresivamente:

| Track | Alcance | Directorios clave en el código |
|---|---|---|
| **A — Fundamentos del Runtime** | Arquitectura del CLR, hosting, startup, sistema de tipos | `src/coreclr/vm/`, `src/native/corehost/` |
| **B — Gestión de Memoria** | GC, alocaciones, Span, Memory, pooling | `src/coreclr/gc/`, `src/libraries/System.Memory/` |
| **C — Motor de Ejecución** | JIT, tiered compilation, intérprete, R2R | `src/coreclr/jit/`, `src/coreclr/nativeaot/` |
| **D — Threading y Async** | Thread pool, Tasks, async/await, channels | `src/libraries/System.Threading*/`, `src/coreclr/vm/threadpool*` |
| **E — Colecciones y LINQ** | Estructuras de datos, internals de LINQ, rendimiento | `src/libraries/System.Collections*/`, `src/libraries/System.Linq/` |
| **F — Networking e I/O** | Sockets, HTTP, pipelines, streams | `src/libraries/System.Net*/`, `src/libraries/System.IO*/` |
| **G — Serialización y Texto** | JSON, XML, regex, encoding, globalización | `src/libraries/System.Text.Json/`, `src/libraries/System.Text.RegularExpressions/` |
| **H — Diagnósticos y Observabilidad** | EventPipe, tracing, logging, counters | `src/libraries/System.Diagnostics*/`, `src/coreclr/debug/` |

---

## Nivel 1 — Fundamentos

> 🎯 *"Soy nuevo en .NET o vengo de otro ecosistema"*

El objetivo del Nivel 1 es construir un modelo mental sólido de qué es .NET, cómo encajan sus piezas y cómo el código va desde C# fuente hasta un proceso en ejecución. Nada de internals del runtime todavía — solo una base clara y precisa.

| Módulo | Tema | Esfuerzo est. | Pregunta clave que responde |
|---|---|---|---|
| 1.1 | [Panorama del Ecosistema .NET](01-foundations-ecosystem-overview.md) | 2h | ¿Qué son el runtime, el SDK y la BCL, y cómo se relacionan? |
| 1.2 | [Estructura de Proyectos y el Sistema de Build](01-foundations-project-structure.md) | 3h | ¿Qué pasa cuando ejecuto `dotnet build`? ¿Qué son todos estos archivos? |
| 1.3 | [El Sistema de Tipos: Valores, Referencias y el Heap](01-foundations-type-system.md) | 4h | ¿Dónde viven mis objetos en memoria y por qué importa? |
| 1.4 | [Flujo de Control, Excepciones y el Call Stack](01-foundations-control-flow.md) | 3h | ¿Qué pasa realmente cuando se lanza una excepción? |
| 1.5 | [Assemblies, Namespaces y el Loader](01-foundations-assemblies.md) | 3h | ¿Cómo encuentra y carga el runtime mi código? |
| 1.6 | [I/O Básico: Archivos, Consola y Streams](01-foundations-basic-io.md) | 3h | ¿Cómo hace `Console.WriteLine` para escribir en la terminal? |
| 1.7 | [Tu Primera Lectura del Código Fuente del Runtime](01-foundations-first-source-reading.md) | 2h | ¿Cómo está organizado el repo `dotnet/runtime` y por dónde empiezo a leer? |

**Después del Nivel 1, podés:** Construir aplicaciones .NET con confianza, entender el pipeline de compilación de C# a IL a código nativo, y navegar la estructura del repositorio.

---

## Nivel 2 — Practicante

> 🎯 *"Construyo apps .NET todos los días pero no sé qué pasa por debajo"*

El Nivel 2 toma los conceptos que usás todos los días — generics, async/await, DI, colecciones — y te muestra cómo funcionan realmente. Vas a empezar a leer código fuente de la BCL regularmente.

| Módulo | Tema | Esfuerzo est. | Pregunta clave que responde |
|---|---|---|---|
| 2.1 | [Generics: De la Sintaxis a la Especialización en Runtime](02-practitioner-generics.md) | 4h | ¿Por qué `List<int>` puede ser más rápido que `ArrayList`? ¿Cómo maneja el runtime los tipos genéricos? |
| 2.2 | [Colecciones en Profundidad](02-practitioner-collections.md) | 5h | ¿Qué estructura de datos debo elegir y cuáles son las características de rendimiento reales? |
| 2.3 | [Async/Await y la Maquinaria de Tasks](02-practitioner-async-await.md) | 6h | ¿Qué genera el compilador para métodos `async`? ¿Cómo funciona el scheduling de `Task`? |
| 2.4 | [Dependency Injection: Del Patrón al Framework](02-practitioner-dependency-injection.md) | 4h | ¿Cómo resuelve servicios `IServiceProvider`? ¿Cuáles son las implicaciones de los lifetimes? |
| 2.5 | [LINQ: De la Sintaxis de Queries a las Máquinas de Iteradores](02-practitioner-linq.md) | 4h | ¿Por qué LINQ es lazy? ¿Qué genera `Select` realmente? |
| 2.6 | [Manejo de Strings y Procesamiento de Texto](02-practitioner-strings.md) | 3h | ¿Por qué los strings son inmutables? ¿Cuándo usar `StringBuilder` vs `string.Create`? |
| 2.7 | [Fundamentos de Networking: HttpClient y Sockets](02-practitioner-networking.md) | 5h | ¿Cómo gestiona `HttpClient` las conexiones? ¿Por qué debo reutilizarlo? |
| 2.8 | [Serialización: Internals de System.Text.Json](02-practitioner-json.md) | 4h | ¿Cómo descubre propiedades el serializador JSON? ¿Qué hace más rápidos a los source generators? |
| 2.9 | [El Contrato de IDisposable y la Gestión de Recursos](02-practitioner-disposable.md) | 3h | ¿Qué pasa si me olvido de hacer dispose? ¿Cómo funciona realmente la finalización? |
| 2.10 | [Configuración, Options y el Modelo de Hosting](02-practitioner-hosting.md) | 4h | ¿Cómo conecta todo `Host.CreateDefaultBuilder`? |

**Después del Nivel 2, podés:** Leer código fuente de la BCL para entender comportamientos del framework, tomar decisiones informadas sobre colecciones y patrones async, y depurar problemas comunes con lifetimes de DI y gestión de recursos.

---

## Nivel 3 — Avanzado

> 🎯 *"Optimizo, depuro problemas profundos y leo código fuente del framework a veces"*

El Nivel 3 es donde pasás de usar .NET a entender sus características de rendimiento y capacidades de diagnóstico. Vas a aprender a leer perfiles de rendimiento, optimizar uso de memoria y entender los patrones de middleware/pipeline que sustentan el framework.

| Módulo | Tema | Esfuerzo est. | Pregunta clave que responde |
|---|---|---|---|
| 3.1 | [Modelo de Memoria: Stack, Heap, Span y Memory](03-advanced-memory-model.md) | 6h | ¿Cómo evitan alocaciones `Span<T>` y `Memory<T>`? ¿Cuándo uso cada uno? |
| 3.2 | [Garbage Collection: Generaciones, Modos y Tuning](03-advanced-gc.md) | 6h | ¿Cómo elijo entre workstation/server GC? ¿Qué dispara las colecciones Gen2? |
| 3.3 | [Colecciones de Alto Rendimiento y Pooling](03-advanced-collections-perf.md) | 5h | ¿Cuándo usar `ArrayPool`, `ObjectPool`, `FrozenDictionary`? |
| 3.4 | [Primitivas de Threading y Sincronización](03-advanced-threading.md) | 6h | ¿Cuál es la diferencia entre `lock`, `SemaphoreSlim`, `SpinLock`? ¿Cuándo importa cada uno? |
| 3.5 | [System.IO.Pipelines e I/O de Alto Rendimiento](03-advanced-pipelines.md) | 5h | ¿Cómo evita copias de buffer Pipelines? ¿Cómo lo usa Kestrel? |
| 3.6 | [Expresiones Regulares: El Motor Compilado](03-advanced-regex.md) | 4h | ¿Cómo funciona `RegexOptions.Compiled`? ¿Qué cambió con el source generator en .NET 7+? |
| 3.7 | [Reflection, Emit y Source Generators](03-advanced-reflection.md) | 6h | ¿Cómo funciona reflection a nivel del runtime? ¿Por qué los source generators son más rápidos? |
| 3.8 | [Criptografía y Primitivas de Seguridad](03-advanced-cryptography.md) | 4h | ¿Cómo envuelve .NET la crypto de la plataforma (CNG, OpenSSL)? ¿Cómo se gestionan las claves? |
| 3.9 | [Diagnósticos: dotnet-trace, counters y EventPipe](03-advanced-diagnostics.md) | 5h | ¿Cómo perfilo una app en producción? ¿Qué eventos emite el runtime? |
| 3.10 | [Interop Nativo: P/Invoke y LibraryImport](03-advanced-native-interop.md) | 5h | ¿Cómo funciona el marshalling? ¿Por qué `LibraryImport` es mejor que `DllImport`? |

**Después del Nivel 3, podés:** Optimizar hot paths con `Span<T>` y pooling, diagnosticar memory leaks y presión del GC, perfilar aplicaciones con herramientas de diagnóstico del runtime, y escribir código de I/O de alto rendimiento.

---

## Nivel 4 — Internals

> 🎯 *"Entiendo la mecánica del CLR, el comportamiento del JIT y el tuning del GC"*

El Nivel 4 te lleva adentro del runtime. Vas a leer código C++ en `src/coreclr/vm/` y `src/coreclr/jit/`, entender cómo se disponen los tipos en memoria, cómo el JIT compila métodos y cómo el GC gestiona el heap a nivel de regiones.

| Módulo | Tema | Esfuerzo est. | Pregunta clave que responde |
|---|---|---|---|
| 4.1 | [Arranque del CLR: De `dotnet` a `Main()`](04-internals-clr-startup.md) | 8h | ¿Qué pasa entre escribir `dotnet run` y que se ejecute tu `Main()`? |
| 4.2 | [El Sistema de Tipos: MethodTable, EEClass y TypeDesc](04-internals-type-system.md) | 10h | ¿Cómo representa el runtime `class`, `struct`, `interface` y generics en memoria? |
| 4.3 | [Compilación JIT: Internals de RyuJIT](04-internals-jit.md) | 12h | ¿Cómo convierte RyuJIT el IL en código nativo? ¿Cuáles son las fases de optimización? |
| 4.4 | [Tiered Compilation y Dynamic PGO](04-internals-tiered-compilation.md) | 6h | ¿Cómo decide el runtime cuándo re-compilar un método? ¿Cómo fluyen los datos de PGO? |
| 4.5 | [Internals del GC: Regiones, Fase de Marcado y Write Barriers](04-internals-gc-deep.md) | 12h | ¿Cómo difiere el GC basado en regiones (.NET 8) de los segmentos? ¿Cómo funcionan los write barriers? |
| 4.6 | [Carga de Assemblies: Binder, ALC y Fusion](04-internals-assembly-loading.md) | 6h | ¿Cómo funciona `AssemblyLoadContext`? ¿Cuál es el pipeline del custom binder? |
| 4.7 | [Internals del Thread Pool y Work Stealing](04-internals-threadpool.md) | 6h | ¿Cómo decide el thread pool gestionado cuántos threads usar? |
| 4.8 | [Maquinaria de Manejo de Excepciones](04-internals-exceptions.md) | 6h | ¿Cómo se implementan las excepciones a nivel nativo? ¿Cuál es la diferencia SEH/PAL? |
| 4.9 | [ReadyToRun (R2R) y Crossgen2](04-internals-r2r.md) | 6h | ¿Cómo funciona la compilación ahead-of-time? ¿Cuáles son los tradeoffs vs JIT? |
| 4.10 | [Compilación NativeAOT](04-internals-nativeaot.md) | 8h | ¿Cómo produce NativeAOT un binario nativo autocontenido? ¿Qué se recorta? |

**Después del Nivel 4, podés:** Navegar `src/coreclr/vm/` y `src/coreclr/jit/` con confianza, entender la salida del JIT y sus decisiones de optimización, hacer tuning del GC para cargas de producción con razonamiento informado, y diagnosticar problemas a nivel del runtime con SOS/LLDB.

---

## Nivel 5 — Experto / Contribuidor

> 🎯 *"Puedo depurar el runtime mismo, contribuir patches y extender la VM"*

El Nivel 5 te prepara para contribuir a `dotnet/runtime`. Vas a compilar el runtime desde el código fuente, escribir y ejecutar tests del runtime, entender el pipeline de CI, y aprender los patrones usados en el codebase para contribuir cambios.

| Módulo | Tema | Esfuerzo est. | Pregunta clave que responde |
|---|---|---|---|
| 5.1 | [Compilar el Runtime desde el Código Fuente](05-expert-building.md) | 8h | ¿Cómo compilo CoreCLR, Mono y las libraries desde el código fuente en mi máquina? |
| 5.2 | [La Infraestructura de Tests del Runtime](05-expert-testing.md) | 6h | ¿Cómo están organizados los tests del runtime? ¿Cómo escribo y ejecuto un test nuevo? |
| 5.3 | [Contribuir un Cambio al JIT](05-expert-jit-contribution.md) | 12h | ¿Cómo agrego una nueva optimización al JIT? ¿Cuál es el proceso de review del PR? |
| 5.4 | [Contribuir un Cambio al GC](05-expert-gc-contribution.md) | 12h | ¿Cómo modifico el comportamiento del GC de forma segura? ¿Cómo valido con GC stress tests? |
| 5.5 | [Agregar una Nueva API a la BCL](05-expert-new-api.md) | 8h | ¿Cuál es el proceso de API review? ¿Cómo actualizo ref assemblies, implemento y testeo? |
| 5.6 | [El Runtime Mono: Arquitectura y Diferencias](05-expert-mono.md) | 10h | ¿En qué difiere Mono de CoreCLR? ¿Cuándo se usa cada uno? |
| 5.7 | [WebAssembly y el Runtime en el Browser](05-expert-wasm.md) | 8h | ¿Cómo corre .NET en el browser? ¿Cuál es el pipeline de build para wasm? |
| 5.8 | [Depurar el Runtime con SOS, LLDB y WinDbg](05-expert-debugging.md) | 8h | ¿Cómo adjunto un depurador nativo al CLR e inspecciono objetos gestionados? |
| 5.9 | [EventPipe y el Servidor de Diagnósticos](05-expert-eventpipe.md) | 6h | ¿Cómo funciona el protocolo de diagnóstico out-of-process? ¿Cómo agrego un nuevo evento? |
| 5.10 | [Hosting Personalizado y la API de Hosting Nativo](05-expert-hosting.md) | 6h | ¿Cómo embebo el CLR en una aplicación nativa? ¿Qué hace `hostfxr`? |

**Después del Nivel 5, podés:** Compilar y testear el runtime desde el código fuente, enviar pull requests a `dotnet/runtime`, depurar crashes nativos en el CLR, y diseñar soluciones que aprovechan un conocimiento profundo del comportamiento del runtime.

---

## Orden de Lectura Recomendado

Si vas a recorrer toda la ruta, este es el orden sugerido que balancea amplitud con profundidad:

```
Nivel 1: 1.1 → 1.2 → 1.3 → 1.4 → 1.5 → 1.6 → 1.7
Nivel 2: 2.1 → 2.3 → 2.2 → 2.5 → 2.9 → 2.4 → 2.6 → 2.7 → 2.8 → 2.10
Nivel 3: 3.1 → 3.2 → 3.4 → 3.3 → 3.9 → 3.5 → 3.10 → 3.7 → 3.6 → 3.8
Nivel 4: 4.1 → 4.2 → 4.3 → 4.4 → 4.5 → 4.6 → 4.7 → 4.8 → 4.9 → 4.10
Nivel 5: 5.1 → 5.2 → 5.8 → 5.5 → 5.3 → 5.4 → 5.6 → 5.7 → 5.9 → 5.10
```

El orden dentro de cada nivel prioriza conceptos fundamentales del runtime antes que temas específicos de libraries, y asegura que las habilidades de diagnóstico se introduzcan lo suficientemente temprano para soportar módulos posteriores.

---

## Rutas Clave del Repositorio

Referencia rápida para navegar `dotnet/runtime`:

| Ruta | Contenido |
|---|---|
| `src/coreclr/vm/` | Máquina virtual del CLR — sistema de tipos, class loader, threads, interop |
| `src/coreclr/jit/` | RyuJIT — el compilador just-in-time |
| `src/coreclr/gc/` | Implementación del garbage collector |
| `src/coreclr/debug/` | Infraestructura de depuración gestionada |
| `src/coreclr/interpreter/` | Intérprete del CLR (nuevo, alternativa al JIT para algunos escenarios) |
| `src/coreclr/nativeaot/` | Compilador y runtime de NativeAOT |
| `src/coreclr/System.Private.CoreLib/` | Parte de la base class library específica de CoreCLR |
| `src/mono/` | Runtime Mono (mobile, WASM, escenarios embebidos) |
| `src/libraries/` | Todas las libraries gestionadas (BCL) |
| `src/libraries/System.Private.CoreLib/` | Parte compartida (agnóstica al runtime) de CoreLib |
| `src/native/corehost/` | Host nativo (ejecutable `dotnet`, `hostfxr`, `hostpolicy`) |
| `docs/design/` | Documentos de diseño que explican features del runtime |
| `docs/workflow/` | Documentación de workflows de build y test |
| `docs/coding-guidelines/` | Convenciones de código y guías de contribución |

---

## Herramientas Esenciales por Nivel

| Nivel | Herramientas que debés conocer |
|---|---|
| 1 | CLI de `dotnet`, Visual Studio / VS Code, depurador de C# |
| 2 | NuGet, `dotnet test`, frameworks de testing unitario (xUnit), logging básico |
| 3 | `dotnet-counters`, `dotnet-trace`, `dotnet-dump`, BenchmarkDotNet, variables de entorno `DOTNET_*` |
| 4 | Extensión del depurador SOS, `DOTNET_JitDisasm`, `DOTNET_GCLog`, PerfView, ETW/EventPipe |
| 5 | LLDB/WinDbg + SOS, `dotnet-dsrouter`, compilar desde el código fuente, `CORE_ROOT`, infra de tests del runtime |

---

## Referencias

| Recurso | Tipo | Cobertura |
|---|---|---|
| [.NET Runtime Design Docs](https://github.com/dotnet/runtime/tree/main/docs/design) | Documentos de diseño | Fundamentos de diseño oficiales para features del runtime |
| [Book of the Runtime (BotR)](https://github.com/dotnet/runtime/tree/main/docs/design/coreclr/botr) | Documentación profunda | Internals del CLR — sistema de tipos, GC, JIT, threading |
| [Pro .NET Memory Management — Konrad Kokosa](https://prodotnetmemory.com/) | Libro | Guía definitiva de internals del GC y memoria |
| [Writing High-Performance .NET Code — Ben Watson](https://www.writinghighperf.net/) | Libro | Patrones prácticos de optimización de rendimiento |
| [Blog de Adam Sitnik](https://adamsitnik.com/) | Blog | Span, BenchmarkDotNet, hardware intrinsics |
| [Stephen Toub — Performance Improvements in .NET (serie anual)](https://devblogs.microsoft.com/dotnet/) | Posts de blog | Changelog detallado de rendimiento anual con links al código fuente |
| [Maoni Stephens — .NET GC Blog](https://maoni0.medium.com/) | Blog | Internals del GC de la arquitecta del GC |
| [Andy Ayers — Posts sobre el Compilador JIT](https://devblogs.microsoft.com/dotnet/) | Posts de blog | JIT tiering, PGO, decisiones de optimización |
| [.NET Source Browser](https://source.dot.net/) | Herramienta | Vista indexada y buscable del código fuente del runtime |
| [SharpLab](https://sharplab.io/) | Herramienta | Ver IL, ASM del JIT y C# lowered para cualquier snippet |

---

*Esta ruta de aprendizaje es un documento vivo. A medida que el runtime evolucione, los módulos se actualizarán para reflejar nuevas features y cambios arquitectónicos.*

*Última actualización: 2026-04-14*
