# SEEKR — Robotics & Embedded Systems Engineering Notes

> **Status:** Pre-build learning completed (Steps 1–19). These are consolidated engineering reference notes, not proof that hardware has been purchased or tested.

## Project Context and How to Use These Notes

SEEKR V1 is a two-wheel differential-drive visual-search robot with a passive caster. In a flat wooden-floor arena enclosed by black tape, it autonomously searches for one of four user-selected known objects, **visually verifies the selected target**, safely stops, and reports **FOUND**. ALIGN/APPROACH and object retrieval are future extensions, **not V1 requirements**. This working V1 scope is documented in [requirements.md](requirements.md).

- **Raspberry Pi 5:** Web interface, camera/perception, search decisions, mission state, and telemetry processing.
- **ESP32:** Encoder measurement, motor feedback control, boundary sensing, local safety, and communication.
- **Ownership:** Toufic leads electronics, embedded firmware, chassis and physical testing; Eli leads perception/ML and high-level search. Integration and interface contracts are shared.
- **Note numbering:** Sections 1–16 are *topic/reference sections*, not the original sequential learning-step numbers.
- **Learning versus implementation:** Concepts below have been studied, but component selection, firmware, calibration, and physical validation are still ahead. See the consolidated implementation checklist at the end.

## Contents

1. DC Motors
2. Motor Drivers & H-Bridges
3. PWM Speed Control
4. Wheel Encoders
5. ESP32 Encoder Reading
6. Closed-Loop Motor Speed Control
7. PID Motor Speed Control
8. Differential-Drive Kinematics
9. Wheel Odometry
10. IMU Fundamentals (optional V1)
11. Robot Power System
12. Raspberry Pi Fundamentals
13. Reliable Raspberry Pi–ESP32 Communication
14. Black-Tape Boundary Sensing
15. Camera and Perception Integration
16. SEEKR V1 System Integration & Behavior Architecture
17. Consolidated Implementation Checklist

## 1. DC Motors

### What is it?
A brushed DC motor converts electrical energy into rotational mechanical energy. Current flowing through coils in a magnetic field produces torque that rotates the shaft. Voltage influences speed, while current is related to torque.

### Why does our robot need it?
SEEKR uses two independently driven wheels for differential-drive movement. Geared DC motors reduce output RPM and increase available wheel torque, allowing controlled movement and sufficient driving force.

## 2. Motor Drivers & H-Bridges

### What is it?
A motor driver is an electronic circuit that uses low-power control signals to switch electrical power from a separate supply to a motor. An H-bridge is a four-switch arrangement that reverses the polarity across a DC motor to control its direction.

### Why does our robot need it?
The ESP32 GPIO pins cannot directly supply the voltage and current required by our geared DC motors. SEEKR needs two independently controlled H-bridge channels to drive its left and right wheels, allowing forward motion, reverse motion, and differential steering.

## 3. PWM Speed Control

### What is it?
Pulse Width Modulation (PWM) controls the average electrical drive by rapidly switching a signal between HIGH and LOW. Duty cycle is the percentage of each period spent HIGH, while frequency is the number of PWM cycles per second.

### Why does our robot need it?
SEEKR uses PWM from the ESP32 to command the motor driver and control the speed of each geared DC motor. Different PWM duty cycles allow the left and right wheels to run at different speeds for differential-drive movement and steering.

## 4. Wheel Encoders

### What is it?
A rotary encoder is a sensor that generates electrical signals as a motor shaft rotates. By counting these signals, the ESP32 can measure rotational displacement and calculate rotational speed. A quadrature encoder uses channels A and B, offset by 90 electrical degrees, so the direction of rotation can also be determined.

Important relationships:

- Wheel counts per revolution depend on the encoder resolution and the gearbox ratio.
- Wheel revolutions = encoder counts / counts per wheel revolution.
- Distance travelled = wheel revolutions × wheel circumference.
- Wheel RPM can be calculated from the number of wheel revolutions measured over a known time interval.

### Why does our robot need it?
SEEKR needs an encoder on each drive motor so the left and right wheel movement can be measured independently. Equal PWM commands do not guarantee equal wheel speeds, so encoder feedback allows the robot to detect and later correct these differences.

Encoders provide direct feedback for wheel-speed control and approximate wheel motion. They support more repeatable straight driving and turning, but cannot eliminate slip or trajectory error.

## 5. ESP32 Encoder Reading

### How does the ESP32 read an encoder?
Quadrature encoders generate digital HIGH/LOW signals on channels A and B. The ESP32 can detect these transitions using GPIO interrupts or its dedicated PCNT pulse-counting hardware. The phase relationship between channels A and B allows the ESP32 to determine rotation direction.

Different counting methods such as x1, x2, and x4 use different numbers of signal edges, which changes the effective encoder resolution.

The ESP32 can periodically read the accumulated encoder counts and use the change in counts over a known time interval to calculate wheel speed.

### Why does our robot need this?
SEEKR needs reliable and independent measurements of its left and right wheel motion. Since equal PWM commands do not guarantee equal wheel speeds, encoder measurements provide the feedback needed for accurate straight-line driving and controlled turning.

See Sections 6–9 for closed-loop control, PI/PID, kinematics, and optional odometry built on these measurements.

## 6. Closed-Loop Motor Speed Control

