# Cardano Smart Contract Development Template

**STATUS: READY FOR PRODUCTION USE** - Last Updated: 2025-10-02

## Repository Information
- **Local Directory**: `/home/jimmyb/templates/cardano`
- **Primary Purpose**: Comprehensive templates and guides for Cardano/Aiken smart contract development

## Important Context

This template collection provides production-ready guidelines for developing secure Cardano smart contracts using Aiken v1.1.17+ and integrating with off-chain infrastructure. It consolidates verified security patterns, critical vulnerability mitigations, and best practices from professional audits (Vacuumlabs, MLabs) and production dApps.

**Target Audience**: Developers building Cardano dApps with Aiken validators and off-chain transaction builders (Lucid Evolution, Mesh SDK).

## Core Development Principles (MANDATORY)

### 1. KISS (Keep It Simple, Stupid)
- Prefer simple validator logic over complex state machines
- Use standard UTxO patterns before creating custom solutions
- Question every abstraction - Cardano's EUTxO model is already powerful
- Simpler validators = lower script execution costs

### 2. TDD (Test-Driven Development)
- Write Aiken tests first using `test` blocks
- Use property-based testing with `aiken check`
- Write off-chain integration tests with your chosen library
- Never deploy validators without comprehensive test coverage
- Test both success and failure paths

### 3. Separation of Concerns (SOC)
- Validators handle on-chain validation only (pure functions returning Bool)
- Off-chain code handles transaction building, signing, submission
- Keep business logic separate from UI components
- Use dedicated services for chain data queries

### 4. DRY (Don't Repeat Yourself)
- Extract common validator patterns into reusable functions
- Use Aiken's module system for shared logic
- Create utility libraries for off-chain transaction building
- Reuse proven security patterns from this template

### 5. Documentation Standards
- Always include actual dates when writing documentation
- Use objective, factual language only
- Avoid marketing terms ("production-ready" allowed only when verified)
- State current Aiken version compatibility explicitly
- Document Plutus version requirements (V3 for Conway era)

### 6. Jimmy's Workflow (Red/Green Checkpoints)
**MANDATORY for all implementation tasks**

Use the Red/Green/Blue checkpoint system to prevent AI hallucination and ensure robust implementation:

- 🔴 **RED (IMPLEMENT)**: Write validators, build transactions, configure infrastructure
- 🟢 **GREEN (VALIDATE)**: Run `aiken check`, deploy to testnet, test transactions
- 🔵 **CHECKPOINT**: Mark completion with machine-readable status, document rollback

**Critical Rules:**
- NEVER skip validation phases (always test on testnet before mainnet)
- NEVER proceed to next checkpoint without GREEN passing
- ALWAYS document rollback procedures
- ALWAYS use explicit validation commands (not assumptions)

**Reference**: See **JIMMYS-WORKFLOW.md** in parent directory for complete workflow system

**Usage**: When working with AI assistants, say: *"Let's use Jimmy's Workflow to execute this plan"*

**Benefits:**
- Prevents "validator compiles ≠ validator is secure" problem
- Forces on-chain testing at every step
- Enables autonomous development with safety gates
- Provides clear rollback paths for validator updates
- Integrates with Aiken's built-in testing framework

## Service Overview

This template collection provides comprehensive guidance for Cardano smart contract development, covering:

**Key Responsibilities:**
- On-chain validator development with Aiken v1.1.17+
- Security vulnerability identification and mitigation
- Off-chain transaction building and chain data integration
- Token standard implementation (CIP-25 NFTs, CIP-68 metadata)
- Production deployment patterns and best practices

**Important Distinctions:**
- **Validators vs Minting Policies**: Validators control spending from script addresses; minting policies control token creation/burning
- **Datum vs Redeemer**: Datum is UTxO metadata; redeemer is the "unlock" argument provided by spender
- **Inline Datum vs Datum Hash**: Inline datums are visible on-chain; datum hashes require separate datum attachment
- **Reference Scripts vs Direct Scripts**: Reference scripts save transaction size; direct scripts are self-contained

## Quick Decision Guide

Use this table to quickly find the right document for your task:

| I Want To... | Primary Document | Supporting Documents |
|--------------|------------------|----------------------|
| **Write an Aiken validator** | `aiken-development-rules.md` | `cardano-security-patterns.md` |
| **Design from scratch** | `validator-design-guide.md` | `cardano-security-patterns.md` |
| **Build transactions (JS/TS)** | `cardano-offchain-integration.md` | `aiken-development-rules.md` |
| **Build backend (Haskell)** | `atlas-framework-guide.md` | `cardano-security-patterns.md` |
| **Choose framework** | `AGENTS.md` (this doc) | `cardano-offchain-integration.md`, `atlas-framework-guide.md` |
| **Learn from Ethereum** | `ethereum-cardano-comparison.md` | `validator-design-guide.md` |
| **Security audit** | `cardano-security-patterns.md` | All validator documents |
| **Mint NFT** | `aiken-development-rules.md` (on-chain) | `cardano-offchain-integration.md` (CIP-25) |

## Current Status

✅ **COMPLETE** - 100% coverage of critical patterns

- ✅ Aiken development rules and syntax guide (v1.1.17)
- ✅ Security vulnerability patterns (5 critical vulnerabilities documented)
- ✅ Validator design guide and discovery framework
- ✅ Ethereum/Cardano comparison for migrating developers
- ✅ Off-chain integration guide (Lucid Evolution, Mesh, Blockfrost, Maestro)
- ✅ Atlas framework guide (Haskell PAB for complex DeFi)
- ✅ Supporting verification document

## Technology Stack

