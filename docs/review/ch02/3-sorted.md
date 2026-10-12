# Chapter 2 — Sorted review

## Summary
- 21 items (C1–C19, D-new1, D-new2). Sorted into **20 errata**, **3 viva prep** and **0 whole-item holds**. Every item has at least one sub-point that must change, so none is viva-only. The viva-prep entries are the sub-points the defender successfully defended, and they need no text change. Merges: E6 merges C10 and C13. E15 takes the attack-section sourcing points of C19. The C19(c) example (`requests-oauthlib2`) is fixed in E6. Nine rebutted or moot sub-points are listed under Holds.
- Errata by impact: E1 (C1), E2 (C2), E3 (C3), E4 (C4), E5 (C9), E6 (C10+C13), E7 (C5), E8 (C6), E9 (C7), E10 (C8), E11 (C11), E12 (C12), E13 (C14), E14 (C18), E15 (C19), E16 (C15), E17 (C16), E18 (C17), E19 (D-new1), E20 (D-new2). Viva prep: V1 (C10, :208), V2 (C15 side note, Ch5:20/Ch8:41), V3 (C2, background concepts the tool does not compute).
- **Results.** No reported number changes and no RQ conclusion changes. Ch2 reports no evaluation result. The fixes bring in four figures that already appear elsewhere or are sourced: 89 % transitive (Ch7:73), "over 17,000" Maven packages (Ch3:102), 45 SPDX relationship types (schema), and "up to 18,000" SolarWinds customers (SEC 8-K).
  - **Abstract EN/PT: unchanged.** Neither says anything about regulation, SPDX, NVD or cycles.
  - Other chapters touched in step: Ch1:11 (E1), Ch6:6 (E2), Ch6:144 and Ch7:75 (E3). Nothing in Ch5:20 or Ch8:41 (see V2). Ch5:91/:123 are made true by E13 without being edited.
- **Settled disputes.**
  - C1, EO 14306: the Federal Register citation is **90 Fed. Reg. 24723** (Vol. 90, No. 111, 11 June 2025, pp. 24723–24726; FR doc. 2025-10804, govinfo). It was signed 6 June 2025. Sec. 1 strikes EO 14144 subsections 2(a)–(b) and does not mention SBOMs. The defender is right that **M-26-05 is dated 23 January 2026**. I read the memo PDF: it is signed by Russell T. Vought and rescinds M-22-18 and M-23-16. Its text: "Agencies may also choose to adopt contractual terms that require a software producer to provide a current software bill of materials (SBOM) upon request." CRA Art. 71: the regulation applies from 11 Dec 2027, Art. 14 from 11 Sep 2026 (so the reporting obligations already apply today) and Chapter IV from 11 Jun 2026. The AI Digital Omnibus did not move the SBOM date. EO 14028 is 86 Fed. Reg. 26633, which matches the existing bib note.
  - C19(b), SolarWinds: **CISA AA20-352A gives no number of organisations.** It confirms only that the malicious DLL was signed with the SolarWinds certificate and shipped in Orion releases. The 18,000 comes from SolarWinds' Form 8-K of 14 Dec 2020, Item 8.01: "fewer than 18,000" customers "may have had an installation" of the affected products. So `CISA_AA20_352A` must not be cited for the figure. The 8-K must be cited, or the figure must go.
  - C18(e), "Impact ≤ 0": the FIRST v3.1 specification §7.1 reads "If Impact ≤ 0: 0, else …". The source is `CVSS31`, which is already in the bib and already cited at :258, so no new source is needed. With S:C and ISS = 0, Impact ≈ −0.218, so "= 0" misses that case. The code's `ISS === 0` test (`vulnerabilityAPI.js:39 @28113e7`) is equivalent, because the smallest non-zero ISS is 0.22.
  - C14: the §7.4 weights in the spec are AV N/A/L/P 0.85/0.62/0.55/0.2; AC 0.77/0.44; PR N 0.85, L 0.62 (0.68 S:C), H 0.27 (0.5 S:C); UI 0.85/0.62; C/I/A H 0.56, L 0.22, N 0. They match `vulnerabilityAPI.js:25-33 @28113e7` exactly.
  - C16: neither side found the **final CISA 2026 Minimum Elements for an SBOM (29 July 2026)**, co-authored with NSA, FBI and 15 international agencies (I read the PDF). It supersedes the August 2025 draft that the critic and defender cite. It adds Component Hash, Component License, SBOM Tool Name and SBOM Generation Context. It keeps "Component Dependency Relationship", which "supports the capability to build a dependency graph". Cite the 2026 document, not the draft. M-26-05 itself points to the 2025 draft, which was current in January. SLSA v1.2 was released on 24 Nov 2025 and adds a Source track (slsa.dev).
  - C4: I re-ran the script on the SPDX 2.3 schema. `relationshipType` has 45 values, 9 of them dependency types, and `packages` and `relationships` are separate top-level properties. In the CycloneDX 1.7 schema, `component.required = ["type","name"]` and `bom-ref` "must be unique within the BOM". `vulnerabilities` is absent from the 1.3 schema and present in 1.4.
  - C13: the defender is right about the tree (I read arXiv 2204.04008). The top-level nodes are AV-100 (distinct malicious package), AV-200 (name confusion) and AV-001 (subvert legitimate package). Dependency confusion is AV-509, under AV-500. Typosquatting is AV-204, with the example "dajngo vs. django". Ladisa et al. call an added prefix or suffix *combosquatting*. The critic's conclusion holds: the four headings are not "primary categories".
  - C2: NetworkX 3.7's `immediate_dominators` docstring cites Cooper–Harvey–Kennedy (2006, `hdl.handle.net/1911/96345`) and Lengauer–Tarjan (TOPLAS 1(1):121–141, 1979, doi 10.1145/357062.357071). The implementation is the iterative Cooper–Harvey–Kennedy algorithm (DFS post-order plus intersect). The defender is right that Ch5:240 defines the dominated set operationally. Ch2 has no definition, and the thesis has no citation for it anywhere.
  - C18(c): NetworkX 3.7 `strongly_connected_components` is "Iterative version of Tarjan's algorithm" (with the Tarjan–Zwick 2024 improvements). Naming Tarjan is right for the tool. CLRS presents the two-pass algorithm, so the citation is what is wrong.
  - C9: I could not read Barabási (2016) ch. 9 (the site serves it through JavaScript), so the attribution of 0.3 stays unverified. The point is moot, because the threshold is deleted (E5).
  - C15: OSV `docs/data.md` (master, today) lists 19 native entries, one of them a duplicate (Haskell), plus 3 converted feeds (Debian, Alpine, NVD CVEs for open-source software). There is no "npm advisories" source.
  - C6: NIST, "NIST Updates NVD Operations to Address Record CVE Growth", 15 April 2026. It prioritises KEV, federal-use and EO 14028 critical software, and marks other CVEs "Lowest Priority". It says NIST "will no longer routinely provide a separate severity score" where a CNA score exists.
  - C7: the CVSS v4.0 specification (FIRST, released Nov 2023; document v1.2, 2024-06-18) has the base metrics AV, AC, AT, PR, UI, VC/VI/VA, SC/SI/SA. Scope is gone.
  - Recomputed from `per_sbom.csv`, and in agreement with both sides: 30 of 90 graphs with ≥ 1 cycle and ≥ 1 non-trivial SCC (max 16 cycles, 5 SCCs); median directShare 0.1148 (≈ 7.7 : 1), with 58 graphs above 1/11; 21 graphs with more than 500 nodes (max 995); lockfiles 53 npm / 36 yarn / 1 pnpm.
  - New bib keys, all checked absent from `Bibliography.bib`: `OMB_M2605`, `CRA2024` (exists, reused), `LengauerTarjan1979`, `Cooper2006`, `NVD2026`, `CVSS40`, `SolarWinds8K2020`, `CISA2026SBOM`. Only if the option is chosen: `EO14306` (D1-B) and `Brandes2001` (D3-B). The defender's `CISA2025SBOM` is **replaced** by `CISA2026SBOM`.
