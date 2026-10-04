Prompt for a CI/CD pipeline visualizer:

Build me an interactive anatomy of this project's CI/CD pipeline, published as an artifact.

DATA
- Read the workflow definitions (for GitHub Actions, .github/workflows/*.yml and *.yaml; adapt
  the same model for GitLab CI, CircleCI and the like) from the default branch,
  not from a stale checkout. Fetch first and record the commit. If the local
  branch is ahead and changes the workflow, read both versions and offer a switch
  between them: the default branch is the one that decides releases.
- Parse the YAML with a real parser, not regex. Use one that keeps comments
  (e.g. `yaml` for Node, ruamel for Python): the comments above jobs and steps
  are usually where the project explains its design. Watch for the first job's
  comment, which parsers often attach to the `jobs:` map rather than the job.
- One node per job. Expand matrix jobs into one node per combination, and
  substitute `${{ matrix.* }}` into names and upload-artifact names, so each
  shard's artifact is its own.
- Per job: id, display name, stage (from section comments or name prefixes),
  runs-on, timeout, services, container, environment, permissions, concurrency
  group, the `if:` as written, continue-on-error, fail-fast.
- Per step: name or first line of `run:`, `uses:` action with its ref and the
  version comment beside a pinned SHA, `if:`, continue-on-error. Map each task
  runner call (npm/pnpm/yarn script, make target, just recipe) to its definition
  and the source file it ends up running, following nested calls.
- Edges: `needs` (control flow) from every shard of a needed matrix job, and
  artifact flow from each upload to every download that takes it, by name or by
  `pattern` glob.
- If the pipeline has a quality or release gate that reads reports from other
  jobs, find which report files it expects by reading the gate's own source at
  the same commit, with its weights and thresholds. Do not copy them from
  memory or from docs. Map each artifact to the gate input it carries by
  tracing it end to end: the upload's artifact name and paths, then each
  download's `name`, `pattern` and `path`, to the files the gate reads. Don't
  assume an artifact is named after its report file or its gate input.
- Flag every artifact that is uploaded and never downloaded, and every gate
  input that nothing uploads. Expect to find one: a scan job whose report
  nothing reads is a common silent gap.
- Triggers: every `on:` event, and the concurrency rule (which runs cancel
  which).
- Timings: from the last ~20 completed runs on the default branch, median and p90
  per job and per step, and the failure rate per job (failures over runs that
  finished it, pass or fail). For example:
  - GitHub Actions: `gh run list --branch <default-branch> --status completed
    --limit 20 --json databaseId,headSha,conclusion`, then for each run
    `gh api repos/{owner}/{repo}/actions/runs/<id>/jobs`, whose steps carry
    their own timestamps
  - GitLab CI: `GET /projects/:id/pipelines?ref=<default-branch>`, then
    `GET /projects/:id/pipelines/:pipeline_id/jobs` (job durations only; step
    timings have to come from the job log's section markers)
  Keep "Set up job" and the post steps apart from the workflow's steps. Match API steps to workflow steps by name and, for repeated names
  (several download-artifact steps), by position among those. Say how many of
  those runs used the current workflow file; older runs ran older definitions.
  Before blaming a rule for cancelled runs, check that it existed then.
- Mark which runs released.
- Keep a one-sentence "why it is shaped this way" per job, distilled from its
  comment (e.g. why a stage is continue-on-error, why the gate uses always()).
- Spot-check every edge against the YAML before rendering.

LAYERS (columns, left to right)
- Trigger, then longest-path layers over `needs`, so the x-axis is pipeline
  order. Stack jobs within a column, grouped by stage, shards in order.

COLOR
- Encode what a job's failure does. Breaks the build: solid critical border.
  continue-on-error (does not fail the run; it costs gate points only if the
  gate's inputs and weights charge for it): dashed warning border. The gate: thick accent border. Jobs that hold a deployment
  environment or `contents: write`: a lock badge.
- Fill by stage. Edges: needs in neutral grey, artifact flow in the accent,
  unconsumed artifacts in critical.
- Validate the palette for colorblind separation in light and dark with a real
  check, not by eye. Critical red against warning amber fails for deuteranopes
  unless their lightness differs a lot. Magenta beside red fails too.
- Job and step names in monospace.

LAYOUT (d3 v7 from cdnjs)
- A deterministic layered DAG (d3-dag, or longest-path layering with barycenter
  crossing reduction), not a force simulation.
- Job cards show name, median duration, a mini step bar and the failure rate.
  The gate card lists its inputs as ports where artifact edges land.
- Leave room under a job for the stub and label of an artifact nothing
  downloads; otherwise the next card hides it.
- A timing ribbon below the DAG: a Gantt of a typical run, each job starting
  when its slowest need finishes, with the critical path highlighted there and
  in the DAG. Use fixed tick steps (every 5 minutes), not d3's default ticks.
- Fit to the viewport on load; scroll to zoom, drag to pan.

INTERACTION
- Hover a job: highlight everything upstream and downstream (needs and
  artifacts), dim the rest. Clears on mouse-out.
- Click a job for a side panel: settings, its `if:` in plain words, why it's
  shaped this way, steps with durations, each script step linked to its
  definition and source file, each `uses:` with ref and version.
- Click an artifact edge: producer, consumer, paths, and the gate input it
  feeds with its weight.
- "What if this job fails / is skipped / is cancelled": propagate through
  needs, `if:` and continue-on-error and show what still runs, the gate's
  verdict, whether a release goes out, and whether the workflow is green.
  Get the semantics right:
  - An `if:` with no status function gets an implicit `success()`, so a failed,
    cancelled or skipped need skips the job.
  - `always()` runs the job regardless.
  - A cancelled job still runs its `if: always()` steps. Its report may or may
    not have been written before the cancel: check whether the expected file
    gets uploaded, and mark the gate input missing only when it doesn't.
  - A failed job leaves a report only if its tool writes one before exiting
    non-zero. Check each tool's exit behaviour.
  - With `fail-fast` (on by default), a shard that fails without
    continue-on-error cancels its queued and running siblings. Mark them
    cancelled before working out what runs, which artifacts exist and what the
    gate sees.
  - Matrix results aggregate: failure beats cancelled beats skipped.
  - A missing gate input is a blocker. A report from a failed producer is
    charged, and it blocks only if even the smallest charge exceeds the gate's
    allowance.
- Also check for the case where a job's failure turns the workflow red but the
  release still goes out because the release needs only the gate. Report it if
  you find one.
- Trigger switch over the workflow's own events greys out jobs that wouldn't
  run (e.g. release on a pull request).
- Median/p90 toggle. Search over job and step names.
- Light and dark themes via CSS tokens; works at phone width. A wide SVG inside
  a grid item needs `min-width: 0` on the item, or the page scrolls sideways.

VERIFY, THEN PUBLISH
- Render the page once in a headless browser and check:
  - every job (and shard) is present, every needs edge is drawn, and artifact
    edges match upload/download names
  - cancelling one shard of a sharded reporting job makes the gate refuse
  - a failing job whose upload is `if: always()` still delivers its report,
    while cancelling it makes that input missing
  - the pull-request trigger hides the release
  - no console errors, and no horizontal scroll at 390px
- Fix what you see, then publish as an artifact.
- Tell me:
  - the job and step counts, the critical path and its median and p90 time,
    and the slowest steps
  - any unconsumed or unproduced artifacts, any red-but-released case, and any
    action not pinned to a SHA
  - what the anatomy cannot see: action internals and composite or reusable
    workflows it didn't expand, images pinned by tag, runner queue time, and
    anything decided outside the workflow file (branch protection, required
    checks, environment rules, whether secrets and variables are set,
    organisation policy)
