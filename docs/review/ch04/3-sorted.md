# Chapter 4 — Sorted review

## Summary
- 19 items (C1–C18, D-new1). Sorted into **16 errata**, **2 viva prep** and **0 holds**. E1 merges C3 and D-new1. E2 and E6 each take one part of C18. E5 merges C12 and the zoom half of C16. The double-tap half of C16 and a sub-point of C1 are viva prep. Five rebutted sub-points are listed under Holds. The errata, by impact: E1 (C3+D-new1), E2 (C2), E3 (C1), E4 (C4), E5 (C12+C16), E6 (C9), E7 (C5), E8 (C6), E9 (C7), E10 (C8), E11 (C10), E12 (C11), E13 (C17), E14 (C15), E15 (C13), E16 (C14). Viva prep: C16 (double-tap) and C1 (the evaluation's cap).
- **Results.** No reported number changes, and no RQ conclusion changes. Ch4 reports no evaluation results. Its numbers are configuration values, and one of them is wrong (E3: MAX_SBOMS 10 → 50). The fixes change what Ch4 says the tool does:
  - E1: Fig. 4.3a and Fig. 4.2 are described as the layouts actually run: rooted at the highest-degree component (`express`), not at the application.
  - E2: the root is no longer said to be drawn green.
  - E4, E5: the browser-side parser and the "Depth: 1" status bar are disclosed.
  - Abstract EN/PT: unchanged by every fix. Other chapters touched: Ch5:4 (E12) only.
- **Settled disputes.**
  - C3: the critic is right, and I re-ran the check. `layoutOpt` (`utils/cytoscapeConfig.js:18-25 @28113e7`, same at HEAD) has no `directed` and no `roots`. Only `DeltaGraphVisualizer.js:61` passes `directed: true`. Cytoscape 3.33.1 (`breadthfirst.mjs:72-85`) then roots each connected component at its maximum-degree node. Rebuilding `code-server` with `_build_graph @28113e7` gives 181 nodes, 263 edges and one connected component. The node of highest degree is `pkg:npm/express@5.0.0-alpha.8` (32); `pkg:npm/code-server@0.0.0` has 22 (out 22, in 0). The undirected BFS levels from `express` are 1, 32, 39, 39, 23, 23, 17, 7, which is what `layout-tree-canvas.png` shows (one top node, then a row of about 32). The directed levels from the root would be 1, 22, 57, 40, 25, 23, 11, 2. I simulated `concentric` (`node.degree()`, level width `maxDegree/4` = 8): the rings are 1, 2, 42 and 136 nodes. `express` is alone at the centre, and `code-server` shares the second ring with `send`. That matches Fig. 4.2: a small group at the centre, then two rings.
  - C2: the critic is right. `convertCycloneDXToGraph` (`parseInputSBOM.js:37-93`) never sets `isPrimary`. The only writer is `swapPrimaryComponent` (`GraphVisualizer.js:234-252`), from the "Set Primary" context menu. In Figs. 4.2 and 4.4, every node except the severity colours is the default blue. HEAD is the same.
  - C4: both are partly right. The defender is right that the parser converts formats rather than reimplementing the Ch5/Ch6 algorithms, and that every Ch7 number comes from REST endpoints. The critic is right that it is undisclosed, that it keys by `bom-ref` alone with no self-loop filter, and that :16 ("vulnerability aggregation") contradicts the Fig. 4.1 caption. I confirmed one more browser-side computation: the Graph Analysis Overview tab counts its degree histograms from the raw `dependencies` list (`graph-analysis/index.js:41-58,168-185`).
  - C5: the defender is right that no serialisation happens (`get_graph` returns the live object). The critic is right that every listed algorithm except HITS first calls `G.to_undirected()` (`impact.py:96`; `graph_analysis.py:26,51,151,279,351,394`), which builds a copy.
  - C6: the defender is right that every cache behind an in-scope endpoint is invalidated on PUT and DELETE (`sboms.py:109-110,119-120`; `graph_analysis.py:458-467`; `audit.py:22-24`). Only the delta and comparison caches (`delta.py:11,84`) are not, and they belong to the out-of-scope router. The critic is right that communities and k-core are cheap. I recomputed from `per_sbom.csv`: communities median 0.018 s, max 0.057 s; k-core median 0.005 s, max 0.030 s; audit median 4.72 s, max 16.39 s; impact median 0.151 s, max 0.99 s (uncached).
  - C7: the defender is right that the overlay queries are sub-second for every SBOM (impact max 0.99 s, global-analysis 0.042 s, k-core 0.030 s, communities 0.057 s). The critic is right that "sub-millisecond" has no source, and that "necessary for a usable interface" is a usability claim that Ch1:67 excludes. Ch7:151 already makes the measured claim ("interactive").
  - C8: the critic is right. `git ls-tree -r 28113e7` has no test file. The `backend/tests/` and `*.test.js` files exist only at HEAD, and they cover post-thesis features.
  - C10: the defender is right that the scope is consistent. Ch7 never uses bridges, and Ch6:64 says the optimiser's outputs are not evaluated. The critic is right that the Ch4 endpoint list is incomplete, and that `main.py:18-21` registers `analysis` before `sboms` too.
  - C16: the defender is right that double-tap is HEAD behaviour, which the screenshot decision allows. The zoom readout is in the status bar (`sbom-graph/index.js:1309-1311`), and Figs. 4.2 and 4.4 show it there.
  - D-new1: confirmed. On load, the layout runs with `padding: 60` (`GraphVisualizer.js:785-795`). Toolbar layouts use `layoutOpt.padding` = 150.
  - Everything else both parties report, I recomputed and it agrees:
    - MAX_SBOMS default 50 (`sbom_store.py:42`); the evaluation set 1000 (`lens.py:44`);
    - blast radius `ancestors + [node_id]` (`impact.py:50`); no topological sort anywhere;
    - `/health` returns `{status, version, maxSboms}`, and only `useSBOMLibrary.js:34-43` reads it;
    - `code-server` `maxDepth` = 7; status-bar depth from `node.ancestors()` (compound parents), always 1;
    - persisted keys for `code-server`: bom-ref, description, group, name, purl, scope, type, version; types library 179, framework 1, application 1.
- **Four decisions are needed** (list at the end). Three of them ask whether to fix a frontend defect at HEAD or disclose it: layout rooting (D1), the green root (D2) and the status-bar depth (D3). The fourth is the wording for testability (D4). D1–D3 share one cost: under any B option `app-overview.png` must be retaken. Once one screenshot is retaken, a second B adds little cost.

## Erratum

### E1 (from C3, D-new1) The tree and radial layouts are described, and Fig. 4.3a captioned, as rooted at the application; they are rooted at the highest-degree component — impact: high (a figure caption that misreads its own figure)
- Settled facts:
  - On the SBOM Graph page, `breadthfirst` runs undirected with no `roots`. Each connected part is rooted at its node of highest degree, and the levels are undirected hop counts.
  - In `code-server` that root is `express` (degree 32), not the application (22). Fig. 4.3a's top node is `express`.
  - `concentric` orders the rings by degree. `express` is at the centre, and the application is in the second ring.
  - `directed: true` (line 174) is used only by the Delta/Comparison visualiser.
  - The load-time radial layout uses a padding of 60 px, not 150.
  - No measure is read from a layout. Depth in Ch7 comes from `graph-analysis`.
- Fix:
  - Ch4:174 — "The \textbf{Tree Layout} (Cytoscape name: \texttt{breadthfirst}, \texttt{directed: true}) organises nodes in horizontal levels according to their distance from the root in a breadth-first traversal. This layout makes dependency depth immediately visible: direct dependencies appear on level one, transitive dependencies on level two and beyond. It is the default layout for the Delta and Comparison views," → "The \textbf{Tree Layout} (Cytoscape name: \texttt{breadthfirst}) organises nodes in horizontal levels by their breadth-first distance from a root. No root is given and edge direction is ignored, so Cytoscape takes the component of highest degree as the root: in \autoref{fig:layout-tree} this is \texttt{express}, not the application, and the levels are therefore not dependency depth, which the backend computes from the root (\autoref{sec:eval-rq1}). Run along the edge direction, it is the default layout for the Delta and Comparison views,"
  - Ch4:176 — "places nodes in concentric rings around a central hub, with the root component at the centre. This is the default layout applied when a new SBOM is first loaded, giving the user an immediate overview of the dependency neighbourhood around the root." → "places nodes in concentric rings by degree, with the components of highest degree at the centre; in \texttt{code-server} the centre is \texttt{express}, and the application is in the next ring. This is the default layout applied when a new SBOM is first loaded, giving the user an immediate view of the most connected components."
  - Ch4:180 — "a padding of 150 pixels" → "a padding of 150 pixels (60 when the radial layout is applied on load)"
  - Ch4:197 — "The tree layout shows dependency depth as levels below the root;" → "The tree layout arranges the components in levels by their distance, ignoring edge direction, from the component of highest degree, here \texttt{express};"
  - **DECISION NEEDED** (D1): disclose as above, or fix the layouts at HEAD.
- Also touches: none. Ch5–Ch8 never describe the layouts. Ch7:73 reports depth from the backend.
- Viva answer: "The layouts are only visual aids. No measure is read from them. Cytoscape roots the tree at the best-connected node unless it is given a root, and the evaluated version gives none. Depth is computed by the backend, from the components that have no consumers."

### E2 (from C2, C18 part) The root is said three times to be drawn green; no code path does so on load, and no figure shows it — impact: high (contradicted by Fig. 4.2)
- Settled facts:
  - `isPrimary` is set only by the "Set Primary" graph-editing action (`GraphVisualizer.js:234-252`, menu item `:612`). The style rule `node[?isPrimary]` never fires on load.
  - All four Ch4 PNGs show the root in default blue.
  - The analyses find roots by in-degree zero (`impact.py:53`), so nothing depends on the flag.
  - The same holds at HEAD.
- Fix:
  - Ch4:88 — "This placement gives the root a consistent position in the graph and allows the frontend to distinguish it visually as the primary node." → "The root thus becomes a node of the graph like any other component."
  - Ch4:128 — "The frontend distinguishes it visually by setting an \texttt{isPrimary} flag in the Cytoscape node data object produced by \texttt{convertCycloneDXToGraph()}, which triggers a distinct green fill on the canvas." → "The canvas does not distinguish it: the green fill for a primary node is applied only when the user marks a node as primary with the graph-editing context menu, which is outside the scope of this thesis."
  - Ch4:164 — delete "The root component --- the project whose dependencies are being analysed --- is distinguished by a green fill and a 2-pixel border, making it immediately identifiable as the anchor of the dependency tree." This also removes the glossary slip "dependency tree" (C18).
  - **DECISION NEEDED** (D2): disclose as above, or fix at HEAD.
- Also touches: none. No other chapter mentions the primary node.
- Viva answer: "The stylesheet has a style for a primary node, but it applies only when the user marks one. On load, the root looks like any other component. The analyses identify roots by in-degree zero."

### E3 (from C1) MAX_SBOMS default given as 10; the code and Ch4:20 say 50 — impact: high (self-contradiction three paragraphs apart), trivial fix
- Settled facts: `sbom_store.py:42` defaults to `"50"` (same at HEAD). Ch4:20 says "fifty", and `useSBOMLibrary.js:26` and the tool glossary say 50.
- Fix: Ch4:82 — "(default: 10)" → "(default: 50)"
- Also touches: none.
- Viva answer: see V2.

### E4 (from C4) The canvas graph comes from a second, browser-side parser that the chapter never mentions; "all graph computation" and "vulnerability aggregation" overstate the backend — impact: medium (undisclosed design fact; internal contradiction with the Fig. 4.1 caption)
- Settled facts:
  - `convertCycloneDXToGraph` (`parseInputSBOM.js:37-93`) keys nodes by `bom-ref` alone. It has no self-loop filter and does not merge shared identifiers.
  - The status-bar depth (E5), the version-conflict predecessor highlight (`GraphVisualizer.js:712-716`) and the Graph Analysis degree histograms (`graph-analysis/index.js:41-58,168-185`) are computed in the browser.
  - OSV lookup and CVSS scoring are in the browser too, as the Fig. 4.1 caption says.
  - In the dataset the two graphs coincide: all 33,309 components have a `bom-ref`, and there are no self-loops, duplicates or dangling references.
  - Every Ch7 number comes from the backend.
- Fix:
  - Ch4:10 — "the backend holds all graph computation" → "the backend holds the graph analyses"
  - Ch4:16 — "graph construction, metric calculation, risk assessment, and vulnerability aggregation." → "graph construction, metric calculation, and risk assessment."
  - Ch4:220, after "and loads them into the Cytoscape instance." add: "This conversion is done in the browser by a second parser, \texttt{convertCycloneDXToGraph()}, which keys nodes by \texttt{bom-ref} alone and applies neither the identifier fallback nor the self-loop filter of \autoref{sec:parsing-pipeline}; on an SBOM whose components lack a \texttt{bom-ref}, the results of the backend would not map onto the canvas. Every component in the evaluation dataset has a \texttt{bom-ref}, and there the two graphs coincide. The status-bar depth and the degree histograms of the Graph Analysis page are also counted in the browser; every number reported in \autoref{cp:evaluation} comes from the backend."
- Also touches: none. Ch5:32 already places the OSV lookup in the browser.
- Viva answer: "The analyses the thesis reports run once, in Python, on the backend graph. The browser converts the same file to draw it and does a few display counts. The two agree on every SBOM in the dataset."

### E5 (from C12, C16 part) The status bar in Figs. 4.2 and 4.4 shows "Depth: 1" for code-server (depth 7), and the zoom readout is placed in the toolbar — impact: medium (visible defect in two figures; text contradicts figures)
- Settled facts:
  - `getStats` (`GraphVisualizer.js:281-293`, HEAD :288) counts `node.ancestors()`. In Cytoscape that is compound-parent lineage, so the depth is always 1.
  - `per_sbom.csv` gives `maxDepth` = 7 for code-server.
  - The zoom percentage is in the status bar (`sbom-graph/index.js:1309-1311`). The toolbar has no readout.
- Fix: Ch4:218 — "The current zoom level is tracked in React component state and displayed in the toolbar." → "The current zoom level is tracked in React component state and displayed in the status bar below the canvas, which also gives the name of the SBOM and its numbers of nodes and edges. The status bar also shows a depth, which in the evaluated version is always 1 because of a defect: it counts compound-node parents, which a dependency graph does not have, instead of dependency levels. The depths reported in \autoref{cp:evaluation} come from the backend."
  - **DECISION NEEDED** (D3): disclose as above, or fix at HEAD.
- Also touches: none.
- Viva answer: "It is a display defect in the status bar. The depth the thesis reports comes from the backend's `graph-analysis` endpoint."

### E6 (from C9, C18 part) Ch4 defines the blast radius as the ancestors only, and adds a "topological ordering" that does not exist — impact: medium (first definition of a central measure disagrees with Ch5, Ch2, the glossary and the code)
- Settled facts:
  - `impact.py:50` is `list(nx.ancestors(G, node_id)) + [node_id]`. Ch2:126, Ch5:219 and the glossary are inclusive, as already decided.
  - `git grep -i topolog 28113e7 -- backend src` finds only "Topology Optimizer". Dominators and the depth histogram exist.
- Fix:
  - Ch4:100 — "to find all components that transitively depend on a given node --- ancestors in the directed graph are precisely those dependants," → "to find all components that transitively depend on a given node, and adds the node itself --- ancestors in the directed graph are precisely those dependents,"
  - Ch4:136 — "these are its \textit{ancestors} in the directed graph" → "these are the component itself and its \textit{ancestors} in the directed graph"; "Dominator-tree construction, depth histograms, and topological ordering all depend on the same directed convention," → "Dominator-tree construction and the depth histogram both depend on the same directed convention,"
- Also touches: none. Ch1:15 speaks of the components "affected by" a vulnerability, which are the ancestors. Ch2:126 is already inclusive.

### E7 (from C5) "All structural algorithms operate directly on the stored DiGraph … without intermediate conversion" — all but HITS run on an undirected copy — impact: medium (contradicts Ch6's G/G_u distinction)
- Settled facts:
  - Articulation points, Louvain, k-core, spectral, small-world, rich-club and optimiser all call `G.to_undirected()` first. HITS runs on `G`.
  - No serialisation step exists: `get_graph` returns the live object.
  - The routers only read the graph.
- Fix:
  - Ch4:54 — "All structural algorithms operate directly on the stored \texttt{DiGraph} object:" → "All structural algorithms start from the stored \texttt{DiGraph} object, most of them through its undirected projection (\autoref{cp:network-sci}):"
  - Ch4:54 — "and returns results without intermediate conversion." → "and returns results without any serialisation."
  - Ch4:146 — "There is no serialisation step between storage and computation: the algorithm receives the graph object and operates on it in place. This design minimises latency for interactive analysis queries, where sub-second response times are necessary for a usable graph exploration interface." → "There is no serialisation step between storage and computation: the algorithm receives the graph object and reads it directly." This also removes the latency claim of E9.
- Also touches: none. It matches Ch6:8 and Ch8:47.

### E8 (from C6) Caching names cheap analyses as expensive, says "constant time", and does not say which endpoints are cached — impact: medium (contradicted by Ch7's timings)
- Settled facts:
  - Communities median 0.018 s and k-core 0.005 s. The audit takes a median of 4.72 s, max 16.39 s, and is cached.
  - Impact, global-analysis and metrics are not cached.
  - Every in-scope cache is invalidated on PUT and DELETE. The delta and comparison caches are not, and they are out of scope.
  - Ch6:8 already states the policy correctly.
- Fix:
  - Ch4:56 — "Community detection and k-core results are expensive to recompute for large graphs; caching ensures that repeated requests for the same SBOM complete in constant time. All caches are explicitly invalidated" → "The audit, whose registry queries take seconds, and the whole-graph analyses of \autoref{cp:network-sci} are cached per SBOM, so a repeated request returns the stored result; the impact, global-analysis and metrics endpoints recompute on every call. The caches of the routers in scope are explicitly invalidated"
  - Ch4:148 — "Several routers maintain module-level caches for computationally expensive results --- community detection and k-core decomposition are cached after the first request for a given SBOM." → "The audit and the whole-graph analyses are cached after the first request for a given SBOM (\autoref{sec:backend})."
- Also touches: none. It matches Ch6:8.
- Viva answer: "The cache matters for the audit, because of the registry calls. For the graph measures it is a convenience. Every cache behind an evaluated endpoint is cleared on edit and delete."

### E9 (from C7) "Sub-millisecond latency" is unsourced; "sub-second response times are necessary" is an untested usability claim — impact: medium (unsupported claims beside Ch7's 17 s maximum)
- Settled facts:
  - Nothing measures or cites the frontend latency.
  - Overlay queries are under 1 s for all 90 SBOMs, while the first audit takes up to 16 s.
  - Ch1:67 excludes usability. Ch7:151 claims only "interactive".
- Fix:
  - Ch4:44 — "Cytoscape.js manages the HTML5 canvas directly and responds to user gestures with sub-millisecond latency." → "Cytoscape.js manages the HTML5 canvas directly."
  - Ch4:146 — last sentence deleted (applied with E7).
- Also touches: none.
- Viva answer: "Every overlay query returned in under a second for every SBOM in the dataset. Only the first audit is slow, and that is the registry lookup, not the graph computation."

### E10 (from C8) "Independently testable" and "can be verified against known graph fixtures" — the evaluated commit has no tests — impact: medium (the stated principal design motivation invites "was it verified?")
- Settled facts:
  - There is no test file at 28113e7. The tests at HEAD cover post-thesis features.
  - No chapter reports a fixture check.
  - Most algorithms are direct NetworkX calls.
- Fix: **DECISION NEEDED** (D4). Recommended (A):
  - Ch4:18 — "can be verified against known graph fixtures independently of the frontend." → "can be tested on known graphs independently of the frontend; most of them are direct calls to NetworkX, and the evaluated version has no test suite of its own."
  - Ch4:16 — "into distinct, independently testable concerns" → "into distinct concerns". This is optional; under (A) "testable" is then qualified at :18 and can stay.
- Also touches: none.
- Viva answer: "The separation makes the algorithms testable without a browser. The evaluated version relies on NetworkX's tested implementations rather than a suite of its own, and the thesis says so. Tests were added later for the post-thesis features."

### E11 (from C10) Scope and endpoint map: the optimiser supplies Ch6's bridges; Graph Analysis is undersold; the endpoint list is incomplete; router order omits `analysis` — impact: medium-low (the measure → endpoint → page map)
- Settled facts:
  - Bridges come only from `/optimizer` (Ch6:204), and Ch7 does not use them.
  - The Graph Analysis page has Overview, Graph Theory and Epidemic Risk tabs (`graph-analysis/index.js:267-269`).
  - `tab:ns-summary` maps every measure to its endpoint.
  - `main.py:18-21` registers `delta`, `graph_analysis` and `analysis` before `sboms`.
- Fix:
  - Ch4:22 — "a Topology Optimiser and a Comparison/Delta analysis view ---" → "a Topology Optimiser (whose endpoint also returns the bridges of \autoref{sec:ns-articulation}, which are not evaluated) and a Comparison/Delta analysis view ---"
  - Ch4:35 — "and Graph Analysis (degree and depth distributions)" → "and Graph Analysis (the degree and depth distributions and the whole-graph measures of \autoref{cp:network-sci})"
  - Ch4:52 — "the \texttt{delta} and \texttt{graph\_analysis} routers are registered before" → "the \texttt{delta}, \texttt{graph\_analysis} and \texttt{analysis} routers are registered before"
  - Ch4:64 — "The remaining endpoints --- \texttt{/vulnerabilities}, \texttt{/\{id\}/impact}, \texttt{/\{id\}/audit}, \texttt{/\{id\}/graph-analysis}, \texttt{/\{id\}/kcore}, \texttt{/\{id\}/communities}, and \texttt{/\{id\}/epidemic} --- are described in \autoref{cp:vuln-analysis} and \autoref{cp:network-sci} respectively." → "The vulnerability and impact endpoints are described in \autoref{cp:vuln-analysis}; the structural endpoints are listed with their measures in \autoref{tab:ns-summary}."
- Also touches: none. Ch1:87, Ch6:22 and Ch8:30/:47 describe bridges as computed, which stays true.

### E12 (from C11) "Ensures stable, unique node identifiers across SBOMs produced by different tools" is a guarantee the code does not give and the data never tests — impact: low-medium (also Ch5:4)
- Settled facts:
  - Components that resolve to the same key merge, and the later attributes overwrite the earlier (`sbom_store.py:23-28`).
  - Keys are per graph.
  - All 90 SBOMs are cdxgen 12.8.5, spec 1.5. Every component has a `bom-ref` and a `purl`, and there are no dangling references, so the fallbacks are never used.
  - Ch5:4 says every node carries a `purl`. That holds in the dataset but not by construction.
- Fix:
  - Ch4:90 — "and ensures stable, unique node identifiers across SBOMs produced by different tools with varying levels of metadata completeness." → "and gives every component an identifier when some fields are missing; components that resolve to the same identifier are merged into one node. In the evaluation dataset every component has a \texttt{bom-ref}, so the fallbacks are never used."
  - Ch4:102 — "This tolerance is necessary because real-world SBOMs do not always maintain" → "This tolerance guards against SBOMs that do not maintain"
  - Ch5:4 — "Every node in that graph carries a \texttt{purl} attribute" → "Every node in the evaluated graphs carries a \texttt{purl} attribute"
- Also touches: Ch5:4.

### E13 (from C17) `/health` is said to gate API-mode features; it returns the library limit, and API mode is a compile-time constant (Ch4:46) — impact: low (two paragraphs disagree)
- Settled facts: `main.py:28-30` returns `{status, version, maxSboms}`. `useSBOMLibrary.js:34-43` reads only `maxSboms` and ignores failures. `DATA_SOURCE = "api"` is a constant.
- Fix: Ch4:62 — "returns a JSON object confirming the service status and version number; the frontend uses this to detect backend availability before enabling API-mode features." → "returns the service status, the version number and the \texttt{MAX\_SBOMS} limit; the frontend reads the limit from it."
- Also touches: none.

### E14 (from C15) Persisted attributes and component types are stated as exhaustive — impact: low
- Settled facts:
  - The scalar filter also keeps `group`, `scope`, `description` and `author`. Its real effect is to drop nested fields.
  - CycloneDX 1.5 defines more types. Only library, framework and application occur in the dataset.
  - The Component section at :209 shows exactly the five listed attributes, matching the screenshot, so it stays.
- Fix:
  - Ch4:92 — "To avoid compatibility issues with NetworkX's attribute handling, only scalar values" → "To keep node data flat, only scalar values"
  - Ch4:92 — "In practice, the persisted attributes are \texttt{name}, \texttt{version}, \texttt{purl}, \texttt{type}, and \texttt{bom-ref}." → "In practice, the persisted attributes include \texttt{name}, \texttt{version}, \texttt{purl}, \texttt{type}, and \texttt{bom-ref}, the ones used by the analyses and the interface."
  - Ch4:124 — "one of \texttt{library}, \texttt{framework}, or \texttt{application}." → "in the evaluation dataset, one of \texttt{library}, \texttt{framework}, or \texttt{application}."
- Also touches: none.

### E15 (from C13) "JSON validation" checks syntax, not CycloneDX conformance — impact: low
- Settled facts:
  - `sboms.py:26-29` checks only `json.loads`.
  - JSON without `components` is stored as an empty graph.
  - A top-level array fails at `cyclonedx.get` (`:50`) with HTTP 500.
- Fix: Ch4:78, after "an HTTP 400 response is returned immediately." add: "The check is for well-formed JSON, not for conformance to the CycloneDX schema."
- Also touches: none. Ch1:65 states the format boundary, which stays true as a scope statement.

### E16 (from C14) Duplicate detection is byte-exact — impact: low
- Settled facts: the hash is over the raw bytes (`sboms.py:32`). After a PUT it is recomputed over normalised JSON (`:95-98`).
- Fix: Ch4:80 — "when the same SBOM is submitted more than once." → "when the same file is submitted more than once."
- Also touches: none.

## Viva prep

### V1 (from C16, double-tap) Do the gestures described match the evaluated version?
- Likely question: "In the version you evaluated, does double-tapping really open the Vulnerabilities page?"
- Answer: The interaction model is described as shown in the figures, which come from the current version. In the evaluated commit a single tap on a coloured node navigated (`GraphVisualizer.js:719-725 @28113e7`). Double-tap was introduced later so that a single tap could select the node for the Component section. No measure depends on the gesture. This follows the screenshot decision. No text change.

### V2 (from C1, sub-point) "Appropriate for a research prototype evaluated on a fixed dataset" — but the evaluation raised the cap to 1000
- Likely question: "Did the 50-SBOM limit constrain the evaluation of 90 SBOMs?"
- Answer: No. The collection pipeline started the backend with `MAX_SBOMS=1000` (`sbom-lens-eval/pipeline/lens.py:44`), and it uploaded, analysed and deleted one SBOM at a time (`lens.py:90-104`), so at most one was stored. The limit only protects an interactive session from unbounded memory growth. No text change.

## Holds
- No whole item. Five sub-points are rebutted and need no action:
  - C4, ":18 is contradicted by a JavaScript reimplementation": the browser parser converts formats and counts for display. It does not reimplement the Ch5/Ch6 algorithms that :18 is about. E4 discloses it.
  - C6, delta and comparison caches never cleared: they belong to the out-of-scope router, and their keys are UUIDs that are never reused. E8 scopes the claim to "the routers in scope".
  - C6, registry results kept for the life of the process: they are part of the audit result and are cleared with it (`audit.py:22-24`).
  - C18, ":35 Dashboard vs the Fig. 4.1 label 'SBOM Library'": the figure names the view after what the Dashboard shows. It is a label, not a contradiction.
  - C10(d), "eight routers": correct at 28113e7. The ninth (`priorities`) is post-thesis.

## Decisions needed
1. **Layout rooting (E1/C3).**
   - (A) Disclose: describe the layouts as they run (hub-rooted tree, degree-ordered rings), using the E1 text. No screenshot changes.
   - (B) Fix at HEAD: pass `roots` = the in-degree-zero nodes and `directed: true` to `breadthfirst` in `applyLayout`. Give `concentric` a value that decreases with distance from the root. Retake `app-overview.png` and `layout-tree-canvas.png`. The text then keeps "rooted at the application", and the caption reads as dependency depth, matching Ch7. This changes HEAD UI, which the screenshot decision permits.
   - (C) Keep the default layouts but drop the claims: say only "levels" and "rings", with no reading of depth. This is shorter than (A) but leaves Fig. 4.3a unexplained.
   - Recommendation: **A**. It is minimal, needs no retake, and no result depends on the layouts. Choose B if the tree figure is meant to show depth, for example as an aid in the viva.
2. **Green root (E2/C2).**
   - (A) Disclose: the root is not distinguished on load, using the E2 text. No retake.
   - (B) Fix at HEAD: set `isPrimary: true` on the `metadata.component` node in `convertCycloneDXToGraph`. Retake all four Ch4 PNGs, and keep the text with "in the current version" wording. Side effect: Export would then write the root back to `metadata.component` (`parseInputSBOM.js:9`), which it does not do today.
   - Recommendation: **A**, unless B is chosen for D1 or D3, since the retakes are then largely already paid for.
3. **Status-bar depth (E5/C12).**
   - (A) Disclose the defect in one sentence (E5 text). No retake.
   - (B) Fix at HEAD: take the depth from the backend's `graph-analysis` `maxDepth`, or do a directed BFS from the in-degree-zero nodes. Retake `app-overview.png` and `node-details.png`. The text then says the status bar gives the depth, with no defect sentence.
   - Recommendation: **B** if any screenshot is retaken for D1 or D2. The fix is a few lines, and "Depth: 1" next to a depth-7 graph is the most visible defect in the figures. Otherwise **A**.
4. **Testability wording (E10/C8).**
   - (A) Keep "can be tested" and state that the evaluated version has no test suite of its own (E10 text).
   - (B) Soften without disclosure: "can be tested on known graphs independently of the frontend". This is still modal, but an examiner will ask whether it was done.
   - (C) Drop the testability argument and keep only auditability: "keeping computation server-side keeps the algorithms in one place, in direct NetworkX calls".
   - Recommendation: **A**. It is consistent with the decision to disclose known defects, and it is honest about 28113e7.
