# Design — 001 Ecosystem foundation

## Decision register

Sixty-two decisions were taken. The register below records those that shape the
architecture, with the reasoning that settled them. It is the normative record;
the narrative form is published separately as a reference page.

### Objective and arbitration

The ecosystem is a commercial showcase for a freelance practice. Every arbitration
is settled by *what does this prove to a client*. This is why Xamarin stays in
scope despite end of support on 1 May 2024: clients run Xamarin applications, and
serving them is market proof rather than nostalgia.

### Why the natives reference nothing

An early draft had the native wrappers implement `Abstractions` directly. That was
inverted. A wrapper that references only its binding is a **standalone product**: a
developer targeting Android alone gets the full native SDK behind a clean C# API,
with no abstraction cost and no contract package. Portability becomes an option
that is added, not a tax that is paid.

The cost is accepted deliberately: nothing forces the two native APIs to resemble
each other, so they will diverge, and the adapters absorb that divergence. The
duplication sits in two thin layers rather than in the domain.

### Why a façade exists

Three adapters without a single entry point would leave the platform switch to the
integrator — `Condition` attributes in their own project file — which is exactly
what a standardisation layer is supposed to remove. `Here.Sdk.Standard` is a
bait-and-switch package: target frameworks and conditional dependencies, plus one
`AddHereSdk()`.

`Standard.Js` stays outside it because Blazor WebAssembly targets `net10.0` exactly
like a console or a WPF application; the target framework cannot distinguish them.
The `net10.0` fallback resolves to the portable REST implementation, which is the
correct default because it works everywhere, and `Here.Sdk.Blazor` pulls
`Standard.Js` explicitly for map rendering.

### Why navigation reaches all five targets

The first cut reserved navigation to the native SDKs. The REST building blocks all
exist: `Routing v8` for the route and its manoeuvres, `Route Matching v8` or local
projection for snapping, platform geolocation for position, speech synthesis for
voice. Only offline maps, HERE positioning and lane guidance remain out of reach.

The boundary therefore runs between `Abstractions.Navigation` — everywhere — and
`Abstractions.Offline` — native only. This is not the trap the segregation was
built to avoid: on the web the implementation is real and functional, merely less
precise. That is ordinary polymorphism, provided the interface exposes only what
both implementations can honour.

### Why the presentation engine pushes whole state

Six idioms, one engine. The engine exposes `TState Current` and
`event EventHandler<TState> StateChanged`.

Push is hostile to MVVM: reassigning the collection on every keystroke costs the
`CollectionView` its selection and scroll position — a defect the prospect sees
first. The diff, stable-by-key collections, fine-grained `PropertyChanged`,
`IRelayCommand` and UI-thread marshalling therefore live in `Presentation.Mvvm`,
written once, and not in the core. MVP and Blazor both prefer push; only MVVM needs
the diff, so only MVVM pays for it.

Exposing `INotifyPropertyChanged` from the core would charge XAML vocabulary to
Android, iOS and the web. Exposing `IObservable<TState>` would impose
`System.Reactive` on the entire stack, Xamarin.Forms included, for a benefit an
`event` covers here.

### Why the idioms differ per generation

Google recommends MVVM with unidirectional data flow, Compose-first; MVP is
documented as the historical pattern. The iOS idiom of 2026 is MVVM-C; Apple's MVC,
whose name `UIViewController` still carries, is what produced the Massive View
Controller the community fled.

But Google's guidance presupposes Compose or XML DataBinding, and .NET Android has
neither. Without a binding engine, MVVM reduces to *an object holds the state, the
view subscribes and repaints by hand* — which is the shape of MVP under another
name. The choice is one of honest naming, not of capability.

Each generation therefore uses the pattern of its era, and the modernisation proof
is carried by the `Standard.Android` sample: an old framework, a current
architecture, a portable contract — and the same logic file as its .NET 10 iOS twin.

### Why the JavaScript column has a wrapper too

An early draft ran `Bindings.Js` straight into `Standard.Js`, giving the web column
four rungs where the native columns have five. Nothing justified the asymmetry.

