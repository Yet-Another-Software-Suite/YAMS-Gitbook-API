# FlyWheelConfig

**Package:** `yams.mechanisms.config`

`FlyWheelConfig` configures a velocity-controlled spinning mechanism.

All builder methods return the `FlyWheelConfig` instance for chaining.

See also: [FlyWheel](../mechanisms/flywheel.md)

{% hint style="info" %}
**Simulation Note:** Moment of inertia is not configured on `FlyWheelConfig`. Set it via `SmartMotorControllerConfig.withMomentOfInertia(Distance length, Mass mass)`.
{% endhint %}

---

## Constructor

```java
FlyWheelConfig()
FlyWheelConfig clone()  // Deep copy
```

---

## Builder Methods

| Method | Parameters | Description |
|--------|-----------|-------------|
| `withDiameter(Distance diameter)` | `diameter` — wheel/roller outer diameter | Wheel size used for angular-to-linear velocity conversion and simulation. |
| `withSpeedometerSimulation(AngularVelocity maxVelocity)` | `maxVelocity` — maximum simulated velocity | Enables a simulated speedometer gauge capped at `maxVelocity`. |
| `withSpeedometerSimulation()` | _(none)_ | Enables the speedometer with the previously configured max velocity. Throws if max velocity was not set first. |
| `disableSpeedometerSimulation()` | _(none)_ | Disables the speedometer simulation. |
| `withSimColor(Color8Bit color)` | `color` — RGB color | Sets the Mechanism2d visualization color. Optional; defaults to orange. |
| `withMechanismPositionConfig(MechanismPositionConfig config)` | `config` — position config | Configures the visualization plane and 3D offset for this mechanism. Optional. |
| `withTelemetry(String name, TelemetryVerbosity verbosity)` | `name` — NT key prefix, `verbosity` — `LOW`/`MEDIUM`/`HIGH` | Enables NetworkTables telemetry for this mechanism. Optional. |

---

## Getters

All getters return `Optional<>` for configured values. `getSimColor()` and `getMechanismPositionConfig()` always return a value.

---

## Example

```java
FlyWheelConfig config = new FlyWheelConfig()
    .withDiameter(Inches.of(4))
    .withSpeedometerSimulation(RPM.of(6000))
    .withTelemetry("Shooter", TelemetryVerbosity.HIGH);
```

{% @github-files/github-code-block url="https://github.com/Yet-Another-Software-Suite/YAMS/blob/master/examples/simple_shooter/java/frc/robot/subsystems/ShooterSubsystem.java#L150-L156" %}
