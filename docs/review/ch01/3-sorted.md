# Chapter 1 and Abstract — Sorted review

## Summary
- 20 items (C1–C18, D-new1, D-new2). Sorted into **16 errata**, **1 viva prep** and **0 holds**. E1 merges C1 and C9, E2 merges C2 and C15, E3 merges C4 and C18, and E10 merges C11 and C17. Rebutted sub-points are listed under Holds. The errata, by impact: E1 (C1+C9), E2 (C2+C15), E3 (C4+C18), E4 (C3), E5 (C6), E6 (C7), E7 (C5), E8 (C8), E9 (C10), E10 (C11+C17), E11 (C13), E12 (C12), E13 (C14), E14 (C16), E15 (D-new1), E16 (D-new2). Viva prep: V1 (from C7).
- **Results.** No reported number is wrong, and no RQ conclusion changes. Ch1 reports no evaluation number. The Abstract's figures match `per_sbom.csv` (I recomputed them; see Settled facts). The fixes add numbers only where Ch7 already states them:
  - E3: Ch1:67 adds "90 open-source npm applications, each with at least one critical known vulnerability" (Ch7:20, :24; Ch8:10).
  - E11: Ch1:7 replaces "orders of magnitude" with "several times". OSSRA 2024 reports 77 % open-source code, which is about 3.3 times the first-party code.
  - Abstract EN/PT: changed by E10 only (PT first paragraph; items (2) and (3) in both languages). The results paragraph (EN:21, PT:8) is unchanged.
  - Other chapters touched: none. Every fix was checked against Ch2–Ch8 wording (see "Also touches" in each erratum).
- **How the fixes compose.** Several items rewrite the same Ch1 sentences. Each such sentence has **one final text**, given in the erratum named below. The other errata point to it and give no text of their own. Every old string below is unique in the file, so the replacements can be applied in any order.

  | Ch1 line | Final text in | Items folded in |
  |---|---|---|
  | :9 (two sentences) | E7 (log4shell sentence); E14 ("attacks", last sentence) | C5; C16, D2 |
  | :15 (sentences 2–5) | E2 | C2, C15, C7 (hedge), C14 (definition), C1 (dominance) |
  | :23 (sentences 1 and 3) | E5 | C6, C7 (scope), C15 ("dependency tree") |
  | :31 (sentence 2) | E8 | C8, C1 |
  | :31 (sentence 4) | E4 | C3, D1 |
  | :41 RO2 | E1 | C1 |
  | :42 RO3 | E1 | C1, D-new2 |
  | :43 RO4 | E4 | C3, C4, D1 |
  | :54 Contribution 2 | E1 | C1 |
  | :55 Contribution 3 | E1 | C1, C2, D-new2 |
  | :56 Contribution 4 | E3 | C4, C18 |
  | :67 | E3 | C4, C2, D3 |
  | :69 (sentences 2 and 4) | E9 (sentence 2); E3 (sentence 4) | C10; C18 |
  | :89 | E3 | C3, C4 |
  | Abstract EN:19, PT:4, PT:6 | E10 | C11, C17 |

