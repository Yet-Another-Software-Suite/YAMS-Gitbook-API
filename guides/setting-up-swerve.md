# Setting Up Swerve Drive

**Goal:** Create a fully functional `SwerveDrive` subsystem with field-relative driving, odometry, and simulation.

---

## Steps

### 1. Configure drive and azimuth motor controllers for each module

Each swerve module needs two `SmartMotorController` instances: one for drive (velocity) and one for azimuth (position). Follow [Configuring a Motor Controller](configuring-a-motor.md) for each.

Drive controllers use velocity PID and a `SimpleMotorFeedforward`. Azimuth controllers use position PID and optionally a `SimpleMotorFeedforward` for friction compensation.

### 2. Create a `SwerveModuleConfig` for each module

Pass the drive and azimuth motor controllers into the `SwerveModuleConfig` constructor. Then set the wheel radius, the module's location relative to the robot center, and the absolute encoder.

```java
SwerveModuleConfig frontLeftConfig = new SwerveModuleConfig(driveMotorFL, azimuthMotorFL)
    .withWheelRadius(Meters.of(0.0508))
    .withLocation(new Translation2d(0.381, 0.381))
    .withAbsoluteEncoderOffset(Rotations.of(0.24))
    .withTelemetry("FrontLeft", TelemetryVerbosity.HIGH);
```

Repeat for `FrontRight`, `BackLeft`, and `BackRight`, adjusting the `Translation2d` signs and encoder offsets.

### 3. Build `SwerveModule` instances and create a `SwerveDriveConfig`

Wrap each `SwerveModuleConfig` in a `SwerveModule`, then collect the four modules into a `SwerveDriveConfig`. Pass the subsystem and all four modules in the constructor, then set kinematic limits and the gyro supplier.

```java
SwerveModule frontLeft  = new SwerveModule(frontLeftConfig);
SwerveModule frontRight = new SwerveModule(frontRightConfig);
SwerveModule backLeft   = new SwerveModule(backLeftConfig);
SwerveModule backRight  = new SwerveModule(backRightConfig);

SwerveDriveConfig driveConfig = new SwerveDriveConfig(this, frontLeft, frontRight, backLeft, backRight)
    .withGyro(imu.getYaw().asSupplier())
    .withMaximumChassisSpeed(MetersPerSecond.of(4.5), DegreesPerSecond.of(360))
    .withTranslationController(new PIDController(1.0, 0, 0))
    .withRotationController(new PIDController(1.0, 0, 0))
    .withTelemetry(TelemetryVerbosity.HIGH);
```

### 4. Construct `SwerveDrive` and wire command factories

