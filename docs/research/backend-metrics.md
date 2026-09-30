# What each graph metric in the SBOM Lens backend actually computes

Source material for Chapter 6. The terms come from `CONTEXT.md`.

## Sources and conventions

- **Code**: the `sbom-lens` repository at commit `6365b09`, which is HEAD as this note is written. All `file:line` references are relative to that repo, e.g. `backend/routers/impact.py:38`.
- **NetworkX**: `backend/requirements.txt:2` pins `networkx>=3.6.1`. NetworkX is not installed in this environment, so the semantics below come from the **NetworkX 3.6.1** wheel downloaded from PyPI. NetworkX references look like `nx:algorithms/dominance.py:15`, meaning a path inside the `networkx/` package of 3.6.1. A deployment that resolves a newer NetworkX could behave differently. Re-check the version before submission.
- **NumPy/SciPy**: the spectral code calls `numpy.linalg.eigh` and `scipy.sparse.linalg.eigs` directly, not NetworkX. For those two functions this note cites their official documentation (numpy.org / docs.scipy.org). `backend/requirements.txt:5` pins `scipy>=1.17.1`.
- "G" always means the directed dependency graph returned by `sbom_store.get_graph`. "G_u" means `G.to_undirected()`.

---

## 1. Graph construction

- **Parser**: `_build_graph(cyclonedx)` at `backend/store/sbom_store.py:14-34` builds an `nx.DiGraph` (`:16`). It runs once when an SBOM is uploaded (`backend/routers/sboms.py:64` → `backend/store/sbom_store.py:45`) and again on `PUT` (`backend/routers/sboms.py:107` → `backend/store/sbom_store.py:62-63`).
- **Nodes**: the node set is `metadata.component`, prepended when present (`sbom_store.py:18-20`), plus every entry of `components[]`. Nested `components` inside a component are **not** walked.
  - **Node id** is `bom-ref`, falling back to `purl`, then `name` (`sbom_store.py:23`). A component with none of these is dropped (`:24`).
  - Two components with the same id collapse into one node. The later one's attributes overwrite the earlier one's (`G.add_node` semantics).
  - **Node attributes** are only the top-level scalar fields of the component: str, int, float or bool (`sbom_store.py:25-28`). Examples are `name`, `version`, `group`, `purl`, `type` and `bom-ref`. Lists and dicts such as `licenses`, `hashes` and `properties` are discarded.
- **Edges**: for each `dependencies[]` entry, the code adds `ref → target` for every `target` in `dependsOn` (`sbom_store.py:29-33`).
  - **Direction is consumer → dependency**, which matches the glossary's "dependency graph".
  - An edge is added only when both endpoints already exist as nodes (`:32`), so dangling references are silently dropped.
  - A `ref` that lists itself becomes a **self-loop**. Nothing removes self-loops, and this matters for k-core and rich-club (see §2.5 and §3.3).
- **Degree meaning under this direction**:
  - `in_degree(v)` is the number of direct consumers (dependents) of v.
  - `out_degree(v)` is the number of direct dependencies of v.
  - `nx.ancestors(G, v)` is everything that transitively depends on v.
  - `nx.descendants(G, v)` is everything v transitively depends on.
- **No virtual root**: whenever the code needs a "root", it takes every node with in-degree 0 (`backend/routers/impact.py:21`, `backend/routers/graph_analysis.py:75`). The glossary's virtual root (CONTEXT.md, "Root") is **not implemented** in the backend.
- **Undirected projection**: many metrics use `G.to_undirected()`. In NetworkX this gives one undirected edge `{u,v}` if either `(u,v)` or `(v,u)` exists, so reciprocal edges merge (`nx:classes/digraph.py:1265-1290`). It is a deep copy, not a view (`as_view=False` default).
- **Storage and caching**:
  - Graphs live only in memory and are lost on restart (`sbom_store.py:1-2`).
  - Every `graph_analysis.py` endpoint memoises its full response in a module-level dict keyed by `sbom_id`: `_communities_cache`, `_kcore_cache`, `_graph_analysis_cache` (`graph_analysis.py:12-14`); `_graph_theory_cache` (`:105`); `_epidemic_cache` (`:291`); `_optimizer_cache` (`:351`).
  - **These caches are never invalidated.** `PUT` and `DELETE` call only `invalidate_audit_cache` (`backend/routers/sboms.py:108`, `:117`; defined at `backend/routers/audit.py:22-24`). After a `PUT`, the k-core, communities, graph-theory, epidemic, optimizer and graph-analysis responses are stale until the server restarts. The audit is cached (`audit.py:16`, `:230-231`, `:251`) and is invalidated.
  - `impact.py` and `metrics.py` do not cache.
- **Routing**: `main.py:17-23` mounts every router under `/sboms`, so for example the k-core endpoint is `GET /sboms/{sbom_id}/kcore`.

---

## 2. Component scale

### 2.1 Articulation points

- **Endpoint**: `GET /sboms/{id}/global-analysis`, function `get_global_analysis`, `backend/routers/impact.py:63-90`.
- **Computation**: `list(nx.articulation_points(G.to_undirected()))` (`impact.py:70-71`). There is no parameter, cap or cache.
- **NetworkX definition**: "any node whose removal (along with all its incident edges) increases the number of connected components of a graph" (`nx:algorithms/components/biconnected.py:265-271`). The function refuses directed input (`@not_implemented_for("directed")`, `:263`), which is why the projection is needed.
- **Response**: `articulationPoints` is an unordered list of node ids (`impact.py:89`).
- **Gap**:
  - These are cut vertices of the **undirected** projection. Removing one splits the graph into more pieces when edges are treated as undirected.
  - That is not the same as "cuts the application off from a dependency". Direction is lost, so a node can be an articulation point purely because it links two sets of consumers, and a node that dominates a subtree from the root need not be one.
  - The thesis's "chokepoint" is defined by the dominated set, not by articulation points (CONTEXT.md). Keep the two apart.

