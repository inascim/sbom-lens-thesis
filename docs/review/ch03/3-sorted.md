# Chapter 3 — Sorted review

## Summary
- 19 items: 16 from the critic, 2 D-new items from the defender, and one I found while checking (S-new1). Some items are merged, which gives **16 errata**, **3 viva prep** and **0 holds**.
  - Errata: E1 (C1, C4), E2 (C11, D-new1), E3 (C3), E4 (C2), E5 (C5, D-new2), E6 (C6), E7 (C10), E8 (C9), E9 (C7), E10 (C8), E11 (C13), E12 (C14), E13 (C12), E14 (C15), E15 (C16), E16 (S-new1, Ch1:9 citations).
  - Viva prep: V1 (C6d, C14: why these four tools), V2 (C6a: SBOMs without edges), V3 (C15: SolarWinds scope). These draw on parts of items that are also errata.
- Every critic objection has a real basis. The defender conceded seven items outright. In the other nine, the rebuttal narrows the fix but does not remove it.
- **Results:** no reported number changes (Ch7 numbers, Abstract, Ch8). E7 changes the *wording* of the RQ1 conclusion at Ch7:90 and the RQ1 headline at Ch8:12. Both say the literature reports small-world clustering for whole ecosystems, and the cited ecosystem studies do not. The figures stay as they are.
- Facts I settled where the two sides disagreed:
  - C5: the **critic was right** that grype's default risk score uses KEV. The Anchore page says "When a vulnerability appears in the KEV catalog, Grype automatically treats it as maximum threat (overriding EPSS)". The defender said KEV was only an annotation, and that is wrong. It is both an annotation and an input to the score.
  - C5: the **defender was right** that OSV-Scanner's guided remediation is experimental and takes only npm `package-lock.json`/`package.json` and Maven `pom.xml`. But the defender's reason for leaving it out ("not an SBOM tool") is wrong: OSV-Scanner's own docs have an "SBOM scanning" section (SPDX and CycloneDX files). So the tool scans SBOMs, but its depth-based prioritisation is not offered for them.
  - C15: the **defender was right** about CISA's "share the same DLL version number". That note says two Orion platform releases ship one DLL version. It does not say the trojanised DLL kept a clean DLL's version.
  - C14: the **defender was right** that guac.sh supports "joined OpenSSF as an Incubating Project". Only the month is unsupported.
  - C13: the **defender was right** that Ponta is in §3.2.3, not §3.2.2.
  - C4: the **defender was right** that Gap 1's subject is "the surveyed tools", which excludes GUAC. Gap 2 ("the tools") and Ch1:29 are still contradicted by GUAC's `patch plan` BFS (`pkg/guacanalytics/patchPlanning.go:54,116`, checked on main).
- I found two more problems the critic and defender missed:
  - Ch1:25, "No mainstream SBOM tool performs this traversal", is false for Sunshine. It is now covered under E2.
  - Ch1:9 cites the NVD record for "three billion Java-based applications" and CISA AA20-352A for "approximately 18,000 customers". I checked both sources, and neither contains the figure (E16).
- Five decisions are needed (list at the end): whether to cite the 2026 SBOM-graph papers and Garcia 2025 in Ch3; the HITS row of Table 3.2; the log4j depth figures; Gap 3 and OSV-Scanner; and the Dependency-Track version shown in Table 3.1.

## Erratum

