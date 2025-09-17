# NANDA Points System - Complete Overview

This document provides a comprehensive overview of the NANDA Points payment system, covering both the core payment infrastructure built by Rahul and the reputation verification system added on top.

## What Rahul Built - Core NANDA Points System

### 1. **MongoDB-Based Payment Infrastructure**
Rahul created a robust TypeScript + Express + MongoDB payment system that implements:

- **Double-entry accounting** with integer minor units (scale=3 by default)
- **MongoDB with Mongoose** for data persistence and ACID transactions
- **WebSocket support** for real-time transaction broadcasting
- **Idempotency handling** to prevent duplicate transactions
- **Comprehensive data models** for agents, wallets, transactions, and events

### 2. **Core Data Models**
Rahul designed and implemented four main data models:

#### **Agent Model** (`src/models/Agent.ts`)
- Stores agent information with DID (Decentralized Identifier)
- Includes Facts integration (primaryFactsUrl, vcStatusUrl, factsDigest)
- Payment configuration with wallet linking
- Issuance policies (welcome grants, faucet settings)
- Status management (active, suspended, expired)

#### **Wallet Model** (`src/models/Wallet.ts`)
- Agent-specific wallets with balance tracking
- Daily spending limits and transaction controls
- Support for different wallet types (user, treasury, fee_pool, escrow)
- Balance snapshots and sequence tracking
- Overdraft protection controls

#### **Transaction Model** (`src/models/Transaction.ts`)
- Comprehensive transaction schema supporting 9 transaction types
- Double-entry postings with debit/credit tracking
- Facts integration for transaction parties
- Metadata support for memos, tags, and client information
- Links to invoices, settlements, and other external systems

#### **Event Model** (`src/models/Event.ts`)
- Real-time event broadcasting system
- Transaction lifecycle events (posted, pending, failed)
- Wallet update notifications
- WebSocket integration for live updates

### 3. **Transaction Engine**
Rahul built a sophisticated transaction processing engine (`src/services/transactionEngine.ts`) that:

- **Supports 9 transaction types**: mint, burn, transfer, earn, spend, hold, capture, refund, reversal
- **Implements double-entry bookkeeping** with proper debit/credit postings
- **Enforces business rules** (insufficient funds, wallet status, overdraft limits)
- **Provides atomicity** with MongoDB transactions
- **Handles idempotency** to prevent duplicate processing
- **Generates balance snapshots** for audit trails

### 4. **RESTful API**
Rahul created a complete REST API (`src/api/`) with endpoints for:

- **Agents API**: Create, read, update, delete agents with DID resolution
- **Wallets API**: Wallet creation, balance queries, proof verification
- **Transactions API**: Transaction creation, querying, and status tracking
- **Health API**: System health monitoring

### 5. **Key Features**
- **Facts Integration**: DID-based identity with verifiable credentials
- **WebSocket Broadcasting**: Real-time transaction events
- **Seed Scripts**: Database initialization with sample data
- **Environment Configuration**: Flexible deployment options
- **Type Safety**: Full TypeScript implementation with Zod validation

## What Was Added - Reputation Verification System

### 1. **Reputation-Based Transaction Security**
Added a comprehensive reputation verification system that allows:

- **Encrypted reputation scores** in transactions
- **Public/private key cryptography** for secure reputation verification
- **Transaction eligibility checking** based on reputation requirements
- **Backward compatibility** with existing transactions

### 2. **Core Reputation Components**

#### **ReputationVerifier** (`src/services/reputationVerifier.ts`)
- **RSA encryption/decryption** using OAEP padding with SHA-256
- **Reputation score validation** (0-100 range, timestamp verification)
- **Agent DID verification** to prevent impersonation
- **Key pair generation** for verifier setup

#### **AgentReputationSigner** (in `reputationVerifier.ts`)
- **Reputation score encryption** using verifier's public key
- **Timestamp inclusion** for replay attack prevention
- **Agent DID binding** for identity verification

#### **TransactionReputationService** (`src/services/transactionReputationService.ts`)
- **Transaction eligibility checking** based on reputation scores
- **Dynamic reputation requirements** by transaction type and amount
- **Reputation verification integration** with transaction processing
- **Error handling** for reputation-related failures

#### **Enhanced Transaction Engine** (`src/services/enhancedTransactionEngine.ts`)
- **Reputation verification** before transaction processing
- **Optional reputation checks** (can be skipped for system transactions)
- **Integration** with existing transaction engine
- **Backward compatibility** with non-reputation transactions