### What is it?
Closed-loop motor control uses encoder feedback to adjust PWM so that a motor's measured speed follows its desired speed.

The ESP32 repeatedly performs:

Measured RPM → Calculate error → Adjust PWM → Repeat

The control loop compares the desired wheel RPM against the encoder-derived RPM and adjusts motor drive accordingly. The error equation and PI/PID terms are developed in Section 7.

### Why does SEEKR need it?
Each drive wheel needs an independent speed-feedback loop, since equal PWM does not imply equal actual speed. This improves motion repeatability under changing loads and supply conditions. Accurate wheel RPM alone cannot prevent slip or guarantee a perfectly accurate trajectory; see Sections 7–9.

## 7. PID Motor Speed Control

### What is PID control?
PID is a closed-loop control method that calculates motor corrections using three components:

- **P (Proportional):** responds to the current error.
- **I (Integral):** responds to accumulated error over time.
- **D (Derivative):** responds to how quickly the measured value is changing.

The wheel-speed error is:

$$
e(t) = \mathrm{RPM}_{\text{target}} - \mathrm{RPM}_{\text{measured}}
$$

A practical motor command can be written as:

$$
\mathrm{PWM} = \mathrm{PWM}_{\text{base}} + P + I + D
$$

where:

$$
P = K_p e(t)
$$

$$
I = K_i \int e(t)\,dt
$$

$$
D = -K_d \frac{d\,\mathrm{RPM}_{\text{measured}}}{dt}
$$

The ESP32 repeatedly:

1. Measures wheel speed using the encoder.
2. Calculates the speed error.
3. Calculates the P, I, and D terms.
4. Updates the PWM command.
5. Repeats the process at a regular control interval.

The PWM output must also be limited to the valid range supported by the motor-control system.

### Why does SEEKR need it?
The ESP32 can run independent PI/PID wheel-speed controllers so commanded wheel speeds are followed despite motor differences, friction, varying load, and battery changes. This improves straight driving and repeatable turning, but not absolute positioning. **PI may be sufficient for V1**; whether a D term is beneficial must be determined experimentally because encoder-derived RPM can be noisy.

### Important PID behavior

#### Proportional gain — Kp
Increasing \(K_p\) makes the controller respond more strongly to the current error.

Too little \(K_p\):
- slow response
- larger remaining error

Too much \(K_p\):
- overshoot
- oscillation
- possible instability

#### Integral gain — Ki
The integral term accumulates error over time and helps eliminate persistent steady-state error.

Too much \(K_i\) can cause:

- overshoot
- oscillation
- integral windup

Integral windup happens when the controller continues accumulating error even though the PWM output has already reached its maximum or minimum limit.

#### Derivative gain — Kd
The derivative term reacts to how quickly the measured wheel speed is changing.

It can help reduce:

- overshoot
- rapid oscillations

However, derivative control is sensitive to measurement noise, which is why some motor-speed controllers use only PI control.

### Controller flow

The basic control loop for each SEEKR wheel is:

```text
Target RPM
    ↓
Compare with measured RPM
    ↓
Calculate error
    ↓
PI/PID controller
    ↓
PWM command
    ↓
Motor driver
    ↓
DC motor
    ↓
Encoder
    ↓
Measured RPM
    └──────────── feedback
```

## 8. Differential-Drive Kinematics

### What is differential-drive kinematics?
Differential-drive kinematics describes how the motion of two independently driven wheels determines the robot's linear and angular motion.

By changing the relative speeds of the left and right wheels, SEEKR can drive straight, follow curved paths, or rotate in place.

### Why does SEEKR need it?
SEEKR's motion controller needs to convert a desired forward velocity and turning rate into individual left and right wheel-speed setpoints.

The ESP32 can use inverse differential-drive kinematics to calculate the required wheel speeds, convert those speeds into RPM, and send the resulting setpoints to the separate left and right PI/PID motor controllers.

### Important equations and concepts

Wheel linear speed is related to wheel angular speed by:

$$
v_{\text{wheel}} = r\omega_{\text{wheel}}
$$

The robot's forward linear velocity is:

$$
v = \frac{v_L + v_R}{2}
$$

The robot's angular velocity is:

$$
\omega = \frac{v_R - v_L}{L}
$$

where:

- $v_L$ = left wheel linear velocity
- $v_R$ = right wheel linear velocity
- $L$ = effective track width between the wheels
- $r$ = wheel radius

To convert desired robot motion into wheel-speed commands:

$$
v_L = v - \frac{\omega L}{2}
$$

$$
v_R = v + \frac{\omega L}{2}
$$

Wheel linear velocity can be converted to RPM using:

$$
\mathrm{RPM} =
\frac{60v_{\text{wheel}}}{2\pi r}
$$

For curved motion, the instantaneous turning radius is:

$$
R = \frac{v}{\omega}
$$

for $\omega \neq 0$.

### Movement rules

- Equal positive wheel speeds → straight forward motion.
- Equal negative wheel speeds → straight reverse motion.
- Unequal wheel speeds → curved motion.
- Right wheel faster than left → robot turns left.
- Left wheel faster than right → robot turns right.
- Equal and opposite wheel speeds → rotation in place.
- Larger track width produces a smaller angular velocity for the same difference in wheel speeds.

