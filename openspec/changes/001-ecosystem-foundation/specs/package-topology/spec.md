# package-topology

## ADDED Requirements

### Requirement: Package identifiers are prefixed, namespaces are not

Every published package identifier SHALL be prefixed `Rinzler78.`. C# namespaces
SHALL remain `Here.Sdk.*`. Only `<PackageId>` carries the prefix.

#### Scenario: identifier and namespace differ
- **GIVEN** the package `Rinzler78.Here.Sdk.Common`
- **WHEN** a consumer adds `using Here.Sdk;`
- **THEN** the types resolve

### Requirement: The ecosystem is eighteen repositories and twenty-eight packages

The ecosystem SHALL consist of exactly eighteen repositories publishing exactly
twenty-eight packages, as enumerated below. Any addition or removal SHALL require
an OpenSpec change.

| Repository | Packages |
|---|---|
| `Rinzler78.Toolkit` | `Rinzler78.Templates`, `Rinzler78.Build` |
| `Here.Sdk.Meta` | none — orchestration cockpit, public |
| `Here.Sdk.Common` | `Common` |
| `Here.Sdk.Abstractions` | `Abstractions`, `.Navigation`, `.Offline`, `.Testing` |
| `Here.Sdk.Core` | `Core` |
| `Here.Sdk.Rest` | `Rest` |
| `Here.Sdk.Navigation` | `Navigation` |
| `Here.Sdk.Presentation` | `Presentation`, `.Mvp`, `.Mvvm`, `.Android`, `.iOS` |
| `Here.Sdk.Bindings.Android` | `Bindings.Android` |
| `Here.Sdk.Bindings.iOS` | `Bindings.iOS` |
| `Here.Sdk.Bindings.Js` | `Bindings.Js` |
| `Here.Sdk.Android` | `Android` |
| `Here.Sdk.iOS` | `iOS` |
| `Here.Sdk.Js` | `Js` |
| `Here.Sdk.Standard` | `Standard`, `.Android`, `.iOS`, `.Js` |
| `Here.Sdk.Blazor` | `Blazor` |
| `Here.Sdk.Forms` | `Forms` |
| `Here.Sdk.Maui` | `Maui` |

During realisation the ecosystem is incomplete by construction, so the audit checks
containment rather than equality: every published package SHALL appear in the table,
and a package outside it SHALL fail the audit. Equality SHALL be asserted only once
the realisation order is complete.

#### Scenario: an unplanned package is caught immediately
- **GIVEN** a package published under the `Rinzler78.` prefix that the table does not list
- **WHEN** the ecosystem audit runs
- **THEN** it reports the package as unplanned

#### Scenario: an incomplete ecosystem does not fail the audit
- **GIVEN** realisation has reached phase 4 and eleven packages are published
- **WHEN** the ecosystem audit runs
- **THEN** it reports eleven of twenty-eight published, without raising a failure

### Requirement: The target framework matrix is complete on both layers

Portable packages SHALL target `netstandard2.0`, `net8.0`, `net9.0` and `net10.0`.

Platform packages SHALL target the Xamarin monikers `monoandroid12.0` and
`xamarinios10`, and the current monikers `net8.0-android`, `net9.0-android`,
`net10.0-android`, `net8.0-ios`, `net9.0-ios`, `net10.0-ios`. Windows and Mac
Catalyst heads, where present, SHALL use `net10.0-windows10.0.19041.0` and
`net10.0-maccatalyst`.

The JavaScript column is exempt. `Microsoft.JSInterop` versions 8, 9 and 10 each
target only their matching .NET major and offer no `netstandard2.0` asset — version
10 targets `net10.0` alone. `Bindings.Js`, `Js` and `Standard.Js` SHALL
therefore target `net8.0`, `net9.0` and `net10.0` only, each conditioning the
matching `Microsoft.JSInterop` version. No Xamarin generation exists in this column.

#### Scenario: a .NET Framework consumer resolves the portable layer
- **GIVEN** a project targeting `net472`
- **WHEN** it references `Rinzler78.Here.Sdk.Rest`
- **THEN** the `netstandard2.0` asset is selected and restore succeeds

### Requirement: Targets are iOS, Android, Blazor, Windows and macOS

Native rendering SHALL be used on Android and iOS. Windows, macOS (Mac Catalyst)
and the browser SHALL render through the HERE Maps API for JavaScript hosted in a
`BlazorWebView`, requiring no additional package.

Linux SHALL be a supported deployment target for the server and web half only;
Linux desktop is out of scope, with no official `BlazorWebView` and no MAUI support.

#### Scenario: a desktop host renders through the JavaScript engine
- **GIVEN** a WPF application referencing `Rinzler78.Here.Sdk.Blazor`
- **WHEN** it hosts the map component in a `BlazorWebView`
- **THEN** the HERE JavaScript map renders, with no additional package and no
  Windows-specific binding

#### Scenario: Linux desktop is refused explicitly
- **GIVEN** a request to add a Linux desktop head
- **WHEN** the target matrix is consulted
- **THEN** it is refused, and the ADR states that neither MAUI nor `BlazorWebView`
  supports Linux desktop

### Requirement: Every package states it is not affiliated with HERE

This is a community project. Every package SHALL carry, in its `<Description>` and
in its `README.md`, a statement that it is not affiliated with, endorsed by, or
supported by HERE Technologies, and that consumers must obtain their own HERE
credentials and comply with HERE's Developer Agreement independently.

Publishing twenty-eight packages named `Here.Sdk.*` without that statement is a
trademark and expectation risk, not a formality.

#### Scenario: the disclaimer travels with the package
- **GIVEN** any published package
- **WHEN** its `.nuspec` description is read on nuget.org
- **THEN** the non-affiliation statement is present
- **AND** a packaging check fails the build if it is missing

