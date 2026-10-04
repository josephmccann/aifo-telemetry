# AI.FO Engine Telemetry

Public, machine-generated test telemetry for the AI.FO financial engine. The
repository holds one file, `telemetry.json`, and its git history is the public
record of how the figures have moved over time. Nothing here is hand-edited.

## Status

Active. The file is replaced automatically after each green nightly
pressure-test run that promotes new figures (one commit per change, subject
`mirror telemetry from AI.FO-Demo@<sha>`). The `generatedAt` and `commit`
fields say when and against which engine commit the figures were measured.

## What the file reports

- `suites`: engine and API test-suite counts (passed, failed, skipped, total).
- `assertionAudit`: assertion counts by class (sourcing, property, snapshot,
  other) and by scope.
- `signals`, `tracks`: registered financial signals and industry tracks.
- `nightly`: the pressure-test ledger (runs, synthetic companies, snapshot
  assertions, first and last run IDs, last promotion time).
- `assertionAccounting`: three separate figures, never substituted for each
  other:
  - `activeAssertions`: what the nightly suite executed. It can decrease when
    aged snapshots move to the in-repo archive; that is retention, not lost
    evidence.
  - `lifetimeAuthoredAssertions`: every assertion ever authored, from an
    append-only ledger. It never decreases.
  - `cumulativeVerifications`: verification work summed across every nightly
    run. Runs that were not green contribute zero, a deliberate undercount.
- `cohortComposition`: the shape of the latest nightly cohort (tier-mix,
  revenue-band, and archetype companies; annual revenue span).
- `fieldNotes`: plain-language notes on what each figure counts, when it can
  decrease, what a green night certifies, and what it does not prove.
- `definitions`: the labeling contract for consumers (see below).

## Reading the `definitions` block

- `definitions.freshness` names the fields to show (`generatedAt`, `commit`,
  `nightly.lastRunId`, `nightly.lastPromotedAt`) and `maxAgeHours` (26).
  Render "as of {generatedAt} (run {lastRunId}, commit {commit})" with a
  derived fresh or stale state; never render a static "Live". Treat the
  figures as stale when `generatedAt` is older than `maxAgeHours` or when
  `nightly.lastPromotedAt` is behind the newest scheduled nightly (compare
  promotion time, not the run-id date alone). Consumers of `/api/telemetry`
  read its `stale` and `staleReasons` fields instead; consumers of this raw
  file apply the rule themselves.
- `definitions.headline.perRun` and `definitions.headline.cumulative` list the
  headline figures. Each entry has a `key`, `label`, `field` (a dotted path into
  this file), `kind`, `definition`, and `computedFrom`. Resolve each value from
  its `field` rather than hardcoding numbers or labels, render `definition`
  verbatim, and never show a `cumulative` figure under a per-run label.

## What a green night means

The nightly cohort mixes randomized health tiers, revenue bands spanning
$50,000 to $50,000,000 in annual revenue, and deterministic archetype
companies. A coverage gate requires every registered signal to fire in its
mapped archetype; if one does not, the run fails and nothing promotes. A new
`telemetry.json` here therefore means the full suites passed, every registered
signal fired as expected that night, and the run's snapshot evidence was
committed to the engine repository.

## How it is produced and deployed

1. On a green nightly run, `generate-telemetry.js` in the private engine
   repository regenerates `telemetry.json` from the test suites, the assertion
   classifier, the signal registry, and the committed ledgers. It refuses to
   run on a red suite or a dirty working tree.
2. The artifact is committed to the engine repository's `master` branch.
3. A GitHub Actions workflow there (`mirror-telemetry`) copies the file to
   `main` in this repository whenever it changes on `master`. It always
   mirrors `master`, and skips the commit when the file is unchanged.

There is no build, CI, or deploy step in this repository.

## Consumers

- `https://app.getaifo.com/api/telemetry` reads the raw file from this
  repository (`main/telemetry.json`), caches it for five minutes, falls back to
  a bundled copy if the mirror is unreachable, and adds `stale`,
  `staleReasons`, and `staleStats` from its own freshness and live-registry
  checks.
- `https://getaifo.com/api/telemetry` proxies that endpoint, and the page at
  https://getaifo.com/how-we-test renders it.

## Running and testing locally

There is no code to run. To check the file is valid JSON:

```sh
python3 -m json.tool telemetry.json > /dev/null && echo ok
```

## Environment variables

None. This repository has no code or configuration.

## Where the definitions live

The methodology (cohort bands and sampling, assertion accounting) is defined in
`docs/SIGNAL_METHODOLOGY.md` in the private engine repository. This README
describes the published file; that document defines it.
