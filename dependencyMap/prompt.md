Prompt for a dependency visualizer:

Build me an interactive dependency map for this project, published as an artifact.

DATA
- Extract from the current source on the default branch, not a stale checkout.
  Record the commit.
- Use the compiler or language server, not regex: the TypeScript type checker
  from tsconfig.json, rust-analyzer, Roslyn, the JDT, go/types. Resolve every
  identifier through it, following imports, re-exports and aliases.
- One node per building block: a top-level function (including a constant
  holding a function), a class or struct, or a type (interface, alias, enum,
  trait). Id is path#name.
- Nested things belong to their block: a method's dependencies are its
  class's, and a closure's are its function's. A call on an instance
  (`board.move()`) resolves to the instance's type.
- Every edge has a kind and a weight. uses = value reference (call,
  construction, read), type = type position only, extends = extends/implements.
  Weight = the number of references.
- Area = package, crate, module or app. Scope is production code only by
  default; tests and tooling are opt-in toggles, off on first load.
- Per block: afferent coupling Ca (blocks that depend on it) and efferent Ce
  (blocks it depends on), counted in blocks, not references. Instability
  I = Ce / (Ca + Ce), null when isolated.
- Detect cycles (strongly connected components larger than one block) and list
  them.
- Layering: if the project declares layers or architecture rules (a layers
  file, ArchUnit, dependency-cruiser, import-linter, a section in the README),
  take them from there, not a re-implementation. Otherwise derive a proposal
  from the directory structure and say it's a proposal.
  - A layer may depend on itself and on layers below it, never above
    ("upward").
  - Areas marked isolated (e.g. client and server apps) must not depend on
    each other ("sideways").
  - A violation is a pair of blocks, of any edge kind, with its rule, summed
    weight and kinds, heaviest first.
- Spot-check a dozen edges against the source before rendering, including at
  least one re-export and one instance-method call.

LAYERS (horizontal bands, top to bottom)
- The project's layers, highest first (typically tests and tooling, apps,
  shared packages, contracts, domain).
- Each band shows its name and block count on the left. Within a band, every
  area gets an outlined region with its name written large and faint behind it
  (at most 120px, shrunk to fit, about 35% opacity), behind edges and blocks.

COLOR
- Fill by kind (function, class, type). Border thickness by Ca, so heavily used
  blocks stand out. A red outline marks blocks in a cycle.
- Edge style by kind: solid for uses, dashed for type, hollow arrow for
  extends. Edges that break the layering in the critical color.
- Validate the palette for colorblind separation in light and dark with a real
  check, not by eye.
- Labels in monospace.

LAYOUT (d3 v7 from cdnjs, force simulation)
- forceY pins each block to its band, forceX pulls it toward its area's centre,
  link strength grows with log(weight), and charge is moderate.
- Blocks are pills, so use rectangular collision, not forceCollide circles.
  Keep every block inside its band and area region: a final force corrects
  velocities so the next step can't cross a region edge, and dragging is
  clamped the same way. Pre-run ~300 ticks so the first frame is settled.
- Edges are gentle curves from dependent to dependency, width by log(weight),
  with arrowheads that stop at the pill border.
- Fit to the viewport on load, band labels included; scroll to zoom, drag to
  pan.

INTERACTION
- Hover a block: highlight what it depends on and what depends on it, dim the
  rest. Clears on mouse-out. Don't re-order the hovered element in the DOM
  during hover; that breaks mouseleave.
- Click to select; click the background to clear.
- Free body diagram in the side panel for the selected block:
  - the block in the middle, up to 10 dependents pulling in from the left and
    up to 10 dependencies pulling out to the right, arrow thickness by
    references, kinds listed per force
  - under it, a resultant arrow placed by instability from stable (left) to
    unstable (right), with Ca, Ce, I and file:line
  - dependents and dependencies as clickable lists that select in turn
- Filters: kind chips, edge-kind chips, area chips, "exported only", and scope
  toggles for tests and tooling. Each removes nodes and edges from the
  simulation, updates counts and lets the rest re-settle. Hidden neighbours
  show greyed in the panel.
- "Cycles" and "Layering violations" buttons zoom to and highlight each one in
  turn.
- Under the diagram, list the layers with their rules, and a table of
  violations (from, to, area, rule, weight) whose rows select the dependent. A
  checkbox hides the dependencies that keep to the layering.
- Drag a block to pin it, double-click to release, and an "Unpin all" button.
  Search highlights matching names. "Fit" and "Re-run layout" buttons.
- Light and dark themes via CSS tokens; works at phone width.

VERIFY, THEN PUBLISH
- If the project already has a dependency or architecture check, compare
  block, edge and violation counts with it for the same scope, and explain any
  difference.
- Render the page once in a headless browser and check:
  - no block outside its band or area region, both at rest and after dragging
    one past the edge
  - no overlap at rest, and area names inside their regions
  - the highlight clears on mouse-out, and filters change the counts
  - the free body diagram matches the Ca/Ce numbers
- Fix what you see, then publish as an artifact.
- Tell me:
  - block and edge counts by kind and area
  - the five most depended-on blocks, and the most unstable blocks that are
    still heavily used
  - every cycle and layering violation
  - what the extraction cannot see: dynamic imports, reflection and
    string-keyed lookups, dependencies injected at runtime, and anything
    outside the compiler's file list
