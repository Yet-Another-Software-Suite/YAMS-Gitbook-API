# SmartMotorControllerTelemetryConfig

**Namespace:** `yams::telemetry`  
**Header:** `yams/telemetry/SmartMotorControllerTelemetryConfig.hpp`

Fine-grained telemetry configuration for a `SmartMotorController`. Every field is **disabled by default**. Opt-in using `WithTelemetryVerbosity()` for a preset bundle, individual `With*()` methods for specific fields, or both to supplement a preset.

For most use-cases, passing a name and verbosity to `SmartMotorControllerConfig::WithTelemetry(string, TelemetryVerbosity)` is sufficient. Use this class only when you need to override which specific fields are published or to route telemetry to DataLog instead of NetworkTables.

## Type Alias

```cpp
using TelemetryVerbosity =
    yams::motorcontrollers::SmartMotorControllerConfig::TelemetryVerbosity;
```

## Constructor

```cpp
SmartMotorControllerTelemetryConfig()
```

## Verbosity Presets

`WithTelemetryVerbosity(TelemetryVerbosity)` enables a cumulative set of fields. Each higher level includes everything from the levels below it.

| Level | Fields enabled |
|-------|---------------|
| `LOW` | Setpoint position/velocity, measurement position/velocity/acceleration, mechanism position/velocity/acceleration, rotor position/velocity, external encoder position/velocity, active closed-loop slot |
| `MID` | Everything in LOW, plus: output voltage, stator current, supply current |
| `HIGH` | Everything in MID, plus: all boolean status flags, tunable setpoints, PID gains (kP/kI/kD), feedforward gains (kS/kV/kG/kA), current limits, ramp rates, limit values, motor temperature |

## Builder Methods — Output Channels

All methods return `SmartMotorControllerTelemetryConfig&` for chaining.

| Method | Description |
|--------|-------------|
| `WithDataLogName(const std::string& dataLogName)` | Enables DataLog recording; writes all enabled fields to this prefix path. |
| `WithNetworkTables(bool enabled)` | Enables or disables NT4 publishing. Default: `true`. |
| `WithoutNetworkTables()` | Disables NT4 publishing. Useful during competition matches. |
| `WithTelemetryVerbosity(TelemetryVerbosity verbosity)` | Enables a preset bundle (`LOW`, `MID`, or `HIGH`). |

## Builder Methods — Boolean Fields

Each method enables a single boolean status flag. Flags are automatically suppressed if the motor controller or configuration does not support them.

| Method | What it logs |
|--------|-------------|
| `WithMechanismLowerLimit()` | Whether the mechanism is at or past its lower soft limit. |
| `WithMechanismUpperLimit()` | Whether the mechanism is at or past its upper soft limit. |
| `WithTemperatureLimit()` | Whether the motor is in an over-temperature condition. |
| `WithVelocityControl()` | Whether velocity closed-loop mode is currently active. |
| `WithElevatorFeedforward()` | Whether an elevator feedforward is in use. |
| `WithArmFeedforward()` | Whether an arm feedforward is in use. |
| `WithSimpleFeedforward()` | Whether a simple motor feedforward is in use. |
| `WithMotionProfile()` | Whether a trapezoidal or exponential motion profile is active. |

## Builder Methods — Numeric Fields

| Method | What it logs |
|--------|-------------|
| `WithSetpointPosition()` | Current position setpoint in mechanism units. |
| `WithSetpointVelocity()` | Current velocity setpoint in mechanism units. |
| `WithOutputVoltage()` | Motor output voltage (V). |
| `WithStatorCurrent()` | Stator (output) current (A). |
| `WithTemperature()` | Motor winding temperature (°C). |
| `WithMeasurementPosition()` | Linear position via mechanism circumference (m). Requires `WithMechanismCircumference`. |
| `WithMeasurementVelocity()` | Linear velocity in m/s. Requires `WithMechanismCircumference`. |
| `WithMechanismPosition()` | Mechanism angular position (degrees). |
| `WithMechanismVelocity()` | Mechanism angular velocity (degrees/s). |
| `WithRotorPosition()` | Raw rotor position (rotations). |
| `WithRotorVelocity()` | Raw rotor velocity (rotations/s). |

## Example

```cpp
#include <yams/telemetry/SmartMotorControllerTelemetryConfig.hpp>
#include <yams/motorcontrollers/SmartMotorControllerConfig.hpp>

using Cfg    = yams::motorcontrollers::SmartMotorControllerConfig;
using TelCfg = yams::telemetry::SmartMotorControllerTelemetryConfig;

// Preset-based configuration applied via SmartMotorControllerConfig — simplest approach
Cfg motorConfig;
motorConfig.WithTelemetry("ShoulderMotor", Cfg::TelemetryVerbosity::HIGH);

// Fine-grained override applied after constructing the motor controller
smc.WithTelemetry(
    TelCfg{}
        .WithTelemetryVerbosity(Cfg::TelemetryVerbosity::MID)
        .WithMechanismPosition()
        .WithMechanismVelocity()
        .WithTemperature()
        .WithArmFeedforward()
        .WithDataLogName("/Robot/Arm/Motor")
        .WithoutNetworkTables());   // log-only during matches
```

{% hint style="info" %}
Fields that are not applicable to the current motor controller or configuration are silently suppressed at runtime.
{% endhint %}

{% hint style="info" %}
`WithTelemetryVerbosity()` is cumulative: combining it with individual `With*()` methods produces the union of all requested fields.
{% endhint %}

## Related Pages

- [SmartMotorController (C++)](smart-motor-controller.md)
- [SmartMotorControllerConfig (C++)](smart-motor-controller-config.md)
- [SmartMotorControllerTelemetryConfig (Java)](../../java/motor-controllers/smart-motor-controller-telemetry-config.md)
