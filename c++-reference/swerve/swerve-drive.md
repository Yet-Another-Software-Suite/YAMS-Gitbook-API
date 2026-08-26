# SwerveDrive

**Namespace:** `yams::mechanisms::swerve`\
**Template:** `template <size_t NumModules = 4>`\
**Java equivalent:** [Java SwerveDrive](../../java-reference/swerve/swerve-drive.md)

`SwerveDrive<N>` manages a swerve drivetrain with `N` modules (default 4). The template parameter must match the number of `SwerveModuleConfig` objects passed to `SwerveDriveConfig`.

***

## Constructor

```cpp
explicit SwerveDrive(SwerveDriveConfig* config)
```

***

## Driving

| Method                          | Signature                                                                           | Description                                                                                    |
| ------------------------------- | ----------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------- |
| `Drive`                         | `frc2::CommandPtr Drive(std::function<frc::ChassisSpeeds()> robotRelativeSpeeds)`   | Continuously sets robot-relative speeds from a supplier. Runs indefinitely.                    |
| `DriveToPose`                   | `frc2::CommandPtr DriveToPose(frc::Pose2d pose)`                                    | Drives to a field-relative pose using the configured translation and rotation PID controllers. |
| `SetRobotRelativeChassisSpeeds` | `void SetRobotRelativeChassisSpeeds(frc::ChassisSpeeds robotRelativeSpeeds)`        | Sets speeds directly (call from periodic, not a command).                                      |
| `SetFieldRelativeChassisSpeeds` | `void SetFieldRelativeChassisSpeeds(frc::ChassisSpeeds fieldRelativeSpeeds)`        | Sets field-relative speeds, rotated by gyro heading.                                           |
| `SetSwerveModuleStates`         | `void SetSwerveModuleStates(wpi::array<frc::SwerveModuleState, NumModules> states)` | Commands module states directly.                                                               |
| `LockPose`                      | `void LockPose()`                                                                   | Sets all modules to an X-pattern (for static defense).                                         |

***

## Odometry & Pose

| Method                                | Signature                                                                               | Description                                           |
| ------------------------------------- | --------------------------------------------------------------------------------------- | ----------------------------------------------------- |
| `GetPose`                             | `frc::Pose2d GetPose()`                                                                 | Current estimated robot pose from the pose estimator. |
| `ResetOdometry`                       | `void ResetOdometry(frc::Pose2d pose)`                                                  | Resets the estimator and reseeds azimuth encoders.    |
| `ZeroGyro`                            | `void ZeroGyro()`                                                                       | Resets the gyro heading to zero.                      |
| `AddVisionMeasurement`                | `void AddVisionMeasurement(frc::Pose2d robotPose, units::second_t timestamp)`           | Fuses a vision pose measurement.                      |
| `AddVisionMeasurement` (with stddevs) | `void AddVisionMeasurement(frc::Pose2d, units::second_t, const wpi::array<double, 3>&)` | Vision measurement with custom standard deviations.   |
| `SetVisionMeasurementStdDevs`         | `void SetVisionMeasurementStdDevs(const wpi::array<double, 3>& stdDevs)`                | Sets default vision measurement standard deviations.  |

***

## State Queries

| Method                       | Signature                                                                | Description                                               |
| ---------------------------- | ------------------------------------------------------------------------ | --------------------------------------------------------- |
| `GetRobotRelativeSpeed`      | `frc::ChassisSpeeds GetRobotRelativeSpeed()`                             | Current robot-relative chassis speeds.                    |
| `GetFieldRelativeSpeed`      | `frc::ChassisSpeeds GetFieldRelativeSpeed()`                             | Current field-relative chassis speeds.                    |
| `GetModulePositions`         | `wpi::array<frc::SwerveModulePosition, NumModules> GetModulePositions()` | Integrated positions of all modules.                      |
| `GetModuleStates`            | `wpi::array<frc::SwerveModuleState, NumModules> GetModuleStates()`       | Current state of all modules.                             |
| `GetGyroAngle`               | `units::degree_t GetGyroAngle()`                                         | Current gyro heading.                                     |
| `GetDistanceFromPose`        | `units::meter_t GetDistanceFromPose(frc::Pose2d pose)`                   | Translational distance from current pose to `pose`.       |
| `GetAngleDifferenceFromPose` | `units::degree_t GetAngleDifferenceFromPose(frc::Pose2d pose)`           | Heading difference to face `pose`.                        |
| `GetModule`                  | `std::optional<SwerveModule*> GetModule(const std::string& name)`        | Returns a specific module by name, or empty if not found. |
| `GetDesiredChassisSpeeds`    | `frc::ChassisSpeeds GetDesiredChassisSpeeds()`                           | Last-commanded robot-relative setpoint (not a measurement). |
| `GetDesiredModuleStates`     | `wpi::array<frc::SwerveModuleState, NumModules> GetDesiredModuleStates()` | Last-commanded module states (not a measurement). |
| `GetRobotRelativeChassisSpeedsFromState` | `frc::ChassisSpeeds GetRobotRelativeChassisSpeedsFromState(const wpi::array<frc::SwerveModuleState, NumModules>&)` | Converts an arbitrary set of module states back into robot-relative chassis speeds. |
| `GetSimPose`                 | `frc::Pose2d GetSimPose()`                                               | Returns the simulated ground-truth pose. See below.        |
| `GetField2d`                 | `frc::Field2d& GetField2d()`                                             | Returns the `Field2d` widget the drive already publishes to SmartDashboard. |