### E1 (from C1, C4) Sunshine, Dependency-Track and GUAC do compute reach or paths; Gaps 1–2 and the tool descriptions say they do not — impact: high
- Settled facts:
  - Sunshine (clone `beaab3f`, 2026-09-28, `sunshine.py`):
    - `ChildrenGatherer.get_children` (:1530–1590) recurses over `depends_on` from the root. It deep-copies the path for every child (:1538), records depths (:1544), and guards against cycles (:1545).
    - It pushes each vulnerability up to every consumer (`add_transitive_vulnerabilities_to_component`, :1513, called at :1554–1557).
    - It aborts above `SEGMENTS_THRESHOLD = 100000` path segments (:1528, :1551).
    - The report has "Transitively vulnerable components" (:1927) and "Transitive vulnerabilities" columns (:1710), and the text at :388 says it shows components "affected either directly or transitively".
    - So Sunshine lists the dependents-only reach of each vulnerability. It is computed by path enumeration, not by a graph search with a visited set, and Sunshine computes nothing else (no dominance, articulation points, centrality, cores or communities).
    - Both parties agree on all of this.
  - Dependency-Track:
    - Frontend PR #667 (merged 2023-12-10) adds "Show in Dependency-Graph": it "redirects the user to the projects dependency graph and highlights the affected component".
    - So DT resolves and shows the path to a vulnerable component. That is reachability for navigation. It does not compute anything over the graph.
  - GUAC:
    - `guacone patch plan` ("query which packages are affected by the vulnerability", `cmd/guacone/cmd/patch.go:46-47`) runs `bfsOfDependents` over `BfsNode{Parents, Depth}`.
    - GUAC is outside the four surveyed tools, so Gap 1 ("the surveyed tools") stands for it. Gap 2 ("the tools") and :10 need to acknowledge it.
  - What remains true:
    - No surveyed tool computes a dominated set or any structural measure.
    - None draws the reach and the paths of a vulnerability on one deduplicated graph.
- Fix:
  - Ch3:10 — "GUAC constructs graphs across multiple SBOMs at portfolio scale, which is a complementary but distinct scope from the within-SBOM structural analysis that SBOM Lens addresses." → "GUAC constructs graphs across multiple SBOMs at portfolio scale and can search them for the dependents of a vulnerable package, but it computes no dominance or structural measures; its scope is complementary to the within-SBOM structural analysis that SBOM Lens addresses."
  - Ch3:14 — "the tree is a navigation aid, not an object of computation. No reachability, dominance or centrality is computed over it." → "the tree is a navigation aid --- it can be opened at the path to an affected component --- not an object of computation. No dominance or centrality is computed over it."
  - Ch3:18 — "so a component reached along several paths is drawn several times, and no graph algorithm is run over the structure." → "so a component reached along several paths is drawn several times. Sunshine does propagate each vulnerability to every component that depends on it, directly or transitively, and lists those components in a table, but it does so by enumerating dependency paths, and it computes nothing over the structure beyond that reachability."
  - Ch3:20 — "either discarded, displayed, or at most traversed as a tree. None treats them as a graph to be analysed." → "either discarded, displayed, or at most traversed as a tree, to find the path to a component or the components that depend on it. None computes anything over them beyond such reachability."
  - Ch3:132 — "\textbf{Gap 1: no graph representation in SBOM tooling.}" → "\textbf{Gap 1: no graph analysis in SBOM tooling.}". Dependency-Track has a dependency-graph view, so "no graph representation" is false.
  - Ch3:132 — "or carried along unchanged, but never turned into a graph on which algorithms run (\autoref{sec:sbom-tools})." → "or carried along unchanged, and at most traversed to find the path to, or the dependents of, a component; no surveyed tool computes structural measures over them (\autoref{sec:sbom-tools})."
  - Ch3:134 — "They do not report which parts of the application depend on it, through which paths, or which components could not be reached without it." → "At most they list the components that depend on it (Sunshine) or open a tree view at the path to it (Dependency-Track); outside the survey, GUAC's patch planning searches the dependents of a vulnerable package. None shows the dependents and the paths together on the dependency graph, and none reports which components could not be reached without it."
- Also touches: Ch1:13, Ch1:25, Ch1:29 and Ch1:81 (E2). CONTEXT.md's "_Avoid_: dependency tree" applies to SBOM Lens's own model, not to Sunshine's view, so "tree" is correct in the Sunshine and DT sentences. No result.
- Viva answer: "Listing a vulnerability's transitive dependents is not new. Sunshine does it, and GUAC can search for them. What SBOM Lens adds is the dominated set, the reach and the paths drawn on one deduplicated graph, and structural measures that also rank components without a CVE. Sunshine enumerates paths, so it duplicates shared components and stops at 100,000 path segments."

### E2 (from C11, D-new1, plus Ch1:25) Ch1 describes the same tools as flat inventories that perform no traversal — impact: high
- Settled facts:
  - Ch1:13 says the four tools "treat SBOMs as flat component inventories" and "provide no answer" to which components depend on a vulnerable one. Sunshine is named but not cited there.
  - Ch1:25: "No mainstream SBOM tool performs this traversal".
  - Ch1:29: GUAC "does not perform within-SBOM graph analysis".
  - All three are contradicted by the code facts in E1.
  - Ch1:29's GUAC sentence is separate from the Garcia sentence the author has already settled, so this fix does not touch the Garcia wording.
  - The defender's Ch1:13 fix changes only the first sentence. The question two sentences later, "they provide no answer to ...", is still false for Sunshine, so it needs changing too.
