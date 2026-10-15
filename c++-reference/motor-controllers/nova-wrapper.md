# NovaWrapper

**Namespace:** `yams::motorcontrollers::local`\
**Extends:** `SmartMotorController`\
**Header:** `#include <yams/motorcontrollers/local/NovaWrapper.hpp>`

`SmartMotorController` implementation for The Thrifty Bot's Thrifty Nova motor controller, using ThriftyLib 2027.

**Java equivalent:** [Java NovaWrapper](../../java-reference/motor-controllers/nova-wrapper.md)

***

## Constructor

```cpp
NovaWrapper(thrifty::Nova* nova, wpi::math::DCMotor motor, SmartMotorControllerConfig* config)
```

| Parameter | Type                          | Description                                                             |
| --------- | ----------------------------- | ----------------------------------------------------------------------- |
| `nova`    | `thrifty::Nova*`              | Nova hardware object. Must outlive this wrapper.                        |
| `motor`   | `wpi::math::DCMotor`          | DC motor model used for simulation (e.g. `wpi::math::DCMotor::NEO(1)`). |
| `config`  | `SmartMotorControllerConfig*` | Configuration to apply. Must outlive this wrapper.                      |

`thrifty::Nova` comes from `<thrifty/nova/Nova.h>` and is created with a CAN bus, a CAN ID and a motor type, for example `thrifty::Nova nova{0, 3, thrifty::Motor::NEO};`.

A vendor config passed with `WithVendorConfig` must be a `thrifty::NovaConfigBatch`. It is applied before the YAMS config, so YAMS options override it. Any other vendor config type throws `SmartMotorControllerConfigurationException`. When `WithResetPreviousConfig(true)` is set, a factory reset is sent first.

***

## Key Methods

`NovaWrapper` implements the full [`SmartMotorController`](smart-motor-controller.md) interface (open-loop outputs, closed-loop setpoints, encoder reads and writes, live configuration setters, telemetry and live tuning). See that page for the complete method list. The notes below cover Nova-specific behavior.

### Closed-Loop Control

The closed-loop controller always runs on the SystemCore and commands the Nova with voltage, the same way the profiled controllers of the other wrappers do. ThriftyLib's simulation ignores the Nova's onboard PID gains, so running the loop on the SystemCore keeps simulation and the real robot the same. PID, feedforward, trapezoidal and exponential profiles, and [LQR](../math/lqr-controller.md) are all evaluated by YAMS.

