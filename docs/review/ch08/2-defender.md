# Chapter 8 — Defender

All code references are at `28113e7` (`git show 28113e7:<path>`) unless stated otherwise. Recomputations were run on `sbom-lens-eval/analysis/out/per_sbom.csv` (90 SBOMs) with the venv Python. I re-read each chapter line the critic cites. The critic's facts hold almost everywhere. Most items are fair omissions or places where Ch8 says a little more than the corrected chapters. None of them changes a result.

## C1. "Structural analyses took milliseconds … at most a few seconds" leaves out the audit, which computes four of the structural measures Ch8 lists
- Verdict: partly rebutted
- Evidence: Recomputed: `time_audit` median 4.72 s, max 16.39 s; `timeTotal` median 5.28 s, max 17.18 s. The audit is a median 91 % of the total. All other endpoints summed: median 0.44 s, max 4.47 s. The slowest single non-audit run is 2.91 s (epidemic). So "at most a few seconds" is true for every analysis except the audit. The audit's cost is the unmaintained-package check: `_check_unmaintained` (`backend/routers/audit.py:189` ff.) queries the npm registry per package. That is network I/O, not structural analysis, and Ch7:151 says so. The critic is right on the core point, though. The audit endpoint also computes entropy, cycles, version conflicts and namespace concentration (`audit.py:240-244`), and Ch8:30 lists these as structural measures. Their own cost was not timed apart from the registry calls. Ch8 drops the word "other" that Ch7:151 uses.
- Viva defence: the slow part is a registry lookup, not a graph computation. The four structural measures in the audit are linear-time and cheap. But the measured number includes the lookup, and Ch8 should say so the way Ch7 does.
- Minimal thesis fix: :18 "its structural analyses took milliseconds in the median case and at most a few seconds" → "every analysis except the audit took milliseconds in the median case and at most three seconds; the audit, which queries the npm registry for each package, took up to 16 seconds".

## C2. The RQ3 headline calls exposure "hidden from a flat inventory", then quotes the inclusive 19 %, which counts the vulnerable components the inventory does list
- Verdict: conceded
- Evidence: Ch7:132 reports 19 % inclusive (vulnerable components themselves a median 8 %) and 12 % (4 %–29 %) for dependents only. Ch7:134 gives both. Ch8:16 gives only 19 %, under the "hidden" headline. Ch7:128 correctly says what a flat list hides: how the component entered, not that it is there. This does not reopen the 19 %/12 % decision. It is about matching Ch7's two-number reporting.
- Viva defence: 19 % is the decided headline measure, and it includes the vulnerable nodes by design. The hidden part is the 12 % that depends on them, and the entry paths of the 75 %.
- Minimal thesis fix: :16 "and 19\% of the application lies in the blast radius of at least one known vulnerability" → "and 19\% of the application is vulnerable or depends on a vulnerable component, 12\% through a dependency alone". Optionally, headline → "most vulnerability exposure is structural: a flat inventory lists it but does not show how it enters".

## C3. Reproducibility: "published with the thesis, so each reported number can be regenerated" is broader than Ch7's scoped claim, and the repositories are not publicly reachable
- Verdict: conceded
- Evidence: (a) Ch7:169 (tail) scopes the claim: the script regenerates Tabs. 7.2–7.6 and the figures; the funnel and other counts "are computed from the same data"; rerunning collection reproduces the dataset "up to changes in the vulnerability databases and in the GitHub repositories". It also says the run record lacks library versions. Ch8:32 drops all of this. (b) `grep -E "github\.com|doi\.org|url\{|zenodo" Chapters/*.tex` finds nothing. No chapter gives an address. I did not re-run the critic's anonymous API check, but the absence of any URL in the thesis is enough.
- Viva defence: the repositories exist and will be made public with the thesis. The reproducibility scope was agreed in the Ch7 review and Ch8 should simply repeat it.
- Minimal thesis fix: :32 "are published with the thesis, so each reported number can be regenerated" → "are published in the \texttt{sbom-lens-eval} repository (\url{…}), from which the tables and figures of \autoref{cp:evaluation} can be regenerated". Add the repository URLs (eval and tool) to Ch7:169 too.