- Fix:
  - Ch1:13 — "The dominant open-source tools in the SBOM ecosystem --- Dependency-Track, grype, syft, and Sunshine/CycloneDX --- treat SBOMs as flat component inventories \citep{DependencyTrack, Grype, Syft}." → "Among the open-source tools in the SBOM ecosystem, Dependency-Track, grype and syft treat SBOMs essentially as component inventories, and Sunshine, which visualises the dependency tree, computes nothing over it beyond reachability \citep{DependencyTrack, Grype, Syft, Sunshine}."
  - Ch1:13 — "these tools report the affected component in isolation --- but they provide no answer to the question a security analyst immediately needs: \textit{which other components in my system depend on this one, directly or transitively, and are therefore also at risk?}" → "these tools report the affected component largely in isolation. Only Sunshine lists the answer to the question a security analyst immediately needs --- \textit{which other components in my system depend on this one, directly or transitively, and are therefore also at risk?} --- and it gives that answer as a table, not on the dependency graph."
  - Ch1:25 — "No mainstream SBOM tool performs this traversal; the analyst must do it manually, if they can do it at all." → "Of the tools surveyed in \autoref{cp:related-work}, only Sunshine performs this traversal, and it reports the result as a list rather than on the graph; with the others, the analyst must do it manually, if they can do it at all."
  - Ch1:29 — "but it does not perform within-SBOM graph analysis to assess the structural risk profile of a single software system." → "and can search the dependents of a vulnerable package, but it does not compute structural measures to assess the risk profile of a single software system."
  - Ch1:81 — "including Dependency-Track, grype, syft, and Sunshine/CycloneDX" → "including Dependency-Track, grype, syft and Sunshine". This uses one name for the tool, as Ch3 does.
- Also touches: E1 (same facts). Abstract:17 ("largely treats SBOMs as flat inventories") is hedged and can stand. No result.
- Viva answer: as E1.

### E3 (from C3) The log4j ecosystem figures are on a basis the source itself withdrew — impact: high
- Settled facts:
  - The post opens: "Editors Note: The below numbers were calculated based on both log4j-core and log4j-api ... The ecosystem impact numbers for just log4j-core, as of 19th December are over 17,000 packages affected, which is roughly 4% of the ecosystem." (saved copy of the post, read again).
  - The depth histogram is introduced as "how deeply an affected log4j package (core or api) first appears".
  - The "majority ... indirect" sentence also uses the core+api basis ("log4j-core or log4j-api").
  - Both parties agree on these facts. The qualitative point (most exposure is indirect and deep) is not contradicted. The source just gives no core-only depth figures.
  - The bib entry's `howpublished` says "Google Open Source Blog", but the URL and the page header are the Google *Security* Blog.
- Fix:
  - Ch3:102 — "an analysis of Maven Central found that 35{,}863 Java artifacts --- more than 8\% of the repository --- depended on an affected version of log4j, that the majority did so only through indirect dependencies, and that for more than 80\% of them the vulnerable package sat more than one level deep, in some cases nine \citep{GoogleLog4j2021}." → "an analysis of Maven Central found that over 17{,}000 Java packages --- about 4\% of the repository --- depended on an affected version of \texttt{log4j-core}. An earlier count in the same analysis, which also included \texttt{log4j-api}, found that the majority of affected artifacts did so only through indirect dependencies, and that for more than 80\% of them the vulnerable package sat more than one level deep, in some cases nine \citep{GoogleLog4j2021}."
  - Bibliography.bib:493 — "howpublished = {Google Open Source Blog, \url{...}}" → "howpublished = {Google Security Blog, \url{...}}"
  - Whether to keep the depth figures at all is **DECISION NEEDED** (D3).
- Also touches: none. The figures appear only at Ch3:102. No result.
- Viva answer: "The core-only count halves the scale, to about 4 %. The source gives depth only on the earlier basis, and the text now says so. The structural point, that exposure is mostly indirect, does not depend on which basis is used."

