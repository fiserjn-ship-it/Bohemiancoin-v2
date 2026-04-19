# Bohemiancoin

> *"Skutečná svoboda vyžaduje pevný řád a osobní odpovědnost."*

A working Rust prototype of the Bohemiancoin protocol — a fixed-state cryptocurrency with 3D geometric Proof-of-Work.

```
cargo run --release
```

---

## Why Bohemiancoin?

| | Bitcoin / Ethereum | Bohemiancoin |
|---|---|---|
| Ledger size | Ever-growing (600 GB+) | Fixed **512 MB** Codex |
| PoW | Hash race | **3D geometric** target hunt |
| Anti-ASIC | Difficult | **Sequential Dependency Chain** |
| Transaction proof | Full chain download | **CBOR receipt** in your pocket |
| History | Every node stores everything | Users hold their own receipts |

---

## Run it

**Locally (Rust 1.75+):**
```bash
git clone https://github.com/your-org/bohemiancoin
cd bohemiancoin
cargo run --release
```

**GitHub Codespaces (no install needed):**
1. Click **Code → Codespaces → Create codespace**
2. In the terminal: `cargo run --release`

---

## Demo output

```
╔════════════════════════════════════════════════════════════╗
║        B O H E M I A N C O I N   prototype  v0.2          ║
║        Fixed-State Ledger  ·  Cubic Seal PoW               ║
║        Ed25519  ·  BLAKE3  ·  3D geometry                  ║
╚════════════════════════════════════════════════════════════╝

── GENESIS ────────────────────────────────────────────────────
  ✓  Alex     : 4237a6…b2a2b1   1000 GRS
  ✓  Beatrice : b54c1b…aebcc5    500 GRS
  ✓  Charlie  : 4bb18c…47344f    200 GRS
  ·  Codex    : 64 KB  (1024 slots, 64 B each)
  ·  Signing  : Ed25519  (ring – production-grade)
  ·  Hashing  : BLAKE3  (slot address · Target T · state root)

── VIRBITIRIUM ────────────────────────────────────────────────
  ✓  Alex → Beatrice 100 GRS   admitted  txid: 2ffa19bc…
  ✓  Beatrice → Charlie 50 GRS admitted  txid: 6acb11b2…
  ✓  Charlie → Alex 300 GRS    rejected: "insufficient available balance"
  ✓  Alex duplicate nonce       rejected: "invalid nonce"

── GREY ZONE ──────────────────────────────────────────────────
  ·  Alex: 1000 total | 101 locked | 899 available

── CUBIC SEAL  –  Mining block #0 ────────────────────────────
  Mining [███████████████████████████████████] 100/100
  ✓  Sealed in 99 ms  ·  1 392 618 iterations
  │  Target T  (0x41d3f446, 0x625361ad, 0x0167dcbc)
  │  Point  P  (0x476dd929, 0x07dd5ba8, 0x0504dc55)
  │  10 PoPR Breadcrumb checkpoints

── CRYSTALLISATION ────────────────────────────────────────────
  ✓  Alex: 894 GRS  ·  Beatrice: 549 GRS  ·  Charlie: 251 GRS
  ✓  Hunter Drip: 6 GRS

── VERIFICATION  (Brno-2) ─────────────────────────────────────
  ✓  Accepted  ·  verified in 10 515 µs
  ✓  State root: d8cecc7c5ed85503 ✓
```

---

## Architecture

```
src/
├── main.rs          # Demo: 2 nodes · 3 wallets · 1 block · full verification
├── codex.rs         # 512 MB fixed-state database · 64-byte slots · O(1) access
├── crypto.rs        # BLAKE3 · Ed25519 (ring) · 3D geometry
├── transaction.rs   # Actum (128 B binary) · Manifest (30 txns) · Merkle root
├── cubic_seal.rs    # 3-phase PoW: CPU → Disk (SeqDepChain) → GPU · PoPR verify
├── virbitirium.rs   # Mempool · 15:10:5 selection · Pendens · spam gate
├── block.rs         # BlockHeader · NetworkHeader · Adaptive difficulty
├── node.rs          # Full node: mine · accept · sync · atomic write
└── wallet.rs        # Ed25519 keygen · Actum signing
```

---

## Core concepts

### Codex – fixed 512 MB state

