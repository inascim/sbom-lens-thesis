# Chapter 7 — Critic

What I checked and how. I read Chapters/07-Evaluation.tex in full, its four tables and four figures, and the passages of the Abstract (EN and PT), Ch1 (objectives), Ch2 (sec:scale-free), Ch3 (sec:rw-dependency-graphs), Ch5 (sec:dominator-tree), Ch6 and Ch8 that restate or underpin Ch7 numbers. I read the whole evaluation pipeline (`sbom-lens-eval/run.py`, `config.py`, `pipeline/github.py`, `sbom.py`, `lens.py`, `vulns.mjs`) and `analysis/analyse.py`. I re-ran `analyse.py` against the committed data into `/tmp/claude-1000/ch07-critic/analysis/out/` with `sbom-lens/venv/bin/python` (scipy 1.18.1, numpy 2.5.3, matplotlib 3.11.2). All four `.tex` tables are byte-identical to `Tables/Chapter7/*.tex`, and `per_sbom.csv` is identical to the committed copy. The four figures are not byte-identical (PDF metadata), but rendered at 100 dpi they are pixel-identical. I then recomputed every in-text number from `data/` with my own scripts (`/tmp/claude-1000/ch07-critic/{check,topk,dom,leaf,multi}.py`): counts and shares from the result JSONs, power-law fits with `powerlaw` 2.0.0, HITS with NetworkX 3.7. Tool code references marked `@28113e7` come from `git -C /home/inasc/projects/sbom-lens show 28113e7:<path>`. The eval pinned `dfc8e23` (`config.py:18`). `git diff --stat dfc8e23 28113e7` touches no file under `backend/routers/` except a new `analysis.py`; `graph_analysis.py`, `impact.py`, `audit.py` and `src/utils/vulnerabilityAPI.js` are unchanged, so every measure Ch7 reports is the 28113e7 code. The code paths cited below are unchanged at HEAD (`5273e78`) unless stated.

## C1. The maximum exponent "3.00" is the `powerlaw` package's default upper bound, not a measured value; 8 of the 87 fits sit on it
- Type: error
- Origin: pre-existing (same fit call at HEAD)
- Severity: high
- Thesis: Chapters/07-Evaluation.tex:77 — "all lie between 2.09 and 3.00, inside the range typical of scale-free networks"; Tables/Chapter7/tab_rq1_profile.tex:17 — "Power-law exponent \(\alpha\) & 87 & 2.62 & 2.50 & 2.84 & 2.09 & 3.00"; Chapters/08-Conclusion.tex:12 — "with fitted exponents between 2.1 and 3.0"; Figures/Chapter7/fig_rq1_distributions.pdf (middle panel)
- Evidence: `graph_analysis.py:128 @28113e7` calls `powerlaw.Fit(degrees, discrete=True, verbose=False)` without `parameter_ranges`. In `powerlaw` 2.0.0 (`venv/lib/python3.14/site-packages/powerlaw/distributions.py`, class `Power_Law`), `DEFAULT_PARAMETER_RANGES = {'alpha': [0, 3]}`. The stored α equals 3.000 in 8 SBOMs: fabric.js, chat-ui, ignite, readme-md-generator, color-thief, AnotherRedisDesktopManager, johnny-five and typeorm. Re-fitting gives `2.999999999999999`, which is the bound. With the same x_min and the range widened to [1, 10], their MLEs are 3.03 to 3.39 (fabric.js 3.147, chat-ui 3.033, ignite 3.139, readme-md-generator 3.247, color-thief 3.067, AnotherRedisDesktopManager 3.386, johnny-five 3.062, typeorm 3.029). If x_min is also re-selected freely under the wider range, 17 of 87 exceed 3, up to 6.72. I first reproduced all 87 stored (α, x_min) pairs exactly from the SBOMs, so this is the evaluated fit. The pile-up is visible in the figure: the tallest bar of the α histogram is the last bin, at 3.0.
- Why it matters: "inside the range typical of scale-free networks" (2 to 3, Ch2:150) holds on the upper side only because the library will not return a value above 3. That is circular. The maximum and the upper part of the distribution (Q3 2.84) are artefacts of the bound, and Ch8 repeats the range.

