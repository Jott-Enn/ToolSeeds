Prompt for a feature parity matrix:

Build me an interactive feature parity matrix for this project, delivered as
an artifact. Rows are features; columns are the implementations that should
provide them. Each cell explains its status with evidence I can click through
to. Adapt the analysis to the project's architecture and available sources.

SCOPE AND CLIENTS
- Analyze the supplied project or current workspace. Record the repositories,
  revisions, uncommitted changes and analysis date. State any inaccessible
  repositories or missing history; keep conclusions within the inspected scope.
  State the build, platform and configuration scope used to judge support.
- Find every relevant product entry point: apps, CLIs, SDKs, bots, plugins,
  integrations or other independently usable implementations. Call these
  clients below. Use manifests, build definitions, package metadata and READMEs
  to identify them, rather than directory names alone. Distinct implementations
  for the same platform get distinct columns. If only one exists, show its
  coverage and say that cross-client parity is not available.
- Record each client's role, languages, production source roots and relevant
  shared dependencies, including versions where available. Shared libraries,
  engines or services that implement features are cores. Show cores separately
  from clients and map the dependencies: a client can use several cores, and
  cores can depend on other cores. An independently usable SDK with both roles
  appears once among clients, marked as shared, and counts once in client
  metrics. Dependencies reference that same identity.
- Count production implementation evidence separately from tests. Exclude
  generated bindings, vendored dependencies and build output from independent
  support counts; determine their roots from the build and generation setup.
  Examples include generated FFI wrappers, Generated/, *.g.cs, *.xcframework,
  node_modules and output directories. Generated API declarations can mention
  every feature without any client using them. Inspect them to trace mappings,
  but never treat their existence as client integration. Where behavior is
  generated, trace its source definition and actual use, or mark it unknown.

FEATURES
- Build the catalog from the sources that exist, merge duplicates and retain
  every origin and source reference:
  - Contract and implementation: public APIs, schemas, routes, commands,
    events, capability registries and user actions. Read the definitions and
    dispatch or registration code. Include client-local capabilities that have
    no server or shared API. Documentation alone is not implementation evidence.
  - Audit: the project's parity audits, feature lists and plans. Import their
    rows as claims, retaining the document date, revision and claimed scope.
    Mark an unavailable date as unknown.
  - History: changes that introduce or extend a feature across clients. Commit
    subjects, scopes, issue references and touched paths suggest candidates;
    inspect the changes before treating them as the same rollout.
    Report unavailable catalog sources instead of inventing their contents.
- Group rows into areas appropriate to the product. Use stable feature IDs
  and names a user would recognize. Group low-level operations into coherent
  features, while retaining the operations as requirements and fingerprints.
- Before scoring, define each feature's required behaviors for each relevant
  client role. These might be producing and consuming an event, importing and
  exporting a format, or configuring and executing an operation. An SDK may
  need a callable API; a graphical client may need a reachable user action.
  Do not assume every feature has send/receive directions or needs a GUI.
- Record applicability and its rationale. A missing implementation is not a
  reason to mark a feature not applicable. Label inferred requirements and
  applicability decisions so they can be reviewed.

EVIDENCE
- Define fingerprints per layer: protocol tokens or endpoint paths where
  relevant; API methods, event types, callbacks and exported symbols; and
  client-specific actions, views, services or configuration. Match exact
  symbols and literal values, following aliases and constants. Handle quoting
  through parsing; normalize prefixes or token variants only when the project's
  interface defines them as equivalent.
- Use fingerprints to locate candidates, then inspect their executable or
  declarative context and relevant call paths. Prefer language-aware tools
  where available. A name match, import, declaration, comment or diagnostic
  string alone does not establish support. Report extraction fallbacks.
- Follow the feature through the actual dependency chain. A client may prove
  integration by calling a shared API without ever containing a wire token.
  For independent implementations, inspect their own implementation path.
  Record the relevant dependency version; support in a newer core does not
  establish support in a client pinned to an older one.
- For each requirement, record the supporting or contradictory evidence,
  file:line, revision, and any unresolved reachability, configuration, platform
  or feature-flag conditions. Distinguish source-supported conclusions from
  behavior actually verified by running a test.
- Classify cells using explicit, reproducible rules. Show the rule that fired.
  Apply n/a first, then unknown if unresolved evidence could change the status;
  otherwise choose full, partial, core only or absent in that order:
  - n/a: outside this client's intended role, with a reason.
  - unknown: access, indirection or incomplete evidence prevents classification.
  - full: all required behaviors have implementation and integration evidence
    appropriate to this client's role, under the recorded conditions.
  - partial: at least one required behavior has client implementation or
    integration evidence, but others are missing or demonstrably not integrated.
    Explain exactly which. Support confined to a core is core only.
  - core only: a relevant core version implements the feature, but the client
    has no integration evidence after its accessible paths have been inspected.
  - absent: no implementation evidence found in the documented, inspectable
    scope. This is a scoped finding, not proof about inaccessible code.
    Assess core columns against their own responsibilities. Declarations alone
    do not make a core full, and core only is a client status.
