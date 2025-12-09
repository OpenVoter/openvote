# OpenVote v0.3

Offline Verified Public Voting Architecture

**Status:** Draft – Developer & Manufacturer Review

**Scope:** EU 868 MHz, Pilot scale (5–100 voters)  

**Last updated:** 9-12-25

OpenVote v0.3 is the first complete technical iteration of the project that unifies **hardware, firmware, cryptography, radio networking, public verification, and adversarial governance** into a single coherent voting architecture.

This document is written for:
- Firmware developers
- Android developers
- Hardware manufacturers
- Security researchers

It intentionally avoids political theory and focuses on implementation reality.



## Table of Contents

1. System Purpose & Design Principles  
2. High-Level System Architecture  
3. Identity & Authentication Model  
4. Public Trust & Supply Chain Security (D3)  
5. Vote Message Protocol  
6. Proof-of-Vote System  
7. Transparent Logging & Public Display  
8. Adversarial (Dual Town Hall) Model – No Neutral Arbitration  
9. GPS Locality Enforcement  
10. Android Application Layers  
11. Adversarial Threat Model Summary  
12. Operational Modes & Use Cases  
13. Manufacturing & Developer Integration Targets  
14. What v0.3 Achieves  
15. Scope & Limitations  
16. Next Evolution Steps  



## 1. System Purpose & Design Principles

### 1.1 Purpose

OpenVote is an offline, permissionless, cryptographically authenticated public voting system built on LoRa mesh networks and hardware-based identity.

Its purpose is to enable:
- Local democratic decision-making  
- Offline voting with no internet dependency  
- Public real-time verification  
- Verifiable outcomes without centralized authorities  



### 1.2 Core Design Principles

- Offline-first  
- Hardware-rooted identity  
- Public trust ceremonies  
- Adversarial verification  
- No neutral arbitration  
- Proof-of-Vote for every voter  
- Open-source firmware, software, and protocols  
- Don’t trust, verify  



## 2. High-Level System Architecture

### 2.1 Core Components

| Component | Role | Trusted | Notes |
|----------|------|----------|------|
| Android Phone | User interface | ❌ | No private keys |
| Wio Tracker L1 | LoRa + GPS node | ✅ | Trusted Node ID |
| SoloKey (USB-C) | Voter + Registrar identity | ✅ | Stores private key |
| Town Hall Android Screen | Public display | ❌ | No credentials stored |



### 2.2 Architecture Overview (ASCII)

```text
[Voter Phone]                        [Town Hall Android Screen]
     │                                         |
     │ Bluetooth                               | Bluetooth 
     ▼                                        ▼
[Wio Tracker L1] ◀──── LoRa Mesh ────▶ [Wio Tracker L1]
     ▲                                        ▲
     |                                         |  
     │ USB-C                                   | USB-C
[SoloKey]                                  [SoloKey]
                                                

```


### 2.3 Key Notes

1. **Phone is interface only**; no credentials stored.  
2. **Tracker battery** enables untethered voting.  
3. **SoloKey** is inserted directly into Tracker; only then can a vote be signed or recorded  
4. Town Hall use dual  screens and Tracker nodes for redundancy (Y/N oversight).  



## 3. Voter Flow

1. OpenVote VM App runs on Android phone.  
2. Phone connects to Wio Tracker L1 via Bluetooth.  
3. Voter inserts their SoloKey.into Tracker USB-C port.  
4. App displays current election(s). Voter selects their choice (Y/N).  
5. App signs the vote using SoloKey private key.  
6. Tracker sends signed vote over the LoRa mesh network to Town Hall Tracker.  
7. Voter receives Proof-of-Vote acknowledgment from Town Hall (displayed on phone).  
8. Optional: voter screenshots acknowledgment for personal record.  

**Key Notes:**
- Tracker operates on its own battery, independent of the phone.  
- Phone is only the user interface; no credentials are stored.  
- Only votes from registered SoloKeys are accepted.  



## 4. Town Hall Flow

1. Town Hall Android Screen runs OpenVote Town Hall App.  
2. Town Hall Tracker L1 receives votes via LoRa mesh.  
3. Tracker forwards votes to the Android app via Bluetooth.  
4. Town Hall app verifies:
   - Tracker Node ID  
   - Credential public key registration  
   - GPS locality (if enforced)  
   - Signature validity  
5. Votes are logged in public append-only record.  
6. Proof-of-Vote acknowledgment sent back to originating Tracker.  
7. Dual Town Halls (Y/N teams) independently log and display votes for transparent verification.  

**Key Notes:**
- USB-C on Town Hall Tracker holds Registrar SoloKey to activate elections.  
- Battery-powered Tracker allows flexible placement; redundancy supports multi-hop mesh reliability.  
- No internet or Wi-Fi required.  



## 5. Identity & Authentication Model

### 5.1 Dual-Layer Authentication

1. **Trusted Hardware Layer** – registered Tracker Node IDs
2.  
3. **Personal Credential Layer** – SoloKey private keys, public key registry  

### 5.2 Root Registrar Key

- Controls voter registration, electoral database, Tracker whitelist, election activation  
- Compromise requires full public re-registration  



## 6. Public Trust & Supply Chain Security (D3)

- **Digital Democracy Day (D3)** – public unboxing, firmware flash, first boot  
- **Public Hardware Registry** – GitHub + offline USB backup, Node IDs, firmware hashes  
- SHA-256 verification on all firmware & media  



