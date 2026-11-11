# SEEKR Version 1 Engineering Requirements

## Mission

SEEKR Version 1 shall autonomously locate and approach a user-selected object in a bounded, flat indoor arena, then report that the target has been found.

## Scope

The Version 1 system consists of a differential-drive mobile rover, a Raspberry Pi 5 for high-level computing, an ESP32 for low-level control, an onboard camera, and downward-facing boundary sensors. A user selects one of four known target objects using a web interface. The robot searches an arena with a black tape boundary and no obstacles, detects the selected object, approaches it, stops nearby, and communicates its result.

## Functional Requirements

| ID | Requirement |
| --- | --- |
| FR-01 | The system shall allow a user to select one of four known target objects through the web interface. |
| FR-02 | The Raspberry Pi shall maintain the high-level search state and issue motion commands to the ESP32. |
| FR-03 | The ESP32 shall control the two drive motors and acquire wheel-encoder data. |
| FR-04 | The system shall acquire camera data and evaluate it for the selected target. |
| FR-05 | The robot shall execute a search behavior while remaining within the bounded arena. |
| FR-06 | The robot shall detect the black tape boundary using downward-facing sensors and execute a defined boundary response. |
| FR-07 | After detecting the selected target, the robot shall attempt to approach and stop near it. |
| FR-08 | The system shall report target-found status to the web interface. |
| FR-09 | The Raspberry Pi and ESP32 shall exchange commands and telemetry over serial/UART. |

## Non-Functional Requirements

| Category | Requirement |
| --- | --- |
| Safety | The robot shall provide a stop behavior and should fail to a safe stopped state when communication or control faults are detected. |
| Maintainability | Hardware interfaces, firmware, high-level software, and protocol definitions should be documented and modular. |
| Observability | Development builds should support logging or inspection of robot state, commands, sensor readings, and detection events. |
| Repeatability | Tests should define arena layout, lighting, target placement, and trial procedure sufficiently to support repeatable comparison. |
| Usability | The target-selection workflow and success/failure status should be understandable without direct access to source code. |

## Constraints

- The Version 1 arena is flat, bounded, indoors, and initially obstacle-free.
- A black tape boundary defines the permitted search area.
- Only four known target objects are in scope; their positions may vary between trials.
- The Raspberry Pi 5 performs Linux-hosted, high-level computing and vision work.
- The ESP32 performs low-level motor, encoder, sensor, and UART work.
- The robot uses two DC drive motors with wheel encoders, a motor driver, a camera, and downward-facing boundary sensors.
- Version 1 prioritizes a constrained, testable baseline over general indoor autonomy.

## Success Criteria

Version 1 will be considered ready for a defined demonstration when documented test procedures show that the robot can:

1. accept a user-selected target;
2. remain inside the taped arena during a search trial;
3. identify the requested target rather than merely any known object;
4. approach and stop near the detected target; and
5. report the result through the user interface.

Quantitative acceptance thresholds will be set after the team establishes the test setup, safety limits, and baseline hardware behavior.

## Explicitly Excluded From Version 1

- Robotic arm integration, grasping, or object retrieval
- Obstacle avoidance
- SLAM or autonomous map construction
- General-purpose object recognition beyond the four known targets
- Unbounded, outdoor, or cluttered-environment navigation
- Claims of optimized or learning-based search performance before validation
