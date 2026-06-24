# SparkWrapper

**Package:** `yams.motorcontrollers.local`\
**Extends:** `SmartMotorController`

`SmartMotorController` implementation for REV SPARK MAX and SPARK FLEX controllers. Construct via `SmartMotorController.create()` or directly via the constructor.

## Constructor

```java
SparkWrapper(SparkBase spark, DCMotor motor, SmartMotorControllerConfig config)
```

`SparkBase` accepts `SparkMax` or `SparkFlex`.

## Supported Hardware

| Hardware                    | `DCMotor` Model           |
| --------------------------- | ------------------------- |
| `SparkMax` with NEO         | `DCMotor.getNEO(1)`       |
| `SparkMax` with NEO 550     | `DCMotor.getNeo550(1)`    |
| `SparkFlex` with NEO Vortex | `DCMotor.getNeoVortex(1)` |

## Example

```java
SmartMotorControllerConfig config = new SmartMotorControllerConfig()
    .withMotorInverted(false)
    .withStatorCurrentLimit(Amps.of(40))
    .withClosedLoopController(0.2, 0.0, 0.0)
    .withExternalEncoderDiscontinuityPoint(Rotations.of(0.5)); // required for SparkAbsoluteEncoder

SmartMotorController motor = new SparkWrapper(
    new SparkMax(3, MotorType.kBrushless),
    DCMotor.getNEO(1),
    config
);
```

## SPARK-Specific Notes

**Absolute encoder discontinuity point.** When using a `SparkAbsoluteEncoder` as the external encoder, `withExternalEncoderDiscontinuityPoint` must be called in the config:

* `Rotations.of(0.5)` — sensor range is `[-0.5, 0.5)`. Use when the mechanism zero is in the middle of the encoder range.
* `Rotations.of(1.0)` — sensor range is `[0, 1)`. Use when the full rotation is the intended range.

{% hint style="info" %}
It is recommended to call `withExternalEncoderDiscontinuityPoint` when using a `SparkAbsoluteEncoder`. Without it, the encoder reports incorrect positions near the wraparound point.
{% endhint %}

**Configuration persistence.** YAMS applies the SPARK configuration on construction using `SparkBase.configure()`. Configuration is applied with `ResetMode.kResetSafeParameters` and `PersistMode.kPersistParameters` by default, which burns settings to flash. Parameters survive power cycles.

**NEO 550 safety.** `checkConfigSafety()` throws `SmartMotorControllerConfigurationException` if no stator current limit is set for a NEO 550, or if the limit exceeds 40 A.

**MAXMotion.** When a trapezoidal or exponential profile is configured and the onboard SPARK closed-loop controller is used, YAMS translates the profile to MAXMotion position mode. When the software PID is active instead, profiling runs on the roboRIO.

**Simulation.** Uses `DCMotorSim` with the configured moment of inertia. The `SimSupplier` can be replaced at runtime via `setSimSupplier`.

## See Also

* [SmartMotorController](smart-motor-controller.md)
* [SmartMotorControllerConfig](smart-motor-controller-config.md)
* [TalonFXWrapper](talonfx-wrapper.md)
