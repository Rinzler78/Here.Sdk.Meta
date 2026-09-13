# secrets

## ADDED Requirements

### Requirement: HERE credentials never enter the repository and never block a contributor

HERE credentials SHALL reach a **build or a test run** through GitHub Actions
secrets in CI — `HERE_ACCESS_KEY_ID`, `HERE_ACCESS_KEY_SECRET`, `HERE_API_KEY` — and
through user secrets or environment variables locally. They SHALL NOT appear in any
committed file, sample configuration, or test fixture.

This concerns how a credential reaches an **application or a test project**. How it
then reaches the **SDK** is a separate rule: the application binds it from its own
configuration and passes it explicitly at registration, and no package reads an
environment variable on its own initiative.

Integration tests SHALL **skip cleanly** when credentials are absent, reporting as
skipped rather than failed, so that a contributor or a fork can run the full suite
without any HERE account.

A pre-commit hook and a CI check SHALL detect credential-shaped strings and fail.
Secrets SHALL be rotated every ninety days, tracked by a scheduled issue in `Meta`.

#### Scenario: a fork runs the suite without credentials
- **GIVEN** a clone with no HERE credentials configured
- **WHEN** `./scripts/test.sh` runs
- **THEN** unit tests pass, integration tests report as skipped, and the exit code
  is zero

#### Scenario: a leaked key is caught before the commit
- **GIVEN** a developer pastes an access key into a sample configuration
- **WHEN** they attempt to commit
- **THEN** the pre-commit hook refuses the commit and names the file and line

### Requirement: No package ever carries a credential, and none has a default

No package SHALL embed a credential, and none SHALL provide a default value for one.
Credentials SHALL reach the SDK exclusively through the registration options —
`AddHereSdk(options => …)` — bound from the integrator's own configuration.

A missing credential SHALL fail **loudly at registration**, naming the option that
is absent and how to supply it, never silently at the first network call where the
symptom is a meaningless authentication error.

The packages SHALL NOT read an environment variable or a well-known file on their
own initiative: implicit credential discovery makes the source of a key invisible to
the integrator and unpredictable across environments.

#### Scenario: a missing key is named at startup
- **GIVEN** an application that registers the SDK without credentials
- **WHEN** the service provider is built
- **THEN** it throws, naming the missing option and the documentation anchor

#### Scenario: no package ships a usable default
- **GIVEN** any published package
- **WHEN** it is scanned for credential-shaped constants
- **THEN** none is found, and the packaging check fails the build if one appears

### Requirement: Shipped applications store credentials honestly

A distributed mobile binary cannot keep a secret: HERE states that credentials are
embedded in the binary and that binaries are trivially decompiled. The specification
SHALL NOT pretend otherwise.

The demonstration applications SHALL therefore:

- set credentials **programmatically** at start-up, never hardcoded in
  `AndroidManifest.xml` or `Info.plist`, so the value is injected at build time from
  a CI secret rather than committed;
- cache them in **`SecureStorage`** — Android Keystore, iOS Keychain — after first
  provisioning, rather than keeping them in plain application state;
- use credentials **bound to their own bundle identifier**, never shared between two
  applications, since HERE issues them per bundle. That binding, together with quota
  alerting, is the real protection — not concealment;
- state in their `README.md` that the embedded credential is a demonstration
  credential, bound and quota-limited, and that a production integrator supplies
  their own.

`dotnet user-secrets` SHALL NOT be relied upon on Android or iOS: it is a
development-time mechanism with no runtime presence on those platforms. Locally,
mobile demonstrations SHALL read an untracked local file that the build injects, and
the desktop, server and web demonstrations SHALL use user secrets or environment
variables.

#### Scenario: no credential is committed in a platform manifest
- **GIVEN** the Android and iOS demonstration projects
- **WHEN** `AndroidManifest.xml` and `Info.plist` are scanned
- **THEN** no `access_key_id`, `access_key_secret` or API key literal is present

#### Scenario: a demonstration key is useless in another application
- **GIVEN** the credential extracted from a published demonstration binary
- **WHEN** it is used from an application with a different bundle identifier
- **THEN** HERE refuses it, because credentials are issued per bundle

### Requirement: Signing material is a secret with its own lifecycle

Publishing through stores and signed artifacts requires material that is neither a
HERE credential nor a build setting, and which SHALL be treated with the same rigour:
the Android keystore and its passwords, and the Apple signing certificate, private
key and provisioning profiles.

This material SHALL live exclusively in GitHub Actions secrets, base64-encoded where
binary, and SHALL be materialised into a temporary keychain or keystore for the
duration of a job and destroyed at its end — never written into the workspace, never
committed, never printed.

Expiry SHALL be tracked: Apple certificates and profiles expire, and an expired
profile fails a release at the worst moment. A scheduled job in `Meta` SHALL open an
issue sixty days before any expiry it knows of.

The self-hosted runner SHALL NOT hold this material in the personal login keychain;
the dedicated account required by `toolchain` exists partly for that reason.

#### Scenario: signing material never touches the workspace
- **GIVEN** a release job that signs an artifact
- **WHEN** the job completes or fails
- **THEN** the temporary keychain is deleted and no key material remains on disk

#### Scenario: an approaching expiry is surfaced early
- **GIVEN** a provisioning profile expiring in fifty-nine days
- **WHEN** the scheduled check runs
- **THEN** it opens an issue in `Meta` naming the profile and its expiry date

### Requirement: Public web demonstrations use a separate domain-restricted key

A WebAssembly demonstration published to GitHub Pages ships its key to every
visitor's browser; no secret mechanism can prevent that. The answer is not to hide
the key but to use one that is **designed to be public**.

A distinct HERE API key SHALL be provisioned for the public web demonstrations,
**whitelisted on the HERE Developer Portal to the Pages domain only**, so that a
request from any other origin is ignored. It SHALL be referred to as
`HERE_JS_DEMO_APIKEY`, SHALL be injected at publication time, and SHALL NEVER be the
key used by server-side demonstrations or by integration tests.

Server-side demonstrations — `Core`, `Rest`, `Navigation`, Blazor Server — SHALL
keep their credentials server-side and SHALL NOT expose them to the browser. That
constraint is the reason those demonstrations are hosted rather than static, and
their `README.md` SHALL say so.

The two-key rule SHALL be verified: a check SHALL fail the publication if the
server-side key appears in any artefact destined for Pages.

#### Scenario: the public key is useless off its domain
- **GIVEN** the WASM demonstration published to Pages, with its key visible in the payload
- **WHEN** the key is replayed from another origin
- **THEN** HERE refuses the request, because the domain is not whitelisted

#### Scenario: the server key never reaches a static artefact
- **GIVEN** a publication to GitHub Pages
- **WHEN** the artefact is scanned before upload
- **THEN** the presence of the server-side key fails the publication
