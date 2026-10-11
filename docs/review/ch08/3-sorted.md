# Chapter 8 — Sorted review

## Summary
- 19 items (C1–C18, D-new1). Sorted into **15 errata**, **1 viva prep** and **0 holds**. E5 merges C7 and D-new1, and E12 merges C11, C12 and C16. Three rebutted sub-points are listed under Holds. The errata, by impact: E1 (C8), E2 (C3), E3 (C2), E4 (C6), E5 (C7+D-new1), E6 (C9), E7 (C1), E8 (C5), E9 (C10), E10 (C4), E11 (C18), E12 (C11+C12+C16), E13 (C13), E14 (C15), E15 (C14). Viva prep: C17.
- **Results.** No reported number is wrong, and no RQ conclusion changes. All Ch8 numbers match `per_sbom.csv` (I recomputed them; see Settled facts). The fixes change how numbers are stated:
  - E3: Ch8:16 adds the 12 % dependents-only reach that Ch7:132 already reports. The decided 19 % stays.
  - E12: Ch8:12 exponent range 2.1–3.0 (median 2.6) → 2.09–2.99 (median 2.58). This is rounding only, matching Ch7:77 and Tab. 7.2.
  - E7: Ch8:18 adds the audit's maximum, 16 s (Tab. 7.6, `time_audit` max 16.39 s).
  - E4: Ch8:14 adds "79 graphs" for HITS. The number is already in Ch7.
  - E5: optional new counts (D4). Advisories published after the release: 8,232 of 8,868 findings. Applications whose every critical finding was published after the release: 60 of 90.
  - Abstract EN/PT: unchanged by every fix. Other chapters touched: Ch2:50 (E11) and Ch7:169 (E2).
- **Settled disputes.**
  - C1: both are right. The audit computes four of the RO3 measures, but its time is the registry check. I timed the four structural parts of the audit (`_compute_entropy`, `_compute_cycles`, `_compute_diamonds`, `_compute_concentration` @28113e7) on all 90 SBOMs on this machine: median 2 ms, max 10 ms. The evaluation never timed them separately, so the thesis cannot claim this. The fix says only what was measured.
  - C3: the critic is right. I re-ran the check: `api.github.com/repos/inascim/sbom-lens-eval` and `/sbom-lens` both return 404, and no chapter has a URL.
  - C4: the defender is right that the existing module is a pairwise diff (`delta.py:14-49`: node and edge set differences, drift score) with no per-application history. Ch1:69 excludes delta analysis because it needs "a temporal data model". The critic is right that the Ch8:67 wording reads as if no comparison existed.
  - C5: the defender is right that :26 holds for the structural results. `handleSave`/`handleSaveAs` (`sbom-graph/index.js:400-440`) make no OSV call, so the critic is right about vulnerability data.
  - C10: the defender is right that "Beyond the objectives" does not contradict the one-to-one mapping. The critic is right that the second sentence has no support in the text. My own check adds one point. "Earlier versions of SBOM Lens" is factually true: commit `c3a3e08` ("Fix graph metrics for thesis evaluation") replaced the immediate-dominator set with the full dominated subtree and fixed the sign of the spectral radius, among other changes. No chapter describes this, and the thesis describes 28113e7 only. Nothing supports "commonly reported in the literature".
  - C15: the defender is right that the v3.0/v3.1 point has no practical effect. CVSS v3.1 kept the v3.0 base equations and redefined only Roundup and an environmental formula. The critic is right that the fallback to the label applies only when no `CVSS_V3` entry exists (`vulnerabilityAPI.js:144-147`).
  - C17: the defender is right. A reference to an undeclared component is not a dependency between components, and Ch4:102 gives the rules.
  - D-new1: confirmed and quantified. Each result file stores `releasePublished` and the OSV `publishedDate` of each finding. Of the 8,868 findings, 8,232 (93 %) were published after the release analysed. In 60 of the 90 applications, every critical finding was published after the release. This counts OSV publication dates, and a GHSA record can be published after the original disclosure, so it is an upper bound on "unknown at release time".
  - Everything else both parties report, I recomputed and it agrees:
    - median 332 nodes; direct share 0.115; 79 fits, α 2.086–2.987, median 2.585;
    - σ min 2.18, median C 0.171, median L 5.01; version conflicts median 23.5; 30 graphs with cycles;
    - τ_b 0.003 / −0.004, with authority defined for 79; zero overlap 61 / 47;
    - transitive-only share 0.75; chokepoint share 0.453; dominated share 0.214; reach 0.186;
    - total time median 5.28 s and max 17.18 s; non-audit sum median 0.44 s and max 4.47 s; slowest single non-audit run 2.91 s (epidemic);
    - lockfiles 53 npm, 36 yarn, 1 pnpm.
