# SwerveDrive

**Namespace:** `yams::mechanisms::swerve`\
**Header:** `#include <yams/mechanisms/swerve/SwerveDrive.hpp>`\
**Template:** `template <size_t NumModules = 4>`\
**Java equivalent:** [Java SwerveDrive](../../java-reference/swerve/swerve-drive.md)

`SwerveDrive<N>` manages a swerve drivetrain with `N` modules (default 4). The template parameter must match the number of modules passed to `SwerveDriveConfig::WithModules(...)`.

{% hint style="info" %}
The C++ API targets WPILib Commands v2 only (`wpi::cmd::CommandPtr`, `wpi::cmd::SubsystemBase`). WPILib has no C++ Commands v3, so there is no C++ equivalent of the Java `yams.commands3` bindings.
{% endhint %}

***

## Constructor

```cpp
explicit SwerveDrive(SwerveDriveConfig* config)
```

***

## Driving

| Method                          | Signature                                                                           | Description                                                                                    |
| ------------------------------- | ----------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------- |
| `Drive`                         | `wpi::cmd::CommandPtr Drive(std::function<wpi::math::ChassisVelocities()> robotRelativeSpeeds)`   | Continuously sets robot-relative speeds from a supplier. Runs indefinitely.                    |
| `DriveToPose`                   | `wpi::cmd::CommandPtr DriveToPose(wpi::math::Pose2d pose)`                                    | Drives to a field-relative pose using the configured translation and rotation PID controllers. |
| `SetRobotRelativeChassisSpeeds` | `void SetRobotRelativeChassisSpeeds(wpi::math::ChassisVelocities robotRelativeSpeeds)`        | Sets speeds directly (call from periodic, not a command).                                      |
| `SetFieldRelativeChassisSpeeds` | `void SetFieldRelativeChassisSpeeds(wpi::math::ChassisVelocities fieldRelativeSpeeds)`        | Sets field-relative speeds, rotated by gyro heading.                                           |
| `SetSwerveModuleStates`         | `void SetSwerveModuleStates(wpi::util::array<wpi::math::SwerveModuleVelocity, NumModules> states)` | Commands module states directly.                                                               |
| `LockPose`                      | `void LockPose()`                                                                   | Commands every module to zero speed with its wheel pointed along its location vector (the X pattern) and clears the desired chassis speeds. Call it every loop (for example from a `Run` command) to hold the lock. |
| `SetAzimuthAngles`              | `void SetAzimuthAngles(wpi::math::Rotation2d angle)`                                   | Turns every module to `angle` with zero drive speed, skipping the velocity deadband that would hold the wheels where they are, and clears the desired chassis speeds. Used by the azimuth live tuning. |
| `GetStateFromRobotRelativeChassisSpeeds` | `wpi::util::array<wpi::math::SwerveModuleVelocity, NumModules> GetStateFromRobotRelativeChassisSpeeds(wpi::math::ChassisVelocities robotRelativeSpeeds)` | Converts robot-relative speeds into module states, applying `SwerveDriveConfig::OptimizeRobotRelativeChassisSpeeds(...)` and the configured center of rotation. |

***

## Odometry & Pose

| Method                                | Signature                                                                               | Description                                           |
| ------------------------------------- | --------------------------------------------------------------------------------------- | ----------------------------------------------------- |
| `GetPose`                             | `wpi::math::Pose2d GetPose()`                                                                 | Current estimated robot pose from the pose estimator. |
| `ResetOdometry`                       | `void ResetOdometry(wpi::math::Pose2d pose)`                                                  | Resets the estimator and reseeds azimuth encoders.    |
| `ZeroGyro`                            | `void ZeroGyro()`                                                                       | Resets the gyro heading to zero.                      |
| `AddVisionMeasurement`                | `void AddVisionMeasurement(wpi::math::Pose2d robotPose, wpi::units::second_t timestamp)`           | Fuses a vision pose measurement.                      |
| `AddVisionMeasurement` (with stddevs) | `void AddVisionMeasurement(wpi::math::Pose2d, wpi::units::second_t, const wpi::util::array<double, 3>&)` | Vision measurement with custom standard deviations.   |
| `SetVisionMeasurementStdDevs`         | `void SetVisionMeasurementStdDevs(const wpi::util::array<double, 3>& stdDevs)`                | Sets default vision measurement standard deviations.  |

