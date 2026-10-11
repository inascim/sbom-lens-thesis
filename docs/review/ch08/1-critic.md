# Chapter 8 — Critic

What I checked. I read Ch8 line by line against Ch1 (objectives, contributions, scope), the Abstract, the current text of Ch2–Ch7 and the sorted reports for Ch3, Ch5, Ch6 and Ch7. I recomputed every Ch8 number from `sbom-lens-eval/analysis/out/per_sbom.csv`, using the venv Python with numpy and csv (pandas is not installed). I checked each system claim against `git -C /home/inasc/projects/sbom-lens show 28113e7:<path>` (marked `@28113e7`), and checked at HEAD (`5273e78`) whether each code fact still holds. I did not reopen the decisions already taken: inclusive blast radius, 8,868 findings, the 28113e7 scope, Fix Priorities, screenshots, namespace scoping, the HITS and power-law handling, tie-aware overlap and the reproducibility scope.

All Ch8 numbers match Ch7 and the data. The corrections from the earlier reviews (79 fits, "single consumer", the RQ1 headline, the Ch8:51 sample wording) have been applied. What remains are claims Ch8 makes in its own words, where it says more than the corrected chapters do. These are about timing, reach, reproducibility, how current the data is, the objectives, and the limitations. I found no high-severity item.

## C1. "Structural analyses took milliseconds … at most a few seconds" leaves out the audit, which computes four of the structural measures Ch8 lists
- Type: overstated
- Origin: pre-existing
- Severity: medium
- Thesis: Chapters/08-Conclusion.tex:18 — "its structural analyses took milliseconds in the median case and at most a few seconds"
- Evidence: `backend/routers/audit.py:240-248 @28113e7` and `backend/README.md:78` show that the `audit` endpoint returns degree entropy, cycles, version conflicts, namespace concentration and unmaintained packages. Ch8:30 lists the first four as RO3 whole-graph structural measures. Recomputed from `per_sbom.csv`: `time_audit` median 4.72 s, max 16.39 s; total per SBOM median 5.28 s, max 17.18 s (Tab. 7.6). Without the audit, the sum of all other endpoints is median 0.44 s and max 4.47 s. Ch7:151 is careful ("Every other analysis had a median between 2 and 150 milliseconds"). Ch8 drops the "other". Each time is also a single wall-clock REST round trip (Ch7 sort, V1), not a measured UI interaction.
- Why it matters: The only performance sentence in the conclusion quietly excludes the slowest analysis. An examiner who opens Tab. 7.6 sees 17 s, not "a few seconds". The audit's time is network-bound, which is a fair explanation, but the sentence has to give it.

## C2. The RQ3 headline calls exposure "hidden from a flat inventory", then quotes the inclusive 19 %, which counts the vulnerable components the inventory does list
- Type: inconsistency
- Origin: pre-existing
- Severity: medium
- Thesis: Chapters/08-Conclusion.tex:16 — "\textbf{RQ3: most vulnerability exposure is structural and hidden from a flat inventory.} … and 19\% of the application lies in the blast radius of at least one known vulnerability."
- Evidence: Ch7:132 now reports both numbers: 19 % inclusive, "It includes the vulnerable components themselves (a median 8\%)", and 12 % (4 %–29 %) for dependents only. Ch7:134 states both ("a fifth … is vulnerable or depends on …, and an eighth depends on one"). Recomputed: reach median 0.186. Ch8 gives only the 19 %, and gives it under a headline about what a flat inventory hides. The vulnerable components themselves are exactly what a flat inventory shows. The same applies, more mildly, to the 75 % transitive-only share. A flat list does show those components. What it hides is how they entered (Ch7:128 says this correctly).
- Why it matters: This does not reopen the 19 %/12 % decision. It is about Ch8 matching Ch7's two-number reporting. Read with its headline, the 19 % overstates hidden exposure by about half.