## 7. Vote Message Protocol

- Compact payload ≤ 64 bytes  
- Includes Election ID, round, Node ID, Credential ID, Y/N vote, timestamp, GPS flag, ECDSA signature  
- Supports multi-hop LoRa mesh, redundancy, and ACKs  



## 8. Proof-of-Vote System

- Town Hall sends P2P acknowledgment with vote hash and ✅ symbol  
- Voter can screenshot or verify against public log  



## 9. Transparent Logging & Public Display

- Live vote stream, running tally, append-only public hash chain  
- Dual Town Halls (Y/N) mirror logs independently  
- Exportable via USB  



## 10. Adversarial (Dual Town Hall) Model – No Neutral Arbitration

- Two independent Town Halls  
- Both receive identical LoRa traffic  
- Independently validate votes  
- Matching hashes = valid outcome  



## 11. GPS Locality Enforcement

- Votes only accepted from Trackers with GPS lock  
- Ensures verifiably local voting 



## 12. Android Application Layers

- **Voter App** – vote submission, proof-of-vote, log lookup  
- **Town Hall App** – vote ingestion, signature verification, live display, export  
- App Distribution: USB-only, SHA-256 verified, offline-first  



## 13. Adversarial Threat Model Summary

- Malicious insiders, fake apps, supply-chain tampering, radio jamming, compromised hardware  
- Security strategy: public trust ceremonies, dual Town Halls, hardware-rooted crypto, proof-of-vote, open verification  



## 14. Operational Modes & Use Cases

- Single-question referenda  
- Multi-round votes  
- Emergency community polling  
- Family/shared device voting  
- Geofenced community decisions  



## 15. Manufacturing & Developer Integration Targets

- **Hardware**: Wio Tracker L1, SoloKeys USB-C  
- **Software**: Tracker firmware, Android Voter + Town Hall Apps, Public Registry tooling  



## 16. Result Finalization & Election Immutability

OpenVote does not rely on a single trusted database for election results. Instead, it produces multiple independent, cryptographically linked public records that together form a tamper-evident and adversarially verified final result.

### 16.1 Cryptographic Vote Integrity

Every vote is individually protected by:

- SoloKey-based ECDSA signature  
- Tracker Node ID binding  
- Election ID and round binding  
- Timestamp and optional GPS locality flag  

Only correctly signed votes from registered hardware and credentials are accepted. Any modification invalidates the signature immediately.



### 16.2 Append-Only Public Vote Log

Each Town Hall maintains a strictly append-only vote ledger.

Each entry is linked as:

````
HASH(n) = SHA256(HASH(n-1) + VOTE_DATA)

````
This creates a continuous cryptographic hash chain where:

- No vote can be edited
- No vote can be deleted
- No vote can be reordered
- Any historical tampering is instantly detectable

This functions as a local, offline, cryptographic ledger without any centralized control or internet dependency.



### 16.3 Dual Town Hall Adversarial Verification

Two independent Town Halls operate simultaneously:

- One operated by the **Y-side**
- One operated by the **N-side**

Both Town Halls:

- Receive the identical LoRa vote stream
- Independently verify signatures
- Independently maintain append-only logs

At finalization:

- If both logs match → the election result is legitimate  
- If logs differ → tampering is mathematically proven  

This removes the need for neutral arbiters and embeds adversarial verification directly into the architecture.



### 16.4 Proof-of-Vote (Individual Verification)

Upon successful vote acceptance:

- The Town Hall transmits a P2P Proof-of-Vote acknowledgment via LoRa
- Includes:
  - Vote hash  
  - Election ID  
  - ✅ confirmation symbol  

Each voter may retain this as a personal receipt and later verify its inclusion in the public log. This enables individual and crowd-sourced auditing.



### 16.5 Live Public Transparency

During voting:

- Votes are displayed in real time on public Town Hall screens
- Running totals and ledger growth are visible to all observers
- No delayed or hidden tabulation phase exists

This prevents:

- Back-room result manipulation  
- Secret tallying  
- Post-hoc “result corrections”  



### 16.6 Offline Archival & Physical Distribution

At the close of voting:

- Final logs are exported via:
  - USB storage  
  - SD cards  

- Multiple identical copies are distributed to:
  - Y-side representatives
  - N-side representatives
  - Community observers

All copies remain cryptographically linked and independently verifiable.



### 16.7 Finalization Condition

An election is considered final and immutable when:

1. Voting is formally closed  
2. Both Town Hall logs cryptographically match  
3. Public final hash is displayed and archived  
4. Offline copies are distributed  

From this point forward, retroactive result manipulation becomes computationally and socially infeasible.



## Closing Statement 

OpenVote makes election results incorruptible by eliminating hidden computation, removing central control, forcing cryptographic identity onto every vote, duplicating the full counting process between opposing sides and giving every voter their own independent proof of inclusion.  

It doesn’t try to make cheating illegal.

It makes cheating self-exposing.



## 17. What v0.3 Achieves

- Offline adversarial voting  
- Public supply-chain trust  
- Proof-of-Vote  
- Family/shared device use  
- Transparent live verification  
- No neutral arbitration  



## 18. Scope & Limitations

- Pilot scale (5–100 voters)  
- EU 868 MHz only  
- Not designed for national elections  
- Offline-only  



## 19. Next Evolution Steps

- Signature compression for LoRa  
- Solar-powered Town Hall hardware 
- Multi-community mesh federation  


