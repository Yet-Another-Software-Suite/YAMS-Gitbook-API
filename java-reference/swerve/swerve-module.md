# SwerveModule

**Package:** `yams.mechanisms.swerve`

Represents one swerve module: a drive motor and an azimuth (steering) motor. `SwerveDrive` manages an array of these internally. You generally do not construct `SwerveModule` directly.

## Constructor

```java
SwerveModule(SwerveModuleConfig config)
```

| Parameter | Type | Description |
|-----------|------|-------------|
| `config` | `SwerveModuleConfig` | Module-level configuration including motors, encoder offset, and location. |

---

## State

| Method | Returns | Description |
|--------|---------|-------------|
| `getState()` | `SwerveModuleState` | Current drive velocity and azimuth angle. |
| `getPosition()` | `SwerveModulePosition` | Integrated drive distance and azimuth angle. |
| `getName()` | `String` | Module name as set in config. |
| `getConfig()` | `SwerveModuleConfig` | The configuration this module was constructed with. |
| `getRawAbsoluteEncoderAngle()` | `Angle` | Absolute encoder angle with no offset applied: the raw reading captured at construction. |

---

## Control

| Method | Returns | Description |
|--------|---------|-------------|
| `setSwerveModuleState(SwerveModuleState state)` | `void` | Commands a drive speed and azimuth angle. |
| `setSwerveModuleState(SwerveModuleState state, Force feedforwardForce)` | `void` | Same, plus a drive-wheel feedforward `Force` applied on top (e.g. from a PathPlanner set-point generator), converted via the drive motor's `SmartMotorControllerConfig.convertToVoltage(...)`/`convertToCurrent(...)`. |
| `seedAzimuthEncoder()` | `void` | Reads the absolute encoder and seeds the relative encoder to match. Call on robot enable. |

---

## Lifecycle

| Method | Returns | Description |
|--------|---------|-------------|
| `updateTelemetry()` | `void` | Publishes module state and encoder readings to NetworkTables. |
| `simIterate()` | `void` | Advances module simulation by one loop iteration. |

---

## Telemetry & DataLog

`setupTelemetry(String mechName)` wires up this module's telemetry under `Mechanisms/<mechName>/modules/<moduleName>`; `SwerveDrive` calls it automatically for every module during its own construction, so you don't normally call it yourself. `updateTelemetry()` then publishes this module's `SwerveModuleState` and absolute encoder angle to NetworkTables at the verbosity from `SwerveModuleConfig.getTelemetryVerbosity()`, or from an explicit `SwerveModuleTelemetryConfig` passed via `SwerveModuleConfig.withTelemetry(name, SwerveModuleTelemetryConfig)`; see [SwerveModuleConfig](swerve-module-config.md#datalog-telemetry) for how to additionally record fields to a WPILib DataLog. For field-level control (or a DataLog name) on the drive/azimuth motors themselves, configure each motor's own `SmartMotorControllerConfig.withTelemetry(name, SmartMotorControllerTelemetryConfig)`.

{% hint style="warning" %}
`SwerveModuleConfig` has no `withDataLogName(String)` method: `setupTelemetry(...)` only consults the `SwerveModuleTelemetryConfig`'s own, separate DataLog name. Use `SwerveModuleConfig.withTelemetry(name, new SwerveModuleTelemetryConfig().withDataLogName(...))` instead; see [SwerveModuleConfig](swerve-module-config.md#datalog-telemetry).
{% endhint %}

---

## See Also

- [SwerveModuleConfig](swerve-module-config.md)
- [C++ SwerveModule](../../c++-reference/swerve/swerve-module.md)
