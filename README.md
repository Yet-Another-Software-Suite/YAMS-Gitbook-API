# YAMS

YAMS (Yet Another Mechanism Suite) is a WPILib vendordep library for FRC robots. It provides a unified motor controller abstraction over REV and CTRE hardware, pre-built mechanism classes for common robot subsystems, and advanced control algorithms — all with simulation support and automatic NetworkTables telemetry. Both Java and C++ are supported with parallel APIs.

## Features

- **Motor controller abstraction** — a single `SmartMotorController` interface over SPARK MAX, SPARK FLEX, TalonFX, and TalonFXS
- **Mechanism classes** — `Arm`, `Elevator`, `Pivot`, `FlyWheel`, `DoubleJointedArm`, `DifferentialMechanism`, and `SwerveDrive`
- **Advanced control** — LQR, exponential motion profiles, trapezoidal profiles, and a derivative-time filter
- **Extended units** — `YUnits` constants and `EasyCRT` multi-encoder absolute position solver
- **Simulation** — built-in sim support for all mechanism classes via WPILib's simulation framework

## Quick Navigation

- [Java API](api/java/README.md)
- [C++ API](api/cpp/README.md)
- [Guides](guides/README.md)

## Supported Hardware

| Device | Vendor | Interface |
|---|---|---|
| SPARK MAX | REV Robotics | CAN |
| SPARK FLEX | REV Robotics | CAN |
| TalonFX | CTR Electronics | CAN |
| TalonFXS | CTR Electronics | CAN |