```java
public class DriveSubsystem extends SubsystemBase {

    private final SwerveDrive swerveDrive;

    public DriveSubsystem() {

        // --- Front Left ---
        SmartMotorControllerConfig driveConfigFL = new SmartMotorControllerConfig(this)
            .withIdleMode(MotorMode.BRAKE)
            .withGearing(new MechanismGearing(GearBox.fromRatio(6.75)))
            .withClosedLoopController(0.1, 0.0, 0.0)
            .withFeedforward(new SimpleMotorFeedforward(0.12, 2.2, 0.3))
            .withStatorCurrentLimit(Amps.of(60));

        SmartMotorControllerConfig azimuthConfigFL = new SmartMotorControllerConfig(this)
            .withIdleMode(MotorMode.Brake)
            .withGearing(new MechanismGearing(GearBox.fromRatio(12.8)))
            .withClosedLoopController(7.0, 0.0, 0.3)
            .withExternalEncoderDiscontinuityPoint(Rotations.of(0.5))
            .withStatorCurrentLimit(Amps.of(40));

        SmartMotorController driveMotorFL = new TalonFXWrapper(
            new TalonFX(10), DCMotor.getKrakenX60(1), driveConfigFL);
        SmartMotorController azimuthMotorFL = new TalonFXWrapper(
            new TalonFX(11), DCMotor.getKrakenX60(1), azimuthConfigFL);

        // Repeat for FR, BL, BR with their CAN IDs and encoder offsets...

        SwerveModuleConfig frontLeftModuleConfig = new SwerveModuleConfig(driveMotorFL, azimuthMotorFL)
            .withWheelRadius(Meters.of(0.0508))
            .withLocation(new Translation2d(0.381, 0.381))
            .withAbsoluteEncoderOffset(Rotations.of(0.24))
            .withTelemetry("FrontLeft", TelemetryVerbosity.HIGH);

        // SwerveModuleConfig frontRightModuleConfig = ...
        // SwerveModuleConfig backLeftModuleConfig  = ...
        // SwerveModuleConfig backRightModuleConfig = ...

        SwerveModule frontLeft = new SwerveModule(frontLeftModuleConfig);
        // SwerveModule frontRight = new SwerveModule(frontRightModuleConfig);
        // SwerveModule backLeft   = new SwerveModule(backLeftModuleConfig);
        // SwerveModule backRight  = new SwerveModule(backRightModuleConfig);

        SwerveDriveConfig driveConfig = new SwerveDriveConfig(this, frontLeft /*, frontRight, backLeft, backRight */)
            .withGyro(gyro.getYaw().asSupplier())
            .withMaximumChassisSpeed(MetersPerSecond.of(4.5), DegreesPerSecond.of(360))
            .withTranslationController(new PIDController(1.0, 0, 0))
            .withRotationController(new PIDController(1.0, 0, 0))
            .withTelemetry(TelemetryVerbosity.HIGH);

        swerveDrive = new SwerveDrive(driveConfig);
    }

    /** Field-relative drive command driven by a SwerveInputStream. */
    public Command driveFieldRelativeCommand(SwerveInputStream inputStream) {
        return run(() -> swerveDrive.setFieldRelativeChassisSpeeds(inputStream.get()))
            .withName("Field Oriented Drive");
    }

    /** Pass vision pose estimates to the odometry estimator. */
    public void addVisionMeasurement(Pose2d pose, double timestampSeconds) {
        swerveDrive.addVisionMeasurement(pose, timestampSeconds);
    }

    public Pose2d getPose() {
        return swerveDrive.getPose();
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

`SwerveInputStream` translates raw joystick axes into swerve chassis speeds, applying deadbands and rate limits. Pass the `SwerveDrive` instance as the first argument to `SwerveInputStream.of()`. Rotation is set separately via `.withControllerRotationAxis()`.

```java
SwerveInputStream driverInput = SwerveInputStream.of(
    swerveDrive,                          // SwerveDrive instance
    () -> -driverController.getLeftY(),   // forward/back
    () -> -driverController.getLeftX()    // strafe
).withControllerRotationAxis(() -> -driverController.getRightX())
 .withDeadband(0.1)
 .withMaximumLinearVelocity(MetersPerSecond.of(4.5))
 .withMaximumAngularVelocity(DegreesPerSecond.of(360));

driveSubsystem.setDefaultCommand(
    driveSubsystem.driveFieldRelativeCommand(driverInput)
);
```

### 6. Azimuth encoder seeding

`seedAzimuthEncoder()` is called automatically on each `SwerveModule` during construction. No additional call is required in `robotInit()`. If you need to re-seed after the CAN bus has fully settled, you can call `seedAzimuthEncoder()` on each `SwerveModule` instance individually.

---

## Examples

{% @github-files/github-code-block url="https://github.com/Yet-Another-Software-Suite/YAMS/blob/master/examples/swerve_drive/java/frc/robot/subsystems/SwerveSubsystem.java" %}

{% @github-files/github-code-block url="https://github.com/Yet-Another-Software-Suite/YAMS/blob/master/examples/swerve_drive_pathplanner/java/frc/robot/subsystems/SwerveSubsystem.java" %}

---

## Related pages

- [SwerveDrive](../api/java/swerve/swerve-drive.md)
- [SwerveInputStream](../api/java/swerve/swerve-input-stream.md)
- [SwerveDriveConfig](../api/java/swerve/swerve-drive-config.md)
- [Configuring a Motor Controller](configuring-a-motor.md)
- [Simulation](simulation.md)
