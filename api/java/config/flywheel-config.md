# FlyWheelConfig

**Package:** `yams.mechanisms.config`

`FlyWheelConfig` configures a velocity-controlled spinning mechanism. Simulation requires either `withMOI`, or both `withDiameter` and `withMass` together.

All builder methods return the `FlyWheelConfig` instance for chaining.

See also: [FlyWheel](../mechanisms/flywheel.md)

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
| `withDiameter(Distance diameter)` | `diameter` — wheel outer diameter | Wheel size. Used together with `withMass` to compute moment of inertia for simulation. |
| `withMass(Mass mass)` | `mass` — wheel mass | Wheel mass for MOI calculation. Used together with `withDiameter`. |
| `withMOI(Distance radius, Mass mass)` | `radius` — wheel radius, `mass` — wheel mass | Directly computes MOI as `0.5 × mass × radius²`. Alternative to specifying `withDiameter` + `withMass`. |
| `withSpeedometerSimulation(AngularVelocity maxVelocity)` | `maxVelocity` — maximum simulated velocity | Enables a simulated speedometer sensor capped at `maxVelocity`. Optional. |
| `withTelemetry(String name, TelemetryVerbosity verbosity)` | `name` — NT key prefix, `verbosity` — `LOW`/`MEDIUM`/`HIGH` | Enables NetworkTables telemetry for this mechanism. Optional. |
| `withSimColor(Color8Bit color)` | `color` — RGB color | Sets the Mechanism2d visualization color. Optional. |
| `withMechanismPositionConfig(MechanismPositionConfig config)` | `config` — position config | Configures the visualization plane and 3D offset for this mechanism. Optional. |

{% hint style="info" %}
For simulation, supply MOI via one of two paths: `withMOI(radius, mass)` directly, or `withDiameter(d)` + `withMass(m)` together. Providing only one of `withDiameter`/`withMass` is insufficient.
{% endhint %}

---

## Getters

All getters return `Optional<>` for configured values. `getSimColor()` and `getMechanismPositionConfig()` always return a value.

---

## Example

```java
FlyWheelConfig config = new FlyWheelConfig()
    .withDiameter(Inches.of(4))
    .withMass(Kilograms.of(0.3))
    .withTelemetry("Shooter", TelemetryVerbosity.HIGH);
```

{% @github-files/github-code-block url="https://github.com/Yet-Another-Software-Suite/YAMS/blob/master/examples/simple_shooter/java/frc/robot/subsystems/ShooterSubsystem.java#L150-L156" %}
