# .NET Runtime Learning Path — Master Index

> 🌐 [Versión en español](../es/00-index.md)

---

## How to Use This Path

This learning path is designed to take you from your first encounter with .NET all the way to contributing patches to the `dotnet/runtime` repository. It is **source-code-first**: every concept is anchored to a real file you can open, read, and debug. The runtime source is your textbook.

The path is organized into **5 progressive levels**. Each level assumes you've internalized the material from the previous one. However, you don't have to follow the path linearly — if you're already an experienced .NET developer, use the self-assessment below to find your starting level, then pick the topic tracks that matter to your work.

Each module is a self-contained Markdown document with learning objectives, a concept map, sequential lessons, a curated source-code reading guide, hands-on exercises, and self-assessment questions. You can complete them at your own pace. The estimated effort times assume you're actively reading source code and doing the exercises — not just skimming the text.

---

## Self-Assessment: Find Your Level

Answer these questions honestly to determine where to start:

### Can you answer YES to all of these?
- [ ] I know what the CLR, BCL, and SDK are and how they relate
- [ ] I can create a console app with `dotnet new` and explain what the generated files do
- [ ] I understand value types vs reference types and can explain when each is allocated on the stack vs heap

**If NO → Start at [Level 1 — Foundations](#level-1--foundations)**

### Can you answer YES to all of these?
- [ ] I use dependency injection, middleware, and async/await daily
- [ ] I understand generics, covariance/contravariance, and the `Task`-based async pattern
- [ ] I can explain what `IDisposable` does and why the dispose pattern exists
- [ ] I've used the `dotnet` CLI for building, testing, and publishing

**If NO → Start at [Level 2 — Practitioner](#level-2--practitioner)**

### Can you answer YES to all of these?
- [ ] I've used `Span<T>` and `Memory<T>` to optimize hot paths
- [ ] I can profile an application with `dotnet-trace` or `dotnet-counters` and interpret the results
- [ ] I understand GC generations, LOH, and can diagnose memory leaks
- [ ] I've read BCL source code on GitHub to understand a framework behavior

**If NO → Start at [Level 3 — Advanced](#level-3--advanced)**

### Can you answer YES to all of these?
- [ ] I can explain JIT tiered compilation and how it affects startup vs steady-state performance
- [ ] I understand `MethodTable`, `EEClass`, and how the runtime represents types
- [ ] I've used `DOTNET_JitDisasm` or `DOTNET_JitDump` to inspect JIT output
- [ ] I can navigate `src/coreclr/vm/` and `src/coreclr/jit/` with reasonable fluency

**If NO → Start at [Level 4 — Internals](#level-4--internals)**

### If you answered YES to all of the above:
**→ You're ready for [Level 5 — Expert / Contributor](#level-5--expert--contributor)**

---

## Path Overview

The learning path covers **8 topic tracks** that weave through all 5 levels. Each track deepens progressively:

| Track | Scope | Key source directories |
|---|---|---|
| **A — Runtime Fundamentals** | CLR architecture, hosting, startup, type system | `src/coreclr/vm/`, `src/native/corehost/` |
| **B — Memory Management** | GC, allocations, Span, Memory, pooling | `src/coreclr/gc/`, `src/libraries/System.Memory/` |
| **C — Execution Engine** | JIT, tiered compilation, interpreter, R2R | `src/coreclr/jit/`, `src/coreclr/nativeaot/` |
| **D — Threading & Async** | Thread pool, Tasks, async/await, channels | `src/libraries/System.Threading*/`, `src/coreclr/vm/threadpool*` |
| **E — Collections & LINQ** | Data structures, LINQ internals, performance | `src/libraries/System.Collections*/`, `src/libraries/System.Linq/` |
| **F — Networking & I/O** | Sockets, HTTP, pipelines, streams | `src/libraries/System.Net*/`, `src/libraries/System.IO*/` |
| **G — Serialization & Text** | JSON, XML, regex, encoding, globalization | `src/libraries/System.Text.Json/`, `src/libraries/System.Text.RegularExpressions/` |
| **H — Diagnostics & Observability** | EventPipe, tracing, logging, counters | `src/libraries/System.Diagnostics*/`, `src/coreclr/debug/` |

---

## Level 1 — Foundations

> 🎯 *"I'm new to .NET or coming from another ecosystem"*

The goal of Level 1 is to build a solid mental model of what .NET is, how its pieces fit together, and how code goes from C# source to running process. No runtime internals yet — just a clear, accurate foundation.

| Module | Topic | Est. effort | Key question it answers |
|---|---|---|---|
| 1.1 | [.NET Ecosystem Overview](01-foundations-ecosystem-overview.md) | 2h | What are the runtime, SDK, BCL, and how do they relate? |
| 1.2 | [Project Structure and the Build System](01-foundations-project-structure.md) | 3h | What happens when I run `dotnet build`? What are all these files? |
| 1.3 | [The Type System: Values, References, and the Heap](01-foundations-type-system.md) | 4h | Where do my objects live in memory and why does it matter? |
| 1.4 | [Control Flow, Exceptions, and the Call Stack](01-foundations-control-flow.md) | 3h | What actually happens when an exception is thrown? |
| 1.5 | [Assemblies, Namespaces, and the Loader](01-foundations-assemblies.md) | 3h | How does the runtime find and load my code? |
| 1.6 | [Basic I/O: Files, Console, and Streams](01-foundations-basic-io.md) | 3h | How does `Console.WriteLine` actually write to the terminal? |
| 1.7 | [Your First Look at the Runtime Source](01-foundations-first-source-reading.md) | 2h | How is the `dotnet/runtime` repo organized and where do I start reading? |

**After Level 1, you can:** Build .NET applications with confidence, understand the compilation pipeline from C# to IL to native code, and navigate the repository structure.

---

## Level 2 — Practitioner

> 🎯 *"I build .NET apps daily but don't know what happens underneath"*

Level 2 takes the concepts you use every day — generics, async/await, DI, collections — and shows you how they actually work. You'll start reading BCL source code regularly.

| Module | Topic | Est. effort | Key question it answers |
|---|---|---|---|
| 2.1 | [Generics: From Syntax to Runtime Specialization](02-practitioner-generics.md) | 4h | Why can `List<int>` be faster than `ArrayList`? How does the runtime handle generic types? |
| 2.2 | [Collections Deep Dive](02-practitioner-collections.md) | 5h | What data structure should I choose and what are the real performance characteristics? |
| 2.3 | [Async/Await and the Task Machinery](02-practitioner-async-await.md) | 6h | What does the compiler generate for `async` methods? How does `Task` scheduling work? |
| 2.4 | [Dependency Injection: From Pattern to Framework](02-practitioner-dependency-injection.md) | 4h | How does `IServiceProvider` resolve services? What are the lifetime implications? |
| 2.5 | [LINQ: From Query Syntax to Iterator Machines](02-practitioner-linq.md) | 4h | Why is LINQ lazy? What does `Select` actually generate? |
| 2.6 | [String Handling and Text Processing](02-practitioner-strings.md) | 3h | Why are strings immutable? When should I use `StringBuilder` vs `string.Create`? |
| 2.7 | [Networking Fundamentals: HttpClient and Sockets](02-practitioner-networking.md) | 5h | How does `HttpClient` manage connections? Why should I reuse it? |
| 2.8 | [Serialization: System.Text.Json Internals](02-practitioner-json.md) | 4h | How does the JSON serializer discover properties? What makes source generators faster? |
| 2.9 | [The IDisposable Contract and Resource Management](02-practitioner-disposable.md) | 3h | What happens if I forget to dispose? How does finalization actually work? |
| 2.10 | [Configuration, Options, and the Hosting Model](02-practitioner-hosting.md) | 4h | How does `Host.CreateDefaultBuilder` wire everything together? |

**After Level 2, you can:** Read BCL source code to understand framework behavior, make informed choices about collections and async patterns, and debug common issues with DI lifetimes and resource management.

---

## Level 3 — Advanced

> 🎯 *"I optimize, debug deep issues, and read framework source sometimes"*

Level 3 is where you shift from using .NET to understanding its performance characteristics and diagnostic capabilities. You'll learn to read performance profiles, optimize memory usage, and understand the middleware/pipeline patterns that underpin the framework.

| Module | Topic | Est. effort | Key question it answers |
|---|---|---|---|
| 3.1 | [Memory Model: Stack, Heap, Span, and Memory](03-advanced-memory-model.md) | 6h | How do `Span<T>` and `Memory<T>` avoid allocations? When should I use each? |
| 3.2 | [Garbage Collection: Generations, Modes, and Tuning](03-advanced-gc.md) | 6h | How do I choose between workstation/server GC? What triggers Gen2 collections? |
| 3.3 | [High-Performance Collections and Pooling](03-advanced-collections-perf.md) | 5h | When should I use `ArrayPool`, `ObjectPool`, `FrozenDictionary`? |
| 3.4 | [Threading Primitives and Synchronization](03-advanced-threading.md) | 6h | What's the difference between `lock`, `SemaphoreSlim`, `SpinLock`? When does each matter? |
| 3.5 | [System.IO.Pipelines and High-Perf I/O](03-advanced-pipelines.md) | 5h | How does Pipelines avoid buffer copies? How does Kestrel use it? |
| 3.6 | [Regular Expressions: The Compiled Engine](03-advanced-regex.md) | 4h | How does `RegexOptions.Compiled` work? What changed with the source generator in .NET 7+? |
| 3.7 | [Reflection, Emit, and Source Generators](03-advanced-reflection.md) | 6h | How does reflection work at the runtime level? Why are source generators faster? |
| 3.8 | [Cryptography and Security Primitives](03-advanced-cryptography.md) | 4h | How does .NET wrap platform crypto (CNG, OpenSSL)? How are keys managed? |
| 3.9 | [Diagnostics: dotnet-trace, counters, and EventPipe](03-advanced-diagnostics.md) | 5h | How do I profile a production app? What events does the runtime emit? |
| 3.10 | [Native Interop: P/Invoke and LibraryImport](03-advanced-native-interop.md) | 5h | How does marshalling work? Why is `LibraryImport` better than `DllImport`? |

**After Level 3, you can:** Optimize hot paths with `Span<T>` and pooling, diagnose memory leaks and GC pressure, profile applications with runtime diagnostic tools, and write high-performance I/O code.

---

## Level 4 — Internals

> 🎯 *"I understand CLR mechanics, JIT behavior, and GC tuning"*

Level 4 takes you inside the runtime itself. You'll read C++ code in `src/coreclr/vm/` and `src/coreclr/jit/`, understand how types are laid out in memory, how the JIT compiles methods, and how the GC manages the heap at the region level.

| Module | Topic | Est. effort | Key question it answers |
|---|---|---|---|
| 4.1 | [CLR Startup: From `dotnet` to `Main()`](04-internals-clr-startup.md) | 8h | What happens between typing `dotnet run` and your `Main()` executing? |
| 4.2 | [The Type System: MethodTable, EEClass, and TypeDesc](04-internals-type-system.md) | 10h | How does the runtime represent `class`, `struct`, `interface`, and generics in memory? |
| 4.3 | [JIT Compilation: RyuJIT Internals](04-internals-jit.md) | 12h | How does RyuJIT convert IL to native code? What are the optimization phases? |
| 4.4 | [Tiered Compilation and Dynamic PGO](04-internals-tiered-compilation.md) | 6h | How does the runtime decide when to re-JIT a method? How does PGO data flow? |
| 4.5 | [GC Internals: Regions, Mark Phase, and Write Barriers](04-internals-gc-deep.md) | 12h | How does the region-based GC (new in .NET 8) differ from segments? How do write barriers work? |
| 4.6 | [Assembly Loading: Binder, ALC, and Fusion](04-internals-assembly-loading.md) | 6h | How does `AssemblyLoadContext` work? What is the custom binder pipeline? |
| 4.7 | [Thread Pool Internals and Work Stealing](04-internals-threadpool.md) | 6h | How does the managed thread pool decide how many threads to use? |
| 4.8 | [Exception Handling Machinery](04-internals-exceptions.md) | 6h | How are exceptions implemented at the native level? What's the SEH/PAL difference? |
| 4.9 | [ReadyToRun (R2R) and Crossgen2](04-internals-r2r.md) | 6h | How does ahead-of-time compilation work? What are the tradeoffs vs JIT? |
| 4.10 | [NativeAOT Compilation](04-internals-nativeaot.md) | 8h | How does NativeAOT produce a self-contained native binary? What gets trimmed? |

**After Level 4, you can:** Navigate `src/coreclr/vm/` and `src/coreclr/jit/` confidently, understand JIT output and optimization decisions, tune GC for production workloads with informed reasoning, and diagnose runtime-level issues with SOS/LLDB.

---

## Level 5 — Expert / Contributor

> 🎯 *"I can debug the runtime itself, contribute patches, extend the VM"*

Level 5 prepares you to contribute to `dotnet/runtime`. You'll build the runtime from source, write and run runtime tests, understand the CI pipeline, and learn the patterns used across the codebase for contributing changes.

| Module | Topic | Est. effort | Key question it answers |
|---|---|---|---|
| 5.1 | [Building the Runtime from Source](05-expert-building.md) | 8h | How do I build CoreCLR, Mono, and libraries from source on my machine? |
| 5.2 | [The Runtime Test Infrastructure](05-expert-testing.md) | 6h | How are runtime tests organized? How do I write and run a new test? |
| 5.3 | [Contributing a JIT Change](05-expert-jit-contribution.md) | 12h | How do I add a new JIT optimization? What's the PR review process? |
| 5.4 | [Contributing a GC Change](05-expert-gc-contribution.md) | 12h | How do I modify GC behavior safely? How do I validate with GC stress tests? |
| 5.5 | [Adding a New BCL API](05-expert-new-api.md) | 8h | What's the API review process? How do I update ref assemblies, implement, and test? |
| 5.6 | [The Mono Runtime: Architecture and Differences](05-expert-mono.md) | 10h | How does Mono differ from CoreCLR? When is each used? |
| 5.7 | [WebAssembly and the Browser Runtime](05-expert-wasm.md) | 8h | How does .NET run in the browser? What's the wasm build pipeline? |
| 5.8 | [Debugging the Runtime with SOS, LLDB, and WinDbg](05-expert-debugging.md) | 8h | How do I attach a native debugger to the CLR and inspect managed objects? |
| 5.9 | [The EventPipe and Diagnostic Server](05-expert-eventpipe.md) | 6h | How does the out-of-process diagnostic protocol work? How do I add a new event? |
| 5.10 | [Custom Hosting and the Native Hosting API](05-expert-hosting.md) | 6h | How do I embed the CLR in a native application? What does `hostfxr` do? |

**After Level 5, you can:** Build and test the runtime from source, submit pull requests to `dotnet/runtime`, debug native crashes in the CLR, and architect solutions that leverage deep knowledge of runtime behavior.

---

## Recommended Reading Order

If you're going through the full path, here's a suggested order that balances breadth with depth:

```
Level 1: 1.1 → 1.2 → 1.3 → 1.4 → 1.5 → 1.6 → 1.7
Level 2: 2.1 → 2.3 → 2.2 → 2.5 → 2.9 → 2.4 → 2.6 → 2.7 → 2.8 → 2.10
Level 3: 3.1 → 3.2 → 3.4 → 3.3 → 3.9 → 3.5 → 3.10 → 3.7 → 3.6 → 3.8
Level 4: 4.1 → 4.2 → 4.3 → 4.4 → 4.5 → 4.6 → 4.7 → 4.8 → 4.9 → 4.10
Level 5: 5.1 → 5.2 → 5.8 → 5.5 → 5.3 → 5.4 → 5.6 → 5.7 → 5.9 → 5.10
```

The order within each level prioritizes foundational runtime concepts before library-specific topics, and ensures diagnostic skills are introduced early enough to support later modules.

---

## Key Repository Paths

Quick reference for navigating `dotnet/runtime`:

| Path | Contains |
|---|---|
| `src/coreclr/vm/` | CLR virtual machine — type system, class loader, threads, interop |
| `src/coreclr/jit/` | RyuJIT — the just-in-time compiler |
| `src/coreclr/gc/` | Garbage collector implementation |
| `src/coreclr/debug/` | Managed debugging infrastructure |
| `src/coreclr/interpreter/` | CLR interpreter (new, alternative to JIT for some scenarios) |
| `src/coreclr/nativeaot/` | NativeAOT compiler and runtime |
| `src/coreclr/System.Private.CoreLib/` | CoreCLR-specific part of the base class library |
| `src/mono/` | Mono runtime (mobile, WASM, embedded scenarios) |
| `src/libraries/` | All managed libraries (BCL) |
| `src/libraries/System.Private.CoreLib/` | Shared (runtime-agnostic) part of CoreLib |
| `src/native/corehost/` | Native host (`dotnet` executable, `hostfxr`, `hostpolicy`) |
| `docs/design/` | Design documents explaining runtime features |
| `docs/workflow/` | Build and test workflow documentation |
| `docs/coding-guidelines/` | Coding conventions and contribution guidelines |

---

## Essential Tools by Level

| Level | Tools you should know |
|---|---|
| 1 | `dotnet` CLI, Visual Studio / VS Code, C# debugger |
| 2 | NuGet, `dotnet test`, unit testing frameworks (xUnit), basic logging |
| 3 | `dotnet-counters`, `dotnet-trace`, `dotnet-dump`, BenchmarkDotNet, `DOTNET_*` env vars |
| 4 | SOS debugger extension, `DOTNET_JitDisasm`, `DOTNET_GCLog`, PerfView, ETW/EventPipe |
| 5 | LLDB/WinDbg + SOS, `dotnet-dsrouter`, building from source, `CORE_ROOT`, runtime test infra |

---

## References

| Resource | Type | Coverage |
|---|---|---|
| [.NET Runtime Design Docs](https://github.com/dotnet/runtime/tree/main/docs/design) | Design docs | Official design rationale for runtime features |
| [Book of the Runtime (BotR)](https://github.com/dotnet/runtime/tree/main/docs/design/coreclr/botr) | Deep docs | Core CLR internals — type system, GC, JIT, threading |
| [Pro .NET Memory Management — Konrad Kokosa](https://prodotnetmemory.com/) | Book | Definitive guide to GC and memory internals |
| [Writing High-Performance .NET Code — Ben Watson](https://www.writinghighperf.net/) | Book | Practical performance optimization patterns |
| [Adam Sitnik's Blog](https://adamsitnik.com/) | Blog | Span, BenchmarkDotNet, hardware intrinsics |
| [Stephen Toub — Performance Improvements in .NET (annual series)](https://devblogs.microsoft.com/dotnet/) | Blog posts | Detailed annual performance changelog with source links |
| [Maoni Stephens — .NET GC Blog](https://maoni0.medium.com/) | Blog | GC internals from the GC architect |
| [Andy Ayers — JIT Compiler Blog Posts](https://devblogs.microsoft.com/dotnet/) | Blog posts | JIT tiering, PGO, optimization decisions |
| [.NET Source Browser](https://source.dot.net/) | Tool | Searchable, indexed view of the runtime source |
| [SharpLab](https://sharplab.io/) | Tool | See IL, JIT ASM, and lowered C# for any snippet |

---

*This learning path is a living document. As the runtime evolves, modules will be updated to reflect new features and architectural changes.*

*Last updated: 2026-04-14*
