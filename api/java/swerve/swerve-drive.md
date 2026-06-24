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

***

## Lifecycle

| Method              | Returns | Description                                                                                     |
| ------------------- | ------- | ----------------------------------------------------------------------------------------------- |
| `updateTelemetry()` | `void`  | Publishes pose, module states, and speeds to NetworkTables. Call from `periodic()`.             |
| `simIterate()`      | `void`  | Advances the swerve physics simulation by one loop iteration. Call from `simulationPeriodic()`. |

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

{% embed url="https://github.com/Yet-Another-Software-Suite/YAMS/tree/master/examples/swerve_drive" %}
Swerve Drive example
{% endembed %}

{% embed url="https://github.com/Yet-Another-Software-Suite/YAMS/tree/master/examples/swerve_drive_pathplanner" %}
Swerve Drive with PathPlanner autonomous
{% endembed %}

***

## See Also

* [SwerveDriveConfig](swerve-drive-config.md)
* [SwerveModule](swerve-module.md)
* [SwerveInputStream](swerve-input-stream.md)
* [C++ SwerveDrive](../../cpp/swerve/swerve-drive.md)
