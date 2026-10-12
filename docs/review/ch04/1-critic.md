# Chapter 4 — Critic

What I checked. I read Ch4 (224 lines) line by line against the evaluated code at `28113e7` (`git -C /home/inasc/projects/sbom-lens show 28113e7:<path>`, marked `@28113e7`), and checked at HEAD (`5273e78`) whether each code fact still holds. I checked it against Ch1 (objectives, scope), Ch3 (gap analysis), Ch5 and Ch6 (endpoints, overlays, measures), Ch7 (pipeline, timings) and Ch8, and against both glossaries (`sbom-lens-thesis/CONTEXT.md`, `sbom-lens/CONTEXT.md`). I recomputed the timing figures from `sbom-lens-eval/analysis/out/per_sbom.csv` with numpy/csv. I rebuilt the `code-server` graph with the `_build_graph` rules and surveyed all 90 evaluation SBOMs (`sbom-lens-eval/data/sboms/*.json`) for identifier, attribute, self-loop and dangling-reference properties. I looked at the four Ch4 PNGs (and counted root-green `#4CAF50` pixels in them). I read the Cytoscape 3.33.1 layout sources in `node_modules` to check how the layouts pick their roots. I did not reopen the decisions already taken: inclusive blast radius, the 28113e7 scope and post-thesis features, Fix Priorities named once, HEAD screenshots without disclosure, no overlay dimming or community collapse in the text, 8,868 findings, the narrow novelty framing, the HITS/powerlaw handling, reproducibility scope, OSV-only data and repository availability.

Overall, the backend pipeline description (§4.2) is mostly accurate: the fallback chain, scalar filter, self-loop and dangling-edge guards, SHA-256 duplicate check, 409 cap and PUT rebuild all match `sbom_store.py` and `sboms.py @28113e7`. The problems are elsewhere. One number contradicts the code and the chapter itself (MAX_SBOMS). Two visual claims are contradicted by the chapter's own figures (green root; tree/radial rooted at the application). The canvas graph comes from a second, separately written parser that the chapter never mentions. And several design-justification sentences (caching, latency, testability, "operates directly on the DiGraph") do not match the code or Ch7's data.

## C1. MAX_SBOMS default is given as 10; the code, Ch4:20 and the tool glossary say 50
- Type: wrong / inconsistency
- Origin: pre-existing
- Severity: high
- Thesis: Chapters/04-System-Design.tex:82 — "the \texttt{MAX\_SBOMS} environment variable (default: 10) caps the number of concurrently stored SBOMs"
- Evidence: `backend/store/sbom_store.py:42 @28113e7` (unchanged at HEAD): `int(os.getenv("MAX_SBOMS", "50"))`. Ch4:20 says "the default limit is fifty". `src/hooks/useSBOMLibrary.js:26 @28113e7` uses `DEFAULT_MAX_SBOMS = 50`. `sbom-lens/CONTEXT.md` (SBOM Library): "50 by default". Related, for the sentence "This limit is appropriate for a research prototype evaluated on a fixed dataset": the evaluation itself started the backend with `MAX_SBOMS=1000` (`sbom-lens-eval/pipeline/lens.py:44`). It uploaded, analysed and deleted one SBOM at a time (`lens.py:90-104`), so the default never constrained the 90-SBOM dataset.
- Why it matters: Two different defaults three paragraphs apart is an error any careful examiner will spot, and one of them is wrong.

