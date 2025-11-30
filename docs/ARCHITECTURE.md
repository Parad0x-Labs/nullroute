# Architecture

## Overview

Phantom Paradox is a batch settlement layer for Solana with privacy features.

```
┌─────────────────────────────────────────────────────────────────┐
│                         CLIENT                                  │
│  Submit intent: {from, to, amount, nonce, signature}            │
└──────────────────────────┬──────────────────────────────────────┘
                           │
                           ▼
┌─────────────────────────────────────────────────────────────────┐
│                    NETTING ENGINE (off-chain)                   │
│                                                                 │
│  ┌─────────────┐    ┌─────────────┐    ┌─────────────┐         │
│  │  Validate   │───▶│   Queue     │───▶│   Batch     │         │
│  │  - sig      │    │   - redis   │    │   - net     │         │
│  │  - nonce    │    │   - persist │    │   - merkle  │         │
│  │  - balance  │    │             │    │             │         │
│  └─────────────┘    └─────────────┘    └──────┬──────┘         │
│                                               │                 │
└───────────────────────────────────────────────┼─────────────────┘
                                                │
                                                ▼
┌─────────────────────────────────────────────────────────────────┐
│                    ON-CHAIN PROGRAM (Solana)                    │
│                                                                 │
│  settle_net_batch(batch_id, merkle_root, deltas[])              │
│                                                                 │
│  Checks:                                                        │
│  - batch_id > last_batch_id (replay protection)                 │
│  - sum(deltas) == 0 (conservation)                              │
│  - authority signature valid                                    │
│                                                                 │
│  Effects:                                                       │
│  - Update vault balances                                        │
│  - Increment last_batch_id                                      │
│  - Emit BatchSettled event                                      │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## Components

### 1. On-Chain Program

Anchor program deployed to Solana. Handles:

| Instruction | Purpose |
|-------------|---------|
| `initialize` | Create GlobalConfig PDA |
| `settle_net_batch` | Apply batched settlement |
| `deposit` | User deposits to vault |
| `withdraw` | User withdraws from vault |

**Program ID:** `8jrMsGNM9HwmPU94cotLQCxGu15iW7Mt3WZeggfwvv2x`

### 2. Netting Engine

Off-chain service that:
- Receives user intents
- Validates signatures and balances
- Queues intents for batching
- Computes net positions (graph-based netting)
- Builds Merkle tree
- Submits settlement TX

### 3. Vault System

PDAs that hold user funds:
- Users deposit to vault (not directly to each other)
- Vault pays out to recipients
- Breaks direct on-chain link between sender/receiver

---

## Settlement Flow

```
1. Users submit intents
   └─▶ {from: A, to: B, amount: 100, nonce: 1, sig: ...}

2. Engine validates
   └─▶ Check signature, balance, nonce uniqueness

3. Engine queues (batch window, e.g. 500ms)
   └─▶ Collect N intents

4. Engine computes net positions
   └─▶ A: -100, B: +100, C: -50, D: +50 ...
   └─▶ Many transfers net to fewer settlements

5. Engine builds Merkle tree
   └─▶ hash(intent_1), hash(intent_2), ...
   └─▶ Root = 32 bytes

6. Engine submits to chain
   └─▶ settle_net_batch(batch_id=42, root=0x..., deltas=[...])

7. Program verifies and applies
   └─▶ Check batch_id, apply deltas, emit event
```

---

## Anonymity Layer

### How It Works

```
VISIBLE ON-CHAIN:
- User A deposits 100 to Vault
- User B receives 100 from Vault

NOT VISIBLE:
- That A's deposit funded B's payout
- The intent linking A → B
```

### Mixing Techniques

| Technique | Description |
|-----------|-------------|
| Batch mixing | Many users in same settlement TX |
| Ghost wallets | Fake transactions in batch |
| Rotating vaults | Vault addresses change per epoch |

### Anonymity Set

```
set_size = real_users + ghost_wallets

P(trace) = 1 / set_size
anonymity = 1 - P(trace)
```

**Note:** This is statistical mixing, not ZK proofs. Anonymity degrades with:
- Low traffic
- Unbalanced flows
- Long-term analysis

---

## Merkle Compression

### Why

Instead of N on-chain transactions:
- 1 on-chain TX with Merkle root
- Proof of inclusion for each intent

### Structure

```
                    ┌─────────────┐
                    │    ROOT     │  ◀── 32 bytes on-chain
                    └──────┬──────┘
                           │
              ┌────────────┴────────────┐
              │                         │
         ┌────┴────┐               ┌────┴────┐
         │  H(0,1) │               │  H(2,3) │
         └────┬────┘               └────┬────┘
              │                         │
        ┌─────┴─────┐             ┌─────┴─────┐
        │           │             │           │
    ┌───┴───┐   ┌───┴───┐     ┌───┴───┐   ┌───┴───┐
    │ L(0)  │   │ L(1)  │     │ L(2)  │   │ L(3)  │
    │intent0│   │intent1│     │intent2│   │intent3│
    └───────┘   └───────┘     └───────┘   └───────┘
```

### Cost Savings

| Intents | Traditional | Compressed | Savings |
|---------|-------------|------------|---------|
| 1,000 | 1,000 TXs | 1 TX | 1,000x |
| 100,000 | 100,000 TXs | 1 TX | 100,000x |

---

## Security Model

### Replay Protection
- Monotonic `batch_id` in GlobalConfig
- Each batch must have `id > last_settled_id`
- Prevents re-submission of old batches

### Conservation
- `sum(deltas) == 0` enforced on-chain
- Cannot create or destroy funds

### Authority
- Settlement requires authorized signer
- Multi-sig possible for production

### Nonce Uniqueness
- Per-user nonces tracked off-chain
- Prevents double-spend within batch window

---

## Failure Modes

| Failure | Mitigation |
|---------|------------|
| Engine crash | Intents persisted to DB, recoverable |
| Invalid batch | Program rejects, no state change |
| Duplicate batch | Replay protection rejects |
| Vault underfunded | Pre-check in engine, reject intent |

---

## Limitations

1. **Centralized engine** - Engine operator can censor/delay (not steal)
2. **Statistical privacy** - Not cryptographic, degrades over time
3. **Batch latency** - Users wait for batch window (configurable)
4. **Off-chain dependency** - Requires engine to be running

---

## Future Work

- Decentralized sequencer network
- ZK proofs for stronger privacy guarantees
- Cross-chain settlement
- Mobile SDK