- **Four decisions are needed** (list at the end): how much regulatory history to write (D1), where dominance is defined (D2), whether to keep betweenness (D3), and how far to rewrite the attack taxonomy (D4).

## Erratum

### E1 (from C1) The regulatory claims are wrong or out of date: EO 14028 did not mandate SBOMs, the US now leaves them to each agency, and the CRA SBOM duty starts in December 2027 — impact: high (the chapter's and Ch1's regulatory motivation)
- Settled facts:
  - EO 14028 Sec. 4(e)(vii) directs NIST to issue *guidance* that includes "providing a purchaser a Software Bill of Materials (SBOM)". No section mandates SBOMs. Sec. 10(j) gives the definition (86 Fed. Reg. 26633).
  - EO 14144 contained no explicit SBOM mandate. Its Sec. 2(a)–(b), which held the attestation and artifact requirements, were struck by EO 14306 (6 June 2025, 90 Fed. Reg. 24723).
  - OMB M-26-05 (23 Jan 2026) rescinds M-22-18 and M-23-16. Agencies "may also choose to adopt contractual terms that require a software producer to provide a current [SBOM] upon request".
  - CRA (Reg. 2024/2847): in force 10 Dec 2024. Art. 14 reporting applies from 11 Sep 2026, and everything else, including Annex I Part II(1), from 11 Dec 2027. Annex I Part II(1) requires an SBOM "covering at the very least the top-level dependencies".
  - :20 cites `EO14028` for the EO 14144 claim. "Two largest software markets" has no source. The same overstatement appears at :20, :56 and :88, and at Ch1:11.
- Fix (recommended D1-A, which drops EO 14144 from the thesis):
  - Ch2:20 — "US Executive Order 14028 (May 2021) mandated SBOMs for software sold to the US federal government; EO 14144 (January 2025) extended and reinforced these requirements \citep{EO14028}. The European Cyber Resilience Act (CRA), which entered into force in December 2024, similarly requires software component transparency for products with digital elements sold in the EU market \citep{CRA2024}. SBOMs have therefore moved from a voluntary best practice to a regulatory requirement in the two largest software markets in the world." → "US Executive Order 14028 (May 2021) directed NIST to issue guidance that includes providing the purchaser of a software product with an SBOM \citep{EO14028}; since January 2026, each federal agency decides whether to require one from its vendors \citep{OMB_M2605}. The European Cyber Resilience Act (CRA), in force since December 2024, will require manufacturers of products with digital elements sold in the EU to draw up an SBOM covering at least the top-level dependencies of each product from December 2027 \citep{CRA2024}. SBOMs are therefore becoming a legal requirement in the EU and remain an expected practice in US federal procurement. An SBOM that lists only top-level dependencies, however, does not record the dependency graph that SBOM Lens analyses."
  - Ch2:56 — "summarises the regulatory responses that have raised SBOM analysis from an optional practice to a compliance requirement." → "summarises the regulatory and standards responses to these threats."
  - Ch2:88 — "US Executive Order 14028 (May 2021) mandated SBOMs for software sold to the US federal government, with Section 10(j) providing the authoritative regulatory definition of SBOM \citep{EO14028}. EO 14144 (January 2025) reinforced and extended these requirements." → "US Executive Order 14028 (May 2021) directed NIST to issue software supply chain guidance that includes providing SBOMs to purchasers, and its Section 10(j) gives the US definition of an SBOM \citep{EO14028}. OMB Memorandum M-26-05 (January 2026) rescinded the government-wide attestation policy that followed the order and left each agency free to require an SBOM from a vendor by contract \citep{OMB_M2605}."
  - Ch2:88, the CRA sentence — "throughout the product lifecycle \citep{CRA2024}." → "throughout the product lifecycle; its vulnerability reporting obligations have applied since September 2026, and its other requirements, including the SBOM, apply from December 2027 \citep{CRA2024}."
  - **DECISION NEEDED** (D1): whether to keep the EO 14144/EO 14306 history.
- Also touches, in step (Ch1:11):
  - "Regulators around the world have responded by mandating greater transparency over software composition." → "Regulators have responded by demanding greater transparency over software composition."
  - "enforces manufacturers and developers of products with digital elements sold in the EU market to document the components of their software" → "requires manufacturers and developers of products with digital elements sold in the EU market to document, from December 2027, the components of their software"
  - "Executive Order 14028, signed in May 2021, similarly mandated that software supplied to the federal government be accompanied by an SBOM, defining it as" → "Executive Order 14028, signed in May 2021, made the SBOM part of federal guidance on software supply chain security, defining it as"
  - "Executive Order 14144, issued in January 2025, reinforced and extended these requirements \citep{EO14144}." → "Since January 2026, however, each federal agency decides whether to require an SBOM from its vendors \citep{OMB_M2605}."
  - "to a regulatory expectation on both sides of the Atlantic" → "to a legal requirement in the EU and an expected practice in US federal procurement"
  - Ch3:10 ("EO 14028 … accelerated adoption") is correct as it stands.
