Prompt for a visualizer:

Build me an interactive call-graph page for this project, published as an artifact.

DATA
- Extract the call graph from the current source on the default branch, not from
  whatever stale branch is checked out. Record the commit it was built from.
- One node per function or method. Qualify names that collide (Type::name, and
  module::Type::name when two modules define the same type).
- Resolve calls as precisely as a regex pass allows: self.x(), Self::x(), Type::x(),
  module::x(), calls through wrapper fields (self.inner.next(), self.0.step()),
  parameter and local variable types, and match-arm bindings. Expand any
  macro_rules that generate impl blocks so macro-made types get real nodes and
  edges. Drop calls you cannot resolve rather than guessing; names that also
  exist in the standard library (next, len, insert, fmt, eq...) never match on
  an unresolved receiver. Skip test functions.
- Spot-check a dozen edges against the source before rendering.

LAYERS (horizontal bands, top to bottom)
1. Public API: inherent public methods on public types, excluding anything the
   project marks as not-really-public (doc(hidden), an "internal" module or
   feature, test/bench-only knobs).
2. Trait plumbing: trait impl methods on public types (iterator impls, std
   traits, serde and the like). Public by construction, not by intent.
3. Operation internals: private helpers belonging to one operation, plus the
   hidden knobs from layer 1.
4. Shared utilities: the common helpers, memory layout and allocation modules
   that every operation uses.

COLOR
- Color nodes by operation family (e.g. construct/drop, get/entry, insert/bulk,
  delete, iterate/range, set, validate/inspect), with a neutral grey for shared
  utilities. Validate the palette for colorblind separation in light and dark.
  Dashed border for feature-gated functions. Every node is labeled in
  monospace with its name.

LAYOUT (d3 v7 from cdnjs, force simulation)
- forceY pins each node to its band; weak link force; moderate charge; no
  strong pull toward the center.
- Nodes are pills, so use rectangular collision, not forceCollide circles, and
  clamp every node inside its band after each tick. Pre-run ~200 ticks so the
  first frame is already layered.
- Edges are gentle curves from caller to callee with an arrowhead that stops at
  the pill border. Band labels at the left show the layer name and count.
- Fit the whole graph to the viewport on load; scroll to zoom, drag to pan.

INTERACTION
- Hover a pill: highlight its callers and callees in their family colors, dim
  everything else. The highlight must clear when the pointer leaves (do not
  re-order the hovered element in the DOM during hover; that breaks mouseleave).
- Click a pill to keep it selected; click the background to clear.
- Side panel for the selected function: file:line, visibility, layer, callers
  and callees as clickable lists that select in turn.
- Legend chips per family toggle whether those functions appear in the graph
  at all (remove nodes and their edges from the simulation, update counts,
  let the rest re-settle). Hidden callers/callees show greyed in the panel.
- Drag a pill to pin it; double-click releases; an "Unpin all" button.
- Search box highlights matching names. "Fit" and "Re-run layout" buttons.
- Light and dark themes via CSS tokens; works at phone width.

VERIFY, THEN PUBLISH
- Render the page once in a headless browser and look at it: no pills outside
  their bands, no overlap at rest, highlight clears on mouse-out, chip toggles
  change the node count. Fix what you see, then publish as an artifact.
- Tell me the node and edge counts, the layer breakdown, and what the
  extractor cannot see (closures, trait objects, std adapters).
