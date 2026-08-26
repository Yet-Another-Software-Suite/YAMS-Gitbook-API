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
`getDesiredChassisSpeeds()` returns a **setpoint**, not the robot's actual measured motion — it is whatever was last passed to `setRobotRelativeChassisSpeeds(...)` (directly, or via `setFieldRelativeChassisSpeeds(...)`/`drive(...)`), cached and republished every `updateTelemetry()` call. If nothing has commanded the drive yet, it returns a zeroed `ChassisSpeeds`. For the drive's actual measured speed, use `getRobotRelativeSpeed()` / `getFieldRelativeSpeed()` instead.
{% endhint %}

{% hint style="info" %}
There is no direct accessor for the underlying `SwerveDrivePoseEstimator` — `SwerveDrive` owns it exclusively to avoid callers accidentally calling `update(...)`/`resetPosition(...)`/`resetPose(...)` on it directly (which would corrupt the pose estimate by feeding it duplicate or out-of-order samples, or desyncing it from the drive's gyro offset). Use `getPose()`, `resetOdometry(Pose2d)`, and `addVisionMeasurement(...)` instead — they cover the estimator interactions `SwerveDrive` supports.
{% endhint %}

{% hint style="info" %}
`getSimPose()` returns a separate, ground-truth `Pose2d` that assumes every module reached its last-commanded `SwerveModuleState` perfectly — it is **not** the same as `getPose()` (the noisy, gyro/odometry-fused estimate). It's updated in simulation by `simIterate()`, by integrating a `Twist2d` built from the desired module states, and is also snapped to the given pose by `resetOdometry(Pose2d)` so it stays in sync with the fused estimate. On a real robot it stays at the configured starting pose. This makes it a convenient "known truth" pose to feed into a simulated vision system (e.g. to generate synthetic AprilTag detections) so you can test vision code end-to-end without a physical camera.
{% endhint %}

***

## Lifecycle

| Method              | Returns | Description                                                                                     |
| ------------------- | ------- | ----------------------------------------------------------------------------------------------- |
| `updateTelemetry()` | `void`  | Publishes pose, module states, and speeds to NetworkTables. Call from `periodic()`.             |
| `simIterate()`      | `void`  | Advances the swerve physics simulation by one loop iteration. Call from `simulationPeriodic()`. |

***

## Telemetry & DataLog

`updateTelemetry()` publishes pose, gyro, chassis speeds, and module states to NetworkTables under `Mechanisms/<name>` at the verbosity configured via `SwerveDriveConfig.withTelemetry(TelemetryVerbosity)`. To additionally record those fields to a WPILib DataLog (for offline review in AdvantageScope), set `SwerveDriveConfig.withDataLogName(String)` — see [SwerveDriveConfig](swerve-drive-config.md#datalog-telemetry).

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

{% @github-files/github-code-block url="https://github.com/Yet-Another-Software-Suite/YAMS/blob/master/examples/swerve_drive_pathplanner/java/frc/robot/subsystems/SwerveSubsystem.java#L157-L190" %}

***

## See Also

* [SwerveDriveConfig](swerve-drive-config.md)
* [SwerveModule](swerve-module.md)
* [SwerveInputStream](swerve-input-stream.md)
* [C++ SwerveDrive](../../c++-reference/swerve/swerve-drive.md)