- New bib entry:
  ```bibtex
  @Misc{OMB_M2605,
      author  = {{Office of Management and Budget}},
      title   = {Adopting a Risk-based Approach to Software and Hardware Security},
      year    = {2026},
      month   = jan,
      note    = {Memorandum M-26-05, 23 January 2026},
      url     = {https://www.whitehouse.gov/wp-content/uploads/2026/01/M-26-05-Adopting-a-Risk-based-Approach-to-Software-and-Hardware-Security.pdf},
      urldate = {2026-10-12},
  }
  ```
  Only under D1-B:
  ```bibtex
  @Misc{EO14306,
      author = {{The White House}},
      title  = {Executive Order 14306: Sustaining Select Efforts to Strengthen the Nation's Cybersecurity and Amending {Executive Order} 13694 and {Executive Order} 14144},
      year   = {2025},
      month  = jun,
      note   = {90 Fed. Reg. 24723},
  }
  ```
- Viva answer: "The EU requires an SBOM from December 2027. In the US, the SBOM is expected practice that each agency may require by contract. The CRA floor is top-level dependencies only, so a compliant SBOM need not contain the graph. That is why the evaluation uses generators that emit the full graph."

### E2 (from C2) The vocabulary chapter does not define or cite dominance, and its claim that every concept "reappears … without re-definition" is false — impact: high (the distinguishing concept of the novelty statement)
- Settled facts:
  - Ch2 never mentions dominators, and no bib entry covers them. Ch5:240 defines the dominated set operationally, without a citation. Dominance is the first item of the settled novelty statement.
  - Betweenness appears in no other chapter and nowhere in `backend/ @28113e7`.
  - Bow-tie, rich club, degree entropy and spectral exposure are first defined in Ch6, so Ch2:4 and Ch6:6 overclaim.
  - `nx.descendants` is used only by the bow-tie code (`graph_analysis.py:249`), not in Ch5.
  - NetworkX 3.7's `immediate_dominators` implements Cooper–Harvey–Kennedy and cites it together with Lengauer–Tarjan.
- Fix (recommended D2-A, a paragraph):
  - Ch2:126, insert at the end of the paragraph, before "Reachability is computed…": "A vertex \(d\) \textit{dominates} a vertex \(v\) with respect to a root \(r\) if every directed path from \(r\) to \(v\) passes through \(d\) \citep{LengauerTarjan1979}. The closest strict dominator of \(v\) is its \textit{immediate dominator}, and linking every vertex to its immediate dominator gives the \textit{dominator tree}. The \textit{dominated set} of \(d\) is its subtree in that tree, without \(d\) itself: the components that the application reaches only through \(d\). SBOM Lens computes the dominator tree with \texttt{nx.immediate\_dominators}, which implements the algorithm of \citet{Cooper2006}; the dominated set is applied in \autoref{cp:vuln-analysis}."
  - Ch2:126 — "SBOM Lens computes these sets using NetworkX's \texttt{nx.ancestors} and \texttt{nx.descendants} functions. Both are applied in \autoref{cp:vuln-analysis} as a vulnerability impact quantification method." → "SBOM Lens computes the blast radius with NetworkX's \texttt{nx.ancestors} (\autoref{cp:vuln-analysis}) and uses \texttt{nx.descendants} in the bow-tie decomposition (\autoref{cp:network-sci})."
  - Ch2:4 — "Every concept defined here reappears by name in \autoref{cp:system-design}, \autoref{cp:vuln-analysis}, and \autoref{cp:network-sci} without re-definition." → "The concepts defined here are used by name in \autoref{cp:system-design}, \autoref{cp:vuln-analysis}, and \autoref{cp:network-sci}; measures specific to SBOM Lens are defined where they are introduced."
  - Betweenness (:174–178): **DECISION NEEDED** (D3).
- Also touches: Ch6:6 — "The theoretical definitions of the underlying concepts were given in" → "The standard concepts used here were defined in". Ch5:240 needs no change; optionally add "(\autoref{sec:connectivity})" after "its immediate dominator".
- New bib entries (keys absent from the bib):
  ```bibtex
  @Article{LengauerTarjan1979,
      author  = {Lengauer, Thomas and Tarjan, Robert Endre},
      title   = {A Fast Algorithm for Finding Dominators in a Flowgraph},
      journal = {ACM Transactions on Programming Languages and Systems},
      year    = {1979},
      volume  = {1},
      number  = {1},
      pages   = {121--141},
      doi     = {10.1145/357062.357071},
  }

  @Report{Cooper2006,
      author      = {Cooper, Keith D. and Harvey, Timothy J. and Kennedy, Ken},
      title       = {A Simple, Fast Dominance Algorithm},
      institution = {Department of Computer Science, Rice University},
      type        = {Technical Report},
      number      = {TR-06-33870},
      year        = {2006},
      url         = {https://hdl.handle.net/1911/96345},
  }
  ```
  Note: Cooper2006 is often cited as "Software Practice & Experience 4, 2001", but that venue record is unreliable. Cite the Rice technical report, as NetworkX does.
- Viva answer: "Dominance is the compiler-theory notion (Lengauer–Tarjan), with the application as the entry node. A component dominates another if every path from the application to it passes through the first. SBOM Lens uses NetworkX's implementation of Cooper, Harvey and Kennedy's algorithm."

### E3 (from C3) "Standard package managers report [a cycle] as an error" is contradicted by npm and by a third of the dataset — impact: high (a definitional claim refuted by the evaluation)
- Settled facts: 30 of 90 graphs have cycles, and all come from npm-family lockfiles that installed. Ch7:75 reports the same 30. Ch2:116 itself says that real SBOMs contain cycles.
- Fix: Ch2:114 — "neither can be resolved without the other, and standard package managers report this as an error." → "neither can be built or updated independently of the other. Some package managers reject such cycles; npm, the ecosystem evaluated in \autoref{cp:evaluation}, installs them."
- Also touches, in step (one graph in three is not "rare"):
  - Ch6:144 — "In a dependency graph built from a lockfile, cycles are rare and each one is worth an analyst's attention:" → "Most of the evaluated graphs have no cycle (\autoref{sec:eval-rq1}), and each one is worth an analyst's attention:"
  - Ch7:75 — "Cycles are rare but not absent: the median graph has none, but 30 of the 90 graphs" → "Cycles are not the norm but are not rare either: the median graph has none, but 30 of the 90 graphs"