### 2.2 Bridges

- **Endpoint**: `GET /sboms/{id}/optimizer`, inside `_compute_optimizer`, `backend/routers/graph_analysis.py:401-402`. The response field is `bridges`.
- **Computation**: `nx.bridges(G_u)`, the full list with no cap. `G_u = G.to_undirected()` (`:362`).
- **NetworkX definition**: "an edge whose removal causes the number of connected components of the graph to increase. Equivalently, a bridge is an edge that does not belong to any cycle". It is computed via chain decomposition and refuses directed input (`nx:algorithms/bridges.py:11-19`, `:54-59`).
- **Response**: `bridges: [{source, target}]` (`graph_analysis.py:402`, `:425`).
- **Gap**:
  - The code comment calls these "firebreak" edges (`:401`).
  - `source`/`target` is the endpoint order that the undirected graph yields. It is **not** the consumer → dependency direction.
  - Bridges are computed only by the optimizer endpoint. `global-analysis` does not return them.

### 2.3 HITS hubs and authorities

- **Endpoints**:
  - Global: `GET /sboms/{id}/global-analysis`, `backend/routers/impact.py:73-85`.
  - Per node: `GET /sboms/{id}/impact?node_id=…`, `impact.py:44-51`. This path recomputes HITS over the whole graph on every request.
- **Computation**: `nx.hits(G, max_iter=200)` on the **directed** G (`impact.py:46`, `:75`). If `PowerIterationFailedConvergence` is raised, the global endpoint returns `{}` (so every node gets 0.0; `:76-77`) and the per-node endpoint returns 0.0 for both scores (`:49-51`). Values are rounded to 6 decimals (`:57-58`, `:81-82`).
- **NetworkX 3.6.1 algorithm** (`nx:algorithms/link_analysis/hits_alg.py:9-96`):
  - Build `A = adjacency_matrix(G)`, where `A[i,j] = 1` iff `i → j`.
  - Take the leading right singular vector: `svds(A, k=1, maxiter=max_iter, tol=1e-8)`, so `a` is the principal eigenvector of AᵀA (`:85`).
  - Then `h = A a` (`:90`).
  - With `normalized=True` (the default), both vectors are divided by their sum, so each sums to 1 (`:91-93`).
  - `max_iter` is passed to ARPACK as `maxiter`, and ARPACK non-convergence is re-raised as `PowerIterationFailedConvergence` (`:86-87`).
  - This matches Kleinberg (1999), cited at `:63-67`.
- **Meaning under consumer → dependency edges**:
  - **Authority** is high for components depended on by many strong hubs: widely used libraries.
  - **Hub** is high for components that depend on many authorities: aggregator or application packages.
- **Response**:
  - global-analysis: `hits: {nodeId: {hub, authority}}` (`impact.py:79-85`, `:90`).
  - impact: `hitsHub`, `hitsAuthority` (`impact.py:57-58`).
- **Gap**:
  - The scores are sum-normalised to 1 over the whole graph and rounded to 6 d.p. On large SBOMs many nodes therefore round to `0.0`.
  - The values are shares, not probabilities or max-scaled scores.
  - The code comment at `impact.py:44` says the global HITS is "in /global-analysis". That is true, but the per-node route does not reuse it.

### 2.4 Spectral exposure score (API field `r0`)

- **Endpoint**: `GET /sboms/{id}/epidemic`, `_compute_epidemic`, `backend/routers/graph_analysis.py:294-334`. It is cached (`:339-340`, `:347`).
- **Graph**: `G_u = G.to_undirected()` (`:302`). If `len(G) < 2`, the function returns `spectralRadius: None, globalR0: None, nodes: []` (`:298-300`).
- **Eigen-decomposition**:
  - **n ≤ 500** (`:304-308`): dense `A = nx.to_numpy_array(G_u)`, then `numpy.linalg.eigh(A)`. NumPy documents that eigh returns eigenvalues "in ascending order", each eigenvector normalised to unit length. The code takes `λ₁ = eigenvalues[-1]` and `v₁ = |eigenvectors[:, -1]|`.
  - **n > 500** (`:309-315`): sparse `nx.to_scipy_sparse_array(G_u, format="csr")`, then `scipy.sparse.linalg.eigs(A, k=1, which="LM")`, where `"LM"` means largest *magnitude* (SciPy docs). The code takes `λ₁ = Re(vals[0])` and `v₁ = |Re(vecs[:,0])|`.
  - On any exception, it falls back to the dense eigh path (`:316-321`).
  - The absolute value is justified in the comment by Perron–Frobenius (`:308`).
  - A self-loop in G becomes a diagonal entry of 1 in A, following the NetworkX convention that the diagonal holds the loop weight (`nx:convert_matrix.py:962-963`).
