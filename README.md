# Compiler-Level-Bug-Curve-Finance-Vyper-2023-70M-
What happened: Malfunctioning reentrancy guards — caused by a compiler bug, not a code mistake — allowed classic reentrancy attacks on several Curve pools using specific Vyper versions, draining tens of millions across multiple protocols simultaneously.
fix(deps): pin Vyper compiler version, re-verify reentrancy locks on
all pools compiled with affected versions (0.2.15 / 0.2.16 / 0.3.0)

Ref: Curve Finance exploit (Jul 2023, $70M+ across multiple pools)
Root cause: a bug in specific Vyper compiler versions caused
reentrancy locks to malfunction on certain pools, even though the
Solidity-equivalent logic looked correct — the vulnerability lived
in the compiler, not the contract source.
Root cause: The vulnerability existed in the compiler toolchain itself. Contract source code was correct; the compiled bytecode was not.
Fix: Track compiler CVEs and advisories continuously, not just at deployment time. Pin and audit your exact compiler version. Re-verify guards with independent bytecode-level analysis, not just source review.
Base takeaway: "My code is correct" isn't the whole story — your toolchain has to be correct too. Subscribe to Solidity AND Vyper security advisories if you use either, and treat compiler upgrades/pins as a security-relevant decision, not a devops afterthought.
Concentrated Liquidity Math Exploit — KyberSwap (2023, $54M)
fix(amm): correct tick-crossing liquidity delta calculation, add
invariant check post-swap to detect impossible liquidity states

Ref: KyberSwap Elastic exploit (Nov 2023, $54M)
Root cause: a subtle rounding/math error in how liquidity was
tracked when swaps crossed multiple price ticks let the attacker
construct a sequence of swaps that made the pool believe it held far
more liquidity than it actually did, enabling massive over-withdrawal.
Fix: Extensive fuzz testing and formal verification specifically targeting edge cases in tick-crossing / liquidity math, not just "normal" swap scenarios. Add post-swap invariant checks (e.g., total liquidity can never exceed sum of deposits).
Base takeaway: Complex AMM math (concentrated liquidity especially) is one of the hardest things to audit by reading code alone. If you're deploying a CL-style AMM on Base, budget for fuzzing/formal verification specifically on the math library — not just a manual line-by-line review.
Price Manipulation via Recursive Lending — Harvest Finance (2020, $24M)
fix(vault): replace instantaneous pool price with TWAP for
share-price calculation in deposit/withdraw math

Ref: Harvest Finance exploit (Oct 2020, $24M)
Root cause: vault share price was calculated directly from a
Curve pool's instantaneous balance ratio, which the attacker
swung sharply using large flash-loan-funded swaps, then deposited
and withdrew at manipulated exchange rates for profit.
What happened: The attacker used flash loans to violently swing a Curve pool's balance ratio, deposited into Harvest's vault while the share price was artificially cheap, swung the pool back, then withdrew at a fair price — pocketing the difference. Repeated in a tight loop for ~$24M.
Root cause: Vault share price was derived directly from a spot balance ratio that could be moved within a single transaction.
Fix: Never price vault shares off an instantaneous on-chain ratio. Use TWAP, or better, price based on a value that can't be moved atomically within the attacker's own transaction.
Base takeaway: This is the same root lesson as bZx, just applied to vault accounting instead of collateral. Any "price per share" calculation in a Base vault, staking contract, or yield aggregator needs the same TWAP scrutiny as a lending oracle.
Cross-Protocol Collateral Exploit — Alpha Homora / Iron Bank (2021, $37.5M)
fix(integration): validate collateral value independently per
protocol, remove implicit trust in composed leverage positions

Ref: Alpha Homora v2 / Cream's Iron Bank exploit (Feb 2021, $37.5M)
Root cause: attacker chained multiple protocols (Alpha Homora,
Iron Bank, Curve, SushiSwap) together in a single transaction to
create a leveraged position whose true collateral value was never
independently validated at each hop of the composition.
Fix: Any protocol integrating with external money-legos should independently validate the economic reality of composed positions, not just trust upstream protocol outputs at face value.
Base takeaway: Base's whole appeal is fast, cheap composability — which means this exact class of cross-protocol exploit is more likely here, not less. If you integrate with another Base protocol's LP tokens, vault shares, or leveraged positions, validate independently. Don't just trust the interface.
X Post:
Recurring Oracle Manipulation — Inverse Finance (2022, $15.6M across two incidents)
fix(oracle): replace low-liquidity Curve pool price feed with
Chainlink + sanity-bound fallback for INV token pricing

Ref: Inverse Finance exploits (Apr 2022 $15.6M, Jun 2022 $1.2M)
Root cause: the protocol's own governance token (INV) was priced
using a thin, low-liquidity on-chain pool as the oracle source —
manipulable with a modest amount of capital, and used TWICE within
two months.
What happened: Twice in two months, attackers manipulated the price of INV (Inverse's own governance token) on a thin liquidity pool, then used the inflated price to borrow far more than the real collateral was worth.
Root cause: Pricing a native/governance token from its own thin liquidity pool — a textbook self-referential oracle risk — and not fixing it thoroughly after the first incident.
Fix: Native governance tokens should never be their own primary collateral price source. Use external, deep-liquidity oracles (Chainlink) with sanity bounds, and audit ALL similar tokens after any single oracle incident — not just the one that got hit.
Base takeaway: The second Inverse hack is the real lesson: getting hacked once and only patching the exact exploited path — without auditing every structurally similar risk — invites a repeat. If one of your price feeds gets manipulated, assume every similarly-designed feed is vulnerable too.
