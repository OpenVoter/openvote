# HELP WANTED – OpenVote

OpenVote is an open-source project building **offline, cryptographically verified public voting over LoRa mesh networks**.

We are now entering the **first prototype phase** and are actively looking for contributors across software, firmware, hardware, and security.

This is an early-stage project with real-world hardware deployment goals.

---

## Core Objectives (Pilot Phase)

- Offline voting with no internet dependency  
- Hardware-rooted identity using SoloKeys  
- LoRa mesh vote transmission using Meshtastic  
- Public, transparent Town Hall display  
- Proof-of-Vote for each voter  
- Adversarial dual-logging (Y/N Town Halls)  
- EU 868 MHz only for v0.3  

---

## Roles We Are Looking For

### 1. Android Developers (High Priority)
**Skills:**
- Kotlin or Java
- USB-C OTG communication
- UI for low-power, offline apps
- Cryptographic signature verification

**Tasks:**
- Voter app (key detect, sign, send vote)
- Town Hall app (receive, verify, display, log)
- USB-only app distribution
- Proof-of-Vote display and screenshot support

---

### 2. Firmware / Embedded Developers (High Priority)
**Skills:**
- ESP32 / Nordic SoC
- LoRa / SX1262 / GPS
- Meshtastic firmware
- USB serial, power negotiation

**Tasks:**
- Custom OpenVote payload handler
- GPS enforcement logic
- Node ID whitelisting
- Relay mode optimization
- Vote ACK & Proof-of-Vote return messages

---

### 3. Cryptography & Security Engineers
**Skills:**
- ECDSA / Ed25519
- Hardware security keys
- Key registration systems
- Threat modeling

**Tasks:**
- Credential issuance flow
- Signature compression for LoRa
- Proof-of-Vote hash design
- Attack surface review (radio, firmware, supply chain)

---

### 4. Meshtastic Community Contributors
**Skills:**
- Mesh routing
- Channel encryption
- Payload size optimization
- Multi-hop reliability

**Tasks:**
- Private channel models
- Tracker relay behavior
- ACK limitations & multi-hop confirmation
- Packet loss mitigation strategies

---

### 5. Technical Documentation Contributors
**Skills:**
- Markdown
- Diagrams
- Specification structuring

**Tasks:**
- Improve architecture diagrams
- Produce wiring diagrams
- Produce developer setup guides
- Flashing & provisioning guides

---

## Hardware Platforms in Use

- **LoRa Tracker:** Seeed Studio Wio Tracker L1  
- **Authentication Key:** SoloKeys USB-C  
- **Display Node:** Android tablet / large Android screen  
- **Distribution:** USB-only, no app stores  
- **Frequency:** EU 868 MHz only  

---

##  Security Philosophy

- Don’t trust, verify  
- No secrecy-based trust  
- Public supply-chain ceremony (D3)  
- Dual adversarial Town Halls  
- Hardware-rooted identity  
- No centralized arbitrator  

If you enjoy adversarial systems, hardware trust, or censorship-resistant infrastructure — this is your project.

---

##  What Exists Today

- OpenVote public overview
- OpenVote v0.3 technical architecture
- Hardware model defined
- Security model defined
- Voting protocol defined

---

##  What Does NOT Exist Yet

- Android apps
- Custom Meshtastic firmware fork
- Registration tooling
- Public logging tools
- Town Hall display software

This is true **greenfield engineering**.

---

##  How To Get Involved

1. Open a GitHub Issue introducing yourself  
2. Describe what you’d like to work on  
3. Link to past projects if available  
4. We align on a small first task  
5. You stay independent — OpenVote is fully open source  

No NDAs. No gatekeepers. No permission required.

---

##  Important Notes

- This system is designed for **local community voting**, not national elections.
- It is intentionally **offline-first**.
- It is intentionally **politically neutral at the infrastructure level**.
- The project prioritizes:
  - Verifiability over convenience  
  - Transparency over secrecy  
  - Public legitimacy over institutional trust  

---

##  Vision

If voting is the foundation of legitimacy, then **the infrastructure that enables it must itself be publicly verifiable**.

OpenVote is an experiment in whether this is possible in the real world.
