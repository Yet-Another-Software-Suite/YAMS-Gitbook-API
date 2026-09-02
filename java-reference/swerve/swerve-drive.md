# SwerveDrive

**Package:** `yams.mechanisms.swerve`

Manages a complete swerve drivetrain: kinematics, module states, odometry via `SwerveDrivePoseEstimator`, and optional vision measurement fusion.

## Constructor

```java
SwerveDrive(SwerveDriveConfig config)
```

| Parameter | Type                | Description                                                            |
| --------- | ------------------- | ---------------------------------------------------------------------- |
| `config`  | `SwerveDriveConfig` | Full drivetrain configuration including modules, kinematics, and gyro. |

***

## Driving

| Method                                                | Returns   | Description                                                                         |
| ----------------------------------------------------- | --------- | ----------------------------------------------------------------------------------- |
| `drive(Supplier<ChassisSpeeds> speedsSupplier)`       | `Command` | Continuously sets robot-relative chassis speeds from a supplier. Runs indefinitely. |
| `setRobotRelativeChassisSpeeds(ChassisSpeeds speeds)` | `void`    | Directly sets robot-relative speeds. Call from `periodic()`, not a command.         |
| `setFieldRelativeChassisSpeeds(ChassisSpeeds speeds)` | `void`    | Sets field-relative speeds, rotating by the gyro angle before applying.             |

***

## Auto-Align (Drive to Pose)

| Method                                     | Returns         | Description                                                                                                 |
| ------------------------------------------- | --------------- | ------------------------------------------------------------------------------------------------------------ |
| `driveToPose(Pose2d pose)`                  | `Command`       | Resets the translation/rotation PID controllers, then continuously drives toward the field-relative `pose`.  |
| `driveToPoseSetpoint(Pose2d targetPose)`    | `ChassisSpeeds` | Computes one loop's robot-relative `ChassisSpeeds` toward `targetPose` using the configured PID controllers. Lower-level building block behind `driveToPose(...)`; reset the PID controllers yourself before looping on this. |
| `setTranslationPID(PIDController controller)` | `void`        | Replaces the translation controller used by `driveToPose(...)`. Only resets the controller (clearing its integrator) if P, I, or D actually changed, so live-tuning doesn't wipe accumulated state every loop. |
| `setRotationPID(PIDController controller)`  | `void`          | Replaces the rotation controller used by `driveToPose(...)`, with the same change-detection guard as `setTranslationPID`. |
| `resetTranslationPID()`                     | `void`          | Resets the translation controller's internal state (e.g. integrator).                                        |
| `resetRotationPID()`                        | `void`          | Resets the rotation controller's internal state.                                                              |