## C2. x_min is not "1 for almost every graph": it is 1 in 66 of 87, and in the other 21 the fit covers less than half of the components with a consumer
- Type: overstated
- Origin: pre-existing
- Severity: medium
- Thesis: Chapters/07-Evaluation.tex:77 — "The lower bound \(x_{\min}\) is 1 for almost every graph, so the fit covers nearly all components with at least one consumer"
- Evidence: stored `scaleFree.xmin` over the 87 fits: 1 in 66, 2 in 17, 3 in 4 (the table's median and Q3 are 1, so the table does not show this). In all 21 graphs with x_min > 1, `nTail` / (components with in-degree > 0) is below 0.5. The minimum is 9.6% (AnotherRedisDesktopManager: 8 of 83 components, x_min 2, and also α at the bound of C1).
- Why it matters: for a quarter of the graphs, the exponent describes a small tail and not "nearly all components". The sentence is used to give the fits credibility.

## C3. Chokepoints are read as hard to remediate, but a vulnerable chokepoint's dominated set is only its own private dependencies
- Type: unsupported
- Origin: pre-existing (Ch5:240 has the same "removed or replaced" framing; Ch7 adds the remediation inference)
- Severity: medium
- Thesis: Chapters/07-Evaluation.tex:130 — "parts of the application are reachable only through them, so they could not be removed or replaced without also losing those parts"; :130 — "These are the components for which remediation is hardest --- the ones that cannot simply be dropped"; :134 — "nearly half of them are structurally irreplaceable"
- Evidence: dominance runs from the root over consumer→dependency edges (`impact.py:11-41 @28113e7`), so every node v dominates is a transitive dependency of v. I checked this on the data: for all 1,186 vulnerable chokepoints, `dominatorTree` ⊆ `nx.descendants(G, v)`. The dominated sets are small: median size 3 (Q1 1, Q3 5), and 27% contain exactly one package. 75.8% of the vulnerable components that have any dependency (1,186 of 1,565) are chokepoints. So "chokepoint" here mostly means "has at least one dependency that nobody else uses". Removing v orphans only packages that exist to serve v. Replacing v with a patched version (the usual remediation) loses nothing. No measurement in Ch7 relates chokepoint status to remediation effort.
- Why it matters: "remediation is hardest" and "structurally irreplaceable" are conclusions about effort that the data does not test. The structural fact points the other way: dropping v also drops its private subtree. The 45% figure is accurate, but its reading is not.

## C4. The RQ3 summary says a fifth of each application "depends on" a vulnerable component; the dependents-only figure is 12%
- Type: inconsistency
- Origin: pre-existing
- Severity: medium
- Thesis: Chapters/07-Evaluation.tex:134 — "and roughly a fifth of each application depends on at least one of them"; cf. :132 — "counting only components that depend on at least one vulnerable component gives a median of 12\% (4\% to 29\%)"
- Evidence: recomputed. Reach (blast radii including the vulnerable nodes, minus the root) has median 0.186, range 0.095–0.349. The union of blast radii without each node itself has median 0.118, range 0.040–0.288, which matches :132. The vulnerable components themselves are median 7.9% of non-root nodes. So the share that *depends on* a vulnerable component is about an eighth, not a fifth. "Each application" also hides a range of 9% to 35%.
- Why it matters: the author kept 19% as reach and added the 12% dependents-only figure precisely to avoid this reading. The summary sentence two lines later restores it.

## C5. "Every number in this chapter is produced by one analysis script" is false; the funnel table and most in-text counts are not output by `analyse.py`
- Type: error
- Origin: pre-existing
- Severity: medium
- Thesis: Chapters/07-Evaluation.tex:4 — "Every number in this chapter is produced by one analysis script from the published dataset"; :169 — "Rerunning the analysis script on the published data regenerates every table and figure of this chapter"
- Evidence: `analyse.py:263-298` writes `per_sbom.csv`, four tables and four figures, and prints the table rows plus five extras (87 fits, 30 bow-tie cores, 90 with a chokepoint, 0 undefined τ, the τ<0 and |τ|<0.2 counts). It does not produce `tab:eval-funnel` (:26-49, typed by hand from `data/skipped.csv`). It also does not produce these in-text numbers: 10,968 / 19,059 stars; 50/40 languages; 53/36/1 lockfiles; 51/22 release years and 2017; 2,668; 8,868 and the four band counts; 125 and 8,743; 17 and 9 (:107); 13 and 14; 66 and 54; 83 and 7 (:128); 986 / 1,186 / 200; 8% and 12% (4%–29%); 99.9%, 23, four, 75% (:159); "almost every" x_min. I recomputed all of them and they are correct (except as noted in C2 and C4), but they come from ad hoc computation.
- Why it matters: the chapter states a reproducibility guarantee it does not meet. A reader who reruns the script cannot regenerate the funnel table or about thirty reported figures.

## C6. The top-k overlap is presented as "starker" evidence, but zero overlap is what an unrelated ranking gives at these k/N
- Type: overstated
- Origin: pre-existing
- Severity: medium
- Thesis: Chapters/07-Evaluation.tex:107 — "The practical measure is starker still. In the median SBOM, none of the \textsc{critical} components is among the equally many components with the largest blast radius ... the top-\(k\) overlap is zero in 66 of the 90 SBOMs"; Chapters/08-Conclusion.tex:14 — "in most applications none of the critical components is among the equally many components with the largest blast radius"
- Evidence: under a random structural ranking, the overlap of k critical components with a random top-k among N is hypergeometric. Its expected value is k/N, with median 0.117 and mean 0.134 over the 90 SBOMs (median k = 3, median N = 25). P(overlap = 0) = C(N−k, k)/C(N, k), which summed over the SBOMs gives an expected 57.0 zeros out of 90. The observed numbers are 66 zeros and a mean of 0.086, so slightly below chance. That fits τ_b ≈ 0 with a slight negative lean. It adds nothing new beyond it. No chance baseline is stated.
- Why it matters: without the baseline, "starker still" reads as a second, stronger finding. With k ≈ 3 of 25, a zero overlap is the expected outcome even for an unrelated ordering. The overlap describes the practical consequence of τ_b ≈ 0, but it is not extra evidence.

## C7. Ties at the k-th place are broken alphabetically by node ID in 33 of 90 SBOMs (blast radius), and the reported overlap counts depend on it
- Type: omission
- Origin: pre-existing (analysis script)
- Severity: medium
- Thesis: Chapters/07-Evaluation.tex:59 — "the share of those \(k\) components that are also among the \(k\) components ranked highest by the structural measure"; :107 — "zero in 66 of the 90 SBOMs for blast radius and in 54 of the 79 rankable SBOMs for authority"; Tables/Chapter7/tab_rq2_agreement.tex:13-14
- Evidence: `analyse.py:116` sorts by `(-v[measure], v["node"])`, so ties at the cut-off are resolved by bom-ref. Blast-radius sizes are small integers. In 33 of 90 SBOMs the k-th value is tied with components outside the top k; for authority this happens in 24 of 79. Resolving those ties in favour of, or against, the CRITICAL components gives 61–67 zeros for blast radius (reported 66), with Q3 0.12 → 0.25, and 47–60 zeros for authority (reported 54), with Q3 0.25 → 0.33. Medians stay 0.00. The chapter does not mention ties in the top-k definition. Ch6's sorted review fixed only the constant-measure case.
- Why it matters: two reported counts and the Q3 column depend on alphabetical order. A tie-aware definition (expected overlap under random tie-breaking, or a reported range) would remove the artefact.

## C8. The spectral analyses were not slowest "on the largest graphs", and the audit's time does depend on the graph
- Type: error
- Origin: pre-existing
- Severity: medium
- Thesis: Chapters/07-Evaluation.tex:151 — "its time depends on the network rather than on the graph"; "Every other analysis had a median between 2 and 150 milliseconds"; "the slowest were the two spectral analyses, which took at most 2.9 seconds on the largest graphs"
- Evidence: from `analysis/out/per_sbom.csv`. The three slowest `epidemic` runs (2.01, 2.13, 2.91 s) are on graphs of 469, 361 and 393 nodes. The three slowest `optimizer` runs (1.62, 1.88, 2.06 s) are on 438, 361 and 469 nodes. The largest graph has 995 nodes. Spearman ρ between node count and time is 0.14 (epidemic) and 0.08 (optimizer). Graphs of ≤500 nodes take the dense `np.linalg.eigh` path (`graph_analysis.py:330-340 @28113e7`), so these peaks are not a size effect. The audit time, by contrast, has ρ = 0.96 with node count, because it issues one registry query per npm node (`audit.py:195-210`). The impact median is 0.151 s, i.e. 151 ms, not "between 2 and 150".
- Why it matters: three factual statements about the timing results are wrong. The real pattern is that audit time scales with the number of packages and spectral time does not track size. Neither undermines "interactive", but both are misreported.

## C9. The abstract says SBOM Lens is shown to be "effective" for prioritising, which Ch7 explicitly says the evaluation does not show
- Type: inconsistency
- Origin: pre-existing
- Severity: medium
- Thesis: Chapters/00-Abstract.tex:21 — "structural and severity rankings of vulnerable components proved unrelated ... SBOM Lens demonstrates that graph theory and network science provide an effective foundation for contextualising and prioritising security risk"; :8 — "revelaram-se independentes ... constituem uma abordagem eficaz para priorizar riscos"; vs Chapters/07-Evaluation.tex:111 — "A \(\tau_b\) near zero does not show that the structural ranking is better than the severity ranking, or the reverse"; :63 — "makes no claim of statistical significance"
- Evidence: Ch7 measures the agreement between orderings (τ_b, overlap) and exposure shares. It has no outcome measure of prioritisation quality: no ground truth, no analyst study, no exploited-vulnerability data. "Proved" and "independentes" (statistical independence) are stronger than a descriptive median τ_b of 0.00 with IQR −0.17 to 0.11.
- Why it matters: the abstract is the most-read claim of the thesis. It asserts an effectiveness result that Ch7, read carefully, disclaims.

## C10. Ch7:77 and Ch8:12 call the distributions heavy-tailed on the strength of a fit; a likelihood-ratio check gives mixed support
- Type: overstated
- Origin: pre-existing
- Severity: medium
- Thesis: Chapters/07-Evaluation.tex:77 — "this is a description of the tail rather than a test against alternative distributions, but it confirms that a few components in each application are depended upon by many others"; Chapters/08-Conclusion.tex:12 — "In 87 of the 90 graphs the in-degree distribution is heavy-tailed"
- Evidence: "a power law could be fitted" (Ch7) becomes "is heavy-tailed" (Ch8). The 3 graphs without a fit are a fitting failure (Ch6:170), not evidence of a light tail. I ran `Fit.distribution_compare` (normalised log-likelihood ratio, α range widened as in C1) on the 87. Against an exponential, the power law is significantly better (p < 0.1) in 50, and the test is inconclusive in 37. Against a log-normal, the power law is never significantly better, and the log-normal is significantly better in 15. The fits and the KS D do not "confirm" anything about the tail. Ch7 itself, two clauses earlier, says this is not a test.
- Why it matters: "confirms" contradicts the caveat in the same sentence. Ch8's "heavy-tailed in 87 of 90" counts successful fits, not tails. A milder wording ("consistent with a heavy tail; in 50 of 87 the tail is significantly heavier than exponential") would be supportable.

## C11. The lockfile check measures only one direction; production packages missing from the graph, and declared dependencies cdxgen never linked, go unreported
- Type: omission
- Origin: pre-existing
- Severity: medium
- Thesis: Chapters/07-Evaluation.tex:159 — "in the median graph, 99.9\% of the nodes are marked as production in the lockfile, and 23 graphs match it exactly, but four graphs fall below 90\% agreement, the lowest at 75\%"
- Evidence: `sbom.py:118-135` records both `onlyInGraph` and `onlyInLockfile`. The 99.9%/75% figures use only `onlyInGraph` (precision). Recall is not reported. In stackedit, 67 of 331 lockfile-production packages (20%) are missing from the graph; wekan misses 53 of 366, promptfoo 25 of 324, MusicFree 25 of 853. Separately, `declaredMatching.unmatchedDeclared` shows 7 SBOMs whose cdxgen root does not link 14 declared production dependencies: teambit/bit 5, lx-music-mobile 4, and yapi, MusicFree, AriaNg, stf and wekan one each. The subtrees of those dependencies are absent from these graphs, including any vulnerabilities in them.
- Why it matters: the threat paragraph gives the impression that the generator only adds stray packages. It also drops production packages in several graphs, which biases size, reach and vulnerability counts downward for those projects.

## C12. The candidate pool is not "the" repositories with at least 100 stars; it is capped at about 10,000 per language
- Type: error
- Origin: pre-existing
- Severity: low
- Thesis: Chapters/07-Evaluation.tex:12 — "The candidates are JavaScript and TypeScript repositories on GitHub with at least 100 stars ... --- 21{,}915 repositories in total"
- Evidence: `config.py:13` `MAX_CANDIDATES_PER_LANGUAGE = 10000`; `github.py:55`. `data/candidates.json` holds 10,960 TypeScript and 10,955 JavaScript repositories, and the least-starred has 466 stars (ben-rogerson/twin.examples). There are far more than 21,915 JS/TS repositories with ≥100 stars. Only the first 1,281 were processed (all ≥10,968 stars), so no result changes.
- Why it matters: it is a factual misstatement of the sampling frame. The README states the cap correctly.

## C13. The funnel and filter descriptions omit cases the code applies
- Type: error
- Origin: pre-existing
- Severity: low
- Thesis: Chapters/07-Evaluation.tex:40 — "SBOM generation & 24 & Root without dependency edges in the generated SBOM"; :16 — "a committed lockfile (\texttt{package-lock.json}, \texttt{yarn.lock} or \texttt{pnpm-lock.yaml}), it is not a workspaces monorepo"
- Evidence: `data/skipped.csv`, stage `cdxgen`: 23 "root has no dependency edges" and 1 "cdxgen failed: CalledProcessError". `config.py:36` also accepts `npm-shrinkwrap.json` (no accepted project uses it). `run.py:82` rejects as "monorepo" any repository with a `workspaces` field *or* a root `pnpm-workspace.yaml` or `lerna.json`. All other funnel counts match `skipped.csv` exactly.
- Why it matters: these are minor, but the caption says the table gives the reasons, and one of the 24 is a generator crash.

## C14. "A leaf package on which nothing else depends" does not occur in the data
- Type: error
- Origin: pre-existing
- Severity: low
- Thesis: Chapters/07-Evaluation.tex:111 — "A \textsc{critical} vulnerability is as likely to be in a leaf package on which nothing else depends as in a component through which much of the application is reached"; Chapters/08-Conclusion.tex:14 — "as likely to affect a leaf package"
- Evidence: every non-root node of a production graph is reachable from the root (`sbom.py:76-85`), so it has at least one dependent. The minimum blast radius excluding the node is 1 over all 2,668 vulnerable components. A "leaf" (no dependencies) is a different thing: 42.7% of CRITICAL and 41.1% of other vulnerable components have none. The pooled comparison does support "as likely": the median blast-radius share is 0.78% for the 337 CRITICAL components and 0.83% for the 2,331 others.
- Why it matters: the image conflates "has no dependencies" with "has no dependents". The second case never occurs, and the first is irrelevant to blast radius.

## C15. "Negligible" for |τ_b| < 0.2 is presented as a convention without a source, and τ_b itself is uncited
- Type: unsupported
- Origin: pre-existing
- Severity: low
- Thesis: Chapters/07-Evaluation.tex:107 — "64 of the 90 have \(|\tau_b| < 0.2\), a level conventionally read as negligible association"; :59 — "\(\tau_b = 1\) means the structural ranking orders the components exactly as the severity bands do"
- Evidence: `Bibliography.bib` has no Kendall entry and no source for an effect-size scale. Common scales put "negligible" below 0.1. Under that cut-off, 38 of 90 qualify, not 64. Also, τ_b = 1 is unattainable when the two variables have different tie structures. Holding the actual band ties and blast-radius values fixed, the maximum attainable τ_b has median 0.79 (min 0.47) across SBOMs.
- Why it matters: the qualitative label carries the RQ2 reading, so it needs a citation or a plainer wording ("small"). The ceiling below 1 does not change the conclusion, but :59's gloss is not exact.

## C16. "Five severity bands" vs the four used
- Type: inconsistency
- Origin: pre-existing
- Severity: low
- Thesis: Chapters/07-Evaluation.tex:59 — "which accounts for the many ties that five severity bands produce"; same line — "(\textsc{critical} \(>\) \textsc{high} \(>\) \textsc{medium} \(>\) \textsc{low})"; :51 — "No finding fell in the unknown band"
- Evidence: the ranking order lists four bands, and the data contains four (`check.py`: 462/4,081/3,622/703, no UNKNOWN). `analyse.py:27` would rank UNKNOWN as 0, i.e. below LOW. ADR 0004 says UNKNOWN "is excluded from the CVSS rankings in RQ2". The case does not arise.
- Why it matters: minor internal inconsistency. Say four bands in the data, or state how UNKNOWN would be treated.

## C17. The timing method is under-specified
- Type: omission
- Origin: pre-existing
- Severity: low
- Thesis: Chapters/07-Evaluation.tex:140 — "measured by the collection pipeline on a single workstation"; :151 — "nearly all of the audit is the unmaintained-package check"
- Evidence: `lens.py:82-88`. Each time is one wall-clock HTTP round trip from the pipeline to a local uvicorn backend, measured once per SBOM with no repetition. It includes JSON serialisation. The impact row is a sum of per-node calls, each of which also recomputes HITS on the whole graph (`impact.py:71-75 @28113e7`). No CPU or memory is given anywhere in the thesis. The share of the audit spent in the unmaintained check was not measured; it is inferred.
- Why it matters: "interactive" and the millisecond figures cannot be put in context or reproduced without the machine and method.

## C18. Smaller cross-chapter mismatches
- Type: inconsistency
- Origin: pre-existing
- Severity: low
- Thesis: Chapters/00-Abstract.tex:8 — "Em 90 aplicações reais" (EN :21 says "90 real npm applications"); Chapters/08-Conclusion.tex:51 — "other ecosystems, where duplicate versions are rarer and dependency trees shallower"
- Evidence: the PT abstract drops "npm", which is the main scope restriction. Ch7:161 supports "duplicate versions are rarer" (Maven) but says nothing about tree depth in other ecosystems, and no source is cited for "shallower". Ch8:51 also omits two filters that Ch7:161 lists (popularity, no monorepos).
- Why it matters: the two abstracts should claim the same scope. Ch8 should not add an unsupported comparison.

## C19. The chapter names only dfc8e23; nothing ties it to the system the thesis describes
- Type: omission
- Origin: changed since 28113e7 (the described system is a descendant of the evaluated one)
- Severity: low
- Thesis: Chapters/07-Evaluation.tex:169 — "SBOM Lens at commit \texttt{dfc8e23}"; :22 — "uploaded to the SBOM Lens backend at a fixed commit"
- Evidence: `git diff --stat dfc8e23 28113e7` changes the frontend (UploadBatchDialog, VulnerabilityPanel, Graph Explorer), adds `backend/routers/analysis.py`, and changes the `MAX_SBOMS` default and `/health`. No analysis router used by the evaluation, and not `vulnerabilityAPI.js`, changed. 28113e7 appears nowhere in the thesis.
- Why it matters: a reader who compares Ch4/Ch5 with the evaluated commit will find features that dfc8e23 lacks. One sentence ("the analysis code is identical in 28113e7, the version described in Chapters 4–6") closes this.

## Checked and found correct
- All four Ch7 tables equal a fresh run of `analyse.py`; `per_sbom.csv` is identical; all four figures render pixel-identical.
- Funnel: 490 / 59 / 135 / 232 / 103 / 24 / 115 (102 + 13) / 2 / 30 / 1 rejected and 90 accepted = 1,281 processed, matching `skipped.csv` (1,191 rows) and `accepted.csv` (90 rows). The order of the table matches `run.py:64-121`. All 103 "production" rejections are pre-cdxgen (`no production dependencies declared`). The error is an HTTP 504 after GitHub retries (`github.py:38-42`).
- 21,915 candidates; sorted most-starred first and cached (`run.py:160-165`). Release: latest non-draft, non-prerelease release with `published_at` ≤ 2024-03-31T23:59:59Z (`github.py:84-102`). cdxgen 12.8.5 with `--no-recurse`; production graph = reachability via `dependencies` + `optionalDependencies` (`sbom.py:50-92`); size 50–1,000 nodes, root included; CRITICAL filter and the one retry on empty OSV results (`run.py:112-121`).
- OSV queries: all 90 between 2026-09-30T21:47Z and 23:27Z, so "30 September 2026" is right, and it is at least 2.5 years after the cutoff. run.json: Node v20.20.2, cdxgen 12.8.5, Python 3.14.4, dfc8e23. Venv: NetworkX 3.7, SciPy 1.18.1, NumPy 2.5.3, powerlaw 2.0.0.
- Accepted set: min stars 10,968, median 19,059; TypeScript 50, JavaScript 40; package-lock 53, yarn 36, pnpm 1; release years 2024: 51, 2023: 22, oldest 2017.
- 2,668 vulnerable components (= unique purls per SBOM, summed); 8,868 findings: 462 CRITICAL, 4,081 HIGH, 3,622 MEDIUM, 703 LOW, no UNKNOWN; 125 duplicate (CVE, purl) pairs, giving 8,743.
- RQ1: median 332 nodes / 534 edges / depth 8; direct share 0.11; 30 graphs with cycles = 30 with a non-trivial SCC = 30 with a bow-tie core; version conflicts median 24, max 137 (the audit `diamonds` = same name with ≥2 versions, `audit.py:63-95`). "Frequently patched in several places" holds: 76 of 90 SBOMs, and 803 of 2,668 vulnerable components, have another vulnerable version of the same package. 87 fits; the 3 unfit graphs have only in-degrees {1,2,3}; KS median 0.024. σ min 2.18, median 16.75; C 0.171; L 5.01. Median p = 0.0099, and the C threshold for σ > 2 has median 0.021, so "about 0.02" is right. λ1 4.68–20.55, threshold 0.049–0.214.
- RQ2: median 25 vulnerable, median 3 CRITICAL (min 1); τ_b blast median 0.003, |τ_b| < 0.2 in 64, ≤ −0.2 in 17, ≥ 0.2 in 9; authority n = 79, > 0.2 in 13 (also ≥ 0.2), < −0.2 in 14; overlap zero 66/90 and 54/79 (as computed by the script; see C7). The 3 converged-but-constant SBOMs (MagicMirror, ChatALL, element-web): under NetworkX 3.7 their vulnerable authorities are ≤ 4e-10, residue of the 1e-8 tolerance, and round to 0 at `impact.py:84`, which is consistent with "authority zero".
- RQ3: transitive share median 0.75, ≥ 0.5 in 83, = 1 in 7 (pooled 1,997/2,668 = 74.9%, consistent with the abstract's "three-quarters"); every SBOM has ≥ 1 vulnerable chokepoint, median 12; 1,186 vulnerable chokepoints, 986 vulnerable articulation points, all 986 among the chokepoints, 200 not (200/1,186 = 0.169, "a sixth"); dominated share median 0.214, max 0.940; reach median 0.19 (0.09–0.35); vulnerable share median 8%; dependents-only median 12% (4%–29%).
- Performance: total median 5.28 s, max 17.18 s; audit is a median 91% of the total; non-audit maximum 2.91 s (epidemic).
- Threats: lockfile agreement over 53 npm-lockfile projects, median 99.87%, 23 exact in both directions, 4 below 90% (screenity 74.5%, github/docs, Next-js-Boilerplate, cz-cli), 37 yarn/pnpm unchecked; τ_b spread with as few as 3 vulnerable components (the τ_b = −1.00 SBOM has 3).
- Abstract/Ch8 numbers that match Ch7: 90 applications, 332, 11%, 87 of 90, small-world in all 90, median 24 conflicts, 30 with cycles, τ_b 0.00 for both measures, 75%, 45%, 21%, 19%, "milliseconds in the median case and at most a few seconds". No Ch7 sentence reports a namespace-concentration number.
- Labels referenced in Ch7 exist (sec:case-log4shell, sec:scale-free, sec:ns-articulation, sec:dedup-storage, sec:rw-call-graphs, sec:concl-future, sec:ns-bowtie, sec:ns-spectral-radius); Pashchenko2018, Valverde2002 and Myers2003 are in the bibliography and fit their use.
