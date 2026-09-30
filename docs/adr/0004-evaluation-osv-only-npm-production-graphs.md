# Evaluation uses OSV only and npm production dependency graphs

Supersedes the ecosystem and vulnerability-source parts of ADR 0003.

The tool's only working vulnerability path queries OSV (`useVulnerabilityFetcher` → `fetchVulnerabilitiesForSBOM`). The NVD and GHSA functions in `vulnerabilityAPI.js` were never called, and NVD's keyword search returns records with no purl, so they could not be mapped to graph nodes anyway. The thesis therefore claims **OSV only**, which itself aggregates GitHub Security Advisories, the PyPA advisory database and others. Chapters 2 and 5 are corrected, and the unused NVD/GHSA code is removed. Severity is taken from the CVSS v3 vector when one exists, otherwise from the advisory's GHSA severity label, otherwise it is recorded as UNKNOWN. UNKNOWN is never silently mapped to LOW in the data, and it is excluded from the CVSS rankings in RQ2.

The evaluation is scoped to **npm**. About 90 JavaScript/TypeScript repositories are chosen from GitHub search by stars. Each must have a committed lockfile at the latest release tag on or before 2024-03-31, must not be a workspaces monorepo, and must yield a **production dependency graph** with at least one known vulnerability and at most 1,000 components. Every rejected candidate is logged with its reason, giving Chapter 7 a selection funnel. A lockfile pins the full transitive tree, so the graph is the project as shipped without any resolver. PyPI and Maven would have needed resolution against today's registries. The cost is external validity: results describe npm graphs, which nest duplicate versions. This is stated in Threats to Validity, and replication for PyPI and Maven is future work.

## Consequences

The production dependency graph is derived by reachability, not with cdxgen's `--required-only`. cdxgen generates the full SBOM from the lockfile, and the pipeline keeps what is reachable from the root through its `dependencies` and `optionalDependencies`. With v1 `package-lock.json` files, cdxgen labels transitive production dependencies "optional", so `--required-only` kept only the direct ones (jellyfin-web v10.8.13: 47 nodes instead of 83). On that project the reachability graph matched the lockfile's own production set exactly, except for one dependency installed from a tarball URL, which has no purl. For npm lockfiles every result records this agreement. Projects whose SBOM gives the root no dependency edges (some pnpm lockfile formats) are skipped and logged.

## Considered Options

- A real three-source integration (GHSA by version range, NVD by CPE): rejected as several days of work for little extra coverage over OSV.
- 30 projects each for npm, PyPI and Maven: rejected because of lockfile and resolver problems (and Maven was not installed).
- Including dev dependencies: rejected, because they are not shipped and push typical apps far past the 1k cap.
