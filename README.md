# Bohemiancoin

> *"Skutečná svoboda vyžaduje pevný řád a osobní odpovědnost."*

A working Rust prototype of the Bohemiancoin protocol — a fixed-state cryptocurrency with a unique **3D geometric consensus mechanism** called Cubic Seal.

```
cargo run --release
```

---

## Why Bohemiancoin?

| | Bitcoin / Ethereum | Bohemiancoin |
|---|---|---|
| Ledger size | Ever-growing (600 GB+) | Fixed **512 MB** Codex |
| Consensus | Hash race (who computes fastest) | **3D geometric precision** (who aims most accurately) |
| Anti-farm | Difficult | Sequential Dependency Chain |
| Transaction proof | Full chain download | **CBOR receipt** in your pocket |
| History | Every node stores everything | Users hold their own receipts |

---

## Run it

**GitHub Codespaces (no install needed — runs in your browser):**
1. Click **Code → Codespaces → Create codespace**
2. Wait ~60 seconds for the environment to start
3. In the terminal type: `cargo run --release`
4. Done.

**Locally (Rust 1.75+):**
```bash
git clone https://github.com/your-org/bohemiancoin
cd bohemiancoin
cargo run --release
```

---

## Demo output

```
╔════════════════════════════════════════════════════════════╗
║        B O H E M I A N C O I N   prototype  v0.2          ║
║        Fixed-State Ledger  ·  Cubic Seal (Proof-of-Geometry)║
║        Ed25519  ·  BLAKE3  ·  3D geometric consensus       ║
╚════════════════════════════════════════════════════════════╝

── GENESIS ────────────────────────────────────────────────────
  ✓  Alex     : 4237a6…b2a2b1   1000 GRS
  ✓  Beatrice : b54c1b…aebcc5    500 GRS
  ✓  Charlie  : 4bb18c…47344f    200 GRS
  ·  Codex    : 64 KB  (1024 slots, 64 B each)
  ·  Signing  : Ed25519  (ring – production-grade)

── VIRBITIRIUM ────────────────────────────────────────────────
  ✓  Alex → Beatrice 100 GRS     admitted  txid: f1a073…
  ✓  Beatrice → Charlie 50 GRS   admitted  txid: ef0be7…
  ✓  Charlie → Alex 300 GRS      rejected: "insufficient available balance"
  ✓  Alex duplicate nonce         rejected: "invalid nonce"

── CUBIC SEAL  –  Geometric consensus ────────────────────────
  Mining [███████████████████████████████████] 100/100
  ✓  Sealed in 99 ms  ·  1 392 618 iterations
  │  Target T  (0x41d3f446, 0x625361ad, 0x0167dcbc)
  │  Point  P  (0x476dd929, 0x07dd5ba8, 0x0504dc55)
  │  10 PoPR Breadcrumb checkpoints

── CRYSTALLISATION ────────────────────────────────────────────
  ✓  Alex: 894 GRS · Beatrice: 549 GRS · Charlie: 251 GRS
  ✓  Hunter Drip: 6 GRS

── VERIFICATION  (Brno-2) ─────────────────────────────────────
  ✓  Accepted · verified in 1 241 µs
  ✓  State root: f05f048f0c7c4a1c ✓
```

---

## Architecture

```
src/
├── main.rs          # Demo: 2 nodes · 3 wallets · 1 block · full verification
├── codex.rs         # 512 MB fixed-state database · 64-byte slots · O(1) access
├── crypto.rs        # BLAKE3 · Ed25519 (ring) · 3D geometry
├── transaction.rs   # Actum (128 B binary) · Manifest (30 txns) · Merkle root
├── cubic_seal.rs    # 3-phase geometric consensus: CPU → Disk → GPU · PoPR verify
├── virbitirium.rs   # Mempool · 15:10:5 selection · Pendens · spam gate
├── block.rs         # BlockHeader · NetworkHeader · Adaptive difficulty
├── node.rs          # Full node: seal · accept · sync · atomic write
└── wallet.rs        # Ed25519 keygen · Actum signing
```

---

## Core concepts

### Codex – fixed 512 MB state

Every node holds an identical 512 MB file. Each account occupies a **64-byte slot** at a deterministic address:

```
offset = BLAKE3(pubkey) mod (512 MB / 64 B)   ← O(1), no search

Slot layout:
[ 0.. 8]  Balance_GRS     u64   main balance
[ 8..16]  Balance_TOL     u64   fractional (1 GRS = 1 000 000 TOL)
[16..24]  Pendens_Amount  u64   locked in Grey Zone
[24..28]  Pendens_Expiry  u32   block# when lock auto-releases
[28..36]  Nonce           u64   replay protection
[36..38]  Status_Flags    u16   Active | Pendens | Frozen | TPM
[38..64]  Reserved        26 B  future use
```

After 4 years (one Epoch = 2 100 000 blocks) the entire Codex is hashed into a 32-byte **Master Hash** stored in the 64 MB Header — then resets. History lives in users' CBOR receipts, not on servers.

