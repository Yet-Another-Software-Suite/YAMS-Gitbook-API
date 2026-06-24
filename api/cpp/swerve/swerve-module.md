# SwerveModule

**Namespace:** `yams::mechanisms::swerve`\
**Java equivalent:** [Java SwerveModule](../../java/swerve/swerve-module.md)

`SwerveModule` represents one swerve module: a drive motor and an azimuth motor. `SwerveDrive<N>` manages an array of these internally; you generally do not construct them directly.

---

## Constructor

```cpp
explicit SwerveModule(config::SwerveModuleConfig* config)
```

---

## State

| Method | Signature | Description |
|--------|-----------|-------------|
| `GetState` | `frc::SwerveModuleState GetState() const` | Current drive velocity and azimuth angle. |
| `GetPosition` | `frc::SwerveModulePosition GetPosition() const` | Integrated drive distance and azimuth angle. |
| `GetName` | `std::string GetName() const` | Module name from config. |
| `GetConfig` | `const config::SwerveModuleConfig& GetConfig() const` | Module configuration. |

---

## Control

| Method | Signature | Description |
|--------|-----------|-------------|
| `SetSwerveModuleState` | `void SetSwerveModuleState(frc::SwerveModuleState state)` | Commands drive speed and azimuth angle. |
| `SeedAzimuthEncoder` | `void SeedAzimuthEncoder()` | Reads the absolute encoder and seeds the relative encoder. Call on robot enable. |

---

## Motor Access

| Method | Signature | Description |
|--------|-----------|-------------|
| `GetDriveMotorController` | `motorcontrollers::SmartMotorController* GetDriveMotorController() const` | Pointer to the drive motor. |
| `GetAzimuthMotorController` | `motorcontrollers::SmartMotorController* GetAzimuthMotorController() const` | Pointer to the azimuth motor. |

---

## Lifecycle

| Method | Signature | Description |
|--------|-----------|-------------|
| `UpdateTelemetry` | `void UpdateTelemetry()` | Publishes module state to NetworkTables. |
| `SimIterate` | `void SimIterate()` | Advances module simulation. |

---

## See Also

- [Java SwerveModule](../../java/swerve/swerve-module.md)
- [SwerveDrive\<N>](swerve-drive.md)
