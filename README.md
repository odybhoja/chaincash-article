
# ChainCash

**Elastic peer-to-peer money, collectively backed by blockchain assets and trust.**

[![License](https://img.shields.io/badge/license-CC0--1.0-blue.svg)](LICENSE)
[![Ergo](https://img.shields.io/badge/built%20on-Ergo-purple.svg)](https://ergoplatform.org)
[![Status](https://img.shields.io/badge/status-prototype-orange.svg)](#status)

ChainCash is a decentralized monetary system that enables flexible money creation by combining trust mechanisms and blockchain-backed assets. Built on the [Ergo blockchain](https://ergoplatform.org), it allows anyone to issue, hold, and transfer digital currency securely—no central authority required.

> **⚠️ Status: Active Research & Prototype**
> ChainCash is prototype software, not a finished wallet-native payment rail. Contracts, server APIs, and refund logic change frequently. Treat this as architectural tooling for builders—verify all details against the latest commits before relying on it in production.

---

## 📋 Table of Contents

- [Why ChainCash?](#why-chaincash)
- [How It Works](#how-it-works)
- [Architecture](#architecture)
- [Installation](#installation)
- [Quick Start](#quick-start)
- [API Reference](#api-reference)
- [Use Cases](#use-cases)
- [Security Model](#security-model)
- [Roadmap](#roadmap)
- [Contributing](#contributing)
- [Resources](#resources)
- [License](#license)

---

## Why ChainCash?

Most cryptocurrencies share two limitations: **rigid supply** and **speculative drift**. Fixed-supply assets resist economic adaptation, while volatility undermines their usefulness as everyday money.

ChainCash takes a different path—one rooted in the historical practice of *free banking*, where privately issued currency competed on trust and redeemability:

- **Elastic supply** — Money expands when issuers commit reserves and contracts when reserves are redeemed. No protocol-hardcoded cap, no monetary committee.
- **Collective backing** — Notes are backed by on-chain collateral (ERG and Ergo-native tokens) plus off-chain trust endorsements.
- **Credible commitments** — Reserve boxes are governed by smart contracts, so issuers can't misappropriate backing. Holders can claim refunds if an issuer defaults.
- **Permissionless issuance** — Anyone with Ergo wallet access can become an issuer. No license, no gatekeeper.

---

## How It Works

ChainCash introduces three core concepts:

### 1. Reserve Boxes

Issuers lock ERG or tokens into Ergo UTXO boxes governed by ChainCash scripts. These boxes prove, publicly and deterministically, that issued notes have real backing.

### 2. Notes

Notes are the money itself. Each note references its reserve and travels peer to peer as a spendable output. Transferring a note is a standard Ergo transaction—no intermediary, no processor, no clearing house.

### 3. Trust Signatures

Notes carry issuer signatures via Ergo's Schnorr scheme. Holders accepting a note implicitly trust the issuer's redeemability. Multi-party endorsement arrangements are supported, letting communities share risk across trusted signers.

```
┌────────────────────────────────────────────────┐
│                ChainCash Layer                  │
│                                                │
│   Issuer Notes ── Reserve Boxes ── Trust       │
│        │               │             │         │
└────────┼───────────────┼─────────────┼─────────┘
         ▼               ▼             ▼
┌────────────────────────────────────────────────┐
│           Ergo Blockchain (eUTXO)              │
│      Deterministic contracts • Sigma ZKPs      │
└────────────────────────────────────────────────┘
```

---

## Installation

### Prerequisites

- **Rust** 1.75+ (`rustup install stable`)
- An **Ergo node** (local) or a public Ergo explorer/API endpoint
- A funded ERG wallet (for reserve locking and transaction fees)

### Build from Source

```bash
git clone https://github.com/BetterMoneyLabs/chaincash-rs.git
cd chaincash-rs
cargo build --release
```

The compiled binary will be available at `target/release/chaincash`.

### Configuration

Copy the example configuration and adjust for your environment:

```bash
cp config.example.toml config.toml
```

Key settings:

```toml
[node]
explorer_url = "https://api.ergoplatform.com"   # or your local node

[wallet]
mnemonic_env = "CHAINCASH_MNEMONIC"            # never hardcode your seed

[server]
host = "127.0.0.1"
port = 9001
```

---

## Quick Start

Spin up a server, create a reserve, issue notes, and send a payment—all in four commands.

### 1. Start the Server

```bash
export CHAINCASH_MNEMONIC="your twelve word seed phrase"
./target/release/chaincash server --config config.toml
```

### 2. Create a Reserve

Lock 1,000 ERG into a ChainCash reserve contract:

```bash
chaincash reserves create --amount 1000
```

After confirmation on-chain, the reserve ID is printed. You'll use it for issuance.

### 3. Issue Notes

Mint 500 units of money against the reserve:

```bash
chaincash notes issue --reserve <RESERVE_ID> --amount 500
```

Notes now sit in your wallet as spendable outputs, ready to circulate.

### 4. Send a Peer-to-Peer Payment

Transfer notes directly to a recipient:

```bash
chaincash notes send --to <RECIPIENT_ADDRESS> --amount 250
```

That's it. The recipient holds ChainCash money backed by verifiable on-chain reserves—settled on Ergo, no third party involved.

---

## API Reference

The server exposes a REST API for programmatic integration. Endpoints include:

| Method | Endpoint | Description |
|--------|----------|-------------|
| `POST` | `/reserves` | Create a new reserve box |
| `GET`  | `/reserves/:id` | Inspect reserve status and backing |
| `POST` | `/notes` | Issue notes against a reserve |
| `POST` | `/notes/transfer` | Send notes to a recipient |
| `GET`  | `/notes/:id` | Retrieve note details and provenance |

See [`docs/server.md`](docs/server.md) for the full specification, request/response schemas, and code samples.

---

## Use Cases

ChainCash's flexibility opens monetary designs impossible under fixed-supply protocols:

- **🏛️ Local currencies** — Communities issue money backed by local business reserves, keeping capital circulating regionally.
- **🌍 Cross-border trade** — Exporters issue notes backed by inventory, cutting FX costs and correspondent-banking delays.
- **🎨 Creator economies** — Artists and musicians monetize future output through personally backed notes.
- **🌾 Microfinance** — Cooperatives extend credit in underserved regions without institutional overhead.
- **🎪 Event currencies** — Festivals issue spendable tokens with exclusive-access perks.

---

## Security Model

ChainCash enforces honesty through cryptography, not regulation:

- **On-chain transparency** — Every reserve, note, and transfer is publicly auditable on Ergo.
- **Contract-enforced refunds** — If an issuer defaults or goes unresponsive, smart contract refund logic lets holders claim proportional shares of the reserve after a timeout.
- **Reputation economics** — Untrustworthy issuers face immediate redemption runs; reliable issuers see their notes circulate at par.

**Known limitations (prototype stage):**

- Some flows require raw Schnorr signature operations rather than standard wallet UX.
- Contract definitions and APIs are subject to change—pin to specific commits in production-like setups.
- Liquidity depends entirely on participating issuers; no guaranteed secondary market exists.

---

## Roadmap

- [ ] Stabilize contract schemas for long-term compatibility
- [ ] Native wallet integration for note transfers
- [ ] Multi-signature trust endorsement UX
- [ ] Secondary-market liquidity tooling
- [ ] Comprehensive testnet deployment guide

---

## Contributing

Contributions are welcome—this project thrives on builders experimenting with monetary primitives.

1. Fork the repository and create a feature branch
2. Run `cargo test` before submitting
3. Follow existing code style (`cargo fmt` / `cargo clippy`)
4. Open a pull request with a clear description

Good first issues are tagged [`help wanted`](https://github.com/BetterMoneyLabs/chaincash-rs/issues).

---

## Resources

- 📖 [Official Documentation](https://docs.ergoplatform.com/uses/chaincash)
- 📄 [ChainCash Whitepaper](https://github.com/BetterMoneyLabs/chaincash/blob/master/docs/whitepaper/chaincash.pdf)
- 🦀 [Scala Reference Implementation](https://github.com/BetterMoneyLabs/chaincash)
- 💬 [Ergo Forum Discussion](https://ergoforum.org/t/chaincash-a-spender-signed-currency-on-ergo)
- 🎥 [Video Overview](https://www.youtube.com/watch?v=NxIlIpO6ZVI)

---

## License

This project is licensed under **CC0-1.0** — released into the public domain. See [LICENSE](LICENSE) for details.

