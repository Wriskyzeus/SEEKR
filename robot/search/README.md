# High-Level Search and Mission Behavior

This Raspberry Pi-side directory is primarily Eli's responsibility. It will contain the search strategy, mission state machine, and target-verification behavior for SEEKR V1.

**V1 behavior:** IDLE → SEARCH → VERIFY → FOUND, with STOPPED/FAULT handling. A candidate of the selected target must be verified before FOUND, and the robot must stop safely and report the result.

ALIGN/APPROACH and target retrieval are deferred to a later version; they are not mandatory in the current V1 mission.

The search software requests robot motion through the agreed Pi–ESP32 interface. It never directly drives motor PWM or bypasses ESP32 boundary or command-timeout protection. See `docs/integration.md`, `docs/architecture.md`, and `docs/requirements.md`.
