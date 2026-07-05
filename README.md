# Compiler-Level-Bug-Curve-Finance-Vyper-2023-70M-
What happened: Malfunctioning reentrancy guards — caused by a compiler bug, not a code mistake — allowed classic reentrancy attacks on several Curve pools using specific Vyper versions, draining tens of millions across multiple protocols simultaneously.
fix(deps): pin Vyper compiler version, re-verify reentrancy locks on
all pools compiled with affected versions (0.2.15 / 0.2.16 / 0.3.0)

Ref: Curve Finance exploit (Jul 2023, $70M+ across multiple pools)
Root cause: a bug in specific Vyper compiler versions caused
reentrancy locks to malfunction on certain pools, even though the
Solidity-equivalent logic looked correct — the vulnerability lived
in the compiler, not the contract source.
