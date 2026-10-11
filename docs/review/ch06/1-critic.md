# Chapter 6 — Critic

Method. I read `Chapters/06-Network-Science.tex` line by line. I checked every formula, threshold and endpoint claim against `backend/routers/{graph_analysis,impact,audit,metrics,analysis}.py` and `backend/store/sbom_store.py` @28113e7. Code references marked `@28113e7` come from `git -C /home/inasc/projects/sbom-lens show 28113e7:<path>`. The evaluation pinned `dfc8e23` (`sbom-lens-eval/config.py:18`). Between the two commits, the only backend changes are `analysis.py`, `main.py`, `sbom_store.py` (+2/−1) and README files, so the measures are identical. I exported the backend to `/tmp/claude-1000/ch06-critic/` and re-ran the code on all 90 SBOMs in `sbom-lens-eval/data/sboms`. I also recomputed values from `data/results/*.json`. Library versions come from `sbom-lens/venv`, the venv the pipeline used (`sbom-lens-eval/README.md`, "backend dependencies installed in `sbom-lens/venv`"): networkx 3.7 (installed 2026-09-30 16:44, before the run started at 21:45 according to `data/run.json`), scipy 1.18.1, numpy 2.5.3, powerlaw 2.0.0. `backend/requirements.txt` only gives `networkx>=3.6.1`. For comparison I downloaded the networkx 3.6.1 wheel. I opened these primary sources: Van Mieghem et al. 2011 (pure.tudelft.nl PDF), Colizza et al. 2006 (arXiv physics/0602134), Humphries & Gurney 2008 (PLoS ONE PDF), and Chakrabarti et al. 2008 (the TISSEC journal version of Wang et al. 2003; the SRDS PDF itself was not reachable). I cross-checked Ch2 (§2.2–2.3), Ch4 graph construction (sec:graph-construction), Ch5 §5.5–5.6, Ch7 and CONTEXT.md. I also looked at both Ch6 screenshots.

## C1. The HITS description matches networkx 3.6.1, but the evaluation ran on networkx 3.7, whose default is power iteration; the 8 convergence failures behind Ch7's excluded SBOMs come from that switch
- Type: error
- Origin: pre-existing (still true at HEAD: `impact.py:90,119` call `nx.hits(G, max_iter=200)` and `requirements.txt` still has `networkx>=3.6.1`)
- Severity: high
- Thesis: Chapters/06-Network-Science.tex:32 — "NetworkX obtains the authority vector as the leading singular vector of the adjacency matrix \(\mathbf{A}\) --- equivalently, the principal eigenvector of \(\mathbf{A}^{\top}\mathbf{A}\) --- and the hub vector as \(\mathbf{h} = \mathbf{A}\mathbf{a}\) ... If the computation fails to converge within the iteration limit, all scores are reported as zero."
- Evidence: In networkx 3.6.1, `hits` calls `sp.sparse.linalg.svds(A, k=1, ...)` and sets `h = A @ a`. This is exactly the text. In networkx 3.7, the version installed in `sbom-lens/venv` before the run, `hits(..., method="power_iteration")` is the default. `_hits_power_iteration` iterates `a = h @ A; h = A @ a; h /= h.max()` and raises `PowerIterationFailedConvergence` after `max_iter`. `impact.py:72,101 @28113e7` pass no `method`. Recomputation on the 90 SBOMs: 8 graphs have all-zero HITS in `global-analysis` (CopyTranslator, stackedit, CyberChef, lowlighter/metrics, marktext, parse-server, typeorm, wekan). For each of the 8, `nx.hits(G, max_iter=200)` raises under 3.7. With `method="svd"`, which is the 3.6.1 behaviour, all 8 converge and give 2 to 31 distinct authority values among their vulnerable components. So all 8 could have been ranked. Ch7:109/157 report 11 unrankable SBOMs "because the HITS computation did not converge".
- Why it matters: The text describes an algorithm the evaluated environment did not run. The failures it discloses as an inherent limit come from an unpinned dependency. Seven of Ch7's 11 RQ2 authority exclusions are not inherent to HITS. Either pin networkx/`method="svd"` and rerun, or describe power iteration and the version.