{% hint style="warning" %}
`GetDesiredChassisSpeeds()` returns a **setpoint**, not the robot's actual measured motion — it is whatever was last passed to `SetRobotRelativeChassisSpeeds(...)` (directly, or via `SetFieldRelativeChassisSpeeds(...)`/`Drive(...)`), cached and republished every `UpdateTelemetry()` call. If nothing has commanded the drive yet, it returns a zeroed `frc::ChassisSpeeds`. For the drive's actual measured speed, use `GetRobotRelativeSpeed()` / `GetFieldRelativeSpeed()` instead.
{% endhint %}

{% hint style="info" %}
There is no direct accessor for the underlying `frc::SwerveDrivePoseEstimator` — `SwerveDrive` owns it exclusively to avoid callers accidentally calling `Update(...)`/`ResetPosition(...)`/`ResetPose(...)` on it directly (which would corrupt the pose estimate by feeding it duplicate or out-of-order samples, or desyncing it from the drive's gyro offset). Use `GetPose()`, `ResetOdometry(frc::Pose2d)`, and `AddVisionMeasurement(...)` instead — they cover the estimator interactions `SwerveDrive` supports.
{% endhint %}

{% hint style="info" %}
`GetSimPose()` returns a separate, ground-truth `frc::Pose2d` that assumes every module reached its last-commanded `frc::SwerveModuleState` perfectly — it is **not** the same as `GetPose()` (the noisy, gyro/odometry-fused estimate). It's updated in simulation by `SimIterate()`, by integrating a `frc::Twist2d` built from the desired module states, and is also snapped to the given pose by `ResetOdometry(frc::Pose2d)` so it stays in sync with the fused estimate. On real hardware it stays at the configured starting pose. This makes it a convenient "known truth" pose to feed into a simulated vision system (e.g. to generate synthetic AprilTag detections) so you can test vision code end-to-end without a physical camera.
{% endhint %}

***

## PID Reset

| Method                | Signature                    | Description                                  |
| --------------------- | ---------------------------- | -------------------------------------------- |
| `ResetAzimuthPID`     | `void ResetAzimuthPID()`     | Resets the azimuth PID controller state.     |
| `ResetTranslationPID` | `void ResetTranslationPID()` | Resets the translation PID controller state. |

***

## Lifecycle

| Method            | Signature                                                 | Description                                                 |
| ----------------- | --------------------------------------------------------- | ----------------------------------------------------------- |
| `UpdateTelemetry` | `void UpdateTelemetry()`                                  | Publishes pose, module states, and speeds to NetworkTables. |
| `SimIterate`      | `void SimIterate()`                                       | Advances swerve simulation one loop iteration.              |
| `GetName`         | `std::string GetName() const`                             | Drivetrain name from config.                                |
| `GetConfig`       | `SwerveDriveConfig& GetConfig()`                          | Returns a reference to the config.                          |
| `GetKinematics`   | `frc::SwerveDriveKinematics<NumModules>& GetKinematics()` | Returns the kinematics object.                              |

***

## Telemetry & DataLog

`UpdateTelemetry()` publishes pose, gyro, chassis speeds, and module states to NetworkTables under `Mechanisms/<name>` at the verbosity configured via `SwerveDriveConfig::WithTelemetry(TelemetryVerbosity)`. To additionally record those fields to a WPILib DataLog (for offline review in AdvantageScope), set `SwerveDriveConfig::WithDataLogName(const std::string&)` — see [SwerveDriveConfig](swerve-drive-config.md#datalog-telemetry).

***

## Example

```cpp
SwerveDriveConfig config{this};
// ... configure modules, gyro, speeds ...

swerve::SwerveDrive<4> m_drive{&config};

// In a command factory:
frc2::CommandPtr DriveCommand(std::function<frc::ChassisSpeeds()> speeds) {
    return m_drive.Drive(speeds);
}
```

{% hint style="info" %}
`SwerveDrive<4>` is the most common instantiation. The template parameter must match the number of modules in `SwerveDriveConfig`. Deviating from 4 is rare in FRC.
{% endhint %}

{% @github-files/github-code-block url="https://github.com/Yet-Another-Software-Suite/YAMS/blob/master/examples/cpptest/src/main/cpp/subsystems/SwerveSubsystem.cpp#L130-L139" %}

***

## See Also

* [Java SwerveDrive](../../java-reference/swerve/swerve-drive.md)
* [SwerveDriveConfig](swerve-drive-config.md)
* [SwerveModule](swerve-module.md)
* [SwerveInputStream](swerve-input-stream.md)
