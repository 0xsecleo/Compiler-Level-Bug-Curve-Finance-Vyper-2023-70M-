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