- **Four decisions are needed** (list at the end): how the repositories are made available (D1), whether Ch8 states that each objective was met (D2), what replaces the "vocabulary" sentence (D3), and whether to quantify the hindsight in the vulnerability data (D4).

## Erratum

### E1 (from C8) Ch8 never says the evaluation leaves RO3's prioritisation aim untested — impact: medium-high (framing of the central claim against the objectives)
- Settled facts:
  - Ch1:42 sets RO3's purpose as "a principled basis for prioritising remediation beyond raw CVSS scores". Ch1:37 promises that the conclusions can be "verified against each stated goal".
  - Ch7:111 says that τ_b ≈ 0 "does not show that the structural ranking is better". This is a decision already taken.
  - Ch8:24–30 describe what was built. They never say what was shown for RO3, and no limitation says the evaluation has no ground truth.
  - Ch8 does not claim superiority anywhere (:14, :33). So this is an omission, not a false statement.
  - "Risk information" at :18 echoes RO4's own wording (Ch1:43, "structural risk information"), so it can stand.
- Fix:
  - Ch8:30, at the end of the paragraph, after "including where a measure is only a coarse indicator." add: "The evaluation shows that these measures carry information that severity does not; it does not show that prioritising by them leads to better remediation, which would need exploitation data or analyst outcomes."
  - Ch8:65 — "whether the structural view changes the decisions analysts actually make" → "whether the structural view changes, and improves, the decisions analysts actually make"
  - **DECISION NEEDED** (D2): whether to also add a verdict at Ch8:24.
- Also touches: none. Abstract EN:21 / PT:8 ("a basis for prioritising it") is consistent with this, because it claims a basis, not a better outcome.
- Viva answer: "RQ2 shows that structure and severity measure different things. Whether acting on structure improves remediation needs exploitation outcomes or a user study. I state that as the open question, and the user study is the first item of future work."

### E2 (from C3) "Each reported number can be regenerated" is broader than Ch7's scoped claim, and no address is given — impact: medium (RO4 contribution claim)
- Settled facts:
  - Ch7:169 scopes the claim. The script regenerates Tabs. 7.2–7.6 and the figures, and the other counts "are computed from the same data". It also records that the library versions are missing from the run record.
  - Ch8:32 restores "each reported number".
  - No chapter, and no bib entry, gives a URL or DOI. Both GitHub repositories return 404 to an anonymous request today.
- Fix:
  - Ch8:32 — "The collection pipeline, the SBOMs and every analysis result are published with the thesis, so each reported number can be regenerated." → "The collection pipeline, the SBOMs, every analysis result and the analysis script are published in the \texttt{sbom-lens-eval} repository [ADDRESS, per D1], from which the tables and figures of \autoref{cp:evaluation} can be regenerated."
  - Ch7:169 — "are published in the \texttt{sbom-lens-eval} repository." → "are published in the \texttt{sbom-lens-eval} repository [ADDRESS, per D1]."
  - **DECISION NEEDED** (D1): the address.
- Also touches: Ch7:169. Optionally Ch4 or Ch1, where SBOM Lens is first introduced, if the tool repository gets an address too.
- Viva answer: "The tables and figures are regenerated by one script from the published data. The funnel and the in-text counts come from the same data. The archive is at [address]."

### E3 (from C2) The RQ3 headline says exposure is hidden from a flat inventory, then gives a figure that includes the vulnerable components the inventory lists — impact: medium (statement of a reported result; Ch8:16)
- Settled facts:
  - Ch7:132 gives 19 % inclusive, of which the vulnerable components alone are a median 8 %, and 12 % (4 %–29 %) dependents-only. Ch7:134 states both numbers.
  - Ch8:16 gives only 19 %. Ch7:128 says what a flat list hides: how a component entered, not that it is there.
  - This does not reopen the inclusive definition. It matches Ch8 to Ch7's reporting.