### Control flow

```mermaid
flowchart TD
    A["Desired robot motion<br/>(v, ω)"]
    B["Differential-drive<br/>inverse kinematics"]
    C["Left wheel<br/>speed setpoint"]
    D["Right wheel<br/>speed setpoint"]
    E["Left PI/PID<br/>controller"]
    F["Right PI/PID<br/>controller"]
    G["Left motor PWM"]
    H["Right motor PWM"]
    I["Left motor"]
    J["Right motor"]

    A --> B
    B --> C
    B --> D
    C --> E
    D --> F
    E --> G
    F --> H
    G --> I
    H --> J
```
## 9. Wheel Odometry

### What is wheel odometry?
Wheel odometry estimates how SEEKR's position and orientation change using measurements from its left and right wheel encoders.

The robot's pose is represented as:

$$
(x,\ y,\ \theta)
$$

where:

- $x$ = horizontal position
- $y$ = vertical position
- $\theta$ = heading/orientation

Odometry starts from a known or assumed pose and repeatedly updates that estimate as the wheels move.

### Why might SEEKR use it?
Wheel odometry estimates relative travel and heading from encoder counts and can provide approximate pose telemetry to the Raspberry Pi. It may support search coverage and debugging, but **continuous (x, y, θ) tracking is not a mandatory V1 feature**; wheel-speed feedback is the immediate requirement.

### Important equations and concepts

Encoder counts can be converted into wheel travel:

$$
\Delta s_i =
\frac{\Delta N_i}{\mathrm{CPR}_i}
\cdot 2\pi r_i
$$

where:

- $\Delta N_i$ = change in encoder counts
- $\mathrm{CPR}_i$ = counts per wheel revolution
- $r_i$ = wheel radius

Average forward travel is:

$$
\Delta s =
\frac{\Delta s_L + \Delta s_R}{2}
$$

Change in heading is approximately:

$$
\Delta \theta =
\frac{\Delta s_R - \Delta s_L}{L}
$$

where $L$ is the effective track width between the wheels.

Using the robot's midpoint heading:

$$
\theta_{\mathrm{mid}}
=
\theta_{\mathrm{old}}
+
\frac{\Delta \theta}{2}
$$

the pose can be updated approximately as:

$$
x_{\mathrm{new}}
=
x_{\mathrm{old}}
+
\Delta s\cos(\theta_{\mathrm{mid}})
$$

$$
y_{\mathrm{new}}
=
y_{\mathrm{old}}
+
\Delta s\sin(\theta_{\mathrm{mid}})
$$

$$
\theta_{\mathrm{new}}
=
\theta_{\mathrm{old}}
+
\Delta\theta
$$

### Kinematics vs odometry
Differential-drive kinematics describes how wheel velocities produce robot linear and angular velocity.

Wheel odometry uses measured wheel movement over time to estimate the robot's changing pose.

### Main sources of error
- wheel slip
- incorrect wheel diameter
- unequal effective wheel radii
- incorrect effective track width
- missed or noisy encoder counts
- incorrect initial pose

These errors accumulate because every new pose estimate depends on the previous estimate.

## 10. IMU Fundamentals

### What is an IMU?
An Inertial Measurement Unit (IMU) typically contains:

- **Accelerometer:** measures specific force along the x, y, and z axes.
- **Gyroscope:** measures angular velocity around the x, y, and z axes.

For a flat differential-drive robot, **yaw** is the most important rotation because it represents the robot turning left or right.

### Why could SEEKR use one?
An IMU can measure chassis rotation independently of the wheel encoders.

This can help SEEKR:

- improve heading estimation
- detect disagreement between wheel motion and actual chassis rotation
- identify possible wheel slip
- improve future localization

An IMU is **optional for SEEKR V1** because encoder odometry can work without one.

### Important concepts

A gyroscope can estimate heading changes by integrating angular velocity:

$$
\theta_{\text{new}}
=
\theta_{\text{old}}
+
(\omega_z - b_z)\Delta t
$$

where:

- $\omega_z$ = measured yaw angular velocity
- $b_z$ = gyro bias
- $\Delta t$ = measurement interval

Gyroscope measurements accumulate drift over time because even small bias errors are integrated continuously.

Accelerometers are noisy and also measure gravity-related specific force. Double-integrating acceleration to estimate position usually produces rapidly growing error.

Basic sensor fusion can combine encoder and gyro measurements to improve heading estimation, although it does not completely eliminate drift.

### Main sources of error
- gyro bias
- accumulated heading drift
- accelerometer noise
- sensor mounting orientation
- calibration error
- sampling-time error

## 11. Robot Power System

### What is SEEKR's power system?
SEEKR's power system consists of the battery, voltage regulators, motor driver, wiring, connectors, switches, and protection devices that safely distribute electrical power to the motors, Raspberry Pi, ESP32, camera, and sensors.

### Why does power-system design matter?
Different components require different voltages and currents.

The motors can draw large startup or stall currents, while the Raspberry Pi and ESP32 require stable power. Poor power-system design can cause voltage drops, overheating, electrical noise, controller resets, or damaged components.

### Important concepts

Electrical power is:

$$
P = VI
$$

For constant power over a duration, energy is:

$$
E = Pt
$$

Approximate battery energy is:

