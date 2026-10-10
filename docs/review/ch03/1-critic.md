# Chapter 3 — Critic

What I checked and how. I read Chapters/03-Related-Work.tex in full and cross-checked it against Ch1, Ch2, Ch6, Ch7 and CONTEXT.md. For the third-party tools I read the current primary sources: the Dependency-Track README, docs and GitHub issues/release notes (latest release 5.2.0, 2026-10-08); the grype README and Anchore docs (v0.120.1), plus the vunnel provider list that builds grype's database; the syft README (v1.54.1); a clone of CycloneDX/Sunshine (HEAD 2026-09-28) with its `sunshine.py`; and the GUAC repo (v1.1.0). For the papers I downloaded and ran `pdftotext` on Kikas 2017 (TU Delft accepted manuscript), Decan 2018 (UMONS copy), Decan 2019 (arXiv 1710.04936), Zimmermann 2019 (USENIX), Liu 2022 (arXiv 2201.03981), Pashchenko 2018 (arXiv 1808.09753) and Ponta 2018 (arXiv 1806.05893). For Valverde 2002 and Myers 2003 I read the abstracts. I opened the Google log4j blog post and CISA AA20-352A. I recomputed Table 3.2 by running the backend at `28113e7` (`git archive 28113e7 backend`, `_build_graph`, `get_node_impact`, `get_global_analysis`, `_compute_epidemic`, `nx.core_number`) on `public/test-sboms/java-vulnerable.json @28113e7`, using the tool venv (NetworkX 3.7). Code references marked `@28113e7` come from that commit. Scratch files are in /tmp/claude-1000/ch03-critic/.

## C1. Sunshine does compute, and show, which components are transitively affected by each vulnerability. This contradicts the description of Sunshine and Gaps 1–2
- Type: error
- Origin: pre-existing
- Severity: high
- Thesis: Chapters/03-Related-Work.tex:18 — "no graph algorithm is run over the structure"; :20 — "the SBOM's dependency relationships are either discarded, displayed, or at most traversed as a tree. None treats them as a graph to be analysed"; :134 — "When a vulnerability is found, the tools report the affected component and its severity. They do not report which parts of the application depend on it"
- Evidence: In `CycloneDX/Sunshine/sunshine.py`, `ChildrenGatherer.get_children` (lines 1530–1588) walks the `dependsOn` graph recursively from the root. It detects cycles (:1545, :1568), records every depth at which a component appears (:1544), and passes vulnerabilities upwards to every consumer through `add_transitive_vulnerabilities_to_component` (:1513–1516, :1554–1557). The generated report has a "Vulnerabilities table" which "shows the components that are affected either directly or transitively", with columns "Directly vulnerable components" and "Transitively vulnerable components" (:388, :398). The component table also has a "Transitive vulnerabilities" column (:1701). For each vulnerability, the transitively vulnerable components are the ancestors of the vulnerable node, i.e. the dependents-only reach that SBOM Lens calls the blast radius.
- Why it matters: Gap 2 is the chapter's main claim of novelty for Chapter 5, and the tool the chapter calls the closest structural viewer already covers part of it. The text should say what Sunshine does compute (the transitive dependents of each vulnerability, as a list). The novelty can then rest on what Sunshine lacks: dominated sets, paths shown on a graph rather than as duplicated rings, and the structural measures.

## C2. The table says Dependency-Track supports SPDX. It does not: it consumes CycloneDX only, and SPDX support was removed
- Type: error
- Origin: pre-existing
- Severity: high
- Thesis: Chapters/03-Related-Work.tex:32 — "Dependency-Track & CycloneDX (preferred), SPDX"
- Evidence: The Dependency-Track README feature list says "Consumes and produces CycloneDX Software Bill of Materials (SBOM)". SPDX appears only as "Supports standardized SPDX license ID's". GitHub issue DependencyTrack/dependency-track#1746 ("Add support for SPDX v3", still open) states: "SPDX support was removed for technical (and other) reasons from prior versions of DT". docs.dependencytrack.org lists CycloneDX as the only SBOM format. None of the 5.x release notes (5.0.0 to 5.2.0) mention SPDX ingestion, only SPDX licence-list bumps.
- Why it matters: This is a factual error in the comparison table. An examiner who uses Dependency-Track will notice it. The cell should read "CycloneDX only".