***

## State Queries

| Method                       | Signature                                                                | Description                                               |
| ---------------------------- | ------------------------------------------------------------------------ | --------------------------------------------------------- |
| `GetRobotRelativeSpeed`      | `wpi::math::ChassisVelocities GetRobotRelativeSpeed()`                             | Current robot-relative chassis speeds.                    |
| `GetFieldRelativeSpeed`      | `wpi::math::ChassisVelocities GetFieldRelativeSpeed()`                             | Current field-relative chassis speeds.                    |
| `GetModulePositions`         | `wpi::util::array<wpi::math::SwerveModulePosition, NumModules> GetModulePositions()` | Integrated positions of all modules.                      |
| `GetModuleStates`            | `wpi::util::array<wpi::math::SwerveModuleVelocity, NumModules> GetModuleStates()`       | Current state of all modules.                             |
| `GetGyroAngle`               | `wpi::units::degree_t GetGyroAngle()`                                    | Current gyro heading (the simulated heading in simulation). |
| `GetDistanceFromPose`        | `wpi::units::meter_t GetDistanceFromPose(wpi::math::Pose2d pose)`                   | Translational distance from current pose to `pose`.       |
| `GetAngleDifferenceFromPose` | `wpi::units::degree_t GetAngleDifferenceFromPose(wpi::math::Pose2d pose)`           | Heading difference to face `pose`.                        |
| `GetModule`                  | `std::optional<SwerveModule*> GetModule(const std::string& name)`        | Returns a specific module by name, or empty if not found. |
| `GetDesiredChassisSpeeds`    | `wpi::math::ChassisVelocities GetDesiredChassisSpeeds()`                           | Last-commanded robot-relative setpoint (not a measurement). |
| `GetDesiredModuleStates`     | `wpi::util::array<wpi::math::SwerveModuleVelocity, NumModules> GetDesiredModuleStates()` | Last-commanded module states (not a measurement). |
| `GetRobotRelativeChassisSpeedsFromState` | `wpi::math::ChassisVelocities GetRobotRelativeChassisSpeedsFromState(const wpi::util::array<wpi::math::SwerveModuleVelocity, NumModules>&)` | Converts an arbitrary set of module states back into robot-relative chassis speeds. |
| `GetSimPose`                 | `wpi::math::Pose2d GetSimPose()`                                               | Returns the simulated ground-truth pose. See below.        |
| `GetField2d`                 | `wpi::Field2d& GetField2d()`                                             | Returns the `Field2d` the drive already publishes at `Mechanisms/<name>/field`. Use it instead of creating your own `Field2d`. |

{% hint style="warning" %}
`GetDesiredChassisSpeeds()` returns a **setpoint**, not the robot's actual measured motion: it is whatever was last passed to `SetRobotRelativeChassisSpeeds(...)` (directly, or via `SetFieldRelativeChassisSpeeds(...)`/`Drive(...)`), cached and republished every `UpdateTelemetry()` call. If nothing has commanded the drive yet, it returns a zeroed `wpi::math::ChassisVelocities`. For the drive's actual measured speed, use `GetRobotRelativeSpeed()` / `GetFieldRelativeSpeed()` instead.
{% endhint %}