## C3. Reproducibility: "published with the thesis, so each reported number can be regenerated" is broader than Ch7's scoped claim, and the repositories are not publicly reachable
- Type: overstated / unsupported
- Origin: pre-existing
- Severity: medium
- Thesis: Chapters/08-Conclusion.tex:32 — "The collection pipeline, the SBOMs and every analysis result are published with the thesis, so each reported number can be regenerated."
- Evidence: (a) Scope. Ch7:169, after the D4 decision, says the script regenerates Tabs. 7.2–7.6 and the figures, and "the selection funnel and the remaining counts in the text are computed from the same data". It also says the run record does not include the library versions, which affect HITS (and, per the Ch7 sort V2, the powerlaw x_min and α). Ch8 restores the unscoped "each reported number". (b) Availability. No chapter gives a URL or DOI for `sbom-lens-eval` or `sbom-lens` (`grep` of Chapters/ and the bib). The eval repo's origin is `git@github.com:inascim/sbom-lens-eval.git`. Anonymous `https://api.github.com/repos/inascim/sbom-lens-eval` and `/repos/inascim/sbom-lens` both return HTTP 404 today, so both are private or absent. I could not verify whether they will be made public at submission.
- Why it matters: The claim is a contribution under RO4. As written, it repeats the over-broad claim that the Ch7 review scoped down, and a reader has no address to check it against.

## C4. Future work proposes "comparison of SBOMs across versions", but the evaluated system already has an SBOM Comparison/Delta module, which Ch4 acknowledges
- Type: inconsistency
- Origin: pre-existing (also at HEAD)
- Severity: medium
- Thesis: Chapters/08-Conclusion.tex:67 — "Persistent storage, support for SPDX input, and comparison of SBOMs across versions of the same application would turn the prototype into a tool for continuous use"
- Evidence: `backend/routers/delta.py:14-80 @28113e7` (`GET /sboms/compare`: added/removed/unchanged nodes and edges, a drift score and label) and a second comparison endpoint with `_comparison_cache` (:84-153). `src/routes.js:65-69 @28113e7` has the "SBOM Comparison" route, with `src/layouts/sbom-comparison` and `src/layouts/sbom-delta`. Ch4:22 says "The architecture also includes modules not evaluated in this thesis --- a Topology Optimiser and a Comparison/Delta analysis view". Ch4:35 lists "an SBOM Comparison view" in the navigation. Ch1:69 also says delta analysis "would require a temporal data model not present in the current system".
- Why it matters: A future-work item that is already in the navigation bar of the evaluated tool will be noticed. The item should be worded as extending and evaluating the existing comparison view, for example with structural-risk tracking across versions.

## C5. "Current vulnerability data" and "the displayed results always describe the current graph" contradict Ch5: OSV is queried once, at upload, and never for edits
- Type: inconsistency
- Origin: pre-existing (also at HEAD)
- Severity: medium
- Thesis: Chapters/08-Conclusion.tex:43 — "Querying OSV directly gives every analysis the same, current vulnerability data"; :26 — "Edits are written back to the backend, which rebuilds the graph and invalidates every cached analysis, so the displayed results always describe the current graph."
- Evidence: Ch5:173 (corrected): "Vulnerabilities are fetched once, at upload: selecting an SBOM or reloading the page does not query OSV again, so an advisory published later reaches an SBOM only if it is uploaded again." The only call site of `fetchVulnsForSBOM` is `src/layouts/dashboard/index.js:86 @28113e7` (`handleUploaded`), and this is unchanged at HEAD. The graph page's `handleSave` (`src/layouts/sbom-graph/index.js:402-413`) PUTs the edited SBOM, and `handleSaveAs` (:415-437) uploads a copy. Neither queries OSV. So a component added by editing never receives a vulnerability lookup. The backend half of :26 is correct: `sboms.py:79-111` rebuilds the graph, and `invalidate_graph_analysis_caches` (`graph_analysis.py:458-468`) clears all six analysis caches plus the audit cache. The only uncleared caches are the out-of-scope `_delta_cache` and `_comparison_cache` (`delta.py:11,84`).
- Why it matters: The VEX trade-off at :43 rests on "current" data, and the RO1 claim rests on "always". The corrected Ch5 says neither holds for vulnerability data, and the Implementation limitation (:53) does not mention staleness.

