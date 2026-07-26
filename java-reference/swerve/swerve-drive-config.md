# SwerveDriveConfig

**Package:** `yams.mechanisms.config`

Configures a complete swerve drivetrain. Takes a `Subsystem` and a varargs array of `SwerveModuleConfig` objects — one per module.

## Constructor

```java
SwerveDriveConfig(Subsystem subsystem, SwerveModuleConfig... modules)
```

| Parameter | Type | Description |
|-----------|------|-------------|
| `subsystem` | `Subsystem` | The subsystem that owns this drivetrain. Used for command requirements. |
| `modules` | `SwerveModuleConfig...` | One config per swerve module, in the order: front-left, front-right, back-left, back-right (by convention). |

All builder methods return `SwerveDriveConfig` for chaining.

---

## Builder Methods

| Method | Parameters | Description |
|--------|-----------|-------------|
| `withTrackWidth(Distance width)` | `width` — lateral wheel-to-wheel distance | Distance between left and right wheels. Used to compute kinematics. |
| `withWheelBase(Distance base)` | `base` — longitudinal wheel-to-wheel distance | Distance between front and rear wheels. |
| `withMaximumChassisSpeed(LinearVelocity speed, AngularVelocity rotSpeed)` | `speed`, `rotSpeed` | Maximum translational and rotational speeds used for input normalization. |
| `withGyro(Supplier<Angle> gyroAngle)` | `gyroAngle` — yaw angle supplier | **Required.** Provides robot heading for field-relative control and odometry. |
| `withTranslationController(PIDController controller)` | `controller` | PID controller for autonomous translation corrections. |
| `withRotationController(PIDController controller)` | `controller` | PID controller for autonomous rotation corrections. |
| `withStartingPose(Pose2d pose)` | `pose` | Initial odometry pose. Defaults to the origin. |
| `withTelemetry(boolean publish)` | `publish` | Enables or disables swerve telemetry publishing to NetworkTables. |

{% hint style="info" %}
`withGyro` is required. Omitting it will prevent field-relative control and odometry from functioning correctly.
{% endhint %}

---

## Example

```java
SwerveDriveConfig config = new SwerveDriveConfig(this, frontLeft, frontRight, backLeft, backRight)
    .withTrackWidth(Inches.of(23.5))
    .withWheelBase(Inches.of(23.5))
    .withMaximumChassisSpeed(MetersPerSecond.of(4.5), RadiansPerSecond.of(Math.PI * 2))
    .withGyro(() -> Degrees.of(m_gyro.getYaw().getValueAsDouble()))
    .withTelemetry(true);
```

---

## See Also

- [SwerveDrive](swerve-drive.md)
- [SwerveModuleConfig](swerve-module-config.md)
