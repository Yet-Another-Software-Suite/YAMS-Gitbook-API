# SwerveModule

**Namespace:** `yams::mechanisms::swerve`\
**Java equivalent:** [Java SwerveModule](../../java-reference/swerve/swerve-module.md)

`SwerveModule` represents one swerve module: a drive motor and an azimuth motor. `SwerveDrive<N>` manages an array of these internally; you generally do not construct them directly.

***

## Constructor

```cpp
explicit SwerveModule(config::SwerveModuleConfig* config)
```

***

## State

| Method        | Signature                                             | Description                                  |
| ------------- | ----------------------------------------------------- | -------------------------------------------- |
| `GetState`    | `frc::SwerveModuleState GetState() const`             | Current drive velocity and azimuth angle.    |
| `GetPosition` | `frc::SwerveModulePosition GetPosition() const`       | Integrated drive distance and azimuth angle. |
| `GetName`     | `std::string GetName() const`                         | Module name from config.                     |
| `GetConfig`   | `const config::SwerveModuleConfig& GetConfig() const` | Module configuration.                        |
| `GetRawAbsoluteEncoderAngle` | `units::degree_t GetRawAbsoluteEncoderAngle() const` | Absolute encoder angle with no offset applied: the raw reading captured at construction. |

***

## Control

| Method                 | Signature                                                 | Description                                                                      |
| ---------------------- | --------------------------------------------------------- | -------------------------------------------------------------------------------- |
| `SetSwerveModuleState` | `void SetSwerveModuleState(frc::SwerveModuleState state)` | Commands drive speed and azimuth angle.                                          |
| `SeedAzimuthEncoder`   | `void SeedAzimuthEncoder()`                               | Reads the absolute encoder and seeds the relative encoder. Call on robot enable. |

***

## Motor Access

| Method                      | Signature                                                                   | Description                   |
| --------------------------- | --------------------------------------------------------------------------- | ----------------------------- |
| `GetDriveMotorController`   | `motorcontrollers::SmartMotorController* GetDriveMotorController() const`   | Pointer to the drive motor.   |
| `GetAzimuthMotorController` | `motorcontrollers::SmartMotorController* GetAzimuthMotorController() const` | Pointer to the azimuth motor. |

***

## Lifecycle

| Method            | Signature                | Description                              |
| ----------------- | ------------------------ | ---------------------------------------- |
| `UpdateTelemetry` | `void UpdateTelemetry()` | Publishes module state to NetworkTables. |
| `SimIterate`      | `void SimIterate()`      | Advances module simulation.              |

***

## Telemetry & DataLog

`SetupTelemetry(const std::string& mechName)` wires up this module's telemetry under `Mechanisms/<mechName>/modules/<moduleName>`; `SwerveDrive<N>` calls it automatically for every module during its own construction, so you don't normally call it yourself. `UpdateTelemetry()` then publishes this module's `SwerveModuleState` and absolute encoder angle to NetworkTables at the verbosity from `SwerveModuleConfig::GetTelemetryVerbosity()`, or from an explicit `telemetry::SwerveModuleTelemetryConfig` passed via `SwerveModuleConfig::WithTelemetry(name, SwerveModuleTelemetryConfig)`; see [SwerveModuleConfig](swerve-module-config.md#datalog-telemetry) for how to additionally record fields to a WPILib DataLog. For field-level control (or a DataLog name) on the drive/azimuth motors themselves, configure each motor's own `SmartMotorControllerConfig::WithTelemetry(name, SmartMotorControllerTelemetryConfig)`.

{% hint style="warning" %}
`SwerveModuleConfig` has no `WithDataLogName(const std::string&)` method: `SetupTelemetry(...)` only consults the `SwerveModuleTelemetryConfig`'s own, separate DataLog name. Use `SwerveModuleConfig::WithTelemetry(name, telemetry::SwerveModuleTelemetryConfig{}.WithDataLogName(...))` instead; see [SwerveModuleConfig](swerve-module-config.md#datalog-telemetry).
{% endhint %}

***

## See Also

* [Java SwerveModule](../../java-reference/swerve/swerve-module.md)
* [SwerveModuleConfig](swerve-module-config.md)
* [SwerveDrive\<N>](swerve-drive.md)
