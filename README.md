# AI.FO Engine Telemetry

Machine-generated telemetry for the AI.FO financial engine. `telemetry.json` is
regenerated after every green nightly pressure-test run and mirrored here
automatically. It is never hand-edited.

The file reports, each measured at a specific commit:

- the engine and API test-suite counts,
- the assertion-audit breakdown (sourcing, property, snapshot, other),
- the financial-signal and industry-track counts,
- the nightly pressure-test ledger (runs, synthetic companies, snapshot assertions).

The numbers shown at https://getaifo.com/how-we-test are served from this file.
The source of truth is the engine's own test suite and assertion classifier; the
artifact is produced by `generate-telemetry.js` in the engine repository and
copied here on each change. The git history of this file is the public record of
how those numbers have moved over time.

If the `stale` flag is ever set on the live endpoint, the published artifact has
not refreshed within its tolerance window and the live engine cross-check is
shown instead.

## The assertion accounting (three figures, no substitutions)

Beginning with the first promoted nightly run after 2026-07-27, the artifact
carries an `assertionAccounting` block with three distinct figures. Each answers
a different question, and none ever stands in for another:

- **activeAssertions**: the regression evidence that executes tonight. The
  nightly suite uses a bounded retention window (the trailing thirty days of
  snapshots plus one representative per calendar month of history), so this
  figure can DECREASE when aged snapshots move to the in-repo archive. A
  decrease here is retention working, not evidence lost.
- **lifetimeAuthoredAssertions**: the sum over every snapshot ever authored,
  recorded in an append-only ledger committed to the engine repository.
  Archived snapshots keep their ledger entries, remain in the repository, and
  are replayable against their recorded engine commit. This figure never
  decreases.
- **cumulativeVerifications**: successful verification work performed to date,
  summed across a per-run ledger that records every nightly execution. Runs
  that did not complete green contribute zero, a deliberate undercount.

The reason for the split: a single headline number invites conflating what ran
tonight with what has accumulated over months. Publishing all three, labeled,
is the honest version.

## What a green night certifies

The nightly cohort is generated in three segments: a randomized health-tier
mix, revenue-band members spanning fifty thousand to fifty million dollars in
annual revenue, and deterministic archetype companies. An armed coverage gate
requires every registered financial signal to fire in its mapped archetype
member; if any does not, the run fails and no snapshot or telemetry promotes
(the failure itself is recorded). So a refreshed `telemetry.json` here means:
the full test suites passed, every registered signal demonstrated its firing
behavior that night, and the run's snapshot evidence was committed to the
engine repository.

The canonical definitions of the cohort bands, sampling rules, and assertion
accounting live in the engine repository's `docs/SIGNAL_METHODOLOGY.md`, the
single methodology authority. This README describes; that document defines.

## Provenance chain

1. The nightly runs from a dedicated clone, hard-reset to the engine
   repository's `master` before every run.
2. On a green run, `generate-telemetry.js` regenerates `telemetry.json` from
   the suites, the assertion classifier, the signal registry, and the committed
   ledgers. The generator refuses to run on a dirty tree or a red suite.
3. The artifact commits to the engine repository, stamped with the commit it
   measured (`commit` and `generatedAt` fields).
4. A repository workflow copies it here whenever it changes on `master`.
5. The live endpoint at getaifo.com reads this mirror at request time.

No number in the artifact is typed by a person. Values are derived at
generation time or the generator refuses.
