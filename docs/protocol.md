# Raspberry Pi ↔ ESP32 Communication Protocol

> **Planning draft, not implemented or finalized.** Physical link is either USB serial or direct 3.3 V-compatible UART, pending board and wiring selection. The serial byte stream needs an application-level protocol.

## Responsibilities

The Raspberry Pi requests authorized robot motion, STOP, and startup/session synchronization. The ESP32 validates commands, enforces local boundary protection, runs both motor-speed loops, and returns readiness, telemetry, and faults.

## Candidate Messages (Illustrative, Not a Final Grammar)

| Direction | Candidate message | Meaning |
| --- | --- | --- |
| Pi → ESP32 | `SET_VELOCITY(v, omega)` | Request robot linear and angular velocity, in units/ranges to be defined. |
| Pi → ESP32 | `SET_WHEEL_SPEEDS(left, right)` | Alternative wheel-speed request; decide which motion API V1 uses. |
| Pi → ESP32 | `STOP` | Revoke motion authorization and request a safe stop. |
| Pi ↔ ESP32 | `HELLO / READY / SESSION` | Establish mutually recognized readiness and a new control session. |
| ESP32 → Pi | `STATUS(...)` | Report state, active motion inhibition, faults and readiness. |
| ESP32 → Pi | `WHEELS(...)` | Report measured left/right speed and encoder information. |
| ESP32 → Pi | `BOUNDARY(...)` | Report tape sensor states and boundary events. |
| ESP32 → Pi | `ACK(...)` | Acknowledge selected commands at a defined acceptance level. |

These symbols describe intent only; the actual textual or binary format will be selected later. **Never interpret `SET_VELOCITY,left,right` as both robot velocity and wheel velocity.** The interface needs one unambiguous definition.

## Reliability and Safety Requirements

- Specify framing, buffering, parsing, field count/types, numerical limits, and invalid/unknown message handling.
- Consider sequence numbers, acknowledgements for selected commands, checksums/CRC, and duplicate detection where justified.
- Commands are accepted only within the currently synchronized, authorized session.
- Motor movement requires **fresh valid motion authorization**; heartbeat alone does not authorize motion.
- Lost fresh motion commands trigger an ESP32-side stop. Invalid messages do not refresh authorization.
- ESP32 boundary inhibition overrides Pi movement requests. Recovery motion requires a separate, explicitly safe rule.
- STOP and critical faults must not be undone by old commands or reconnection.
- Reset or restart requires safe outputs and a new readiness/synchronization sequence.
- Perception freshness is checked on the Pi independently of serial-link health.

## Decisions to Finalize During Implementation

- USB serial versus direct UART; wiring, common reference, logic compatibility.
- One motion API: robot `(v, omega)` or wheel-speed targets; units, signs, limits.
- Framing and schema; session identifier; command sequence/ACK semantics.
- Integrity checking (CRC/checksum), retries and idempotence.
- Command, telemetry, timeout, and control-loop intervals.
- Startup handshake, STOP/fault and boundary recovery semantics.
- Versioning and compatibility tests.

Log important decisions and link test results as the protocol becomes executable.