- **Settled disputes.**
  - C2: the defender is right that :27 and RO3's "principled basis" (:42) can stand. Ch8:24 accepts RO3 "with the qualification stated for RO3". The critic is right about :15 and :55, which state as fact what Ch7:111 and Ch8:30 say is not tested. Ch1 has no counterpart to the Ch8:30 qualification, so I add one sentence to the scope paragraph (:67). That is where Ch1 already says what the evaluation does not test.
  - C3: the defender is right that Ch7:4 derives the RQs from RO4 and defines them before Ch8 answers them, so stating them in Ch1 is an author choice (D1). Both are right that "The research question is …" (:31, singular) and the Ch7 outline (:89) are wrong as they stand. Those two fixes apply whatever D1 decides.
  - C4: the defender is right that hindsight and OSV-only are decided as disclosed in Ch7/Ch8. The npm-only sample and the critical-vulnerability filter belong in Ch1. "At scale" (:56) is wrong: 90 accepted out of 1,281 processed (Ch7:24). How much more of the framing goes into Ch1 is D3.
  - C7: the defender is right that "closest existing tool" matches Ch3:10. The two 2026 papers (Ch3:138) are research, not tools. The hedge is required by the decided novelty framing. :81 "the gap" contradicts Ch3:130 ("four capabilities") and Ch3:4 ("four named research gaps"), so it is replaced by the four gap names of Ch3:132–138.
  - C8: the defender is right that the blast radius is exact over the recorded edges (`ancestors + [node_id]`, Ch5:219). "Accurate and complete picture" still reads as a claim about real exposure, which Ch7:157 and Ch8:45 deny. The fix keeps the exactness and states what it is relative to.
  - C12: the defender is right that the gloss follows the CRA's own definition (Art. 3(39): "a formal record containing details and supply chain relationships of components"). "All software components" overstates the minimum, which is top-level dependencies (Ch2:20). "Has thus transitioned … to a legal requirement" contradicts "from December 2027" in the same paragraph, because today is October 2026. The fix changes only the gloss and the tense, as the decision allows.
  - C13: confirmed by web search. OSSRA 2024: 96 % of 1,067 codebases contain open source, and 77 % of the scanned code is open source. The defender's second point also holds: OSSRA's component count already includes transitive components, so "introduces its own transitive closure of further dependencies" counts them twice.
  - C14: the defender is right that Ch5:219 declares the graph sense of "upstream" on purpose. Ch1 switches senses within four paragraphs, so the fix drops the word where it is not needed (:13, :25). :9 keeps the supply-chain sense of Ch2:62. The :15 definition must include \(v\) (Ch2:126, Ch5:219, CONTEXT.md).
  - C15: the defender is right that "single points of structural failure" matches Ch6:24 ("natural points of fragility"). "Disconnect the graph entirely" and "would affect a disproportionately large portion" overstate. The second is blast-radius language (Ch2:126), not centrality.
  - C16: the defender is right that the three incidents are a legitimate choice. Whether to add a 2025 npm case is D2. I checked the sources. The CISA alert of 23 September 2025 supports "a self-replicating worm … Shai-Hulud … has compromised over 500 packages" and its spread through compromised developer accounts. It does **not** mention `chalk`, `debug` or phishing, and neither does GitHub's blog of 22 September 2025. The defender's optional sentence, which cites CISA for `chalk` and `debug`, is therefore not supported by its citation. The D2-B text below names only what CISA states.
  - C17: the defender is right that the abstract may group the work differently from the contributions, and that "to estimate how readily" is the thesis's own gloss (CONTEXT.md, spectral radius). Both are right that k-core appears in both (3) and (4), and that "dependency concentration using entropy metrics and k-core decomposition" misnames Ch6. Concentration is namespace concentration (Ch6:156), degree entropy is a separate measure (Ch6:130), and the core number measures dense interconnection (Ch6:54). The blast radius and paths (Gap 2, Ch3:134) appear in neither abstract, so E10 adds them to (2).
  - Everything else both parties report, I recomputed and it agrees: 90 SBOMs; median 332 nodes; direct share 0.115; transitive-only share 0.75; τ_b 0.003 (blast, 90) and −0.004 (authority, 79); chokepoint share 0.453; dominated share 0.214; reach 0.186.
- **Three decisions are needed** (list at the end): whether Ch1 states RQ1–RQ3 explicitly (D1), whether to add the 2025 npm incident (D2), and how much of the sample framing goes into Ch1 (D3).

## Erratum

### E1 (from C1, C9) Ch1 omits dominance, paths and chokepoints, assigns impact traversal to Ch6, and calls Ch6 the "evaluation" — impact: high (objectives and contributions against what Ch5–Ch8 deliver)
- Settled facts:
  - `grep -in "domin\|chokepoint\|path"` over Ch1 matches only "build pipeline(s)" (:7, :9).
  - Ch3:10, :132–134 and Ch8:4 make dominance, and the dependents and paths shown together on the graph, the settled novelty. Ch8:16 headlines chokepoints. Ch7:130: articulation points miss a sixth of the vulnerable chokepoints.
  - Blast radius, paths and dominated set are Ch5 subsections (`sec:blast-radius` Ch5:215, `sec:dominator-tree` Ch5:236). Ch6:4 says Ch5 computed them. Ch8:28 reports them under RO2.
  - Ch1 puts impact traversal under RO3 and Contribution 3 "(\autoref{cp:network-sci})" (:42, :55). The Ch5 outline (:85) omits them. :17 calls Ch6 the "evaluation".
  - Ch8:24 says the ROs "map one-to-one". With the fix, RO2 promises what Ch8:28 reports.