## C4. Future work proposes "comparison of SBOMs across versions", but the evaluated system already has an SBOM Comparison/Delta module, which Ch4 acknowledges
- Verdict: partly rebutted
- Evidence: Confirmed: `backend/routers/delta.py:13-30` (`GET /sboms/compare?before=&after=`, node/edge set differences); Ch4:22 and Ch4:35 name the module and the navigation entry. But the existing module compares two arbitrary SBOMs by ID. It has no notion of "the same application across versions" and no structural-risk tracking. Ch1:69 puts delta analysis out of scope because it "would require a temporal data model not present in the current system". The Ch8:67 sentence ends with the purpose: "in which the structural risk of an application could be tracked as its dependencies evolve". That is the temporal model Ch1:69 says is missing. So the proposal is not already built. The wording, however, reads as if no comparison existed.
- Viva defence: a pairwise diff of node and edge sets exists, outside the scope of the thesis. What is future work is a version history of one application, with structural measures tracked over it.
- Minimal thesis fix: :67 "and comparison of SBOMs across versions of the same application" → "and a version history for each application, extending the existing (unevaluated) SBOM comparison view of \autoref{cp:system-design}".

## C5. "Current vulnerability data" and "the displayed results always describe the current graph" contradict Ch5: OSV is queried once, at upload, and never for edits
- Verdict: partly rebutted
- Evidence: :26 is about the graph and its analyses. The critic confirms that half: `sboms.py:79-111` rebuilds the graph and `invalidate_graph_analysis_caches` clears every analysis cache. The out-of-scope delta caches are the only exception. So "the displayed results always describe the current graph" is correct for the structural results. :43 contrasts OSV with producer-embedded data. In that contrast "current" means current at query time, not frozen at build time, and that holds. The critic is right on what is missing: the only call site of `fetchVulnsForSBOM` is `dashboard/index.js:86` (upload). `handleSave`/`handleSaveAs` do not query OSV. The graph overlay matches records by PURL from the global store (`sbom-graph/index.js:102-107`). So a component added by editing shows a vulnerability only if another upload already brought in a record for its PURL. Ch5:173 says so; Ch8 does not.
- Viva defence: the structural analyses do follow edits. Vulnerability lookup is a separate, upload-time step, and Ch5 discloses that it is not refreshed.
- Minimal thesis fix: :43 "the same, current vulnerability data" → "the same vulnerability data, current at upload". :26 "so the displayed results always describe the current graph" → "so the structural results always describe the current graph; vulnerability data is fetched only at upload (\autoref{cp:vuln-analysis})". Add to :53: "vulnerability records are not refreshed after upload".

## C6. The limitations omit the implementation-dependent results disclosed in Ch6 and Ch7: HITS non-convergence, the powerlaw bound, and library-version dependence
- Verdict: conceded
- Evidence: Confirmed. `impact.py:72-79` returns 0.0 on `PowerIterationFailedConvergence`. Ch7:109 reports 8 non-convergent and 3 all-zero graphs. Ch7:157 says the authority results rest on 79. Ch7:77 reports 8 fits returned at α = 3. Ch7:169 says the run record lacks library versions and that NetworkX affects HITS. Ch8:41-53 mentions none of this. Ch8:14 and :33 give the HITS median without "of 79".
- Viva defence: all of this is disclosed where the results are reported. The decision was to keep the stored results and disclose them. The conclusion should carry one sentence of that disclosure.
- Minimal thesis fix: add a limitation paragraph: "\textbf{Numerical robustness.} When HITS power iteration does not converge, SBOM Lens reports every score as zero, and when the power-law fit fails it returns the bound \(\alpha = 3\); neither is flagged in the interface. In the evaluation this left 11 of the 90 graphs without an authority ranking and 8 without a power-law fit. HITS and community results also depend on the NetworkX version." At :14 add "(79 rankable SBOMs)" after "HITS authority".