$$
E_{\text{battery}} \approx V \times \mathrm{Ah}
$$

in watt-hours when voltage is in volts and capacity is in amp-hours.

Approximate runtime can be estimated using:

$$
t \approx \frac{E_{\text{usable}}}{P_{\text{average}}}
$$

Voltage drop across wiring is:

$$
V_{\text{drop}} = IR
$$

Power lost as heat in resistance is:

$$
P_{\text{heat}} = I^2R
$$

Other important concepts:

- **Series battery cells:** voltage adds while amp-hour capacity remains approximately the same.
- **Parallel battery cells:** amp-hour capacity adds while voltage remains approximately the same.
- **Buck converter:** efficiently reduces a higher DC voltage to a lower DC voltage.
- **Common ground:** connected electronics share a common voltage reference where required.
- **Stall current:** must be considered when selecting the motor driver, battery, wiring, and protection.
- **Protected battery pack:** SEEKR should use a commercially protected rechargeable battery pack rather than a custom pack made from loose lithium cells.

### Basic power architecture

```mermaid
flowchart TD
    Battery["Protected Battery Pack"]
    Fuse["Main Fuse"]
    Switch["Main Power Switch"]

    MotorBranch["Motor Power Branch"]
    LogicBranch["Logic Power Branch"]

    Driver["Motor Driver"]
    Motors["Left & Right Motors"]

    Regulator["Voltage Regulator"]
    Pi["Raspberry Pi 5"]
    ESP["ESP32"]
    Sensors["Camera & Sensors"]

    Battery --> Fuse
    Fuse --> Switch

    Switch --> MotorBranch
    Switch --> LogicBranch

    MotorBranch --> Driver
    Driver --> Motors

    LogicBranch --> Regulator
    Regulator --> Pi
    Regulator --> ESP
    Regulator --> Sensors
```

The final architecture will depend on the actual battery voltage and the voltage/current requirements of the selected components.

## 12. Raspberry Pi Fundamentals

### What is the Raspberry Pi's role in SEEKR?
The Raspberry Pi 5 is SEEKR's high-level computer.

It runs Linux and is responsible for:

- hosting the web interface
- receiving the user's target selection
- acquiring camera data
- running object detection and perception
- running the search algorithm
- maintaining the high-level robot state machine
- deciding the desired robot motion
- communicating with the ESP32
- receiving and logging robot telemetry

### Why use both a Raspberry Pi and ESP32?

The two controllers have different jobs.

**Raspberry Pi 5:**
- computationally demanding tasks
- computer vision
- networking
- web interface
- search and decision-making
- high-level robot behavior

**ESP32:**
- motor control
- PWM generation
- encoder acquisition
- PI/PID control loops
- boundary-sensor monitoring
- immediate low-level safety behavior

Linux does not guarantee perfectly consistent execution timing because many processes share the processor. This can create **timing jitter**.

The ESP32 is therefore better suited for the timing-sensitive control loops that directly operate the robot's hardware.

### Important Concepts

- **Single-board computer (SBC):** A complete computer built onto one circuit board.
- **Operating system:** Software that manages the computer's hardware and running applications.
- **Linux process:** A currently running program.
- **Timing jitter:** Variation in when a task actually executes.
- **GPIO:** General-purpose digital input/output pins. Raspberry Pi GPIO operates using 3.3 V logic.
- **USB:** A standard interface that can connect compatible devices and may also be used for serial communication.
- **UART:** A serial communication interface using transmit and receive signals.
- **Wi-Fi / HTTP:** Allows a user's browser to communicate with SEEKR's web interface.
- **Camera interface:** Allows the Raspberry Pi to receive image data from the robot's camera.
- **SSH:** Allows remote terminal access to the Raspberry Pi over a network.
- **systemd:** Linux service manager that can automatically start SEEKR software when the Pi boots.
- **Watchdog / timeout:** A safety mechanism that detects lost communication or stalled software and can force the robot into a safe state.

### Pi–ESP32 Integration Boundary

The Raspberry Pi runs application-level decisions; the ESP32 runs time-sensitive control and safety. The Pi can use **USB serial or direct UART**, with the exact physical connection selected during implementation. Communication basics and reliability are gathered together in Section 13 to avoid two competing definitions.

### High-Level Behavior Ownership

The Pi owns mission transitions, while the ESP32 enforces local motor safety. The **single authoritative V1 state diagram** is in Section 16; it ends at verified FOUND, not APPROACH.

### SEEKR's High-Level Data Flow

1. The user selects a target through the web interface.
2. The Raspberry Pi receives the target selection.
3. The camera provides image frames to the Raspberry Pi.
4. The perception system evaluates the images for the selected object.
5. The search and behavior logic determines the desired robot movement.
6. The Raspberry Pi sends a motion command to the ESP32.
7. The ESP32 controls the motors and monitors encoders and boundary sensors.
8. The ESP32 sends telemetry and safety events back to the Raspberry Pi.
9. The Raspberry Pi updates the robot state and web interface.

### Startup Sequence

A simplified SEEKR startup sequence is:

```mermaid
flowchart TD
    A["Power On"] --> B["ESP32 Initializes<br/>Motors Remain Stopped"]
    B --> C["Raspberry Pi Boots Linux"]
    C --> D["SEEKR Software Starts"]
    D --> E["Camera and Serial Communication Initialize"]
    E --> F{"Pi and ESP32 Ready?"}
    F -->|No| G["Remain Stopped<br/>Wait or Report Fault"]
    G --> F
    F -->|Yes| H["IDLE<br/>Ready for Mission"]

    style A fill:#2563eb,color:#fff,stroke:#1d4ed8
    style B fill:#374151,color:#fff,stroke:#6b7280
    style C fill:#374151,color:#fff,stroke:#6b7280
    style D fill:#374151,color:#fff,stroke:#6b7280
    style E fill:#374151,color:#fff,stroke:#6b7280
    style F fill:#d97706,color:#fff,stroke:#b45309
    style G fill:#b91c1c,color:#fff,stroke:#991b1b
    style H fill:#15803d,color:#fff,stroke:#166534
```

The ESP32 must keep the motors stopped until both controllers complete their readiness checks. SEEKR only enters the IDLE state when the required systems are ready.

## 13. Reliable Raspberry Pi–ESP32 Communication

### How do the Raspberry Pi and ESP32 communicate?

Through bidirectional serial communication (USB serial or direct UART).

- **Raspberry Pi → ESP32:** Motion commands, STOP, and control information.
- **ESP32 → Raspberry Pi:** Telemetry, readiness, boundary events, and fault information.

### Why does SEEKR need a communication protocol?

Serial communication transfers bytes, not complete commands automatically.

A protocol defines message boundaries, data formats, command meanings, validation rules, and failure behavior so both controllers can communicate reliably and safely.

### Serial Foundations (Learned During Raspberry Pi Fundamentals)

- **Serial communication:** SEEKR's Raspberry Pi and ESP32 exchange a stream of bytes using either USB serial or direct UART, depending on the final hardware configuration.
- **TX and RX:** TX transmits data and RX receives data.
- **Baud rate:** Defines the communication rate. Both devices must use compatible communication settings.
- **8N1:** A common UART configuration using 8 data bits, no parity bit, and 1 stop bit.
- **Common ground:** Direct UART communication requires the two controllers to share a voltage reference unless isolation is used.
- **Byte stream:** Serial communication does not automatically provide complete application-level messages. The software must determine where messages begin and end.
- **Message framing:** A rule used to separate individual messages. A simple example is terminating each message with a newline character.
- **Commands:** Messages from the Raspberry Pi requesting robot actions, such as movement or stopping.
- **Telemetry:** Information sent from the ESP32 to the Raspberry Pi, such as wheel speeds, encoder information, odometry, boundary states, and faults.
- **Different update rates:** Motor-control loops run much faster than high-level commands and telemetry. These tasks do not need to operate at the same frequency.
- **Heartbeat:** A periodic message indicating that communication is still active.
- **Communication timeout:** If valid control communication disappears for too long, the ESP32 should place the robot into a safe stopped state.
- **Startup handshake:** The Raspberry Pi and ESP32 should confirm readiness before normal autonomous operation begins.

These concepts establish the basic communication architecture. Reliable parsing, acknowledgements, corruption detection, recovery, reconnection, and protocol design are covered in Step 16.

### Reliability Concepts

- **Framing:** Defines where a message starts and ends.
- **Buffering:** Temporarily stores incoming bytes until a complete message is received.
- **Parsing:** Extracts command types and data fields.
- **Validation:** Rejects malformed, incomplete, unknown, non-finite, or unsafe commands.
- **Acknowledgement (ACK):** Confirms a defined level of message acceptance, not necessarily completion of the requested physical action.
- **Sequence number:** Helps identify duplicate, missing, and obsolete messages.
- **Checksum / CRC:** Detects many accidental transmission errors but does not guarantee delivery or safety.
- **Command freshness:** Movement commands must remain sufficiently recent to authorize motion.
- **Heartbeat:** Indicates communication health but does not independently authorize movement.
- **Session synchronization:** Establishes a valid communication context after startup, reset, or reconnection.
- **Idempotence:** Repeating certain commands, such as STOP, should not create unintended additional effects.

### What should the Raspberry Pi send?

- Desired linear and angular velocity, or left/right wheel-speed targets.
- STOP commands.
- Movement authorization and session information.
- Startup synchronization and configuration commands.

The Raspberry Pi should not continuously send raw motor PWM values. Low-level motor control belongs to the ESP32.

### What should the ESP32 return?

- Controller readiness and communication status.
- Measured wheel speeds.
- Encoder measurements.
- Boundary-sensor readings and events.
- Fault and safety states.
- Optional wheel-odometry estimates.
- Acknowledgements for selected commands.

### Safety Behavior

The ESP32 must enforce safety independently of the Raspberry Pi.

- Validate commands before execution.
- Reject invalid commands without changing motor targets.
- Stop the motors when fresh valid movement commands stop arriving.
- Allow boundary protection to override normal movement requests.
- Prevent outdated commands from restarting motion after STOP.
- Keep motor-driver outputs safe during startup and controller resets.
- Require synchronization after faults, resets, and reconnection.
- Never automatically resume previous motion after reconnection.

## 14. Black-Tape Boundary Sensing

### What is boundary sensing?

Boundary sensing allows SEEKR to detect the black-tape perimeter of its wooden-floor arena using downward-facing infrared (IR) reflectance sensors.

