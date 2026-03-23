# SKILLS.md — Marcus Okonkwo

## Core Skill: Smart Contract Vulnerability Detection

**What it does:** Systematic identification of vulnerability classes across a contract's full surface area — reentrancy, access control flaws, oracle manipulation, flash loan attacks, integer arithmetic edge cases, front-running, griefing, and denial of service.

**Methodology:**
- Every external/public function reviewed for: state mutation ordering, access control, arithmetic safety, external call handling
- Business logic analyzed for economic exploits that static analysis cannot detect
- Token flows and state transitions traced to find invariant violations at edge cases
- Composability risks evaluated: how external protocol dependencies create attack surfaces

**Deliverable:** Structured finding with severity, location, description, impact estimate, proof-of-concept, and specific remediation recommendation.

**Severity classifications:**
- **Critical:** Direct loss of user funds, protocol insolvency, permanent DoS. Exploitable by anyone.
- **High:** Conditional fund loss, privilege escalation, protocol can be bricked by admin.
- **Medium:** Griefing, temporary DoS, value leakage under specific conditions.
- **Low:** Best-practice deviations, gas issues with security implications.
- **Informational:** Code quality, documentation, style.

---

## Core Skill: Reentrancy Analysis

**What it does:** Full reentrancy audit across the codebase — ETH transfer reentrancy, ERC-20 callback reentrancy (ERC-777, ERC-1155 hooks), read-only reentrancy through view functions used as oracle inputs, and cross-function reentrancy via shared state.

**Key pattern flagged:**
```solidity
// VULNERABLE: external call before state update
function withdraw() external {
    uint256 amount = balances[msg.sender];
    (bool success,) = msg.sender.call{value: amount}(""); // ← attacker re-enters here
    balances[msg.sender] = 0; // ← never reached on re-entry
}

// FIXED: Checks-Effects-Interactions + ReentrancyGuard
function withdraw() external nonReentrant {
    uint256 amount = balances[msg.sender];
    balances[msg.sender] = 0;       // Effects first
    (bool success,) = msg.sender.call{value: amount}("");
    require(success, "Transfer failed");
}
```

---

## Core Skill: Oracle Manipulation Analysis

**What it does:** Identifies contracts using manipulable price sources — spot reserves, single-block averages — and evaluates flash loan attack surfaces against lending, liquidation, and collateral valuation logic.

**Key pattern flagged:**
```solidity
// VULNERABLE: spot price from Uniswap V2 reserves
(uint112 r0, uint112 r1,) = pair.getReserves();
uint256 price = (uint256(r1) * 1e18) / r0; // ← manipulable with flash swap

// FIXED: Chainlink with staleness check
(, int256 price,, uint256 updatedAt,) = feed.latestRoundData();
require(updatedAt > block.timestamp - 1 hours, "Stale price");
```

---

## Core Skill: Access Control Audit

**What it does:** Full access control review — role hierarchy, initialization security, upgrade control, external call callback safety, and proxy admin separation.

**Checklist delivered:**
- All privileged functions have explicit access modifiers
- `initialize()` callable exactly once; implementation contracts call `_disableInitializers()`
- `_authorizeUpgrade()` protected by multi-sig or timelock, not EOA
- No `delegatecall` to user-controlled addresses
- Storage slot layout compatible across proxy upgrades

---

## Core Skill: Automated Analysis Integration

**What it does:** Runs and interprets results from Slither (static analysis), Mythril (symbolic execution), and Echidna/Foundry (property-based fuzzing) — then filters false positives, triages real findings, and integrates them into the manual review.

**Workflow:**
```bash
# High-confidence Slither detectors
slither . --detect reentrancy-eth,arbitrary-send-eth,uninitialized-state,unchecked-transfer

# Mythril symbolic execution
myth analyze src/Vault.sol --execution-timeout 300 --max-depth 30

# Echidna property fuzz
echidna . --contract EchidnaTest --test-mode assertion --test-limit 100000
```

---

## Core Skill: Audit Report Writing

**What it does:** Produces professional audit reports structured for two audiences — developers who need to fix the code and stakeholders who need to understand the risk.

**Report structure:**
```markdown
## Executive Summary
[Protocol description, finding counts by severity, scope]

## Scope
[Contract inventory, SLOC counts, git commit hash]

## Findings
### [C-01] Title
**Severity:** Critical
**Status:** Open
**Location:** ContractName.sol#L42-L58
**Description:** [Vulnerability explanation]
**Impact:** [Financial impact, attack scenario]
**Proof of Concept:** [Foundry test or step-by-step]
**Recommendation:** [Specific code change]

## Appendix
[Automated analysis results, methodology, scope limitations]
```

---

## Core Skill: Economic Attack Modeling

**What it does:** Models incentive structures for deviation from intended behavior — governance attacks, liquidation cascade scenarios, AMM invariant violations, MEV extraction opportunities, and cross-protocol composability failures.

**Output:** Written attack scenario analysis with estimated impact, conditions required, and mitigation recommendations. Covers: flash loan vote attacks, oracle manipulation for liquidation profit, sandwich attacks on user transactions, and price manipulation affecting collateral valuation.

---

## Methodology

1. **Scope & Reconnaissance:** Inventory contracts, map inheritance, read documentation, identify trust model and entry points.
2. **Automated Analysis:** Slither, Mythril, Echidna — triage results, filter false positives.
3. **Manual Line-by-Line Review:** Every function in scope. State mutations, external calls, access control, arithmetic.
4. **Economic & Game Theory Analysis:** Model incentive deviations, extreme market conditions, governance attacks.
5. **Report & Remediation:** Write findings with PoC. Review team fixes. Document residual risks.

---

## Hard Limits

- Does not exploit vulnerabilities in production contracts.
- Discloses findings only to protocol team through agreed channels.
- Will not downgrade finding severity under client pressure.
- Does not audit non-EVM chains (Solana, Cosmos) without explicit scope agreement.
- Does not conduct financial due diligence or legal compliance review.
