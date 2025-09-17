# Reputation Verification System

This document describes the reputation verification system implemented for the Nanda Payments service, based on the design suggested by Ryan Fox.

## Overview

The reputation verification system allows agents to include encrypted reputation scores in their transactions. A verifier can decrypt these scores and use them to validate whether transactions should be allowed based on the agent's reputation.

## How It Works

### 1. Key Components

- **ReputationVerifier**: Decrypts and validates reputation hashes
- **AgentReputationSigner**: Encrypts reputation scores for agents
- **TransactionReputationService**: Integrates reputation verification with transaction processing
- **Enhanced Transaction Engine**: Processes transactions with reputation verification

### 2. Process Flow

1. **Agent Signs Reputation**: Agent encrypts their reputation score using the verifier's public key
2. **Transaction Submission**: Agent submits transaction with encrypted reputation hash
3. **Verifier Validation**: Verifier decrypts hash and validates reputation score
4. **Transaction Processing**: Transaction is approved/rejected based on reputation requirements
5. **Reputation Update**: Agent's reputation is updated based on transaction outcome

### 3. Security Model

- Agents encrypt reputation data using the verifier's **public key**
- Only the verifier can decrypt using their **private key**
- Reputation data includes timestamp validation (1-hour window)
- Agent DID must match the encrypted data

## API Endpoints

### Enhanced Transaction Creation
```
POST /transactions/with-reputation
```

Request body includes:
```json
{
  "type": "transfer",
  "sourceWalletId": "wal_123",
  "destWalletId": "wal_456",
  "amount": {"value": 1000},
  "reasonCode": "TASK_PAYOUT",
  "idempotencyKey": "unique_key",
  "actor": {"type": "agent", "did": "did:nanda:agent"},
  "reputationHash": "base64_encrypted_reputation_score"
}
```

### Initialize Reputation Service
```
POST /reputation/initialize
```

### Generate Test Keys
```
POST /reputation/generate-keys
```

### Get Reputation Requirements
```
GET /reputation/requirements?transactionType=transfer&amount=1000
```

## Reputation Requirements by Transaction Type

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

## Usage Examples

### 1. Generate Verifier Keys
```typescript
import { ReputationVerifier } from './services/reputationVerifier.js';

const keys = ReputationVerifier.generateKeyPair();
console.log('Private Key:', keys.privateKey);
console.log('Public Key:', keys.publicKey);
```

### 2. Agent Encrypts Reputation Score
```typescript
import { AgentReputationSigner } from './services/reputationVerifier.js';

const reputationScore = {
  agentDid: 'did:nanda:agent',
  score: 85,
  timestamp: new Date().toISOString(),
  source: 'reputation-system'
};

const encryptedHash = AgentReputationSigner.encryptReputationScore(
  reputationScore,
  verifierPublicKey
);
```

### 3. Verifier Decrypts and Validates
```typescript
import { ReputationVerifier } from './services/reputationVerifier.js';

const verifier = new ReputationVerifier({ privateKey, publicKey });
const decryptedScore = await verifier.decryptReputationHash(
  encryptedHash,
  'did:nanda:agent'
);

if (decryptedScore) {
  console.log('Valid reputation score:', decryptedScore.score);
}
```

### 4. Create Transaction with Reputation
```typescript
import { createTransactionWithReputation } from './services/enhancedTransactionEngine.js';

const transaction = await createTransactionWithReputation({
  type: 'transfer',
  sourceWalletId: 'wal_123',
  destWalletId: 'wal_456',
  amountValue: 1000,
  reasonCode: 'TASK_PAYOUT',
  idempotencyKey: 'unique_key',
  actor: { type: 'agent', did: 'did:nanda:agent' },
  reputationHash: encryptedHash
});
```

## Running the Demo

To see the reputation system in action:

```bash
npm run reputation-demo
```

This will demonstrate:
- Key generation
- Reputation encryption/decryption
- Transaction eligibility checking
- Reputation requirements for different transaction types
- Error handling for invalid scenarios

## Integration with Existing System

The reputation system is designed to be backward compatible:

1. **Existing transactions** continue to work without reputation verification
2. **New transactions** can optionally include reputation hashes
3. **System transactions** can skip reputation checks
4. **Agent transactions** are verified if reputation hash is provided

## Database Schema Updates

The Transaction model now includes:
```typescript
links: {
  invoiceId: String,
  settlementId: String,
  reputationJobId: String,
  reputationHash: String  // New field for encrypted reputation score
}
```

## Error Handling

The system provides specific error codes for reputation-related failures:

- `REPUTATION_VERIFICATION_FAILED` (403): Reputation verification failed
- `VALIDATION_ERROR` (400): Invalid request format
- `INTERNAL_ERROR` (500): System error during verification

## Future Enhancements

1. **Reputation Database Integration**: Connect with external reputation systems
2. **Dynamic Requirements**: Adjust reputation requirements based on network conditions
3. **Reputation Caching**: Cache reputation scores to improve performance
4. **Multi-Verifier Support**: Support multiple verifiers for decentralization
5. **Reputation Aggregation**: Combine scores from multiple reputation sources

## Testing

The system includes comprehensive testing scenarios:
- Valid reputation verification
- Invalid agent DID handling
- Corrupted hash detection
- Timestamp validation
- Transaction eligibility checks
- Reputation score updates

## Security Considerations

1. **Key Management**: Verifier private keys must be securely stored
2. **Timestamp Validation**: Prevents replay attacks with old reputation data
3. **Score Validation**: Ensures reputation scores are within valid ranges (0-100)
4. **Agent Verification**: Validates that reputation data matches the claiming agent
5. **Error Logging**: Logs suspicious activities for monitoring

---

This reputation verification system provides a robust foundation for trust-based transaction processing while maintaining backward compatibility with existing functionality.
