# Chapter 6 — Sorted review

## Summary
- 20 items (C1–C17, D-new1–D-new3). Sorted into **13 errata**, **3 viva prep** and **1 hold**. Some items are merged: E1 covers C1, C2, D-new1 and D-new2; E6 covers C4 and D-new3. The errata, by impact: E1 (C1+C2+D-new1+D-new2), E2 (C5), E3 (C3), E4 (C7), E5 (C6), E6 (C4+D-new3), E7 (C8), E8 (C17), E9 (C15), E10 (C9), E11 (C12), E12 (C13), E13 (C14). Viva prep: C11, C10, plus one new point from my own check (V3, Louvain partitions depend on the NetworkX version). Hold: C16.
- **Results.** Only the RQ2 HITS-authority figures can change. **The RQ2 conclusion does not change under either option.** The median τ_b stays 0.00, and the median top-k overlap stays 0.00. The Abstract, Ch8:14 and Ch8:33 do not change under either option.
  - Option A (keep the stored results). The τ_b row is unchanged. The top-k overlap for authority goes from n = 90 → 79, Q3 0.17 → 0.25, and "zero in 64" → "zero in 54 of 79". Ch7:109 and Ch7:157 get a corrected explanation. Ch7:169 adds the library versions.
  - Option B (HITS by SVD). τ_b for authority goes from n = 79 → 87 and Q1 −0.14 → −0.13. The counts go from 13/14 to 15/15 (≥ 0.2 / ≤ −0.2). The overlap goes to n = 87, Q3 0.25 and "zero in 58 of 87". The figure is regenerated (87 points), and the Ch7:103 caption changes from 79 to 87.
- **Settled disputes.**
  - The critic is right on C1/C2. I reproduced every stored HITS value under NetworkX 3.7 (power iteration). The 8 non-convergent SBOMs converge with the SVD method. NetworkX 3.6.1 gives identical scores on all 90 graphs.
  - Two critic slips. C1 says "seven of Ch7's 11"; the right number is 8. C12 says "67–90 components"; the defender is right with 87/73/125 components with consumers.
  - On C16 the defender is right. The source itself says "rate", says its results "also hold for the continuous case", and contrasts itself with the mean-field approach.
- **Three decisions are needed** (list at the end): how to handle HITS (D1, affects reported results), whether to keep the maximum rich-club coefficient in RQ1 (D2, affects a reported table row), and whether to drop or report the articulation/chokepoint comparison (D3).

## Erratum

