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
