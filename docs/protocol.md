# Raspberry Pi ↔ ESP32 Communication Protocol

> **Initial draft — subject to change during development.** This document records candidate UART messages for early integration. It is not a final protocol specification.

## Purpose

UART communication will allow the Raspberry Pi to request low-level robot actions and allow the ESP32 to report sensor and controller state. Message framing, acknowledgement rules, timing, units, error handling, and exact field definitions must be finalized through implementation and test.

## Candidate Command Messages: Raspberry Pi to ESP32

| Message | Candidate format | Intended meaning |
| --- | --- | --- |
| Set velocity | `SET_VELOCITY,left,right` | Request left and right motor commands. Units and permissible range are TBD. |
| Stop | `STOP` | Request that drive motion stop. |
| Get status | `GET_STATUS` | Request current controller/sensor status. |

## Candidate Telemetry Messages: ESP32 to Raspberry Pi

| Message | Candidate format | Intended meaning |
| --- | --- | --- |
| Encoders | `ENCODERS,left,right` | Report left and right encoder readings; count convention is TBD. |
| Boundary | `BOUNDARY,left,center,right` | Report boundary-sensor readings or states; encoding is TBD. |
| Status | `STATUS,state` | Report a controller state or diagnostic status; state vocabulary is TBD. |

## Initial Design Questions

- Should each message be newline-delimited, length-prefixed, or use another framing method?
- Which side owns command acknowledgement, retries, and timeouts?
- What units, signs, limits, and update rates apply to velocity commands?
- Are encoder values absolute counts, deltas, or both?
- Are boundary fields raw values, normalized values, or thresholded booleans?
- Which fault, watchdog, emergency-stop, and boot-state messages are required?
- How should malformed messages, lost communication, and version mismatches be handled?

## Change Control

Protocol changes should be recorded here before or alongside dependent firmware and Raspberry Pi software changes. Once hardware testing begins, the team should add an explicit message grammar, field ranges, timing expectations, error responses, and compatibility/versioning policy.