### E4 (from C2) Dependency-Track does not ingest SPDX — impact: high
- Settled facts:
  - README (master) :38: "Consumes and produces [CycloneDX] Software Bill of Materials (SBOM)". SPDX appears only at :93, "Supports standardized SPDX license ID's".
  - Issue #1746: "SPDX support was removed ... from prior versions of DT".
  - The 5.0–5.2 release notes mention SPDX only as licence-list bumps (#7502).
  - Both parties agree.
- Fix: Ch3:32 — "Dependency-Track & CycloneDX (preferred), SPDX &" → "Dependency-Track & CycloneDX only (SPDX for licence IDs) &"
- Also touches: none. No result.

### E5 (from C5, D-new2) Prioritisation in existing tools is described as CVSS-only or severity-only; grype's default risk score includes EPSS and KEV — impact: medium
- Settled facts:
  - Anchore docs: "By default, Grype sorts vulnerability results by risk score", combining "EPSS ... or presence in CISA's Known Exploited Vulnerabilities (KEV) catalog" with CVSS. KEV is "treated as maximum threat (overriding EPSS)". So the critic is right and the defender is wrong (see Summary).
  - Every input to that score describes the vulnerability. So the core of Gap 3, "describe the vulnerability rather than where the affected component sits", holds.
  - DT README :67: "Helps to prioritize mitigation by incorporating support for ... EPSS". So Ch3:14 "prioritised by severity" and Ch1:23 "ranked at most by CVSS score" understate both tools. Ch1:23 also contradicts Ch3:136.
  - OSV-Scanner: guided remediation is experimental, offers "Prioritising vulnerabilities by dependency depth, severity" and "... by the total number of transitive vulnerabilities fixed", and takes only npm and Maven manifests and lockfiles. The scanner itself does accept SBOM files ("SBOM scanning", docs/usage.md).
- Fix:
  - Ch3:14 — "findings are listed and prioritised by severity" → "findings are listed and prioritised by severity and exploit likelihood (EPSS)". If E1 is applied, this sits in the same sentence as E1's :14 change.
  - Ch3:136 — "Findings are ranked by CVSS score, sometimes refined by exploit likelihood (EPSS), which describe the vulnerability rather than where the affected component sits." → "Findings are ranked by CVSS score, sometimes combined with exploit likelihood (EPSS) and known exploitation (CISA KEV) into a risk score, as grype does by default; all of these describe the vulnerability rather than where the affected component sits."
  - Ch1:23 — "ranked at most by CVSS score" → "ranked by CVSS score, at most combined with exploit likelihood (EPSS) or known exploitation (KEV) --- measures of the vulnerability, not of where the component sits"
  - Whether to add OSV-Scanner to Gap 3 is **DECISION NEEDED** (D4).
- Also touches: Ch1:23. Ch7:157 (construct validity, "combine severity with exploit likelihood") is already consistent. No result: RQ2 compares against severity *bands*, and Ch7:157 already discloses that limitation.
- Viva answer: "Every score the tools use, CVSS, EPSS, KEV or grype's combined risk score, describes the vulnerability. The closest prior structural signal is dependency depth in OSV-Scanner's experimental lockfile remediation. SBOM Lens uses position beyond depth and also ranks components with no CVE."

### E6 (from C6) The gaps are stated as absences from the literature without a search that could show it — impact: medium
- Settled facts:
  - Both arXiv papers exist (arXiv API, read again).
    - Zięba-Kozarzewski, 2607.22140 (v1 2026-07-24, v2 2026-08-13) bills itself as "the first large-scale characterization of the declared dependency graph across 78,612 real-world SBOM files". It finds 52.9 % with no edges, 8.8 % degenerate and 38.3 % well connected.
    - Baird & Moin, 2601.20158 (2026-01-28), trains an HGAT on enriched SBOM graphs to predict vulnerable components.
  - Neither applies the network-science toolkit of §3.2.4 (degree distributions, clustering, path lengths, centrality, communities, power-law fitting), and neither tests whether ecosystem properties hold for one application. So the specific claim survives, and the defender is right on that.
  - But the first paper does study single-SBOM graph topology at scale. So "has not been systematically analysed" (:138) and "absent from ... the research literature" (:130) are too absolute.
  - Ch3 cites Garcia2025 nowhere. Ch1:29 cites it and points to Ch3 as reaching "the same conclusion".
- Fix:
  - Ch3:130 — "four capabilities absent from current tooling and the research literature" → "four capabilities that, to our knowledge, current tooling and the research literature do not provide"
  - Ch3:78 — "What the literature has not done is apply this toolkit systematically" → "What the literature has not done, to our knowledge, is apply this toolkit systematically"
  - Ch3:138 — "has not been systematically analysed with these methods" → "has not, to our knowledge, been systematically analysed with these methods"
  - Whether to cite the two 2026 papers and Garcia 2025 in Ch3 is **DECISION NEEDED** (D1).
- Also touches: Ch1:29, only if D1 adds Garcia to Ch3. The settled Ch1 wording does not change. No result.

### E7 (from C10) "Established ... scale-free graphs" is not what the cited ecosystem studies show; Ch7:90 and Ch8:12 credit ecosystems with small-world clustering — impact: medium (touches the wording of the RQ1 conclusion, not its numbers)
- Settled facts (I read the critic's `pdftotext` copies):
  - Kikas: "the distribution of the number of dependents is skewed". There is no scale-free claim.
  - Decan 2019: "we hypothesise ... For example, we found a very unequal distribution of connectivity for each ecosystem, characteristic of power law or Pareto law behaviour". There is no fit.
  - Zimmermann: power law only in citing prior work. "Small" means "packages are densely connected via dependencies". It is not a measured small-world coefficient.
  - None of them measures clustering. Valverde 2002 and Myers 2003 report small-world structure, but for class and collaboration graphs inside software, not for ecosystems.
  - Both parties agree.
- Fix:
  - Ch3:52 — "have established that open-source ecosystems form large, scale-free graphs in which" → "have established that open-source ecosystems form large graphs with highly unequal, heavy-tailed connectivity, in which"
  - Ch7:90 — "the properties the literature reports for entire ecosystems: a heavy-tailed in-degree distribution, small-world clustering, and a large majority of transitive dependencies." → "two properties the literature reports for entire ecosystems, a heavy-tailed in-degree distribution and a large majority of transitive dependencies, together with the small-world structure reported for the internal structure of software \citep{Valverde2002, Myers2003}."
  - Ch8:12 — "\textbf{RQ1: the dependency graph of a single application has the structure found in whole ecosystems.}" → "\textbf{RQ1: the dependency graph of a single application has the heavy-tailed structure found in whole ecosystems.}"
- Also touches: Ch7:90 and Ch8:12. These are reported conclusions, and only the attribution changes: every figure (87/90 heavy-tailed, α 2.09–3.00, σ median 16.8, all 90 small-world) stays. Ch7:57 and Ch7:77 refer to the ecosystem studies only for heavy tails and concentration, so they can stand. Ch2:152 (ecosystems "exhibit both mechanisms" of preferential attachment) is uncited but is not touched by this item.
- Viva answer: "The ecosystem studies report highly unequal connectivity, not fitted power laws. That is why SBOM Lens fits by maximum likelihood rather than asserting scale-freeness. Small-world structure was reported for software's internal structure, and RQ1 finds it at the scale of one application."

### E8 (from C9) Ponta et al. do not report vulnerabilities found unreachable — impact: medium
- Settled facts:
  - Ponta (arXiv 1806.05893): "vulnerable constructs are reachable ... in 131 out of 496 applications ... 390 pairs ... In 32 cases, the reachability could only be determined through the combination of techniques, which represents a 7.9% increase of evidence that vulnerable code is potentially executable".
  - There is no count of unreachable vulnerabilities. Both parties agree.
  - The same paragraph's first sentence says reachability filters "to only genuinely exploitable paths". The paper's own term is "potentially executable", and reachable is not the same as exploitable.
- Fix:
  - Ch3:68 — "and showed that such code-centric analysis identifies many reported vulnerabilities as unreachable." → "and showed that combining the two finds reachable vulnerable code that either technique alone would miss."
  - Ch3:68 — "filtering CVE noise to only genuinely exploitable paths" → "filtering CVE noise to the paths along which the vulnerable code can actually be executed"
- Also touches: none. Ch7:157 refers to §3.2.3 only for the over-approximation, which still holds. No result.

### E9 (from C7) grype's database is not built from OSV — impact: medium
- Settled facts:
  - The vunnel provider directory (listed again today) has alma … wolfi, plus `nvd`, `github`, `epss` and `kev`, and **no `osv`**.
  - Both parties agree.
- Fix: Ch3:33 — "Aggregated DB (NVD, OSV, GHSA), EPSS, CISA KEV" → "Aggregated DB (NVD, GHSA, OS-distribution feeds), EPSS, CISA KEV"
- Also touches: none. No result.

### E10 (from C8) The Dependency-Track source list is out of date — impact: medium
- Settled facts:
  - README :59–65: NVD, GitHub Advisories, Sonatype OSS Index, Snyk, Trivy, OSV, VulnDB. EPSS is at :67.
  - 5.1.0 (2026-08-27): "Implement initial KEV support" (#6516) and "Add JVN ... data source" (#6640).
  - The 4.14.x line is still maintained (4.14.5, 2026-10-05) and has no KEV.
  - The bib `urldate` is 2026-10-09.
  - Both parties agree.
- Fix: Ch3:32 — "NVD, OSV, GHSA, EPSS, Snyk, VulnDB" → "NVD, GHSA, OSV, OSS Index, Snyk, Trivy, VulnDB, JVN; EPSS, CISA KEV (5.1+)"
  - Whether to give the version in the caption is **DECISION NEEDED** (D5).
- Also touches: E4 (same cell row). No result.

### E11 (from C13) Not all reviewed vulnerability research is ecosystem-wide — impact: low
- Settled facts:
  - Ponta is in §3.2.3, so it is not covered by :62 or :134 (the defender is right).
  - Pashchenko studies "the 200 most popular OSS Java libraries used by SAP", with counts per library. That is a sample of projects, not an ecosystem.
  - Liu 2022 resolves one tree per package, but its findings are npm-wide statistics.
- Fix:
  - Ch3:62 — "it measures propagation across an ecosystem." → "it measures propagation across an ecosystem, or counts vulnerable dependencies per library across a sample \citep{Pashchenko2018}, rather than analysing the structure of one project's graph."
  - Ch3:134 — "does so across whole ecosystems" → "aggregates over whole ecosystems or samples of projects"
- Also touches: none. No result.

### E12 (from C14) Three citations or claims do not support their sentence — impact: low
- Settled facts:
  - NVD CVE-2021-44228: the record's English description (NVD API, fetched again) contains neither "transitive" nor any scale figure.
  - GUAC: guac.sh supports joining OpenSSF as an incubating project. The month (March 2024) is not on the cited page, and the bib has only `year = {2024}`.
  - Stars as of today: Sunshine 126, Dependency-Track 4,269, syft 9,658, grype 12,995. So "widespread production use" is unsupported for Sunshine.
- Fix:
  - Ch3:100 — "the log4shell vulnerability exposed the hidden scale of transitive dependency risk \citep{NIST_CVE_2021_44228}" → "the log4shell vulnerability \citep{NIST_CVE_2021_44228} exposed the hidden scale of transitive dependency risk \citep{GoogleLog4j2021}"
  - Ch3:10 — "joined OpenSSF as an incubating project in March 2024 \citep{GUAC2024}" → "joined OpenSSF as an incubating project in 2024 \citep{GUAC2024}"
  - Ch3:10 — "Four open-source tools are in widespread production use:" → "Four open-source tools are surveyed --- three in wide use, and Sunshine, the CycloneDX project's own SBOM viewer:"
- Also touches: Ch1:13 (E2 removes "dominant"). Ch1:9 has the same citation pattern (E16).

### E13 (from C12) Table 3.2's HITS authority value is a six-way tie — impact: low
- Settled facts:
  - I recomputed it (13 nodes, 13 edges, `nx.hits(max_iter=200)`). log4j-core, spring-webmvc, jackson-databind, commons-text, struts2-core and h2 each score 0.1667. Every other node scores 0, including commons-collections, the only node with in-degree 2.
  - The value is correct but carries no information. Both parties agree.
- Fix: **DECISION NEEDED** (D2).
  - Option A, Ch3:119 — "HITS authority & 0.167 (sum over all components: 1) \\" → "HITS authority & 0.167, tied with the application's five other direct dependencies (all other components: 0) \\"
  - Option B: delete the row.
- Also touches: none. No result.
- Viva answer: "In a star-rooted toy graph, HITS concentrates all authority on the root's direct dependencies. The row reports the tool's output faithfully, and the text now says it is a tie."

### E14 (from C15) "The same components before and after the injection" is loose — impact: low
- Settled facts:
  - CISA: the malicious DLL was added "into the SolarWinds software lifecycle" and "then signed by the legitimate SolarWinds code signing certificate".
  - The "same DLL version number" note concerns two platform releases, so the critic misread it. The defender is right.
  - The injection happened during the build, so no clean SBOM of that release existed. Names and versions would match an uncompromised build. Only file hashes from an independent build could differ.
  - The section's conclusion (detection needs provenance controls) holds.
- Fix: Ch3:92 — "An SBOM of Orion would have listed the same components before and after the injection, and" → "An SBOM of Orion would have listed the same component names and versions as an uncompromised build --- only file hashes, compared against an independent reference build, could have differed --- and"
- Also touches: none.

### E15 (from C16) Framing slips: CVSS/EPSS clause, "attack" heading, "13 components" — impact: low
- Settled facts:
  - §3.1 has no CVSS/EPSS discussion apart from table cells. After E5, there is also one clause at :14.
  - log4shell was a disclosure, not an attack. :97 says so itself.
  - The SBOM has 12 `components` plus `metadata.component`. `_build_graph` (`sbom_store.py:18-20 @28113e7`) prepends the metadata component, which gives 13 nodes. I recomputed this.
- Fix:
  - Ch3:4 — "surveys four open-source SBOM tools using a feature matrix, weaving in the CVSS/EPSS scoring context established in \autoref{cp:background}." → "surveys four open-source SBOM tools using a feature matrix."
  - Ch3:81 — "\section{Supply Chain Attack Case Studies}" → "\section{Supply Chain Incident Case Studies}"
  - Ch3:104 — "a small application of 13 components that depends on \texttt{log4j-core} directly" → "a small application with 12 components, which depends on \texttt{log4j-core} directly"
  - Ch3:109 — "(13 components, 13 dependency edges)" → "(13 nodes --- the application and 12 components --- and 13 dependency edges)"
  - Ch1:81 — "and supply chain attack case studies" → "and supply chain incident case studies"
- Also touches: Ch1:81. Ch1:9 ("a succession of high-profile attacks", which includes log4shell) is loose in the same way. Changing it is optional.

### E16 (S-new1, found by the sorter) Ch1:9 attributes two figures to sources that do not contain them — impact: low
- Settled facts:
  - Ch1:9 — "present as a transitive dependency in an estimated three billion Java-based applications \citep{NIST_CVE_2021_44228}". The NVD record (NVD API, fetched today) contains neither "billion" nor "transitive".
  - Ch1:9 — "distributed to approximately 18,000 customers \citep{CISA_AA20_352A}". A full-text search of the saved AA20-352A page finds no "18,000". The figure is usually traced to SolarWinds' SEC filing, which is not in the bibliography.
- Fix:
  - Ch1:9 — "present as a transitive dependency in an estimated three billion Java-based applications \citep{NIST_CVE_2021_44228}" → "present, often as a transitive dependency, across the Java ecosystem \citep{NIST_CVE_2021_44228, GoogleLog4j2021}"
  - Ch1:9 — "that was subsequently distributed to approximately 18,000 customers \citep{CISA_AA20_352A}" → "that was subsequently distributed to its customers \citep{CISA_AA20_352A}". If you want to keep the number, add a source that states it.
- Also touches: none. This is outside Ch3. I include it because it is the E12 pattern applied to the same two sources.

## Viva prep

### V1 (from C6d, C14) Why these four tools, and how was the survey done?
- Likely question: "You survey four tools out of dozens and conclude that the literature lacks something. What was your search method?"
- Answer: The survey is purposive, not systematic. It covers three widely used tools (Dependency-Track, grype and syft; thousands of GitHub stars each) and the CycloneDX project's own viewer, because that viewer is the closest structural tool. Garcia et al. 2025 classify 84 open-source SBOM tools and have no category for analysing the dependency structure an SBOM records (Ch1:29). The gap claims are now hedged "to our knowledge" (E6).

### V2 (from C6a) Many real SBOMs have no dependency edges
- Likely question: "Zięba-Kozarzewski 2026 finds that 52.9 % of SBOMs in the wild declare no edges. Isn't SBOM Lens useless on them?"
- Answer: On such SBOMs a structural analysis has nothing to work with, and SBOM Lens says so rather than inferring "no path, so unreachable". The evaluation generates its SBOMs from lockfiles and rejects 24 candidates whose root has no dependency edges (Ch7:40) and 103 with an empty production graph (Ch7:39). Generator quality is a precondition, not something the tool assumes. Citing the paper in Ch7's threats to validity is part of D1.

### V3 (from C15) Was the SolarWinds artefact not inside Orion's dependency graph?
- Likely question: "The trojanised DLL is a component of Orion. Why is SUNBURST outside SBOM Lens's scope?"
- Answer: From the vendor's side it is a component, but nothing in an SBOM changed except a hash that had no clean reference to compare with. So SBOM analysis could not detect it, and E14 now says this precisely. From the customers' side, Orion is a deployed product, which is a portfolio question (GUAC). Ch3:94 frames it from the customers' side, and that framing can stand.

## Holds
- None. Each objection is supported by the code, the sources or the data. Where the defender rebutted part of an item (C1, C4, C5, C6, C13, C14, C15, C16), the rebuttal narrows the fix but does not dismiss it.

## Decisions needed
1. **Cite the recent SBOM-graph work and Garcia 2025 in Ch3 (E6/C6).**
   - (A) Hedge only (the E6 fixes). The two 2026 papers stay uncited, and an examiner who knows them may raise them.
   - (B) Hedge, and add at Ch3:138, after "with these methods": "Recent work has begun to study SBOMs as graphs --- whether their dependency edges are declared at all \citep{ZiebaKozarzewski2026}, and learned prediction of vulnerable components over enriched SBOM graphs \citep{Baird2026} --- but not with the network-science measures above." This needs two new bib entries: `@Misc{ZiebaKozarzewski2026, author={Zięba-Kozarzewski, Artur}, title={No Edges, No Verdict: A Large-Scale Empirical Study of Declared Dependency Graphs in 78K {SBOMs} in the Wild}, year={2026}, eprint={2607.22140}, archiveprefix={arXiv}}` and `@Misc{Baird2026, author={Baird, Laura and Moin, Armin}, title={Cascaded Vulnerability Attacks in Software Supply Chains}, year={2026}, eprint={2601.20158}, archiveprefix={arXiv}}`. Optionally, also cite the 52.9 % figure in Ch7's threats to validity as the reason for the edge filter.
   - (C) As B, and also add to §3.1's first paragraph: "A 2025 landscape study of 84 open-source SBOM tools \citep{Garcia2025} has no category for analysing the dependency structure an SBOM records." This reuses only what Ch1:29 already attributes to Garcia. Do not attach "most widely used" to that citation.
2. **HITS row of Table 3.2 (E13/C12).** (A) Keep the row and state the tie. (B) Delete the row. The table then shows no HITS value, though Ch6 still defines HITS.
3. **log4j depth figures (E3/C3).** (A) Keep them, labelled as the earlier core+api count (the E3 text). (B) Drop the depth and "majority indirect" figures, and keep only "over 17,000 packages, about 4 %". This is simpler, but it loses the evidence for "deep".
4. **Gap 3 and OSV-Scanner (E5/C5).** (A) Add after Ch3:136's second sentence: "The nearest exception is the experimental guided remediation of OSV-Scanner, which, for npm and Maven manifests and lockfiles, also weighs dependency depth and the number of vulnerabilities an upgrade fixes; it uses no position in the graph beyond depth." Gap 3's heading then reads as "beyond depth". (B) Leave it out. An examiner who knows OSV-Scanner may raise it, and the answer is the E5 viva answer.
5. **Dependency-Track version in Table 3.1 (E10/C8).** (A) Keep "(5.1+)" in the cell only. (B) Also add to the caption: "Capabilities as of October 2026 (Dependency-Track 5.2)." This makes clear that the table describes the current tools, while the 4.14 line, which is still maintained, lacks KEV.
