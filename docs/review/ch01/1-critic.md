# Chapter 1 and Abstract — Critic

What I checked. I read Ch1 (`Chapters/01-Introduction.tex`, 91 lines) and both abstracts (`Chapters/00-Abstract.tex`, PT :4–:10, EN :17–:23) line by line. I compared them with the current text of Ch2–Ch8 and with the sorted reports `docs/review/ch0{2..8}/3-sorted.md`. I recomputed the Abstract's numbers from `sbom-lens-eval/analysis/out/per_sbom.csv` using system Python with csv and statistics (pandas is not installed). Results: 90 rows; median transitive-only share 0.75 (pooled 1,997 / 2,668 = 0.749); median τ_b (blast) 0.003; median τ_b (authority) −0.004 over 79 SBOMs; median nodes 332. Ch1 itself reports no evaluation number. I checked system claims against `git -C /home/inasc/projects/sbom-lens show 28113e7:<path>` (marked `@28113e7`). I checked the bib entries behind every Ch1 citation. I verified external facts with web search: OSSRA 2024 (96 % of codebases contain open source, 77 % of scanned code is open source) and the September 2025 npm compromises. I did not reopen the decisions already taken: inclusive blast radius, the 28113e7 scope, the narrow novelty framing, heavy-tail/small-world wording, HITS on 79 graphs, "essentially unrelated", hindsight disclosure, the regulation wording at Ch1:11, OSV-only data, the Garcia2025 sentence at Ch1:29, and repository availability.

Both abstracts are consistent with Ch7 and Ch8 on every number. Ch1 has a different problem. It was written before the system and the evaluation took their final shape, and only its tool and regulation sentences were patched in the earlier reviews. What it promises differs from what the corrected chapters deliver, in both directions. It leaves out the thesis's distinctive deliverables: dominance, chokepoints, dependency paths, the three research questions, and the npm-only evaluation. It also still asserts that structural priority is better, which Ch7 and Ch8 explicitly disclaim. In the Portuguese abstract, the first paragraph and one clause are not translations of the English.

## C1. Ch1 never mentions dominance, chokepoints or dependency paths, which are the thesis's stated novelty and the RQ3 headline
- Type: inconsistency / omission
- Origin: pre-existing
- Severity: high
- Thesis: Chapters/01-Introduction.tex:42 — RO3 "including impact traversal …, centrality and articulation point analysis …, and community detection"; :55 — Contribution 3 lists impact traversal, centrality, articulation points and community detection; :71 — "The contribution lies in their systematic application to SBOM dependency graphs as a coherent risk assessment framework"
- Evidence: `grep -i "domin\|chokepoint\|path"` over Ch1 finds no match. Ch3:10 and :134 frame the novelty narrowly: GUAC "computes no dominance or structural measures". Gap 2: "None shows the dependents and the paths together on the dependency graph, and none reports which components could not be reached without it". SBOM Lens "computes … its blast radius, the dependency paths that lead to it, and its dominated set". Ch3:37 (Tab. 3.1) lists "dominated sets". Ch8:4: SBOM Lens "computes dominance and structural measures". Ch8:16 (RQ3): chokepoints are 45 % of vulnerable components and dominate 21 % of the graph. Ch8:28 places blast radius, "up to five shortest dependency paths" and the "dominated set" under RO2. Ch8:35 lists "blast radius … from its dominated set, chokepoints from articulation points" as a contribution. Ch7:130: articulation points miss a sixth of the vulnerable chokepoints. The English abstract (:19) does name "dominance analysis".
- Why it matters: The one capability that Ch3 says distinguishes SBOM Lens from Sunshine, Dependency-Track and GUAC is absent from the objectives, the contributions and the novelty statement. Ch1 instead presents articulation points (:15, :55) as the single-point-of-failure measure, which is the measure Ch7 shows to be the weaker one. Ch8:24 says that each RO "was met" and maps "one-to-one" to Ch1. The reader cannot check that, because Ch8's RO2 covers deliverables that Ch1's RO2 never promised.