## C2. The root component is not drawn green: no code path sets `isPrimary` on load, and none of the four figures shows a green node
- Type: wrong
- Origin: pre-existing
- Severity: high
- Thesis: Chapters/04-System-Design.tex:128 — "The frontend distinguishes it visually by setting an \texttt{isPrimary} flag in the Cytoscape node data object produced by \texttt{convertCycloneDXToGraph()}, which triggers a distinct green fill on the canvas." Also :164 — "The root component … is distinguished by a green fill and a 2-pixel border, making it immediately identifiable as the anchor of the dependency tree", and :88 — "allows the frontend to distinguish it visually as the primary node".
- Evidence: `src/utils/parseInputSBOM.js:37-93 @28113e7`: `convertCycloneDXToGraph` builds node data as `{...component, id: bom-ref}` and never sets `isPrimary`. The only writer of `isPrimary: true` is `swapPrimaryComponent` (`GraphVisualizer.js:234-252 @28113e7`). That function runs only from the "⭐ Set Primary" context-menu item (`GraphVisualizer.js:612`), a graph-editing action. `git grep isPrimary` finds no other writer, at 28113e7 or at HEAD. The style rule `node[?isPrimary]` (`GraphVisualizer.js:27-33`) exists but never fires on load. The figures agree: `app-overview.png`, `node-details.png`, `layout-tree-canvas.png` and `layout-force-canvas.png` contain zero `#4CAF50` pixels. In Fig. 4.2, every node, including the centre, is the default blue.
- Why it matters: The text says this three times, and Fig. 4.2, captioned as showing the default view, plainly contradicts it. The fix is easy: say the root is not visually distinguished by default. Or, if the code is to be changed, that is post-thesis behaviour.

## C3. Neither the tree nor the radial layout is rooted at the application; the Fig. 4.3 caption misreads the tree figure
- Type: wrong
- Origin: pre-existing
- Severity: high
- Thesis: Chapters/04-System-Design.tex:174 — "The \textbf{Tree Layout} (Cytoscape name: \texttt{breadthfirst}, \texttt{directed: true}) organises nodes in horizontal levels according to their distance from the root … direct dependencies appear on level one, transitive dependencies on level two and beyond." :176 — "The \textbf{Radial Layout} (… \texttt{concentric}) places nodes in concentric rings around a central hub, with the root component at the centre." :197 (caption) — "The tree layout shows dependency depth as levels below the root".
- Evidence: (a) `directed: true` is never passed. The toolbar calls `applyLayout(name)` (`sbom-graph/index.js:878 @28113e7`), which runs `{...layoutOpt, name, animate, animationDuration}` (`GraphVisualizer.js:254-259`). `layoutOpt` (`utils/cytoscapeConfig.js:18-25`) has no `directed` and no `roots`. Only `DeltaGraphVisualizer.js` sets `directed: true`, and it is the same at HEAD. In Cytoscape 3.33.1 (`node_modules/cytoscape/src/extensions/layout/breadthfirst.mjs:72-85`), when `directed` is false and `roots` is undefined, the root of each connected component is its *maximum-degree* node, and levels are undirected BFS distances. For `code-server` (rebuilt with the `_build_graph` rules: 181 nodes, 263 edges), the maximum-degree node is `pkg:npm/express@5.0.0-alpha.8` (degree 32), not the application `pkg:npm/code-server@0.0.0` (degree 22). The single node at the top of `layout-tree-canvas.png` is that hub, not `code-server`. Levels therefore count undirected hops from `express`, not dependency depth from the root. (b) `concentric`'s default (`concentric.mjs:18-19`) is `node.degree()`, so the highest-degree nodes go to the centre. The root sits there only when it happens to have the highest degree, and in `code-server` it does not. The crop of the centre of `app-overview.png` shows a small group of nodes, not a single root. (c) Line 174 quotes a configuration (`directed: true`) that the SBOM Graph page does not use.
- Why it matters: The caption tells the reader to read Fig. 4.3a as dependency depth below the application, and the figure shows something else. Ch7 reports depth (`maxDepth` = 7 for code-server) by a different method, so an examiner who compares the two will see that they do not match. Either describe the layouts as they behave (hub-rooted, undirected levels; degree-ordered rings), or treat passing `roots`/`directed` as a post-thesis change.

