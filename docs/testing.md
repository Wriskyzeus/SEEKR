# SEEKR Version 1 Testing Strategy

## Purpose

Testing will progress from isolated hardware checks to repeatable end-to-end autonomous-search trials. No test results are recorded in this document yet; it defines the initial strategy and the measurements the team intends to collect.

## Test Levels

| Level | Goal | Example tests |
| --- | --- | --- |
| Component | Verify individual devices and interfaces | Individual motors, encoders, boundary sensors, camera, UART |
| Subsystem | Verify related components together | Motor/encoder control, perception pipeline, web-to-Pi target selection |
| Integration | Verify Raspberry Pi and ESP32 behavior together | Command/telemetry exchange, boundary response, target approach |
| System | Verify the Version 1 mission | Complete autonomous search in the defined arena |

## Initial Test Matrix

| Test | Objective | Example observations to record |
| --- | --- | --- |
| Individual motors | Confirm each motor drives in both directions under controlled commands | Direction, response consistency, current concerns, abnormal behavior |
| Encoder accuracy | Relate encoder readings to known wheel movement | Counts, repeatability, distance/rotation estimate error |
| Straight-line driving | Assess ability to drive a repeatable straight path | Drift, distance travelled, heading change, trial conditions |
| Turning | Assess repeatable turning behavior | Commanded behavior, observed heading change, repeatability |
| Boundary detection | Verify detection of black tape and defined response | Sensor readings, detection rate, response timing, missed events |
| Pi ↔ ESP32 communication | Verify command delivery and telemetry reporting | Message integrity, latency, parsing errors, disconnect recovery |
| Object recognition | Evaluate recognition of selected known targets | Detection accuracy, false detections, lighting/viewpoint notes |
| Target approach | Evaluate movement from detection toward target | Approach success, stopping distance error, target reacquisition |
| Complete autonomous search | Exercise selection, search, detection, approach, stop, and reporting | Search outcome, time, distance, boundary events, logs |

## Metrics To Record

| Metric | Description |
| --- | --- |
| Search success rate | Fraction of defined trials in which the selected target is found and approached successfully. |
| Average search time | Time from search start to success or defined termination condition. |
| Distance travelled | Estimated or measured path length per trial. |
| Object-detection accuracy | Correctness of detections under documented test conditions. |
| False detections | Detections reported for an incorrect target or when no valid target is present. |
| Boundary violations | Instances in which the robot crosses, or fails to respond appropriately to, the arena boundary. |
| Stopping distance error | Difference between intended and observed stopping distance relative to the target. |

## Test Discipline

- Document arena dimensions, tape layout, target identity and position, lighting, camera configuration, and software/firmware revision for each trial.
- Define a safe manual-stop procedure before powered motion tests.
- Start with low-speed, supervised tests and increase complexity only after reviewing results.
- Keep raw logs and observations separate from conclusions; record failures as well as successful trials.
- Add acceptance thresholds only after baseline measurements are available.