## C2. Ch1 still asserts that structural position should drive priority, which Ch7 and Ch8 explicitly do not show
- Type: overstated
- Origin: pre-existing
- Severity: high
- Thesis: Chapters/01-Introduction.tex:15 — "components that a security analyst should treat as higher-priority regardless of their individual CVSS score"; :27 — "the analyst has no principled basis for deciding where to act first"; :55 — "identify the components whose structural position makes them highest-priority targets for monitoring"; :42 — RO3 "so as to give the analyst a principled basis for prioritising remediation beyond raw CVSS scores"
- Evidence: Ch7:111: "A τ_b near zero does not show that the structural ranking is better than the severity ranking, or the reverse; it shows that the two measure different things". Ch8:30: "it does not show that prioritising by them leads to better remediation, which would need exploitation data or analyst outcomes". Ch8:24: RO3 is met only "with the qualification stated for RO3". The Ch7 sort (E2) removed the same claim from the Abstract, and the Abstract now says "a basis for prioritising it". Ch6:33–34: about half of the components of the median graph have HITS authority zero, so "centrality analysis" cannot single out "highest-priority targets" across the graph.
- Why it matters: The decision for the Abstract (no effectiveness claim for prioritisation) was not carried into Ch1. Ch1:15 and :55 state as fact that analysts *should* prioritise structurally. That is the very claim the conclusion says the evidence cannot support, and an examiner who reads Ch1 and then Ch8:30 will see the mismatch. RO3's "principled basis" can stand, since Ch8 accepts it with the qualification. The normative sentences at :15 and :55 are the problem, and Ch1 could say in the RO3 statement that whether such prioritisation improves remediation is not tested.

## C3. Ch1 states no research questions; Ch7 and Ch8 are organised around RQ1–RQ3
- Type: inconsistency / omission
- Origin: pre-existing
- Severity: medium
- Thesis: Chapters/01-Introduction.tex:31 — "The research question is not whether these capabilities can be built --- the system exists --- but what they reveal when applied to real-world open-source SBOMs, a question answered empirically in \autoref{cp:evaluation}"; :89 — the Ch7 outline: "It reports graph-level and node-level metrics, vulnerability findings, and structural analysis results"
- Evidence: Ch7:4 introduces "three research questions, derived from the fourth research objective (\autoref{sec:objectives})", and Ch7:57–61 defines RQ1 (graph profile), RQ2 (structural vs severity ranking) and RQ3 (hidden exposure). Ch8:10 says the evaluation "answered three research questions", and Ch8:12–16 is structured by RQ. Ch1's RO4 (:43) says only "demonstrate, through graph-level and node-level metrics". The Ch7 outline at :89 does not mention the RQs, the performance section (Ch7:137) or the threats to validity (Ch7:154).
- Why it matters: The introduction is where an examiner expects the RQs. Ch8 answers three questions that the reader has never been asked. Stating RQ1–RQ3 in one sentence under RO4 (or in :31) and in the Ch7 outline would close the loop.