What a raw C# interop over a JavaScript API exposes is `IJSObjectReference` handles
with manual lifetimes, everything asynchronous, `DisposeAsync` to remember, and
events to rewire by hand. That is as unidiomatic in C# as a raw JNI projection, so
the wrapper earns its place for the same reason on both sides — and it makes
`Here.Sdk.Js` a standalone product for someone who only builds Blazor and wants the
full HERE map behind a clean API, without the portable contract.

The asymmetry also mattered for reading: the staircase only makes sense if the three
columns have the same steps. A column with one step missing makes a reader wonder
what was skipped, or conclude the web was treated as second class.

### Why contracts are declared per project rather than set globally

Trimming and coverage could both have been fixed ecosystem-wide. They are not,
because a single global value is either unreachable somewhere or meaningless
everywhere.

A uniform 90 % coverage floor cannot be met by a JNI or Objective-C projection, and
an unreachable gate is a gate that gets disabled. A uniform `IsTrimmable` would make
the bindings claim a property they cannot honour, which invites the complacent
suppression the principles forbid.

So each project declares `<RinzlerTrimContract>` and `<RinzlerCoverageContract>`, and the
declaration costs an ADR. The vocabulary is uniform, the value is local, and the
justification is written down — which is stronger than a global number nobody
believes, and cheaper than an exemption list maintained centrally.

### Why neither Xamarin head reaches a store

The freeze is not only a build problem, it is a distribution one, and on both
platforms at once.

Apple has required Xcode 16 and the iOS 18 SDK for submission since 24 April 2025;
the frozen chain builds with Xcode 15.4. Google Play has required target API 36 for
new apps and updates since 31 August 2026, and already hides apps below target
API 35 from new users on recent devices; Xamarin.Android's last supported target is
API 34.

Both Xamarin heads are therefore distributed as signed release artifacts, with the
cause and its date stated in each `README.md`. This does not weaken the market proof
— a client with a Xamarin estate is not looking for a store listing, they are
looking for someone who can still build, maintain and modernise what they run. The
flagship modernisation demonstration, `Standard.Android` on Xamarin.Android, is
distributed the same way; the façade demonstration, on MAUI, carries the store
presence.

### Why the public web demonstrations ship a visible key

A WebAssembly page published to GitHub Pages hands its key to every visitor; no
secret mechanism changes that. The answer is a key **designed to be public**: a
separate HERE API key whitelisted on the Developer Portal to the Pages domain, so a
request from any other origin is ignored.

That is why the server-side demonstrations — `Core`, `Rest`, `Navigation` — are
hosted rather than static: their credentials must not reach a browser at all. The
constraint produces the architecture, and the two-key rule is verified rather than
trusted.

### Why the toolchain is frozen rather than abandoned

No hosted runner keeps Xcode 15 after 2 November 2026; Azure DevOps consumes the
same images and offers no escape; Bitrise retires its Xcode 15.x stacks on
16 September 2026. Apple's licence permits two virtual machines per Apple host, so
every cloud provider runs racks of Macs — EC2 Mac is physical Apple hardware billed
with a 24-hour minimum allocation.

A self-hosted runner is therefore not a fallback for want of better; it is the only
route the market leaves open at a reasonable cost. Operating a deliberately frozen
toolchain, documented against a dated market survey, is itself the demonstration.

### Why build order differs from publication order

Designing `Abstractions` from documentation and discovering ten positions later that
the native types do not fit reproduces exactly the crooked history this reset
exists to remove. The natives are built first, standing alone; the contract is
extracted from all three capability sets.

The only obstacle — repositories reference each other solely through published
packages — is lifted by prerelease versions. Release Please already versions
`develop` builds that way.

A spike survives, reduced to JavaScript alone. Extracting from the two most capable
implementations reliably produces a contract the third cannot satisfy; the JS
capability matrix must be known before extraction, and it costs documentation
reading and a trial page rather than a binding project.

## Why no unit test runs on a device

Three test frameworks were candidates, and the constraint that separated them was
the frozen chain: xUnit v3 4.0.1 and NUnit 4.4.0 ship no `netstandard` asset, so
neither can be referenced from `monoandroid12.0` or `xamarinios10`. That pointed at
a two-framework ecosystem — modern above, xUnit 2.x on the Xamarin heads.