## C7. The evaluation-sample limitation omits generator quality: the pnpm loss and the graphs that are lower bounds
- Verdict: conceded
- Evidence: Ch7:161 ("only one pnpm project remains, and the findings describe npm and yarn installations"), Ch7:159 (four graphs below 90 % agreement; two miss 14 % and 20 %; seven roots lack declared dependencies, so their counts are lower bounds), Ch7:157 (severity band, not numeric CVSS or EPSS). Ch8:51 lists only the author's filters.
- Viva defence: the validity threats are reported in full in Ch7. Ch8 summarises the sample, and the generator limits belong in that summary.
- Minimal thesis fix: :51 "The evaluation covers popular npm applications only" → "The evaluation covers popular npm applications installed with npm or yarn only (cdxgen produced no usable graph for pnpm projects)". Add after "Each of these filters shapes the sample": "So does the SBOM generator: for seven applications the generated graph is incomplete, and its counts are lower bounds. The severity baseline is the severity band, not numeric CVSS or exploit likelihood."

## C8. Ch8 maps contributions to objectives but never says whether each objective was met; RO3's "principled basis for prioritising remediation" is not shown by the evaluation
- Verdict: conceded
- Evidence: Ch1:37 promises the conclusions can be "verified against each stated goal". Ch1:42 sets RO3's purpose as "a principled basis for prioritising remediation beyond raw CVSS scores". Ch7:111 says τ_b ≈ 0 "does not show that the structural ranking is better". Ch8 never claims it is better: :14 and :33 report only that the rankings are unrelated. But Ch8:24-30 describe what was built, not what was shown, and no limitation says there is no ground truth for prioritisation. Ch8:18 "risk information" goes slightly beyond Ch7:111's "information a flat inventory cannot provide".
- Viva defence: the evaluation shows that structure carries information severity does not. Whether acting on it improves remediation needs exploitation data or a user study, and Ch8:65 already proposes the user study.
- Minimal thesis fix: :18 "carries risk information" → "carries information about exposure". At the end of the RO3 paragraph (:30) add: "The evaluation shows that these measures carry information that severity does not; it does not show that prioritising by them leads to better remediation, which would need exploitation data or analyst outcomes (\autoref{sec:concl-future})." Optionally add one sentence per objective stating whether it was met.

## C9. "Existing SBOM tooling treats an SBOM as an inventory … SBOM Lens treats it as a directed dependency graph" drops the qualifications settled in Ch3
- Verdict: conceded
- Evidence: Ch3:20 and Ch3:132 limit the claim to "the surveyed tools" and allow path and dependent queries. The settled novelty is narrow. Ch8:4 generalises to "Existing SBOM tooling" with no qualification.
- Viva defence: the sentence is an opening summary. The precise claim is in Ch3, and the fix is a short qualification.
- Minimal thesis fix: :4 "Existing SBOM tooling treats an SBOM as an inventory of components and their known vulnerabilities; SBOM Lens treats it as a directed dependency graph and analyses that graph." → "The SBOM tools surveyed in \autoref{sec:sbom-tools} treat an SBOM as an inventory, at most tracing the paths to a component or its dependents; SBOM Lens builds a directed dependency graph and computes structural measures over it."

## C10. The "vocabulary" contribution claims measures "commonly reported in the literature and in earlier versions of SBOM Lens" were misnamed; no chapter shows either
- Verdict: partly rebutted (the second sentence is conceded)
- Evidence: `grep -i -E "earlier version|misnam"` over Chapters/ finds only Ch8:35. No chapter names a misnamed measure in the literature or describes an earlier version. Ch6:48's "for historical reasons" implies an earlier naming but does not describe it. The misnamings the thesis does disclose (`r0`, "Diamond Dependencies", Ch6:48, Ch6:150) are in the evaluated tool. The second sentence of :35 is therefore unsupported. The first sentence holds: Ch6 does make each of the four distinctions it lists. The "fifth contribution" objection is weak: the paragraph opens "Beyond the objectives", so it does not contradict the one-to-one mapping of :24.
- Viva defence: the distinctions are real, and the thesis makes them. The claim about the literature and about earlier versions has no reference, and it should go.
- Minimal thesis fix: :35 replace the last sentence with: "Where the interface uses a name that suggests more than is computed --- the \texttt{r0} field, the \emph{Diamond Dependencies} label --- the thesis states what is actually computed."