- Fix:
  - Ch1:17 — "The design, implementation, and evaluation of SBOM Lens are described in \autoref{cp:system-design}, \autoref{cp:vuln-analysis}, and \autoref{cp:network-sci}, respectively, with empirical results presented in \autoref{cp:evaluation}." → "The design of SBOM Lens is described in \autoref{cp:system-design}, its vulnerability analysis in \autoref{cp:vuln-analysis} and its structural analysis in \autoref{cp:network-sci}, with empirical results presented in \autoref{cp:evaluation}."
  - Ch1:41 (RO2, final text) — "so that the analyst can immediately see which nodes carry known vulnerabilities and at what severity. (\autoref{cp:vuln-analysis})" → "so that the analyst can immediately see which nodes carry known vulnerabilities and at what severity, and, for any vulnerable component, to compute its blast radius, the dependency paths through which it enters the application, and its dominated set. (\autoref{cp:vuln-analysis})"
  - Ch1:42 (RO3, final text, with E16) — "including impact traversal to enumerate all components affected by a vulnerability in a given node, centrality and articulation point analysis to identify components with the highest structural propagation potential, and community detection to surface the modular structure of the dependency ecosystem ---" → "including centrality and articulation point analysis to identify components with the highest structural propagation potential, and community detection to surface the modular structure of the application's dependency graph ---"
  - Ch1:54 (Contribution 2, final text) — "giving the analyst an immediate picture of which nodes in the graph carry known vulnerabilities. (\autoref{cp:vuln-analysis})" → "giving the analyst an immediate picture of which nodes in the graph carry known vulnerabilities. For any vulnerable component, SBOM Lens computes its blast radius, up to five shortest dependency paths through which it enters the application, and its dominated set --- the components that the application reaches only through it. (\autoref{cp:vuln-analysis})"
  - Ch1:55 (Contribution 3, final text, with E2 and E16) — "to support analyst decision-making beyond raw CVE listings. Impact traversal identifies all components that depend --- directly or transitively --- on a given vulnerable node, making the blast radius of any vulnerability immediately visible. Centrality analysis and articulation point detection identify the components whose structural position makes them highest-priority targets for monitoring, regardless of whether they currently carry a known CVE. Community detection reveals the modular structure of the dependency ecosystem, supporting" → "to support analyst decision-making beyond raw CVE listings. Centrality analysis and articulation point detection identify the components whose structural position makes them candidates for closer monitoring, regardless of whether they currently carry a known CVE. Community detection reveals the modular structure of the application's dependency graph, supporting"
  - Ch1:71 — "articulation point detection, centrality analysis, and community detection are well-established techniques" → "dominator trees, articulation point detection, centrality analysis, and community detection are well-established techniques"
  - Ch1:85 — "the vulnerability management workflow, and the graph overlay that visually surfaces vulnerable nodes by severity band." → "the vulnerability management workflow, the graph overlay that visually surfaces vulnerable nodes by severity band, and, for each vulnerable component, its blast radius, dependency paths and dominated set."
  - The :15 and :31 parts are in E2 and E8.
- Also touches: none. The wording follows Ch8:28 ("up to five shortest dependency paths through which it enters the application") and Ch2:126 ("the components that the application reaches only through d").
- Viva answer: "The system computed dominance and paths throughout. They belong to the vulnerability analysis, RO2, which is where Ch5 describes them and Ch8 reports them. The structural measures of RO3 do not depend on a known vulnerability."

### E2 (from C2, C15) Ch1:15 and :55 state that analysts should prioritise structurally and give centrality the job of blast radius — impact: high (the claim Ch7:111 and Ch8:30 disclaim)
- Settled facts:
  - Ch7:111: τ_b ≈ 0 "does not show that the structural ranking is better". Ch8:30: the evaluation "does not show that prioritising by them leads to better remediation". The Abstract already makes no effectiveness claim (decision taken).
  - :15 "should treat as higher-priority" and :55 "highest-priority targets" state the claim as fact.
  - What a compromise "would affect" is the blast radius (Ch2:126), a reachability measure. Centrality locates components that are structurally central (Ch6:34, "a foundational library at the heart of the dependency graph"). Authority is zero for about half the components of the median graph (Ch6:34).
  - Ch6:22: an articulation point "disconnects the graph", which is weaker than "entirely".
  - "Dependency tree" is on CONTEXT.md's _Avoid_ list.
  - :27 and RO3's "principled basis" (:42) stand (see Holds).
- Fix:
  - Ch1:15 (final text for sentences 2–5, with E1, E6 and E13) — "Given a directed dependency graph, the set of all components affected by a vulnerability in component \(v\) is the set of ancestors of \(v\) in the graph, computable in linear time. Graph-theoretic centrality measures further allow the identification of components that, if compromised, would affect a disproportionately large portion of the dependency tree --- components that a security analyst should treat as higher-priority regardless of their individual CVSS score. Articulation points identify components whose removal would disconnect the graph entirely, representing single points of structural failure. These techniques are well-established in graph theory \citep{CLRS4th, Newman2018} but have not been systematically applied to SBOM dependency graphs." → "Given a directed dependency graph, the set of all components affected by a vulnerability in component \(v\) is \(v\) together with its ancestors in the graph, computable in linear time, and the dominator tree identifies the components that the application reaches only through \(v\) \citep{LengauerTarjan1979}. Graph-theoretic centrality measures further identify the components on which much of the dependency graph rests --- components that a security analyst may want to examine whatever their individual CVSS score. Articulation points identify components whose removal would split the graph, representing single points of structural failure. These techniques are well-established in graph theory \citep{CLRS4th, Newman2018} but have not, to our knowledge, been systematically applied to SBOM dependency graphs."
  - Ch1:55: the final text is in E1. It replaces "highest-priority targets for monitoring" with "candidates for closer monitoring".
  - Ch1:67, at the end of the paragraph, after "falls outside the scope of this work." add: "The evaluation therefore shows whether the structural measures carry information that severity does not, not whether prioritising by them leads to better remediation."
- Also touches: none. `LengauerTarjan1979` is already in the bib (Ch2:126). The added sentence restates Ch8:30.
- Viva answer: "The introduction motivates structural prioritisation as a hypothesis. The evaluation shows that structure and severity measure different things. Whether acting on structure improves remediation is not tested, and the thesis says so in Ch1, Ch7 and Ch8."