Taking the architecture seriously dissolved the problem instead. If a platform
assembly holds nothing but glue, nothing in it is worth a unit test, no test project
targets a platform moniker, and the `netstandard` constraint applies to nobody. One
framework, one target framework, no condition.

The general form of the rule matters more than the saving: code that is hard to test
is code in the wrong place. Financing a device runner — XHarness is not on
nuget.org, so it would also mean a non-nuget.org feed in several repositories — buys
the ability to leave logic where it should not be.

## Why one user interface suite rather than eighteen

Eighteen heads need an emulator or a device. Writing eighteen suites would let
eighteen heads drift into eighteen slightly different scenarios while every suite
stayed green, which is the opposite of what the gallery claims. Since every
application already replays one canonical scenario, the test is the same test by
construction, and a single parameterised project turns the claim into a proof.

The Appium client is a driver, not a runtime: the test assembly runs on the agent and
the application under test runs on the emulator, the simulator, or the runner
itself. So the suite is an ordinary `net10.0`
project, and the framework question answers itself — xUnit v3, like everything else.

## Why the cockpit never owns another repository's decision

An SDK upgrade traverses the whole ecosystem without belonging to any repository,
which makes a central proposal in `Meta` tempting. It was rejected: a repository that
needs the cockpit to know what to do is not autonomous, and the cockpit becomes both
a point of failure and a shared state that drifts — the previous attempt's manual
progress tracker did exactly that, and it lied.

The signal therefore travels with the artefact. The surface diff decides the
published version, and the version is what downstream repositories already read.
`Meta` detects, informs and aggregates; the decision stays where the work is.

What makes this more than a slogan is that the diff is generated rather than
written. `Microsoft.CodeAnalysis.PublicApiAnalyzers` turns any surface change into a
compilation error until `PublicAPI.Unshipped.txt` records it, so the verdict is
produced by the compiler and committed. Upstream and projected surfaces stay
distinct artefacts: `api.xml` and the Sharpie output say what HERE changed, before
anything is decided; `PublicAPI.*.txt` says what we expose, after transformation.

## Why the toolkit is agnostic and the domain policy is a separate package

Three things were tangled in one package: a mechanism, a vocabulary, and a
position. They have three different lifetimes, and the first version gave them one.

The mechanism — how to validate a layer name, how to refuse an outward dependency,
how to require an analyser or a description fragment — outlives the domain
entirely. It belongs in `Rinzler78.Build`, which knows how to refuse and not what
to refuse, and which builds a project that declares no architecture at all rather
than failing it for having none.

The vocabulary — which layers exist, what each may not depend on, what every
package description must say — is shared by nineteen repositories and must be true
in all of them at the same moment. A non-affiliation statement present in eighteen
repositories and missing in the nineteenth is worth nothing: it is the missing one
that gets read. That is what makes it a published package, `Here.Sdk.Build`, rather
than template content, which drifts by design and is only reconciled by a nightly
audit.

The position — the single layer a project sits in — belongs to that project and to
nobody else. It is generated into each repository's own `.props` by the template,
and it is the only one of the three that may legitimately differ everywhere.

This is the same shape the ecosystem applies to everything else: a neutral engine,
a shared specialisation, and a local declaration. Applying it to the toolchain as
well is not symmetry for its own sake — it is what makes the toolkit an asset that
survives this project.

## Alternatives rejected

