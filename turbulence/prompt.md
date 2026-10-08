Prompt for a churn vs complexity map:

Build me an interactive churn vs complexity map for this project, published
as an artifact. Every source file is a point placed by how often it changes
and how hard it is to work in, so the files that are both stand out as
refactoring candidates. This follows Michael Feathers, "Getting Empirical
about Refactoring"
(https://www.stickyminds.com/article/getting-empirical-about-refactoring),
and the turbulence gem that put it into practice for Ruby
(https://github.com/chad/turbulence).

DATA
- Analyze the history reachable from the default branch's current commit,
  not `--all`: churn on unmerged or abandoned branches is not churn in the
  code I'm reading. Don't use `--first-parent` either; with `--no-merges` it
  drops every commit that arrived through a merged pull request. Record
  the commit, the history window and the analysis date. If the clone is
  shallow, say so and say how far back the history goes.
- Unit = file by default, with functions inside each file as a drill-down.
  Production code only by default; tests, generated code, vendored code,
  lockfiles, fixtures and build output are excluded, each excluded group with
  a count and a toggle. Test code inside production files counts as tests
  too: Rust `#[cfg(test)]` modules, Python doctests, inline spec blocks. Strip
  it from complexity and size, and report how much was stripped. Take generated and vendored roots from the build
  setup and .gitattributes (`linguist-generated`, `linguist-vendored`), not
  from guessing.
- Churn, per file, over a stated window (default: all history, with a
  last-12-months toggle):
  - commits that touched the file, and lines added plus deleted;
  - leave out the commit that created the file: being written is not churn;
  - follow renames (`git log --follow`, or `-M` with the `old => new` and
    `{a => b}` numstat forms resolved to the current path). A renamed file
    must not lose its history, and the old path must not show up as a ghost.
    When analyzing a subdirectory, read the log for the whole repository and
    keep what resolves into it: a path-filtered log silently drops history
    from before the directory itself was renamed;
  - skip merge commits, and skip the revisions listed in
    `.git-blame-ignore-revs` (mass reformatting, license headers). If that
    file doesn't exist, list the commits that touched more than ~25% of
    files and ask me whether to ignore them, defaulting to keeping them;
  - also record distinct authors and the date of the last change.
- Complexity, per function, from the language's own tool, not a regex: flog
  for Ruby, radon or lizard for Python, eslint complexity or the TypeScript
  compiler API for TS/JS, gocyclo/gocognit for Go, clippy's cognitive
  complexity or rust-code-analysis for Rust, lizard as the fallback for
  anything else. Say which tool and which metric (cyclomatic, cognitive,
  ABC/flog).
- Use one metric per plot. Cyclomatic, cognitive and flog scores sit on
  different scales, so in a polyglot repository either pick a metric every
  language's tool reports (cyclomatic is the common one) or give each
  metric its own plot, thresholds and ranking. Never put unlike raw scores
  on one axis.
- Check the tool before trusting it: compare the number of functions it
  reports per file with the number the language declares, and look at the
  three longest functions' scores. Lightweight parsers can lose function
  boundaries partway through a file, or barely count `match`/`switch` arms,
  and both make complex code look simple. If the check fails, switch tools
  and say why.
- Roll function complexity up to the file two ways, and show both: total
  (what turbulence uses, which grows with file size) and the worst function.
  A long file of simple functions and a short file with one monster are
  different problems.
- Spot-check five files by hand: the churn against `git log --follow
  --numstat` for that path, the complexity against running the tool on that
  one file.

QUADRANTS
- Both axes use `log1p` (log(1 + value)), so files with zero churn or zero
  complexity still plot, at the origin. Points, thresholds, zoom and the
  ranking all use the same transform; tick labels show the raw values.
- Split each axis at a stated threshold (default: the median of the raw
  values, drawn through the same transform) and name the quadrants:
  - upper right, Danger Zone: complex and changes often. Refactor here first.
  - lower left, Healthy Closure: simple and stable. Leave it alone.
  - upper left, Cowboy Code: complex but rarely touched. Watch it.
  - lower right, Fertile Ground: simple but changing often, often config
    or a new abstraction that hasn't been extracted yet.
- Rank Danger Zone files by score = sqrt(u² + v²), where u = log1p(churn) /
  log1p(max churn) and v = log1p(complexity) / log1p(max complexity), both
  over the files in view, highest first. Show the top ten as a list and say
  the formula on the page.

LAYOUT
- Scatter plot: x = churn, y = complexity, both on the `log1p` scale, with
  the threshold lines and quadrant names drawn faint behind the points. One
  point per file, coloured by top-level directory (validate the palette for
  colourblind separation in light and dark), sized by lines of code.
- Label the top ten without overlap. Fit to the viewport; scroll to zoom,
  drag to pan.
- A treemap view as a toggle: area = lines of code, colour = quadrant, nested
  by directory. Directories get their own churn and complexity rolled up.
- A table under the chart with every file: path, churn (commits, lines),
  complexity (total, worst function), authors, last changed, quadrant.
  Sortable, and the rows select the point.
- Light and dark themes via CSS tokens; works at phone width.

INTERACTION
- Hover a point: path, both metrics, quadrant. Click: a side panel with the
  file's functions ranked by complexity with file:line links to the forge at
  the recorded commit, a sparkline of its churn per month, its authors and
  its five largest commits linked.
- Time slider: recompute churn for windows ending at earlier commits (one
  per month or per tag), with complexity from that commit, and animate the
  points between them. Files moving into the Danger Zone are the warning;
  files moving out show which refactorings worked. Precompute the frames.
- Filters: directory chips, quadrant chips, the exclusion toggles, the churn
  window, a toggle between total and worst-function complexity. Counts and
  thresholds update.
- Search highlights matching paths.

VERIFY, THEN PUBLISH
- If the project already runs turbulence, code-climate churn, or a similar
  hotspot tool, compare its top ten with yours and explain each difference
  (window, renames, ignored revisions, metric).
- Check that no excluded file (generated, vendored, tests when off)
  contributed a point, and that no deleted or old renamed path appears.
- Render once in a headless browser: labels don't overlap, quadrant lines sit
  at the stated thresholds, the slider frames move points, the side panel's
  links point at real lines.
- Fix what you see, then publish as an artifact.
- Tell me:
  - files analyzed and excluded by group, the history window, the tools and
    metrics used
  - the Danger Zone top ten with a sentence on why each is there
  - files that entered or left the Danger Zone in the last year
  - commits ignored as mass changes, and renames followed
  - what the numbers can't tell you: churn from a big planned migration looks
    like instability, complexity tools disagree with each other, code moved
    between files resets both metrics, and churn says nothing about bugs
    unless you also join it with fix commits