## C2. Ch7 attributes all 11 unrankable SBOMs to non-convergence; only 8 failed to converge, and in the other 3 HITS converged but gave every vulnerable component an authority of zero
- Type: inconsistency
- Origin: pre-existing
- Severity: medium
- Thesis: Chapters/06-Network-Science.tex:32 — "If the computation fails to converge within the iteration limit, all scores are reported as zero." Relied on by Chapters/07-Evaluation.tex:109 — "11 SBOMs could not be ranked: in these graphs the HITS computation did not converge within the iteration limit set in SBOM Lens, which then reports every score as zero (\autoref{sec:hits})"
- Evidence: `analyse.py:109` sets τ to NaN when `len(set(values)) <= 1`. For MagicMirror, ChatALL and element-web, `nx.hits` converges (global-analysis scores are non-zero), but `impact[...]["hitsAuthority"]` is 0.0 for every one of their 28/61/17 vulnerable components. Under the SVD method these 3 remain constant too (1 distinct value). So HITS authority concentrates on a block of the graph that contains none of the vulnerable components.
- Why it matters: Two different failure modes are reported as one. The second one is a property of HITS on these graphs (see C3), not an iteration limit.

## C3. HITS authority does not "rank libraries by how central they are": on these DAGs it localises on a small block, and most components get an authority of (near) zero
- Type: overstated
- Origin: pre-existing
- Severity: medium
- Thesis: Chapters/06-Network-Science.tex:34 — "Authority is the more useful of the two for risk: it ranks libraries by how central they are to the dependency structure, weighting a consumer's vote by that consumer's own importance rather than merely counting consumers as in-degree does."
- Evidence: Over the 82 SBOMs where HITS converged, the share of nodes with a non-zero authority (after the 6-decimal rounding of `impact.py:108`) has a median of 0.46 (min 0.07). In the median SBOM, half of the vulnerable components have authority 0. In the first SBOM (476 nodes), 325 nodes have an authority below 5e-7. A rank-1 leading singular vector localises like this when the graph has a dominant dense bipartite block (Kleinberg's "tightly-knit community" effect). C2 shows the extreme case.
- Why it matters: As an ordering of a project's libraries, authority is undefined (tied at zero) for most of them. Ch7's RQ2 result for authority partly reflects these ties, not a structural ordering. The sentence should say that authority ranks the components of the dominant hub–authority block.

## C4. "Chapter 7 compares them" is false: Chapter 7 never compares articulation points with dominance
- Type: inconsistency
- Origin: n/a (text)
- Severity: medium
- Thesis: Chapters/06-Network-Science.tex:26 — "SBOM Lens reports both and \autoref{cp:evaluation} compares them."
- Evidence: `grep -n -i articulat Chapters/07-Evaluation.tex` finds no match. RQ1 lists its measures (Ch7:57) without articulation points. RQ3 uses only dominated sets (Ch7:130). `analyse.py` never reads `global-analysis.articulationPoints`.
- Why it matters: This promises an empirical result that does not exist. Either delete the clause or add the comparison, for example the overlap between articulation points and chokepoints, which is cheap to compute from `data/results`.

## C5. The summary misstates RQ2: Chapter 7 compares severity bands, not CVSS scores, and uses two structural measures, not "these measures"
- Type: inconsistency
- Origin: n/a (text)
- Severity: medium
- Thesis: Chapters/06-Network-Science.tex:186 — "by comparing rankings based on these measures with a ranking by CVSS score alone."
- Evidence: Ch7:59 ranks "by their worst severity band (critical > high > medium > low)" against blast-radius size and HITS authority only. Ch7:157 says explicitly that rankings "by numeric CVSS score ... were not measured". `analyse.py:107` covers `("blast", "authority")` only.
- Why it matters: The chapter's last sentence sets up an evaluation that Chapter 7 says it did not do. Write "a ranking by severity band, using blast radius and HITS authority".

## C6. The small-world reading confuses undirected hops with exposure: a typical component is depended on, transitively, by about 1 % of the graph
- Type: overstated
- Origin: n/a (text)
- Severity: medium
- Thesis: Chapters/06-Network-Science.tex:178 — "A value of \(\sigma\) well above one means that components form tightly clustered neighbourhoods while any two remain a few hops apart: a vulnerability introduced anywhere is structurally close to everything." (Ch2:162 repeats it: "a vulnerability can propagate across the entire graph in relatively few hops")
- Evidence: \(L\) is computed on \(G_u\) (`graph_analysis.py:151-183 @28113e7`). So a "hop" may go down to a dependency and back up to another consumer, which is not a propagation path. A vulnerability reaches only the ancestors (`impact.py:50`). Recomputed over all 90 SBOMs: the per-SBOM median share of the graph that transitively depends on a component is 1.25 % (max 6.7 %), while the median \(L\) is 5.0 (Ch7:79). Humphries & Gurney (2008) also report that their S grows roughly linearly with n (\(S \approx 0.023 n^{0.96}\)), because \(C_{\mathrm{rand}} = p \sim 1/n\). With p ≈ 0.01 here, any C above ≈ 0.02 already gives σ > 2.
- Why it matters: This contradicts the chapter's own caution about undirected measures (:10, :50, :58) and Ch8:47. It also overstates what Ch7:79's "every graph has σ above 2" shows.

## C7. The maximum rich-club coefficient is near-binary on these graphs: φ = 1 in 44 SBOMs, almost always from a single edge between the two highest-degree nodes
- Type: overstated
- Origin: pre-existing
- Severity: medium
- Thesis: Chapters/06-Network-Science.tex:103 — "returns the full curve, its maximum, and the smallest \(k\) at which \(\phi(k)\) exceeds \(0.5\)"; :105 — "A rising curve means that the hubs of the dependency graph form a densely interlinked club"
- Evidence: Recomputed from `richClub` with `_build_graph`: `maxRho = 1.0` in 44 of 90 SBOMs. In 39 of these, the first k with φ = 1 leaves \(N_k = 2\), i.e. the two top-degree nodes (usually the root and one hub) are adjacent. The 0.5 threshold is reached in 54 SBOMs, with \(N_k = 2\) in 21 and \(N_k \le 3\) in 38. Ch7 reports this maximum as an RQ1 statistic (`tab_rq1_profile.tex:24`: median 0.67, Q3 1.00, max 1.00). Colizza et al. 2006 (p. 3 of the arXiv text) note that φ increases monotonically even in ER graphs. They also point out that finite-size effects such as large cliques "set in even in completely random networks".
- Why it matters: The "maximum" and the "0.5 threshold" are dominated by two- or three-node subgraphs, so they describe whether two hubs share an edge, not a club. :105 already discloses the lack of normalisation, but not this degeneracy, and Ch7 still reports the maximum.

## C8. The unmaintained-package check appears in the figure and dominates Ch7's timings, but is never defined; its "last release" is npm's `time.modified`
- Type: omission
- Origin: pre-existing
- Severity: medium
- Thesis: Chapters/06-Network-Science.tex:116 — "showing the whole-graph audit: degree entropy, dependency cycles, version conflicts ..., namespace concentration, and unmaintained packages"; :184 — "\autoref{tab:ns-summary} lists every measure described in this chapter"
- Evidence: `audit.py:189-228 @28113e7` flags a package when its date is more than `UNMAINTAINED_YEARS = 2` years old (`:18`). For npm the date is `data["time"]["modified"]` (`:156`), which is the last change to the registry document and not the last release. Failed lookups are counted in `skippedCount`. No section of Ch6 (or any chapter; `grep -i unmaintain Chapters/*.tex` finds only :115-116 and Ch7:151) defines the check, and the summary table omits it. Ch7:151 says "nearly all of the audit is the unmaintained-package check".
- Why it matters: An examiner who sees "58 unmaintained packages" in the figure has no definition, threshold or data source. A short subsection, or a sentence plus a table row, is needed, and it must state that the npm date is the metadata modification date.

## C9. "Zero for almost every SBOM" is contradicted by the dataset: one third of the graphs have cycles and a non-zero directed spectral radius
- Type: overstated
- Origin: n/a (text)
- Severity: low
- Thesis: Chapters/06-Network-Science.tex:10 — "A dependency graph is acyclic or nearly so ..., and the adjacency matrix of a directed acyclic graph is nilpotent ... A spectral radius computed on \(G\) itself would therefore be zero for almost every SBOM"
- Evidence: The nilpotency statement is correct. But 30 of 90 SBOMs have a non-trivial SCC (Ch7:75; `bowTie.nonTrivialSccCount`). For those 30, `max|eig(nx.to_numpy_array(G))|` ranges from 1.0 to 2.24.
- Why it matters: The argument still holds, because the directed radius is then small and set only by the cycles. But "almost every" is false by the thesis's own numbers. "Zero for every acyclic SBOM, and determined only by the few cycles otherwise" is accurate.

## C10. Spectral exposure is zero for up to a third of the nodes of connected graphs, because of 4-decimal rounding, not disconnection
- Type: omission
- Origin: pre-existing
- Severity: low
- Thesis: Chapters/06-Network-Science.tex:50 — "if \(G_u\) is disconnected, ... every node elsewhere scores zero"; :46 — "The ranking of components by \(s_i\) is exactly their ranking by eigenvector centrality"
- Evidence: All 90 \(G_u\) are connected (`nx.is_connected`, and `lccFraction` = 1.0 everywhere). Yet in 62 SBOMs some nodes have `r0 == 0`, up to 32.6 % of nodes (median 1.2 %). The cause is `round(..., 4)` in `graph_analysis.py:357 @28113e7`. By Perron–Frobenius, the true entries are strictly positive on a connected graph. The response also has a `globalR0` field equal to λ1 (`:364`, comment "β=δ=1 → R0 = λ1"), which :48 does not mention.
- Why it matters: The "exact ranking" holds only before rounding. The API returns ties at zero at the tail of the ranking. The `globalR0` field contradicts :48's "not a basic reproduction number" disclaimer.

## C11. The spectral component scores are often dominated by the application root itself, which the text never says
- Type: omission
- Origin: n/a (text)
- Severity: low
- Thesis: Chapters/06-Network-Science.tex:48 — "In the dependency graph this identifies the densely interlinked region through which a compromise would most readily travel"; :64 (edge rankings)
- Evidence: The metadata component is a node (`sbom_store.py:18-20`). The root is the component with the highest `r0` in 27 of 90 SBOMs. On average 5.9 of the 20 top-ranked optimizer edges are incident to the root. \(\sqrt{\deg(\text{root})}/\lambda_1\) has a median of 0.69, so the root's star alone accounts for most of λ1. The root is also in the innermost k-core in 25 SBOMs.
- Why it matters: The "most exposed component" is often the application, which an analyst cannot remediate. The reading of λ1 as a property of the libraries should mention that the root's direct-dependency star contributes most of it.

## C12. The power-law guard is described as the only reason no fit is returned, but the 3 failed fits in Ch7 passed that guard
- Type: omission
- Origin: pre-existing
- Severity: low
- Thesis: Chapters/06-Network-Science.tex:164 — "When fewer than three components have consumers, or the in-degrees take fewer than two distinct values, no fit is attempted."
- Evidence: `graph_analysis.py:122-134 @28113e7` also returns an empty fit when `powerlaw.Fit` raises or gives a NaN α. The three SBOMs without α (patch-package, transformers.js, karma) have in-degrees {1,2,3} with 67–90 components. So the guard passed and the fit itself failed. Ch7:77 explains them as "the in-degrees take too few distinct values", which is true in spirit but is not the rule :164 states.
- Why it matters: This is a minor gap between the stated rule and Ch7's three exclusions. Add "or the fit does not converge".

## C13. The Java namespace rule does not work on Maven SBOMs, and the namespace is not derived "from its Package URL" outside npm/PyPI
- Type: error
- Origin: pre-existing (still true at HEAD; distinct from the scoped-npm defect, which is not reopened here)
- Severity: low
- Thesis: Chapters/06-Network-Science.tex:158 — "SBOM Lens derives a namespace for each component from its Package URL --- the scope of a scoped npm package, the first dotted segments of a Java-style name, or the leading token of a PyPI name"
- Evidence: For anything other than `pkg:npm/` and `pkg:pypi/`, `audit.py:110-116 @28113e7` splits the component `name` on dots. A Maven component's `name` is the artifactId. For example, `_extract_namespace('pkg:maven/org.springframework/spring-core@5.3.1','spring-core')` returns `None`. The groupId sits in the PURL namespace / `group` field and is never read. A Go module `github.com/foo/bar` becomes its own namespace. HEAD's `audit.py` has the same fallback. No Ch6/Ch7 sentence reports a namespace-concentration number; I checked Ch1, Ch7 and Ch8.
- Why it matters: The thesis claims a Java rule that never fires on Java SBOMs. It does not affect the npm-only evaluation, but the sentence is wrong as a description of the tool.

## C14. Not every endpoint caches, and not every measure uses NetworkX
- Type: error
- Origin: pre-existing
- Severity: low
- Thesis: Chapters/06-Network-Science.tex:8 — "all measures are computed by the backend with NetworkX ... each endpoint caches its result per SBOM until the SBOM is edited or deleted."
- Evidence: `global-analysis` and `impact` (`impact.py:41-117 @28113e7`) and `metrics` have no cache. `impact` recomputes HITS for the whole graph on every call (`:72`). Caches exist for `communities`, `kcore`, `graph-analysis`, `graph-theory`, `epidemic`, `optimizer` (`graph_analysis.py:458-467`) and `audit`. Invalidation happens on PUT/DELETE (`sboms.py:109-110,119-120`). The power-law fit uses `powerlaw`, the eigenpairs use numpy/scipy, and version conflicts and namespaces are plain Python.
- Why it matters: Small factual slips, but they describe the system's behaviour. "Most endpoints" fixes the caching claim.

## C15. Ch2's definitions give directional readings that Ch6 explicitly rejects (k-core, eigenvector centrality, articulation points)
- Type: inconsistency
- Origin: n/a (text)
- Severity: low
- Thesis: Chapters/06-Network-Science.tex:58 — "the core number says nothing about direction; a high value means dense interconnection, not depth in the dependency chain"; :50 — "The score is undirected, so it does not distinguish a component that many others depend on from one that depends on many others"; :26 (articulation ≠ chokepoint)
- Evidence: Ch2:208 — high-coreness components "are depended upon by many others (high in-degree) and themselves depend on many others (high out-degree). A vulnerability in a high-core component has maximum blast radius". Ch2:182 — eigenvector centrality: "being depended upon by important components counts more", while Ch6 computes it on \(G_u\). Ch2:134 — an articulation point's vulnerability "propagates to every component in the severed partition, with no alternative dependency path". That is the dominance reading Ch6:26 separates from articulation. Ch2:180 also says "HITS scores are computed iteratively until convergence" (power iteration), whereas Ch6:32 describes SVD (see C1).
- Why it matters: A reader meets the opposite interpretation two chapters earlier. Ch2 should be aligned with Ch6's caveats.

## C16. Wang et al. model discrete-time probabilities under a mean-field approximation, not "rates" of an exact SIS process
- Type: misattribution
- Origin: n/a (text)
- Severity: low
- Thesis: Chapters/06-Network-Science.tex:124 — "in which each infected node infects each neighbour at rate \(\beta\) and recovers at rate \(\delta\), an infection dies out when \(\beta / \delta < 1/\lambda_1\) \citep{Wang2003}"
- Evidence: The journal version, Chakrabarti et al. 2008 (ACM TISSEC 10(4), "An earlier version of this paper appeared in SRDS 2003"), says: "Our model works with small discrete time-steps ... infected ... with probability β ... cured with probability δ". The threshold (Theorem 1, "β/δ < τ ⇒ infection dies out") is proved for their non-linear dynamical system built on an independence assumption. I could not open the SRDS 2003 PDF itself (the CMU URLs return 404), so this is checked against the extended version.
- Why it matters: This is a minor precision issue. "Probability per time step, in a mean-field model" is faithful to the source.

## C17. Communities are presented as vulnerability containment, but they are computed undirected and propagation runs only to consumers
- Type: overstated
- Origin: n/a (text)
- Severity: low
- Thesis: Chapters/06-Network-Science.tex:85 — "For risk, the partition shows where a vulnerability is likely to stay contained and where it is not. A vulnerable component inside a community with few edges leaving it affects mainly that community"
- Evidence: `graph_analysis.py:26 @28113e7` runs Louvain on `G.to_undirected()`. Who is affected is `nx.ancestors` (`impact.py:50`). Edges leaving a community towards its dependencies carry no exposure, and a single consumer edge into the community exposes every transitive consumer outside it. No measurement in the thesis relates communities to blast radius.
- Why it matters: This is the same direction caveat the chapter applies elsewhere. "Likely" is unsupported; "may" with the undirected caveat would be accurate.

## Checked and holds
- Undirected projection: every `G_u` is `G.to_undirected()` (u–v adjacent if either depends on the other). It is used for articulation points (`impact.py:96-97`), bridges and edge ranking (`graph_analysis.py:394-416`), core number (`:51`), Louvain (`:26`), rich club (`:279-283`), eigenpairs (`:351`), and small world (`:151`). HITS, SCCs/bow-tie, cycles, entropy and power law run on directed `G`. Every row of `tab:ns-summary` matches its endpoint and graph, including bridges in `optimizer` and HITS in both `global-analysis` and `impact`.
- Articulation points/bridges: `@not_implemented_for("directed")` in NetworkX; linear time. Ch6:26's distinction between dominance and articulation is mathematically correct.
- Nilpotency of a DAG's adjacency matrix and symmetric/real spectrum on `G_u`; Perron–Frobenius non-negativity on a connected graph; `abs()` of the eigenvector (`:335,340`).
- Spectral exposure formula \(s_i = \lambda_1 v_{1,i}/\max_j v_{1,j}\) (`:354-358`); dense `eigh` up to 500 nodes, `eigsh(which="LA")` above (`:330-340`); the bipartite ±λ1 argument; API field named `r0`.
- Edge perturbation estimate \(\Delta\lambda_1 \approx 2 v_{1,u} v_{1,v}\) for unit-norm v1 (`:404`). Van Mieghem et al. 2011 give \(\lambda_1(A)-\lambda_1(A_m) \simeq 2\sum (x_1)_u (x_1)_v\) and recommend removing the link with the largest eigenvector product. Top 20 edges, percentage of λ1 (`:405,413`). Abstraction candidates: in-degree ≥ 3 (`FAN_IN_THRESHOLD = 3`), sorted by (score, fan-in), top 20 (`:421-435`).
- Core number: `nx.core_number` cites Batagelj–Zaversnik O(m). Self-loops are dropped at build (`sbom_store.py:31-33`) and Ch4:102 says so. The response holds `coreNumbers`, `maxCore` and `innermostNodes`.
- Louvain: `seed=42`, resolution 1, `weight="weight"` with no edge attributes (Ch4:138), so unit weights. NetworkX shuffles nodes with the seed. No modularity in the response. Communities figure: the "Community (Colors)" overlay exists (`src/layouts/sbom-graph/index.js:865` at HEAD) and the caption is accurate.
- Bow-tie: largest SCC as core, IN = ancestors − core, OUT = descendants − core, the rest lumped as "tendrils"; sets reported only when `len(core) > 1` (`:229-273`); `nonTrivialSccCount`. 30 SBOMs have a non-trivial core, which matches Ch7:75.
- Rich club: \(\phi(k)=2E_k/(N_k(N_k-1))\) over degree > k (matches Colizza 2006 and NetworkX); `normalized=False`; threshold is `v > 0.5`; the "rises naturally" caveat matches Colizza 2006.
- Epidemic threshold \(1/\lambda_1\) (`:363`); star \(\lambda_1=\sqrt n\); path \(\lambda_1 = 2\cos(\pi/(n+1)) < 2\).
- Entropy: out-degree, zero-degree nodes included, log2; Low < 1.5, Medium ≤ 3.0, High above (`audit.py:27-47`); bound by log of the number of distinct values. (All 90 SBOMs are labelled Medium, range 1.68–2.86.)
- Cycles: `nx.simple_cycles` (Johnson with SCC handling in 3.7's docstring), cap 1,000, count/nodes/first ten (`audit.py:50-60`).
- Version conflicts: group/name key, ≥ 2 distinct non-empty versions, key `diamonds`, no edge search (`audit.py:63-95`).
- Namespace: top 10, share of components with a namespace, flag `> 60` (`audit.py:120-140`), unscoped npm excluded. No namespace-concentration number appears in Ch6, Ch7, Ch1 or Ch8.
- Power law: in-degrees > 0, `powerlaw.Fit(discrete=True)`, xmin by minimum KS D. On 5 SBOMs I compared α with an exact discrete-zeta MLE and the values agree to 3 decimals, so it is the exact MLE, not the Clauset approximation. α, σ, xmin, D, nTail, top-5 hubs and histogram are all returned.
- Small world: LCC of `G_u`, `average_clustering`, exact L ≤ 500 nodes, 500 seeded BFS sources otherwise (the sample includes the zero self-distance, a bias of at most 0.2 % at n ≤ 1000), \(C_{\mathrm{rand}}=p\), \(L_{\mathrm{rand}}=\ln n/\ln(np)\), \(p=2m/(n(n-1))\) (`:147-208`). Humphries & Gurney (2008, eq. 10) give \(L_{\mathrm{rand}} \sim \ln n/\ln\langle k\rangle\). C = 0 gives σ = 0.
- Risk Assessment figure: the screenshot shows Entropy, Dependency Cycles, Diamond Dependencies, Namespace Concentration and Unmaintained Packages, as the caption says. The `\screenshot` note text is not printed once the PNG exists (`ULOMain.tex:31-41`).
- Consistency with Ch5: blast radius includes the node (Ch6:4, :184 are compatible); HITS in `impact` holds only the analysed node's two scores (Ch5:233 and Ch6:32 agree). The optimizer is correctly excluded from the evaluation (:64).
