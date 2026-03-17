# ◈ Bitcoin Bloom Scanner

> **v1.0.0** · Offline Bitcoin address lookup using a Bloom filter · No API · No network · Pure in-browser cryptography

A single-file, self-contained HTML tool that generates Bitcoin private keys and checks derived addresses against a locally loaded set using a **probabilistic Bloom filter** — entirely offline, with zero network requests during scanning.

---

## ⚡ Quick Start

1. **Download** `bitcoin_bloom_scanner.html` from the [Releases](../../releases) page
2. **Open locally** — double-click or drag into any modern browser (`file://` protocol)
3. Click **◎ Load Sample** to instantly load the Bitcoin Puzzle + Genesis address set, OR paste your own addresses into the filter panel
4. Click **⚙ Build Filter** to compile the Bloom filter
5. Choose a scan mode and press **▶ Start Scan**

> No installation. No server. No dependencies to install. Everything runs in your browser.

---

## 🔬 What Is a Bloom Filter?

A Bloom filter is a space-efficient probabilistic data structure. Given a set of addresses loaded into it, it can answer the question "is this address in the set?" in **constant time**, regardless of how large the set is.

| Property | Detail |
|----------|--------|
| **No false negatives** | If an address is in your list, it will *always* be detected |
| **Possible false positives** | A small configurable % of non-matching addresses may trigger a hit |
| **Space efficient** | 1,000 addresses → ~14 KB of memory. 1,000,000 → ~14 MB |
| **Instant lookup** | O(1) per address regardless of set size |

This tool uses **7 independent MurmurHash3-derived hash functions** over a bit array. The estimated false positive rate (FPR) is shown live in the filter panel and colour-coded green/amber/red.

> **Important:** Every Bloom filter hit should be manually verified using the blockchain explorer links provided in the Hits tab. A hit means the address *may* be in your set — not that it definitely is.

---

## 📋 Features

### Scan Modes

| Mode | Description |
|------|-------------|
| **Sequential** | Starts at key `#1`, increments by 1. Position saved to `localStorage` — resumes automatically on re-open. |
| **Random** | Cryptographically random 256-bit keys via `window.crypto.getRandomValues`. Can be constrained to a custom range. |
| **Era 2009–13** | Cycles through historically weak entropy patterns: ultra-low keys, Unix timestamp seeds (2009–2013), block-height seeds (0–300k), Randstorm 32-bit and 48-bit ranges. |

### Loading Addresses into the Filter

| Method | How |
|--------|-----|
| **Paste** | Type or paste addresses directly into the text area (one per line, or comma-separated) |
| **Load File** | Click the file button and select a `.txt` or `.csv` file from disk |
| **Load Sample** | Click ◎ to instantly load the Bitcoin Puzzle unsolved addresses + Genesis block + early known addresses |

Accepted address formats: P2PKH (`1…`), P2SH (`3…`), Bech32 (`bc1…`)

### Key Generation & Derivation

All cryptography runs entirely in the browser:

- **secp256k1** elliptic curve via [elliptic.js v6.5.4](https://github.com/indutny/elliptic)
- **SHA-256** via [js-sha256 v0.9.0](https://github.com/emn178/js-sha256)
- **RIPEMD-160** — implemented inline (no dependency)
- **Base58Check** encoding for addresses and WIF private keys
- Both **uncompressed** and **compressed** addresses are derived and tested against the filter for every key

### Performance

| Speed Setting | Interval | Approx. keys/s |
|---------------|----------|-----------------|
| Slow | 500ms | ~2/s |
| Normal | 100ms | ~100/s |
| Fast | 20ms | ~500/s |
| Max + Batch 100 | 1ms | ~5,000–20,000/s* |

*Depends on hardware. Modern laptops sustain 10k–50k keys/s at Max + Batch 100.

### Hits & Export

- **Hits tab** — every Bloom filter match stored with full key details (HEX, DEC, WIF, WIF-C, both addresses) + blockchain explorer links for manual verification
- **Export JSON** — all hits as a structured JSON file
- **Export CSV** — hits as a spreadsheet-ready CSV
- **All Scanned CSV** — full scan history (up to 50,000 most recent records)
- **Save HTML** — saves the entire tool as a standalone HTML file

### Persistence

| `localStorage` key | Contents |
|--------------------|----------|
| `bbs_pos` | Current sequential scan position (hex) |
| `bbs_hits` | JSON array of all Bloom filter hits |

---

## 📐 Bloom Filter Technical Details

```
Optimal bit array size:  m = -(n × ln(p)) / ln(2)²
Optimal hash count:      k = (m/n) × ln(2)

Where:
  n = number of items (addresses loaded)
  p = desired false positive rate (default: 0.001 = 0.1%)
  m = total bits in filter
  k = number of hash functions (clamped to 3–15)
```

**Example for 1,000 addresses at 0.1% FPR:**
- Bit array: ~14,377 bits (~1.8 KB)
- Hash functions: 10
- Any address lookup: 10 bit reads, constant time

**Example for 100,000 addresses at 0.1% FPR:**
- Bit array: ~1.4M bits (~175 KB)
- Hash functions: 10
- Still just 10 bit reads per lookup

---

## ⚙️ Usage Examples

### Example 1 — Test with the Bitcoin Puzzle set

```
1. Open bitcoin_bloom_scanner.html locally
2. Click ◎ Load Sample
3. Click ⚙ Build Filter  (loads 26 addresses, FPR < 0.0001%)
4. Set Mode: SEQ, Speed: Normal, Batch: 10
5. Click ▶ Start Scan
6. Watch the log — the key #1 address hits almost immediately
   (key #1 → 1EHNa6Q4Jz2uvNExL497mE43ikXhwF6kZm, included in sample set)
```

### Example 2 — Scan your own address list

```
1. Create a text file: my_addresses.txt
   One address per line:
     1A1zP1eP5QGefi2DMPTfTL5SLmv7DivfNa
     1BgGZ9tcN4rm9KBzDn7KprQz87SZ26SAMH
     ...

2. Open the tool, click 📂 Load File, select my_addresses.txt
3. Click ⚙ Build Filter
4. Choose mode, start scanning
```

### Example 3 — Era 2009–13 research

```
1. Load your address list and build the filter
2. Click 📅 ERA 2009–13
3. Set Speed: Max, Batch: 100
4. Click ▶ Start Scan
   The scanner cycles through: ultra-low keys, Unix timestamp seeds,
   block-height ranges, and Randstorm 32/48-bit patterns
```

### Example 4 — Large address set (100k+ addresses)

```
1. Prepare a .txt file with one address per line
2. Click 📂 Load File
3. After building the filter, check the FPR display
   If FPR > 1%, consider splitting your list into sessions
4. Use Max speed + Batch 100 for maximum throughput
5. Export hits periodically via the Export tab
```

---

## ⚠️ Legal & Ethical Disclaimer

This tool is provided **for educational and cryptographic research purposes only**.

- The Bitcoin 256-bit keyspace contains ~10⁷⁷ possible keys. Finding a funded wallet by chance is computationally infeasible.
- **You must not** attempt to access, transfer, or claim funds from Bitcoin addresses that do not belong to you. Doing so constitutes theft and/or unauthorised computer access under the laws of most jurisdictions.
- Load only addresses you own, have written permission to audit, or that are from publicly released research datasets (e.g. the Bitcoin Puzzle challenge).
- The authors of this tool accept no liability for misuse.

---

## 🏗️ Architecture

The entire tool is one `.html` file — no build step, no npm, no server.

```
bitcoin_bloom_scanner.html
├── CSS          — terminal aesthetic, amber-on-black, CRT scanline overlay
├── HTML         — filter panel, stats grid, controls, tabs (Scanner/Hits/Export/Info)
└── JavaScript
    ├── BloomFilter class  — MurmurHash3 × 7, Uint8Array bit array
    ├── Crypto             — secp256k1, SHA256, RIPEMD160, Base58Check, WIF
    ├── Key generation     — sequential, random, era modes
    ├── Scan loop          — setInterval, configurable speed + batch size
    └── Export             — JSON, CSV, HTML self-save
```

Two CDN libraries are loaded at runtime:
```html
<script src="https://cdnjs.cloudflare.com/ajax/libs/elliptic/6.5.4/elliptic.min.js"></script>
<script src="https://cdnjs.cloudflare.com/ajax/libs/js-sha256/0.9.0/sha256.min.js"></script>
```

For fully offline use, download these scripts and replace the `<script src="…">` tags with inline `<script>` blocks.

---

## 📋 Changelog

### v1.0.0
- Initial release
- Bloom filter with 7 MurmurHash3 hash functions, configurable FPR
- Sequential, Random, and Era 2009–13 scan modes
- Sample address set (Bitcoin Puzzle + Genesis + early wallets)
- File import (.txt / .csv)
- False positive rate meter (live, colour-coded)
- Hits tab with full key details and explorer links
- Export: JSON, CSV, full scan history CSV
- localStorage persistence for position and hits
- Max speed mode (~5k–20k+ keys/s)

---

## 📄 License

MIT — see [LICENSE](LICENSE) for full terms.