### E3 (from C4, C18) Ch1 never says the evaluation is npm-only and restricted to vulnerable applications, and it promises "at scale" and a workflow discussion — impact: medium (scope of every result)
- Settled facts:
  - `grep -c npm` over Ch1 gives 0. The Abstract (EN:21, PT:8), Ch7:4 and Ch8:10 say npm.
  - Ch7:20 accepts only projects with a critical vulnerability. Ch7:161: "The findings describe vulnerable, medium-sized npm applications". 90 were accepted out of 1,281 processed (Ch7:24), which is not "at scale".
  - Ch7 has no section on analyst workflows. It interprets results in single sentences (Ch7:107, :111, :128).
  - Ch7:16 makes the lockfile the property the evaluation depends on.
  - Hindsight and OSV-only are disclosed in Ch7/Ch8 (decision taken). Whether to repeat them here is D3.
- Fix:
  - Ch1:43 (RO4): the final text is in E4. It contains "against the SBOMs of real-world open-source npm applications".
  - Ch1:56 (Contribution 4, final text) — "We evaluate the framework against a dataset of real-world open-source project SBOMs, collected at scale from public repositories. Through graph-level and node-level metrics we demonstrate that the approach surfaces structural risk information that is not available through flat-inventory analysis, and we discuss the implications for security analyst workflows." → "We evaluate the framework against a dataset of SBOMs of real-world open-source npm applications, collected reproducibly from public repositories. Through graph-level and node-level metrics we demonstrate that the approach surfaces structural risk information that is not available through flat-inventory analysis, and we interpret the results from an analyst's point of view."
  - Ch1:67 (first sentence, final text per D3) — "it reports graph-level and node-level measures computed on real-world open-source project SBOMs." → see D3. The recommended form (D3-A) is "it reports graph-level and node-level measures computed on the production dependency graphs of 90 open-source npm applications, each with at least one critical known vulnerability (\autoref{sec:eval-dataset}), and its findings describe such applications." The end-of-paragraph sentence from E2 is added whatever D3 decides.
  - Ch1:69 (sentence 4) — "SBOM Lens operates on source-level SBOMs generated from package manifests;" → "SBOM Lens operates on source-level SBOMs generated from package manifests and lockfiles;"
  - Ch1:89 (final text, with E4) — "\autoref{cp:evaluation} evaluates the framework against a set of real-world open-source project SBOMs. It reports graph-level and node-level metrics, vulnerability findings, and structural analysis results, and discusses what the analysis reveals about structural risk in software supply chains." → "\autoref{cp:evaluation} evaluates the framework on the production dependency graphs of 90 open-source npm applications. It answers three research questions with graph-level and node-level metrics, reports the analysis time and the threats to validity, and discusses what the analysis reveals about structural risk in software supply chains."
  - **DECISION NEEDED** (D3): how much of the sample framing goes into :67.
- Also touches: none. "Collected reproducibly from public repositories" is Ch8:32's wording. "Production dependency graphs" is Ch7:18 and Ch8:10. The :89 text matches the sections of Ch7:4 (`sec:eval-rq1`–`sec:eval-rq3`, `sec:eval-performance`, `sec:eval-threats`).
- Viva answer: "The abstract and the introduction both say npm. The one filter that shapes every RQ, at least one critical vulnerability, is stated in the scope. The other threats are quantified in Ch7."

### E4 (from C3) Ch1 says "the research question" in the singular, and the Ch7 outline does not mention the RQs — impact: medium (the loop from introduction to conclusion)
- Settled facts:
  - Ch7:4 and :57–61 define RQ1–RQ3 "derived from the fourth research objective (\autoref{sec:objectives})". Ch8:10–16 answers them.
  - Ch1:31 "The research question is …" reads as if the thesis had one question. Ch1:89 mentions neither the RQs nor the performance and threats sections.
  - Whether Ch1 states the RQs explicitly is D1.
- Fix:
  - Ch1:31 (sentence 4, final text per D1). Under D1-A or D1-B: "The research question is not whether these capabilities can be built --- the system exists --- but what they reveal when applied to real-world open-source SBOMs, a question answered empirically in \autoref{cp:evaluation}." → "The question is not whether these capabilities can be built --- the system exists --- but what they reveal when applied to real-world open-source SBOMs, a question answered empirically in \autoref{cp:evaluation}." For D1-C, see the decision.
  - Ch1:43 (RO4, final text per D1). Under D1-A: "\item \textbf{Empirical evaluation.} To evaluate the framework against a set of real-world open-source project SBOMs and demonstrate, through graph-level and node-level metrics, that the approach surfaces structural risk information that is not available through flat-inventory analysis. (\autoref{cp:evaluation})" → "\item \textbf{Empirical evaluation.} To evaluate the framework against the SBOMs of real-world open-source npm applications and demonstrate, through graph-level and node-level metrics, that the approach surfaces structural risk information that is not available through flat-inventory analysis. The evaluation asks what the dependency graphs of these applications look like (RQ1), whether a structural view of their vulnerable components orders them differently from the severity ranking a flat inventory provides (RQ2), and how much of their vulnerability exposure is hidden from a flat inventory (RQ3). (\autoref{cp:evaluation})". Under D1-B or D1-C, use the same text without the sentence "The evaluation asks … (RQ3)."
  - Ch1:89: the final text is in E3 ("It answers three research questions …"). It applies under every D1 option.
  - **DECISION NEEDED** (D1).
