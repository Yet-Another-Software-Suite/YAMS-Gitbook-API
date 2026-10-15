# NovaWrapper

**Package:** `yams.core.motorcontrollers.local`\
**Extends:** `SmartMotorController`

`SmartMotorController` implementation for The Thrifty Bot's Thrifty Nova motor controller, using ThriftyLib 2027.

## Constructor

```java
NovaWrapper(Nova nova, DCMotor motor, SmartMotorControllerConfig<?> config)
```

`Nova` is `com.thrifty.nova.Nova`, created with `new Nova(canBus, canId)` or `new Nova(canBus, canId, Motor.MotorType)`. `config` is the `SmartMotorControllerConfig` from `yams.commands2.config` or `yams.commands3.config`.

A vendor config passed with `withVendorConfig` must be a `NovaConfig.Config`, such as a `NovaConfigBatch`. It is applied before the YAMS config, so YAMS options overwrite it. Any other vendor config type throws `SmartMotorControllerConfigurationException`.

## Supported Hardware

| Hardware              | `Motor.MotorType` | `DCMotor` Model     |
| --------------------- | ----------------- | ------------------- |
| Thrifty Nova with NEO | `NEO`             | `DCMotor.getNEO(1)` |

ThriftyLib also defines `MotorType.MINION`, `MotorType.PULSAR` and `MotorType.BRUSHED`. The `DCMotor` you pass is what YAMS uses for simulation and safety checks, so pass the model of the motor that is actually connected.

## Example

```java
import com.thrifty.core.Motor.MotorType;
import com.thrifty.nova.Nova;
import com.thrifty.nova.NovaConfig;
import com.thrifty.nova.NovaConfigBatch;
import yams.commands2.config.SmartMotorControllerConfig;
import yams.core.motorcontrollers.SmartMotorController;
import yams.core.motorcontrollers.local.NovaWrapper;

SmartMotorControllerConfig config = new SmartMotorControllerConfig(this)
    .withMotorInverted(false)
    .withStatorCurrentLimit(Amps.of(40))
    .withClosedLoopController(0.2, 0.0, 0.0)
    .withVendorConfig(new NovaConfigBatch(NovaConfig.tempThrottleEnable(true)));

// Nova on CAN bus 0, CAN ID 3, driving a NEO
SmartMotorController motor = new NovaWrapper(
    new Nova(0, 3, MotorType.NEO),
    DCMotor.getNEO(1),
    config
);
```

## Nova-Specific Notes

**Closed loop runs on the SystemCore.** In `ControlMode.CLOSED_LOOP` the PID, feedforward and motion profiles run in YAMS on the SystemCore, and the Nova is commanded with voltage. ThriftyLib's simulation ignores the Nova's onboard PID gains, so this keeps simulation and the real robot the same. YAMS raises a `closed loop controller is running on the SystemCore` alert while it does this.

**External encoders.** Pass one of the following to `withExternalEncoder(Object)`:

* `FeedbackSensorType.ABS` (`com.thrifty.core.Motor.FeedbackSensorType`): an absolute encoder wired to the Nova's data port. Its type comes from the vendor config, such as `NovaConfig.absoluteEncoderType(Motor.AbsoluteEncoderType.REV_ENCODER)`. Its velocity reads 0 until `NovaConfig.absoluteFramePeriod(...)` enables its status frame.
* `FeedbackSensorType.QUAD`: a quadrature encoder wired to the Nova's data port. Its type comes from the vendor config, such as `NovaConfig.quadratureEncoderType(Motor.QuadratureEncoderType.custom(2048))`.
* A `CanEncoder` (`com.thrifty.canEncoder.CanEncoder`): a Thrifty CAN Encoder on the same CAN bus.

Any other encoder throws `IllegalArgumentException`. `withExternalEncoderZeroOffset` and `withExternalEncoderDiscontinuityPoint` are only available for absolute encoders (`ABS` or a `CanEncoder`). The discontinuity point defaults to 1 rotation (range `[0, 1)`); use `Rotations.of(0.5)` for `[-0.5, 0.5)`.

**Limitations.**

* The Nova's internal encoder cannot be inverted: `withEncoderInverted` throws `IllegalArgumentException` and `setEncoderInverted` throws `UnsupportedOperationException`.
* The Nova has a single ramp rate. Setting different `withOpenLoopRampRate` and `withClosedLoopRampRate` values throws `SmartMotorControllerConfigurationException`.
* `withVendorControlRequest` is not supported and throws `SmartMotorControllerConfigurationException`.
* `setEncoderVelocity` is not supported and throws `UnsupportedOperationException`.
* Native followers given to `withFollowers` must be `Nova` instances.

**Configuration.** All settings are sent to the Nova together as one `NovaConfigBatch`: a factory reset first when `withResetPreviousConfig(true)` (the default), then the vendor config, then the YAMS config.

**Simulation.** Uses `DCMotorSim` with the configured moment of inertia. External encoders follow the simulated mechanism, since ThriftyLib's simulation does not move them. The simulated current draw is reported to [BatterySim](../sensor/battery-sim.md).

**Telemetry.** Every `SmartMotorControllerTelemetry` field is supported.

## See Also

* [SmartMotorController](smart-motor-controller.md)
* [SmartMotorControllerConfig](smart-motor-controller-config.md)
* [SparkWrapper](spark-wrapper.md)
* [TalonFXWrapper](talonfx-wrapper.md)