### E1 (from C1, C2, D-new1, D-new2) HITS: the text describes NetworkX 3.6.1's SVD, the evaluation ran 3.7's power iteration; 11 unrankable SBOMs have two causes; the authority top-k overlap includes 11 alphabetical rankings — impact: high (touches reported results)
- Settled facts (all recomputed by me from `sbom-lens-eval/data`, with the 28113e7 backend, which is identical to the defender's export, run with `sbom-lens/venv/bin/python`):
  - The evaluation environment had NetworkX 3.7, SciPy 1.18.1, NumPy 2.5.3 and powerlaw 2.0.0. The pipeline starts the backend from `config.SBOM_LENS_VENV` (`pipeline/lens.py:46`). I re-ran every endpoint under 3.7 on the 90 stored SBOMs. Every stored measure is reproduced exactly, apart from list order and the order of the "first ten cycles" sample in one SBOM. So 3.7 is what ran. `data/run.json` does not record the Python libraries, and `backend/requirements.txt` gives only `networkx>=3.6.1` (D-new2 is correct).
  - In 3.7, `nx.hits` defaults to `method="power_iteration"`. It starts from h = 1/N and iterates `a = h @ A; h = A @ a; h /= h.max()` until the L1 change in h is below 1e-8. After `max_iter` iterations it raises `PowerIterationFailedConvergence`. Both scores are normalised to sum one at the end. The SVD code of 3.6.1 survives as `method="svd"`. `impact.py:72,101 @28113e7` pass no `method`.
  - Exactly 8 SBOMs fail to converge in 200 iterations: CopyTranslator, stackedit, CyberChef, lowlighter/metrics, marktext, parse-server, typeorm and wekan. Their relative gap (σ1−σ2)/σ1 is between 6.1e-5 and 4.7e-2. Power iteration converges after 202 to 4,465 iterations, except typeorm, which does not converge within 20,000.
  - In the other 82, the stored authorities equal power iteration, 3.7 SVD and 3.6.1, to the 6 decimals of `impact.py:84`. The only output difference is that SVD writes some zeros as `-0.0`. This changes no comparison or rank.
  - On the 8, 3.6.1 and 3.7-SVD agree on all 3,079 rounded hub/authority values.
  - In 3 more SBOMs every vulnerable component has authority 0, so they cannot be ranked under either method. These are MagicMirror, ChatALL and element-web (C2 is correct). So Ch7:109's single cause is wrong for 3 of the 11.
  - D-new1 is correct. `analyse.py:114` computes the overlap whenever 0 < k < |vuln|, and ties are broken by node ID. So the 11 constant-authority SBOMs contribute alphabetical overlaps: ten give 0.0 and stackedit gives 0.167. They are inside the n = 90 row of `tab_rq2_agreement.tex:14` and the "64" of Ch7:107. Ch7:157 ("rest on the remaining 79") is therefore false for the overlap.
  - Under 3.6.1 nothing else reported in Ch7 changes. Cycle counts, cycle nodes, the RQ1 measures, blast radii and dominated sets are identical. The Louvain partitions differ in 67 of 90 SBOMs (community count in 29), but no community number is reported in Ch7 (see V3).
  - Recomputed RQ2 authority figures:

    | Variant | τ_b n | median | Q1 | Q3 | min | max | ≥0.2 | ≤−0.2 | overlap n | median | Q1 | Q3 | min | max | overlap = 0 |
    |---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
    | Stored (as printed) | 79 | 0.00 | −0.14 | 0.14 | −0.50 | 0.46 | 13 | 14 | 90 | 0.00 | 0.00 | 0.17 | 0.00 | 1.00 | 64 |
    | A: stored + D-new1 fix | 79 | 0.00 | −0.14 | 0.14 | −0.50 | 0.46 | 13 | 14 | 79 | 0.00 | 0.00 | 0.25 | 0.00 | 1.00 | 54 |
    | B: SVD + D-new1 fix | 87 | 0.00 | −0.13 | 0.14 | −0.50 | 0.46 | 15 | 15 | 87 | 0.00 | 0.00 | 0.25 | 0.00 | 1.00 | 58 |

    (Median τ_b is −0.004 in all three variants, so it prints as 0.00. |τ_b| < 0.2: 52 → 57; this count is not printed in the thesis. Without the D-new1 fix, B would give overlap n = 90 and 61 zeros. Blast-radius rows are unaffected in every variant.)
- Fix: **DECISION NEEDED** (D1). Both options include the D-new1 change to the evaluation analysis. This is a change to `sbom-lens-eval`, not to SBOM Lens, so it does not touch the 28113e7 rule:
  - `sbom-lens-eval/analysis/analyse.py:114` — `if k and k < len(vuln):` → `if k and k < len(vuln) and len(set(values)) > 1:`. Then regenerate the tables and figures and copy `tab_rq2_agreement.tex` and `fig_rq2_tau.pdf` into the thesis. Do not edit the table by hand.

  **Option A — keep the stored results (NetworkX 3.7, power iteration).** Every change:
  - Ch6:32 — "NetworkX obtains the authority vector as the leading singular vector of the adjacency matrix \(\mathbf{A}\) --- equivalently, the principal eigenvector of \(\mathbf{A}^{\top}\mathbf{A}\) --- and the hub vector as \(\mathbf{h} = \mathbf{A}\mathbf{a}\), each normalised to sum to one over the graph." → "In the NetworkX version used for the evaluation (3.7), this runs power iteration: starting from uniform hub scores, it alternates \(\mathbf{a} \leftarrow \mathbf{A}^{\top}\mathbf{h}\) and \(\mathbf{h} \leftarrow \mathbf{A}\mathbf{a}\), rescaling \(\mathbf{h}\) at each step, until \(\mathbf{h}\) changes by less than \(10^{-8}\) in total. The limit is the leading singular-vector pair of the adjacency matrix \(\mathbf{A}\) --- the authority vector is the principal eigenvector of \(\mathbf{A}^{\top}\mathbf{A}\) --- and both vectors are normalised to sum to one over the graph. Earlier NetworkX versions computed the same vectors directly, by a sparse singular value decomposition."
  - Ch6:32 — "If the computation fails to converge within the iteration limit, all scores are reported as zero." → "Power iteration converges slowly when the two largest singular values of \(\mathbf{A}\) are close; if it has not converged after 200 iterations, SBOM Lens reports every score as zero."
  - Ch7:109 — "For HITS authority, 11 SBOMs could not be ranked: in these graphs the HITS computation did not converge within the iteration limit set in SBOM Lens, which then reports every score as zero (\autoref{sec:hits})." → "For HITS authority, 11 SBOMs could not be ranked. In 8, power iteration had not converged after 200 iterations, because the two largest singular values of their adjacency matrices are close, and SBOM Lens then reports every score as zero (\autoref{sec:hits}). In the other 3, HITS converged, but every vulnerable component has authority zero, because authority is concentrated on a part of the graph that contains none of them."
  - Ch7:107 — "the top-\(k\) overlap is zero in 66 SBOMs for blast radius and in 64 for authority." → "the top-\(k\) overlap is zero in 66 of the 90 SBOMs for blast radius and in 54 of the 79 rankable SBOMs for authority."
  - Ch7:157 — "HITS authority could not be computed for 11 of the 90 graphs; the RQ2 results for authority rest on the remaining 79." → "HITS authority could not rank the vulnerable components of 11 of the 90 graphs (\autoref{sec:eval-rq2}); the RQ2 results for authority, including the top-\(k\) overlap, rest on the remaining 79." Optional sensitivity sentence: "In 8 of the 11, the cause is NetworkX 3.7's power iteration; NetworkX 3.6.1, which computes HITS by singular value decomposition, converges on all 8, and with its scores the median \(\tau_b\) for authority remains 0.00 over 87 SBOMs."
  - Ch7:169 — "Each run records the versions of the tools it used: SBOM Lens at commit \texttt{dfc8e23}, cdxgen~12.8.5 and Node.js~20.20.2." → "Each run records the versions of the tools it used: SBOM Lens at commit \texttt{dfc8e23}, cdxgen~12.8.5 and Node.js~20.20.2. The backend ran with Python~3.14.4, NetworkX~3.7, SciPy~1.18.1, NumPy~2.5.3 and powerlaw~2.0.0; the run record does not include these library versions, and the NetworkX version affects the HITS results (\autoref{sec:hits})."
  - `Tables/Chapter7/tab_rq2_agreement.tex:14` (regenerated) — `Top-\(k\) overlap: HITS authority & 90 & 0.00 & 0.00 & 0.17 & 0.00 & 1.00 \\` → `Top-\(k\) overlap: HITS authority & 79 & 0.00 & 0.00 & 0.25 & 0.00 & 1.00 \\`.
  - Unchanged: `tab_rq2_agreement.tex:12` (τ_b row), `fig_rq2_tau.pdf` and its caption at Ch7:103 (79 SBOMs), the rest of Ch7:107 ("median of 0.00 and 13 … against 14"), Ch8:14, Ch8:33, the Abstract, and Ch2:180 ("computed iteratively until convergence", which is now accurate).
  - Eval repo (not thesis): add `pip freeze` of `sbom-lens/venv` to `run.json`, or commit a constraints file.

  **Option B — compute HITS by SVD and report new numbers.** How to produce them:
  - B1 (recommended if B). Install `networkx==3.6.1` in a copy of the backend venv. This is allowed by 28113e7's own `requirements.txt` (`networkx>=3.6.1`), so **the SBOM Lens code stays at 28113e7** and the thesis can still describe that commit. Re-run only the `global-analysis` and `impact` endpoints on the 90 stored SBOMs and their stored vulnerable-node lists. Do not re-run cdxgen or OSV, because the vulnerability data would drift. Splice the new `hits`, `hitsHub` and `hitsAuthority` into `data/results/*.json`, apply the D-new1 fix, and re-run `analysis/analyse.py`. Copy `tab_rq2_agreement.tex` and `fig_rq2_tau.pdf` into the thesis.
    - Timings. Keep the stored timings and say so; this is my recommendation. If the two endpoints are re-timed, the `global analysis` and `impact` rows of `tab_performance.tex`, the total row and `fig_performance.pdf` change by amounts I cannot predict (machine-dependent). My local estimate of HITS time per SBOM: median 0.067 s for power iteration against 0.059 s for SVD.
    - A full re-collection under 3.6.1 is not advisable. It would change the Louvain partitions (not reported), the audit's date-dependent unmaintained check and its timings (which dominate the performance table), and, if OSV is re-queried, the vulnerability data.
  - B2. Patch `impact.py:72,101` to `nx.hits(G, max_iter=200, method="svd")`. This gives the same numbers as B1, but it creates a new commit. **The evaluated system would no longer be 28113e7.** That conflicts with the decided rule that the thesis describes 28113e7, unless the thesis discloses a patched evaluation build.
  - B3. Compute SVD HITS inside `analyse.py`. Not advisable: it breaks the script's stated rule (`analyse.py:3`) that every value is an SBOM Lens output.
  - Text changes under B (B1):
    - Ch6:32 — keep the SVD description and make it version-specific: "NetworkX obtains the authority vector" → "In the NetworkX version used for the evaluation (3.6.1), NetworkX obtains the authority vector". After "If the computation fails to converge within the iteration limit, all scores are reported as zero." add: "This did not occur in the evaluation. NetworkX~3.7 replaced this computation by power iteration, which converges slowly when the two largest singular values of \(\mathbf{A}\) are close; with the same limit of 200 iterations it fails on 8 of the 90 evaluation graphs."
    - Ch7:103 (caption) — "and by HITS authority (79 SBOMs)." → "and by HITS authority (87 SBOMs)."
    - Ch7:107 — "HITS authority gives the same picture, with a median of 0.00 and 13 SBOMs above 0.2 against 14 below \(-0.2\)." → "HITS authority gives the same picture, with a median of 0.00 and 15 SBOMs above 0.2 against 15 below \(-0.2\)."
    - Ch7:107 — "the top-\(k\) overlap is zero in 66 SBOMs for blast radius and in 64 for authority." → "the top-\(k\) overlap is zero in 66 of the 90 SBOMs for blast radius and in 58 of the 87 rankable SBOMs for authority."
    - Ch7:109 — whole first sentence → "For HITS authority, 3 SBOMs could not be ranked: HITS converged, but every vulnerable component has authority zero, because authority is concentrated on a part of the graph that contains none of them (\autoref{sec:hits})."
    - Ch7:157 — "HITS authority could not be computed for 11 of the 90 graphs; the RQ2 results for authority rest on the remaining 79." → "HITS authority could not rank the vulnerable components of 3 of the 90 graphs; the RQ2 results for authority, including the top-\(k\) overlap, rest on the remaining 87."
    - Ch7:169 — after "Node.js~20.20.2." add: "The backend ran with Python~3.14.4, SciPy~1.18.1, NumPy~2.5.3, powerlaw~2.0.0 and NetworkX~3.7, except that the HITS scores of the global-analysis and impact endpoints were recomputed with NetworkX~3.6.1 (\autoref{sec:hits}); every other measure reported in this chapter is identical under the two versions. Timings are those of the original run."
    - `tab_rq2_agreement.tex:12` — `79 & 0.00 & -0.14 & 0.14 & -0.50 & 0.46` → `87 & 0.00 & -0.13 & 0.14 & -0.50 & 0.46`.
    - `tab_rq2_agreement.tex:14` — `90 & 0.00 & 0.00 & 0.17 & 0.00 & 1.00` → `87 & 0.00 & 0.00 & 0.25 & 0.00 & 1.00`.
    - `fig_rq2_tau.pdf` — regenerated. The HITS box now has 87 points. Median and whiskers are almost unchanged (min −0.50, max 0.46).
    - Unchanged: Ch8:14, Ch8:33, the Abstract and the blast-radius rows. Ch2:180 ("computed iteratively") can stay, because the ARPACK SVD is also iterative.
- Also touches: Ch6:32; Ch7:103 (B only), :107, :109, :157, :169; `tab_rq2_agreement.tex:12` (B only) and :14; `fig_rq2_tau.pdf` (B only); `analyse.py:114`; eval `data/results` (B only) and `run.json`. Not the Abstract, not Ch8.
- Viva answer: "The evaluated environment installed NetworkX 3.7, whose HITS default changed from SVD to power iteration. With the 200-iteration limit in SBOM Lens, power iteration fails on 8 graphs whose two leading singular values are nearly equal. Three more graphs converge, but no vulnerable component gets non-zero authority. With the SVD scores, which NetworkX 3.6.1 gives and the declared requirements allow, 87 graphs are rankable and the median τ_b is still 0.00. So RQ2's 'unrelated orderings' holds either way. Every other stored measure is reproduced exactly under 3.7, so the dependency version affected HITS and nothing else reported."

### E2 (from C5) The summary names the wrong RQ2 baseline and too many measures — impact: medium (describes a reported result)
- Settled facts: Ch7:59 ranks by worst severity band against blast radius and HITS authority only. Ch7:157 says numeric-CVSS rankings "were not measured". `analyse.py:107` covers only `("blast", "authority")`. Both parties agree.
- Fix: Ch6:186 — "by comparing rankings based on these measures with a ranking by CVSS score alone." → "by comparing the ranking of each project's vulnerable components by severity band with their rankings by blast radius and by HITS authority."
- Also touches: none.

### E3 (from C3) Authority does not rank all libraries: it is zero outside one co-citation block — impact: medium (bears on how the RQ2 authority result is read)
- Settled facts:
  - Over the 82 SBOMs where HITS converges, the median share of nodes with non-zero authority is 0.461 (min 0.070). In the median SBOM, half the vulnerable components have authority 0 (median share 0.500). Both recomputed by me.
  - The defender's mechanism is sound. AᵀA links components only through a shared direct consumer, and the Perron vector of a reducible matrix sits on its dominant block. Ch3:119 already shows the effect ("all other components: 0").
  - The text claims a ranking "by how central they are to the dependency structure", which overstates what HITS gives.
- Fix: Ch6:34 — "Authority is the more useful of the two for risk: it ranks libraries by how central they are to the dependency structure, weighting a consumer's vote by that consumer's own importance rather than merely counting consumers as in-degree does." → "Authority is the more useful of the two for risk: it weights a consumer's vote by that consumer's own importance rather than merely counting consumers as in-degree does. On dependency graphs, however, it does not order every library. Two components are linked in \(\mathbf{A}^{\top}\mathbf{A}\) only when they share a direct consumer, and the authority vector is concentrated on the most strongly linked group of such components; in the evaluation, about half of the components of the median graph, and half of its vulnerable components, have authority zero. Authority therefore ranks the components of that group and ties the rest at zero."
- Optional: at Ch7:107, after the authority sentence, add "(about half of the vulnerable components of the median SBOM are tied at zero authority)".
- Also touches: Ch7:107 (optional). This is the mechanism behind E1's 3 constant SBOMs.
- Viva answer: "Authority is the second, contrasting structural ordering. The RQ2 conclusion rests equally on blast radius, which is defined everywhere."

### E4 (from C7) The maximum rich-club coefficient is set by two or three nodes — impact: medium (touches a reported RQ1 table row)
- Settled facts: I recomputed with the defender's script and the critic's counts are exact. maxRho = 1 in 44 of 90 SBOMs; in 39 of these the first k with φ = 1 leaves N_k = 2. The 0.5 threshold is reached in 54 SBOMs, with N_k = 2 in 21 and N_k ≤ 3 in 38. With N_k = 2, φ ∈ {0, 1} by construction. Ch7 reports the maximum: `tab_rq1_profile.tex:24` (median 0.67, Q3 1.00, max 1.00) and Ch7:57.
- Fix: Ch6:105 — after "The \(0.5\) cut-off is a presentation threshold without theoretical basis." add: "On SBOM graphs the maximum and the cut-off are dominated by the last points of the curve: in 39 of the 44 evaluation graphs whose maximum is 1, that value comes from the two highest-degree components alone being adjacent. Both therefore say whether the top two or three hubs are linked, rather than whether a club exists."
- Ch7: **DECISION NEEDED** (D2).
  - Option A: keep the RQ1 row and add a caveat at Ch7:57: "the maximum rich-club coefficient" → "the maximum rich-club coefficient (which, on these graphs, mostly reflects whether the two or three highest-degree components are adjacent; \autoref{sec:ns-richclub})".
  - Option B: drop the row from `RQ1_ROWS` in `analyse.py`, regenerate `tab_rq1_profile.tex` (removing line 24), and delete ", the maximum rich-club coefficient" from Ch7:57.
- Also touches: Ch7:57 and `tab_rq1_profile.tex:24` (depending on D2). Ch8:49 (0.5 threshold) stays correct.

### E5 (from C6) The small-world reading treats undirected closeness as exposure — impact: medium (interpretation of RQ1; no number changes)
- Settled facts:
  - L is computed on G_u (`graph_analysis.py:151-183`). Exposure follows `nx.ancestors`.
  - I recomputed the per-SBOM median share of a component's transitive consumers: median 1.24 %, max 6.59 %. The defender's figures are the more exact ones.
  - Median p = 0.0099 and median L/L_rand = 1.01. So in the median graph any C above about 0.021 already gives σ > 2, while the minimum C in the dataset is 0.046. "Every graph has σ above 2" is therefore a weak bound.
  - The defender's partial rebuttal holds: the factual clause about G_u is true. The risk gloss is wrong.
- Fix:
  - Ch6:178 — "a vulnerability introduced anywhere is structurally close to everything." → "in the undirected projection, every component is close to every other. Closeness in \(G_u\) is not exposure: a vulnerability reaches only the transitive consumers of a component, usually a small fraction of the graph."
  - Ch2:162 — "Short average path lengths mean that a vulnerability can propagate across the entire graph in relatively few hops, even in a large SBOM." → "Short average path lengths mean that any two components are linked by a short chain of dependency relations, even in a large SBOM, although a vulnerability propagates along such a chain in one direction only, from a dependency to its consumers."
  - Ch7:79 — after "rather than a tree." add: "The bound itself is weak: with edge densities around \(p \approx 0.01\), the random-graph clustering \(C_{\mathrm{rand}} = p\) is so small that, in the median graph, any clustering above about 0.02 already gives \(\sigma > 2\)."
- Also touches: Ch2:162, Ch7:79. Ch8:12 ("every graph is small-world … about five hops apart") stays true by the σ > 1 definition and can stand.
- Viva answer: "σ describes undirected interlinking, as Ch8:47 says. Exposure is measured by the directed blast radius."

### E6 (from C4, D-new3) "Chapter 7 compares them" is false; and in single-root graphs every articulation point is a chokepoint — impact: medium
- Settled facts:
  - `grep -i articulat` finds nothing in Ch7 or `analyse.py`, so C4 is right.
  - D-new3 is right. Its proof is valid: a component C of G_u − v that does not contain the root can be reached from the root only through v, so v dominates C. Its premises hold in the data: 0 SBOMs have more than one in-degree-0 node, and 0 nodes are unreachable from the root.
  - Recomputed: all 7,790 articulation points are chokepoints. Among vulnerable components there are 1,186 chokepoints and 986 articulation points, and all 986 are chokepoints.
  - So Ch6:26's first example cannot occur in the evaluated graphs. The second example (a dominator need not be a cut vertex) is correct.
- Fix:
  - Ch6:26 — "An articulation point may separate two groups of consumers that have nothing to do with the root, and a component that dominates a subtree need not disconnect the undirected graph if some unrelated edge happens to link that subtree back to the rest." → "When every component is reachable from a single root, as in all the evaluated SBOMs, every articulation point is also a chokepoint, since the part of the graph it cuts off from the root can be reached only through it. The converse fails: a component that dominates a subtree need not disconnect the undirected graph if some unrelated edge happens to link that subtree back to the rest."
  - Ch6:26 — "SBOM Lens reports both and \autoref{cp:evaluation} compares them." → **DECISION NEEDED** (D3).
    - Option A: "SBOM Lens reports both." No new result.
    - Option B: keep "and \autoref{cp:evaluation} compares them on the vulnerable components" and add to Ch7:130, after "a median of 12.": "All 986 vulnerable articulation points are chokepoints, but 200 of the 1{,}186 vulnerable chokepoints are not articulation points, so the undirected measure would miss a sixth of them." This adds a new reported number.
- Also touches: Ch7:130 (B only). Ch8:35 ("distinguishes … chokepoints from articulation points") stays true.

### E7 (from C8) The unmaintained-package check is shown and timed but never defined — impact: medium
- Settled facts:
  - `audit.py:18` sets `UNMAINTAINED_YEARS = 2`. The cut-off is `datetime.now(timezone.utc) - timedelta(days=730)` (:191).
  - For npm the date is `data["time"]["modified"]` (:156), the last change to the registry document. For PyPI it is the latest upload of any release (:177-183).
  - Failed lookups go to `skippedCount`. Only `pkg:npm/` and `pkg:pypi/` are checked.
  - No chapter defines the check. Ch6:116 shows it, and Ch7:151 says it dominates the timings.
- Fix:
  - Ch6, new subsection after :158 (end of Namespace Concentration): "\subsection{Unmaintained Packages}\label{sec:ns-unmaintained} SBOM Lens flags a component as unmaintained when the date it obtains from the package registry is more than two years before the date of the analysis, and returns the flagged components through \texttt{GET /sboms/\{id\}/audit}. For npm packages the date is the registry's \texttt{time.modified} field, the last change to the package's registry record, which is not necessarily its last release; for PyPI packages it is the most recent upload of any release. Components whose lookup fails are counted separately, and components from other ecosystems are not checked. The check is a registry heuristic, not a graph measure: it queries the registry once per package at analysis time, so its result for an older release depends on when the analysis is run, and it says nothing about the version actually installed."
  - `tab:ns-summary`, after Ch6:210 — add `Unmaintained packages & Whole graph & components & \texttt{audit} \\`.
  - Ch6:184 — "version conflicts, concentration and the degree distribution" → "version conflicts, concentration, unmaintained packages and the degree distribution".
  - Ch6:186 — "from which version conflicts and namespace concentration --- but nothing else in the table --- could be derived" → "from which version conflicts, namespace concentration and, with a registry lookup, unmaintained packages --- but nothing else in the table --- could be derived".
- Also touches: Ch7:151 (no change needed; it can now reference `\autoref{sec:ns-unmaintained}`).

### E8 (from C17) Communities presented as containment without the direction caveat — impact: low
- Settled facts: Louvain runs on `G.to_undirected()` (`graph_analysis.py:26`), and exposure follows ancestors. The undirected computation is disclosed at :85, so the defender is right there. The containment reading is still unsupported: no measurement links communities to blast radius.
- Fix: Ch6:85 — "the partition shows where a vulnerability is likely to stay contained and where it is not. A vulnerable component inside a community with few edges leaving it affects mainly that community;" → "the partition suggests where a vulnerability may stay contained. Because a vulnerability reaches only consumers, what matters is how many components outside the community depend on it: a vulnerable component in a community on which few outside components depend affects mainly that community;"
- Also touches: none.

### E9 (from C15) Ch2's definitions give directional readings that Ch6 rejects — impact: low
- Settled facts: the quotes are as the critic gives them (Ch2:134, :182, :208). Ch2:180 ("computed iteratively until convergence") matches the evaluated 3.7 behaviour. Under D1 Option A it stays; under Option B it is still acceptable, because ARPACK is iterative.
- Fix:
  - Ch2:134 — "affecting it propagates to every component in the severed partition, with no alternative dependency path." → "affecting it touches both sides of the split."
  - Ch2:182 — "being depended upon by important components counts more than being depended upon by peripheral ones." → "being linked to important components counts more than being linked to peripheral ones. SBOM Lens computes it on the undirected projection of the dependency graph (\autoref{sec:spectral-exposure}), where depending on a component and being depended on by it count alike."
  - Ch2:208 — "They are depended upon by many others (high in-degree) and themselves depend on many others (high out-degree). A vulnerability in a high-core component has maximum blast radius and maximum replacement difficulty:" → "They are linked, in either direction, to many components that are themselves densely linked. Such a component is hard to replace:"
- Also touches: Ch2 only.

### E10 (from C9) "Zero for almost every SBOM" — 30 of 90 have cycles — impact: low
- Settled facts: 30 SBOMs have a non-trivial SCC. Their directed spectral radius runs from 1.000 to 2.236 (recomputed); the undirected λ1 runs from 4.7 to 20.6.
- Fix: Ch6:10 — "would therefore be zero for almost every SBOM and would carry no information." → "would be zero for every acyclic SBOM and, in the others, would be set only by their few cycles."
- Also touches: none.

### E11 (from C12) The power-law "no fit" rule omits failed fits — impact: low
- Settled facts: `graph_analysis.py:122-134` also returns no fit when `powerlaw.Fit` raises or α is NaN. The three SBOMs without α have in-degrees {1, 2, 3}, so they pass the stated guard. They have **87, 73 and 125** components with consumers (the defender is right; the critic's "67–90" is wrong). Ch7:77's explanation is substantively right.
- Fix: Ch6:164 — "no fit is attempted." → "no fit is attempted; a fit that fails, as it can when the in-degrees take only a few distinct values, also returns no exponent."
- Also touches: none (Ch7:77 stays).

### E12 (from C13) The Java namespace rule never fires on Maven SBOMs — impact: low
- Settled facts: `_extract_namespace('pkg:maven/org.springframework/spring-core@5.3.1','spring-core')` returns `None`, and a Go name returns itself (both checked by me). Only npm and PyPI PURLs are parsed. No namespace number is reported, and this item is distinct from the decided scoped-npm issue.
- Fix: Ch6:158 — "from its Package URL --- the scope of a scoped npm package, the first dotted segments of a Java-style name, or the leading token of a PyPI name ---" → "from its Package URL for npm (the scope of a scoped package) and PyPI (the leading token of the name), and otherwise from the first two dot-separated segments of the component name, which for Maven, whose component name is the artifact identifier, rarely yields a namespace ---"
- Also touches: none.

### E13 (from C14) Not every endpoint caches; not every measure uses NetworkX — impact: low
- Settled facts: caches exist for communities, kcore, graph-analysis, graph-theory, epidemic, optimizer and audit. `impact`, `global-analysis` and `metrics` have none. Eigenpairs use NumPy/SciPy and the fit uses `powerlaw` (Ch6:46 and :164 already say so).
- Fix: Ch6:8 — "Third, all measures are computed by the backend with NetworkX \citep{Hagberg2008} on the graph cached at ingestion, and are exposed through the REST endpoints listed in \autoref{sec:ns-summary}; each endpoint caches its result per SBOM until the SBOM is edited or deleted." → "Third, all measures are computed by the backend on the NetworkX graph \citep{Hagberg2008} cached at ingestion, with NumPy and SciPy for the eigenpairs and the \texttt{powerlaw} package for the degree fit, and are exposed through the REST endpoints listed in \autoref{sec:ns-summary}; the whole-graph endpoints cache their result per SBOM until the SBOM is edited or deleted, while the impact, global-analysis and metrics endpoints recompute it on every call."
- Also touches: none.

## Viva prep

### V1 (from C11) The root often tops the spectral exposure ranking
- Likely question: "Your most exposed component is often the application itself. Is that useful?"
- Answer: Yes, in 27 of 90 graphs the root has the highest score, and the median √deg⁺(root)/λ1 is 0.69 (recomputed). The root's direct-dependency star is real exposure surface. Ch3's log4j table already shows it ("maximum in the graph: 2.77, the application itself"). Library scores are read relative to one another. The defender's partial rebuttal holds: Ch6 makes no false claim, it only omits this. Optional sentence at the end of Ch6:48: "Because the application itself is a node and its direct dependencies form a star, the root often has the highest score (27 of the 90 evaluation graphs); the scores of libraries are best read relative to one another."

### V2 (from C10) Rounding ties the least-exposed components at zero; `globalR0` field
- Likely question: "You say the ranking is exactly eigenvector centrality, but many scores are zero on connected graphs."
- Answer: s_i as defined ranks exactly. The API rounds to 4 decimals (`graph_analysis.py:357`), which ties the bottom of the ranking at 0. This affects 62 SBOMs, a median of 1.2 % of nodes and up to 32.6 %; all 90 G_u are connected. The top of the ranking is unaffected. `globalR0` repeats λ1 under the same historical naming that :48 discloses. Optional text: at Ch6:46 add "(up to the four-decimal rounding of the response, which ties the least-exposed components at zero)". At Ch6:48, "is named \texttt{r0} for historical reasons; it is not" → "is named \texttt{r0}, and a field \texttt{globalR0} repeats \(\lambda_1\), for historical reasons; neither is".

### V3 (new, from my check for E1) Louvain partitions depend on the NetworkX version
- Likely question: "Is the seeded Louvain partition reproducible?"
- Answer: Only for a fixed library version, as Ch6:76 already says ("for a given graph and library version"). Under NetworkX 3.6.1 the partitions differ from the stored 3.7 ones in 67 of 90 SBOMs, and the community count differs in 29. No community figure is reported in Ch7, so nothing changes. It is the reason the library versions belong in the reproducibility record (E1).

## Holds
- **C16 (Wang et al. "rates").** The defender is right, and I checked the primary source (Chakrabarti et al. 2008, TISSEC, "an earlier version … appeared in SRDS 2003"). §3 says "a universal infection rate β for each edge … and a virus death rate δ". It also says "Our model works with small discrete time-steps … only for ease of exposition; the same results also hold for the continuous case". The paper contrasts itself with the "mean-field" approach and relies instead on an explicit independence assumption. Ch6:124's wording matches the source. No change. Optional precision: after "\citep{Wang2003}" add "(under an assumption that the infection states of neighbours are independent)".

## Decisions needed
1. **How to handle HITS (E1; affects reported results).**
   - (A) Keep the stored NetworkX 3.7 results. Rewrite Ch6:32 for power iteration with 200 iterations. Split Ch7:109 into 8 non-convergent and 3 all-zero. Fix the authority overlap (`analyse.py:114`): n 90 → 79, Q3 0.17 → 0.25, "64" → "54 of the 79". Rewrite Ch7:157 and add the library versions at Ch7:169. The τ_b row, the figure and its caption are unchanged.
   - (B) Recompute HITS by SVD. Preferably re-run only `global-analysis` and `impact` under NetworkX 3.6.1, which keeps the code at 28113e7; patching `method="svd"` would leave 28113e7. The τ_b row becomes 87 / 0.00 / −0.13 / 0.14 / −0.50 / 0.46, with 15 ≥ 0.2 and 15 ≤ −0.2. The overlap becomes 87 / Q3 0.25 / "58 of the 87". The figure is regenerated, the caption goes 79 → 87, and Ch7:109/157/169 and Ch6:32 are rewritten. Keep the stored timings or re-time two rows of `tab_performance.tex`.
   - Under both: the RQ2 conclusion, Ch8:14, Ch8:33 and the Abstract are unchanged.
2. **The maximum rich-club coefficient in RQ1 (E4).** (A) Keep `tab_rq1_profile.tex:24` and add a caveat at Ch7:57. (B) Drop the row and the Ch7:57 mention.
3. **Articulation points vs chokepoints (E6).** (A) Delete the promise that Ch7 compares them. (B) Report the comparison in Ch7:130: all 986 vulnerable articulation points are chokepoints, and 200 of the 1,186 vulnerable chokepoints are not articulation points.