## C6. The limitations omit the implementation-dependent results disclosed in Ch6 and Ch7: HITS non-convergence, the powerlaw bound, and library-version dependence
- Type: omission
- Origin: pre-existing
- Severity: medium
- Thesis: Chapters/08-Conclusion.tex:41-53 — the Limitations section. It has no paragraph on any of these. :14 and :33 report HITS results with no n.
- Evidence: Ch7:109 says that for HITS, "In 8, power iteration had not converged after 200 iterations … and SBOM Lens then reports every score as zero"; 3 more are all-zero. `impact.py:72-79 @28113e7` returns 0.0 on `PowerIterationFailedConvergence`. Ch7:77 says that in 8 graphs the powerlaw fit is marked failed and "SBOM Lens returns these fits with α = 3". Ch7:169 says "the run record does not include these library versions, and the NetworkX version affects the HITS results". The Ch6 sort (V3) says Louvain partitions depend on the NetworkX version. The Ch7 sort (V2) says powerlaw 1.5 changes x_min in 74 of 87 graphs. Recomputed: `tau_authority` and `overlap_authority` are defined for 79 of 90.
- Why it matters: These are real limitations of the tool, not only of the evaluation. The tool silently reports zeros or a bound as if they were results. Ch8:47-49 lists weaker caveats (thresholds, baselines) and leaves these out. A reader of Ch8 alone would not know that 11 of 90 graphs have no authority ranking.

## C7. The evaluation-sample limitation omits generator quality: the pnpm loss and the graphs that are lower bounds
- Type: omission
- Origin: pre-existing
- Severity: medium
- Thesis: Chapters/08-Conclusion.tex:51 — "The evaluation covers popular npm applications only, not organised as monorepos, … Each of these filters shapes the sample"
- Evidence: Ch7:161: "cdxgen produced no dependency edges from the root for 23 projects with a pnpm lockfile, so only one pnpm project remains, and the findings describe npm and yarn installations." Ch7:159: four graphs fall below 90 % lockfile agreement, two miss 14 % and 20 % of production packages, and in seven SBOMs "the size, reach and vulnerability counts of those graphs are lower bounds". The 37 yarn/pnpm graphs could not be checked. The Ch7 sort (E9) proposed adding "npm and yarn" to Ch8:51 as an option, and it was not taken. Ch7:157 also notes that the RQ2 baseline is the severity band, not numeric CVSS or EPSS, and Ch8 does not carry this over.
- Why it matters: Ch8 presents the sample as shaped only by the author's filters. The generator, an uncontrolled tool, removed nearly every pnpm project and truncated some graphs. Both affect how far the results generalise.

## C8. Ch8 maps contributions to objectives but never says whether each objective was met; RO3's "principled basis for prioritising remediation" is not shown by the evaluation
- Type: omission / overstated
- Origin: pre-existing
- Severity: medium
- Thesis: Chapters/08-Conclusion.tex:24 — "The four research objectives … map one-to-one to the contributions of this work."; :30 (RO3 paragraph); :18 — "the position of a component in the dependency graph carries risk information that a flat inventory of components and vulnerabilities does not"
- Evidence: Ch1:37 promises that the objectives allow "the conclusions drawn in \autoref{cp:conclusion} to be verified against each stated goal". Ch1:42 (RO3) aims "to give the analyst a principled basis for prioritising remediation beyond raw CVSS scores". Ch7:111: "A τ_b near zero does not show that the structural ranking is better than the severity ranking". The settled framing is that the evaluation shows structure carries information severity does not, and not that structural prioritisation is better. Ch8:30 describes only what is computed. It never says that the prioritisation half of RO3 is untested, and no limitation says the evaluation has no ground truth (exploitation or analyst outcome). Ch8:65 proposes a user study about whether "the structural view changes the decisions", which is a different question. :18 also turns "information" into "risk information", which assumes the point that is not tested.
- Why it matters: The conclusion is where an examiner checks objectives against results. Without a statement that RO3 is met for the measures but not shown for prioritisation, a reader can take the RO3 objective as achieved. That runs against the Abstract's reworded framing.

## C9. "Existing SBOM tooling treats an SBOM as an inventory … SBOM Lens treats it as a directed dependency graph" drops the qualifications settled in Ch3
- Type: overstated
- Origin: pre-existing
- Severity: medium
- Thesis: Chapters/08-Conclusion.tex:4 — "Existing SBOM tooling treats an SBOM as an inventory of components and their known vulnerabilities; SBOM Lens treats it as a directed dependency graph and analyses that graph."
- Evidence: Ch3:20: "the SBOM's dependency relationships are either discarded, displayed, or at most traversed as a tree, to find the path to a component or the components that depend on it". Ch3:134: Sunshine lists dependents, Dependency-Track opens the path, and GUAC's patch planning searches dependents. Ch3:132 limits "treat an SBOM as an inventory" to "the surveyed tools" and qualifies it in the next clause. Settled framing: the novelty is narrow, namely dominance, structural measures, and reach plus paths on one deduplicated graph.
- Why it matters: The opening sentence of the conclusion restates the broad novelty claim that the Ch3 review narrowed. "Existing SBOM tooling" is also wider than the four tools surveyed.

