# Chapter 2 — Critic

What I checked. I read Ch2 line by line and compared each definition with its primary source: the CycloneDX 1.4–1.7 specification and ECMA-424, the SPDX 2.3 relationship table, ECMA-427 (PURL), the NTIA minimum elements, the EO 14028 text, the EO 14306 and OMB M-26-05 changes, the CRA application dates, the NVD status page, OSV's `docs/data.md`, SLSA v1.0 and the CVSS v3.1 specification. I also checked each claim against the code at `28113e7` (`git show 28113e7:<path>`, marked `@28113e7`): `backend/store/sbom_store.py`, `routers/impact.py`, `routers/graph_analysis.py`, `routers/audit.py`, `src/utils/vulnerabilityAPI.js`, `GraphVisualizer.js` and `utils/severity.js`. I recomputed the structural figures from `sbom-lens-eval/analysis/out/per_sbom.csv` with plain `csv`/`statistics`. I compared every forward reference with the current Ch1, Ch3 and Ch5–Ch8. I did not reopen the decisions already taken: the inclusive blast radius, the `28113e7` scope, the narrow novelty framing, the heavy-tail wording, HITS/powerlaw handling, OSV-only data, the Ch4 UI items and repository availability. Where I could not verify a figure, I say so.

Most of the chapter is sound. The CVSS v3.1 formula (apart from one edge case), the severity bands and colours, the log4shell vector, the k-core and HITS definitions and the inclusive blast-radius definition all match the sources and the code. Three kinds of problem remain. (1) Statements about regulation and databases were true in 2021–2023 and are no longer true in October 2026. (2) Some standards facts are wrong (the SPDX characterisation, the CycloneDX version history). (3) The chapter's vocabulary has drifted from what Ch5/Ch6 and the code use. The most consequential case: the concept the novelty rests on, dominance, is never defined, while betweenness and modularity, which SBOM Lens never computes, are.

## C1. The regulatory claims are out of date: the US no longer requires SBOMs, and the CRA SBOM obligation does not apply until December 2027
- Type: wrong / out of date / mis-cited
- Origin: pre-existing (also in Ch1:11)
- Severity: high
- Thesis: Chapters/02-Background.tex:20 — "US Executive Order 14028 (May 2021) mandated SBOMs for software sold to the US federal government; EO 14144 (January 2025) extended and reinforced these requirements \citep{EO14028}. The European Cyber Resilience Act (CRA), which entered into force in December 2024, similarly requires software component transparency … SBOMs have therefore moved from a voluntary best practice to a regulatory requirement in the two largest software markets in the world." Also :88 — "EO 14028 … mandated SBOMs … EO 14144 (January 2025) reinforced and extended these requirements."
- Evidence: (a) EO 14028 did not mandate SBOMs. Sec. 4(e)(vii) directs NIST to issue *guidance* that includes "providing a purchaser a Software Bill of Materials (SBOM) for each product directly or by publishing it on a public website". The binding instrument was OMB M-22-18, which left SBOMs to each agency. (b) EO 14306 (6 June 2025) substantially amended EO 14144 and removed its attestation and technical-data requirements (ABA, Crowell, DWT client alerts, June 2025). (c) OMB M-26-05 (February 2026) rescinded M-22-18 and M-23-16 and "permits agencies to choose whether to require their vendors to provide SBOMs" (DWT, 10 Feb 2026). (d) CRA: in force 10 Dec 2024. Reporting obligations apply from 11 Sep 2026, and the Annex I requirements, which include the SBOM, apply from 11 Dec 2027 (Reg. 2024/2847 Art. 71). Annex I Part II(1) requires an SBOM "covering at the very least the top-level dependencies", which is a weaker requirement than the full graph SBOM Lens needs. (e) The EO 14144 sentence cites `EO14028`, but `EO14144` exists in the bib (Ch1:11 uses it). (f) "Two largest software markets" has no source. (g) :20 and :88 repeat each other.
- Why it matters: The chapter's regulatory motivation, "moved … to a regulatory requirement", is false today for the US and premature for the EU. An examiner who knows EO 14306 or M-26-05 will notice. The CRA "top-level dependencies" floor is worth stating, because it shows why many compliant SBOMs will not carry the transitive graph that SBOM Lens analyses.

