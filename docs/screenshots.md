# Thesis to-do (branch `ulo-template`)

## 1. Take the screenshots

Save each one as a PNG at the path shown. The next build replaces the "SCREENSHOT PENDING" box with the image, so no LaTeX editing is needed.

For every shot:
- Use the latest SBOM Lens UI.
- Use the light theme.
- Keep the same browser window size throughout. The full-window shots should match each other.
- Take high-resolution shots: browser zoom 100% on a large window, or a 2× device pixel ratio. They are printed at text width.
- Where you can, keep post-evaluation features out of the frame: the Fix Priorities page and the fixed-version fields.

SBOMs:
- **code-server** is the 181-component npm SBOM from the evaluation dataset.
- **Java sample** is `my-spring-app`, in `sbom-lens/public/test-sboms/java-vulnerable.json`. It is the SBOM from Table 3.2.

| # | Save as | Figure / section | SBOM | What to capture | Expected result |
|---|---|---|---|---|---|
| ☐ 1 | `Figures/Chapter4/app-overview.png` | `fig:app-overview`, §4.1.2 Frontend Layer | code-server | Full window: side navigation plus SBOM Graph page, default radial layout, no overlay | |
| ☐ 2a | `Figures/Chapter4/layout-tree.png` | `fig:layouts` (a), §4.4.2 | code-server | Graph canvas only, Tree layout | Printed at one third of the page width, so crop tight to the graph |
| ☐ 2b | `Figures/Chapter4/layout-radial.png` | `fig:layouts` (b) | code-server | Graph canvas only, Radial layout | Same crop and size as 2a |
| ☐ 2c | `Figures/Chapter4/layout-force.png` | `fig:layouts` (c) | code-server | Graph canvas only, Force-directed layout | Same crop and size as 2a |
| ☐ 3 | `Figures/Chapter4/node-details.png` | `fig:node-details`, §4.4.3 | Java sample | `log4j-core` selected, Details panel open | |
| ☐ 4 | `Figures/Chapter5/vulnerabilities-page.png` | `fig:vulnerabilities-page`, §5.4.2 | code-server | Vulnerabilities page: records table with status column, plus the severity chart | Ideally with records in more than one status |
| ☐ 5 | `Figures/Chapter5/vuln-overlay.png` | `fig:vuln-overlay`, §5.5.1 | Java sample | Only the severity overlay (*Vulnerable components*) active | Vulnerable nodes coloured by band, the rest dimmed. This replaces the old screenshot, which showed something else |
| ☐ 6 | `Figures/Chapter5/blast-radius.png` | `fig:blast-radius`, §5.5.2 | Java sample | *Analyze Impact* on `log4j-core`, only Blast Radius active | `my-spring-app` and `log4j-core` purple, the rest dimmed (matches Table 3.2) |
| ☐ 7 | `Figures/Chapter5/dominated-set.png` | `fig:dominated-set`, §5.5.3 | Java sample | *Analyze Impact* on `log4j-core`, Blast Radius and Dominator Tree both active | Purple as above, plus `log4j-api` in cyan |
| ☐ 8 | `Figures/Chapter6/communities.png` | `fig:communities`, §6.2.1 | code-server | Force-directed layout, only *Community (Colors)* active | |
| ☐ 9 | `Figures/Chapter6/risk-assessment.png` | `fig:risk-assessment`, §6.3 | code-server | Risk Assessment page with the audit results: cycles, version conflicts, namespace concentration | |

## 2. Tell Claude what differs

The captions and surrounding text describe what the thesis already claimed. After adding the PNGs, list every shot whose content differs from its caption or from the paragraph that cites it. Claude will then update the text to match the latest UI. Shots that are likely to need text changes:
- **#3, the Details panel.** §4.4.3 says the panel shows `name, version, purl, type, bom-ref`. The latest panel is an accordion with graph metrics, vulnerabilities and dependency paths.
- **#5, the severity overlay.** §5.5.1 says it is switched on from the Analysis menu. In the latest UI the entry is *Vulnerable components*. Check the colours too: the caption says green-yellow for LOW.
- **#7, the dominator overlay.** §5.5.3 says that choosing a new node refreshes only the blast radius. Check whether the latest UI still behaves that way.

## 3. Other open items

- [ ] **Push `master`** (`git push origin master`). It holds the Ch5 review commit, so it has to be pushed before you open the `ulo-template` PR. Otherwise that commit will appear inside the PR.
- [ ] **Review and merge `ulo-template`** on GitHub: the ULO template switch plus these screenshot slots.
- [ ] **AI declaration.** Read the suggestions in the comment block in `Matter/04-AI.tex`. They cover ULO's policy reference, the uses of AI the current sentence leaves out, a draft replacement, and how to add ULO's AI appendix. Decide whether to adopt them.
- [ ] **Bibliography.** Optionally add a year or access date to the 12 `@Misc` entries that biber warns about: DependencyTrack, Grype, Syft, OWASPCycloneDX, PurlSpec, SLSA, MITRE_CVE, NVD, OSV, GHSA, EPSS, Sunshine.
- [ ] **Acronym list.** Optionally add one with real entries (SBOM, CVE, CVSS, PURL, OSV…). It was left out of the ULO switch.
- [ ] **Chapter review.** Continue the adversarial chapter review after Ch5.