{% hint style="info" %}
There is no direct accessor for the underlying `wpi::math::SwerveDrivePoseEstimator`. `SwerveDrive` owns it exclusively to avoid callers accidentally calling `Update(...)`/`ResetPosition(...)`/`ResetPose(...)` on it directly (which would corrupt the pose estimate by feeding it duplicate or out-of-order samples, or desyncing it from the drive's gyro offset). Use `GetPose()`, `ResetOdometry(wpi::math::Pose2d)`, and `AddVisionMeasurement(...)` instead; they cover the estimator interactions `SwerveDrive` supports.
{% endhint %}

{% hint style="info" %}
`GetSimPose()` returns a separate, ground-truth `wpi::math::Pose2d` that assumes every module reached its last-commanded `wpi::math::SwerveModuleVelocity` perfectly; it is **not** the same as `GetPose()` (the noisy, gyro/odometry-fused estimate). It's updated in simulation by `SimIterate()`, by integrating a `wpi::math::Twist2d` built from the desired module states, and is also snapped to the given pose by `ResetOdometry(wpi::math::Pose2d)` so it stays in sync with the fused estimate. On real hardware it stays at the configured starting pose. This makes it a convenient "known truth" pose to feed into a simulated vision system (e.g. to generate synthetic AprilTag detections) so you can test vision code end-to-end without a physical camera.
{% endhint %}

***

## Auto-Align (Drive to Pose) & PID Control

| Method                | Signature                    | Description                                  |
| --------------------- | ---------------------------- | -------------------------------------------- |
| `DriveToPoseSetpoint` | `wpi::math::ChassisVelocities DriveToPoseSetpoint(wpi::math::Pose2d targetPose)` | Computes one loop's robot-relative `ChassisVelocities` toward `targetPose` using the configured PID controllers. Lower-level building block behind `DriveToPose(...)`; call `ResetTranslationPID()`/`ResetRotationPID()` yourself before looping on this. |
| `SetTranslationPID`   | `void SetTranslationPID(wpi::math::PIDController controller)` | Replaces the translation controller used by `DriveToPose(...)`. Only resets the controller (clearing its integrator) if P, I, or D actually changed, so live-tuning doesn't wipe accumulated state every loop. |
| `SetRotationPID`      | `void SetRotationPID(wpi::math::PIDController controller)` | Replaces the rotation controller used by `DriveToPose(...)`, with the same change-detection guard as `SetTranslationPID`. |
| `ResetRotationPID`    | `void ResetRotationPID()`    | Resets the rotation PID controller state. |
| `ResetTranslationPID` | `void ResetTranslationPID()` | Resets the translation PID controller state. |

{% hint style="info" %}
When both a translation and a rotation controller are configured, `SwerveDrive` also publishes a live-tuning command through WPILib tunables at `Mechanisms/<name>/tuning/driveToPose` (see [Telemetry & DataLog](#telemetry--datalog) below). Running it resets both PID controllers, then every loop checks a tunable `AutoAlignEnabled` boolean and, while `true`, calls `DriveToPoseSetpoint(...)` against a tunable target pose (`autoalign/setpoint/x`/`autoalign/setpoint/y`/`autoalign/setpoint/rot`, published under `Tuning/<name>/` in NetworkTables), useful for tuning `WithTranslationController`/`WithRotationController` gains (or driving to an arbitrary pose) without redeploying code. The same command also drives the shared module drive/azimuth PID tuning fields (`modules/drive/*`, `modules/azimuth/*`); see [Telemetry & DataLog](#telemetry--datalog) below.
{% endhint %}

***

## Lifecycle

| Method            | Signature                                                 | Description                                                 |
| ----------------- | --------------------------------------------------------- | ----------------------------------------------------------- |
| `UpdateTelemetry` | `void UpdateTelemetry()`                                  | Publishes pose, module states, and speeds to NetworkTables. |
| `SimIterate`      | `void SimIterate()`                                       | Advances swerve simulation one loop iteration.              |
| `GetName`         | `std::string GetName() const`                             | Drivetrain name from config.                                |
| `GetConfig`       | `SwerveDriveConfig& GetConfig()`                          | Returns a reference to the config.                          |
| `GetKinematics`   | `wpi::math::SwerveDriveKinematics<NumModules>& GetKinematics()` | Returns the kinematics object.                              |

***

## Telemetry & DataLog

`UpdateTelemetry()` publishes pose, gyro, chassis speeds, and module states to NetworkTables under `Mechanisms/<name>` (tunable fields go under `Tuning/<name>`) at the verbosity configured via `SwerveDriveConfig::WithTelemetry(const std::string& name, TelemetryVerbosity)`. It also calls `SwerveModule::UpdateTelemetry()` for each module in the drive.

For full control over exactly which fields are published (including the auto-align PID gains `TranslationP/I/D`, `RotationP/I/D`, the tunable target pose fields `AutoAlignPoseX`/`AutoAlignPoseY`/`AutoAlignPoseRotation` (`autoalign/setpoint/x`, `autoalign/setpoint/y`, `autoalign/setpoint/rot`), the `AutoAlignEnabled` boolean, and the shared module tuning fields `ModulesDriveP/I/D`/`ModulesDriveKs/Kv/Ka`/`ModulesDriveVelocity`/`ModulesDriveTuningEnabled`/`ModulesDriveInPlace` (`modules/drive/feedback/p,i,d`, `modules/drive/feedforward/s,v,a`, `modules/drive/velocity`, `modules/drive/enabled`, `modules/drive/inplace`) and `ModulesAzimuthP/I/D`/`ModulesAzimuthKs/Kv/Ka`/`ModulesAzimuthAngle`/`ModulesAzimuthTuningEnabled` (`modules/azimuth/feedback/p,i,d`, `modules/azimuth/feedforward/s,v,a`, `modules/azimuth/angle`, `modules/azimuth/enabled`), all used by the live-tuning dashboard command at `Mechanisms/<name>/tuning/driveToPose`), pass a `telemetry::SwerveDriveTelemetryConfig` to `SwerveDriveConfig::WithTelemetry(name, SwerveDriveTelemetryConfig)`. The module tuning fields apply to every module on the drive at once, not to a single module, and the feedback/feedforward gains are pre-seeded from the first module's configured PID controller and `wpi::math::SimpleMotorFeedforward`, not `0`. To additionally record fields to a WPILib DataLog (for offline review in AdvantageScope), call `.WithDataLogName(const std::string&)` on that `SwerveDriveTelemetryConfig`; see [SwerveDriveConfig](swerve-drive-config.md#datalog-telemetry).

{% hint style="warning" %}
`AutoAlignEnabled`, `ModulesDriveTuningEnabled`, and `ModulesAzimuthTuningEnabled` are mutually exclusive. `ApplyTuningValues(...)` only acts on one at a time (priority: auto-align, then drive tuning, then azimuth tuning) and forces the others' NetworkTables value back to `false` if more than one is `true`. Both module tuning modes build a `wpi::util::array<wpi::math::SwerveModuleVelocity, NumModules>` (one entry per module) and command it in one call via `SwerveDrive::SetSwerveModuleStates(...)` rather than the drive/azimuth `SmartMotorController`s directly: while `ModulesDriveInPlace` is `false` (the default), drive tuning commands each module to an angle of `0°` while driving the tuned velocity; while `ModulesDriveInPlace` is `true`, it instead orients each module tangent to its position around the robot center (that module's location angle plus `90°`), so a positive `ModulesDriveVelocity` spins the whole robot counter-clockwise, matching WPILib's positive rotation direction. Azimuth tuning always commands `0` velocity alongside the tuned angle. When drive tuning is off (and auto-align isn't active), every module's drive motor is continuously commanded to `0` velocity.
{% endhint %}

***

## Example

```cpp
yams::mechanisms::swerve::SwerveDriveConfig config;
config.WithSubsystem(this);
// ... configure modules, gyro, speeds ...

yams::mechanisms::swerve::SwerveDrive<4> m_drive{&config};

// In a command factory:
wpi::cmd::CommandPtr DriveCommand(std::function<wpi::math::ChassisVelocities()> speeds) {
    return m_drive.Drive(speeds);
}
```

{% hint style="info" %}
`SwerveDrive<4>` is the most common instantiation. The template parameter must match the number of modules in `SwerveDriveConfig`. Deviating from 4 is rare in FRC.
{% endhint %}

{% @github-files/github-code-block url="https://github.com/Yet-Another-Software-Suite/YAMS/blob/master/examples/cpptest/src/main/cpp/subsystems/SwerveSubsystem.cpp#L139-L159" %}

***

## See Also

* [Java SwerveDrive](../../java-reference/swerve/swerve-drive.md)
* [SwerveDriveConfig](swerve-drive-config.md)
* [SwerveModule](swerve-module.md)
* [SwerveInputStream](swerve-input-stream.md)