In closed-loop mode the wrapper starts the closed-loop notifier and raises an alert (see [Alerts](#alerts)). Set the loop period with `WithClosedLoopControlPeriod(...)` (default 20 ms); it is only valid in closed-loop mode.

### Encoder Reads

| Method                                                                            | Nova behavior                                                                                                  |
| --------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------- |
| `GetMechanismPosition()` / `GetMechanismVelocity()`                               | External encoder when `WithUseExternalFeedbackEncoder(true)` is set, otherwise the motor's internal encoder.   |
| `GetRelativeMechanismPosition()` / `GetRelativeMechanismVelocity()`               | Always the motor's internal encoder, with the motor gearing applied.                                           |
| `GetExternalEncoderMechanismPosition()` / `GetExternalEncoderMechanismVelocity()` | The external encoder with the external encoder gearing applied. Empty when no external encoder is attached.    |
| `GetSupplyCurrent()`                                                              | Supply current reported by the Nova (the sim supplier's value in simulation).                                  |

### Restrictions

| Option                                                         | Behavior                                                                                                      |
| -------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------- |
| `WithOpenLoopRampRate` together with `WithClosedLoopRampRate`  | The Nova has one ramp rate. Different open and closed loop values throw `SmartMotorControllerConfigurationException`. |
| `WithEncoderInverted(true)` / `SetEncoderInverted`             | The Nova's internal encoder cannot be inverted. `WithEncoderInverted` throws a `std::invalid_argument` when the config is applied, and `SetEncoderInverted` throws a `std::runtime_error`.                                                       |
| `WithVendorControlRequest`                                     | Not supported on the Nova. Throws `SmartMotorControllerConfigurationException`.                               |
| `SetEncoderVelocity` (on a real robot)                         | Throws `std::runtime_error`; the Nova cannot have its encoder velocity set.                                   |
| Closed loop tolerance with an [LQR](../math/lqr-controller.md)                              | Throws `std::invalid_argument`.                                                                               |

***

## External Encoders

Pass one of the following to `WithExternalEncoder(...)`:

| Value                                        | Encoder                                                                                                                                                                             |
| -------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `thrifty::Motor::FeedbackSensorType::ABS`    | An absolute encoder on the Nova's data port. Its type comes from the vendor config, such as `thrifty::NovaConfig::AbsoluteEncoderType(thrifty::Motor::AbsoluteEncoderType::REV_ENCODER)`. |
| `thrifty::Motor::FeedbackSensorType::QUAD`   | A quadrature encoder on the Nova's data port. Its type comes from the vendor config, such as `thrifty::NovaConfig::QuadratureEncoderType(thrifty::Motor::QuadratureEncoderType::Custom(2048))`. |
| `thrifty::CanEncoder*`                       | A Thrifty CAN Encoder on the same CAN bus.                                                                                                                                          |

Any other value throws `std::invalid_argument`.

The external encoder discontinuity point (absolute and CAN encoders) accepts 0.5 rotations, which reports `[-0.5, 0.5)`, or 1 rotation (the default), which reports `[0, 1)`. Zero offset and discontinuity point are only available for absolute encoders and throw a `SmartMotorControllerConfigurationException` for a quadrature encoder. A quadrature encoder counts from where it powers on, so it is set to the starting position (or zero) when the config is applied.

`WithExternalEncoderZeroOffset`, `WithExternalEncoderDiscontinuityPoint`, `WithExternalEncoderInverted` and `WithExternalEncoderGearing` all require `WithExternalEncoder(...)`; setting them without one throws `SmartMotorControllerConfigurationException`.

When no starting position is configured and an absolute or CAN encoder is attached, the internal encoder is seeded from it when the config is applied.

***

## Followers

`WithFollowers` accepts `thrifty::Nova*` followers only. Any other type throws `std::invalid_argument`. Use `WithLooselyCoupledFollowers` to forward setpoints to other `SmartMotorController` instances instead.

***

## Simulation

In simulation the wrapper creates a DC motor simulation from the motor model, the configured MOI and the motor gearing, and reports its current draw to [`BatterySim`](../sensor/battery-sim.md) every loop. `SimIterate()` also steps ThriftyLib's own simulation for the Nova and any attached `thrifty::CanEncoder`. Set a realistic moment of inertia with `SmartMotorControllerConfig::WithMOI(...)` for realistic current draw.

***

## Alerts

Alerts are published in the `YAMS` alert group and cleared when the wrapper is destroyed.

| Alert                                                          | When                                                                 |
| -------------------------------------------------------------- | -------------------------------------------------------------------- |
| `<name> closed loop controller is running on the SystemCore.`  | The config uses closed-loop mode, which on a Nova always runs on the SystemCore. |

***

## Example

```cpp
#include <thrifty/nova/Nova.h>
#include <wpi/commands2/SubsystemBase.hpp>
#include <yams/motorcontrollers/SmartMotorControllerConfig.hpp>
#include <yams/motorcontrollers/local/NovaWrapper.hpp>

using namespace yams::motorcontrollers;
using namespace yams::motorcontrollers::local;

class ArmSubsystem : public wpi::cmd::SubsystemBase {
 public:
  ArmSubsystem() {
    m_motorConfig.WithSubsystem(this)
        .WithFeedback(1.0, 0.0, 0.0)
        .WithStatorCurrentLimit(40.0_A)
        .WithMotorInverted(false)
        .WithVendorConfig(thrifty::NovaConfigBatch{thrifty::NovaConfig::TempThrottleEnable(true)})
        .WithClosedLoopMode()
        .WithTelemetry("ArmMotor", SmartMotorControllerConfig::TelemetryVerbosity::HIGH);

    m_motor.emplace(&m_nova, wpi::math::DCMotor::NEO(1), &m_motorConfig);
  }

 private:
  thrifty::Nova m_nova{0, 3, thrifty::Motor::NEO};
  SmartMotorControllerConfig m_motorConfig;  // must outlive the wrapper
  std::optional<NovaWrapper> m_motor;
};
```

{% hint style="info" %}
`TelemetryVerbosity::HIGH` together with `WithSubsystem(this)` automatically registers the `Live Tuning` command for the subsystem. See [SmartMotorController](smart-motor-controller.md#telemetry-and-live-tuning).
{% endhint %}

***

## See Also

* [SmartMotorController](smart-motor-controller.md)
* [SmartMotorControllerConfig](smart-motor-controller-config.md)
* [SparkWrapper](spark-wrapper.md)
* [TalonFXWrapper / TalonFXSWrapper](talonfx-wrapper.md)
* [BatterySim](../sensor/battery-sim.md)
* [Java NovaWrapper](../../java-reference/motor-controllers/nova-wrapper.md)