- Fix: Ch8:16:
  - "\textbf{RQ3: most vulnerability exposure is structural and hidden from a flat inventory.}" → "\textbf{RQ3: most vulnerability exposure is structural, and a flat inventory does not show how it arises.}"
  - "and 19\% of the application lies in the blast radius of at least one known vulnerability." → "and 19\% of the application's components are vulnerable or depend on a vulnerable component (12\% depend on one)."
- Also touches: none. The numbers are already in Ch7:132.
- Viva answer: "19 % is the blast-radius reach, which includes the vulnerable nodes by design. The part a flat list cannot show is the 12 % that depends on them, and the paths by which the 75 % transitive-only components enter."

### E4 (from C6) The limitations omit the silent numerical edge cases (HITS non-convergence, the power-law bound) and the library-version dependence — impact: medium (limitation of the tool and of a reported result)
- Settled facts:
  - `impact.py:72-79` and `:100-103 @28113e7` return zeros on `PowerIterationFailedConvergence`, with no flag in the response.
  - `graph_analysis.py:128-137` returns α = 3 when powerlaw marks the fit failed (Ch7 sort E1).
  - In the evaluation: 8 graphs did not converge and 3 were all-zero, leaving 79 rankable (recomputed). 8 fits sat on the bound.
  - Ch7:169 says the NetworkX version affects HITS.
  - Ch8:14 and :33 give the HITS median with no n. Ch8:41–53 mention none of this.
- Fix:
  - Ch8, new paragraph after :49: "\textbf{Numerical edge cases.} When HITS power iteration does not converge, SBOM Lens reports every score as zero, and when no valid power-law fit exists it returns the package's bound \(\alpha = 3\); neither case is flagged. In the evaluation each occurred in 8 graphs, and the HITS results, which rest on 79 of the 90 graphs, depend on the NetworkX version (\autoref{sec:hits})."
  - Ch8:14 — "is 0.00 both for blast-radius size and for HITS authority," → "is 0.00 both for blast-radius size and for HITS authority (in the 79 graphs where it is defined),". This is applied together with E13 on the same sentence.
- Also touches: none. The numbers are already in Ch7:77, :109 and :157.
- Viva answer: "Both cases are disclosed where the results are reported. The stored results were kept as SBOM Lens produced them. Flagging them in the response is a small fix that I did not make to the evaluated version."

### E5 (from C7, D-new1) The sample limitation omits the generator's effect and the hindsight in the vulnerability data — impact: medium (external and internal validity of every result)
- Settled facts:
  - Ch7:161: cdxgen produced no usable graph for 23 pnpm projects, and 1 pnpm project remains. Lockfiles are 53 npm, 36 yarn and 1 pnpm (recomputed).
  - Ch7:159: in seven SBOMs the root lacks declared dependencies, and their counts are lower bounds.
  - Ch8:51 lists only the author's filters.
  - D-new1: the queries were made on 30 September 2026 (Ch7:20), and the releases date from 2017 to 2024. 8,232 of 8,868 findings were published by OSV after the release, and in 60 of 90 applications every critical finding was. "Known" means known in 2026.
  - The critic's sub-point on the severity baseline is a Hold (see below).
- Fix: Ch8:51:
  - "Each of these filters shapes the sample: the results describe" → "Each of these filters shapes the sample, and so does the SBOM generator, which produced no usable graph for all but one \texttt{pnpm} project and an incomplete one, whose counts are lower bounds, for seven applications. The results describe"
  - "Vulnerability data reflects the date on which OSV was queried." → "Vulnerability data reflects OSV on 30 September 2026, so it includes advisories published after each release, which an analyst could not have seen at the time."
  - **DECISION NEEDED** (D4): whether to quantify.
- Also touches: Ch7:159 or :161 under D4-B, where the count would also belong.
- Viva answer: "The study is structural, so hindsight does not bias it. It measures where vulnerabilities known today sit in a release's graph. It is not a claim about what an analyst could have known at release. Most advisories post-date the release they affect."

