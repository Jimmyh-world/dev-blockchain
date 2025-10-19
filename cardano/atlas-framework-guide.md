# Atlas Framework - Haskell Plutus Application Backend Guide

**Last Updated**: 2025-10-02
**Atlas Version**: 0.14.1
**Maintainer**: Genius Yield (Dr. Lars Brünjes, MLabs, Well-Typed, Plank)
**Status**: Production-ready, active development

---

## Table of Contents

1. [Introduction](#introduction)
2. [When to Use Atlas](#when-to-use-atlas)
3. [Core Concepts](#core-concepts)
4. [Setup & Installation](#setup--installation)
5. [Transaction Building](#transaction-building)
6. [Testing Framework](#testing-framework)
7. [Data Providers](#data-providers)
8. [Integration with Plutus Validators](#integration-with-plutus-validators)
9. [Comparison to Lucid/Mesh](#comparison-to-lucidmesh)
10. [Production Deployment](#production-deployment)
11. [Best Practices](#best-practices)
12. [Common Patterns](#common-patterns)
13. [Troubleshooting](#troubleshooting)
14. [Resources](#resources)

---

## Introduction

### What is Atlas?

**Atlas** is an open-source, Haskell-native Plutus Application Backend (PAB) framework for building off-chain code for Cardano smart contracts. It abstracts blockchain complexity to accelerate dApp development.

**Developed By**: Genius Yield in collaboration with MLabs, Well-Typed, and Plank
**Open-Sourced**: March 2023
**Lead Architect**: Dr. Lars Brünjes (Plutus architect at IOG)

### Key Features

- **Type Safety**: Haskell's type system provides compile-time guarantees
- **Code Sharing**: Reuse types and logic between on-chain (Plutus) and off-chain code
- **Declarative Transactions**: GYTxSkeleton system for composable transaction building
- **Comprehensive Testing**: Unified testing across emulator and real networks
- **Modular Providers**: Support for Maestro, Blockfrost, Kupo, DB Sync
- **Production-Proven**: Powers Genius Yield DEX and World Mobile Token

### What Problems Does Atlas Solve?

1. **Complexity Abstraction**: Simplifies UTxO selection, fee calculation, transaction balancing
2. **Code Duplication**: Eliminates duplication between on-chain and off-chain (both Haskell)
3. **Testing Challenges**: Provides emulator and private testnet testing
4. **Data Provider Flexibility**: Pluggable blockchain data sources
5. **Type Safety**: Leverages Haskell's strong type system

---

## When to Use Atlas

### Ideal Use Cases

**✅ Choose Atlas When:**
- Team has **Haskell expertise** or willingness to invest in learning
- Building **complex DeFi protocols** (DEXs, lending, derivatives)
- **Type safety** is critical (financial applications)
- **Backend-heavy** architecture
- **Code sharing** between Plutus validators and off-chain logic desired
- Need **comprehensive testing** framework
- **High-assurance** application requirements

**Production Examples:**
- Genius Yield DEX (order-book with concentrated liquidity)
- Genius X Launchpad (smart contract-based launchpad)
- World Mobile Token infrastructure
- Genius Yield Smart Liquidity Vault

---

### When to Consider Alternatives

**❌ Atlas May Not Be Ideal When:**
- Team lacks Haskell experience (steep learning curve)
- **Frontend-focused** dApp (Atlas is backend-only)
- Need **browser wallet integration** (CIP-30 not directly supported)
- **Rapid prototyping** required (JS/TS faster for simple cases)
- **Simple use cases** (payment-only, basic NFT minting)
- Prefer **JavaScript/TypeScript** stack

**Alternatives:**
- **Lucid Evolution**: TypeScript, frontend/backend, moderate complexity
- **Mesh SDK**: TypeScript, full-stack, React-friendly, higher abstraction

---

## Core Concepts

### 1. Transaction Skeletons (GYTxSkeleton)

**Purpose**: Declarative specification of transaction requirements

**Key Idea**: Define WHAT a transaction must include, not HOW to build it

**Monoid Instance**: Combine skeletons with `<>` operator

```haskell
-- Individual requirements
skeleton1 = mustHaveInput someUtxo
skeleton2 = mustHaveOutput someOutput
skeleton3 = mustBeSignedBy somePubKey

-- Combine declaratively
fullSkeleton = skeleton1 <> skeleton2 <> skeleton3
```

**Atlas Handles Automatically:**
- UTxO selection
- Coin balancing
- Fee calculation
- Change output generation
- Minimum ADA requirements
- Collateral management

---

### 2. Monads for Operations

**Three Primary Monads:**

#### GYTxQueryMonad
**Purpose**: Query blockchain state (read-only)

```haskell
queryAddressUtxos :: GYAddress -> GYTxQueryMonad m => m [GYUtxo]
queryAddressUtxos addr = utxosAtAddress addr
```

#### GYTxMonad
**Purpose**: Construct and submit transactions

```haskell
sendPayment :: GYAddress -> GYValue -> GYTxMonad m => m GYTxId
sendPayment addr value = do
  let skeleton = mustHaveOutput (mkGYTxOut addr value)
  sendSkeleton skeleton
```

#### GYTxGameMonad
**Purpose**: Testing with multi-wallet scenarios

```haskell
testMultiUser :: TestInfo -> GYTxGameMonad ()
testMultiUser testInfo = do
  asUser user1 $ operation1  -- Execute as user1
  asUser user2 $ operation2  -- Execute as user2
  asUser user1 $ operation3  -- Back to user1
```

---

### 3. Data Providers

**Modular System**: Pluggable blockchain data sources

**Supported Providers:**

| Provider | Best For | Requirements |
|----------|---------|--------------|
| **Maestro** | Production, high performance | API key (paid) |
| **Blockfrost** | Development, budget-friendly | API key (free tier) |
| **Kupo + Local Node** | Full autonomy, decentralization | Cardano node + Kupo indexer |
| **DB Sync** | Historical data, analytics | Full node + PostgreSQL |

**Recommendation**: Maestro for production, Kupo for decentralization

---

### 4. Testing Backends

**Two Testing Environments:**

#### CLB Emulator
- **Speed**: Very fast (milliseconds per test)
- **Use Case**: Rapid iteration, unit testing
- **Limitations**: Some features not fully supported
- **Fresh Ledger**: Each test starts with clean state

#### Private Testnet (Privnet)
- **Realism**: Full Cardano network (3 nodes)
- **Use Case**: Integration testing, pre-deployment validation
- **Features**: Staking, governance, all Cardano features
- **Slower**: Real block times (~1-2 seconds)

**Unified Testing**: Same test code runs on both backends

---

## Setup & Installation

### Prerequisites

**Required Tools:**
```bash
# GHC (Glasgow Haskell Compiler) 9.6.7+
# Cabal 3.12.1.0+
# Recommended: ghcup for Haskell toolchain management

# Install ghcup
curl --proto '=https' --tlsv1.2 -sSf https://get-ghcup.haskell.org | sh

# Install GHC and Cabal
ghcup install ghc 9.6.7
ghcup install cabal 3.12.1.0

# Set as default
ghcup set ghc 9.6.7
ghcup set cabal 3.12.1.0
```

**Optional: Nix (for reproducible builds)**
```bash
# Install Nix
sh <(curl -L https://nixos.org/nix/install) --daemon

# Enable flakes
mkdir -p ~/.config/nix
echo "experimental-features = nix-command flakes" >> ~/.config/nix/nix.conf
```

---

### Installing Atlas

**Method 1: Add to Cabal Project**

Create `cabal.project` file:
```cabal
packages: .

source-repository-package
  type: git
  location: https://github.com/geniusyield/atlas
  tag: v0.14.1  -- Use latest version
```

Add to `your-project.cabal`:
```cabal
build-depends:
    base >= 4.9 && < 5
  , atlas-cardano >= 0.14
```

**Method 2: Clone and Build**
```bash
# Clone repository
git clone https://github.com/geniusyield/atlas.git
cd atlas

# Build with Cabal
cabal update
cabal build all

# Build with Nix (optional)
nix build
```

---

### Configuration

**Provider Configuration Example:**

```haskell
import GeniusYield.Providers.Maestro

-- Maestro provider configuration
maestroConfig :: GYNetworkId -> String -> GYProviders
maestroConfig netId apiKey = maestroProvider netId apiKey

-- Initialize for Preprod testnet
let providers = maestroConfig GYTestnetPreprod "YOUR_MAESTRO_API_KEY"
```

**Environment Variables:**
```bash
# .env file
MAESTRO_API_KEY=your_api_key_here
NETWORK=preprod  # or mainnet

# For Kupo provider
KUPO_URL=http://localhost:1442
CARDANO_NODE_SOCKET_PATH=/path/to/node.socket
```

---

## Transaction Building

### Basic Payment Transaction

```haskell
{-# LANGUAGE OverloadedStrings #-}

import GeniusYield.TxBuilder
import GeniusYield.Types

-- Simple payment function
sendAda :: GYAddress -> Integer -> GYTxMonad m => m GYTxId
sendAda recipientAddr lovelaceAmount = do
  let value = valueFromLovelace lovelaceAmount
      output = mkGYTxOut recipientAddr value Nothing
      skeleton = mustHaveOutput output

  sendSkeleton skeleton
```

**Usage:**
```haskell
-- Send 10 ADA
txId <- sendAda "addr_test1..." 10_000_000
```

---

### Multi-Output Transaction

```haskell
-- Send to multiple recipients
sendToMultiple :: [(GYAddress, GYValue)] -> GYTxMonad m => m GYTxId
sendToMultiple recipients = do
  let outputs = [mkGYTxOut addr val Nothing | (addr, val) <- recipients]
      skeleton = mconcat [mustHaveOutput out | out <- outputs]

  sendSkeleton skeleton
```

**Usage:**
```haskell
-- Send to 3 addresses
let recipients =
      [ ("addr1...", valueFromLovelace 5_000_000)
      , ("addr2...", valueFromLovelace 3_000_000)
      , ("addr3...", valueFromLovelace 2_000_000)
      ]

txId <- sendToMultiple recipients
```

---

### Transaction with Metadata

```haskell
import qualified Data.Map as Map

-- Transaction with metadata
sendWithMetadata :: GYAddress -> GYValue -> GYTxMetadata -> GYTxMonad m => m GYTxId
sendWithMetadata addr value metadata = do
  let output = mkGYTxOut addr value Nothing
      skeleton = mustHaveOutput output
              <> mustHaveMetadata metadata

  sendSkeleton skeleton
```

**Usage:**
```haskell
-- Create metadata (label 674 is common for messages)
let metadata = Map.singleton 674 (toJSON ["Hello", "Cardano"])

txId <- sendWithMetadata "addr_test1..." (valueFromLovelace 2_000_000) metadata
```

---

### Spending from Script Address

```haskell
-- Spend UTxO from validator
spendFromValidator :: GYValidator -> GYRedeemer -> GYUtxo -> GYTxMonad m => m GYTxId
spendFromValidator validator redeemer utxo = do
  let skeleton = mustHaveInput (GYTxIn utxo (Just redeemer))
              <> mustHaveRefInput validator  -- Reference script
              <> mustBeSignedBy myPubKeyHash

  sendSkeleton skeleton
```

---

### Combining Skeletons (Monoid Pattern)

```haskell
-- Build complex transaction from parts
complexTransaction :: GYTxMonad m => m GYTxId
complexTransaction = do
  -- Part 1: Spend from script
  let spendSkeleton = mustHaveInput scriptInput

  -- Part 2: Create new outputs
  let outputSkeleton = mustHaveOutput output1
                    <> mustHaveOutput output2

  -- Part 3: Add metadata
  let metadataSkeleton = mustHaveMetadata metadata

  -- Part 4: Signatories
  let signingSkeleton = mustBeSignedBy pubKey1
                     <> mustBeSignedBy pubKey2

  -- Combine all parts
  let fullSkeleton = spendSkeleton
                  <> outputSkeleton
                  <> metadataSkeleton
                  <> signingSkeleton

  sendSkeleton fullSkeleton
```

---

## Testing Framework

### Unit Testing with CLB Emulator

**Setup Test:**
```haskell
{-# LANGUAGE NumericUnderscores #-}

import GeniusYield.Test.Clb

-- Define test
simplePaymentTest :: TestInfo -> IO ()
simplePaymentTest = mkTestFor GYTestnetPreprod $ \testInfo -> do
  -- Test runs in GYTxGameMonad
  let user1Addr = testInfoUser1Address testInfo
      user2Addr = testInfoUser2Address testInfo

  -- Execute as user1
  asUser (testInfoUser1 testInfo) $ do
    -- Send 5 ADA to user2
    txId <- sendAda user2Addr 5_000_000
    return ()

  -- Verify user2 received funds
  asUser (testInfoUser2 testInfo) $ do
    utxos <- queryUtxos user2Addr
    let total = sum [utxoValue u | u <- utxos]
    -- Assert total >= 5 ADA
    return ()
```

**Run Test:**
```bash
cabal test
```

---

### Multi-User Interaction Test

```haskell
-- Test with multiple users interacting
multiUserTest :: TestInfo -> IO ()
multiUserTest = mkTestFor GYTestnetPreprod $ \testInfo -> do
  let user1 = testInfoUser1 testInfo
      user2 = testInfoUser2 testInfo
      user3 = testInfoUser3 testInfo

  -- User1 creates escrow
  escrowUtxo <- asUser user1 $ createEscrow 10_000_000

  -- User2 participates
  asUser user2 $ participateInEscrow escrowUtxo

  -- User3 participates
  asUser user3 $ participateInEscrow escrowUtxo

  -- User1 releases funds
  asUser user1 $ releaseEscrow escrowUtxo

  -- Verify final state
  asUser user2 $ do
    balance <- queryBalance
    -- Assert balance increased
    return ()
```

---

### Private Testnet Testing

**Setup Privnet Test:**
```haskell
import GeniusYield.Test.Privnet

-- Same test, different backend
privnetTest :: TestInfo -> IO ()
privnetTest = mkPrivnetTestFor GYPrivnet $ \testInfo -> do
  -- Exact same test code as emulator
  -- But runs on real Cardano network (3 nodes)
  asUser (testInfoUser1 testInfo) $ do
    txId <- sendAda user2Addr 5_000_000
    return ()
```

**Run Privnet Tests:**
```bash
# Requires WoofPool privnet setup
# See: https://github.com/woofpool/cardano-private-testnet-setup

cabal test --test-options="--privnet"
```

---

### Testing Best Practices

**1. Test Levels:**
- **UPLC Function Tests**: Validate individual validator logic
- **Contract Tests**: Test validators in isolation
- **Operation Tests**: Full transaction workflows (Atlas focus)

**2. Use Both Backends:**
- **CLB Emulator**: Fast iteration during development
- **Privnet**: Final validation before testnet deployment

**3. Property-Based Testing:**
```haskell
import Test.QuickCheck

-- Property: sending ADA preserves total balance
prop_balancePreserved :: Property
prop_balancePreserved = forAll arbitrary $ \amount ->
  amount > 0 ==> runTest $ do
    balanceBefore <- totalBalance
    sendAda addr amount
    balanceAfter <- totalBalance
    return $ balanceBefore == balanceAfter + fees
```

---

## Data Providers

### Maestro Provider (Recommended for Production)

**Setup:**
```haskell
import GeniusYield.Providers.Maestro

maestroProviderConfig :: GYNetworkId -> String -> GYProviders
maestroProviderConfig netId apiKey =
  maestroProvider netId apiKey

-- For Preprod
let providers = maestroProviderConfig GYTestnetPreprod "YOUR_API_KEY"

-- For Mainnet
let providers = maestroProviderConfig GYMainnet "YOUR_API_KEY"
```

**Get API Key**: https://www.gomaestro.org/

**Advantages:**
- High-performance indexing
- Optimized for liveliness and accuracy
- Batch query support
- +99% uptime SLA

**Cost**: Compute Credits-based pricing

---

### Blockfrost Provider

**Setup:**
```haskell
import GeniusYield.Providers.Blockfrost

blockfrostProviderConfig :: GYNetworkId -> String -> GYProviders
blockfrostProviderConfig netId projectId =
  blockfrostProvider netId projectId

-- For Preprod
let providers = blockfrostProviderConfig GYTestnetPreprod "preprod_YOUR_ID"
```

**Get Project ID**: https://blockfrost.io/

**Advantages:**
- Free tier available
- Easy setup
- IPFS support

**Disadvantages:**
- Some API functions suboptimal for Atlas
- Rate limits (10 req/sec free tier)

**Recommendation**: Use for development/testing, not production

---

### Local Provider (Kupo + Cardano Node)

**Setup:**
```haskell
import GeniusYield.Providers.Kupo

kupoProviderConfig :: GYNetworkId -> String -> FilePath -> GYProviders
kupoProviderConfig netId kupoUrl nodeSocketPath =
  kupoProvider netId kupoUrl nodeSocketPath

-- For local node
let providers = kupoProviderConfig
                  GYTestnetPreprod
                  "http://localhost:1442"  -- Kupo URL
                  "/path/to/node.socket"   -- Node socket
```

**Requirements:**
- Running Cardano node
- Kupo indexer (https://cardanosolutions.github.io/kupo/)

**Advantages:**
- Full decentralization
- No external dependencies
- Complete control

**Disadvantages:**
- Infrastructure overhead (184+ GB for node)
- Requires maintenance

---

## Integration with Plutus Validators

### Shared Types Pattern

**Key Advantage**: Use same types on-chain and off-chain

**On-Chain (Plutus Validator):**
```haskell
-- src/OnChain/Types.hs
{-# LANGUAGE TemplateHaskell #-}

module OnChain.Types where

import PlutusTx
import PlutusTx.Prelude

data BetDatum = BetDatum
  { betOracle    :: PubKeyHash
  , betDeadline  :: POSIXTime
  , betAmount    :: Integer
  } deriving Show

PlutusTx.unstableMakeIsData ''BetDatum
```

**Off-Chain (Atlas Operation):**
```haskell
-- src/OffChain/Operations.hs
import OnChain.Types  -- Same types!

createBet :: BetDatum -> GYTxMonad m => m GYTxId
createBet datum = do
  let value = valueFromLovelace (betAmount datum)
      output = mkGYTxOut scriptAddress value (Just $ datumFromPlutus datum)
      skeleton = mustHaveOutput output

  sendSkeleton skeleton
```

**Benefits:**
- No serialization errors
- Type safety across layers
- Code reuse

---

### Validator Interaction Workflow

**Step 1: Define Plutus Validator**
```haskell
{-# LANGUAGE TemplateHaskell #-}

import PlutusTx

validator :: Validator
validator = mkValidatorScript $$(PlutusTx.compile [|| mkValidator ||])
  where
    mkValidator :: Datum -> Redeemer -> ScriptContext -> Bool
    mkValidator datum redeemer ctx =
      -- Validation logic
      traceIfFalse "Condition failed" condition
```

**Step 2: Load in Atlas**
```haskell
import GeniusYield.Types.Script

-- Convert Plutus validator to Atlas type
atlasValidator :: GYValidator
atlasValidator = validatorFromPlutus validator

-- Get script address
scriptAddress :: GYAddress
scriptAddress = addressFromValidator GYTestnetPreprod atlasValidator
```

**Step 3: Lock Funds to Validator**
```haskell
lockToValidator :: Datum -> Integer -> GYTxMonad m => m GYTxId
lockToValidator datum lovelaceAmount = do
  let value = valueFromLovelace lovelaceAmount
      datumHash = hashDatum (datumFromPlutus datum)
      output = mkGYTxOut scriptAddress value (Just datumHash)
      skeleton = mustHaveOutput output

  sendSkeleton skeleton
```

**Step 4: Spend from Validator**
```haskell
spendFromValidator :: GYUtxo -> Redeemer -> GYTxMonad m => m GYTxId
spendFromValidator utxo redeemer = do
  let skeleton = mustHaveInput (GYTxIn utxo (Just $ redeemerFromPlutus redeemer))
              <> mustHaveRefInput atlasValidator
              <> mustBeSignedBy myPubKeyHash
              <> mustHaveValidityRange validityRange

  sendSkeleton skeleton
```

---

## Comparison to Lucid/Mesh

### Feature Comparison Table

| Feature | Atlas | Lucid Evolution | Mesh SDK |
|---------|-------|-----------------|----------|
| **Language** | Haskell | TypeScript | TypeScript |
| **Runtime** | Backend/Server | Browser, Node.js, Deno | Browser, Node.js, React |
| **Learning Curve** | Steep (Haskell) | Moderate | Easy |
| **Type Safety** | Excellent (Haskell) | Good (TypeScript) | Good (TypeScript) |
| **Plutus Integration** | Native (same language) | Via serialization | Via higher abstraction |
| **Testing** | Emulator + Privnet | Similar | Standard JS testing |
| **Wallet Integration** | Backend only | Direct CIP-30 | Built-in CIP-30 + UI |
| **Transaction Building** | Declarative skeletons | Fluent API | Builder pattern |
| **Data Providers** | Modular (4 options) | Built-in (2-3 options) | Built-in providers |
| **Production Use** | Genius Yield, WMT | 34+ projects | Widely adopted |
| **Community** | Smaller (Haskell) | Growing (JS) | Large (JS) |
| **Setup Complexity** | High (GHC, Cabal) | Low (npm) | Low (npm) |
| **Best For** | Complex backend, DeFi | Full-stack, backend focus | Frontend, full-stack |

---

### When to Choose Atlas vs Lucid/Mesh

**Choose Atlas When:**
- ✅ Team has Haskell developers
- ✅ Building complex DeFi (DEX, lending, derivatives)
- ✅ Type safety is critical
- ✅ Sharing code between Plutus and off-chain
- ✅ Need comprehensive testing (emulator + privnet)
- ✅ Backend-heavy architecture

**Choose Lucid Evolution When:**
- ✅ JavaScript/TypeScript team
- ✅ Backend or full-stack application
- ✅ Moderate complexity
- ✅ Need quick setup
- ✅ Working with any validator type (Aiken, Plutus)

**Choose Mesh SDK When:**
- ✅ React-based frontend
- ✅ Need UI components
- ✅ Full-stack dApp
- ✅ Easy learning curve preferred
- ✅ Web development background

---

## Production Deployment

### Infrastructure Requirements

**Haskell Runtime:**
```bash
# Compile to binary
cabal build

# Binary location
.cabal-store/ghc-9.6.7/your-app-0.1.0.0/x/your-app/build/your-app/your-app

# Run binary
./your-app
```

**Data Provider:**
- **Maestro**: API key (recommended for production)
- **Local**: Cardano node (184+ GB) + Kupo indexer

**Server Requirements:**
- **CPU**: 2+ cores
- **RAM**: 4-8 GB for application
- **Storage**: Minimal (unless running local node)
- **OS**: Linux (Ubuntu, Debian, NixOS)

---

### Deployment Best Practices

**1. Use Maestro or Local Provider (Not Blockfrost)**
- Maestro: High performance, production SLA
- Local: Full decentralization

**2. Binary Optimization**
```cabal
-- your-app.cabal
ghc-options: -O2 -threaded -rtsopts -with-rtsopts=-N
```

**3. Error Handling**
```haskell
import Control.Exception

-- Wrap operations with error handling
safeOperation :: GYTxMonad m => m (Either String GYTxId)
safeOperation = do
  result <- try operation
  case result of
    Left (err :: SomeException) -> return $ Left (show err)
    Right txId -> return $ Right txId
```

**4. Monitoring**
- Log all transactions
- Monitor provider health
- Track success/failure rates
- Alert on errors

**5. Gradual Rollout**
- Test on Preprod testnet
- Deploy to mainnet with limits
- Monitor closely
- Scale gradually

---

### Deployment Example (Systemd Service)

**Create service file: `/etc/systemd/system/atlas-app.service`**
```ini
[Unit]
Description=Atlas Cardano Application
After=network.target

[Service]
Type=simple
User=cardano
WorkingDirectory=/home/cardano/atlas-app
ExecStart=/home/cardano/atlas-app/bin/your-app
Restart=on-failure
RestartSec=10

Environment="MAESTRO_API_KEY=your_key_here"
Environment="NETWORK=mainnet"

[Install]
WantedBy=multi-user.target
```

**Start service:**
```bash
sudo systemctl daemon-reload
sudo systemctl enable atlas-app
sudo systemctl start atlas-app
sudo systemctl status atlas-app
```

---

## Best Practices

### 1. Transaction Building

**✅ DO:**
- Use declarative skeletons
- Combine skeletons with `<>` (Monoid)
- Let Atlas handle balancing and fees
- Define reusable operations

**❌ DON'T:**
- Manually calculate fees
- Manually select UTxOs (unless specific reason)
- Hardcode addresses or values
- Skip error handling

---

### 2. Testing

**✅ DO:**
- Test on CLB emulator first (fast iteration)
- Validate on privnet before testnet
- Test multi-user interactions
- Test failure scenarios
- Use property-based testing for complex logic

**❌ DON'T:**
- Skip testing on privnet
- Deploy to mainnet without testnet validation
- Only test happy paths
- Rely on single test backend

---

### 3. Type Safety

**✅ DO:**
- Share types between on-chain and off-chain
- Use newtype wrappers for clarity
- Leverage Haskell's type system
- Define custom error types

**❌ DON'T:**
- Use raw `Integer` everywhere
- Ignore type errors
- Use `error` for control flow
- Skip type signatures

---

### 4. Error Handling

**✅ DO:**
```haskell
import Control.Monad.Except

-- Use Either for operations that can fail
operation :: GYTxMonad m => m (Either OperationError GYTxId)
operation = do
  result <- try executeOperation
  case result of
    Left err -> return $ Left (ProviderError err)
    Right txId -> return $ Right txId
```

**❌ DON'T:**
```haskell
-- Avoid throwing exceptions for expected failures
operation = do
  utxo <- findUtxo
  if isNothing utxo
    then error "UTxO not found"  -- ❌ BAD
    else proceed
```

---

## Common Patterns

### Pattern 1: Payment Operation

```haskell
-- Reusable payment operation
sendPayment :: GYAddress -> GYValue -> GYTxMonad m => m GYTxId
sendPayment recipient amount = do
  let output = mkGYTxOut recipient amount Nothing
      skeleton = mustHaveOutput output
  sendSkeleton skeleton
```

---

### Pattern 2: Script Interaction

```haskell
-- Lock and spend pattern
lockAndSpend :: GYTxMonad m => m (GYTxId, GYTxId)
lockAndSpend = do
  -- Lock
  let datum = MyDatum {field = value}
      lockSkeleton = mustHaveOutput (scriptOutput datum)
  lockTxId <- sendSkeleton lockSkeleton

  -- Wait for confirmation
  awaitTxConfirmed lockTxId

  -- Find locked UTxO
  utxos <- queryUtxos scriptAddress
  let lockedUtxo = head utxos  -- Proper selection in real code

  -- Spend
  let redeemer = MyRedeemer {action = Claim}
      spendSkeleton = mustHaveInput (scriptInput lockedUtxo redeemer)
  spendTxId <- sendSkeleton spendSkeleton

  return (lockTxId, spendTxId)
```

---

### Pattern 3: Batching Operations

```haskell
-- Batch multiple operations into single transaction
batchOperations :: [Operation] -> GYTxMonad m => m GYTxId
batchOperations ops = do
  let skeletons = [operationToSkeleton op | op <- ops]
      combinedSkeleton = mconcat skeletons
  sendSkeleton combinedSkeleton
```

---

### Pattern 4: Conditional Transaction Building

```haskell
-- Build transaction based on conditions
conditionalTx :: Bool -> GYTxMonad m => m GYTxId
conditionalTx includeMetadata = do
  let baseSkeleton = mustHaveOutput output
      metadataSkeleton = if includeMetadata
                           then mustHaveMetadata metadata
                           else mempty  -- Empty skeleton (Monoid identity)
      fullSkeleton = baseSkeleton <> metadataSkeleton
  sendSkeleton fullSkeleton
```

---

## Troubleshooting

### Issue: "Dependency resolution failed"

**Cause**: Haskell dependency conflicts (common with Cardano libraries)

**Solution:**
```bash
# Update package index
cabal update

# Clear cache
rm -rf ~/.cabal/packages/hackage.haskell.org/*

# Use constraints in cabal.project
constraints:
  cardano-api == 8.40.0.0,
  plutus-ledger-api == 1.21.0.0
```

---

### Issue: "UTxO not found" in tests

**Cause**: CLB emulator limitations or timing issues

**Solution:**
```haskell
-- Use privnet for realistic testing
mkPrivnetTestFor GYPrivnet $ \testInfo -> do
  -- Or add delays in emulator
  waitNSlots 5  -- Wait for confirmation
  utxos <- queryUtxos addr
```

---

### Issue: "Insufficient collateral"

**Cause**: Wallet needs UTxO with only ADA (no tokens) >= 5 ADA

**Solution:**
```haskell
-- Ensure wallet has suitable collateral UTxO
-- Atlas should handle automatically, but verify wallet state
utxos <- queryUtxos walletAddress
let collateralCandidates = filter isOnlyAda utxos
```

---

### Issue: Slow compilation times

**Cause**: Large Haskell dependency tree

**Solution:**
```bash
# Use Nix for caching
nix build

# Or increase Cabal jobs
cabal build --jobs=$ncpus

# Cache builds
cabal install --lib  # Install libraries to cache
```

---

### Issue: "Transaction too large"

**Cause**: Transaction exceeds 16 KB limit

**Solution:**
```haskell
-- Use reference scripts
let skeleton = mustHaveRefInput validatorRefScript
            <> mustHaveInput (scriptInput utxo redeemer)
-- Instead of attaching full script each time
```

---

## Resources

### Official Documentation
- **Atlas Website**: https://atlas-app.io
- **Haddock API Docs**: https://haddock.atlas-app.io
- **GitHub Repository**: https://github.com/geniusyield/atlas
- **Example Projects**: https://github.com/geniusyield/atlas-examples

### Learning Resources
- **Genius Academy**: https://academy.geniusyield.co/articles/quick-guide-atlas-the-plutus-application-backend-for-cardano
- **Sports Betting Tutorial**: https://atlas-app.io (walkthrough)
- **Vesting Example**: Video walkthrough available
- **Cardano StackExchange**: #Atlas tag

### Community
- **Support**: Cardano StackExchange (#Atlas tag)
- **GitHub Issues**: https://github.com/geniusyield/atlas/issues
- **Genius Yield**: https://www.geniusyield.co

### Related Frameworks
- **Lucid Evolution**: https://anastasia-labs.github.io/lucid-evolution/
- **Mesh SDK**: https://meshjs.dev/
- **Cardano Developer Portal**: https://developers.cardano.org/

### Production Examples
- **Genius Yield DEX**: https://www.geniusyield.co
- **Genius X Launchpad**: Smart contract launchpad
- **World Mobile Token**: https://worldmobiletoken.com

### Technical References
- **CLB Emulator**: https://www.mlabs.city/blog/testing-dapps-on-cardano-with-clb-emulator
- **Atlas 2.0 Proposal**: https://projectcatalyst.io/funds/12/cardano-use-cases-product/genius-yield-or-atlas-20-pab-improvements-and-advanced-utxo-features
- **Plutus Documentation**: https://plutus.readthedocs.io

---

## Conclusion

Atlas is a **production-ready, powerful framework** for building Cardano dApps in Haskell. It excels in:
- ✅ Type safety and correctness
- ✅ Native Plutus integration
- ✅ Comprehensive testing
- ✅ Complex DeFi protocol development

**Best suited for**: Haskell-proficient teams building sophisticated, backend-heavy applications where type safety and code sharing between on-chain and off-chain layers are critical.

**Considerations**: Steep learning curve, backend-only focus, and Haskell infrastructure requirements make it less suitable for frontend-focused or simple applications.

For teams committed to Haskell and building complex financial protocols, Atlas is the most mature and capable option in the Cardano ecosystem.

---

**Document Version**: 1.0
**Last Updated**: 2025-10-02
**Atlas Version**: 0.14.1
**Verified Against**: Official documentation, GitHub repository, production deployments

---

## Related Documents

- **Decision Point**: `cardano-offchain-integration.md` - Compare Atlas to Lucid/Mesh before choosing
- **Plutus Validators**: Works natively with Plutus (Haskell), less documentation for Aiken integration
- **Security**: `cardano-security-patterns.md` - Same vulnerabilities apply, implement in Haskell
- **Complete Workflow**: `cardano-ai-rules.md` - AI behavior for Atlas development
- **Overview**: `AGENTS.md` - When to choose Atlas vs other frameworks