- Also touches: none. The RQ wording is Ch7:4's, word for word.
- Viva answer: "The RQs operationalise RO4. The introduction names them, and Ch7 defines the measures that answer them."

### E5 (from C6) Ch1:23 says the tools have "no representation of the relationships between components", which Ch3 contradicts — impact: medium (the opening claim of the problem statement)
- Settled facts:
  - Ch3:14: Dependency-Track displays the dependency tree and can open it at the path to an affected component.
  - Ch3:18: Sunshine draws a sunburst and lists the dependents of a vulnerable component in a table. Ch1:13 says the same two paragraphs earlier.
  - Ch3:20, :132: the relationships are "discarded, displayed, or at most traversed … None computes anything over them beyond such reachability".
  - The decided novelty is narrow. Ch8:4 says "The SBOM tools surveyed in \autoref{sec:sbom-tools}".
  - "Dependency tree" for the graph model is on CONTEXT.md's _Avoid_ list.
- Fix:
  - Ch1:23 (sentence 1, final text) — "Existing SBOM analysis tooling suffers from a structural limitation that is architectural rather than incidental: it treats the dependency graph as a flat list." → "The SBOM analysis tools surveyed in \autoref{sec:sbom-tools} share a structural limitation that is architectural rather than incidental: they treat the dependency graph as a flat list."
  - Ch1:23 (sentence 3, final text) — "There is no representation of the relationships between components, no indication of how many other components depend on a given vulnerable package, and no means of determining whether a vulnerable component sits at the centre of the dependency tree or at its periphery." → "Where the relationships between components are shown --- as Dependency-Track's dependency tree, which can be opened at the path to an affected component, or as Sunshine's sunburst and its table of dependents --- they are at most traversed, never analysed: there is no means of determining whether a vulnerable component sits at the centre of the dependency graph or at its periphery."
- Also touches: none. The defender's "serve navigation, not computation" would contradict Ch1:13 and Ch3:18, because Sunshine computes the dependents. "At most traversed" is Ch3:20's wording.
- Viva answer: "The settled gap is computation over the structure, not its display. Dependency-Track and Sunshine show the structure, and Sunshine traverses it, but neither computes dominance or structural measures over it."

### E6 (from C7) The novelty and gap statements are unhedged, GUAC is called "the closest existing work", and Ch1 speaks of one gap where Ch3 has four — impact: medium (novelty framing)
- Settled facts:
  - The decided framing is narrow novelty with "to our knowledge" (Ch3:130, :138).
  - Ch3:10 places GUAC at "the graph-adjacent end of the spectrum" among tools. The nearest research is Zięba-Kozarzewski 2026 and Baird 2026 (Ch3:138).
  - Ch3:4 and :130 name four gaps (Ch3:132–138).
- Fix:
  - Ch1:15: the hedge is in E2's final text ("have not, to our knowledge, been systematically applied").
  - Ch1:29 — "The closest existing work is GUAC" → "The closest existing tool is GUAC"
  - Ch1:81 — "It identifies the gap that SBOM Lens addresses: the absence of graph-structural risk analysis in current tooling." → "It identifies four gaps that, to our knowledge, current tooling and research leave open and that SBOM Lens addresses: no graph analysis in SBOM tools, no vulnerability impact in graph terms, no structural prioritisation beyond severity, and no graph analysis at the scale of a single application."
- Also touches: none. The four names paraphrase the bold headings of Ch3:132–138.
- Viva answer: see V1.

### E7 (from C5) Ch1:9 says a transitive dependency "remains invisible until it is exploited" — impact: medium (the motivating sentence)
- Settled facts:
  - A flat inventory lists transitive components (Ch3:16, grype).
  - Ch2:66: "A flat SBOM inventory can reveal whether Log4j is present; it cannot reveal how deeply embedded it is, how many other components depend on it". Ch7:128 says the same, and the Ch8 sort (C2) settled it.
  - "Until it is exploited" has no source. `GoogleLog4j2021` is about depth in the dependency chain.
- Fix: Ch1:9 (final text) — "without a structural model of the dependency graph, the presence of a transitive dependency remains invisible until it is exploited." → "without a structural model of the dependency graph, an organisation may see that a vulnerable transitive dependency is present, but not how it entered the system or what depends on it."
- Also touches: none.
- Viva answer: "A flat inventory tells you that log4j is present. It does not tell you through which of your dependencies it arrived, or what else is exposed. That is what the graph adds."

### E8 (from C8) "An accurate and complete picture of the blast radius" contradicts the disclosed over-approximation and lower bounds — impact: medium
- Settled facts:
  - The blast radius is exact reachability over the recorded edges (`ancestors + [node_id]`, Ch5:219).
  - It over-approximates code-level exposure (Ch7:157, Ch8:45). The counts of seven SBOMs are lower bounds (Ch7:159).
  - The impact view also returns paths and the dominated set (Ch5:217, :238).
