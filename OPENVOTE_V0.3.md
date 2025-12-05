# OpenVote v0.3

Offline Verified Public Voting Architecture

**Status:** Draft – Developer & Manufacturer Review

**Scope:** EU 868 MHz, Pilot scale (5–100 voters)  

**Last updated:** 4-12-25

OpenVote v0.3 is the first complete technical iteration of the project that unifies **hardware, firmware, cryptography, radio networking, public verification, and adversarial governance** into a single coherent voting architecture.

This document is written for:
- Firmware developers
- Android developers
- Hardware manufacturers
- Security researchers

It intentionally avoids political theory and focuses on implementation reality.

---

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

---

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
| SoloKey (USB-C) | Voter identity | ✅ | Stores private key |
| Town Hall Android Screen | Public display | ❌ | No credentials stored |



### 2.2 Architecture Overview (ASCII)

```text
[Voter Phone]
     │
     │ Bluetooth 
     ▼
[Wio Tracker L1] ◀──── LoRa Mesh ────▶ [Town Hall Tracker]
     ▲                                         │
     │ USB-C                                   ▼
[SoloKey]                             [Town Hall Android Screen]

```


### 2.3 Key Notes

1. **Phone is interface only**; no credentials stored.  
2. **Tracker battery** enables untethered voting.  
3. **SoloKey** is inserted directly into Tracker; only then can a vote be signed.  
4. Bluetooth replaces USB link from phone to Tracker; simplifies deployment.  
5. Town Hall may use **dual Tracker nodes** for redundancy (Y/N oversight).  

---

## 3. Voter Flow

1. OpenVote Voter App runs on Android phone.  
2. Phone connects to Wio Tracker L1 via **Bluetooth**.  
3. Voter inserts their **SoloKey** into Tracker USB-C port.  
4. App displays current election(s). Voter selects their choice (Y/N).  
5. Tracker signs the vote using **SoloKey private key**.  
6. Tracker sends signed vote over the **LoRa mesh network** to Town Hall node(s).  
7. Voter receives **Proof-of-Vote** acknowledgment from Town Hall (displayed on phone).  
8. Optional: voter screenshots acknowledgment for personal record.  

**Key Notes:**
- Tracker operates **on its own battery**, independent of the phone.  
- Phone is only the user interface; no credentials are stored.  
- Only votes from registered Tracker IDs + SoloKeys are accepted.  

---

## 4. Town Hall Flow (Updated)

1. Town Hall Android Screen runs OpenVote Town Hall App.  
2. Town Hall Tracker L1 receives votes via **LoRa mesh**.  
3. Tracker forwards votes to the Android app via **Bluetooth**.  
4. Town Hall app verifies:
   - Tracker Node ID  
   - Credential public key registration  
   - GPS locality (if enforced)  
   - Signature validity  
5. Votes are logged in **public append-only record**.  
6. **Proof-of-Vote acknowledgment** sent back to originating Tracker.  
7. Dual Town Halls (Y/N teams) independently log and display votes for **transparent verification**.  

**Key Notes:**
- USB-C on Town Hall Tracker may optionally hold **Registrar SoloKey** to activate elections.  
- Battery-powered Tracker allows flexible placement; redundancy supports multi-hop mesh reliability.  
- No internet or Wi-Fi required.  

---

## 5. Identity & Authentication Model

### 5.1 Dual-Layer Authentication

1. **Trusted Hardware Layer** – registered Tracker Node IDs  
2. **Personal Credential Layer** – SoloKey private keys, public key registry  

### 5.2 Root Registrar Key

- Controls voter registration, Tracker whitelist, election activation  
- Compromise requires full public re-registration  

---

## 6. Public Trust & Supply Chain Security (D3)

- **Digital Democracy Day (D3)** – public unboxing, firmware flash, first boot  
- **Public Hardware Registry** – GitHub + offline USB backup, Node IDs, firmware hashes  
- SHA-256 verification on all firmware & media  

---

## 7. Vote Message Protocol

- Compact payload ≤ 64 bytes  
- Includes Election ID, round, Node ID, Credential ID, Y/N vote, timestamp, GPS flag, ECDSA signature  
- Supports multi-hop LoRa mesh, redundancy, and ACKs  

---

## 8. Proof-of-Vote System

- Town Hall sends P2P acknowledgment with vote hash and ✅ symbol  
- Voter can screenshot or verify against public log  

---

## 9. Transparent Logging & Public Display

- Live vote stream, running tally, append-only public hash chain  
- Dual Town Halls (Y/N) mirror logs independently  
- Exportable via USB  

---

## 10. Adversarial (Dual Town Hall) Model – No Neutral Arbitration

- Two independent Town Halls  
- Both receive identical LoRa traffic  
- Independently validate votes  
- Matching hashes = valid outcome  

---

## 11. GPS Locality Enforcement

- Votes only accepted from Trackers with GPS lock  
- Ensures **verifiably local voting**  

---

## 12. Android Application Layers

- **Voter App** – vote submission, proof-of-vote, log lookup  
- **Town Hall App** – vote ingestion, signature verification, live display, export  
- Distribution: USB-only, SHA-256 verified, offline-first  

---

## 13. Adversarial Threat Model Summary

- Malicious insiders, fake apps, supply-chain tampering, radio jamming, compromised hardware  
- Security strategy: public trust ceremonies, dual Town Halls, hardware-rooted crypto, proof-of-vote, open verification  

---

## 14. Operational Modes & Use Cases

- Single-question referenda  
- Multi-round votes  
- Emergency community polling  
- Family/shared device voting  
- Geofenced community decisions  

---

## 15. Manufacturing & Developer Integration Targets

- **Hardware**: Wio Tracker L1, SoloKeys USB-C  
- **Software**: Meshtastic firmware fork, Android Voter + Town Hall Apps, Public Registry tooling  

---

## 16. What v0.3 Achieves

- Offline adversarial voting  
- Public supply-chain trust  
- Proof-of-Vote  
- Family/shared device use  
- Transparent live verification  
- No neutral arbitration  

---

## 17. Scope & Limitations

- Pilot scale (5–100 voters)  
- EU 868 MHz only  
- Not designed for national elections  
- Offline-only  

---

## 18. Next Evolution Steps

- Enclave-based signing  
- Signature compression for LoRa  
- Solar-powered Town Hall nodes  
- Disaster-resilient voting  
- Multi-community mesh federation  

---

**End of OpenVote v0.3**
