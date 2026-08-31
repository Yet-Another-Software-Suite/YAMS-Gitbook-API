# SmartMotorControllerTelemetryConfig

**Package:** `yams.telemetry`

Fine-grained telemetry configuration for a [SmartMotorController](smart-motor-controller.md). Every field is **disabled by default**. Opt-in using `withTelemetryVerbosity()` for a preset bundle, individual `with*()` methods for specific fields, or both to supplement a preset.

For most use-cases, passing a name and verbosity to `SmartMotorControllerConfig.withTelemetry(String, TelemetryVerbosity)` is sufficient. Use this class only when you need to override which specific fields are published or to route telemetry to DataLog instead of NetworkTables.

## Constructor

```java
SmartMotorControllerTelemetryConfig()
```

## Verbosity Presets

`withTelemetryVerbosity(TelemetryVerbosity)` enables a cumulative set of fields. Each higher level includes everything from the levels below it.

| Level  | Fields enabled                                                                                                                                                                                         |
| ------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `LOW`  | Setpoint position/velocity, measurement position/velocity/acceleration, mechanism position/velocity/acceleration, rotor position/velocity, external encoder position/velocity, active closed-loop slot |
| `MID`  | Everything in LOW, plus: output voltage, stator current, supply current                                                                                                                                |
| `HIGH` | Everything in MID, plus: all boolean status flags, tunable setpoints, PID gains (kP/kI/kD), feedforward gains (kS/kV/kG/kA), current limits, ramp rates, limit values, motor temperature               |

## Builder Methods: Output Channels

| Method                                                 | Description                                                                                         |
| ------------------------------------------------------ | --------------------------------------------------------------------------------------------------- |
| `withDataLogName(String dataLogName)`                  | Enables DataLog recording; writes all enabled fields to this prefix path (e.g. `"motors/shooter"`). |
| `withNetworkTables(boolean enabled)`                   | Enables or disables NT4 publishing. Default: `true`.                                                |
| `withoutNetworkTables()`                               | Disables NT4 publishing. Useful during competition matches to reduce CAN/network load.              |
| `withTelemetryVerbosity(TelemetryVerbosity verbosity)` | Enables a preset bundle of fields (`LOW`, `MID`, or `HIGH`).                                        |

## Builder Methods: Boolean Fields

Each method enables a single boolean status flag. The flag is automatically suppressed if the motor controller or configuration does not support it.

| Method                      | What it logs                                                   |
| --------------------------- | -------------------------------------------------------------- |
| `withMechanismLowerLimit()` | Whether the mechanism is at or past its lower soft limit.      |
| `withMechanismUpperLimit()` | Whether the mechanism is at or past its upper soft limit.      |
| `withTemperatureLimit()`    | Whether the motor is in an over-temperature condition.         |
| `withVelocityControl()`     | Whether velocity closed-loop mode is currently active.         |
| `withElevatorFeedforward()` | Whether an `ElevatorFeedforward` is in use.                    |
| `withArmFeedforward()`      | Whether an `ArmFeedforward` is in use.                         |
| `withSimpleFeedforward()`   | Whether a `SimpleMotorFeedforward` is in use.                  |
| `withMotionProfile()`       | Whether a trapezoidal or exponential motion profile is active. |

## Builder Methods: Numeric Fields

Each method enables a single numeric (double) field.

| Method                      | What it logs                                                                                                          |
| --------------------------- | --------------------------------------------------------------------------------------------------------------------- |
| `withSetpointPosition()`    | Current position setpoint in mechanism units (read-only; non-tunable).                                                |
| `withSetpointVelocity()`    | Current velocity setpoint in mechanism units.                                                                         |
| `withOutputVoltage()`       | Motor output voltage (V).                                                                                             |
| `withStatorCurrent()`       | Stator (output) current (A).                                                                                          |
| `withTemperature()`         | Motor winding temperature (°C).                                                                                       |
| `withMeasurementPosition()` | Linear position from mechanism circumference conversion (m). Only available when `withMechanismCircumference` is set. |
| `withMeasurementVelocity()` | Linear velocity in m/s. Only available when `withMechanismCircumference` is set.                                      |
| `withMechanismPosition()`   | Mechanism angular position (degrees).                                                                                 |
| `withMechanismVelocity()`   | Mechanism angular velocity (degrees/s).                                                                               |
| `withRotorPosition()`       | Raw rotor position (rotations).                                                                                       |
| `withRotorVelocity()`       | Raw rotor velocity (rotations/s).                                                                                     |

