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

---

## Control

| Method | Returns | Description |
|--------|---------|-------------|
| `setSwerveModuleState(SwerveModuleState state)` | `void` | Commands a drive speed and azimuth angle. |
| `seedAzimuthEncoder()` | `void` | Reads the absolute encoder and seeds the relative encoder to match. Call on robot enable. |

---

## Lifecycle

| Method | Returns | Description |
|--------|---------|-------------|
| `updateTelemetry()` | `void` | Publishes module state and encoder readings to NetworkTables. |
| `simIterate()` | `void` | Advances module simulation by one loop iteration. |

---

## Telemetry & DataLog

`updateTelemetry()` publishes this module's state and absolute encoder angle to NetworkTables. To additionally record the absolute encoder angle to a WPILib DataLog, set `SwerveModuleConfig.withDataLogName(String)` — see [SwerveModuleConfig](swerve-module-config.md#datalog-telemetry). For field-level control (or a DataLog name) on the drive/azimuth motors themselves, configure each motor's own `SmartMotorControllerConfig.withTelemetry(name, SmartMotorControllerTelemetryConfig)`.

---

## See Also

- [SwerveModuleConfig](swerve-module-config.md)
- [C++ SwerveModule](../../c++-reference/swerve/swerve-module.md)