- **Per-node score**: `r0_i = λ₁ · v₁[i] / max_j v₁[j]`, rounded to 4 d.p. (`:323-328`). If `max v₁ = 0`, the divisor becomes 1 (`:323`).
- **Response**: `nodes: [{id, r0}]` (`:333`).
- **Gap**:
  - The glossary's *spectral exposure score* is "a component's entry in the dominant eigenvector — its eigenvector centrality". The code returns that entry **max-normalised and multiplied by λ₁**. The top node therefore scores exactly λ₁, and the scores are neither unit-L2-norm (like `nx.eigenvector_centrality`, `nx:algorithms/centrality/eigenvector.py:13`, whose docstring states unit Euclidean norm at `:73`) nor shares summing to 1.
  - The ranking is identical to eigenvector centrality, but the magnitude is not.
  - ADR 0001: the field is named `r0`, but it is not a reproduction number, and no simulation runs.
  - The `abs()` step is valid only when the leading eigenvalue is simple. That holds on a connected undirected graph (Perron–Frobenius). If G_u is **disconnected**, v₁ is supported on the component with the largest λ₁, and every node in other components scores 0. If two components tie, the eigenvector is an arbitrary mix and `abs()` is not meaningful.
  - For **n > 500**, `which="LM"` returns the eigenvalue of largest magnitude. On a bipartite graph (every tree is bipartite) the spectrum is symmetric, so `−λ₁` has the same magnitude. ARPACK may then return a negative "spectral radius", which also flips the sign of every `r0` and of `r0ReductionPct`. This is not guarded. The dense path does not have the problem.

### 2.5 k-core

- **Endpoint**: `GET /sboms/{id}/kcore`, `get_kcore`, `backend/routers/graph_analysis.py:42-62`. It is cached.
- **Computation**: `nx.core_number(G.to_undirected())` (`:51`). `maxCore` is the maximum core number, or 0 for an empty graph (`:52`). `innermostNodes` is every node whose core number equals `maxCore` (`:53`).
- **NetworkX definition**: "A k-core is a maximal subgraph that contains nodes of degree k or more. The core number of a node is the largest value k of a k-core containing that node". It uses the Batagelj–Zaversnik O(m) algorithm (`nx:algorithms/core.py:47-86`).
  - Here the degree is the undirected degree, i.e. the number of distinct neighbours after reciprocal edges merge.
  - **It raises `NetworkXNotImplemented` if the graph has self-loops** (`nx:algorithms/core.py:92-97`). The endpoint does not catch this, so an SBOM whose dependency list contains a self-reference returns HTTP 500.
- **Response**: `coreNumbers {id: k}`, `maxCore`, `innermostNodes` (`graph_analysis.py:55-60`).
- **Gap**: direction is ignored. A high core number means densely interlinked neighbourhoods in the projection, not "deep in the dependency chain".

### 2.6 Topology optimizer

- **Endpoint**: `GET /sboms/{id}/optimizer`, `_compute_optimizer`, `backend/routers/graph_analysis.py:354-427`. It is cached.
- **Setup**: if `n < 2` it returns empty lists (`:359-360`). Otherwise it recomputes `λ₁` and `v₁` on G_u with the same dense/sparse/fallback logic as §2.4 (`:362-382`). All the caveats from §2.4 apply: disconnected graphs, and a possible negative λ₁ for n > 500.

**Edge rankings (OPT-01)** (`:384-399`):
- For every undirected edge `{u,v}` of G_u: `deltaLambda1 = 2·v₁[u]·v₁[v]`, rounded to 6 d.p.
- `r0ReductionPct = 100·deltaLambda1/λ₁`, rounded to 2 d.p. It is 0.0 if λ₁ ≤ 0.
- Edges are sorted by `deltaLambda1` descending, and the **top 20** are returned (`:398-399`).
- The formula is the first-order perturbation estimate of the drop in λ₁ when edge (u,v) is removed from a symmetric adjacency matrix with a unit-norm v₁: Δλ₁ ≈ 2·v_u·v_v. This is the edge score used in link-removal spectral-radius reduction (e.g. Van Mieghem et al. 2011; Tong et al. 2012, "NetMelt").
- Response: `edgeRankings: [{source, target, deltaLambda1, r0ReductionPct}]`.

**Bridges (OPT-02)**: see §2.2.

**Abstraction candidates (OPT-03)** (`:404-421`):
- The per-node score is recomputed exactly as in §2.4 (`r0Score = λ₁·v₁[i]/max v₁`).
- The candidate set is **every node with directed in-degree ≥ 3** (`FAN_IN_THRESHOLD = 3`, `:407-410`), i.e. at least three direct consumers.
- Candidates are sorted by `(r0Score, fanIn)` descending, and the top 20 are returned (`:420-421`).
- Response: `abstractionCandidates: [{id, fanIn, r0Score, explanation}]`. `explanation` is the same template string for every candidate (`:415-418`).

**Gaps**:
- `deltaLambda1` is a linear approximation, not a recomputed eigenvalue.
- `r0ReductionPct` is the approximate percentage drop in **λ₁**, not in any reproduction number.
- For a self-loop edge the true first-order term is `v_u²`, not `2v_u²`.
- `source`/`target` again follow undirected order.
- The "abstraction candidate" rule has **no threshold on the spectral score**. Every node with fan-in ≥ 3 qualifies. Yet the explanation text claims an "elevated R₀ score" for each candidate (`:416`).

---

## 3. Cluster scale

### 3.1 Louvain communities