## C2. The vocabulary chapter never defines dominance, the measure that distinguishes SBOM Lens, but defines betweenness and modularity, which SBOM Lens never computes
- Type: inconsistency / unsupported
- Origin: pre-existing
- Severity: high
- Thesis: Chapters/02-Background.tex:4 — "Every concept defined here reappears by name in \autoref{cp:system-design}, \autoref{cp:vuln-analysis}, and \autoref{cp:network-sci} without re-definition."; :174–178 (betweenness, Brandes); :190–194 (modularity \(Q\), "values above approximately 0.3").
- Evidence: (a) Ch2 contains no definition of a dominator, the immediate dominator, the dominator tree or a dominated set. Ch5:235–238 introduces them operationally (`nx.immediate_dominators`, virtual root) without a formal definition. No chapter or bib entry cites Lengauer–Tarjan, Cooper–Harvey–Kennedy or any other source for dominance: `grep -i 'Lengauer\|Cooper\|dominat' Bibliography.bib` returns nothing. Yet dominance is the first item in the agreed novelty statement (Ch3) and appears in Ch1, Ch5, Ch6, Ch7:61–63 and Ch8:16, 28, 35. (b) Betweenness centrality appears in no other chapter (`grep -i betweenness Chapters/0[1345678]*` is empty) and in no code at `28113e7` (no `betweenness` in `backend/`). (c) Ch6:85 says "The modularity score of the partition is not reported by the endpoint" (`routers/graph_analysis.py:18-40 @28113e7` returns only communities). So the formula and the 0.3 threshold describe nothing the tool outputs. (d) Bow-tie, rich-club, degree entropy, Clauset–Shalizi–Newman fitting and spectral exposure are defined only in Ch6 (:91, :99, :132, :170, :42). That is acceptable, but it contradicts Ch6:6: "The theoretical definitions of the underlying concepts were given in \autoref{sec:graph-theory} and \autoref{sec:network-science}". (e) Similarly, `nx.descendants` (:126 "Both are applied in \autoref{cp:vuln-analysis}") is used only in the bow-tie code, not in Ch5.
- Why it matters: The background chapter spends a subsection on a centrality measure the tool lacks. It leaves the thesis's distinguishing concept without a definition or a citation. An examiner checking "dominator" against the literature will find no reference at all.

## C3. "Standard package managers report [a cycle] as an error" contradicts npm and the thesis's own data: 30 of 90 graphs have cycles
- Type: wrong / inconsistency
- Origin: pre-existing
- Severity: high
- Thesis: Chapters/02-Background.tex:114 — "if package \(A\) depends on package \(B\) which depends on \(A\), neither can be resolved without the other, and standard package managers report this as an error."
- Evidence: `per_sbom.csv`: 30 of 90 SBOMs have ≥1 simple cycle and ≥1 non-trivial SCC (max 16 cycles, 5 SCCs). All 90 come from npm/pnpm/yarn lockfiles (`lockfile` column). Ch7:75 reports the same "30 of the 90 graphs". npm, pip and Go modules install circular dependency chains without error. Cargo forbids them only for normal dependencies. Ch2:116 itself goes on to say SBOMs "may contain directed cycles, arising from circular version-pinned dependencies". Ch6:144 relies on this subsection ("no component on it can be installed or updated independently … (\autoref{sec:dags})").
- Why it matters: A definitional subsection states something the evaluation refutes a third of the time. It also mislabels as anomalous a structure that the target ecosystem allows by design.

