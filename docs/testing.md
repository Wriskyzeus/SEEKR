# SEEKR Version 1 Testing Strategy

> Planning strategy only; no physical test results or numerical thresholds have been claimed.

## Purpose

Demonstrate that SEEKR can safely search a taped arena, **visually verify the user-selected object**, stop, and report FOUND. Target approach and near-target stopping-distance tests are deferred beyond V1.

## Progressive Test Levels

| Level | Objective | Example test |
| --- | --- | --- |
| Component | Validate one part/interface safely | One motor + encoder, sensor readings, camera capture |
| Subsystem | Validate related parts | Two-wheel PI/PID, boundary inhibit, target verification on recorded frames |
| Integration | Validate Pi/ESP32 and behavioral handoffs | Authorized motion + telemetry, STOP, timeout/reconnect, simulated detections |
| System | Validate full V1 mission | Autonomous search → VERIFY → safe stop → FOUND report |

## Initial Test Matrix

| Test | What to verify | Observations |
| --- | --- | --- |
| Power-up and reset | Motor outputs remain safe on startup/reset | Voltage rails, unexpected motion, fault state |
| Single motor + encoder | Correct direction, counts, and controllability | Encoder increments, PWM response, current/heat concerns |
| Two motors and closed loop | Independent speed tracking | Setpoint/measured speed, drift, oscillation |
| Differential drive | Repeatable straight/turn behavior | Motion error, encoder logs, speed |
| Tape sensor calibration | Floor/tape discrimination and fault handling | Thresholds, missed/false events, lighting |
| Physical containment | Whole robot stays within defined limit | Stopping distance, footprint, boundary violations |
| Serial reliability | Commands/telemetry, invalid input, timeout, reconnect | Latency, rejected messages, inhibited motion |
| Perception verification | Selected target distinguished and reliably confirmed | True/false detections, observation age, confusion among four objects |
| Perception health | Frozen/stale frames do not permit unsafe decisions | Timeouts, state transitions, stopped behavior |
| Behavior integration | Simulated detection → SEARCH/VERIFY/FOUND | State changes, stop request and confirmation |
| End-to-end V1 | Target selection through verified FOUND | Search time, correct result, boundary events, logs |

## Metrics

- **Verified search success rate:** Fraction of trials ending in correctly verified selected-target FOUND with safe stopping.
- **Time to verified FOUND:** Duration from authorized start to verified success; record timeouts/failures separately.
- **False FOUND rate:** Trials in which the wrong target or an unverified observation triggers success.
- **Missed target / timeout rate:** Trials with no successful confirmation before a defined limit.
- **Boundary violations:** Events where the defined containment limit is exceeded.
- **Boundary response:** Detection latency, braking/stopping distance, and recovery behavior.
- **Motion tracking:** Desired versus measured wheel speed; odometry/path length only if used.
- **Reliability:** Command/telemetry faults, resets, perception freshness violations, and recovery success.

## Test Discipline

1. Define arena dimensions, tape width, permitted containment limit, test objects, lighting, and starting position.
2. Document hardware configuration and firmware/software revision.
3. Establish and test a manual stop before autonomous drive tests.
4. Begin with slow, supervised trials; do not test containment by assuming detection alone is sufficient.
5. Record raw observations/logs, pass/fail outcomes, and conditions.
6. Set measurable numerical acceptance criteria after initial baseline measurements; do not fabricate success rates.

## Deferred Tests

ALIGN/APPROACH, near-target stopping accuracy, object retrieval, obstacle avoidance, and mapping belong to future versions unless the team explicitly changes V1 scope.
