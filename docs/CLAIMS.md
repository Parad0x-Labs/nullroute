# Claims & Reality

What we claim. What we don't. With receipts.

---

## We Claim

### ✓ Batch Settlement Works

Multiple intents settle in single on-chain TX.

**Evidence:**
- TX: [5b1MtoyP1BRVn7SgfQtKCTyHDE1mfFD4k1oC8DtJmjnexrSWaBRvxJ1HgP6GYFYwZmzoVNpfwW2cVuLF1HVo9YJo](https://explorer.solana.com/tx/5b1MtoyP1BRVn7SgfQtKCTyHDE1mfFD4k1oC8DtJmjnexrSWaBRvxJ1HgP6GYFYwZmzoVNpfwW2cVuLF1HVo9YJo?cluster=devnet)
- Instruction: `settle_net_batch`
- Result: Success

### ✓ Replay Protection Works

Duplicate batches rejected.

**Evidence:**
- GlobalConfig tracks `last_batch_id`
- Submitting same batch_id twice fails
- Tested and verified

### ✓ Anonymous Payment Path Exists

Sender deposits to vault. Receiver paid from vault. No direct A→B on-chain.

**Evidence:**
- TX: [32YDUGw5kSsMSJ8KvdAwaAAKxJEA3YLnN8dvjx5DCb6cy49xXPa4NkpAeE5K7y3LDogyiFxSBDBgHcwkhGsxeTvh](https://explorer.solana.com/tx/32YDUGw5kSsMSJ8KvdAwaAAKxJEA3YLnN8dvjx5DCb6cy49xXPa4NkpAeE5K7y3LDogyiFxSBDBgHcwkhGsxeTvh?cluster=devnet)
- Sender and receiver wallets don't touch in TX

### ✓ Merkle Compression Works

N intents → 1 merkle root (32 bytes on-chain).

**Evidence:**
- Root: `0x94164fb6fdbf78ecf19d8fd496e376567fdc82c488bff2e667754777793f9958`
- Verify: [TestLabs](https://labsx402.github.io/test/docs/test.html)

### ✓ Token-2022 Integration Works

PDOX token with transfer fee.

**Evidence:**
- Token: [4ckvALSiB6Hii7iVY9Dt6LRM5i7xocBZ9yr3YGNtVRwF](https://explorer.solana.com/address/4ckvALSiB6Hii7iVY9Dt6LRM5i7xocBZ9yr3YGNtVRwF?cluster=devnet)
- TX: [4LaL3ctzQYWuRGBUDWnL2TuMdeDJh2iEYWB6PgAJh3GzNhvrouKdcBg1ed3Gafd8euXcyibBrnPg4euacouEcjLC](https://explorer.solana.com/tx/4LaL3ctzQYWuRGBUDWnL2TuMdeDJh2iEYWB6PgAJh3GzNhvrouKdcBg1ed3Gafd8euXcyibBrnPg4euacouEcjLC?cluster=devnet)

---

## We Don't Claim

### ✗ "1M TPS guaranteed"

**Reality:** Theoretical ceiling under ideal conditions. Actual throughput depends on:
- Network congestion
- Hardware specs
- Batch composition

Current tested: ~100K intents/batch in <1s on t3.medium.

### ✗ "Zero cost"

**Reality:** Near-zero. Batching amortizes cost across many intents. There's still:
- Base TX fee (~0.000005 SOL)
- Compute units
- Priority fees during congestion

### ✗ "Perfect anonymity"

**Reality:** Statistical mixing, not cryptographic. Anonymity degrades with:
- Low traffic
- Unbalanced flows
- Long-term analysis
- Sophisticated chain analysis

### ✗ "Unruggable"

**Reality:** Rug-resistant. Safeguards include:
- Timelocks on config changes
- Multi-sig capability
- Transparent vault accounting

But with sufficient collusion + timelock bypass, rug is theoretically possible.

### ✗ "Production ready"

**Reality:** Devnet ready. Before mainnet:
- Security audit required
- Stress testing at scale
- Monitoring infrastructure
- Incident response procedures

---

## Theoretical vs Measured

| Metric | Theoretical | Measured (Devnet) |
|--------|-------------|-------------------|
| Max batch size | 10M+ | 1M (CPU-bound) |
| Min latency | <100ms | ~500ms |
| Cost/intent (1M batch) | $0.00000025 | ~$0.000001 |
| Anonymity (PARADOX) | 99.999997% | Calculated, not empirically verified |

---

## Common Misconceptions

### "It's ZK"

No. We use Merkle trees for compression, not ZK proofs for privacy. 

Privacy comes from mixing (batching + ghost wallets + vault indirection), not cryptographic zero-knowledge.

### "It's a mixer like Tornado"

Similar goal, different approach:
- Tornado: fixed denominations, ZK proofs, Ethereum
- Phantom Paradox: arbitrary amounts, Merkle batching, Solana

### "The benchmarks are fake"

Run them yourself:
```bash
solana program show 8jrMsGNM9HwmPU94cotLQCxGu15iW7Mt3WZeggfwvv2x --url devnet
```

Click any TX link in this doc. Verify on Solana Explorer.

### "It's centralized"

The netting engine is centralized (currently). This means:
- Engine can censor/delay (not steal)
- Engine downtime = no new batches
- Users can always withdraw from vault directly

Decentralization is a roadmap item, not a current claim.

---

## Verification Checklist

| Claim | How to Verify |
|-------|---------------|
| Program deployed | `solana program show <ID> --url devnet` |
| Token exists | `spl-token display <MINT> --url devnet` |
| TX succeeded | Click Solana Explorer link |
| Merkle math | Run verification script in TestLabs |
| Netting math | Run verification script in TestLabs |

---

## If Something's Wrong

Open an issue. Provide:
1. What you tested
2. Expected result
3. Actual result
4. TX signature (if applicable)

We'll investigate and update docs if claims are inaccurate.