## C4. The canvas graph is built by a second, separately written frontend parser with different identity rules, which the chapter never mentions
- Type: omission / overstated
- Origin: pre-existing
- Severity: medium
- Thesis: Chapters/04-System-Design.tex:10 — "the backend holds all graph computation"; :18 — "Running graph algorithms in the browser would require reimplementing the backend graph logic in JavaScript, duplicating code that would be harder to test and audit"; :70 — "Every SBOM in SBOM Lens enters through a single ingest endpoint and exits as a cached \texttt{nx.DiGraph} that all downstream routers query directly"; :220 — "Selecting an SBOM converts the stored CycloneDX data to Cytoscape node and edge objects"; :16 — the backend owns "… risk assessment, and vulnerability aggregation".
- Evidence: `src/utils/parseInputSBOM.js:37-93 @28113e7` (`convertCycloneDXToGraph`) builds the Cytoscape graph independently of `_build_graph`, and it differs in three ways. (i) Node id is `bom-ref` only, with no `purl`/`name` fallback. A component without `bom-ref` gets `id: undefined` on the canvas, so the backend's purl- or name-keyed results (blast radius, articulation points, k-core, HITS) cannot be mapped onto it. (ii) There is no self-loop filter (`:72-83`). (iii) There is no merging of components that share an identifier. Other graph computations also run in the browser: the status-bar depth (`GraphVisualizer.js:281-293`, see C12), the version-conflict predecessor highlight (`:712-716`, `e.target.predecessors`), the per-SBOM finding match (`sbom-graph/index.js:103-107`) and, per the Fig. 4.1 caption and Ch5:32, the OSV lookup and CVSS scoring. In the evaluation dataset the two parsers happen to agree: all 33,309 components across the 90 SBOMs have a `bom-ref`, with no self-loops, no duplicate identifiers and no dangling references (my survey of `data/sboms/*.json`).
- Why it matters: The design rationale (:18) presents server-side computation as avoiding a JavaScript reimplementation, yet the graph that the analyst sees is a JavaScript reimplementation. Anyone loading an SBOM without `bom-ref`s (which CycloneDX allows, as Ch4:90 notes) would see overlays fail silently. Under the "disclose known defects" decision, this belongs in the text. "Vulnerability aggregation" as a backend responsibility (:16) also contradicts the chapter's own Fig. 4.1 caption.

## C5. "All structural algorithms operate directly on the stored DiGraph … without intermediate conversion" — most of them run on a fresh undirected copy
- Type: wrong / inconsistency
- Origin: pre-existing
- Severity: medium
- Thesis: Chapters/04-System-Design.tex:54 — "All structural algorithms operate directly on the stored \texttt{DiGraph} object: these include articulation-point detection, k-core decomposition, HITS centrality, Louvain community detection, and the spectral analysis … This avoids any serialisation step between storage and computation: the router receives a Python object reference, runs the algorithm, and returns results without intermediate conversion." Also :146 — "the algorithm receives the graph object and operates on it in place."
- Evidence: Of the five algorithms listed, only HITS runs on `G` (`impact.py:72,101 @28113e7`). Articulation points: `G.to_undirected()` (`impact.py:96`). Louvain: `louvain_communities(G.to_undirected(), …)` (`graph_analysis.py:26`). k-core: `nx.core_number(G.to_undirected())` (`:51`). Spectral: `G_u = G.to_undirected()` (`:351`), and the same for small-world (`:151`), rich-club (`:279`) and optimizer (`:394`). `to_undirected()` builds a full copy on every uncached call. The spectral code then also converts to a NumPy/SciPy matrix. Ch6:22,46,56,76 and Ch8:47 state correctly that these measures are computed on \(G_u\).
- Why it matters: The claim is false as stated, and it contradicts Ch6's careful directed/undirected distinction. The defensible version is narrower: no serialisation or database round trip, but an in-memory projection for most measures.

## C6. Caching: the chapter names the wrong analyses as expensive, and "all caches are invalidated" is not true
- Type: wrong / unsupported
- Origin: pre-existing
- Severity: medium
- Thesis: Chapters/04-System-Design.tex:56 — "Community detection and k-core results are expensive to recompute for large graphs; caching ensures that repeated requests for the same SBOM complete in constant time. All caches are explicitly invalidated when an SBOM is updated or deleted"; repeated at :148.
- Evidence: Ch7's own timings (per_sbom.csv, recomputed): `time_communities` median 0.018 s, max 0.057 s; `time_kcore` median 0.005 s, max 0.031 s. The expensive calls are `time_audit` (median 4.72 s, max 16.39 s), which is cached (`audit.py:16,233-254`), and `time_impact` (median 0.151 s summed per SBOM, max 0.99 s), which is *not* cached and recomputes full HITS on every call (`impact.py:71-72`). `/global-analysis` is also uncached. On invalidation: `_delta_cache` and `_comparison_cache` (`delta.py:11,84 @28113e7`), keyed by SBOM pairs, are never cleared. `sboms.py:109-110,119-120` invalidates only the audit and graph-analysis caches. The audit cache also keeps the registry "unmaintained" results for the lifetime of the process. The evaluation never exercises a warm cache, because each SBOM is uploaded, queried once per endpoint and deleted (`sbom-lens-eval/pipeline/lens.py:90-104`).
- Why it matters: The justification is contradicted by the thesis's own measurements, and the universal "all caches" claim is false (even though delta is out of scope). The accurate version is: whole-graph results, including the slow audit, are cached per SBOM; impact is not; the comparison caches are not invalidated.