## C4. The scope section does not say the evaluation is npm-only, restricted to vulnerable applications, OSV-only, or read with hindsight
- Type: omission / overstated
- Origin: pre-existing
- Severity: medium
- Thesis: Chapters/01-Introduction.tex:63 — "This section defines the explicit boundaries of the work to prevent misreading of the claims made in subsequent chapters"; :43 — "evaluate the framework against a set of real-world open-source project SBOMs"; :56 — "a dataset of real-world open-source project SBOMs, collected at scale from public repositories"; :67 — "computed on real-world open-source project SBOMs"; :89 — "a set of real-world open-source project SBOMs"
- Evidence: `grep -c npm Chapters/01-Introduction.tex` returns 0. Ch7:4: "90 real npm applications". Ch7:20: only projects with at least one critical vulnerability are accepted. Ch7:161: "The findings describe vulnerable, medium-sized npm applications". Ch7:159: 93 % of findings post-date the release. Ch8:41 and :53 list OSV-only data and the sample filters as the main limitations. The Abstract (EN:21, PT:8) already says "npm". "At scale" (:56): 90 SBOMs accepted from 1,281 candidates processed (Ch7:24).
- Why it matters: Ch1 claims a section that "prevents misreading". Yet Ch1 writes "real-world open-source project SBOMs" four times, which a reader takes as cross-ecosystem, and never mentions the selection condition that every application has a critical vulnerability, although that condition shapes all three RQ results. One sentence in the "Evaluation methodology" paragraph would be enough: npm, 50–1,000 components, at least one critical finding, OSV on 30 September 2026. "At scale" overstates 90 applications.

## C5. Ch1:9 says a transitive dependency is "invisible" without a graph model, but a flat inventory lists transitive components; what it hides is how they entered
- Type: overstated / inconsistency
- Origin: pre-existing
- Severity: medium
- Thesis: Chapters/01-Introduction.tex:9 — "without a structural model of the dependency graph, the presence of a transitive dependency remains invisible until it is exploited"
- Evidence: Ch3:16: grype "reports the known vulnerabilities of every package it finds", and syft generates the full SBOM. Both list transitive packages. Ch7:128: "one who sees the vulnerable components in a flat list still has to work out, for each, how it entered the application --- the dependency path SBOM Lens shows directly". The Ch8 sort (C2) made the same correction for Ch8:16: "A flat list does show those components. What it hides is how they entered". The `GoogleLog4j2021` source cited at :9 is about how deep log4j sat in dependency chains, not about it being unlisted.
- Why it matters: This is the motivating sentence that links log4shell to the thesis, and a reader who knows SCA tools will dispute it, since any SCA tool lists log4j-core. Ch7 states what a graph adds: the path, the depth and the dependents. "Until it is exploited" is also unsupported.

## C6. Ch1:23 says the surveyed tools have "no representation of the relationships between components", but Ch3 shows that Dependency-Track and Sunshine display them
- Type: inconsistency
- Origin: pre-existing (survived the Ch3 E2 edits, which changed :13, :25 and :29 only)
- Severity: medium
- Thesis: Chapters/01-Introduction.tex:23 — "Existing SBOM analysis tooling suffers from a structural limitation that is architectural rather than incidental: it treats the dependency graph as a flat list. … There is no representation of the relationships between components, no indication of how many other components depend on a given vulnerable package"; :13 — "these tools report the affected component largely in isolation"
- Evidence: Ch3:14: Dependency-Track "can display the dependency tree recorded in a CycloneDX SBOM … it can be opened at the path to an affected component". Tab. 3.1 (Ch3:32): "Dependency tree view". Ch3:18: Sunshine renders "components, dependencies and vulnerabilities as an interactive sunburst" and "lists those components [that depend on it] in a table". This is "an indication of how many other components depend on a given vulnerable package". Ch3:132 (Gap 1) states the correct line: dependency relationships are "discarded, displayed as a tree, or carried along unchanged, and at most traversed …; no surveyed tool computes structural measures over them". Ch1:13 and :25 mention Sunshine's dependents table but never Dependency-Track's path view, which is one of the settled novelty facts.
- Why it matters: The problem statement opens with a universal claim that Ch3, the thesis's own survey, contradicts two chapters later. The settled gap is "no computation over the structure", not "no representation of it".

