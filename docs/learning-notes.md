# Robotics & Embedded Systems Learning Notes

These notes document the concepts I learn while building the autonomous mapping robot.

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

Encoders will support accurate straight-line driving, controlled turning, closed-loop speed control, and wheel odometry.

### What I still need to understand
Practical encoder wiring, ESP32 interrupts and pulse counters, motor calibration, PID control, differential-drive odometry, and reducing errors caused by wheel slip, electrical noise, and missed encoder counts.