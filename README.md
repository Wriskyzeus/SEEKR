# SEEKR — Autonomous Visual Search Robot

> A mobile robot that autonomously searches a bounded indoor arena for a user-selected object.

SEEKR is an engineering project that combines embedded systems, mobile robotics, computer vision, and a web-based control interface. A user selects one of four known target objects; the robot then explores a constrained search area, detects the requested object with an onboard camera, approaches it, stops nearby, and reports success.

## Project Overview

Version 1 is intentionally scoped to a flat, bounded indoor arena with a black tape perimeter and no obstacles. This controlled setting lets the us validate the essential system loop—perception, embedded motor control, boundary handling, communication, and autonomous search—before taking on SLAM, obstacle avoidance, and operation in less structured environments.

The Raspberry Pi 5 serves as the high-level computer, running Linux, vision, search logic, and the web interface. An ESP32 is responsible for time-sensitive low-level work such as motor control, encoder feedback, and boundary-sensor processing. The two controllers communicate over serial/UART.

## Problem Being Solved

Finding a specific object in an indoor space is a foundational robotics task. It requires a robot to connect a human intent ("find this object") to perception, movement, safety constraints, and a clear completion condition.

SEEKR addresses this problem in a deliberately constrained environment. Rather than claiming general-purpose autonomy immediately, the project establishes a reliable baseline for visual target search and approach. That baseline is intended to make future work on mapping, path planning, and object manipulation more measurable and less ambiguous.

