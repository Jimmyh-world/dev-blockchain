# Cardano/Aiken Development Rules - Template Collection

**Version**: 1.0
**Last Updated**: 2025-10-02
**Status**: VERIFIED - Based on professional audits and official documentation
**Purpose**: Reusable AI-optimized rules for Cardano smart contract development

---

## Overview

This collection provides verified, production-ready development rules for Cardano/Aiken smart contracts. All patterns are based on:
- ✅ Official Aiken v1.1.17 documentation
- ✅ Professional security audits (Vacuumlabs, MLabs)
- ✅ CIP-52 (Cardano Audit Best Practice Guidelines)
- ✅ Production deployments (SundaeSwap, Minswap, JPG Store, Lenfi)

---

## Rule Files

### 1. aiken-development-rules.md (~34KB)

**Purpose**: Comprehensive Aiken development patterns adapted from Solidity best practices

**Contents**:
- Verified v1.1.17 syntax and patterns
- Complete validator templates
- Parse-Validate-Succeed pattern
- Performance optimization
- Testing standards
- Integration with Jimmy's Workflow

**Use When**: Writing any Aiken validator

**Verification**: All code examples tested with Aiken v1.1.17

---

### 2. cardano-security-patterns.md (~23KB)

**Purpose**: UTxO-specific security vulnerabilities and verified mitigation patterns

**Contents**:
- 5 Critical Verified Vulnerabilities:
  1. Double/Multiple Satisfaction (CRITICAL)
  2. Missing UTxO Authentication (HIGH)
  3. Incomplete Token Validation (HIGH)
  4. Unbounded Datum/Value (HIGH)
  5. Insufficient Staking Control (MEDIUM)
- Verified mitigation patterns for each
- Attack scenarios with examples
- Security checklist

**Use When**: Security audit, validator design, code review

**Verification**: All vulnerabilities confirmed by professional auditors

---

### 3. validator-design-guide.md (~8KB)

**Purpose**: Design-first approach for validator development (UTxO model)

**Contents**:
- Discovery questions (UTxO-focused)
- Type design methodology
- State machine design
- Security analysis framework
- Off-chain integration planning
- Implementation workflow with Jimmy's Workflow

**Use When**: Starting new validator project

**Verification**: Adapted from Solidity design guide + UTxO model principles

---

### 4. ethereum-cardano-comparison.md (~9KB)

**Purpose**: Help Ethereum developers transition to Cardano

**Contents**:
- Verified comparison table
- Architectural differences
- Pattern translation guide (Solidity → Aiken)
- Security model comparison
- Fee model comparison
- When to use each chain

**Use When**: Learning Cardano from Ethereum background

**Verification**: Cross-referenced with official documentation from both chains

---

### 5. cardano-offchain-integration.md (~35KB)

**Purpose**: Complete guide for off-chain development with Aiken validators

**Contents**:
- Technology selection (Lucid Evolution vs Mesh, Blockfrost vs Maestro)
- Setup & configuration with security best practices
- Transaction building patterns
- Aiken validator integration (loading, locking, spending)
- CIP-25 NFT minting and CIP-68 token implementation
- Performance optimization strategies
- Recommended tech stack combinations
- Comprehensive troubleshooting

**Use When**: Building off-chain code, integrating wallets, querying chain data

**Verification**: Based on January 2025 research of official documentation and production usage

---

### 6. atlas-framework-guide.md (~45KB)

**Purpose**: Complete guide for Atlas Haskell Plutus Application Backend

**Contents**:
- When to use Atlas vs Lucid/Mesh (decision matrix)
- Core concepts (GYTxSkeleton, monads, providers, testing)
- Setup & installation (GHC, Cabal, Nix)
- Transaction building patterns in Haskell
- Testing framework (CLB emulator + private testnet)
- Data providers (Maestro, Blockfrost, Kupo, DB Sync)
- Integration with Plutus validators
- Production deployment considerations
- Common patterns and best practices

**Use When**: Building complex DeFi with Haskell, type-safe backend development

**Verification**: Based on Atlas v0.14.1, official docs, and production usage (Genius Yield DEX)

---

### 7. JIMMYS-WORKFLOW.md (~47KB)

**Purpose**: Complete Red/Green Checkpoint validation system (reference copy)

**Contents**:
- Core philosophy (ASSUME NOTHING, EXPLICIT OVER IMPLICIT, FAIL FAST)
- Color-coded phases (🔴 RED: IMPLEMENT → 🟢 GREEN: VALIDATE → 🔵 CHECKPOINT)
- Machine-readable checkpoint format (JSON)
- Enhanced validation templates (frontend, backend, database)
- Rollback procedures (git, file-based, migration, configuration)
- Common workflow patterns (single feature, multi-step, parallel, dependent)
- Autonomous execution mode for AI assistants
- Failure recovery patterns
- Integration with TDD and TodoWrite

**Use When**: Every implementation task - prevents AI hallucination through mandatory validation

**Verification**: Platform-wide standard integrated throughout all Cardano documents

---

### 8. AGENTS.md (~13KB)

**Purpose**: AI assistant guidelines for Cardano template directory

**Contents**:
- Quick Decision Guide (task → document mapping)
- Common Questions & Quick Answers
- Technology stack overview (Aiken, Lucid, Mesh, Atlas, Blockfrost, Maestro)
- Jimmy's Workflow integration for Cardano development
- Security best practices checklist
- Common patterns and examples
- Build & test commands (Aiken, TypeScript, Haskell)
- Environment setup

