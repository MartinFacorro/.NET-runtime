# Nivel 5: Experto / Contribuidor — Agregar una Nueva API a la BCL

> **Perfil objetivo:** Desarrollador listo para contribuir una nueva API publica a las bibliotecas de clase base de .NET
> **Esfuerzo estimado:** 8 horas
> **Prerrequisitos:** Nivel 3, [Modulo 5.1](05-expert-building.md), [Modulo 5.2](05-expert-testing.md)
> [English version](../en/05-expert-new-api.md)

---

## Objetivos de Aprendizaje

Al finalizar este modulo vas a poder:

1. Navegar el ciclo de vida completo de una API desde la propuesta inicial, pasando por la review de FXDC, la implementacion, testing, hasta el merge.
2. Escribir una propuesta de API efectiva usando la plantilla de dotnet/runtime, con codigo de escenarios y justificacion de diseno.
3. Actualizar un ref assembly (proyecto `ref/`) siguiendo el patron `throw null;` y explicar por que existen los ref assemblies.
4. Implementar una nueva API en el proyecto `src/` siguiendo las convenciones de codigo de dotnet/runtime, incluyendo comentarios XML doc y patrones platform-specific.
5. Escribir tests exhaustivos usando patrones `[Theory]`/`[InlineData]`, cubriendo edge cases, entradas null y escenarios negativos.
6. Navegar el flujo de trabajo del PR incluyendo checks de CI, validacion de compatibilidad de API, code review y requisitos de merge.

---

## Mapa Conceptual

```mermaid
graph LR
    classDef proposal fill:#264653,stroke:#264653,color:#ffffff
    classDef review fill:#2a9d8f,stroke:#2a9d8f,color:#ffffff
    classDef impl fill:#e9c46a,stroke:#e9c46a,color:#000000
    classDef test fill:#f4a261,stroke:#f4a261,color:#000000
    classDef merge fill:#e76f51,stroke:#e76f51,color:#ffffff

    IDEA["Idea / Necesidad"]:::proposal
    PROPOSAL["Propuesta de API<br/>(GitHub Issue)"]:::proposal
    DISCUSSION["Discusion<br/>con Area Owner"]:::review
    FXDC["Review de FXDC<br/>(api-ready-for-review)"]:::review
    APPROVED["api-approved"]:::review
    REF["Actualizar ref/<br/>(Ref Assembly)"]:::impl
    SRC["Implementar en src/<br/>(Source Assembly)"]:::impl
    TEST["Escribir Tests<br/>(proyecto tests/)"]:::test
    PR["Pull Request"]:::merge
    CI["CI + ApiCompat"]:::merge
    MERGE["Merge"]:::merge

    IDEA --> PROPOSAL --> DISCUSSION --> FXDC --> APPROVED --> REF --> SRC --> TEST --> PR --> CI --> MERGE
```

---

## Guia de Lectura del Codigo Fuente

| Dificultad | Archivo / Ruta | Proposito |
|------------|----------------|-----------|
| ★★ | `docs/project/api-review-process.md` | Documentacion oficial del proceso de API review |
| ★★ | `docs/coding-guidelines/adding-api-guidelines.md` | Guia paso a paso para agregar APIs al repo |
| ★★ | `docs/coding-guidelines/updating-ref-source.md` | Como actualizar el ref assembly despues de agregar una API |
| ★★ | `docs/coding-guidelines/coding-style.md` | Reglas de estilo de C# aplicadas en todo el repo |
| ★★ | `docs/coding-guidelines/framework-design-guidelines-digest.md` | Resumen del libro Framework Design Guidelines |
| ★★ | `src/libraries/System.Collections/ref/System.Collections.cs` | Ejemplo de ref assembly -- nota el patron `throw null;` |
| ★★ | `src/libraries/System.Collections/ref/System.Collections.csproj` | Ejemplo de proyecto ref -- usa `$(NetCoreAppCurrent)` |
| ★★ | `src/libraries/System.Collections/src/System.Collections.csproj` | Ejemplo de proyecto src -- assembly de implementacion |
| ★★★ | `src/libraries/System.Collections/src/System/Collections/Generic/LinkedList.cs` | Ejemplo de implementacion con XML docs |
| ★★ | `src/libraries/System.Collections/tests/System.Collections.Tests.csproj` | Estructura de ejemplo del proyecto de tests |
| ★★ | `.editorconfig` | Reglas de auto-formateo del codebase |

---

## Curriculum

### Leccion 1 — El Proceso de API Review

#### Que vas a aprender

Toda API publica que se agrega a dotnet/runtime pasa por un proceso formal de review. Esto no es burocracia -- es una de las barreras de calidad mas importantes del ecosistema .NET. Las APIs son efectivamente permanentes una vez publicadas: removerlas o cambiarlas romperia millones de aplicaciones. Entender este proceso es el primer paso hacia una contribucion exitosa.

#### El comite de review: FXDC

