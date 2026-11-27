# SEEKR V1 Hardware Selection and Power Budget

**Status:** Step 20 planning template. **No final products, ratings, or purchases have been approved.**

## Selection Principles

- Select components from real manufacturer specifications, not appearance or a generic shopping list.
- Verify supply voltage, logic levels, continuous/peak current, connectors, mechanics, and mounting geometry together.
- Use a commercially protected battery pack and appropriate fusing/switching. Do not assemble a loose-cell lithium pack for V1.
- Keep a bench-test configuration separate from a complete battery-powered chassis.
- The initial physical milestone is one motor and one encoder with an ESP32, suitably rated driver and protected supply.

## Component Decision Log

| Subsystem | Planned function | Key specifications to confirm | Chosen part | Status |
| --- | --- | --- | --- | --- |
| ESP32 | Motor/sensor control | Pin count, PWM, encoder/PCNT, ADC, logic levels | TBD | Not selected |
| Raspberry Pi 5 | Vision and high-level behavior | Power, camera/interface, serial connectivity | Raspberry Pi 5 (planned) | Board acquisition TBD |
| DC gearmotors ×2 | Wheel drive | Rated voltage, unloaded/load/stall current, torque, RPM, gearbox | TBD | Not selected |
| Encoders ×2 | Wheel feedback | Voltage, signal type, CPR/PPR definition, motor/wheel placement | TBD | Not selected |
| Motor driver | Two motor channels | Supply voltage, continuous/peak/stall current, braking behavior, input logic | TBD | Not selected |
| Wheels + caster | Differential-drive chassis | Wheel diameter, traction, alignment, caster ground clearance | TBD | Not selected |
| Battery + protection | Mobile power | Protected pack, voltage range, current delivery, capacity, charging method | TBD | Not selected |
| Regulators | Logic power | Output voltage/current, input range, transient behavior, Pi margin | TBD | Not selected |
| Boundary sensors | Black-tape safety | Analog/digital output, mounting height, IR contrast, coverage | TBD | Not selected |
| Camera | Target recognition | Pi compatibility, FoV, resolution, focus, cable/mount | TBD | Not selected |
| Fuse, switch, wiring | Safety and distribution | Rated current/voltage, wire gauge, connectors, accessibility | TBD | Not selected |
| Chassis materials | Mechanical support | Mass, rigidity, mounts, footprint, service access | TBD | Not selected |

## Power-Budget Worksheet (Fill After Selection)

| Rail / load | Voltage | Typical current | Peak/startup current | Source / regulator |
| --- | --- | --- | --- | --- |
| Left motor | TBD | TBD | TBD | Motor power branch |
| Right motor | TBD | TBD | TBD | Motor power branch |
| Raspberry Pi + camera | TBD | TBD | TBD | Stable logic regulator |
| ESP32 | TBD | TBD | TBD | Logic regulator |
| Boundary sensors and peripherals | TBD | TBD | TBD | Compatible regulated rail |

Budget motor stall/startup separately from average runtime, size wiring and protection for relevant fault/current conditions, and confirm regulators are not exposed to motor disturbances.

## Mechanical and Sensor Constraints

- Two drive wheels and one passive caster.
- Record wheel radius, effective track width, chassis footprint, camera mount, battery center of mass, and ground clearance.
- Determine tape width, sensor mounting height, swept robot envelope, and stopping margin by physical testing.
- Sensor positions and count are **not** finalized; any four-corner arrangement is an evaluation candidate.

## Decision Procedure

For each part, record the exact model, manufacturer datasheet/link, source, quantity, price, compatibility checks, and reason for selection. Update this file when decisions are approved by the team.

**First procurement target:** only the parts needed for a safe single-motor/encoder bench test, unless an earlier purchase is clearly justified by compatibility and cost.