- Fix: Ch1:31 (sentence 2, final text, with E1) — "The first is \textit{impact traversal}: given any vulnerable component, the system immediately identifies all components that depend on it, at any depth, giving the analyst an accurate and complete picture of the blast radius." → "The first is \textit{impact traversal}: given any vulnerable component, the system identifies its blast radius --- the component and every component that depends on it, at any depth, as far as the SBOM records the dependencies --- together with the dependency paths through which it enters the application and its dominated set, the components that the application reaches only through it."
- Also touches: none. The definition matches CONTEXT.md and Ch2:126.
- Viva answer: "SBOM Lens computes the blast radius exactly over the dependencies the SBOM records. How faithful the SBOM is, and whether the vulnerable code executes, are separate limitations, stated in Ch7 and Ch8."

### E9 (from C10) Ch1:69 says SBOM comparison needs a temporal model "not present", but a pairwise comparison view exists — impact: medium-low
- Settled facts:
  - `GET /compare` diffs the node and edge sets of two stored SBOMs (`backend/routers/delta.py@28113e7`). Ch4:22 lists the view as present but not evaluated.
  - Ch8:69 (as corrected): "extending the pairwise SBOM comparison view outside the scope of this thesis (\autoref{sec:architecture-overview})".
- Fix: Ch1:69 (sentence 2, final text) — "is a distinct analytical problem that would require a temporal data model not present in the current system." → "is a distinct analytical problem: SBOM Lens includes a pairwise comparison view (\autoref{sec:architecture-overview}), which is not evaluated, and tracking a project across versions would require a temporal data model not present in the current system."
- Also touches: none.
- Viva answer: "A set difference between two SBOMs exists and was not evaluated. Tracking one project's history would need the temporal model, and that is the part that is absent."

### E10 (from C11, C17) The Portuguese abstract says something different from the English, and both misname the Ch6 measures and omit the blast radius — impact: medium (the text read first)
- Settled facts:
  - PT:4 makes the *formats* lack *tools* that produce *actionable* intelligence. EN:17 says that existing tooling treats SBOMs as flat inventories. Ch1:67 excludes any claim that the output is actionable.
  - PT:6 (2): "incluindo o cálculo de scores CVSS v3.1" attaches the computation to OSV and uses the anglicism "scores".
  - PT:6 (3) has no counterpart to EN "via articulation point and dominance analysis".
  - k-core appears in both (3) and (4), in EN and PT. "Dependency concentration using entropy metrics and k-core" misnames namespace concentration (Ch6:156), degree entropy (Ch6:130) and the core number (Ch6:54).
  - Blast radius and paths (Gap 2, Ch3:134; the basis of RQ3) appear in neither abstract.
  - The rest of the PT text is AO90 European Portuguese ("deteção", "espetral"). The fixes keep that.
- Fix:
  - PT:4 — "mas os formatos existentes carecem de ferramentas analíticas que transformem dados de inventário em inteligência de risco acionável." → "mas as ferramentas existentes tratam os SBOMs, em grande medida, como inventários planos e não como grafos estruturados passíveis de análise rigorosa."
  - EN:19 (2) — "with full CVSS v3.1 base score computation;" → "with full CVSS v3.1 base score computation and, for each vulnerable component, its blast radius and the dependency paths through which it enters the application;"
  - PT:6 (2) — "que agrega os GitHub Security Advisories e outras fontes, incluindo o cálculo de scores CVSS v3.1;" → "que agrega os GitHub Security Advisories e outras fontes, com o cálculo integral da pontuação base CVSS v3.1 e, para cada componente vulnerável, o seu raio de impacto e os caminhos de dependência pelos quais entra na aplicação;"
  - EN:19 (3) — "which identifies single points of failure via articulation point and dominance analysis, circular dependencies, version conflicts, and dependency concentration using entropy metrics and k-core decomposition;" → "which identifies single points of failure via articulation point and dominance analysis, circular dependencies and version conflicts, and measures namespace concentration and degree entropy;"
  - PT:6 (3) — "que identifica pontos de falha única, dependências circulares, conflitos de versão e concentração de dependências através de métricas de entropia e decomposição em núcleos;" → "que identifica pontos de falha única (através da análise de pontos de articulação e de dominância), dependências circulares e conflitos de versão, e mede a concentração por espaço de nomes e a entropia da distribuição de graus;"
  - (4) is unchanged in both languages. EN still says "k-core decomposition" and PT "decomposição k-core", now once each.
- Also touches: none. The results paragraphs (EN:21, PT:8) are unchanged and already say the same thing.
- Viva answer: "The abstract lists the measures Ch6 computes, named as Ch6 names them. The Portuguese is a translation of the English."

### E11 (from C13) "Attack surface … orders of magnitude larger than the first-party code" is contradicted by the cited report — impact: low-medium (a quantitative claim in the first paragraph)
- Settled facts:
  - OSSRA 2024: 96 % of 1,067 codebases contain open source, 77 % of the scanned code is open source (about 3.3 times the first-party code), and 526 components on average. The 96 % and "over 500" at :7 are correct.
  - The `per_sbom.csv` median direct share of 0.115 gives about 8.7 components per declared dependency, still below an order of magnitude.
  - OSSRA's component count already includes transitive components, so "introduces its own transitive closure" counts them twice.