### Cubic Seal – Proof-of-Geometry

This is the key innovation. Traditional cryptocurrencies reward whoever computes the most hashes per second — an arms race that favours industrial farms.

**Cubic Seal rewards geometric precision, not raw speed.**

Every validator gets their own unique Target T in a 3D space. The goal is to find a point P that lands as close as possible to T. Two validators with identical hardware will have completely different targets — there is no advantage to pooling resources.

```
Phase 1 · CPU   Derive start point P0 (unique per validator)
                P0 = BLAKE3(prev_block ‖ hunter_addr ‖ hw_id)

Phase 2 · Disk  Walk 10 000 × 4 KB pages through the Codex Labyrinth
                Sequential Dependency Chain:
                addr[i+1] = BLAKE3(data[i] ‖ state[i])
                → reading from RAM produces a different Target T → invalid block

Phase 3 · GPU   Find Nonce s.t. d²(Point_P, Target_T) < R²
                Tiebreaker: smaller d wins (precision, not speed)

Verify  · fast  10 Breadcrumb checkpoints + re-derive P from nonce
                Verification takes < 2 ms
```

**Why this matters:**
- A farm of 1 000 machines does not gain 1 000× advantage — each machine hunts its own unique target
- RAM-disk tricks produce wrong results — the Labyrinth must read real disk data
- The winner is whoever aimed most precisely, not whoever has the most ASICs

### Virbitirium – fair mempool · 15:10:5

Each block picks exactly **30 transactions**:
- **15 slots** — highest fee (market priority)
- **10 slots** — longest waiting (social fairness — patient users are guaranteed inclusion)
- **5 slots**  — system / Smart-Acta

### Grey Zone (Pendens)

Funds are locked in the Codex the moment a transaction enters the Virbitirium. Double-spend is impossible by design — no waiting for confirmations. If a transaction expires, the lock dissolves automatically across all nodes without any central command.

### CBOR receipt – audit in your pocket

After each transaction commits, the wallet holds a tiny `.cbor` file (~160 bytes):
- Transaction hash
- Merkle proof (5 hashes)
- Chronos timestamp
- State root anchor

In year 2126: show your `.cbor`, compare against the Master Hash in any node's 64 MB Header → **100% proof of payment.** The network itself forgets — history lives in users' pockets.

---

## Anti-farm mechanisms

| Mechanism | What it does |
|---|---|
| **Sequential Dep. Chain** | Each jump address depends on the previous disk read — RAM spoofing gives wrong Target T |
| **Codex Salt per-block** | XOR mask changes every block — stale Codex copies give wrong addresses |
| **Proof-of-Physical-Read** | 10 Breadcrumbs prove the walk visited real disk pages |
| **Unique Target per validator** | Pooling resources gives no geometric advantage |
| **TPM Attestation** *(roadmap)* | Hardware-signed identity, impossible to clone |

---

## Currency

| | |
|---|---|
| Main unit | 1 Groš (**GRS**) |
| Sub-unit | 1 Tolar (**TOL**) — 1 GRS = 1 000 000 TOL |
| Total supply | **42 000 000 GRS** (fixed forever) |
| Emission | 3 Epochs × 14 000 000 GRS · each ≈ 4 years |
| Block reward | **6.6 GRS** Drip per block (linear release) |
| Epoch bonus | **140 000 GRS** to Grand Strike winner |

---

## Known limitations (prototype)

Being transparent about what this is and isn't:

- **No tests yet** — first contribution opportunity
- **Codex in RAM** — production needs `memmap2` for real disk I/O (Phase 2 anti-farm properties require actual disk latency)
- **No P2P network** — nodes simulated in one process
- **hw_id = hunter address** — real implementation needs actual hardware fingerprint
- **CBOR simulated** — structure shown, not yet serialised to file

---

## Roadmap

- [ ] Unit tests (`cargo test`)
- [ ] Real disk Codex (`memmap2`)
- [ ] P2P networking (libp2p / tokio)
- [ ] CBOR receipt serialisation (`ciborium`)
- [ ] TPM Attestation (Hardware Salt v2)
- [ ] Master Hash Epoch (Grand Strike / 4-year emission)
- [ ] Sharded Universe (scaling beyond 8M accounts)

---

## Cryptography

| Purpose | Algorithm |
|---|---|
| Signatures | **Ed25519** via `ring` (production-grade) |
| All hashing | **BLAKE3** (faster than SHA-256, natively parallel) |
| Geometric consensus | 3D Euclidean distance² in `u128` (no sqrt needed) |

---

## Contributing

The protocol was designed by a non-programmer. The prototype was built with AI assistance (Claude). Looking for Rust developers who find the geometric consensus idea interesting.

Good first issues:
- Write unit tests for any module
- Replace in-memory Codex with `memmap2`
- Code review and critique

---

## License

MIT © Bohemiancoin Contributors
