
# OpenVote Town Hall App — OpenVote Architecture 

Target device: Android large screen display

Purpose: Central display and verification node for OpenVote elections — receives votes over LoRa mesh, validates them, maintains append-only logs, provides Proof-of-Vote to voters, and ensures transparency and auditability.

Status: Specification / architecture (implementation-ready).

Scope: EU pilot with up to 100 voters, offline verified voting.




## Table of Contents

1. Goals & Design Principles


2. High-level architecture


3. App responsibilities


4. UI / UX — screens and flow


5. LoRa interface


6. Vote validation & logging


7. GPS / Tracker verification


8. Proof-of-Vote confirmation


9. Redundancy & adversarial logging


10. Security & key management


11. Offline export & archival


12. Build / test / QA plan


13. Release / distribution


14. Appendix #1: Payload examples

15. Appendix #2: Registrar key integration 





## 1. Goals & Design Principles

Minimalist, readable UX: clear, large fonts, symbols for validation and errors.

Offline-first: fully functional without internet or cellular connection.

Adversarial verification: dual Town Hall logs (Y/N) to prevent single-point corruption.

Transparent & auditable: append-only logs, live public display, Proof-of-Vote messages to voters.

Resilient & deterministic: identical software across Town Halls; predictable behavior for live and demo events.

Plug-and-play: compatible with Wio Tracker L1 LoRa devices broadcasting OpenVote votes.





## 2. High-level architecture

   :LoRa Mesh
[Tracker L1 Hardware]
   ↕ BLE
[Town Hall Android Screen — OpenVote Town Hall App]
   ├─ Dual-screen redundancy (Y/N teams)
   ├─ LoRa receive/transmit
   ├─ Display: Live totals, rejected votes, Proof-of-Vote
   └─ Offline append-only log

Roles:

Town Hall app: receives votes, validates payloads, enforces electoral roll, logs all activity, sends P2P Proof-of-Vote confirmations.

Redundant setup: two Town Hall screens (Y/N) running identical software independently.





## 3. App responsibilities

Maintain live vote totals and rejected vote counts.

Validate each incoming vote:

Tracker ID registered and trusted

Credential ID registered in electoral roll

Signature valid (ECDSA)

Round matches current election

GPS allowed / policy enforced

One vote per credential per round


Maintain append-only hash chain of votes:

HASH(n) = SHA256(HASH(n-1) + VOTE_DATA)

Detect tampering immediately


Send Proof-of-Vote confirmation back to voter via LoRa message.

Display live totals on public screen with minimal text, blocks of color, and large numbers.

Show rejected votes with categorized counts.

Countdown to election finalization.

Display final certificate with root hash, totals, timestamp, and verification indicators.





## 4. UI / UX — screens and flow

Style: minimalist, tech panel, high-contrast, colorblind-friendly.

Core Screens:

1. Live Voting Screen

Status header: Election ID, round, status (Open/Closed/Finalized), LoRa mesh & Tracker connection indicators

Main vote totals: horizontal bars (YES / NO)

Rejected vote counters: numbered boxes with icons

Telemetry footer: packets/sec, accepted/rejected totals, hash chain status



2. Audit Log Screen (Read-Only)

Scrollable list of votes: short hash, timestamp, Y/N badge, Tracker ID, Credential ID

Top banner: Hash Chain Integrity: VALID / FAILURE



3. Countdown Screen

Full-screen numeric countdown until election finalization

Large font, black background



4. Final Certificate Screen

Photographable display

Election ID, final YES/NO/Total votes, root hash, timestamp

Ledger status: Sealed & Verified




Hidden Overlays:

Registrar Key overlay (activate via long press)

Enables “Close Election” button

Locks or unlocks voting system


Demo / Operator control panel (triple-tap top-left)

Controls for demo: vote speed, bias, error simulation

Reset, force-close election, export log



Navigation:

State-driven, no menus or logins