## C3. The log4j ecosystem figures are ones the cited source itself withdrew: they count dependents of log4j-api as well. The corrected figure is about 17,000 packages (about 4%)
- Type: error
- Origin: pre-existing
- Severity: high
- Thesis: Chapters/03-Related-Work.tex:102 — "35{,}863 Java artifacts --- more than 8\% of the repository --- depended on an affected version of log4j ... for more than 80\% of them the vulnerable package sat more than one level deep, in some cases nine \citep{GoogleLog4j2021}"
- Evidence: The post (security.googleblog.com/2021/12/understanding-impact-of-apache-log4j.html) opens with: "Editors Note: The below numbers were calculated based on both log4j-core and log4j-api, as both were listed on the CVE. Since then, the CVE has been updated with the clarification that only log4j-core is affected. The ecosystem impact numbers for just log4j-core, as of 19th December are over 17,000 packages affected, which is roughly 4% of the ecosystem." The 35,863, 8%, ">80% more than one level deep" and "nine levels" figures are all in the body, which uses the superseded basis. The very next sentence of the thesis (:102) says the question is whether an application contains `log4j-core`, and Table 3.2 treats `log4j-api` as not vulnerable.
- Why it matters: A headline number is quoted from a source that corrects it in its first paragraph. The text should use the log4j-core figure (over 17,000, about 4%), or say that the depth histogram was computed on the core+api basis.

