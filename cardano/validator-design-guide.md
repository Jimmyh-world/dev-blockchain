# Validator Design Guide - UTxO Model Design-First Approach

**Created**: 2025-10-02
**Version**: 1.0
**Adapted From**: Solidity Design Guide → UTxO/Functional Paradigm
**Purpose**: Design-first validator development for Cardano/Aiken

---

## Overview

Design validators BEFORE writing code. The UTxO model requires different thinking than account-based models.

**Key Principle**: Think in terms of UTxO transformations, not state mutations.

---

## Phase 1: Discovery Questions (UTxO-Focused)

### A. Validator Purpose

**Ask:**
1. **What UTxO locking logic does this validator enforce?**
   - Single sentence answer
   - Example: "Locks funds until owner signature + deadline"

2. **Who can unlock these UTxOs and under what conditions?**
   - List actors (owner, beneficiary, etc.)
   - List conditions (signatures, time, state)

3. **What state lives in the datum?**
   - What data must be stored on-chain?
   - What can stay off-chain?

4. **What actions should the redeemer support?**
   - List possible actions (Unlock, Cancel, etc.)
   - What proof needed for each?

### B. Security Requirements

**Ask:**
1. **What value does this validator protect?**
   - ADA amount
   - Specific tokens/NFTs
   - Protocol state

2. **What are the attack vectors?**
   - Check all 5 critical vulnerabilities:
     - Double satisfaction?
     - UTxO authentication needed?
     - Multiple token types?
     - Datum could grow unbounded?
     - Staking credentials matter?

3. **What signatures/credentials are required?**
   - Owner signature?
   - Multi-sig threshold?
   - Role-based access?

4. **What time constraints apply?**
   - Deadlines?
   - Time windows?
   - Validity range requirements?

### C. State Design (UTxO-Specific)

**Ask:**
1. **What data MUST be in the datum?**
   - Owner/beneficiary addresses
   - Amounts
   - Deadlines
   - State machine state

2. **How does state transition?**
   - Draw state machine diagram
   - What triggers each transition?

3. **What invariants must hold?**
   - Value preservation
   - State progression (no backwards)
   - Signature requirements

4. **How does off-chain track UTxOs?**
   - Query by NFT?
   - Query by address?
   - Index datum fields?

---

## Phase 2: Type Design

### Document in `docs/design/validator-specification.md`

```markdown
# Validator: [Name]

## Purpose
[One sentence from discovery]

## Datum Type Design

\`\`\`aiken
type MyDatum {
  field1: Type  // Purpose: [why needed]
                // Invariant: [what must be true]
                // Size: [estimate bytes]
  field2: Type  // Purpose: [...]
}
\`\`\`

**Total Estimated Size**: [X bytes] (limit: 16KB)

## Redeemer Type Design

\`\`\`aiken
type MyRedeemer {
  action: Action,  // Possible: [list all actions]
  proof: ByteArray,  // Validation: [what this proves]
}

type Action {
  Action1 { param: Type }
  Action2
  Action3 { param1: Type, param2: Type }
}
\`\`\`

## State Machine

\`\`\`
[StateA] --[Action1 + Signature]--> [StateB]
[StateA] --[Action2 + Time > Deadline]--> [StateC]
[StateB] --[Action3]--> [StateD]
\`\`\`

**Invariants:**
- Can only progress forward (no backwards transitions)
- Terminal states (Completed, Cancelled) are final
- Each transition requires specific proofs

## Security Analysis

### Vulnerabilities Considered

1. ✅ **Double Satisfaction**: [Mitigation strategy]
2. ✅ **UTxO Authentication**: [NFT/token authentication plan]
3. ✅ **Token Validation**: [Whitelist/validation approach]
4. ✅ **Unbounded Datum**: [Size limits enforced]
5. ✅ **Staking Control**: [Staking credential requirements]

### Attack Scenarios

**Scenario 1**: [Describe attack]
- **Mitigation**: [How design prevents it]

**Scenario 2**: [Describe attack]
- **Mitigation**: [How design prevents it]
```

---

## Phase 3: Off-Chain Design

### Transaction Building Plan

**Document expected transaction structure:**

```markdown
## Off-Chain Integration

### Transaction Structure for [Action]

**Inputs:**
- UTxO at script address
- Datum: [structure]
- Value: [amount + tokens]

**Outputs:**
- Output 1: [where, how much, datum if continuation]
- Output 2: [...]

**Requirements:**
- Extra Signatories: [list required signatures]
- Validity Range: {lower: [bound], upper: [bound]}
- Mint: [if any tokens minted]
- Redeemer: [structure]

**Example (Lucid):**
\`\`\`typescript
const tx = await lucid
  .newTx()
  .collectFrom([utxo], redeemer)
  .pay.ToAddress(beneficiary, {lovelace: amount})
  .addSigner(await lucid.wallet().address())
  .validFrom(Date.now())
  .validTo(deadline)
  .complete()
\`\`\`
```