An IR emitter illuminates the surface, while a receiver measures reflected infrared light. Different materials produce different readings, but visible color does not necessarily determine infrared reflectance.

The actual floor and tape must be tested before selecting detection thresholds.

### Why does SEEKR need it?

SEEKR must autonomously search for objects while remaining inside its designated arena.

The ESP32 monitors boundary sensors independently of the Raspberry Pi, allowing it to execute protective behavior even when high-level software is delayed or unresponsive.

Boundary safety always takes priority over ordinary movement commands.

### Important Concepts

- **IR reflectance:** Amount of infrared light reflected from a surface.
- **Analog output:** Variable electrical measurement read through an ADC.
- **Digital output:** Binary classification, often produced by a comparator.
- **Threshold:** Measurement used to distinguish tape from flooring.
- **Calibration:** Measuring actual floor and tape responses to determine reliable thresholds.
- **Hysteresis:** Different thresholds for entering and leaving a detection state, reducing rapid switching.
- **Filtering:** Reduces measurement noise but may introduce detection delay.
- **False positive:** Detecting tape when the sensor is over the floor.
- **False negative:** Failing to detect tape when the sensor is over it.
- **Detection latency:** Delay between a detectable boundary condition and the start of the protective response.
- **Stopping distance:** Distance traveled during detection delay and braking.
- **Swept envelope:** The complete area occupied by SEEKR's chassis, wheels, caster, and other protrusions as it moves or rotates.
- **Boundary inhibit:** Protective state preventing ordinary motion commands from overriding a boundary event.

### Sensor Placement and Calibration

Boundary coverage must consider the robot's **entire physical footprint**, not just its front wheels.

Important considerations:

- forward and reverse movement
- curved movement and rotation
- wheels, caster, and chassis overhang
- tape width and sensor position
- detection latency and braking distance
- required safety margin

Four sensor regions near the chassis corners are an initial layout candidate, **not a finalized design**.

The final number and placement of sensors will depend on chassis geometry and real-world testing.

Calibration will involve measuring floor and tape reflectance under different lighting conditions, mounting heights, operating speeds, and floor locations.

### Boundary Safety Behavior

The ESP32 must enforce boundary protection independently of the Raspberry Pi.

When an unsafe boundary condition occurs:

1. The ESP32 inhibits ordinary movement commands.
2. The motors execute a safe stop.
3. The ESP32 reports the boundary event to the Raspberry Pi.
4. Boundary protection remains active until recovery conditions are satisfied.
5. The Raspberry Pi may request a cautious recovery maneuver.
6. The ESP32 permits only motion consistent with its safety restrictions.

Reversing is not automatically safe. If the safe recovery direction cannot be established, SEEKR must remain stopped.

Critical sensor faults or communication failures must prevent unsafe autonomous movement.

**Detecting the tape does not automatically guarantee that SEEKR remains inside the arena.** Physical containment must be verified using real stopping-distance and sensor-coverage tests.

## 15. Camera and Perception Integration

### What is the camera's role in SEEKR?

The camera captures images of SEEKR's arena and target objects. These images are processed by Eli's perception software on the Raspberry Pi 5.

Important camera characteristics include:

- **Resolution:** Affects image detail and processing workload.
- **Frame rate:** Determines capture frequency, which may differ from object-detection frequency.
- **Field of view:** Determines how much of the environment is visible.
- **Mounting position and orientation:** Affect the interpretation of target locations.
- **Lighting, focus, and motion blur:** Affect detection reliability.

Camera-to-robot alignment must be checked after installation.

### What does the perception system output?

The perception subsystem should produce structured observations containing:

- target identity or class
- detection confidence
- bounding-box coordinates
- target-center coordinates, in pixels or normalized units
- observation timestamp
- optional frame identifier
- explicit `DETECTED`, `NO_DETECTION`, or `ERROR` status

A detection describes what the camera observed; it is **not a motor command**.

Detection confidence does not necessarily represent a calibrated probability of correctness.

### Important Concepts

- **Frame:** An image captured at a particular time.
- **FPS:** Frames captured or processed per second.
- **Field of view:** Angular extent visible to the camera.
- **Classification:** Identifies image content.
- **Object detection:** Identifies objects and estimates their image locations.
- **Bounding box:** Rectangle approximately enclosing a detected object.
- **Normalized coordinates:** Image coordinates scaled relative to image dimensions.
- **Alignment error:** Difference between the object's horizontal image position and the desired position.
- **Deadband:** Small acceptable range of alignment error.
- **Observation freshness:** Whether a detection is recent enough to be trusted for a decision.
- **False positive:** Incorrectly reporting a target.
- **False negative:** Failing to detect a visible target.
- **VERIFY:** Confirming a candidate target using additional trustworthy observations.

### How Perception Connects to Robot Motion

```mermaid
flowchart TD
    A["Camera Captures Frame"] --> B["Perception System<br/>Eli"]
    B --> C["Structured Detection Result"]
    C --> D["Pi Behavior and Search Logic<br/>Eli"]
    D --> E["Desired Motion or STOP"]
    E --> F["Serial Communication"]
    F --> G["ESP32 Motion Control<br/>Toufic"]
    G --> H["Motor Driver and Motors"]
    H --> I["Physical Robot Movement"]
    G --> J["Encoder and Safety Telemetry"]
    J --> D
```