## C7. Novelty and gap statements are unhedged, and GUAC is called "the closest existing work" although Ch3 cites closer SBOM-graph studies
- Type: overstated / inconsistency
- Origin: pre-existing
- Severity: medium
- Thesis: Chapters/01-Introduction.tex:15 — "These techniques are well-established in graph theory … but have not been systematically applied to SBOM dependency graphs"; :29 — "The closest existing work is GUAC"; :81 — "It identifies the gap that SBOM Lens addresses: the absence of graph-structural risk analysis in current tooling"
- Evidence: Ch3:130: "four capabilities that, *to our knowledge*, current tooling and the research literature do not provide". Ch3:138 (Gap 4): "has not, *to our knowledge*, been systematically analysed with these methods. Recent work has begun to study SBOMs as graphs --- … \citep{ZiebaKozarzewski2026}, and learned prediction of vulnerable components over enriched SBOM graphs \citep{Baird2026}". Ch3:136 names OSV-Scanner's guided remediation, which weighs dependency depth, as "the nearest exception" for prioritisation. Ch3:130 identifies four gaps, while Ch1:81 speaks of "the gap". The decision on the Ch3 framing requires the gap claims to be hedged "to our knowledge".
- Why it matters: Ch1 makes the strongest form of the novelty claim, without the hedge, and leaves out the two 2026 papers that the corrected Ch3 treats as the nearest academic work. The decision on the Ch3 framing applies to Ch1 too.

## C8. "Accurate and complete picture of the blast radius" contradicts the disclosed over-approximation and lower bounds
- Type: overstated
- Origin: pre-existing
- Severity: medium
- Thesis: Chapters/01-Introduction.tex:31 — "the system immediately identifies all components that depend on it, at any depth, giving the analyst an accurate and complete picture of the blast radius"
- Evidence: Ch8:45: "may never execute the vulnerable code …; exposure is therefore over-approximated". Ch7:157: "blast radius and reach over-approximate actual exposure". Ch7:159: generator errors; in seven SBOMs "the size, reach and vulnerability counts of those graphs are lower bounds". Ch8:55: vulnerability records are fetched only at upload and not refreshed. The blast radius is complete only with respect to the edges recorded in the SBOM.
- Why it matters: "Accurate and complete" is a correctness claim that the thesis's own limitations section refutes. "Exact with respect to the dependency relationships the SBOM records" is what the system delivers.

## C9. Chapter mapping errors: Ch6 is called the "evaluation", and impact traversal is assigned to Ch6 although it is in Ch5
- Type: wrong
- Origin: pre-existing
- Severity: medium
- Thesis: Chapters/01-Introduction.tex:17 — "The design, implementation, and evaluation of SBOM Lens are described in \autoref{cp:system-design}, \autoref{cp:vuln-analysis}, and \autoref{cp:network-sci}, respectively, with empirical results presented in \autoref{cp:evaluation}"; :42 and :55 — impact traversal under RO3 / Contribution 3 "(\autoref{cp:network-sci})"; :85 — the Ch5 outline lists OSV, CVSS, the workflow and "the graph overlay", with no blast radius, paths or dominator tree
- Evidence: Ch5 contains `\subsection{Blast Radius Analysis}` (`sec:blast-radius`, Ch5:214) and `\subsection{Dominator Tree Analysis}` (Ch5:235). Ch6:4 opens with "\autoref{cp:vuln-analysis} … measured … its blast radius … and … its dominated set. This chapter turns to risk that does not depend on any particular vulnerability being known". Ch8:28 maps these features to RO2/Ch5. Ch6 is "Structural Risk and Graph Analysis" and contains no evaluation, which is Ch7. As a result, the outline paragraphs (:79–:91) never mention blast radius, paths or dominance in any chapter.
- Why it matters: Ch1:37 promises that "each objective maps directly to one of the implementation or evaluation chapters" so that Ch8 can verify them. As written, RO3's first deliverable points to a chapter that does not contain it, and Ch8 silently remaps it to RO2. At :17, "design, implementation, and evaluation … respectively" mislabels Ch5 and Ch6.