## C4. SPDX is mischaracterised: 45 relationship types, an explicit DEPENDS_ON edge, separate packages/relationships arrays, and SPDX 3.0 is ignored
- Type: wrong / out of date
- Origin: pre-existing
- Severity: medium
- Thesis: Chapters/02-Background.tex:30 — "it defines approximately twenty-five relationship types … the dependency relationship is represented implicitly as one type among many, and SPDX 2.3 has no native structure for embedding vulnerability data. … does not natively support the graph structure required for the analysis SBOM Lens performs."; :32 — "Its key structural advantage over SPDX is the explicit separation of components and their dependency relationships into two distinct top-level arrays."; :36 "eliminating the inference step that SPDX dependency reconstruction would require."
- Evidence: The SPDX 2.3.1 spec, §11 "Relationships between SPDX elements", lists 45 relationship types (DESCRIBES … OTHER), including nine dependency-specific types (DEPENDS_ON, DEPENDENCY_OF, BUILD_/DEV_/OPTIONAL_/PROVIDED_/TEST_/RUNTIME_DEPENDENCY_OF, DEPENDENCY_MANIFEST_OF). DEPENDS_ON is an explicit directed edge, not an implicit one. SPDX 2.3 JSON has separate top-level `packages` and `relationships` arrays, so the "key structural advantage" does not exist. The real difficulty is that edges can be stated in either direction (DEPENDS_ON vs DEPENDENCY_OF) and must be normalised. SPDX 3.0 (April 2024) adds a Security profile with vulnerability elements. Ch8:69 lists "support for SPDX input" as future work, which contradicts "does not natively support the graph structure".
- Why it matters: The case for CycloneDX-only rests on a wrong reading of SPDX. A fair justification would be scope and the dataset's generator (cdxgen, `sbom-lens-eval/pipeline/sbom.py:38`), not SPDX's inability to express a graph.

## C5. CycloneDX facts: VEX/`vulnerabilities[]` came in 1.4, not 1.6; only `type` and `name` are required; `bom-ref` is unique, not "stable"
- Type: wrong / inconsistency
- Origin: pre-existing
- Severity: medium
- Thesis: Chapters/02-Background.tex:32 — "CycloneDX 1.6 added native support for VEX … via a \texttt{vulnerabilities[]} array."; :42 — "each carrying at minimum a \texttt{bom-ref} …, a \texttt{name}, a \texttt{version}, a \texttt{purl} …, and a \texttt{type}."; :44 — "It is a stable, local reference … When \texttt{bom-ref} is absent …"
- Evidence: CycloneDX v1.4 (released 12 Jan 2022, cyclonedx.org/news/cyclonedx-v1.4-released) introduced the `vulnerabilities` array and VEX. 1.6 (2024) added attestations, CBOM and ML-BOM, and became ECMA-424 1st ed. (June 2024). 1.7 (28 Oct 2025) is ECMA-424 2nd ed. (11 Dec 2025), which the chapter has right. In the 1.4–1.7 schemas `component.required` is `["type","name"]`; version, purl and bom-ref are optional. The chapter's own :44 and the code comment "bom-ref is optional in CycloneDX — fall back to purl, then name" (`backend/store/sbom_store.py:22 @28113e7`) contradict "at minimum". The spec requires a bom-ref only to be unique within the BOM, and generators regenerate it, so it is not stable across documents. The vertex set at :46 also omits two things the code does (`sbom_store.py:21-34 @28113e7`): nested `components[].components` are ignored, and self-loops and `dependsOn` targets that are not components are dropped (Ch6:56 states the self-loop rule).
- Why it matters: These are checkable spec facts in the section that "specifies exactly which fields … SBOM Lens consumes".

## C6. "NVD provides the authoritative CVSS v3.1 base scores for most published CVEs" no longer holds
- Type: out of date
- Origin: pre-existing
- Severity: medium
- Thesis: Chapters/02-Background.tex:230 — "enriches CVE entries with CVSS base scores (versions 2.0, 3.x, and 4.0) … NVD provides the authoritative CVSS v3.1 base scores for most published CVEs and is the primary reference for severity data in the security industry."
- Evidence: The NVD enrichment backlog dates from February 2024. On 2 Apr 2025 NIST marked pre-2018 un-enriched CVEs as "Deferred". From 15 Apr 2026 NVD prioritises only KEV, federal-use and EO 14028 critical software; all other CVEs get "Lowest Priority"/"Not Scheduled" (nist.gov/itl/nvd status updates; Help Net Security, 16 Apr 2026). The same announcement says NIST stops issuing independent severity scores and relies on CNA scores. NVD stopped assigning CVSS v2.0 to new CVEs in July 2022. "Primary reference … in the security industry" has no source.
- Why it matters: A reader would infer that NVD severity is complete and authoritative. As of 2026 it is neither, and this actually strengthens the OSV choice (Ch5:16–20). The sentence should be dated or rewritten.