The Pi evaluates the selected target, detection confidence, observation freshness, and robot state before requesting movement.

The ESP32 translates permitted motion requests into wheel-speed targets, regulates the motors, and enforces low-level safety.

**Perception does not directly control motor PWM.**

### Integration and Reliability

- Maintain clear interfaces between perception, behavior logic, and motor control.
- Use consistent image and robot coordinate conventions.
- Distinguish frame capture time from processing completion time.
- Avoid processing excessive backlogs of outdated frames.
- Never treat stale detections as fresh observations.
- A working serial connection does not guarantee healthy perception.
- Inhibit perception-dependent motion when observations become untrustworthy.
- Distinguish `NO_DETECTION` from a perception-system failure.
- Use ESP32 telemetry to diagnose motion-control problems.
- Boundary and communication safety override camera-driven movement.
- Test perception using recorded images and simulated detection results before full robot integration.

### FOUND, ALIGN, and APPROACH

The baseline SEEKR V1 objective is to **locate and visually confirm the user-selected object**.

- **VERIFY:** Confirms that the selected object has been reliably detected.
- **FOUND:** Reports that the required success condition has been satisfied.
- **ALIGN:** Optionally rotates the robot to center the target.
- **APPROACH:** Optionally moves toward the target, requiring additional distance-estimation and safety considerations.

**Current working V1 scope:** VERIFY followed by a safe stop and FOUND. ALIGN and APPROACH are **deferred**, unless Toufic and Eli explicitly change the agreed requirements before implementation.

## 16. SEEKR V1 System Integration & Behavior Architecture

### Complete SEEKR V1 Architecture

SEEKR uses two controllers with separate responsibilities:

**Raspberry Pi 5:**
- Web interface and target selection
- Camera capture and perception
- Search algorithm and high-level behavior
- Mission state machine
- Motion requests and telemetry processing

**ESP32:**
- Motor-driver control
- Wheel-encoder acquisition
- Closed-loop wheel-speed regulation
- Boundary-sensor monitoring
- Independent motion safety
- Communication watchdog and fault reporting

The robot uses two geared DC motors with encoders and one passive caster.

A protected battery and voltage-regulation system provide suitable power to the electronics and motors.

The Pi sends authorized movement requests to the ESP32. The ESP32 executes permitted movement and returns telemetry, readiness, and fault information.

Hardware interfaces define electrical and physical connections. Software interfaces define exchanged messages and expected behavior.

### Robot State Machine

SEEKR's high-level mission logic runs on the Raspberry Pi.

```mermaid
flowchart TD
    IDLE["IDLE / Ready"] -->|"Authorized start"| SEARCH["SEARCH"]
    SEARCH -->|"Selected-target candidate"| VERIFY["VERIFY"]
    VERIFY -->|"Rejected / insufficient evidence"| SEARCH
    VERIFY -->|"Confirmed selected target + safe stop"| FOUND["FOUND"]
    SEARCH -->|"User stop"| STOPPED["STOPPED"]
    VERIFY -->|"User stop"| STOPPED
    SEARCH -->|"Critical fault"| FAULT["FAULT"]
    VERIFY -->|"Critical fault"| FAULT
    STOPPED -->|"Reset + ready"| IDLE
    FAULT -->|"Cleared + reauthorized"| IDLE
```

- **IDLE:** Await a new authorized mission with motors stopped.
- **SEARCH:** Autonomously explore the arena and look for the selected object.
- **VERIFY:** Confirm or reject a selected-target candidate using fresh trustworthy observations.
- **FOUND:** Verification succeeded, the robot is safely stopped, and success can be reported.
- **STOPPED:** Normal mission interruption, with motor movement inhibited.
- **FAULT:** Abnormal condition requiring recovery/clearance and fresh authorization.

**ALIGN and APPROACH are not V1 states.** They are planned optional future-version extensions; neither is needed to demonstrate visual search. Pi mission transitions do not override ESP32-side boundary or communication safety.

Each state still requires implementation-level entry conditions, exit conditions, and permitted actions.

### Subsystem Interfaces and Team Responsibilities

**Eli — High-Level Software**
- Computer vision and perception
- Machine learning
- Search algorithms
- High-level robot behavior

**My Responsibilities — Hardware and Embedded Systems**
- Electronics and power distribution
- ESP32 firmware
- Motor and encoder control
- Differential-drive motion
- Boundary sensing
- Chassis and mechanical integration
- Pi–ESP32 communication integration
- Physical testing and calibration

**Shared Responsibilities**
- System architecture
- Interface contracts
- Safety coordination
- Integration testing
- Mission acceptance criteria

Perception produces structured observations. High-level logic converts trustworthy observations into desired movement. The ESP32 validates motion requests, controls the motors, and reports physical feedback.

Shared interfaces must define units, coordinate conventions, timestamps, readiness, command validity, and fault information.

Both sides should support independent testing with simulated inputs.

### Safety and Fault Coordination

