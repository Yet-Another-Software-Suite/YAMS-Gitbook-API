# PivotConfig

**Package:** `yams.mechanisms.config`

`PivotConfig` configures a rotational mechanism in the XY plane (e.g., a turret or shooter hood). For mechanisms rotating in the XZ plane, use [ArmConfig](arm-config.md) instead.

All builder methods return the `PivotConfig` instance for chaining.

See also: [Pivot](../mechanisms/pivot.md)

---

## Constructor

```java
PivotConfig()
PivotConfig(PivotConfig cfg)  // Copy constructor
PivotConfig clone()           // Deep copy
```

---

## Builder Methods

| Method | Parameters | Description |
|--------|-----------|-------------|
| `withHardLimits(Angle min, Angle max)` | `min` — lower bound, `max` — upper bound | Sets the physical rotation limits. The motor controller will not command positions outside these angles. Optional. |
| `withTelemetry(String name, TelemetryVerbosity verbosity)` | `name` — NT key prefix, `verbosity` — `LOW`/`MEDIUM`/`HIGH` | Enables NetworkTables telemetry for this mechanism. Optional. |
| `withSimColor(Color8Bit color)` | `color` — RGB color | Sets the Mechanism2d visualization color. Optional. |
| `withMechanismPositionConfig(MechanismPositionConfig config)` | `config` — position config | Configures the visualization plane and 3D offset for this mechanism. Optional. |

---

## Getters

All getters return `Optional<>` for configured values, following the same pattern as [ArmConfig](arm-config.md) getters. `getSimColor()` and `getMechanismPositionConfig()` always return a value.

---

## Example

```java
PivotConfig config = new PivotConfig()
    .withHardLimits(Degrees.of(-180), Degrees.of(180))
    .withTelemetry("Turret", TelemetryVerbosity.MEDIUM);
```
