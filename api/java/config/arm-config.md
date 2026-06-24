# ArmConfig

**Package:** `yams.mechanisms.config`

`ArmConfig` configures a single-jointed arm mechanism. All builder methods return the `ArmConfig` instance for chaining.

See also: [Arm](../mechanisms/arm.md)

---

## Constructor

```java
ArmConfig()
ArmConfig clone()  // Deep copy
```

---

## Builder Methods

| Method | Parameters | Description |
|--------|-----------|-------------|
| `withLength(Distance distance)` | `distance` — arm link length | **Required for simulation.** Sets the physical length of the arm link. Used by `SingleJointedArmSim`. |
| `withHardLimits(Angle min, Angle max)` | `min` — lower bound, `max` — upper bound | Sets the physical rotation limits. The motor controller will not command positions outside these angles. |
| `withTelemetry(String name, TelemetryVerbosity verbosity)` | `name` — NT key prefix, `verbosity` — `LOW`/`MEDIUM`/`HIGH` | Enables NetworkTables telemetry for this mechanism. Optional. |
| `withSimColor(Color8Bit color)` | `color` — RGB color | Sets the Mechanism2d visualization color. Defaults to a built-in color if not set. Optional. |
| `withMechanismPositionConfig(MechanismPositionConfig config)` | `config` — position config | Configures the visualization plane and 3D offset for this mechanism. Optional. |

---

## Getters

| Method | Returns | Description |
|--------|---------|-------------|
| `getTelemetryName()` | `Optional<String>` | The configured NT prefix, or empty if telemetry is not set. |
| `getTelemetryVerbosity()` | `Optional<TelemetryVerbosity>` | The configured verbosity, or empty. |
| `getLength()` | `Optional<Distance>` | The arm length, or empty if not configured. |
| `getLowerHardLimit()` | `Optional<Angle>` | Lower hard limit angle, or empty. |
| `getUpperHardLimit()` | `Optional<Angle>` | Upper hard limit angle, or empty. |
| `getSimColor()` | `Color8Bit` | The visualization color. Always returns a value; falls back to a built-in default. |
| `getMechanismPositionConfig()` | `MechanismPositionConfig` | The position/plane configuration. |

---

## Example

```java
ArmConfig config = new ArmConfig()
    .withLength(Meters.of(0.6))
    .withHardLimits(Degrees.of(-90), Degrees.of(90))
    .withTelemetry("Arm", TelemetryVerbosity.HIGH);
```
