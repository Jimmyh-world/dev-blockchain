# Cardano Off-Chain Integration Guide

**Last Updated**: 2025-10-02
**Aiken Version**: v1.1.17+
**Plutus Version**: V3 (Conway Era)

---

## Table of Contents

1. [Introduction](#introduction)
2. [Technology Selection](#technology-selection)
3. [Setup & Configuration](#setup--configuration)
4. [Core Transaction Building Patterns](#core-transaction-building-patterns)
5. [Aiken Validator Integration](#aiken-validator-integration)
6. [Token Standards Implementation](#token-standards-implementation)
7. [Security Best Practices](#security-best-practices)
8. [Performance Optimization](#performance-optimization)
9. [Recommended Tech Stacks](#recommended-tech-stacks)
10. [Troubleshooting](#troubleshooting)
11. [Resources](#resources)

---

## Introduction

This guide covers off-chain development for Cardano dApps using Aiken validators. Off-chain code handles transaction building, wallet integration, chain data queries, and transaction submission. This guide assumes you have Aiken validators already developed (see **aiken-development-rules.md** for validator development).

### What is Off-Chain Code?

**On-Chain (Validators)**:
- Written in Aiken
- Deployed to blockchain as scripts
- Validate transactions (pure functions returning Bool)
- Deterministic execution

**Off-Chain (Transaction Builders)**:
- Written in TypeScript/JavaScript
- Run in browsers, Node.js, or Deno
- Build transactions, query UTxOs, manage wallets
- Submit transactions to blockchain

**Separation is critical**: Validators validate; off-chain code coordinates.

---

## Technology Selection

### Transaction Builders: Lucid Evolution vs Mesh SDK

| Criterion | Lucid Evolution | Mesh SDK |
|-----------|-----------------|----------|
| **Best For** | Backend, transaction building, contract development | Frontend, full-stack dApps, UI components |
| **Package** | `@lucid-evolution/lucid` v0.4.29+ | `@meshsdk/core` v1.9.0+ |
| **Size** | Lightweight (Rust/WASM core) | <60kB (100% TypeScript) |
| **React Support** | No built-in components | React hooks and components (`@meshsdk/react`) |
| **API Style** | Fluent, minimal (`lucid.newTx().pay().complete()`) | Builder pattern, cardano-cli-like |
| **Aiken Support** | Recommended by Aiken team | Official Aiken integration guide |
| **Documentation** | Good, improving | Excellent with live demos |
| **Learning Curve** | Moderate | Easy (especially for React devs) |
| **Maintenance** | Actively maintained by Anastasia Labs | Actively maintained, frequent updates |
| **Community Rank** | 4th in 2024 Cardano Developer Survey | 3rd in 2024 Cardano Developer Survey |

**Recommendation:**
- **Choose Lucid Evolution** if: Backend-focused, minimal dependencies, working closely with Aiken
- **Choose Mesh SDK** if: Full-stack dApp, React frontend, need UI components
- **Use Both** if: Mesh for frontend (hooks, UI) + Lucid for backend (business logic)

---

### Chain Data Providers: Blockfrost vs Maestro

| Criterion | Blockfrost | Maestro |
|-----------|-----------|---------|
| **Best For** | General purpose, budget-conscious, IPFS needs | High-performance, DeFi, batch operations |
| **Package** | `@blockfrost/blockfrost-js` | `@maestro-org/typescript-sdk` v1.6.3 |
| **Free Tier** | Yes (forever free STARTER plan) | Limited trial/development tier |
| **Rate Limits** | 10 req/sec, 500 burst | Compute Credits-based (more flexible) |
| **IPFS Support** | Yes (gateway + pinning) | No |
| **Batch Queries** | Individual address queries | Multiple addresses in single request |
| **Performance** | Standard API response times | High-performance (100 addresses: 2m15s → 9s) |
| **Uptime SLA** | Enterprise-grade (no specific %) | +99% uptime guarantee |
| **Transaction Management** | Basic submission | Advanced (retries, rollback notifications) |
| **DeFi Data** | Standard chain data | High-fidelity DeFi protocol feeds |
| **Pricing Model** | Request-based tiers | Usage-based Compute Credits |
| **Documentation** | Excellent, comprehensive | Top-notch, comprehensive |

**Recommendation:**
- **Choose Blockfrost** if: Budget-conscious, need IPFS, standard API needs, prefer free tier
- **Choose Maestro** if: High-performance needs, batch operations, DeFi data, willing to pay for premium

---

## Setup & Configuration

### Installing Lucid Evolution

```bash
# Install Lucid Evolution
npm install @lucid-evolution/lucid

# TypeScript (recommended)
npm install --save-dev typescript @types/node
```

**Basic Setup:**
```typescript
import { Lucid, Blockfrost } from "@lucid-evolution/lucid";

// Initialize with Blockfrost
const lucid = await Lucid.new(
  new Blockfrost(
    "https://cardano-preprod.blockfrost.io/api/v0",
    "YOUR_PROJECT_ID"  // Get from blockfrost.io
  ),
  "Preprod"  // or "Mainnet"
);

// Select wallet (browser wallet)
lucid.selectWallet(await window.cardano.nami.enable());

// Or select wallet (seed phrase - backend only)
lucid.selectWalletFromSeed("your 24-word mnemonic here");
```

---

### Installing Mesh SDK

```bash
# Install Mesh SDK (core)
npm install @meshsdk/core

# Install React components (optional)
npm install @meshsdk/react

# TypeScript types included
```

**Basic Setup:**
```typescript
import { BlockfrostProvider, MeshWallet, Transaction } from "@meshsdk/core";

// Initialize provider
const blockchainProvider = new BlockfrostProvider("YOUR_PROJECT_ID");

// Create wallet
const wallet = new MeshWallet({
  networkId: 0,  // 0 = testnet, 1 = mainnet
  fetcher: blockchainProvider,
  submitter: blockchainProvider,
  key: {
    type: "mnemonic",
    words: ["your", "24", "word", "mnemonic", "..."]
  }
});
```

**React Setup:**
```tsx
import { MeshProvider, useWallet } from "@meshsdk/react";

function App() {
  return (
    <MeshProvider>
      <YourComponent />
    </MeshProvider>
  );
}

function YourComponent() {
  const { wallet, connected, connect } = useWallet();

  return (
    <button onClick={() => connect('nami')}>
      Connect Wallet
    </button>
  );
}
```

---

### Provider Setup: Blockfrost

**Get API Key:**
1. Visit https://blockfrost.io/
2. Create account (free)
3. Create project (select network: Preprod for testing, Mainnet for production)
4. Copy `project_id` (this is your API key)

**Environment Variables:**
```bash
# .env file
BLOCKFROST_PROJECT_ID_PREPROD=preprod_your_key_here
BLOCKFROST_PROJECT_ID_MAINNET=mainnet_your_key_here
CARDANO_NETWORK=Preprod
```

**Security:**
```typescript
// ✅ GOOD: Use environment variables
const apiKey = process.env.BLOCKFROST_PROJECT_ID_PREPROD;

// ❌ BAD: Never hardcode API keys
const apiKey = "preprod_abc123...";  // DON'T DO THIS

// ✅ GOOD: Client-side apps - proxy through backend
// Frontend calls your backend API, backend calls Blockfrost
```

---

### Provider Setup: Maestro

**Get API Key:**
1. Visit https://www.gomaestro.org/
2. Sign up for account
3. Create project
4. Copy API key from dashboard

**Environment Variables:**
```bash
# .env file
MAESTRO_API_KEY=your_api_key_here
CARDANO_NETWORK=Preprod  # or Mainnet
```

**Lucid Evolution Integration:**
```typescript
import { Lucid, Maestro } from "@lucid-evolution/lucid";

const lucid = await Lucid.new(
  new Maestro({
    network: "Preprod",  // or "Mainnet"
    apiKey: process.env.MAESTRO_API_KEY
  }),
  "Preprod"
);
```

**Mesh Integration:**
```typescript
import { MaestroProvider } from "@meshsdk/core";

const provider = new MaestroProvider({
  network: "Preprod",
  apiKey: process.env.MAESTRO_API_KEY
});
```

---

## Core Transaction Building Patterns

### Pattern 1: Simple Payment Transaction

**Lucid Evolution:**
```typescript
const tx = await lucid.newTx()
  .payToAddress("addr_test1...", { lovelace: 5000000n })  // 5 ADA
  .complete();

const signedTx = await tx.sign().complete();
const txHash = await signedTx.submit();

console.log(`Transaction submitted: ${txHash}`);
```

**Mesh SDK:**
```typescript
const tx = new Transaction({ initiator: wallet })
  .sendLovelace("addr_test1...", "5000000");  // 5 ADA

const unsignedTx = await tx.build();
const signedTx = await wallet.signTx(unsignedTx);
const txHash = await wallet.submitTx(signedTx);

console.log(`Transaction submitted: ${txHash}`);
```

---

### Pattern 2: Multi-Asset Transfer

**Lucid Evolution:**
```typescript
const policyId = "ace7bcc2ce705679149746620de3a84660ce57573df54b5a096e39a2";
const assetName = "TestToken";

const tx = await lucid.newTx()
  .payToAddress("addr_test1...", {
    lovelace: 2000000n,  // Min ADA
    [policyId + assetName]: 100n  // 100 tokens
  })
  .complete();

const signedTx = await tx.sign().complete();
const txHash = await signedTx.submit();
```

**Mesh SDK:**
```typescript
const tx = new Transaction({ initiator: wallet })
  .sendAssets("addr_test1...", [
    {
      unit: policyId + assetName,
      quantity: "100"
    }
  ]);

const unsignedTx = await tx.build();
const signedTx = await wallet.signTx(unsignedTx);
const txHash = await wallet.submitTx(signedTx);
```

---

### Pattern 3: Querying UTxOs

**Lucid Evolution:**
```typescript
// Get UTxOs at an address
const utxos = await lucid.utxosAt("addr_test1...");

// Get UTxOs with specific asset
const nftUtxos = utxos.filter(utxo => {
  return Object.keys(utxo.assets).some(asset =>
    asset.startsWith(policyId)
  );
});

// Get specific UTxO by output reference
const specificUtxo = await lucid.utxoByUnit(policyId + assetName);
```

**Mesh SDK:**
```typescript
// Get UTxOs at an address
const utxos = await wallet.getUtxos();

// Get UTxOs with specific asset
const nftUtxos = utxos.filter(utxo => {
  return utxo.output.amount.some(asset =>
    asset.unit === policyId + assetName
  );
});
```

**Blockfrost Direct:**
```typescript
import { BlockFrostAPI } from "@blockfrost/blockfrost-js";

const API = new BlockFrostAPI({
  projectId: process.env.BLOCKFROST_PROJECT_ID
});

const utxos = await API.addressesUtxos("addr_test1...");
```

---

### Pattern 4: Transaction Metadata

**Lucid Evolution:**
```typescript
const tx = await lucid.newTx()
  .payToAddress("addr_test1...", { lovelace: 2000000n })
  .attachMetadata(674, {
    msg: ["Hello", "Cardano"]
  })
  .complete();

const signedTx = await tx.sign().complete();
const txHash = await signedTx.submit();
```

**Mesh SDK:**
```typescript
const tx = new Transaction({ initiator: wallet })
  .sendLovelace("addr_test1...", "2000000")
  .setMetadata(674, {
    msg: ["Hello", "Cardano"]
  });

const unsignedTx = await tx.build();
const signedTx = await wallet.signTx(unsignedTx);
const txHash = await wallet.submitTx(signedTx);
```

---

## Aiken Validator Integration

### Loading Validator from plutus.json

**Step 1: Build Aiken Validator**
```bash
# In your Aiken project directory
aiken build

# This generates plutus.json with compiled validators
```

**Step 2: Load in TypeScript (Lucid Evolution)**
```typescript
import { Lucid, SpendingValidator, Data } from "@lucid-evolution/lucid";
import { applyParamsToScript } from "@lucid-evolution/lucid";
import blueprint from "./plutus.json";  // Generated by aiken build

// Load validator from blueprint
const validator: SpendingValidator = {
  type: "PlutusV3",
  script: blueprint.validators[0].compiledCode
};

// If validator has parameters, apply them
const params = [param1, param2];  // Your validator parameters
const validatorWithParams: SpendingValidator = {
  type: "PlutusV3",
  script: applyParamsToScript(blueprint.validators[0].compiledCode, params)
};

// Get script address
const scriptAddress = lucid.utils.validatorToAddress(validator);
console.log(`Script address: ${scriptAddress}`);
```

**Step 3: Load in TypeScript (Mesh SDK)**
```typescript
import { serializePlutusScript } from "@meshsdk/core";
import { applyParamsToScript } from "@meshsdk/core-csl";
import blueprint from "./plutus.json";

// Load and apply parameters
const scriptCbor = applyParamsToScript(
  blueprint.validators[0].compiledCode,
  []  // Parameters if any
);

// Get script address
const scriptAddr = serializePlutusScript(
  { code: scriptCbor, version: "V3" }
).address;

console.log(`Script address: ${scriptAddr}`);
```

---

### Locking Funds to Validator

**Lucid Evolution:**
```typescript
import { Data } from "@lucid-evolution/lucid";

// Define datum type (matches your Aiken validator)
const DatumSchema = Data.Object({
  owner: Data.Bytes(),
  deadline: Data.Integer()
});

type Datum = Data.Static<typeof DatumSchema>;
const Datum = DatumSchema as unknown as Datum;

// Create datum
const datum: Datum = {
  owner: "a2c20c77887ace1cd986193e4e75babd8993cfd56995cd5cfce609c2",
  deadline: 1735689600000n  // Unix timestamp in milliseconds
};

// Build transaction to lock funds
const tx = await lucid.newTx()
  .payToContract(
    scriptAddress,
    { inline: Data.to(datum, Datum) },  // Inline datum (recommended)
    { lovelace: 10000000n }  // Lock 10 ADA
  )
  .complete();

const signedTx = await tx.sign().complete();
const txHash = await signedTx.submit();
```

**Mesh SDK:**
```typescript
import { MeshTxBuilder } from "@meshsdk/core";

const txBuilder = new MeshTxBuilder();

// Create datum (must match Aiken validator structure)
const datum = {
  owner: "a2c20c77887ace1cd986193e4e75babd8993cfd56995cd5cfce609c2",
  deadline: 1735689600000
};

// Build transaction
txBuilder
  .txOut(scriptAddress, [{ unit: "lovelace", quantity: "10000000" }])
  .txOutInlineDatum(datum);

const unsignedTx = txBuilder.complete();
const signedTx = await wallet.signTx(unsignedTx);
const txHash = await wallet.submitTx(signedTx);
```

---

### Spending from Validator

**Lucid Evolution:**
```typescript
import { Data } from "@lucid-evolution/lucid";

// Define redeemer type (matches your Aiken validator)
const RedeemerSchema = Data.Object({
  action: Data.Literal("Claim")
});

type Redeemer = Data.Static<typeof RedeemerSchema>;
const Redeemer = RedeemerSchema as unknown as Redeemer;

// Find UTxO at script address
const scriptUtxos = await lucid.utxosAt(scriptAddress);
const utxoToSpend = scriptUtxos[0];  // Select specific UTxO

// Create redeemer
const redeemer: Redeemer = {
  action: "Claim"
};

// Build transaction to spend from validator
const tx = await lucid.newTx()
  .collectFrom([utxoToSpend], Data.to(redeemer, Redeemer))
  .attachSpendingValidator(validator)
  .addSigner(await lucid.wallet.address())  // Add required signatories
  .validFrom(Date.now() - 100000)  // Validity interval
  .validTo(Date.now() + 200000)
  .complete();

const signedTx = await tx.sign().complete();
const txHash = await signedTx.submit();
```

**Mesh SDK:**
```typescript
import { MeshTxBuilder } from "@meshsdk/core";

const txBuilder = new MeshTxBuilder();

// Find UTxO
const scriptUtxos = await provider.fetchAddressUTxOs(scriptAddress);
const utxoToSpend = scriptUtxos[0];

// Create redeemer
const redeemer = { action: "Claim" };

// Build transaction
txBuilder
  .spendingPlutusScriptV3()
  .txIn(utxoToSpend.input.txHash, utxoToSpend.input.outputIndex)
  .spendingReferenceTxInInlineDatumPresent()
  .spendingReferenceTxInRedeemerValue(redeemer)
  .txInScript(scriptCbor)
  .requiredSignerHash(await wallet.getChangeAddress())
  .invalidBefore(Date.now() - 100000)
  .invalidAfter(Date.now() + 200000);

const unsignedTx = await txBuilder.complete();
const signedTx = await wallet.signTx(unsignedTx);
const txHash = await wallet.submitTx(signedTx);
```

---

### Using Reference Scripts

**Benefits:**
- Reduces transaction size (scripts not included in every tx)
- Lowers transaction fees
- Enables larger scripts

**Step 1: Create Reference Script UTxO (Lucid Evolution)**
```typescript
const tx = await lucid.newTx()
  .payToAddressWithData(
    await lucid.wallet.address(),  // Your address
    { scriptRef: validator },  // Attach script as reference
    { lovelace: 10000000n }  // Min ADA to hold reference
  )
  .complete();

const signedTx = await tx.sign().complete();
const txHash = await signedTx.submit();

// Save this UTxO reference for later use
```

**Step 2: Spend Using Reference Script**
```typescript
// Find reference script UTxO
const refScriptUtxo = await lucid.utxosByOutRef([
  { txHash: "...", outputIndex: 0 }  // Your reference script UTxO
]);

// Build transaction using reference script
const tx = await lucid.newTx()
  .collectFrom([utxoToSpend], redeemer)
  .readFrom(refScriptUtxo)  // Reference the script (don't attach)
  .complete();
```

---

## Token Standards Implementation

### CIP-25: NFT Metadata Standard

**Minting NFT with CIP-25 Metadata (Mesh SDK)**
```typescript
import { Transaction, ForgeScript, AssetMetadata, Mint } from "@meshsdk/core";

// Create minting policy (simple one-time mint)
const usedAddress = await wallet.getUsedAddresses();
const address = usedAddress[0];

const forgingScript = ForgeScript.withOneSignature(address);

// Define NFT metadata (CIP-25)
const assetMetadata: AssetMetadata = {
  name: "My NFT",
  image: "ipfs://QmRhTTbUrPYEw3mJGGhQqQST9k86v1DPBiTTWJGKDJsVFw",
  mediaType: "image/png",
  description: "This is my first Cardano NFT",
  files: [
    {
      name: "My NFT",
      mediaType: "image/png",
      src: "ipfs://QmRhTTbUrPYEw3mJGGhQqQST9k86v1DPBiTTWJGKDJsVFw"
    }
  ]
};

// Define asset to mint
const asset: Mint = {
  assetName: "MyNFT001",
  assetQuantity: "1",
  metadata: assetMetadata,
  label: "721",  // CIP-25 standard label
  recipient: address
};

// Build and submit transaction
const tx = new Transaction({ initiator: wallet });
tx.mintAsset(forgingScript, asset);

const unsignedTx = await tx.build();
const signedTx = await wallet.signTx(unsignedTx);
const txHash = await wallet.submitTx(signedTx);
```

**Uploading to IPFS (Blockfrost)**
```typescript
import { BlockFrostIPFS } from "@blockfrost/blockfrost-js";

const IPFS = new BlockFrostIPFS({
  projectId: process.env.BLOCKFROST_PROJECT_ID
});

// Upload file
const added = await IPFS.add('./path/to/nft-image.png');
console.log(`IPFS hash: ${added.ipfs_hash}`);
// Use: ipfs://QmRhTTbUrPYEw3mJGGhQqQST9k86v1DPBiTTWJGKDJsVFw

// Pin to ensure persistence
await IPFS.pin(added.ipfs_hash);
```

---

### CIP-68: On-Chain Metadata Standard

**CIP-68 Structure:**
- **Reference Token** (label 100): Holds metadata in datum, quantity = 1
- **User Token** (label 222): User holds this, quantity = variable

**Minting CIP-68 Tokens:**
```typescript
// Reference NFT asset name: (100) + asset_name
const refPrefix = "000643b0";  // Label 100 in hex
const userPrefix = "000de140";  // Label 222 in hex
const assetName = "4d79546f6b656e";  // "MyToken" in hex

// Reference token (holds metadata)
const referenceAssetName = refPrefix + assetName;

// User token (held by user)
const userAssetName = userPrefix + assetName;

// Metadata structure (in datum of reference token)
const metadata = {
  name: "My Token",
  image: "ipfs://...",
  description: "CIP-68 token",
  // Additional fields...
};

// Mint both tokens in same transaction
// Reference token goes to script address with metadata in datum
// User token goes to user address
```

**Full implementation**: See Mesh documentation at https://meshjs.dev/ and search for CIP-68 examples.

---

## Security Best Practices

### 1. API Key Management

**✅ DO:**
```typescript
// Use environment variables
const apiKey = process.env.BLOCKFROST_PROJECT_ID;

// Client-side apps: proxy through backend
// Frontend → Your Backend API → Blockfrost
fetch('/api/query-utxos', {
  method: 'POST',
  body: JSON.stringify({ address })
});
```

**❌ DON'T:**
```typescript
// Never hardcode API keys
const apiKey = "preprod_abc123...";  // ❌

// Never commit .env files
// Add to .gitignore:
// .env
// .env.local
// .env.production
```

---

### 2. Transaction Validation Before Signing

**✅ DO:**
```typescript
// Build transaction
const tx = await lucid.newTx()
  .payToAddress(recipientAddr, { lovelace: amount })
  .complete();

// Inspect transaction before signing
console.log("Transaction details:", tx.toString());
console.log("Fee:", tx.body.fee);
console.log("Outputs:", tx.body.outputs);

// Only sign if everything looks correct
const signedTx = await tx.sign().complete();
```

**❌ DON'T:**
```typescript
// Blindly sign without inspection
const tx = await lucid.newTx().payToAddress(...).complete();
const signedTx = await tx.sign().complete();  // ❌ Didn't verify!
```

---

### 3. Wallet Security

**✅ DO:**
```typescript
// Production: Use hardware wallets through browser extensions
lucid.selectWallet(await window.cardano.nami.enable());

// Backend: Secure key management
import { generateMnemonic } from "bip39";
const mnemonic = generateMnemonic(256);  // 24 words
// Store in secure vault (AWS Secrets Manager, Azure Key Vault, etc.)

// Development/Testing: Testnet keys only
lucid.selectWalletFromSeed(process.env.TESTNET_SEED_PHRASE);
```

**❌ DON'T:**
```typescript
// Never store mainnet seed phrases in code or .env files committed to Git
const mnemonic = "word1 word2 ... word24";  // ❌ NEVER DO THIS

// Never reuse testnet keys on mainnet
```

---

### 4. Input Validation

**✅ DO:**
```typescript
function validateAddress(addr: string): boolean {
  try {
    // Use library validation
    lucid.utils.getAddressDetails(addr);
    return true;
  } catch {
    return false;
  }
}

function validateAmount(amount: bigint): boolean {
  return amount > 0n && amount <= 45000000000000000n;  // Max ADA supply
}

// Use before building transactions
if (!validateAddress(recipientAddr)) {
  throw new Error("Invalid recipient address");
}
```

---

### 5. Error Handling

**✅ DO:**
```typescript
async function submitTransaction(tx: Transaction) {
  try {
    const signedTx = await tx.sign().complete();
    const txHash = await signedTx.submit();

    console.log(`Transaction submitted: ${txHash}`);

    // Wait for confirmation
    await lucid.awaitTx(txHash);
    console.log("Transaction confirmed!");

    return txHash;
  } catch (error) {
    if (error.message.includes("UTxO")) {
      console.error("UTxO error - likely already spent");
    } else if (error.message.includes("fee")) {
      console.error("Insufficient funds for fee");
    } else {
      console.error("Transaction failed:", error);
    }
    throw error;
  }
}
```

---

## Performance Optimization

### 1. Batch Queries (Maestro)

**❌ Slow (100 addresses, 2 minutes 15 seconds):**
```typescript
// Blockfrost: One request per address
const allUtxos = [];
for (const addr of addresses) {
  const utxos = await blockfrostAPI.addressesUtxos(addr);
  allUtxos.push(...utxos);
}
```

**✅ Fast (100 addresses, 9 seconds):**
```typescript
// Maestro: Batch query
const allUtxos = await maestroClient.addresses
  .utxosByAddresses(addresses);  // All at once
```

---

### 2. Caching UTxO Queries

**✅ DO:**
```typescript
// Cache UTxOs for short periods (20-60 seconds)
const utxoCache = new Map<string, { utxos: UTxO[], timestamp: number }>();

async function getCachedUtxos(address: string): Promise<UTxO[]> {
  const cached = utxoCache.get(address);
  const now = Date.now();

  if (cached && now - cached.timestamp < 60000) {  // 1 minute cache
    return cached.utxos;
  }

  const utxos = await lucid.utxosAt(address);
  utxoCache.set(address, { utxos, timestamp: now });

  return utxos;
}
```

---

### 3. Rate Limit Handling

**✅ DO:**
```typescript
async function retryWithBackoff<T>(
  fn: () => Promise<T>,
  maxRetries = 3
): Promise<T> {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await fn();
    } catch (error) {
      if (error.status === 429) {  // Rate limited
        const delay = Math.pow(2, i) * 1000;  // Exponential backoff
        console.log(`Rate limited, retrying in ${delay}ms...`);
        await new Promise(resolve => setTimeout(resolve, delay));
      } else {
        throw error;
      }
    }
  }
  throw new Error("Max retries exceeded");
}

// Use:
const utxos = await retryWithBackoff(() => lucid.utxosAt(address));
```

---

### 4. Reference Scripts

**✅ DO:**
```typescript
// Use reference scripts for validators used frequently
// Saves ~5-10 KB per transaction (significant fee savings)

const tx = await lucid.newTx()
  .collectFrom([utxoToSpend], redeemer)
  .readFrom([refScriptUtxo])  // Read reference instead of attaching
  .complete();

// Fee savings: ~0.1-0.5 ADA per transaction
```

---

## Recommended Tech Stacks

### Stack 1: Simple dApp (Budget-Friendly)
**Best For**: MVPs, small projects, learning

- **Transaction Builder**: Lucid Evolution
- **Chain Data**: Blockfrost (free tier)
- **Frontend**: React + Lucid Evolution
- **Deployment**: Vercel/Netlify

**Pros**: Free tier sufficient, simple setup, good documentation
**Cons**: Limited performance for high-volume apps

---

### Stack 2: Full-Stack dApp (Developer-Friendly)
**Best For**: Production dApps with React frontends

- **Transaction Builder**: Mesh SDK
- **Chain Data**: Blockfrost (paid tier)
- **Frontend**: React + `@meshsdk/react` hooks
- **Backend**: Node.js + Mesh SDK
- **Deployment**: Vercel (frontend) + AWS Lambda (backend)

**Pros**: Best DX, UI components included, comprehensive SDK
**Cons**: Slightly larger bundle size

---

### Stack 3: High-Performance DeFi
**Best For**: DeFi protocols, high-volume applications

- **Transaction Builder**: Lucid Evolution
- **Chain Data**: Maestro
- **Frontend**: React + Lucid Evolution
- **Backend**: Node.js + Lucid Evolution + Maestro batch queries
- **Deployment**: AWS (auto-scaling)

**Pros**: Best performance, batch queries, DeFi data feeds
**Cons**: Higher cost, more complex setup

---

### Stack 4: Hybrid (Best of Both Worlds)
**Best For**: Professional production dApps

- **Frontend**: Mesh SDK (`@meshsdk/react` for UI)
- **Backend**: Lucid Evolution (business logic)
- **Chain Data**: Maestro (production) + Blockfrost (development)
- **Deployment**: Vercel (frontend) + AWS ECS (backend)

**Pros**: Optimal tools for each layer, separation of concerns
**Cons**: More setup, managing two libraries

---

## Troubleshooting

### Issue: Transaction Fails with "UTxO not found"

**Cause**: UTxO already spent or indexer not yet updated

**Solution:**
```typescript
// Wait for blockchain indexing (20-60 seconds after transaction)
await new Promise(resolve => setTimeout(resolve, 60000));

// Retry query
const utxos = await lucid.utxosAt(address);

// Or use transaction confirmation
await lucid.awaitTx(txHash);
```

---

### Issue: "Insufficient collateral"

**Cause**: Smart contract transactions require collateral (5 ADA minimum)

**Solution:**
```typescript
// Ensure wallet has UTxO with only ADA (no tokens) >= 5 ADA
const tx = await lucid.newTx()
  .collectFrom([utxoToSpend], redeemer)
  .attachSpendingValidator(validator)
  .complete();  // Lucid auto-selects collateral

// Or manually specify collateral UTxO
const tx = await lucid.newTx()
  .collectFrom([utxoToSpend], redeemer)
  .attachSpendingValidator(validator)
  .addCollateral([collateralUtxo])  // Manual collateral
  .complete();
```

---

### Issue: Blockfrost 429 Rate Limit Error

**Cause**: Exceeded 10 requests/second or daily limit

**Solution:**
```typescript
// Implement retry with exponential backoff (see Performance section)
// Or upgrade to paid tier
// Or switch to Maestro for higher rate limits
```

---

### Issue: "Redeemer evaluation failed"

**Cause**: Validator rejected redeemer (validation logic failed)

**Solution:**
```typescript
// Check validator logic in Aiken
// Use `trace` statements for debugging
// Test validator on testnet first

// Ensure redeemer matches expected type
const redeemer = Data.to({ action: "Claim" }, RedeemerSchema);

// Check transaction context (signatories, validity interval, etc.)
const tx = await lucid.newTx()
  .collectFrom([utxoToSpend], redeemer)
  .attachSpendingValidator(validator)
  .addSigner(await lucid.wallet.address())  // Required signatory?
  .validFrom(Date.now())  // Validity interval correct?
  .validTo(Date.now() + 300000)
  .complete();
```

---

### Issue: Transaction Too Large (>16KB)

**Cause**: Transaction exceeds maximum size

**Solution:**
```typescript
// 1. Use reference scripts (saves 5-10 KB)
.readFrom([refScriptUtxo])

// 2. Use reference inputs for large datums
.readFrom([datumRefUtxo])

// 3. Split into multiple transactions
// Transaction 1: Lock funds
// Transaction 2: Spend funds

// 4. Optimize datum size (see cardano-security-patterns.md)
```

---

## Resources

### Official Documentation
- **Lucid Evolution**: https://anastasia-labs.github.io/lucid-evolution/
- **Mesh SDK**: https://meshjs.dev/
- **Blockfrost**: https://docs.blockfrost.io/
- **Maestro**: https://docs.gomaestro.org/

### Integration Guides
- **Aiken + Lucid**: https://aiken-lang.org/example--hello-world/end-to-end/lucid
- **Aiken + Mesh**: https://meshjs.dev/guides/aiken
- **CIP-25 NFTs**: https://cips.cardano.org/cip/CIP-25
- **CIP-68 Tokens**: https://cips.cardano.org/cip/CIP-68

### Learning Resources
- **Cardano Developer Portal**: https://developers.cardano.org/
- **Mesh Playground**: https://meshjs.dev/apis (live demos)
- **Cardano Stack Exchange**: https://cardano.stackexchange.com/

### Related Template Docs
- **aiken-development-rules.md**: Validator development guide
- **cardano-security-patterns.md**: Critical security vulnerabilities
- **validator-design-guide.md**: Design-first approach
- **AGENTS.md**: AI assistant guidelines for this template

---

**Document Version**: 1.0
**Last Updated**: 2025-10-02
**Verified With**: Lucid Evolution v0.4.29, Mesh SDK v1.9.0, Blockfrost API v0, Maestro API v1

---

## Related Documents

- **Validator Development**: `aiken-development-rules.md` - Build validators that this guide will interact with
- **Validator Loading**: Reference plutus.json blueprints from Aiken builds
- **Security**: `cardano-security-patterns.md` - Ensure off-chain code validates security requirements
- **Alternative (Haskell)**: `atlas-framework-guide.md` - For Haskell teams building complex backends
- **Complete Workflow**: `cardano-ai-rules.md` - AI assistant behavior for transaction building
- **Overview**: `AGENTS.md` - Technology stack selection guide
