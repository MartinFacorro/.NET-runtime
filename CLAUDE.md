# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

This is the **dotnet/runtime** repository - the .NET runtime, libraries, and shared host. It contains three major components:

- **CoreCLR** (`src/coreclr/`): Primary .NET execution engine (JIT/RyuJIT, GC, type system). C/C++.
- **Mono** (`src/mono/`): Lightweight alternative runtime for mobile, WebAssembly, constrained environments.
- **Libraries** (`src/libraries/`): Managed class libraries (BCL) - `System.Collections`, `System.Net.Http`, `System.Text.Json`, etc.
- **Host** (`src/native/corehost/`, `src/installer/`): Native `dotnet` executable bootstrapping the runtime.

**System.Private.CoreLib** spans multiple directories: runtime-agnostic code in `src/libraries/System.Private.CoreLib/`, with runtime-specific counterparts in `src/coreclr/System.Private.CoreLib/` and `src/mono/System.Private.CoreLib/`. It must be built in the same configuration as its runtime.

## Build Commands

**Windows uses `build.cmd`; Linux/macOS uses `build.sh`.** All commands below use `build.sh` - substitute accordingly.

### Baseline build (MUST run before making changes)

| Component | Baseline Command |
|-----------|-----------------|
| CoreCLR | `./build.sh clr+libs+host` |
| Mono | `./build.sh mono+libs` |
| Libraries | `./build.sh clr+libs -rc release` |
| WASM Libraries | `./build.sh mono+libs -os browser` |
| Host | `./build.sh clr+libs+host -rc release -lc release` |
| System.Private.CoreLib | `./build.sh clr+libs -rc checked` |
| Runtime Tests | `./build.sh clr+libs -lc release -rc checked` |

The baseline build can take up to 40 minutes. After building, configure the SDK:
```bash
export PATH="$(pwd)/.dotnet:$PATH"
```

### Building and testing a specific library

```bash
cd src/libraries/<LibraryName>
dotnet build
dotnet build /t:test ./tests/<TestProject>.csproj
```

### Rebuilding System.Private.CoreLib after changes

```bash
./build.sh clr.corelib+clr.nativecorelib+libs.pretest -rc checked
```

### CoreCLR runtime tests

```bash
# Build and run all
cd src/tests && ./build.sh && ./run.sh

# Build a single test project (use -priority1 for priority > 0 tests)
src/tests/build.sh -Test tracing/eventpipe/eventsvalidation/GCEvents.csproj x64 Release -priority1

# Generate Core_Root layout (required before running individual tests)
src/tests/build.sh -GenerateLayoutOnly x64 Release

# Run a single test (exit code 100 = pass)
export CORE_ROOT=$(pwd)/artifacts/tests/coreclr/<os>.x64.Release/Tests/Core_Root
cd artifacts/tests/coreclr/<os>.x64.Release/<test-path>/
$CORE_ROOT/corerun <TestName>.dll
```

### Configuration flags

- `-rc` / `-runtimeConfiguration`: CoreCLR config (Debug/Checked/Release)
- `-lc` / `-librariesConfiguration`: Libraries config
- `-hc` / `-hostConfiguration`: Host config
- `-c` / `-configuration`: Default for all unqualified subsets

### Disabling warnings-as-errors during development

```bash
export TreatWarningsAsErrors=false
```

## Library Project Layout

Each library under `src/libraries/<LibraryName>/` follows:

| Directory | Purpose |
|-----------|---------|
| `ref/` | Reference assembly (public API surface). Must be updated when adding public APIs. |
| `src/` | Implementation source code |
| `tests/` | Test projects |
| `gen/` | Source generators (if any) |

## Code Style and Conventions

Follow `.editorconfig` at repo root. Key rules:

- **Allman-style braces** (each brace on its own line). Four spaces, no tabs.
- **Field naming**: `_camelCase` for private/internal instance fields, `s_` prefix for static, `t_` prefix for `[ThreadStatic]`. Public fields use PascalCase.
- **Avoid `this.`** unless absolutely necessary.
- **Always specify visibility** as the first modifier.
- **Use language keywords** over BCL types (`int` not `Int32`).
- **`var`** only when type is explicit on the right-hand side (`var x = new Foo(...)` is OK, `var x = GetFoo()` is not).
- **Constants**: PascalCase for all constant fields and locals.
- **Null checks**: Use `is null` / `is not null`, never `== null` / `!= null`.
- **Use `nameof`** instead of string literals for member names.
- **Pattern matching and switch expressions** wherever possible.
- **`LibraryImport`** (source-generated) over `DllImport` for new P/Invoke declarations.
- **Non-public helpers**: prefer `static` for stateless, `sealed` when no inheritance needed.
- **File-scoped namespaces** and single-line using directives preferred.
- **Final return** of a method should be on its own line.
- Use `ObjectDisposedException.ThrowIf` where applicable.
- Use `?.` (null-conditional) when applicable.

### File header (required for new files)

```csharp
// Licensed to the .NET Foundation under one or more agreements.
// The .NET Foundation licenses this file to you under the MIT license.
```

### Platform-specific code (in order of preference)

1. Partial classes with platform-specific files: `Stream.Unix.cs`, `Stream.Windows.cs`
2. Entirely separate files when the whole class differs
3. `#if` directives as last resort

### Native interop pattern

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

Interop files live under `src/libraries/Common/src/Interop/`, organized by platform and library.

## Testing Conventions

- Prefer `[Theory]` with `[InlineData]`/`[MemberData]` over multiple duplicate `[Fact]` methods.
- Add new tests to existing test files rather than creating new files.
- When adding test files, match the directory convention of sibling tests (flat files vs per-test subdirectories).
- Do not emit "Act", "Arrange", or "Assert" comments.
- Do not add regression comments citing GitHub issue/PR numbers unless explicitly asked.
- Ensure new files are listed in the `.csproj` if other files in that folder are.
- When running tests, use filters and check run counts to confirm tests actually executed.

## MSBuild Properties

- Use `$(NetCoreAppCurrent)` for target framework - never hardcode TFM versions.
- Use `$(NetFrameworkMinimum)` for .NET Framework targets.
- Avoid `TargetFramework` conditions in the first `PropertyGroup` (causes DesignTimeBuild issues).
- Use `TargetFrameworkIdentifier` to condition on framework family, `TargetPlatformIdentifier` for OS-specific conditions.

## Important Correctness Warnings

- **Public API changes** require updating both `ref/` and `src/` consistently.
- **Shared/conditioned files**: Changes in `src/libraries/` may affect multiple TFMs or OSes via `Condition` attributes.
- **Don't hand-edit generated code** (`.g.cs` files, `gen/` output).
- **CoreLib cross-runtime**: When editing shared CoreLib code, check for runtime-specific counterparts in both `src/coreclr/` and `src/mono/`.

## Key Documentation Paths

- Build workflows: `docs/workflow/building/`
- Test workflows: `docs/workflow/testing/`
- Coding guidelines: `docs/coding-guidelines/`
- API review process: `docs/project/api-review-process.md`
- XML doc guidelines: `.github/prompts/docs.prompt.md`