- Viva answer: "npm resolves circular dependencies at install and at load time. The DAG is the ideal, not the rule, and SBOM Lens models a general digraph for that reason."

### E4 (from C4) SPDX is mischaracterised (45 relationship types, an explicit DEPENDS_ON, separate arrays) — impact: medium (the stated justification for CycloneDX-only)
- Settled facts:
  - The SPDX 2.3 schema has 45 relationship types, 9 of them for dependencies, and separate `packages` and `relationships` arrays.
  - DEPENDS_ON is explicit. The real cost is normalising several typed forms and both directions.
  - "SPDX 2.3 has no native structure for embedding vulnerability data" is defensible: there are only `externalRefs` of category SECURITY. SPDX 3.0 (2024) adds a Security profile.
  - The dataset's generator is cdxgen (CycloneDX). Ch8:69 lists SPDX input as future work.
- Fix:
  - Ch2:30 — "it defines approximately twenty-five relationship types including \texttt{DESCRIBES}, \texttt{CONTAINS}, \texttt{DEPENDS\_ON}, \texttt{GENERATED\_FROM}, and \texttt{OPTIONAL\_DEPENDENCY\_OF} --- but the dependency relationship is represented implicitly as one type among many, and SPDX 2.3 has no native structure for embedding vulnerability data. SPDX is widely adopted for licence auditing workflows but does not natively support the graph structure required for the analysis SBOM Lens performs." → "SPDX 2.3 defines 45 relationship types including \texttt{DESCRIBES}, \texttt{CONTAINS}, \texttt{DEPENDS\_ON}, \texttt{GENERATED\_FROM}, and \texttt{OPTIONAL\_DEPENDENCY\_OF} --- and dependencies are stated with several of them, in either direction (\texttt{DEPENDS\_ON}, \texttt{DEPENDENCY\_OF}, \texttt{DEV\_DEPENDENCY\_OF} and others), so they must be normalised before a dependency graph can be built. SPDX 2.3 has no element for vulnerability data; SPDX 3.0 (2024) adds one in its Security profile."
  - Ch2:32 — "Its key structural advantage over SPDX is the explicit separation of components and their dependency relationships into two distinct top-level arrays. A CycloneDX document's" → "A CycloneDX document's"
  - Ch2:36 — "CycloneDX provides the most explicit and semantically unambiguous graph representation available in any SBOM format, eliminating the inference step that SPDX dependency reconstruction would require." → "CycloneDX states every dependency in a single directed form, and it is the format produced by the generator used in the evaluation; SPDX input, which requires normalising its relationship types, is left as future work (\autoref{sec:concl-future})."
  - Ch2:22 — "is motivated directly by the richness and explicitness of its dependency graph representation" → "is motivated by its single, explicit dependency representation"
- Also touches: none required. Ch1:65 ("richest representation … within the OWASP ecosystem … without the need for format translation") is consistent with the new wording.
- Viva answer: "SPDX can express the graph, but it uses several relationship types in both directions. CycloneDX has one `dependsOn` form, and it is what the generator emits. An SPDX importer is a normalisation step, and I list it as future work."

### E5 (from C9, C2(c)) Community "containment" contradicts directed propagation, and the 0.3 modularity threshold is unsupported and unused — impact: medium
- Settled facts:
  - A vulnerability reaches exactly anc(v), whatever the community boundaries (Ch2:126, `impact.py:50`).
  - Ch6:85 already rewrites containment in directed terms and says that Q is not reported. Louvain optimises Q, so the formula stays.
  - The 0.3 attribution to Barabási 2016 is unverified, and sparse, tree-like graphs score a high Q by construction (Guimerà et al. 2004).
- Fix:
  - Ch2:194 — "with the same degree sequence; values above approximately \(0.3\) are generally considered to indicate meaningful community structure \citep{Barabasi2016}." → "with the same degree sequence. SBOM Lens reports the partition, not its modularity score."
  - Ch2:198 — "A vulnerability that enters one high-modularity community tends to spread within it, contained by the sparse inter-community edges. Communities with many inter-community connections are higher-risk propagation bridges." → "Because a vulnerability reaches only the consumers of the vulnerable component, what a community says about risk depends on how many components outside it depend on its members (\autoref{sec:ns-communities})."
- Also touches: none.
- Viva answer: "Communities describe coupled subsystems. Exposure follows the consumers of a component, whatever the partition. The tool does not output Q, so I give no threshold."

### E6 (from C10, C13, C19(c)) The attack taxonomy is mis-cited, its example is mislabelled, and the k-core triage claim has nothing in Ch6 to land on — impact: medium
- Settled facts:
  - Ladisa et al. have three top-level nodes (AV-100, AV-200, AV-001). The four headings come from different depths of the tree.
  - Typosquatting (AV-204) uses the example "dajngo vs. django". An added suffix is combosquatting in their terms, and no malicious `requests-oauthlib2` is documented.
  - Ladisa et al. list account take-over (AV-602) alongside social engineering.
  - The core number is undirected (`graph_analysis.py:51`). Ch6:53–58 never mentions name-confusion attacks. Ch6:158 ties the taxonomy to namespace concentration, and Ch3:94 already names blast radius and paths as the post-disclosure signals.
- Fix (recommended D4-A, minimal):
  - Ch2:72 — "Four primary categories are relevant to the SBOM analysis context." → "Four attack vectors from this taxonomy are relevant to the SBOM analysis context."
  - Ch2:76 — "--- typically through social engineering over months or years ---" → "--- by taking over a maintainer's account or, as in the following example, through prolonged social engineering ---"
  - Ch2:80 — "for example, \texttt{requests-oauthlib2} alongside \texttt{requests-oauthlib}" → "for example, \texttt{dajngo} for \texttt{django} \citep{Ladisa2023}"
  - Ch2:82 — "Graph-based SBOM analysis complements detection for the latter two categories. K-core decomposition (\autoref{cp:network-sci}) reveals how deeply embedded a potentially substituted package is in the dependency network, providing a structural triage signal for incident response." → "Graph-based SBOM analysis complements detection for these attacks. Once a substituted package is identified, its blast radius and dependency paths (\autoref{cp:vuln-analysis}) show which parts of the application include it, and namespace concentration (\autoref{cp:network-sci}) shows how much of the graph depends on a single publisher."
  - **DECISION NEEDED** (D4).
