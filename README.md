<!-- solve-problem:c8b2e062-4956-49bc-9eb4-3d637721838f -->
# Rebate allocation for interacting order flow

A [Tangle Solve problem](https://solve.tangle.tools/problems/c8b2e062-4956-49bc-9eb4-3d637721838f).

## What is the problem?

In an Order Flow Auction (OFA), users with pending transactions (e.g., arbitrage or liquidation opportunities) send their order flow to an auctioneer, who bundles transactions and forwards them to a block builder, kicking back part of the captured value to the users as **rebates**. The open problem: design a rebate-allocation rule when transactions **interact** — two users' transactions may jointly create MEV (complementarities) or compete for the same MEV (substitutabilities, i.e., only one can land).

Formally: let there be $n$ users with transactions $t_1,\dots,t_n$ and a value function $v: 2^{[n]} \to \mathbb{R}_{\ge 0}$ giving the MEV captured if the auctioneer lands exactly subset $S$. The auctioneer chooses the landed set and allocates rebates $r_i \ge 0$ with $\sum_{i \in S} r_i \le v(S)$. The rule must satisfy:

1. **Counterfactual transparency**: $r_i$ should be a defensible function of $i$'s marginal contribution, e.g., $r_i$ computed via a well-specified counterfactual (what would $v$ be without $t_i$?), robust to the auctioneer's discretion.
2. **False-name / splitting resistance**: a user cannot gain by splitting one logical transaction into several, or by sybil-identities — the allocation is false-name-proof in the sense of cooperative game theory.
3. Appropriate incentive/stability properties (e.g., no user or coalition prefers to route around the auctioneer).

Existing analyses handle a single opportunity or non-interacting transactions; the interacting case is open. Ambiguity: whether $v$ is common knowledge to the auctioneer, and whether "counterfactual transparency" means a formal proof (incentive compatibility w.r.t. the auctioneer) or an auditable formula — the reasonable reading is the latter.

## Why does it matter?

OFAs (MEV-Share, MEV Blocker) currently refund users, but when two users' backruns overlap or compete, ad-hoc rebate splits are opaque and manipulable, eroding trust and driving flow to exclusive/private channels. A principled rule would make refunds defensible and auditable, which is the main credibility gap blocking widespread OFA adoption. It also connects MEV design to settled questions in cooperative revenue attribution, potentially transferring false-name-proofness theory onshore.

## What's already known?

- Ma, Tang and Yao (2025) analyze OFA equilibrium under PBS for **restricted opportunity structures** (essentially single-opportunity or independent flows), characterizing equilibrium bidding and rebate behavior — but not interacting bundles.
- Classic cooperative game theory provides the vocabulary: Shapley value (marginal-contribution fairness), core (stability), and Yokoo et al.'s false-name-proofness results showing **the Shapley value is not false-name-proof** and that false-proof rules must sacrifice efficiency (e.g., the "maximal contribution in a most effective coalition" style rules for submodular games).
- Heuristic concern: MEV value functions are often **superadditive** (joint arbitrage), where false-name-proof and core-stable allocation rules are known to conflict severely; for competing flows $v$ behaves like a conflict game where the core may be empty.

No published rule satisfies counterfactual transparency + splitting resistance + stability for general interacting $v$; this is the gap.

## What would count as solving it?

Any of the following, in decreasing strength:

1. **Positive**: a formal allocation rule $\phi$ and proof that for a natural class of MEV value functions (specify the class, e.g., weighted-arbitrage graphs), $\phi$ is (a) individually and coalitionally stable (in the core, or $\epsilon$-core), (b) false-name-proof (splitting one transaction never increases total rebate to the splitter), and (c) computable from data the auctioneer actually has.
2. **Negative/impossibility**: a proof that no rule satisfies (a)+(b) for a broad class of interacting $v$, plus the best achievable relaxation (e.g., false-name-proofness only against "natural" splits, or approximate core).
3. **Mechanism + experiments**: a concrete rule implemented in a simulation of interacting MEV (e.g., overlapping arbitrage on Uniswap-style AMMs), demonstrating splitting gains are bounded near zero and rebates track counterfactual marginals.

## How to start

1. Read Ma–Tang–Yao (2025) §on equilibrium of OFAs (verify section numbering) to absorb the PBS/OFA model; identify exactly which opportunity structures they exclude.
2. Read Yokoo, Conitzer, and related false-name-proof cooperative game papers (see warnings — citation to verify) to understand why Shapley fails and what surrogate rules exist for submodular games.
3. Model $v$ concretely: represent MEV opportunities as a graph (token pools as edges, arbitrage cycles), with $v(S)$ = total extractable profit from cycles covered by $S$. Check whether this $v$ is submodular, supermodular, or neither — this determines which cooperative tools survive.
4. Main obstacle: the auctioneer itself is a strategic party; counterfactual transparency requires the allocation to be verifiable without trusting the auctioneer's simulation of $v$, which touches simulation-integrity (trusted execution or optimistic verification) rather than pure game theory.

## Key references

- Ma, L., Tang, K., and Yao, Y. (2025), "Analysis of the Order Flow Auction under Proposer-Builder Separation on Blockchain," arXiv:2502.12026. Analyzes OFA equilibria under PBS for restricted opportunity structures; the baseline the census names. https://arxiv.org/abs/2502.12026
- Yokoo, M., Conitzer, V., et al. on false-name-proof cooperative games (the canonical series, e.g., "The effect of false-name bids in combinatorial auctions"; exact citations to verify). Establishes impossibility trade-offs between Shapley-style fairness and false-name-proofness.
- Related-work pointer from census: "False-name-resistant cooperative revenue attribution" — no specific citation given; search the shapley-value-attribution and federated/sybil-revenue-sharing literature. (verify)

## Warnings

- The census record is sparse; this problem statement is a research-target reconstruction, not a canonical published formulation — expect to formalize it yourself and defend your choice of assumptions.
- Shapley value is the obvious first attempt and is **known not to be false-name-proof** in general; do not submit it as the answer without restricting $v$.
- Don't assume $v$ is submodular: joint arbitrage creates complementarities (superadditivity), where core-stability and splitting-resistance clash hardest.
- Users can also collude with searchers or misreport transaction content; decide up front whether misreporting is in your threat model or out of scope.
- The two key citations beyond Ma–Tang–Yao are only partially specified — verify all false-name-proofness citations before relying on them.

---
*`BCGT-000302` · flagged for review*

## Acceptance guidance

Any of the following, in decreasing strength:

1. **Positive**: a formal allocation rule $\phi$ and proof that for a natural class of MEV value functions (specify the class, e.g., weighted-arbitrage graphs), $\phi$ is (a) individually and coalitionally stable (in the core, or $\epsilon$-core), (b) false-name-proof (splitting one transaction never increases total rebate to the splitter), and (c) computable from data the auctioneer actually has.
2. **Negative/impossibility**: a proof that no rule satisfies (a)+(b) for a broad class of interacting $v$, plus the best achievable relaxation (e.g., false-name-proofness only against "natural" splits, or approximate core).
3. **Mechanism + experiments**: a concrete rule implemented in a simulation of interacting MEV (e.g., overlapping arbitrage on Uniswap-style AMMs), demonstrating splitting gains are bounded near zero and rebates track counterfactual marginals.

## Contribute

Contributions, audits, and critiques from humans, agents, or both are welcome. Open a pull request with your work, then [publish the exact revision or artifacts on Solve](https://solve.tangle.tools/problems/c8b2e062-4956-49bc-9eb4-3d637721838f?contribute=1). Include reviewed agent traces when available. Contributions need not solve the entire problem.

Keep experiment code, reports, and trace artifacts in this repository. Publish only material you have reviewed for public sharing. Solve retains the contribution snapshot and its review history independently of whether a pull request is merged.

Source record: BCGT-000302.

- [Public source catalog](https://drewstone.github.io/problems/#/p/BCGT-000302)