- **Endpoint**: `GET /sboms/{id}/communities`, `get_communities`, `backend/routers/graph_analysis.py:17-39`. It is cached.
- **Computation**: `louvain_communities(G.to_undirected(), seed=42)` (`:26`). All other parameters keep their defaults: `weight="weight"`, `resolution=1`, `threshold=1e-7`, `max_level=None` (`nx:algorithms/community/louvain.py:16-18`).
  - No edge carries a `weight` attribute, so every edge counts as weight 1.
  - Community ids are the enumeration order of the returned list (`graph_analysis.py:28-30`).
- **NetworkX algorithm**: Louvain greedy modularity optimisation (Blondel et al. 2008). Nodes are visited in a random-shuffle order, which is why the seed exists. Levels repeat until the modularity gain falls below `threshold` (`nx:algorithms/community/louvain.py:19-60`; random shuffle at `:106-107`). Self-loops are treated as "previously reduced communities" (`:62-68`).
- **Response**: `communityCount`, `communityMap {id: communityIndex}`, `communitySizes [int]` (`graph_analysis.py:32-37`).
- **Gap**: the modularity value itself is **not** returned. Communities come from the undirected projection. The result is deterministic for a fixed seed and NetworkX version.

### 3.2 Bow-tie decomposition

- **Endpoint**: `GET /sboms/{id}/graph-theory` → `bowTie`, `_compute_bow_tie`, `backend/routers/graph_analysis.py:202-248`.
- **Computation** on the directed G:
  - `sccs = list(nx.strongly_connected_components(G))` (`:219`; NetworkX yields a set per SCC, `nx:algorithms/components/strongly_connected.py:17-29`).
  - `giant_scc = max(sccs, key=len)` (`:220`).
  - `OUT = descendants(G, any node of giant_scc) ∪ giant_scc − giant_scc` (`:222`, `:229`). NetworkX defines `descendants` as "all nodes reachable from source" (`nx:algorithms/dag.py:39-50`).
  - `IN = ⋃ ancestors(G, n) for n ∈ giant_scc − giant_scc` (`:224-228`). NetworkX defines `ancestors` as "all nodes having a path to source" (`nx:algorithms/dag.py:76-88`).
  - `tendrils = V − giant_scc − IN − OUT` (`:230`).
  - `nonTrivialSccCount` is the number of SCCs with more than one node (`:232`).
  - Percentages are over all nodes, rounded to 2 d.p. (`:234-235`).
- **Response**: `scc, in_set, out_set, tendrils, total, scc_pct, in_pct, out_pct, tendrils_pct, nonTrivialSccCount` (`:237-248`).
- **Gap**:
  - In the Broder et al. bow-tie, the core is a *giant* SCC. A dependency graph is usually acyclic or nearly so, and then every SCC has size 1. `max` then returns **an arbitrary single node** (the first size-1 SCC in generator order) as the "core", and IN, OUT and tendrils are measured relative to that one node. The decomposition is only meaningful when `nonTrivialSccCount > 0` and the largest SCC is large. The code does not check this.
  - `tendrils` lumps together true tendrils, tubes and disconnected components.
  - Under consumer → dependency edges, IN means the consumers of the core and OUT means the dependencies of the core.

### 3.3 Rich-club coefficient

- **Endpoint**: `GET /sboms/{id}/graph-theory` → `richClub`, `_compute_rich_club`, `backend/routers/graph_analysis.py:251-268`.
- **Computation**: `nx.rich_club_coefficient(G.to_undirected(), normalized=False, seed=42)` (`:257`). If there are fewer than 2 nodes, it returns empty/None (`:254-255`).
- **NetworkX definition**: φ(k) = 2·E_k / (N_k·(N_k−1)), where N_k is the number of nodes with degree **greater than** k and E_k is the number of edges among them (`nx:algorithms/richclub.py:14-26`, implemented in `_compute_rc`, `:103-138`, formula at `:137`).
  - With `normalized=False` there is no random-graph (double-edge-swap) null model, and **the `seed` has no effect** (`:92-99`).
  - Keys run only over degrees where N_k > 1.
  - **It raises a bare `Exception` if the graph has self-loops** (`:87-90`). This is uncaught, so the whole `/graph-theory` response becomes HTTP 500.
- **Post-processing**:
  - `rhoByDegree = [{k, rho}]`, rounded to 4 d.p. (`graph_analysis.py:258-260`).
  - `threshold` is the smallest k with φ(k) > 0.5 (`:261`).
  - `maxRho` is the maximum φ (`:262`).
- **Gap**: the coefficient is **unnormalised**, so it cannot say whether hubs link to each other more than chance would (McAuley et al. 2007, the paper NetworkX cites, argues for normalisation). The `threshold` at 0.5 is an ad-hoc cut-off with no source in the code. Degree is undirected degree.

---

## 4. Whole-graph scale

### 4.1 Spectral radius and epidemic threshold

- **Endpoint**: `GET /sboms/{id}/epidemic`, `backend/routers/graph_analysis.py:294-348`.
- **Computation**: λ₁ as in §2.4, rounded to 4 d.p. It is returned **twice**: `spectralRadius` and `globalR0` hold the same value (`:330-332`). The comment reads `# β=δ=1 → R0 = λ1` (`:332`).
- **Response**: `spectralRadius`, `globalR0`, `nodes` (`:330-334`).
- **Gap**:
  - The glossary defines the epidemic threshold as 1/λ₁. **The backend does not compute or return 1/λ₁**, so the thesis must derive it. The frontend shows `globalR0` and `spectralRadius` directly (`src/layouts/graph-analysis/index.js:686-689`).
  - `globalR0` is λ₁ under an assumed β = δ = 1 (SIS spreading-to-recovery ratio 1). It is not an estimated reproduction number (ADR 0001).
  - λ₁ is computed on the **undirected** adjacency, not the directed one.
  - For n > 500, the value can come out negative on bipartite graphs (§2.4). λ₁ equals the spectral radius only when it is the largest-magnitude eigenvalue.

