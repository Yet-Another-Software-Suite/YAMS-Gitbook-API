# SwerveDriveConfig

**Namespace:** `yams::mechanisms::swerve`\
**Header:** `yams/mechanisms/swerve/SwerveDriveConfig.hpp`\
**Java equivalent:** [Java SwerveDriveConfig](../../java-reference/swerve/swerve-drive-config.md)

Configures a complete `SwerveDrive<N>`. Uses a fluent builder pattern: every `With*` method returns `*this` for chaining.

***

## Constructor

```cpp
SwerveDriveConfig() = default;
```

Build the config with `WithSubsystem()` and `WithModules()`, then pass a pointer to it into `SwerveDrive<N>`'s constructor.

***

## Type Alias

```cpp
using TelemetryVerbosity =
    yams::motorcontrollers::SmartMotorControllerConfig::TelemetryVerbosity;
```

***

## Builder Methods

All methods return `SwerveDriveConfig&` for chaining.

| Method | Parameters | Description |
|--------|-----------|-------------|
| `WithSubsystem` | `frc2::SubsystemBase* subsystem` | Associates a command subsystem with this drive (used for command requirements). |
| `WithModules` | `std::vector<SwerveModule*> modules` | Sets the modules that make up this drive. Vector size must equal the `NumModules` template argument of `SwerveDrive`. |
| `WithGyro` | `std::function<units::degree_t()> gyroSupplier` | **Required.** Supplies the current gyro heading for field-relative control and odometry. |
| `WithGyroVelocity` | `std::function<units::degrees_per_second_t()> angularVelocitySupplier` | Supplies gyro angular velocity. Required for angular-velocity skew correction. |
| `WithGyroOffset` | `units::degree_t offset` | Offset subtracted from the raw gyro reading in `GetGyroAngle()`. |
| `WithGyroInverted` | `bool inverted` | Negates the raw gyro angle when `true`. |
| `WithStartingPose` | `frc::Pose2d pose` | Initial field-relative odometry pose. Defaults to the origin. |
| `WithMaximumChassisSpeed` | `units::meters_per_second_t speed, units::degrees_per_second_t angularVelocity` | Maximum chassis linear/angular speeds used for wheel-speed desaturation. |
| `WithMaximumModuleSpeed` | `units::meters_per_second_t speed` | Maximum module linear speed used for wheel-speed desaturation. |
| `WithCenterOfRotation` | `frc::Translation2d center` | Overrides the chassis center of rotation (default is the robot center). |
| `WithCenterOfRotation` | `units::meter_t forward, units::meter_t left` | Overrides the center of rotation via forward/left distances. |
| `WithDiscretizationTime` | `units::second_t dt` | Discretization timestep used to compensate for `ChassisSpeeds` skew. |
| `WithSimDiscretizationTime` | `units::second_t dt` | Discretization timestep used only in simulation. Falls back to `WithDiscretizationTime` if unset. |
| `WithGyroAngularVelocityScaleFactor` | `double scaleFactor` | Scale factor `[0, 1]` applied to gyro angular velocity for skew correction on the real robot. |
| `WithSimGyroAngularVelocityScaleFactor` | `double scaleFactor` | Scale factor used only in simulation. Falls back to `WithGyroAngularVelocityScaleFactor` if unset. |
| `WithTranslationController` | `frc::PIDController controller` | PID controller (meters) used by `DriveToPose`. |
| `WithRotationController` | `frc::PIDController controller` | PID controller (radians) used by `DriveToPose`. Continuous input over `[-π, π]` is enabled automatically. |
| `WithSimTranslationController` | `frc::PIDController controller` | Translation controller used only in simulation. Falls back to `WithTranslationController` if unset. |
| `WithSimRotationController` | `frc::PIDController controller` | Rotation controller used only in simulation. Falls back to `WithRotationController` if unset. |
| `WithTelemetry` | `const std::string& name, TelemetryVerbosity verbosity` | Sets the telemetry name (NetworkTables/DataLog prefix, defaults to `"swerve"`) and verbosity for drive-level fields (pose, gyro, chassis speeds, module states, auto-align PID gains) in one call. |
| `WithTelemetry` | `const std::string& name, telemetry::SwerveDriveTelemetryConfig telemetryConfig` | Sets the telemetry name and configures telemetry with an explicit `SwerveDriveTelemetryConfig`, taking precedence over the verbosity-based overload. |

{% hint style="info" %}
`WithGyro` is required. Omitting it causes `GetGyroAngle()` to throw and prevents field-relative control and odometry from functioning correctly.

Both `WithTelemetry` overloads require a name; there is no name-less `WithTelemetry(...)` or standalone `WithTelemetryName(...)` on `SwerveDriveConfig` (unlike, say, `SmartMotorControllerConfig`). `SwerveDriveConfig` also has no `WithDataLogName(...)`/`GetDataLogName()` of its own: DataLog output is configured on a `SwerveDriveTelemetryConfig` instead. See below.
{% endhint %}

***

## DataLog Telemetry