## C7. CVSS v3.1 is presented as "the standard"; v4.0 (Nov 2023) is current, and Ch5/Ch8 already handle v4-only records
- Type: out of date / inconsistency
- Origin: pre-existing
- Severity: medium
- Thesis: Chapters/02-Background.tex:222 — "establishes the CVSS v3.1 scoring system used to quantify severity"; :254 — "The Common Vulnerability Scoring System (CVSS) v3.1 is the standard for quantifying the severity of software vulnerabilities".
- Evidence: FIRST published CVSS v4.0 on 1 Nov 2023. GHSA and NVD publish v4.0 vectors (Ch2:230 itself lists 4.0, and NVD corrected about 4,500 v4.0 scores on 28 Apr 2026). Ch5:30 says "CVSS v4 vectors are not scored, because the v3.1 formula does not apply to them". Ch8:41 and :63 list v4-only records falling to the unknown band as a limitation. Ch2 never says that v4.0 exists as a successor or that its metric set differs (AT, VC/VI/VA, SC/SI/SA, no Scope).
- Why it matters: Without that sentence the background does not set up the limitation Ch5 and Ch8 rely on, and "the standard" is three years out of date.

## C8. The adjacency matrix and spectral radius are introduced on the directed graph, where Ch6 shows they are zero; one citation does not support 1/λ₁
- Type: inconsistency / mis-cited
- Origin: pre-existing
- Severity: medium
- Thesis: Chapters/02-Background.tex:106 — "\(A_{ij} = 1\) if the directed edge \(i \to j\) exists … The adjacency matrix is the basis for eigenvector centrality analysis"; :182 — "solved as the leading eigenvector of the adjacency matrix \(\mathbf{A}\)"; :214 — "the spectral radius \(\lambda_1\) of the adjacency matrix governs how readily a contagion spreads …, with \(1/\lambda_1\) acting as its epidemic threshold \citep{Wang2003,PastorSatorras2001}".
- Evidence: Ch6:10 says "the adjacency matrix of a directed acyclic graph is nilpotent: every one of its eigenvalues is zero". The code uses the symmetric adjacency of `G.to_undirected()` (`routers/graph_analysis.py:320-366 @28113e7`). :182 mentions the projection, but :106 and :214 point to the directed \(\mathbf{A}\) just defined. Wang et al. (2003) derive the SIS threshold \(1/\lambda_1\) for undirected graphs. Pastor-Satorras & Vespignani (2001) derive a degree-based mean-field threshold \(\langle k\rangle/\langle k^2\rangle\), which vanishes in scale-free networks; it is not \(1/\lambda_1\). Ch6:124 correctly cites only Wang2003. :106 "stored in compressed sparse format in practice" holds only for graphs with more than 500 nodes; smaller ones use dense `numpy.linalg.eigh` (`graph_analysis.py:330-340`).
- Why it matters: As written, the background defines a quantity that is identically zero for every acyclic SBOM. Ch6 then has to undo this.

## C9. Community "containment" contradicts directed propagation and Ch6:85; the 0.3 modularity heuristic is mis-attributed and unreliable on tree-like graphs
- Type: inconsistency / mis-cited / overstated
- Origin: pre-existing
- Severity: medium
- Thesis: Chapters/02-Background.tex:198 — "A vulnerability that enters one high-modularity community tends to spread within it, contained by the sparse inter-community edges. Communities with many inter-community connections are higher-risk propagation bridges."; :194 — "values above approximately \(0.3\) are generally considered to indicate meaningful community structure \citep{Barabasi2016}".
- Evidence: A vulnerability reaches exactly \(\mathrm{anc}(v)\) (Ch2:126; `impact.py:50 @28113e7`) whatever the community boundaries are. Ch6:85 was corrected to say so: "Because a vulnerability reaches only consumers, what matters is how many components outside the community depend on it". The 0.3 rule of thumb comes from Newman & Girvan (2004) and Clauset–Newman–Moore (2004). I could not find it stated in Barabási (2016). Guimerà, Sales-Pardo & Amaral (2004, PRE 70:025101) show that random graphs reach high modularity, and sparse, nearly tree-like graphs such as these SBOMs score high almost by construction. SBOM Lens does not report \(Q\) anyway (C2).
- Why it matters: Ch2 states a propagation model that the thesis's own definitions and Ch6 reject.

