# SwerveModuleConfig

**Package:** `yams.mechanisms.config`\
**C++ equivalent:** [C++ SwerveModuleConfig](../../c++-reference/swerve/swerve-module-config.md)

Configures one swerve module. Takes drive and azimuth motor controllers in the constructor.

## Constructor

```java
SwerveModuleConfig(SmartMotorController driveMotor, SmartMotorController azimuthMotor)
```

| Parameter | Type | Description |
|-----------|------|-------------|
| `driveMotor` | `SmartMotorController` | Motor controller for the drive wheel. |
| `azimuthMotor` | `SmartMotorController` | Motor controller for the steering mechanism. |

Use the no-arg constructor plus `withSmartMotorController(SmartMotorController, SmartMotorController)` when the motors aren't available yet at construction time.

All builder methods return `SwerveModuleConfig` for chaining.

---

## Builder Methods

| Method | Parameters | Description |
|--------|-----------|-------------|
| `withSmartMotorController(SmartMotorController driveMotor, SmartMotorController azimuthMotor)` | `driveMotor`, `azimuthMotor` | Sets both motor controllers (use with the no-arg constructor). |
| `withCosineCompensation(boolean compensate)` | `compensate` | Scales drive speed by `cos(angle_error)` to reduce skew while steering. |
| `withAbsoluteEncoder(Object absoluteEncoder)` | `absoluteEncoder` | Same-vendor absolute encoder object for the azimuth `SmartMotorController`. |
| `withAbsoluteEncoder(Supplier<Angle> locationProvider)` | `locationProvider` | Cross-vendor absolute encoder angle supplier. |
| `withAbsoluteEncoder(DoubleSupplier rotationSupplier)` | `rotationSupplier` | Cross-vendor absolute encoder angle supplier, in rotations. |
| `withAbsoluteEncoderGearing(GearBox gearing)` | `gearing` | Gearing between the absolute encoder shaft and the azimuth mechanism, if not 1:1. |
| `withAbsoluteEncoderOffset(Angle offset)` | `offset` | Absolute encoder reading when the module is at its forward (zero) position, bevel left. |
| `withLocation(Translation2d location)` | `location` — module position on chassis | Module position relative to robot center, in meters. Used in kinematics calculations. |
| `withLocation(Distance front, Distance left)` | `front`, `left` | Module position via separate forward/left distances. |
| `withLocation(Distance distance, Angle angle)` | `distance`, `angle` | Module position via polar coordinates from the center of rotation. |
| `withDistanceFromCenterOfRotation(Distance front, Distance left)` | `front`, `left` | Equivalent to `withLocation(Distance, Distance)`. |
| `withWheelRadius(Distance radius)` | `radius` — wheel radius | Drive wheel radius used for velocity and position conversion. |
| `withWheelDiameter(Distance diameter)` | `diameter` | Drive wheel diameter used for velocity and position conversion. |
| `withMinimumVelocity(LinearVelocity speed)` | `speed` | Below this speed, the module holds its current azimuth instead of tracking. |
| `withOptimization(boolean swerveModuleStateOptimization)` | `swerveModuleStateOptimization` | Enables/disables `SwerveModuleState.optimize()` (flip by 180° to minimize rotation). |
| `withTelemetry(String telemetryName, TelemetryVerbosity telemetryVerbosity)` | `telemetryName`, `telemetryVerbosity` | NetworkTables key prefix and verbosity for this module's own fields (absolute encoder angle). |
| `withDataLogName(String dataLogName)` | `dataLogName` | Logs this module's telemetry (currently just the absolute encoder field) to a WPILib DataLog under this name, in addition to NetworkTables. See below. |

{% hint style="info" %}
`withAbsoluteEncoderOffset` must be calibrated per module. An incorrect offset causes the module to drive at an angle on startup until `seedAzimuthEncoder()` corrects it.
{% endhint %}

---

## DataLog Telemetry

`withDataLogName(name)` sends this module's absolute-encoder field to a WPILib `DataLog` under the given prefix, in addition to NetworkTables:

```java
SwerveModuleConfig frontLeftConfig = new SwerveModuleConfig(driveMotor, azimuthMotor)
    .withWheelRadius(Inches.of(2))
    .withLocation(new Translation2d(0.298, 0.298))
    .withAbsoluteEncoderOffset(Degrees.of(45.2))
    .withTelemetry("FrontLeft", TelemetryVerbosity.HIGH)
    .withDataLogName("swerve/frontLeft");  // logs this module's absolute encoder angle only
```

{% hint style="warning" %}
`withDataLogName` on `SwerveModuleConfig` is independent of `SwerveDriveConfig.withDataLogName` (neither cascades to the other) and does **not** cascade to this module's drive/azimuth motors. To get field-level DataLog control (or a DataLog name) on the drive or azimuth motor itself, configure that motor's own `SmartMotorControllerConfig.withTelemetry(name, SmartMotorControllerTelemetryConfig)` before constructing the `SwerveModule` — see [SmartMotorControllerTelemetryConfig](../motor-controllers/smart-motor-controller-telemetry-config.md).
{% endhint %}

---

## Example

```java
SwerveModuleConfig frontLeft = new SwerveModuleConfig(driveMotor, azimuthMotor)
    .withWheelRadius(Inches.of(2))
    .withLocation(new Translation2d(0.298, 0.298))
    .withAbsoluteEncoderOffset(Degrees.of(45.2))
    .withTelemetry("FrontLeft", TelemetryVerbosity.HIGH)
    .withDataLogName("swerve/frontLeft");
```

---

## See Also

- [C++ SwerveModuleConfig](../../c++-reference/swerve/swerve-module-config.md)
- [SwerveModule](swerve-module.md)
- [SwerveDriveConfig](swerve-drive-config.md)
