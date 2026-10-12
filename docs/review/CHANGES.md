# Thesis revision: summary of changes

Every chapter, plus the English and Portuguese abstracts, went through an adversarial review. For each one, a critic raised objections backed by evidence, a defender answered them, and a sorter turned the result into fixes and viva-prep notes. The author decided every open question. In total, **125 fixes** were applied. **No research result or conclusion changed.** The fixes correct how results are stated, align the text with the evaluated code (commit `28113e7`), and bring the background up to date (October 2026). The full review files are in `docs/review/chNN/`.

## Template and figures
- The thesis moved to the University of Leiria and Oeste (ULO) template, which replaces the IPLeiria one.
- Application screenshots were added: the interface, the layouts, the Details panel with the severity overlay, the Vulnerabilities page, blast radius, the dominated set, communities and Risk Assessment. All use the `code-server` SBOM.
- Small interface fixes in SBOM Lens came out of the screenshot work:
  - a Component section in the Details panel;
  - a component name for each vulnerability record;
  - double-tap to open a component's vulnerability records;
  - fixed-height Risk Assessment cards;
  - namespace counting for scoped npm packages.

## Bibliography
- Sources were corrected: the real 2025 SBOM landscape study (Garcia et al.), one edition of Cormen, and access dates for web sources.
- New references:
  - dominance algorithms (Lengauer–Tarjan 1979; Cooper, Harvey and Kennedy 2006);
  - recent SBOM-graph studies (Zięba-Kozarzewski 2026; Baird 2026);
  - Kendall's τ (1938, 1945);
  - OMB M-26-05, the NVD 2026 change, CVSS 4.0, CISA's 2026 SBOM minimum elements, the SolarWinds 8-K and CISA's 2025 npm alert.

## Chapter by chapter
- **Abstract (EN/PT).**
  - No longer claims that structural measures are better for prioritising.
  - Structure and severity are now described as "essentially unrelated".
  - The Portuguese version now says the same as the English.
  - Both list the blast radius, dependency paths and the Ch6 measures by their correct names.
- **Ch1 Introduction.**
  - Dominance, chokepoints and dependency paths are now named as contributions.
  - Research questions RQ1–RQ3 are stated.
  - The scope says the sample is 90 npm applications, each with a critical vulnerability.
  - Novelty claims are hedged ("to our knowledge"), and the four gaps match Ch3.
  - Regulation is up to date.
  - The 2025 npm worm (Shai-Hulud) is added.
- **Ch2 Background.**
  - Regulation corrected:
    - EO 14028 issued guidance rather than mandating SBOMs;
    - US agencies now decide for themselves whether to require one;
    - the EU CRA requires an SBOM from December 2027.
  - Dominance is now defined and cited.
  - The unused betweenness definition was removed.
  - Facts corrected: dependency cycles (npm allows them), SPDX, CycloneDX versions, NVD, CVSS 4.0, OSV, SLSA, the epidemic threshold and the attack taxonomy.
- **Ch3 Related Work.**
  - Novelty is now framed narrowly: some tools already list dependents or show paths. What is new is dominance, structural measures, and reach and paths on one graph.
  - Tool descriptions corrected for Dependency-Track, grype and GUAC.
  - The log4j figures are now on the basis the source itself uses.
  - Ecosystem studies are described as finding heavy tails rather than confirmed power laws.
- **Ch4 System Design.**
  - The text now describes the system as it was built:
    - how the layouts are actually rooted;
    - the root is not highlighted;
    - the storage limit is 50;
    - a second, browser-side parser exists;
    - a status-bar depth defect is disclosed.
  - Caching, latency and testing claims corrected.
- **Ch5 Vulnerability Analysis.**
  - One definition of blast radius, which includes the vulnerable component.
  - Corrected descriptions of the remediation lifecycle, the timeline, when data is fetched, and deduplication.
  - Overclaims removed.
- **Ch6 Network Science.**
  - HITS described as it ran (NetworkX 3.7), with its convergence failures disclosed.
  - The rich-club row dropped.
  - Articulation points compared with chokepoints.
  - The unmaintained-package check defined.
- **Ch7 Evaluation.**
  - Power-law fits that hit the fitting bound are now counted as no fit: α for 79 of 90 graphs (2.09–2.99).
  - Top-k overlap handles ties and is read against chance.
  - The data's hindsight is disclosed: 93% of findings were published after the release analysed.
  - Generator losses are disclosed.
  - The reproducibility claim is scoped to tables and figures, with library versions recorded.
- **Ch8 Conclusion.**
  - Each research objective is stated as met, with one limit: the evaluation does not show that structural prioritisation gives better remediation.
  - Numbers aligned with Ch7.
  - New limitations: numerical edge cases, generator effects, OSV queried only at upload, and hindsight.

## Still open
- Zenodo DOIs for the two repositories, cited in Ch7 and Ch8.
- A final layout pass (84 overfull lines).
- A decision on the AI-use declaration.