### 4.2 Degree entropy

- **Endpoint**: `GET /sboms/{id}/audit` → `entropy`, `_compute_entropy`, `backend/routers/audit.py:27-47`.
- **Computation**:
  - The histogram of **out-degrees** over all nodes of G, including out-degree 0 (`:33`).
  - `H = −Σ_k p(k)·log₂ p(k)`, where p(k) is the fraction of nodes with out-degree k (`:35-39`). It is rounded to 3 d.p. (`:40`).
  - Label: `Low` if H < 1.5, `Medium` if 1.5 ≤ H ≤ 3.0, `High` if H > 3.0 (`:41-46`).
  - An empty graph returns `{bits: 0.0, label: "Low"}` (`:30-31`).
- **Response**: `entropy: {bits, label}` (`:47`, `:245`).
- **Gap**:
  - The docstring ("Shannon entropy of out-degree distribution", `:28`) is accurate. Under consumer → dependency edges, **out-degree is the number of direct dependencies**, not dependents. This measures how varied the direct-dependency counts are, not how concentrated consumers are on popular libraries (which would be in-degree).
  - H is bounded by log₂ of the number of distinct out-degree values, so it grows with graph size.
  - The 1.5 and 3.0 bands have no stated source.
  - No NetworkX function is involved.

### 4.3 Cycles and SCCs

- **Endpoint**: `GET /sboms/{id}/audit` → `cycles`, `_compute_cycles`, `backend/routers/audit.py:50-60`.
- **Computation**: `list(islice(nx.simple_cycles(G), 1000))` on the **directed** G (`:52`; `CYCLES_CAP = 1000`, `:19`).
  - `hit_cap` is `len == 1000` (`:53`).
  - `count` is the int, or the string `"1000+"` when the cap is hit (`:57`).
  - `cycleNodeIds` is the union of nodes in the enumerated cycles (`:55`).
  - `topCycles` is the first 10 cycles yielded (`:59`).
- **NetworkX**:
  - A simple cycle is "a closed path where no node appears twice". Distinct means "not cyclic permutations of each other" (`nx:algorithms/cycles.py:106-112`).
  - The unbounded case uses Johnson's algorithm, restricted to SCCs (`:113-122`).
  - **Self-loops count as cycles of length 1** (docstring `:129-131`). They are yielded **first**, before Johnson's search (`:205`), so any self-loops appear at the head of `topCycles`.
- **SCCs**: the audit does **not** report SCCs. The only SCC figure in the backend is `nonTrivialSccCount` in the bow-tie (§3.2, `graph_analysis.py:232`).
- **Response**: `cycles: {count, cycleNodeIds, topCycles}`.
- **Gap**:
  - `count` has mixed types (int or string).
  - If exactly 1000 cycles exist, the code reports `"1000+"`.
  - `topCycles` are simply the first ten in generation order, not ranked.
  - When the cap is hit, `cycleNodeIds` covers only the enumerated cycles, so it may miss nodes that lie on cycles. The node set of non-trivial SCCs would give the complete set.

### 4.4 Version conflicts (API: "diamonds")

- **Endpoint**: `GET /sboms/{id}/audit` → `diamonds`, `_compute_diamonds`, `backend/routers/audit.py:63-92`.
- **Computation**:
  - Nodes are grouped by the raw `name` attribute (`:68-73`).
  - A group is flagged when it has **≥ 2 distinct non-empty `version` strings** (`:79-80`). An empty version is never counted as a conflict (`:78`).
  - The code does not look at edges.
- **Response**: `diamonds: {count, diamonds: [{package, versions (sorted), nodeIds}], diamondNodeIds}` (`:88-92`).
- **Gap**:
  - This is the glossary's **version conflict**, not a diamond: no graph structure (A→B, A→C, B→D, C→D) is checked.
  - The grouping key is `name` only. `group`/namespace is ignored even though it is stored as a node attribute (`sbom_store.py:25-28`). Two different packages with the same name in different groups are therefore reported as one conflicting package: Maven artifacts with the same artifactId in different groupIds, or npm scoped packages if the generator puts the scope in `group`.
  - Version strings are compared literally, so `1.0` and `1.0.0` count as a conflict.

### 4.5 Namespace concentration

- **Endpoint**: `GET /sboms/{id}/audit` → `concentration`, `_compute_concentration` and `_extract_namespace`, `backend/routers/audit.py:95-137`.
- **Namespace rule** (`:95-114`):
  - `pkg:npm/@scope/name` gives `@scope`. **Unscoped npm is skipped** (`None`, `:102`).
  - `pkg:pypi/x-y-z` gives the first `-`-separated token, `x` (`:103-106`).
  - Otherwise the rule falls back to the `name`: `@scope/…` gives `@scope`, and a dotted name gives its first two dot-segments, e.g. `org.springframework` (`:108-113`).
  - Everything else is `None` and not counted.