## C10. K-core as a triage signal for typosquatting and dependency confusion is unsupported and appears nowhere in Ch6
- Type: unsupported / inconsistency
- Origin: pre-existing
- Severity: medium
- Thesis: Chapters/02-Background.tex:82 — "Graph-based SBOM analysis complements detection for the latter two categories. K-core decomposition (\autoref{cp:network-sci}) reveals how deeply embedded a potentially substituted package is in the dependency network, providing a structural triage signal for incident response."; also :208 — "replacing it requires finding an alternative that satisfies all components in the inner core simultaneously."
- Evidence: Ch6:53–58 (core number) never mentions name-confusion attacks, and the forward reference has nothing to land on. The measure that tells you who is affected by a substituted package is its blast radius (\(\mathrm{anc}(v)\)). Core number is undirected (Ch6:58: "says nothing about direction"), so a high value can come from a package's own dependencies rather than its consumers. In Ch6 the attack taxonomy is linked to namespace concentration (Ch6:158), not to k-core. No source is given for the replacement claim at :208.
- Why it matters: It claims an applied use case that the thesis neither implements nor evaluates.

## C11. "Log4j was a transitive dependency in millions of Java applications" is uncited and conflicts with Ch3's measured figure
- Type: unsupported / inconsistency
- Origin: pre-existing
- Severity: medium
- Thesis: Chapters/02-Background.tex:66 — "Apache Log4j was a transitive dependency in millions of Java applications; many of the affected development teams were entirely unaware that their software included it."
- Evidence: Ch3:102 cites GoogleLog4j2021: "over 17,000 Java packages — about 4% of the repository — depended on an affected version". `GoogleLog4j2021` and `NIST_CVE_2021_44228` are in the bib but not cited here. "Millions of applications" and the claim about teams being unaware have no source.
- Why it matters: An uncited figure that is two orders of magnitude above the one the thesis does cite.

## C12. The power-law framing goes beyond the agreed hedging: preferential attachment in ecosystems is asserted without a source
- Type: unsupported / overstated
- Origin: pre-existing
- Severity: medium
- Thesis: Chapters/02-Background.tex:152 — "Open-source package ecosystems exhibit both mechanisms: new packages are added continuously, and new packages tend to depend on already-popular libraries."; :150 — "\(\gamma \in [2, 3]\) for most empirical cases \citep{Barabasi1999}".
- Evidence: No source is given for preferential attachment in package ecosystems. Ch3:76–78 cites only software-internal studies plus Clauset2009's warning, and the agreed position is "heavy-tailed, consistent with, not a confirmed power law". Barabási–Albert (1999) report γ ≈ 2.1–2.3 for the WWW and actors and γ ≈ 4 for the power grid. The "most empirical cases" generalisation is contested (Broido & Clauset 2019, "Scale-free networks are rare", Nat. Commun. 10:1017, not in bib). BA showed that growth plus preferential attachment is *sufficient* for scaling, not that it explains every observed scaling ("explains hub emergence").
- Why it matters: The background asserts a generative mechanism the thesis never tests, which undercuts the careful hedging in Ch3, Ch6:172 and Ch7.

