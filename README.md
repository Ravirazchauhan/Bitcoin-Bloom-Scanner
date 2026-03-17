Here's what I built — a completely new standalone tool, zero API calls, zero network required after the initial CDN load.

How It Works
The Bloom Filter is the core engine. It's a probabilistic data structure that answers "is this address in my set?" in constant time regardless of how many addresses are loaded. Internally it uses 7 MurmurHash3-derived hash functions over a bit array sized optimally for your address count. Key properties:

No false negatives — if an address is in your list, it will always be detected
Tiny false positive rate — configurable, defaults to 0.1% (shown live in the panel)
For 1,000 addresses the filter uses ~14KB of memory. For 1 million addresses, ~14MB

The Scanner generates keys entirely offline — no API calls, no HTTP, nothing leaves the browser. Each key is run through full secp256k1 → SHA256 → RIPEMD160 → Base58Check derivation to produce both uncompressed and compressed addresses, and both are tested against the filter.

Key Features

◎ Load Sample — instantly loads the Bitcoin Puzzle unsolved addresses + Genesis block + early known addresses to let you test immediately
📂 Load File — drag in any .txt/.csv with one address per line (handles hundreds of thousands)
False positive meter — colour-coded bar: green (<0.5%), amber (<2%), red (high)
Max speed mode — 1ms interval + batch 100 = ~50,000+ keys/second depending on your machine
Hits tab — every BF hit stored with full key details + explorer links for manual balance verification
Export — hits as JSON or CSV, full scan history as CSV
localStorage persistence — sequential scan position and hits survive page refresh