### E6 (from C9) The opening sentence restates the broad novelty claim that the Ch3 review narrowed — impact: medium (novelty framing)
- Settled facts:
  - Ch3:20 and :132–134 limit the claim to the surveyed tools, and they note that the tools list dependents and paths.
  - The settled novelty is dominance, structural measures, and reach plus paths on one graph. Ch8:4 says "Existing SBOM tooling" without qualification.
- Fix: Ch8:4 — "Existing SBOM tooling treats an SBOM as an inventory of components and their known vulnerabilities; SBOM Lens treats it as a directed dependency graph and analyses that graph." → "The SBOM tools surveyed in \autoref{sec:sbom-tools} treat an SBOM as an inventory, at most tracing the paths to a component or listing its dependents; SBOM Lens builds a directed dependency graph and computes dominance and structural measures over it."
- Also touches: none. Abstract EN:17 says "largely treats", which is already qualified.
- Viva answer: "Other tools traverse dependents and paths. What SBOM Lens adds is dominance and the structural measures, on the same graph as reach and paths."

### E7 (from C1) "Structural analyses took … at most a few seconds" leaves out the audit — impact: medium (the only performance claim in Ch8)
- Settled facts:
  - The audit returns entropy, cycles, version conflicts and concentration (`audit.py:240-244`). Ch8:30 lists these four as RO3 measures.
  - The audit took a median of 4.72 s and at most 16.39 s.
  - Every other endpoint has a median of 1.6–151 ms, and the slowest single run was 2.91 s.
  - Ch7:151 says "every other analysis".
- Fix: Ch8:18 — "its structural analyses took milliseconds in the median case and at most a few seconds." → "every analysis except the audit took milliseconds in the median case and under three seconds at most; the audit, which also queries the npm registry once per package, took up to 16 seconds."
- Also touches: none. The numbers are already in Tab. 7.6 and Ch7:151.
- Viva answer: "The audit's time is the registry lookup. Timed separately on the same 90 SBOMs on my machine, its four graph measures take a median of 2 ms and at most 10 ms. The evaluation did not time them apart, so the thesis reports the endpoint as measured."

### E8 (from C5) "Current vulnerability data" and "the displayed results always describe the current graph" overstate how fresh the vulnerability data is — impact: medium
- Settled facts:
  - OSV is queried only from `handleUploaded` (`dashboard/index.js:86`). Saving an edit makes no OSV call (`sbom-graph/index.js:400-440`).
  - The backend half of :26 is correct: the graph is rebuilt and all caches are invalidated.
  - Ch5:173, as corrected, states the staleness.
- Fix:
  - Ch8:26 — "so the displayed results always describe the current graph." → "so the structural results always describe the current graph."
  - Ch8:43 — "the same, current vulnerability data" → "the same vulnerability data, current at upload,"
  - Ch8:53 — "and it accepts CycloneDX JSON only." → "it accepts CycloneDX JSON only; and it queries OSV only when an SBOM is uploaded, so vulnerability records are not refreshed and a component added by editing is not looked up."
- Also touches: none.
- Viva answer: "Structural analyses follow every edit. The vulnerability lookup is an upload-time step, and re-uploading refreshes it."

### E9 (from C10) The "vocabulary" contribution claims misnamed measures in the literature and in earlier versions, and no chapter shows either — impact: medium (unsupported contribution claim)
- Settled facts:
  - No chapter names a misnamed measure in the literature.
  - The earlier-version claim is true in the git history (`c3a3e08` replaced the immediate-dominator set with the dominated subtree and fixed the spectral-radius sign). But the thesis describes 28113e7 only, and nothing in it shows this.
  - The misnamings the thesis does disclose are in the evaluated tool: `r0` (Ch6:48) and "Diamond Dependencies" (Ch6:116).
  - The first sentence of :35 holds, because Ch6 makes all four distinctions.
- Fix: **DECISION NEEDED** (D3). Recommended minimal form (A): Ch8:35 — "Several measures commonly reported in the literature and in earlier versions of SBOM Lens were found, on inspection, to compute something other than their names suggested; the thesis reports each measure as it is actually computed." → "Where the interface uses a name that suggests more than is computed --- the \texttt{r0} field, the \emph{Diamond Dependencies} label --- the thesis states what is actually computed."
- Also touches: none.
- Viva answer: "During development, several of the tool's own measures turned out to compute something else. For example, the dominator overlay first showed only immediately dominated nodes. They were corrected before the evaluation. The thesis reports the corrected versions and names the interface labels that still overstate."