`SwerveDriveConfig` has no `WithDataLogName(...)` method: DataLog output is configured on a [`SwerveDriveTelemetryConfig`](../../c++-reference/swerve/swerve-drive.md#telemetry--datalog) and wired in via `WithTelemetry(name, SwerveDriveTelemetryConfig)`:

```cpp
SwerveDriveConfig driveConfig;
driveConfig.WithSubsystem(this)
    .WithModules({&frontLeft, &frontRight, &backLeft, &backRight})
    .WithGyro([gyroPtr = &gyro]() -> units::degree_t {
      return units::degree_t{units::turn_t{gyroPtr->GetYaw().GetValue()}};
    })
    .WithTelemetry("swerve",
        yams::telemetry::SwerveDriveTelemetryConfig{SwerveDriveConfig::TelemetryVerbosity::HIGH}
            .WithDataLogName("swerve"));  // logs pose, gyro, chassis speeds, module states
```

`SwerveDriveTelemetryConfig{TelemetryVerbosity}` is shorthand for `SwerveDriveTelemetryConfig{}.WithTelemetryVerbosity(verbosity)`.

{% hint style="warning" %}
`WithTelemetry(name, SwerveDriveTelemetryConfig)` replaces the default field selection entirely; call `WithTelemetryVerbosity(...)` (or the individual `With*()` field methods) on it yourself, since a default-constructed `SwerveDriveTelemetryConfig` has every field disabled. This DataLog name does **not** cascade to the modules passed to `WithModules()`, or to those modules' drive/azimuth motors. Each `SwerveModuleConfig` needs its own telemetry setup (see [SwerveModuleConfig](swerve-module-config.md#datalog-telemetry)), and each motor's `SmartMotorControllerConfig` needs its own `WithTelemetry(name, SmartMotorControllerTelemetryConfig)` for field-level control and its own DataLog name.
{% endhint %}

***

## Getters

| Method | Signature | Description |
|--------|-----------|-------------|
| `GetModules` | `const std::vector<SwerveModule*>& GetModules() const` | The configured module pointers. |
| `GetSubsystem` | `frc2::SubsystemBase* GetSubsystem() const` | The associated subsystem. |
| `GetInitialPose` | `frc::Pose2d GetInitialPose() const` | The starting pose. |
| `GetMaximumChassisLinearVelocity` | `std::optional<units::meters_per_second_t> GetMaximumChassisLinearVelocity() const` | Optional max chassis linear speed. |
| `GetMaximumChassisAngularVelocity` | `std::optional<units::degrees_per_second_t> GetMaximumChassisAngularVelocity() const` | Optional max chassis angular speed. |
| `GetMaximumModuleLinearVelocity` | `std::optional<units::meters_per_second_t> GetMaximumModuleLinearVelocity() const` | Optional max module speed. |
| `GetCenterOfRotation` | `std::optional<frc::Translation2d> GetCenterOfRotation() const` | Optional center-of-rotation override. |
| `GetTelemetryVerbosity` | `std::optional<TelemetryVerbosity> GetTelemetryVerbosity() const` | Configured telemetry verbosity, if any. |
| `GetTelemetryName` | `const std::string& GetTelemetryName() const` | Telemetry name prefix for the drive (defaults to `"swerve"`). |
| `GetSwerveDriveTelemetryConfig` | `std::optional<telemetry::SwerveDriveTelemetryConfig> GetSwerveDriveTelemetryConfig()` | The explicit config passed to `WithTelemetry(name, SwerveDriveTelemetryConfig)`, if any. Moves it out; intended to be called exactly once, by `SwerveDrive` itself. |
| `GetGyroOffset` | `units::degree_t GetGyroOffset() const` | Stored gyro offset (zero if never set). |
| `GetGyroAngle` | `units::degree_t GetGyroAngle() const` | Gyro angle with inversion and offset applied. Throws if no gyro supplier was configured. |
| `GetTranslationPID` | `frc::PIDController& GetTranslationPID()` | Active translation controller (sim variant if simulating and configured). |
| `GetRotationPID` | `frc::PIDController& GetRotationPID()` | Active rotation controller (sim variant if simulating and configured). |

***

## Example

```cpp
SwerveDriveConfig config;
config.WithSubsystem(this)
    .WithModules({&m_fl.value(), &m_fr.value(), &m_bl.value(), &m_br.value()})
    .WithGyro([gyroPtr = &m_gyro]() -> units::degree_t {
      return units::degree_t{units::turn_t{gyroPtr->GetYaw().GetValue()}};
    })
    .WithMaximumChassisSpeed(units::meters_per_second_t{4.5},
                             units::degrees_per_second_t{360})
    .WithStartingPose(frc::Pose2d{units::meter_t{0}, units::meter_t{0}, frc::Rotation2d{}})
    .WithTranslationController(frc::PIDController{1, 0, 0})
    .WithRotationController(frc::PIDController{1, 0, 0})
    .WithTelemetry("swerve",
        yams::telemetry::SwerveDriveTelemetryConfig{SwerveDriveConfig::TelemetryVerbosity::HIGH}
            .WithDataLogName("swerve"));

swerve::SwerveDrive<4> m_drive{&config};
```

***

## See Also

* [Java SwerveDriveConfig](../../java-reference/swerve/swerve-drive-config.md)
* [SwerveDrive](swerve-drive.md)
* [SwerveModuleConfig](swerve-module-config.md)