- Motor outputs must default to a safe state during startup and controller resets.
- The ESP32 independently enforces boundary protection.
- Missing fresh authorized motion commands must cause a safe stop.
- The Pi must monitor perception freshness and detect stalled or unreliable observations.
- Critical faults inhibit unsafe movement and prevent automatic restart.
- Boundary recovery permits only explicitly safe motion.
- Significant wheel-speed tracking errors require investigation and protective handling.
- Reset or reconnection requires synchronization before new movement is authorized.
- Physical containment must be verified using actual stopping distances and full chassis dimensions.

A functioning serial connection does not guarantee functioning perception, motors, or sensors.

### Integration and Testing Strategy

SEEKR will be integrated progressively:

1. Inspect power connections and verify safe initial power-up.
2. Test one DC motor and its encoder.
3. Test both motors independently and together.
4. Implement and tune closed-loop wheel-speed control.
5. Validate differential-drive movement and basic odometry.
6. Calibrate boundary sensors.
7. Test boundary stopping, containment, and recovery.
8. Test Pi–ESP32 commands, telemetry, timeouts, and resets.
9. Test simulated high-level movement requests.
10. Test the camera and perception subsystem independently.
11. Integrate Eli's high-level software with the embedded system.
12. Conduct repeated autonomous-search trials.

Testing should record configurations, measured results, failures, and relevant logs.

### Transition to Implementation

The theory block is complete; no physical subsystem is claimed as built or validated. The first physical milestone is **safe bench power, one geared DC motor, and encoder acquisition with the ESP32**. The next section collects the remaining decisions and tests into a single checklist rather than repeating "still need to learn" after each topic.

## 17. Consolidated Implementation Checklist

> This checklist is **work still to build, decide, calibrate, or test**, not theory the team has yet to study. Exact models, acceptance thresholds, firmware, and system behavior will be refined from measured results. Ownership and current V1 scope remain governed by [requirements.md](requirements.md).

### A. Before Hardware Procurement (Shared, Toufic Leads Hardware)

- [ ] Confirm visual verification → safe stop → FOUND as the working V1 success criterion with Eli; only change ALIGN/APPROACH scope explicitly.
- [ ] Review compatible motor ratings, wheel geometry, encoder signal levels/count conventions, dual-channel driver continuous/peak currents, and safe braking.
- [ ] Select a protected battery/bench supply, motor and logic rails, appropriately rated regulators, fuses, switches, connectors, and wiring; complete a power budget.
- [ ] Plan mechanical footprint, track width, camera location, tape width, boundary-sensor placement and expected stopping margin.
- [ ] Specify the first bench wiring diagram and safe stop procedure before energizing motors.
- [ ] Track confirmed components and datasheets in [hardware.md](hardware.md).

### B. Embedded System and Mechanical Bring-up (Toufic)

- [ ] Wire and test one motor and encoder with a protected, current-limited bench supply.
- [ ] Verify encoder resolution/count direction, voltage compatibility, reliable ESP32 acquisition, and wheel speed measurements.
- [ ] Add the second drive motor and tune independent PI/PID speed regulation; verify PWM saturation, braking, and stop behavior.
- [ ] Build a stable differential-drive chassis; measure wheel radii/track width and characterize straight motion/turning.
- [ ] Implement and optionally report wheel odometry if it adds value for V1 search and debugging.
- [ ] Select and calibrate boundary sensors against the *actual* floor/tape; confirm full chassis coverage, latency, stopping distance, and safe recovery.
- [ ] Check power stability and motor-safe startup/reset outputs.

### C. Pi, Communication, and Interfaces (Shared)

- [ ] Configure Raspberry Pi OS, SSH, camera capture, application startup, and logging.
- [ ] Choose USB serial or direct UART, define unambiguous velocity units/signs, frame schema, sequence/session rules, integrity checks, and timeouts in [protocol.md](protocol.md).
- [ ] Implement and test Pi–ESP32 parsing, telemetry, STOP, authorization, resets, command freshness, and reconnection without unsafe auto-resume.
- [ ] Agree with Eli on perception outputs, image coordinate conventions, capture timestamps/frame IDs, NO_DETECTION vs ERROR, and observation-age rules.
- [ ] Integrate Eli's search and VERIFY logic with a simulated or bench-tested motion interface. Record shared decisions in [integration.md](integration.md).

### D. Full V1 Validation (Shared)

- [ ] Define and test a real, repeatable verification rule for the selected one of four objects.
- [ ] Distinguish ordinary STOPPED from abnormal FAULT, and verify startup/readiness/fault handling.
- [ ] Test bad sensor signals, stale frames, dropped serial commands, motor/encoder faults, and boundary sensor failures.
- [ ] Run low-speed supervised trials, validate physical containment, and then repeated full mission trials.
- [ ] Measure verified search success, false FOUND, time to detection, containment violations, braking behavior, and relevant fault/telemetry logs; set quantitative acceptance criteria from real tests.

### E. Deferred / Future Versions (Not V1 Dependencies)

- Optional IMU installation, gyro calibration, and encoder/IMU heading fusion.
- Continuous localization beyond useful encoder-based estimates, SLAM, LiDAR, obstacle avoidance, mapping, or ROS 2.
- Camera-based ALIGN, APPROACH, near-target stopping, and later gripper/object retrieval.

**Completion boundary:** The reference concepts in Sections 1–16 have been learned. The checklist above is the remaining engineering work for V1; do not interpret unchecked items as an invitation to restart the theory curriculum.
