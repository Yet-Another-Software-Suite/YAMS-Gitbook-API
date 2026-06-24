# LQRConfig

**Package:** `yams.math`

`LQRConfig` is a builder for `LinearSystemLoop` objects used by `LQRController`. It encapsulates the linear system model, Kalman filter, and LQR regulator into a single configurable object.

## Enum: `LQRType`

| Value | Description |
| ----- | ----------- |
| `FLYWHEEL` | Single-state system (angular velocity) |
| `ARM` | Two-state system (angle, angular velocity) |
| `ELEVATOR` | Two-state system (position, velocity) |

## Constructor

```java
LQRConfig(DCMotor motor, MechanismGearing gearing, MomentOfInertia moi)
```

All builder methods return the `LQRConfig` object for chaining.

## Builder Methods

| Method | Parameters | Description |
| ------ | ---------- | ----------- |
| `withRelms(Voltage relms)` | `relms` | Control effort cost. Higher values penalize voltage usage more. |
| `withControlEffort(Voltage effort)` | `effort` | Alias for `withRelms`. |
| `withMaxVoltage(Voltage voltage)` | `voltage` | Saturates the LQR output at this voltage. |
| `withMeasurementDelay(Time delay)` | `delay` | Compensates for sensor latency. |
| `withAggressiveness(double aggressiveness)` | `aggressiveness` — 1 to 100 | Higher values make the controller more responsive (and potentially less stable). |
| `withFlyWheel(AngularVelocity qelms, AngularVelocity modelTrust, AngularVelocity encoderTrust)` | — | Configures state cost, model noise, and encoder noise for flywheel systems. |
| `withElevator(Distance qelmsPosition, LinearVelocity qelmsVelocity, Distance modelPositionTrust, LinearVelocity modelVelocityTrust, Distance encoderPositionTrust, Mass mass, Distance drumRadius)` | — | Full elevator configuration with state costs and noise parameters. |
| `withArm(Angle qelmsPosition, AngularVelocity qelmsVelocity, Angle modelPositionTrust, AngularVelocity modelVelocityTrust, Angle encoderPositionTrust)` | — | Arm configuration with state costs and noise parameters. |

{% hint style="info" %}
The qelms parameters ("Q elements") define how much you care about state error. Smaller qelms values mean the controller tolerates more error but uses less voltage. The rElms ("R elements") penalize control effort — larger values produce gentler outputs.
{% endhint %}

## Getters

| Method | Returns | Description |
| ------ | ------- | ----------- |
| `getType()` | `LQRType` | The configured mechanism type. |
| `getPeriod()` | `Time` | The control loop period. |
| `getSystem()` | `LinearSystem<?, ?, ?>` | The computed linear system model. |
| `getKalmanFilter(LinearSystem<?, ?, ?> plant)` | `KalmanFilter<?, ?, ?>` | Constructs and returns the Kalman observer. |
| `getRegulator(LinearSystem<?, ?, ?> plant)` | `LinearQuadraticRegulator<?, ?, ?>` | Constructs and returns the LQR regulator. |
| `getLoop()` | `LinearSystemLoop<?, ?, ?>` | Constructs and returns the full system loop (convenience method). |
| `getLoop(LinearSystem, LinearQuadraticRegulator, KalmanFilter)` | `LinearSystemLoop<?, ?, ?>` | Constructs the loop from individually-provided components. |

## See Also

- [LQRController](lqr-controller.md)
- [C++ LQRConfig](../../cpp/math/lqr-config.md)
