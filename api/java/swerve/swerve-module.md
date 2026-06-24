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

## See Also

- [SwerveModuleConfig](swerve-module-config.md)
