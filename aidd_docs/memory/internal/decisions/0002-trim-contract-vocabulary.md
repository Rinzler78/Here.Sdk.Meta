# 0002 — The trimming contract takes the specification's vocabulary

## Decision

`RinzlerTrimContract` takes `Committed`, `Analyzed` or `OutOfScope`, as the `quality` specification words it. `Rinzler78.Build` changes; the specification does not.

## Reason

- The two vocabularies named different things. `Trimmable` and `AotCompatible` named a marking a project carries. `Committed`, `Analyzed` and `OutOfScope` name a commitment a project makes, and each has a rule behind it.
- `Analyzed` has no counterpart among the old values: it runs the trim analyser and marks nothing, because `IsTrimmable` is a promise to every integrator and this value makes none.
- Later work is written against the specification's names: the sizes of every `Committed` library are measured and published (task 1.8c).
- Changing the specification would have reopened requirements already reviewed, to fit a package that nothing consumed yet.

## Consequence

- `Committed` sets `IsTrimmable` from `net6.0` and `IsAotCompatible` from `net7.0`, `Analyzed` sets `EnableTrimAnalyzer`, `OutOfScope` sets nothing. Each applies only where trimming exists, `net6.0` and later: `IsAotCompatible` on `net6.0` switches the trim checks off, so a `Committed` library would look enforced and not be.
- Any other value fails the build and names the three.
- The change reaches consumers when `Rinzler78.Build` is published, by task 1.8f.

## Source

`openspec/changes/001-ecosystem-foundation/specs/quality/spec.md`, task 1.8a of `tasks.md`, `Rinzler78.Toolkit` pull request #22.
