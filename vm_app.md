
# OpenVote VM App — OpenVote Architecture

Target device: Android phone (v7+ recommended)

Purpose: Minimalist voting machine app for OpenVote — constructs votes, handles SoloKey authentication, GPS policy, LoRa transmission, displays Proof-of-Vote, interfaces with Town Hall, supports offline operation.

Status: Specification / architecture (implementation-ready).

Scope: EU pilot with up to 100 voters, offline verified voting.


Table of contents

1. Goals & Design Principles


2. High-level architecture


3. App responsibilities (what it MUST do)


4. UI / UX — screens and flow


5. BLE interface with Tracker firmware


6. Vote payload composition & signing


7. GPS policy & validation


8. Vote rate limiting & enforcement


9. Proof-of-Vote confirmation


10. Logging, telemetry & offline storage


11. Security & key management


12. Build / test / QA plan


13. Release / distribution (USB sticks / app bundle)


14. Appendix #1: Example payloads & JSON messages
    
    
15. Appendix #2: Voter key integration




## 1. Goals & Design Principles

Minimalist UX: clear, readable, large fonts; focus on verified voting, not decoration.

Offline-first: no internet or cell dependency; fully functional using Bluetooth + LoRa.

Separation of concerns: app handles all voting logic, verification, electoral roll checks, timestamping, GPS policy, and Proof-of-Vote; firmware handles hardware access only.

Transparency: every step auditable; user can see vote acceptance / rejection and save a screenshot as receipt.

Resilient & deterministic: predictable behavior for demos and real elections; enforce one vote per round; reject duplicates.

Plug-and-play: compatible with any Tracker with OpenVote firmware + SoloKey C.





## 2. High-level architecture

[Voter Android Phone — OpenVote VM App]
   ↕ BLE
[Wio Tracker L1 — OpenVote Firmware]
   |
   ├─ USB-C <-> [SoloKey (U2F/FIDO HID)]
   ├─ GNSS
   └─ LoRa radio

Roles:

App: constructs and validates votes, enforces rules, interacts with Tracker, displays Proof-of-Vote, optionally stores offline logs.

Firmware: exposes hardware, signs challenge with SoloKey, transmits via LoRa.




## 3. App responsibilities

Detect Tracker over Bluetooth, handle pairing manually.

Detect SoloKey insertion/removal; require presence before allowing vote.

Prompt user to insert SoloKey and display confirmation (Solokey detected).

Compose vote payload:

Election ID

Voting round

Node ID (Tracker)

Credential ID (SoloKey public key fingerprint)

Vote choice Y/N

Timestamp

GPS flag (if applicable)


Send challenge to SoloKey via firmware; receive ECDSA signature.

Send payload to firmware for LoRa transmission.

Enforce vote rate limiting: one vote per round per credential; block duplicates.

Display Proof-of-Vote upon Town Hall confirmation: Vote Accepted + Hash or Rejected + Reason.

Reset app state when SoloKey removed to allow multiple users on same device.

Verify firmware SHA-256 and display for transparency.

Store offline logs for auditing; optionally export via USB.





## 4. UI / UX — screens and flow

Style: minimalist, high-contrast, large text, symbols for errors.

Screens:

1. Tracker & Key detection

Prompt: “Insert SoloKey”

Status: “SoloKey detected” or “No key”



2. Vote screen (Y/N)

Buttons for YES / NO

Disable until SoloKey present

Show last known GPS / policy indicator



3. Vote confirmation / Proof-of-Vote

Display: Vote Accepted + Hash or Rejected + Reason

Reminder: “Remove SoloKey”



4. Offline log / settings

Display recent votes

Firmware SHA-256 / version

Export button (USB)




Navigation: state-driven, no menus; reset on key removal.




## 5. BLE interface with Tracker firmware

Connect to custom OpenVote GATT service.

BLE Characteristics used:

fw_info — firmware metadata

key_state — detect key presence

sign_request / sign_response — challenge signing

lora_send / lora_status — payload Tx

gps_report — GNSS position

debug_log — optional telemetry



Notes: app must respect BLE MTU limits, chunk data if needed.




## 6. Vote payload composition & signing

Compact JSON / binary payload:

````
{
  "election_id": "E2025-01",
  "round": 1,
  "node_id": "TRACKER123",
  "credential_id": "SKFINGERPRINT",
  "vote": "Y",
  "timestamp": 1700000000,
  "gps_ok": true,
  "signature": "3045022100..."
}
````

Payload transmitted via Tracker for LoRa propagation.

Total payload ≤ 240 bytes preferred to fit Meshtastic limits.





## 7. GPS policy & validation

App reads last known GNSS fix from firmware.

Time window: 1 hour default.

GPS outside allowed region → reject vote.

Display reason in Proof-of-Vote if rejected.





## 8. Vote rate limiting & enforcement

Enforce one vote per round per credential.

Duplicate attempts increment rejected vote counter at Town Hall.

Reset state when key removed → allows multiple family members on same device.

Minimum interval between votes (configurable, e.g., 3 minutes).





## 9. Proof-of-Vote confirmation

Town Hall sends back P2P confirmation message for each vote.

App displays:

✅ Vote Accepted + Hash

❌ Rejected + Reason


Screenshotable by voter; serves as personal receipt.





## 10. Logging, telemetry & offline storage

Maintain local append-only vote log (encrypted optional).

Log firmware version and SHA-256 for transparency.

Exportable via USB.

Telemetry: vote count, rejected votes, GPS status, LoRa tx info.





## 11. Security & key management

SoloKey required for every vote; app refuses to proceed without key.

Lost/broken key → voter must re-register; new credential ID recorded.

Never store private keys; signature ephemeral.

Only manual BLE pairing; no automatic connections.

Display firmware hash for verification.





## 12. Build / test / QA plan

Target: Android 7+; Android Studio / Gradle build.

Unit tests: BLE interface, payload creation, GPS policy, duplicate vote enforcement.

Integration tests: pair with Tracker firmware in lab; simulate LoRa propagation; validate Proof-of-Vote.

Demo: use Figma VM screens to simulate vote flow for presentation.





## 13. Release / distribution

Deliver app via USB stick; auto-update if app already installed.

Optionally provide signed APK and SHA-256 hash for verification.





## 14. Appendix #2 Example payloads & JSON messages

Vote request to Tracker: challenge (32 bytes)

Sign response from Tracker: DER-encoded signature

LoRa transmit request: hex-encoded payload

Rejected vote reasons:

Not on roll

Invalid signature

Duplicate credential

GPS outside allowed region

Untrusted Tracker

Wrong round

Malformed packet


## Appendix #2: Voter key integration

The OpenVote VM app will support secure vote signing through a Solokey C authentication token connected to a Tracker L1 (ESP32-S3) over USB-C.

### Features

Solokey Presence Detection

The paired Tracker L1 automatically detects when a Solokey is inserted or removed.

The VM app displays:

“Insert Solokey”

“Solokey detected”

“Remove Solokey” (after Proof-of-Vote received)


Bluetooth Pass-Through Signing

The VM app sends a vote-signing challenge to the Tracker L1 over Bluetooth.

The Tracker forwards the challenge to the Solokey over USB, receives the signature, and returns it to the VM app.

Firmware Compatibility

No full CTAP/FIDO2 stack is required on the Tracker.

Only minimal USB-HID pass-through is used.

Existing ESP32-S3 USB-OTG support makes these additions small and stable.


### Result

The VM app achieves hardware-rooted authentication without increasing device complexity, enabling secure vote creation on any standard Android phone.





## Implementation checklist (starter)

[ ] Create Android Studio skeleton project

[ ] Implement BLE service discovery, pairing, GATT interaction

[ ] Implement SoloKey detection prompts & signing workflow

[ ] Compose vote payload, send to Tracker, receive confirmation

[ ] Display Proof-of-Vote / rejection

[ ] Enforce GPS and rate-limiting policies

[ ] Offline append-only logging & USB export

[ ] Figma demo screens integration for presentation / video





## Rationale:
The app is the intelligent layer — it enforces voting rules, assembles cryptographically verifiable payloads, handles voter input, and displays confirmations. Tracker firmware provides hardware trust, LoRa propagation, and signature signing. Together they implement OpenVote’s offline verified, cryptographically auditable public voting system.
