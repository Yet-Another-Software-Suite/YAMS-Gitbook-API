# ElevatorConfig

**Package:** `yams.mechanisms.config`

`ElevatorConfig` configures a linear elevator mechanism. All builder methods return the `ElevatorConfig` instance for chaining.

See also: [Elevator](../mechanisms/elevator.md)

---

## Constructor

```java
ElevatorConfig()
ElevatorConfig clone()  // Deep copy
```

---

## Builder Methods

| Method | Parameters | Description |
|--------|-----------|-------------|
| `withCarriageWeight(Mass mass)` | `mass` — carriage mass | **Required for simulation.** Used by `ElevatorSim` to model gravity. |
| `withHardLimits(Distance min, Distance max)` | `min` — bottom limit, `max` — top limit | Sets the physical height range. The motor will not command positions outside these bounds. |
| `withAngle(Angle angle)` | `angle` — angle from horizontal | Elevator mounting angle. Defaults to 90° (vertical). Optional. |
| `withHorizontalElevator()` | _(none)_ | Disables gravity simulation. Useful for horizontal linear sliders. Optional. |
| `withTelemetry(String name, TelemetryVerbosity verbosity)` | `name` — NT key prefix, `verbosity` — `LOW`/`MEDIUM`/`HIGH` | Enables NetworkTables telemetry for this mechanism. Optional. |
| `withSimColor(Color8Bit color)` | `color` — RGB color | Sets the Mechanism2d visualization color. Optional. |
| `withMechanismPositionConfig(MechanismPositionConfig config)` | `config` — position config | Configures the visualization plane and 3D offset for this mechanism. Optional. |

> **Note:** Drum radius (spool size) is configured on `SmartMotorControllerConfig` via `.withDrumRadius(Distance chainPitch, int teeth)` or `.withMechanismCircumference(Distance)`. Cascading elevator stages are configured via `SmartMotorControllerConfig.withCascadingElevatorStages(int stages)`.

---

## Getters

All getters return `Optional<>` for configured values, following the same pattern as [ArmConfig](arm-config.md) getters. `getSimColor()` and `getMechanismPositionConfig()` always return a value.

---

## Example

```java
ElevatorConfig config = new ElevatorConfig()
    .withCarriageWeight(Kilograms.of(4.5))
    .withHardLimits(Meters.of(0.0), Meters.of(1.2))
    .withTelemetry("Elevator", TelemetryVerbosity.HIGH);
```

{% @github-files/github-code-block url="https://github.com/Yet-Another-Software-Suite/YAMS/blob/master/examples/simple_elevator/java/frc/robot/subsystems/ElevatorSubsystem.java#L113-L126" %}
