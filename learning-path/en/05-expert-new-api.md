# Level 5: Expert / Contributor — Adding a New BCL API

> **Target profile:** Developer ready to contribute a new public API to the .NET Base Class Libraries
> **Estimated effort:** 8 hours
> **Prerequisites:** Level 3, [Module 5.1](05-expert-building.md), [Module 5.2](05-expert-testing.md)
> [Version en espanol](../es/05-expert-new-api.md)

---

## Learning Objectives

By the end of this module you will be able to:

1. Navigate the full API lifecycle from initial proposal through FXDC review, implementation, testing, and merge.
2. Write an effective API proposal issue using the dotnet/runtime template, with scenario code and design rationale.
3. Update a reference assembly (`ref/` project) following the `throw null;` pattern and explain why ref assemblies exist.
4. Implement a new API in the `src/` project following dotnet/runtime coding conventions, including XML doc comments and platform-specific patterns.
5. Write comprehensive tests using `[Theory]`/`[InlineData]` patterns, covering edge cases, null inputs, and negative scenarios.
6. Navigate the PR workflow including CI checks, API compatibility validation, code review, and merge requirements.

---

## Concept Map

```mermaid
graph LR
    classDef proposal fill:#264653,stroke:#264653,color:#ffffff
    classDef review fill:#2a9d8f,stroke:#2a9d8f,color:#ffffff
    classDef impl fill:#e9c46a,stroke:#e9c46a,color:#000000
    classDef test fill:#f4a261,stroke:#f4a261,color:#000000
    classDef merge fill:#e76f51,stroke:#e76f51,color:#ffffff

    IDEA["Idea / Need"]:::proposal
    PROPOSAL["API Proposal<br/>(GitHub Issue)"]:::proposal
    DISCUSSION["Discussion<br/>with Area Owner"]:::review
    FXDC["FXDC Review<br/>(api-ready-for-review)"]:::review
    APPROVED["api-approved"]:::review
    REF["Update ref/<br/>(Reference Assembly)"]:::impl
    SRC["Implement in src/<br/>(Source Assembly)"]:::impl
    TEST["Write Tests<br/>(tests/ project)"]:::test
    PR["Pull Request"]:::merge
    CI["CI + ApiCompat"]:::merge
    MERGE["Merge"]:::merge

    IDEA --> PROPOSAL --> DISCUSSION --> FXDC --> APPROVED --> REF --> SRC --> TEST --> PR --> CI --> MERGE
```

---

## Source Reading Guide

| Difficulty | File / Path | Purpose |
|------------|-------------|---------|
| ★★ | `docs/project/api-review-process.md` | The official API review process documentation |
| ★★ | `docs/coding-guidelines/adding-api-guidelines.md` | Step-by-step guide for adding APIs to the repo |
| ★★ | `docs/coding-guidelines/updating-ref-source.md` | How to update the reference assembly after adding an API |
| ★★ | `docs/coding-guidelines/coding-style.md` | C# coding style rules enforced across the repo |
| ★★ | `docs/coding-guidelines/framework-design-guidelines-digest.md` | Digest of the Framework Design Guidelines book |
| ★★ | `src/libraries/System.Collections/ref/System.Collections.cs` | Example reference assembly -- notice `throw null;` pattern |
| ★★ | `src/libraries/System.Collections/ref/System.Collections.csproj` | Example ref project -- uses `$(NetCoreAppCurrent)` |
| ★★ | `src/libraries/System.Collections/src/System.Collections.csproj` | Example src project -- implementation assembly |
| ★★★ | `src/libraries/System.Collections/src/System/Collections/Generic/LinkedList.cs` | Example implementation with XML docs |
| ★★ | `src/libraries/System.Collections/tests/System.Collections.Tests.csproj` | Example test project structure |
| ★★ | `.editorconfig` | Auto-formatting rules for the codebase |

---

## Curriculum

### Lesson 1 — The API Review Process

#### What you'll learn