- Also touches: none. The :208 entrenchment claim is V1.
- Viva answer: "After disclosure, triage needs to know who includes the package and how: the blast radius and the paths. The core number measures entrenchment, not exposure."

### E7 (from C5) CycloneDX spec facts: VEX came in 1.4; only `type` and `name` are required; `bom-ref` is unique, not stable; the vertex rules are incomplete — impact: medium
- Settled facts:
  - `vulnerabilities` is absent from the 1.3 schema and present in 1.4 (release note, 12 Jan 2022).
  - The 1.7 schema requires `["type","name"]`, and `bom-ref` "must be unique within the BOM".
  - `sbom_store.py:21-34 @28113e7` reads only top-level `components`, drops self-loops, and drops edges whose endpoints are not nodes.
- Fix:
  - Ch2:32 — "CycloneDX 1.6 added native support for VEX" → "CycloneDX 1.4 (2022) added native support for VEX"
  - Ch2:42 — "each carrying at minimum a \texttt{bom-ref} (a stable local document identifier), a \texttt{name}, a \texttt{version}, a \texttt{purl} (Package URL), and a \texttt{type}." → "each with a required \texttt{type} and \texttt{name} and, usually, a \texttt{bom-ref} (an identifier unique within the document), a \texttt{version} and a \texttt{purl} (Package URL)."
  - Ch2:44 — "It is a stable, local reference that uniquely identifies" → "It is a local reference that uniquely identifies"
  - Ch2:46 — after "plus the root component from \texttt{metadata.component}" add " (nested components are not read)". After "SBOM Lens adds directed edges \(u \to v_1, u \to v_2, \ldots\)" add ", omitting self-loops and references to components not in the document".
- Also touches: none. These rules match Ch4:102 and Ch6:56.

### E8 (from C6) "NVD provides the authoritative CVSS v3.1 base scores for most published CVEs" no longer holds — impact: medium
- Settled facts:
  - NIST's notice of 15 Apr 2026 (quoted under Settled disputes) prioritises KEV, federal and critical software, and stops routinely providing separate scores where a CNA score exists.
  - CVSS v2 has not been generated for new CVEs since July 2022.
  - "Primary reference … in the security industry" has no source.
- Fix: Ch2:230 — "with CVSS base scores (versions 2.0, 3.x, and 4.0), Common Platform Enumeration (CPE) product matching strings, and categorical severity ratings \citep{NVD}. NVD provides the authoritative CVSS v3.1 base scores for most published CVEs and is the primary reference for severity data in the security industry." → "with CVSS base scores (version 2.0 until 2022, then 3.x and 4.0), Common Platform Enumeration (CPE) product matching strings, and categorical severity ratings \citep{NVD}. Since April 2026, NVD enriches only a priority subset of CVEs (those known to be exploited or affecting federal or critical software) and otherwise relies on the scores supplied by the CVE numbering authority \citep{NVD2026}."
- Also touches: none. The change supports Ch5:16–20.
- New bib entry:
  ```bibtex
  @Misc{NVD2026,
      author  = {{National Institute of Standards and Technology}},
      title   = {{NIST} Updates {NVD} Operations to Address Record {CVE} Growth},
      year    = {2026},
      month   = apr,
      note    = {News release, 15 April 2026},
      url     = {https://www.nist.gov/news-events/news/2026/04/nist-updates-nvd-operations-address-record-cve-growth},
      urldate = {2026-10-12},
  }
  ```
- Viva answer: "NVD's enrichment backlog and its 2026 prioritisation are one more reason to use OSV, whose records carry the advisory's own vector."

### E9 (from C7) CVSS v3.1 is called "the standard", and v4.0 is never introduced, although Ch5:30 and Ch8:41 rely on it — impact: medium
- Settled facts: CVSS v4.0 was released in Nov 2023. Its base metrics add AT, split impact into VC/VI/VA and SC/SI/SA, and drop Scope. Ch2:230 mentions "4.0" only in passing. Ch5:30 and Ch8:41 treat v4-only records as a limitation.
- Fix: Ch2:254 — "The Common Vulnerability Scoring System (CVSS) v3.1 is the standard for quantifying the severity of software vulnerabilities \citep{CVSS31}." → "The Common Vulnerability Scoring System (CVSS) is the standard for quantifying the severity of software vulnerabilities; SBOM Lens implements version 3.1 \citep{CVSS31}. Its successor, CVSS v4.0 (2023), changes the base metrics: it adds Attack Requirements, separates the impact on the vulnerable system from the impact on subsequent systems, and drops Scope \citep{CVSS40}. A v3.1 score therefore cannot be computed from a v4.0 vector." Ch2:222 can stay.
- Also touches: none. Ch5:30 and Ch8:41 become grounded.
- New bib entry:
  ```bibtex
  @Misc{CVSS40,
      author  = {{Forum of Incident Response and Security Teams}},
      title   = {Common Vulnerability Scoring System Version 4.0: Specification Document},
      year    = {2023},
      url     = {https://www.first.org/cvss/v4.0/specification-document},
      urldate = {2026-10-12},
  }
  ```

### E10 (from C8) The adjacency matrix and spectral radius read as directed, and PastorSatorras2001 does not give 1/λ₁ — impact: medium
- Settled facts:
  - :182 already names the undirected projection. :106 and :214 do not, and Ch6:10 shows that the directed spectrum is zero for a DAG.
  - Wang et al. (2003) give 1/λ₁ for undirected graphs. Pastor-Satorras & Vespignani (2001) give ⟨k⟩/⟨k²⟩, not 1/λ₁.
  - The sparse solver is used only for more than 500 nodes (`graph_analysis.py:329-340`), which is 21 of the 90 graphs.
