Prompt for a quality-gate report:

Build me an interactive report of this project's quality gate (the score the
build is gated on: a CRAP score, a weighted debt index, or whatever the
project computes), published as an artifact.

Find the gate first: the command CI runs to pass or fail on quality, the code
that computes it, and its baseline file. If the project has no single scored
gate, stop and tell me rather than inventing one.

DATA
- Take every number from the project's own gate, not a re-implementation. Run
  it on the current default branch (not a stale checkout) and record the commit.
  Use its report-only mode if it has one, and read its machine output: total,
  pass/fail, reasons, per-input points and every finding (input, file:line,
  detail, points).
- Inputs that need pipeline evidence (coverage, mutation, acceptance runs,
  flakiness, DAST, SAST, dependency CVEs) come from reports that are usually
  missing or stale locally. Pull them from the latest successful CI run on the
  default branch, into the paths the gate reads, and run any merge steps CI runs
  (e.g. sharded reports) before scoring. For example:
  - GitHub Actions: `gh run list --branch main --status success --limit 1`, then
    `gh run download <run-id> -n coverage -D coverage/`
  - GitLab CI: `glab ci artifact main <job-name>`, or
    `GET /projects/:id/jobs/artifacts/main/download?job=<job-name>`
  - Jenkins: `<job-url>/lastSuccessfulBuild/artifact/<path>`
  - CircleCI: `GET /project/:slug/<build-number>/artifacts`, then each artifact's
    `url`
  Record the run or pipeline ID for each artifact.
  If an artifact can't be fetched, leave that input missing exactly as the gate
  would. Never substitute zero.
- Keep three statuses distinct:
  - measured: clean, or with findings
  - pending: not wired yet
  - missing: no evidence, which blocks the gate and appears in its reasons
  Missing must never look like clean.
- Extract or compute what the report charts:
  - per-unit complexity, with module bodies counted as units
  - per-file stats: lines, summed complexity, units, worst unit, and kind
    (production, test or tooling)
  - a power-law fit of the complexity distribution: alpha ± SE, xmin, KS, and
    observed versus expected over the cap
  - lines of code per first-parent commit. Say so if the clone is shallow.
- Gates: ceiling, baseline, drift allowance. Show where each came from
  (environment override or default).
- Spot-check a dozen findings against the source before rendering:
  - does the line exist?
  - is the complexity right?
  - is the "unused" export really unreferenced?

SECTIONS (top to bottom)
- Verdict: the total in large type, pass or fail, every reason on its own line,
  ceiling, baseline + drift, and the timestamp.
- Budget bar: one stacked bar of the total, one segment per input, with the
  ceiling and baseline + drift as markers. Missing inputs appear as hatched
  zero-width flags so they can't disappear.
- Inputs table: input, points, status, note, and the weight rule in words
  ("complexity: 1 per point over 10", "unused export: 30 each", "SAST error: 40,
  suppressed: 0").
- Findings: grouped by input, every finding, sortable by points and by path.
- Complexity distribution: log-log CCDF of units with the fitted line from xmin
  and the cap marked; the most complex units listed.
- File size against complexity: log-log scatter colored by kind, with the
  fitted trend (exponent, r²). Name the outliers far above the line (dense
  logic) and far below it (data-heavy files).
- Lines of code over time: production and test as lines, tooling separately,
  and the test:production ratio on its own chart (never a second y-axis).

COLOR
- One hue per input family: static analysis, test evidence, security.
- Status uses semantic tokens (good, warning, critical) plus a hatched pattern
  for missing. Validate the palette for colorblind separation in light and dark
  with a real check, not by eye.
- Paths and unit names in monospace.

LAYOUT (d3 v7 from cdnjs)
- A single column that reads like a report, with the verdict and budget bar
  above the fold. Charts scale to the container, and every chart has a table of
  its numbers beside it, so nothing is reachable only by hovering.

INTERACTION
- Click a budget-bar segment or an input row to filter the findings to it;
  click again to clear.
- Search filters findings, units and files by path or name. Directory facet
  chips (one per top-level area) filter every section at once and update counts.
- Hover a scatter point for file, lines, complexity and worst unit. Click to
  open its findings and units in a side panel.
- What-if: a checkbox per finding, and one per input that toggles all of its
  findings (indeterminate when mixed). Unticking recomputes the total, the bar
  and the verdict against every gate, saying how many findings and points were
  left out. If the gate has its own what-if logic, reuse it.
  - Label it clearly as a simulation, with a reset button.
  - Missing evidence cannot be unticked: a blocker stays a blocker.
- Light and dark themes via CSS tokens; works at phone width.

VERIFY, THEN PUBLISH
- Render the page once in a headless browser and check:
  - the total matches the gate's output, and per-input sums match its
    per-input numbers
  - missing inputs show as missing
  - filters change the counts
  - the what-if restores the true total on reset and still refuses while an
    input is missing
- Fix what you see, then publish as an artifact.
- Tell me:
  - the total, pass or fail and why
  - which inputs came from CI (with the run ID) and which from the local run
  - what the analysers cannot see, for example name-based unused-export checks
    that read colliding identifiers as used, duplication windows shorter than
    the detector's minimum, complexity that skips type-level logic, and
    suppression comments the scanners trust without a reason