- Record related test files separately, including fingerprint hit counts.
  Test references are not proof of coverage or passing tests. State which
  tests were actually run and their results, if any.
- Record the last commit touching an evidence file as file recency. Do not
  present it as the feature's introduction date or last behavioral change.

AUDIT DIFF AND ROLLOUTS
- Preserve every imported audit claim. Compare it with current evidence when
  feature, client and scope are comparable; show disagreements with both
  statuses, their dates or revisions, and the reason. Keep unverified or
  incomparable claims distinct from contradictions. A discrepancy alone does
  not establish which source is stale.
- Cluster rollout candidates using the actual feature changes. Similar commit
  subjects or touched client directories are clues, not sufficient evidence.
  Keep uncertain groupings visible as uncertain.
- For each verified rollout, show which clients received its actual behavior,
  the supporting commits, and which still lack it at the analyzed revisions.
  A shared change counts for a client only when its integration and dependency
  version deliver that behavior.
  Lag is days since the first verified client implementation, using a stated
  timestamp convention. This measures implementation history, not release or
  user adoption. Account for integration through shared dependencies.
- Distinguish missing implementation, unknown support, and verified
  implementation with an unknown date. Missing or shallow history must not
  become a claim that a client never received a feature.

LAYOUT
- Use a matrix with a sticky header and feature column. Group clients first
  and cores separately, with collapsible feature areas. Each status has a
  glyph, text label and color so it can be understood without color.
- Per client, show full / applicable features, excluding n/a. Include unknown
  in the denominator and display its count separately. Show the fraction as
  well as the percentage; use n/a for an empty denominator. Core columns do
  not contribute to client parity comparisons.
- Per row, compute spread using only client cells classified full, partial,
  core only or absent: their total minus the largest status-group count. Show
  that total and the unknown count alongside it; label ties and use n/a with
  fewer than two known cells. Scores and counts must state whether they cover
  all data or the filtered selection.
- Selecting a cell opens a side panel with its status, decision rule, required
  behaviors, conditions, fingerprints and hit counts, implementation evidence,
  related tests, file recency and any audit claims.
- Link evidence to real file lines on the detected source host at the recorded
  commit. For local or uncommitted source, provide a clearly identified source
  excerpt or local reference; do not link it to a commit that lacks that code.
- Add a rollout view: rollouts as rows, clients as columns, and cells showing
  lag, missing implementation, unknown support, unknown date or n/a. Sort by
  clients still missing the change; distinguish uncertain cases.
- Use light and dark themes via CSS tokens, keyboard-operable controls and
  visible focus. At phone width, keep horizontal matrix scrolling inside its
  own container. Check text contrast and color-vision-deficiency simulations
  in both themes, then correct ambiguous states.

INTERACTION
- Filters: gaps, audit disagreements, origin, area and client. A gap is an
  applicable cell not established as full; distinguish confirmed gaps from
  unknowns. Update the visible counts. Search feature names and fingerprints.
- Hovering or focusing a row or column highlights it. Clicking a cell selects
  it; keyboard users must be able to select cells too.
- What-if: select a feature and core, and model an implementation change that
  preserves the existing public interface. Using the traced dependencies,
  distinguish clients expected to inherit the change after an update, rebuild
  or deployment; clients needing their own changes; clients with no existing
  integration; and unknown cases. Show the dependency path and assumptions,
  including versions and flags. Existing API use does not imply that a new
  capability or changed API will propagate automatically.

VERIFY, THEN PUBLISH
- Spot-check at least twelve cells, or all cells if fewer exist. Cover every
  observed status and, where present, an independent client, a feature reached
  through a core API, and a version or flag condition. Check the requirements
  and call paths, not just the fingerprint matches.
- Verify that excluded bindings, vendored dependencies and build output
  contributed zero independent support hits. Check any generated-behavior
  mappings against their source definitions and actual integration.
- Recompute scores, spread, filter counts and rollout lags from the captured
  data. Ensure unknown and n/a cases follow the stated rules. Check evidence
  links against the referenced revisions and line numbers.
- Render the page in a headless browser. Check sticky headers while scrolling
  in both directions, filter counts, cell selection, side-panel links,
  keyboard operation, both themes, narrow-screen layout and console errors.
  Exercise the what-if against at least one traced dependency, if present.
- Fix issues found, then deliver the artifact using the environment's supported
  mechanism. If unavailable, provide a self-contained HTML file. Include the
  evidence data and analysis metadata needed to inspect the conclusions.
  Identify any validation that could not run; do not claim it passed.
- Tell me:
  - clients and cores found, their roots, roles and dependencies
  - feature counts by origin and area, noting overlapping origins
  - parity scores, unknown counts and features verified in only one client;
    distinguish these from features confirmed absent in every other client
  - every comparable audit disagreement and any unresolved audit claims
  - rollouts with missing clients or uncertain histories
  - what the extraction cannot establish: dynamic dispatch, constructed
    identifiers, runtime configuration, reachability, flags, unreadable code,
    missing repositories or history, and source presence versus shipped behavior
