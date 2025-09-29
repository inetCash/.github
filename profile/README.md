# An Introduction to a Digital Cash System

This document outlines an e-cash design **without a blockchain**, **without a DAG** and no transactions by design, where *cash units* are the primitives rather than transactions.
The goal is to provide anonymous, unlinkable digital cash with minimal metadata leakage, yet still maintain verifiable supply and key history.

This document might be slightly outdated as well. Maybe you should update it.

---

## 1. Core Idea: Blind Signatures

Each user generates their own **serial number** for a new note.  
The serial is *blinded* (Schnorr blinding) and sent to a mint for signing.  
The mint never sees the real serial, but the signature passes through the blinding.  
After unblinding, the user has a valid signature on a unique note ID.

- Later, the mint can check if a serial has been spent.  
- The mint **cannot** link the serial to its withdrawal.  
- Result: unlinkable digital cash.

This follows Chaum’s 1982 model, but implemented with elliptic curves (ristretto420 or ristretto255/decaf448).

---

## 2. Spent-set Instead of a Ledger

Instead of a global transaction ledger, the system maintains only a **spent-set**:  
a hash table of serials that have already been redeemed.

- When paying, you reveal `(serial, signature)`.  
- The receiver verifies the signature and queries the mint: *“Is this serial spent?”*  
- If not: the note is valid, and the serial is added to the spent-set.  

No transaction history → no traceability.

---

## 3. Federation of Guardians

The mint is not a single server.  
Instead, multiple **guardians** collectively run the mint:

- Keys are generated via **DKG (Distributed Key Generation)**.  
- Each guardian holds a key share. 
- Currently two aggregated threshold multisignature schemes are considered: 
  - With **FROST threshold Schnorr**, `t of n` guardians can jointly produce a valid signature.  
  - With **BLS threshold signatures**, `t of n` guardians can combine their shares into one compact signature that is constant-size and efficiently verifiable.  
- No single guardian can act alone.

Think of it as a multi-sig bank stamp at protocol level.

---

## 4. Epochs and the Signing Chain

Time is split into **epochs**.  
At the start of each epoch, guardians agree on an *epoch header*:

m_E = H(
“EPOCH” || epoch_id || committee_set_hash ||
rewards_root || balances_root || total_supply
)

- Committees sign `m_E` collectively.  
- These signatures form a **signing chain** (a public book of epoch seals).  
- Anyone can verify that keys and supply are consistent across epochs.

When checking a note, you validate both its blind signature **and** that the key used was part of a valid epoch header.

---

## 5. Inflation and Rewards

The system can implement inflation (e.g. halvings or tail emission).  
Rewards are assigned to guardian balances in a small governance state.  

Guardians can later **redeem rewards into cash** via blind issuance, ensuring even rewards stay unlinkable.

---

## 6. Wallet Design

(NB!!! Wallet design is highly likely to be redesigned!)

A wallet deterministically derives:

- **Serials** (unique note IDs)  
- **Blinding factors**

Typically: BIP39 mnemonic → seed → HKDF → per-denom PRFs.  
All notes are stored locally in a DB (e.g. SQLite).  

Backup modes:  
- Encrypted DB, or  
- **Locked vouchers** signed by guardians, which can be redeemed later.

---

## 7. Why This Matters

- **Privacy by design**: No global ledger, only a spent-set.  
- **Compact**: Notes are just `(serial, Schnorr signature)`.  
- **Decentralized trust**: Guardians co-sign epoch headers with threshold crypto.  
- **Verifiable supply**: Everyone can check total emission via the signing chain.  
- **Rust-friendly**: Entire stack uses curves with mature crates (dalek, frost, ristretto).  

---

## 8. Example Flow

1. Alice wants to withdraw 20.  
2. Wallet generates serials for 2×10 notes.  
3. Serials are blinded → sent to the mint.  
4. Guardians blind-sign, Alice unblinds.  
5. Alice now has two valid 10-unit notes in her DB.  
6. When paying Bob: she sends `(serial, signature)` ×2.  
7. Bob verifies and sends to mint for re-issuance.  
8. Mint checks spent-set, re-issues fresh notes to Bob.  

Result: real digital cash, unlinkable and non-traceable.

---


<!--

**Here are some ideas to get you started:**

🙋‍♀️ A short introduction - what is your organization all about?
🌈 Contribution guidelines - how can the community get involved?
👩‍💻 Useful resources - where can the community find your docs? Is there anything else the community should know?
🍿 Fun facts - what does your team eat for breakfast?
🧙 Remember, you can do mighty things with the power of [Markdown](https://docs.github.com/github/writing-on-github/getting-started-with-writing-and-formatting-on-github/basic-writing-and-formatting-syntax)
-->