Every public API added to dotnet/runtime goes through a formal review process. This is not bureaucracy -- it is one of the most important quality gates in the .NET ecosystem. APIs are effectively permanent once shipped: removing or changing them would break millions of applications. Understanding this process is the first step toward a successful contribution.

#### The review board: FXDC

The Framework Design Core (FXDC) is the group that conducts API reviews. Reviews happen weekly, typically on Tuesdays from 10am to 12pm Pacific Time, and are streamed live on the .NET Foundation YouTube channel. After each review, notes are published in the [dotnet/apireviews](https://github.com/dotnet/apireviews) repository.

#### The lifecycle of an API

Open `docs/project/api-review-process.md`. The process has five stages:

**1. File an issue.** Use the [API Proposal template](https://github.com/dotnet/runtime/issues/new?assignees=&labels=api-suggestion&template=02_api_proposal.yml&title=%5BAPI+Proposal%5D%3A+). The issue gets the `api-suggestion` label. Your proposal should include:
- A clear description of the problem you are solving
- Scenario code showing how the API would be used
- A sketch of the proposed API surface (not necessarily final)

**2. An owner is assigned.** The [area owner](https://github.com/dotnet/runtime/blob/main/docs/project/issue-guide.md#areas) for the relevant namespace sponsors the issue and drives the discussion.

**3. Discussion.** The owner and community refine the proposal. The goal is to reach a concrete, actionable design. If changes are needed, the label `api-needs-work` is applied. You should edit the original issue description (not add new comments) to keep the proposal current. A small "Updates:" changelog at the bottom helps reviewers track what changed.

**4. Owner decision.** The owner either:
- Labels the issue `api-ready-for-review` (moves to formal review)
- Closes the issue as not actionable, won't fix, or won't fix as proposed

**5. FXDC review.** The review board examines the proposal and either:
- **Approves** it (label changes to `api-approved`)
- Sends it back with **needs work** (`api-needs-work`)
- **Rejects** it (issue is closed)

#### The golden rule

From the review process doc: "Pull requests against dotnet/runtime shouldn't be submitted before getting approval." Do not submit a PR with your implementation before the API is approved. Work in a branch in your own fork if you want to prototype.

#### Three ways to get reviewed

1. **Backlog**: File the issue with `api-ready-for-review`. Reviews are processed oldest-first.
2. **Fast track**: Add both `api-ready-for-review` and `blocking` labels. Blocking issues are reviewed before the backlog.
3. **Dedicated review**: For area owners only, when the proposal needs 60+ minutes.

#### Exercise

1. Browse [https://aka.ms/ready-for-api-review](https://aka.ms/ready-for-api-review) and read two or three approved proposals. Notice the format: problem statement, scenario code, API shape.
2. Read the review notes for an approved API in [dotnet/apireviews](https://github.com/dotnet/apireviews). Pay attention to the kinds of feedback FXDC gives: naming choices, overload design, null handling, consistency with existing APIs.
3. Open `docs/coding-guidelines/framework-design-guidelines-digest.md` and read the naming conventions section. Identify three rules that would apply to your next API proposal.

---

### Lesson 2 — Writing the API Proposal

#### What you'll learn

A good proposal is the difference between an API that sails through review and one that goes through multiple rounds of revision. This lesson covers how to write a proposal that FXDC can evaluate efficiently.

#### The proposal template

The template asks for several sections. Here is how to fill each one effectively:

**Background and motivation**: Explain the problem. Why does the current API surface not solve it? Link to Stack Overflow questions, GitHub issues, or real-world code that shows the pain point. Reviewers are more likely to approve an API when they can see concrete evidence of developer need.

**API proposal**: Provide a C# code block showing the public surface. Use the condensed format FXDC expects:

```csharp
namespace System.Collections.Generic;

public class List<T>
{
    // Existing members omitted for brevity

    // NEW: proposed API
    public int RemoveAll(ReadOnlySpan<T> items);
}
```

Include only the new members. Mark them clearly with comments. Specify parameter names, return types, generic constraints, and any attributes. The shape does not need to be final -- FXDC will refine it -- but it must be concrete enough to evaluate.

**Usage examples**: Show 2-3 realistic scenarios. These are the most important part of the proposal. Reviewers use them to evaluate usability:

```csharp
// Scenario 1: Remove multiple known-bad items from a list
List<string> names = GetNames();
ReadOnlySpan<string> blocked = ["spam", "test", "admin"];
int removed = names.RemoveAll(blocked);
Console.WriteLine($"Removed {removed} blocked names");
```

**Alternative designs**: Show that you considered other approaches and explain why you chose this one. This demonstrates design maturity and preempts common review feedback.

**Risks**: Identify breaking changes, performance implications, or platform-specific concerns.

#### Naming conventions

Open `docs/coding-guidelines/framework-design-guidelines-digest.md`. The key naming rules:

- **PascalCase** for all public identifiers (types, methods, properties, events, constants)
- **camelCase** for parameters
- Prefix type parameters with `T` (e.g., `TKey`, `TValue`)
- Use descriptive names: `GetOrAdd` not `GetAdd`, `TryParse` not `Parse2`
- Avoid abbreviations: `GetConnection` not `GetConn`
- Boolean properties/methods read as questions: `IsEmpty`, `HasValue`, `CanSeek`
- Async methods end with `Async`: `ReadAsync`, `WriteAsync`
- Factory methods use `Create*`: `CreateInstance`, `CreateScope`
- `Try*` pattern for methods that can fail without exceptions: returns `bool`, value via `out` parameter

#### Design considerations

These are the questions FXDC will ask. Prepare answers:

1. **Does this belong in the BCL?** Or should it be a NuGet package? The BCL should contain primitives that many libraries build on. If only one application domain needs it, it probably belongs elsewhere.
2. **Is the naming consistent?** Look at peer APIs in the same type or namespace. If `List<T>` has `AddRange(IEnumerable<T>)`, a new batch-add method should follow the same naming pattern.
3. **What about null?** Should the method accept `null` inputs? Throw `ArgumentNullException`? Use nullable annotations?
4. **What about empty inputs?** Should `RemoveAll(ReadOnlySpan<T>.Empty)` be a no-op or throw?
5. **Performance implications?** Will this cause allocations? Is there a `Span`-based overload?
6. **Overload design?** Provide the most common case first, with additional overloads for advanced scenarios. Follow the "progressive disclosure" principle.

#### Exercise

1. Pick an API you wish existed in the BCL. Write a proposal following the template: background, API surface, 2 usage examples, 1 alternative design, and risks.
2. Review your proposal against the naming guidelines digest. Does every name follow the conventions?
3. Search the dotnet/runtime issues for `label:api-approved` and find an API similar to yours. Compare the proposal format and level of detail.

---

### Lesson 3 — Updating the Reference Assembly

#### What you'll learn

Once an API is approved, the first code change is updating the reference assembly. This lesson explains what reference assemblies are, why they exist, and how to update them correctly.

#### What is a reference assembly?

A reference assembly contains only the public API surface of a library -- type signatures, method signatures, and attributes -- but no implementation. Method bodies are replaced with `throw null;`. The compiler uses reference assemblies during compilation to verify that your code uses the correct types and members, but the actual behavior comes from the implementation assembly at runtime.

Open `src/libraries/System.Collections/ref/System.Collections.cs` and look at the pattern:

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
        // ... every public member listed with throw null; bodies
    }
}
```

Notice several things:
- **Every public member** is listed, with its full signature
- Methods that return a value use `throw null;` as the body
- Methods that return `void` have empty bodies `{ }`
- Properties use `get { throw null; }` and `set { }` patterns
- The file header includes the API review process comment
- Types are `partial` (the implementation is in `src/`)

#### Why reference assemblies exist

1. **Compilation speed**: Compiling against a reference assembly is faster because the compiler does not need to parse implementation code.
2. **API surface contract**: The reference assembly is the definitive record of the public API. If it is not in `ref/`, it is not part of the public API, even if a `public` member exists in `src/`.
3. **Binary compatibility**: The API compatibility tooling (`ApiCompat`) compares reference assemblies across versions to detect breaking changes.
4. **Decoupling**: The implementation assembly's identity may differ from the reference assembly. Types can be moved between implementation assemblies without breaking the compile-time contract.

#### The ref project structure

Open `src/libraries/System.Collections/ref/System.Collections.csproj`:

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

Key points:
- Uses `$(NetCoreAppCurrent)` -- never hardcode the TFM version
- References only other `ref/` projects, never `src/` projects
- The `.Forwards.cs` file contains type forwards for types that live in a different implementation assembly

#### How to update the ref assembly

Follow the process from `docs/coding-guidelines/updating-ref-source.md`:

**Step 1: Implement first.** Write the API in `src/` and build the source assembly. If you are adding a new public type, the build may fail with a `TypeMustExist` error. Work around this by building with ApiCompat assembly validation disabled:

```bash
dotnet build /p:ApiCompatValidateAssemblies=false
```

**Step 2: Use GenAPI to update ref.** From the `src/` directory, run:

```bash
dotnet msbuild /t:GenerateReferenceAssemblySource
```

GenAPI generates the reference assembly source from the built implementation assembly. However, it may produce mismatches that need manual fixups. The tool is the preferred way to update ref assemblies because it keeps them consistent with the actual build output.

**Step 3: Build the ref assembly.** Navigate to the `ref/` directory and build:

```bash
cd src/libraries/<LibraryName>/ref
dotnet build
```

**Step 4: Verify.** The ref assembly must compile and the API surface must match what was approved.

#### Manual editing rules

If you must edit the ref source manually:

- Use fully qualified type names in attributes
- Keep members in the same order GenAPI produces (alphabetical within groups)
- Return types and parameter types must match the implementation exactly
- Nullable annotations must match
- Default parameter values must match
- Generic constraints must match
- Never add implementation logic -- always `throw null;` or `{ }`

#### For System.Runtime specifically

If your API touches types in `System.Private.CoreLib` (which surfaces through `System.Runtime`), the process is slightly different:

```bash
# From System.Runtime/src directory:
dotnet build --no-incremental /t:GenerateReferenceAssemblySource
```

Then filter out unrelated changes from the generated output. This is necessary because `System.Runtime` exposes a very large surface that aggregates many underlying assemblies.

#### Exercise

1. Open `src/libraries/System.Collections/ref/System.Collections.cs`. Find a method that returns `void` and one that returns a value. Verify the `{ }` vs `throw null;` pattern.
2. Open the ref `.csproj` and the src `.csproj` side by side. Notice that ref references other ref projects, while src references `$(CoreLibProject)`.
3. Read `docs/coding-guidelines/updating-ref-source.md`. Identify the three different paths depending on your library type (regular, System.Runtime, full facade).

---

### Lesson 4 — Implementing the API

#### What you'll learn

With the reference assembly updated and building, you now implement the actual logic in the `src/` project. This lesson covers the coding conventions, documentation requirements, and platform-specific patterns used in dotnet/runtime.

#### Implementation project structure

Open `src/libraries/System.Collections/src/System.Collections.csproj`. The implementation project:

```xml
<Project Sdk="Microsoft.NET.Sdk">
  <PropertyGroup>
    <TargetFramework>$(NetCoreAppCurrent)</TargetFramework>
    <AllowUnsafeBlocks>true</AllowUnsafeBlocks>
  </PropertyGroup>
  <ItemGroup>
    <Compile Include="System\Collections\Generic\LinkedList.cs" />
    <!-- More source files -->
    <Compile Include="$(CoreLibSharedDir)System\Collections\HashHelpers.cs"
             Link="Common\System\Collections\HashHelpers.cs" />
    <!-- Shared common files -->
  </ItemGroup>
  <ItemGroup>
    <ProjectReference Include="$(CoreLibProject)" />
  </ItemGroup>
</Project>
```

Key observations:
- Source files live under the namespace directory structure: `System/Collections/Generic/LinkedList.cs`
- Shared files from `CoreLibSharedDir` and `CommonPath` are linked via the `Link` attribute
- The project references `$(CoreLibProject)`, not other `src/` projects directly

#### Coding conventions

Open `docs/coding-guidelines/coding-style.md` and `.editorconfig`. The most critical rules:

**Braces and formatting**:
- Allman-style braces: every brace on its own line
- Four spaces indentation, no tabs
- No more than one blank line between members

**Naming**:
- `_camelCase` for private/internal instance fields
- `s_` prefix for static fields, `t_` prefix for `[ThreadStatic]` fields
- PascalCase for constants, methods, properties, types
- Visibility is always the first modifier: `public static` not `static public`

**Type usage**:
- Language keywords over BCL types: `int` not `Int32`, `string` not `String`
- `var` only when the type is explicit on the right: `var list = new List<int>()` is OK, `var result = GetResult()` is not
- `nameof(parameter)` instead of `"parameter"` in exception messages
- `is null` / `is not null` instead of `== null` / `!= null`

**Modern patterns**:
- Use `LibraryImport` (source-generated) over `DllImport` for new P/Invoke declarations
- Prefer `static` for stateless helpers, `sealed` when no inheritance is needed
- File-scoped namespaces preferred for new files
- Use `ObjectDisposedException.ThrowIf` where applicable

#### File header

Every new file must start with:

```csharp
// Licensed to the .NET Foundation under one or more agreements.
// The .NET Foundation licenses this file to you under the MIT license.
```

#### XML documentation

New public APIs must have triple-slash XML doc comments. The comments are the source of truth for IntelliSense and the official API documentation at learn.microsoft.com.

```csharp
/// <summary>
/// Removes all elements that match any of the specified items.
/// </summary>
/// <param name="items">The items to remove from the list.</param>
/// <returns>The number of elements removed from the list.</returns>
/// <exception cref="ArgumentNullException">
/// <paramref name="items"/> is <see langword="null"/>.
/// </exception>
public int RemoveAll(ReadOnlySpan<T> items)
{
    // implementation
}
```

From `docs/coding-guidelines/adding-api-guidelines.md`: "If your new API or the APIs it calls throw any exceptions, those need to be manually documented by adding the `<exception></exception>` elements."

#### Argument validation

The runtime follows a strict pattern for argument validation:

```csharp
public void Insert(int index, T item)
{
    ArgumentOutOfRangeException.ThrowIfNegative(index);
    ArgumentOutOfRangeException.ThrowIfGreaterThan(index, _size);

    // implementation
}
```

For null checks:

```csharp
public void AddRange(IEnumerable<T> collection)
{
    ArgumentNullException.ThrowIfNull(collection);

    // implementation
}
```

Use the `ThrowHelper` pattern for hot paths where you want to keep the throw logic out of the inlined method body.

#### Platform-specific code

From `CLAUDE.md`, in order of preference:

1. **Partial classes with platform-specific files**: `Stream.Unix.cs`, `Stream.Windows.cs`
2. **Entirely separate files** when the whole class differs by platform
3. **`#if` directives** as a last resort

Platform conditions in MSBuild use `TargetPlatformIdentifier`:

```xml
<Compile Include="MyType.Windows.cs" Condition="'$(TargetPlatformIdentifier)' == 'windows'" />
<Compile Include="MyType.Unix.cs" Condition="'$(TargetPlatformIdentifier)' != 'windows'" />
```

#### Native interop pattern

If your API requires native calls:

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

Interop files live under `src/libraries/Common/src/Interop/`, organized by platform (e.g., `Interop/Windows/Kernel32/`, `Interop/Unix/System.Native/`).

#### Building your implementation

```bash
cd src/libraries/<LibraryName>
dotnet build
```

If building fails with ApiCompat errors because the ref assembly has not been updated yet, use:

```bash
dotnet build /p:ApiCompatValidateAssemblies=false
```

To disable warnings-as-errors during development:

```bash
export TreatWarningsAsErrors=false
```

#### Exercise

1. Open `src/libraries/System.Collections/src/System/Collections/Generic/LinkedList.cs`. Study the coding style: braces, naming, field conventions, XML doc comments.
2. Find an example of argument validation using `ArgumentNullException.ThrowIfNull` in the `src/libraries/` tree. Note how validation happens at the top of the method before any logic.
3. Look at a library that has platform-specific files (e.g., `src/libraries/System.IO.FileSystem/src/`). Find the `.Windows.cs` and `.Unix.cs` partial class split pattern.

---

### Lesson 5 — Writing Tests

#### What you'll learn

Tests are not optional in dotnet/runtime. Every new API must have comprehensive tests before the PR can be merged. This lesson covers the test project structure, conventions, and patterns used across the BCL.

#### Test project structure

Each library has a `tests/` directory. Open `src/libraries/System.Collections/tests/`:

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

Key conventions:
- Tests are organized by type within subdirectories (e.g., `Generic/List/`)
- Test files match the feature they test: `List.Generic.Tests.AddRange.cs` tests `AddRange`
- A main test class file covers the core type: `List.Generic.Tests.cs`
- The test `.csproj` references `$(NetCoreAppCurrent)`
- Add new tests to existing test files rather than creating new files when possible

#### Test patterns: Theory and InlineData

The repo strongly prefers `[Theory]` with data-driven tests over multiple individual `[Fact]` methods:

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

Use `[MemberData]` for more complex data sets:

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

#### What to test

For every new API, cover these categories:

**1. Happy path**: The normal, expected usage scenarios.

**2. Edge cases**:
- Empty collections / empty input
- Single-element collections
- Very large collections
- Duplicate values
- Default values (`default(T)`, `null` for reference types)

**3. Boundary conditions**:
- `int.MaxValue`, `int.MinValue` for integer parameters
- Zero-length spans
- Index at start/end of collection

**4. Negative tests** (expected failures):
- `null` arguments should throw `ArgumentNullException`
- Out-of-range indices should throw `ArgumentOutOfRangeException`
- Operations on disposed objects should throw `ObjectDisposedException`
- Invalid enum values, negative counts, etc.

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

**5. Concurrency tests** (if the API has thread-safety implications):
- Concurrent reads should not corrupt state
- Concurrent writes should throw or be documented as safe

#### Test conventions specific to dotnet/runtime

From `CLAUDE.md` and observed patterns:

- Do **not** emit "Arrange", "Act", "Assert" comments
- Do **not** add regression comments citing GitHub issue/PR numbers unless explicitly asked
- Ensure new files are listed in the `.csproj` if other files in that folder are
- When running tests, use filters and check run counts to confirm tests actually executed
- The test class file is usually `abstract partial` with a generic type parameter, allowing concrete test classes per element type

#### Running tests

```bash
cd src/libraries/<LibraryName>
dotnet build /t:Test ./tests/<TestProject>.csproj
```

To run specific tests:

```bash
dotnet test ./tests/<TestProject>.csproj --filter "FullyQualifiedName~RemoveAll"
```

Check the run count in the output to confirm your tests actually ran. A common mistake is writing tests that compile but are not discovered by the test runner due to missing attributes or incorrect class visibility.

#### Exercise

1. Open `src/libraries/System.Collections/tests/Generic/List/List.Generic.Tests.cs`. Study the abstract test class pattern. Notice how it inherits from `IList_Generic_Tests<T>` and provides factory methods.
2. Find a test file that uses `[Theory]` with `[MemberData]`. Count the number of test cases and note how edge cases are covered.
3. Write a test plan (not code) for a hypothetical `List<T>.RemoveAll(ReadOnlySpan<T>)` method. List at least 10 distinct test scenarios covering happy path, edge cases, boundaries, and negative tests.

---

### Lesson 6 — The PR Workflow

#### What you'll learn

With the ref assembly updated, implementation written, and tests passing, you are ready to submit a pull request. This lesson covers the CI pipeline, API compatibility checks, the review process, and what to expect before merge.

#### Before submitting the PR

1. **Confirm the API is approved.** The issue should have the `api-approved` label. Never submit an implementation PR for an unapproved API.

2. **Run tests locally.** Build and test the full library:

```bash
cd src/libraries/<LibraryName>
dotnet build
dotnet build /t:Test ./tests/<TestProject>.csproj
```

3. **Run formatting checks.** The repo uses `dotnet-format`:

```bash
dotnet format --verify-no-changes
```

4. **Check API compatibility locally.** Build both ref and src assemblies -- the build system runs ApiCompat automatically. If it fails, the ref assembly does not match the implementation.

#### Creating the PR

A good PR for a new API includes:

**Title**: Concise, descriptive, under 70 characters. Example: `Add List<T>.RemoveAll(ReadOnlySpan<T>)`

**Description**:
- Link to the approved API issue: `Fixes #12345`
- Brief summary of the implementation approach
- Any design decisions or tradeoffs
- Test summary: what scenarios are covered

**Commit hygiene**:
- One logical commit per change, or squash before submitting
- The commit message should reference the issue

#### CI checks

When you submit a PR, the CI pipeline runs automatically. The key checks:

**1. Build validation**: The entire repo builds (debug and release configurations, multiple platforms).

**2. ApiCompat**: Validates that:
- The reference assembly matches the implementation assembly
- No existing APIs were accidentally removed or modified
- No breaking changes were introduced

If ApiCompat fails with a `TypeMustExist` or `MemberMustExist` error, your ref assembly is missing something the implementation exposes. If it fails with a `CannotRemove*` error, you accidentally removed an existing API.

**3. Test runs**: Tests run across multiple platforms (Windows, Linux, macOS) and configurations (Debug, Release). Your tests must pass everywhere.

**4. Formatting**: Code style is checked against `.editorconfig`. Violations fail the build.

**5. API diff**: For PRs that modify `ref/` assemblies, a diff is generated showing the API surface change. Reviewers use this to verify the change matches the approved API shape.

#### ApiCompat in detail

ApiCompat is the automated API compatibility validator. It works by comparing:
- The ref assembly against the implementation assembly (they must agree on the public surface)
- The current ref assembly against the previous version's ref assembly (no breaking changes)

From `docs/coding-guidelines/updating-ref-source.md`: when adding a new type, the build may fail because ApiCompat checks run before the ref assembly is updated. The workaround:

```bash
dotnet build /p:ApiCompatValidateAssemblies=false
```

Use this **only** during development. The final PR must pass ApiCompat without this flag.

Common ApiCompat errors and fixes:

| Error | Cause | Fix |
|-------|-------|-----|
| `TypeMustExist` | New type in src/ but missing in ref/ | Add the type to the ref assembly |
| `MemberMustExist` | New member in src/ but missing in ref/ | Add the member to the ref assembly |
| `CannotRemoveType` | Type was removed from ref/ | Restore the type (this would be a breaking change) |
| `CannotChangeVisibility` | Visibility mismatch between ref/ and src/ | Make them match |

#### Code review

Your PR will be reviewed by:
- **The area owner**: Ensures the implementation matches the approved API design
- **Community reviewers**: May provide additional feedback on style, performance, or correctness
- **CI bots**: Automated checks must all pass

Common review feedback on new API PRs:
- **Missing edge case tests**: Reviewers will ask for null tests, empty input tests, boundary tests
- **XML doc quality**: Comments must be accurate and complete
- **Naming consistency**: Parameter names, method names should match established patterns
- **Performance**: Avoid unnecessary allocations in hot paths
- **Thread safety**: Document whether the API is thread-safe or not

#### After merge

Once your PR is merged:
1. The API is part of the next .NET preview release
2. XML doc comments will be synced to [dotnet-api-docs](https://github.com/dotnet/dotnet-api-docs) for the official documentation
3. The API will appear in IntelliSense in Visual Studio and VS Code
4. The API is effectively permanent -- it cannot be removed without a major version bump and a formal deprecation process

#### The complete workflow summary

```
1. Identify need → Write API proposal issue → Label: api-suggestion
2. Discussion with area owner → Refinement → Label: api-ready-for-review
3. FXDC review → Label: api-approved
4. Fork the repo → Create a branch
5. Update ref/ assembly (add new public surface, throw null; bodies)
6. Implement in src/ (coding conventions, XML docs, argument validation)
7. Write tests in tests/ (happy path, edge cases, negative tests)
8. Build and test locally
9. Submit PR → Link to approved issue
10. CI passes → Code review → Address feedback
11. Merge → API ships in next preview
```

#### Exercise

1. Search the dotnet/runtime PR history for a recently merged PR that adds a new public API. Read the PR description, the CI check results, and the review comments. Note how the reviewer and contributor interact.
2. Open a library's `ref/` and `src/` projects side by side. Add a fake method to `src/` and try building. Observe the ApiCompat error. Then add the matching entry to `ref/` and build again.
3. Write a checklist for your own future API contributions. Include each step from proposal to merge, and the commands you need at each stage.

---

## Summary

Adding a new public API to the .NET BCL is one of the most impactful contributions you can make. It is also one of the most structured: the process is designed to ensure that every API is well-designed, well-implemented, well-tested, and permanent.

The key steps:
1. **Proposal**: File an issue with scenario code, API shape, and design rationale
2. **Review**: FXDC evaluates and approves the API surface
3. **Reference assembly**: Update `ref/` with the public surface using `throw null;` bodies
4. **Implementation**: Write the logic in `src/` following coding conventions and XML doc requirements
5. **Tests**: Cover happy path, edge cases, boundaries, and negative scenarios in `tests/`
6. **PR**: Submit with CI passing, link to the approved issue, address review feedback

The files that matter:
- `src/libraries/<Library>/ref/<Library>.cs` -- the public API contract
- `src/libraries/<Library>/src/` -- the implementation
- `src/libraries/<Library>/tests/` -- the test suite
- `docs/project/api-review-process.md` -- the process documentation
- `docs/coding-guidelines/` -- all the coding conventions

---

## Further Reading

- [API Review Process](https://github.com/dotnet/runtime/blob/main/docs/project/api-review-process.md) -- the official process documentation
- [Framework Design Guidelines (book)](https://amazon.com/dp/0135896460) -- the canonical reference on API design
- [Framework Design Guidelines Digest](https://github.com/dotnet/runtime/blob/main/docs/coding-guidelines/framework-design-guidelines-digest.md) -- condensed version of the book
- [Adding API Guidelines](https://github.com/dotnet/runtime/blob/main/docs/coding-guidelines/adding-api-guidelines.md) -- step-by-step implementation guide
- [Updating Ref Source](https://github.com/dotnet/runtime/blob/main/docs/coding-guidelines/updating-ref-source.md) -- reference assembly update process
- [API Review Notes](https://github.com/dotnet/apireviews) -- published review notes and decisions
- [API Review Schedule](https://apireview.net/schedule) -- weekly review meeting times

---

## Glossary

| Term | Definition |
|------|-----------|
| **API proposal** | A GitHub issue describing a new API, following the dotnet/runtime template |
| **FXDC** | Framework Design Core -- the API review board |
| **api-suggestion** | Label for new API proposals awaiting triage |
| **api-ready-for-review** | Label indicating a proposal is ready for formal FXDC review |
| **api-approved** | Label indicating FXDC has approved the API shape |
| **api-needs-work** | Label indicating the proposal needs revisions before approval |
| **Reference assembly** | An assembly containing only the public API surface, no implementation |
| **`throw null;`** | The placeholder body pattern used in reference assembly method implementations |
| **GenAPI** | Tool that generates reference assembly source from a built implementation |
| **ApiCompat** | Build-time validator that ensures ref and src assemblies agree on the public surface |
| **Area owner** | The dotnet/runtime team member responsible for a specific namespace or feature area |
| **BCL** | Base Class Library -- the set of fundamental .NET libraries |
| **TFM** | Target Framework Moniker -- e.g., `net9.0`, represented by `$(NetCoreAppCurrent)` in MSBuild |
| **`$(NetCoreAppCurrent)`** | MSBuild property for the current .NET version TFM -- never hardcode the version |
