# SwerveModuleConfig

**Namespace:** `yams::mechanisms::config`\
**Header:** `yams/mechanisms/config/SwerveModuleConfig.hpp`\
**Java equivalent:** [Java SwerveModuleConfig](../../java-reference/swerve/swerve-module-config.md)

Configures one swerve module. Takes pointers to the drive and azimuth `SmartMotorController`s. Uses a fluent builder pattern — every `With*` method returns `*this` for chaining.

***

## Constructors

```cpp
SwerveModuleConfig() = default;

SwerveModuleConfig(motorcontrollers::SmartMotorController* drive,
                   motorcontrollers::SmartMotorController* azimuth);
```

| Parameter | Type | Description |
|-----------|------|-------------|
| `drive` | `SmartMotorController*` | Motor controller for the drive wheel (must outlive this config). |
| `azimuth` | `SmartMotorController*` | Motor controller for the steering mechanism (must outlive this config). |

Use the default constructor plus `WithSmartMotorController()` when the motors aren't available yet at construction time.

***

## Type Alias

```cpp
using TelemetryVerbosity =
    yams::motorcontrollers::SmartMotorControllerConfig::TelemetryVerbosity;
```

***

## Builder Methods

All methods return `SwerveModuleConfig&` for chaining.

| Method | Parameters | Description |
|--------|-----------|-------------|
| `WithSmartMotorController` | `SmartMotorController* driveMotor, SmartMotorController* azimuthMotor` | Sets both motor controllers (use with the default constructor). |
| `WithCosineCompensation` | `bool compensate` | Scales drive speed by `cos(angle_error)` to reduce skew while steering. |
| `WithAbsoluteEncoder` | `std::function<units::degree_t()> supplier` | Supplies an external (cross-vendor) absolute encoder angle. |
| `WithAbsoluteEncoderGearing` | `const gearing::GearBox& gearing` | Gearing between the absolute encoder shaft and the azimuth mechanism. |
| `WithAbsoluteEncoderOffset` | `units::degree_t offset` | Offset so zero corresponds to the wheel pointing forward, bevel left. |
| `WithLocation` | `frc::Translation2d location` | Module position from the robot center (X forward, Y left), in meters. |
| `WithLocation` | `units::meter_t front, units::meter_t left` | Module position via separate forward/left distances. |
| `WithLocation` | `units::meter_t distance, units::degree_t angle` | Module position via polar coordinates from the center of rotation. |
| `WithWheelRadius` | `units::meter_t radius` | Wheel radius; derives the circumference used for linear-distance conversion. |
| `WithWheelDiameter` | `units::meter_t diameter` | Wheel diameter; derives the circumference used for linear-distance conversion. |
| `WithMinimumVelocity` | `units::meters_per_second_t speed` | Below this speed, the module holds its current azimuth instead of tracking. |
| `WithOptimization` | `bool enable` | Enables/disables `SwerveModuleState` optimization (flip by 180° to minimize rotation). Default `true`. |
| `WithTelemetry` | `const std::string& name, TelemetryVerbosity verbosity` | Sets the telemetry name and verbosity for this module's own fields (absolute encoder angle). |
| `WithDataLogName` | `const std::string& dataLogName` | Logs this module's telemetry (currently just the absolute encoder field) to a WPILib DataLog under this name, in addition to NetworkTables. See below. |

{% hint style="info" %}
`WithAbsoluteEncoderOffset` must be calibrated per module. An incorrect offset causes the module to drive at an angle on startup until `SeedAzimuthEncoder()` corrects it.
{% endhint %}

***

## DataLog Telemetry

`WithDataLogName(name)` sends this module's absolute-encoder field to a WPILib `DataLog` under the given prefix, in addition to NetworkTables:

```cpp
SwerveModuleConfig frontLeftConfig{&driveMotorFL, &azimuthMotorFL};
frontLeftConfig.WithWheelRadius(units::meter_t{0.0508})
    .WithLocation(units::meter_t{0.381}, units::meter_t{0.381})
    .WithTelemetry("FrontLeft", SwerveModuleConfig::TelemetryVerbosity::HIGH)
    .WithDataLogName("swerve/frontLeft");  // logs this module's absolute encoder angle only
```

{% hint style="warning" %}
`WithDataLogName` on `SwerveModuleConfig` is independent of `SwerveDriveConfig::WithDataLogName` (neither cascades to the other) and does **not** cascade to this module's drive/azimuth motors. To get field-level DataLog control (or a DataLog name) on the drive or azimuth motor itself, configure that motor's own `SmartMotorControllerConfig::WithTelemetry(name, SmartMotorControllerTelemetryConfig)` before constructing the `SwerveModule` — see [SmartMotorControllerTelemetryConfig](../motor-controllers/smart-motor-controller-telemetry-config.md).
{% endhint %}

***

## Getters

| Method | Signature | Description |
|--------|-----------|-------------|
| `GetDriveMotor` | `motorcontrollers::SmartMotorController* GetDriveMotor() const` | The drive motor controller pointer. |
| `GetAzimuthMotor` | `motorcontrollers::SmartMotorController* GetAzimuthMotor() const` | The azimuth motor controller pointer. |
| `GetTelemetryName` | `std::optional<std::string> GetTelemetryName() const` | Configured telemetry name, if any. |
| `GetTelemetryVerbosity` | `std::optional<TelemetryVerbosity> GetTelemetryVerbosity() const` | Configured telemetry verbosity, if any. |
| `GetLocation` | `std::optional<frc::Translation2d> GetLocation() const` | Module location from the center of rotation. |
| `GetStateOptimization` | `bool GetStateOptimization() const` | Whether state optimization is enabled. |
| `GetAbsoluteEncoderAngle` | `units::degree_t GetAbsoluteEncoderAngle() const` | Current absolute encoder angle with gearing and offset applied. Falls back to the azimuth motor's mechanism position if no encoder supplier is set. |
| `GetRawAbsoluteEncoderAngle` | `std::function<units::degree_t()> GetRawAbsoluteEncoderAngle() const` | Absolute encoder angle supplier without offsets applied. Falls back to the azimuth motor's mechanism position (plus its external encoder zero offset, on a real robot) if no encoder supplier is set. |
| `GetAbsoluteEncoderSupplier` | `std::optional<std::function<units::degree_t()>> GetAbsoluteEncoderSupplier() const` | The absolute encoder supplier, if configured via `WithAbsoluteEncoder()`. |
| `GetDataLogName` | `std::optional<std::string> GetDataLogName() const` | Configured DataLog name prefix, if any. |
| `GetOptimizedState` | `frc::SwerveModuleState GetOptimizedState(frc::SwerveModuleState state) const` | Applies all enabled optimizations (min-velocity clamp, state optimization, cosine compensation) to `state`. |

***

## Example

```cpp
SwerveModuleConfig frontLeftConfig{&driveSMC, &azimuthSMC};
frontLeftConfig
    .WithAbsoluteEncoder([encoderPtr]() -> units::degree_t {
      return units::degree_t{units::turn_t{encoderPtr->GetAbsolutePosition().GetValue()}};
    })
    .WithLocation(units::inch_t{24}, units::inch_t{24})
    .WithOptimization(true)
    .WithTelemetry("FrontLeft", SwerveModuleConfig::TelemetryVerbosity::HIGH)
    .WithDataLogName("swerve/frontLeft");
```

***

## See Also

* [Java SwerveModuleConfig](../../java-reference/swerve/swerve-module-config.md)
* [SwerveModule](swerve-module.md)
* [SwerveDriveConfig](swerve-drive-config.md)
