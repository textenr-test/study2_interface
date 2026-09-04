# Text Enrichment Reader Study — Round 2

Static GitHub Pages experiment for a second, independent 30-reader Prolific cohort. Study version: `2026-09-04-r2-v1`.

Round 2 uses the same 38 document packages, enriched conditions, timing, rating task, and external Prolific flow as Round 1. It has a fresh 30-slot allocation namespace and shuffle seed, a separate collector service and private workbook, and no color-vision plate.

## Study design

- 38 Korean-language documents and six enriched conditions: D1, D2, W, D3, D4, and D5.
- Every participant completes 114 main trials in three sets of 38 and sees three distinct enriched versions of every document.
- Two optional breaks follow trials 38 and 76. A 60-second countdown is recommended, but participants can continue immediately. A clearly labeled attention check appears once per set after trials 12, 50, and 88; the required responses are +1, +3, and +1.
- Every trial uses a 750 ms fixation, a 1,000 ms simultaneous baseline/enriched display, and a −3…+3 left-to-right rating that is normalized to the enriched version.
- The longer member of each pair determines one shared scale. Only unused bottom whitespace is excluded from fitting; document text is never cropped.

All 30 participant allocations are pre-generated and checked into `assignments/` rather than sampled in the browser. The allocation satisfies all protocol constraints:

- 3 distinct conditions per participant-document.
- 19 occurrences of every condition per participant.
- 15 readers per document-condition pair.
- 5 readers per document-condition-set cell.
- 6 co-occurrences per document for every unordered condition pair.
- 6 or 7 occurrences per participant-set-condition cell.
- 19 baseline-left and 19 baseline-right trials in every participant-set.
- 7/8 or 8/7 side crossing within every document-condition pair.
- Different document positions across sets and no overlap between the last five documents of one set and first five of the next.

The committed master allocation contains 3,420 rows (`30 × 3 × 38`).

## Stimuli

`stimuli/` contains 38 self-contained JSON packages and 266 HTML stimuli imported from the Google Drive folder named `final output`. `scripts/import-final-output.mjs` verifies every HTML file against the SHA-256 recorded in its source manifest before packaging it. `stimuli/index.json` records the package hashes and source metadata.

The current source validation statuses are 35 pass and 3 warning. Review `P6_DOC_A`, `P13_DOC_A`, and `P13_DOC_B` before analysis.

## Data pipeline

The browser queues each response immediately, confirms each accepted row with the private Google Apps Script collector, and keeps the current set's trial payloads in local recovery storage. At trials 38, 76, and 114, the collector checks the whole set; the browser retries only missing rows and clears the local payloads after the checkpoint succeeds. Each accepted trial is written in two representations:

- `Trials`: flat, CSV-ready canonical record.
- `TrialJSON`: the same canonical record serialized as JSON.
- `Participants`: allocation, resumable progress, and screening/quality summaries.
- `Events`: consent, device and eligibility checks, comprehension attempts, server-side slot reservation, assignment loading, practice/main-study transitions, attention attempts, breaks, timing interruptions, connectivity, screen-outs, and completion.

The server writes `slot_reserved` itself during the locked reservation transaction. This makes the allocation auditable even if the browser disconnects immediately after receiving its slot. Pre-allocation lifecycle events start after consent; no participant data is sent before consent.

Before either break screen, the browser blocks until the server confirms every global trial index through 38 or 76. Final completion requires all indices 1–114, exactly 38 rows per set, three passed attention checks, two acknowledged break screens, and the final event.

`logs/final-trial-log-template.csv` and `logs/final-trial-log-template.json` are the pre-saved final schemas. `setupStudyWorkbook()` also creates empty `text-enrichment-final-log.csv` and `.json` files next to the private spreadsheet. A successful final submission refreshes both exports; an authorized researcher can also run `exportStudyLogs()` on demand. The collector exposes no public read or export route. The retained `color_test_json` compatibility column is intentionally blank in Round 2.

## Launch status

Researcher preview works without remote writes or Prolific redirects. Live Prolific entry requires all of these new Round 2 values in `study-config.js`:

1. `dataEndpoint`
2. `redirects.complete`
3. `redirects.screenedOut`
4. `redirects.incompatibleDevice`
5. `redirects.failedComprehension`
6. `redirects.noConsent`

Eligibility failures use the paid `screenedOut` path. An incompatible device and two failed instruction-check attempts use distinct `Request a return` paths. Declining consent uses the `noConsent` return path. Successful participants reach `complete` only after the server confirms the full 114-trial record.

The Round 1 collector URL and completion codes are deliberately not reused. Until a new Apps Script `/exec` URL and new Prolific completion URLs are entered, live entry is blocked by the configuration check while preview mode remains available.

The interface screens out anyone who reports Korean as a native or comfortably used language. Keep this exclusion aligned with the approved protocol, preregistration, recruitment copy, and analysis plan.

## Local checks

```sh
npm test
python3 -m http.server 4173
```

Preview:

```text
http://localhost:4173/?preview=1&slot=1
```

Fast automated-flow preview:

```text
http://localhost:4173/?preview=1&slot=1&fast=1
```

Preview mode never writes remote data, never redirects to Prolific, and exposes no participant-facing log.

Published Round 2 interface:

```text
https://textenr-test.github.io/study2_interface/
```

## Repository structure

- `app.js`, `styles.css`, `index.html`: participant interface.
- `study-config.js`: study version, timings, endpoints, checks, and breaks.
- `assignment.js`: validated loading of immutable slot files.
- `assignments/`: master CSV/JSON plus 30 participant slot files.
- `stimuli/`: Drive `final output` stimulus packages.
- `logs/`: canonical CSV/JSON log templates.
- `apps-script/`: private Google Sheet collector and export pipeline.
- `docs/prolific-setup.md`: deployment checklist.
- `tests/`, `scripts/`: design, schema, UI, and stimulus integrity checks.

Do not put credentials or private researcher tokens in this public repository. GitHub Pages serves code and stimuli only; all participant data must remain in the restricted researcher-controlled Sheet and Drive exports.
