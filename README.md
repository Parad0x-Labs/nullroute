# PHANTOM PARADOX

Anonymous payment infrastructure on Solana.

**Status:** Devnet live  
**Program:** `8jrMsGNM9HwmPU94cotLQCxGu15iW7Mt3WZeggfwvv2x`  
**Token:** `4ckvALSiB6Hii7iVY9Dt6LRM5i7xocBZ9yr3YGNtVRwF` (PDOX, Token-2022)

---

## What's Deployed

| Component | Status | Proof |
|-----------|--------|-------|
| On-chain program | ✓ Deployed | [Explorer](https://explorer.solana.com/address/8jrMsGNM9HwmPU94cotLQCxGu15iW7Mt3WZeggfwvv2x?cluster=devnet) |
| GlobalConfig PDA | ✓ Initialized | Program-owned |
| Batch settlement | ✓ Working | [TX](https://explorer.solana.com/tx/5b1MtoyP1BRVn7SgfQtKCTyHDE1mfFD4k1oC8DtJmjnexrSWaBRvxJ1HgP6GYFYwZmzoVNpfwW2cVuLF1HVo9YJo?cluster=devnet) |
| Replay protection | ✓ Working | Tested |
| Anonymous payment | ✓ Working | [TX](https://explorer.solana.com/tx/32YDUGw5kSsMSJ8KvdAwaAAKxJEA3YLnN8dvjx5DCb6cy49xXPa4NkpAeE5K7y3LDogyiFxSBDBgHcwkhGsxeTvh?cluster=devnet) |
| Token transfers | ✓ Working | [TX](https://explorer.solana.com/tx/4LaL3ctzQYWuRGBUDWnL2TuMdeDJh2iEYWB6PgAJh3GzNhvrouKdcBg1ed3Gafd8euXcyibBrnPg4euacouEcjLC?cluster=devnet) |

---

## Architecture

```
User Intent
    │
    ▼
┌─────────────────────────────────┐
│         Netting Engine          │
│  ┌───────────┐  ┌───────────┐   │
│  │ Validate  │─▶│  Queue    │   │
│  └───────────┘  └─────┬─────┘   │
│                       ▼         │
│  ┌───────────────────────────┐  │
│  │    Batch + Compress       │  │
│  │    (Merkle root)          │  │
│  └─────────────┬─────────────┘  │
└────────────────┼────────────────┘
                 ▼
┌─────────────────────────────────┐
│      On-Chain Program           │
│  settle_net_batch()             │
│  - verify batch_id (replay)     │
│  - apply cash deltas            │
│  - emit settlement event        │
└─────────────────────────────────┘
```

---

## Verified Claims

### Batch Settlement
- Multiple intents settle in single on-chain TX
- Merkle root commitment (32 bytes on-chain regardless of batch size)
- Replay protection via monotonic batch_id

**Proof:** [5b1MtoyP1BRVn7SgfQtKCTyHDE1mfFD4k1oC8DtJmjnexrSWaBRvxJ1HgP6GYFYwZmzoVNpfwW2cVuLF1HVo9YJo](https://explorer.solana.com/tx/5b1MtoyP1BRVn7SgfQtKCTyHDE1mfFD4k1oC8DtJmjnexrSWaBRvxJ1HgP6GYFYwZmzoVNpfwW2cVuLF1HVo9YJo?cluster=devnet)

### Anonymous Payments
- Sender deposits to vault
- Receiver paid from vault (different funding source)
- No direct on-chain link A→B

**Proof:** [32YDUGw5kSsMSJ8KvdAwaAAKxJEA3YLnN8dvjx5DCb6cy49xXPa4NkpAeE5K7y3LDogyiFxSBDBgHcwkhGsxeTvh](https://explorer.solana.com/tx/32YDUGw5kSsMSJ8KvdAwaAAKxJEA3YLnN8dvjx5DCb6cy49xXPa4NkpAeE5K7y3LDogyiFxSBDBgHcwkhGsxeTvh?cluster=devnet)

### Token-2022 Integration
- PDOX token with 3% transfer fee
- Fee collected automatically on transfer

**Proof:** [4LaL3ctzQYWuRGBUDWnL2TuMdeDJh2iEYWB6PgAJh3GzNhvrouKdcBg1ed3Gafd8euXcyibBrnPg4euacouEcjLC](https://explorer.solana.com/tx/4LaL3ctzQYWuRGBUDWnL2TuMdeDJh2iEYWB6PgAJh3GzNhvrouKdcBg1ed3Gafd8euXcyibBrnPg4euacouEcjLC?cluster=devnet)

---

## What We Don't Claim

| Claim | Reality |
|-------|---------|
| "1M TPS guaranteed" | Theoretical ceiling. Actual throughput depends on conditions. |
| "Zero cost" | Near-zero. Batching amortizes cost, doesn't eliminate it. |
| "Perfect anonymity" | Statistical mixing. Degrades with low traffic / unbalanced flows. |
| "Unruggable" | Rug-resistant. Requires collusion + timelock bypass. |

---

## Benchmarks (Devnet, t3.medium)

| Batch Size | Dispatch Time | Notes |
|------------|---------------|-------|
| 10K | ~109ms | Optimal for real-time |
| 100K | ~847ms | Sweet spot |
| 1M | ~8.2s | Good for settlement |

*Dispatch = netting + merkle + on-chain. Does not include queue wait.*

---

## Anonymity Model

This is **mixing**, not ZK proofs.

```
Anonymity = 1 - (1 / set_size)

STANDARD: 12 wallets  → 91.6% (1 in 12 chance to trace)
MAX:      1000 wallets → 99.9% (1 in 1000)
PARADOX:  39M wallets  → 99.999997% (1 in 39M)
```

ZK proofs provide cryptographic privacy. We provide statistical privacy through:
- Batch mixing
- Ghost wallet injection
- Rotating vault addresses

Trade-off: faster and cheaper, but not information-theoretic.

---

## Verify Yourself

### Check Program Exists
```bash
solana program show 8jrMsGNM9HwmPU94cotLQCxGu15iW7Mt3WZeggfwvv2x --url devnet
```

### Check Token Exists
```bash
spl-token display 4ckvALSiB6Hii7iVY9Dt6LRM5i7xocBZ9yr3YGNtVRwF --url devnet
```

### Verify TX Signatures
Click any TX link above. Check:
- Program invoked matches our program ID
- Instruction succeeded
- Accounts modified as expected

---

## Build

```bash
anchor build
anchor deploy --provider.cluster devnet
```

Requires:
- Rust 1.75+
- Anchor 0.29+
- Solana CLI 1.17+

---

## License

MIT

---

## Links

- [TestLabs](https://labsx402.github.io/test/docs/test.html) - Verify compression/netting math
- [Architecture Docs](./docs/ARCHITECTURE.md)
- [Benchmark Details](./docs/BENCHMARKS.md)