---

## Phase 4: Implementation Plan

### Step-by-Step Implementation (Using Jimmy's Workflow)

**Step 1: Define Types**

🔴 **IMPLEMENT**:
- Create `lib/[validator]/types.ak`
- Define Datum type
- Define Redeemer type
- Define State/Action types

🟢 **VALIDATE**:
```bash
aiken check  # Types compile
```

🔵 **CHECKPOINT**: Types defined

---

**Step 2: Implement Validation Helpers**

🔴 **IMPLEMENT**:
- Create `lib/[validator]/validation.ak`
- Implement pure validation functions
- Add security checks (5 vulnerabilities)
- Add helper functions

🟢 **VALIDATE**:
```bash
aiken check  # Compiles
```

🔵 **CHECKPOINT**: Helpers implemented

---

**Step 3: Write Tests FIRST (TDD)**

🔴 **IMPLEMENT**:
- Write test for each action
- Write test for each failure case
- Write property tests for invariants

🟢 **VALIDATE**:
```bash
aiken check  # Tests compile (will fail - not implemented)
```

🔵 **CHECKPOINT**: Tests written (RED phase)

---

**Step 4: Implement Validator**

🔴 **IMPLEMENT**:
- Create `validators/[name].ak`
- Implement validator using helpers
- Follow Parse-Validate-Succeed pattern

🟢 **VALIDATE**:
```bash
aiken check  # Compiles
aiken test   # All tests pass (GREEN phase)
aiken build  # Builds successfully
```

🔵 **CHECKPOINT**: Validator implemented

---

**Step 5: Security Audit**

🔴 **IMPLEMENT**:
- Review against 5 critical vulnerabilities
- Add security tests if gaps found
- Implement additional mitigations

🟢 **VALIDATE**:
- Security checklist: All 5 vulnerabilities addressed
- Security tests: All passing
- Manual review: No anti-patterns

🔵 **CHECKPOINT**: Security audit complete

---

## Design Patterns Catalog

### Pattern 1: Simple Locking (Time + Signature)

```
Purpose: Lock funds until time + owner signature

Datum: { owner, deadline, amount }
Redeemer: { Unlock | Cancel }
State: Locked → Unlocked
Security: Owner sig + before deadline
```

### Pattern 2: Escrow (Multi-Party)

```
Purpose: Mediated exchange between parties

Datum: { seller, buyer, arbiter, price }
Redeemer: { Complete | Dispute | Cancel }
State: Active → Completed/Disputed/Cancelled
Security: Party signatures + value preservation
```

### Pattern 3: Vesting (Progressive Unlock)

```
Purpose: Release funds gradually over time

Datum: { beneficiary, schedule: List<(Time, Amount)> }
Redeemer: { Claim { period_index } }
State: Vesting → PartiallyVested → FullyVested
Security: Time validation + partial value release
```

### Pattern 4: DAO Voting (Governance)

```
Purpose: Weighted voting with token governance

Datum: { proposal, votes: List<Vote>, deadline }
Redeemer: { Vote { choice, token_proof } | Execute }
State: Voting → Passed/Failed
Security: Token-weighted votes + double-vote prevention
```

---

## Integration with Jimmy's Workflow

**Complete Design Workflow**:

🔴 **RED Phase 1: Discovery** → Answer all discovery questions
🟢 **GREEN Phase 1: Validate** → Review answers with KISS principle
🔵 **CHECKPOINT 1**: Requirements clear

🔴 **RED Phase 2: Type Design** → Design datum/redeemer/state types
🟢 **GREEN Phase 2: Validate** → Review against security checklist
🔵 **CHECKPOINT 2**: Types designed

🔴 **RED Phase 3: Security Analysis** → Map attack vectors, plan mitigations
🟢 **GREEN Phase 3: Validate** → Verify all 5 vulnerabilities addressed
🔵 **CHECKPOINT 3**: Security designed

🔴 **RED Phase 4: Implementation** → Write code following design
🟢 **GREEN Phase 4: Validate** → Tests pass, security checks complete
🔵 **CHECKPOINT 4**: Validator complete

---

**Design first, implement second. The UTxO model rewards careful planning.**

**Version**: 1.0
**Last Updated**: 2025-10-02
**Maintained By**: Jimmy + AI Coding Assistants

---

## Related Documents

- **Before Design**: `ethereum-cardano-comparison.md` - Understand UTxO model differences
- **During Design**: `cardano-security-patterns.md` - Security analysis checklist
- **After Design**: `aiken-development-rules.md` - Implementation patterns
- **Off-Chain Planning**: `cardano-offchain-integration.md` - Transaction building considerations
- **Complete Workflow**: `cardano-ai-rules.md` - AI assistant behavior during design
- **Overview**: `AGENTS.md` - Development workflow and principles