## C7. Latency claims ("sub-millisecond", "sub-second response times are necessary") are unsupported or contradicted by Ch7
- Type: unsupported / inconsistency
- Origin: pre-existing
- Severity: medium
- Thesis: Chapters/04-System-Design.tex:44 — "Cytoscape.js manages the HTML5 canvas directly and responds to user gestures with sub-millisecond latency"; :146 — "This design minimises latency for interactive analysis queries, where sub-second response times are necessary for a usable graph exploration interface."
- Evidence: No frontend latency was measured anywhere in the thesis or in `sbom-lens-eval`, and no citation is given. Ch1:67 excludes any usability study. Ch7:151 reports a median of 5.3 s and a maximum of 17.2 s per SBOM, "nearly all of it" the audit, which backs the Risk Assessment page. `time_epidemic` reaches 2.91 s and `time_optimizer` 2.06 s (per_sbom.csv). By Ch4:146's own standard, several pages fall short.
- Why it matters: An examiner will set "sub-second … necessary" beside Ch7's 17 s. Drop the sub-millisecond figure, or cite it, and restate :146 as "no serialisation overhead", without the usability claim.

## C8. "Independently testable" and "can be verified against known graph fixtures": the evaluated commit has no tests
- Type: unsupported
- Origin: pre-existing
- Severity: medium
- Thesis: Chapters/04-System-Design.tex:16 — "distinct, independently testable concerns"; :18 — "The principal motivation for this separation is analytical correctness … the algorithms … can be verified against known graph fixtures independently of the frontend."
- Evidence: `git ls-tree -r 28113e7` contains no test files. `.planning/codebase/TESTING.md @28113e7` says "No test files exist in this codebase." The first tests (`backend/tests/test_reach.py`, `test_priorities.py`, `test_audit_namespace.py`, `src/utils/*.test.js`) appear only after the evaluated commit, at HEAD, and they cover post-thesis features. No chapter reports verifying the algorithms against fixtures (grep of `Chapters/` for fixture/pytest/test suite finds only Ch4:18).
- Why it matters: "Analytical correctness" is offered as the principal design motivation, which invites the question "was it verified?". The answer at 28113e7 is no. Either say "could be" plainly as a property of the design, with no implication that it was done, or point to whatever correctness checks were actually made (e.g. Ch7's cross-checks).

## C9. The edge-semantics paragraph defines the blast radius as the ancestors only, and adds a "topological ordering" that does not exist
- Type: inconsistency / wrong
- Origin: pre-existing
- Severity: medium
- Thesis: Chapters/04-System-Design.tex:136 — "Blast-radius analysis identifies all components that would be affected if a given component were compromised; these are its \textit{ancestors} in the directed graph … Dominator-tree construction, depth histograms, and topological ordering all depend on the same directed convention"; :100 — "Blast-radius computation, for example, uses \texttt{nx.ancestors(G, node\_id)} to find all components that transitively depend on a given node".
- Evidence: `impact.py:50 @28113e7`: `list(nx.ancestors(G, node_id)) + [node_id]`. Ch5:219 and the thesis glossary ("A given component together with every component that depends on it") include the node itself, as does the agreed decision. `git grep topolog 28113e7 -- backend src` finds no topological sort, and no chapter describes one.
- Why it matters: This does not reopen the inclusive definition; it asks Ch4 to state it. Ch4 is the chapter where the reader first meets the blast radius, and its definition disagrees with Ch5, the glossary and the code. The "topological ordering" is an invented feature.

## C10. Scope and endpoint map: the "out-of-scope" optimiser supplies an in-scope measure, and the endpoint list omits the endpoints Ch5/Ch6 use most
- Type: inconsistency / omission
- Origin: pre-existing
- Severity: medium
- Thesis: Chapters/04-System-Design.tex:22 — "modules not evaluated in this thesis --- a Topology Optimiser and a Comparison/Delta analysis view --- which are shown in \autoref{fig:architecture} with a visual distinction"; :35 — "Graph Analysis (degree and depth distributions)"; :64 — "The remaining endpoints --- \texttt{/vulnerabilities}, \texttt{/\{id\}/impact}, \texttt{/\{id\}/audit}, \texttt{/\{id\}/graph-analysis}, \texttt{/\{id\}/kcore}, \texttt{/\{id\}/communities}, and \texttt{/\{id\}/epidemic} --- are described in …".
- Evidence: (a) Bridges are listed as a component-scale RO3 measure in Ch1:87, Ch6:22 and Ch8:30, and they come only from `GET /sboms/{id}/optimizer` (Ch6:22 and the summary-table row at Ch6:204, "Bridges & Component & \(G_u\) & \texttt{optimizer}"; `graph_analysis.py:386-454 @28113e7`). Ch6:64 also describes the optimizer's edge rankings. Fig. 4.1 draws "Optimizer" dashed (`architecture.mmd:10`), and Ch4:22 calls the Topology Optimiser out of scope. So the reader cannot tell whether the optimizer endpoint is in scope or not. (b) The Graph Analysis page at 28113e7 has three tabs, Overview, Graph Theory and Epidemic Risk (`src/layouts/graph-analysis/index.js:267-269`). Those tabs present Ch6's degree-distribution fit, small-world, bow-tie, rich-club and spectral-radius results, not just "degree and depth distributions". (c) The list at :64 omits `/global-analysis` (articulation points and all HITS scores; Ch5:232, Ch6:22,32), `/graph-theory` (Ch6:91,103,170,182), `/optimizer` (Ch6:22,64) and `POST /sboms/analysis` (named at Ch4:52). It also writes the paths without the `/sboms` prefix that Ch6 uses. (d) Ch4:52 says "the \texttt{delta} and \texttt{graph\_analysis} routers are registered before \texttt{sboms}"; `main.py:18-21 @28113e7` registers `analysis` before `sboms` too. HEAD has a ninth router (`priorities`, `main.py:26`), so "eight" holds only at 28113e7, which is correct for the thesis.
- Why it matters: Ch4 is where an examiner looks for the map from measure to endpoint to page. As written, the map sends bridges to an out-of-scope module and leaves the graph-theory and HITS endpoints out.

## C11. Identifier claims are broader than the code or the data support
- Type: overstated / inconsistency
- Origin: pre-existing
- Severity: medium
- Thesis: Chapters/04-System-Design.tex:90 — "This chain … ensures stable, unique node identifiers across SBOMs produced by different tools with varying levels of metadata completeness"; :102 — "This tolerance is necessary because real-world SBOMs do not always maintain a complete correspondence between the \texttt{components[]} and \texttt{dependencies[]} sections."
- Evidence: (a) Uniqueness is not ensured. Two components that fall back to the same `name` (e.g. two versions of one package without `bom-ref`/`purl`) collapse into one node, and `G.add_node` silently overwrites the attributes of the first (`sbom_store.py:23-28`). (b) "Across SBOMs" does not hold: identifiers are per-graph, and `bom-ref` values are generator-specific. (c) Nothing in the thesis tests robustness to "different tools". All 90 evaluation SBOMs come from cdxgen 12.8.5 with spec 1.5 (`metadata.tools`), all 33,309 components have a `bom-ref`, and there are no dangling references, self-loops or duplicate identifiers (my survey). The guards are sound defensive coding, but the dataset never exercises them. (d) Ch5:4 says "Every node in that graph carries a \texttt{purl} attribute … as defined in \autoref{sec:node-representation}". Ch4's fallback chain explicitly allows nodes without one, and those nodes cannot be matched to any finding (`vulnerabilityAPI.js:89`, `sbom-graph/index.js:105` filter on purl).
- Why it matters: The sentence turns a fallback heuristic into a guarantee. State what the chain does: it picks the first available field, does not guarantee uniqueness, and was exercised only on cdxgen SBOMs. Then Ch5:4 and Ch4 will agree.

## C12. Both full-window figures show "Depth: 1" for code-server, a frontend defect that is not disclosed
- Type: omission
- Origin: pre-existing
- Severity: medium
- Thesis: Chapters/04-System-Design.tex:39-41 (Fig. 4.2) and :213-215 (Fig. 4.4) — both show the status bar "code-server | Nodes: 181 | Edges: 263 | Depth: 1"; the text never mentions the status bar.
- Evidence: `GraphVisualizer.js:281-293 @28113e7` (same at HEAD, line 288) computes depth from `node.ancestors()`, which in Cytoscape returns *compound-parent* ancestors, not graph predecessors. It is always empty here, so depth is always 1. per_sbom.csv gives `maxDepth` = 7 for code-server, and Ch7 reports depth as a key RQ1 measure.
- Why it matters: The figures show a visibly wrong depth beside the correct node and edge counts. Under the "disclose known defects" decision, one sentence should say that the status-bar depth is a known defect and that depth is reported in Ch7 from the backend's `graph-analysis`.

## C13. "JSON validation" is not CycloneDX validation; non-CycloneDX JSON is stored as an empty graph, and some valid JSON gives a 500
- Type: overstated
- Origin: pre-existing
- Severity: low
- Thesis: Chapters/04-System-Design.tex:78 — "The first check is JSON validation … This guard ensures that no partially constructed graph can enter the store from a malformed file."
- Evidence: `sboms.py:26-29 @28113e7` checks only `json.loads`. There is no `bomFormat`/`specVersion` check. An SPDX JSON or any JSON object without `components` is stored with an empty graph, and all analyses then run on 0 nodes. A JSON array or scalar raises `AttributeError` at `cyclonedx.get` (`sboms.py:50`), which gives HTTP 500. The frontend's check (`SBOMUploader/index.js:27-29`) asks only for "components or metadata". Ch1:65 makes CycloneDX JSON the sole supported format.
- Why it matters: The format boundary in Ch1 is enforced only loosely. One clause ("the backend checks well-formed JSON, not conformance to CycloneDX") fixes it.

## C14. Duplicate detection is byte-exact, so "the same SBOM submitted more than once" means "the same file"
- Type: overstated
- Origin: pre-existing
- Severity: low
- Thesis: Chapters/04-System-Design.tex:80 — "prevents double-counting in metrics when the same SBOM is submitted more than once."
- Evidence: The hash covers the raw bytes (`sboms.py:32`). A re-serialised file, or a regenerated one (cdxgen writes a fresh `serialNumber` and timestamp), is not detected. After `PUT /sboms/{id}`, the stored hash is recomputed over `json.dumps({metadata, components, dependencies}, sort_keys=True)` (`sboms.py:95-98`), so re-uploading the original file is no longer recognised as a duplicate.
- Why it matters: This is minor, but "same SBOM" should read "byte-identical file".

## C15. The node attribute list and the type enumeration do not match the code or CycloneDX
- Type: wrong
- Origin: pre-existing
- Severity: low
- Thesis: Chapters/04-System-Design.tex:92 — "In practice, the persisted attributes are \texttt{name}, \texttt{version}, \texttt{purl}, \texttt{type}, and \texttt{bom-ref}"; :124 — "\texttt{type} --- CycloneDX component type; one of \texttt{library}, \texttt{framework}, or \texttt{application}"; :92 — "To avoid compatibility issues with NetworkX's attribute handling, only scalar values … are retained".
- Evidence: Across the 90 SBOMs, the scalar keys persisted are also `group` (33,309 of 33,309 components), `scope` (33,219), `description` (66) and `author` (64). For code-server: `{name, version, purl, type, bom-ref, group, scope, description}`. CycloneDX 1.5 defines more component types (container, operating-system, device, firmware, file, platform, device-driver, machine-learning-model, data). Only library/framework/application *occur* in this dataset (32,296 / 923 / 90). NetworkX accepts arbitrary attribute values, so the "compatibility" rationale is doubtful. The filter's practical effect is to drop `licenses`, `hashes`, `externalReferences` and `properties`.
- Why it matters: These are small factual errors in the data-model definition, which later chapters cite. Say "including …" and "in this dataset".

## C16. Interaction model: some gestures are as at HEAD only, and the zoom readout is placed wrongly
- Type: out of date / wrong
- Origin: pre-existing
- Severity: low
- Thesis: Chapters/04-System-Design.tex:207 — "Tapping a node selects it … Double-tapping a vulnerable node opens its records on the Vulnerabilities page."; :218 — "The current zoom level is tracked in React component state and displayed in the toolbar."
- Evidence: At 28113e7, a *single* tap on a node carrying a severity class navigates to the Vulnerabilities page (`GraphVisualizer.js:719-725`, `sbom-graph/index.js:1287-1288`). A long press calls `handleOpenNodeDetails`, which only `console.log`s (`:707-709`, `index.js:1285`). `dbltap` appears only at HEAD (`GraphVisualizer.js:769`). This item does not dispute the screenshot-driven Component section. It concerns a gesture that no figure shows, so under the "28113e7 is the evaluated system" rule it should either describe 28113e7 or say "in the current version". The zoom percentage is in the canvas status bar, not the toolbar, at both commits (`index.js:1309-1311 @28113e7`). Figs. 4.2 and 4.4 show it bottom-right under the canvas.
- Why it matters: These are minor, but the text contradicts the figures (zoom) and silently uses HEAD behaviour (double-tap).

## C17. The `/health` description does not match its use, and contradicts Ch4:46
- Type: wrong / inconsistency
- Origin: pre-existing
- Severity: low
- Thesis: Chapters/04-System-Design.tex:62 — "A health endpoint at \texttt{GET /health} returns a JSON object confirming the service status and version number; the frontend uses this to detect backend availability before enabling API-mode features."
- Evidence: `main.py:28-30 @28113e7` returns `{status, version, maxSboms}`. The only frontend caller (`useSBOMLibrary.js:34-43`) reads `maxSboms` and ignores failures. API mode is the compile-time constant `DATA_SOURCE = "api"` (`config/dataSource.js:12`), as Ch4:46 itself says, so nothing is "enabled" by the health check. The eval pipeline uses `/health` as a readiness probe (`lens.py:50-55`).
- Why it matters: Two paragraphs of the same section give incompatible accounts of how API mode is chosen.

## C18. Terminology departs from the glossary in a few places
- Type: inconsistency
- Origin: pre-existing
- Severity: low
- Thesis: Chapters/04-System-Design.tex:164 — "the anchor of the dependency tree"; :100 — "those dependants"; :35 — "the Dashboard (the SBOM library …)" vs Fig. 4.1 "SBOM Library" as a view (`architecture.mmd:7`).
- Evidence: `sbom-lens-thesis/CONTEXT.md`: Dependency graph — "_Avoid_: dependency tree (it is not a tree)". The rest of the thesis spells it "dependents" (Ch3:132,134; Ch8). `sbom-lens/CONTEXT.md`: the SBOM Library is "shown on the Dashboard", so the page is the Dashboard, while the figure labels the view "SBOM Library". (Ch5:221 also says "dependency tree"; that is outside this review.)
- Why it matters: Cosmetic, but the glossary exists to prevent exactly these slips.

## Summary by severity
- High (3): C1 MAX_SBOMS 10 vs 50; C2 root never drawn green (code and all four figures); C3 tree/radial layouts are hub- or degree-rooted, not root-rooted, and the Fig. 4.3 caption misreads the figure.
- Medium (9): C4 second frontend parser; C5 undirected copies vs "directly on DiGraph"; C6 caching rationale and invalidation; C7 latency claims; C8 no tests at 28113e7; C9 blast radius as ancestors only, plus the phantom topological ordering; C10 optimiser scope and endpoint map; C11 identifier guarantees and Ch5:4; C12 "Depth: 1" defect in the figures.
- Low (6): C13–C18.
