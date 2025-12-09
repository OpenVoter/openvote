# Wio Tracker L1 — Firmware Architecture

Target device: Wio Tracker L1 (ESP32-based, built-in LoRa + GNSS + USB-C + battery + BLE)

Purpose: Minimal, secure hardware bridge for OpenVote — expose SoloKey signing, GPS, and LoRa transmit to the Meshtastic VM (voter) Android app over BLE. Firmware intentionally does not implement voting logic; that lives in the [VM app] <add link>

Status: Specification / architecture (implementation-ready).

Scope: EU 868 MHz pilot; single-node behaviour; demo & pilot-ready.



## Table of contents

1. Goals & Design Principles


2. High-level architecture


3. Firmware responsibilities (what it MUST do)


4. BLE API (GATT) — characteristics & formats


5. USB / SoloKey integration (U2F/FIDO over HID)


6. LoRa transmit interface & constraints


7. GPS interface & policy support


8. State machine


9. Security: hardening, secrets handling, attestation


10. Logs, telemetry & debug interface


11. Build / test / QA plan


12. Release / firmware signing / distribution


13. Minimal BOM & hardware notes


14. Appendix: example JSON messages & packet size considerations





## 1. Goals & Design Principles

Separation of concerns: firmware = hardware access + simple policy; Meshtastic VM app = vote construction, user UX, electoral rules, cryptography validation.

Minimal attack surface: tiny, auditable code path for SoloKey handling and BLE transport; no storage of private material.

Deterministic state machine: predictable, inspectable states for demos and audits.

Transparency: expose firmware version and SHA-256 checksum at connection time.

Robustness: graceful handling of USB disconnects, BLE drops, LoRa TX failures and retries.

Privacy & safety: never persist private keys or signatures; zero buffers after use.





## 2. High-level architecture

[VM App on Android device]
   ↕ BLE (GATT)
[Wio Tracker L1 Firmware]
   ├─ USB-C <-> SoloKey (U2F/FIDO HID)
   ├─ GNSS
   └─ LoRa radio (Meshtastic send API)

Roles:

App: builds OpenVote payload, handles electoral roll, timestamping policy, retry, chaining, reads GPS if needed, verifies Town Hall confirmations.

Firmware: expose hardware services reliably and securely; accept app payloads for transmission; return signatures and status.




## 3. Firmware responsibilities

Mandatory

Enumerate USB devices; detect SoloKey plug/unplug.

Offer SoloKey public key (or key fingerprint) to the app.

Accept a binary "challenge" from the app and produce signature using SoloKey (via U2F/FIDO signing).

Provide a BLE GATT interface (see section 4) for all interactions.

Accept payload byte array from app and send over LoRa via Meshtastic send API.

Provide GPS reading and basic status (fix/no-fix, timestamp).

Compute and advertise firmware SHA-256 on boot and over BLE.

Implement simple retry for LoRa transmission (configurable attempts).

Clean sensitive buffers immediately after use.

Produce a minimal debug log over a BLE characteristic (append-only, non-sensitive).


Forbidden / Not implemented

Firmware must not assemble vote semantics (Y/N, election ID) — app does this.

Firmware must not store private keys or signatures persistently.

Firmware must not decide voter eligibility or enforce electoral roll — app/Town Hall do that.





## 4. BLE API (GATT) — characteristics & formats

Use a custom service UUID for OpenVote:
0000OV01-0000-1000-8000-00805f9b34fb (replace OV01 with chosen stable UUID per repo)

Service: OpenVote (primary)

Handle	Name	Type	Direction	Description