## C11. RQ1 summary: "every graph is small-world … tightly clustered" drops Ch7's weak-bound caveat, and the measures were not "developed for ecosystems"
- Verdict: partly rebutted
- Evidence: Ch7:79 itself uses "tightly interlinked neighbourhoods" and "about five hops apart" (median L = 5.0), with the median C = 0.17 far above C_rand. So the Ch8 description matches Ch7's. What Ch8 drops is the sentence that the bound is weak. The small-world sentence sits in the body, not in the heavy-tailed headline, so it does not attribute small-world structure to ecosystems. The critic is right on "Graph measures developed for ecosystems": Ch7:90 says "The structural measures of \autoref{cp:network-sci}", which are general network science. The "five hops" is undirected (Ch6:184). C16 covers that.
- Viva defence: the description is Ch7's own. The last sentence was meant to say that ecosystem-scale findings carry over, and it is easily reworded.
- Minimal thesis fix: :12 "Graph measures developed for ecosystems therefore have structure to work with" → "The structural measures of \autoref{cp:network-sci} therefore have structure to work with". Optionally add after the small-world clause: "(against a weak random baseline)".

## C12. RQ1 range "between 2.1 and 3.0" invites reading the upper end as the package bound Ch7 excludes
- Verdict: partly rebutted
- Evidence: Recomputed: min 2.086, max 2.987. Rounding to 2.1 and 3.0 is correct, and Ch8 says "79 of the 90", so the bound fits are already excluded. The risk of misreading is small but real, because 3.0 is the bound value.
- Viva defence: the numbers are correctly rounded and the count excludes the bound fits.
- Minimal thesis fix: :12 "between 2.1 and 3.0" → "between 2.09 and 2.99".

## C13. RQ2 summary: the zero top-k overlap is presented as further evidence, though Ch7 now says it is what an unrelated ranking gives; "unrelated" is stronger than Ch7 and the Abstract
- Verdict: partly rebutted
- Evidence: Ch7:107 itself reports the overlap as "the practical measure [that] shows what this means for an analyst", then says it restates τ_b. Ch8:14 joins it with "and". It does not call it independent or further evidence. The finding is correct (61/90 recomputed by the critic). The critic is right that "unrelated" drops Ch7's and the Abstract's "essentially", and 26 of 90 SBOMs have |τ_b| ≥ 0.2. The Ch7 sort already judged :33 acceptable.
- Viva defence: the overlap is how the τ_b result looks to an analyst. It is not offered as a separate test.
- Minimal thesis fix: :14 "gives unrelated orderings" → "gives essentially unrelated orderings"; "and in most applications" → "so that in most applications". :33 "unrelated" → "essentially unrelated".

## C14. "The dependency paths through which it enters" overstates the impact output: at most five shortest paths
- Verdict: conceded
- Evidence: `impact.py:52-63`: `nx.all_shortest_paths` from each root, stopping at 5 paths in total. Ch5:221 says "up to five shortest paths".
- Viva defence: the paths are a sample for explanation, not an enumeration. The blast radius and dominated set are complete.
- Minimal thesis fix: :28 "the dependency paths through which it enters the application" → "up to five shortest dependency paths through which it enters the application".

## C15. RO2 and Limitations: severity handling is described more cleanly than the code. v3.0 vectors are scored with the v3.1 formula, an unparseable v3 vector does not fall back to the label, and UNKNOWN is drawn in the LOW colour
- Verdict: partly rebutted
- Evidence: (a) v3.0 vectors. True that `cvssV3BaseScore` ignores the version prefix (`vulnerabilityAPI.js:14-20`, :61-70). But CVSS v3.1 did not change the base-score equations of v3.0. It changed the definition of Roundup (floating-point precision) and environmental formulas. Scoring a v3.0 base vector with the v3.1 equations gives the same base score, except for rare rounding edge cases. So this point has no practical effect, and "CVSS v3.1 vectors only" (:41) is the only misstatement: v3.0 vectors are scored too. (b) Unparseable vector. Confirmed: `vulnerabilityAPI.js:145-147` falls back to the label only when no `CVSS_V3` entry exists. (c) UNKNOWN colour: confirmed and disclosed in Ch5:108. Ch8:28 says nodes are coloured "by their highest severity band", which is not false. No evaluation finding was UNKNOWN (Ch7:51).
- Viva defence: no result depends on any of this. The v3.0 point is harmless because the base equations are identical.
- Minimal thesis fix: :41 "computed from CVSS v3.1 vectors only" → "computed from CVSS v3 vectors only". :28 "falling back to the advisory's own severity label" → "falling back, when no v3 vector is present, to the advisory's own severity label".