Screens transition automatically or via Registrar Key commands





## 5. LoRa interface

Listen for incoming vote payloads from any Tracker device.

Broadcast Proof-of-Vote confirmation back to voter.

Enforce one vote per credential per round.

Optionally propagate to redundant Town Hall device for parallel logging.





## 6. Vote validation & logging

Append-only hash chain: each vote hash linked to previous vote

Reject votes failing validation: increment appropriate rejected counter

Hash chain stored locally, exportable to USB / SD

Optional logging of all votes for community audit





## 7. GPS / Tracker verification

Confirm Tracker Node ID is registered and trusted

Check last known GPS fix from payload; enforce time & location window

Rejected GPS votes added to separate counter





## 8. Proof-of-Vote confirmation

Send P2P LoRa message to voter device:

Vote Accepted + Hash

Rejected + Reason if invalid


Allows voter to screenshot as personal receipt





## 9. Redundancy & adversarial logging

Two Town Halls (Y/N) receive same votes independently

Both maintain separate hash chains

Compare logs at end:

Match → legitimate

Divergence → tampering detected


Eliminates need for neutral authority





## 10. Security & key management

Registrar Key (SoloKey) required to open or close election

Voting cannot start without key inserted

Offline append-only logs prevent tampering

Display firmware SHA-256 / version for transparency





## 11. Offline export & archival

Export logs to USB or SD card at end of election

Multiple copies stored by opposing teams and community reps

Includes final root hash for audit





## 12. Build / test / QA plan

Android Studio / Gradle build for tablet

Unit tests: vote validation, hash chain, GPS enforcement, duplicate vote detection

Integration tests: LoRa mesh propagation, Proof-of-Vote confirmations

Demo mode with Figma simulation or preloaded data





## 13. Release / distribution

Deliver APK on USB stick for offline install

Provide SHA-256 for verification

Optional auto-update from USB if newer version





## 14. Appendix #1: Payload examples

Incoming vote payload: compact JSON or binary ≤ 240 bytes

Proof-of-Vote message: hash + status

Rejected vote categories:

Not on roll

Invalid signature

Duplicate credential

GPS policy violation

Untrusted Tracker

Wrong round

Malformed packet




## Appendix #2: Registrar key integration

The Town Hall app  pairs with a Tracker L1 (ESP32-S3) acting as the local registrar device.

A Solokey C is inserted into the Tracker to authorize election operations.

### Features

Registrar Solokey Detection

The Tracker L1 detects insertion/removal of the Registrar Solokey and communicates state changes to the Town Hall app via Bluetooth.

Status indicators now show:

🔐 Registrar Key Required

🔓 Registrar Key Detected


Bluetooth Pass-Through Authentication

When closing or opening an election, the Town Hall app (running on an android display screen) sends signing requests to the Tracker, which forwards them directly to the Solokey via Bluetooth.

The signed authorization packet is returned via Bluetooth.

LoRa Mesh Integration

LoRa vote packets are received by the Tracker and forwarded to the Town Hall app.

Only registrar actions—open, close, finalize—require the Solokey.


### Result

The Town Hall app supports hardware-rooted authorization without adding complexity to the mobile app, relying on the Tracker as a simple USB-HID gateway.



## Implementation checklist (starter)

[ ] Android Studio project skeleton

[ ] LoRa interface & message parser

[ ] Append-only hash chain logging

[ ] Rejected vote counters & display

[ ] Proof-of-Vote transmission

[ ] Public screen visualization: vote totals, rejected votes, countdown

[ ] Final certificate screen generation

[ ] Demo mode / Registrar Key overlay



### Rationale

The Town Hall app acts as the trusted public ledger, verifying each vote, providing real-time transparency, issuing Proof-of-Vote, and maintaining redundant, adversarial logs to ensure incorruptible election results. Together with Tracker firmware and OpenVote VM apps, it completes the OpenVote offline, cryptographically verified voting ecosystem.

