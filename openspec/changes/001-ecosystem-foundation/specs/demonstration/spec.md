# demonstration

## ADDED Requirements

### Requirement: One visual application per level, exercising only its own layer

Every published package SHALL be demonstrated by a visual application under
`samples/` in the owning repository. An application SHALL NOT reference any package
situated above the layer it demonstrates.

Coverage is per package, and the exemptions below are the only ones. The eight-layer
map of `ecosystem-architecture` governs what an application may **reference**; it
does not group applications. `Rest` and `Navigation` share layer 4 and are
demonstrated separately, because what each proves is different.

| Exempt package | Why |
|---|---|
| `Rinzler78.Build` | A props/targets package; its demonstration is that eighteen repositories import it and behave identically in IDE and CI. |
| `Abstractions`, `.Navigation`, `.Offline` | Contracts with no implementation; demonstrated through `Abstractions.Testing`, which fakes all three. |
| `Presentation.Mvp`, `.Mvvm`, `.Android`, `.iOS` | Pattern adapters; demonstrated through the native and cross-platform samples that consume them. |
| `Rinzler78.Templates` | A `dotnet new` template pack; its demonstration is that the harness regenerates from it byte-identically. |

Any package outside the exemption table without a demonstration SHALL be reported by
the **ecosystem audit** defined in `ecosystem-architecture`. The eight-layer map
governs what an application may reference, not how many applications exist.

The sample matrix is therefore an executable proof of the dependency graph, which
no demonstration concentrated at the top of the stack can provide.

#### Scenario: a sample cannot reach upward
- **GIVEN** the sample application of `Here.Sdk.Bindings.Android`
- **WHEN** its dependency graph is inspected
- **THEN** `Here.Sdk.Android` is absent

### Requirement: All applications replay one canonical scenario

Every demonstration SHALL implement the same user journey — search a place,
compute a route, follow the guidance. Each `README.md` SHALL state in one sentence
what this level adds relative to the level below.

The reader does not discover thirty-one different applications; they see one
journey replayed at thirty-one altitudes. The progression is the demonstration.

#### Scenario: the gallery reads as a staircase
- **GIVEN** the gallery page published by `Meta`
- **WHEN** a visitor opens it
- **THEN** the applications are ordered by layer, each with its one-sentence delta

### Requirement: Applications multiply by project system, never by .NET vintage

Where both generations exist, one application SHALL be produced per project system
— one Xamarin, one .NET on the current LTS. Intermediate vintages (`net8.0`,
`net9.0`) SHALL be proven by CI compatibility projects that reference the produced
package and fail the build on regression, not by additional applications.

#### Scenario: net8.0 is proven without an application
- **GIVEN** a compatibility project targeting `net8.0`
- **WHEN** the package under test introduces an API regression
- **THEN** the build fails and the pull request is blocked

### Requirement: Each application declares its target architecture

| Package | Framework | Architecture |
|---|---|---|
| `Bindings.iOS` | .NET 10 iOS | none, deliberately — one screen |
| `Bindings.Android` | .NET 10 Android | none, deliberately — one screen |
| `Bindings.Js` | Blazor WASM | none, deliberately — one component |
| `iOS` | Xamarin.iOS | MVC Cocoa |
| `iOS` | .NET 10 iOS | MVVM-C |
| `Android` | Xamarin.Android | MVP |
| `Android` | .NET 10 Android | MVVM, unidirectional data flow |
| `Js` | Blazor WASM | components and injected services |
| `Standard.iOS` | .NET 10 iOS | MVVM-C |
| `Standard.Android` | **Xamarin.Android** | **MVVM, unidirectional data flow** |
| `Standard.Js` | Blazor WASM | components and injected services |
| `Standard` façade | MAUI, four heads | MVVM, plus provider selector |
| `Blazor` | WASM, Server, WPF host, WinForms host | components and injected services |
| `Forms` | Xamarin.Forms — Android and iOS | MVVM |
| `Maui` | MAUI — four heads | MVVM |
| `Abstractions.Testing` | Blazor WASM | components and injected services |
| `Common` | Blazor WASM | components and injected services |
| `Core` | Blazor Server | components and injected services, secrets held server-side |
| `Rest` | Blazor Server | components and injected services |
| `Navigation` | Blazor Server | components and injected services |
| `Presentation` | Blazor WASM | components and injected services |

Each choice SHALL be justified by an ADR in the owning repository. The binding
demonstrations SHALL carry no architecture: introducing a pattern would obscure
what they prove.

#### Scenario: the Android ADR explains the apparent regression
- **GIVEN** a reviewer who knows Google recommends MVVM with unidirectional data flow
- **WHEN** they find MVP in the Xamarin.Android sample
- **THEN** `ADR-0001` states that Google's guidance presupposes Compose or XML
  DataBinding, that .NET Android has neither, and that without a binding engine
  MVVM degenerates into MVP under another name

### Requirement: The standardisation level shares one logic file across generations

The `Standard.iOS` and `Standard.Android` demonstrations SHALL share the same
business-logic source file, compiled unchanged into a **.NET 10 iOS** application
and into a **Xamarin.Android** application.

One file, two platforms, two framework generations, two native user interfaces —
and the Android side additionally proves in-place modernisation of a Xamarin estate
without rewriting it.

The file SHALL be **linked**, not copied: both project files reference the same path
under a shared folder, so there is one source of truth and no possibility of drift.

#### Scenario: both projects link the same path
- **GIVEN** the two standardisation samples
- **WHEN** CI resolves the `Compile` items of both projects
- **THEN** both include the identical absolute path to the shared logic file
- **AND** the presence of a copy of that file anywhere under `samples/` fails the build

### Requirement: The façade demonstration swaps providers at runtime

The façade application SHALL expose a selector that switches, on Android and iOS,
between the native provider and the JavaScript provider hosted in a
`BlazorWebView`, with no change to business-logic code — only the dependency
injection registration differs.

Its `README.md` SHALL state explicitly that JavaScript rendering on mobile is a
demonstration of interchangeability and not a production option: no offline maps,
no HERE positioning, and WebView performance.

#### Scenario: the map engine changes, the code does not
- **GIVEN** the façade application running on Android
- **WHEN** the user switches the provider selector
- **THEN** the map re-renders through the JavaScript engine
- **AND** no business-logic assembly is reloaded or recompiled