Every node holds an identical 512 MB file. Each account occupies a **64-byte slot** at a deterministic address:

```
offset = BLAKE3(pubkey) mod (512 MB / 64 B)   ← O(1), no search

Slot layout (spec v5):
[ 0.. 8]  Balance_GRS     u64   main balance
[ 8..16]  Balance_TOL     u64   fractional (1 GRS = 1 000 000 TOL)
[16..24]  Pendens_Amount  u64   locked in Grey Zone
[24..28]  Pendens_Expiry  u32   block# when lock auto-releases
[28..36]  Nonce           u64   replay protection
[36..38]  Status_Flags    u16   Active | Pendens | Frozen | TPM
[38..64]  Reserved        26 B  future use
```

After 4 years (one Epoch = 2 100 000 blocks) the entire Codex is hashed into a 32-byte **Master Hash** stored in the 64 MB Header — then reset. History lives in users' CBOR receipts, not on servers.

### Cubic Seal – 3D Proof-of-Work

```
Phase 1 · CPU   P0 = BLAKE3(prev_block ‖ hunter_addr ‖ hw_id)

Phase 2 · Disk  Sequential Dependency Chain (anti-RAM-disk):
                addr[0]   = BLAKE3(P0 ‖ hw_id)
                data[i]   = Codex.read(addr[i], 4 096 B)
                addr[i+1] = BLAKE3(data[i] ‖ state[i])

Phase 3 · GPU   Find Nonce s.t. d²(Point_P, Target_T) < R²
                T = BLAKE3(prev ‖ chronos ‖ hunter ‖ manifest)

Verify  · fast  Re-walk labyrinth, check 10 Breadcrumb checkpoints,
                re-derive P from nonce, confirm d² < R²
```

### Virbitirium – fair mempool · 15:10:5

Each block picks exactly **30 transactions**:
- **15 slots** — highest fee (market priority)
- **10 slots** — longest waiting (social fairness)
- **5 slots**  — system / Smart-Acta

### Grey Zone (Pendens)

Funds are locked in the Codex the moment a transaction enters the Virbitirium — before mining. Double-spend is impossible by design. If a transaction expires (Chronos ticket + expiry blocks), the lock dissolves automatically across all nodes.

### Anti-farm measures

| Mechanism | What it does |
|---|---|
| Sequential Dep. Chain | Each labyrinth jump address depends on previous disk read — RAM spoofing yields wrong Target T |
| Codex Salt per-block | XOR mask changes with every block — stale Codex copies give wrong addresses |
| Proof-of-Physical-Read | 10 Breadcrumbs prove the walk visited real disk pages |
| TPM Attestation *(roadmap)* | Hardware-signed identity, impossible to clone |

### CBOR receipt

After each transaction is committed, the wallet holds a tiny `.cbor` file:
- Transaction hash
- Merkle proof (5 hashes, 160 bytes)
- Chronos timestamp
- State root anchor

In year 2126: produce your `.cbor`, compare against the Master Hash in any node's 64 MB Header → **100% proof of payment**.

---

## Currency

| | |
|---|---|
| Main unit | 1 Groš (**GRS**) |
| Sub-unit | 1 Tolar (**TOL**) — 1 GRS = 1 000 000 TOL |
| Total supply | **42 000 000 GRS** (fixed, mathematically immutable) |
| Emission | 3 Epochs × 14 000 000 GRS · each ≈ 4 years |
| Block reward | **6.6 GRS** Drip per block (linear release) |
| Epoch bonus | **140 000 GRS** to Grand Strike winner |

---

## Roadmap

- [ ] P2P networking — libp2p / tokio gossip
- [ ] Full 512 MB Codex — `memmap2` memory-mapped I/O
- [ ] CBOR receipt serialisation — `ciborium`
- [ ] TPM Attestation — Hardware Salt v2
- [ ] Master Hash Epoch — Grand Strike (4-year emission cycle)
- [ ] Sharded Universe — Expansion protocol (scaling beyond 8M accounts)

---

## Cryptography

| Purpose | Algorithm |
|---|---|
| Signatures | **Ed25519** via `ring` (production-grade) |
| All hashing | **BLAKE3** (faster than SHA-256, natively parallel) |
| PoW geometry | 3D Euclidean distance² in `u128` (no sqrt) |

---

## License

Jan Fiser