**Use When**: Configuring AI assistants for Cardano development

**Verification**: Follows agents.md standard with Cardano-specific adaptations

---

## Quick Start

### For New Cardano Project

```bash
# Copy all rules to your project
cp ~/templates\ and\ prompts/cardano/*.md /path/to/project/docs/

# Or reference from centralized location (recommended)
# Create .ai-rules/ directory and copy there
```

### For Learning

**Recommended Reading Order**:
1. `AGENTS.md` - Overview of technology stack and workflow
2. `JIMMYS-WORKFLOW.md` - Understand RED/GREEN/CHECKPOINT validation system
3. `ethereum-cardano-comparison.md` - Understand model differences
4. `validator-design-guide.md` - Learn design-first approach
5. `cardano-security-patterns.md` - Master security vulnerabilities
6. `aiken-development-rules.md` - Reference during validator development
7. `cardano-offchain-integration.md` - Build transactions with Lucid/Mesh
8. `atlas-framework-guide.md` - (Optional) Haskell backend for complex DeFi

### For Aiken Reference Guide Integration

**See**: Parent directory `TEMPLATE-COLLECTION-README.md` for integration with Aiken projects

---

## Rule File Details

| File | Size | Topics | Updated |
|------|------|--------|---------|
| **aiken-development-rules.md** | ~34KB | Syntax, patterns, testing, performance | 2025-10-02 |
| **cardano-security-patterns.md** | ~23KB | 5 critical vulnerabilities, mitigations | 2025-10-02 |
| **validator-design-guide.md** | ~8KB | Design methodology, discovery questions | 2025-10-02 |
| **ethereum-cardano-comparison.md** | ~9KB | Model comparison, pattern translation | 2025-10-02 |
| **cardano-offchain-integration.md** | ~35KB | Off-chain libraries, transaction building, wallets | 2025-10-02 |
| **atlas-framework-guide.md** | ~45KB | Haskell PAB, testing, Plutus integration | 2025-10-02 |
| **JIMMYS-WORKFLOW.md** | ~47KB | RED/GREEN checkpoint system, validation gates | 2025-10-02 |
| **AGENTS.md** | ~13KB | AI assistant guidelines, tech stack, workflow | 2025-10-02 |
| **supporting document.md** | ~22KB | Verification sources, accuracy checks | 2025-10-02 |

**Total**: ~235KB of verified, production-ready Cardano/Aiken knowledge

---

## Verification Sources

**Official**:
- Aiken: https://aiken-lang.org
- Cardano: https://docs.cardano.org
- Cardano Developer Portal: https://developers.cardano.org
- CIP Standards: https://cips.cardano.org/

**Off-Chain Libraries**:
- Lucid Evolution: https://anastasia-labs.github.io/lucid-evolution/
- Mesh SDK: https://meshjs.dev/
- Atlas Framework: https://atlas-app.io/ (Haddock: https://haddock.atlas-app.io)
- Genius Yield: https://www.geniusyield.co/ (Atlas developers)
- Blockfrost: https://docs.blockfrost.io/
- Maestro: https://docs.gomaestro.org/

**Security**:
- CIP-52: Cardano Audit Best Practice Guidelines
- Vacuumlabs: https://vacuumlabs.com/blog
- MLabs: Common Plutus Security Vulnerabilities
- Plutonomicon: https://plutonomicon.github.io

**Research**:
- "The Extended UTXO Model" (Chakravarty et al., 2020)
- IOG Research: https://iohk.io/en/research

**Community**:
- Awesome Aiken: https://github.com/aiken-lang/awesome-aiken
- Aiken GitHub: https://github.com/aiken-lang/aiken
- Cardano Stack Exchange: https://cardano.stackexchange.com/

---

## Integration

### With AGENTS.md Template

Reference Cardano rules in project AGENTS.md:

```markdown
## Technology-Specific Guidelines

**Cardano/Aiken Development**:
- See `.ai-rules/AGENTS.md` for Cardano development overview
- See `.ai-rules/aiken-development-rules.md` for validator patterns
- See `.ai-rules/cardano-security-patterns.md` for security vulnerabilities
- See `.ai-rules/validator-design-guide.md` for design approach
- See `.ai-rules/cardano-offchain-integration.md` for transaction building (Lucid/Mesh)
- See `.ai-rules/atlas-framework-guide.md` for Haskell backend (Atlas PAB)
```

### With Jimmy's Workflow

Cardano rules integrate with Jimmy's Workflow (see `JIMMYS-WORKFLOW.md`):
- 🔴 RED: Use design guide and development rules
- 🟢 GREEN: Use security checklist for validation
- 🔵 CHECKPOINT: Verify all 5 vulnerabilities addressed

**Complete System**:
- Workflow methodology: `JIMMYS-WORKFLOW.md`
- Validator development: Applied in `aiken-development-rules.md`
- Security validation: Integrated with `cardano-security-patterns.md`
- Design process: Four-phase design in `validator-design-guide.md`

---

## Maintenance

**Update Triggers**:
- New Aiken version released (check syntax changes)
- New security vulnerability discovered (add to security-patterns.md)
- New production pattern identified (add to development-rules.md)
- Community feedback on patterns

**Version History**:
- v1.0 (2025-10-02): Initial verified release

---

**These rules represent the current best practices for Cardano/Aiken development as of October 2025, verified against authoritative sources and production deployments.**

**Template Collection**: See `../TEMPLATE-COLLECTION-README.md`
**Version**: 1.0 (Verified)
**Maintained By**: Jimmy + AI Coding Assistants