{% hint style="info" %}
`SwerveDrive` also publishes a live-tuning command to SmartDashboard at `Mechanisms/<name>/tuning/driveToPose` (see [Telemetry & DataLog](#telemetry--datalog) below). Running it resets both PID controllers, then every loop checks a tunable `AutoAlignEnabled` boolean and, while `true`, calls `driveToPoseSetpoint(...)` against a tunable target pose (`autoalign/setpoint/x`/`autoalign/setpoint/y`/`autoalign/setpoint/rot`) published to NetworkTables, useful for tuning `withTranslationController`/`withRotationController` gains — or driving to an arbitrary pose — without redeploying code. The `TranslationP/I/D`/`RotationP/I/D` tuning entries are pre-seeded from the PID controllers passed to `withTranslationController`/`withRotationController` (not `0`), the same way `SmartMotorControllerTelemetry` seeds its `kP`/`kI`/`kD` entries from the motor's configured PID.
{% endhint %}

***

## Odometry & Pose

| Method                                                | Returns                  | Description                                                                    |
| ----------------------------------------------------- | ------------------------ | ------------------------------------------------------------------------------ |
| `getPose()`                                           | `Pose2d`                 | Returns the current estimated robot pose.                                      |
| `resetOdometry(Pose2d pose)`                          | `void`                   | Resets the pose estimator to `pose`. Also reseeds all module azimuth encoders. |
| `addVisionMeasurement(Pose2d pose, double timestamp)` | `void`                   | Fuses a vision-derived pose into the estimator at `timestamp` (FPGA seconds).  |
| `getChassisSpeeds()`                                  | `ChassisSpeeds`          | Returns the current robot-relative chassis speeds derived from module states.  |
| `getModuleStates()`                                   | `SwerveModuleState[]`    | Returns the current state (speed and angle) of each module.                    |
| `getModulePositions()`                                | `SwerveModulePosition[]` | Returns the integrated position of each module.                                |
| `getDesiredChassisSpeeds()`                           | `ChassisSpeeds`          | Returns the last-commanded robot-relative setpoint (not a measurement).        |
| `getDesiredModuleStates()`                            | `SwerveModuleState[]`    | Returns the last-commanded module states (not a measurement).                  |
| `getRobotRelativeChassisSpeedsFromState(SwerveModuleState[])` | `ChassisSpeeds`   | Converts an arbitrary set of module states back into robot-relative chassis speeds. |
| `getSimPose()`                                        | `Pose2d`                 | Returns the simulated ground-truth pose. See below.                            |
| `getField2d()`                                        | `Field2d`                | Returns the `Field2d` widget the drive already publishes to SmartDashboard.    |

{% hint style="warning" %}
`getDesiredChassisSpeeds()` returns a **setpoint**, not the robot's actual measured motion: it is whatever was last passed to `setRobotRelativeChassisSpeeds(...)` (directly, or via `setFieldRelativeChassisSpeeds(...)`/`drive(...)`), cached and republished every `updateTelemetry()` call. If nothing has commanded the drive yet, it returns a zeroed `ChassisSpeeds`. For the drive's actual measured speed, use `getRobotRelativeSpeed()` / `getFieldRelativeSpeed()` instead.
{% endhint %}

{% hint style="info" %}
There is no direct accessor for the underlying `SwerveDrivePoseEstimator`. `SwerveDrive` owns it exclusively to avoid callers accidentally calling `update(...)`/`resetPosition(...)`/`resetPose(...)` on it directly (which would corrupt the pose estimate by feeding it duplicate or out-of-order samples, or desyncing it from the drive's gyro offset). Use `getPose()`, `resetOdometry(Pose2d)`, and `addVisionMeasurement(...)` instead; they cover the estimator interactions `SwerveDrive` supports.
{% endhint %}

{% hint style="info" %}
`getSimPose()` returns a separate, ground-truth `Pose2d` that assumes every module reached its last-commanded `SwerveModuleState` perfectly; it is **not** the same as `getPose()` (the noisy, gyro/odometry-fused estimate). It's updated in simulation by `simIterate()`, by integrating a `Twist2d` built from the desired module states, and is also snapped to the given pose by `resetOdometry(Pose2d)` so it stays in sync with the fused estimate. On a real robot it stays at the configured starting pose. This makes it a convenient "known truth" pose to feed into a simulated vision system (e.g. to generate synthetic AprilTag detections) so you can test vision code end-to-end without a physical camera.
{% endhint %}

***

## Lifecycle

| Method              | Returns | Description                                                                                     |
| ------------------- | ------- | ----------------------------------------------------------------------------------------------- |
| `updateTelemetry()` | `void`  | Publishes pose, module states, and speeds to NetworkTables. Call from `periodic()`.             |
| `simIterate()`      | `void`  | Advances the swerve physics simulation by one loop iteration. Call from `simulationPeriodic()`. |

***

## Telemetry & DataLog

`updateTelemetry()` publishes pose, gyro, chassis speeds, and module states to NetworkTables under `Mechanisms/<name>` at the verbosity configured via `SwerveDriveConfig.withTelemetry(String name, TelemetryVerbosity)`. It also calls `SwerveModule.updateTelemetry()` for each module in the drive.

For full control over exactly which fields are published (including the auto-align PID gains `TranslationP/I/D`, `RotationP/I/D`, the tunable target pose fields `AutoAlignPoseX`/`AutoAlignPoseY`/`AutoAlignPoseRotation` (`autoalign/setpoint/x`, `autoalign/setpoint/y`, `autoalign/setpoint/rot`), the `AutoAlignEnabled` boolean, and the shared module tuning fields `ModulesDriveP/I/D`/`ModulesDriveKs/Kv/Ka`/`ModulesDriveVelocity`/`ModulesDriveTuningEnabled` (`modules/drive/feedback/p,i,d`, `modules/drive/feedforward/s,v,a`, `modules/drive/velocity`, `modules/drive/enabled`) and `ModulesAzimuthP/I/D`/`ModulesAzimuthKs/Kv/Ka`/`ModulesAzimuthAngle`/`ModulesAzimuthTuningEnabled` (`modules/azimuth/feedback/p,i,d`, `modules/azimuth/feedforward/s,v,a`, `modules/azimuth/angle`, `modules/azimuth/enabled`), all used by the live-tuning dashboard command), pass a [`SwerveDriveTelemetryConfig`](../../java-reference/swerve/swerve-drive.md) to `SwerveDriveConfig.withTelemetry(String name, SwerveDriveTelemetryConfig)`. The module tuning fields apply to every module on the drive at once, not to a single module, and the feedback/feedforward gains are pre-seeded from the first module's configured PID controller and `SimpleMotorFeedforward`, not `0`. To additionally record fields to a WPILib DataLog (for offline review in AdvantageScope), call `.withDataLogName(String)` on that `SwerveDriveTelemetryConfig`; see [SwerveDriveConfig](swerve-drive-config.md#datalog-telemetry).

{% hint style="warning" %}
`AutoAlignEnabled`, `ModulesDriveTuningEnabled`, and `ModulesAzimuthTuningEnabled` are mutually exclusive. `applyTuningValues(...)` only acts on one at a time (priority: auto-align, then drive tuning, then azimuth tuning) and forces the others' NetworkTables value back to `false` if more than one is `true`. Both module tuning modes build a `SwerveModuleState[]` (one entry per module) and command it in one call via `SwerveDrive.setSwerveModuleStates(...)` rather than the drive/azimuth `SmartMotorController`s directly: drive tuning holds each module's current angle while driving the tuned velocity, and azimuth tuning always commands `0` velocity alongside the tuned angle. When drive tuning is off (and auto-align isn't active), every module's drive motor is continuously commanded to `0` velocity.
{% endhint %}

***

## Example Usage

```java
public class SwerveSubsystem extends SubsystemBase {
    private final SwerveDrive m_drive;

    public SwerveSubsystem() {
        m_drive = new SwerveDrive(/* SwerveDriveConfig */);
    }

    public Command driveCommand(Supplier<ChassisSpeeds> speeds) {
        return m_drive.drive(speeds);
    }

    @Override
    public void periodic() {
        m_drive.updateTelemetry();
    }
}
```

{% @github-files/github-code-block url="https://github.com/Yet-Another-Software-Suite/YAMS/blob/master/examples/swerve_drive/java/frc/robot/subsystems/SwerveSubsystem.java#L176-L215" %}

{% @github-files/github-code-block url="https://github.com/Yet-Another-Software-Suite/YAMS/blob/master/examples/swerve_drive_pathplanner/java/frc/robot/subsystems/SwerveSubsystem.java#L157-L19X" %}

***

## See Also

* [SwerveDriveConfig](swerve-drive-config.md)
* [SwerveModule](swerve-module.md)
* [SwerveInputStream](swerve-input-stream.md)
* [C++ SwerveDrive](../../c++-reference/swerve/swerve-drive.md)