- **Computation**:
  - Nodes are counted per namespace (`:119-125`).
  - `pct = count/total·100`, rounded to 1 d.p., where **`total` counts only nodes that received a namespace** (`:127-130`).
  - The top 10 are returned (`:130`).
  - `concentrationRisk` is `top pct > 60` (`:136`).
- **Response**: `concentration: {topNamespaces: [{namespace, count, pct}], totalPackages, concentrationRisk}` (`:133-137`).
- **Gap**:
  - "Namespace" is a heuristic. For PyPI it is a name prefix, e.g. `python-dateutil` → `python`, not a registry namespace.
  - Maven PURLs (`pkg:maven/group/name`) are not parsed; only a dotted `name` would be used.
  - The npm check looks for a literal `@`. The PURL spec percent-encodes it as `%40`, so encoded scoped npm PURLs fall through to "unscoped", unless the fallback on `name` catches them.
  - The percentage denominator excludes unclassified packages, so `pct` can overstate concentration relative to the whole SBOM.
  - The 60 % threshold has no stated source.
  - No NetworkX function is involved.

### 4.6 Power-law fit (API: `scaleFree`)

- **Endpoint**: `GET /sboms/{id}/graph-theory` → `scaleFree`, `_compute_scale_free`, `backend/routers/graph_analysis.py:108-135`.
- **Computation**:
  - The degree used is **in-degree** (number of direct consumers). Nodes with in-degree 0 are excluded (`:110`).
  - If fewer than 3 such **nodes** exist, the function returns nulls (`:111-112`).
  - Otherwise it builds the frequency table `count(d)` for each distinct in-degree d (`:117-119`) and runs **ordinary least-squares linear regression** of `log₁₀ count(d)` on `log₁₀ d` with `scipy.stats.linregress` (`:121-123`).
  - `alpha = −slope`, rounded to 3 d.p. `r_squared = r²`, rounded to 3 d.p. (`:124`, `:131-132`).
  - `hubNodes` is the top 5 nodes by in-degree (`:127-128`).
  - `logLogData` is the raw `(d, count)` pairs (`:125`).
- **Response**: `alpha, r_squared, hubNodes, logLogData` (`:130-135`).
- **Gap**:
  - The estimator is OLS on an unbinned log-log histogram. There is no maximum-likelihood estimate, no x_min selection and no goodness-of-fit test against alternatives (Clauset, Shalizi & Newman 2009 show this estimator is biased). R² is not evidence of a power law.
  - The docstring says "Fit a power-law to the in-degree distribution" (`:109`), which is accurate on degree choice.
  - Edge case: the guard counts *nodes*, not *distinct degrees*. If all in-degree>0 nodes share one degree, `linregress` receives identical x values and raises, which is uncaught and gives HTTP 500. With exactly 2 distinct degrees, r² = 1 trivially.

### 4.7 Small-world coefficient σ

- **Endpoint**: `GET /sboms/{id}/graph-theory` → `smallWorld`, `_compute_small_world`, `backend/routers/graph_analysis.py:138-199`.
- **Computation**:
  - `G_u = G.to_undirected()`. If `|G_u| < 4`, it returns nulls (`:142-154`).
  - It restricts to the **largest connected component** of G_u (`:156-159`), then sets n and m to that component's node and edge counts.
  - `C = nx.average_clustering(G_lcc)` (`:161`). NetworkX gives `C = (1/n) Σ c_v` with `c_u = 2T(u)/(deg(u)(deg(u)−1))` (`nx:algorithms/cluster.py:325-334`, `:388-396`). With the default `count_zeros=True`, nodes of degree < 2 contribute 0 (`:347`, `:410`).
  - **L**, the average shortest path length:
    - If `n ≤ 500`, `nx.average_shortest_path_length(G_lcc)`, i.e. `Σ_{s≠t} d(s,t)/(n(n−1))` (`nx:algorithms/shortest_paths/generic.py:326-336`, `:438`).
    - Otherwise it **samples 500 source nodes with `random.sample` and no seed** (`graph_analysis.py:166-174`), averages all BFS distances from them (`single_source_shortest_path_length`, which includes the source at distance 0, `nx:algorithms/shortest_paths/unweighted.py:22`), and sets `sampled: true`.
  - **Random baseline (analytic, not simulated)** (`:176-178`):
    - `p = 2m/(n(n−1))`.
    - `C_random = p`.
    - `L_random = ln n / ln(n·p)`, or ∞ if n·p ≤ 1.
    - These are the Erdős–Rényi approximations.
  - `cRatio = C/C_random`, `lRatio = L/L_random`, and `σ = cRatio/lRatio` (`:180-186`). This is the Humphries & Gurney (2008) small-world coefficient.
  - Rounding: σ to 3 d.p.; C, L, cRatio, lRatio and lccFraction to 4 d.p. (`:190-199`).
- **Response**: `sigma, avgClustering, avgPathLength, cRatio, lRatio, lccSize, lccFraction, sampled`.
- **Gap**:
  - There **is** a random-graph baseline and a σ, but it is the closed-form ER approximation. It does not use NetworkX's `nx.sigma`, which averages over `nrand` degree-preserving rewired references (`nx:algorithms/smallworld.py:250-262`).
  - The ER baseline ignores the degree sequence.
  - For n > 500 the result is **non-reproducible** because the sample is unseeded. It is fixed only by the cache until restart.
  - The sampled L includes the 500 zero self-distances in the denominator, so it underestimates L by a factor of about (n−1)/n. The error is negligible, but the method differs from the exact path.
  - Tree-like graphs have C = 0, which gives `sigma = 0.0` rather than null.
  - Everything is computed on the undirected LCC only. `lccFraction` reports its share of nodes.

