# Thesis to-do (branch `ulo-template`)

## 1. Screenshots

All shots use the `code-server` SBOM and the latest UI, with the light theme and the same window size. Save each PNG at the path shown; the next build replaces its "SCREENSHOT PENDING" box. The earlier versions of the four retakes are in `Figures/retake-old/`, which git ignores.

**Done** (in the thesis now): `Chapter4/app-overview.png` (also used as the radial layout), `Chapter4/layout-tree.png` and `layout-force.png` (cropped by Claude to `*-canvas.png`), `Chapter5/blast-radius.png` (`basic-ftp`), `Chapter5/dominated-set.png` (`express`), `Chapter6/communities.png`.

**To retake:**

| ☐ | Save as | Why | What to capture |
|---|---|---|---|
| ☐ | `Figures/Chapter4/node-details.png` | New Component section in the Details panel (sbom-lens `03f5fb8`) | **No overlay active**, tap `basic-ftp`. The Component section (name, version, PURL, type, bom-ref, severity) is open at the top. With the severity overlay on, tapping a vulnerable node opens the Vulnerabilities page instead |
| ☐ | `Figures/Chapter5/vuln-overlay.png` | The old shot had the Java sample imported on top of code-server (194 nodes / 276 edges) | Reload `code-server` (status bar 181 nodes, 263 edges), only *Vulnerable components* active |
| ☐ | `Figures/Chapter5/vulnerabilities-page.png` | Component column was empty (fixed in `ec33079`) | Vulnerabilities page, `code-server`, with the Component column filled in |
| ☐ | `Figures/Chapter6/risk-assessment.png` | Namespace concentration was empty (fixed in `018cd20`); the page was also scrolled, hiding its title | Risk Assessment page scrolled to the top, with namespace concentration filled in |

Restart the backend before retaking `risk-assessment.png`, so that it runs the namespace fix.

Optional: the full-window overlay shots print fairly small at 0.8 of the text width, because the side navigation takes a third of each image. If you want them larger, crop out the side navigation, or ask Claude to crop them.

## 2. After the retakes

Tell Claude if any retaken shot differs from its caption or from the text that cites it, and Claude will adjust the text.

## 3. Other open items

- [ ] **Push `master`** (`git push origin master`). It holds the Ch5 review commit, so it has to be pushed before you open the `ulo-template` PR. Otherwise that commit will appear inside the PR.
- [ ] **Review and merge `ulo-template`** on GitHub: the ULO template switch plus these screenshot slots.
- [ ] **AI declaration.** Read the suggestions in the comment block in `Matter/04-AI.tex`. They cover ULO's policy reference, the uses of AI the current sentence leaves out, a draft replacement, and how to add ULO's AI appendix. Decide whether to adopt them.
- [ ] **Bibliography.** Optionally add a year or access date to the 12 `@Misc` entries that biber warns about: DependencyTrack, Grype, Syft, OWASPCycloneDX, PurlSpec, SLSA, MITRE_CVE, NVD, OSV, GHSA, EPSS, Sunshine.
- [ ] **Acronym list.** Optionally add one with real entries (SBOM, CVE, CVSS, PURL, OSV…). It was left out of the ULO switch.
- [ ] **Chapter review.** Continue the adversarial chapter review after Ch5.