- Fix:
  - Ch2:106 — "so \(\mathbf{A}\) is stored in compressed sparse format in practice. The adjacency matrix is the basis for eigenvector centrality analysis described in \autoref{sec:centrality}." → "so \(\mathbf{A}\) is usually stored in compressed sparse format; SBOM Lens does so for graphs of more than 500 components. The adjacency matrix of the undirected projection is the basis for the eigenvector centrality described in \autoref{sec:centrality}."
  - Ch2:214 — "the spectral radius \(\lambda_1\) of the adjacency matrix governs how readily a contagion spreads over a network, with \(1/\lambda_1\) acting as its epidemic threshold \citep{Wang2003,PastorSatorras2001}" → "network structure governs how readily a contagion spreads \citep{PastorSatorras2001}, and on an undirected network the spectral radius \(\lambda_1\) of the adjacency matrix sets the epidemic threshold \(1/\lambda_1\) \citep{Wang2003}"
- Also touches: none. Ch6:124 already cites only Wang2003.

### E11 (from C11) "A transitive dependency in millions of Java applications" is uncited and departs from Ch3's measured figure — impact: medium
- Settled facts: the claim is uncited. Ch3:102 cites over 17,000 Maven Central packages, many of them depending on log4j only indirectly. The defender is right that this counts artifacts, not applications, so the two figures do not strictly conflict. But no source gives "millions".
- Fix: Ch2:66 — "Apache Log4j was a transitive dependency in millions of Java applications; many of the affected development teams were entirely unaware that their software included it." → "Apache Log4j was a dependency of over 17{,}000 packages in Maven Central alone, many of them only through indirect dependencies (\autoref{sec:case-log4shell}) \citep{GoogleLog4j2021}, and many organisations could not easily tell whether their own software included it."
- Also touches: none. The figure is consistent with Ch1:9 and Ch3:102.

### E12 (from C12) Preferential attachment in package ecosystems is asserted without a source, which goes beyond the agreed hedging — impact: medium
- Settled facts:
  - "γ between 2 and 3" is the textbook generalisation (Barabási 2016), not a finding of BA 1999, which reports 2.1–2.3 and 4.
  - Ecosystem growth and concentration are documented in the bib (Kikas2017: transitive dependencies growing and an increasing impact of the most popular package; Decan2019). Preferential attachment is not documented and is never tested.
  - This does not reopen the heavy-tail decision.
- Fix:
  - Ch2:150 — "where \(\gamma \in [2, 3]\) for most empirical cases \citep{Barabasi1999}." → "where \(\gamma\) typically lies between 2 and 3 \citep{Barabasi2016}, although whether a power law fits better than other heavy-tailed distributions must be tested \citep{Clauset2009}."
  - Ch2:152 — "This ``rich get richer'' dynamic explains hub emergence. Open-source package ecosystems exhibit both mechanisms: new packages are added continuously, and new packages tend to depend on already-popular libraries." → "This ``rich get richer'' dynamic is one explanation of how hubs emerge. Open-source package ecosystems grow continuously and concentrate their dependencies on a small number of popular libraries \citep{Kikas2017,Decan2019}, which is consistent with, but does not demonstrate, preferential attachment."
- Also touches: none. The wording matches Ch6:172 and Ch7:90.

### E13 (from C14) Ch5 says Ch2 "fully defines" the CVSS metric values, and Ch2 does not give them — impact: medium (broken cross-reference)
- Settled facts: the values are only in the code and, for PR, in Ch5:110. The spec's §7.4 values are quoted under Settled disputes and match the code.
- Fix: Ch2:266 — "where each metric takes its numerical value from the lookup table." → "where each metric takes its numerical value from the lookup table: AV is 0.85 (Network), 0.62 (Adjacent), 0.55 (Local) or 0.20 (Physical); AC is 0.77 (Low) or 0.44 (High); PR is 0.85 (None), 0.62 (Low; 0.68 if Scope is Changed) or 0.27 (High; 0.50 if Scope is Changed); UI is 0.85 (None) or 0.62 (Required); and C, I and A are each 0.56 (High), 0.22 (Low) or 0 (None) \citep{CVSS31}."
- Also touches: none. Ch5:91 and :123 become true as written.

### E14 (from C18) Attribution and algorithm details: σ, Tarjan, the articulation-point root case, Impact ≤ 0, the severity bands — impact: low-medium
- Settled facts:
  - σ is from Humphries & Gurney (in the bib, cited at Ch6:178), who use σ > 1.
  - CLRS gives the two-pass SCC algorithm, while NetworkX uses Tarjan's.
  - The DFS root is an articulation point iff it has ≥ 2 tree children.
  - The spec says "Impact ≤ 0" (`CVSS31`, already cited at :258, so no new source).
  - Ch5:104–108 merges None into Low and adds UNKNOWN.
  - Brandes cited to Barabasi2016 depends on D3.
- Fix:
  - Ch2:160 — "\citep{WattsStrogatz1998}. The small-world coefficient \(\sigma" → "\citep{WattsStrogatz1998}. The small-world coefficient \citep{HumphriesGurney2008} \(\sigma", and "\(\sigma \gg 1\) indicates" → "\(\sigma > 1\) indicates"
  - Ch2:116 — "SCCs can be computed in \(O(|V| + |E|)\) time with Tarjan's algorithm \citep{CLRS4th}." → "SCCs can be computed in \(O(|V| + |E|)\) time \citep{CLRS4th}; NetworkX uses a variant of Tarjan's algorithm."
  - Ch2:136 — "A vertex \(v\) is an articulation point if any DFS-tree child" → "A non-root vertex \(v\) is an articulation point if any DFS-tree child". After "to reach \(v\)'s ancestors" add "; the DFS root is one if it has at least two DFS-tree children".
  - Ch2:274 — "If \(\mathrm{Impact} = 0\)" → "If \(\mathrm{Impact} \le 0\)"
  - Ch2:276 — "SBOM Lens uses these severity bands to colour-code" → "SBOM Lens uses these bands, with None merged into Low (\autoref{sec:severity-colour}), to colour-code"
- Also touches: none.
- Viva answer: "Each is a precision slip in the text. The implementation is right in every case: `ISS === 0` is equivalent to Impact ≤ 0."