## C10. The "vocabulary" contribution claims measures "commonly reported in the literature and in earlier versions of SBOM Lens" were misnamed; no chapter shows either
- Type: unsupported
- Origin: pre-existing (also at HEAD)
- Severity: medium
- Thesis: Chapters/08-Conclusion.tex:35 — "Several measures commonly reported in the literature and in earlier versions of SBOM Lens were found, on inspection, to compute something other than their names suggested; the thesis reports each measure as it is actually computed."
- Evidence: No chapter identifies a measure in the literature that computes something other than its name, or describes an earlier version of SBOM Lens. The misnamings the thesis does disclose are in the evaluated version itself. Ch6:48: the spectral exposure score "is named \texttt{r0} for historical reasons" (`graph_analysis.py:357` and `:365` `globalR0` @28113e7). Ch6:116: version conflicts are "labelled \emph{Diamond Dependencies} in the interface". `graph_analysis.py:430` explains abstraction candidates as "elevated R₀ score". The paragraph also adds a fifth contribution, after :24 says the four objectives "map one-to-one to the contributions" and Ch1:50 says "This work makes four contributions".
- Why it matters: A contribution claim needs a reference. As it stands the sentence implies a critique of the literature that the thesis does not make, and it places in "earlier versions" names that the evaluated tool still shows.

## C11. RQ1 summary: "every graph is small-world … tightly clustered" drops Ch7's weak-bound caveat, and the measures were not "developed for ecosystems"
- Type: overstated / misattribution
- Origin: pre-existing
- Severity: low
- Thesis: Chapters/08-Conclusion.tex:12 — "every graph is small-world: components form tightly clustered neighbourhoods while remaining about five hops apart … Graph measures developed for ecosystems therefore have structure to work with at the scale of one application."
- Evidence: Ch7:79: "The bound itself is weak: … in the median graph, any clustering above about 0.02 already gives σ > 2". The median C is 0.171 (recomputed). Ch7:90 attributes small-world structure to software-internal studies (Valverde2002, Myers2003), not to ecosystems (Ch3 sort E7). In Ch8 it sits under a headline about "structure found in whole ecosystems". Ch7:90 says "The structural measures of \autoref{cp:network-sci} therefore have structure to work with". Those measures (articulation points, HITS, k-core, Louvain, spectral) are general network science, not "developed for ecosystems". The "five hops" is the undirected path length on the largest component (`_compute_small_world`, `graph_analysis.py:151-174`), which Ch6:184 says "is not exposure".
- Why it matters: These are small, but each one moves the reading a little further than Ch7 or Ch3 allow.

## C12. RQ1 range "between 2.1 and 3.0" invites reading the upper end as the package bound Ch7 excludes
- Type: inconsistency
- Origin: pre-existing
- Severity: low
- Thesis: Chapters/08-Conclusion.tex:12 — "with exponents between 2.1 and 3.0 (median 2.6)"
- Evidence: Recomputed: 79 fits, min 2.086, median 2.585, max 2.987. Ch7:77 and Tab. 7.2 give 2.09–2.99. The rounding is arithmetically correct, but 3.0 is exactly the powerlaw 2.0.0 bound that Ch7:77 uses to exclude eight fits.
- Why it matters: "Below 3" or "2.1 to 2.99" avoids the reader asking whether the bound is still in the range.

## C13. RQ2 summary: the zero top-k overlap is presented as further evidence, though Ch7 now says it is what an unrelated ranking gives; "unrelated" is stronger than Ch7 and the Abstract
- Type: overstated
- Origin: pre-existing
- Severity: low
- Thesis: Chapters/08-Conclusion.tex:14 — "gives unrelated orderings … and in most applications none of the critical components is among the equally many components with the largest blast radius"; :33 — "were found to be unrelated (median \(\tau_b = 0.00\))"
- Evidence: Ch7:107: "an unrelated ranking would place on average a share k/N of them in the top k (a mean of 0.13); the observed means are 0.09 … and 0.12 …, so the overlap restates the τ_b result in practical terms rather than adding to it". Recomputed: zero overlap in 61/90 (blast) and 47/79 (authority). Ch7:107 and the Abstract (EN :21, after the Ch7 sort E2) say "essentially unrelated". 26 of 90 SBOMs have |τ_b| ≥ 0.2 (Ch7:107: 17 ≤ −0.2, 9 ≥ 0.2). The Ch7 sort judged :33 acceptable, so this is minor. But :14 now uses the overlap as a stand-alone finding, and Ch7 no longer does.
- Why it matters: The conclusion should not add weight to the overlap that Ch7 explicitly takes away.