---

## 5. Also in the backend

### 5.1 Blast radius

- **Endpoint**: `GET /sboms/{id}/impact?node_id=…`, `get_node_impact`, `backend/routers/impact.py:9-60`. It is not cached. An unknown node returns 404 (`:14-15`).
- **Computation**: `list(nx.ancestors(G, node_id)) + [node_id]` (`:18`). This gives every node with a directed path *to* the node, i.e. all transitive consumers (`nx:algorithms/dag.py:76-88`), plus the node itself.
- **Response**: `blastRadius`, a list with the node last and the rest unordered (`:55`).
- **Gap**: this matches the glossary, with one difference: the list **includes the component itself**, so |blastRadius| = consumers + 1. The backend does not compute **vulnerability reach** (the share of the graph inside at least one vulnerable blast radius). The thesis or evaluation pipeline has to derive it from the per-node blast radii.

### 5.2 Dominators (field `dominatorTree`)

- **Endpoint**: same function, `backend/routers/impact.py:34-42`.
- **Computation**: `idom = nx.immediate_dominators(G, node_id)` with `start = node_id` (`:38`). Then `dominated = [n for n, d in idom.items() if d == node_id]` (`:39`). If `NetworkXError` is raised, the result is `[]` (`:40-42`).
- **NetworkX 3.6.1**: the function returns "the immediate dominators of each node reachable from `start`, except for `start` itself". It uses the Cooper–Harvey–Kennedy iterative algorithm (`nx:algorithms/dominance.py:15-89`; `start` is removed from the result at `:89`).
- **What this yields under consumer → dependency edges**:
  - The nodes reachable from `node_id` are its **dependencies**, not its consumers.
  - The result is the set of the vulnerable component's dependencies whose immediate dominator is the vulnerable component. That is **one level** of the dominator tree rooted at the vulnerable node.
- **Response**: `dominatorTree` (`:56`).
- **Gap**:
  - ADR 0002 records the decision to compute dominance **from the root**, taking a component's dominated set as its subtree. **At commit `6365b09` the code still starts from `node_id`**, so the fix has not been applied. `git log` shows `impact.py` was last changed in `b953b84`.
  - Even after re-rooting, `d == node_id` would return only the *immediately* dominated children, not the full dominated set (the whole subtree). The glossary warns that "dominator tree" is the wrong name for this field.
  - With no virtual root and several in-degree-0 nodes, "root" is ambiguous.

### 5.3 Dependency paths (shortest paths)

- **Endpoint**: same function, `backend/routers/impact.py:20-32`.
- **Computation**:
  - The roots are all in-degree-0 nodes in `G.nodes()` insertion order (`:21`); the metadata component comes first (`sbom_store.py:19-20`).
  - For each root, the code collects `nx.all_shortest_paths(G, root, node_id)` (unweighted BFS, `nx:algorithms/shortest_paths/generic.py:442-470`).
  - It stops at **5 paths in total** (`impact.py:27-32`).
  - `NetworkXNoPath` and `NodeNotFound` are swallowed (`:29-30`).
- **Response**: `paths`, lists of node ids ordered root → … → node (`:59`).
- **Gap**:
  - These are only *shortest* paths, not all paths.
  - The cap is global, so if the first root has ≥ 5 shortest paths, no other root is sampled.
  - A node reachable only through a cycle with no in-degree-0 ancestor gets `[]`.

### 5.4 Unmaintained check

- **Endpoint**: `GET /sboms/{id}/audit` → `unmaintained`, `_check_unmaintained`, `backend/routers/audit.py:140-225`.
- **Computation**:
  - Only nodes with `pkg:npm/` or `pkg:pypi/` PURLs are checked (`:192-199`).
  - Cutoff: `now − 365·2 days` (`UNMAINTAINED_YEARS = 2`, `:18`, `:188`).
  - All requests run concurrently with a 5 s timeout (`:204-207`).
  - **npm**: GET `registry.npmjs.org/{name}`. A record with no `versions` is treated as a tombstone and skipped. The date used is **`time.modified`** (`:146-157`).
  - **PyPI**: GET `pypi.org/pypi/{name}/json`, then take the latest `upload_time_iso_8601` (falling back to `upload_time`) across all files of all releases (`:168-181`).
  - A package is flagged if its date is earlier than the cutoff (`:214`).
  - Any failure, non-200 response or missing date increments `skippedCount` (`:209-212`).
- **Response**: `unmaintained: {unmaintainedCount, unmaintained: [{nodeId, package, lastRelease}], skippedCount}` (`:221-225`).
- **Gap**:
  - The npm docstring promises the "last release date" (`:142`), but `time.modified` is the last modification of the registry document, which metadata-only changes can bump. It can make a package look maintained when it is not.
  - Maven and other ecosystems are never checked, and they are counted neither as skipped nor as maintained.
  - The result depends on the query date, and it is cached until a `PUT`/`DELETE` or restart.

### 5.5 Other endpoints