### E15 (from C19(a,b,d,e)) Sourcing and direction slips in the supply-chain and graph sections — impact: low-medium
- Settled facts:
  - Median direct share is 11.5 %, a ratio of about 7.7 : 1, and 58 of 90 graphs are below 10×.
  - AA20-352A gives no organisation count. The 18,000 is SolarWinds' 8-K ("fewer than 18,000" customers may have installed).
  - xz is uncited at :76, although `NVD_CVE_2024_3094` is in the bib.
  - The NVD CVE API has no purl search.
  - With consumer→dependency edges, a vulnerability at v reaches only anc(v).
- Fix:
  - Ch2:64 — "routinely outnumber direct dependencies by an order of magnitude in large projects." → "usually far outnumber direct dependencies: in the median application evaluated in \autoref{cp:evaluation}, 89\% of the components enter transitively."
  - Ch2:74 — "(December 2020, CISA Alert AA20-352A) is the canonical example: malicious code was inserted into a signed software update and distributed to approximately 18,000 organisations, including US government agencies, via a trusted vendor relationship." → "(December 2020) is the canonical example: malicious code was inserted into a signed software update \citep{CISA_AA20_352A}, which up to 18{,}000 customers, including US government agencies, installed through a trusted vendor relationship \citep{SolarWinds8K2020}."
  - Ch2:76 — after "(CVE-2024-3094, CVSS 10.0, March 2024)" add " \citep{NVD_CVE_2024_3094}"
  - Ch2:48 — "OSV uses purl natively; NVD uses CPE but can be cross-referenced via purl." → "OSV uses purl natively; NVD uses CPE."
  - Ch2:134 — "a vulnerability, deprecation, or licence change affecting it touches both sides of the split." → "a vulnerability, deprecation, or licence change affecting it reaches every part of the split that depends on it."
- Also touches: none. Ch1:9 ("thousands", citing CISA) remains correct.
- New bib entry:
  ```bibtex
  @Misc{SolarWinds8K2020,
      author  = {{SolarWinds Corporation}},
      title   = {Form 8-K: Current Report},
      year    = {2020},
      month   = dec,
      note    = {Filed with the U.S. Securities and Exchange Commission, 14 December 2020, Item 8.01},
      url     = {https://www.sec.gov/Archives/edgar/data/1739942/000162828020017451/swi-20201214.htm},
      urldate = {2026-10-12},
  }
  ```
  The alternative that needs no new entry is "distributed to thousands of organisations … \citep{CISA_AA20_352A}", matching Ch1:9.

### E16 (from C15) OSV: "over fifty sources", "npm advisories", ranges "rather than" lists, `/v1/query` — impact: low
- Settled facts: about 21 sources (18 native plus 3 converted). npm data arrives through GHSA. The schema has both `ranges` and `versions`. SBOM Lens uses `/v1/querybatch` (`vulnerabilityAPI.js:95`, Ch5:26). "Most precise source" has no source.
- Fix:
  - Ch2:238 — "defines version range bounds using \texttt{introduced} and \texttt{fixed} fields rather than fixed version lists, enabling precise matching against any version in a given range. OSV aggregates vulnerability data from over fifty upstream sources including RustSec, PyPI advisories, npm advisories, the GitHub Advisory Database, the Debian Security Tracker, and the Alpine Security Database." → "defines affected versions both as ranges, with \texttt{introduced} and \texttt{fixed} bounds, and as explicit version lists. OSV aggregates vulnerability data from about twenty upstream sources, including the GitHub Advisory Database (which covers npm), the PyPI and Go databases and RustSec, and converts Debian, Alpine and NVD records for open-source software \citep{OSV}."
  - Ch2:240 — "OSV's query API (\texttt{/v1/query}) accepts a package purl directly, making it the most precise source for open-source component vulnerability matching and the vulnerability source used by SBOM Lens." → "OSV's query API accepts a package purl directly, which is why SBOM Lens uses it as its vulnerability source."
- Also touches: none (see V2).

### E17 (from C16) SLSA is described by its retired v0.1 level model, and the CISA guidance is uncited and now final — impact: low
- Settled facts:
  - SLSA v1.0 has a Build track (L0–L3). v1.2 (24 Nov 2025) adds a Source track.
  - CISA published the final *2026 Minimum Elements for a Software Bill of Materials (SBOM)* on 29 July 2026, with NSA, FBI and international partners. It adds component hash, licence, tool name and generation context, and keeps the dependency relationship.
- Fix:
  - Ch2:90 — "provides a complementary four-level maturity framework for source integrity, build integrity, and provenance attestation" → "provides complementary tracks of maturity levels for build integrity and provenance and, since version 1.2 (2025), for source integrity"
  - Ch2:90 — "CISA has further published guidance on SBOM minimum elements, sharing mechanisms, and tooling recommendations." → "In 2026, CISA and partner agencies updated the NTIA minimum elements, adding among others component hashes, licences and the name of the generating tool, and keeping the dependency relationship \citep{CISA2026SBOM}."
  - Bib `SLSA`: add `note = {Version 1.2}`.