## C14. "The dependency paths through which it enters" overstates the impact output: at most five shortest paths
- Type: overstated
- Origin: pre-existing
- Severity: low
- Thesis: Chapters/08-Conclusion.tex:28 — "for any vulnerable component the analyst obtains its blast radius, the dependency paths through which it enters the application, and its dominated set"
- Evidence: `impact.py:52-63 @28113e7`: `nx.all_shortest_paths` from each root, `break` at 5. Ch5:221: "up to five shortest paths from the root packages".
- Why it matters: A vulnerable component can enter through many paths of different lengths. "All paths" is the reading the sentence invites, and the tool does not give it.

## C15. RO2 and Limitations: severity handling is described more cleanly than the code. v3.0 vectors are scored with the v3.1 formula, an unparseable v3 vector does not fall back to the label, and UNKNOWN is drawn in the LOW colour
- Type: error (minor)
- Origin: pre-existing (also at HEAD)
- Severity: low
- Thesis: Chapters/08-Conclusion.tex:28 — "Severity is derived from the CVSS v3.1 vector …, falling back to the advisory's own severity label, and otherwise recorded as unknown rather than silently treated as low. Vulnerable components are coloured on the graph by their highest severity band"; :41 — "Severity is computed from CVSS v3.1 vectors only"
- Evidence: `vulnerabilityAPI.js:144-147 @28113e7` selects any entry of type `CVSS_V3`. The OSV schema uses that type for both v3.0 and v3.1 vectors. `parseCVSSSeverity` (:61-70) scores any string starting with `CVSS:` through `cvssV3BaseScore`, which never checks the version prefix (:14-20). So v3.0 vectors are scored with the v3.1 formula, not excluded. If a `CVSS_V3` entry exists but cannot be scored, the result is UNKNOWN without trying `database_specific.severity` (:145-147). Ch5:108: "On the graph, UNKNOWN nodes are drawn with the LOW colour". The evaluation data does not store vectors, so I could not count the v3.0 cases (unverified). No evaluation finding was UNKNOWN (Ch7:51).
- Why it matters: No result depends on this. But the summary of RO2 should not be cleaner than Ch5, which already discloses the colour mapping.

## C16. The undirected-measures limitation omits the small-world coefficient, the one measure Ch8 itself reads as distance
- Type: omission
- Origin: pre-existing
- Severity: low
- Thesis: Chapters/08-Conclusion.tex:47 — "Several measures --- articulation points, bridges, core numbers, communities, the rich-club coefficient and all spectral measures --- are computed on the undirected projection"
- Evidence: `_compute_small_world` uses `G.to_undirected()` and its largest connected component (`graph_analysis.py:151-174 @28113e7`). Ch6:184: "in the undirected projection, every component is close to every other. Closeness in \(G_u\) is not exposure". The Ch6 sort (E5) relied on "σ describes undirected interlinking, as Ch8:47 says", but Ch8:47 does not list σ, C or L.
- Why it matters: Ch8:12 reports "about five hops apart" without qualification. The limitation meant to cover it does not name it.

## C17. RO1 says every declared dependency becomes an edge; dangling references and self-loops are dropped
- Type: overstated
- Origin: pre-existing
- Severity: low
- Thesis: Chapters/08-Conclusion.tex:26 — "a directed dependency graph in which every component is a node and every declared dependency a consumer-to-dependency edge"
- Evidence: `_build_graph` (`store/sbom_store.py:14-34 @28113e7`). It adds an edge only if `ref != target and G.has_node(ref) and G.has_node(target)`. Components with no `bom-ref`, `purl` or `name` get no node. Components sharing a `bom-ref` collapse to one node. Nested `components` are not read. Ch4:102 already says "Entries in \texttt{dependencies[]} that reference undeclared components are silently skipped."
- Why it matters: This is minor, but it is the one sentence that defines the graph in the conclusion, and Ch4 is more precise.

