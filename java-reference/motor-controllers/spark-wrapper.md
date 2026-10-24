# SparkWrapper

**Package:** `yams.core.motorcontrollers.local`\
**Extends:** `SmartMotorController`

`SmartMotorController` implementation for REV SPARK MAX and SPARK FLEX controllers.

## Constructor

```java
SparkWrapper(SparkBase spark, DCMotor motor, SmartMotorControllerConfig<?> config)
```

`SparkBase` accepts `SparkMax` or `SparkFlex`. `config` is the `SmartMotorControllerConfig` from `yams.commands2.config` or `yams.commands3.config`. A vendor config passed with `withVendorConfig` must be a `SparkMaxConfig` for a `SparkMax` and a `SparkFlexConfig` for a `SparkFlex`.

## Supported hardware

| Hardware                    | `DCMotor` Model           |
| --------------------------- | ------------------------- |
| `SparkMax` with NEO         | `DCMotor.getNEO(1)`       |
| `SparkMax` with NEO 550     | `DCMotor.getNeo550(1)`    |
| `SparkFlex` with NEO Vortex | `DCMotor.getNeoVortex(1)` |

## Example

```java
SmartMotorControllerConfig config = new SmartMotorControllerConfig(this)
    .withMotorInverted(false)
    .withStatorCurrentLimit(Amps.of(40))
    .withClosedLoopController(0.2, 0.0, 0.0)
    .withExternalEncoderDiscontinuityPoint(Rotations.of(0.5)); // required for SparkAbsoluteEncoder

SmartMotorController motor = new SparkWrapper(
    new SparkMax(CANPort.CAN_S0, 3, MotorType.kBrushless),
    DCMotor.getNEO(1),
    config
);
```

## SPARK-specific notes

**Absolute encoder discontinuity point.** When using a `SparkAbsoluteEncoder` as the external encoder, `withExternalEncoderDiscontinuityPoint` must be called in the config:

* `Rotations.of(0.5)`: sensor range is `[-0.5, 0.5)`. Use when the mechanism zero is in the middle of the encoder range.
* `Rotations.of(1.0)`: sensor range is `[0, 1)`. Use when the full rotation is the intended range.

{% hint style="info" %}
It is recommended to call `withExternalEncoderDiscontinuityPoint` when using a `SparkAbsoluteEncoder`. Without it, the encoder reports incorrect positions near the wraparound point.
{% endhint %}

**Supported external encoders.** The SPARK's absolute encoder (`SparkAbsoluteEncoder`), the SPARK MAX alternate encoder, the SPARK Flex external encoder (quadrature, counts per revolution from the vendor config), or an encoder the SPARK reads over CAN (a REVLib `DetachedEncoder`).

**Configuration persistence.** YAMS applies the SPARK configuration on construction using `SparkBase.configure()`. Safe parameters are reset first when `withResetPreviousConfig(true)` (the default). Settings are burned to flash with `PersistMode.kPersistParameters` only while the robot is disabled; configuration changes made while enabled are not persisted.

**NEO 550 safety.** `checkConfigSafety()` throws `SmartMotorControllerConfigurationException` if no stator current limit is set for a NEO 550, or if the limit exceeds 40 A.

**MAXMotion.** When a trapezoidal profile is configured and the onboard SPARK closed loop controller is used, YAMS translates the profile to MAXMotion position and velocity control. When the closed loop runs on the SystemCore instead (for example with an exponential profile or an [LQR controller](../math/lqr-controller.md)), YAMS raises a `closed loop controller is running on the SystemCore` alert.

**Simulation.** Uses `DCMotorSim` with the configured moment of inertia. The simulated SPARK closes its loop every 1 ms (1 kHz), like the real SPARK, stepping through each `withSimulationPeriod` period. Because REVLib's MAXMotion simulation does not follow the profile, YAMS runs the trapezoidal profile in `simIterate()` and has the simulated SPARK hold each point with position control. The `SimSupplier` can be replaced at runtime via `setSimSupplier`.

## See also

* [SmartMotorController](smart-motor-controller.md)
* [SmartMotorControllerConfig](smart-motor-controller-config.md)
* [NovaWrapper](nova-wrapper.md)
* [TalonFXWrapper](talonfx-wrapper.md)