### On-Chain (Validators)
- **Language**: Aiken v1.1.17+ (current stable as of January 2025)
- **Plutus Version**: V3 (Conway era compatibility)
- **Testing**: Aiken built-in test framework, property-based testing
- **Build Tool**: `aiken build` (generates plutus.json)
- **Validation**: Parse-Validate-Succeed pattern (Aiken's CEI equivalent)

### Off-Chain (Transaction Building)
- **JavaScript/TypeScript Libraries**:
  - Lucid Evolution v0.4.29+ (lightweight, backend-focused, Aiken-recommended)
  - Mesh SDK v1.9.0+ (full-stack, React-friendly, comprehensive)
- **Haskell Framework**:
  - Atlas v0.14.1+ (Plutus Application Backend, production-proven, complex DeFi)
- **Chain Data Providers**:
  - Blockfrost (free tier, IPFS support, widely adopted)
  - Maestro (high-performance, batch queries, DeFi feeds, recommended for Atlas)
  - Kupo + Local Node (full decentralization, Atlas-supported)
- **Wallet Integration**: CIP-30 browser wallets (Lucid/Mesh), seed phrase wallets (all frameworks)
- **Token Standards**: CIP-25 (NFT metadata), CIP-68 (on-chain metadata), CIP-57 (Plutus blueprints)

### Infrastructure
- **Networks**: Mainnet, Preprod (testnet), Preview (testnet), Sanchonet (governance testnet)
- **Deployment**: Cardano CLI, Lucid/Mesh transaction submission
- **Monitoring**: Chain explorers (cardanoscan.io, cexplorer.io, adastat.net)

## Build & Test Commands

### Validator Development (Aiken)
```bash
# Install Aiken (latest)
cargo install aiken

# Check Aiken version
aiken --version  # Should be v1.1.17 or later

# Build validators (generates plutus.json)
aiken build

# Run all tests
aiken check

# Run specific test
aiken check -m module_name

# Format code
aiken fmt

# Watch mode (rebuild on changes)
aiken build --watch
```

### Off-Chain Development (Node.js/TypeScript)
```bash
# Install dependencies (Lucid Evolution example)
npm install @lucid-evolution/lucid

# Install dependencies (Mesh SDK example)
npm install @meshsdk/core @meshsdk/react

# TypeScript compilation
npm run typecheck

# Run integration tests
npm test

# Linting
npm run lint
```

### Off-Chain Development (Haskell/Atlas)
```bash
# Install Haskell toolchain (ghcup)
curl --proto '=https' --tlsv1.2 -sSf https://get-ghcup.haskell.org | sh

# Install GHC and Cabal
ghcup install ghc 9.6.7
ghcup install cabal 3.12.1.0

# Build Atlas project
cabal update
cabal build

# Run tests (CLB emulator)
cabal test

# Run tests (private testnet)
cabal test --test-options="--privnet"
```

### Deployment & Testing
```bash
# Test on Preprod testnet
# (Set network in your off-chain code to "Preprod")

# Deploy to mainnet
# (Set network to "Mainnet" and use production API keys)

# Check transaction on explorer
# Visit: https://cardanoscan.io/transaction/[TX_HASH]
```

## Repository Structure

```
/home/jimmyb/templates/cardano/
├── aiken-development-rules.md          # Aiken v1.1.17 syntax and patterns
├── cardano-security-patterns.md        # 5 critical vulnerabilities + mitigations
├── validator-design-guide.md           # Design-first approach for validators
├── ethereum-cardano-comparison.md      # Migration guide for Ethereum devs
├── cardano-offchain-integration.md     # Lucid/Mesh/Blockfrost/Maestro guide
├── atlas-framework-guide.md            # Atlas Haskell PAB guide
├── JIMMYS-WORKFLOW.md                  # RED/GREEN/CHECKPOINT validation system
├── supporting document.md              # Verification and sources
├── AGENTS.md                           # This file - AI assistant guidelines
└── README.md                           # User-facing overview
```

## Development Workflow

### Starting Work on a Validator
1. Read **validator-design-guide.md** for discovery questions
2. Check **cardano-security-patterns.md** for critical vulnerabilities
3. Review **aiken-development-rules.md** for syntax and patterns
4. **Use Jimmy's Workflow**: Plan → Implement → Validate → Checkpoint
5. Follow TDD approach - write Aiken tests first
6. Implement validator logic following Parse-Validate-Succeed pattern
7. Run `aiken check` to ensure tests pass

### Before Deploying to Mainnet
1. Run all tests: `aiken check`
2. Build validators: `aiken build`
3. Test on Preprod testnet with real transactions
4. Verify all 5 critical vulnerabilities are mitigated
5. Ensure no credentials are exposed in off-chain code
6. Document validator parameters and expected behavior
7. Use Jimmy's Workflow checkpoints to validate security completeness

### Off-Chain Integration Workflow
1. Review **cardano-offchain-integration.md** for library selection
2. Choose transaction builder (Lucid Evolution or Mesh)
3. Choose chain data provider (Blockfrost or Maestro)
4. Implement transaction building with proper datum/redeemer handling
5. Test on testnet before mainnet
6. Implement proper error handling and retry logic

### Documentation Updates
1. Update README.md with any new patterns discovered
2. Add inline comments for complex validator logic
3. Update this AGENTS.md if development approach changes
4. Document all security decisions with dates
5. Keep version numbers current (Aiken, Lucid, Mesh)

## Known Issues & Technical Debt

### 🔴 Critical Issues
None currently - all documented patterns are production-ready as of January 2025.

### 🟡 Important Issues
None currently - comprehensive coverage of common scenarios.

### 📝 Technical Debt
1. Could add more advanced DeFi patterns (AMM, lending protocols)
2. Could expand CIP-68 examples with more token types
3. Could add Kupmios and Ogmios as alternative chain data providers

## Cardano-Specific Guidelines

### Validator Code Style (Aiken)
- Use snake_case for functions and variables
- Use PascalCase for types and constructors
- Keep validator functions pure (no side effects)
- Maximum validator complexity: aim for < 100 lines per validator
- Use `expect` for pattern matching, not `when` (Aiken v1.1.17+)
- Always validate all inputs explicitly (no implicit trust)

### Testing Requirements
- Minimum 80% test coverage for validators
- Test both success paths and failure paths
- Use property-based testing for complex logic
- Test edge cases: empty lists, zero amounts, max values
- Integration tests on testnet before mainnet deployment

### Security Considerations (CRITICAL)
**Always check for these 5 vulnerabilities:**

1. **Double Satisfaction** - Validate each input independently
2. **Missing UTxO Authentication** - Use NFT tokens to identify correct UTxOs
3. **Incomplete Token Validation** - Check policy ID, asset name, AND quantity
4. **Unbounded Datum/Value** - Enforce datum < 16KB, value < 5KB limits
5. **Insufficient Staking Control** - Verify withdrawal credentials

**Reference**: See **cardano-security-patterns.md** for detailed mitigations

### Off-Chain Security
- Store API keys in environment variables (never commit)
- Use backend proxy for Blockfrost/Maestro in client-side apps
- Validate transaction outputs before signing
- Use hardware wallets for high-value operations
- Test transaction building with small amounts first

## Common Patterns & Examples

### Pattern 1: NFT-Authenticated UTxO Spending
**Use Case**: Prevent double satisfaction attacks

```aiken
validator spend_with_nft(nft_policy: PolicyId, nft_name: AssetName) {
  spend(
    datum: Option<MyDatum>,
    redeemer: MyRedeemer,
    input: OutputReference,
    self: Transaction,
  ) {
    // Find input being spent
    expect Some(own_input) =
      list.find(self.inputs, fn(i) { i.output_reference == input })

    // Require NFT in input value
    expect quantity_of(own_input.output.value, nft_policy, nft_name) == 1

    // Additional validation...
    True
  }
}
```

### Pattern 2: Off-Chain Transaction Building (Lucid Evolution)
**Use Case**: Spend from validator with proper datum/redeemer

```typescript
import { Lucid, Blockfrost, Data } from "@lucid-evolution/lucid";

// Initialize
const lucid = await Lucid.new(
  new Blockfrost("https://cardano-preprod.blockfrost.io/api/v0", API_KEY),
  "Preprod"
);

// Load wallet
lucid.selectWallet(await window.cardano.nami.enable());

// Load validator
const validator = {
  type: "PlutusV3",
  script: applyParamsToScript(blueprintValidator, [params])
};

// Find UTxO at script address
const scriptAddress = lucid.utils.validatorToAddress(validator);
const [scriptUtxo] = await lucid.utxosAt(scriptAddress);

// Build transaction
const redeemer = Data.to(100n);
const tx = await lucid.newTx()
  .collectFrom([scriptUtxo], redeemer)
  .attachSpendingValidator(validator)
  .complete();

// Sign and submit
const signedTx = await tx.sign().complete();
const txHash = await signedTx.submit();
```

### Pattern 3: Token Validation (Preventing Token Forgery)
**Use Case**: Validate policy ID, asset name, AND quantity

```aiken
fn validate_token_payment(
  value: Value,
  expected_policy: PolicyId,
  expected_name: AssetName,
  expected_quantity: Int,
) -> Bool {
  let actual_quantity = quantity_of(value, expected_policy, expected_name)

  // Check all three components
  actual_quantity == expected_quantity &&
  expected_quantity > 0  // Prevent zero-quantity bypass
}
```

## Dependencies & Integration

### External Services
- **Blockfrost**: Chain data indexing, UTxO queries, transaction submission, IPFS gateway
- **Maestro**: High-performance chain data, batch queries, DeFi protocol feeds
- **Cardano Node**: Direct node access (alternative to hosted providers)
- **Kupo**: Lightweight chain indexer (for Atlas local provider)

### Related Patterns
- **Parse-Validate-Succeed**: Aiken's recommended validator structure
- **CIP-25**: NFT metadata standard (JSON in transaction metadata)
- **CIP-68**: On-chain metadata standard (reference token + user token)
- **CIP-57**: Plutus blueprint format (plutus.json schema)

## Environment Variables

### Aiken (Build-Time)
```bash
# None required - Aiken is standalone
```

### Off-Chain (Runtime)
```bash
# Required for Blockfrost
BLOCKFROST_PROJECT_ID=your_project_id_here

# Required for Maestro
MAESTRO_API_KEY=your_api_key_here

# Network selection
CARDANO_NETWORK=Preprod  # or Mainnet

# Optional: Wallet seed phrase (NEVER commit to repo)
WALLET_SEED_PHRASE="24-word mnemonic here"  # Use .env file, add to .gitignore

# For Atlas (Haskell)
KUPO_URL=http://localhost:1442  # If using Kupo provider
CARDANO_NODE_SOCKET_PATH=/path/to/node.socket  # If using local node
```

## Troubleshooting

### Common Issues

**Issue**: `aiken check` fails with "expect pattern match failed"
**Solution**: Review your test data - ensure it matches the exact type structure expected by the validator. Use `trace` statements to debug.

**Issue**: Transaction fails on-chain with "Script execution failed"
**Solution**: Check redeemer format matches validator expectations. Ensure all validation logic passes. Review transaction context (inputs, outputs, signatories).

**Issue**: "UTxO not found" when querying script address
**Solution**: Verify you're on correct network (Preprod vs Mainnet). Check script address generation - parameters must match exactly. Wait for blockchain indexing (can take 20-60 seconds).

**Issue**: Blockfrost rate limit (429 error)
**Solution**: Implement exponential backoff retry logic. Consider upgrading to paid tier. For high-volume apps, use Maestro or run own Cardano node.

**Issue**: Transaction too large (> 16KB)
**Solution**: Use reference scripts instead of attaching scripts to every transaction. Split into multiple transactions if necessary. Consider using reference inputs for large datums.

## Resources & References

### Official Documentation
- Aiken Language: https://aiken-lang.org/
- Cardano Developer Portal: https://developers.cardano.org/
- CIP Standards: https://cips.cardano.org/

### Off-Chain Libraries
- Lucid Evolution: https://anastasia-labs.github.io/lucid-evolution/
- Mesh SDK: https://meshjs.dev/
- Blockfrost: https://docs.blockfrost.io/
- Maestro: https://docs.gomaestro.org/

### Security Resources
- Vacuumlabs Audits: Search "Vacuumlabs Cardano audit" for public reports
- MLabs Security: https://mlabs.city/ (security audits and development)
- Cardano Stack Exchange: https://cardano.stackexchange.com/

### Learning Resources
- Aiken Examples: https://github.com/aiken-lang/aiken/tree/main/examples
- Mesh Aiken Integration: https://meshjs.dev/guides/aiken
- This Template Collection: Read all .md files in this directory

## Common Questions & Quick Answers

**Q: How do I prevent double satisfaction attacks?**
**A**: See `cardano-security-patterns.md` → Vulnerability #1 → Use NFT-authenticated UTxOs

**Q: What's the current Aiken version?**
**A**: v1.1.17+ (as of October 2025) - See AGENTS.md#technology-stack

**Q: Should I use Atlas, Lucid, or Mesh?**
**A**: See Quick Decision Guide above or `atlas-framework-guide.md#when-to-use-atlas`
- **Atlas**: Haskell teams, complex DeFi
- **Lucid**: JS/TS teams, backend-focused
- **Mesh**: JS/TS teams, full-stack with React

**Q: How do I mint an NFT with metadata?**
**A**:
- On-chain (validator): `aiken-development-rules.md` → One-shot minting pattern
- Off-chain (metadata): `cardano-offchain-integration.md` → CIP-25 implementation

**Q: Where's the security checklist?**
**A**: `cardano-security-patterns.md` → Check all 5 critical vulnerabilities before deployment

**Q: How do I test validators?**
**A**:
- Unit tests: `aiken check` (see `aiken-development-rules.md#testing`)
- Integration: Preprod testnet transactions
- Production: `atlas-framework-guide.md#testing-framework` for comprehensive testing

**Q: What are the 5 critical vulnerabilities?**
**A**: See `cardano-security-patterns.md`:
1. Double Satisfaction
2. Missing UTxO Authentication
3. Incomplete Token Validation
4. Unbounded Datum/Value
5. Insufficient Staking Control

**Q: How do I load an Aiken validator in Lucid/Mesh?**
**A**: `cardano-offchain-integration.md#aiken-validator-integration` → Load from plutus.json

**Q: When should I use reference scripts?**
**A**: For validators used frequently - saves 5-10 KB per transaction (see `cardano-offchain-integration.md#performance-optimization`)

**Q: How do I design a validator from scratch?**
**A**: Follow `validator-design-guide.md` → Discovery questions → Security analysis → Implementation

## Important Reminders for AI Assistants

1. **Always use Jimmy's Workflow** for validator development and off-chain integration
2. **Test on Preprod testnet** before any mainnet deployment
3. **Check all 5 critical vulnerabilities** for every validator (see cardano-security-patterns.md)
4. **Use Aiken v1.1.17+ syntax** - verify with `aiken --version`
5. **Plutus V3 required** for Conway era (current as of January 2025)
6. **Never skip checkpoints** - validator compilation ≠ validator security
7. **Update version numbers** when Aiken, Lucid, or Mesh release new versions
8. **Document actual dates** - always include "as of YYYY-MM-DD" in documentation
9. **Validate explicitly** - run `aiken check`, test transactions, verify on explorer
10. **Security first** - when in doubt, consult cardano-security-patterns.md
11. **Choose appropriate framework** - Lucid/Mesh for JS/TS teams, Atlas for Haskell/complex DeFi
12. **Test comprehensively** - Use Atlas emulator + privnet for production-critical applications

---

**This document follows the [agents.md](https://agents.md/) standard for AI coding assistants.**

**Template Version**: 1.0
**Last Updated**: 2025-10-02
