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
