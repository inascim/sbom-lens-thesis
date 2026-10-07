# Thesis to-do (branch `ulo-template`)

## 1. Screenshots

All shots use the `code-server` SBOM and the latest UI, with the light theme and the same window size. Save each PNG at the path shown; the next build replaces its "SCREENSHOT PENDING" box. The earlier versions of the four retakes are in `Figures/retake-old/`, which git ignores.

**Done** (in the thesis now): `Chapter4/app-overview.png` (also used as the radial layout), `Chapter4/layout-tree.png` and `layout-force.png` (cropped by Claude to `*-canvas.png`), `Chapter5/blast-radius.png` (`basic-ftp`), `Chapter5/dominated-set.png` (`express`), `Chapter6/communities.png`.

**Also done:** `Chapter4/node-details.png`, which is also used as the severity-overlay figure in §5.5.1, so no separate `vuln-overlay.png` is needed. And `Chapter5/vulnerabilities-page.png`.

**To retake:**

| ☐ | Save as | Why | What to capture |
|---|---|---|---|
| ☐ | `Figures/Chapter6/risk-assessment.png` | Namespace concentration was empty (fixed in sbom-lens `018cd20`); the cards now have fixed heights with lists scrolling inside them (`45ba23f`) | Restart the backend and reload the frontend. Take the Risk Assessment page scrolled to the top, with namespace concentration filled in |

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