## C13. The attack categories are attributed to Ladisa et al. but do not match their taxonomy
- Type: mis-cited
- Origin: pre-existing
- Severity: medium
- Thesis: Chapters/02-Background.tex:72 — "\citet{Ladisa2023} provide a systematic taxonomy … Four primary categories are relevant to the SBOM analysis context." followed by build pipeline compromise, compromised maintainer, dependency confusion, typosquatting.
- Evidence: Ladisa et al. (S&P 2023) organise their attack tree under the root "inject malicious code into dependency tree". It has two top-level branches: *develop and advertise a distinct malicious package*, where typosquatting, combosquatting and dependency confusion sit together under "create name confusion", and *inject into a legitimate package* (via sources, during build, during distribution, via maintainer account takeover). The four headings in the chapter are a reasonable selection, but not "primary categories" of that taxonomy. Two of them are siblings of a single leaf group. Also, "typically through social engineering over months or years" (:76) generalises from xz; account takeover (e.g. ua-parser-js, 2021) is the more common route.
- Why it matters: The citation should support "categories drawn from", not "four primary categories".

## C14. Ch5 says Ch2 "fully defines" the numerical CVSS metric values; Ch2 does not
- Type: inconsistency
- Origin: pre-existing
- Severity: medium
- Thesis: Chapters/02-Background.tex:258 — "where \(C\), \(I\), and \(A\) are numerical values from the specification's lookup table"; :266 "each metric takes its numerical value from the lookup table".
- Evidence: Ch5:91: "The CVSS v3.1 base score formula and the numerical values for each metric are fully defined in \autoref{sec:cvss}". Ch5:123: "Substituting the numerical values specified in \autoref{sec:cvss} gives: AV = 0.85 …". The values exist only in the code (`src/utils/vulnerabilityAPI.js:25-34 @28113e7`) and in Ch5:110 (PR only).
- Why it matters: The cross-reference sends the reader to a table that does not exist. The fix is either a short table in Ch2 or a change to Ch5.

## C15. OSV description: "over fifty upstream sources", "npm advisories", ranges "rather than" version lists, `/v1/query`
- Type: wrong / inconsistency
- Origin: pre-existing
- Severity: low
- Thesis: Chapters/02-Background.tex:238 — "defines version range bounds using \texttt{introduced} and \texttt{fixed} fields rather than fixed version lists … OSV aggregates vulnerability data from over fifty upstream sources including RustSec, PyPI advisories, npm advisories, …"; :240 — "OSV's query API (\texttt{/v1/query}) accepts a package purl directly, making it the most precise source".
- Evidence: OSV's `docs/data.md` (master) lists 18 native sources plus 3 converted ones (Debian, Alpine and NVD CVEs for OSS), about 21 in total. There is no separate "npm advisories" source; npm data arrives through GHSA. The OSV schema has both `affected[].ranges` and an explicit `affected[].versions` list. SBOM Lens calls `/v1/querybatch` and then `/v1/vulns/{id}` (Ch5:26–28). "Most precise source" has no source. Note also that OSV ingests NVD CVEs for open-source software, which qualifies Ch5:20 "vulnerabilities that exist only in NVD … are not reported".
- Why it matters: A checkable count is wrong by more than a factor of two, and the endpoint named differs from Ch5's.

## C16. SLSA is described as a "four-level" framework covering source integrity, which is the retired v0.1 model
- Type: out of date
- Origin: pre-existing
- Severity: low
- Thesis: Chapters/02-Background.tex:90 — "SLSA provides a complementary four-level maturity framework for source integrity, build integrity, and provenance attestation".
- Evidence: SLSA v1.0 (April 2023) replaced levels 1–4 with a Build track L0–L3 and dropped the source requirements ("For version 1.0 the Source aspects were removed"). Source-track work reappears only in later versions (current v1.2). The bib entry `SLSA` has no version. "CISA has further published guidance on SBOM minimum elements" has no citation. The relevant document is CISA's draft *2025 Minimum Elements for an SBOM* (22 Aug 2025), which adds Component Hash, License, Tool Name and Generation Context to the NTIA list given at :18.
- Why it matters: Small, but it is the standards paragraph of a 2026 thesis.

## C17. HITS "computed iteratively until convergence" omits what Ch6 discloses
- Type: inconsistency
- Origin: pre-existing
- Severity: low
- Thesis: Chapters/02-Background.tex:180 — "HITS scores are computed iteratively until convergence and applied in \autoref{cp:network-sci}."
- Evidence: Ch6:32: "if it has not converged after 200 iterations, SBOM Lens reports every score as zero" (`impact.py:71-77, 100-103 @28113e7`). This does not reopen the HITS decision. It only aligns the background sentence with it ("or until an iteration cap").
- Why it matters: The same "until convergence" phrasing was a point in the Ch6/Ch7 reviews.

