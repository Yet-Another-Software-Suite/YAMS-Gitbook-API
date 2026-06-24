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
| `withDrumRadius(Distance radius)` | `radius` — spool/drum radius | Sets the drum radius for converting motor rotations to linear distance. Required when not using `withMechanismCircumference` on the motor config. |
| `withAngle(Angle angle)` | `angle` — angle from horizontal | Elevator mounting angle. Defaults to 90° (vertical). Set to 0° for horizontal elevators. Optional. |
| `withStages(int numStages)` | `numStages` — number of cascading stages | Number of cascading stages (e.g., `2` for a 2-stage elevator). Multiplies the effective travel distance. Optional. |
| `withHorizontalElevator(boolean horizontal)` | `horizontal` | When `true`, disables gravity simulation. Useful for horizontal linear sliders. Optional. |
| `withTelemetry(String name, TelemetryVerbosity verbosity)` | `name` — NT key prefix, `verbosity` — `LOW`/`MEDIUM`/`HIGH` | Enables NetworkTables telemetry for this mechanism. Optional. |
| `withSimColor(Color8Bit color)` | `color` — RGB color | Sets the Mechanism2d visualization color. Optional. |
| `withMechanismPositionConfig(MechanismPositionConfig config)` | `config` — position config | Configures the visualization plane and 3D offset for this mechanism. Optional. |

---

## Getters

All getters return `Optional<>` for configured values, following the same pattern as [ArmConfig](arm-config.md) getters. `getSimColor()` and `getMechanismPositionConfig()` always return a value.

---

## Example

```java
ElevatorConfig config = new ElevatorConfig()
    .withCarriageWeight(Kilograms.of(4.5))
    .withHardLimits(Meters.of(0.0), Meters.of(1.2))
    .withDrumRadius(Meters.of(0.02286))  // ~0.9 inch radius
    .withStages(2)
    .withTelemetry("Elevator", TelemetryVerbosity.HIGH);
```

{% @github-files/github-code-block url="https://github.com/Yet-Another-Software-Suite/YAMS/blob/master/examples/simple_elevator/java/frc/robot/subsystems/ElevatorSubsystem.java#L113-L126" %}
