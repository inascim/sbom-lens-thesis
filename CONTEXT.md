# SBOM Lens Thesis

The master's thesis describing SBOM Lens, a tool that turns a CycloneDX SBOM into a dependency graph and analyses its structure and known vulnerabilities. This glossary fixes the terms the thesis chapters use.

## Graph model

**Dependency graph**:
The directed graph built from one SBOM: components are nodes, and an edge runs from a consumer to the component it depends on.
_Avoid_: dependency tree (it is not a tree), SBOM graph (when meaning the model rather than the rendering)

**Production dependency graph**:
The dependency graph restricted to what ships: runtime dependencies only, excluding development and test dependencies.
_Avoid_: full graph, lockfile graph

**Root**:
The component the SBOM describes (the application itself); a node with no consumers. Where an SBOM has several, analyses treat them as children of one virtual root.

## Vulnerability analysis

**Blast radius**:
A given component together with every component that depends on it, directly or transitively — everything that inherits its vulnerability.
_Avoid_: impact set, affected set

**Dominated set**:
The components whose every path from the root passes through a given component; they would be cut off from the application if it were removed.
_Avoid_: dominator tree (that is the whole structure, not one node's set)

**Vulnerability reach**:
The share of the dependency graph that lies inside the blast radius of at least one vulnerable component — a static measure, not a simulated one.
_Avoid_: spread, propagation (when meaning this static measure)

**Transitive-only vulnerable component**:
A vulnerable component that the application does not declare directly: its shortest dependency path from the root has length two or more.
_Avoid_: indirect vulnerability, hidden dependency

**Vulnerable chokepoint**:
A vulnerable component that is also a chokepoint: part of the application can be reached only through it.

**Chokepoint**:
A component with a non-empty dominated set — a single point that the application cannot route around.
_Avoid_: bottleneck

**Severity band**:
One of CRITICAL, HIGH, MEDIUM, LOW, or UNKNOWN for a vulnerability; a component takes the highest band among its vulnerabilities. UNKNOWN means no CVSS v3 score or advisory severity was available. It is not LOW.
_Avoid_: severity level, risk level

## Structural risk

**Structural risk**:
Risk that follows from a component's position in the dependency graph, independent of whether it currently has a known CVE.
_Avoid_: topological risk, graph risk

**Flat-inventory analysis**:
Analysis that treats an SBOM as a list of components (name, version, CVEs) and ignores the edges between them. The baseline SBOM Lens is compared against.
_Avoid_: list-based analysis, traditional SCA

**Version conflict**:
The same package appearing in one dependency graph at two or more distinct versions. The textbook "diamond dependency conflict" is one way it arises.
_Avoid_: diamond, diamond dependency

## Spectral analysis

**Spectral radius**:
The largest eigenvalue λ₁ of the graph's (undirected) adjacency matrix; a single whole-graph number for how readily a compromise could spread. Its reciprocal, 1/λ₁, is the **epidemic threshold**.

**Spectral exposure score**:
A component's entry in the dominant eigenvector — its eigenvector centrality — read as its share of the graph's spread potential.
_Avoid_: R0, reproduction number (these imply an epidemic simulation that SBOM Lens does not run)

**Propagation simulation**:
Running a dynamic contagion model (e.g. SIR) over the dependency graph through time. Out of scope for this thesis; future work only.
_Avoid_: epidemic model (when meaning the static spectral analysis)
