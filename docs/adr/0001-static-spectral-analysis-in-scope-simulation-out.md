# Static spectral analysis is in scope; propagation simulation is future work

The backend's `/epidemic` endpoint computes the spectral radius and the dominant eigenvector but does not simulate contagion over time. Chapter 6 therefore presents it as static spectral analysis — the spectral radius with its epidemic threshold 1/λ₁, and a per-component **spectral exposure score** (eigenvector centrality) — while dynamic SIR-style **propagation simulation** stays future work (Section 2.4.6, Chapter 8). The thesis does not use "R0" for the per-node score, even though the API field is named `r0`, because a reproduction number would imply a simulation that was never run.

## Considered Options

- Keep all of it as future work: rejected, because it leaves out an implemented endpoint, and Chapter 6's eigenvector centrality rests on the same computation.
- Present it as a full epidemic model: rejected as an overclaim.