| Rejected | Reason |
|---|---|
| Docker and devcontainers | iOS is structurally out of reach — Apple forbids macOS virtualisation off Apple hardware — so a containerised environment would cover half the ecosystem. Reversible in three lines if remote work becomes useful. |
| Mac Catalyst through the native SDK | The HERE `.xcframework` documents no `maccatalyst` slice; a declared target framework would compile and fail at link time in the integrator's build. It returns through the JavaScript route. |
| MapLibre-GL JS | Using a different rendering engine inside a HERE wrapper contradicts the package's purpose. |
| Xamarin.UITest | Deprecated. Appium drives at platform level, so the framework generation is transparent to it. |
| Per-file coverage threshold | Forces tests on code with no behaviour; replaced by diff coverage and mutation score. |
| Templates held by Meta | Would recreate the coupling the original ADR condemned and make eighteen repositories depend on a nineteenth. |
| A single view contract across idioms | A presenter does not have the shape of a ViewModel, and the platform imposes it. Interfaces diverge; implementations converge. |
| MVVM on both native generations | Google recommends it and iOS converged on MVVM-C, but that guidance presupposes Compose or XML DataBinding, which .NET Android lacks. Each generation uses the pattern of its era instead, and the modernisation proof moves to the `Standard.Android` sample. |
| A JavaScript column without a wrapper | Would leave the web with four rungs where the natives have five, for a binding whose surface is as unidiomatic as a JNI projection. |
| A uniform coverage floor | Unreachable on the binding projections, and an unreachable gate is one nobody enforces. Replaced by a declared contract per project, each costing an ADR. |
| A rented dedicated Mac as fallback | Apple's licence imposes a 24-hour minimum billing per allocation; hundreds of euros a month for a frozen chain touched rarely. The chain accepts blocking. |
| Fan-out release notification | A package two levels down would compile against a dependency still on the previous version. The cascade is topological, wave by wave. |
| A device-based unit test runner | XHarness is absent from nuget.org, so it would add a non-nuget.org feed and an emulator to the continuous integration of several repositories — to test logic that should have been moved into a portable assembly. |
| Two test frameworks, split by generation | Only necessary if a test project targets a Xamarin moniker. None does, so the split buys nothing and costs a conditioned `Directory.Packages.props`. |
| One Appium suite per head | Eighteen copies of one canonical scenario, free to drift apart while all staying green. |
| NUnit for the user interface suite | Inherited from the Appium documentation. The client is framework-agnostic, and a second framework for one medium is a second thing to keep aligned. |
| A central upgrade proposal in `Meta` | Makes seventeen repositories depend on the cockpit for a decision that is theirs. Repositories are autonomous; the package and its version are the protocol. |
| Scraping the HERE release notes | The documentation site is a single-page application whose Android and iOS release-note URLs return byte-identical documents. The examples repository publishes dated releases through the GitHub API instead. |
| Shipping `PublicAPI.Shipped.txt` inside the package for consumers to assert against | Duplicates what the compiler already does, and adds twenty-eight downstream baselines to keep true. A duplicated baseline eventually lies. |
| FluentAssertions 8 and later | Ships the Xceed Community License Agreement, restricted to non-commercial use, on an ecosystem whose purpose is to win commercial work. The Apache-2.0 fork keeps the API without the question. |
| Integration tests sharing the unit test project | Would make diff coverage depend on whether a credential is configured, so the same commit passes or fails by environment. |
| An integration test mechanism in all nineteen repositories | Empty ceremony in the thirteen that reach no network, and a mechanism useful nowhere stops being verified. |
| Both Xcode versions on one machine, selected per job | Keeps a May 2024 Xcode on a 2026 operating system with no Apple support and no recourse once it is unsigned, and requires updating a machine whose purpose is to stay frozen. |
| A second virtualised macOS for the modern heads | Permitted by Apple's licence, but adds a system to administer while a hosted image already carries the required Xcode. Kept as the fallback. |
| Appium's Windows driver for the MAUI Windows heads | It delegates to Microsoft's WinAppDriver, whose last release is v1.2.99 of 1 July 2021. FlaUI drives UI Automation directly and is actively released. |
| A second copy of the canonical scenario for Windows | Two drivers do not justify two scenarios. The scenario is written once behind a driver abstraction, with one adapter each — the same shape as the presentation layer. |
| A toolkit named agnostically but bound to one domain | The first version refused to pack any project whose description omitted a HERE non-affiliation statement, under a package named `Rinzler78.Build`. A package published under a personal prefix cannot impose a third party's commercial notice on unrelated consumers. |
| Folding the domain policy into the agnostic package | Would make the toolkit unusable outside this ecosystem and waste the `Rinzler78.` prefix, which would then name nothing reusable. |
| Leaving the domain policy entirely to templates | Correct for a project's own layer, wrong for the layer vocabulary and the non-affiliation statement: those must be true across nineteen repositories at the same moment, and generated copies drift by design. |