## C10. Ch1:69 says SBOM comparison "would require a temporal data model not present", but the evaluated system has a pairwise comparison view that Ch4 and Ch8 acknowledge
- Type: inconsistency
- Origin: pre-existing
- Severity: medium
- Thesis: Chapters/01-Introduction.tex:69 — "SBOM comparison and delta analysis --- the comparison of two SBOMs for the same project across time to detect dependency changes --- is a distinct analytical problem that would require a temporal data model not present in the current system"
- Evidence: `backend/routers/delta.py:14-30 @28113e7`: `GET /compare?before=&after=` diffs the node and edge sets of two stored SBOMs. `src/layouts/sbom-comparison/index.js`, `src/hooks/useDelta.js` and `useComparisonAnalysis.js` exist at 28113e7. Ch4:22: "modules not evaluated in this thesis --- a Topology Optimiser … and a Comparison/Delta analysis view". Ch8:69: "extending the pairwise SBOM comparison view outside the scope of this thesis (\autoref{sec:architecture-overview})". Ch1:69 treats the Topology Optimiser as present but not evaluated, in the same paragraph, yet presents comparison as absent.
- Why it matters: The scope boundary is fine, but its justification is false as written, since two SBOMs can be compared for dependency changes without a temporal model. The Ch8 sort fixed Ch8 and left Ch1. The fix is the Optimiser treatment: a pairwise comparison view exists, is not evaluated, and per-application history is out of scope.

## C11. The Portuguese abstract's first paragraph makes a different claim from the English one, and (3) drops "articulation point and dominance analysis"
- Type: inconsistency (translation)
- Origin: pre-existing
- Severity: medium
- Thesis: Chapters/00-Abstract.tex:4 — "mas os formatos existentes carecem de ferramentas analíticas que transformem dados de inventário em inteligência de risco acionável"; :6 — "(3) a avaliação de risco estrutural, que identifica pontos de falha única, dependências circulares, …"; :6 — "incluindo o cálculo de scores CVSS v3.1"
- Evidence: EN :17: "yet existing tooling largely treats SBOMs as flat inventories rather than structured graphs amenable to rigorous analysis". The PT text says that the *formats* lack analytical *tools* producing *actionable* intelligence. This is a different subject and adds "acionável", although Ch1:67 explicitly excludes "any assessment of whether security analysts find the tool's output actionable". EN :19: "identifies single points of failure via articulation point and dominance analysis". PT omits the "via …" clause, so the only mention of dominance in either abstract disappears from the Portuguese version (see C1). EN "full CVSS v3.1 base score computation" becomes PT "cálculo de scores CVSS v3.1", and the "incluindo" makes the clause read as if OSV computes the scores.
- Why it matters: The two abstracts must make the same claims. Suggested PT for :4: "…, mas as ferramentas existentes tratam-nos, em grande medida, como inventários planos e não como grafos estruturados passíveis de análise rigorosa." For (3): "…, que identifica pontos de falha única, através da análise de pontos de articulação e de dominância, …". For (2): "…, com o cálculo integral da pontuação base CVSS v3.1". "Scores" is an anglicism; PT-PT would use "pontuações".

## C12. The CRA sentence in Ch1:11 glosses the SBOM as covering "all software components and their dependency relationships" and says it "has transitioned" to a legal requirement; Ch2 says "top-level dependencies" and "becoming"
- Type: inconsistency
- Origin: pre-existing (residue of the Ch2 E1 edit; the corrected dates are not reopened)
- Severity: low
- Thesis: Chapters/01-Introduction.tex:11 — "The CRA explicitly names the SBOM --- a machine-readable inventory of all software components and their dependency relationships --- as a key instrument …"; "The SBOM has thus transitioned from a niche software engineering practice to a legal requirement in the EU … and the volume of SBOMs in circulation is set to grow substantially in coming years"
- Evidence: Ch2:21: the CRA requires an SBOM "covering at least the top-level dependencies of each product from December 2027 … SBOMs are therefore *becoming* a legal requirement … An SBOM that lists only top-level dependencies, however, does not record the dependency graph that SBOM Lens analyses". Ch2:88: the SBOM requirement applies "from December 2027". Today, October 2026, it is not yet in application. The growth forecast has no source.
- Why it matters: Placed next to "the CRA explicitly names the SBOM", the gloss implies that the CRA mandates exactly the full-graph SBOM SBOM Lens needs, which is the inference Ch2 now warns against. Changing "has thus transitioned" to "is becoming" and adding "covering at least the top-level dependencies" aligns the two chapters without reopening the facts.