## C18. Attribution and algorithm details: σ, Brandes, Tarjan, the articulation-point rule, the CVSS Impact ≤ 0 case
- Type: mis-cited / incomplete
- Origin: pre-existing
- Severity: low
- Thesis: :160 σ formula cited only to WattsStrogatz1998, "\(\sigma \gg 1\) indicates small-world structure"; :178 "The Brandes algorithm … \citep{Barabasi2016}"; :116 "Tarjan's algorithm \citep{CLRS4th}"; :136 "A vertex \(v\) is an articulation point if any DFS-tree child \(w\) of \(v\) satisfies \(\mathrm{low}[w] \geq \mathrm{disc}[v]\)"; :274 "If \(\mathrm{Impact} = 0\), the base score is \(0.0\)".
- Evidence: σ was defined by Humphries & Gurney (2008). Ch6:178 cites both sources, and Humphries & Gurney use σ > 1, not ≫ 1. Brandes (2001) should be cited for Brandes' algorithm. CLRS presents Kosaraju's two-pass algorithm for SCCs, not Tarjan's (NetworkX uses a Tarjan/Nuutila variant). The low/disc rule holds only for non-root vertices: the DFS root is an articulation point iff it has ≥ 2 tree children. The CVSS v3.1 spec says "If Impact sub score ≤ 0". Under Scope Changed with ISS = 0, Impact = 7.52·(−0.029) − 3.25·(−0.02)^15 ≈ −0.22, so "= 0" misses the case (the code tests `ISS === 0`, `vulnerabilityAPI.js:39`, which is equivalent). :276 says SBOM Lens uses the five FIRST bands, but Ch5:108 collapses None into Low and adds UNKNOWN (drawn with the Low colour, `sbom-graph/index.js:229 @28113e7`).
- Why it matters: Each is minor, but together they are the kind of slip an examiner in graph theory or CVSS will notice.

## C19. Small factual and sourcing gaps in the supply-chain section
- Type: unsupported / overstated
- Origin: pre-existing
- Severity: low
- Thesis: :64 — "Transitive dependencies … routinely outnumber direct dependencies by an order of magnitude in large projects."; :74 — "distributed to approximately 18,000 organisations"; :76 xz-utils facts; :80 — "for example, \texttt{requests-oauthlib2} alongside \texttt{requests-oauthlib}"; :48 — "NVD uses CPE but can be cross-referenced via purl"; :134 — "a vulnerability … touches both sides of the split".
- Evidence: (a) `per_sbom.csv` `directShare`: the median is 11.5%, so the transitive:direct ratio is about 7.7:1, and 58 of 90 graphs have a direct share above 1/11 (ratio below 10×). Ch7:73 reports 11%/89%. "Roughly an order of magnitude" is fair; "routinely" is not, and there is no citation. (b) The SolarWinds figure comes from its SEC 8-K ("fewer than 18,000" customers *downloaded* the update). It is uncited here, although `CISA_AA20_352A` is in the bib. The xz facts are likewise uncited, although `NVD_CVE_2024_3094` is in the bib (Ch1:9 cites both). (c) I found no documented malicious `requests-oauthlib2`, and adding "2" is combosquatting rather than typosquatting. A documented case (e.g. `requesys`/`requesrs` targeting `requests`, 2022, or npm `crossenv`, 2017) would be safer, or the example should be marked as hypothetical. (d) The NVD CVE API offers no purl search, so the purl cross-reference claim needs a source or should be removed. (e) Under consumer→dependency edges, a vulnerability at an articulation point reaches only its ancestors, not "both sides" (cf. :162, which states the one-direction rule, and Ch6:26).
- Why it matters: Easy fixes, mostly adding the citations the bib already holds.

---

Counts: high 3 (C1–C3), medium 11 (C4–C14), low 5 (C15–C19).