0x01	fw_info	String	Notify/Read	Firmware metadata JSON: { "version": "v0.1", "sha256": "<hex>", "build": "<date>" }
0x02	key_state	JSON	Notify/Read	`{ "present": true
0x03	sign_request	Write (with response)	App -> FW	Binary: challenge (raw bytes) — response via sign_response
0x04	sign_response	Binary	Notify	`{ "status":"ok"
0x05	lora_send	Write (with response)	App -> FW	JSON: { "payload":"<hex>", "topic":"openvote", "req_id":"<uuid>" } — response via lora_status
0x06	lora_status	JSON	Notify	`{ "req_id":"<uuid>", "tx":"ok"
0x07	gps_report	JSON	Notify/Read	`{ "lat":xx.xxxxx, "lon":yy.yyyyy, "fix":true
0x08	debug_log	String	Notify/Read	Append-only safe debug messages (no secrets)
0x09	control	Write	App->FW	Admin: `{ "action":"reboot"


Notes:

Use compact JSON with fields in fixed order for predictability.

Signature binary should be returned as hex-coded string within sign_response for simplicity over BLE.

All messages to/from BLE should include a small req_id/msg_id so app can match responses if multiple requests are in flight.

For large binary transfers (payload), use chunking if necessary (MTU-aware). Include sequence numbers.





## 5. USB / SoloKey integration (U2F/FIDO over HID)

Tracker must present a USB Host stack to talk to SoloKey (USB-C). Implementation steps:

1. USB enumeration: on device connection check vendor/product IDs (SoloKey VIDs/PIDs). Accept standard FIDO/U2F devices.


2. U2F HID protocol: implement the minimal subset needed:

getInfo — to check supported algorithms and capabilities

getPublicKey / registration — to fetch public key or credential ID if available (or derive fingerprint)

sign — send challenge and receive signature (raw ECDSA r||s or DER)



3. Challenge flow: app sends a random challenge (e.g., 32 bytes). Firmware passes this to SoloKey using sign command and returns the signature.


4. Timeouts & errors: define a 10s default timeout for signing operations; propagate errors in sign_response.


5. Key fingerprint: compute a stable fingerprint (SHA-256 of public key bytes) to use as CredentialID in vote payloads.



Security:

All signature operations are ephemeral; the private key never leaves SoloKey.

On key removal, clear all key-related buffers immediately.




## 6. LoRa transmit interface & constraints

Payload: app composes OpenVote payload as binary, likely encoded compactly to fit Meshtastic payload limits. Firmware treats it as opaque bytes.

Max size: Meshtastic/Tracker payload typical limits vary — aim to support at least 240–250 bytes; implement chunking if app requests larger payloads, but app should avoid exceeding ~240 bytes.

Transmission: provide lora_send characteristic that queues payload for Tx; firmware replies with lora_status.

Retries: default 3 retries on failure; backoff 500ms between attempts; app can request different retry counts.

Acknowledgements: firmware should report msg_id and rssi on success; on fail report reason code.

Radio configuration: EU 868 MHz settings; ensure duty cycle and regional compliance.





## 7. GPS interface & policy support

Read GNSS at configurable cadence (default: 1 Hz).

Expose raw lat/lon/fix/timestamp via gps_report.

Provide helper: gps_age (current_ts - gps_ts).

The app will implement locality policy (e.g., last-known GPS ≤ 1 hour). Firmware only provides the sensor data.

Default behavior: if no fix available, report fix:false and still send best-known position (if any).





## 8. State machine

Finite states:

BOOT — compute firmware SHA-256, init peripherals.

IDLE — BLE advertising, waiting for paired app.

WAITING_FOR_KEY — key not present.

KEY_PRESENT — key inserted; idle for app request.

READY — BLE connected + key present; can accept sign/send requests.

SIGNING — performing SoloKey sign operation.

SENDING — sending LoRa payload.

ERROR — any non-recoverable error; app must read debug_log and clear.

RESET — soft reboot stage.


Transitions:

BOOT → IDLE

IDLE → WAITING_FOR_KEY on power (if no key) or KEY_PRESENT if key detected

KEY_PRESENT → READY when BLE connection established (paired app present)

READY → SIGNING on sign_request

SIGNING → READY on success/fail

READY → SENDING on lora_send

SENDING → READY on tx success/fail

ANY → ERROR on critical faults

ERROR → RESET requires app control clear





## 9. Security: hardening, secrets handling, attestation

Hardening

Use secure coding patterns (no dynamic exec, bounds-check all buffers).

Disable any unrelated services.

Use compile-time flags to strip debug in production.


Secrets

Never store private keys, signatures, or full public key blobs persistently.

Public key fingerprints may be cached temporarily but must be cleared on key removal.


Firmware attestation

Compute SHA-256 of firmware binary at boot; advertise via fw_info.

Provide a documented process to verify firmware binary on Github: signed releases and published SHA-256.


Transport

BLE pairing must use at least LE Secure Connections if supported; encourage device bonding to avoid accidental connections. However, app/firmware pairing model is manual and expected to be local / verified.


Tamper evidence

Log firmware version and sha256 in the app on connection; show to voter if needed.





## 10. Logs, telemetry & debug interface

debug_log characteristic is append-only and safe to expose; avoid secrets.

Typical entries:

BOOT OK v0.1 sha256:...

KEY DETECTED fp:...

SIGN REQ id:...

SIGN OK id:...

LORA TX OK req:... msgid:...

LORA TX FAIL req:... err:...

KEY REMOVED


Log retention: circular buffer in RAM (size tunable), cleared on reboot.





## 11. Build / test / QA plan

Build

Toolchain: ESP-IDF or Arduino-ESP32 (choose consistent stack used by Wio Tracker vendor).

CI: GitHub Actions builds for each tagged release; produce binary and SHA-256.


Unit tests

Mock USB SoloKey interactions; test signing code paths.

Mock BLE client; test GATT behavior.


Integration tests

Run with Meshtastic VM app in emulator (or real device) — ensure BLE sequence works.

Run LoRa send simulation (or test bench) to assert payload handling and retries.


Security tests

Fuzz USB HID interactions.

Pen-test BLE characteristics (attempt oversized writes, wrong content).


Acceptance

Run a 24-hour pilot with 5 devices:

Reboot cycles

Insert/remove keys

Simulated TX failures






## 12. Release / firmware signing / distribution

Sign firmware binaries (GPG or similar) and publish SHA-256 in repo.

Provide explicit reproducible build instructions.

For public trust: perform a D3 ceremony: public firmware hash verification printed on the hardware registry and included on the USB distribution media.





## 13. Minimal BOM & hardware notes

Wio Tracker L1 (vendor image) — verify USB host capability (OTG host) on USB-C port.

SoloKeys C (USB-C) — ensure compatibility with FIDO/U2F HID.

Ensure Tracker has enough power budget for USB host + SoloKey operation; if not, require SoloKey powered by Tracker battery.





## 14. Appendix — Example messages & constraints

Example sign_request flow

App → sign_request (Write): raw-challenge (32 bytes)

FW → sign_response (Notify):


{
  "status": "ok",
  "signature": "3045022100... (DER hex)",
  "key_fingerprint": "6f8a3c..."
}

Example lora_send request

App writes:

{
  "req_id":"uuid-v1",
  "payload":"0a1b2c3d... (hex)",
  "topic":"openvote",
  "retries": 3
}

Firmware notifies lora_status:

{
  "req_id":"uuid-v1",
  "tx":"ok",
  "msg_id":"abcd1234",
  "rssi":-72
}

Payload size

Keep payload & signature together to < 240 bytes if possible.

If app must send larger data, chunk it with sequence numbers.





## Implementation checklist (starter)

[ ] Create project skeleton (ESP-IDF / Arduino)

[ ] Implement USB Host enumerator; detect SoloKey VID/PID

[ ] Implement minimal U2F/FIDO HID sign and getInfo

[ ] Implement BLE service & characteristics per GATT table

[ ] Implement LoRa sendPacket() wrapper & retry

[ ] Implement GNSS read & gps_report notifier

[ ] Implement firmware SHA-256 on boot & fw_info

[ ] Implement debug log buffer and debug_log characteristic

[ ] Implement state machine and tests

[ ] CI build + signed release + SHA-256 publish




## Notes & rationale

Keeping vote logic out of firmware minimizes attack surface and simplifies verification. The app (OpenVote VM) is responsible for constructing the minimal OpenVote payload and applying business rules; the Tracker is a bicycle-pump: it physically signs (via SoloKey), transmits, and reports hardware state.

The BLE API is intentionally small and JSON-friendly for rapid app development. Production work may compress JSON to binary TLV for efficiency.

Firmware SHA-256 and D3 ceremony enable supply-chain trust. Voters and auditors can independently verify the firmware binaary.