### E10 (from C4) Future work proposes SBOM comparison, which the evaluated tool already offers as a pairwise view — impact: medium-low
- Settled facts:
  - `GET /sboms/compare` exists (`delta.py:14-49`), and the navigation has the view (Ch4:22, :35).
  - It is pairwise, with no per-application history. Ch1:69 excludes delta analysis.
- Fix: Ch8:67 — "and comparison of SBOMs across versions of the same application would turn" → "and a version history for each application, extending the pairwise SBOM comparison view outside the scope of this thesis (\autoref{sec:architecture-overview}), would turn"
- Also touches: none.
- Viva answer: "A set difference between two SBOMs exists and was not evaluated. Future work is a version history of one application, with the structural measures tracked over it."

### E11 (from C18) Ch2 contradicts Ch8 on the vulnerability source; the NVD proposal ignores Ch5's CPE argument — impact: low-medium (cross-chapter inconsistency)
- Settled facts:
  - Ch2:50 says "sourced directly from NVD, OSV, and GitHub Security Advisories".
  - Ch5:20, Ch8:41 and the code (`vulnerabilityAPI.js`, OSV `querybatch` only) say OSV only.
  - Ch5:16 argues that CPE matching is imprecise.
- Fix:
  - Ch2:50 — "Vulnerability data is sourced directly from NVD, OSV, and GitHub Security Advisories rather than from embedded VEX data;" → "Vulnerability data is obtained from OSV rather than from embedded VEX data;"
  - Ch8:61 — "Direct integration of NVD by CPE" → "Direct integration of NVD, despite the imprecision of CPE matching (\autoref{sec:multi-source-rationale}),"
- Also touches: Ch2:50.
- Viva answer: "NVD would add only advisories with no open-source record, and CPE imprecision is why it was not the first choice."

### E12 (from C11, C12, C16) The RQ1 summary drops the weak-baseline and undirected caveats, misattributes the measures, and rounds the range to the bound — impact: low (wording of a reported result; Ch8:12, :47)
- Settled facts:
  - Ch7:79 says "The bound itself is weak". Ch7:90 says "The structural measures of \autoref{cp:network-sci}", not measures "developed for ecosystems".
  - L is computed on the undirected largest component (`graph_analysis.py:151-174`), and Ch6:184 says this closeness "is not exposure". σ is missing from the Ch8:47 list, although the Ch6 sort relied on it being there.
  - α is 2.086–2.987, median 2.585, so 3.0 is the bound value.
- Fix:
  - Ch8:12 — "with exponents between 2.1 and 3.0 (median 2.6)" → "with exponents between 2.09 and 2.99 (median 2.58)"
  - Ch8:12 — "and every graph is small-world: components form tightly clustered neighbourhoods while remaining about five hops apart." → "and every graph is small-world against a weak random baseline: components form tightly clustered neighbourhoods while remaining about five undirected hops apart."
  - Ch8:12 — "Graph measures developed for ecosystems therefore have structure to work with" → "The structural measures of \autoref{cp:network-sci} therefore have structure to work with"
  - Ch8:47 — "the rich-club coefficient and all spectral measures" → "the rich-club coefficient, the small-world coefficient and all spectral measures"
- Also touches: none. The numbers match Ch7:77 and Tab. 7.2.

### E13 (from C13) "Unrelated" is stronger than Ch7 and the Abstract — impact: low (wording of RQ2)
- Settled facts:
  - Ch7:107 and Abstract EN:21 say "essentially unrelated", and 26 of 90 SBOMs have |τ_b| ≥ 0.2.
  - Ch7:107 says the overlap restates τ_b. Ch8:14 joins the two with "and".
- Fix:
  - Ch8:14 — "gives unrelated orderings" → "gives essentially unrelated orderings"
  - Ch8:14 — ", and in most applications none of the critical components" → ", so that in most applications none of the critical components". This is combined with the E4 insertion just before it.
  - Ch8:33 — "were found to be unrelated" → "were found to be essentially unrelated"
- Also touches: none.

