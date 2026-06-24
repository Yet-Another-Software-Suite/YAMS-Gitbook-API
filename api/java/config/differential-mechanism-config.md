# DifferentialMechanismConfig

**Package:** `yams.mechanisms.config`

`DifferentialMechanismConfig` configures a two-motor differential mechanism. Unlike the other config classes, the two motor controllers are supplied in the constructor and cannot be changed after construction.

All builder methods return the `DifferentialMechanismConfig` instance for chaining.

See also: [DifferentialMechanism](../mechanisms/differential-mechanism.md)

---

## Constructor

```java
DifferentialMechanismConfig(SmartMotorController leftSMC, SmartMotorController rightSMC)
```

| Parameter | Type | Description |
|-----------|------|-------------|
| `leftSMC` | `SmartMotorController` | The "left" motor. Its output mixes with the right motor to produce tilt and twist. |
| `rightSMC` | `SmartMotorController` | The "right" motor. |

Both parameters are **required**.

---

## Builder Methods

| Method | Parameters | Description |
|--------|-----------|-------------|
| `withTiltStartingPosition(Angle angle)` | `angle` — starting tilt angle | Seeds the tilt encoder with an initial position. Optional. |
| `withTwistStartingPosition(Angle angle)` | `angle` — starting twist angle | Seeds the twist encoder with an initial position. Optional. |
| `withLength(Distance length)` | `length` — arm length | Sets the mechanism arm length for simulation. Optional. |
| `withMOI(Distance length, Mass mass)` | `length` — arm length, `mass` — arm mass | Computes moment of inertia for simulation. Optional. |
| `withTelemetry(String name, TelemetryVerbosity verbosity)` | `name` — NT key prefix, `verbosity` — `LOW`/`MEDIUM`/`HIGH` | Enables NetworkTables telemetry for this mechanism. Optional. |
| `withSimColor(Color8Bit color)` | `color` — RGB color | Sets the Mechanism2d visualization color. Optional. |
| `withMechanismPositionConfig(MechanismPositionConfig config)` | `config` — position config | Configures the visualization plane and 3D offset for this mechanism. Optional. |

---

## Example

```java
SmartMotorController leftSMC = SmartMotorController.create(
    new CANSparkMax(1, MotorType.kBrushless),
    DCMotor.getNEO(1),
    leftConfig
);
SmartMotorController rightSMC = SmartMotorController.create(
    new CANSparkMax(2, MotorType.kBrushless),
    DCMotor.getNEO(1),
    rightConfig
);

DifferentialMechanismConfig config = new DifferentialMechanismConfig(leftSMC, rightSMC)
    .withLength(Inches.of(12))
    .withTelemetry("Wrist", TelemetryVerbosity.HIGH);
```

{% embed url="https://github.com/Yet-Another-Software-Suite/YAMS/tree/master/examples/differential_mechanism" %}
Differential Mechanism example
{% endembed %}
