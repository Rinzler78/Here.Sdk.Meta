# testing

## ADDED Requirements

### Requirement: Unit tests run on the build agent, never on a device

Every unit test project SHALL target `net10.0` and use xUnit v3. No
test project SHALL target a platform moniker, and no unit test SHALL require an
emulator, a simulator or a physical device.

This is enforceable only because of the rule it depends on: an assembly targeting
`monoandroid12.0`, `xamarinios10`, `net10.0-android` or `net10.0-ios` exposes no
asset a `net10.0` test project can reference. Platform assemblies SHALL therefore
contain glue alone — type projection, view wiring, native call — and any logic worth
a unit test SHALL live in a portable assembly. Code that is hard to test is code in
the wrong place; the answer is to move the code, never to add a harness able to test
it where it stands.

A single framework is possible only because of the same rule. xUnit v3 ships
`net472` and `net8.0` assets and no `netstandard` asset, so it could not be
referenced from the two Xamarin generations. Since no test project targets them, the
constraint applies to nobody and the ecosystem needs one version, not two.

#### Scenario: a unit test proposed against platform code is refused
- **GIVEN** a contributor proposes a unit test for a type in `Here.Sdk.Standard.Android`
- **WHEN** the review asks what the test covers
- **THEN** the logic is moved into a portable assembly and tested there
- **AND** the platform assembly keeps only the call that could not move

#### Scenario: no device runner enters the toolchain
- **GIVEN** a proposal to add a device-based unit test runner
- **WHEN** it is evaluated
- **THEN** it is refused, and the ADR records that the logic belongs in a portable
  assembly instead

### Requirement: One assertion library and one substitute library, chosen on licence and analyser

Assertions SHALL use `AwesomeAssertions`, the Apache-2.0 community fork of
FluentAssertions 7. FluentAssertions 8 and later ship the Xceed Community License
Agreement, restricted to non-commercial use — and this ecosystem exists to win
commercial work, so the grey area is not one to stand in on a repository a client
may audit.

Substitutes SHALL use `NSubstitute`, together with `NSubstitute.Analyzers.CSharp`.
The analyser is the reason: it catches at compile time the misuses that otherwise
produce a green test asserting nothing — a configured call on a non-virtual member,
a substitute never consumed. On an ecosystem where warnings are errors, a first-party
analyser that has reached 1.0 outweighs the wider familiarity of the alternative,
whose own analyser has not.

Hand-written doubles SHALL be permitted where an interface is small enough that a
double reads better than a configured substitute, but SHALL NOT be the default: a
hand-written double drifts from the contract it imitates without anything failing.

#### Scenario: a misconfigured substitute fails the build
- **GIVEN** a test configuring a return value on a non-virtual member
- **WHEN** it is compiled
- **THEN** the analyser reports it, and `TreatWarningsAsErrors` fails the build

#### Scenario: the licence of a test dependency is auditable
- **GIVEN** a client auditing the repository's dependency licences
- **WHEN** they inspect the test dependencies
- **THEN** every one resolves to an OSI-approved expression, with no
  non-commercial-use restriction

### Requirement: Platform assemblies are guaranteed without unit tests

An assembly that carries only glue SHALL declare `<HereCoverageContract>Reduced` or
`Exempt`, and SHALL be guaranteed by two mechanisms instead: the API-coverage test
defined in `quality`, which asserts the projected surface against the upstream
artefact, and the canonical user interface suite defined below.

#### Scenario: a glue assembly is not held to the portable floor
- **GIVEN** `Here.Sdk.Bindings.iOS`, whose contract is `Exempt`
- **WHEN** the coverage gate runs
- **THEN** it does not fail the build for absent unit tests
- **AND** the API-coverage test and the user interface suite must both be green

### Requirement: Integration tests live in their own project, in the five repositories that reach the network

Tests consuming a real HERE credential SHALL live in a separate project,
`tests/<PackageId>.IntegrationTests`, never alongside the unit tests of the same
package, and SHALL NOT run in the same pass.

The reason is measurable rather than aesthetic. The diff coverage gate is set at
95 %, and a threshold means nothing unless its denominator is stable. Sharing one
project would make coverage rise when a credential is present and fall when it is
not, so the same commit would pass or fail depending on the environment. Separate
projects make the figure deterministic and make it impossible to fire a network call
inside the unit pass by accident.