- Also touches: none. Ch2:18 (the NTIA 2021 seven fields) stays as a historical statement.
- New bib entry (replaces the defender's `CISA2025SBOM`):
  ```bibtex
  @Report{CISA2026SBOM,
      author      = {{Cybersecurity and Infrastructure Security Agency}},
      title       = {2026 Minimum Elements for a Software Bill of Materials ({SBOM})},
      institution = {CISA},
      year        = {2026},
      month       = jul,
      note        = {Co-authored with NSA, FBI and international partners; published 29 July 2026},
      url         = {https://www.cisa.gov/resources-tools/resources/2026-minimum-elements-software-bill-materials-sbom},
      urldate     = {2026-10-12},
  }
  ```

### E18 (from C17) "HITS scores are computed iteratively until convergence" omits the iteration cap that Ch6 discloses — impact: low
- Settled facts: `nx.hits(G, max_iter=200)`. Ch6:32 discloses the zero fallback, as already decided.
- Fix: Ch2:180 — "HITS scores are computed iteratively until convergence and applied in \autoref{cp:network-sci}." → "HITS scores are computed iteratively, up to an iteration limit (\autoref{sec:hits}), and applied in \autoref{cp:network-sci}."
- Also touches: none.

### E19 (from D-new1) The hub/leaf contrast contradicts the thesis's edge orientation — impact: low
- Settled facts: edges run consumer→dependency (:46, :102). A leaf has no dependencies (Ch6:138), and widely used utility libraries are often leaves.
- Fix: Ch2:154 — "far exceeding that of a vulnerability in a leaf node." → "far exceeding that of a vulnerability in a component that few others depend on."
- Also touches: none.

### E20 (from D-new2) The root definition is narrower than the code — impact: low
- Settled facts: roots are all vertices of in-degree zero (`impact.py:52`), joined under a virtual root (`impact.py:19-24`, Ch5:240). In every evaluated SBOM the two coincide.
- Fix: Ch2:108 — "In SBOM Lens, the root node is the project being analysed, identified by \texttt{metadata.component} in the CycloneDX document." → "In a well-formed SBOM, the only root is the project being analysed, identified by \texttt{metadata.component} in the CycloneDX document; when there are several roots, SBOM Lens treats each in-degree-zero vertex as one (\autoref{sec:dominator-tree})."
- Also touches: none.

## Viva prep

### V1 (from C10, :208) Is a high-coreness component really "hard to replace"?
- Likely question: "You say that replacing a high-coreness component requires satisfying the whole inner core. Where is that shown?"
- Answer: it is an interpretation of dense undirected interconnection, shared with Ch6:58, which states that the core number "says nothing about direction". It is not a measured claim. Optional softening: Ch2:208 "replacing it requires finding an alternative that satisfies all components in the inner core simultaneously" → "replacing it affects many components that are themselves densely linked".

### V2 (from C15 side note) Do NVD-only vulnerabilities reach SBOM Lens through OSV's NVD conversion?
- Likely question: "OSV converts NVD records. So isn't Ch5:20 wrong to say that NVD-only vulnerabilities are not reported?"
- Answer: OSV's converted NVD records describe affected git repositories and commit ranges, not registry packages, so a `pkg:npm/…` purl query does not match them. For SBOM Lens's purl queries, Ch5:20 and Ch8:41 hold. No change (see Holds). This rests on OSV's documented conversion pipeline. I did not test a specific record.

### V3 (from C2) Why does the background define concepts the tool does not compute?
- Likely question: "Why define betweenness and modularity at all?" (This depends on D3.)
- Answer: modularity is the objective Louvain optimises, so its definition is needed even though Q is not reported (E5). Betweenness is the standard path-based centrality, and a reader compares it with HITS and eigenvector centrality. If it is kept (D3-B), the text says that SBOM Lens does not compute it.

## Holds
- No whole item. Rebutted or moot sub-points, with no action:
  - C1: the critic's date for M-26-05 ("February 2026") is wrong. It is 23 January 2026 (memo PDF).
  - C13: the critic's account of the Ladisa tree is wrong (see Settled disputes). The conclusion stands and is handled in E6.
  - C11: "conflicts by two orders of magnitude" is rebutted, because artifacts and applications are different populations. The uncited figure is still fixed (E11).
  - C12: "γ ∈ [2,3] is false" is rebutted. It is the textbook statement, and only the citation placement changes (E12).
  - C18(c): naming Tarjan is correct for NetworkX. Only the CLRS citation changes (E14).
  - C9: the attribution of 0.3 to Barabási (2016) is unverified and moot, because the sentence is deleted.
  - C7: the critic's "≈4,500 v4.0 scores corrected on 28 Apr 2026" is unverified and unused.
  - C6: the "Deferred" step of 2 Apr 2025 is unverified and unused. The 15 Apr 2026 notice suffices.
  - C15 side note: the defender's optional in-step edit to Ch5:20 and Ch8:41 is not made (see V2).

## Decisions needed
1. **Regulatory history (E1/C1).**
   - (A) Drop EO 14144 everywhere (Ch1:11, Ch2:20, :88). State EO 14028 guidance, M-26-05 and the CRA dates. One new bib entry (`OMB_M2605`), and `EO14144` stops being cited. **Recommended**: it is shortest, accurate today, and avoids a sentence that only records a requirement being added and then removed.
   - (B) Keep the history at Ch2:88: "EO 14144 (January 2025) added attestation requirements for federal software suppliers \citep{EO14144}, which EO 14306 (June 2025) removed \citep{EO14306}." Add `EO14306` (90 Fed. Reg. 24723). This is complete but adds a sentence with no bearing on SBOM Lens.
   - (C) Reduce the regulation to Ch2:88 only, and replace the :20 paragraph with one sentence pointing to \autoref{sec:policy-response}. This removes the :20/:88 repetition the critic noted, but it is a larger edit.
2. **Where dominance is defined (E2/C2).**
   - (A) One paragraph at the end of the ancestor/descendant paragraph in \autoref{sec:connectivity}, with LengauerTarjan1979 and Cooper2006 (text in E2). **Recommended**: minimal, and it sits next to the blast radius it is contrasted with.
   - (B) A new subsection "Dominators" in \autoref{sec:graph-theory}, with a small worked example. This gives the novelty concept more weight but adds length.
   - (C) Add the two citations at Ch5:240 only, and fix Ch2:4. This leaves Ch2 without the concept the novelty rests on.
3. **Betweenness centrality (E2/C2, C18(b)).**
   - (A) Delete Ch2:174–178. **Recommended**: no chapter and no code uses it, and E2's new :4 sentence then holds exactly.
   - (B) Keep it, add "SBOM Lens does not compute betweenness." at its end, and cite Brandes for the algorithm in place of Barabasi2016:
     ```bibtex
     @Article{Brandes2001,
         author  = {Brandes, Ulrik},
         title   = {A Faster Algorithm for Betweenness Centrality},
         journal = {The Journal of Mathematical Sociology},
         year    = {2001},
         volume  = {25},
         number  = {2},
         pages   = {163--177},
         doi     = {10.1080/0022250X.2001.9990249},
     }
     ```
   - The modularity formula stays under either option, because Louvain optimises it. Only the 0.3 threshold goes (E5).
4. **Attack taxonomy (E6/C13).**
   - (A) Minimal: "four attack vectors from this taxonomy", Ladisa's own typosquatting example, the account-take-over wording, and the k-core sentence replaced (text in E6). **Recommended.**
   - (B) Restructure :72–82 around Ladisa's three top-level branches (distinct malicious package, name confusion, subversion of a legitimate package), with the four examples placed under them. This is more faithful to the source, but it rewrites the section without changing anything downstream.
