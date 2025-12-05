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

###

##3. Identity & Authentication Model

##3.1 Dual-Layer Authentication

A. Trusted Hardware Layer

Each Tracker has a unique Node ID

Only registered Trackers may transmit votes



B. Personal Credential Layer

Each voter holds a SoloKey

SoloKey stores private key

Public key registered to voter ID

Family/shared-device use case supported:

Many SoloKeys → One Tracker





###3.2 Root Registrar Key

A designated SoloKey controls:

Voter credential registration

Tracker whitelist

Election activation credentials





##4. Public Trust & Supply Chain Security

###4.1 Digital Democracy Day (D3)

Public ceremony including:

Device unboxing

Firmware flashing

First boot

Hash verification (SHA-256)

Device fingerprint capture

Entry into Public Hardware Registry


###4.2 Public Hardware Registry

GitHub repository (public)

Password-protected USB mirror (offline backup)

Includes:

Tracker Node IDs

Firmware hashes

Activation timestamps






##5. Vote Message Protocol

###5.1 Compact Payload Format (≤ 64 bytes)

Binary-packed fields:

Election ID (compressed)

Voting round

Tracker Node ID

Credential ID (public key hash)

Vote choice (Y/N)

Timestamp (compressed)

GPS flag + coarse location

ECDSA signature (truncated)


###5.2 Mesh Behavior

Multi-hop LoRa routing

Redundant packet broadcast

Town Hall P2P ACK + Proof-of-Vote return





##6. Proof-of-Vote System

Each valid vote triggers a P2P return message to the voter containing:

Election ID

Vote hash

Town Hall Node ID

Timestamp

✅ symbol for visual confirmation


Voter can:

Screenshot confirmation

Independently verify appearance in the public Town Hall log





##7. Transparent Logging & Public Display

Town Hall displays:

Live vote stream

Running tally

Public hash chain


Logs are:

Append-only

Time-ordered

Exportable via USB






##8. No Neutral Arbitration Model

###8.1 Dual Town Hall Architecture

Y Team Town Hall

N Team Town Hall


Both:

Receive identical radio traffic

Independently validate

Independently tally

Independently publish final vote hashes


Results must match mathematically. No central referee exists.




##9. GPS Locality Enforcement

Only Trackers with:

GPS fix

Approved geographic region can submit votes.



Used to:

Prevent remote replay

Enforce local residency

Enable geofenced elections





##10. Android Application (OpenVote App)

###10.1 Voter App Features

SoloKey validation

Election selection

Vote casting

Proof-of-Vote receipt display

Public log lookup


###10.2 Town Hall App Features

Vote ingestion

Signature validation

Live display

USB export

Hash comparison mode


###10.3 Distribution Model

USB-only distribution

Public SHA-256 verification

No app stores

No internet dependency





###11. Adversarial Threat Model Summary

OpenVote assumes:

Malicious insiders

Compromised hardware

Jammed radios

Fake apps

Supply chain tampering


Security strategy:

Public trust ceremonies

Dual adversarial Town Halls

Hardware-rooted cryptography

Proof-of-Vote

Open source app and firmware 

Don't trust, verify.





##12. Operational Modes

Single-question referenda

Multi-round votes

Emergency votes

Family/shared-device voting

Geofenced community decisions





##13. Manufacturing & Developer Integration Targets

###13.1 Hardware Partners

Seeed Studio (Wio Tracker L1)

SoloKeys (USB-C security keys)


###13.2 Software Stack

Meshtastic firmware fork

OpenVote Android app

Town Hall display software

Public registry tooling





###14. What v0.3 Achieves

Openvote v0.3 delivers for the first time:

✅ Unified hardware + crypto + radio + legitimacy
✅ Offline adversarial voting
✅ Public supply chain trust
✅ Proof-of-Vote
✅ No neutral arbitration
✅ Family & shared-device voting
✅ Transparent real-time verification





##15. Scope of v0.3

Target voters: 5–100

Geography: EU 868 MHz

Pilot-scale, community governance

Not for national elections

No internet dependency





##16. Future updates (work in progress)

Hardware enclave signing

Vote compression improvements

Multi-community mesh interlinks

Solar Town Hall nodes

Disaster-resilient voting modes





##Summary Statement##

OpenVote v0.3 is the first complete technical realization of a voting system that removes institutional trust and replaces it with public cryptographic legitimacy, adversarial verification, and physically observable process.