### E14 (from C15) RO2 and the limitations describe severity handling more cleanly than the code — impact: low (no result affected)
- Settled facts:
  - Any `CVSS_V3` entry, whether v3.0 or v3.1, is scored with the v3.1 equations. The base equations are the same, so there is no practical effect.
  - The fallback to the label applies only when no `CVSS_V3` entry exists (`vulnerabilityAPI.js:144-147`). An unscorable v3 vector gives UNKNOWN.
  - No evaluation finding was UNKNOWN. The UNKNOWN-as-LOW colour is disclosed in Ch5:108.
- Fix:
  - Ch8:28 — "Severity is derived from the CVSS v3.1 vector by a client-side implementation of the specification, falling back to the advisory's own severity label," → "Severity is derived from the CVSS v3 vector by a client-side implementation of the v3.1 specification, falling back, when no such vector is present, to the advisory's own severity label,"
  - Ch8:41 — "Severity is computed from CVSS v3.1 vectors only;" → "Severity is computed from CVSS v3 vectors only;"
- Also touches: none. Ch1:41/:54 ("CVSS v3.1 base score computation") are about the formula and are correct.
- Viva answer: "v3.0 and v3.1 share the base equations. 'Recorded as unknown rather than treated as low' is true of the data. The graph colour shortcut is disclosed in Ch5."

### E15 (from C14) "The dependency paths" overstates the output: at most five shortest paths — impact: low
- Settled facts: `impact.py:52-63` collects shortest paths from the roots and stops at 5 in total. Ch5:221 says "up to five shortest paths".
- Fix: Ch8:28 — "the dependency paths through which it enters the application" → "up to five shortest dependency paths through which it enters the application"
- Also touches: none.

## Viva prep

### V1 (from C17) Does every declared dependency become an edge?
- Likely question: "You say every component is a node and every dependency an edge. Is that so?"
- Answer: Edges are added only between declared nodes, and self-loops are dropped (`sbom_store.py:14-34`). A reference to an undeclared component is not a dependency between two components, and Ch4:102 says such entries are skipped. On the node side, components without `bom-ref`, `purl` or `name` get no node, colliding IDs merge, and nested `components` are not read. Ch8:26 is a one-line summary. Optional: Ch8:26 "every component is a node and every declared dependency a consumer-to-dependency edge" → "each declared component is a node and each dependency between declared components a consumer-to-dependency edge".

## Holds
- No whole item. Three sub-points are rebutted and need no action:
  - C10, "fifth contribution": the paragraph opens "Beyond the objectives", so it does not contradict the one-to-one mapping of :24 or Ch1:50.
  - C7, severity baseline: Ch8:14 already says "by severity band", and Ch7:157 covers numeric CVSS and EPSS.
  - C13, overlap as "further evidence": Ch8 does not call it independent. E13's "so that" removes the remaining ambiguity.

## Decisions needed
1. **Repository availability (E2/C3).** Both repositories are private today.
   - (A) Make `sbom-lens-eval` and `sbom-lens` public before submission and give the GitHub URLs at Ch8:32 and Ch7:169. This is quick, but the address can change.
   - (B) Archive a tagged release with a DOI (e.g. Zenodo) and cite it in the bibliography. This is durable and citable, and takes a little more effort.
   - (C) Keep them private, write "available from the author on request", and drop "published". This weakens the RO4 contribution.
2. **Objective verdict (E1/C8).**
   - (A) Add only the RO3 qualification sentence.
   - (B) Also change Ch8:24 "map one-to-one to the contributions of this work." → "map one-to-one to the contributions of this work; each was met, with the qualification stated for RO3." This fulfils Ch1:37 explicitly, and the author must be willing to assert it.
3. **Vocabulary sentence (E9/C10).**
   - (A) Replace it with the interface-names sentence. This is recommended, because it stays within the 28113e7 scope.
   - (B) Keep the "earlier versions" claim and support it with one sentence on the corrections made before the evaluation (`c3a3e08`), dropping "the literature". This describes pre-evaluation history the thesis otherwise omits.
   - (C) Delete the last sentence without replacement.
4. **Quantify the hindsight (E5/D-new1).**
   - (A) Qualitative sentence only.
   - (B) Add "(93\% of the findings, and every critical finding in 60 of the 90 applications)". This needs the count added to `analyse.py`, or computed from the same data, to stay within the reproducibility scope. It is an upper bound, because OSV publication dates can be later than the original disclosure.