## C16. The undirected-measures limitation omits the small-world coefficient, the one measure Ch8 itself reads as distance
- Verdict: conceded
- Evidence: `_compute_small_world` uses `G.to_undirected()` and its largest component (`graph_analysis.py:151-174`). Ch6:184 says closeness in G_u "is not exposure". Ch8:47 does not list σ, C or L, and the Ch6 sort (E5) relied on it doing so.
- Viva defence: Ch6 states it plainly. The list in Ch8 is just incomplete.
- Minimal thesis fix: :47 "the rich-club coefficient and all spectral measures" → "the rich-club coefficient, the small-world coefficient with its clustering and path length, and all spectral measures".

## C17. RO1 says every declared dependency becomes an edge; dangling references and self-loops are dropped
- Verdict: partly rebutted
- Evidence: `sbom_store.py:14-34` adds an edge only between declared nodes and skips self-loops. A reference to an undeclared component is not a dependency between two components, so the sentence is true in its natural reading. Self-loops "carry no dependency information" (code comment). The node side is a fair catch: components with no `bom-ref`, `purl` or `name` get no node, IDs that collide merge into one node, and nested `components` are not read. Ch4:102 is more precise.
- Viva defence: this is a one-sentence summary. Ch4 gives the exact rules.
- Minimal thesis fix: :26 "in which every component is a node and every declared dependency a consumer-to-dependency edge" → "in which each declared component is a node and each dependency between declared components a consumer-to-dependency edge (\autoref{cp:system-design} gives the exact rules)".

## C18. Two internal mismatches around the vulnerability-source discussion Ch8 hosts
- Verdict: conceded
- Evidence: Ch2:50 says "Vulnerability data is sourced directly from NVD, OSV, and GitHub Security Advisories". That contradicts Ch5:20, Ch8:41 and the code (OSV `querybatch` only). Ch5:16 argues that CPE matching is imprecise for open-source packages. Ch8:61 proposes NVD-by-CPE integration with no caveat. Ch8 itself is correct on the source. The fix for the first mismatch is in Ch2.
- Viva defence: NVD integration would only fill the gap for advisories that have no open-source record. CPE imprecision is the reason it was not done first.
- Minimal thesis fix: Ch2:50 "Vulnerability data is sourced directly from NVD, OSV, and GitHub Security Advisories rather than from embedded VEX data" → "Vulnerability data is obtained from OSV rather than from embedded VEX data". Ch8:61 "Direct integration of NVD by CPE" → "Direct integration of NVD, with care for the imprecision of CPE matching (\autoref{cp:vuln-analysis}),".

## D-new1. "Critical known vulnerability" is known as of 2026, not as of each release, and Ch8 does not say so
- Type: omission
- Origin: pre-existing
- Severity: low
- Thesis: Chapters/08-Conclusion.tex:10 — "each with at least one critical known vulnerability"; :51 — "Vulnerability data reflects the date on which OSV was queried."
- Evidence: Ch7:20: "The queries were made on 30 September 2026." Ch7:161: releases from 2017 to 2024 (tags on or before 31 March 2024, Ch8:51). So the vulnerabilities are judged in hindsight. Many advisories were published after the release they affect, and none of them would have been visible to an analyst at release time. Ch8:51 gives no date and does not draw this consequence. No chapter states it explicitly.
- Why it matters: "known vulnerability" reads as "known at the time". The sample selection (at least one CRITICAL) and the RQ3 exposure figures depend on advisories that could not have been acted on at release. The analysis is still valid as a structural study, but the framing should be explicit.
- Minimal thesis fix: :51 "Vulnerability data reflects the date on which OSV was queried." → "Vulnerability data reflects OSV on 30 September 2026, so it includes advisories published after each release was made."
