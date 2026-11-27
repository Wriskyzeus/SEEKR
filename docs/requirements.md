# SEEKR Version 1 Engineering Requirements

**Status:** Working V1 baseline for team agreement; quantitative thresholds and final interface contracts remain open.

## Mission

SEEKR V1 shall autonomously **locate and visually verify** one of four user-selected known objects in a flat, bounded indoor arena, safely stop, and report FOUND. Approaching or retrieving the object is not a V1 requirement.

## Operating Environment and System Constraints

- Flat indoor wooden floor; black-tape perimeter; no intentional obstacles.
- Four known target objects; object positions can vary between trials.
- Two independently driven geared DC wheels with encoders and one passive caster.
- Raspberry Pi 5 runs camera/perception, web interface, search and mission behavior.
- ESP32 runs motor control, encoder acquisition, boundary monitoring, and immediate motion safety.
- Protected battery, appropriate regulation, and a defined stop mechanism.
- Pi↔ESP32 serial interface (USB serial or direct UART, pending hardware selection).

## Functional Requirements

| ID | Requirement |
| --- | --- |
| FR-01 | The web interface shall permit selection of one of four known target objects. |
| FR-02 | An authorized start command shall initiate an autonomous search; STOP shall interrupt the mission. |
| FR-03 | The Pi shall manage SEARCH, VERIFY, FOUND, STOPPED and FAULT mission states, with explicit transition conditions. |
| FR-04 | The camera/perception system shall provide observations of the selected target with sufficient identity, location and timing information for verification. |
| FR-05 | SEARCH shall move the robot autonomously while respecting the bounded arena and active safety restrictions. |
| FR-06 | VERIFY shall use a defined confirmation rule before declaring the user-selected target FOUND; a rejected or insufficient candidate shall not produce success. |
| FR-07 | Once the selected target is verified, the system shall stop safely and report FOUND to the web interface. |
| FR-08 | The ESP32 shall control two drive motors using encoder feedback and report essential motion telemetry. |
| FR-09 | The ESP32 shall monitor black-tape sensors and locally inhibit unsafe movement independently of the Pi. |
| FR-10 | The Pi and ESP32 shall exchange validated, time-bounded motion commands, readiness, telemetry, and relevant fault states. |
| FR-11 | Invalid or stale motion authorization, reset, or critical faults shall not cause autonomous movement to resume without a fresh valid authorization. |

## Non-Functional Requirements

| Area | Requirement |
| --- | --- |
| Safety | Local boundary protection and loss-of-command stopping shall override ordinary movement. Verify containment including chassis footprint and stopping distance. |
| Reliability | Perception observations must meet a freshness rule; frozen/stale observations must not drive perception-dependent motion. |
| Observability | Development tests shall record commanded and measured motion, boundary/fault events, and detection/verification outcomes. |
| Repeatability | Trials shall record arena, tape, lighting, target identity/position, software revisions, and procedures. |
| Maintainability | Keep clear subsystem ownership, modular interfaces, and documented electrical and software contracts. |
| Usability | The web interface shall clearly indicate selected target, mission status, and successful/failed/stopped outcomes. |

## V1 Acceptance Criteria (Qualitative Until Measured)

The robot must demonstrate that it can:
1. Receive the selected target and start safely.
2. Move autonomously while respecting its defined taped-area containment criterion.
3. Distinguish and verify the requested object (not merely any known object).
4. Stop safely when the verification criterion is met.
5. Report FOUND for the correct target, and distinguish ordinary stop from fault.
6. Respond safely to boundary detection and relevant communication/control failures.

The team will define quantitative pass thresholds and a repeatable trial protocol after the first hardware and perception baseline tests.

## Explicitly Excluded from V1

- Target ALIGN/APPROACH and near-target stopping-distance requirement.
- Gripper, retrieval, robotic arm, and object pickup.
- Obstacle avoidance, LiDAR, SLAM, mapping or unrestricted navigation.
- Mandatory IMU/fused localization, ROS 2, or arbitrary-object recognition.

## Change Control

If Eli and Toufic agree to add target approach to V1, update `README.md`, `docs/architecture.md`, `docs/requirements.md`, `docs/testing.md`, and related integration criteria before implementing it.
