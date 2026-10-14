<!-- solve-problem:207f955b-d4ae-42e2-8db4-ed8d9c156b28 -->
# Privacy-preserving mining-topology measurement

A [Tangle Solve problem](https://solve.tangle.tools/problems/207f955b-d4ae-42e2-8db4-ed8d9c156b28).

## What is the problem?

**Setting.** Bitcoin-like proof-of-work networks form a peer-to-peer gossip network (the "mining topology" — which nodes miners connect to, propagation paths of blocks/transactions). This topology is deliberately obscured: nodes do not advertise identities, peers are chosen pseudo-randomly, and_addr-relay is not directly observable. The problem: design a measurement methodology that (1) reconstructs or statistically estimates properties of the mining/connectivity topology (propagation delay structure, node degree distribution, centrality of miner-adjacent nodes, partition risk), and (2) provides **quantified uncertainty under missing data** — i.e., a coverage guarantee or confidence statement about which parts of the topology are unobserved, and (3) is **privacy-preserving**: the published measurement must leak as little as possible that would help an adversary perform targeted attacks (eclipsing, BGP hijacking, partition attacks, transaction-origin deanonymization).

Formally: given observations O collected via an active or passive measurement protocol P on a network G = (V, E), produce an estimate f(O) ≈ T(G) for a topology statistic T (e.g., degree distribution, betweenness of relay nodes, block-propagation-delay graph), together with (a) a missingness model M(O, G) quantifying what P fails to see, and (b) a disclosure bound, e.g., a differential-privacy-style guarantee: the released statistic is ε-insensitive to any single node's adjacency, so it cannot be inverted into a targeting directory of high-value nodes.

**Ambiguity note:** "mining topology" may mean (a) the full P2P graph restricted to miner-adjacent nodes, or (b) the block-propagation DAG. The brief covers both; a solution should state which.

## Why does it matter?

A privacy-preserving map of mining connectivity would let researchers and auditors quantify network-layer centralization and partition vulnerability without handing attackers a directory of the exact nodes to eclipse or hijack — precisely the failure mode of naive measurement. Decker–Wattenhofer-style results (block propagation delay → stale rates → security) could be continuously monitored on mainnet rather than inferred from old testnet snapshots. Conversely, if it stays open, the community must choose between blindness and dangerous disclosure; adversarial actors with superior private telemetry gain an asymmetric advantage.

## What's already known?

- **Decker & Wattenhofer (2013)**: passive measurement via block-relay timing on the pre-2013 Bitcoin network; established that propagation delay dominates stale rate. Requires a large vantage-point deployment; doesn't address missingness or privacy.
- **Delgado-Segura et al. TxProbe (2019)**: active probing using orphan transactions to reconstruct topology on Bitcoin **testnet** via temporarily-accepted orphans tracing peer links. Mainnet-specific obstacles (address-tx relay semantics changed, orphan handling, potential griefing/DoS concerns, and — critically — the released topology itself is a targeting directory). TxProbe gives no missing-data quantification and no disclosure bound.
- **addr-timing / watermarking attacks** (e.g., Miller et al. 2015 "Discovering Bitcoin's Public Network Topology and Influential Nodes" — (verify)) infer adjacency via ADDR-message timing; again disclosure-full and measurement-noise-unquantified.
- **Related theory**: privacy-preserving network tomography and differentially private graph statistics (edge-DP degree distribution, noisy spectral estimates) exist in the statistics literature but have not been combined with active blockchain probing or applied to miner subgraphs.
- **Gap**: no method jointly does (estimation) + (missingness quantification) + (disclosure bound) on mainnet.

## What would count as solving it?

A rigorous methodology R with all three components demonstrated:
1. **Estimation**: R recovers target statistics T(G) on testnet to stated accuracy, and on mainnet via a deployment that does not harm consensus.
2. **Missing-data inference**: a formal model (e.g., MCAR/MAR over vantage-point coverage) yielding confidence intervals for T(G) validated against ground truth (testnet or simulated networks with known G, e.g., shadow-network simulators like Shadow/BitNodes-style ground truth — verify availability).
3. **Disclosure bound**: a proof (edge/node-level differential privacy or an adversary-game lower bound) that the published output limits an attacker's ability to identify high-centrality miner-adjacent nodes; ideally a quantified utility–privacy tradeoff curve.

Variants: (a) full DP-style guarantee with weaker accuracy; (b) tighter accuracy with only empirical disclosure experiments; (c) lower-bound result showing inherent tension between topology fidelity and privacy — any of these is publishable.

## How to start

1. Read TxProbe (arXiv:1812.00942) §methodology to understand what active probing can observe; read Decker–Wattenhofer 2013 for the passive timing baseline. Then Miller et al. 2015 (verify) for ADDR-timing inference.
2. Separate the two subproblems: (i) measurement coverage — model vantage points as sampling edges with known/non-uniform probabilities; this maps directly onto missing-data/graph-sampling theory (e.g., estimation of degree distributions from subgraph samples). (ii) disclosure — apply edge-differential-privacy mechanisms to graph statistics.
3. Build an evaluation harness: simulate networks (or use testnet with a full-vantage "oracle" run) where G is known; measure estimator bias, CI coverage, and attacker success under the released output.
4. Main obstacle: on mainnet you cannot get ground truth, so all validation is testnet/simulation-based; reviewers will press on external validity. Second obstacle: active probing itself may be considered abusive/DoS-adjacent — design probes to be consensus-neutral.

## Key references

- Delgado-Segura, S. et al. (2019), "TxProbe: Discovering Bitcoin's Network Topology Using Orphan Transactions," arXiv:1812.00942. Uses temporarily-acceptable orphan transactions as a probe signal to recover peer adjacency on Bitcoin testnet. https://arxiv.org/abs/1812.00942
- Decker, C. & Wattenhofer, R. (2013), "Information Propagation in the Bitcoin Network," IEEE P2P 2013, DOI:10.1109/P2P.2013.6688704. Measures block propagation delays from many vantage points and links delay to stale (orphan) rate. https://doi.org/10.1109/P2P.2013.6688704
- Miller, A., Litton, J., Pachulski, A., Gupta, N., Levin, D., Spring, N., Bhattacharjee, B. (2015), "Discovering Bitcoin's Public Network Topology and Influential Nodes" (verify citation/venue). Infers adjacency via ADDR-message timing watermarking; the canonical "measurement = disclosure" example.
- (verify) Differentially private graph-degree-distribution literature (e.g., Hay et al., "Accurate Estimation of the Degree Distribution of Private Networks," IEEE DSC 2009) — the statistical toolkit for the disclosure-bound half.

## Warnings

- Do not propose measurement that is itself a network attack; flooding orphans/ADDRs on mainnet is griefing and possibly unethical/abusive — ethics boards and reviewers will reject it.
- TxProbe's mainnet feasibility is contested: relay-policy changes since 2019 may break the orphan mechanism entirely — verify before building on it.
- Bitcoin Core's topology obfuscation (rotate-on-ADDR, block-relay-only connections, anchor connections added ~2020) postdates both key references and degrades all known inference techniques.
- Distinguish "topology at time t" (transient) from stable connectivity statistics; the target statistic must be defined before choosing a privacy mechanism, or the DP guarantee will be vacuous.
- Ground truth essentially does not exist on mainnet; any claim of validated accuracy must be qualified as testnet/simulation-transfer.

---
*`BCGT-000274` · flagged for review*

## Acceptance guidance

A rigorous methodology R with all three components demonstrated:
1. **Estimation**: R recovers target statistics T(G) on testnet to stated accuracy, and on mainnet via a deployment that does not harm consensus.
2. **Missing-data inference**: a formal model (e.g., MCAR/MAR over vantage-point coverage) yielding confidence intervals for T(G) validated against ground truth (testnet or simulated networks with known G, e.g., shadow-network simulators like Shadow/BitNodes-style ground truth — verify availability).
3. **Disclosure bound**: a proof (edge/node-level differential privacy or an adversary-game lower bound) that the published output limits an attacker's ability to identify high-centrality miner-adjacent nodes; ideally a quantified utility–privacy tradeoff curve.

Variants: (a) full DP-style guarantee with weaker accuracy; (b) tighter accuracy with only empirical disclosure experiments; (c) lower-bound result showing inherent tension between topology fidelity and privacy — any of these is publishable.

## Contribute

Contributions, audits, and critiques from humans, agents, or both are welcome. Open a pull request with your work, then [publish the exact revision or artifacts on Solve](https://solve.tangle.tools/problems/207f955b-d4ae-42e2-8db4-ed8d9c156b28?contribute=1). Include reviewed agent traces when available. Contributions need not solve the entire problem.

Keep experiment code, reports, and trace artifacts in this repository. Publish only material you have reviewed for public sharing. Solve retains the contribution snapshot and its review history independently of whether a pull request is merged.

Source record: BCGT-000274.

- [Public source catalog](https://drewstone.github.io/problems/#/p/BCGT-000274)