Such tests SHALL exist only in the repositories that genuinely reach the network —
`Rest`, the three binding repositories, and `Navigation`. The requirement that
integration tests skip cleanly without a credential is easy to honour in five
repositories and becomes empty ceremony in the other thirteen, which have nothing to
call. A mechanism present everywhere and useful nowhere stops being verified.

That the thirteen others have no network test is itself a property: it holds only if
the portable layer is genuinely isolated from the network, which the executable
Clean Architecture check already asserts.

#### Scenario: coverage does not depend on the environment
- **GIVEN** the same commit built with and without HERE credentials configured
- **WHEN** the diff coverage gate runs on the unit pass
- **THEN** it reports the same figure in both cases

#### Scenario: a network test proposed outside the five is refused
- **GIVEN** a proposal adding an integration test to `Here.Sdk.Presentation`
- **WHEN** it is reviewed
- **THEN** it is refused, because a presentation package that reaches the network
  violates the architecture check before it violates this requirement

### Requirement: One parameterised user interface suite, executed once per head

Eighteen heads carry a graphical shell: nine in the iOS and Mac Catalyst family,
seven Android, two Windows. Fourteen of them require an emulator or a simulator —
the seven iOS and the seven Android; the two Mac Catalyst and the two Windows heads
run natively on their own runner. They SHALL be exercised by **one** test project,
not eighteen, executed once per head by a continuous integration matrix.

Two drivers are needed, and the split is not a preference. Appium SHALL drive the
sixteen Apple and Android heads, Mac Catalyst through its `mac2` driver. The two
Windows heads SHALL be driven by FlaUI directly against UI Automation, because
Appium's Windows driver delegates to Microsoft's WinAppDriver, whose last release is
`v1.2.99` of 1 July 2021.

The canonical scenario SHALL be written **once**, against a driver abstraction, with
one adapter per driver. This is the same shape as the presentation layer: interfaces
diverge because the platform imposes it, implementations converge because the
behaviour is one. A second copy of the scenario for Windows would reintroduce
exactly the drift this requirement exists to prevent.

The reason is not economy. Every application already replays one canonical scenario,
so the test is the same test by construction; writing it eighteen times would allow
eighteen heads to drift into eighteen slightly different scenarios while every suite
stayed green. One suite turns "the same journey everywhere" into an executable proof.

The suite SHALL be a `net10.0` xUnit v3 project. Both drivers are clients: the test
assembly runs on the agent, and the application under test runs on the emulator, the
simulator, or the runner itself. The test assembly never runs on a device.

#### Scenario: one head drifts and the shared test catches it
- **GIVEN** the Xamarin.Forms Android head stops honouring one step of the canonical scenario
- **WHEN** the matrix runs
- **THEN** the shared suite fails on that configuration alone
- **AND** the seventeen other configurations stay green

#### Scenario: a new head costs a matrix row
- **GIVEN** a nineteenth head is added
- **WHEN** its user interface coverage is set up
- **THEN** a matrix entry is added and no test project is created

### Requirement: One user interface testing stack per medium, and no more

Appium SHALL drive the Apple and Android heads, Xamarin included — Appium operates
at platform level, so the framework generation is transparent to it. FlaUI SHALL
drive everything Windows: the two MAUI Windows heads in full, and a launch smoke
test alone for the WPF and WinForms hosts in the `Blazor` repository, whose only
content is a `BlazorWebView`. `bUnit` SHALL cover Razor components, Playwright the
browser end-to-end journeys.

Every one of these stacks SHALL be driven from xUnit v3, the ecosystem's single test
framework. The Appium client is framework-agnostic, so no medium justifies a second
one.

The navigation engine SHALL be verified by replaying recorded GPS traces as pure
unit tests, on the agent, with no device.

#### Scenario: the desktop shells are smoke-tested, not re-tested
- **GIVEN** a WPF host whose only content is a `BlazorWebView`
- **WHEN** FlaUI runs
- **THEN** it asserts the window opens and the WebView has loaded, nothing more
