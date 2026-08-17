# SwerveDriveConfig

**Package:** `yams.mechanisms.config`\
**C++ equivalent:** [C++ SwerveDriveConfig](../../c++-reference/swerve/swerve-drive-config.md)

Configures a complete swerve drivetrain. Takes a `Subsystem` and a varargs array of `SwerveModule` objects — one per module.

## Constructor

```java
SwerveDriveConfig(Subsystem swerveSubsystem, SwerveModule... modules)
```

| Parameter | Type | Description |
|-----------|------|-------------|
| `swerveSubsystem` | `Subsystem` | The subsystem that owns this drivetrain. Used for command requirements. |
| `modules` | `SwerveModule...` | One module per corner, in the order: front-left, front-right, back-left, back-right (by convention). |

Use the no-arg constructor plus `withSubsystem(Subsystem)` and `withModules(SwerveModule...)` when the modules aren't available yet at construction time.

All builder methods return `SwerveDriveConfig` for chaining.

---

## Builder Methods

| Method | Parameters | Description |
|--------|-----------|-------------|
| `withSubsystem(Subsystem subsystem)` | `subsystem` | Sets the owning subsystem (use with the no-arg constructor). |
| `withModules(SwerveModule... modules)` | `modules` | Sets the swerve modules for the drive. |
| `withGyro(Supplier<Angle> gyro)` | `gyro` — yaw angle supplier | **Required.** Provides robot heading for field-relative control and odometry. |
| `withGyroOffset(Angle offset)` | `offset` | Offset subtracted from the raw gyro reading in `getGyroAngle()`. |
| `withGyroInverted(boolean inverted)` | `inverted` | Negates the raw gyro angle when `true`. |
| `withGyroVelocity(Supplier<AngularVelocity> supplier)` | `supplier` | Supplies gyro angular velocity. Required for angular-velocity skew correction. |
| `withStartingPose(Pose2d pose)` | `pose` | Initial odometry pose. Defaults to the origin. |
| `withMaximumChassisSpeed(LinearVelocity speed, AngularVelocity rotSpeed)` | `speed`, `rotSpeed` | Maximum translational and rotational speeds used for wheel-speed desaturation. |
| `withMaximumModuleSpeed(LinearVelocity speed)` | `speed` | Maximum module speed used for wheel-speed desaturation. |
| `withCenterOfRotation(Translation2d centerOfRotation)` | `centerOfRotation` | Overrides the chassis center of rotation (default is the robot center). |
| `withCenterOfRotation(Distance forward, Distance left)` | `forward`, `left` | Overrides the center of rotation via forward/left distances. |
| `withDiscretizationTime(Time dt)` | `dt` | Discretization timestep used to compensate for `ChassisSpeeds` skew. |
| `withSimDiscretizationTime(Time dt)` | `dt` | Discretization timestep used only in simulation. Falls back to `withDiscretizationTime` if unset. |
| `withGyroAngularVelocityScaleFactor(double scaleFactor)` | `scaleFactor` | Scale factor `[0, 1]` applied to gyro angular velocity for skew correction on the real robot. |
| `withSimGyroAngularVelocityScaleFactor(double scaleFactor)` | `scaleFactor` | Scale factor used only in simulation. Falls back to `withGyroAngularVelocityScaleFactor` if unset. |
| `withTranslationController(PIDController controller)` | `controller` | PID controller for autonomous translation corrections (`DriveToPose`). |
| `withRotationController(PIDController controller)` | `controller` | PID controller for autonomous rotation corrections (`DriveToPose`). |
| `withSimTranslationController(PIDController controller)` | `controller` | Translation controller used only in simulation. Falls back to `withTranslationController` if unset. |
| `withSimRotationController(PIDController controller)` | `controller` | Rotation controller used only in simulation. Falls back to `withRotationController` if unset. |
| `withTelemetry(TelemetryVerbosity verbosity)` | `verbosity` | Sets the telemetry verbosity for drive-level fields (pose, gyro, chassis speeds, module states). |
| `withDataLogName(String dataLogName)` | `dataLogName` | Logs the drive's telemetry to a WPILib DataLog under this name prefix, in addition to NetworkTables. See below. |

{% hint style="info" %}
`withGyro` is required. Omitting it will prevent field-relative control and odometry from functioning correctly.
{% endhint %}

---

## DataLog Telemetry

`withDataLogName(name)` logs this drive's pose, gyro angle, and chassis speeds/module states (whatever `withTelemetry`'s verbosity publishes) to a WPILib `DataLog`, in addition to NetworkTables — it does not replace NT4 publishing, and there is no swerve-level equivalent of `SmartMotorControllerTelemetryConfig.withoutNetworkTables()`.

```java
SwerveDriveConfig config = new SwerveDriveConfig(this, frontLeft, frontRight, backLeft, backRight)
    .withGyro(() -> Degrees.of(m_gyro.getYaw().getValueAsDouble()))
    .withTelemetry(TelemetryVerbosity.HIGH)
    .withDataLogName("swerve");  // logs pose, gyro, chassis speeds, module states
```

{% hint style="warning" %}
`withDataLogName` on `SwerveDriveConfig` does **not** cascade to the modules passed to the constructor, or to those modules' drive/azimuth motors. Each `SwerveModuleConfig` needs its own `withDataLogName(...)` (see [SwerveModuleConfig](swerve-module-config.md#datalog-telemetry)), and each motor's `SmartMotorControllerConfig` needs its own `withTelemetry(name, SmartMotorControllerTelemetryConfig)` for field-level control and its own DataLog name.
{% endhint %}

---

## Example

```java
SwerveDriveConfig config = new SwerveDriveConfig(this, frontLeft, frontRight, backLeft, backRight)
    .withGyro(() -> Degrees.of(m_gyro.getYaw().getValueAsDouble()))
    .withMaximumChassisSpeed(MetersPerSecond.of(4.5), RadiansPerSecond.of(Math.PI * 2))
    .withTranslationController(new PIDController(1, 0, 0))
    .withRotationController(new PIDController(1, 0, 0))
    .withTelemetry(TelemetryVerbosity.HIGH)
    .withDataLogName("swerve");
```

---

## See Also

- [C++ SwerveDriveConfig](../../c++-reference/swerve/swerve-drive-config.md)
- [SwerveDrive](swerve-drive.md)
- [SwerveModuleConfig](swerve-module-config.md)