- Fix: Ch1:7 — "Each of these components introduces its own transitive closure of further dependencies, forming a supply chain whose attack surface is orders of magnitude larger than the first-party code that developers directly author." → "Many of these components arrive as dependencies of other components, forming a supply chain whose code is several times larger than the first-party code that developers directly author."
- Also touches: none. Ch2:64 ("89 % of the components enter transitively") supports "many".
- Viva answer: "The point is that most of the code an application ships is not written by its developers. That holds at a factor of three in OSSRA and about eight in my npm sample."

### E12 (from C12) Ch1:11 glosses the CRA's SBOM as "all software components" and says it "has transitioned" to a legal requirement — impact: low (residue of the Ch2 edit; tense and gloss only)
- Settled facts:
  - CRA Art. 3(39) defines the SBOM as "a formal record containing details and supply chain relationships of components". Annex I Part II(1) requires "at the very least the top-level dependencies" (Ch2:20).
  - The SBOM obligations apply from December 2027 (Ch1:11 itself, Ch2:88). Today is October 2026. Ch2:20 says "becoming".
  - The growth forecast has no source.
  - The dates and the EO/OMB wording stay as decided.
- Fix:
  - Ch1:11 — "--- a machine-readable inventory of all software components and their dependency relationships ---" → "--- a formal record of a product's components and their supply chain relationships, which must cover at least its top-level dependencies ---"
  - Ch1:11 — "The SBOM has thus transitioned from a niche software engineering practice to a legal requirement in the EU and an expected practice in US federal procurement, and the volume of SBOMs in circulation is set to grow substantially in coming years." → "The SBOM is thus becoming a legal requirement in the EU and remains an expected practice in US federal procurement, and the volume of SBOMs in circulation is likely to grow."
- Also touches: none. The wording is Ch2:20's ("becoming a legal requirement in the EU and remain an expected practice in US federal procurement").

### E13 (from C14) Ch1 uses "upstream" in both senses within four paragraphs, and :15 defines the blast radius without \(v\) — impact: low
- Settled facts:
  - :9 uses the supply-chain sense (Ch2:62). :13 and :25 use Ch5:219's graph sense, in which consumers are "upstream".
  - Ch2:126, Ch5:219 and CONTEXT.md define the blast radius as \(v\) together with its ancestors.
- Fix:
  - Ch1:13 — "the largest number of upstream dependents" → "the largest number of dependents"
  - Ch1:25 — "traversing the dependency graph upstream from the vulnerable component to find all packages that depend on it" → "traversing the dependency graph from the vulnerable component back to all packages that depend on it"
  - Ch1:15: the definition is in E2's final text ("is \(v\) together with its ancestors in the graph").
- Also touches: none. Ch5:219 keeps its declared convention.

### E14 (from C16) "A succession of high-profile attacks" includes a vulnerability, and three incidents "establish" a trend — impact: low
- Settled facts:
  - log4shell was a vulnerability, not an attack.
  - Three cases cannot establish that attacks are "increasing in sophistication".
  - Whether to add the 2025 npm worm is D2. The CISA alert supports Shai-Hulud and "over 500 packages". It does not mention `chalk` or `debug` (see Summary).
- Fix:
  - Ch1:9 — "by a succession of high-profile attacks" → "by a succession of high-profile incidents"
  - Ch1:9 (last sentence, final text per D2) — "Together, these three incidents establish that supply chain attacks are diverse in method, severe in potential impact, and increasing in sophistication." → see D2. Recommended (D2-B): "In September 2025, a self-replicating worm known as Shai-Hulud spread through the npm registry, using compromised maintainer accounts to publish malicious versions of more than 500 packages \citep{CISA2025npm}. Together, these four incidents illustrate that supply chain risks are diverse in method and severe in potential impact."
  - **DECISION NEEDED** (D2).
  - New BibTeX (D2-B only; key `CISA2025npm` is not in `Bibliography.bib`; checked against the CISA page today):
    ```
    @Misc{CISA2025npm,
        author      = {{Cybersecurity and Infrastructure Security Agency}},
        title       = {Widespread Supply Chain Compromise Impacting {npm} Ecosystem},
        year        = {2025},
        month       = sep,
        note        = {Alert, published 23 September 2025},
        url         = {https://www.cisa.gov/news-events/alerts/2025/09/23/widespread-supply-chain-compromise-impacting-npm-ecosystem},
        urldate     = {2026-10-12},
    }
    ```
- Also touches: none. Ch3:4 ("a pair of supply chain incidents") is about Ch3's case studies and is unaffected. Ch2:76 already describes the compromised-maintainer vector.
- Viva answer: "The incidents illustrate the kinds of supply chain risk. They are not evidence of a trend. The 2025 npm worm is the case closest to the evaluated ecosystem, and namespace concentration (Ch6) measures how much of a graph depends on one publisher."

