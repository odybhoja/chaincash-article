# Chaincash: Peer to peer money for everyone

Traditional monetary systems depend on central banks to control liquidity, while conventional cryptocurrencies fix supply to strict algorithmic curves. Both models struggle with localized trade flexibility. Standard decentralized stablecoins lock up immense capital in over-collateralized pools, leaving local economies vulnerable during broader market contractions.

**Chaincash** solves this liquidity bottleneck on the **Ergo** blockchain. By combining smart contract collateralization with sovereign trust networks, Chaincash implements elastic **peer to peer** **money** creation. Instead of relying on centralized institutions or rigid single-asset pools, every participant can issue, transfer, and redeem credit notes backed by local trust and distributed reserves.

---

## 1. The Technical Foundations of Chaincash on Ergo

### Elastic Credit vs. Hard-Collateral Stablecoins

Conventional crypto stablecoins require global reserve pools to back digital tokens. This rigid model restricts trade when base reserve assets drop in value or experience extreme market volatility. In contrast, free banking economics rely on localized credit extension where trust supplements hard collateral.

Chaincash translates this historical free-banking architecture onto the blockchain. It decouples token creation from centralized issuers, allowing market participants to issue custom notes backed by a flexible combination of native assets and personal reputation.

### The eUTXO Model and Endorsement Chains

Chaincash relies on Ergo’s Extended UTXO (eUTXO) architecture to represent notes as unspent transaction outputs carrying state in custom registers.

* **Stateful Registers:** Ergo boxes store vital metadata—such as note face value, issuer public keys, and historical spending chains—inside registers (`R4` through `R9`).
* **Accumulated Endorsement:** When an agent spends or transfers a note, they add their digital signature to its on-chain history.
* **Cumulative Backing:** Every peer in the payment chain enhances the note’s credibility. If an initial issuer defaults, receivers retain redemption rights against previous endorsers.

---

## 2. Core Smart Contract Primitives and Registers

### Reserve Contracts (`reserve.es`)

Reserve contracts manage the underlying physical or crypto collateral that backs issued notes. Written in ErgoScript, the reserve contract acts as an autonomous custodian.

* **Collateral Locking:** Users deposit ERG, SigUSD, or custom Ergo tokens into a reserve box controlled by `reserve.es`.
* **Note Issuance:** The reserve contract verifies that the issuer's public key matches the request and issues a corresponding digital note token.
* **Redemption Handlers:** When a holder presents a note for settlement, the reserve contract automatically releases the specified base assets to the holder while burning or updating the note state.

### Note Contracts (`note.es`) and Authenticated AVL Trees

The note contract governs how digital **money** moves between peers without centralized authorization.

* **AVL-Tree Tracking:** The note contract utilizes Ergo's native AVL trees stored in registers to maintain an immutable, cryptographic record of all historical endorsers.
* **Transfer Validation:** The contract verifies spending chain integrity during transfers (`transferNote`), ensuring that previous signatures remain intact and tamper-proof.
* **Non-Fungible History:** Because every note accumulates a unique list of endorsers and reserve ties, notes carry distinct risk profiles rather than uniform fungibility.

---

## 3. How to Technically Use Chaincash for a Payment Today

Deploying and executing a **peer to peer** payment with Chaincash requires running the official Rust server implementation (`chaincash-rs`) alongside an Ergo node.

### Step 1: Setting Up the `chaincash-rs` Server Node

To participate as a buyer or seller, spin up a local server node that communicates with the Ergo blockchain.

1. **Clone and Compile the Server:**
Fetch the repository and compile the binaries using Cargo:
```bash
git clone https://github.com/ChainCashLabs/chaincash-rs.git
cd chaincash-rs
cargo build --release

```


2. **Launch the Node API:**
Run the executable to start the REST API service (defaulting to port `8080`) and initialize the local SQLite database:
```bash
./target/release/chaincash-rs run

```



### Step 2: Creating Reserves and Issuing Digital Notes

Before making a payment, an issuer must establish a reserve box on **Ergo** and create a signed note.

1. **Fund the Reserve Box:**
Call the `/api/v1/reserve/create` endpoint to deposit ERG as backing collateral:
```bash
curl -X POST http://localhost:8080/api/v1/reserve/create \
  -H "Content-Type: application/json" \
  -d '{
    "amount": 1000000000,
    "nanoErgs": 500000000
  }'

```


2. **Mint the Note Token:**
Issue a digital note against the newly created reserve. The server generates a unique note box on the Ergo blockchain containing your digital signature in the box register.

### Step 3: Configuring Acceptance Predicates and Executing Payments

In a decentralized credit network, sellers evaluate incoming notes before accepting them as **money**. The receiving server runs **client-side acceptance predicates** defined in TOML configuration files.

1. **Define Seller Acceptance Rules (`config/local.toml`):**
Configure your node to automatically accept notes that meet specific trust or collateral requirements:
```toml
[acceptance]
predicates = ["config/predicates/trusted_merchants.toml"]

```


Specify whitelist criteria in `trusted_merchants.toml`:
```toml
type = "whitelist"
kind = "endorser"
agents = [
  "030c8f9c4dc08f3c006fa85a47c9156dedbede000a8b764c6e374fd097e873ba04"
]

```


2. **Evaluate and Process the Payment Transaction:**
When a buyer transfers a note to pay for goods, the merchant's server calls the evaluation API:
```bash
curl -X POST http://localhost:8080/api/v1/acceptance/evaluate \
  -H "Content-Type: application/json" \
  -d '{"noteId": "3b8a1...f09e"}'

```


If the evaluation returns `true`, the merchant signs the transfer transaction (`transferNote`). The transaction updates the note's AVL tree on the Ergo blockchain, appending the merchant’s signature and finalizing the payment.

---

## 4. Settlement, Redemption, and Security Mechanisms

### Executing Multi-Hop Note Redemptions

A note holder can hold or pass the note indefinitely. However, when the final holder requires base assets (such as ERG), they trigger an on-chain redemption:

* **Primary Settlement:** The holder submits the note to the original issuer's reserve contract (`reserve.es`). The contract pays out base ERG directly.
* **Secondary Cascading Settlement:** If the primary reserve lacks liquidity, the contract checks upstream endorsers in the spending chain. The holder can claim collateral from any previously signed endorser’s reserve.

### Managing Risk with Receipt Contracts and Sigma Protocols

Chaincash balances participant safety and network growth through automated cryptographic safeguards:

* **Receipt Contracts (`receipt.es`):** When a secondary endorser pays out a note redemption, the Ergo smart contract issues an on-chain receipt. This receipt gives the endorser a legal and programmatic claim to recover funds from the defaulting issuer.
* **Zero-Knowledge Privacy:** Utilizing Ergo’s Sigma protocols, users can verify trust predicates and endorsement histories off-chain without broadcasting their entire transaction graph to public monitors.
* **Economic Spam Protection:** Smart contract transaction fees and non-zero redemption costs prevent bad actors from flooding the network with junk credit notes.

---

## Building Sovereign Financial Systems Today

**Chaincash** transforms how we think about liquidity, turning passive participants into active credit nodes on the **Ergo** blockchain. By combining stateful eUTXO smart contracts, client-side acceptance rules, and multi-signature endorsement chains, Chaincash creates a resilient framework for **peer to peer** **money**.

Whether you run a local business network, coordinate supply chain settlements, or build decentralized financial tools, Chaincash offers the open-source infrastructure needed to issue sovereign credit without reliance on legacy banks.