## C13. "Attack surface … orders of magnitude larger than the first-party code" is contradicted by the cited report
- Type: unsupported
- Origin: pre-existing
- Severity: low
- Thesis: Chapters/01-Introduction.tex:7 — "forming a supply chain whose attack surface is orders of magnitude larger than the first-party code that developers directly author"
- Evidence: The cited Synopsys 2024 OSSRA (web search, synopsys.com OSSRA pages) reports 96 % of 1,067 codebases containing open source, and 77 % of all scanned code originating from open source. That is about 3.3 times the first-party code, not orders of magnitude. The "96 %" and "over 500 components" (526 on average) figures at :7 are correct. The Ch7 data give a component ratio of about 8:1 (median direct share 11 %, `per_sbom.csv`), still less than one order of magnitude. The bib entry `Synopsys2024OSSRA` has no URL. Synopsys's application security business has since been renamed Black Duck, which matters only if the URL is added.
- Why it matters: This is a quantitative claim in the first paragraph, and it is refuted by the source cited in the same sentence. "Many times larger" is enough.

## C14. Ch1 uses "upstream" in opposite senses, and its blast-radius definitions are dependents-only
- Type: inconsistency
- Origin: pre-existing
- Severity: low
- Thesis: Chapters/01-Introduction.tex:9 — "a single trusted upstream vendor can become an unwitting vector for cascading compromise across an entire downstream ecosystem"; :13 — "the largest number of upstream dependents"; :25 — "traversing the dependency graph upstream from the vulnerable component to find all packages that depend on it"; :15 — "the set of all components affected by a vulnerability in v is the set of ancestors of v"; :31 — "identifies all components that depend on it … the blast radius"
- Evidence: Ch2:62: "each upstream dependency is a supplier whose code is inherited by all downstream" consumers. This is the standard sense, and Ch1:9 uses it. Ch5:219 deliberately calls consumers "upstream". Ch5:219: the blast radius is `ancestors + [node_id]`, "B together with these consumers". That is the settled inclusive definition, and Ch1:15/:31 give the exclusive one.
- Why it matters: Within two paragraphs, "upstream" first means the supplier and then the consumer. With "ancestors" alone, Ch1 defines the blast radius differently from Ch5 and Ch7. Ch1:15 should add "together with v itself", and the direction words should follow Ch2's convention.

## C15. Smaller overstatements in Ch1:15 about articulation points and centrality
- Type: overstated
- Origin: pre-existing
- Severity: low
- Thesis: Chapters/01-Introduction.tex:15 — "Articulation points identify components whose removal would disconnect the graph entirely, representing single points of structural failure"; "Graph-theoretic centrality measures further allow the identification of components that, if compromised, would affect a disproportionately large portion of the dependency tree"
- Evidence: Ch6:21: an articulation point's removal "disconnects the graph", and it is computed on the undirected projection, so the graph splits but is not disconnected "entirely". Ch6:25: articulation points are not chokepoints, and Ch7:130 shows they miss a sixth of them. What a compromise would "affect" is the blast radius (Ch5), which is a reachability measure, not a centrality measure. HITS authority is zero for about half the components (Ch6:33), and the spectral score is undirected and "does not distinguish a component that many others depend on from one that depends on many others" (Ch6:50). "Dependency tree" contradicts the graph model, and CONTEXT.md lists it under "_Avoid_".
- Why it matters: This is the paragraph that introduces the method, and it attributes to centrality the job that blast radius and dominance do.

