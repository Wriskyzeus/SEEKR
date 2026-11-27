# SEEKR — Autonomous Visual Search Robot

> A two-wheel differential-drive rover that autonomously searches a bounded indoor arena for one of four user-selected objects.

**Status:** V1 planning complete; hardware selection and physical prototyping next. Nothing in this repository should be interpreted as a claim that the finished robot has been built or validated.

## V1 Mission

1. A user selects one of four known targets on a web interface and starts the mission.
2. SEEKR autonomously searches a flat, obstacle-free wooden-floor arena enclosed by black tape.
3. The Raspberry Pi camera/perception system identifies a candidate of the selected target.
4. The mission logic **verifies** the detection; failed verification returns to SEARCH.
5. When the selected object is confirmed, the robot stops safely and reports **FOUND** through the web interface.

**V1 does not require target alignment, physical approach, grasping, or retrieval.** ALIGN/APPROACH may be a subsequent version, followed by object retrieval and more advanced navigation. If team requirements change, the scope documents and test criteria will change together.

## Architecture

- **Raspberry Pi 5 (high level):** web application, camera, perception, target selection, search decisions, mission state machine, logs.
- **ESP32 (low level):** motor PWM/direction, two encoder inputs, closed-loop wheel-speed control, downward boundary sensors, immediate motion inhibit and command watchdog.
- **Mechanical:** two independently driven geared DC wheels plus a passive caster; custom chassis.
- **Communication:** bidirectional serial link (USB serial or direct UART to be selected), with motion commands and telemetry.
- **Power:** protected battery and appropriately rated, regulated motor and logic branches; final design pending part selection.

The ESP32 can stop the robot independently of the Pi. Boundary detection, communication timeouts, and invalid motion commands must not be bypassed by high-level search behavior.

## Team Responsibilities

- **Toufic — hardware & embedded:** ESP32 firmware, motors, encoders, motor driver, power, boundary sensing, chassis/CAD, Pi–ESP32 integration, and physical tests.
- **Eli — perception & autonomy:** computer vision/ML, high-level search and behavior software.
- **Shared:** interface contracts, safety coordination, integration, experiment design, and V1 acceptance criteria.

## Repository Guide

| Path | Purpose |
| --- | --- |
| `firmware/esp32/` | ESP32 firmware and hardware-facing control |
| `robot/perception/` | Raspberry Pi camera and perception integration |
| `robot/search/` | High-level autonomous search and target verification |
| `robot/communication/` | Pi-side serial communications |
| `robot/control/` | Pi-side command orchestration; motor feedback loops remain on ESP32 |
| `app/` | Web interface |
| `cad/` | Mechanical designs and fabrication |
| `docs/` | Requirements, architecture, protocol, integration, hardware, testing, learning notes |
| `tests/` | Test assets and automated checks |
| `scripts/` | Development utilities |
| `simulation/` | Optional simulated inputs and search experiments |
| `data/` | Test/example data; large or sensitive datasets should not be committed blindly |

## Engineering Documents

- [Requirements](docs/requirements.md)
- [Architecture](docs/architecture.md)
- [Protocol draft](docs/protocol.md)
- [Hardware selection log](docs/hardware.md)
- [Integration plan](docs/integration.md)
- [Testing strategy](docs/testing.md)
- [Learning notes](docs/learning-notes.md)

## Next Milestone

Choose verified compatible components and perform a **safe single-motor + encoder bench test** with the ESP32 before assembling the complete rover. No specific motor, regulator, battery, or sensor model has been committed to yet.

## Out of Scope for V1

Approach/retrieval, obstacle avoidance, SLAM, LiDAR, room mapping, tape-free navigation, and arbitrary-object recognition.
