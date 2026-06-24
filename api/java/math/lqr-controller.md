# LQRController

**Package:** `yams.math`

`LQRController` wraps a `LinearSystemLoop` and provides type-safe `calculate()` overloads for different mechanism types.

## Constructors

```java
LQRController(LQRConfig config)   // Preferred
LQRController(LQRType type, LinearSystemLoop<?, ?, ?> loop, Time period)
```

## Methods

| Method | Returns | Description |
| ------ | ------- | ----------- |
| `reset(Angle angle, AngularVelocity velocity)` | `void` | Resets the observer for arm or flywheel systems. Call on enable. |
| `reset(Distance distance, LinearVelocity velocity)` | `void` | Resets the observer for elevator systems. |
| `updateConfig(LQRConfig config)` | `void` | Replaces the underlying system loop at runtime. |

## Calculate Overloads

| Method | For Mechanism | Returns |
| ------ | ------------- | ------- |
| `calculate(Angle measured, Angle position, AngularVelocity velocity)` | Arm | `Voltage` |
| `calculate(Distance measured, Distance position, LinearVelocity velocity)` | Elevator | `Voltage` |
| `calculate(AngularVelocity measured, AngularVelocity velocity)` | Flywheel | `Voltage` |
| `calculate(LinearVelocity measured, LinearVelocity velocity)` | Flywheel (linear surface speed) | `Voltage` |

## Getters

| Method | Returns | Description |
| ------ | ------- | ----------- |
| `getType()` | `LQRType` | The mechanism type this controller was configured for. |
| `getConfig()` | `Optional<LQRConfig>` | The `LQRConfig` used to construct this controller, or empty if constructed from raw components. |

## Usage with SmartMotorControllerConfig

```java
LQRConfig lqrCfg = new LQRConfig(DCMotor.getNEO(1), gearing, moi)
    .withArm(Degrees.of(5), DegreesPerSecond.of(10),
             Degrees.of(1), DegreesPerSecond.of(5), Degrees.of(0.5))
    .withControlEffort(Volts.of(12));

SmartMotorControllerConfig motorConfig = new SmartMotorControllerConfig()
    .withLQRController(lqrCfg);
```

## See Also

- [LQRConfig](lqr-config.md)
- [C++ LQRController](../../cpp/math/lqr-controller.md)