## C16. Motivation: no npm incident, although the evaluation is npm-only and the largest recent supply chain attacks were on npm; "increasing in sophistication" rests on three cases
- Type: out of date / unsupported
- Origin: pre-existing
- Severity: low
- Thesis: Chapters/01-Introduction.tex:9 — "Together, these three incidents establish that supply chain attacks are diverse in method, severe in potential impact, and increasing in sophistication"
- Evidence: The three incidents are SolarWinds (a build system), log4shell (a vulnerability, not an attack; Ch3 sort E15 flagged it as optional) and xz (a C library). None is in npm. In September 2025 a phished maintainer account was used to publish malicious versions of `chalk`, `debug` and other npm packages with billions of weekly downloads, and the self-replicating "Shai-Hulud" worm then compromised more than 500 npm packages (CISA alert, 23 September 2025; BleepingComputer, FortiGuard). Three anecdotes cannot "establish" a trend.
- Why it matters: This is optional, but a 2025 npm case would tie the motivation to the npm evaluation, where transitive-only exposure is 75 %. The three-incident list now ends in March 2024, 30 months before the data date. "Establish … increasing" should be "illustrate".

## C17. Abstract EN: k-core is listed twice, the four "domains" do not match the four contributions, and the spectral clause overstates
- Type: inconsistency / style
- Origin: pre-existing
- Severity: low
- Thesis: Chapters/00-Abstract.tex:19 — "(3) structural risk assessment, which identifies … dependency concentration using entropy metrics and k-core decomposition; and (4) graph analysis, applying community detection, k-core decomposition, and spectral graph theory to estimate how readily a compromise could spread through the graph"
- Evidence: k-core decomposition appears in both (3) and (4). Ch6:56 uses the core number for dense interconnection, not "dependency concentration", which in Ch6:155 is namespace concentration. Ch1:52–56 has four contributions: visualisation, vulnerability analysis, structural analysis and evaluation. The abstract splits Ch6 into (3) and (4) and does not list evaluation or blast radius / impact traversal, which is Ch1's first RO3 capability. The spectral radius is computed on the undirected projection (Ch6:11, Ch8:47), so it treats spreading to dependencies and to consumers alike, whereas a vulnerability reaches only consumers (Ch6:86). Ch7:81 calls it "a single index of spread potential". PT :6 has the same structure.
- Why it matters: These are minor, but the abstract is read first. Suggest removing the second "k-core decomposition", replacing "dependency concentration … k-core" with "namespace concentration and degree entropy", adding blast radius under (2), and writing "to index" instead of "to estimate".

## C18. Minor wording against the evaluated scope
- Type: style / overstated
- Origin: pre-existing
- Severity: low
- Thesis: Chapters/01-Introduction.tex:69 — "SBOM Lens operates on source-level SBOMs generated from package manifests"; :56 — "we discuss the implications for security analyst workflows"
- Evidence: Ch7:16–18: the evaluated SBOMs are generated by cdxgen from the root *lockfile*, and a lockfile is required because a manifest alone would resolve against today's registry. SBOM Lens itself accepts any CycloneDX JSON (Ch4). Ch7 has no section on analyst workflows. It has two one-sentence interpretations (Ch7:107, :128), and Ch1:67 excludes any analyst assessment.
- Why it matters: "Lockfiles" is the property the evaluation relies on. "Discuss the implications for analyst workflows" promises a discussion that Ch7 does not contain. "Interpret the results from an analyst's point of view" would be accurate.

## Summary
- high: 2 (C1, C2)
- medium: 9 (C3–C11)
- low: 7 (C12–C18)
- Numbers: the Abstract's only figures (90, "three-quarters", essentially unrelated rankings) match `per_sbom.csv` and Ch7. Ch1 reports no evaluation number. Its external numbers (96 %, over 500, CVSS 10.0, 84 tools) are correct, but the "orders of magnitude" claim beside them is not (C13).