### 3. **Reputation Requirements by Transaction Type**

| Transaction Type | Base Min Score | High Amount (>10k) |
|-----------------|----------------|-------------------|
| transfer        | 50             | 70                |
| earn            | 40             | 40                |
| spend           | 60             | 60                |
| mint            | 80             | 80                |
| burn            | 30             | 30                |
| hold            | 50             | 50                |
| capture         | 65             | 65                |
| refund          | 45             | 45                |
| reversal        | 90             | 90                |

### 4. **New API Endpoints**
Added reputation-specific endpoints:

- **`POST /transactions/with-reputation`**: Enhanced transaction creation with reputation verification
- **`POST /reputation/initialize`**: Initialize reputation service with keys
- **`POST /reputation/generate-keys`**: Generate new verifier key pairs
- **`GET /reputation/requirements`**: Get reputation requirements for transaction types

### 5. **Database Schema Updates**
Extended the Transaction model to include:

```typescript
links: {
  invoiceId: String,
  settlementId: String,
  reputationJobId: String,
  reputationHash: String  // New field for encrypted reputation score
}
```

### 6. **Security Features**
- **RSA-OAEP encryption** with SHA-256 hashing
- **Timestamp validation** (1-hour window) to prevent replay attacks
- **Agent DID verification** to prevent impersonation
- **Score range validation** (0-100) for reputation scores
- **Error logging** for suspicious activities

### 7. **Demo and Testing**
Created comprehensive demonstration system:

- **`src/reputationDemo.ts`**: Complete reputation system demonstration
- **Key generation and encryption/decryption** examples
- **Transaction eligibility checking** scenarios
- **Error handling** demonstrations
- **Integration testing** with existing payment system

## System Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                    NANDA Points System                     │
├─────────────────────────────────────────────────────────────┤
│  Core Payment System (Rahul)                              │
│  ├── MongoDB + Mongoose (Data Layer)                      │
│  ├── Express.js (API Layer)                               │
│  ├── WebSocket (Real-time Events)                         │
│  ├── Transaction Engine (Business Logic)                  │
│  └── REST API (Agent, Wallet, Transaction endpoints)      │
├─────────────────────────────────────────────────────────────┤
│  Reputation System (Added Layer)                          │
│  ├── ReputationVerifier (Encryption/Decryption)           │
│  ├── TransactionReputationService (Eligibility Checking)  │
│  ├── Enhanced Transaction Engine (Integration)            │
│  └── Reputation API Endpoints                             │
└─────────────────────────────────────────────────────────────┘
```

## Key Benefits

### **Rahul's Core System Provides:**
- **Reliable payment processing** with double-entry accounting
- **Scalable MongoDB architecture** with proper indexing
- **Real-time event broadcasting** via WebSockets
- **Comprehensive API** for agent and wallet management
- **Type-safe implementation** with full TypeScript support

### **Reputation System Adds:**
- **Trust-based transaction filtering** based on agent reputation
- **Cryptographic security** for reputation verification
- **Flexible reputation requirements** by transaction type
- **Backward compatibility** with existing payment flows
- **Enhanced security** against malicious agents

## Usage Examples

### **Basic Payment (Rahul's System)**
```typescript
// Create a simple transfer transaction
const transaction = await createTransaction({
  type: 'transfer',
  sourceWalletId: 'wal_123',
  destWalletId: 'wal_456',
  amountValue: 1000,
  reasonCode: 'TASK_PAYOUT',
  idempotencyKey: 'unique_key'
});
```

### **Reputation-Enhanced Payment (Added System)**
```typescript
// Create transaction with reputation verification
const transaction = await createTransactionWithReputation({
  type: 'transfer',
  sourceWalletId: 'wal_123',
  destWalletId: 'wal_456',
  amountValue: 1000,
  reasonCode: 'TASK_PAYOUT',
  idempotencyKey: 'unique_key',
  actor: { type: 'agent', did: 'did:nanda:agent' },
  reputationHash: encryptedReputationScore
});
```

## Future Enhancements

1. **Reputation Database Integration**: Connect with external reputation systems
2. **Dynamic Requirements**: Adjust reputation requirements based on network conditions
3. **Multi-Verifier Support**: Support multiple verifiers for decentralization
4. **Reputation Aggregation**: Combine scores from multiple reputation sources
5. **Advanced Analytics**: Reputation trend analysis and reporting

---

This system represents a complete payment infrastructure with integrated reputation verification, providing both the reliability of traditional payment systems and the trust mechanisms needed for decentralized agent marketplaces.
