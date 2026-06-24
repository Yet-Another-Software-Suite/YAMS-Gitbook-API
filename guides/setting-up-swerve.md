# Setting Up Swerve Drive

**Goal:** Create a fully functional `SwerveDrive` subsystem with field-relative driving, odometry, and simulation.

---

## Steps

### 1. Configure drive and azimuth motor controllers for each module

Each swerve module needs two `SmartMotorController` instances: one for drive (velocity) and one for azimuth (position). Follow [Configuring a Motor Controller](configuring-a-motor.md) for each.

Drive controllers use velocity PID and a `SimpleMotorFeedforward`. Azimuth controllers use position PID and optionally a `SimpleMotorFeedforward` for friction compensation.

### 2. Create a `SwerveModuleConfig` for each module

Provide the wheel radius, the module's location relative to the robot center, and the absolute encoder offset angle.

```java
SwerveModuleConfig frontLeftConfig = new SwerveModuleConfig()
    .withName("FrontLeft")
    .withWheelRadius(Meters.of(0.0508))
    .withModuleLocation(new Translation2d(0.381, 0.381))
    .withAbsoluteEncoderOffset(Rotations.of(0.24))
    .withDriveMotorConfig(driveMotorConfigFL)
    .withAzimuthMotorConfig(azimuthMotorConfigFL)
    .withDriveMotor(driveMotorFL)
    .withAzimuthMotor(azimuthMotorFL);
```

Repeat for `FrontRight`, `BackLeft`, and `BackRight`, adjusting the `Translation2d` signs and encoder offsets.

### 3. Create a `SwerveDriveConfig`

Collect the four module configs, set kinematic limits, and provide a gyro supplier.

```java
SwerveDriveConfig driveConfig = new SwerveDriveConfig()
    .withModules(frontLeftConfig, frontRightConfig, backLeftConfig, backRightConfig)
    .withMaxTranslationalSpeed(MetersPerSecond.of(4.5))
    .withMaxRotationalSpeed(RadiansPerSecond.of(Math.PI * 2))
    .withGyroSupplier(imu::getRotation2d)
    .withTelemetry(TelemetryVerbosity.HIGH);
```

### 4. Construct `SwerveDrive` and wire command factories

```java
public class DriveSubsystem extends SubsystemBase {

    private final SwerveDrive swerveDrive;

    public DriveSubsystem(Supplier<Rotation2d> gyroSupplier) {

        // --- Front Left ---
        SmartMotorControllerConfig driveConfigFL = new SmartMotorControllerConfig()
            .withIdleMode(MotorMode.Brake)
            .withGearing(new MechanismGearing(GearBox.fromRatio(6.75)))
            .withVelocityPID(0.1, 0.0, 0.0)
            .withFeedforward(new SimpleMotorFeedforward(0.12, 2.2, 0.3))
            .withStatorCurrentLimit(Amps.of(60));

        SmartMotorControllerConfig azimuthConfigFL = new SmartMotorControllerConfig()
            .withIdleMode(MotorMode.Brake)
            .withGearing(new MechanismGearing(GearBox.fromRatio(12.8)))
            .withClosedLoopController(7.0, 0.0, 0.3)
            .withExternalEncoderDiscontinuityPoint(Rotations.of(0.5))
            .withStatorCurrentLimit(Amps.of(40));

        SmartMotorController driveMotorFL = SmartMotorController.create(
            new TalonFX(10), DCMotor.getKrakenX60(1), driveConfigFL);
        SmartMotorController azimuthMotorFL = SmartMotorController.create(
            new TalonFX(11), DCMotor.getKrakenX60(1), azimuthConfigFL);

        // Repeat for FR, BL, BR with their CAN IDs and encoder offsets...

        SwerveModuleConfig frontLeftModuleConfig = new SwerveModuleConfig()
            .withName("FrontLeft")
            .withWheelRadius(Meters.of(0.0508))
            .withModuleLocation(new Translation2d(0.381, 0.381))
            .withAbsoluteEncoderOffset(Rotations.of(0.24))
            .withDriveMotor(driveMotorFL)
            .withAzimuthMotor(azimuthMotorFL);

        // SwerveModuleConfig frontRightModuleConfig = ...
        // SwerveModuleConfig backLeftModuleConfig  = ...
        // SwerveModuleConfig backRightModuleConfig = ...

        SwerveDriveConfig driveConfig = new SwerveDriveConfig()
            .withModules(frontLeftModuleConfig /*, frontRightModuleConfig, ... */)
            .withMaxTranslationalSpeed(MetersPerSecond.of(4.5))
            .withMaxRotationalSpeed(RadiansPerSecond.of(Math.PI * 2))
            .withGyroSupplier(gyroSupplier)
            .withTelemetry(TelemetryVerbosity.HIGH);

        swerveDrive = new SwerveDrive(driveConfig);
    }

    /** Field-relative drive command driven by a SwerveInputStream. */
    public Command driveFieldRelativeCommand(SwerveInputStream inputStream) {
        return swerveDrive.driveFieldRelative(inputStream);
    }

    /** Pass vision pose estimates to the odometry estimator. */
    public void addVisionMeasurement(Pose2d pose, double timestampSeconds) {
        swerveDrive.addVisionMeasurement(pose, timestampSeconds);
    }

    public Pose2d getEstimatedPose() {
        return swerveDrive.getEstimatedPose();
    }

    @Override
    public void periodic() {
        swerveDrive.updateTelemetry();
    }

    @Override
    public void simulationPeriodic() {
        swerveDrive.simIterate();
    }
}
```

### 5. Set up driver input with `SwerveInputStream`

`SwerveInputStream` translates raw joystick axes into swerve chassis speeds, applying deadbands and rate limits.

```java
SwerveInputStream driverInput = SwerveInputStream.of(
    () -> -driverController.getLeftY(),
    () -> -driverController.getLeftX(),
    () -> -driverController.getRightX()
).withDeadband(0.1)
 .withMaxTranslationalSpeed(MetersPerSecond.of(4.5))
 .withMaxRotationalSpeed(RadiansPerSecond.of(Math.PI * 2));

driveSubsystem.setDefaultCommand(
    driveSubsystem.driveFieldRelativeCommand(driverInput)
);
```

### 6. Seed azimuth encoders at robot init

{% hint style="warning" %}
Call `seedAzimuthEncoder()` on each swerve module during `robotInit()` — not in the subsystem constructor. The absolute encoder needs the robot to be powered and the CAN bus to be settled before the seed is reliable. Calling it too early can result in incorrect starting angles and the robot crabbing on enable.
{% endhint %}

```java
// In Robot.java robotInit():
driveSubsystem.swerveDrive.seedAzimuthEncoders();
```

---

## Examples

{% embed url="https://github.com/Yet-Another-Software-Suite/YAMS/tree/master/examples/swerve_drive" %}
Swerve Drive example
{% endembed %}

{% embed url="https://github.com/Yet-Another-Software-Suite/YAMS/tree/master/examples/swerve_drive_pathplanner" %}
Swerve Drive with PathPlanner autonomous
{% endembed %}

---

## Related pages

- [SwerveDrive](../api/java/swerve/swerve-drive.md)
- [SwerveInputStream](../api/java/swerve/swerve-input-stream.md)
- [SwerveDriveConfig](../api/java/swerve/swerve-drive-config.md)
- [Configuring a Motor Controller](configuring-a-motor.md)
- [Simulation](simulation.md)
