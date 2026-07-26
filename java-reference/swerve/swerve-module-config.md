# SwerveModuleConfig

**Package:** `yams.mechanisms.config`

Configures one swerve module. Takes drive and azimuth motor controllers in the constructor.

## Constructor

```java
SwerveModuleConfig(SmartMotorController driveMotor, SmartMotorController azimuthMotor)
```

| Parameter | Type | Description |
|-----------|------|-------------|
| `driveMotor` | `SmartMotorController` | Motor controller for the drive wheel. |
| `azimuthMotor` | `SmartMotorController` | Motor controller for the steering mechanism. Must have an absolute encoder configured. |

All builder methods return `SwerveModuleConfig` for chaining.

---

## Builder Methods

| Method | Parameters | Description |
|--------|-----------|-------------|
| `withWheelRadius(Distance radius)` | `radius` — wheel radius | Drive wheel radius used for velocity and position conversion. |
| `withLocation(Translation2d location)` | `location` — module position on chassis | Module position relative to robot center, in meters. Used in kinematics calculations. |
| `withAbsoluteEncoderOffset(Angle offset)` | `offset` | Absolute encoder reading when the module is at its forward (zero) position. |
| `withTelemetryName(String name)` | `name` | NetworkTables key prefix for this module's telemetry entries. |
| `withSimColor(Color8Bit color)` | `color` | Module color used in simulation visualization. |

{% hint style="info" %}
`withAbsoluteEncoderOffset` must be calibrated per module. An incorrect offset causes the module to drive at an angle on startup until `seedAzimuthEncoder()` corrects it.
{% endhint %}

---

## Example

```java
SwerveModuleConfig frontLeft = new SwerveModuleConfig(driveMotor, azimuthMotor)
    .withWheelRadius(Inches.of(2))
    .withLocation(new Translation2d(0.298, 0.298))
    .withAbsoluteEncoderOffset(Degrees.of(45.2))
    .withTelemetryName("FrontLeft");
```

---

## See Also

- [SwerveModule](swerve-module.md)
- [SwerveDriveConfig](swerve-drive-config.md)