## Builder Methods: Custom / Escape Hatch

`withCustom(...)` is an escape hatch for enabling or disabling any `BooleanTelemetryField` or `DoubleTelemetryField` by value, without a dedicated `with*()` method. It's useful for fields that don't have a named builder method, or for bulk enabling/disabling several fields at once with an array. Both enums live on `SmartMotorControllerTelemetry` (`SmartMotorControllerTelemetry.BooleanTelemetryField`, `SmartMotorControllerTelemetry.DoubleTelemetryField`).

| Method                                                          | Description                                                             |
| ---------------------------------------------------------------- | ------------------------------------------------------------------------ |
| `withCustom(BooleanTelemetryField field, boolean value)`        | Enables (`true`) or disables (`false`) a single boolean field.          |
| `withCustom(DoubleTelemetryField field, boolean value)`         | Enables (`true`) or disables (`false`) a single numeric field.          |
| `withCustom(BooleanTelemetryField[] fields, boolean value)`     | Enables or disables every boolean field in the array with one call.     |
| `withCustom(DoubleTelemetryField[] fields, boolean value)`      | Enables or disables every numeric field in the array with one call.     |

```java
import yams.telemetry.SmartMotorControllerTelemetry.BooleanTelemetryField;
import yams.telemetry.SmartMotorControllerTelemetry.DoubleTelemetryField;

SmartMotorControllerTelemetryConfig telemetryCfg =
    new SmartMotorControllerTelemetryConfig()
        .withTelemetryVerbosity(TelemetryVerbosity.LOW)
        // enable one field that isn't part of LOW
        .withCustom(DoubleTelemetryField.SupplyCurrent, true)
        // disable a batch of fields at once
        .withCustom(new BooleanTelemetryField[]{
            BooleanTelemetryField.MotorInversion,
            BooleanTelemetryField.EncoderInversion
        }, false);
```

## Example

```java
import yams.telemetry.SmartMotorControllerTelemetryConfig;
import yams.motorcontrollers.SmartMotorControllerConfig.TelemetryVerbosity;

// Start from a MID preset and add individual fields on top
SmartMotorControllerTelemetryConfig telemetryCfg =
    new SmartMotorControllerTelemetryConfig()
        .withTelemetryVerbosity(TelemetryVerbosity.MID)
        .withMechanismPosition()
        .withMechanismVelocity()
        .withTemperature()
        .withArmFeedforward()
        .withDataLogName("motors/shoulder")
        .withoutNetworkTables();   // log-only during matches

// Apply to a constructed motor controller
motor.withTelemetry(telemetryCfg);
```

{% hint style="info" %}
Fields that are not applicable to the current motor controller or configuration are silently suppressed. For example, `withStatorCurrent()` is ignored on motor controllers that do not report stator current, and `withMeasurementPosition()` is ignored unless `withMechanismCircumference()` was set on the `SmartMotorControllerConfig`.
{% endhint %}

{% hint style="info" %}
`withTelemetryVerbosity()` is cumulative: calling it multiple times or combining it with individual field methods results in the union of all requested fields.
{% endhint %}

{% hint style="warning" %}
`withCustom(...)` is currently Java-only; the C++ port does not yet expose a `WithCustom(...)` escape hatch.
{% endhint %}

## Related Pages

* [SmartMotorController](smart-motor-controller.md)
* [SmartMotorControllerConfig](smart-motor-controller-config.md)
* [SmartMotorControllerTelemetryConfig (C++)](../../c++-reference/motor-controllers/smart-motor-controller-telemetry-config.md)
