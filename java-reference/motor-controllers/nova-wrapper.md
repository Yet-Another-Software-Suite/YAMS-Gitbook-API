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

## Supported hardware

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

## Nova-specific notes

**Closed loop runs on the Nova.** In `ControlMode.CLOSED_LOOP` a PID, with or without a trapezoidal position profile, runs on the Nova (ThriftyLib 2027.0.0-alpha-6):

* `SLOT_0` and `SLOT_1` are the Nova's `Pid0` and `Pid1` slots, and its `Mp0` and `Mp1` motion profile slots, which take the feedforward's kS, kV, kA and kG (cosine gravity for an `ArmFeedforward`).
* YAMS sends the Nova a sensor-to-mechanism ratio, so targets, gains and soft limits are in mechanism rotations. kI and kD are sent as configured (per second).
* The closed loop tolerance is the slot's allowable error.
* The mechanism limits become the Nova's soft limits.
* Live setters (gains, profile, tolerance, limits) update the Nova.

Only an [LQR controller](../math/lqr-controller.md), an exponential profile or a velocity trapezoidal profile run on the SystemCore, which commands the Nova with voltage. Only then does YAMS raise the `closed loop controller is running on the SystemCore` alert.

When the Nova runs the closed loop, `applyConfig` throws `SmartMotorControllerConfigurationException` for:

| Option | Why |
| ------ | --- |
| A feedforward without `withTrapezoidalProfile(...)` | The Nova applies a feedforward only in its motion profile, which is for position only. For velocity control with a feedforward, use a velocity trapezoidal profile, which runs on the SystemCore. |
| `SLOT_2` or `SLOT_3` | The Nova has two closed loop slots. |
| Only one mechanism limit | The Nova's soft limits need both an upper and a lower limit. |
| `withTemperatureCutoff`, `withClosedLoopControllerMaximumVoltage`, `withFeedbackSynchronizationThreshold`, `withClosedLoopControlPeriod` | Only the SystemCore's closed loop applies them. |
| An absolute encoder closing the loop with wrapping other than `[0, 1)` | The Nova reads an absolute encoder closing its loop over `[0, 1)`. `withContinuousWrapping` also needs that encoder on the mechanism (1:1 external encoder gearing). |

**External encoders.** Pass one of the following to `withExternalEncoder(Object)`:

* `FeedbackSensorType.ABS` (`com.thrifty.core.Motor.FeedbackSensorType`): an absolute encoder wired to the Nova's data port. Its type comes from the vendor config, such as `NovaConfig.absoluteEncoderType(Motor.AbsoluteEncoderType.REV_ENCODER)`. Its velocity reads 0 until `NovaConfig.absoluteFramePeriod(...)` enables its status frame.
* `FeedbackSensorType.QUAD`: a quadrature encoder wired to the Nova's data port. Its type comes from the vendor config, such as `NovaConfig.quadratureEncoderType(Motor.QuadratureEncoderType.custom(2048))`.
* A `CanEncoder` (`com.thrifty.canEncoder.CanEncoder`): a Thrifty CAN Encoder on the same CAN bus.

Any other encoder throws `IllegalArgumentException`. `withExternalEncoderZeroOffset` is only available for absolute encoders (`ABS` or a `CanEncoder`). An absolute encoder reads `[0, 1)` by default. Set another range with `withWrapping(min, max)`, such as `withWrapping(Rotations.of(-0.5), Rotations.of(0.5))`; any range works when the SystemCore runs the closed loop, but only `[0, 1)` when the Nova closes its loop on the encoder. Only `withContinuousWrapping` turns on the Nova's `absWrapping` (see [Wrapping](smart-motor-controller-config.md#wrapping)).

**Limitations.**

* The Nova's internal encoder cannot be inverted: `withEncoderInverted` throws `IllegalArgumentException` and `setEncoderInverted` throws `UnsupportedOperationException`.
* The Nova has a single ramp rate. Setting different `withOpenLoopRampRate` and `withClosedLoopRampRate` values throws `SmartMotorControllerConfigurationException`.
* `withVendorControlRequest` is not supported and throws `SmartMotorControllerConfigurationException`.
* `setEncoderVelocity` is not supported and throws `UnsupportedOperationException`.
* Native followers given to `withFollowers` must be `Nova` instances.

**Configuration.** All settings are sent to the Nova together as one `NovaConfigBatch`: a factory reset first when `withResetPreviousConfig(true)` (the default), then the vendor config, then the YAMS config.

**Simulation.** Uses `DCMotorSim` with the configured moment of inertia. ThriftyLib's simulation ignores the Nova's PID, so a `NovaSimSupplier` runs the Nova's closed loop in simulation. `setSimSupplier` also makes a ThriftyLib `MotorSim` from the sim supplier, which drives a `CanEncoder` (through `CanEncoderConfig.simMotor`) and the Nova's data port encoders in simulation. The simulated current draw is reported to [BatterySim](../sensor/battery-sim.md).

**Telemetry.** Every `SmartMotorControllerTelemetry` field is supported.

## See also

* [SmartMotorController](smart-motor-controller.md)
* [SmartMotorControllerConfig](smart-motor-controller-config.md)
* [SparkWrapper](spark-wrapper.md)
* [A301Wrapper](a301-wrapper.md)
* [TalonFXWrapper](talonfx-wrapper.md)