## C4. "Never turned into a graph on which algorithms run" is contradicted by GUAC, which the chapter itself surveys, and by Dependency-Track
- Type: overstated
- Origin: pre-existing
- Severity: high
- Thesis: Chapters/03-Related-Work.tex:132 — "Dependency relationships are discarded, displayed as a tree, or carried along unchanged, but never turned into a graph on which algorithms run"; :134 — "They do not report which parts of the application depend on it, through which paths"; :14 — "the tree is a navigation aid, not an object of computation"
- Evidence: (a) GUAC: `cmd/guacone/cmd/patch.go` defines `patch plan [flags] purl`, "query which packages are affected by the vulnerability associated with specified packageName or packageVersion", with a `search-depth` flag. It is backed by `pkg/guacanalytics/patchPlanning.go`, a BFS (`BfsNode{Parents, Depth}`) over the dependency graph. Google's GUAC announcement describes the goal as indicating "the blast radius of a bad package or vulnerability" for a patch plan. (b) Dependency-Track: frontend PR #667 (merged 2023-12-10) adds a "Show in Dependency-Graph" button in a vulnerability's Affected Projects list. It "redirects the user to the projects dependency graph and highlights the affected component". It is backed by `GET /api/v1/component/project/{projectUuid}/dependencyGraph/{componentUuids}` (issue #7636). So DT does compute and show the path from the project to a vulnerable component. (c) Outside the survey, `npm explain` "will print the chain of dependencies causing a given package to be installed in the current project" (docs.npmjs.com).
- Why it matters: Gap 1 and Gap 2 are stated as absolutes, and the chapter's own survey contradicts them. GUAC is set aside at :10 and :138 on scope (portfolio, not single SBOM), but GUAC can ingest one SBOM, and the gaps are not worded in terms of scope. The defensible claim is narrower. No surveyed tool computes dominated sets or structural measures on the single-SBOM graph, and none shows the vulnerability on the graph together with its reach.

## C5. Gap 3 ignores existing structural and exploitability prioritisation in open-source tools: grype ranks by a combined risk score, and OSV-Scanner prioritises by dependency depth
- Type: omission
- Origin: pre-existing
- Severity: medium
- Thesis: Chapters/03-Related-Work.tex:136 — "Findings are ranked by CVSS score, sometimes refined by exploit likelihood (EPSS), which describe the vulnerability rather than where the affected component sits"
- Evidence: The Anchore docs ("Interpreting the results", oss.anchore.com/docs/guides/vulnerability/interpreting-results/) say that grype by default "sorts results by risk score rather than single metrics", combining EPSS or CISA KEV (KEV "treated as maximum threat") with CVSS impact. OSV-Scanner guided remediation (google.github.io/osv-scanner/experimental/guided-remediation/) analyses "the entire transitive graph". It offers "Prioritising direct dependency upgrades by the total number of transitive vulnerabilities fixed" and "Prioritising vulnerabilities by dependency depth, severity". OSV-Scanner is not surveyed, although SBOM Lens relies on OSV.
- Why it matters: Depth-based and fix-count-based prioritisation is a weak but real form of structural prioritisation, already in a widely used open-source tool from the same OSV project. Gap 3 should acknowledge it and place SBOM Lens's measures (centrality, spectral exposure, cores, communities) beyond depth.

## C6. Gap 4 and §3.2.4 claim the single-SBOM graph "has not been systematically analysed". Recent SBOM-graph studies exist, and Garcia et al. 2025 is not used in Ch3
- Type: omission
- Origin: pre-existing
- Severity: medium
- Thesis: Chapters/03-Related-Work.tex:78 — "What the literature has not done is apply this toolkit systematically to the dependency graph of a single application"; :138 — "The dependency graph of a single application, as described by its SBOM, has not been systematically analysed with these methods"; :130 — "four capabilities absent from current tooling and the research literature"
- Evidence: (a) Zięba-Kozarzewski, "No Edges, No Verdict: A Large-Scale Empirical Study of Declared Dependency Graphs in 78K SBOMs in the Wild", arXiv 2607.22140 (July 2026). It characterises the declared dependency-graph topology of 77,092 SBOMs and finds that 52.9% declare no edges, and that Syft-generated container SBOMs leave 91.4–98.0% of components orphaned. It frames SBOMs as "consumed ... as dependency graphs: vulnerability triage, reachability filtering, and impact analysis all traverse the edges an SBOM declares". (b) Baird and Moin, "Cascaded Vulnerability Attacks in Software Supply Chains", arXiv 2601.20158 (January 2026), models enriched SBOMs as heterogeneous graphs for vulnerability analysis. (c) Liu et al. 2022 (cited at :60) resolve a dependency tree per root package and analyse per-package "vulnerable paths" (Finding-2: "one vulnerable point introduce[s] 8 vulnerable paths" on average), i.e. single-project graphs aggregated over the ecosystem. (d) Garcia et al. 2025 (Bibliography.bib:100, cited in Ch1:29) surveys 84 open-source SBOM tools. Ch3 surveys four and does not cite it. The chapter describes no search method, so "absent from ... the research literature" rests on what was surveyed.
- Why it matters: The gaps are claimed as absences, but the survey does not show the absence. (a) is directly on topic and also bears on a threat to validity: many real SBOMs have no usable graph. Citing Garcia 2025 in §3.1 would give the four-tool survey a basis for being representative. Hedging Gap 4 to "not, to our knowledge, with these network-science measures as a routine part of SBOM analysis" would make it defensible.

## C7. The grype database is not built from OSV, and its main sources (distribution feeds) are omitted
- Type: error
- Origin: pre-existing
- Severity: medium
- Thesis: Chapters/03-Related-Work.tex:33 — "Aggregated DB (NVD, OSV, GHSA), EPSS, CISA KEV"
- Evidence: grype's database is built by vunnel. Its provider directory (`anchore/vunnel/src/vunnel/providers`) contains alma, alpine, amazon, arch, bitnami, chainguard, chainguard_libraries, debian, echo, eol, epss, fedora, github, govulndb, hummingbird, kev, mariner, minimos, nvd, oracle, photon, rhel, rocky, secureos, sles, ubuntu and wolfi. There is no `osv` provider. Anchore's docs list NVD, GHSA and distribution-specific sources (Alpine, Debian, Ubuntu, RHEL, SUSE, Amazon Linux, Oracle). Some providers (alma, rocky, bitnami, govulndb) publish in the OSV *format*, but osv.dev is not a source. EPSS and KEV are correct.
- Why it matters: The cell should read e.g. "NVD, GHSA, OS-distribution feeds; EPSS, KEV". As written, it suggests grype and SBOM Lens draw on the same source.

## C8. The Dependency-Track vulnerability-source cell is out of date: KEV (since 5.1.0, August 2026), OSS Index, Trivy and JVN are missing
- Type: error
- Origin: pre-existing (KEV became true only after DT 5.1.0, 2026-08-27)
- Severity: medium
- Thesis: Chapters/03-Related-Work.tex:32 — "NVD, OSV, GHSA, EPSS, Snyk, VulnDB"
- Evidence: The README lists NVD, GitHub Advisories, Sonatype OSS Index, Snyk, Trivy, OSV and VulnDB, plus EPSS. The 5.1.0 release notes include "Implement initial KEV support" (#6516), "Add VulnCheck KEV data source" (#7027) and "Add JVN (Japan Vulnerability Notes) vulnerability data source" (#6640). The current release is 5.2.0 (2026-10-08). The bib entry has `urldate = {2026-10-09}`, so the table claims to reflect the current version.
- Why it matters: The table credits grype and Sunshine with KEV but not Dependency-Track, which now has it. That makes Dependency-Track look less capable than it is.

## C9. The Ponta et al. citation does not support "identifies many reported vulnerabilities as unreachable"
- Type: misattribution
- Origin: pre-existing
- Severity: medium
- Thesis: Chapters/03-Related-Work.tex:68 — "\citet{Ponta2018} ... showed that such code-centric analysis identifies many reported vulnerabilities as unreachable"
- Evidence: Ponta et al. (arXiv 1806.05893) report only positive reachability results: "vulnerable constructs are reachable, statically or dynamically, in 131 out of 496 applications ... 390 pairs of applications and vulnerabilities whose constructs were reachable. In 32 cases, the reachability could only be determined through the combination of techniques, which represents a 7.9% increase" (§IV). The paper gives no count or share of reported vulnerabilities found unreachable. Its quantitative contribution is that combining static and dynamic analysis finds more reachable cases.
- Why it matters: The sentence uses the paper as evidence for a finding it does not report. Either cite a study that measures unreachability, or reword to "determines whether the vulnerable code is reachable, and through which call paths".

## C10. "Have established that open-source ecosystems form large, scale-free graphs" is not what the three cited studies show, and it sits two paragraphs before the chapter's own warning (Clauset 2009) about such claims
- Type: overstated
- Origin: pre-existing
- Severity: medium
- Thesis: Chapters/03-Related-Work.tex:52 — "Empirical studies of package dependency networks have established that open-source ecosystems form large, scale-free graphs"; cf. :78 — "\citet{Clauset2009} showed that many reported power laws do not withstand rigorous fitting"
- Evidence: Kikas 2017 (full text) does not claim or test scale-freeness. Its findings are about growth of transitive dependencies and removal vulnerability. Decan 2019 (§ discussion) "hypothesise[s]" that ecosystems show complex-network behaviour and reports "a very unequal distribution of connectivity ... characteristic of power law or Pareto law behaviour", with no power-law fit. Zimmermann 2019 mentions a power law only when citing prior work (Wittern et al.: "exhibiting a similar effect to a power law distribution"). None of the three fits a degree distribution.
- Why it matters: The chapter warns that power-law claims need rigorous fitting, and its own lead sentence makes one without that support. "Heavy-tailed" or "highly unequal", which the sources do support, is enough for the argument. The same applies to Ch7:90, "the properties the literature reports for entire ecosystems: ... small-world clustering". None of the cited ecosystem studies reports clustering.

## C11. Ch1 calls Sunshine a flat-inventory tool; Ch3 calls it the one tool whose purpose is to show dependency structure
- Type: inconsistency
- Origin: pre-existing
- Severity: medium
- Thesis: Chapters/01-Introduction.tex:13 — "Dependency-Track, grype, syft, and Sunshine/CycloneDX --- treat SBOMs as flat component inventories"; Chapters/03-Related-Work.tex:18 — "It is the only surveyed tool whose primary purpose is to show the dependency structure"
- Evidence: Both sentences are as quoted. C1 shows that Sunshine also propagates vulnerabilities through the structure. Ch1:81 also says "Sunshine/CycloneDX", while Ch3 names the tool "Sunshine (CycloneDX project)".
- Why it matters: The two chapters describe the same tool in opposite ways. This is not the Ch1:29 sentence, which the author has already settled.

## C12. Table 3.2's HITS authority value is a six-way tie and does not discriminate. The only component with two consumers scores 0
- Type: omission
- Origin: pre-existing
- Severity: low
- Thesis: Chapters/03-Related-Work.tex:119 — "HITS authority & 0.167 (sum over all components: 1)"
- Evidence: Recomputed with `get_global_analysis @28113e7` (`impact.py:91-95`). All six direct dependencies of `my-spring-app` (log4j-core, spring-webmvc, jackson-databind, commons-text, struts2-core, h2) have authority 0.166667. All other nodes have 0, including `commons-collections`, the only node with in-degree 2. The principal eigenvector of AᵀA sits entirely on the root's star.
- Why it matters: The value is correct, but a reader will take 0.167 as information about log4j-core when it is just 1/6. Either say "tied with the other five direct dependencies", or drop the row. The other rows carry a comparison ("maximum in the graph") and this one does not.

## C13. The research reviewed in §3.2.2 is not all ecosystem-scale: Pashchenko and Ponta analyse individual libraries and applications
- Type: overstated
- Origin: pre-existing
- Severity: low
- Thesis: Chapters/03-Related-Work.tex:62 — "Like the studies of the previous subsection, however, it measures propagation across an ecosystem"; :134 — "The research that models vulnerability propagation does so across whole ecosystems"
- Evidence: Pashchenko 2018 studies "the 200 most popular OSS Java libraries used by SAP" and counts vulnerable dependencies per library (abstract). Ponta 2018 scans "about 500 applications" one at a time. Liu 2022 resolves one dependency tree per root package (C6c).
- Why it matters: The scale argument is the chapter's main way of separating SBOM Lens from prior work. The accurate distinction is per-project *reachability or counting*, not *no per-project analysis*.

## C14. Several citations do not support the specific sentence they are attached to
- Type: unsupported
- Origin: pre-existing
- Severity: low
- Thesis: Chapters/03-Related-Work.tex:100 — "the log4shell vulnerability exposed the hidden scale of transitive dependency risk \citep{NIST_CVE_2021_44228}"; :10 — "joined OpenSSF as an incubating project in March 2024 \citep{GUAC2024}"; :10 — "Four open-source tools are in widespread production use"
- Evidence: The NVD CVE-2021-44228 page is a vulnerability record. It says nothing about transitive dependency scale; GoogleLog4j2021 does. The GUAC date is correct (OpenSSF blog and InfoQ, 7 March 2024), but the cited `https://guac.sh` home page is not a source for it. "Widespread production use" has no citation, and Sunshine has 126 GitHub stars (GitHub API, 2026-10-09).
- Why it matters: These are minor. Move the NVD citation to the CVE identifier only, cite the OpenSSF announcement for the date, and drop "widespread production use" or support it.

## C15. "No analysis of that SBOM ... could have detected the attack" overlooks that the trojanised DLL is a component whose hash would change. The DLL also sits inside Orion's own dependency graph
- Type: overstated
- Origin: pre-existing
- Severity: low
- Thesis: Chapters/03-Related-Work.tex:92 — "An SBOM of Orion would have listed the same components before and after the injection, and no analysis of that SBOM --- flat or graph-based --- could have detected the attack"; :94 — "the compromised artefact was a product deployed across an organisation rather than a library inside one application's dependency graph"
- Evidence: CISA AA20-352A: "The adversary added a malicious version of the binary solarwinds.orion.core.businesslayer.dll into the SolarWinds software lifecycle, which was then signed by the legitimate SolarWinds code signing certificate". The alert also says affected platform versions "share the same DLL version number". So names and versions are unchanged, but a CycloneDX `hashes` field would differ. Detecting the change needs a trusted reference build, which is the provenance point the text already makes. From the vendor's side, the compromised artefact is a component inside Orion's dependency graph.
- Why it matters: The conclusion (detection needs provenance controls, not SBOM graph analysis) holds. "Same components" and "no analysis could" are stronger than needed. "Same component names and versions" would be exact.

## C16. Small framing inconsistencies in the chapter introduction
- Type: inconsistency
- Origin: pre-existing
- Severity: low
- Thesis: Chapters/03-Related-Work.tex:4 — "weaving in the CVSS/EPSS scoring context established in \autoref{cp:background}"; :4 and :81 — "a pair of supply chain incidents" / "Supply Chain Attack Case Studies"; :109 and :104 — "13 components"
- Evidence: §3.1 contains no CVSS/EPSS discussion apart from the EPSS cells of Table 3.1. Gap 3 (:136) is the only place it comes up. log4shell was a vulnerability disclosure, not an attack on the supply chain, as :97 itself says ("disclosure"). The Java SBOM has 12 entries in `components` plus the `metadata.component` application. `_build_graph` (`sbom_store.py:14-35 @28113e7`) turns these into 13 nodes, so "13 components" counts the application as a component.
- Why it matters: These are wording issues. "13 nodes (the application and 12 components)" would match how CycloneDX counts.

## Checked and holds
- Table 3.2, every row, recomputed at `28113e7` (13 nodes, 13 edges). Blast radius = {my-spring-app, log4j-core} (`impact.py:50`). Path = [my-spring-app → log4j-core] (`impact.py:53-64`). Dominated set = {log4j-api} (`impact.py:11-38`). log4j-core is in `articulationPoints` (undirected). Authority 0.166667, sum 1.000002. Spectral score 1.1504, maximum 2.7653 at my-spring-app = λ1 (`graph_analysis.py:343-364`, consistent with Ch6:42-46). Core number 1, max 2. "Includes the vulnerable node" agrees with the author's decision.
- The SBOM Lens row of Table 3.1. The measures listed match `impact.py` and `graph_analysis.py @28113e7` (articulation points, HITS, eigen/spectral, `core_number`, `louvain_communities(seed=42)`, whole-graph metrics). Input is JSON only (`sboms.py:25-29`). OSV + CVSS v3.1 match Ch5.
- Sunshine cells: CycloneDX JSON only (`index.html:229` tells users to convert SPDX and CycloneDX XML first). EPSS and CISA KEV enrichment (README, `-e`). SQL query interface ("Query my SBOM ... using SQL", `sunshine.py:1101-1115`). Sunburst chart (`type: 'sunburst'`). A component on several paths is drawn several times (recursive per-path children, `sunshine.py:1530-1560`). CycloneDX-project ownership.
- syft row: CycloneDX, SPDX and Syft JSON output plus conversion (README). No vulnerability analysis. Images, filesystems and directories as sources.
- grype: scans images, filesystems and SBOMs (README). Reads SPDX (JSON/XML/tag-value) and CycloneDX (JSON/XML) (Anchore scan-targets doc). EPSS and KEV present.
- Dependency-Track: OWASP Flagship (README badge). Portfolio-wide continuous monitoring. EPSS, NVD, OSV, GHSA, Snyk and VulnDB are all sources. It has a dependency-graph view.
- GUAC joined OpenSSF as an incubating project on 7 March 2024 (OpenSSF blog, InfoQ). Only the citation is weak (C14).
- Kikas 2017: JavaScript, Ruby and Rust; "at least one single package whose removal can affect more than 30% of projects"; transitive dependencies grew 60% in a year.
- Decan 2019: seven ecosystems including npm; growth; "high and increasing number of transitive dependencies".
- Zimmermann 2019: both clauses are near-verbatim from the abstract ("individual packages could impact large parts of the entire ecosystem ... a very small number of maintainer accounts could be used to inject malicious code into the majority of all packages").
- Decan 2018: dependents remain exposed because of constraints ("More than 40% of all package releases depending on a vulnerable package release cannot be fixed automatically ... because the imposed dependency constraints do not allow them").
- Liu 2022: dependency trees resolved with npm's rules; transitive propagation; constraints and maintenance block fixes (Finding-5).
- Pashchenko 2018: test/provided-scope dependencies are not deployed; about 20% of vulnerable dependencies are not deployed. The link to Ch7's production-graph restriction (Ch7:18) is consistent.
- Valverde 2002 (software architecture/class graphs, scale-free and small-world) and Myers 2003 (collaboration graphs, scale-free and small-world) match their abstracts.
- Clauset 2009 → SBOM Lens fits by MLE with KS-chosen x_min: `graph_analysis.py:113-128 @28113e7` (`powerlaw.Fit(degrees, discrete=True)`), on in-degrees, as Ch6:164 says.
- The component-level over-approximation paragraph (:70) is consistent with Ch7:157. The dominated set as "removed or replaced" (:94, :104) is consistent with CONTEXT.md.
- The Google blog figures (35,863; >8%; majority indirect; >80% more than one level deep; up to nine) are quoted accurately from the blog body. The problem is the editor's correction (C3). The authors (Wetter, Ringland) are correct.
- CISA AA20-352A supports "legitimately signed".
- Unverified: Louridas 2008 "norm rather than the exception". The paper is closed-access and I could not open the text. The general finding (power laws across many software structures) is how the paper is commonly summarised.