## C18. Two internal mismatches around the vulnerability-source discussion Ch8 hosts
- Type: inconsistency
- Origin: pre-existing
- Severity: low
- Thesis: Chapters/08-Conclusion.tex:41 — "SBOM Lens relies on OSV as its only vulnerability source"; :61 — "Direct integration of NVD by CPE … would reduce dependence on OSV's aggregation"
- Evidence: Ch2:50 refers forward to this discussion in Ch8 but says "Vulnerability data is sourced directly from NVD, OSV, and GitHub Security Advisories rather than from embedded VEX data". That contradicts Ch8:41 and Ch5:20 ("OSV as its sole vulnerability source"), and the code (`vulnerabilityAPI.js:95`, the OSV `querybatch` only). Ch5:16 argues that CPE matching "is imprecise: a package may have several CPE strings or none, and name-based matching can return vulnerabilities for unrelated products". Ch8:61 proposes CPE matching without addressing this.
- Why it matters: Ch8 is correct on the source, but the chapter that points to it (Ch2:50) is not. The NVD/CPE proposal reads as if Ch5's argument against it had not been made.

## Checked and holds
- Ch8:10: 90 npm applications, each with at least one CRITICAL vulnerability (Ch7:20, Tab. 7.1).
- Ch8:12: median 332 nodes (Tab. 7.2). Direct share median 0.115 → 11 %. A power law fitted in 79 of 90, matching Ch7:77 after the D1-A decision. Every σ > 1 (min 2.18). Median L 5.01. Version conflicts median 23.5 → 24 (Tab. 7.2 prints 24). Graphs with cycles: 30, which equals the graphs with a non-trivial SCC. The headline wording "heavy-tailed structure found in whole ecosystems" matches the Ch3 sort E7.
- Ch8:14: median τ_b 0.003 (blast) and −0.004 (authority) → 0.00. Zero overlap in 61/90, so "in most applications" holds. "Package with a single consumer" matches the Ch7 sort E13.
- Ch8:16: transitive-only median 0.75. Minimum chokepoints 1, so "every application" holds. Chokepoint share median 0.453 → 45 %. Dominated share median 0.214 → 21 %. Reach median 0.186 → 19 %, inclusive as decided.
- Ch8:18: max nodes 995 ("up to a thousand"). The non-audit endpoint medians are 1.6–151 ms ("milliseconds").
- Ch8:33: pooled transitive-only share 0.749, so "three-quarters" holds either per SBOM or pooled.
- Ch8:26: the backend rebuilds the graph on PUT and invalidates all six analysis caches and the audit cache (`sboms.py:79-111`, `graph_analysis.py:458-468`). JSON export exists (`sbom-graph/index.js:342`). Cytoscape rendering and editing exist.
- Ch8:30: every listed measure exists at 28113e7 (`graph_analysis.py`, `audit.py`, `impact.py`). Louvain, k-core, rich-club, articulation points and bridges use `to_undirected()`.
- Ch8:43: no code path reads the CycloneDX `vulnerabilities` array. Ch2:50 lists it as not consumed.
- Ch8:45: the component-level over-approximation matches Ch3:70, Ch7:157 and the Ch5 sort E7.
- Ch8:47: spectral measures, articulation points, bridges, core numbers, communities and rich-club are undirected in the code.
- Ch8:49: entropy labels at 1.5/3.0 bits (`audit.py:41-43`); namespace flag `top_pct > 60` (`audit.py:139`); rich-club threshold `v > 0.5`, unnormalised (`graph_analysis.py:283-287`); `FAN_IN_THRESHOLD = 3` (:421); closed-form `C_rand = p` and `L_rand = ln n / ln(np)` (:186-188). No likelihood-ratio test in the code or in Ch7, consistent with the D3 decision.
- Ch8:51: matches the Ch7 sort E18 wording.
- Ch8:53: in-memory `SBOMStore` and `VulnStore`; CORS for localhost only, with no auth dependency on any router; CycloneDX JSON only (Ch1:65, Ch4:20).
- Ch8:59: no SIR or other simulation exists at 28113e7. `_compute_epidemic` is purely spectral (`graph_analysis.py:343-366`), consistent with Ch2:216 and Ch6:126.
- Ch8:61 (EPSS/KEV, CVSS 4.0, VEX), :63 (call-graph reachability), :65 (PyPI/Maven, user study) and :67 (persistence, SPDX): none exists at 28113e7. Each is consistent with the forward references in Ch2:216, Ch2:278, Ch3:70, Ch4:20, Ch5:20 and Ch1:65-67.
- No Fix Priorities or other post-evaluation feature appears as a contribution.
- Every `\autoref` in Ch8 resolves to an existing label.