- **`routers/metrics.py`** (`GET /sboms/{id}/metrics`, `backend/routers/metrics.py:8-25`) returns `nodeCount`, `edgeCount`, `avgOutDegree` (= m/n, 2 d.p.) and `topDependedOn`, the top 5 by **out-degree** (`:16-17`, `:24`). **Mislabel**: out-degree is the number of direct dependencies, so these are the components that *depend on* the most, not the most *depended on*. That would be in-degree.
- **`/graph-analysis`** (`backend/routers/graph_analysis.py:65-102`):
  - `depthHistogram` is the minimum BFS depth from any in-degree-0 root (`:75-86`).
  - `degreeHistogram` is the **in-degree** histogram (`:89-92`), even though the field is called just "degree".
  - The comment says unreachable nodes "get depth -1" (`:83`). In fact they are simply omitted from the histogram.

---

## 6. Discrepancies

| Metric | Name implies | Code does | file:line |
|---|---|---|---|
| Dominators (`dominatorTree`) | Dominated set from the application root (glossary; ADR 0002) | `immediate_dominators` started at the **vulnerable node**, which gives its own exclusive dependencies, and only the immediate children (one level) | `backend/routers/impact.py:38-39` |
| Spectral exposure score (`r0`) | Reproduction number / eigenvector centrality entry | `λ₁·v₁[i]/max v₁` on the undirected adjacency, with no simulation; max node = λ₁ | `backend/routers/graph_analysis.py:323-328` |
| `globalR0` | Reproduction number | A duplicate of `spectralRadius` (λ₁, undirected), under an assumed β = δ = 1 | `backend/routers/graph_analysis.py:331-332` |
| Epidemic threshold | 1/λ₁ returned by the tool | Not computed anywhere in the backend | `backend/routers/graph_analysis.py:330-334` |
| λ₁ for n > 500 | Spectral radius (≥ 0) | `eigs(which="LM")` can return −λ₁ on bipartite graphs; unguarded | `backend/routers/graph_analysis.py:313-314`, `:375-376` |
| Version conflicts (`diamonds`) | Diamond dependency structure | Same `name` with ≥ 2 distinct non-empty versions; no edges examined, `group` ignored | `backend/routers/audit.py:63-92` |
| Degree entropy | Spread of dependents / popularity | Shannon entropy of the **out-degree** (direct dependencies) histogram; bands 1.5/3.0 are ad hoc | `backend/routers/audit.py:33-46` |
| Power-law `alpha` | Fitted scale-free exponent | OLS on log₁₀ frequency vs log₁₀ in-degree, no MLE or x_min; can raise when all degrees are equal | `backend/routers/graph_analysis.py:110-124` |
| Small-world σ | σ against a random reference (e.g. `nx.sigma`) | Analytic ER baseline `C_r = p`, `L_r = ln n/ln(np)` on the undirected LCC; L sampled without a seed for n > 500 | `backend/routers/graph_analysis.py:156-186` |
| Bow-tie core | Giant SCC | `max(sccs, key=len)`; in an acyclic graph this is an arbitrary single node | `backend/routers/graph_analysis.py:219-230` |
| Bow-tie `tendrils` | Tendrils | Everything not in core, IN or OUT, including tubes and disconnected parts | `backend/routers/graph_analysis.py:230` |
| Rich-club | Rich-club effect vs. null model | Unnormalised φ(k); `seed=42` unused; `threshold` = first k with φ > 0.5 | `backend/routers/graph_analysis.py:257-261` |
| Abstraction candidates | High fan-in **and** elevated spectral score | Every node with in-degree ≥ 3; no score threshold; the same "elevated R₀" text for all | `backend/routers/graph_analysis.py:407-421` |
| Edge rankings `r0ReductionPct` | Reduction in reproduction number | First-order estimate `2v_u v_v` of the λ₁ drop, as % of λ₁; the undirected endpoint order is shown as source/target | `backend/routers/graph_analysis.py:388-397` |
| Articulation points / bridges | Components/edges the application cannot route around | Cut vertices / cut edges of the **undirected** projection | `backend/routers/impact.py:70-71`, `backend/routers/graph_analysis.py:402` |
| k-core | Structural core of the dependency graph | Undirected core number; HTTP 500 if any self-loop | `backend/routers/graph_analysis.py:51` |
| Rich-club / graph-theory | — | HTTP 500 if any self-loop (NetworkX raises) | `backend/routers/graph_analysis.py:257` |
| Cycles `count` | Number of cycles | int, or `"1000+"` when exactly 1000 are enumerated; self-loops count as cycles | `backend/routers/audit.py:52-59` |
| Namespace concentration `pct` | Share of the SBOM | Share of the *namespaced* subset only; unscoped npm skipped; PyPI "namespace" = first hyphen token | `backend/routers/audit.py:95-137` |
| Unmaintained (npm) | Last release date | `time.modified` of the registry document | `backend/routers/audit.py:153-157` |
| Blast radius | Transitive consumers | Transitive consumers **plus the node itself** | `backend/routers/impact.py:18` |
| `topDependedOn` | Most depended-on components | Top 5 by **out-degree** (most dependencies) | `backend/routers/metrics.py:17`, `:24` |
| `degreeHistogram` | Degree histogram | In-degree histogram | `backend/routers/graph_analysis.py:89-92` |
| graph_analysis caches | Current results | Never invalidated on `PUT`/`DELETE` (only the audit cache is) | `backend/routers/graph_analysis.py:12-14`; `backend/routers/sboms.py:108`, `:117` |
| Root | One (virtual) root | Every in-degree-0 node; no virtual root | `backend/routers/impact.py:21`; `backend/routers/graph_analysis.py:75` |
| Vulnerability reach | Computed by the tool | Not computed in the backend | — |
