Prompt for turning a tool seed into a repo tool:

Here is a tool seed (a prompt that describes an artifact an agent builds by
hand, e.g. another prompt.md from this library). Don't build that artifact. Build
a tool in this repository that produces it, so that one command gives
byte-identical output for the same inputs, with no agent involved.

SHAPE
- Follow the project's existing tools: where they live, how they are named, how
  they are wired into the task runner (package.json scripts, Makefile, justfile,
  cargo xtask...). Read two of them before writing anything.
- An entry point that only does I/O (files, git, network, arguments) and a pure
  core beside it: sources at one commit in, model out. No file system, git,
  network, clock or randomness in the core.
- Outputs:
  - results/<name>.json: the model, with keys and arrays sorted and ids stable
  - results/<name>.html: one self-contained page, with libraries inlined from
    the local dependency tree, not a CDN
  - a `--markdown` (or similar) flag that prints the summary the seed's
    "Tell me…" paragraph asks for
- Timestamps come from the commit (`git show -s --format=%cI`), never from
  now(). Record the SHA. Say so when the clone is shallow.

REMOVE EVERY NON-DETERMINISTIC STEP IN THE SEED
- Network data (e.g. `gh run list` / `gh run download` on GitHub, `glab` or the
  REST API on GitLab, any HTTP API): a separate `--fetch`
  step writes the raw responses to a cache directory. The default run reads
  only local files. Anything absent is "missing", never zero or empty.
- Layout: compute it in the core and put coordinates in the JSON (fixed tick
  count, seeded, no Math.random). The page only draws, and drags from there.
- Simulations and what-ifs: precompute every outcome in the core so the page
  looks them up instead of re-deriving the rules in browser JavaScript.
- Rules the seed says to "take from the project" (gate weights, layers,
  thresholds): import them from a pure module. Don't copy them, and don't
  import a script with top-level side effects to get at its constants. If the
  rule must be read at another commit, parse it from that commit's source and
  test the parser.
- The page's own script ships as a string inside the page. Keep it free of
  decisions so it doesn't need testing beyond "it parses and renders".

TURN EVERY JUDGEMENT STEP INTO A TEST
- "Spot-check N items against the source" becomes unit tests with small fixture
  inputs, one for each edge case the seed names.
- "Render in a headless browser and look at it" becomes a browser test with
  one assertion per check in the seed's VERIFY section.
- "Compare counts with …" becomes a test that the JSON counts match the
  source.
- Add a golden test: run twice on a fixture and get identical outputs.

WIRE IT IN
- Add it to the CI job it belongs to and upload results/<name>.* as an
  artifact.
- Keep the new code inside the project's own quality bar. If the project
  gates on complexity, duplication or unused exports, run that gate and get
  the new code to zero points.
- Run the type checker, the linter (zero warnings if the project has zero),
  the formatter and the tests.
- Write scratch files (screenshots, probes) outside the repository.
- Replace the seed in the project with a short note: "run <command> and publish
  results/<name>.html". Keep the seed's old body as the spec the tests enforce.

DONE WHEN
- A clean checkout runs the command twice and gets the same results/<name>.*
  both times.
- Every sentence in the seed's VERIFY section has a test that fails if it
  breaks.
- Tell me the command, what it writes, what `--fetch` caches, and which seed
  requirements could not be made deterministic, and why.