### E15 (from D-new1) The format scope justifies CycloneDX by a "richest representation" claim that Ch2 contradicts — impact: low
- Settled facts:
  - Ch2:30: SPDX 2.3 defines 45 relationship types. Ch2:36 gives the corrected reason: "CycloneDX states every dependency in a single directed form, and it is the format produced by the generator used in the evaluation".
  - CycloneDX is OWASP's only SBOM format, so "within the OWASP ecosystem" says nothing.
- Fix: Ch1:65 — "CycloneDX JSON provides the richest representation of component dependency relationships within the OWASP ecosystem, and its structured" → "CycloneDX states every dependency in a single directed form, it is the format produced by the generator used in the evaluation, and its structured"
- Also touches: none.

### E16 (from D-new2) Community detection is said to reveal "the dependency ecosystem", the scale the thesis says it does not work at — impact: low
- Settled facts: Ch3:54 and :138 separate the ecosystem scale (prior work) from one application's graph. Ch6:76 runs Louvain on one SBOM's graph.
- Fix: Ch1:42 and :55, "the modular structure of the dependency ecosystem" → "the modular structure of the application's dependency graph". Both are contained in E1's final texts.
- Also touches: none.

## Viva prep

### V1 (from C7) Why GUAC, and not the 2026 SBOM-graph papers, as the closest work?
- Likely question: "You call GUAC the closest tool. Zięba-Kozarzewski and Baird study SBOMs as graphs in 2026. Aren't they closer?"
- Answer: GUAC is the closest *tool*. It builds graphs across many SBOMs and searches dependents, but it computes no dominance or structural measures (Ch3:10). The 2026 papers are research. One asks whether SBOMs declare their dependency edges at all. The other learns to predict vulnerable components over enriched SBOM graphs. Neither applies network-science measures to one application's graph, or asks whether ecosystem-scale properties hold there (Ch3:138). The gap is stated "to our knowledge".

## Holds
- No whole item. Five sub-points are rebutted and need no action:
  - C2, :27 and RO3 (:42): "no principled basis for deciding where to act first" describes the flat-list problem and claims no solution. RO3's "principled basis" is the objective that Ch8:24 accepts with the qualification.
  - C8, exactness: the blast radius is exact over the recorded edges. The fix keeps that and adds what it is relative to.
  - C12, gloss: the relational gloss follows CRA Art. 3(39). Only "all" and the tense are changed.
  - C14, Ch5's convention: Ch5:219 declares the graph sense of "upstream" on purpose, and it stays.
  - C17, grouping and "to estimate": an abstract may group the work differently from the contributions, and the third paragraph covers the evaluation. "To estimate how readily a compromise could spread" is the thesis's own gloss (CONTEXT.md).

## Decisions needed
1. **Research questions in Ch1 (E4/C3).**
   - (A) State RQ1–RQ3 in RO4 (:43), in Ch7:4's wording, and name them in the Ch7 outline (:89). This closes the loop to Ch8:12–16 at the objectives that Ch8:24 verifies.
   - (B) Name them only in the outline (:89: "It answers three research questions …") and leave Ch7:4 to define them, as now.
   - (C) State them in the problem statement (:31): "The question is not whether these capabilities can be built --- the system exists --- but what they reveal when applied to real-world open-source SBOMs: what their dependency graphs look like (RQ1), whether a structural view of their vulnerable components orders them differently from the severity ranking a flat inventory provides (RQ2), and how much of their vulnerability exposure is hidden from a flat inventory (RQ3), questions answered empirically in \autoref{cp:evaluation}." RO4 then has no RQ sentence. This fits less well with Ch7:4's "derived from the fourth research objective".
   - Recommendation: (A).
2. **The 2025 npm incident (E14/C16).**
   - (A) No new incident. The last sentence of :9 becomes "Together, these three incidents illustrate that supply chain risks are diverse in method and severe in potential impact."
   - (B) Add the Shai-Hulud sentence with the new `CISA2025npm` entry, and "four incidents" in the closing sentence (text in E14). This ties the motivation to the npm evaluation and brings the record past March 2024. The source supports only what the sentence says. Do not add `chalk`/`debug` without a further source.
   - Recommendation: (B). It is one sentence and one verified entry, and it is the only incident in the evaluated ecosystem.
3. **Sample framing in Ch1 (E3/C4).** First sentence of :67 (the E2 sentence is added at the end in every option).
   - (A) "it reports graph-level and node-level measures computed on the production dependency graphs of 90 open-source npm applications, each with at least one critical known vulnerability (\autoref{sec:eval-dataset}), and its findings describe such applications." This states the ecosystem and the one filter that shapes every RQ.
   - (B) Text (A), followed by "Vulnerability data reflects OSV on 30 September 2026, so it includes advisories published after each release (\autoref{sec:eval-threats})." This repeats in Ch1 the hindsight already disclosed in Ch7:159 and Ch8:53.
   - (C) "it reports graph-level and node-level measures computed on the production dependency graphs of open-source npm applications (\autoref{sec:eval-dataset}), whose selection and its limits are described in \autoref{sec:eval-threats}." This is the shortest, and it leaves the critical-vulnerability filter to Ch7.
   - Recommendation: (A).
