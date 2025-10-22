# nil.js Architecture Documentation

## Table of Contents
1. [Overview](#overview)
2. [Project Structure](#project-structure)
3. [Core Components](#core-components)
4. [Data Flow](#data-flow)
5. [Key Patterns](#key-patterns)
6. [Dependencies](#dependencies)
7. [Build and Distribution](#build-and-distribution)

---

## Overview

### Purpose
nil.js is a TypeScript client library for interacting with the =nil; blockchain network. It provides a comprehensive SDK for building decentralized applications (dApps) that can:
- Connect to =nil; blockchain nodes via JSON-RPC
- Manage wallets (which are smart contracts in =nil;)
- Sign and send transactions
- Deploy and interact with smart contracts
- Handle multi-currency tokens
- Perform synchronous and asynchronous inter-contract calls

### Key Characteristics
- **Type-Safe**: Full TypeScript implementation with strict type checking
- **Modular**: Clear separation of concerns with well-defined modules
- **Isomorphic**: Works in both Node.js and browser environments
- **Standards-Based**: Uses standard cryptographic libraries and protocols
- **Blockchain-Specific**: Tailored for =nil;'s unique architecture (sharding, wallet-as-contract)

---

## Project Structure

```
nil.js/
├── src/                          # Source code
│   ├── clients/                  # Client implementations
│   │   ├── PublicClient.ts       # Read-only blockchain client
│   │   ├── FaucetClient.ts       # Faucet interaction client
│   │   ├── BaseClient.ts         # Base client functionality
│   │   ├── CometaService.ts      # Cometa service integration
│   │   └── types/                # Client-related types
│   │
│   ├── contracts/                # Pre-built contract interfaces
│   │   ├── WalletV1/             # Wallet v1 implementation
│   │   └── Faucet/               # Faucet contract interface
│   │
│   ├── contract-factory/         # Contract deployment utilities
│   │   ├── ContractFactory.ts    # Factory pattern for contracts
│   │   └── contractInteraction.ts
│   │
│   ├── signers/                  # Key management and signing
│   │   ├── LocalECDSAKeySigner.ts # ECDSA signing implementation
│   │   ├── mnemonic.ts           # BIP39 mnemonic support
│   │   ├── privateKey.ts         # Private key utilities
│   │   ├── publicKey.ts          # Public key derivation
│   │   └── types/                # Signer interfaces
│   │
│   ├── encoding/                 # Serialization and encoding
│   │   ├── externalMessage.ts    # External message encoding
│   │   ├── deployPart.ts         # Deployment data encoding
│   │   ├── ssz.ts                # SSZ serialization schemas
│   │   ├── poseidon.ts           # Poseidon hash function
│   │   ├── fromBytes.ts          # Byte conversion utilities
│   │   ├── fromHex.ts            # Hex conversion utilities
│   │   └── toHex.ts              # Hex conversion utilities
│   │
│   ├── transport/                # Network transport layer
│   │   ├── HttpTransport.ts      # HTTP/HTTPS transport
│   │   ├── MockTransport.ts      # Testing transport
│   │   └── types/                # Transport interfaces
│   │
│   ├── rpc/                      # JSON-RPC client
│   │   └── rpcClient.ts          # RPC communication layer
│   │
│   ├── types/                    # Type definitions
│   │   ├── Block.ts              # Block types
│   │   ├── IMessage.ts           # Message types
│   │   ├── IReceipt.ts           # Receipt types
│   │   ├── Token.ts              # Token types
│   │   ├── Hex.ts                # Hex string types
│   │   └── ...
│   │
│   ├── utils/                    # Utility functions
│   │   ├── address.ts            # Address calculation and validation
│   │   ├── assert.ts             # Assertion utilities
│   │   ├── hex.ts                # Hex utilities
│   │   ├── block.ts              # Block utilities
│   │   ├── receipt.ts            # Receipt utilities
│   │   └── refiners.ts           # Data refinement utilities
│   │
│   ├── errors/                   # Error definitions
│   │   ├── BaseError.ts          # Base error class
│   │   ├── block.ts              # Block-related errors
│   │   ├── encoding.ts           # Encoding errors
│   │   └── shardId.ts            # Shard ID errors
│   │
│   ├── wallets/                  # Wallet interfaces
│   │   └── WalletInterface.ts    # Common wallet interface
│   │
│   ├── index.ts                  # Main entry point
│   └── version.ts                # Version information
│
├── test/                         # Test files
│   ├── vitest.config.ts          # Unit test configuration
│   └── vitest.integration.config.ts # Integration test config
│
├── examples/                     # Example usage
│   ├── deployWallet.ts           # Wallet deployment example
│   ├── asyncCall.ts              # Async call example
│   ├── syncCall.ts               # Sync call example
│   ├── tokenMint.ts              # Token minting example
│   └── ...
│
├── rollup/                       # Build configuration
│   ├── rollup.config.js          # Rollup build config
│   └── rollupUtils.js            # Build utilities
│
├── dist/                         # Build output (generated)
│   ├── niljs.cjs                 # CommonJS bundle
│   ├── niljs.esm.js              # ES Module bundle
│   └── niljs.d.ts                # Type definitions
│
├── package.json                  # Package configuration
├── tsconfig.json                 # TypeScript configuration
├── biome.json                    # Biome linter configuration
├── eslint.config.js              # ESLint configuration
└── README.md                     # Documentation
```

---

## Core Components

### 1. Client Layer

#### PublicClient
**Purpose**: Provides read-only access to the blockchain.

**Key Methods**:
```typescript
- getBlockByHash(hash: Hex): Promise<Block>
- getBlockByNumber(blockNumber: Hex | BlockTag): Promise<Block>
- getBalance(address: IAddress): Promise<bigint>
- getCode(address: IAddress): Promise<Uint8Array>
- getMessageByHash(hash: Hex): Promise<ProcessedMessage>
- getMessageReceiptByHash(hash: Hex): Promise<ProcessedReceipt>
- sendRawMessage(message: Uint8Array): Promise<Hex>
- call(callArgs: CallArgs, blockNumberOrHash: Hex | BlockTag)
- chainId(): Promise<number>
- getCurrencies(address: IAddress): Promise<Record<string, bigint>>
```

**Architecture**:
```
User Code → PublicClient → BaseClient → RPC Client → Transport → =nil; Node
```

#### FaucetClient
**Purpose**: Interacts with the faucet contract for funding addresses on devnet.

**Key Methods**:
```typescript
- withdrawTo(address: Hex, amount: bigint): Promise<Hex>
```

### 2. Signer Layer

#### LocalECDSAKeySigner
**Purpose**: Handles cryptographic signing operations using secp256k1.

**Key Methods**:
```typescript
- sign(data: Uint8Array): Promise<Uint8Array>
- getPublicKey(): Uint8Array
- getAddress(shardId: number): Uint8Array
```

**Cryptographic Flow**:
```
Private Key → secp256k1 → Signature (r, s, recovery)
Private Key → secp256k1 → Public Key (33 bytes compressed)
Public Key → Poseidon Hash → Address (with shard ID)
```

**Security Features**:
- Uses `@noble/curves` for secp256k1 implementation
- Supports mnemonic phrase (BIP39) key derivation
- Never exposes private key after initialization
- Proper signature format with recovery byte

### 3. Wallet Layer

#### WalletV1
**Purpose**: Represents a wallet as a smart contract on =nil;.

**Key Characteristics**:
- Wallet is a smart contract, not just a key pair
- Must be deployed before use
- Requires balance before deployment
- Supports both async and sync contract calls

**Key Methods**:
```typescript
- selfDeploy(waitTillConfirmation: boolean): Promise<Uint8Array>
- sendMessage(params: SendMessageParams): Promise<Hex>
- syncSendMessage(params: SendSyncMessageParams): Promise<Hex>
- deployContract(params: DeployParams): Promise<{hash: Hex, address: Hex}>
- setCurrencyName(name: string): Promise<Hex>
- mintCurrency(amount: bigint): Promise<Hex>
- burnCurrency(amount: bigint): Promise<Hex>
```

**Deployment Flow**:
```
1. Generate key pair
2. Calculate wallet address (deterministic from pubkey + salt + shardId)
3. Fund address via faucet
4. Deploy wallet contract
5. Wait for confirmation
```

### 4. Encoding Layer

#### ExternalMessageEnvelope
**Purpose**: Encodes and signs messages for blockchain submission.

**Message Structure**:
```typescript
{
  isDeploy: boolean,      // Deployment flag
  to: Uint8Array,         // Destination address
  chainId: number,        // Chain ID (replay protection)
  seqno: number,          // Sequence number (replay protection)
  data: Uint8Array,       // Message payload
  authData: Uint8Array,   // Signature
  feeCredit: bigint       // Fee credit for processing
}
```

**Encoding Process**:
```
Message Data → SSZ Serialization → Poseidon Hash → Sign with ECDSA → Encoded Message
```

**Key Features**:
- SSZ (Simple Serialize) for deterministic encoding
- Poseidon hash for blockchain-specific hashing
- Separation of signing hash and message hash
- Recovery parameter included in signature

### 5. Transport Layer

#### HttpTransport
**Purpose**: Handles HTTP/HTTPS communication with blockchain nodes.

**Features**:
- Configurable endpoint
- JSON-RPC protocol support
- Isomorphic (works in Node.js and browsers)
- Uses `isomorphic-fetch` for compatibility

**Configuration**:
```typescript
const transport = new HttpTransport({
  endpoint: "https://rpc-endpoint.nil.foundation"
});
```

---

## Data Flow

### Transaction Flow (Async Message)

```
1. User initiates transaction
   ↓
2. WalletV1.sendMessage()
   ↓
3. Encode function call data (viem)
   ↓
4. Get current seqno from blockchain
   ↓
5. Create ExternalMessageEnvelope
   ↓
6. Sign message with LocalECDSAKeySigner
   ↓
7. SSZ serialize signed message
   ↓
8. Calculate Poseidon hash
   ↓
9. Send via PublicClient.sendRawMessage()
   ↓
10. Transport sends JSON-RPC request
    ↓
11. Blockchain processes and returns hash
    ↓
12. User can poll for receipt
```

### Query Flow (Read Operation)

```
1. User requests data
   ↓
2. PublicClient.getBalance() / getCode() / etc.
   ↓
3. Format JSON-RPC request
   ↓
4. Transport sends HTTP request
   ↓
5. Parse JSON-RPC response
   ↓
6. Convert types (hex to bigint, etc.)
   ↓
7. Return typed data to user
```

### Wallet Deployment Flow

```
1. Generate private key (or from mnemonic)
   ↓
2. Derive public key
   ↓
3. Calculate wallet address (pubkey + salt + shardId)
   ↓
4. Fund address via Faucet
   ↓
5. Prepare deployment data (bytecode + args)
   ↓
6. Create external deployment message
   ↓
7. Sign and send deployment message
   ↓
8. Poll for deployment confirmation
   ↓
9. Wallet ready for use
```

---

## Key Patterns

### 1. Builder Pattern
Used for message construction:
```typescript
const message = new ExternalMessageEnvelope({
  isDeploy: true,
  to: address,
  chainId: 1,
  seqno: 0,
  data: encodedData,
  authData: new Uint8Array(0)
});
```

### 2. Factory Pattern
Used for contract deployment:
```typescript
const factory = new ContractFactory({
  client,
  signer,
  wallet
});
```

### 3. Strategy Pattern
Transport layer allows different strategies:
```typescript
// HTTP Transport
const httpTransport = new HttpTransport({ endpoint });

// Mock Transport (for testing)
const mockTransport = new MockTransport({ responses });
```

### 4. Decorator Pattern
Client methods wrap RPC calls with type conversion and error handling.

### 5. Type Safety Pattern
Extensive use of TypeScript generics and branded types:
```typescript
type Hex = `0x${string}`;
type IAddress = `0x${string}`;
type IPrivateKey = `0x${string}`;
```

---

## Dependencies

### Production Dependencies

#### Cryptography
- **@noble/curves** (v1.4.0): secp256k1 cryptography
- **poseidon-lite** (v0.3.0): Poseidon hash function
- **@scure/bip39** (v1.3.0): BIP39 mnemonic support

#### Blockchain
- **viem** (v2.16.3): Ethereum utilities (ABI encoding, etc.)
- **abitype** (v1.0.2): ABI type definitions

#### Serialization
- **@chainsafe/ssz** (v0.16.0): SSZ serialization
- **@chainsafe/persistent-merkle-tree** (v0.7.2): Merkle tree for SSZ

#### RPC
- **@open-rpc/client-js** (v1.8.1): JSON-RPC client
- **isomorphic-fetch** (v3.0.0): Universal fetch API

#### Smart Contracts
- **@nilfoundation/smart-contracts** (v0.3.0): Pre-compiled contracts

#### Utilities
- **tiny-invariant** (v1.3.3): Runtime assertions
- **ts-essentials** (v10.0.2): TypeScript utilities
- **zod** (v3.23.8): Schema validation
- **events** (v3.3.0): Event emitter

### Development Dependencies

#### Build Tools
- **rollup** (v4.17.2): Module bundler
- **rollup-plugin-esbuild** (v6.1.1): Fast transpilation
- **rollup-plugin-dts** (v6.1.0): Type definition bundling
- **@rollup/plugin-node-resolve** (v15.2.3): Module resolution
- **@rollup/plugin-json** (v6.1.0): JSON importing

#### Testing
- **vitest** (v1.6.0): Test framework
- **@vitest/coverage-v8** (v1.6.0): Code coverage

#### Linting
- **@biomejs/biome**: Fast linter and formatter
- **eslint** (v9.13.0): Code linting
- **eslint-plugin-jsdoc** (v50.4.3): JSDoc linting

#### TypeScript
- **typescript** (v5.6.2): TypeScript compiler
- **typescript-eslint** (v8.12.2): TypeScript ESLint rules

---

## Build and Distribution

### Build Process

```bash
npm run build
```

**Steps**:
1. Clean `dist/` directory with rimraf
2. Rollup bundles source code:
   - **CJS bundle**: `dist/niljs.cjs` (CommonJS for Node.js)
   - **ESM bundle**: `dist/niljs.esm.js` (ES Modules for modern environments)
   - **Type definitions**: `dist/niljs.d.ts` (TypeScript declarations)
3. esbuild transpiles TypeScript to JavaScript
4. Bundle size reported by rollup-plugin-filesize

### Module Exports

```json
{
  "main": "dist/niljs.cjs",           // CommonJS entry
  "module": "dist/niljs.esm.js",      // ES Module entry
  "types": "dist/niljs.d.ts",         // TypeScript types
  "exports": {
    ".": {
      "types": "./dist/niljs.d.ts",
      "import": "./dist/niljs.esm.js",
      "require": "./dist/niljs.cjs"
    }
  }
}
```

### Distribution

**Package Name**: `@nilfoundation/niljs`  
**Registry**: npm (https://registry.npmjs.org/)  
**Scope**: `@nilfoundation`  
**Access**: Public

### Version Management

Current version: **0.21.0**

Follows semantic versioning (SemVer):
- Major: Breaking changes
- Minor: New features (backward compatible)
- Patch: Bug fixes

---

## =nil; Blockchain Specifics

### Sharding Architecture
=nil; uses a sharded architecture:
- **Main Shard**: Coordinates execution shards
- **Execution Shards**: Process transactions and smart contracts
- Each wallet/contract belongs to a specific shard
- Cross-shard communication is async by default

### Wallet as Smart Contract
Unlike Ethereum, wallets in =nil; are smart contracts:
- Must be deployed before use
- Have bytecode and storage
- Can implement custom logic
- Address is deterministic from pubkey, salt, and shard ID

### Message Types
- **External Messages**: From users/dApps to blockchain
- **Internal Messages**: Between smart contracts
- **Async Messages**: Cross-shard or within same shard
- **Sync Messages**: Only within same shard (immediate execution)

### Multi-Currency Support
- Each contract can own custom currencies
- Currency ownership by contract address
- Minting and burning supported
- Bounce mechanism for failed transfers

### Fee Model
- **Fee Credit**: Pre-paid fee for message processing
- Separate from message value
- Can be refunded to specified address
- Prevents denial-of-service attacks

---

## Testing Strategy

### Unit Tests
- Located alongside source files (*.test.ts)
- Test individual functions and classes
- Mock external dependencies

### Integration Tests
- Test full workflows
- Use actual blockchain connections (devnet)
- Test contract deployment and interaction

### Test Configuration
- **Vitest** for fast test execution
- Coverage reporting with V8
- Separate configs for unit and integration tests

---

## API Surface

### Main Exports

```typescript
// Clients
export { PublicClient, FaucetClient, CometaService };

// Signers
export { LocalECDSAKeySigner };
export { generateRandomPrivateKey, generateRandomKeyPair };
export { privateKeyFromPhrase };

// Contracts
export { WalletV1, Faucet };

// Contract Factory
export { ContractFactory };

// Encoding
export { 
  bytesToHex, 
  hexToBytes, 
  toHex,
  externalMessageEncode,
  externalDeploymentMessage 
};

// Transport
export { HttpTransport, MockTransport };

// Types
export type { 
  Hex, 
  IAddress, 
  ISigner, 
  Block,
  IReceipt,
  ProcessedMessage,
  Token
};

// Errors
export { 
  BaseError,
  BlockNotFoundError,
  InvalidShardIdError 
};

// Utils
export { 
  calculateAddress,
  getShardIdFromAddress,
  waitTillCompleted
};
```

---

## Performance Considerations

### Optimizations
1. **Lazy Loading**: Modules loaded on demand
2. **Tree Shaking**: Unused code eliminated in build
3. **Caching**: Chain ID cached in PublicClient
4. **Efficient Serialization**: SSZ for compact encoding
5. **Async Operations**: Non-blocking I/O throughout

### Bundle Sizes
- CJS bundle: ~XXX KB (varies)
- ESM bundle: ~XXX KB (varies)
- Type definitions: ~XXX KB (varies)

---

## Extension Points

### Custom Transports
Implement `ITransport` interface:
```typescript
interface ITransport {
  request<T>(request: {
    method: string;
    params: unknown[];
  }): Promise<T>;
}
```

### Custom Signers
Implement `ISigner` interface:
```typescript
interface ISigner {
  sign(data: Uint8Array): Promise<Uint8Array>;
  getPublicKey(): Uint8Array;
  getAddress(shardId: number): Uint8Array;
}
```

### Custom Contracts
Extend `WalletInterface` or create new contract classes following the pattern in `WalletV1`.

---

## Future Considerations

### Potential Enhancements
1. **Hardware Wallet Support**: Integration with Ledger/Trezor
2. **Multi-Signature Wallets**: Support for multi-sig wallet contracts
3. **Event Subscriptions**: WebSocket support for real-time events
4. **Advanced Key Derivation**: BIP32/BIP44 hierarchical deterministic wallets
5. **Gas Estimation**: More sophisticated gas estimation
6. **Batching**: Batch multiple RPC requests
7. **Retry Logic**: Automatic retry with exponential backoff
8. **State Management**: Integration with state management libraries

---

## Conclusion

nil.js provides a well-architected, type-safe SDK for =nil; blockchain interaction. Its modular design, comprehensive type system, and clear separation of concerns make it suitable for building robust dApps while maintaining security and developer experience.

The architecture follows established patterns from Ethereum tooling (like ethers.js and viem) while adapting to =nil;'s unique features such as sharding, wallet-as-contract, and multi-currency support.