El Framework Design Core (FXDC) es el grupo que conduce las API reviews. Las reviews ocurren semanalmente, tipicamente los martes de 10am a 12pm hora del Pacifico, y se transmiten en vivo en el canal de YouTube de .NET Foundation. Despues de cada review, las notas se publican en el repositorio [dotnet/apireviews](https://github.com/dotnet/apireviews).

#### El ciclo de vida de una API

Abri `docs/project/api-review-process.md`. El proceso tiene cinco etapas:

**1. Crear un issue.** Usa la [plantilla de API Proposal](https://github.com/dotnet/runtime/issues/new?assignees=&labels=api-suggestion&template=02_api_proposal.yml&title=%5BAPI+Proposal%5D%3A+). El issue recibe la etiqueta `api-suggestion`. Tu propuesta debe incluir:
- Una descripcion clara del problema que resuelve
- Codigo de escenarios mostrando como se usaria la API
- Un boceto de la superficie publica propuesta (no necesariamente final)

**2. Se asigna un owner.** El [area owner](https://github.com/dotnet/runtime/blob/main/docs/project/issue-guide.md#areas) del namespace relevante patrocina el issue y conduce la discusion.

**3. Discusion.** El owner y la comunidad refinan la propuesta. El objetivo es llegar a un diseno concreto y accionable. Si se necesitan cambios, se aplica la etiqueta `api-needs-work`. Deberias editar la descripcion original del issue (no agregar comentarios nuevos) para mantener la propuesta actualizada. Un pequeno changelog "Updates:" al final ayuda a los reviewers a rastrear que cambio.

**4. Decision del owner.** El owner puede:
- Etiquetar el issue como `api-ready-for-review` (pasa a review formal)
- Cerrar el issue como no accionable, won't fix o won't fix as proposed

**5. Review de FXDC.** El comite examina la propuesta y decide:
- **Aprueba** (la etiqueta cambia a `api-approved`)
- Devuelve con **needs work** (`api-needs-work`)
- **Rechaza** (el issue se cierra)

#### La regla de oro

Del documento de proceso de review: "Pull requests against dotnet/runtime shouldn't be submitted before getting approval." No envies un PR con tu implementacion antes de que la API este aprobada. Trabaja en una rama de tu propio fork si queres prototipar.

#### Tres formas de entrar a review

1. **Backlog**: Crea el issue con `api-ready-for-review`. Las reviews se procesan de la mas vieja a la mas nueva.
2. **Fast track**: Agrega las etiquetas `api-ready-for-review` y `blocking`. Los issues bloqueantes se revisan antes del backlog.
3. **Review dedicada**: Solo para area owners, cuando la propuesta necesita 60+ minutos.

#### Ejercicio

1. Navega [https://aka.ms/ready-for-api-review](https://aka.ms/ready-for-api-review) y lee dos o tres propuestas aprobadas. Nota el formato: declaracion del problema, codigo de escenarios, forma de la API.
2. Lee las notas de review de una API aprobada en [dotnet/apireviews](https://github.com/dotnet/apireviews). Presta atencion al tipo de feedback que da FXDC: elecciones de nombres, diseno de overloads, manejo de null, consistencia con APIs existentes.
3. Abri `docs/coding-guidelines/framework-design-guidelines-digest.md` y lee la seccion de convenciones de nombres. Identifica tres reglas que aplicarian a tu proxima propuesta de API.

---

### Leccion 2 — Escribir la Propuesta de API

#### Que vas a aprender

Una buena propuesta es la diferencia entre una API que pasa la review sin problemas y una que necesita multiples rondas de revision. Esta leccion cubre como escribir una propuesta que FXDC pueda evaluar eficientemente.

#### La plantilla de propuesta

La plantilla pide varias secciones. Asi se llena cada una efectivamente:

**Background and motivation**: Explica el problema. Por que la superficie actual de la API no lo resuelve? Enlaza a preguntas de Stack Overflow, issues de GitHub o codigo real que muestre el dolor. Los reviewers son mas propensos a aprobar una API cuando pueden ver evidencia concreta de necesidad del desarrollador.

**API proposal**: Provee un bloque de codigo C# mostrando la superficie publica. Usa el formato condensado que espera FXDC:

```csharp
namespace System.Collections.Generic;

public class List<T>
{
    // Miembros existentes omitidos por brevedad

    // NUEVO: API propuesta
    public int RemoveAll(ReadOnlySpan<T> items);
}
```

Incluye solo los miembros nuevos. Marcalos claramente con comentarios. Especifica nombres de parametros, tipos de retorno, constraints genericos y atributos. La forma no necesita ser final -- FXDC la va a refinar -- pero debe ser lo suficientemente concreta para evaluar.

**Usage examples**: Mostra 2-3 escenarios realistas. Estos son la parte mas importante de la propuesta. Los reviewers los usan para evaluar usabilidad:

```csharp
// Escenario 1: Remover multiples items conocidos de una lista
List<string> names = GetNames();
ReadOnlySpan<string> blocked = ["spam", "test", "admin"];
int removed = names.RemoveAll(blocked);
Console.WriteLine($"Se removieron {removed} nombres bloqueados");
```

**Alternative designs**: Mostra que consideraste otros enfoques y explica por que elegiste este. Esto demuestra madurez de diseno y anticipa feedback comun de la review.

**Risks**: Identifica breaking changes, implicaciones de rendimiento o preocupaciones platform-specific.

#### Convenciones de nombres

Abri `docs/coding-guidelines/framework-design-guidelines-digest.md`. Las reglas clave de nombres:

- **PascalCase** para todos los identificadores publicos (tipos, metodos, propiedades, eventos, constantes)
- **camelCase** para parametros
- Prefijo `T` para type parameters (e.g., `TKey`, `TValue`)
- Usa nombres descriptivos: `GetOrAdd` no `GetAdd`, `TryParse` no `Parse2`
- Evita abreviaturas: `GetConnection` no `GetConn`
- Propiedades/metodos booleanos se leen como preguntas: `IsEmpty`, `HasValue`, `CanSeek`
- Metodos async terminan con `Async`: `ReadAsync`, `WriteAsync`
- Metodos factory usan `Create*`: `CreateInstance`, `CreateScope`
- Patron `Try*` para metodos que pueden fallar sin excepciones: retorna `bool`, valor via parametro `out`

#### Consideraciones de diseno

Estas son las preguntas que FXDC va a hacer. Prepara respuestas:

1. **Pertenece esto a la BCL?** O deberia ser un paquete NuGet? La BCL debe contener primitivas sobre las que muchas bibliotecas construyen. Si solo un dominio de aplicacion lo necesita, probablemente va en otro lado.
2. **Es consistente el naming?** Mira APIs pares en el mismo tipo o namespace. Si `List<T>` tiene `AddRange(IEnumerable<T>)`, un nuevo metodo de adicion por lote debe seguir el mismo patron de nombres.
3. **Que pasa con null?** Deberia el metodo aceptar entradas null? Lanzar `ArgumentNullException`? Usar anotaciones de nullable?
4. **Que pasa con entradas vacias?** `RemoveAll(ReadOnlySpan<T>.Empty)` deberia ser un no-op o lanzar excepcion?
5. **Implicaciones de rendimiento?** Esto causa allocations? Hay un overload basado en `Span`?
6. **Diseno de overloads?** Provee el caso mas comun primero, con overloads adicionales para escenarios avanzados. Segui el principio de "revelacion progresiva".

#### Ejercicio

1. Elige una API que desees que existiera en la BCL. Escribi una propuesta siguiendo la plantilla: background, superficie de la API, 2 ejemplos de uso, 1 diseno alternativo y riesgos.
2. Revisa tu propuesta contra el resumen de guias de nombres. Cada nombre sigue las convenciones?
3. Busca en los issues de dotnet/runtime `label:api-approved` y encuentra una API similar a la tuya. Compara el formato y nivel de detalle de la propuesta.

---

### Leccion 3 — Actualizar el Ref Assembly

#### Que vas a aprender

Una vez que la API esta aprobada, el primer cambio de codigo es actualizar el ref assembly. Esta leccion explica que son los ref assemblies, por que existen y como actualizarlos correctamente.

#### Que es un ref assembly?

Un ref assembly contiene solo la superficie publica de la API de una biblioteca -- firmas de tipos, firmas de metodos y atributos -- pero nada de implementacion. Los cuerpos de los metodos se reemplazan con `throw null;`. El compilador usa los ref assemblies durante la compilacion para verificar que tu codigo usa los tipos y miembros correctos, pero el comportamiento real viene del assembly de implementacion en tiempo de ejecucion.

Abri `src/libraries/System.Collections/ref/System.Collections.cs` y mira el patron:

```csharp
// Licensed to the .NET Foundation under one or more agreements.
// The .NET Foundation licenses this file to you under the MIT license.
// ------------------------------------------------------------------------------
// Changes to this file must follow the https://aka.ms/api-review process.
// ------------------------------------------------------------------------------

namespace System.Collections.Generic
{
    public partial class LinkedList<T> : ICollection<T>, IEnumerable<T>, ...
    {
        public LinkedList() { }
        public int Count { get { throw null; } }
        public void AddFirst(LinkedListNode<T> node) { }
        public LinkedListNode<T> AddFirst(T value) { throw null; }
        // ... cada miembro publico listado con cuerpos throw null;
    }
}
```

Nota varias cosas:
- **Cada miembro publico** esta listado, con su firma completa
- Metodos que retornan un valor usan `throw null;` como cuerpo
- Metodos que retornan `void` tienen cuerpos vacios `{ }`
- Propiedades usan los patrones `get { throw null; }` y `set { }`
- El header del archivo incluye el comentario del proceso de API review
- Los tipos son `partial` (la implementacion esta en `src/`)

#### Por que existen los ref assemblies

1. **Velocidad de compilacion**: Compilar contra un ref assembly es mas rapido porque el compilador no necesita parsear codigo de implementacion.
2. **Contrato de superficie de API**: El ref assembly es el registro definitivo de la API publica. Si no esta en `ref/`, no es parte de la API publica, incluso si un miembro `public` existe en `src/`.
3. **Compatibilidad binaria**: Las herramientas de compatibilidad de API (`ApiCompat`) comparan ref assemblies entre versiones para detectar breaking changes.
4. **Desacople**: La identidad del assembly de implementacion puede diferir del ref assembly. Los tipos pueden moverse entre assemblies de implementacion sin romper el contrato en tiempo de compilacion.

#### La estructura del proyecto ref

Abri `src/libraries/System.Collections/ref/System.Collections.csproj`:

```xml
<Project Sdk="Microsoft.NET.Sdk">
  <PropertyGroup>
    <TargetFramework>$(NetCoreAppCurrent)</TargetFramework>
  </PropertyGroup>
  <ItemGroup>
    <Compile Include="System.Collections.cs" />
    <Compile Include="System.Collections.Forwards.cs" />
  </ItemGroup>
  <ItemGroup>
    <ProjectReference Include="..\..\System.Runtime\ref\System.Runtime.csproj" />
  </ItemGroup>
</Project>
```

Puntos clave:
- Usa `$(NetCoreAppCurrent)` -- nunca hardcodees la version del TFM
- Solo referencia otros proyectos `ref/`, nunca proyectos `src/`
- El archivo `.Forwards.cs` contiene type forwards para tipos que viven en un assembly de implementacion diferente

#### Como actualizar el ref assembly

Segui el proceso de `docs/coding-guidelines/updating-ref-source.md`:

**Paso 1: Implementa primero.** Escribi la API en `src/` y compila el assembly fuente. Si estas agregando un tipo publico nuevo, el build puede fallar con un error `TypeMustExist`. Para solucionarlo, compila con la validacion de ApiCompat deshabilitada:

```bash
dotnet build /p:ApiCompatValidateAssemblies=false
```

**Paso 2: Usa GenAPI para actualizar ref.** Desde el directorio `src/`, ejecuta:

```bash
dotnet msbuild /t:GenerateReferenceAssemblySource
```

GenAPI genera el codigo fuente del ref assembly desde el assembly de implementacion compilado. Sin embargo, puede producir discrepancias que necesitan correcciones manuales. La herramienta es la forma preferida de actualizar ref assemblies porque los mantiene consistentes con la salida real del build.

**Paso 3: Compila el ref assembly.** Navega al directorio `ref/` y compila:

```bash
cd src/libraries/<NombreBiblioteca>/ref
dotnet build
```

**Paso 4: Verifica.** El ref assembly debe compilar y la superficie de la API debe coincidir con lo aprobado.

#### Reglas de edicion manual

Si debes editar el codigo fuente del ref manualmente:

- Usa nombres de tipo completamente calificados en atributos
- Mantene los miembros en el mismo orden que produce GenAPI (alfabetico dentro de grupos)
- Los tipos de retorno y tipos de parametro deben coincidir exactamente con la implementacion
- Las anotaciones de nullable deben coincidir
- Los valores por defecto de parametros deben coincidir
- Los constraints genericos deben coincidir
- Nunca agregues logica de implementacion -- siempre `throw null;` o `{ }`

#### Para System.Runtime especificamente

Si tu API toca tipos en `System.Private.CoreLib` (que se expone a traves de `System.Runtime`), el proceso es ligeramente diferente:

```bash
# Desde el directorio System.Runtime/src:
dotnet build --no-incremental /t:GenerateReferenceAssemblySource
```

Despues filtra los cambios no relacionados de la salida generada. Esto es necesario porque `System.Runtime` expone una superficie muy grande que agrega muchos assemblies subyacentes.

#### Ejercicio

1. Abri `src/libraries/System.Collections/ref/System.Collections.cs`. Encuentra un metodo que retorne `void` y uno que retorne un valor. Verifica el patron `{ }` vs `throw null;`.
2. Abri el `.csproj` del ref y el `.csproj` del src uno al lado del otro. Nota que ref referencia otros proyectos ref, mientras que src referencia `$(CoreLibProject)`.
3. Lee `docs/coding-guidelines/updating-ref-source.md`. Identifica las tres rutas diferentes segun el tipo de tu biblioteca (regular, System.Runtime, full facade).

---

### Leccion 4 — Implementar la API

#### Que vas a aprender

Con el ref assembly actualizado y compilando, ahora implementas la logica real en el proyecto `src/`. Esta leccion cubre las convenciones de codigo, requisitos de documentacion y patrones platform-specific usados en dotnet/runtime.

#### Estructura del proyecto de implementacion

Abri `src/libraries/System.Collections/src/System.Collections.csproj`. El proyecto de implementacion:

```xml
<Project Sdk="Microsoft.NET.Sdk">
  <PropertyGroup>
    <TargetFramework>$(NetCoreAppCurrent)</TargetFramework>
    <AllowUnsafeBlocks>true</AllowUnsafeBlocks>
  </PropertyGroup>
  <ItemGroup>
    <Compile Include="System\Collections\Generic\LinkedList.cs" />
    <!-- Mas archivos fuente -->
    <Compile Include="$(CoreLibSharedDir)System\Collections\HashHelpers.cs"
             Link="Common\System\Collections\HashHelpers.cs" />
    <!-- Archivos comunes compartidos -->
  </ItemGroup>
  <ItemGroup>
    <ProjectReference Include="$(CoreLibProject)" />
  </ItemGroup>
</Project>
```

Observaciones clave:
- Los archivos fuente viven bajo la estructura de directorios del namespace: `System/Collections/Generic/LinkedList.cs`
- Los archivos compartidos de `CoreLibSharedDir` y `CommonPath` se vinculan via el atributo `Link`
- El proyecto referencia `$(CoreLibProject)`, no otros proyectos `src/` directamente

#### Convenciones de codigo

Abri `docs/coding-guidelines/coding-style.md` y `.editorconfig`. Las reglas mas criticas:

**Llaves y formateo**:
- Llaves estilo Allman: cada llave en su propia linea
- Cuatro espacios de indentacion, no tabs
- No mas de una linea en blanco entre miembros

**Nombres**:
- `_camelCase` para campos de instancia private/internal
- Prefijo `s_` para campos static, prefijo `t_` para campos `[ThreadStatic]`
- PascalCase para constantes, metodos, propiedades, tipos
- La visibilidad siempre es el primer modificador: `public static` no `static public`

**Uso de tipos**:
- Keywords del lenguaje sobre tipos BCL: `int` no `Int32`, `string` no `String`
- `var` solo cuando el tipo es explicito a la derecha: `var list = new List<int>()` esta bien, `var result = GetResult()` no
- `nameof(parameter)` en vez de `"parameter"` en mensajes de excepcion
- `is null` / `is not null` en vez de `== null` / `!= null`

**Patrones modernos**:
- Usa `LibraryImport` (generado por source) sobre `DllImport` para nuevas declaraciones P/Invoke
- Preferi `static` para helpers sin estado, `sealed` cuando no se necesita herencia
- Namespaces file-scoped preferidos para archivos nuevos
- Usa `ObjectDisposedException.ThrowIf` donde corresponda

#### Header de archivo

Todo archivo nuevo debe empezar con:

```csharp
// Licensed to the .NET Foundation under one or more agreements.
// The .NET Foundation licenses this file to you under the MIT license.
```

#### Documentacion XML

Las nuevas APIs publicas deben tener comentarios de documentacion XML con triple-slash. Los comentarios son la fuente de verdad para IntelliSense y la documentacion oficial de la API en learn.microsoft.com.

```csharp
/// <summary>
/// Remueve todos los elementos que coinciden con alguno de los items especificados.
/// </summary>
/// <param name="items">Los items a remover de la lista.</param>
/// <returns>La cantidad de elementos removidos de la lista.</returns>
/// <exception cref="ArgumentNullException">
/// <paramref name="items"/> es <see langword="null"/>.
/// </exception>
public int RemoveAll(ReadOnlySpan<T> items)
{
    // implementacion
}
```

De `docs/coding-guidelines/adding-api-guidelines.md`: "If your new API or the APIs it calls throw any exceptions, those need to be manually documented by adding the `<exception></exception>` elements."

#### Validacion de argumentos

El runtime sigue un patron estricto para validacion de argumentos:

```csharp
public void Insert(int index, T item)
{
    ArgumentOutOfRangeException.ThrowIfNegative(index);
    ArgumentOutOfRangeException.ThrowIfGreaterThan(index, _size);

    // implementacion
}
```

Para checks de null:

```csharp
public void AddRange(IEnumerable<T> collection)
{
    ArgumentNullException.ThrowIfNull(collection);

    // implementacion
}
```

Usa el patron `ThrowHelper` para hot paths donde quieras mantener la logica de throw fuera del cuerpo del metodo inlineado.

#### Codigo platform-specific

De `CLAUDE.md`, en orden de preferencia:

1. **Clases parciales con archivos platform-specific**: `Stream.Unix.cs`, `Stream.Windows.cs`
2. **Archivos completamente separados** cuando toda la clase difiere por plataforma
3. **Directivas `#if`** como ultimo recurso

Las condiciones de plataforma en MSBuild usan `TargetPlatformIdentifier`:

```xml
<Compile Include="MyType.Windows.cs" Condition="'$(TargetPlatformIdentifier)' == 'windows'" />
<Compile Include="MyType.Unix.cs" Condition="'$(TargetPlatformIdentifier)' != 'windows'" />
```

#### Patron de interop nativo

Si tu API requiere llamadas nativas:

```csharp
internal static partial class Interop
{
    internal static partial class Kernel32
    {
        [LibraryImport(Libraries.Kernel32)]
        internal static partial int GetCurrentProcessId();
    }
}
```

Los archivos de interop viven bajo `src/libraries/Common/src/Interop/`, organizados por plataforma (e.g., `Interop/Windows/Kernel32/`, `Interop/Unix/System.Native/`).

#### Compilar tu implementacion

```bash
cd src/libraries/<NombreBiblioteca>
dotnet build
```

Si el build falla con errores de ApiCompat porque el ref assembly no fue actualizado todavia, usa:

```bash
dotnet build /p:ApiCompatValidateAssemblies=false
```

Para deshabilitar warnings-as-errors durante el desarrollo:

```bash
export TreatWarningsAsErrors=false
```

#### Ejercicio

1. Abri `src/libraries/System.Collections/src/System/Collections/Generic/LinkedList.cs`. Estudia el estilo de codigo: llaves, nombres, convenciones de campos, comentarios XML doc.
2. Encuentra un ejemplo de validacion de argumentos usando `ArgumentNullException.ThrowIfNull` en el arbol de `src/libraries/`. Nota como la validacion ocurre al inicio del metodo antes de cualquier logica.
3. Busca una biblioteca que tenga archivos platform-specific (e.g., `src/libraries/System.IO.FileSystem/src/`). Encuentra el patron de split de clases parciales `.Windows.cs` y `.Unix.cs`.

---

### Leccion 5 — Escribir Tests

#### Que vas a aprender

Los tests no son opcionales en dotnet/runtime. Toda nueva API debe tener tests exhaustivos antes de que el PR pueda mergearse. Esta leccion cubre la estructura del proyecto de tests, convenciones y patrones usados a traves de la BCL.

#### Estructura del proyecto de tests

Cada biblioteca tiene un directorio `tests/`. Abri `src/libraries/System.Collections/tests/`:

```
tests/
    BitArray/
    Generic/
        Dictionary/
        HashSet/
        LinkedList/
        List/
            List.Generic.Tests.cs
            List.Generic.Tests.AddRange.cs
            List.Generic.Tests.BinarySearch.cs
            ...
        Queue/
        SortedDictionary/
        SortedList/
        SortedSet/
        Stack/
    StructuralComparisonsTests.cs
    System.Collections.Tests.csproj
```

Convenciones clave:
- Los tests se organizan por tipo dentro de subdirectorios (e.g., `Generic/List/`)
- Los archivos de test coinciden con la funcionalidad que prueban: `List.Generic.Tests.AddRange.cs` testea `AddRange`
- Un archivo principal de clase de test cubre el tipo central: `List.Generic.Tests.cs`
- El `.csproj` de tests referencia `$(NetCoreAppCurrent)`
- Agrega tests nuevos a archivos de test existentes en vez de crear archivos nuevos cuando sea posible

#### Patrones de test: Theory e InlineData

El repo prefiere fuertemente `[Theory]` con tests data-driven sobre multiples metodos `[Fact]` individuales:

```csharp
[Theory]
[InlineData(0)]
[InlineData(1)]
[InlineData(75)]
public void RemoveAll_VariousSizes(int count)
{
    List<int> list = Enumerable.Range(0, count).ToList();
    ReadOnlySpan<int> toRemove = [0, 50, 74];

    int removed = list.RemoveAll(toRemove);

    Assert.Equal(Math.Min(count, toRemove.Length), removed);
}
```

Usa `[MemberData]` para conjuntos de datos mas complejos:

```csharp
public static IEnumerable<object[]> RemoveAll_TestData()
{
    yield return new object[] { new List<int> { 1, 2, 3 }, new int[] { 2 }, 1, new int[] { 1, 3 } };
    yield return new object[] { new List<int> { 1, 2, 3 }, new int[] { 4 }, 0, new int[] { 1, 2, 3 } };
    yield return new object[] { new List<int>(), new int[] { 1 }, 0, Array.Empty<int>() };
}

[Theory]
[MemberData(nameof(RemoveAll_TestData))]
public void RemoveAll_ReturnsExpected(List<int> list, int[] toRemove, int expectedRemoved, int[] expectedList)
{
    Assert.Equal(expectedRemoved, list.RemoveAll(toRemove.AsSpan()));
    Assert.Equal(expectedList, list);
}
```

#### Que testear

Para cada nueva API, cubre estas categorias:

**1. Camino feliz**: Los escenarios normales y esperados de uso.

**2. Edge cases**:
- Colecciones vacias / entrada vacia
- Colecciones de un solo elemento
- Colecciones muy grandes
- Valores duplicados
- Valores por defecto (`default(T)`, `null` para tipos de referencia)

**3. Condiciones de frontera**:
- `int.MaxValue`, `int.MinValue` para parametros enteros
- Spans de longitud cero
- Indice al inicio/fin de la coleccion

**4. Tests negativos** (fallos esperados):
- Argumentos `null` deben lanzar `ArgumentNullException`
- Indices fuera de rango deben lanzar `ArgumentOutOfRangeException`
- Operaciones en objetos disposed deben lanzar `ObjectDisposedException`
- Valores de enum invalidos, conteos negativos, etc.

```csharp
[Fact]
public void RemoveAll_NullItems_ThrowsArgumentNullException()
{
    var list = new List<string> { "a", "b" };

    Assert.Throws<ArgumentNullException>("items", () => list.RemoveAll(null));
}

[Fact]
public void RemoveAll_EmptyList_ReturnsZero()
{
    var list = new List<int>();
    ReadOnlySpan<int> toRemove = [1, 2, 3];

    Assert.Equal(0, list.RemoveAll(toRemove));
}

[Fact]
public void RemoveAll_EmptySpan_ReturnsZero()
{
    var list = new List<int> { 1, 2, 3 };

    Assert.Equal(0, list.RemoveAll(ReadOnlySpan<int>.Empty));
    Assert.Equal(3, list.Count);
}
```

**5. Tests de concurrencia** (si la API tiene implicaciones de thread-safety):
- Lecturas concurrentes no deben corromper el estado
- Escrituras concurrentes deben lanzar excepcion o estar documentadas como seguras

#### Convenciones de test especificas de dotnet/runtime

De `CLAUDE.md` y patrones observados:

- **No** emitas comentarios "Arrange", "Act", "Assert"
- **No** agregues comentarios de regresion citando numeros de issue/PR de GitHub a menos que se te pida explicitamente
- Asegurate de que los archivos nuevos esten listados en el `.csproj` si otros archivos en esa carpeta lo estan
- Cuando ejecutes tests, usa filtros y verifica los conteos de ejecucion para confirmar que los tests realmente corrieron
- La clase de archivo de test usualmente es `abstract partial` con un parametro de tipo generico, permitiendo clases de test concretas por tipo de elemento

#### Ejecutar tests

```bash
cd src/libraries/<NombreBiblioteca>
dotnet build /t:Test ./tests/<ProyectoTest>.csproj
```

Para ejecutar tests especificos:

```bash
dotnet test ./tests/<ProyectoTest>.csproj --filter "FullyQualifiedName~RemoveAll"
```

Verifica el conteo de ejecucion en la salida para confirmar que tus tests realmente corrieron. Un error comun es escribir tests que compilan pero no son descubiertos por el test runner debido a atributos faltantes o visibilidad incorrecta de la clase.

#### Ejercicio

1. Abri `src/libraries/System.Collections/tests/Generic/List/List.Generic.Tests.cs`. Estudia el patron de clase de test abstracta. Nota como hereda de `IList_Generic_Tests<T>` y provee metodos factory.
2. Encuentra un archivo de test que use `[Theory]` con `[MemberData]`. Cuenta la cantidad de test cases y nota como se cubren los edge cases.
3. Escribi un plan de test (no codigo) para un hipotetico metodo `List<T>.RemoveAll(ReadOnlySpan<T>)`. Lista al menos 10 escenarios de test distintos cubriendo camino feliz, edge cases, fronteras y tests negativos.

---

### Leccion 6 — El Flujo de Trabajo del PR

#### Que vas a aprender

Con el ref assembly actualizado, la implementacion escrita y los tests pasando, estas listo para enviar un pull request. Esta leccion cubre el pipeline de CI, los checks de compatibilidad de API, el proceso de review y que esperar antes del merge.

#### Antes de enviar el PR

1. **Confirma que la API esta aprobada.** El issue debe tener la etiqueta `api-approved`. Nunca envies un PR de implementacion para una API no aprobada.

2. **Ejecuta tests localmente.** Compila y testea la biblioteca completa:

```bash
cd src/libraries/<NombreBiblioteca>
dotnet build
dotnet build /t:Test ./tests/<ProyectoTest>.csproj
```

3. **Ejecuta checks de formateo.** El repo usa `dotnet-format`:

```bash
dotnet format --verify-no-changes
```

4. **Verifica compatibilidad de API localmente.** Compila los assemblies ref y src -- el sistema de build ejecuta ApiCompat automaticamente. Si falla, el ref assembly no coincide con la implementacion.

#### Crear el PR

Un buen PR para una nueva API incluye:

**Titulo**: Conciso, descriptivo, menos de 70 caracteres. Ejemplo: `Add List<T>.RemoveAll(ReadOnlySpan<T>)`

**Descripcion**:
- Link al issue de API aprobado: `Fixes #12345`
- Resumen breve del enfoque de implementacion
- Decisiones de diseno o tradeoffs
- Resumen de tests: que escenarios se cubren

**Higiene de commits**:
- Un commit logico por cambio, o squash antes de enviar
- El mensaje de commit debe referenciar el issue

#### Checks de CI

Cuando envias un PR, el pipeline de CI se ejecuta automaticamente. Los checks clave:

**1. Validacion de build**: Todo el repo compila (configuraciones debug y release, multiples plataformas).

**2. ApiCompat**: Valida que:
- El ref assembly coincide con el assembly de implementacion
- No se removieron o modificaron accidentalmente APIs existentes
- No se introdujeron breaking changes

Si ApiCompat falla con un error `TypeMustExist` o `MemberMustExist`, a tu ref assembly le falta algo que la implementacion expone. Si falla con un error `CannotRemove*`, accidentalmente removiste una API existente.

**3. Ejecucion de tests**: Los tests corren en multiples plataformas (Windows, Linux, macOS) y configuraciones (Debug, Release). Tus tests deben pasar en todas partes.

**4. Formateo**: El estilo de codigo se verifica contra `.editorconfig`. Las violaciones hacen fallar el build.

**5. Diff de API**: Para PRs que modifican assemblies `ref/`, se genera un diff mostrando el cambio en la superficie de la API. Los reviewers lo usan para verificar que el cambio coincide con la forma aprobada de la API.

#### ApiCompat en detalle

ApiCompat es el validador automatizado de compatibilidad de API. Funciona comparando:
- El ref assembly contra el assembly de implementacion (deben concordar en la superficie publica)
- El ref assembly actual contra el ref assembly de la version anterior (sin breaking changes)

De `docs/coding-guidelines/updating-ref-source.md`: cuando agregas un tipo nuevo, el build puede fallar porque los checks de ApiCompat corren antes de que el ref assembly este actualizado. La solucion temporal:

```bash
dotnet build /p:ApiCompatValidateAssemblies=false
```

Usa esto **solo** durante el desarrollo. El PR final debe pasar ApiCompat sin este flag.

Errores comunes de ApiCompat y sus soluciones:

| Error | Causa | Solucion |
|-------|-------|----------|
| `TypeMustExist` | Tipo nuevo en src/ pero falta en ref/ | Agrega el tipo al ref assembly |
| `MemberMustExist` | Miembro nuevo en src/ pero falta en ref/ | Agrega el miembro al ref assembly |
| `CannotRemoveType` | Se removio un tipo de ref/ | Restaura el tipo (esto seria un breaking change) |
| `CannotChangeVisibility` | Discrepancia de visibilidad entre ref/ y src/ | Hacelos coincidir |

#### Code review

Tu PR va a ser revisado por:
- **El area owner**: Asegura que la implementacion coincide con el diseno de la API aprobada
- **Reviewers de la comunidad**: Pueden proveer feedback adicional sobre estilo, rendimiento o correctitud
- **Bots de CI**: Todos los checks automatizados deben pasar

Feedback comun de review en PRs de nuevas APIs:
- **Tests de edge cases faltantes**: Los reviewers van a pedir tests de null, tests de entrada vacia, tests de frontera
- **Calidad de XML doc**: Los comentarios deben ser precisos y completos
- **Consistencia de nombres**: Nombres de parametros y metodos deben coincidir con patrones establecidos
- **Rendimiento**: Evitar allocations innecesarias en hot paths
- **Thread safety**: Documentar si la API es thread-safe o no

#### Despues del merge

Una vez que tu PR esta mergeado:
1. La API es parte del proximo release preview de .NET
2. Los comentarios XML doc se sincronizaran a [dotnet-api-docs](https://github.com/dotnet/dotnet-api-docs) para la documentacion oficial
3. La API aparecera en IntelliSense en Visual Studio y VS Code
4. La API es efectivamente permanente -- no se puede remover sin un bump de version major y un proceso formal de deprecation

#### Resumen completo del flujo de trabajo

```
1. Identificar necesidad → Escribir issue de propuesta de API → Etiqueta: api-suggestion
2. Discusion con area owner → Refinamiento → Etiqueta: api-ready-for-review
3. Review de FXDC → Etiqueta: api-approved
4. Forkear el repo → Crear una rama
5. Actualizar ref/ assembly (agregar nueva superficie publica, cuerpos throw null;)
6. Implementar en src/ (convenciones de codigo, XML docs, validacion de argumentos)
7. Escribir tests en tests/ (camino feliz, edge cases, tests negativos)
8. Compilar y testear localmente
9. Enviar PR → Enlazar al issue aprobado
10. CI pasa → Code review → Atender feedback
11. Merge → La API se publica en el proximo preview
```

#### Ejercicio

1. Busca en el historial de PRs de dotnet/runtime un PR mergeado recientemente que agregue una nueva API publica. Lee la descripcion del PR, los resultados de los checks de CI y los comentarios de review. Nota como interactuan el reviewer y el contribuidor.
2. Abri los proyectos `ref/` y `src/` de una biblioteca uno al lado del otro. Agrega un metodo falso a `src/` e intenta compilar. Observa el error de ApiCompat. Despues agrega la entrada correspondiente a `ref/` y compila de nuevo.
3. Escribi una checklist para tus futuras contribuciones de API. Incluye cada paso desde la propuesta hasta el merge, y los comandos que necesitas en cada etapa.

---

## Resumen

Agregar una nueva API publica a la BCL de .NET es una de las contribuciones mas impactantes que podes hacer. Tambien es una de las mas estructuradas: el proceso esta disenado para asegurar que cada API este bien disenada, bien implementada, bien testeada y sea permanente.

Los pasos clave:
1. **Propuesta**: Crea un issue con codigo de escenarios, forma de la API y justificacion de diseno
2. **Review**: FXDC evalua y aprueba la superficie de la API
3. **Ref assembly**: Actualiza `ref/` con la superficie publica usando cuerpos `throw null;`
4. **Implementacion**: Escribi la logica en `src/` siguiendo convenciones de codigo y requisitos de XML doc
5. **Tests**: Cubre camino feliz, edge cases, fronteras y escenarios negativos en `tests/`
6. **PR**: Envia con CI pasando, enlaza al issue aprobado, atende feedback de review

Los archivos que importan:
- `src/libraries/<Biblioteca>/ref/<Biblioteca>.cs` -- el contrato de la API publica
- `src/libraries/<Biblioteca>/src/` -- la implementacion
- `src/libraries/<Biblioteca>/tests/` -- la suite de tests
- `docs/project/api-review-process.md` -- la documentacion del proceso
- `docs/coding-guidelines/` -- todas las convenciones de codigo

---

## Lectura Adicional

- [API Review Process](https://github.com/dotnet/runtime/blob/main/docs/project/api-review-process.md) -- documentacion oficial del proceso
- [Framework Design Guidelines (libro)](https://amazon.com/dp/0135896460) -- la referencia canonica sobre diseno de APIs
- [Framework Design Guidelines Digest](https://github.com/dotnet/runtime/blob/main/docs/coding-guidelines/framework-design-guidelines-digest.md) -- version condensada del libro
- [Adding API Guidelines](https://github.com/dotnet/runtime/blob/main/docs/coding-guidelines/adding-api-guidelines.md) -- guia de implementacion paso a paso
- [Updating Ref Source](https://github.com/dotnet/runtime/blob/main/docs/coding-guidelines/updating-ref-source.md) -- proceso de actualizacion del ref assembly
- [API Review Notes](https://github.com/dotnet/apireviews) -- notas de review publicadas y decisiones
- [API Review Schedule](https://apireview.net/schedule) -- horarios semanales de reuniones de review

---

## Glosario

| Termino | Definicion |
|---------|-----------|
| **API proposal** | Un issue de GitHub describiendo una nueva API, siguiendo la plantilla de dotnet/runtime |
| **FXDC** | Framework Design Core -- el comite de API review |
| **api-suggestion** | Etiqueta para nuevas propuestas de API esperando triaje |
| **api-ready-for-review** | Etiqueta indicando que una propuesta esta lista para review formal de FXDC |
| **api-approved** | Etiqueta indicando que FXDC aprobo la forma de la API |
| **api-needs-work** | Etiqueta indicando que la propuesta necesita revisiones antes de la aprobacion |
| **Ref assembly** | Un assembly que contiene solo la superficie publica de la API, sin implementacion |
| **`throw null;`** | El patron de cuerpo placeholder usado en implementaciones de metodos de ref assemblies |
| **GenAPI** | Herramienta que genera codigo fuente de ref assembly desde una implementacion compilada |
| **ApiCompat** | Validador en tiempo de build que asegura que los assemblies ref y src concuerden en la superficie publica |
| **Area owner** | El miembro del equipo de dotnet/runtime responsable de un namespace o area de funcionalidad especifica |
| **BCL** | Base Class Library -- el conjunto de bibliotecas fundamentales de .NET |
| **TFM** | Target Framework Moniker -- e.g., `net9.0`, representado por `$(NetCoreAppCurrent)` en MSBuild |
| **`$(NetCoreAppCurrent)`** | Propiedad de MSBuild para el TFM de la version actual de .NET -- nunca hardcodees la version |
